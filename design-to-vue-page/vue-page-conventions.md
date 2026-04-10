# Vue 页面生成 — 共享约定

本文件被 `figma-to-vue-page` 和 `zeplin-to-vue-page` 两个 skill 共同引用。
在执行"生成 Vue 页面"步骤前，必须完整读取本文件。

---

## 一、图片资产规范

### 命名规范

**禁止使用 UUID 或哈希值作为文件名**，必须根据图片的视觉含义或用途命名（Kebab-case）：

| 场景 | 命名示例 |
|------|---------|
| 页面主背景 | `hero-bg.png`、`banner-bg.png` |
| 网站 Logo | `logo.png`、`logo-white.png` |
| 区块标题标签 | `section-label-original.png`、`section-label-recommend.png` |
| 区块装饰图标 | `section-icon-1.png`、`section-icon-heart.png` |
| 查看更多按钮 | `btn-more.png`、`btn-more-pink.png` |
| 导航菜单图标 | `nav-icon-home.png`、`nav-icon-category.png` |
| 轮播箭头 | `arrow-left.png`、`arrow-right.png` |
| 返回箭头 | `icon-back.svg` |
| 搜索图标 | `icon-search.svg` |
| Tab 图标（激活） | `icon-tab-home-active.svg` |
| 角色/装饰插图 | `char-hero.png`、`decor-star.png` |
| 漫画封面（占位） | `cover-1.png`、`cover-2.png` |
| Footer 背景 | `footer-bg.png` |
| 统计图标 | `icon-stat-view.png`、`icon-stat-like.png` |
| 返回顶部 | `btn-back-to-top.png` |

完整语义命名参考：

| 元素类型 | 语义文件名模式 |
|---|---|
| Hero 背景 | `hero-bg.png` |
| Hero 标题装饰框 | `hero-title-frame.png` |
| Hero 副标题装饰框 | `hero-subtitle-frame.png` |
| Hero 装饰星/光效 | `hero-deco-star-left.png`、`hero-subtitle-shine.png` |
| 分类 tab 图标 | `cat-icon-romance.png`、`cat-arrow-romance.png` |
| 区块标签 | `section-{name}-label.png` |
| 区块图标 | `section-{name}-icon.png` |
| 更多按钮 | `btn-more-{name}.png` |
| 轮播箭头 | `icon-arrow-prev.png`、`icon-arrow-next.png` |
| 通用图标 | `icon-search.png`、`icon-history.png` |
| 漫画封面 | `cover-{section}-{n}.png` |
| Footer 图标 | `footer-icon-service.png`、`footer-logo.png` |

验证：完成后用 rg 确认页面中无 UUID 引用：

```bash
rg "[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}" app/pages/your-page/index.vue
# 无输出 = 合格
```

### 2x 图与图片压缩

所有图标、背景图、装饰图等图片资产**必须使用 2x 分辨率**，确保在 Retina 屏幕上清晰显示。
CSS 中按**设计稿标注 1x 尺寸**设置显示尺寸：

```less
// 图片实际像素 400×200px（2x），CSS 按 1x 尺寸展示
.section-icon {
  width: 200px;   // 400 / 2
  height: 100px;  // 200 / 2
}

// 背景图按 1x 尺寸设置 background-size
.hero-bg {
  background-image: url('/images/page/hero-bg.png');
  background-size: 720px 360px;  // 1x 尺寸
}
```

所有超过 **100KB** 的图片资产放入 `public/images/` 前**必须压缩**：

```bash
# PNG 压缩（brew install pngquant）
pngquant --quality=80-95 --speed 1 --force --ext .png public/images/page/hero-bg.png

# JPG 压缩（sips macOS 内置）
sips -s format jpeg -s formatOptions 85 input.jpg --out output.jpg

# 批量压缩目录下所有超过 100KB 的 PNG
for f in public/images/page/*.png; do
  size=$(stat -f%z "$f")
  if [ "$size" -gt 102400 ]; then
    pngquant --quality=80-95 --speed 1 --force --ext .png "$f"
    echo "Compressed: $f"
  fi
done
```

> `pngquant` 有损压缩（视觉无损），通常减小 60–80% 体积。

---

## 二、Vue 页面模板

```vue
<template>
  <div class="page-container">
    <!-- 各模块 -->
  </div>
</template>

<script lang="ts" setup>
definePageMeta({ layout: 'empty' })
</script>

<style lang="less" scoped>
@import '@/assets/styles/config/variable.less';

.page-container {
  width: 100%;
  min-height: 100vh;
  background: #fff;
}
</style>
```

