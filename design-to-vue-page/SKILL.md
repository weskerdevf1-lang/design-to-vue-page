---
name: design-to-vue-page
description: 将 Figma 或 Zeplin 设计稿 1:1 还原为 Vue/Nuxt 页面，自动识别设计工具并调用对应 MCP 工具完成节点解析、图片资产提取、样式转换及代码生成。当用户提供 Figma 链接（figma.com）或 Zeplin 链接（zpl.io / app.zeplin.io）并要求生成页面、还原设计稿时使用此 skill。
---

# 设计稿 → Vue 页面生成

## 工具

| 设计工具 | MCP 服务 | 核心工具 |
|---------|---------|---------|
| Figma | `user-f2c-mcp` | `get_image(fileKey, nodeId, format)` · `get_code(fileKey, nodeId)` |
| Zeplin | `user-zeplin` | `get_screen(url)` · `download_layer_asset(id, path, type)` · `get_design_tokens(id)` |

---

## Step 1：识别设计工具

根据用户提供的 URL 判断工作流：

```
figma.com/design/...  → 执行【Figma 工作流】
zpl.io/...            → 执行【Zeplin 工作流】
app.zeplin.io/...     → 执行【Zeplin 工作流】
```

---

## Figma 工作流

### F1. 解析 URL

```
https://www.figma.com/design/{fileKey}/...?node-id={rawNodeId}
```

- `fileKey`：URL 路径第三段
- `nodeId`：`node-id` 参数值，将 `-` 替换为 `:`（如 `8359-26796` → `8359:26796`）

### F2. 获取整体设计图

```
get_image(fileKey, nodeId, format="jpg")
```

下载到 `public/images/{pageName}/` 留存参考，用 `sips` 裁剪局部区域分析各区块：

```bash
sips -c {height} {width} --padToHeightWidth {height} {width} -z {h} {w} input.jpg --out section.png
```

### F3. 分析节点结构

```
get_image(fileKey, nodeId, format="svg")   # 检查顶层 <g> 坐标/尺寸，以及 Pattern 背景
get_code(fileKey, nodeId)                  # 获取 F2C Tailwind/TSX 代码
```

若 `get_code` 超过 100k 字符，用 Python 分段提取：

```bash
python3 - <<'EOF'
import re
with open("figma_extracted/page/index.tsx") as f:
    content = f.read()
positions = re.findall(r'absolute top-\[(\d+)px\].*?left-\[(\d+)px\]', content)
texts = re.findall(r'"([^"]{2,50})"', content)
EOF
```

对大型 frame，通过子 `node-id` 单独提取各区域（Header/Banner/Footer）。

### F4. 提取图片资源

**图片命名规范和 2x/压缩规范见共享约定文件**（Step 最后必须读取）。

```
get_image(fileKey, nodeId, format="png", scaleSize=2)   # 必须 2x
```

F2C 自动保存到 `assets/` 的 UUID 文件名，需批量重命名再移入 `public/images/`：

```bash
DEST=public/images/aa; SRC=/tmp/figma_aa/assets
cp "$SRC/xxx-hero-bg.png" "$DEST/hero-bg.png"
for i in 1 2 3 4 5 6; do cp "$SRC/cover-s1-${i}.png" "$DEST/cover-original-${i}.png"; done
```

### F5. F2C 尺寸换算

**F2C 输出基于 2x 像素，实现时一律除以 2**：

| F2C 值 | 实现值 |
|--------|--------|
| `top-[128px]` | `top: 64px` |
| `w-[1440px]` | `width: 720px` |
| `text-[32px]` | `font-size: 16px` |

---

## Zeplin 工作流

### Z1. 获取设计数据

```
get_screen(url="https://zpl.io/xxxx", includeVariants=true)
```

返回包含 `variants[].layers` 的图层树 JSON。输出较大时写入文件，用 Read 工具 + offset/limit 分段读取。

### Z2. 解析图层数据

| type | 含义 | 关键字段 |
|------|------|---------|
| `group` | 容器/布局组 | `rect`, `layers` |
| `text` | 文字 | `content`, `textStyles` |
| `shape` | 矩形/圆形 | `rect`, `fills`, `borders` |

**Zeplin 输出为 1x 像素，无需除以 2**（与 F2C 不同）。

颜色：`fills[0].color → { r, g, b, a }` → `f"rgba({r}, {g}, {b}, {a})"`

### Z3. 分析布局结构

```python
def collect_texts(layers):
    for l in layers:
        if l.get("type") == "text":
            print(l["content"], l["rect"])
        for child in l.get("layers", []):
            collect_texts([child])
```

识别：顶部 Nav（y=0~100）、搜索栏、内容区、底部 Tab（y = 页面高-80~100）。

### Z4. 提取图片资产

**图片命名规范和 2x/压缩规范见共享约定文件**（Step 最后必须读取）。

```
download_layer_asset("5:36", "/path/to/public/images/bb/", "svg")
```

