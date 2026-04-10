design-to-vue-page Skill 说明
功能概述
将 Figma 或 Zeplin 设计稿 1:1 还原为 Vue/Nuxt 页面。自动识别设计工具类型，调用对应 MCP 工具完成节点解析、图片资产提取、样式转换与代码生成。

适用场景
用户提供 Figma 链接（figma.com/design/...）并要求生成页面
用户提供 Zeplin 链接（zpl.io/... 或 app.zeplin.io/...）并要求生成页面
要求还原设计稿、1:1 实现 UI、按设计图生成 Vue 文件
工具支持
设计工具	MCP 服务	核心能力
Figma	user-f2c-mcp	获取设计图（JPG/PNG/SVG）、F2C 代码提取
Zeplin	user-zeplin	获取屏幕图层树、下载可导出资产、读取 Token
工作流程
1. 识别 URL → 判断 Figma / Zeplin
2. 解析设计数据（节点结构 / 图层树）
3. 提取图片资产（2x 规范、命名规范、压缩）
4. 读取 vue-page-conventions.md（通用编码约定）
5. 生成 Vue 页面代码（模板 + Less 样式）
关键规则
规则	Figma	Zeplin
坐标单位	F2C 输出 2x，实现时 ÷2	1x，直接使用
图片导出	scaleSize=2（必须 2x）	@2x 导出规格或 sips 放大
尺寸换算	px ÷ 20 = rem	px ÷ 20 = rem
中间容器偏移	必须累加所有父层偏移量	直接读 rect 坐标
背景纹理	需从 SVG 手动提取 Pattern	图层 fills 直接获取
依赖文件
文件	用途
vue-page-conventions.md	通用编码约定（17 个规范节，生成代码前必读）
conventions.md	项目结构、Less 变量、路径约定
触发示例
请根据这个 Figma 链接生成页面：https://www.figma.com/design/xxx/...
请按 Zeplin 设计稿还原这个页面：https://zpl.io/xxxx
