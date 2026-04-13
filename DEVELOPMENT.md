# CPA Panel 开发文档

## 项目概述

单 HTML 文件的 CLI Proxy API (CPA) 管理面板，纯前端零依赖，通过 CPA Management API 与服务端交互。面向拥有多个 CPA 服务端需要统一管理的用户。

- **仓库**: https://github.com/ender049/cpapanel
- **在线**: http://ender049.github.io/cpapanel/
- **唯一工作文件**: `index.html`（约 360 行，HTML+CSS+JS 单文件）

## 参考仓库

- 官方管理面板: https://github.com/router-for-me/Cli-Proxy-API-Management-Center
- 第三方: https://github.com/kongkongyo/Cli-Proxy-API-Management-Center
- API 文档: https://help.router-for.me/cn/management/api.html

## 文件结构

```
index.html                    — 唯一工作文件，全部代码
├── <style>                   — CSS（约 150 行）
├── <body> HTML               — 页面骨架 + 模态框
│   ├── header                — 标题栏 + 字体/刷新控制
│   ├── main-tabs             — 服务管理 / 使用统计 Tab
│   ├── credPanel             — 服务管理面板
│   ├── statsPanel            — 使用统计面板
│   ├── 模态框: sModal        — 添加/编辑服务端
│   ├── 模态框: dModal        — 删除确认
│   ├── 模态框: addCredModal  — 添加凭证（OAuth + 文件上传）
│   ├── 模态框: apiKeyModal   — API Key 管理
│   ├── 模态框: settingsModal — 网络设置
│   └── toast                 — 提示消息
└── <script>                  — JS 逻辑（约 200 行）
    ├── 变量声明              — 状态变量
    ├── 服务端 CRUD           — save/load/connect
    ├── 渲染函数              — renderCred/renderStats/renderCredSrv/renderFC
    ├── 配额渲染              — rCl/rCo/rGe/rAn/rKi
    ├── OAuth 流程            — startOAuth/pollOAuth/submitCallback
    ├── API Key 管理          — fetchApiKeys/addApiKey/delAk/genApiKey
    ├── 网络设置              — loadSettingsCfg/saveStCfg/clearStCfg
    ├── 自动刷新              — setAutoRefresh/startArCountdown/manualRefresh
    └── 工具函数              — fmtK/fmtTs/esc/maskKey/encStore/decStore
```

## 代码风格约定

- **极度压缩**: 变量名和函数名用缩写（`renderCredSrv`→`renderCredSrv` 渲染函数保留了可读名，内部变量用短名）
- **无注释**: 代码中不加注释
- **模板字面量**: 大量使用 JS 模板字符串拼接 HTML，注意避免在 `${}` 内部嵌套 `h+=` 操作（会导致模板闭合错误）
- **CSS 变量**: 主题色用 `--bg0/bg1/bg2/bdr/t1/t2/acc/ok/err/warn` 等 CSS 变量
- **组件 class**: `.b`(按钮) `.fi`(输入框) `.mo`(模态框) `.ld`(loading) `.tm`(次要文字) `.fbar`(筛选栏) `.tw`(表格容器) `.tbar`(汇总栏) `.sec-title`(统计区标题)

## 核心变量

```javascript
servers       // 服务端列表 [{id,name,url,key,status,files,usage}]
quotaCache    // 配额缓存 {serverId: {provider: data}}
usageCache    // 使用统计缓存 {serverId: usageData}
loadingFiles  // 正在加载配额的文件名集合
collapsed     // 折叠的服务端ID集合
editId        // 当前编辑的服务端ID
delId         // 当前删除确认的服务端ID
mainTab       // 当前Tab: 'cred' | 'stats'
timeRange     // 统计周期: '7h' | '24h' | '7d' | 'all'
statsServer   // 统计面板服务端筛选: 'all' | serverId
epExpanded    // API KEY 展开状态
detFilter     // 请求明细筛选 {model,cred,ep,srv}
refreshing    // 正在刷新的服务端ID集合
statsRefreshing // 统计是否正在刷新
statsFlash    // 统计面板是否需要闪烁
credFlash     // 凭证面板需要闪烁的服务端ID集合
statsCollapsed// 统计面板折叠的section key集合
firstLoad     // 是否首次加载
autoRefreshSec / autoRefreshCountdown / autoRefreshCountdownTimer
```

