---
name: aoccqa-scenario-expander
metadata:
  version: 1.0.2
description: >
  Compare confirmed requirements and normalized product/country rules against a QUALIFIED existing Test Case
  baseline, and draft only evidence-backed supplementary cases, existing-case enhancements, or parameterized
  Test Data. Foreground state-transition (狀態流轉) and role/identity (身分別) coverage as the primary
  expansion axes, while still checking every applicable dimension (core flow, condition/setting, data,
  boundary, error, schedule, cross-system, country/locale, product type, regression). Use when an existing or
  changed AOCCQA (ASUS EC / Magento) feature already has reviewed, maintained, executed, or previous-version
  Test Cases that need stronger coverage, OR to reinforce the coverage gaps of a draft produced by
  aoccqa-tc-generator that has ALREADY been independently reviewed. Requires a qualified baseline: with no
  baseline, return `Needs Baseline` and route to aoccqa-tc-generator — do not create the first Test Case set.
  Do not parse raw FSD/PRD/screenshots/Figma/API (route to aoccqa-fsd-parser), do not normalize scattered
  rules, do not invent product behavior, and do not approve final delivery (route to aoccqa-quality-reviewer).
---

# AOCCQA Scenario Expander(情境擴充器)

強化**合格既有 Test Case Baseline**,找可追溯、有證據的覆蓋缺口。**前景化「狀態流轉」「身分別」兩軸**,再掃其餘適用維度。擴充測試邏輯,非產品需求。

產線位置:`aoccqa-fsd-parser`(解析需求→六段報告)→ `aoccqa-tc-generator`(產第一版測項)→ **`aoccqa-scenario-expander`(對照 baseline 補缺口)** → `aoccqa-quality-reviewer`(Gate 2 審查放行)。

## 一、定位與職責邊界

- **已確認 Requirement Matrix** = 功能行為唯一權威。
- `Normalized Rule Context`(國別/Website/角色/狀態/設定/資格/資料/排程/locale/整合)= 規則權威。
- 既有 Test Case **只當覆蓋基線**,非未載明產品行為來源。
- 條件、觸發、觀察點、Expected Result 皆有依據才新增或強化情境。
- 未定義或衝突結果一律標 `Blocked`;**不得把 QA 經驗或歷史行為當產品事實**。
- **不產第一版完整測項** → 路由 `aoccqa-tc-generator`。
- **不解析原始 FSD/PRD/截圖/Figma/API/利害關係人訊息** → 路由 `aoccqa-fsd-parser`。
- **不正規化零散或衝突規則** → 路由規則正規化,或閘門回 `Needs Rule Loading`。
- **不靜默改寫、刪除、合併、重排、重新編號或放行既有 Test Case。**
- 擴充結果標 `Supplementary Test Case Draft`,交回 `aoccqa-quality-reviewer` 審查。

## 二、必要輸入盤點(先做,缺就停)

| 項目 | 必要性 | 說明 |
|---|---|---|
| 已確認 / 帶狀態標記的 Requirement Matrix | 必要 | 功能行為權威;含 Requirement ID 或可追溯需求鍵 |
| 合格 Existing Test Case Baseline | 必要 | 見「三、執行閘門」資格判定 |
| In Scope / Out of Scope | 必要 | 界定本輪擴充範圍 |
| `Normalized Rule Context` + Rule ID | 適用時必要 | 國別/Website/locale/角色/設定/商品/排程/整合差異 |
| 「無其他產品或市場規則適用」明確聲明 | 適用時 | 沒規則差異要明說,否則視為 `Missing` |
| Test Data / 環境 / 時間 / 抽樣 / 協助限制 | 選填 | 影響可執行性 |
| 受保護案例文字或必要排序 | 選填 | 不得更動的既有內容 |

**不得**從缺漏資訊推定 `Disabled`/`Unsupported`/`Not Applicable` 或任何 Expected Result。

## 三、執行閘門 Execution Gate

### 3.1 Baseline 資格判定(核心)

**接受**任一為合格 baseline:

