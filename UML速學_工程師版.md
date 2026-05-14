# UML 速學筆記 — 工程師導向

> 對象：有軟體開發背景但對 UML 不熟的工程師（你）
> 目的：用程式碼類比快速掌握本期中報告會出現的 UML 概念
> 來源：`textbooks/digest/`（每節有頁碼註記，遇衝突回原始 PDF）

---

## TL;DR — 用程式員語言看 UML

| UML 概念 | 程式員怎麼理解 |
|----------|----------------|
| Use Case Diagram | 「**這個系統有哪些 API endpoint，誰會打**」的視覺地圖 |
| Use Case Description | 每個 endpoint 的 **OpenAPI spec**（含成功路徑、失敗分支、前後置條件） |
| Class Diagram | 你的 **ORM Model + 型別系統**，只看名詞跟關係 |
| Activity Diagram | 一個 function / handler 的**流程圖**（含 if/else、parallel） |
| Sequence Diagram | 多個 service 之間的**時序圖**（前端 → 後端 → DB → 第三方） |
| State Machine | 某個物件的**狀態機**（OrderStatus enum 怎麼跳） |
| Component Diagram | 系統的**模組架構圖**（service 之間怎麼依賴） |
| Actor | 呼叫 API 的角色（user、cron job、外部系統） |
| Stereotype `«include»` | 把可重用邏輯**抽成 helper function** |
| Generalization | **繼承**（is-a） |
| Aggregation / Composition | 兩種**組合關係**（強弱差別 = 父物件死了子物件死不死） |

---

## 0. 為什麼工程師要懂 UML？

> 「我會寫 code 為什麼要學畫圖？」

四個實用價值（**對工程師**）：

1. **設計階段對齊**：你跟 PM、QA、其他工程師討論時，UML 是**共同語言**。比 ER 圖 / API doc 更早出現，是真正的「需求 → 實作」橋樑。
2. **抓 edge case**：寫 Use Case Description 的「延伸流程」時，常會發現自己沒想過的 fail path（並行衝突、驗證失敗、第三方逾時）。
3. **看懂老 codebase**：很多舊系統的文件就是 UML。看不懂圖等於看不懂歷史。
4. **跨語言**：UML 是**語言中立**的。你用 TS、Go、Python 寫的東西，類別關係都能用同一張 Class Diagram 表達。

> **重要心態**：不要為了 UML 而 UML。教材原話：「不為 UML 而 UML，以完成分析工作為第一優先。」見 [02-methodology.md](textbooks/digest/02-methodology.md)。

---

## 1. UML 全景：4+1 View

來源：[02-methodology.md](textbooks/digest/02-methodology.md)

UML 不是一張圖，是**五種視角**：

```
       DESIGN VIEW              IMPLEMENTATION VIEW
       （類別、物件）              （元件、模組）
                ┌─────────────┐
                │  USE CASE   │   ← 中心：使用者觀點
                │    VIEW     │
                └─────────────┘
       PROCESS VIEW            DEPLOYMENT VIEW
       （process、thread）        （server、容器）
```

| View | 對工程師類比 | 對應 UML 圖 |
|------|--------------|-------------|
| **Use Case** | API 列表、user story | Use Case Diagram |
| **Design** | 類別圖、型別系統 | Class Diagram、Object Diagram |
| **Process** | 多執行緒、event loop | Activity、Sequence Diagram |
| **Implementation** | 模組/套件依賴圖 | Component Diagram |
| **Deployment** | docker-compose、k8s topology | Deployment Diagram |

每個 view 都搭配**動態模型**（互動圖、狀態圖、活動圖）來補充行為。

**期中報告主要走前三個 view**：UC → Class → Activity/Sequence。

---

## 2. Use Case Diagram（使用案例圖）

來源：[04-use-case.md](textbooks/digest/04-use-case.md)

### 2.1 工程師類比

把它想成 **API 端點清單的視覺化**：
- **Actor** = 呼叫 API 的人（user role / system role / cron）
- **Use Case** = 一個業務操作（**不是**單一 API call，而是「使用者目標」）
- **連線** = 「這個 actor 可以做這件事」

### 2.2 三類 Actor

