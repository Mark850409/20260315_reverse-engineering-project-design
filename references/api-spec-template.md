# API 規格文件（API-Spec）模板

> 使用說明：逆向工程時，從路由定義、控制器邏輯、Schema 驗證碼中推導各欄位的業務語義。
> 標記系統：`[事實]` 有程式碼佐證｜`[推測]` 合理推論｜`[待確認]` 需開發者確認

---

## 執行摘要

> [一段話描述 API 的整體設計理念、版本、主要用途。]
>
> 範例：「本 API 為 RESTful 風格，版本 v1，採用 Bearer Token 認證，主要供前端 SPA 及 CLI 工具使用。」

---

## 1. API 總覽

| 項目 | 值 | 說明 |
|------|-----|------|
| Base URL | `https://api.example.com` | [事實/推測] |
| API 版本 | `v1` | [事實/推測] |
| 協定 | HTTPS | [事實/推測] |
| 資料格式 | JSON (`application/json`) | [事實/推測] |
| 文字編碼 | UTF-8 | [事實/推測] |
| API 文件 | `/api/docs` (Swagger UI) | [事實/推測] |
| Rate Limiting | [N 次/分鐘，或「未識別」] | [推測] |

---

## 2. 認證說明

### 2.1 認證機制

> [事實/推測] 說明 Token 的取得方式及存放位置。

**Token 取得流程：**

```
POST /api/auth/login
Body: { "username": "...", "password": "..." }
Response: { "token": "<raw_token>", "user": {...} }
```

**請求 Header 格式：**

```
Authorization: Bearer <token>
```

### 2.2 權限層級對照表

| 角色 (Role) | 說明 | 可用 Endpoints |
|------------|------|----------------|
| `admin` | 系統管理員，擁有所有權限 | 全部 |
| `maintainer` | 維護者，可發布與管理自己的資源 | 除管理員專屬外全部 |
| `user` | 一般用戶，僅可讀取與安裝 | 公開讀取 + 個人操作 |
| 匿名 | 未登入，僅能瀏覽公開資料 | 公開讀取 |

### 2.3 細粒度權限 (Permissions)

> [推測] 若系統採用 ABAC，列出已識別的 Permission 字串：

| Permission 字串 | 說明 |
|----------------|------|
| `skill:create` | 發布技能 |
| `skill:delete` | 刪除技能 |
| `mcp:create` | 發布 MCP Server |
| `admin:access` | 存取管理後台 |

---

## 3. 通用規範

### 3.1 請求格式

- Content-Type: `application/json`
- 日期格式：ISO 8601（`2024-01-15T10:30:00Z`）
- 布林值：`true` / `false`（小寫）
- 空值：`null`

### 3.2 回應格式

**成功回應：**

```json
{
  "data": { ... },      // 單筆資源
  "items": [ ... ],    // 列表資源（或其他欄位名）
  "total": 100,        // 分頁總數
  "page": 1,
  "pages": 10,
  "per_page": 20
}
```

**錯誤回應：**

```json
{
  "status": "error",
  "code": "RESOURCE_NOT_FOUND",
  "message": "人類可讀的錯誤說明",
  "errors": {           // 表單驗證錯誤（可選）
    "field_name": ["錯誤訊息"]
  }
}
```

### 3.3 分頁規範

| 參數 | 說明 | 預設值 |
|------|------|--------|
| `page` | 頁碼，從 1 開始 | `1` |
| `per_page` | 每頁筆數 | `20`（可能因 endpoint 不同） |

### 3.4 排序規範

| 參數 | 說明 | 範例 |
|------|------|------|
| `sort` | 排序欄位 | `created_at`、`downloads` |
| `order` | 排序方向 | `asc`、`desc` |

---

## 4. 錯誤碼定義

### 4.1 HTTP 狀態碼對應

| HTTP 狀態碼 | 語義 | 常見情境 |
|------------|------|---------|
| `200 OK` | 成功 | GET、PATCH 成功 |
| `201 Created` | 建立成功 | POST 新增資源 |
| `204 No Content` | 成功無回應體 | DELETE 成功 |
| `400 Bad Request` | 請求格式錯誤 | 缺少必填欄位、型別錯誤 |
| `401 Unauthorized` | 未認證 | Token 遺失或失效 |
| `403 Forbidden` | 無權限 | 角色或 Permission 不符 |
| `404 Not Found` | 資源不存在 | 路由或 ID 錯誤 |
| `409 Conflict` | 衝突 | 唯一鍵重複（username、name） |
| `422 Unprocessable` | 語意錯誤 | 欄位格式合法但邏輯不通 |
| `500 Internal Error` | 伺服器錯誤 | 未預期的例外 |

