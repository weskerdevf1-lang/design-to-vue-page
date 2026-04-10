# 设计稿 → Vue 页面生成

适用于 **Figma**（figma.com）和 **Zeplin**（zpl.io / app.zeplin.io）两种设计工具，skill 会根据链接自动识别工作流。

---

## 基础用法

```
请阅读并严格遵循 @skills/design-to-vue-page/SKILL.md 中的所有规则，
基于以下设计稿链接生成 Vue 页面：

设计稿链接：https://www.figma.com/design/xxx?node-id=xxx
输出路径：app/pages/[页面名]/index.vue
```

---

## 完整用法（推荐）

```
请阅读并严格遵循 @skills/design-to-vue-page/SKILL.md 中的所有规则，
基于以下设计稿链接生成 Vue 页面：

设计稿链接：https://www.figma.com/design/xxx?node-id=xxx
输出路径：app/pages/[页面名]/index.vue

要求：
- 1:1 还原设计稿，包括尺寸、间距、颜色、字体
- 响应式适配 1920 / 1440 / 1280px
- 所有样式用 rem（基准 20px），禁止 Less & 嵌套
- class 命名语义化 Kebab-case，禁止 Tailwind
- 图片文件名语义化，下载到 public/images/[页面名]/
```

---

## 修复 / 提升还原度

```
请阅读并严格遵循 @skills/design-to-vue-page/SKILL.md 中的所有规则，
对比以下设计稿与当前页面 @app/pages/[页面名]/index.vue 的差异并修复：

设计稿链接：https://www.figma.com/design/xxx?node-id=xxx
目标文件：app/pages/[页面名]/index.vue

重点检查：
- Hero/Banner 尺寸与定位（vw + aspect-ratio）
- 文字描边（-webkit-text-stroke + paint-order）
- 深色背景文字颜色
- flex 容器方向与 margin% 基准维度
- Less 样式中无 & 嵌套残留
```

---

## 注意事项

- prompt 里必须包含 `@skills/design-to-vue-page/SKILL.md` 的 `@` 引用，AI 才会读取完整 skill 规则
- **Figma** 设计稿坐标为 **2x**，换算时所有尺寸除以 2
- **Zeplin** 坐标为 **1x**，直接使用无需除以 2
- Zeplin：`exportable: true` 的图层调用 `download_layer_asset` 下载；`exportable: false` 的图层用 CSS 实现
- 图片下载后命名必须语义化，禁止使用 UUID 文件名