| Actor | 程式員例子 | 報告中例子 |
|-------|------------|-----------|
| **User** | 真實使用者 | Guest、Member、Admin |
| **System** | 外部服務 | ECPay 綠界 |
| **Timer** | cron job、scheduler | 每日匯入排程 |

> Actor **不是真實的人**，是**角色**。同一個人可能扮演多個 Actor。

### 2.3 三種關係（最容易混淆）

#### include（包含）— 必定執行

> 「a includes b」表示 a **一定會呼叫** b

工程師類比：**helper function**

```python
def handle_paid_unlock(user, lawyer_id):       # UC07
    create_payment_order(user, lawyer_id)      # UC17 (include)
    payment_callback = wait_for_ecpay()
    handle_payment_callback(payment_callback)  # UC18 (include)
```

圖形：**虛線箭頭 + `«include»` 標籤**，箭頭從「呼叫者」指向「被包含者」。

#### extend（擴充）— 條件執行

> 「a extends b」表示 a **在特定條件下擴充** b 的行為

工程師類比：**middleware / decorator / feature flag**

```python
def view_lawyer_profile(user, lawyer_id):           # UC03
    show_free_info()
    if has_unlock_record(user, lawyer_id):          # 擴充點條件
        show_paid_content()                          # UC08 extends UC03
```

圖形：**虛線箭頭 + `«extend»` 標籤**，箭頭**從擴充 UC 指向被擴充 UC**（常被畫反！）。

#### generalization（一般化）— 繼承

工程師類比：**class inheritance**

```python
class Guest:
    def search_lawyer(self): ...
    def view_free_page(self): ...

class Member(Guest):                # Member is-a Guest
    def submit_review(self): ...

class UnlockedMember(Member):       # UnlockedMember is-a Member
    def view_paid_content(self): ...
```

在 UC 圖中，這代表 Member 自動繼承 Guest 所有可用的 UC，不必重複連線。

圖形：**實線 + 空心三角形箭頭**，箭頭指向父類。

### 2.4 容易踩雷的規則

1. **Actor 與 UC 連線不要加箭頭**（單純的 has-access 關係）。教材原話：「連結線沒有方向性。」
2. **UC 名稱以動詞開頭**：`Search Lawyer`、`Create Order`，不是 `Lawyer Search`。
3. **include / extend / generalization 的箭頭方向不要記反**：
   - include：caller → callee
   - extend：擴充者 → 被擴充者（**反直覺**）
   - generalization：child → parent

---

## 3. Use Case Description（使用案例描述）

來源：[05-use-case-description.md](textbooks/digest/05-use-case-description.md)、[use-case-description-template.md](textbooks/digest/use-case-description-template.md)

### 3.1 工程師類比

每個 UC 一份描述 = **OpenAPI spec 的擴充版**。它比 OpenAPI 多了：
- **前置條件**（Precondition）— 類似 middleware 檢查
- **延伸流程**（Extension）— 所有 error case 的處理
- **觸發事件**（Trigger）— 什麼動作會打這個 endpoint

### 3.2 必填欄位（11 個）

| 欄位 | 對應程式概念 |
|------|--------------|
| 使用案例名稱 | function name (動詞開頭) |
| 相關需求 | issue / ticket reference (`REQ-S001`) |
| 完成目標 | function 的 docstring 一行 summary |
| 前置條件 | `assert` / middleware check |
| 成功的結束狀態 | return value 應該長什麼樣 |
| 失敗的結束狀態 | exception 或 error response |
| 主要角色 | 呼叫者（authn 後的 user） |
| 次要角色 | 被呼叫的外部 service |
| 觸發事件 | 什麼 UI 操作 / event 啟動這個 flow |
| 主要流程 | happy path 步驟（編號 1, 2, 3...） |
| 延伸流程 | error / branch 處理（用 `主步驟.分支序` 編號） |

### 3.3 主要流程的寫法

- 每一步都用「**主詞 + 動詞**」格式：「使用者輸入...」「系統驗證...」
- 步驟用整數編號（1, 2, 3...）
- 如果某步驟是呼叫被 include 的 UC，寫成：`Include::<UC name>`

### 3.4 延伸流程的編號規則

