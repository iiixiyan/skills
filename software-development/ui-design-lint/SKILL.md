     1|---
     2|name: ui-design-lint
     3|description: Web UI设计规范检查 — 对比目标页面与DESIGN.md设计规范，基于CSS Token精确对比，输出分级整改报告，生成修正后的HTML页面
     4|category: software-development
     5|version: 2.0.0
     6|---
     7|
     8|# UI Design Lint — 设计规范检查与整改
     9|
    10|## 触发条件
    11|需要对网页进行UI设计规范检查时加载：
    12|- 检查已有页面是否符合团队设计规范（`references/DESIGN.md`）
    13|- 输出分级整改报告（阻断/建议/信息）
    14|- 生成修正后的HTML页面（CSS变量Token化 + 内联修复注释）
    15|
    16|## ⛔ 强制约束
    17|1. **严禁使用脚本** — 所有检查由AI通过浏览器工具完成
    18|2. **Token锚定** — 检查时对比的是 DESIGN.md §0 中定义的 CSS 变量 Token，不是模糊的"颜色像不像"
    19|3. **逐项不跳过** — 按 §1→§10 顺序逐条对比，跳过项需标注原因
    20|4. **严重度分级** — 每项标注 🔴阻断/🟡建议/🔵信息
    21|
    22|---
    23|
    24|## 内置设计规范
    25|
    26|设计规范文件：`references/DESIGN.md`（v4.0）
    27|
    28|### 规范章节速查
    29|
    30|| 章节 | 内容 | Token数 | 阻断项数 |
    31||------|------|---------|----------|
    32|| §0 | CSS变量Token体系 | 55 | — |
    33|| §1 | 颜色系统 | 18 | 2 |
    34|| §2 | 字体系统 | 7层级 | 2 |
    35|| §3 | 间距系统 | 12场景 | 5 |
    36|| §4 | 圆角系统 | 6级 | 2 |
    37|| §5 | 阴影系统 | 5级 | 2 |
    38|| §6 | 过渡动画 | 3场景 | 0 |
    39|| §7 | 标准页面布局模板 | — | 架构级 |
    40|| §8 | 组件规范(11个) | — | 组件级 |
    41|| §9 | 响应式断点 | 5级 | 0 |
    42|| §10 | 可访问性 | 4项 | 2 |
    43|
    44|---
    45|
    46|## 三步检查流程
    47|
    48|### 步骤1：加载规范 + 捕获目标页面
    49|
    50|**1a. 加载 DESIGN.md**
    51|加载 `references/DESIGN.md`，牢记 §0 Token 表（55个Token的精确值）。
    52|
    53|**1b. 捕获页面（三管齐下）**
    54|
    55|① **结构快照**：`browser_snapshot(full=true)` — 获取完整DOM结构
    56|② **样式提取**：`browser_console` 提取CSS变量和计算样式：
    57|```javascript
    58|JSON.stringify({
    59|  // Token 匹配
    60|  tokens: {
    61|    primary: getComputedStyle(document.documentElement).getPropertyValue('--color-primary').trim(),
    62|    textPrimary: getComputedStyle(document.documentElement).getPropertyValue('--color-text-primary').trim(),
    63|    fontSans: getComputedStyle(document.documentElement).getPropertyValue('--font-sans').trim(),
    64|    sidebarWidth: getComputedStyle(document.documentElement).getPropertyValue('--spacing-sidebar-width').trim(),
    65|    cardRadius: getComputedStyle(document.documentElement).getPropertyValue('--radius-2xl').trim(),
    66|    // ... 提取所有55个Token
    67|  },
    68|  // 关键元素计算样式
    69|  elements: {
    70|    body: { font: getComputedStyle(document.body).fontFamily, size: getComputedStyle(document.body).fontSize, bg: getComputedStyle(document.body).backgroundColor },
    71|    h1: document.querySelector('h1') ? { size: getComputedStyle(document.querySelector('h1')).fontSize, weight: getComputedStyle(document.querySelector('h1')).fontWeight } : null,
    72|    card: document.querySelector('.card,[class*=card]') ? { bg: getComputedStyle(document.querySelector('.card,[class*=card]')).backgroundColor, radius: getComputedStyle(document.querySelector('.card,[class*=card]')).borderRadius } : null,
    73|  }
    74|})
    75|```
    76|
    77|③ **视觉截图**：`browser_vision` — 获取颜色、间距、对齐等视觉信息
    78|
    79|**1c. 快速诊断：Token存在性检查**
    80|如果 `--color-primary` 返回空字符串 → **页面未使用Token体系 → 整个 §1-§5 自动标为 🔴**
    81|
    82|---
    83|
    84|### 步骤2：逐条对比（§1→§10）
    85|
    86|对每个章节执行以下对比：
    87|
    88|**§1 颜色** — 对比每一个Token值是否与规范一致（精确HEX匹配）
    89|```
    90|规范: --color-primary = #2563EB
    91|页面: --color-primary = "#1a56db" → ❌ 值不匹配 → 🔴阻断
    92|页面: --color-primary = ""         → ❌ Token缺失 → 🔴阻断
    93|页面: --color-primary = "#2563EB"  → ✅
    94|```
    95|
    96|**§2 字体** — 检查 font-family 回退链完整性 + 字号层级值
    97|**§3 间距** — 检查 4px 倍数规则 + 关键场景值
    98|**§4-5 圆角/阴影** — Token值精确对比
    99|**§6 过渡** — 检查按钮/链接/输入框是否有 transition
   100|**§7 布局** — browser_vision + snapshot 对比标准布局模板
   101|**§8 组件** — 对页面中存在的每个组件类型逐项检查
   102|**§9 响应式** — 调整浏览器窗口宽度检查断点行为
   103|**§10 可访问性** — 对比度计算 + focus状态 + label关联
   104|
   105|---
   106|
   107|### 步骤3：生成报告 + 修正页面
   108|
   109|#### 3a. 整改报告（分级输出）
   110|
   111|```markdown
   112|## 🔍 UI Design Lint 报告
   113|
   114|**目标页面**：{URL}
   115|**检查时间**：{ISO时间}
   116|**规范版本**：DESIGN.md v4.0
   117|
   118|---
   119|
   120|### 📊 总体评估
   121|
   122|| Token匹配率 | 阻断项 | 建议项 | 信息项 | 合规率 |
   123||------------|--------|--------|--------|--------|
   124|| {X}/{55} | {N} | {N} | {N} | {X}% |
   125|
   126|---
   127|
   128|### 🔴 阻断项（必须修复）
   129|
   130|| # | 规范条款 | 规范值 | 实际值 | 修复方案 |
   131||---|---------|--------|--------|---------|
   132|| 1 | §0 --color-primary | #2563EB | #1a56db | 替换为规范值 |
   133|
   134|---
   135|
   136|### 🟡 建议项（强烈推荐修复）
   137|
   138|| # | 规范条款 | 规范值 | 实际值 | 修复方案 |
   139||---|---------|--------|--------|---------|
   140|
   141|---
   142|
   143|### 🔵 信息项（可优化）
   144|
   145|| # | 规范条款 | 说明 |
   146||---|---------|------|
   147|
   148|---
   149|
   150|### Token 对照表
   151|
   152|| Token | 规范值 | 实际值 | 状态 |
   153||-------|--------|--------|------|
   154|| --color-primary | #2563EB | #2563EB | ✅ |
   155|| --font-size-h1 | 28px | 26px | ⚠️ |
   156|| ... | ... | ... | ... |
   157|```
   158|
   159|#### 3b. 修正HTML页面
   160|
   161|生成完整的独立HTML文件：
   162|```html
   163|<!DOCTYPE html>
   164|<html lang="zh-CN">
   165|<head>
   166|  <meta charset="UTF-8">
   167|  <meta name="viewport" content="width=device-width, initial-scale=1.0">
   168|  <title>{原标题}-fixed</title>
   169|  <style>
   170|    /* ══════════════════════════════════════
   171|       DESIGN.md v4.0 Token System
   172|       自动生成于 {时间}
   173|       ══════════════════════════════════════ */
   174|    :root {
   175|      /* FIX: §0 - 补充完整的55个CSS变量Token */
   176|      --color-primary: #2563EB;
   177|      /* ... 所有Token */
   178|    }
   179|    
   180|    /* FIX: §2.2 - H1字号从26px修正为28px */
   181|    h1 { font-size: var(--font-size-h1); }
   182|  </style>
   183|</head>
   184|<body>
   185|  <!-- 保留原页面内容，样式全部Token化 -->
   186|  <!-- FIX: §8.5 - 卡片圆角从8px修正为16px -->
   187|</body>
   188|</html>
   189|```
   190|
   191|**修正原则**：
   192|- 内容/功能不变，只修样式
   193|- CSS全部内联 + Token化
   194|- 每个修改处标注 `<!-- FIX: §X 条款 -->`
   195|- 文件名：`{原页面}-fixed.html`
   196|
   197|---
   198|
   199|## 特殊场景处理
   200|
   201|### Token体系完全缺失
   202|如果页面未使用任何CSS变量 → 报告直接标注"❌ 未接入Token体系"，所有颜色/间距/圆角/阴影检查项全部 🔴阻断，并生成带完整Token的修正HTML。
   203|
   204|### 部分Token匹配但值不对
   205|逐项记录偏差值，生成对照表。
   206|
   207|### 页面使用不同Token名
   208|如果页面用了 `--primary` 而非 `--color-primary` → 🟡建议：统一Token命名。
   209|
   210|### 响应式未实现
   211|如果页面在 xs/sm 断点下布局错乱 → 🟡建议实现响应式适配。
   212|
   213|---
   214|
   215|## 版本记录
   216|- v2.0.0 (2026-05-15)：对齐 DESIGN.md v4.0，增加Token精确对比体系、严重度分级、表格/表单/图标/Badge组件规范、过渡动画规范
   217|- v1.0.0 (2026-05-15)：初始版本，三步检查流程
   218|