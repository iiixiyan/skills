# 设计规范参考文档 (现代企业级 SaaS 风格 - 多页面-v2 版本)

本文档定义了团队统一使用的 UI 设计规范，作为 `ui-design-lint` Skill 的内置基准规范。

---

## 0. CSS 变量 Token 体系

所有检查以以下 Token 为基准，目标页面的 CSS 变量名必须与此一致：

```css
:root {
  /* === 品牌色 === */
  --color-primary: #2563EB;
  --color-primary-hover: #1D4ED8;
  --color-primary-soft: #EFF6FF;
  --color-primary-active-bg: rgba(37, 99, 235, 0.18);

  /* === 中性色 === */
  --color-text-primary: #101828;
  --color-text-secondary: #667085;
  --color-text-muted: #98A2B3;
  --color-text-inverse: #FFFFFF;
  --color-border: #E5E7EB;
  --color-bg-page: #F5F7FB;
  --color-bg-card: #FFFFFF;
  --color-bg-hover: #F9FAFB;

  /* === 侧边栏 === */
  --color-sidebar-bg-start: #0F172A;
  --color-sidebar-bg-end: #111827;
  --color-sidebar-text: #D0D5DD;
  --color-sidebar-text-muted: #667085;

  /* === 语义色 === */
  --color-success: #12B76A;
  --color-success-light: #ECFDF3;
  --color-warning: #F79009;
  --color-warning-light: #FFFAEB;
  --color-error: #F04438;
  --color-error-light: #FEF3F2;

  /* === 字体 === */
  --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Microsoft YaHei", "Noto Sans SC", Arial, sans-serif;
  --font-size-h1: 28px;
  --font-size-h2: 20px;
  --font-size-h3: 16px;
  --font-size-body: 14px;
  --font-size-body-sm: 13px;
  --font-size-caption: 12px;
  --font-weight-bold: 700;
  --font-weight-semibold: 600;
  --font-weight-medium: 500;
  --font-weight-normal: 400;
  --line-height-h1: 36px;
  --line-height-h2: 28px;
  --line-height-h3: 24px;
  --line-height-body: 22px;
  --line-height-body-sm: 20px;
  --line-height-caption: 18px;

  /* === 间距 === */
  --spacing-unit: 4px;
  --spacing-sidebar-width: 252px;
  --spacing-topbar-height: 68px;
  --spacing-page-padding: 24px;
  --spacing-page-padding-mobile: 16px;
  --spacing-card-padding: 24px;
  --spacing-card-gap: 20px;
  --spacing-section-gap: 24px;
  --spacing-nav-item-height: 42px;
  --spacing-button-height: 40px;
  --spacing-input-height: 40px;
  --spacing-touch-min: 44px;

  /* === 圆角 === */
  --radius-sm: 8px;
  --radius-md: 10px;
  --radius-lg: 12px;
  --radius-xl: 14px;
  --radius-2xl: 16px;
  --radius-full: 9999px;

  /* === 阴影 === */
  --shadow-sm: 0 1px 2px 0 rgba(16, 24, 40, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(16, 24, 40, 0.06);
  --shadow-lg: 0 12px 32px rgba(16, 24, 40, 0.08);
  --shadow-xl: 0 20px 40px rgba(16, 24, 40, 0.12);
  --shadow-focus: 0 0 0 2px rgba(37, 99, 235, 0.5);

  /* === 过渡 === */
  --transition-fast: 150ms ease;
  --transition-normal: 200ms ease;
  --transition-slow: 300ms ease;
}
```

---

## 1. 颜色系统

### 1.1 品牌色 (Primary Colors)

| 语义名称 | Token | HEX | 使用场景 |
|----------|-------|-----|----------|
| Primary | `--color-primary` | #2563EB | 主按钮、主要链接、选中态 |
| Primary Hover | `--color-primary-hover` | #1D4ED8 | 主按钮 hover |
| Primary Soft | `--color-primary-soft` | #EFF6FF | 主按钮 hover 背景、选中背景 |
| Primary Active BG | `--color-primary-active-bg` | rgba(37,99,235,0.18) | 侧边栏导航 active 背景 |

### 1.2 中性色 (Neutral Colors)

