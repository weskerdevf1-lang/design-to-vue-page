# 项目约定参考

## 技术栈

- **框架**: Nuxt 4 + Vue 3.5（SSR）
- **样式**: Tailwind CSS + Less（全局注入 `variable.less`）
- **组件库**: Element Plus
- **状态**: Pinia（`AppStore`）
- **路由**: 文件路由（`app/pages/`）

## 目录结构

```
app/
  pages/          # 页面文件
  components/     # 组件（c- 前缀）
  assets/
    styles/
      config/variable.less   # Less 变量
      index.less              # 全局样式入口
public/
  images/         # 静态图片资源
```

## 自定义组件

| 组件 | 用途 |
|------|------|
| `<c-request-view>` | 封装 API 请求 + loading/error 状态 |
| `<c-image>` | 图片组件（懒加载/占位） |
| `<c-btn>` | 按钮组件 |

## Less 变量（variable.less）

常用变量（实际值以文件为准）：
- `@primary` — 主色（粉色/品牌色）
- `@yellow` — 强调黄色
- `@blue-accent` — 蓝色
- `@text-main` — 主文字色
- `@text-sub` — 次要文字色

## 字体

```less
font-family: 'PingFang SC', 'Source Han Sans SC', sans-serif;
```

## 响应式断点

```
sm: 640px
md: 768px
lg: 1024px
xl: 1280px
```

## HttpUtil 示例

```ts
import { HttpUtil, ToastUtil } from '~/utils'

const { data } = await HttpUtil.post('/api/xxx', { param: value })
if (!data) return ToastUtil.error('请求失败')
```
