---
name: beijing-danchang-handicap-prediction
description: 北京单场让球胜平负(让球SPF)专用预测 — 基于实时官方数据，独立预测让球后的胜平负结果，严禁从让球数反推
---

# 北京单场让球胜平负预测

## 核心原则（必须遵守）

0. **每场比赛必须从多个数据源同时获取数据** — 不依赖单一数据源，并行拉取所有可用数据源后进行交叉验证和综合分析
1. **根据比赛特征动态选择预测模型** — 分析联赛类型、排名差、赛季阶段、球队状态等特征后，选择最适合的模型，而非固定用一个模型
2. **预测目标是让球后的胜平负结果** — 即官方SP值对应的让球SPF，不是普通胜平负
3. **严禁从让球数反推预测** — 禁止"因为让球X所以主队难赢"这类逻辑，预测必须基于球队自身实力分析
4. **所有数据必须实时获取** — 禁止使用训练数据截止前的内部知识，必须通过API或爬虫获取最新数据
5. **让球数仅供验证参考** — 在独立分析完成后，用让球数来检验预测是否合理，而非用让球数来推导

## 数据源 — 并行多源获取（必须）

### 核心策略：每场比赛同时从多个数据源拉取，交叉验证

```python
# 并行获取模板 — 每场比赛调此函数
import requests, json, concurrent.futures

def fetch_match_data(match_info):
    """
    并行获取所有可用数据源的数据
    返回融合后的数据结构
    """
    home = match_info["home"]
    away = match_info["away"]
    league = match_info["league"]
    date = match_info["date"]
    
    results = {}
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=6) as executor:
        futures = {
            executor.submit(get_espn, league, date, home, away): "espn",
            executor.submit(get_sofascore, league, date, home, away): "sofascore",
            executor.submit(get_500_com, date, match_info.get("match_id")): "lottery",
            executor.submit(get_weather, match_info.get("city", "")): "weather",
        }
        
        for future in concurrent.futures.as_completed(futures):
            source = futures[future]
            try:
                results[source] = future.result(timeout=15)
            except Exception as e:
                results[source] = {"error": str(e), "available": False}
    
    return fuse_data(results)  # 交叉验证融合
```

### 数据源清单

#### 1️⃣ ESPN API — 战绩、O/U盘口（主流联赛）
```python
LEAGUE_SLUGS = {
    "沙特": "ksa.1", "瑞典": "swe.1", "意甲": "ita.1", "英超": "eng.1",
    "英冠": "eng.2", "西甲": "esp.1", "葡超": "por.1", "俄超": "rus.1",
    "荷甲": "ned.1", "丹超": "den.1", "罗甲": "rou.1", "波兰甲": "pol.1",
    "挪威": "nor.1"
}
```

#### 2️⃣ SofaScore API — 全球全覆盖（已验证罗甲/波兰甲/瑞士/挪威）
```python
LEAGUE_SOFA = {
    "波兰甲": {"tournament": 202, "season": 76477},
    "罗甲":    {"tournament": 11428, "season": 80923},
    "瑞士甲":  {"tournament": 216, "season": 77150},
    "挪威":    {"tournament": 20, "season": 87809},
    "瑞典":    {"tournament": 24, "season": 87808},
    "丹超":    {"tournament": 23, "season": 87810},
    "荷甲":    {"tournament": 7, "season": 72433},
}
```

#### 3️⃣ 中国彩票源 — 官方让球数/SP
- 500彩票网: `https://odds.500.com/api/zqdc/?date=YYYY-MM-DD`
- 备份: Soccerway / 球探网 qiutan.com

#### 4️⃣ 天气API — 户外比赛辅助
```python
# openweathermap: lat/lon + date → 降雨/风速/温度
# 雨雪天 → 进球偏少，防守方占优
# 大风 → 传中成功率下降，地面配合有利
```

#### 5️⃣ 战意评估（无需API，基于联赛阶段判断）
- 赛季末段（4-6月）关注：争冠/保级/欧战资格/无欲无求
- 从积分榜排名 + 剩余轮次判断
- 从SofaScore获取各队近5场积分趋势

### 数据融合（fuse_data）规则

当多个数据源对同一指标给出不同值时：
| 情况 | 处理策略 |
|------|---------|
| ESPN+SofaScore 一致 | 置信采纳 |
| 仅一个源有数据 | 标注"单一来源"，置信度降1星 |
| 数据冲突 | 取主流数据源优先，次源作为偏离参考 |
| 全部缺失 | 用最近5场平均补缺，标注"数据插补" |

### 实战已验证数据源成功率
| 数据源 | 覆盖率 | 成功率 | 适用场景 |
|-------|-------|-------|---------|
| **Sofascore** | 全球全覆盖 | 100% | ✅ 首选，所有联赛第一站 |
| **ESPN** | 主流联赛 | ~70% | 网络不稳定，作为Sofascore补充 |
| 500彩票网 | 中国彩市 | 需验证 | 获取官方让球数和SP |
| Soccerway | 全球联赛 | 低 | 最后备选 |

