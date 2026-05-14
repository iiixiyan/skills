---
name: ui-design-lint
description: Web UI设计规范检查 — 对比目标页面与DESIGN.md设计规范，基于CSS Token精确对比，输出分级整改报告，生成修正后的HTML页面
category: software-development
version: 2.0.0
---

# UI Design Lint — 设计规范检查与整改

## 触发条件
需要对网页进行UI设计规范检查时加载：
- 检查已有页面是否符合团队设计规范（`references/DESIGN.md`）
- 输出分级整改报告（阻断/建议/信息）
- 生成修正后的HTML页面（CSS变量Token化 + 内联修复注释）

## ⛔ 强制约束
1. **严禁使用脚本** — 所有检查由AI通过浏览器工具完成
2. **Token锚定** — 检查时对比的是 DESIGN.md §0 中定义的 CSS 变量 Token，不是模糊的"颜色像不像"
3. **逐项不跳过** — 按 §1→§10 顺序逐条对比，跳过项需标注原因
4. **严重度分级** — 每项标注 🔴阻断/🟡建议/🔵信息

---

## 内置设计规范

设计规范文件：`references/DESIGN.md`（v4.0）

### 规范章节速查

| 章节 | 内容 | Token数 | 阻断项数 |
|------|------|---------|----------|
| §0 | CSS变量Token体系 | 55 | — |
| §1 | 颜色系统 | 18 | 2 |
| §2 | 字体系统 | 7层级 | 2 |
| §3 | 间距系统 | 12场景 | 5 |
| §4 | 圆角系统 | 6级 | 2 |
| §5 | 阴影系统 | 5级 | 2 |
| §6 | 过渡动画 | 3场景 | 0 |
| §7 | 标准页面布局模板 | — | 架构级 |
| §8 | 组件规范(11个) | — | 组件级 |
| §9 | 响应式断点 | 5级 | 0 |
| §10 | 可访问性 | 4项 | 2 |

---

## 三步检查流程

### 步骤1：加载规范 + 捕获目标页面

**1a. 加载 DESIGN.md**
加载 `references/DESIGN.md`，牢记 §0 Token 表（55个Token的精确值）。

**1b. 捕获页面（三管齐下）**

① **结构快照**：`browser_snapshot(full=true)` — 获取完整DOM结构
② **样式提取**：`browser_console` 提取CSS变量和计算样式：
```javascript
JSON.stringify({
  // Token 匹配
  tokens: {
    primary: getComputedStyle(document.documentElement).getPropertyValue('--color-primary').trim(),
    textPrimary: getComputedStyle(document.documentElement).getPropertyValue('--color-text-primary').trim(),
    fontSans: getComputedStyle(document.documentElement).getPropertyValue('--font-sans').trim(),
    sidebarWidth: getComputedStyle(document.documentElement).getPropertyValue('--spacing-sidebar-width').trim(),
    cardRadius: getComputedStyle(document.documentElement).getPropertyValue('--radius-2xl').trim(),
    // ... 提取所有55个Token
  },
  // 关键元素计算样式
  elements: {
    body: { font: getComputedStyle(document.body).fontFamily, size: getComputedStyle(document.body).fontSize, bg: getComputedStyle(document.body).backgroundColor },
    h1: document.querySelector('h1') ? { size: getComputedStyle(document.querySelector('h1')).fontSize, weight: getComputedStyle(document.querySelector('h1')).fontWeight } : null,
    card: document.querySelector('.card,[class*=card]') ? { bg: getComputedStyle(document.querySelector('.card,[class*=card]')).backgroundColor, radius: getComputedStyle(document.querySelector('.card,[class*=card]')).borderRadius } : null,
  }
})
```

③ **视觉截图**：`browser_vision` — 获取颜色、间距、对齐等视觉信息

**1c. 快速诊断：Token存在性检查**
如果 `--color-primary` 返回空字符串 → **页面未使用Token体系 → 整个 §1-§5 自动标为 🔴**

---

### 步骤2：逐条对比（§1→§10）

对每个章节执行以下对比：