```
主流程：
  1. 使用者輸入查詢條件
  2. 使用者送出
  3. 系統驗證
  4. 系統查詢
  5. 系統回傳結果

延伸流程：
  3.1 條件為空 → 顯示錯誤
  4.1 查詢例外 → 記錄並回傳通用錯誤
```

意思：「主流程第 3 步可能分支出 3.1」。本期中報告 18 份 UC Description 全部依此規則。

---

## 4. Class Diagram（類別圖）

來源：[06-class-diagram.md](textbooks/digest/06-class-diagram.md)

### 4.1 工程師類比

最直觀的 UML 圖。就是你 ORM Model 的視覺化：

```python
# Python 程式碼
class Lawyer:
    id: UUID
    name: str
    law_firm_id: UUID
    
    def get_profile(self) -> dict: ...
    def list_case_profiles(self) -> list: ...
```

對應 UML：
```
┌─────────────────┐
│    Lawyer       │
├─────────────────┤
│ +id: UUID       │
│ +name: String   │
│ +lawFirmId: UUID│
├─────────────────┤
│ +getProfile()   │
│ +listCases()    │
└─────────────────┘
```

### 4.2 能見度符號

| 符號 | 意義 | 程式語言對應 |
|------|------|--------------|
| `+` | public | Python: 無底線 / Java: public |
| `-` | private | Python: `__name` / Java: private |
| `#` | protected | Python: `_name` / Java: protected |
| `~` | package | Java: 預設 / Python: 無 |

> 分析階段通常都寫 `+`，不糾結於可見性 — 這是 design 階段才細化的事。

### 4.3 操作（Operation / Member Function）

教材重點：「**分析階段只寫敘述性操作**」。

❌ 不要寫：
```
+register(email: String, password: String): User
+findById(id: UUID): Optional<Lawyer>
```

✅ 應該寫：
```
+register()
+findById()
```

理由：實作細節（型別、回傳值、例外）應該等到 design 階段才補。分析階段只表達「這個類別有什麼能力」。

### 4.4 多重性（Multiplicity）

寫在關係線的兩端，工程師類比：**TypeORM 的 `OneToMany` / `ManyToOne` 設定**。

| 表示法 | 意義 | TS / Python 範例 |
|--------|------|-------------------|
| `1` | 恰好一個 | `user.profile: Profile` (non-null) |
| `0..1` | 零或一個 | `user.deletedAt: Date | null` |
| `0..*` | 零或多個 | `user.posts: Post[]` (可為空陣列) |
| `1..*` | 一或多個 | `order.items: Item[]` (至少一個) |
| `m..n` | 指定範圍 | `class.students: Student[]` (3..50) |

> `*` 是「multiplicity 無上限」，**不是萬用字元**。

### 4.5 五種關係（**最容易搞混**）

#### Association（關聯）— 一般使用

工程師類比：**a 物件持有 b 物件的 reference**

```python
class User:
    reviews: list[Review]  # User has many reviews
```

圖形：**實線**（雙向）或 **實線+箭頭**（單向）。

#### Aggregation（聚合）— 「整體—部分」，弱關係

工程師類比：**team.members** — 隊伍解散了，球員還是球員，可以加入別隊。

圖形：**實線 + 空心菱形**，菱形在「整體」端。

```
LawFirm ◇──── Lawyer
       (空心)
```

意思：事務所解散 → 律師仍存在（可以加入別的事務所）。

#### Composition（組合）— 「整體—部分」，強關係

工程師類比：**人 與 心臟** — 人死了心臟也沒了。也像 **Order 與 OrderLineItem** — 訂單刪了，line item 必然一起刪。

圖形：**實線 + 實心菱形**，菱形在「整體」端。

```
Lawyer ◆──── CaseProfile
      (實心)
```

意思：律師資料刪了 → 案件輪廓也跟著刪。

> **記憶口訣**：空心菱形 = 鬆散（可分離）、實心菱形 = 緊密（同生共死）。

#### Generalization（一般化）— 繼承

工程師類比：**class inheritance**

圖形：**實線 + 空心三角形箭頭**，箭頭指向父類。

