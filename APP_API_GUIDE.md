# OMA App API 连接器 — 功能介绍与使用说明

面向「已经提供 HTTP API 的应用」。不要用浏览器去点这些应用的按钮。

模型只看到三个工具：`list_apps`、`app_describe`、`app_invoke`。  
新接一个 App = 在工作区放一份 YAML，**不必改 Python、不必重新打包（源码运行时）**。

---

## 1. 它是什么、不是什么

| 是 | 不是 |
|---|---|
| 声明式 REST/HTTP 转接器 | 每个 App 写一套 SDK |
| 白名单 op + origin + 环境变量鉴权 | 任意 URL 的万能 HTTP 客户端 |
| 和 Runtime 同一条审批 / trace / finish 链路 | 另起的 Computer Use 循环 |
| 有官方 API 时的首选 | 替代 `browser_*`（那是给没 API 的网页用的） |

决策：

```
目标能不能用官方 HTTP API？
  能 → list_apps → app_describe → app_invoke
  不能 → browser_open / snapshot / click / type
本地文件 → read_file / write_file / bash
```

---

## 2. 清单放哪

按优先级从低到高扫描，**同名 App 以后者为准**：

```
工作区/apps/*.yaml
工作区/.opencode/apps/*.yaml     ← 旧路径，继续兼容
工作区/.oma/apps/*.yaml          ← 推荐，覆盖前两处
```

GitHub 推荐路径：

```
<工作区>/.oma/apps/github.yaml
```

例如 Windows：

```
C:\Users\<你>\Desktop\oma-core\.oma\apps\github.yaml
```

只识别 `.yaml` / `.yml`。改清单后**下一轮新任务**就会被 `list_apps` 读到（每次 `app_invoke` 都会重新扫盘）。

---

## 3. 三个工具

### 3.1 `list_apps`

无参数。返回工作区里所有清单的摘要：name、description、base_url、ops 列表。

### 3.2 `app_describe`

```json
{ "app": "github" }
```

返回该 App 每个 op 的 `method`、`path`、`write`（POST/PUT/PATCH/DELETE 为 true）、`description`。

### 3.3 `app_invoke`

```json
{
  "app": "github",
  "op": "create_issue",
  "path_params": { "owner": "huanglynch", "repo": "MultiAgentSwarm" },
  "query": {},
  "body": { "title": "demo", "body": "from OMA" },
  "headers": {}
}
```

| 字段 | 含义 |
|---|---|
| `app` | 清单里的 `name` |
| `op` | 清单 `ops` 下的键 |
| `path_params` | 填充 path 里的 `{owner}` `{repo}` 等 |
| `query` | URL 查询串 |
| `body` | JSON 请求体（写操作常用） |
| `headers` | 额外头；**不能覆盖 Authorization** |

约束：

- 只能打清单里出现过的 op，不能拼任意 path
- 最终 URL 的 host 必须落在 `allow_origins`（未写则默认 `base_url` 的 host）
- 只允许 `http://` / `https://`（由 `base_url` + path 拼出）
- token 从环境变量读取，不进模型上下文
- 响应体会按 `tool_result_max_chars` 截断

返回大致为：

```json
{
  "ok": true,
  "app": "github",
  "op": "me",
  "method": "GET",
  "url": "https://api.github.com/user",
  "status": 200,
  "write": false,
  "body": { "...": "github json" }
}
```

`ok` 在 HTTP 2xx 时为 true。4xx/5xx 仍返回 body，便于排错。

orchestrator 角色**没有**这三个工具，由 worker / main 调用。

---

## 4. 清单字段

```yaml
name: github                 # list_apps / app_invoke 用的名字
description: 给人看的一句话
base_url: https://api.github.com
timeout_sec: 30              # 可选，默认约 30
allow_origins:               # 可选；省略则只用 base_url 的 host
  - api.github.com
auth:
  type: bearer               # none | bearer | header | basic
  env: GITHUB_TOKEN
ops:
  操作名:
    method: GET              # 默认 GET
    path: /repos/{owner}/{repo}/issues
    description: 给模型看的说明
```

`path` 中 `{name}` 必须在 `app_invoke.path_params` 里提供，否则报 `missing path param`。

### 鉴权

| type | 写法 | 发出的头 |
|---|---|---|
| `none` | `auth: { type: none }` | 无 |
| `bearer` | `env: GITHUB_TOKEN` | `Authorization: Bearer <值>` |
| `header` | `header: X-API-Key` + `env: FOO_KEY` | 指定头 |
| `basic` | `user_env` / `pass_env` | `Authorization: Basic ...` |

也可以在 YAML 里写 `token` / `value` / `user` / `password`，**不推荐**（会进工作区文件）。生产只用 `env`。

---

## 5. GitHub：从零到调用

### 5.1 建 Token

1. GitHub → Settings → Developer settings → Personal access tokens  
   - 新账号用 **Fine-grained**；经典 PAT 也可以  
2. Fine-grained 最少权限示例：  
   - Repository access：只勾要用的仓库  
   - Permissions：`Issues` Read and write；若只要读资料则 Metadata + Contents Read  
3. 经典 PAT 范围示例：`repo`（读写 issue/PR/内容）或 `public_repo`  
4. 复制 token，只保存到环境变量，不要写进 YAML、不要发给模型

### 5.2 环境变量（Windows）

当前用户永久：

```bat
setx GITHUB_TOKEN "ghp_xxxxxxxx"
```

关掉并重新打开 GUI / 控制台后再启动 OMA（`setx` 对已打开的进程无效）。

当前窗口临时：