## 动态模型选择器 — 按比赛特征选模型

### 选择逻辑（每场比赛重新评估）

```python
def select_model(match_info, fused_data):
    """
    根据比赛特征动态选择最佳预测模型
    """
    league = match_info["league"]
    rank_diff = fused_data.get("rank_diff", 0)  # 排名差
    season_phase = get_season_phase(league)       # season: 季初/季中/季末
    form_volatility = fused_data.get("form_volatility", 0)  # 近期状态波动
    home_advantage = fused_data.get("home_advantage_score", 0)  # 主场优势分
    is_cup_match = match_info.get("is_cup", False)
    
    # === 模型选择决策树 ===
    
    # 赛季末段（最后6轮）
    if season_phase == "END":
        return "SFM"  # Seasonal Form Momentum — 侧重战意
    
    # 强弱分明（排名差 > 8位）
    if abs(rank_diff) > 8:
        return "PPM"  # Power Projection — 实力投射
    
    # 主场强势 vs 客场弱势
    if home_advantage > 7.0:
        return "HPM"  # Home Pulse — 主场脉冲
    
    # 状态波动大（两队近期W-D-L交替）
    if form_volatility > 0.6:
        return "FFM"  # Form Flame — 状态火焰
    
    # 保级区对话（排名都在下游）
    if is_relegation_derby(fused_data):
        return "RPM"  # Relegation Pressure
    
    # 中游无欲无求对话
    if is_midtable_meaningless(fused_data):
        return "CPM"  # Chaos Prediction
    
    # 杯赛（单场淘汰制）
    if is_cup_match:
        return "HDM"  # Handicap Differential
    
    # 默认（中游对话，数据完整）
    return "RDM"  # Real Differential
```

### 模型选择速查表
| 比赛特征 | 推荐模型 | 侧重点 |
|---------|---------|-------|
| 赛季末最后6轮 | SFM | 战意+冲刺状态 |
| 排名差>8位 | PPM | 实力差距投射 |
| 主场优势分高 | HPM | 主场脉冲效应 |
| 状态波动大 | FFM | 近期火焰状态 |
| 保级区对话 | RPM | 保级压力 |
| 中游无欲无求 | CPM | 混沌均势 |
| 杯赛淘汰制 | HDM | 让球差分 |
| 数据完整、中游 | RDM | 纯实力差值 |
| 连胜/不败纪录 | UPM | 不败金身压力 |
| 动荡球队（换帅/内讧） | TMM | 动荡匹配 |

## 自创预测模型体系（让球SPF专用）— 10大模型

### 模型1: HDM（Handicap Differential Model）— 适合杯赛/淘汰赛
让球差分模型，侧重近期交战和关键比赛表现。

```python
HDI = FormGap×0.25 + GoalDiffRate×0.20 + HomeAwayGap×0.20 + H2H_HandicapHistory×0.20 + MomentumCrv×0.15
```

- FormGap: (主队近5场积分 - 客队近5场积分) / 5，归一化到-10~+10
- GoalDiffRate: 主队场均净胜球 - 客队场均净胜球，修正主场优势+0.35
- HomeAwayGap: 主队主场场均净胜球 - 客队客场场均净胜球
- H2H_HandicapHistory: 历史同让球数下的结果统计
- MomentumCrv: 赛季走势曲线斜率（末程冲刺/保级/无欲无求修正）

**输出判断**: HDI ≥ 让球数+0.5 → 让球主胜 | HDI ≤ 让球数-0.5 → 让球客胜 | 中间 → 让球平

### 模型2: RDM（Real Differential Model）— 默认通用型
纯实力差值评估，完全不碰让球数。

```python
RDI = AttackStrength×0.30 + DefenseSolidity×0.25 + SetPieceDiff×0.15 + StaminaIndex×0.15 + SquadDepth×0.15
```

1. 先独立算出预期净胜球差（正=主队净胜）
2. 再以此为基准判断让球胜平负
3. **禁止**: 不允许先看让球数再调参数

### 模型3: SFM（Seasonal Form Momentum）— 赛季末专用
侧重战意和走势，赛季末最后6轮最佳。

```python
SFI = FinalStretchPPG×0.30 + TargetUrgency×0.25 + OppositionStrength×0.20 + RestDays×0.15 + RefBias×0.10
```

- FinalStretchPPG: 最后6轮场均积分
- TargetUrgency: 争冠/保级/欧战资格/无欲无求的战意加权
- OppositionStrength: 对手同期表现的反向加权

### 模型4: PPM（Power Projection Model）— 强弱分明专用
排名差≥8位时使用，侧重实力差距。

```python
PPI = TableGap×0.15 + StarPower×0.25 + AwayStability×0.20 + PressingDifferential×0.25 + BenchDepth×0.15
```

- TableGap: 积分榜差距
- StarPower: 核心球员个人能力差值
- AwayStability: 客队客场稳定性
- PressingDifferential: 高位逼抢效率差
- BenchDepth: 板凳深度

### 模型5: HPM（Home Pulse Model）— 主场强势专用
主场优势明显（主场场均得分联赛前三）时使用。

