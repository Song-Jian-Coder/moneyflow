# Tasks

## 1. 安装依赖与 shadcn/ui 初始化

- [x] 1.1 在 `frontend/` 目录安装辅助工具库：运行 `npm install clsx tailwind-merge lucide-react date-fns`，验证 `package.json` 包含这些依赖
- [x] 1.2 安装 Tailwind 插件：运行 `npm install -D tailwindcss-animate @tailwindcss/typography`，验证 `package.json` devDependencies 包含这些包
- [x] 1.3 初始化 shadcn/ui：在 `frontend/` 目录运行 `npx shadcn@latest init --yes`，验证生成 `components.json` 配置文件且内容正确
- [x] 1.4 安装 shadcn/ui 基础组件：运行 `npx shadcn@latest add button card dialog input table badge popover select tooltip --yes`，验证 `components/ui/` 目录包含对应 `.tsx` 文件

## 2. Tailwind CSS v4 主题配置

- [x] 2.1 编辑 `frontend/app/globals.css`：在 `@import "tailwindcss"` 之后添加 `@theme` 块，定义品牌色（brand-50/100/500/600/700/900）、资金流向色（flow-inflow/outflow/neutral）和自定义动画（fade-in/slide-in-from-bottom 等），验证文件语法正确
- [x] 2.2 在 `frontend/app/globals.css` 添加 `:root` CSS 变量块：定义 shadcn/ui 主题变量（--background、--foreground、--primary、--card 等），验证变量定义完整
- [x] 2.3 在 `frontend/app/globals.css` 添加 `.dark` CSS 变量块：定义深色模式主题变量值，验证与 `:root` 变量名一致
- [x] 2.4 在 `frontend/app/globals.css` 添加全局基础样式：`* { border-color }` 和 `body { background-color; color }` 基础重置，验证文件语法正确
- [x] 2.5 在 `frontend/app/globals.css` 添加自定义滚动条样式（`::-webkit-scrollbar` 系列规则），验证文件语法正确
- [x] 2.6 在 `frontend/app/globals.css` 添加资金流向色工具类（`.text-flow-inflow`、`.bg-flow-inflow` 等）和文本截断工具类（`.text-truncate`、`.line-clamp-2`、`.line-clamp-3`），验证类名可用

## 3. next.config.ts 增强

- [x] 3.1 编辑 `frontend/next.config.ts`：添加 `images.remotePatterns` 配置（允许 `https` 协议的图片域名），验证文件语法正确
- [x] 3.2 在 `frontend/next.config.ts` 添加 `experimental.serverComponentsExternalPackages: ['echarts']`（为后续 ECharts 做准备），验证文件语法正确

## 4. 功能页面路由占位

- [x] 4.1 创建 `frontend/app/industry-flow/page.tsx`：展示"行业资金流 — 开发中"占位内容和返回首页链接，验证访问 `/industry-flow` 页面正常渲染
- [x] 4.2 创建 `frontend/app/market-flow/page.tsx`：展示"大盘资金流 — 开发中"占位内容和返回首页链接，验证访问 `/market-flow` 页面正常渲染
- [x] 4.3 创建 `frontend/app/ai-assistant/page.tsx`：展示"AI 助手 — 开发中"占位内容和返回首页链接，验证访问 `/ai-assistant` 页面正常渲染

## 5. 目录结构扩展

- [x] 5.1 创建空目录占位：`frontend/components/homepage/`、`frontend/hooks/`、`frontend/public/images/`、`frontend/public/icons/`。由于 Git 不跟踪空目录，在每个目录创建 `.gitkeep` 文件，验证目录存在
- [x] 5.2 创建 `frontend/lib/utils.ts`：实现 `cn()` 函数（clsx + tailwind-merge），这是 shadcn/ui 组件的依赖，验证 TypeScript 编译无错误

## 6. 环境变量配置

- [x] 6.1 创建 `frontend/.env.local`：定义 `NEXT_PUBLIC_API_URL=http://localhost:8080` 和 `NEXT_PUBLIC_APP_TITLE=MoneyFlow`，验证文件存在
- [x] 6.2 创建 `frontend/.env.example`：复制 `.env.local` 内容作为模板（不含敏感值），验证文件存在
- [x] 6.3 确认 `frontend/.gitignore` 包含 `.env.local`（排除本地环境变量），验证 Git 不会追踪 `.env.local`

## 7. 构建与验证

- [x] 7.1 运行 `cd frontend; npm run build`，验证 Next.js 生产构建成功，无 TypeScript 编译错误，输出 `.next/` 目录
- [x] 7.2 运行 `cd frontend; npm run dev`，验证开发服务器启动在 `http://localhost:3000`，首页正常渲染
- [x] 7.3 验证路由：分别访问 `/`、`/todos`、`/industry-flow`、`/market-flow`、`/ai-assistant` 和不存在的 `/404-test`，确认所有页面正常渲染或展示 404
- [x] 7.4 验证 shadcn/ui 组件：在首页临时导入 `<Button>` 组件，确认渲染正常后移除测试代码
- [x] 7.5 验证 Tailwind 主题色：在首页临时使用 `text-brand-600`、`text-flow-inflow` 等类名，确认颜色正确渲染后移除测试代码
- [x] 7.6 验证环境变量：在首页通过 `process.env.NEXT_PUBLIC_APP_TITLE` 读取并展示，确认值正确后移除测试代码

## Workflow follow-up

- 归档变更：实现完成并通过审查后执行 `openspec archive`
- 同步规范：归档时将 delta spec 合并到 `openspec/specs/frontend-framework/spec.md`
