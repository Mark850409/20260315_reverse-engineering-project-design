# 開發規範文件模板（從逆向工程萃取）

## 文件資訊
- **系統名稱**：[系統名稱]
- **規範版本**：1.0（從現有系統萃取）
- **分析日期**：[日期]
- **適用範圍**：[新功能開發 / 全系統重構 / 特定模組]

> **使用方式**：每條規範標記 ✅ 現況 → 📏 標準 → 💡 原因

---

## 1. 命名規範

### 1.1 程式碼命名
✅ **現況**：[從程式碼觀察到的命名方式]
📏 **標準**：
- 函式名稱：`動詞 + 名詞`，例：`getUserById()`, `createOrder()`
- 布林變數：`is/has/can + 形容詞`，例：`isActive`, `hasPermission`
- 常數：`SCREAMING_SNAKE_CASE`，例：`MAX_RETRY_COUNT`

💡 **原因**：一致的命名慣例降低閱讀認知負擔

### 1.2 API Endpoint 命名
✅ **現況**：[從 API 文件/路由設定觀察]
📏 **標準（RESTful）**：
```
GET    /resources              # 列表
GET    /resources/:id          # 單筆
POST   /resources              # 新增
PUT    /resources/:id          # 完整更新
PATCH  /resources/:id          # 部分更新
DELETE /resources/:id          # 刪除
POST   /resources/:id/action   # 動詞操作
```
💡 **原因**：RESTful 慣例使 API 行為可預測，降低整合成本

### 1.3 資料庫命名
✅ **現況**：[從 Schema 觀察]
📏 **標準**：
- 資料表：`snake_case` 複數（例：`users`, `order_items`）
- 欄位：`snake_case`（例：`created_at`, `user_id`）
- 外鍵：`[參照表單數]_id`（例：`user_id`, `product_id`）
- 時間欄位標準：`created_at`, `updated_at`, `deleted_at`

💡 **原因**：一致的 Schema 命名讓 ORM 自動推導關聯，減少手動設定

---

## 2. 架構規範

### 2.1 分層架構與依賴方向
✅ **現況**：[識別到的分層結構]
📏 **標準（依賴方向單向）**：
```
Controller/Handler  →  只能呼叫 Service
Service/UseCase     →  只能呼叫 Repository
Repository/Store    →  只能呼叫 Database/External
```
**禁止**：
- ❌ Service 直接操作 HTTP Request/Response 物件
- ❌ Repository 包含業務邏輯
- ❌ Controller 直接查詢資料庫

💡 **原因**：清晰的依賴方向使各層可獨立測試，業務邏輯不受框架污染

### 2.2 模組邊界原則
📏 **標準**：
- 每個模組單一職責，只處理一個業務領域
- 對外介面需有型別定義
- 使用依賴注入降低耦合，避免循環依賴

---

## 3. API 設計規範

### 3.1 統一回應格式
✅ **現況**：[從 API 回應結構觀察]
📏 **標準**：
```json
// 成功
{ "success": true, "data": { ... }, "meta": { "page": 1, "total": 100 } }

// 失敗
{ "success": false, "error": { "code": "RESOURCE_NOT_FOUND", "message": "友善說明" } }
```

### 3.2 HTTP 狀態碼
| 狀態碼 | 使用情境 |
|-------|---------|
| 200 | 成功（GET/PUT/PATCH） |
| 201 | 建立成功（POST） |
| 204 | 成功無內容（DELETE） |
| 400 | 請求格式/驗證失敗 |
| 401 | 未認證 |
| 403 | 無權限 |
| 404 | 資源不存在 |
| 422 | 業務邏輯驗證失敗 |
| 500 | 伺服器內部錯誤 |

---

## 4. 資料庫規範

### 4.1 索引策略
📏 **標準**：
- 所有外鍵欄位必須建立索引
- 常用查詢條件欄位建立索引（`status`, `created_at`）
- 複合索引：選擇性高的欄位放前面
- 避免在低選擇性欄位建立獨立索引（如布林欄位）

### 4.2 Migration 規範
📏 **標準**：
- 每次 Schema 變更獨立一份 Migration
- 必須包含可回滾的 `down()` 方法
- 禁止在 Production 直接執行 DDL

---

## 5. 安全規範

### 5.1 認證標準
✅ **現況**：[從程式碼觀察到的認證機制]
📏 **標準**：
- Access Token 過期：15-60 分鐘；Refresh Token：7-30 天
- API Key 必須使用環境變數，禁止存入版本控制
- 敏感操作需要二次驗證

### 5.2 輸入驗證
📏 **標準**：
- 所有外部輸入在 Controller 層統一驗證
- 資料庫查詢必須使用 Parameterized Query，禁止字串拼接

---

## 6. 測試規範

### 6.1 覆蓋要求
📏 **標準**：
- 業務邏輯層（Service）必須有單元測試
- 關鍵 API 流程需有整合測試
- 新功能 PR 合併前須通過測試

### 6.2 測試命名
📏 **標準**：
```
describe('[被測試的單元]', () => {
  it('should [預期行為] when [條件]', () => { ... })
  it('should throw [錯誤類型] when [錯誤條件]', () => { ... })
})
```

---

## 7. 技術債清單（從現有系統識別）

| 問題描述 | 嚴重程度 | 影響範圍 | 建議優先級 |
|---------|---------|---------|----------|
| [技術債 1] | 高/中/低 | [模組] | P0/P1/P2 |

---

## 8. 規範執行機制

### 自動化強制
- [ ] Linter 規則設定（ESLint/Pylint/Rubocop）
- [ ] 格式化工具（Prettier/Black）
- [ ] Pre-commit hooks
- [ ] CI 自動執行測試與 lint

### PR Review Checklist
```markdown
- [ ] 命名符合規範
- [ ] 分層依賴方向正確
- [ ] API 回應格式符合標準
- [ ] 有對應的測試
- [ ] 無安全漏洞（SQL Injection / XSS）
- [ ] 不引入循環依賴
```