```python
HPI = MaradonaAura×0.30 + Top4Motivation×0.20 + MidtableResistance×0.20 + GoalDiffPace×0.15 + RotationImpact×0.15
```

- MaradonaAura: 主场特殊氛围加成（魔鬼主场）
- Top4Motivation: 争冠/欧战动力
- MidtableResistance: 中游球队主场韧性
- GoalDiffPace: 主客场净胜球差距
- RotationImpact: 轮换影响

### 模型6: FFM（Form Flame Model）— 状态波动大专用
两队近期状态（W-D-L记录）差异明显时使用。

```python
FFI = WinStreak×0.25 + H2H_Dominance×0.20 + HomeShield×0.25 + OppMomentum×0.15 + WeekdayFactor×0.15
```

### 模型7: CPM（Chaos Prediction Model）— 中游无欲无求专用
赛季末中游混战、双方都无明确战意时使用。

```python
CPI = EndSeasonMomentum×0.30 + MidtableMeaningless×0.20 + HomeBottomDensity×0.25 + SetPieceDiff×0.15 + RefereeStyle×0.10
```

### 模型8: RPM（Relegation Pressure Model）— 保级区专用
一方或双方都在降级区边缘时使用。

```python
RPI = RelegationFear×0.30 + AwaySolidity×0.25 + AttackDeadlock×0.20 + GoalDiffTrend×0.15 + SubImpact×0.10
```

### 模型9: UPM（Unbeaten Pressure Model）— 不败纪录专用
某队携连胜/不败纪录出战，存在纪录压力。

```python
UPI = InvinciblePressure×0.25 + ChampionEdge×0.20 + HomeDominance×0.25 + HistoricalWeight×0.15 + MilestoneMatch×0.15
```

### 模型10: TMM（Turmoil Match Model）— 动荡球队专用
球队近期有换帅/内讧/关键球员离队等动荡事件。

```python
TMI = FanPressure×0.20 + ManagerCrisis×0.25 + RelegationUrgency×0.25 + LondonDerby×0.15 + HeadCoachRecord×0.15
```

## 模型组合策略

每次预测不仅输出首选模型结果，还输出备选模型（1-2个不同类别模型的交叉验证）：

```python
def aggregate_predictions(match_info, fused_data):
    """
    多模型交叉验证，输出最终预测
    """
    # 1. 选取主模型
    primary = select_model(match_info, fused_data)
    result_p = run_model(primary, fused_data)
    
    # 2. 选取1-2个备选（不同类别的）
    alternatives = select_alternatives(primary, match_info)
    results_a = [run_model(alt, fused_data) for alt in alternatives]
    
    # 3. 投票决策
    all_results = [result_p] + results_a
    final = majority_vote_with_confidence(all_results)
    
    # 4. 如果模型间分歧大（2/3 vs 1/3），降1星置信度
    # 如果模型一致，升1星
    final.confidence = adjust_confidence(final, all_results)
    
    return final
```

## 预测输出模板

```
【编号】联赛：主队 vs 客队（让球数：X）
├─ 实时数据：主队 W-D-L / 客队 W-D-L / O/U
├─ 实力评估：（模型评分、预期净胜球、关键因子得分）
├─ 独立判断：让球胜 / 让球平 / 让球负
├─ 置信度：★★★☆☆
└─ 备注：（简短理由，3-4句话）
```

## 球队中文名映射（补充）
| 中文 | 英文(ESPN) | 备注 |
|-----|-----------|------|
| 阿克伦 | Akron Tolyatti | 俄超 |
| 布雷达 | NAC Breda | 荷甲 |
| 海伦芬 | Heerenveen | 荷甲 |
| 梅塔洛 | Metaloglobus | 罗甲(ESPN可能无数据) |
| 赫曼施 | Hermannstadt | 罗甲 |
| 兰讷斯 | Randers FC | 丹超 |
| 欧登塞 | Odense Boldklub | 丹超 |
| 克拉科 | Cracovia | 波兰甲(ESPN可能无数据) |
| 拉多米 | Radomiak | 波兰甲 |
| 奥尔格 | Örgryte IS | 瑞典超 |
| 布赖合 | Al Taawoun | 沙特 |
| 新未来 | Neom SC | 沙特 |
| 利雅青 | Al Shabab | 沙特 |
| 里斯 | Sporting CP | 葡超 |
| 里奥阿 | Rio Ave | 葡超 |

## 数据待确认处理
- 罗甲/波兰甲等ESPN未收录联赛：标注"数据待确认"，使用可获取的替代数据源
- 英冠战绩记录可能为空：使用近5场战绩（可从其他API获取）
- O/U盘口缺失：不影响让球胜平负预测

## 注意事项
1. **严禁行为**：先看让球数再编造理由（"因为让2球所以..."），预测必须从比赛本身出发
2. 让球数只作为最终验证工具，不作为分析起点
3. 实时数据获取失败时，如实标注，不得编造
4. 置信度≤3星时，明确提示风险
5. 所有玄学模型不得用于让球SPF预测（玄学仅限于比分娱乐预测）
