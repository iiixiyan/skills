---
name: beijing-danchang-prediction
description: 北京单场/竞足赛前预测系统 — 自创量化模型 + 玄学因子双轨分析，逐场输出模型比分与玄学比分
---

# 北京单场/竞足赛前预测系统

## 触发条件
当用户提供北京单场（北单）或竞足比赛清单（含场次编号、主客队、开赛时间），且要求做预测分析时，加载本技能。

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
    with concurrent.futures.ThreadPoolExecutor(max_workers=6) as executor:
        futures = {
            executor.submit(get_espn, ...): "espn",
            executor.submit(get_sofascore, ...): "sofascore",
            executor.submit(get_500_com, ...): "lottery",
        }
        results = {}
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
```

#### 5️⃣ 战意评估（无需API，基于联赛阶段判断）
- 赛季末段（4-6月）关注：争冠/保级/欧战资格/无欲无求

### 数据融合规则
| 情况 | 处理策略 |
|------|---------|
| 多个数据源一致 | 置信采纳 |
| 仅一个源有数据 | 标注"单一来源"，置信度降1星 |
| 全部缺失 | 用最近5场平均补缺，标注"数据插补" |

## 自创量化模型体系

### 1. IPI 综合指数模型（中游对话）
`IPI = FormRating×0.30 + xGDiff×0.20 + GoalConv×0.20 + DefResil×0.15 + Fatigue×0.15`

### 2. FFM 状态火焰模型（状态差异明显）
`FFI = WinStreak×0.25 + H2H_Dominance×0.20 + HomeShield×0.25 + OppMomentum×0.15 + WeekdayFactor×0.15`

### 3. PPM 实力投射模型（强弱分明）
`PPI = TableGap×0.15 + StarPower×0.25 + AwayStability×0.20 + PressingDifferential×0.25 + BenchDepth×0.15`

### 4. HPM 主场脉冲模型（主场强势）
`HPI = MaradonaAura×0.30 + Top4Motivation×0.20 + MidtableResistance×0.20 + GoalDiffPace×0.15 + RotationImpact×0.15`

### 5. TMM 动荡匹配模型（波动大比赛）
`TMI = FanPressure×0.20 + ManagerCrisis×0.25 + RelegationUrgency×0.25 + LondonDerby×0.15 + HeadCoachRecord×0.15`

### 6. CPM 英冠混沌模型（赛季末垃圾对话）
`CPI = EndSeasonMomentum×0.30 + MidtableMeaningless×0.20 + HomeBottomDensity×0.25 + SetPieceDiff×0.15 + RefereeStyle×0.10`

### 7. LPM 西甲定位模型（西甲中下游）
`LPI = HomeFortress×0.25 + CatalanCurse×0.20 + RelegationGap×0.20 + TechnicalDiff×0.15 + HeadCoachTactics×0.20`

### 8. GPM 葡超差距模型（强弱悬殊）
`GPI = BudgetGap×0.30 + TitleRace×0.25 + HomeWeakness×0.20 + PhysicalDiff×0.15 + RefBias×0.10`

### 9. RPM 保级压力模型（保级队对阵）
`RPI = RelegationFear×0.30 + AwaySolidity×0.25 + AttackDeadlock×0.20 + GoalDiffTrend×0.15 + SubImpact×0.10`

### 10. UPM 不败金身模型（不败纪录球队）
`UPI = InvinciblePressure×0.25 + ChampionEdge×0.20 + HomeDominance×0.25 + HistoricalWeight×0.15 + MilestoneMatch×0.15`

## 玄学预测体系

### 玄学因子库
- **球衣颜色定律**: 特定颜色组合历史胜率偏差
- **日期数字玄学**: 特定日期历史结果复刻
- **主场特殊条件**: 新草皮/新灯光/更衣室更换/球场谢幕
- **连胜连败阈值**: 连胜N场后必有平局临界值
- **特定月份定律**: 5月主场不败/升班马崩盘等季节性规律
- **裁判偏好**: 主队哨/大球哨/点球哨
- **球员个人玄学**: 生日出场/换发型/剃光头/球衣号码
- **历史复刻周期**: 完整年周期后的结果对称
- **颜色相克**: 红克加泰/蓝克绿/白克红等规律
- **命名定律**: 反City/反B字头/反London等
- **里程碑比赛**: 教练百场/门将生日/裁判百场等仪式性反弹
- **联赛季节规律**: 斋月后首轮/赛季末收官战/保级冲刺期

### 每场输出2个玄学比分，每个附带3-5条具体依据

## 球队中文名映射表
| 中文 | 英文(ESPN) | 缩写 |
|-----|-----------|------|
| 新未来 | Neom SC | NEOM |
| 利雅青 | Al Shabab | SHA |
| 天狼星 | IK Sirius | SIR |
| 厄尔格 | Örgryte IS | ÖRG |
| 布赖合 | Al Taawoun | TAA |
| 吉达国 | Al Ahli | AHL |
| 那不勒 | Napoli | NAP |
| 博洛尼 | Bologna | BOL |
| 热刺 | Tottenham Hotspur | TOT |
| 利兹联 | Leeds United | LEE |
| 米尔沃 | Millwall | MIL |
| 赫尔城 | Hull City | HUL |
| 巴列卡 | Rayo Vallecano | RAY |
| 赫罗纳 | Girona | GIR |
| 里奥阿 | Rio Ave | RAV |
| 里斯 | Sporting CP | SCP |
| 阿马多 | Estrela | EST |
| 法马利 | FC Famalicao | FCF |
| 本菲卡 | Benfica | SLB |
| 布拉加 | Braga | SCB |

## 注意事项
- ESPN可能超时，准备curl备用方案
- 英冠记录字段可能为空，标注数据待确认
- 玄学依据必须附带具体数据，禁止空洞表述