### 代码规范

- 独立页面使用 `definePageMeta({ layout: 'empty' })`
- 样式用 **Less scoped**，引入 `variable.less` 使用 `@primary`、`@yellow` 等变量，**禁止在 template 中用 Tailwind class 定义样式**
- 图片使用 `<img>` 或 `<c-image>` 组件，路径从 `/images/` 开始
- 动态数据占位用 `v-for` + 静态数组，真实接口后改接 `c-request-view`

---

## 三、语义化标签规范

根据内容含义选用对应的 HTML 语义化标签，**禁止全部使用 `<div>`**：

| 场景 | 使用标签 |
|------|----------|
| 页面主体区域 | `<main>` |
| 顶部导航栏 | `<header>` + `<nav>` |
| 底部信息栏 | `<footer>` |
| 独立内容区块 | `<section>` |
| 文章/卡片内容 | `<article>` |
| 侧边栏/辅助内容 | `<aside>` |
| 导航链接列表 | `<nav>` + `<ul>` + `<li>` + `<a>` |
| 按钮操作 | `<button>` |
| 标题文字 | `<h1>` ~ `<h6>` |
| 段落文字 | `<p>` |
| 图片 | `<img>` |
| 表单 | `<form>` + `<input>` / `<label>` |

```vue
<!-- ✅ 正确：语义化结构 -->
<header class="home-header">
  <img class="home-header-logo" src="/images/logo.png" alt="logo" />
  <nav class="home-header-nav">
    <ul class="home-header-nav-list">
      <li v-for="item in navList" :key="item.path" class="home-header-nav-item">
        <a :href="item.path">{{ item.label }}</a>
      </li>
    </ul>
  </nav>
</header>

<main class="home-main">
  <section class="home-banner">...</section>
  <section class="home-section">...</section>
</main>

<footer class="home-footer">...</footer>

<!-- ❌ 错误：全部 div 无语义 -->
<div class="header">
  <div class="nav"><div class="nav-item">首页</div></div>
</div>
```

---

## 四、Element Plus 组件规范

优先使用 Element Plus 组件替代手写 UI 原语，项目已全局注册无需单独 import：

| 场景 | 使用组件 |
|------|----------|
| 轮播/Banner | `<el-carousel>` + `<el-carousel-item>` |
| Tab 切换 | `<el-tabs>` + `<el-tab-pane>` |
| 弹出层/模态框 | `<el-dialog>` |
| 气泡提示 | `<el-tooltip>` |
| 下拉菜单 | `<el-dropdown>` |
| 搜索输入框 | `<el-input>` |
| 分页 | `<el-pagination>` |
| 骨架屏 | `<el-skeleton>` |
| 图片懒加载 | `<el-image>` |
| 按钮 | `<el-button>`（或项目封装的 `<c-btn>`） |

自定义组件样式时，用 `:deep()` 穿透：

```less
.home-banner {
  :deep(.el-carousel__arrow) {
    background: rgba(0, 0, 0, 0.5);
  }
}
```

---

## 五、Class 命名规范

所有 class 名称必须遵循 **语义化 Kebab-case 格式**，用连字符分隔单词和层级，**不使用 Tailwind 工具类**。

命名策略：以页面/区块名称为前缀，逐层向下细化：
- 页面根：`{page}-page`（如 `home-page`）
- 区块：`{page}-{block}`（如 `home-header`、`home-banner`、`home-footer`）
- 区块内元素：`{page}-{block}-{element}`（如 `home-header-logo`、`home-footer-link`）

```vue
<!-- ✅ 正确 -->
<div class="home-header">
  <nav class="home-header-nav">
    <a class="home-header-nav-item">首页</a>
  </nav>
</div>

<!-- ❌ 错误：Tailwind 工具类、无语义名称、下划线命名 -->
<div class="flex items-center gap-4">
<div class="box1">
<div class="header_nav">
```

---

## 六、v-for 使用规范

根据 UI 结构识别可复用的列表/网格，提取为数组用 `v-for` 渲染，避免重复硬编码 DOM：

```vue
<script lang="ts" setup>
interface NavItem { label: string; path: string; active?: boolean }
const navList = ref<NavItem[]>([
  { label: '首页', path: '/', active: true },
  { label: '分类', path: '/category' },
])
</script>

<template>
  <nav class="home-header-nav">
    <a
      v-for="item in navList"
      :key="item.path"
      :class="['home-header-nav-item', { 'is-active': item.active }]"
    >{{ item.label }}</a>
  </nav>
</template>
```

