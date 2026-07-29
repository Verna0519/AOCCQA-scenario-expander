# AOCCQA Scenario Expander(測項情境擴充器)

AOCCQA(ASUS EC / Magento)分析產線的 **測項情境擴充器**。對照一份**合格的既有 Test Case Baseline**,前景化「**狀態流轉(狀態流轉)**」與「**身分別(role/identity)**」兩軸,補上可追溯、有證據的覆蓋缺口。擴充的是**測試邏輯**,不是產品需求。

> 產線位置:`aoccqa-fsd-parser`(解析需求→六段報告) → `aoccqa-tc-generator`(產第一版測項) → **`aoccqa-scenario-expander`(對照 baseline 補缺口)** → `aoccqa-quality-reviewer`(Gate 2 審查放行)

---

## 一、如何使用

### 前提:必須有「合格 baseline」

本 skill **不產第一版測項**。啟動時會先跑「執行閘門」,沒有合格既有測項就回 `Needs Baseline` 並路由到 `aoccqa-tc-generator`。合格 baseline 指:維護中回歸測項、前一版功能/需求測項、經 QA 審查或實際執行過的測項、上線/正式核准測項,或 QA 明確指定的已審查測項集。

**必要輸入**(缺就停):

- 已確認 / 帶狀態標記的 **Requirement Matrix**(含 Requirement ID)
- 合格的 **Existing Test Case Baseline**
- **In Scope / Out of Scope** 界定
- 適用時:`Normalized Rule Context` + Rule ID(國別/Website/locale/角色/設定/商品/排程/整合差異)

### 在 Claude Code / Claude 桌面版(Skill)

1. 將本 repo 作為 skill 安裝(放入 skills 目錄或透過 plugin marketplace)。
2. 觸發依據是 `SKILL.md` frontmatter 的 `description`——當你的請求符合「既有測項需要補覆蓋」情境時會自動建議;也可直接點名叫用。
3. 提供上述必要輸入,skill 會依「輸出契約」回傳繁中報告。

### 在 ChatGPT / Codex / API(`agents/openai.yaml`)

介面設定於 [`agents/openai.yaml`](agents/openai.yaml),已開放 `chatgpt` / `codex` / `api` / `atlas`,並允許隱式叫用。預設叫用語:

```
Use $aoccqa-scenario-expander to compare my confirmed requirements and country
rules against a QUALIFIED existing Test Case baseline. Foreground state-transition
(狀態流轉) and role/identity (身分別) coverage, then propose only evidence-backed
additions, enhancements, or parameterization. If no qualified baseline exists,
return Needs Baseline and route to aoccqa-tc-generator.
```

### 職責邊界(它**不做**的事)

- ❌ 不產第一版完整測項 → 路由 `aoccqa-tc-generator`
- ❌ 不解析原始 FSD/PRD/截圖/Figma/API → 路由 `aoccqa-fsd-parser`
- ❌ 不正規化零散/衝突規則 → 回 `Needs Rule Loading`
- ❌ 不靜默改寫、刪除、合併、重排、重新編號或放行既有測項
- ❌ 不把 QA 經驗或歷史行為當產品事實;未定義/衝突結果一律標 `Blocked`
- ✅ 最終仍須交回 `aoccqa-quality-reviewer` 審查放行

---

## 二、此 skill 的內容架構

### 檔案結構

```
aoccqa-scenario-expander/
├─ SKILL.md            # 主檔:frontmatter(觸發 description)+ 七段工作流程 + 輸出契約
├─ agents/
│  └─ openai.yaml      # ChatGPT / Codex / API / Atlas 介面設定與預設叫用語
├─ README.md           # 本文件
├─ LICENSE
└─ .gitignore
```

### `SKILL.md` 的邏輯結構

| 段落 | 內容 |
|---|---|
| **frontmatter** | `name` + `description`——**唯一的觸發依據**,界定何時該用、何時該路由到其他 skill |
| **一、定位與職責邊界** | Requirement Matrix / Rule Context 為權威;既有測項只當覆蓋基線 |
| **二、必要輸入盤點** | 缺就停;不得從缺漏資訊推定 Expected Result |
| **三、執行閘門** | baseline 資格判定 → 回 `Ready` / `Needs Rule Loading` / `Needs Baseline` / `Blocked` / `Out of Scope`,唯 `Ready` 才進工作流程 |
| **四、知識庫 fallback** | 歧義卡住時查 `/aoccqa-knowledge-base`,附「只查不載」的 `jq` token 鐵則 |
| **五、工作流程(Step 1–12)** | 正規化既有覆蓋 → 建覆蓋基線矩陣 → **前景化兩主軸** → 其餘維度 → 國別 Gate → 決策表 → 確認真缺口 → 選處置 → 抽 Test Data → 草擬測項 → 處理 Blocked → 反向檢查 |
| **六、輸出契約** | 12 項依序回傳(閘門結果、狀態流轉矩陣、身分別矩陣、國別適用性、缺口矩陣、強化建議、`NEW-xxx` 草稿、追溯表、Blocked+澄清、重複/parameterize、剩餘未覆蓋) |
| **七、完成準則** | 兩軸完整展開、每提案可追溯有證據、未靜默更動既有測項才算完成 |

**兩大核心軸**(Step 3,主要產出):

- **A. 狀態流轉**:允許轉移 / 禁止轉移 / 終態 / 冪等 / no-op / CTA 旗標 / 排程驅動 / 跨系統同步(Magento↔AOM↔EC)/ 併發競態。
- **B. 身分別**:角色清單(Guest/Member/user_group/Admin/ACL/第三方)× 關鍵動作的可見性、可操作性、權限允許拒絕、資料隔離。

---

## 三、兩個分支的差異(main vs uncompressed)

本 repo 用**兩個分支**維護**同一個 skill 的兩種版本**。兩者的**邏輯、架構、段落、輸出契約與 `agents/openai.yaml` 完全相同**,差別**只在 `SKILL.md` 的內文 token 密度**。

| 分支 | 版本 | `SKILL.md` 大小 | 用途 |
|---|---|---|---|
| **`main`** | **壓縮版(最終版本)** | 229 行 / 約 17.8 KB | 正式使用。措辭精簡,每次載入省 token |
| **`uncompressed`** | 未壓縮完整版 | 230 行 / 約 18.4 KB | 對照 / 審閱參考。措辭較完整易讀 |

差異示例(同一句):

```diff
- 強化**合格既有 Test Case Baseline**,找可追溯、有證據的覆蓋缺口。      # main(壓縮)
+ 強化一份**合格的既有 Test Case Baseline**,找出可追溯、有證據的覆蓋缺口。 # uncompressed(完整)
```

> 兩版**行為完全一致**——壓縮版只是把連接詞、助詞、贅字收斂以節省 token(整體約差 3.5%),不刪任何規則、步驟或判定邏輯。

**該用哪個?**

- 正式部署 / 讓 agent 載入執行 → 用 **`main`**(省 token)。
- 人工審閱、對照規格、教學說明 → 看 **`uncompressed`**(讀起來較順)。

**切換方式:**

```bash
git checkout uncompressed   # 未壓縮完整版
git checkout main           # 壓縮版(正式)
```

> ⚠️ 兩分支刻意保持分離:請**不要**把 `uncompressed` 合併進 `main`(會用完整版覆蓋掉壓縮版,失去省 token 的意義)。

---

## 授權

見 [`LICENSE`](LICENSE)。