| 语义名称 | Token | HEX | 使用场景 |
|----------|-------|-----|----------|
| Text Primary | `--color-text-primary` | #101828 | 一级标题、重要正文 |
| Text Secondary | `--color-text-secondary` | #667085 | 次级文字、描述 |
| Text Muted | `--color-text-muted` | #98A2B3 | 辅助文字、占位符 |
| Text Inverse | `--color-text-inverse` | #FFFFFF | 深色背景上的文字 |
| Border Default | `--color-border` | #E5E7EB | 默认边框 |
| Background Page | `--color-bg-page` | #F5F7FB | 页面背景 |
| Background Card | `--color-bg-card` | #FFFFFF | 卡片、浮层背景 |
| Background Hover | `--color-bg-hover` | #F9FAFB | hover 背景 |

### 1.3 侧边栏专用色

| 语义名称 | Token | HEX | 使用场景 |
|----------|-------|-----|----------|
| Sidebar BG Start | `--color-sidebar-bg-start` | #0F172A | 侧边栏背景起始色 |
| Sidebar BG End | `--color-sidebar-bg-end` | #111827 | 侧边栏背景结束色（渐变） |
| Sidebar Text | `--color-sidebar-text` | #D0D5DD | 侧边栏文字 |
| Sidebar Text Muted | `--color-sidebar-text-muted` | #667085 | 侧边栏次级文字 |

### 1.4 语义色 (Semantic Colors)

| 语义名称 | Token | HEX | 使用场景 |
|----------|-------|-----|----------|
| Success | `--color-success` | #12B76A | 成功状态 |
| Success Light | `--color-success-light` | #ECFDF3 | 成功背景 |
| Warning | `--color-warning` | #F79009 | 警告状态 |
| Warning Light | `--color-warning-light` | #FFFAEB | 警告背景 |
| Error | `--color-error` | #F04438 | 错误状态、危险操作 |
| Error Light | `--color-error-light` | #FEF3F2 | 错误背景 |

### 1.5 色彩对比度要求

| 场景 | 最小对比度 | WCAG 等级 | 违规严重度 |
|------|-----------|-----------|-----------|
| 正文与背景 | 4.5:1 | AA | 🔴阻断 |
| 大号文本(≥18px bold)与背景 | 3:1 | AA | 🔴阻断 |
| 非文本UI元素 | 3:1 | AA | 🟡建议 |

---

## 2. 字体系统

### 2.1 字体族

```
Token: --font-sans
Value: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC",
       "Microsoft YaHei", "Noto Sans SC", Arial, sans-serif
```

**回退策略**：macOS → Windows → Linux → 通用无衬线

### 2.2 字号层级

| 层级 | Token | 字号 | 字重 | 行高 | 颜色Token | 违规严重度 |
|------|-------|------|------|------|-----------|-----------|
| H1 | `--font-size-h1` | 28px | 700 | 36px | --color-text-primary | 🔴阻断 |
| H2 | `--font-size-h2` | 20px | 600 | 28px | --color-text-primary | 🟡建议 |
| H3 | `--font-size-h3` | 16px | 600 | 24px | --color-text-primary | 🟡建议 |
| Body | `--font-size-body` | 14px | 400 | 22px | --color-text-secondary | 🔴阻断 |
| Body Small | `--font-size-body-sm` | 13px | 400 | 20px | --color-text-secondary | 🟡建议 |
| Caption | `--font-size-caption` | 12px | 400 | 18px | --color-text-muted | 🟡建议 |
| Label | — | 12px | 500 | 16px | --color-text-muted | 🟡建议 |

---

## 3. 间距系统

### 3.1 基础单位

```
Token: --spacing-unit
Value: 4px（所有间距为此单位的整数倍）
```

### 3.2 常用场景间距