适合 `v-for` 的场景：导航菜单、Tab 切换、卡片网格、轮播 Dot、跑马灯列表、Footer 链接组。

---

## 七、TypeScript 类型规范

所有 `ref`、`reactive`、函数参数和返回值均须显式标注类型：

```ts
// ✅ ref 加泛型
const activeTab = ref<string>('today')
const bannerIndex = ref<number>(0)

// ✅ 复杂对象先定义 interface
interface SectionData {
  id: number
  title: string
  icon: string
  cards: ComicCard[]
}
const sections = ref<SectionData[]>([])

// ✅ computed 标注返回类型
const currentSection = computed<SectionData | undefined>(
  () => sections.value.find(s => s.id === activeId.value)
)

// ❌ 不加类型
const tab = ref('today')
const list = ref([])
```

---

## 八、Less 样式规范

### 禁止 `&` 嵌套

**所有选择器必须完整书写**，便于全局搜索和阅读：

```less
// ✅ 正确：完整选择器，无 &
.home-header { width: 100%; height: 3.2rem; }
.home-header-nav { display: flex; align-items: center; }
.home-header-nav-item { font-size: 0.8rem; color: #fff; }
.home-header-nav-item.is-active { background: @pink; }

// ❌ 错误：使用 & 嵌套
.home-header {
  &-nav { &-item { &.is-active { } } }  // 禁止
}
```

完整禁止模式：

| 禁止写法 | 正确写法 |
|---|---|
| `.parent { &.is-active { } }` | `.parent.is-active { }` |
| `.parent { &:first-child { } }` | `.parent:first-child { }` |
| `.parent { &:last-child { } }` | `.parent:last-child { }` |
| `.parent { img { } }` | `.parent img { }` |
| `.parent { span { } }` | `.parent span { }` |
| `.parent { a { } }` | `.parent a { }` |

`@media` 块内的选择器同样禁止 `&` 嵌套。

检验方式：

```bash
rg "&[.:#\[]|&\." app/pages/your-page/index.vue
# 无输出 = 合格
```

### 单位规范：px → rem

**所有样式尺寸值必须使用 `rem`，禁止直接使用 `px`（以下情况除外）。**

项目 `global.less` 响应式根字号：

| 分辨率 | html font-size | 说明 |
|---|---|---|
| 1920px（设计基准） | 20px | 1rem = 20px |
| 1440–1919px | 18px | 自动缩小到 90% |
| 768–1439px | 16px | 自动缩小到 80% |

以 **20** 为除数换算：`rem = px / 20`

```
64px → 3.2rem   32px → 1.6rem   24px → 1.2rem   16px → 0.8rem
14px → 0.7rem   12px → 0.6rem    8px → 0.4rem    4px → 0.2rem
```

保留 px 的例外情况：

| 情形 | 说明 |
|---|---|
| `1px` 细线边框 | 保持物理像素精度 |
| `@media (max-width: 1440px)` 断点 | 媒体查询始终用 px |
| `min-width: 1200px` 页面最小宽度 | 结构性约束 |
| `@content-width: 1440px` 变量 | 用于 `calc()` + vw |

clamp 中的 px 也要转换：

```less
// ✅ 正确
font-size: clamp(2.4rem, 5vw, 4.8rem);   /* 48/20 ~ 96/20 */
// ❌ 错误
font-size: clamp(48px, 5vw, 96px);
```

检验脚本：

```bash
python3 - << 'EOF'
import re
with open('app/pages/your-page/index.vue') as f:
    content = f.read()
style = content[content.find('<style'):content.rfind('</style>')]
found = [(m.group(), m.start()) for m in re.finditer(r'\d+(?:\.\d+)?px', style)
         if not re.search(r'@media|min-width:\s*1[0-9]{3}|@content-width',
                          style[max(0, m.start()-50):m.start()])]
for val, _ in found[:20]:
    print(val)
print(f'Total: {len(found)}')
EOF
```

---

## 九、响应式适配规范

设计稿基于 **1920px** 宽度，内容区域宽 **1440px**，**禁止使用固定 `width: 1920px` 锁死页面宽度**。

