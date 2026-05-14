     1|# 设计规范参考文档 (现代企业级 SaaS 风格 - 多页面-v2 版本)
     2|
     3|本文档定义了团队统一使用的 UI 设计规范，作为 `ui-design-lint` Skill 的内置基准规范。
     4|
     5|---
     6|
     7|## 0. CSS 变量 Token 体系
     8|
     9|所有检查以以下 Token 为基准，目标页面的 CSS 变量名必须与此一致：
    10|
    11|```css
    12|:root {
    13|  /* === 品牌色 === */
    14|  --color-primary: #2563EB;
    15|  --color-primary-hover: #1D4ED8;
    16|  --color-primary-soft: #EFF6FF;
    17|  --color-primary-active-bg: rgba(37, 99, 235, 0.18);
    18|
    19|  /* === 中性色 === */
    20|  --color-text-primary: #101828;
    21|  --color-text-secondary: #667085;
    22|  --color-text-muted: #98A2B3;
    23|  --color-text-inverse: #FFFFFF;
    24|  --color-border: #E5E7EB;
    25|  --color-bg-page: #F5F7FB;
    26|  --color-bg-card: #FFFFFF;
    27|  --color-bg-hover: #F9FAFB;
    28|
    29|  /* === 侧边栏 === */
    30|  --color-sidebar-bg-start: #0F172A;
    31|  --color-sidebar-bg-end: #111827;
    32|  --color-sidebar-text: #D0D5DD;
    33|  --color-sidebar-text-muted: #667085;
    34|
    35|  /* === 语义色 === */
    36|  --color-success: #12B76A;
    37|  --color-success-light: #ECFDF3;
    38|  --color-warning: #F79009;
    39|  --color-warning-light: #FFFAEB;
    40|  --color-error: #F04438;
    41|  --color-error-light: #FEF3F2;
    42|
    43|  /* === 字体 === */
    44|  --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Microsoft YaHei", "Noto Sans SC", Arial, sans-serif;
    45|  --font-size-h1: 28px;
    46|  --font-size-h2: 20px;
    47|  --font-size-h3: 16px;
    48|  --font-size-body: 14px;
    49|  --font-size-body-sm: 13px;
    50|  --font-size-caption: 12px;
    51|  --font-weight-bold: 700;
    52|  --font-weight-semibold: 600;
    53|  --font-weight-medium: 500;
    54|  --font-weight-normal: 400;
    55|  --line-height-h1: 36px;
    56|  --line-height-h2: 28px;
    57|  --line-height-h3: 24px;
    58|  --line-height-body: 22px;
    59|  --line-height-body-sm: 20px;
    60|  --line-height-caption: 18px;
    61|
    62|  /* === 间距 === */
    63|  --spacing-unit: 4px;
    64|  --spacing-sidebar-width: 252px;
    65|  --spacing-topbar-height: 68px;
    66|  --spacing-page-padding: 24px;
    67|  --spacing-page-padding-mobile: 16px;
    68|  --spacing-card-padding: 24px;
    69|  --spacing-card-gap: 20px;
    70|  --spacing-section-gap: 24px;
    71|  --spacing-nav-item-height: 42px;
    72|  --spacing-button-height: 40px;
    73|  --spacing-input-height: 40px;
    74|  --spacing-touch-min: 44px;
    75|
    76|  /* === 圆角 === */
    77|  --radius-sm: 8px;
    78|  --radius-md: 10px;
    79|  --radius-lg: 12px;
    80|  --radius-xl: 14px;
    81|  --radius-2xl: 16px;
    82|  --radius-full: 9999px;
    83|
    84|  /* === 阴影 === */
    85|  --shadow-sm: 0 1px 2px 0 rgba(16, 24, 40, 0.05);
    86|  --shadow-md: 0 4px 6px -1px rgba(16, 24, 40, 0.06);
    87|  --shadow-lg: 0 12px 32px rgba(16, 24, 40, 0.08);
    88|  --shadow-xl: 0 20px 40px rgba(16, 24, 40, 0.12);
    89|  --shadow-focus: 0 0 0 2px rgba(37, 99, 235, 0.5);
    90|
    91|  /* === 过渡 === */
    92|  --transition-fast: 150ms ease;
    93|  --transition-normal: 200ms ease;
    94|  --transition-slow: 300ms ease;
    95|}
    96|```
    97|
    98|---
    99|
   100|## 1. 颜色系统
   101|
   102|### 1.1 品牌色 (Primary Colors)
   103|
   104|| 语义名称 | Token | HEX | 使用场景 |
   105||----------|-------|-----|----------|
   106|| Primary | `--color-primary` | #2563EB | 主按钮、主要链接、选中态 |
   107|| Primary Hover | `--color-primary-hover` | #1D4ED8 | 主按钮 hover |
   108|| Primary Soft | `--color-primary-soft` | #EFF6FF | 主按钮 hover 背景、选中背景 |
   109|| Primary Active BG | `--color-primary-active-bg` | rgba(37,99,235,0.18) | 侧边栏导航 active 背景 |
   110|
   111|### 1.2 中性色 (Neutral Colors)
   112|
   113|| 语义名称 | Token | HEX | 使用场景 |
   114||----------|-------|-----|----------|
   115|| Text Primary | `--color-text-primary` | #101828 | 一级标题、重要正文 |
   116|| Text Secondary | `--color-text-secondary` | #667085 | 次级文字、描述 |
   117|| Text Muted | `--color-text-muted` | #98A2B3 | 辅助文字、占位符 |
   118|| Text Inverse | `--color-text-inverse` | #FFFFFF | 深色背景上的文字 |
   119|| Border Default | `--color-border` | #E5E7EB | 默认边框 |
   120|| Background Page | `--color-bg-page` | #F5F7FB | 页面背景 |
   121|| Background Card | `--color-bg-card` | #FFFFFF | 卡片、浮层背景 |
   122|| Background Hover | `--color-bg-hover` | #F9FAFB | hover 背景 |
   123|
   124|### 1.3 侧边栏专用色
   125|
   126|| 语义名称 | Token | HEX | 使用场景 |
   127||----------|-------|-----|----------|
   128|| Sidebar BG Start | `--color-sidebar-bg-start` | #0F172A | 侧边栏背景起始色 |
   129|| Sidebar BG End | `--color-sidebar-bg-end` | #111827 | 侧边栏背景结束色（渐变） |
   130|| Sidebar Text | `--color-sidebar-text` | #D0D5DD | 侧边栏文字 |
   131|| Sidebar Text Muted | `--color-sidebar-text-muted` | #667085 | 侧边栏次级文字 |
   132|
   133|### 1.4 语义色 (Semantic Colors)
   134|
   135|| 语义名称 | Token | HEX | 使用场景 |
   136||----------|-------|-----|----------|
   137|| Success | `--color-success` | #12B76A | 成功状态 |
   138|| Success Light | `--color-success-light` | #ECFDF3 | 成功背景 |
   139|| Warning | `--color-warning` | #F79009 | 警告状态 |
   140|| Warning Light | `--color-warning-light` | #FFFAEB | 警告背景 |
   141|| Error | `--color-error` | #F04438 | 错误状态、危险操作 |
   142|| Error Light | `--color-error-light` | #FEF3F2 | 错误背景 |
   143|
   144|### 1.5 色彩对比度要求
   145|
   146|| 场景 | 最小对比度 | WCAG 等级 | 违规严重度 |
   147||------|-----------|-----------|-----------|
   148|| 正文与背景 | 4.5:1 | AA | 🔴阻断 |
   149|| 大号文本(≥18px bold)与背景 | 3:1 | AA | 🔴阻断 |
   150|| 非文本UI元素 | 3:1 | AA | 🟡建议 |
   151|
   152|---
   153|
   154|## 2. 字体系统
   155|
   156|### 2.1 字体族
   157|
   158|```
   159|Token: --font-sans
   160|Value: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC",
   161|       "Microsoft YaHei", "Noto Sans SC", Arial, sans-serif
   162|```
   163|
   164|**回退策略**：macOS → Windows → Linux → 通用无衬线
   165|
   166|### 2.2 字号层级
   167|
   168|| 层级 | Token | 字号 | 字重 | 行高 | 颜色Token | 违规严重度 |
   169||------|-------|------|------|------|-----------|-----------|
   170|| H1 | `--font-size-h1` | 28px | 700 | 36px | --color-text-primary | 🔴阻断 |
   171|| H2 | `--font-size-h2` | 20px | 600 | 28px | --color-text-primary | 🟡建议 |
   172|| H3 | `--font-size-h3` | 16px | 600 | 24px | --color-text-primary | 🟡建议 |
   173|| Body | `--font-size-body` | 14px | 400 | 22px | --color-text-secondary | 🔴阻断 |
   174|| Body Small | `--font-size-body-sm` | 13px | 400 | 20px | --color-text-secondary | 🟡建议 |
   175|| Caption | `--font-size-caption` | 12px | 400 | 18px | --color-text-muted | 🟡建议 |
   176|| Label | — | 12px | 500 | 16px | --color-text-muted | 🟡建议 |
   177|
   178|---
   179|
   180|## 3. 间距系统
   181|
   182|### 3.1 基础单位
   183|
   184|```
   185|Token: --spacing-unit
   186|Value: 4px（所有间距为此单位的整数倍）
   187|```
   188|
   189|### 3.2 常用场景间距
   190|
   191|| 场景 | Token | 值 | 允许偏差 | 违规严重度 |
   192||------|-------|-----|---------|-----------|
   193|| 侧边栏宽度 | `--spacing-sidebar-width` | 252px | ±4px | 🔴阻断 |
   194|| 侧边栏 padding | — | 24px 18px | ±2px | 🟡建议 |
   195|| 页面顶部导航高度 | `--spacing-topbar-height` | 68px | ±2px | 🔴阻断 |
   196|| 页面边缘留白 | `--spacing-page-padding` | 24px | ±4px | 🟡建议 |
   197|| 移动端留白 | `--spacing-page-padding-mobile` | 16px | ±2px | 🟡建议 |
   198|| 卡片内部 padding | `--spacing-card-padding` | 24px | ±4px | 🔴阻断 |
   199|| 卡片之间间距 | `--spacing-card-gap` | 20-24px | ±4px | 🟡建议 |
   200|| 导航项高度 | `--spacing-nav-item-height` | 42px | ±2px | 🔴阻断 |
   201|| 按钮高度 | `--spacing-button-height` | 40px | ±2px | 🔴阻断 |
   202|| 输入框高度 | `--spacing-input-height` | 40px | ±2px | 🟡建议 |
   203|| 触控最小尺寸 | `--spacing-touch-min` | 44px | 不可低于 | 🟡建议 |
   204|
   205|### 3.3 间距倍数规则
   206|
   207|所有 margin/padding/gap 必须为 `--spacing-unit (4px)` 的整数倍。
   208|- 违规示例：margin: 7px → ❌
   209|- 合规示例：margin: 8px → ✅
   210|
   211|---
   212|
   213|## 4. 圆角系统
   214|
   215|| 名称 | Token | 值 | 用途 | 违规严重度 |
   216||------|-------|-----|------|-----------|
   217|| radius-sm | `--radius-sm` | 8px | 按钮、输入框 | 🔴阻断 |
   218|| radius-md | `--radius-md` | 10px | 导航项、小容器 | 🟡建议 |
   219|| radius-lg | `--radius-lg` | 12px | Logo、面板 | 🟡建议 |
   220|| radius-xl | `--radius-xl` | 14px | 中卡片 | 🟡建议 |
   221|| radius-2xl | `--radius-2xl` | 16px | 大卡片、主容器 | 🔴阻断 |
   222|| radius-full | `--radius-full` | 9999px | 胶囊形、圆形头像 | 🔴阻断 |
   223|
   224|---
   225|
   226|## 5. 阴影系统
   227|
   228|| 名称 | Token | 值 | 用途 | 违规严重度 |
   229||------|-------|-----|------|-----------|
   230|| shadow-sm | `--shadow-sm` | 0 1px 2px 0 rgba(16,24,40,0.05) | 轻微阴影 | 🟡建议 |
   231|| shadow-md | `--shadow-md` | 0 4px 6px -1px rgba(16,24,40,0.06) | 中等阴影 | 🟡建议 |
   232|| shadow-lg | `--shadow-lg` | 0 12px 32px rgba(16,24,40,0.08) | 卡片阴影（标准） | 🔴阻断 |
   233|| shadow-xl | `--shadow-xl` | 0 20px 40px rgba(16,24,40,0.12) | 浮层、下拉 | 🟡建议 |
   234|| shadow-focus | `--shadow-focus` | 0 0 0 2px rgba(37,99,235,0.5) | Focus 状态 | 🔴阻断 |
   235|
   236|---
   237|
   238|## 6. 过渡动画
   239|
   240|| 场景 | Token | 值 | 违规严重度 |
   241||------|-------|-----|-----------|
   242|| 颜色/背景过渡 | `--transition-fast` | 150ms ease | 🟡建议 |
   243|| 标准交互 | `--transition-normal` | 200ms ease | 🟡建议 |
   244|| 展开/收起 | `--transition-slow` | 300ms ease | 🟡建议 |
   245|
   246|**必须添加 transition 的元素**：按钮、链接、输入框、导航项
   247|
   248|---
   249|
   250|## 7. 标准页面布局模板
   251|
   252|```
   253|┌──────────────┬────────────────────────────────────┐
   254|│              │ Topbar                             │
   255|│              │ height: 68px, bg: #FFF             │
   256|│   Sidebar    │ border-bottom: 1px solid #E5E7EB   │
   257|│              ├────────────────────────────────────┤
   258|│   width:     │                                    │
   259|│   252px      │ Main Content Area                  │
   260|│              │ padding: 24px                      │
   261|│   bg:        │ bg: #F5F7FB                        │
   262|│   gradient   │                                    │
   263|│   #0F172A→   │ ┌────────────────────────────────┐ │
   264|│   #111827    │ │ Card                           │ │
   265|│              │ │ bg: #FFF, radius: 16px         │ │
   266|│   text:      │ │ padding: 24px                  │ │
   267|│   #D0D5DD    │ │ border: 1px solid #E5E7EB     │ │
   268|│              │ │ shadow: shadow-lg              │ │
   269|│              │ └────────────────────────────────┘ │
   270|│              │                                    │
   271|│              │ gap between cards: 20-24px         │
   272|└──────────────┴────────────────────────────────────┘
   273|```
   274|
   275|**布局检查要点**：
   276|1. 侧边栏固定左侧，宽度 252px
   277|2. 顶部导航 sticky，高度 68px
   278|3. 主内容区 margin-left: 252px, padding: 24px
   279|4. 卡片间距 20-24px
   280|5. 页面背景 #F5F7FB
   281|
   282|---
   283|
   284|## 8. 组件规范
   285|
   286|### 8.1 侧边栏 Sidebar
   287|
   288|```css
   289|.sidebar {
   290|  width: var(--spacing-sidebar-width);          /* 252px */
   291|  background: linear-gradient(180deg,
   292|    var(--color-sidebar-bg-start) 0%,          /* #0F172A */
   293|    var(--color-sidebar-bg-end) 100%);          /* #111827 */
   294|  color: var(--color-text-inverse);             /* #FFFFFF */
   295|  padding: 24px 18px;
   296|  position: fixed;
   297|  inset: 0 auto 0 0;
   298|}
   299|```
   300|
   301|**关键检查项**：宽度 | 背景渐变方向(180deg) | 颜色 | padding
   302|
   303|### 8.2 导航项 Nav Item
   304|
   305|```css
   306|.nav-item {
   307|  height: var(--spacing-nav-item-height);        /* 42px */
   308|  border-radius: var(--radius-md);              /* 10px */
   309|  color: var(--color-sidebar-text);             /* #D0D5DD */
   310|  display: flex;
   311|  align-items: center;
   312|  gap: 10px;
   313|  padding: 0 12px;
   314|  margin-bottom: 6px;
   315|  font-size: var(--font-size-body);             /* 14px */
   316|  transition: all var(--transition-fast);       /* 150ms ease */
   317|}
   318|.nav-item.active,
   319|.nav-item:hover {
   320|  background: var(--color-primary-active-bg);
   321|  color: var(--color-text-inverse);
   322|}
   323|```
   324|
   325|### 8.3 顶部导航 Topbar
   326|
   327|```css
   328|.topbar {
   329|  height: var(--spacing-topbar-height);          /* 68px */
   330|  background: var(--color-bg-card);             /* #FFFFFF */
   331|  border-bottom: 1px solid var(--color-border); /* #E5E7EB */
   332|  padding: 0 var(--spacing-page-padding);       /* 0 24px */
   333|  display: flex;
   334|  align-items: center;
   335|  justify-content: space-between;
   336|  position: sticky;
   337|  top: 0;
   338|  z-index: 10;
   339|}
   340|```
   341|
   342|### 8.4 按钮 Button
   343|
   344|| 类型 | 背景Token | 文字色Token | 边框 | 圆角Token | 高度Token |
   345||------|-----------|-------------|------|-----------|-----------|
   346|| Primary | `--color-primary` | `--color-text-inverse` | none | `--radius-sm` | `--spacing-button-height` |
   347|| Primary Hover | `--color-primary-hover` | `--color-text-inverse` | none | `--radius-sm` | `--spacing-button-height` |
   348|| Secondary | `--color-bg-card` | `--color-text-secondary` | 1px solid `--color-border` | `--radius-sm` | `--spacing-button-height` |
   349|| Secondary Hover | `--color-bg-hover` | `--color-text-primary` | 1px solid #D0D5DD | `--radius-sm` | `--spacing-button-height` |
   350|| Ghost | transparent | `--color-text-secondary` | none | `--radius-sm` | `--spacing-button-height` |
   351|
   352|**触控适配**：移动端（<640px）按钮最小 44px 高度。
   353|
   354|### 8.5 卡片 Card
   355|
   356|```css
   357|.card {
   358|  background: var(--color-bg-card);
   359|  border: 1px solid var(--color-border);
   360|  border-radius: var(--radius-2xl);
   361|  padding: var(--spacing-card-padding);
   362|  box-shadow: var(--shadow-lg);
   363|}
   364|```
   365|
   366|### 8.6 输入框 Input
   367|
   368|```css
   369|.input {
   370|  height: var(--spacing-input-height);
   371|  padding: 0 14px;
   372|  border: 1px solid var(--color-border);
   373|  border-radius: var(--radius-sm);
   374|  font-size: var(--font-size-body);
   375|  color: var(--color-text-primary);
   376|  background: var(--color-bg-card);
   377|  transition: all var(--transition-fast);
   378|}
   379|.input:focus {
   380|  outline: none;
   381|  border-color: var(--color-primary);
   382|  box-shadow: var(--shadow-focus);
   383|}
   384|```
   385|
   386|### 8.7 表格 Table
   387|
   388|```css
   389|.table {
   390|  width: 100%;
   391|  border-collapse: collapse;
   392|  font-size: var(--font-size-body);
   393|}
   394|.table th {
   395|  height: 44px;
   396|  padding: 0 16px;
   397|  color: var(--color-text-secondary);
   398|  font-weight: var(--font-weight-medium);
   399|  font-size: var(--font-size-caption);
   400|  text-align: left;
   401|  background: var(--color-bg-hover);
   402|  border-bottom: 1px solid var(--color-border);
   403|}
   404|.table td {
   405|  height: 52px;
   406|  padding: 0 16px;
   407|  color: var(--color-text-primary);
   408|  border-bottom: 1px solid var(--color-border);
   409|}
   410|.table tr:hover td {
   411|  background: var(--color-bg-hover);
   412|}
   413|```
   414|
   415|### 8.8 表单 Form
   416|
   417|```css
   418|.form-group {
   419|  margin-bottom: 20px;
   420|}
   421|.form-label {
   422|  display: block;
   423|  margin-bottom: 6px;
   424|  font-size: var(--font-size-body);
   425|  font-weight: var(--font-weight-medium);
   426|  color: var(--color-text-primary);
   427|}
   428|.form-hint {
   429|  margin-top: 4px;
   430|  font-size: var(--font-size-caption);
   431|  color: var(--color-text-muted);
   432|}
   433|.form-error {
   434|  margin-top: 4px;
   435|  font-size: var(--font-size-caption);
   436|  color: var(--color-error);
   437|}
   438|```
   439|
   440|### 8.9 图标 Icon
   441|
   442|| 属性 | 规范 | 违规严重度 |
   443||------|------|-----------|
   444|| 图标库 | 统一使用同一图标集（如 Lucide/Heroicons） | 🟡建议 |
   445|| 默认尺寸 | 20px × 20px | 🟡建议 |
   446|| 小尺寸 | 16px × 16px | 🟡建议 |
   447|| 颜色 | 继承父元素 color，不单独设置 | 🟡建议 |
   448|| 可点击图标 hover | color → `--color-primary` + `transition-fast` | 🟡建议 |
   449|
   450|### 8.10 头像 Avatar
   451|
   452|```css
   453|.avatar {
   454|  width: 32px;
   455|  height: 32px;
   456|  border-radius: var(--radius-full);
   457|  background: var(--color-primary-soft);
   458|  color: var(--color-primary);
   459|  display: grid;
   460|  place-items: center;
   461|  font-weight: var(--font-weight-bold);
   462|  font-size: var(--font-size-caption);
   463|}
   464|```
   465|
   466|### 8.11 Badge 标签
   467|
   468|```css
   469|.badge {
   470|  display: inline-flex;
   471|  align-items: center;
   472|  padding: 2px 8px;
   473|  border-radius: var(--radius-full);
   474|  font-size: var(--font-size-caption);
   475|  font-weight: var(--font-weight-medium);
   476|}
   477|.badge-success { background: var(--color-success-light); color: var(--color-success); }
   478|.badge-warning { background: var(--color-warning-light); color: var(--color-warning); }
   479|.badge-error   { background: var(--color-error-light);   color: var(--color-error); }
   480|```
   481|
   482|---
   483|
   484|## 9. 响应式断点
   485|
   486|| 断点 | 宽度 | 侧边栏 | 布局 | 边缘留白 |
   487||------|------|--------|------|----------|
   488|| xs | < 640px | 隐藏（汉堡菜单） | 单列 | 16px |
   489|| sm | 640-767px | 收缩为图标 | 单列 | 20px |
   490|| md | 768-1023px | 252px | 标准 | 24px |
   491|| lg | 1024-1279px | 252px | 完整 | 24px |
   492|| xl | ≥ 1280px | 252px | 宽屏 | 28px |
   493|
   494|**触控适配**：< 640px 所有可点击元素 ≥ 44px × 44px
   495|
   496|---
   497|
   498|## 10. 可访问性
   499|
   500|### 10.1 颜色对比度
   501|