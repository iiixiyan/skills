---
name: beijing-danchang-handicap-prediction
description: 北京单场让球胜平负(让球SPF)专用预测 — 基于实时官方数据，独立预测让球后的胜平负结果，严禁从让球数反推
---

# 北京单场让球胜平负预测

## 核心原则（必须遵守）

1. **预测目标是让球后的胜平负结果** — 即官方SP值对应的让球SPF，不是普通胜平负
2. **严禁从让球数反推预测** — 禁止"因为让球X所以主队难赢"这类逻辑，预测必须基于球队自身实力分析
3. **所有数据必须实时获取** — 禁止使用训练数据截止前的内部知识，必须通过API或爬虫获取最新数据
4. **让球数仅供验证参考** — 在独立分析完成后，用让球数来检验预测是否合理，而非用让球数来推导

## 数据源（多层次架构）

### 第一层：官方数据源（优先级最高）

```python
# 1. 中国竞彩网官方（SP值、让球数、开奖结果）
# 北单开奖SP值查询
# URL: http://www.sporttery.cn/ 或 https://www.sporttery.cn/
# 北单专区: http://www.bjlot.com/

# 2. 500彩票网（实时赔率、盘口走势、让球数）
# 北京单场页面: https://odds.500.com/fenxi/zqdc-1.shtml
# 具体赛事: https://odds.500.com/fenxi/zqdc-{match_id}.shtml
# API接口: https://odds.500.com/api/zqdc/?date=YYYY-MM-DD
```

### 第二层：国际数据API

```python
import requests, json
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"}

# ========= ESPN API（战绩、O/U盘口）=========
LEAGUE_SLUGS = {
    "沙特": "ksa.1", "瑞典": "swe.1", "意甲": "ita.1",
    "英超": "eng.1", "英冠": "eng.2", "西甲": "esp.1",
    "葡超": "por.1", "俄超": "rus.1", "荷甲": "ned.1",
    "丹超": "den.1", "罗甲": "rou.1", "波兰甲": "pol.1",
}

def get_espn_data(league, date):
    slug = LEAGUE_SLUGS.get(league)
    if not slug: return None
    url = f"https://site.api.espn.com/apis/site/v2/sports/soccer/{slug}/scoreboard?dates={date}"
    r = requests.get(url, headers=headers, timeout=15)
    # 返回: 主客队名、W-D-L记录、O/U盘口

# ========= API-Football (api-football.com / RapidAPI) =========
# 需注册免费key，覆盖全球100+联赛
# 端点: https://v3.football.api-sports.io/
# 获取赛程: GET /fixtures?date=2026-05-11&league=...
# 获取球队统计: GET /teams/statistics?fixture=...
# 获取赔率: GET /odds?fixture=...
# 联赛ID: Serie A=135, Premier League=39, La Liga=140, 
#          Primeira Liga=94, Allsvenskan=113, Superliga=119,
#          Saudi League=307, Romanian Liga I=283,
#          Ekstraklasa=106, Swiss Challenge League=551

# ========= SofaScore API（实时数据、xG、近期状态）=========
# 获取联赛赛事: https://api.sofascore.com/api/v1/unique-tournament/{tournament_id}/season/{season_id}/events/round/{round_id}
# 获取球队详情: https://api.sofascore.com/api/v1/team/{team_id}
# 获取赛事统计: https://api.sofascore.com/api/v1/event/{event_id}/statistics
# 获取伤病信息: https://api.sofascore.com/api/v1/team/{team_id}/injuries
```

### 第三层：Web Scraping（ESPN/API无法覆盖的联赛）

```python
# ========= Flashscore（罗甲、波兰甲、瑞士甲等小众联赛）=========
# 赛程页面: https://www.flashscore.com/football/romania/liga-1/
# 球队页面: https://www.flashscore.com/team/{team-slug}/
# 注意: Flashscore需处理Cloudflare反爬

# ========= Transfermarkt（身价、伤病、历史交锋）=========
# 球队页面: https://www.transfermarkt.com/{team}/startseite/verein/{id}
# 伤病页面: https://www.transfermarkt.com/{team}/verletzungen/verein/{id}
# 注意: Transfermarkt有反爬限制，需设置合理间隔

# ========= Soccerway（全部联赛覆盖，包括瑞士、挪威等）=========
# 赛程: https://int.soccerway.com/matches/YYYY/MM/DD/
# 球队: https://int.soccerway.com/teams/{country}/{team}/
# 优点: 覆盖全球200+联赛，HTML结构简单易解析

# ========= 球探网/雪球网（中文数据，国内可访问）=========
# 球探网: http://www.qiutan.com/
# 雪球网: https://www.zqiu.com/
```

### 第四层：伤病/战意/场外数据

```python
# ========= Premier Injuries（英超伤病）=========
# URL: https://www.premierinjuries.com/

# ========= Transfermarkt 伤病页 =========
# URL: https://www.transfermarkt.com/{team}/verletzungen/verein/{id}

# ========= 球队官方社媒/新闻（战意评估）=========
# 赛前发布会: 教练言论、关键球员状态
# 赛季末段需关注: 保级/争冠/欧战资格/已保级无欲无求

# ========= 天气数据（户外比赛）=========
# API: https://api.openweathermap.org/data/2.5/weather
# 雨雪天气影响总进球数10-15%，大风影响传中成功率
```

### 数据获取优先级策略

```
1. 先命中500彩票网 → 获取北单官方让球数和SP参考
2. 再查ESPN API → 获取战绩和O/U盘口
3. ESPN查不到 → 查Soccerway（罗甲/波兰甲/瑞士甲/挪超等）
4. Soccerway查不到 → 查Transfermarkt
5. 所有API都失败 → 标注"数据待确认"，不编造
```

## 自创预测模型体系（让球SPF专用）

### 模型1: HDM（Handicap Differential Model）
评估两队实力差是否足以覆盖让球数。

```
HDI = FormGap×0.25 + GoalDiffRate×0.20 + HomeAwayGap×0.20 + H2H_HandicapHistory×0.20 + MomentumCrv×0.15
```

- FormGap: (主队近5场积分 - 客队近5场积分) / 5，归一化到-10~+10
- GoalDiffRate: 主队场均净胜球 - 客队场均净胜球，修正主场优势+0.35
- HomeAwayGap: 主队主场场均净胜球 - 客队客场场均净胜球
- H2H_HandicapHistory: 历史同让球数下的结果统计
- MomentumCrv: 赛季走势曲线斜率（末程冲刺/保级/无欲无求修正）

**输出判断**:
- HDI ≥ 让球数+0.5 → 让球主胜
- HDI ≤ 让球数-0.5 → 让球客胜
- 中间区间 → 让球平

### 模型2: RDM（Real Differential Model）
纯实力差值评估，完全不碰让球数，独立求出净胜球预期后对照让球线。

```
RDI = AttackStrength×0.30 + DefenseSolidity×0.25 + SetPieceDiff×0.15 + StaminaIndex×0.15 + SquadDepth×0.15
```

1. 先独立算出预期净胜球差（正=主队净胜）
2. 再以此为基准判断让球胜平负
3. **禁止步骤**：不允许先看让球数再调参数

### 模型3: SFM（Seasonal Form Momentum Model）
赛季末段专用，侧重战意和走势。

```
SFI = FinalStretchPPG×0.30 + TargetUrgency×0.25 + OppositionStrength×0.20 + RestDays×0.15 + RefBias×0.10
```

- FinalStretchPPG: 最后6轮场均积分
- TargetUrgency: 争冠/保级/欧战资格/无欲无求的战意加权
- OppositionStrength: 对手同期表现的反向加权

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