```less
// 页面根容器
.page-root {
  min-width: 1200px;
  width: 100%;
  min-height: 100vh;
}

// 内容区居中方案
@content-width: 1440px;
@content-side-padding: 24px;

.page-content-inner {
  max-width: @content-width;
  width: 100%;
  margin: 0 auto;
  padding: 0 @content-side-padding;
  box-sizing: border-box;
}

// 全宽容器内对齐内容边缘
.ticker-content-start {
  margin-left: max(@content-side-padding, calc((100vw - @content-width) / 2));
}

// 卡片网格 — 禁止固定宽度，用 Grid
.comic-section-cards {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 1.1rem;
}

.comic-section-card-cover {
  width: 100%;
  aspect-ratio: 3 / 4;
  height: auto;
  object-fit: cover;
}

// 弹性搜索框
.page-header-search {
  flex: 1;
  min-width: 200px;
  max-width: 614px;
}

// 固定宽度块安全降级
.footer-divider {
  width: min(1074px, calc(100% - 48px));
}
```

媒体查询断点（只调整 layout，不重复覆盖 vw/clamp 已处理的值）：

```less
@media (max-width: 1440px) {
  .page-section-cards { gap: 0.8rem; }
}

@media (max-width: 1280px) {
  .page-section-cards { grid-template-columns: repeat(5, 1fr); }
  .page-header-nav-item { padding: 0 0.8rem; font-size: 0.7rem; }
}
```

---

## 十、Banner / 轮播图比例

**禁止固定高度 + `background-size: 100% 100%`**，改用 `aspect-ratio` 等比缩放：

```less
// ✅ 正确
.page-banner {
  width: 100%;
  aspect-ratio: 1920 / 735;
  position: relative;
  overflow: hidden;
}

.page-banner-bg {
  width: 100%;
  height: 100%;
  background: url('/images/{page}/banner-bg.png') center / cover no-repeat;
}
```

Banner 内部绝对定位子元素改为**百分比定位**：

```less
.page-banner-arrow { top: 41%; }         /* 303 / 735 */
.page-banner-dots  { bottom: 2.7%; }     /* 20 / 735 */
```

---

## 十一、Section 标签标题间距

当 Section 头部使用背景图作为标签形状，标题文字对齐方式需从设计稿精确提取：

```less
// ✅ 正确：顶部对齐 + margin-top
.section-label {
  display: flex;
  align-items: flex-start;   // 不是 center！
  width: 13.75rem;
  height: 4.2rem;
}

.section-label-title {
  margin-top: 0.85rem;       // 从设计稿量取换算
  margin-left: 3.6rem;
  font-size: 1.6rem;
  white-space: nowrap;
}
```

---

## 十二、布局辅助规范

### 绝对定位布局模式

```less
.page-section {
  position: relative;
  width: 100%;
  aspect-ratio: 1920 / 367;  // 等比缩放，禁止写死高度 px
}

.page-section-inner {
  position: absolute;
  top: 8.7%;    // 32 / 367
  left: 0.83%;  // 16 / 1920
}
```

### 通知弹层 / 浮层横向定位

设计稿中 `right: 227px`（以 1920px 为基准）换算为百分比：

```
right: 11.82%;   /* 227 / 1920 × 100% */
```

通用公式：`百分比 = 原始 px 值 / 设计稿宽 × 100%`

### Marquee 跑马灯（Ticker）

```less
@keyframes marquee {
  0%   { transform: translateX(0); }
  100% { transform: translateX(-50%); }
}
.ticker-inner {
  display: flex;
  animation: marquee 20s linear infinite;
}
```

### 光效/叠加图层（mix-blend-mode）

设计中 `mix-blend-mode: screen` / `hard-light` 直接用 CSS 同名属性实现：

```less
.deco-star-right {
  mix-blend-mode: hard-light;
  pointer-events: none;
}
.subtitle-shine {
  position: absolute;
  mix-blend-mode: screen;
  pointer-events: none;
}
```

---

## 十三、Hero 内部绝对定位元素尺寸自适应

Hero 区域内所有装饰框宽高不能写死像素，需要随 viewport 等比缩放：

```less
/* 宽度用 vw：原始宽度 / 设计稿总宽 × 100 */
.hero-title-box {
  width: 26.77vw;           /* 514 / 1920 × 100 */
  aspect-ratio: 514 / 129;  /* height 由此自动计算，不要同时写 height */
  background: url('...') center / 100% 100% no-repeat;
}

/* 字号用 clamp 防止过小 */
.hero-title-text {
  font-size: clamp(2.4rem, 5vw, 4.8rem);  /* 48/20 ~ 96/20 */
}
```

- 不要在媒体查询中重复覆盖 `width` / `height` / `font-size`；`vw` + `clamp` 已连续适配

---

## 十四、flex 容器规范

### flex 方向与 align-items 须与设计完全一致

| 设计值 | CSS 等价 |
|---|---|
| flex-col | `flex-direction: column` |
| items-center | `align-items: center` |
| items-start | `align-items: flex-start` |
| justify-start | `justify-content: flex-start` |