## API 接口映射

所有请求通过 `fetch()` 直接调用 CPA 服务端，请求头 `Authorization: Bearer <管理密钥>`。

| 功能 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 认证文件列表 | GET | `/v0/management/auth-files` | 返回 `{files: [...]}` |
| 下载文件 | GET | `/v0/management/auth-files/download?name=` | 返回 blob |
| 上传文件 | POST | `/v0/management/auth-files` | multipart `file` 字段 |
| 使用统计 | GET | `/v0/management/usage` | 返回 `{apis: {...}}` |
| 代理请求 | POST | `/v0/management/api-call` | body: `{authIndex,method,url,header,data}` |
| API Key 列表 | GET | `/v0/management/api-keys` | 返回 `{"api-keys": [...]}` |
| API Key 更新 | PUT | `/v0/management/api-keys` | body: 完整数组 |
| API Key 删除 | DELETE | `/v0/management/api-keys?index=N` | 按索引删除 |
| 代理URL | GET/PUT/DELETE | `/v0/management/proxy-url` | `{value: "socks5://..."}` |
| 重试次数 | GET/PUT/DELETE | `/v0/management/request-retry` | `{value: 3}` |
| 路由策略 | GET/PUT/DELETE | `/v0/management/routing/strategy` | `{value: "round-robin"}` |
| OAuth URL | GET | `/{provider}-auth-url?is_webui=true` | 返回 `{url, state}` |
| OAuth 状态 | GET | `/v0/management/get-auth-status?state=` | 轮询 |
| OAuth 回填 | POST | `/v0/management/oauth-callback` | `{provider, redirect_url}` |

## 关键数据结构

### 服务端 (servers[])
```javascript
{
  id: "timestamp_string",
  name: "生产环境",
  url: "http://192.168.1.1:8317",   // 自动补 http:// 和 :8317
  key: "管理密钥",
  status: "online" | "offline" | null,
  files: [{name, status, provider, email, ...}],
  usage: {total_requests, success_count, failure_count, total_tokens, apis: {...}}
}
```

### 认证文件
```javascript
{
  auth_index: "68c39aa6610af09a",
  status: "active",
  id_token: {chatgpt_account_id, plan_type},
  email: "xxx@outlook.com",
  provider: "codex",
  last_refresh: "2025-08-31T01:23:45Z"
}
```

### 使用统计
```
usage.apis -> 按端点分组 -> 每端点下 models -> 每个 model 下 details[]
detail: {timestamp, source, auth_index, failed, tokens: {input_tokens, output_tokens, reasoning_tokens, cached_tokens, total_tokens}}
```

## OAuth 流程

1. 前端调 `GET /{provider}-auth-url?is_webui=true`
2. CPA 启动本地 HTTP 服务接收回调
3. 前端显示授权 URL（不自动打开），用户手动复制/打开
4. 每 3 秒轮询 `GET /get-auth-status?state=xxx`
5. 备用：用户粘贴回调 URL 到 `POST /oauth-callback`

支持的提供商: anthropic, codex, gemini-cli, antigravity

## 配额提供商

| 提供商 | 渲染函数 | 配额结构 | 恢复时间字段 |
|--------|---------|---------|-------------|
| Claude | `rCl()` | `limits[key].remaining` | `limits[key].resets_at` |
| Codex | `rCo()` | `primary_window.used_percent` | `reset_at` / `reset_time` |
| Gemini CLI | `rGe()` | `buckets[].remaining_fraction` | `resetTime` / `reset_time` |
| Antigravity | `rAn()` | `models[key].remaining_fraction` | `resetTime` / `reset_time` |
| Kimi | `rKi()` | `used/limit` | `resetHint` |