**§1 颜色** — 对比每一个Token值是否与规范一致（精确HEX匹配）
```
规范: --color-primary = #2563EB
页面: --color-primary = "#1a56db" → ❌ 值不匹配 → 🔴阻断
页面: --color-primary = ""         → ❌ Token缺失 → 🔴阻断
页面: --color-primary = "#2563EB"  → ✅
```

**§2 字体** — 检查 font-family 回退链完整性 + 字号层级值
**§3 间距** — 检查 4px 倍数规则 + 关键场景值
**§4-5 圆角/阴影** — Token值精确对比
**§6 过渡** — 检查按钮/链接/输入框是否有 transition
**§7 布局** — browser_vision + snapshot 对比标准布局模板
**§8 组件** — 对页面中存在的每个组件类型逐项检查
**§9 响应式** — 调整浏览器窗口宽度检查断点行为
**§10 可访问性** — 对比度计算 + focus状态 + label关联

---

### 步骤3：生成报告 + 修正页面

#### 3a. 整改报告（分级输出）

```markdown
## 🔍 UI Design Lint 报告

**目标页面**：{URL}
**检查时间**：{ISO时间}
**规范版本**：DESIGN.md v4.0

---

### 📊 总体评估

| Token匹配率 | 阻断项 | 建议项 | 信息项 | 合规率 |
|------------|--------|--------|--------|--------|
| {X}/{55} | {N} | {N} | {N} | {X}% |

---

### 🔴 阻断项（必须修复）

| # | 规范条款 | 规范值 | 实际值 | 修复方案 |
|---|---------|--------|--------|---------|
| 1 | §0 --color-primary | #2563EB | #1a56db | 替换为规范值 |

---

### 🟡 建议项（强烈推荐修复）

| # | 规范条款 | 规范值 | 实际值 | 修复方案 |
|---|---------|--------|--------|---------|

---

### 🔵 信息项（可优化）

| # | 规范条款 | 说明 |
|---|---------|------|

---

### Token 对照表

| Token | 规范值 | 实际值 | 状态 |
|-------|--------|--------|------|
| --color-primary | #2563EB | #2563EB | ✅ |
| --font-size-h1 | 28px | 26px | ⚠️ |
| ... | ... | ... | ... |
```

#### 3b. 修正HTML页面

生成完整的独立HTML文件：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{原标题}-fixed</title>
  <style>
    /* ══════════════════════════════════════
       DESIGN.md v4.0 Token System
       自动生成于 {时间}
       ══════════════════════════════════════ */
    :root {
      /* FIX: §0 - 补充完整的55个CSS变量Token */
      --color-primary: #2563EB;
      /* ... 所有Token */
    }
    
    /* FIX: §2.2 - H1字号从26px修正为28px */
    h1 { font-size: var(--font-size-h1); }
  </style>
</head>
<body>
  <!-- 保留原页面内容，样式全部Token化 -->
  <!-- FIX: §8.5 - 卡片圆角从8px修正为16px -->
</body>
</html>
```

**修正原则**：
- 内容/功能不变，只修样式
- CSS全部内联 + Token化
- 每个修改处标注 `<!-- FIX: §X 条款 -->`
- 文件名：`{原页面}-fixed.html`

---

## 特殊场景处理

### Token体系完全缺失
如果页面未使用任何CSS变量 → 报告直接标注"❌ 未接入Token体系"，所有颜色/间距/圆角/阴影检查项全部 🔴阻断，并生成带完整Token的修正HTML。

### 部分Token匹配但值不对
逐项记录偏差值，生成对照表。

### 页面使用不同Token名
如果页面用了 `--primary` 而非 `--color-primary` → 🟡建议：统一Token命名。

### 响应式未实现
如果页面在 xs/sm 断点下布局错乱 → 🟡建议实现响应式适配。

---

## 版本记录
- v2.0.0 (2026-05-15)：对齐 DESIGN.md v4.0，增加Token精确对比体系、严重度分级、表格/表单/图标/Badge组件规范、过渡动画规范
- v1.0.0 (2026-05-15)：初始版本，三步检查流程