**漏写 `flex-direction: column` 会导致内框左对齐而非居中。**

### flex 容器内 margin % 的正确参考维度

CSS 中 `margin-top/bottom` 的百分比**相对于容器宽度**（非高度）：

```
margin-top:  px值 / 容器宽度px × 100%
margin-left: px值 / 容器宽度px × 100%
```

> 注意：`position: absolute` 元素的 `top/bottom %` 相对**定位祖先的高度**，规则不同。

### flex 子元素 width 不能省略

设计稿中若子元素有明确宽度，必须换算为 `%` 并显式设置，否则 flex 子元素尺寸折叠：

```less
.subtitle-upgrade-wrap {
  width: 46.57%;   /* 193 / 414.46 — 不能省略 */
  aspect-ratio: 193 / 96;
  position: relative;
}
```

---

## 十五、自定义字体引入

特殊字体在 `app/assets/styles/config/font.less` 中声明：

```less
@font-face {
    font-family: 'MF YuanHei (Noncommercial)';
    src: url('@/assets/fonts/MFYuanHei-Noncommercial.woff2') format('woff2'),
         url('@/assets/fonts/MFYuanHei-Noncommercial.ttf') format('truetype');
    font-weight: normal;
    font-style: normal;
    font-display: swap;
}
```

Fallback 栈：`'MF YuanHei (Noncommercial)', 'PingFang SC', 'Source Han Sans SC', sans-serif;`

- 商业字体需购买授权；`Noncommercial` 版本仅供个人非商业使用
- **禁止引入 npm 字体包**（@fontsource/xxx），应提供实际字体文件

---

## 十六、深色背景文字颜色规范

在深色背景（深紫、深蓝、黑色）上，如果不显式设置 `color`，文字会继承默认黑色导致不可见：

- 未激活 nav 项：`color: rgba(178, 178, 178, 1)`
- 激活 nav 项：`color: rgba(255, 255, 255, 1)`
- 搜索框 placeholder：`color: rgba(163, 166, 181, 1)`

Footer 联系图标：**对所有同类图标统一样式**，禁止只用 `:first-of-type`：

```less
// ✗ 错误
.contact-icon:first-of-type { background: @pink; }

// ✓ 正确
.contact-icon {
  background: @pink;
  border: 0.1rem solid #000;
  border-radius: 50%;
}
```

---

## 十七、文字描边规范（Text Stroke / 包边）

设计工具导出通常**不包含** Text Stroke 信息（只保留 text-shadow），若设计稿字符四周有均匀硬边对比色，需手动补加。

### CSS 实现

```css
-webkit-text-stroke: 0.3rem rgba(255, 255, 255, 1);
paint-order: stroke fill;          /* stroke 先画，fill 后画，保证文字颜色不被覆盖 */
text-shadow: 0.15rem 0.15rem 0 rgba(38, 63, 191, 1);
```

### 描边宽度估算

`paint-order: stroke fill` 时，实际可见外描边 = 总宽 ÷ 2：

| 字号（设计稿） | 目标可见外描边 | `-webkit-text-stroke` 值 |
|---|---|---|
| ~48px | 2–3px | 0.2–0.3rem |
| ~72px | 5–6px | 0.5–0.6rem |
| ~96px | 10px | 1.0rem |
| ~120px | 12px | 1.2rem |

### 关键注意事项

- **字体 weight**：漫画/游戏风格标题必须加 `font-weight: 900`
- **line-height**：带描边大字标题必须保持 `line-height: normal`，禁止改为 `line-height: 1`
- **禁用 overflow:hidden**：当文字为 `position:absolute; top:0; left:0` 时，直接父容器不能加 `overflow:hidden`，否则描边会被裁切。为裁切图片叠加层，单独增加一个裁切容器：

```html
<div class="text-wrap">
  <span class="upgrade-text">升级</span>
  <div class="shine-clip">  <!-- 专门裁切容器 -->
    <img class="shine" src="..." />
  </div>
</div>
```

```less
.text-wrap { position: relative; /* 不加 overflow:hidden */ }
.shine-clip { position: absolute; inset: 0; overflow: hidden; pointer-events: none; }
.upgrade-text {
  position: absolute; top: 0; left: 0;
  -webkit-text-stroke: 0.8rem rgba(255, 255, 255, 1);
  paint-order: stroke fill;
  line-height: normal;
}
```

- **装饰图层叠顺序**：装饰图必须放在 title-box、subtitle-box **之后**，DOM 顺序即 z 轴顺序