## Token 计算规则

- **total_tokens**: 直接用 API 返回值，不自行计算
- **输入/输出/缓存/推理**: 从 detail 累加，**只统计成功请求**（failed 为 false）
- **cached_tokens**: 是 input_tokens 的子集
- **成功率**: success_count / total_requests * 100%

## 路由策略

官方支持: `round-robin`（轮询）、`fill-first`（填满优先）

## API Key 生成

```javascript
// 格式: sk- + 17位随机字符
function genApiKey() {
  const cs = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  const a = new Uint8Array(17);
  crypto.getRandomValues(a);
  return 'sk-' + Array.from(a, b => cs[b % cs.length]).join('');
}
```

## 刷新机制

### 自动刷新
- 可配置间隔: 30s / 60s / 2m / 5m，存 localStorage
- 倒计时显示在按钮上（如 `53s 刷新`）
- 倒计时归零后停止计时器，等 connectAll() 完成后再重新开始
- 手动点击刷新按钮时显示 loading 动画，数据返回后恢复倒计时

### 刷新范围
- 自动刷新: 认证文件列表 + 使用统计
- 不含: 配额查询（需手动点刷新按钮）

### 闪烁反馈
- `statsFlash`: 统计面板数据刷新后闪烁（data-flash CSS 动画）
- `credFlash`: 凭证面板按服务端ID闪烁
- 闪烁元素: `.tbar`(汇总栏)、`.tw`(表格容器)、`.su`(服务端统计栏)、`.sbdy`(文件区)
- 闪烁完成后自动清除标志

## 注意事项

### HTTPS 混合内容
- 页面通过 HTTPS 加载时，浏览器会阻止向 HTTP 的 CPA 服务端发 fetch 请求
- 解决方案: 通过 HTTP 访问面板，或将文件下载到本地打开
- README 中已有说明

### 服务端地址处理
```javascript
// 自动去掉 /v0/management 后缀
nu = u.replace(/\/v0\/management\/?$/, '');
// 自动补 http://
if (!nu.startsWith('http://') && !nu.startsWith('https://')) nu = 'http://' + nu;
// 自动补默认端口
if (!nu.match(/:\d+/)) nu += ':8317';
```

### 数据存储
- 服务端配置存在 localStorage，用 encStore/decStore 简单混淆（Base64 + 反转）
- 运行时数据（files/status/usageCache/quotaCache）不缓存，每次从 API 实时获取

### 移动端适配
- 768px 断点: `.tbar` 改为 flex-wrap 自然换行，减小间距和字号
- `.fg` 改为单列布局
- `.sb` 改为垂直排列

### 已知的代码坑
1. **模板字面量嵌套**: 统计面板的表格渲染不要在 `${stc?'':`...`}` 内部用 `h+=`，必须用局部变量 `t` 拼接后再追加到 `h`
2. **data-flash 不能硬编码**: `tbar` 的 `data-flash` 必须用 `${fl}` 变量，不能写死，否则每次渲染都闪
3. **配额代理请求**: 通过 `/v0/management/api-call` 代理，Authorization 用 `$TOKEN$` 占位符，CPA 服务端会替换为实际 token

## 开发命令

```bash
# 本地预览（任选一个）
python3 -m http.server 8080
npx serve .

# 验证 JS 语法
python3 -c "
import re
with open('index.html') as f: html = f.read()
m = re.search(r'<script>(.*?)</script>', html, re.DOTALL)
js = m.group(1)
depth = sum(1 if c=='{' else -1 if c=='}' else 0 for c in js)
print('Brace balance:', depth)
print('Backtick even:', js.count(chr(96)) % 2 == 0)
"

# 提交推送
git add index.html && git commit -m "描述" && git push
```
