---
name: post-match-review-and-optimization
description: 赛后复盘方法论 — 对比预测vs实际结果，识别系统性偏差，迭代优化模型参数，提升下一轮预测准确率
---

# 赛后复盘与模型优化方法论

## 触发条件
在任何足球预测（北单让球SPF或竞足比分预测）完成并获取到实际比赛结果后，需要对模型进行复盘和参数优化时加载本技能。

## 核心原则
1. **不回测不前进去。不优化——每次预测后必须复盘**
2. **追踪准确率，更要追踪偏差模式** — 方向正确率 vs 比分精确率 vs GD偏差分布
3. **逐场分析，不要只看统计数字** — 数字告诉你错了，逐场告诉你为什么错
4. **修复一个偏差，不要引入新的——每次改完重新全量回测**

## 复盘流程

### 第一步：数据采集
从ESPN API获取比赛实际结果：

```python
# 获取指定日期的联赛比分
curl -s "https://site.api.espn.com/apis/site/v2/sports/soccer/{slug}/scoreboard?dates=YYYYMMDD"
```

联赛slug映射：
- 意甲: ita.1, 英超: eng.1, 西甲: esp.1, 葡超: por.1
- 英冠: eng.2, 丹超: den.1, 沙特: ksa.1, 俄超: rus.1
- 瑞典: swe.1, 荷甲: ned.1

### 第二步：逐场对比

对每场比赛记录：
| 字段 | 说明 |
|------|------|
| № | 场次编号 |
| 联赛 | 联赛名称 |
| 对阵 | 主队 vs 客队 |
| 盘口/让球 | 官方让球数（北单）或 0（竞足） |
| 旧预测 | 模型给出的预测（含首选方向/比分） |
| 实际结果 | 实际比分和方向 |
| 命中状态 | ✅或❌ |
| GD偏差 | 预期净胜球 - 实际净胜球（关键指标） |
| 偏差原因分析 | 根因分析 |

### 第三步：系统性偏差识别

#### 常见偏差模式

| 偏差类型 | 症状 | 典型根因 | 修复方案 |
|---------|------|---------|---------|
| **主场高估** | 主胜预测多但命中低 | 5月主场优势衰减，或模型HomeAdvantage权重过高 | 加MAY_HOME_COEFFICIENT（如0.88），降低MaradonaAura权重 |
| **平局保守** | 平局预测过多，实际有显著方向 | ±0.55判定窗口太宽，模型"不敢选方向" | 缩窄窗口至±0.35，让模型更果断 |
| **战意误判** | 排名靠队的预测完全反了 | urgency_v2/v3阈值设置错误，或无欲无求队给了太高战意 | 检查战意评分函数逻辑，用积分差替代排位 |
| **不败迷信** | UPM模型翻车，不败纪录队反被逼平 | 赛季末不败纪录=双刃剑，强队松懈 | 5月禁用UPM，改换RPM，加H2H修正 |
| **比分太相似** | 两个预测比分几乎一样，覆盖不了实际 | 两个比分从同一个GD导出，差异太小 | 改用映射表（不同GD区间对应不同比分对） |
| **极端比分漏掉** | 实际0-2但模型只预测1-1/0-0 | CPM等保守模型不覆盖极端客胜 | 在评分表中手动加入爆冷比分（如0-2/2-3） |
| **强弱颠倒** | 排名差>8的场次方向完全反了 | PPM中TableGap权重过重，忽略了客队战意 | 降低TableGap权重，增加AwayStability/StarPower |

#### 关键分析指标
```python
# 计算方向准确率
direction_ok = sum(1 for p in predictions if p.direction == p.actual) / total

# 计算比分精确率
score_ok = sum(1 for p in predictions if p.score in p.predicted_scores) / total

# 计算平均GD偏差（评估模型"偏度"）
avg_gd_bias = sum(p.expected_gd - p.actual_gd for p in predictions) / total
# 正值=模型高估主队，负值=模型低估主队

# 计算偏差分布
import statistics
gd_biases = [p.expected_gd - p.actual_gd for p in predictions]
mean_bias = statistics.mean(gd_biases)
stdev_bias = statistics.stdev(gd_biases)
# stdev大=模型不稳定，时准时不准
```

### 第四步：针对性修复

#### A. 模型选择器修复
- 赛季末英冠 → CPM（禁SFM）
- 赛季末葡超中游 → RPM（禁SFM）
- 赛季末高排名不败队 → 转RPM（禁UPM）