| 场景 | Token | 值 | 允许偏差 | 违规严重度 |
|------|-------|-----|---------|-----------|
| 侧边栏宽度 | `--spacing-sidebar-width` | 252px | ±4px | 🔴阻断 |
| 侧边栏 padding | — | 24px 18px | ±2px | 🟡建议 |
| 页面顶部导航高度 | `--spacing-topbar-height` | 68px | ±2px | 🔴阻断 |
| 页面边缘留白 | `--spacing-page-padding` | 24px | ±4px | 🟡建议 |
| 移动端留白 | `--spacing-page-padding-mobile` | 16px | ±2px | 🟡建议 |
| 卡片内部 padding | `--spacing-card-padding` | 24px | ±4px | 🔴阻断 |
| 卡片之间间距 | `--spacing-card-gap` | 20-24px | ±4px | 🟡建议 |
| 导航项高度 | `--spacing-nav-item-height` | 42px | ±2px | 🔴阻断 |
| 按钮高度 | `--spacing-button-height` | 40px | ±2px | 🔴阻断 |
| 输入框高度 | `--spacing-input-height` | 40px | ±2px | 🟡建议 |
| 触控最小尺寸 | `--spacing-touch-min` | 44px | 不可低于 | 🟡建议 |

### 3.3 间距倍数规则

所有 margin/padding/gap 必须为 `--spacing-unit (4px)` 的整数倍。
- 违规示例：margin: 7px → ❌
- 合规示例：margin: 8px → ✅

---

## 4. 圆角系统

| 名称 | Token | 值 | 用途 | 违规严重度 |
|------|-------|-----|------|-----------|
| radius-sm | `--radius-sm` | 8px | 按钮、输入框 | 🔴阻断 |
| radius-md | `--radius-md` | 10px | 导航项、小容器 | 🟡建议 |
| radius-lg | `--radius-lg` | 12px | Logo、面板 | 🟡建议 |
| radius-xl | `--radius-xl` | 14px | 中卡片 | 🟡建议 |
| radius-2xl | `--radius-2xl` | 16px | 大卡片、主容器 | 🔴阻断 |
| radius-full | `--radius-full` | 9999px | 胶囊形、圆形头像 | 🔴阻断 |

---

## 5. 阴影系统

| 名称 | Token | 值 | 用途 | 违规严重度 |
|------|-------|-----|------|-----------|
| shadow-sm | `--shadow-sm` | 0 1px 2px 0 rgba(16,24,40,0.05) | 轻微阴影 | 🟡建议 |
| shadow-md | `--shadow-md` | 0 4px 6px -1px rgba(16,24,40,0.06) | 中等阴影 | 🟡建议 |
| shadow-lg | `--shadow-lg` | 0 12px 32px rgba(16,24,40,0.08) | 卡片阴影（标准） | 🔴阻断 |
| shadow-xl | `--shadow-xl` | 0 20px 40px rgba(16,24,40,0.12) | 浮层、下拉 | 🟡建议 |
| shadow-focus | `--shadow-focus` | 0 0 0 2px rgba(37,99,235,0.5) | Focus 状态 | 🔴阻断 |

---

## 6. 过渡动画

| 场景 | Token | 值 | 违规严重度 |
|------|-------|-----|-----------|
| 颜色/背景过渡 | `--transition-fast` | 150ms ease | 🟡建议 |
| 标准交互 | `--transition-normal` | 200ms ease | 🟡建议 |
| 展开/收起 | `--transition-slow` | 300ms ease | 🟡建议 |

**必须添加 transition 的元素**：按钮、链接、输入框、导航项

---

## 7. 标准页面布局模板

```
┌──────────────┬────────────────────────────────────┐
│              │ Topbar                             │
│              │ height: 68px, bg: #FFF             │
│   Sidebar    │ border-bottom: 1px solid #E5E7EB   │
│              ├────────────────────────────────────┤
│   width:     │                                    │
│   252px      │ Main Content Area                  │
│              │ padding: 24px                      │
│   bg:        │ bg: #F5F7FB                        │
│   gradient   │                                    │
│   #0F172A→   │ ┌────────────────────────────────┐ │
│   #111827    │ │ Card                           │ │
│              │ │ bg: #FFF, radius: 16px         │ │
│   text:      │ │ padding: 24px                  │ │
│   #D0D5DD    │ │ border: 1px solid #E5E7EB     │ │
│              │ │ shadow: shadow-lg              │ │
│              │ └────────────────────────────────┘ │
│              │                                    │
│              │ gap between cards: 20-24px         │
└──────────────┴────────────────────────────────────┘
```

