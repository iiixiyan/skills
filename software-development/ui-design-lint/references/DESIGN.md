# 设计规范参考文档 (现代企业级 SaaS 风格 - 多页面-v1 版本)

本文档定义了团队统一使用的 UI 设计规范，作为 `ui-style-auditor` Skill 的内置基准规范。用户可以通过提供自定义规范来覆盖此默认设置。

---

## 1. 颜色系统

### 1.1 品牌色 (Primary Colors)

| 语义名称 | HEX | RGB | HSL | 使用场景 |
|---|---|---|---|---|
| Primary | #2563EB | 37, 99, 235 | 217, 91%, 60% | 主按钮、主要链接、选中态 |
| Primary Dark | #1D4ED8 | 29, 78, 216 | 224, 76%, 52% | 主按钮 hover |
| Primary Soft | #EFF6FF | 239, 246, 255 | 217, 94%, 97% | 主按钮 hover 背景、选中背景 |
| Primary Active BG | rgba(37, 99, 235, 0.18) | - | - | 侧边栏导航 active 背景 |

### 1.2 中性色 (Neutral Colors)

| 语义名称 | HEX | RGB | HSL | 使用场景 |
|---|---|---|---|---|
| Text Primary | #101828 | 16, 24, 40 | 222, 43%, 11% | 一级标题、重要正文 |
| Text Secondary | #667085 | 102, 112, 133 | 220, 13%, 46% | 次级文字、描述 |
| Text Muted | #98A2B3 | 152, 162, 179 | 220, 15%, 65% | 辅助文字、占位符 |
| Text Inverse | #FFFFFF | 255, 255, 255 | 0, 0%, 100% | 深色背景上的文字 |
| Border Default | #E5E7EB | 229, 231, 235 | 210, 14%, 91% | 默认边框 |
| Background Page | #F5F7FB | 245, 247, 251 | 220, 10%, 97% | 页面背景 |
| Background Card | #FFFFFF | 255, 255, 255 | 0, 0%, 100% | 卡片、浮层背景 |
| Background Hover | #F9FAFB | 249, 250, 251 | 210, 8%, 98% | hover 背景 |

### 1.3 侧边栏专用色

| 语义名称 | HEX | RGB | 使用场景 |
|---|---|---|---|
| Sidebar BG Start | #0F172A | 15, 23, 42 | 侧边栏背景起始色 |
| Sidebar BG End | #111827 | 17, 24, 39 | 侧边栏背景结束色（渐变） |
| Sidebar Text | #D0D5DD | 208, 213, 221 | 侧边栏文字 |
| Sidebar Text Muted | #667085 | 102, 112, 133 | 侧边栏次级文字 |

### 1.4 语义色 (Semantic Colors)

| 语义名称 | HEX | RGB | HSL | 使用场景 |
|---|---|---|---|---|
| Success | #12B76A | 18, 183, 106 | 150, 82%, 39% | 成功状态、完成操作 |
| Success Light | #ECFDF3 | 236, 253, 243 | 150, 83%, 96% | 成功背景 |
| Warning | #F79009 | 247, 144, 9 | 35, 94%, 50% | 警告状态 |
| Warning Light | #FFFAEB | 255, 250, 235 | 48, 99%, 96% | 警告背景 |
| Error | #F04438 | 240, 68, 56 | 5, 86%, 58% | 错误状态、危险操作 |
| Error Light | #FEF3F2 | 254, 243, 242 | 4, 92%, 97% | 错误背景 |

---

## 2. 字体系统

### 2.1 字体族

```css
--font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Microsoft YaHei", Arial, sans-serif;
```

**说明**：
- 首选系统字体（-apple-system 用于 macOS/iOS）
- 中文优先 PingFang SC（苹方）然后 Microsoft YaHei（微软雅黑）
- 英文优先 Segoe UI（Windows）
- 不推荐使用 Inter，中文环境优先用系统字体

### 2.2 字号层级

| 层级 | 用途 | 字号 | 字重 | 行高 | 颜色 |
|---|---|---|---|---|---|
| H1 | 页面主标题 | 26px | 700 | 34px | Text Primary |
| H2 | 区块标题 | 18-20px | 600-700 | 26-28px | Text Primary |
| H3 | 卡片标题 | 16px | 600 | 24px | Text Primary |
| Body | 正文 | 14px | 400 | 22px | Text Secondary |
| Body Small | 小号正文 | 13px | 400 | 20px | Text Secondary |
| Caption | 辅助文字 | 12px | 400 | 18px | Text Muted |
| Label | 标签文字 | 12px | 500 | 16px | Text Muted |

---

## 3. 间距系统

### 3.1 基础间距单位

```
base-unit: 4px
```

### 3.2 常用场景间距

| 场景 | 间距 |
|---|---|
| 侧边栏宽度 | 252px |
| 侧边栏 padding | 24px 18px |
| 页面顶部导航高度 | 68px |
| 页面边缘留白 | 24-28px |
| 卡片内部 padding | 24px |
| 卡片之间间距 | 20-24px |
| 大卡片圆角 | 16px |
| 导航项高度 | 42px |
| 导航项圆角 | 10px |
| 按钮高度 | 40px |
| 按钮圆角 | 8-10px |
| Logo 区域 margin-bottom | 32px |
| 区块之间间距 | 24px |

---

## 4. 圆角系统

### 4.1 圆角 Token

| 名称 | 值 | 用途 |
|---|---|---|
| radius-sm | 8px | 按钮、输入框、小元素 |
| radius-md | 10px | 导航项、小容器 |
| radius-lg | 12px | 卡片、面板 |
| radius-xl | 14px | 侧边栏底部卡片 |
| radius-2xl | 16px | 大卡片、主容器 |
| radius-full | 9999px | 胶囊形、圆形头像 |

