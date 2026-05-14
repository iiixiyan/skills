---
name: ui-design-lint
description: Web UI设计规范检查 — 对比目标页面与DESIGN.md设计规范，基于CSS Token精确对比，输出分级整改报告与修正HTML，支持CI/CD集成
category: software-development
version: 2.1.0
---

# UI Design Lint — 设计规范检查与整改

## 触发条件
需要对网页进行UI设计规范检查时加载：
- 检查已有页面是否符合团队设计规范（`references/DESIGN.md`）
- 输出分级整改报告（阻断/建议/信息）及 JSON 格式（CI/CD 可用）
- 生成修正后的HTML页面（CSS变量Token化 + 内联修复注释）

---

## ⛔ 约束与边界

### 脚本使用策略（精确表述）
- **允许**：通过 `browser_console` 执行诊断 JS 提取 CSS Token 和计算样式（这是浏览器工具的内置能力）
- **禁止**：引入外部脚本文件（如 axe-core）、在页面中注入第三方库、运行本地 Python/Node 脚本来做检查
- **降级策略**：若页面 CSP 限制导致 `browser_console` JS 执行失败 → 自动退化为「纯快照+视觉检查」模式，Token 对比改为视觉近似判断（对比度误差 ±15%），并在报告顶部标注 ⚠️ CSP 降级

### 输入前置条件
| 条件 | 说明 |
|------|------|
| 必须有可访问的 URL | 或可在当前浏览器环境打开的本地路径（`file://`） |
| 页面需登录 | 用户需在当前浏览器会话中**已完成登录**（技能不处理认证流程） |
| 不支持仅设计稿 | Figma/Sketch 设计稿不在本技能范围（未来可扩展独立技能） |
| 页面需完整渲染 | 纯 SSR 骨架屏/加载中状态会误报，需等页面完全加载 |

---

## 内置设计规范

设计规范文件：`references/DESIGN.md`（v4.0）

### 规范章节速查

| 章节 | 内容 | Token数 | 阻断项数 | 检查方式 |
|------|------|---------|----------|----------|
| §0 | CSS变量Token体系 | 55 | — | JS提取精确对比 |
| §1 | 颜色系统 | 18 | 2 | Token精确HEX匹配 |
| §2 | 字体系统 | 7层级 | 2 | JS提取+结构快照 |
| §3 | 间距系统 | 12场景 | 5 | JS提取+4px倍数规则 |
| §4 | 圆角系统 | 6级 | 2 | Token精确匹配 |
| §5 | 阴影系统 | 5级 | 2 | Token精确匹配 |
| §6 | 过渡动画 | 3场景 | 0 | JS检测transition属性 |
| §7 | 标准页面布局模板 | — | 架构级 | 结构快照+视觉截图 |
| §8 | 组件规范(11个) | — | 组件级 | 结构快照+JS样式提取 |
| §9 | 响应式断点 | 5级 | 0 | 调整窗口宽度验证 |
| §10 | 可访问性 | 4项 | 2 | JS对比度计算+结构检查 |

---

## 适配你的设计规范

本技能内置的 `DESIGN.md` 是**示例规范**，团队可按以下方式适配：

### 快速适配三步
1. **替换 Token 表**：编辑 `references/DESIGN.md` 的 §0，改为你的 CSS 变量名和值
2. **调整章节结构**：增删 §1–§10 中的章节（如不需要响应式可删除 §9）
3. **更新组件规范**：§8 中替换为你们实际使用的组件（按钮/卡片/表格等）

### 不改的部分
- 三步检查流程（加载→对比→报告）对所有规范通用
- 严重度分级体系（🔴🟡🔵）通用
- 输出格式（Markdown报告+JSON+修正HTML）通用
- Token 存在性检查逻辑通用

### 规范耦合度说明
- **§0 Token 表**：强依赖（检查的锚点，必须存在）
- **§1–§10 章节**：弱依赖（可按需增删，检查流程自适应）
- **组件规范**：参考性（检查时只对比页面中实际存在的组件）

---

## 三步检查流程

### 步骤1：加载规范 + 捕获目标页面

**1a. 加载 DESIGN.md**
加载 `references/DESIGN.md`，牢记 §0 Token 表（55个Token的精确值）。

**1b. 捕获页面（三管齐下）**

① **结构快照**：`browser_snapshot(full=true)` — 获取完整DOM结构