### 4.2 自訂錯誤碼（如有）

| 錯誤碼 | HTTP 狀態 | 說明 |
|--------|----------|------|
| `TOKEN_EXPIRED` | 401 | Token 已過期 |
| `PERMISSION_DENIED` | 403 | 缺少特定 Permission |
| `RESOURCE_NOT_FOUND` | 404 | 指定資源不存在 |
| `DUPLICATE_NAME` | 409 | 名稱重複 |

---

## 5. Endpoint 目錄

> 按「資源」分組，每個資源標題下列出所有相關 Endpoint。

---

### 5.1 認證 (Auth)

---

#### `POST /api/auth/login` — 登入 / 自動註冊

> [事實] 說明業務語義：初次使用自動建立帳號，回傳原始 Token（伺服器僅存 Hash）。

**權限要求：** 無（匿名）

**Request Body：**

```json
{
  "username": "string, 必填",
  "password": "string, 必填（若系統不驗密碼則可為任意值）"
}
```

**Response 200：**

```json
{
  "token": "raw_token_string",
  "user": {
    "id": 1,
    "username": "alice",
    "role": "user",
    "permissions": []
  }
}
```

**Response 400：**

```json
{
  "message": "Missing username or password"
}
```

**注意事項：**
- Token 僅在此回應中以明文返回，之後伺服器只保存 Hash，請妥善儲存
- `[推測]` 若用戶名為 `admin` 且系統初次啟動，自動授予管理員角色

---

#### `GET /api/auth/me` — 取得目前用戶資訊

**權限要求：** 已登入

**Response 200：**

```json
{
  "id": 1,
  "username": "alice",
  "email": "alice@example.com",
  "role": "user",
  "permissions": ["skill:create"]
}
```

---

### 5.2 技能 (Skills)

---

#### `GET /api/skills` — 列出 / 搜尋技能

**權限要求：** 無（匿名）

**Query Parameters：**

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `q` | string | 否 | 全文搜尋（name、description、author） |
| `tag` | string | 否 | 標籤過濾（精確比對） |
| `category` | string | 否 | 分類過濾 |
| `sort` | string | 否 | `created_at`、`downloads`（預設 `created_at`） |
| `page` | integer | 否 | 頁碼（預設 `1`） |
| `per_page` | integer | 否 | 每頁筆數（預設 `20`） |

**Response 200：**

```json
{
  "skills": [
    {
      "id": 1,
      "name": "my-skill",
      "display_name": "My Skill",
      "description": "...",
      "author": "alice",
      "tags": ["coding", "python"],
      "category": "coding",
      "downloads": 42,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 100,
  "page": 1,
  "pages": 5,
  "per_page": 20
}
```

---

#### `POST /api/skills` — 發布 / 更新技能

**權限要求：** `skill:create`

**Request Body（multipart/form-data 或 JSON）：**

```json
{
  "name": "string, 必填, slug 格式",
  "version": "string, 必填, semver 格式",
  "description": "string, 必填",
  "author": "string, 選填（預設取自登入帳號）",
  "tags": ["string"],
  "category": "string",
  "skill_md": "string, 必填, SKILL.md 內容",
  "bundle": "binary, 選填, .tar.gz 附件"
}
```

**Response 200（已存在更新）/ 201（首次發布）：**