- 維護中回歸(Regression)測項;
- 前一版功能或需求版本測項;
- 經 QA 審查或實際執行過測項;
- 上線使用或正式核准測項;
- QA 明確指定為 baseline 的已審查測項集。

**拒絕或暫停**當唯一候選為:

- 同輪 `aoccqa-tc-generator` 剛產、**尚未獨立審查**的草稿;
- 同份 Requirement Matrix 產出、僅供自我比對的另一草稿;
- 條件與 Expected Result 不足以判覆蓋的清單;
- 與當前功能範圍無關的測項集。

> 例外:同輪 Generator 草稿**唯使用者明確要求且已通過獨立審查**才可作 baseline,須記錄此例外。無此例外時,無 baseline = `Needs Baseline`。

### 3.2 閘門結果(擴充前必回一種)

- `Ready`:必要輸入齊備且 baseline 合格。
- `Needs Rule Loading`:適用規則零散/缺漏/衝突,無法比對 → 先正規化規則。
- `Needs Baseline`:無合格既有測項 → 路由 `aoccqa-tc-generator`,再 `aoccqa-quality-reviewer`。
- `Blocked`:缺漏資訊會改變適用性或 Expected Result。
- `Out of Scope`:請求比對超出提供範圍。

**唯 `Ready` 才進工作流程。**

## 四、知識庫 fallback(`/aoccqa-knowledge-base`)

歧義卡住覆蓋比對、且屬「既存可查回」事實(既有名詞/欄位/狀態/映射/系統行為、既核准國別/Website/locale/角色/商品/排程/整合規則、已知規則來源/版本/適用性、維護中歷史測項或回歸關係)時,**先查知識庫再問人**。

不得用知識庫發明缺漏商業決策、核准提案規則、猜新功能行為,或取代 PM/RD/QA 必要確認。

**Token 鐵則**(對齊 `aoccqa-knowledge-base`):只查不載。先讀 `references/kb_manifest.json` 決定開哪檔、用哪 key,再用 bash `jq`/`grep` 取符合的那幾筆,**勿 Read 整個大 JSON**;大量分析交 subagent 只回濃縮結果。常用食譜:

```bash
KB=/path/to/aoccqa-knowledge-base/references
jq '.terms[]|select(.term=="X" or (.aliases[]?=="X"))' "$KB/Definition_AOCCQA_glossary.json"      # 名詞/狀態/公式
jq '.relations[]|select(.backend|test("X"))' "$KB/Definition_AOCCQA_relations.json"                  # 後台→前台連動
jq '.relations[]|select(.feature=="X")' "$KB/Definition_AOCCQA_system_relations.json"                # 跨系統狀態同步
jq '[.issues[]|select(.feature=="X")|{key,summary,status}]' "$KB/QA_task_UAT-QA_classified.json"     # 歷史單/回歸點
jq '.features[]|select(.feature=="X")' "$KB/Guide_AOCCQA_testpoint_library.json"                     # 測試點素材
```

每次查詢:①下窄查詢(功能+Requirement/Rule ID+國別/Website+版本或日期+未解點);②優先取精確、現行、已核准且可追溯來源條目;③記錄知識庫 item ID/path、來源、版本、生效日、狀態、比對範圍;④唯明確核准、現行、且符合功能/版本/國別/Website/條件才直接採用;⑤結果零散或需規則調解 → 回 `Needs Rule Loading`;⑥無精確結果或過期/衝突/未核准/範圍不符 → 保持 `Blocked` 並問一澄清問題。

**優先序**:1. 已確認 Requirement Matrix → 2. 適用 `Normalized Rule Context` → 3. 現行已核准知識庫證據 → 4. 既有測項(僅覆蓋證據)。知識庫結果**永不**靜默覆蓋已確認需求或規則;記錄衝突並回 `Blocked` 或 `Needs Rule Loading`。

## 五、工作流程(閘門 `Ready` 後才執行)

### Step 1｜正規化既有覆蓋(不改動原案)

