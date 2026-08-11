- **`npm start` 时：**  
    你访问 `http://localhost:3000` → Dev Server 收到请求 → 它在**内存中**把 `App.tsx` 即时转成 `App.js` → 把结果“流式传输”给浏览器 → 浏览器渲染。  
    **关键**：你电脑的硬盘上**根本没有**这个 `App.js` 文件，它在内存里，一关机就消失。
- **`npm run build` 时：**  
    构建工具**提前**把所有 `.tsx`、`.scss`、图片全部翻译、压缩、合并 → 实实在在地**写入硬盘**的 `dist` 文件夹 → 你要把这个文件夹里的**真实文件**上传到服务器。

jsx，tsx源码必须经过构建工具打包成浏览器可识别的html、css和js。

Next.js 的 `next build` 命令，除了这些基础工作，还额外做了两件大事[-2](https://nextjs.org/docs/app/guides/building)[-6](https://nextjs.org/docs/15/pages/building-your-application/rendering.md)：

1. **预渲染 (Prerendering)**：Next.js 默认会**提前**把一些页面生成好完整的 HTML 文件。这个 HTML 已经包含了页面的内容，而不是像普通 React 项目那样，只有一个空的 `<div id="root">`[-3](https://nextjs.org/docs/pages/building-your-application/rendering?utm_source=www.saaseliteacademy.com&utm_medium=referral&utm_campaign=mastering-saas-scalability-7-crucial-steps-for-building-a-strong-architecture)[-6](https://nextjs.org/docs/15/pages/building-your-application/rendering.md)。这有助于搜索引擎爬虫获取网页内容，因此对SEO友好。
    
2. **为不同“运行时”打包**：Next.js 的构建产物，会明确区分哪些代码是给**服务器**用的，哪些是给**浏览器（客户端）**用的，因为它的渲染可以发生在两个地方[-9](https://nextjs.org/docs/app/getting-started/server-and-client-components)。