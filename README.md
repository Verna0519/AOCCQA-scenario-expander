# AOCCQA-scenario-expander

AOCCQA 分析產線的**測項情境擴充器**:對照合格既有 Test Case Baseline,前景化
「狀態流轉」與「身分別」兩軸,補可追溯、有證據的覆蓋缺口。

## 分支

| 分支 | 內容 | 用途 |
|---|---|---|
| `main` | **壓縮版 SKILL.md(最終版本)** | 正式使用,每次載入省 token |
| `uncompressed` | 未壓縮完整版 SKILL.md | 對照 / 審閱參考 |

兩分支邏輯、架構、`agents/openai.yaml` 完全相同,僅內文 token 密度不同。

## 檔案

- `SKILL.md` — 主檔(觸發依據為 frontmatter `description`)
- `agents/openai.yaml` — ChatGPT/Codex/API 介面設定
- `README.md` / `LICENSE` / `.gitignore`

## 產線位置

fsd-parser → tc-generator → **scenario-expander** → quality-reviewer