- `exportable: true` → 下载；`exportable: false`（形状/文字）→ CSS 实现
- 如需 2x，在 Zeplin 配置 @2x 导出规格，或用 `sips -Z 800 icon.png` 手动放大

---

## 生成 Vue 页面（两者共用）

**在生成代码前，必须先读取共享约定文件：**
`skills/design-to-vue-page/vue-page-conventions.md`

该文件包含以下所有通用编码约定：
- Vue 页面模板、`definePageMeta`、代码规范
- 语义化 HTML 标签、Element Plus 组件规范
- Class 命名规范（Kebab-case）、v-for 使用规范
- TypeScript 类型规范
- Less 样式规范（禁止 `&` 嵌套、完整禁止模式）
- 单位规范（px → rem，÷20）、保留 px 例外
- 响应式适配（页面容器、内容居中、卡片 Grid、弹性搜索框）
- Banner 比例（aspect-ratio）、Section 标签间距
- 绝对定位布局模式、浮层百分比定位、Marquee、mix-blend-mode
- Hero 装饰框尺寸自适应（vw + aspect-ratio + clamp）
- flex 容器规范（方向、margin % 维度、子元素 width）
- 自定义字体引入（禁止 npm 字体包）
- 深色背景文字颜色规范
- 文字描边规范（text-stroke、overflow:hidden 陷阱、装饰图层叠顺序）
- 图片命名规范、2x 规范、压缩规范

---

## Figma 专有规范

### 页面背景（Pattern / 纹理背景）

F2C 不包含 Frame Fill 背景，需从 SVG 手动提取，否则页面白屏：

```bash
curl -o /tmp/figma_page.svg "{svgUrl}"
python3 -c "
with open('/tmp/figma_page.svg') as f: c = f.read()
print('Has pattern' if 'pattern0_' in c else 'No pattern')
"
```

提取 base64 PNG：

```python
import base64
with open('/tmp/figma_page.svg') as f: content = f.read()
href_idx = content.find('xlink:href="data:image/png;base64,')
start = href_idx + len('xlink:href="data:image/png;base64,')
img_data = base64.b64decode(content[start:content.find('"', start)])
open('public/images/{page}/page-bg-pattern.png', 'wb').write(img_data)
```

从 `<pattern>` 的 `width/height`（objectBoundingBox 比例值）× 元素实际尺寸得到 CSS `background-size`。

### Hero 内部元素定位规范

**所有 Hero 内部绝对定位元素直接以 `.hero-bg` 为基准，禁止多级嵌套百分比链。**

F2C 常将标题区包裹在一个隐式中间容器内（如 `absolute top-[365.61px]`），必须**累加所有父层偏移**：

```
中间容器 top = 365.61px，title-box top = 42.53px，hero 高 = 735px
→ 实际 top = (365.61 + 42.53) / 735 = 55.53%  ✅
→ 直接用 42.53 / 735 = 5.79%  ❌（漏掉中间容器！）
```

坐标换算公式：

```
hero-relative-top  = (∑父容器top偏移 + element.top) / hero-height × 100%
hero-relative-left = (∑父容器left偏移 + element.left) / hero-width × 100%
vw-width           = element-width / design-width × 100
```

装饰框内文字边距换算为容器自身的百分比（不用 rem）：

```
padding-top  = F2C mt值 / 容器高 × 100%
padding-left = F2C ml值 / 容器宽 × 100%
```

### F2C 图片遗漏检查

```bash
for f in /tmp/figma_xx/assets/*.png; do
  sips -g pixelWidth -g pixelHeight "$f" 2>/dev/null | grep -E "pixel|$f"
done
```

逐一核对每张 UUID 图片在页面中的对应实现，特别注意局部容器右侧的小尺寸装饰图。

### 清理临时文件

```bash
rm -f figma_*.png figma_*.jpg figma_*.svg figma_node.svg
rm -rf figma_extracted/
```

---

## 注意事项

**Figma 专项：**
- F2C 第一次调用可能超时，直接重试
- 大文件超 100k 字符时用 Python 分段解析
- F2C `flex-col items-center` 必须对应 `flex-direction: column; align-items: center`，漏写 `flex-direction: column` 导致内框左对齐
- F2C 每次导出 UUID 文件名可能不同，但内容相同，跨 session 无需重新下载

**Zeplin 专项：**
- 输出较大时写入文件，用 Read + offset/limit 分段读取
- `designTokens` 可能为空对象，直接用图层颜色值
- 移动端页面建议 `width: 375px` 居中展示
- `exportable: false` 的形状/文字用 CSS 实现，不下载

**通用：**
- Banner 拉伸变形 → 避免 `background-size: 100% 100%`，改用 `aspect-ratio + cover`
- Section 标题偏位 → 检查设计稿是 `items-start` 还是 `items-center`，补全 `margin-top`
- `&` 嵌套残留 → `rg "&[.:#\[]|&\." app/pages/your-page/index.vue` 检查

## 详细参考

- 通用编码约定见 [vue-page-conventions.md](vue-page-conventions.md)
- 项目结构与变量见 [conventions.md](conventions.md)
