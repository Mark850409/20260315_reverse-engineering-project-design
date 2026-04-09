# 逆向工程推導專案設計（Reverse Engineering Project Design）

從既有程式碼、API、資料庫 Schema 或產品行為，系統性推導並產出完整專案設計文件套件的 **Agent Skill**。

## 適用情境

- 接手舊專案、缺少正式文件，需要補齊 PRD／架構／API／資料庫說明。
- 重構或交接前，希望先「文件化現況」再動手改程式。
- 需要從程式與 Schema 反推需求、設計決策與團隊開發規範。

在 Cursor／Claude 等支援 Skills 的環境中，當使用者提出上述類型需求時，應啟用本技能；完整觸發條件與行為定義見 [`SKILL.md`](SKILL.md)。

## 核心理念

以「已存在的事實」重建「當初的設計意圖」，並區分可信度：

| 標記 | 意義 |
|------|------|
| `[事實]` | 有程式碼或文件直接佐證 |
| `[推測]` | 合理推論，無直接證據 |
| `[待確認]` | 需開發者確認 |

分析維度包含：邊界、結構、行為與技術決策；產出前可依 `references/analysis-checklist.md` 對照輸入類型深化分析。

## 預設產出文件

分析完成後，預設於專案 `docs/`（或使用者指定路徑）產出下列文件：

| 檔案 | 說明 |
|------|------|
| `PRD.md` | 需求規格書 |
| `SDD.md` | 軟體設計文件（含 C4 架構圖，Mermaid） |
| `API-Spec.md` | API 規格 |
| `Schema.md` | 資料表結構文件（含 ERD，Mermaid） |
| `ADR.md` | 技術決策記錄 |
| `Dev-Standards.md` | 開發規範（現況／標準／原因） |

架構圖嵌入 SDD；ERD 嵌入 Schema。各文件章節與撰寫要點以 `references/` 內對應模板為準。

## 目錄結構

```
reverse-engineering-project-design/
├── README.md          # 本說明
├── SKILL.md           # 技能主規格（流程、觸發、品質檢查清單）
└── references/        # 模板與分析輔助
    ├── prd-template.md
    ├── sdd-template.md
    ├── api-spec-template.md
    ├── schema-template.md
    ├── adr-template.md
    ├── dev-standards-template.md
    ├── erd-guide.md
    └── analysis-checklist.md
```

## 使用方式

1. **在支援 Agent Skills 的編輯器／助理中**：將本資料夾納入技能路徑，或依工具說明註冊 `SKILL.md`。
2. **觸發任務時**：提供程式庫路徑、API 文件、Schema 匯出或產品說明；可指定只要部分文件或自訂輸出目錄。
3. **產出後**：依 `SKILL.md` 建議與開發者核對 `[推測]`／`[待確認]` 項目，並迭代補強。

## 品質要點（摘要）

- 術語跨文件一致；圖表使用可渲染的 Mermaid。
- API 規格需含成功與失敗範例；Schema 需有欄位業務意義而非僅型別。
- ADR 需交代未採用的替代方案與取捨。

完整檢核表見 [`SKILL.md`](SKILL.md) 第四步。

## 授權與貢獻

若本技能為內部或開源專案的一部分，請依上層儲存庫的授權與貢獻指南為準。