逐一從每條 baseline 測項擷取:Test Case ID、Requirement ID、Rule ID、Category、Feature、國別/Website/locale、actor/角色、**起始狀態、目標狀態**、Pre-condition、決定性條件、動作/系統事件/觸發、Expected Result 與觀察點、Test Data、已涵蓋情境類型。

> Requirement ID 出現在某測項**不代表已覆蓋**;須確認決定性條件與可觀察 Expected Result 兩者都被驗證。保留原文,判讀不確定處記下,勿靜默修補。

### Step 2｜建覆蓋基線矩陣

每條已確認 Requirement 與適用 Rule 對應到:完全覆蓋 Test Case ID、部分覆蓋 ID 與缺的檢查、未覆蓋條件、被阻擋組合、Out of Scope / Not Applicable 維度。

### Step 3｜前景化兩主軸(核心產出)

**先展開兩軸,再處理其餘維度。** 每候選須追溯 Requirement ID 或 Rule ID;狀態集合/角色集合**只從需求或規則取,不得臆造**。

**A. 狀態流轉(State Transition)展開清單:**

- **允許轉移**:每個「起始狀態 —(觸發)→ 目標狀態」各一條,含觸發條件與觀察點。
- **禁止/非法轉移**:嘗試不被允許轉移 → 應被阻擋,引用原文錯誤訊息或系統行為。
- **終態**:到達後不可再轉移;再觸發應無效或報錯。
- **重複動作/冪等**:同動作重複觸發 → 無變化或冪等結果(依規則)。
- **無變化(no-op)**:條件不滿足時狀態不變且無副作用。
- **CTA/旗標狀態切換**:如 buy/pre_order/back_order/notify_me、Single Purchasable(對照 glossary 狀態值)。
- **排程驅動狀態**:排程/Job 改狀態(特價生效、Pre-Order→On-Sale)於**視窗前/中/後**與失敗時。
- **跨系統狀態同步**:Magento↔AOM↔EC 狀態一致性與**時序**(下單、扣庫存/Reservation、貨態、取消回寫、退款/Credit Memo);對照 `Definition_AOCCQA_system_relations`。
- **併發/競態**:兩來源同時改同狀態的結果;規則未定義 → 標 `Blocked`,不臆造。

**B. 身分別(Role/Identity)展開清單:**

- **角色清單來源**:只從需求/規則取,如 Guest、Member、各會員群組(user_group)、Admin、各後台 ACL 操作角色、第三方(WMS/金流/物流)。
- **每個適用角色 × 關鍵動作**:可見性、可操作性、權限允許/拒絕、資料可及範圍。
- **訪客 vs 會員差異**:限購(user_group_limit)、價格可見性、功能可用性、需登入才可執行的動作。
- **權限不足路徑**:無權角色觸發 → 應拒絕 + 錯誤訊息原文/導轉登入。
- **資料隔離/個資**:跨帳號不可見他人資料;個資遮蔽規則。
- **角色 × 狀態/國別/Website scope 交互**:用 Step 6 決策表建模,**不做笛卡兒積**。
- 未定義角色行為 → `Blocked`,不得臆造。

### Step 4｜其餘適用維度(次要,僅有證據缺口才補)

僅當 Requirement Matrix 或 Rule Context 確立該維度存在才檢查,對齊 `Guide_AOCCQA_coverage_and_sequencing` 12 涵蓋維度:核心流程(正常/替代/禁止/復原)、條件與設定(Enabled/Disabled、合格/不合格、齊全/缺必填、Website Scope)、輸入與資料(有效/無效/空/null/重複/未變更/映射/邊界)、錯誤與異常(驗證/API/Job/匯入/下載/缺資料/部分同步/逾時/重試/復原)、價格與計算(公式/特價/稅/幣別/時區)、通知(Email/SMS 時機與收件人/多語系)、排程與時序、跨系統一致性、國別/Website/locale、商品與資料型態(Simple/Configurable/Bundle/Addon/Freebie)、回歸(歷史 bug 與既有 QA task 回歸點)。