**布局检查要点**：
1. 侧边栏固定左侧，宽度 252px
2. 顶部导航 sticky，高度 68px
3. 主内容区 margin-left: 252px, padding: 24px
4. 卡片间距 20-24px
5. 页面背景 #F5F7FB

---

## 8. 组件规范

### 8.1 侧边栏 Sidebar

```css
.sidebar {
  width: var(--spacing-sidebar-width);          /* 252px */
  background: linear-gradient(180deg,
    var(--color-sidebar-bg-start) 0%,          /* #0F172A */
    var(--color-sidebar-bg-end) 100%);          /* #111827 */
  color: var(--color-text-inverse);             /* #FFFFFF */
  padding: 24px 18px;
  position: fixed;
  inset: 0 auto 0 0;
}
```

**关键检查项**：宽度 | 背景渐变方向(180deg) | 颜色 | padding

### 8.2 导航项 Nav Item

```css
.nav-item {
  height: var(--spacing-nav-item-height);        /* 42px */
  border-radius: var(--radius-md);              /* 10px */
  color: var(--color-sidebar-text);             /* #D0D5DD */
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 12px;
  margin-bottom: 6px;
  font-size: var(--font-size-body);             /* 14px */
  transition: all var(--transition-fast);       /* 150ms ease */
}
.nav-item.active,
.nav-item:hover {
  background: var(--color-primary-active-bg);
  color: var(--color-text-inverse);
}
```

### 8.3 顶部导航 Topbar

```css
.topbar {
  height: var(--spacing-topbar-height);          /* 68px */
  background: var(--color-bg-card);             /* #FFFFFF */
  border-bottom: 1px solid var(--color-border); /* #E5E7EB */
  padding: 0 var(--spacing-page-padding);       /* 0 24px */
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: sticky;
  top: 0;
  z-index: 10;
}
```

### 8.4 按钮 Button

| 类型 | 背景Token | 文字色Token | 边框 | 圆角Token | 高度Token |
|------|-----------|-------------|------|-----------|-----------|
| Primary | `--color-primary` | `--color-text-inverse` | none | `--radius-sm` | `--spacing-button-height` |
| Primary Hover | `--color-primary-hover` | `--color-text-inverse` | none | `--radius-sm` | `--spacing-button-height` |
| Secondary | `--color-bg-card` | `--color-text-secondary` | 1px solid `--color-border` | `--radius-sm` | `--spacing-button-height` |
| Secondary Hover | `--color-bg-hover` | `--color-text-primary` | 1px solid #D0D5DD | `--radius-sm` | `--spacing-button-height` |
| Ghost | transparent | `--color-text-secondary` | none | `--radius-sm` | `--spacing-button-height` |

**触控适配**：移动端（<640px）按钮最小 44px 高度。

### 8.5 卡片 Card

```css
.card {
  background: var(--color-bg-card);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-2xl);
  padding: var(--spacing-card-padding);
  box-shadow: var(--shadow-lg);
}
```

### 8.6 输入框 Input

```css
.input {
  height: var(--spacing-input-height);
  padding: 0 14px;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-sm);
  font-size: var(--font-size-body);
  color: var(--color-text-primary);
  background: var(--color-bg-card);
  transition: all var(--transition-fast);
}
.input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: var(--shadow-focus);
}
```

### 8.7 表格 Table

```css
.table {
  width: 100%;
  border-collapse: collapse;
  font-size: var(--font-size-body);
}
.table th {
  height: 44px;
  padding: 0 16px;
  color: var(--color-text-secondary);
  font-weight: var(--font-weight-medium);
  font-size: var(--font-size-caption);
  text-align: left;
  background: var(--color-bg-hover);
  border-bottom: 1px solid var(--color-border);
}
.table td {
  height: 52px;
  padding: 0 16px;
  color: var(--color-text-primary);
  border-bottom: 1px solid var(--color-border);
}
.table tr:hover td {
  background: var(--color-bg-hover);
}
```

### 8.8 表单 Form

```css
.form-group {
  margin-bottom: 20px;
}
.form-label {
  display: block;
  margin-bottom: 6px;
  font-size: var(--font-size-body);
  font-weight: var(--font-weight-medium);
  color: var(--color-text-primary);
}
.form-hint {
  margin-top: 4px;
  font-size: var(--font-size-caption);
  color: var(--color-text-muted);
}
.form-error {
  margin-top: 4px;
  font-size: var(--font-size-caption);
  color: var(--color-error);
}
```

