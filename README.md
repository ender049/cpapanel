# CPA Panel

> 一个轻量的 [CLI Proxy API](https://github.com/router-for-me) 统一管理面板，单 HTML 文件，开箱即用。

## 在线访问

**[cpa.vlrat.top](https://cpa.vlrat.top)**

## 为什么做这个

当你有多个 CPA 服务端时，官方管理面板需要逐个打开不同网页切换查看，功能多但日常只用到一小部分。这个面板把多个服务端集中到一个页面，只保留最常用的功能。

## 功能

- **多服务端管理** — 添加/编辑/删除多个 CPA 服务端，统一查看
- **凭证管理** — 认证文件列表、配额查看（含恢复时间）、OAuth 登录、文件上传、Token 刷新
- **API Key 管理** — 查看/生成/复制/删除 API Key
- **网络设置** — 代理 URL、请求重试次数、路由策略配置
- **使用统计** — 模型/API Key/凭证维度统计，请求明细，可按时间范围和服务端筛选
- **自动刷新** — 可配置刷新间隔，倒计时按钮一键手动刷新
- **手机适配** — 响应式布局，移动端可用
- **数据安全** — 服务端配置简单混淆存储，运行时数据不缓存，每次从 API 实时获取

## 支持的提供商

Claude / Codex / Gemini CLI / Antigravity / Kimi

## 使用方法

直接在浏览器打开 `index.html`，或部署到任意静态托管服务。

添加服务端时填入 CPA 管理地址（如 `http://ip:8317/v0/management`）和密钥即可。

## 技术细节

- 纯前端单 HTML 文件，零依赖
- 所有数据通过 CPA Management API 实时获取
- 服务端配置存储在浏览器 localStorage（简单混淆）
- 由 GLM-5.1 辅助开发

## License

MIT