### 4.2 组件圆角规范

| 组件 | 圆角 | 说明 |
|---|---|---|
| 大卡片 | 16px | radius-2xl |
| 中卡片 | 14px | radius-xl |
| 导航项 | 10px | radius-md |
| 按钮 | 8-10px | radius-sm ~ md |
| 输入框 | 8px | radius-sm |
| Logo 标识 | 12px | radius-lg |
| 头像 | 50% | radius-full |
| Badge 标签 | 9999px | radius-full |

---

## 5. 阴影系统

### 5.1 阴影 Token

| 名称 | 值 | 用途 |
|---|---|---|
| shadow-sm | 0 1px 2px 0 rgba(16, 24, 40, 0.05) | 轻微阴影 |
| shadow-md | 0 4px 6px -1px rgba(16, 24, 40, 0.06) | 中等阴影 |
| shadow-lg | 0 12px 32px rgba(16, 24, 40, 0.08) | 主卡片阴影（标准） |
| shadow-xl | 0 20px 40px rgba(16, 24, 40, 0.12) | 大阴影、浮层 |
| shadow-focus | 0 0 0 2px rgba(37, 99, 235, 0.5) | focus 状态 |

### 5.2 应用场景

| 场景 | 阴影 |
|---|---|
| Logo 标识 | shadow-lg 或自定义投影 |
| 主卡片 | shadow-lg（推荐使用） |
| 浮层下拉 | shadow-xl |
| 顶部导航 | 无阴影，仅底部边框 |

---

## 6. 组件规范

### 6.1 侧边栏 Sidebar

```css
.sidebar {
  width: 252px;
  background: linear-gradient(180deg, #0f172a 0%, #111827 100%);
  color: #fff;
  padding: 24px 18px;
  position: fixed;
  inset: 0 auto 0 0;
}
```

**规范**：
- 宽度：252px
- 背景：深色渐变 (#0f172a → #111827)
- 文字颜色：#FFFFFF
- 内边距：24px 18px

### 6.2 导航项 Nav Item

```css
.nav-item {
  height: 42px;
  border-radius: 10px;
  color: #d0d5dd;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 12px;
  margin-bottom: 6px;
  font-size: 14px;
}
.nav-item.active,
.nav-item:hover {
  background: rgba(37, 99, 235, 0.18);
  color: #fff;
}
```

**规范**：
- 高度：42px
- 圆角：10px
- Active 背景：rgba(37, 99, 235, 0.18)
- 字重：普通(400) / 选中(600)

### 6.3 顶部导航 Topbar

```css
.topbar {
  height: 68px;
  background: #FFFFFF;
  border-bottom: 1px solid #E5E7EB;
  padding: 0 24px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: sticky;
  top: 0;
  z-index: 10;
}
```

**规范**：
- 高度：68px
- 背景：#FFFFFF
- 边框底：1px solid #E5E7EB
- Position：sticky

### 6.4 按钮 Button

| 类型 | 背景色 | 文字色 | 边框 | 圆角 | 高度 |
|---|---|---|---|---|---|
| Primary | #2563EB | #FFFFFF | none | 8-10px | 40px |
| Primary Hover | #1D4ED8 | #FFFFFF | none | 8-10px | 40px |
| Secondary | #FFFFFF | #667085 | 1px solid #E5E7EB | 8-10px | 40px |
| Secondary Hover | #F9FAFB | #101828 | 1px solid #D0D5DD | 8-10px | 40px |
| Ghost | transparent | #667085 | none | 8-10px | 40px |

### 6.5 卡片 Card

```css
.card {
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 16px;
  padding: 24px;
  /* 可选：添加阴影 */
  box-shadow: 0 12px 32px rgba(16, 24, 40, 0.08);
}
```

**规范**：
- 背景：#FFFFFF
- 边框：1px solid #E5E7EB
- 圆角：16px
- 内边距：24px
- 阴影：推荐 shadow-lg

### 6.6 输入框 Input

```css
.input {
  height: 40px;
  padding: 0 14px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  font-size: 14px;
  color: #101828;
  background: #ffffff;
}
.input:focus {
  outline: none;
  border-color: #2563eb;
  box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.5);
}
```

**规范**：
- 高度：40px
- 圆角：8px
- 边框：1px solid #E5E7EB
- Focus：边框 #2563EB + 2px focus ring

### 6.7 头像 Avatar

```css
.avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: #EFF6FF;
  color: #2563EB;
  display: grid;
  place-items: center;
  font-weight: 700;
  font-size: 12px;
}
```

---

## 7. 响应式断点

### 7.1 断点定义

| 断点名称 | 宽度范围 | 行为 |
|---|---|---|
| xs | < 640px | 隐藏侧边栏，显示汉堡菜单 |
| sm | 640px - 767px | 侧边栏收缩为图标 |
| md | 768px - 1023px | 标准布局 |
| lg | 1024px - 1279px | 完整布局 |
| xl | ≥ 1280px | 宽屏布局 |

### 7.2 移动端适配

- **触控目标**：最小 44px × 44px
- **页面边缘留白**：16px
- **侧边栏策略**：隐藏或抽屉式
- **卡片策略**：单列布局

---

## 8. 可访问性规范

### 8.1 颜色对比度

| 场景 | 最小对比度 | WCAG 等级 |
|---|---|---|
| 正文与背景 | 4.5:1 | AA |
| 大号文本与背景 | 3:1 | AA |

### 8.2 Focus 状态

```css
*:focus-visible {
  outline: none;
  box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.5);
}
```



*文档版本：3.2 *
*最后更新：2026-05-13*
*说明：本规范为默认规范，用户可通过提供自定义规范来覆盖*
