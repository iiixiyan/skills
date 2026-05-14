---
name: 59itou-data-fetch
description: 从59itou.com获取北单/竞足比赛列表与详情 — 列表提取matchID → 逐场获取9个Tab完整数据
category: sports
---

# 59itou 比赛数据获取（北单 + 竞足）

## 触发条件
需要从59itou.com获取北单或竞足比赛列表及详情数据时加载。

## ⛔ 强制规则：严禁总结
获取详情页数据时，必须原样展示 `document.body.innerText` 的完整原始内容。禁止任何形式的总结、归纳、格式化、省略、简化。看到什么展示什么，一字不改。

## ⚠️ 重要
该网站使用 **Vant UI** 框架，所有数据通过JS动态渲染。curl/requests无法获取数据，必须使用 `browser_navigate` + `browser_console`。

---

## 北单（北京单场）

### 列表页
```
https://kt.59itou.com/883/danchang/
```
从 `.item[id]` 元素获取matchID：
```javascript
var items = document.querySelectorAll('.item[id]');
items.forEach(function(item) {
    var id = item.id;          // matchID
    var text = item.textContent; // 场次/联赛/主客队/让球/SP
});
```

### 详情页
```
https://kt.59itou.com/39/match3/?matchid={matchid}&lotteryId=45&lottery_style=dc
```

---

## 竞足（竞彩足球）

### 列表页
```
https://kt.59itou.com/192/jingcai/
```
⚠️ 竞足列表页没有matchID（DOM无 `.item[id]`），优先从北单列表获取matchID后复用。

### 竞足matchID获取（两步兜底）
**优先**：北单列表 `.item[id]` + 球队名交叉匹配。

**兜底**：竞足独有的比赛（如沙职不在北单列表），点击该场"分析"文字，页面跳转后从URL提取matchID：
```javascript
// N = 0-indexed 比赛序号
var item = document.querySelectorAll('.matchitem')[N];
var allSpans = item.querySelectorAll('span, p, div');
allSpans.forEach(function(el) {
    if (el.textContent.trim() === '分析') el.click();
});
// 跳转后: window.location.href → matchid=XXXXXXX
```
⚠️ 点击后可能跳转到不同路径（如 `/175/match3/` 或 `/456/match3/`），matchID在URL参数中，路径前缀不重要。

### 详情页
URL格式：`/379/match3/?matchid={matchid}&lotteryId=90&lottery_style=jczq`（路径前缀可能变化如/175/或/456/，matchID在参数中，前缀不重要）

```
https://kt.59itou.com/379/match3/?matchid={matchid}&lotteryId=90&lottery_style=jczq
```

---

## 详情页 Tab 结构（北单和竞足完全相同）

9个Tab，索引：

| 索引 | Tab名 | 内容 | 状态 |
|:---:|:----|:-----|:----:|
| 0 | 直播 | （默认，同战绩） | ✅ |
| 1 | 直播 | （重复） | - |
| 2 | 阵容 | 首发/替补/身价/阵型/技术统计/伤停 | ✅ |
| 3 | 战绩 | 近10场/主客战绩/H2H交锋/近期赛程/综合实力 | ✅ |
| 4 | 欧指 | 36家公司赔率表/概率转换/常见比分/指数变化 | ✅ |
| 5 | 亚指 | 盘口水位/赢盘率/盘口走势 | ✅ |
| 6 | 情报 | 付费内容 | 🔒 |
| 7 | 推荐 | 牛人推荐 | ⚠️ 常有空 |
| 8 | 排名 | 联赛完整积分榜/球队统计 | ✅ |
| 9 | 盈亏 | 必发交易量/盈亏/冷热指数（部分付费） | ⚠️ |

### Tab切换方法
```javascript
// ⚠️ 不能用 browser_click 或 dispatchEvent（Vant UI 下均不可靠）
// 必须用原生 .click() + 按文本匹配
var tabs = document.querySelectorAll('.van-tab');
for (var i = 0; i < tabs.length; i++) {
    if (tabs[i].textContent.includes('欧指')) {   // 目标Tab名称
        tabs[i].click();
        break;
    }
}
// 等待 1-2秒后读取（用 terminal sleep 1 而非 setTimeout）
```
**等待与验证**：JS端click后，用 `terminal sleep 1 && echo done` 等待渲染，再用 `browser_console` 获取 `document.body.innerText`。验证：检查结果是否含目标Tab关键词（如'概率转换'→欧指，'必发交易盈亏'→盈亏）。若不匹配，重新执行click+等待。

**为什么dispatchEvent/browser_click不可靠**：此站Vant UI的van-tab组件只响应原生click事件，不响应合成MouseEvent或程序化ref点击。直接调用DOM元素的`.click()`是唯一可靠方式。

### 提取数据
```javascript
document.body.innerText  // 获取当前Tab全部文本
// 截取关键段落减少token消耗:
document.body.innerText.substring(startIdx, endIdx)
// 定位关键词: t.indexOf('概率转换'), t.indexOf('必发交易盈亏')
```

---

## 参数速查

| 参数 | 北单 | 竞足 |
|:---|:---|:---|
| 列表路径 | `/883/danchang/` | `/192/jingcai/` |
| 详情路径 | `/39/match3/` | `/379/match3/` |
| lotteryId | `45` | `90` |
| lottery_style | `dc` | `jczq` |
| matchID来源 | `.item[id]` | 北单共用；独有比赛点"分析"获取 |

## 注意事项
1. 必须使用 `browser_navigate` + `browser_console`，curl无法获取
2. Tab切换后等待 **800ms-1s** 让内容渲染
3. 竞足列表无matchID，通过北单列表获取后共用
4. 情报Tab(6)为付费内容，推荐Tab(7)常为空
5. 盈亏Tab(9)部分数据需付费订阅
6. 列表页日期分组从 `document.body.innerText` 中提取（如"5月14日 周四 共24场"）
7. ⚠️ **队名翻译不一致**：竞足列表页与详情页可能使用不同中文译名（例：列表页"胡巴卡德"=详情页"卡迪西亚"；"拉斯决心"="哈森姆"；"巴伦西亚"="瓦伦西亚"）。交叉匹配时优先对联赛+排名，而非队名。
