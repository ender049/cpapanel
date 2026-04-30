# CPA Panel

> 一个轻量的 [CLI Proxy API](https://github.com/router-for-me) 统一管理面板，单 HTML 文件，开箱即用。

## 在线访问

**[ender049.github.io/cpapanel](http://ender049.github.io/cpapanel/)**

## 为什么做这个

当你有多个 CPA 服务端时，官方管理面板需要逐个打开不同网页切换查看，功能多但日常只用到一小部分。这个面板把多个服务端集中到一个页面，只保留最常用的功能。

## 功能

- **多服务端管理** — 添加/编辑/删除多个 CPA 服务端，统一查看
- **凭证管理** — 认证文件列表、配额查看（含恢复时间）、OAuth 登录、文件上传、Token 刷新、启用/禁用
- **凭证下载** — 支持下载 `CPA凭证`，以及将 Codex 凭证转换为 `auth.json`
- **API Key 管理** — 查看/生成/复制/删除 API Key
- **网络设置** — 代理 URL、请求重试次数、路由策略配置
- **自动刷新** — 可配置刷新间隔，倒计时按钮一键手动刷新
- **手机适配** — 响应式布局，移动端可用
- **数据安全** — 服务端配置简单混淆存储，运行时数据不缓存，每次从 API 实时获取

## 支持的提供商

Claude / Codex / Gemini CLI / Antigravity / Kimi

## 使用方法

直接在浏览器打开 `index.html`，或部署到任意静态托管服务。

添加服务端时填入 CPA 服务地址（如 `192.168.1.1:8317`）和密钥即可，端口可省略则默认 8317。

> **注意：** 如果通过 HTTPS 访问本面板（如 GitHub Pages），浏览器会阻止向 HTTP 的 CPA 服务端发请求。请通过 HTTP 访问面板（如 `http://your-domain`），或将面板文件下载到本地直接打开。

## 技术细节

- 纯前端单 HTML 文件，零依赖
- 所有数据通过 CPA Management API 实时获取
- 服务端配置存储在浏览器 localStorage（简单混淆）
- 由 GLM-5.1 辅助开发

## License

MIT