```json
{
  "id": 1,
  "name": "my-skill",
  "version": "1.0.1",
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Response 403 — 修改他人技能：**

```json
{
  "message": "You don't own this skill"
}
```

**副作用：** 建立或更新 `SkillVersion` 記錄，自動計算 SHA-256 checksum。

---

#### `GET /api/skills/{name}` — 取得技能詳情

**權限要求：** 無

**Path Parameter：** `name` — 技能唯一名稱（slug）

**Response 200：**

```json
{
  "id": 1,
  "name": "my-skill",
  "description": "...",
  "versions": ["1.0.0", "1.0.1"],
  "latest_version": "1.0.1",
  "skill_md": "# My Skill\n...",
  "downloads": 42
}
```

---

#### `GET /api/skills/{name}/download` — 下載技能包

**權限要求：** 無

**Query Parameters：**

| 參數 | 型別 | 說明 |
|------|------|------|
| `version` | string | 指定版本，不填則下載最新版 |

**Response 200：** `Content-Type: application/gzip`，二進位 tar.gz 串流

**副作用：** `Skill.downloads` 計數器 +1

---

### 5.3 MCP Servers (MCPs)

---

#### `GET /api/mcps` — 列出 / 搜尋 MCP Servers

**權限要求：** 無

**Query Parameters：** 同 `/api/skills`（支援 `q`、`category`、`sort`、`page`、`per_page`）

**Response 200：**

```json
{
  "mcps": [
    {
      "id": 1,
      "name": "github-mcp",
      "display_name": "GitHub MCP",
      "description": "...",
      "transport": "stdio",
      "tools": [
        {"name": "create_issue", "description": "..."}
      ],
      "installs": 15,
      "category": "coding"
    }
  ],
  "total": 50
}
```

---

#### `POST /api/mcps` — 發布 MCP Server

**權限要求：** `mcp:create`（或登入即可，依系統設定）

**Request Body：**

```json
{
  "name": "string, 必填, slug 格式",
  "display_name": "string",
  "description": "string, 必填",
  "transport": "sse | stdio | http",
  "endpoint_url": "string, SSE 傳輸時必填",
  "local_config": [
    {
      "type": "node | python | docker",
      "command": "npx",
      "package": "@org/mcp-server",
      "env": ["API_KEY"],
      "env_values": {"API_KEY": "secret"}  // 僅用於工具內省，不存入DB
    }
  ],
  "tags": ["string"],
  "category": "string"
}
```

**Response 201：**

```json
{
  "id": 1,
  "name": "github-mcp",
  "transport": "stdio",
  "tools": []  // 工具清單由後台非同步內省填充
}
```

**注意事項：**
- `env_values` 中的敏感環境變數**不會**存入資料庫，僅在工具內省過程中使用
- 工具清單（`tools`）透過背景執行緒非同步填充，可能需稍等後才會出現

---

#### `GET /api/mcps/{name}/connect` — 取得連線設定

**權限要求：** 無

**Response 200：**

```json
{
  "sse_proxy_url": "https://api.example.com/api/mcps/github-mcp/sse",
  "configs": {
    "claude_desktop": { ... },
    "node": { "command": "npx", "args": ["..."], "env": {} },
    "python": { "command": "python", "args": ["-m", "..."] },
    "docker": { "command": "docker", "args": ["run", "..."] }
  }
}
```

---

#### `GET /api/mcps/{name}/sse` — SSE 代理串流

**權限要求：** 無

**Response：** `Content-Type: text/event-stream`，持續推送 JSON-RPC 訊息

---

#### `POST /api/mcps/{name}/messages` — JSON-RPC 訊息轉發

**權限要求：** 無

**Request Body：** 標準 JSON-RPC 2.0 格式

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": { "name": "...", "arguments": {} }
}
```

---

### 5.4 管理後台 (Admin)

> 所有管理端點均需 `admin:access` Permission。

---

#### `GET /api/admin/categories` — 取得分類清單

**Response 200：**

```json
[
  {"id": "coding", "label": "程式開發", "icon": "💻"},
  {"id": "web", "label": "Web 瀏覽", "icon": "🌐"}
]
```

---

#### `GET /api/admin/skills` — 管理員列出所有技能

**Query Parameters：** `q`、`sort`、`page`、`per_page`

---

#### `PATCH /api/admin/skills/{name}` — 管理員修改技能元數據

**Request Body（任意欄位）：**

```json
{
  "category": "coding",
  "description": "更新說明",
  "tags": ["python", "automation"]
}
```

---

#### `DELETE /api/admin/skills/{name}` — 永久刪除技能

**Response 204**（無回應體）

**注意：** 同時刪除所有 `SkillVersion` 記錄（CASCADE）

---

#### `POST /api/admin/skills/classify-all` — AI 批次自動分類技能

**Response 200：**

```json
{
  "classified": 42,
  "skipped": 3,
  "details": [{"name": "my-skill", "category": "coding"}]
}
```

---

#### `GET /api/admin/users` — 列出所有使用者

#### `POST /api/admin/users` — 建立使用者

#### `PATCH /api/admin/users/{user_id}` — 修改使用者（含角色/權限）

#### `DELETE /api/admin/users/{user_id}` — 刪除使用者

---

#### `GET /api/admin/mcps` — 管理員列出所有 MCP Servers

#### `PATCH /api/admin/mcps/{name}` — 管理員修改 MCP 元數據

#### `DELETE /api/admin/mcps/{name}` — 永久刪除 MCP Server

#### `POST /api/admin/mcps/classify-all` — AI 批次自動分類 MCP Servers

---

## 6. 待確認問題清單

1. `[待確認]` Rate Limiting 策略為何？是否有針對匿名 / 登入用戶的差異限制？
2. `[待確認]` Token 有效期限為何？是否有 Refresh Token 機制？
3. `[待確認]` `POST /api/auth/login` 是否驗證密碼？還是任意密碼均可登入？
4. `[待確認]` `/api/skills/{name}/download` 的 version 不填時，是否取最新 stable 版或最新任意版？
5. `[待確認]` MCP Server 的工具內省失敗時，是否有重試機制或通知機制？