② **样式提取**：`browser_console` 执行诊断 JS：
```javascript
JSON.stringify({
  tokens: {
    primary: getComputedStyle(document.documentElement).getPropertyValue('--color-primary').trim(),
    textPrimary: getComputedStyle(document.documentElement).getPropertyValue('--color-text-primary').trim(),
    fontSans: getComputedStyle(document.documentElement).getPropertyValue('--font-sans').trim(),
    sidebarWidth: getComputedStyle(document.documentElement).getPropertyValue('--spacing-sidebar-width').trim(),
    cardRadius: getComputedStyle(document.documentElement).getPropertyValue('--radius-2xl').trim(),
    // ... 提取所有55个Token
  },
  elements: {
    body: { font: getComputedStyle(document.body).fontFamily, size: getComputedStyle(document.body).fontSize },
    h1: document.querySelector('h1') ? { size: getComputedStyle(document.querySelector('h1')).fontSize } : null,
    buttons: [...document.querySelectorAll('button,[role=button],.btn,a[href]')].slice(0,10).map(el => ({
      tag: el.tagName, height: getComputedStyle(el).height, radius: getComputedStyle(el).borderRadius,
      hasTransition: getComputedStyle(el).transition !== 'all 0s ease 0s'
    })),
    inputs: [...document.querySelectorAll('input,textarea,select')].slice(0,5).map(el => ({
      tag: el.tagName, height: getComputedStyle(el).height, hasLabel: !!el.closest('label') || !!document.querySelector(`[for="${el.id}"]`)
    })),
    cards: [...document.querySelectorAll('[class*=card],.card')].slice(0,5).map(el => ({
      radius: getComputedStyle(el).borderRadius, padding: getComputedStyle(el).padding,
      hasShadow: getComputedStyle(el).boxShadow !== 'none'
    }))
  },
  layout: {
    sidebar: (() => { const el = document.querySelector('[class*=sidebar],.sidebar,nav'); return el ? {width: getComputedStyle(el).width, position: getComputedStyle(el).position} : null; })(),
    topbar: (() => { const el = document.querySelector('[class*=topbar],.topbar,header'); return el ? {height: getComputedStyle(el).height, position: getComputedStyle(el).position} : null; })(),
  }
})
```

③ **视觉截图**：`browser_vision` — 获取颜色、间距、对齐等视觉信息

**1c. 快速诊断：Token存在性检查**
如果 `--color-primary` 返回空字符串 → **页面未使用Token体系 → 整个 §1-§5 自动标为 🔴**

**1d. CSP 降级检查**
如果步骤 1b 的 JS 执行被 CSP 拦截（返回空或报错）→ 启用降级模式：
- Token 对比退化为视觉近似判断（对比度 ±15%）
- 间距退化为像素估算（基于截图比例）
- 在报告顶部增加：`⚠️ CSP 降级模式 — JS 提取失败，以下结果为视觉估算，精度 ±15%`

---

### 步骤2：逐条对比（§1→§10）

对每个章节执行以下对比。跳过某项需标注原因。

#### §1 颜色 — Token值精确HEX匹配
```
规范: --color-primary = #2563EB
页面: "#1a56db" → ❌ 值不匹配 → 🔴阻断
页面: ""         → ❌ Token缺失 → 🔴阻断  
页面: "#2563EB"  → ✅
```

#### §2 字体 — font-family回退链 + 字号层级值
检查项：font-family 是否含完整回退链（macOS→Win→Linux→通用）| 字号层级值偏差 ≤2px

#### §3 间距 — 4px倍数 + 关键场景值
检查项：所有 margin/padding/gap 是否为 4px 整数倍 | 侧边栏宽度 252px±4px | 卡片内边距 24px±4px

#### §4-5 圆角/阴影 — Token精确对比
检查项：圆角值是否匹配 Token | 阴影值是否匹配 Token

#### §6 过渡 — transition存在性
检查项：按钮/链接/输入框是否有 `transition` 属性 | 时长是否在 150-300ms 范围

#### §7 布局 — 结构+视觉对比标准模板（详细判定规则）

**对比基准**：DESIGN.md §7 标准页面布局模板

**判定规则**（逐项检查）：
| # | 检查项 | 判定标准 | 严重度 |
|---|--------|---------|--------|
| 1 | 侧边栏存在性 | 页面中存在 `position:fixed`、`width≈252px`、`inset:0 auto 0 0` 的元素 | 🔴阻断 |
| 2 | 侧边栏宽度 | 计算宽度 = 252px ± 4px | 🔴阻断 |
| 3 | 侧边栏背景 | 深色渐变（起 #0F172A 终 #111827）或接近深色 | 🟡建议 |
| 4 | 顶部导航高度 | 68px ± 2px | 🔴阻断 |
| 5 | 顶部导航定位 | `position:sticky; top:0` | 🟡建议 |
| 6 | 顶部导航背景 | 白色 #FFFFFF + 底部边框 1px solid #E5E7EB | 🟡建议 |
| 7 | 主内容区 margin-left | = 侧边栏宽度（252px） | 🔴阻断 |
| 8 | 主内容区 padding | 24px ± 4px | 🟡建议 |
| 9 | 主内容区背景 | #F5F7FB | 🟡建议 |

