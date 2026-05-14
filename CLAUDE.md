# CLAUDE.md

本檔給 Claude Code（與其他協作工具）讀取，用來理解這個 repo 的脈絡與規則。

## Repo 用途

台北商業大學（NTUB）「**物件導向系統分析與設計**」課程的學期專題。

- 課表：https://ntcbadm1.ntub.edu.tw/TchWeb/Cur_SelTeaching.aspx
- 專題主題：**律師案件圖譜（Lawyer Case Atlas）**
  - 法律科技搜尋平台，整合法院公開判決、律師/事務所公開資訊、新聞聲量、社群討論、會員心得
  - 商業模式：類面試趣的付費解鎖；金流串綠界（ECPay）
  - MVP 範圍：民事案件、近五年資料

## 兩階段交付

| 階段 | 交付物 | 形式 | 狀態 |
|------|--------|------|------|
| **期中** | UML 系列圖（Use Case、Activity、Sequence、Class/Domain、State Machine、Component 等） | HTML | 進行中 |
| **期末** | 系統規格書（System Specification Document） | HTML → 最後轉檔下載為 PDF | 尚未開始 |

兩階段都以 HTML 為主要載體；期末成果最後會輸出為 PDF，因此版面、字型、分頁要考慮 PDF 列印相容性（A4、避免依賴互動才能看到的內容）。

## 必須遵守的規範

**所有報告產出必須對齊 `textbooks/` 下的課程教材。** 這些是老師發的講義，是評分基準。

### 教材原始 PDF（評分基準，遇衝突以此為準）

位於 `textbooks/OneDrive_1_2026-4-9/`：

- `01-Introduction.pdf`
- `02-Methodology.pdf`
- `03-Requirments.pdf`
- `04-Use_case.pdf`
- `05-Use_case_description.pdf`
- `06-Class_diagram.pdf`
- `使用案例描述-範本.pdf` — **Use Case Description 的版型基準**

> textbooks/ 目錄之後會持續加入新教材；遇到新檔案要為它建立對應 digest 並更新索引。

### 教材重點摘要（給 Claude 快速查閱）

`textbooks/digest/` 是規範性內容摘要，**先看 digest 再讀 PDF**。索引：[textbooks/digest/README.md](textbooks/digest/README.md)。

| 工作 | 必讀 digest |
|------|-------------|
| 寫 / 改 Use Case Diagram | [04-use-case.md](textbooks/digest/04-use-case.md) |
| 寫 / 改 Use Case Description | [use-case-description-template.md](textbooks/digest/use-case-description-template.md) + [05-use-case-description.md](textbooks/digest/05-use-case-description.md) |
| 畫 / 改 Class Diagram、Domain Model | [06-class-diagram.md](textbooks/digest/06-class-diagram.md) |
| 寫需求分析章節、事件表、SRS | [03-requirements.md](textbooks/digest/03-requirements.md) |
| 討論方法論、UML 4+1 觀點、RUP | [02-methodology.md](textbooks/digest/02-methodology.md) |
| 課程定位、評分、進度、OO 四特徵 | [01-introduction.md](textbooks/digest/01-introduction.md) |

**digest 是索引，不是真相**：發現 digest 與 PDF 不一致時，以 PDF 為準並修正 digest。

術語、圖形符號、文件結構都以教材為準，不要套用其他學派或工具預設。

## 檔案地圖

```
.
├── index.html                          # 期中主交付物：UML 規格簡報頁（Mermaid）
├── 律師案件圖譜_需求訪談正式版_v1.0.md  # 需求訪談正式版文件
├── 律師案件圖譜_需求訪談.html           # 需求訪談 HTML 版
├── 基於需求訪談衍伸文件.md              # UML 建模指引（Actor / Use Case / Domain Model / 流程）
└── textbooks/                          # 課程教材（評分基準）
    └── OneDrive_1_2026-4-9/
```

## 工作守則

- 預設聚焦在期中 UML；除非明確要求，不要跳去寫程式或實作 MVP。
- 改 UML 內容優先動 `index.html` 內的 Mermaid 區塊，並讓 UML 與需求訪談文件保持一致。
- 需求訪談文件 (`律師案件圖譜_需求訪談正式版_v1.0.md`) 是上游真相；UML 必須回推到訪談內容。
- 「衍伸文件」(`基於需求訪談衍伸文件.md`) 是建模指引，可參考但 UML 細節以教材術語為準。
- 期末規格書 HTML 撰寫時，要主動考量 PDF 輸出（列印樣式 / 分頁 / 中文字型）。