> 勿硬把每功能都套 Guest/Member、API 逾時、最大值、所有國別。**沒證據的維度不是擴充目標。**

### Step 5｜國別適用性 Gate(涉國別/Website/locale/區域商品/市場規則才跑)

每個國別/Website,**只從 `Normalized Rule Context`** 判定:功能推出狀態(Enabled/Disabled/Not Applicable/Missing)、適用角色與權限、合格商品/資料型態、設定範圍與預設值、在地狀態/排程/日期/幣別/電話/地址/翻譯規則、跨系統映射或整合差異、預期行為與觀察點、明確排除項。每個適用性判定須追溯 Rule ID 或來源。

用語界定:`Not Applicable`=規則明說不適用;`Out of Scope`=可能適用但本輪排除;`Missing`=未提供規則,要問;`Disabled`=規則明說停用。**永不**把 `Missing` 換成 `Disabled` 或 `Not Applicable`。

### Step 6｜交互條件建模(決策表,避免笛卡兒積)

多個已確認條件交互時用決策表。**保留一組合只當它**:代表獨立 Requirement/Rule、改變適用性/流程/觀察點/Expected Result、涵蓋有意義獨立失效風險,或被明確要求。

### Step 7｜確認真缺口

候選為真缺口才處理:無既有案例覆蓋某已確認 Requirement/Rule;案例只驗部分 Expected Result;角色/狀態/條件/國別/系統/時序會改變行為;已確認負向/錯誤/邊界規則未被驗證;必要跨系統一致性缺席;或提案具獨立缺陷偵測價值。**不得**只為衝數量或重複純資料變體而新增。

### Step 8｜選一種處置(每候選恰選一種)

- `Add New Case`:獨立前置/流程/失效原因/觀察點/Expected Result。
- `Enhance Existing Case`:同流程已存在但缺某必要檢查 → 指出既有 Test Case ID、要改的**欄位與位置**、建議文字、來源、原因;**永不靜默套用**。
- `Parameterize`:測試邏輯與 Expected Result 相同,只差 Test Data。
- `Duplicate`:baseline 已覆蓋該需求與決定性條件。
- `Blocked`:適用性或 Expected Result 未定義或衝突。
- `Out of Scope` / `Not Applicable`:見用語界定。

### Step 9｜抽 Test Data

擷取具體資料優先序:Requirement Matrix / 正規化規則資料 → Feature → Pre-condition → Test case → Steps。Test Data 可含國別、Website、`角色=商品型態*數量`、日期(`YYYY/MM/DD`)、狀態、欄位值、檔案內容、缺漏/非法值、可重用帳號資料;共用 Steps 與 Expected Result 時用 `[Data Set 1]`、`[Data Set 2]`;無特定資料用 `無特殊測試資料`。**不要**把動作、頁面導覽、Expected Result、PM/RD 協助、一般環境設定放進 Test Data。兩來源衝突 → 記 `Blocked`,不擇一;資料會改流程或 Expected Result → 拆測項,不 parameterize。

### Step 10｜草擬補充測項(對齊 `Guide_AOCCQA_testcase_conventions`)

- 用暫時 ID `NEW-001`、`NEW-002`…;保留原測項與編號。
- 欄位對齊既有慣例:`ID | Category | Pre-condition | Test case | Steps | Expected result | Test result | Note`。
- `Category` 用需求或 baseline 已存在分類名。
- `Test case`:一句肯定式驗證目的,含決定性條件與預期行為。
- `Pre-condition`:只放執行前已為真的事實。
- `Steps`:只放動作、觸發、檢查位置;標路徑起點(如 `Path: Magento > System > Data Transfer > Import`);一步一行、動詞開頭;PM/RD/WMS/第三方協助標該步括號內;排程 Job 描述為「等待自動執行」除非來源明示手動。
- `Steps` 與 `Expected result` **一對一編號**;每條 Expected result 可觀察、足以判 Pass/Fail。
- 定義時明確寫出 UI 原文、欄位值、資料寫入、狀態、API 回應、Job 結果、Report、檔案內容或 Audit Log;錯誤訊息**引用原文**(對照 `Reference_AOCCQA_quicklookup`);涉金額附公式;前後台連動要涵蓋「後台動作→前台呈現」(對照 `Definition_AOCCQA_relations`)。
- **不得**只用 `功能正常`、`結果正確`、`資料無誤`、`符合預期` 當唯一 Expected result。