```bat
set GITHUB_TOKEN=ghp_xxxxxxxx
python oma4_bot_gui.py
```

PowerShell：

```powershell
$env:GITHUB_TOKEN = "ghp_xxxxxxxx"
```

Linux / macOS：

```bash
export GITHUB_TOKEN=ghp_xxxxxxxx
```

### 5.3 放置清单

把本仓库提供的 `github.yaml` 拷到：

```
<工作区>/.oma/apps/github.yaml
```

确认 `auth.env` 为 `GITHUB_TOKEN`，与上面变量名一致。

### 5.4 在 GUI 里怎么用

对 Agent 直接说人话即可，例如：

- 「用 GitHub API 看我是谁」
- 「列出 huanglynch/MultiAgentSwarm 的 open issue」
- 「在 huanglynch/MultiAgentSwarm 建一个 issue，标题是 OMA 连接器自测」

期望工具序列：

```
list_apps
app_describe   app=github
app_invoke     app=github  op=me
```

或：

```
app_invoke
  app: github
  op: list_issues
  path_params: { owner: huanglynch, repo: MultiAgentSwarm }
  query: { state: open, per_page: 10 }
```

建 issue：

```
app_invoke
  app: github
  op: create_issue
  path_params: { owner: huanglynch, repo: MultiAgentSwarm }
  body:
    title: OMA App Connector 自测
    body: |
      由 OMA app_invoke 创建，可关闭。
```

搜自己的待办：

```
app_invoke
  app: github
  op: search_issues
  query:
    q: "repo:huanglynch/MultiAgentSwarm is:open assignee:@me"
```

### 5.5 自检失败时

| 现象 | 原因 |
|---|---|
| `unknown app: github` | YAML 不在三个扫描目录，或 `name:` 不是 github |
| `missing bearer token env=GITHUB_TOKEN` | 变量没进**当前** GUI 进程 |
| `origin not allowed` | `allow_origins` 漏了 `api.github.com` |
| HTTP 401 | token 错或过期 |
| HTTP 403 | token 范围不够，或触发二次限制 |
| HTTP 404 | owner/repo 写错，或 token 看不到私有库 |
| `missing path param: repo` | 调了带 `{repo}` 的 op 却没传 `path_params.repo` |

浏览器打开 github.com 不能代替上述调用。

---

## 6. 更多示例

### 6.1 无鉴权连通性（httpbin）

文件：`.oma/apps/httpbin.yaml`（可用附带的 `httpbin.example.yaml` 改名）

```
app_invoke  app=httpbin  op=get   query={ "hello": "oma" }
app_invoke  app=httpbin  op=post  body={ "ping": true }
```

用来验证连接器本身，不依赖 GitHub。

### 6.2 自定义 Header 的内部网关

```yaml
name: llmhub
description: 公司 OpenAI 兼容网关的管理面
base_url: https://aihub.example.com
allow_origins: [aihub.example.com]
auth:
  type: header
  header: X-API-Key
  env: AIHUB_ADMIN_KEY
ops:
  list_models:
    method: GET
    path: /v1/models
  get_key:
    method: GET
    path: /admin/keys/{key_id}
```

调用：

```
app_invoke app=llmhub op=list_models
app_invoke app=llmhub op=get_key path_params={ key_id: "abc" }
```

### 6.3 Basic 认证的旧系统

```yaml
name: jira_legacy
base_url: https://jira.example.com
allow_origins: [jira.example.com]
auth:
  type: basic
  user_env: JIRA_USER
  pass_env: JIRA_TOKEN
ops:
  myself:
    method: GET
    path: /rest/api/2/myself
  search:
    method: GET
    path: /rest/api/2/search
```

```
app_invoke
  app: jira_legacy
  op: search
  query: { jql: "assignee=currentUser() AND status!=Done", maxResults: 20 }
```

### 6.4 同一工作区多个 App

```
.oma/apps/github.yaml
.oma/apps/httpbin.yaml
.oma/apps/llmhub.yaml
```

`list_apps` 会一次列出全部。模型按用户目标选 `app`。

### 6.5 和浏览器分工的一轮任务

用户：「先打开项目主页看看 README 渲染，再用 API 建 issue。」

1. `browser_open` `https://github.com/huanglynch/MultiAgentSwarm`（读页面）  
2. `app_invoke` `create_issue`（写操作走 API）

不要用 `browser_click` 去按 “New issue”。

---

## 7. 给 Agent 的固定说法（可当用户话术）

- 「先 list_apps，有 github 就 app_describe，再 invoke，不要开浏览器。」
- 「只调用清单里的 op；缺 path 参数先问我 owner/repo。」
- 「写操作（create_issue、comment_issue）先用一句话说将要提交的 title/body。」

---

## 8. 安全注意

- Token 只放环境变量或系统保险库，不放 YAML、不放对话、不放 trace 期望里  
- `allow_origins` 尽量写死官方 API host，不要 `*`  
- Fine-grained token 只开用到的仓库和权限  
- orchestrator 调不到 `app_invoke`，避免「规划 bot」直接改远端  
- 连接器**不会**执行清单外的 URL，这是有意的；临时探测请用 `web_fetch` 或另写一个 op

---

## 9. 和打包 / 源码运行

源码运行：把 `oma/tools_apps.py` 放到包内，清单放工作区即可。  

onefile EXE：hidden-import 加 `oma.tools_apps`。清单仍读**启动 EXE 时的工作区** `.oma/apps/`，不是解包目录。  
目标机需要 `requests`、`pyyaml`（GUI 打包脚本已含）。不需要 Playwright 也能用 App API。