**判定方式**：
- 检查项 1-3：从步骤1b的 `layout.sidebar` 数据判断
- 检查项 4-6：从步骤1b的 `layout.topbar` 数据判断
- 检查项 7-9：从 `browser_snapshot` 结构 + `browser_vision` 截图判断

#### §8 组件 — 逐组件样式对比（详细判定规则）

仅检查页面中**实际存在**的组件类型。每个组件的检测规则：

| 组件 | 关键判定点（CSS属性精确对比） | 严重度 |
|------|---------------------------|--------|
| 侧边栏 | width=252px±4px, background含深色渐变, color≈#D0D5DD | 🔴阻断 |
| 导航项 | height=42px±2px, border-radius=10px±2px, active背景≈rgba(37,99,235,0.18) | 🔴阻断 |
| 顶栏 | height=68px±2px, background=#FFF, border-bottom存在 | 🔴阻断 |
| 主按钮 | height=40px±2px, border-radius=8px±2px, bg≈#2563EB | 🔴阻断 |
| 次按钮 | height=40px±2px, border-radius=8px±2px, border存在, bg=#FFF | 🟡建议 |
| 大卡片 | border-radius=16px±2px, padding=24px±4px, bg=#FFF, 有阴影 | 🔴阻断 |
| 输入框 | height=40px±2px, border-radius=8px, border=1px solid #E5E7EB | 🟡建议 |
| 表格 | th height=44px, td height=52px, border-bottom存在 | 🟡建议 |
| 表单 | label存在且关联input, margin-bottom=20px | 🟡建议 |
| 头像 | width=height=32px, border-radius=50% | 🟡建议 |
| Badge | border-radius=9999px, 语义背景色匹配 | 🟡建议 |

#### §9 响应式 — 断点行为验证（详细判定规则）

**必须测试的断点**：xs(<640px) / sm(640-767px) / md(768-1023px) / lg(1024-1279px)

**判定规则**：
| # | 检查场景 | 判定标准 | 严重度 |
|---|---------|---------|--------|
| 1 | xs 断点侧边栏 | 隐藏或变为汉堡菜单（抽屉式） | 🔴阻断 |
| 2 | xs 断点卡片 | 单列布局，无横向溢出 | 🔴阻断 |
| 3 | xs 断点留白 | 页面边缘 16px ± 4px | 🟡建议 |
| 4 | sm/md 断点 | 侧边栏可收缩为图标或保持252px | 🟡建议 |
| 5 | 所有断点 | 无横向滚动条（overflow-x: hidden 或有对应处理） | 🔴阻断 |
| 6 | 触控目标 | xs 断点下可点击元素 ≥ 44px × 44px | 🟡建议 |

**测试方式**：用 `browser_console` 调整 viewport 尺寸后执行 `browser_snapshot` + `browser_vision`

#### §10 可访问性 — WCAG 2.1 AA（详细判定规则）

**遵循标准**：WCAG 2.1 AA

**判定规则**：
| # | 检查项 | 判定标准 | 检查方式 | 严重度 |
|---|--------|---------|---------|--------|
| 1 | 正文对比度 | 正文(<18px)与背景 ≥ 4.5:1 | JS计算或视觉估算 | 🔴阻断 |
| 2 | 大文本对比度 | 大文本(≥18px bold)与背景 ≥ 3:1 | JS计算或视觉估算 | 🔴阻断 |
| 3 | Focus可见性 | `*:focus-visible` 有可见轮廓或阴影 | `browser_press` Tab键后截图 | 🔴阻断 |
| 4 | 表单label关联 | 所有input有 `<label>` 且 `for` 属性匹配 | 结构快照检查 | 🔴阻断 |
| 5 | 图片alt | 所有 `<img>` 有非空 `alt` 属性 | 结构快照检查 | 🟡建议 |
| 6 | 页面标题 | 存在 `<h1>` | 结构快照检查 | 🟡建议 |
| 7 | 跳过快照 | 存在 `Skip to content` 链接 | 结构快照检查 | 🔵信息 |