### Step 11｜處理 Blocked 候選

每個 Blocked:①指出 Requirement ID、Rule ID、國別/範圍、缺的規則;②符合知識庫 fallback 條件時查 `/aoccqa-knowledge-base` 並記錄結果;③說明哪個缺口關不掉、為何改變 Expected Result;④知識庫無法以現行核准證據解決時,才用具體情境問**一個**白話問題;⑤指派優先級 `P0`(無法起測或核心結果未知)/`P1`(重要覆蓋或國別適用性被卡)/`P2`(非核心細節);⑥只以未受影響候選繼續。

### Step 12｜反向檢查

確認:每提案追溯到 Requirement ID 與適用 Rule ID;無提案已被覆蓋;每提案有獨立驗證價值;純資料差異已 parameterize;**擴充前已跑國別適用性**;未引入不受支持角色/狀態/國別/商品/系統/上限/結果;**狀態流轉與身分別兩軸的允許/禁止/終態/權限拒絕都已覆蓋**;Steps 與 Expected result 一對一;受保護內容未動;未解與剩餘未覆蓋項可見。

## 六、輸出契約(依序回傳,繁中呈現)

1. 執行閘門結果(含狀態;若查過知識庫加 `知識庫查詢` 子表)
2. 擴充摘要
3. **狀態流轉覆蓋矩陣**(本 skill 主軸)
4. **身分別覆蓋矩陣**(本 skill 主軸)
5. 國別適用性矩陣(適用時)
6. 其餘維度覆蓋缺口矩陣
7. 既有案例強化建議
8. 補充測項草稿(`NEW-xxx`)
9. Requirement/Rule ↔ Test Case 追溯表
10. Blocked 情境與澄清問題(含 P0/P1/P2)
11. 重複與 Parameterization 建議
12. 剩餘未覆蓋項

**知識庫查詢**子表:

| Query | Matched Item | Version/生效日 | 核准狀態 | 範圍相符 | 解決結果 | 對閘門影響 |
|---|---|---|---|---|---|---|

**覆蓋缺口矩陣**(狀態流轉/身分別/其餘維度共用):

| Gap ID | Requirement ID | Rule ID | 國別/範圍 | 擴充維度(狀態流轉/身分別/…) | 缺的情境或檢查 | 既有 Test Case | 證據狀態 | 處置 | 原因 |
|---|---|---|---|---|---|---|---|---|---|

**既有案例強化建議**:

| 既有 Test Case | 欄位與位置 | 建議新增 | Requirement/Rule 來源 | 原因 |
|---|---|---|---|---|

**證據狀態**:`Supported`(明文)/`Derivable`(已確認規則的直接組合,須引用每個來源)/`Assumption Allowed`(使用者明確允許並標記的假設)/`Blocked`(結果或適用性未定義)。`Derivable` **不授權**新角色、狀態、數值上限、錯誤回應、重試或商業結果。

## 七、完成準則

**不得**宣稱擴充完成,除非:執行閘門回 `Ready`;每已確認 Requirement 與適用 Rule 都與 baseline 比對過;**狀態流轉與身分別兩軸已完整展開(允許/禁止/終態/重複/無變化;各適用角色的可見/可操作/權限拒絕/資料隔離)**;需要時已評估國別適用性;每提案有證據或明確允許假設;每 Blocked 都指出缺的規則與影響;每符合條件歧義都記錄知識庫查詢與證據結果;未引入不受支持行為;未靜默更動任何既有測項;重複/拆分/合併/parameterization 決策都已檢查;剩餘未覆蓋項已明列。

輸出標 `Supplementary Test Case Draft`,仍須經 QA 或 `aoccqa-quality-reviewer` 審查。