```
        Animal
          ▲
          │ (空心三角形)
       ┌──┴──┐
      Dog   Cat
```

#### Dependency（相依）— 「a 用到 b」但不持有 b

工程師類比：**function 參數**（暫時用，不存）

```python
def send_notification(user: User, mailer: MailerService):  
    mailer.send(user.email)  # 用了 MailerService 但不存它
```

圖形：**虛線 + 箭頭**，箭頭指向被依賴方。

#### Realization（具體化）— 實作介面

工程師類比：**`class MyService implements IService`**

圖形：**虛線 + 空心三角形**，箭頭指向被實作的 interface。

### 4.6 關係強度排序

從弱到強：

```
Dependency  <  Association  <  Aggregation  <  Composition  <  Generalization/Realization
   (用)         (持有)         (組成)          (組成)            (是)
```

教材原話：「越是核心，強度越大。」

### 4.7 抽象類別 vs 介面

| | 抽象類別 (Abstract Class) | 介面 (Interface) |
|---|---|---|
| Stereotype | `«abstract»` | `«interface»` |
| 可有屬性？ | ✓ | ✗ |
| 方法可實作？ | 部分（含抽象方法） | ✗（只宣告） |
| 工程語言 | Java `abstract class` | Java `interface` / TS `interface` |

---

## 5. Activity Diagram（活動圖）

來源：[02-methodology.md](textbooks/digest/02-methodology.md)（W10-11 才會深入講）

### 5.1 工程師類比

**function flowchart**，但支援 fork / join（並行）。

| 符號 | 意義 | 對應程式 |
|------|------|----------|
| 黑點 | 起點 | function 開頭 |
| 圓角矩形 | 動作 | 一行 code |
| 菱形 | 決策 | `if / else` |
| 粗橫線（fork） | 分岔 | 並行 `Promise.all([])` |
| 粗橫線（join） | 合流 | `await` 全部完成 |
| 包圍黑點 | 終點 | `return` |

本報告 §02 搜尋律師流程中，用 `fork` 並行查詢律師基本資料、案件輪廓、事務所、風險摘要 — 對應後端的 parallel query。

---

## 6. Sequence Diagram（循序圖）

來源：[02-methodology.md](textbooks/digest/02-methodology.md)

### 6.1 工程師類比

**API 呼叫的時序圖**，類似 Chrome DevTools 的 Network waterfall + 多個 actor。

```
User       Frontend     Backend     ECPay      DB
 │            │            │          │         │
 │ click      │            │          │         │
 ├───────────▶│            │          │         │
 │            │ POST /unlock│          │         │
 │            ├───────────▶│          │         │
 │            │            │ INSERT   │         │
 │            │            ├──────────────────▶│
 │            │            │          │ orderId│
 │            │            │◀─────────────────┤
 │            │            │ create   │         │
 │            │            ├─────────▶│         │
 │            │            │ payUrl   │         │
 │            │            │◀─────────┤         │
```

### 6.2 框起來的區塊

| 區塊 | 意義 | 程式對應 |
|------|------|----------|
| `alt / else` | 二擇一分支 | `if / else` |
| `opt` | 可選分支 | `if (cond) { ... }` |
| `loop` | 迴圈 | `for / while` |
| `par` | 並行 | `Promise.all` |

本報告 §03 付費解鎖循序圖用了 `alt 未登入 / 已登入 → alt 已解鎖 / 未解鎖`，這是 nested if。

---

## 7. State Machine Diagram（狀態機圖）

### 7.1 工程師類比

物件 / row 的**狀態 enum + transition function**。

```typescript
type ReviewStatus = 
  | 'Draft' 
  | 'PendingReview' 
  | 'Published' 
  | 'Hidden' 
  | 'Removed';

// transition function 在 UML 就是箭頭上的「事件」標籤
function submit(r: Review) {
  if (r.status === 'Draft') r.status = 'PendingReview';
}
```

| 符號 | 意義 |
|------|------|
| `[*]` → 狀態 | 初始狀態 |
| 狀態 → `[*]` | 終止狀態 |
| 狀態 → 狀態 (有標籤) | transition (上面寫觸發事件) |

本報告 §06 Review 狀態機、§07 PaymentOrder、§08 RiskInfo 都是這個概念。

