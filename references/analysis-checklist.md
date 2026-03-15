# 各輸入類型分析清單

## 程式碼分析清單

### 第一輪：快速掃描（2-3 分鐘）
- [ ] 識別主要程式語言與框架
- [ ] 閱讀 README / package.json / requirements.txt
- [ ] 掃描頂層目錄結構
- [ ] 識別進入點（main.py / index.js / App.java）

### 第二輪：結構分析
- [ ] **模組拆分邏輯**：目錄名稱是否對應業務領域？
- [ ] **命名慣例**：是否有 Controller/Service/Repository 分層？
- [ ] **設定管理**：環境變數、設定檔揭示的架構決策
- [ ] **依賴清單**：第三方套件暗示的功能（例：stripe → 金流、sendgrid → 郵件）

### 第三輪：業務邏輯萃取
- [ ] **核心 Domain 物件**：主要的 Model/Entity 名稱
- [ ] **CRUD 操作**：哪些資源可以被建立/讀取/更新/刪除？
- [ ] **業務規則**：Validation 邏輯、條件判斷揭示的業務限制
- [ ] **狀態管理**：enum 定義、status 欄位揭示的狀態機

### 第四輪：技術決策識別
- [ ] **資料庫**：ORM 設定、Migration 檔案
- [ ] **快取策略**：Redis/Memcached 使用模式
- [ ] **非同步處理**：Queue/Worker 設定
- [ ] **API 設計**：RESTful 慣例 vs GraphQL vs RPC
- [ ] **認證授權**：middleware 鏈、JWT/Session 設定

---

## API 文件 / Swagger 分析清單

### 資源識別
- [ ] 列出所有 endpoint 路徑
- [ ] 按資源分組（/users, /orders, /products）
- [ ] 識別 CRUD 操作模式

### 資料模型萃取
- [ ] 從 request/response schema 建立資料模型
- [ ] 識別必填欄位 vs 選填欄位（業務規則線索）
- [ ] 注意 enum 值（揭示業務狀態）

### 認證授權分析
- [ ] Security scheme 類型（Bearer/ApiKey/OAuth）
- [ ] 哪些 endpoint 需要認證？
- [ ] 是否有角色區分（admin endpoints vs user endpoints）？

### 業務流程重建
- [ ] 識別操作順序依賴（需要先 POST /orders 才能 POST /payments）
- [ ] 識別 webhook / callback endpoint（非同步流程）
- [ ] 識別批次操作 endpoint

---

## 資料庫 Schema 分析清單

### 實體識別
- [ ] 列出所有資料表名稱
- [ ] 識別核心業務表 vs 輔助表（log/audit/config）
- [ ] 識別關聯表（多對多的橋接表）

### 關聯分析
- [ ] Foreign Key 約束揭示的實體關係
- [ ] 推測 1:1 / 1:N / N:M 關係
- [ ] 識別階層關係（parent_id self-reference）

### 業務邏輯推導
- [ ] NOT NULL 約束 → 必填業務規則
- [ ] UNIQUE 約束 → 唯一性業務規則
- [ ] CHECK 約束 → 值域業務規則
- [ ] DEFAULT 值 → 業務預設狀態

### 特殊欄位模式識別
| 欄位模式 | 業務含義 |
|---------|---------|
| `deleted_at` / `is_deleted` | 軟刪除，資料保留需求 |
| `tenant_id` / `org_id` | 多租戶 SaaS 架構 |
| `created_by` / `updated_by` | 稽核/合規需求 |
| `status` enum | 核心業務流程狀態機 |
| `version` / `revision` | 樂觀鎖或版本控制 |
| `metadata` JSON | 彈性擴展需求，或 schema 設計未穩定 |
| `position` / `sort_order` | 使用者可排序的項目 |
| `expires_at` | 時效性資料（token/訂閱/快取） |

### 索引分析
- [ ] 索引欄位揭示的查詢熱點
- [ ] 複合索引揭示的常見查詢模式
- [ ] 缺少索引的外鍵（潛在效能問題）

---

## 已部署產品分析清單

### 功能探索
- [ ] 系統性點擊所有主要功能
- [ ] 注意 URL 結構（揭示路由設計）
- [ ] 觀察網路請求（DevTools → Network）
- [ ] 識別第三方腳本（GA/Segment/Intercom 揭示業務工具棧）

### 使用者流程重建
- [ ] 新使用者 Onboarding 流程
- [ ] 核心 Happy Path（主要業務流程）
- [ ] 設定/管理流程

### 技術棧識別
- [ ] 前端框架（Chrome DevTools / Wappalyzer）
- [ ] CDN / 雲端服務（HTTP 回應標頭）
- [ ] API 設計風格（Network 標籤）