### 8.9 图标 Icon

| 属性 | 规范 | 违规严重度 |
|------|------|-----------|
| 图标库 | 统一使用同一图标集（如 Lucide/Heroicons） | 🟡建议 |
| 默认尺寸 | 20px × 20px | 🟡建议 |
| 小尺寸 | 16px × 16px | 🟡建议 |
| 颜色 | 继承父元素 color，不单独设置 | 🟡建议 |
| 可点击图标 hover | color → `--color-primary` + `transition-fast` | 🟡建议 |

### 8.10 头像 Avatar

```css
.avatar {
  width: 32px;
  height: 32px;
  border-radius: var(--radius-full);
  background: var(--color-primary-soft);
  color: var(--color-primary);
  display: grid;
  place-items: center;
  font-weight: var(--font-weight-bold);
  font-size: var(--font-size-caption);
}
```

### 8.11 Badge 标签

```css
.badge {
  display: inline-flex;
  align-items: center;
  padding: 2px 8px;
  border-radius: var(--radius-full);
  font-size: var(--font-size-caption);
  font-weight: var(--font-weight-medium);
}
.badge-success { background: var(--color-success-light); color: var(--color-success); }
.badge-warning { background: var(--color-warning-light); color: var(--color-warning); }
.badge-error   { background: var(--color-error-light);   color: var(--color-error); }
```

---

## 9. 响应式断点

| 断点 | 宽度 | 侧边栏 | 布局 | 边缘留白 |
|------|------|--------|------|----------|
| xs | < 640px | 隐藏（汉堡菜单） | 单列 | 16px |
| sm | 640-767px | 收缩为图标 | 单列 | 20px |
| md | 768-1023px | 252px | 标准 | 24px |
| lg | 1024-1279px | 252px | 完整 | 24px |
| xl | ≥ 1280px | 252px | 宽屏 | 28px |

**触控适配**：< 640px 所有可点击元素 ≥ 44px × 44px

---

## 10. 可访问性

### 10.1 颜色对比度

| 场景 | 最小对比度 | WCAG | 违规严重度 |
|------|-----------|------|-----------|
| 正文 (< 18px) 与背景 | 4.5:1 | AA | 🔴阻断 |
| 大文本 (≥ 18px bold) | 3:1 | AA | 🔴阻断 |
| 非文本UI (边框/图标) | 3:1 | AA | 🟡建议 |

### 10.2 Focus 状态

```css
*:focus-visible {
  outline: none;
  box-shadow: var(--shadow-focus);
}
```

### 10.3 其他要求

- 所有 `<img>` 必须有 `alt` 属性 → 🟡建议
- 表单输入框必须有 `<label>` 关联 → 🔴阻断
- 页面必须有 `<h1>` → 🟡建议
- 跳过快照链接（Skip to content）→ 🟡建议

---

## 附录A：违规严重度定义

| 级别 | 标识 | 含义 | 是否阻断发布 |
|------|------|------|-------------|
| 🔴 阻断 | BLOCKER | 核心规范违反，必须修复 | ✅ 是 |
| 🟡 建议 | WARNING | 最佳实践偏离，强烈建议修复 | ❌ 否 |
| 🔵 信息 | INFO | 可优化项，酌情处理 | ❌ 否 |

---

## 附录B：颜色对比度速查表

| 前景色 | 背景色 | 对比度 | WCAG AA |
|--------|--------|--------|---------|
| #101828 | #FFFFFF | 15.4:1 | ✅ |
| #667085 | #FFFFFF | 4.8:1 | ✅ |
| #98A2B3 | #FFFFFF | 3.2:1 | ❌ (仅大文本可用) |
| #2563EB | #FFFFFF | 5.4:1 | ✅ |
| #FFFFFF | #2563EB | 5.4:1 | ✅ |
| #D0D5DD | #111827 | 10.2:1 | ✅ |

---

*文档版本：4.1*
*最后更新：2026-05-15*
*关联Skill：ui-design-lint v2.1*