---

## 8. Component Diagram（元件圖）

工程師類比：**docker-compose 服務拓樸 + 模組依賴圖**。

| 符號 | 意義 | 對應 |
|------|------|------|
| 矩形 + 兩個小矩形 | 元件 / 模組 | service / module |
| 實線 + 箭頭 | 依賴 | imports / calls |
| `database` | 資料庫 | postgres / redis |
| 套件框 (package) | 邏輯分組 | namespace / monorepo workspace |

本報告 §09 把系統拆成 4 層：前端、後端 API、核心模組 (8 個)、資料層、外部系統。

---

## 9. 需求分析相關術語

來源：[03-requirements.md](textbooks/digest/03-requirements.md)

### 9.1 SRS（System Requirement Specification）— 系統需求規格書

期末交付物。架構：
1. 專案概述 / 目標 / 範圍
2. 系統架構
3. 詞彙表 / 名詞定義
4. 需求內容
5. 更新日誌

### 9.2 功能性 vs 非功能性需求

| 類型 | 例子 | 工程師類比 |
|------|------|------------|
| 功能性 | 使用者可以搜尋律師 | 業務邏輯 / API |
| 非功能性 | API 回應 < 500ms | NFR（performance、security、SLA） |

### 9.3 Event Table（事件表）

教材推薦的需求分析工具，**Use Case 的上游**：

| 欄位 | 意義 | 工程例子 |
|------|------|----------|
| 事件名稱 | 主詞 + 動詞 + 受詞 | `User clicks search` |
| 觸發點 | 引發事件的資料 | `search params` |
| 來源 | 來源 actor | `Web frontend` |
| 活動 | 系統做什麼 | `Query DB & Search Index` |
| 回應 | 輸出 | `Lawyer list JSON` |
| 目的地 | 輸出到哪 | `Frontend` |

### 9.4 Stakeholder（利害關係人）

> 能影響系統或受系統影響的人。

工程師類比：**RACI 矩陣的所有 R/A/C/I 角色**。本報告中包含老師（評分者）、潛在使用者、律師代表、平台維運者。

---

## 10. 方法論術語

來源：[02-methodology.md](textbooks/digest/02-methodology.md)

| 術語 | 是什麼 | 工程師類比 |
|------|--------|------------|
| **SDLC** | Software Development Life Cycle | 從規劃到交付的整體流程 |
| **Waterfall** | 階段循序的開發法 | 傳統大型專案 |
| **Prototype** | 雛型法，邊做邊改 | 「先做 mockup 給 PM 看」的流程 |
| **Spiral** | 螺旋式，含風險評估 | 每個 sprint 前評估技術風險 |
| **Agile** | 敏捷式 | Scrum、XP |
| **RUP** | Rational Unified Process | UML 三巨頭提出的方法論 |

報告本身不深入講方法論，但老師可能問「你們是用哪種開發法的」 — 答 **RUP-like 反覆漸增**最安全（因為 RUP 是 use-case driven，跟我們的報告吻合）。

---

## 11. 常見混淆對照

| 容易搞混的 | 差別 |
|------------|------|
| `include` vs `extend` | include = **必定執行**（helper function）/ extend = **條件擴充**（middleware） |
| `Aggregation` vs `Composition` | 聚合 = 整體沒了部分還在（球隊/球員）/ 組合 = 整體沒了部分也沒（訂單/明細） |
| `Class` vs `Object` | 類別 = 藍圖 / 物件 = 蓋出來的房子 |
| `Association` vs `Dependency` | 關聯 = 持有 reference / 相依 = 用過就丟（function 參數） |
| `Abstract Class` vs `Interface` | 抽象類可有屬性與實作的方法 / 介面只能宣告 |
| 4 階段（瀑布）vs 4 階段（RUP） | 瀑布：規劃/分析/設計/實施 順序走 / RUP：初始/詳述/建構/移轉 反覆漸增 |
| `Generalization` (UC) vs `Generalization` (Class) | UC 中也可以一般化 Actor 或 UC，意義跟類別繼承相同 |

---

## 12. 報告中你必須能即興解釋的 10 個術語