#### B. 权重参数调优
```python
# 修正公式
# SFM: FinalStretchPPG×0.25 + TargetUrgency×0.30 + OppositionStrength×0.25 + RestDays×0.10 + RefBias×0.10
# PPM: TableGap×0.10 + StarPower×0.30 + AwayStability×0.25 + PressingDifferential×0.20 + BenchDepth×0.15
# CPM: EndSeasonMomentum×0.35 + MidtableMeaningless×0.25 + HomeBottomDensity×0.20 + SetPieceDiff×0.15 + RefereeStyle×0.05
# RPM: RelegationFear×0.25 + HomeDesperation×0.15 + AwaySolidity×0.20 + AttackDeadlock×0.20 + GoalDiffTrend×0.10 + SubImpact×0.10
```

#### C. 比分映射表修复
核心原则：一个模型+不同GD区间输出**两个不同的比分**（主/备）。

关键技巧：
- 平局区间（GD接近0）必须是两种不同进球数的平局：1-1和0-0，或1-1和2-2
- 主胜区间必须覆盖不同进球差：2-1（小胜）和1-0（险胜），或2-0（完胜）和3-1（大胜）
- 特殊模型的爆冷处理：
  - HPM（0.15~0.7）要加2-3，因为排位靠前的主队可能松懈
  - CPM（-0.3~0.3）要加0-2，因为混沌模型客队可能爆冷
  - GPM（≤-2.5）要加1-4，因为再弱的队主场也可能进球

#### D. 战意评分修复 (urgency_v3)
```python
def urgency_v3(rank, pts, league_size, max_pts, remaining_matches):
    ppm = max_pts / (league_size * 2)
    rl = (league_size - 3) * ppm
    if rank <= 3 and max_pts - pts < 9: return 1.0, "争冠"
    elif pts < rl + 10: return 0.9, "保级"
    elif rank <= 8 and pts >= get_euro_line(league_size) - 5: return 0.8, "欧战"
    elif pts > get_euro_line(league_size) - 12 and remaining_matches >= 3: return 0.6, "理论欧战"
    elif pts > rl + 10 and pts < get_euro_line(league_size) - 12: return 0.1, "无欲无求"
    else: return 0.3, "中游保守"
```

#### E. 5月衰减系数
```python
MAY_HOME_COEFFICIENT = 0.88  # 主场打88折
MAY_SLACK = {
    "争冠": 0.90, "欧战": 0.85, "理论欧战": 0.80,
    "保级": 1.15, "无欲无求": 0.75, "中游保守": 0.90
}
```

### 第五步：全量回测验证

每次修改后必须：
1. **重新跑全量回测**（不要只测修过的那一场）
2. 对比新旧准确率：比分精确率 + 方向正确率
3. 确认没有引入倒退（💥原来对的现在不对了）
4. 如果倒退存在，调整方案缩小影响

```python
# 回测模板
old_correct = N  # 旧模型命中数
new_correct = M  # 新模型命中数
print(f"旧:{old_correct}/10 → 新:{new_correct}/10")
regressions = [m for m in matches if old_model[m]==✅ and new_model[m]==❌]
print(f"倒退场次: {len(regressions)}")
```

### 第六步：推送到GitHub

使用GitHub Contents API推送（当git push超时时的备选方案）：
- 更新 `sports/beijing-danchang-handicap-prediction/SKILL.md`（北单技能）
- 更新 `sports/beijing-danchang-prediction/SKILL.md`（竞足技能）
- 同步到 `skills/` 目录

## 经验教训

1. **方向准确率≠比分准确率** — 两个要分别追踪，优化方向的方法和优化比分的方法不同
2. **盲目保守是最大的陷阱** — 平局多=准确率高只是假象。缩窄判定窗口后短期准确率可能下降，但长期预测更有区分度
3. **一分耕耘不一定一分收获** — 排名#3倒灶输#8这种场次100场里可能只有1-2场，为了覆盖它而牺牲其他场次的准确率不划算。但5月季节性和英冠Chaos是系统性偏差，必须修
4. **爆冷不是无迹可寻** — "保级客战鸡血"已验证通过（006 米尔沃0-2赫尔城），下次预测可直接使用此玄学因子
5. **H2H历史权重不可忽视** — 布拉加历史克制本菲卡（本菲卡不败记录下仍被2-2逼平），这个信息藏在历史数据里但不是常规排名能反映的