**对比度计算**：优先用 JS 提取 `getComputedStyle` 的 color/backgroundColor 后计算 WCAG 对比度比值；若 CSP 降级则用截图色值近似估算。

**关于 axe-core**：不引入外部库，所有可访问性检查通过浏览器原生 API 完成。

---

### 步骤3：生成报告 + 修正页面 + 集成输出

#### 3a. 整改报告（Markdown 人类可读 + JSON 机器可读）

**Markdown 报告**：
```markdown
## 🔍 UI Design Lint 报告

**目标页面**：{URL}
**检查时间**：{ISO时间}
**规范版本**：DESIGN.md v4.0
**检查模式**：{正常 / ⚠️ CSP降级}

---

### 📊 总体评估

| Token匹配率 | 阻断项 | 建议项 | 信息项 | 合规率 | CI状态 |
|------------|--------|--------|--------|--------|--------|
| {X}/{55} | {N} | {N} | {N} | {X}% | {PASS/FAIL} |

> CI状态：阻断项 > 0 → **FAIL**

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
```

**JSON 报告**（CI/CD 可解析）：
```json
{
  "meta": {
    "url": "https://example.com",
    "timestamp": "2026-05-15T12:00:00Z",
    "spec_version": "4.0",
    "skill_version": "2.1.0",
    "mode": "normal",
    "csp_degraded": false
  },
  "summary": {
    "token_match_rate": "48/55",
    "compliance_rate": "87%",
    "blockers": 3,
    "warnings": 5,
    "info": 2,
    "ci_pass": false
  },
  "blockers": [
    {"id": 1, "section": "§1.1", "token": "--color-primary", "expected": "#2563EB", "actual": "#1a56db", "fix": "Replace --color-primary value with #2563EB"}
  ],
  "warnings": [],
  "info": [],
  "token_table": [
    {"token": "--color-primary", "expected": "#2563EB", "actual": "#2563EB", "status": "pass"}
  ]
}
```

#### 3b. 修正HTML页面

生成完整的独立HTML文件，遵循修正原则（见下方）。

#### 3c. CI/CD 集成与代码仓库联动

**CI/CD 集成指南**：

| 集成方式 | 说明 |
|----------|------|
| 失败阈值 | **阻断项 > 0 → CI FAIL**（建议作为流水线质量门禁） |
| 输出格式 | JSON 报告（`lint-report.json`）+ Markdown 报告（`lint-report.md`） |
| 退出码 | 阻断项 > 0 → exit 1；建议项 > 10 → exit 0 with warning |
| 示例用法 | `curl -X POST ... | jq .summary.ci_pass` |

**代码仓库联动指南**：

| 场景 | 推荐做法 |
|------|---------|
| 修改回写源CSS | 不直接覆盖，从修正HTML中提取 `<!-- FIX: §X -->` 注释对应的CSS变更，手动应用到源文件 |
| 生成 patch/diff | 将原HTML与-fixed.html做 `diff`，输出 `changes.patch` |
| 只改CSS文件 | 使用 `browser_console` 提取的 Token 值，生成 `token-overrides.css` 补丁文件 |
| 批量检查 | 对项目所有页面循环执行，输出汇总报告 |

**与项目集成的推荐文件结构**：
```
project/
├── .design-lint/
│   ├── DESIGN.md           # 团队规范文件
│   ├── reports/
│   │   └── 2026-05-15/
│   │       ├── dashboard-lint-report.md
│   │       └── dashboard-lint-report.json
│   └── fixes/
│       └── dashboard-fixed.html
```

---

## 修正HTML原则

- 内容/功能不变，只修样式
- CSS全部内联 + Token化（`:root` 补全所有规范Token）
- 每个修改处标注 `<!-- FIX: §X 条款：原值 → 规范值 -->`
- 文件名：`{原页面}-fixed.html`
- 保留原页面注释和结构，仅替换样式属性值

---

## 版本记录
- v2.1.0 (2026-05-15)：修正脚本策略表述；新增CSP降级模式；新增输入前置条件；新增「适配你的设计规范」章节；细化 §7/§8/§9/§10 判定规则为逐项可执行标准；新增 JSON 报告格式与 CI/CD 集成指南；新增代码仓库联动指南
- v2.0.0 (2026-05-15)：对齐 DESIGN.md v4.0，增加Token精确对比体系、严重度分级、表格/表单/图标/Badge组件规范、过渡动画规范
- v1.0.0 (2026-05-15)：初始版本，三步检查流程