老師若隨機抽問，**這 10 個必須能用一句話解釋**：

1. **Use Case Diagram** — 從使用者觀點呈現系統能做什麼的圖
2. **Actor** — 與系統互動的角色（人、外部系統、Timer 都算）
3. **include** — 一個 UC 必定包含另一個 UC 的功能（虛線箭頭從呼叫者指向被包含者）
4. **extend** — 一個 UC 在特定條件下擴充另一個 UC（虛線箭頭從擴充者指向被擴充者）
5. **Generalization** — 一般化關係，跟物件導向繼承相同概念
6. **Use Case Description** — 每個 UC 的詳細表格，含前置條件、主流程、延伸流程
7. **Multiplicity** — 關係兩端的數量限制（1, 0..\*, 1..\* 等）
8. **Aggregation vs Composition** — 都是整體—部分，前者弱（可分離）、後者強（同生共死）
9. **Stereotype** — 用 `«…»` 標籤擴充 UML 詞彙（如 `«interface»`、`«include»`）
10. **4+1 View** — UML 的五個視角（Use Case 中心 + Design / Process / Implementation / Deployment）

---

## 13. 學習路徑建議

**今天就要報告？** 看 §1（4+1 View）、§2（UC）、§4（Class）、§11（混淆）、§12（10 大術語）。

**有 1 小時？** 全部看一遍，重點熟悉 §4（Class）跟 §3（UC Description）。

**有半天？** 配合 [textbooks/digest/](textbooks/digest/) 的對應 MD 一起讀，每讀完一節到 [index.html](index.html) 找一張對應圖，把術語對到圖上找出來。

---

## 14. 延伸對照表：你已會的 vs UML 對應

| 你已會的（工程師日常） | UML 對應 |
|----------------------|----------|
| ER 圖 | Class Diagram（但 UML 還含行為） |
| OpenAPI Spec | Use Case Description |
| Chrome Network waterfall | Sequence Diagram |
| Redux state machine | State Machine Diagram |
| Flowchart (graphviz) | Activity Diagram |
| Mermaid `flowchart`/`classDiagram`/`sequenceDiagram` | 對應 UML，只是工具不同 |
| docker-compose 拓樸 | Component / Deployment Diagram |
| user story（As a... I want... so that...） | Use Case 名稱 + 完成目標 |
| Microservices 依賴圖 | Component Diagram |
| inheritance / `extends` | Generalization |
| `interface` / `implements` | Realization (`«interface»` + 虛線三角形) |
| Decorator pattern | extend 關係 |
| Dependency Injection | Dependency 關係 |
| `OneToMany` / `ManyToOne` | Multiplicity |

如果你能用「左邊那欄」描述系統，你就**已經會 UML 80%**，剩下只是換符號。

---

## 附錄 A：本報告各 UML 圖在哪一節

| index.html 章節 | 圖類型 | 對應教材 |
|-----------------|--------|----------|
| §01 | Use Case Diagram | [04-use-case.md](textbooks/digest/04-use-case.md) |
| §01-A (18 份) | Use Case Description | [05-use-case-description.md](textbooks/digest/05-use-case-description.md) |
| §02 | Activity Diagram (搜尋) | 未深入講 |
| §03 | Sequence Diagram (付費) | 未深入講 |
| §04 | Activity Diagram (心得審核) | 未深入講 |
| §05 | Class Diagram | [06-class-diagram.md](textbooks/digest/06-class-diagram.md) |
| §06–08 | State Machine | 未深入講 |
| §09 | Component Diagram | 未深入講 |
| §10 | Activity Diagram (資料匯入) | 未深入講 |

> **「未深入講」** 的圖是 W10 之後的進度，但因為工程經驗你大概早就會了，老師若問可以直接秀對應的程式碼概念。

---

## 附錄 B：快速查證的 grep 指令

```bash
# 查 textbooks/digest/ 中所有提到「include」的段落
grep -rn "include" textbooks/digest/

# 查報告中提到「多重性」的地方
grep -n "多重性\|0\.\.\*\|1\.\.\*" index.html

# 列出所有 UC 編號
grep -oE 'UC[0-9]{2}' index.html | sort -u
```
