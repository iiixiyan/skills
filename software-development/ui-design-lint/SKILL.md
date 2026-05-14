---
name: ui-design-lint
description: Web UI设计规范检查 — 对比目标页面与DESIGN.md设计规范，输出整改报告，生成修正后的HTML页面
category: software-development
version: 1.0.0
---

# UI Design Lint — 设计规范检查与整改

## 触发条件
需要对网页进行UI设计规范检查时加载。适用于：
- 检查已有页面是否符合团队设计规范
- 输出结构化整改报告
- 生成修正后的HTML页面

## ⛔ 强制约束
1. **严禁使用脚本** — 所有检查由AI通过浏览器工具（browser_navigate/browser_snapshot/browser_console）视觉分析完成
2. **规范锚定** — 所有判断必须引用 DESIGN.md 具体条目，不得主观发挥
3. **逐项检查** — 按 DESIGN.md 章节顺序逐条对比，不得跳过
4. **报告结构化** — 必须包含：违规项、严重程度、规范依据、整改方案、修正代码

---

## 内置设计规范

设计规范文件位于 `references/DESIGN.md`，启动时自动加载。

---

## 检查流程（三步法）

### 步骤1：加载规范 + 捕获页面

**1a. 加载 DESIGN.md**
```
读取 references/DESIGN.md 获取完整设计规范
```

**1b. 捕获目标页面**
- 使用 `browser_navigate` 打开目标URL
- 使用 `browser_snapshot(full=true)` 获取完整页面结构
- 使用 `browser_console` 提取关键样式信息：
  ```javascript
  // 获取页面CSS变量和关键样式
  JSON.stringify({
    bodyFont: getComputedStyle(document.body).fontFamily,
    bodySize: getComputedStyle(document.body).fontSize,
    colors: {
      primary: getComputedStyle(document.documentElement).getPropertyValue('--primary-color'),
      bg: getComputedStyle(document.body).backgroundColor,
    },
    spacing: {
      bodyPadding: getComputedStyle(document.body).padding,
    }
  })
  ```
- 使用 `browser_vision` 截图获取视觉排版信息

### 步骤2：逐条对比

按 DESIGN.md 章节顺序，逐条检查：

| 检查维度 | 对比方式 |
|----------|----------|
| 色彩/主题 | browser_console 提取CSS变量 vs DESIGN.md 色板 |
| 字体/排版 | browser_snapshot 文本层级 + console 字体属性 vs DESIGN.md 字体规范 |
| 间距/布局 | browser_vision 截图分析 + snapshot 结构 vs DESIGN.md 栅格/间距 |
| 组件样式 | browser_snapshot 组件识别 vs DESIGN.md 组件规范 |
| 交互状态 | browser_click 触发交互 + browser_snapshot 检查响应 vs DESIGN.md 状态规范 |

每次对比记录：
- ✅ 符合项
- ⚠️ 偏差项（轻微）
- ❌ 违规项（严重）

### 步骤3：生成报告 + 修正页面

输出两部分：
1. **整改报告**（Markdown表格）
2. **修正后的HTML页面**（完整可运行的HTML文件）

---

## 输出格式

### 整改报告

```markdown
## 🔍 UI设计规范检查报告

**目标页面**：{URL}
**检查时间**：{时间}
**规范版本**：DESIGN.md v{X}

---

### 📊 总体评估

| 检查项总数 | 通过 | 偏差 | 违规 | 合规率 |
|-----------|------|------|------|--------|
| {N} | {n} | {n} | {n} | {X}% |

---

### ❌ 违规项

| # | 规范章节 | 规范要求 | 实际情况 | 严重程度 | 整改方案 |
|---|---------|---------|---------|---------|---------|
| 1 | {章节} | {要求} | {实际} | 🔴严重/🟡中等 | {方案} |

### ⚠️ 偏差项

| # | 规范章节 | 规范要求 | 实际情况 | 偏差说明 |
|---|---------|---------|---------|---------|

### ✅ 符合项

| # | 规范章节 | 规范要求 | 验证结果 |
|---|---------|---------|---------|
```

### 修正HTML输出

生成完整的HTML文件，包含：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>{修正后标题}</title>
  <style>
    /* 所有样式严格遵循 DESIGN.md 规范 */
  </style>
</head>
<body>
  <!-- 修正后的HTML结构 -->
</body>
</html>
```

保存为 `{原页面名称}-fixed.html`

修正原则：
- 保留原页面功能与内容不变
- 仅修改样式/布局以符合DESIGN.md
- CSS全部内联（避免外部依赖）
- 在修改处添加 `<!-- FIX: {规范条目} -->` 注释
- 每个修改点标注对应的DESIGN.md条款

---

## DESIGN.md 规范文件结构（模板）

```markdown
# UI设计规范 v1.0

## 1. 色彩系统
- 主色: #XXXXXX
- 辅色: #XXXXXX
- ...

## 2. 字体排版
- 正文字体: ...
- 字号层级: ...

## 3. 间距与布局
- 栅格系统: ...
- 组件间距: ...

## 4. 组件规范
- 按钮: ...
- 表单: ...
- ...

## 5. 交互状态
- hover: ...
- active: ...
- ...
```
