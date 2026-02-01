# GitHub Repository 管理指南

## 命名規範 (Naming Convention)

採用 `[prefix]-[project-name]-[suffix]` 模式，前綴與後綴視專案性質選用。

命名使用全小寫，單字間以 dash (`-`) 分隔。

#### 前綴 (Prefix) - 專案定位

- `lab`：技術鑽研、課程練習或學習筆記。
- `poc`：針對特定技術問題的小型實驗。
- `pkg`：模組化套件，使用 SemVer 版本管理。
- `tool`：腳本、CLI 工具或自動化程式。

#### 後綴 (Suffix) - 交付狀態

- `starter`：具生產力水準的模板，包含測試、Docker 與 CI 設定。
- `guide`：以教學或文件為主，程式碼為輔。
- `archived`：停止維護，僅供存檔參考（同步設定 GitHub Archive 狀態）。

## Engineering Discipline

### Git Commit Message Convention

採用 `[type](scope): subject` 格式，以使用英文為主。

- type：
  - `feat`：新功能
  - `fix`：修復 Bug
  - `docs`：文件更新
  - `style`：程式碼格式（不影響邏輯）
  - `refactor`：重構程式碼（不是新增功能或修復 Bug）
  - `perf`：改善效能
  - `test`：新增或修改測試
  - `chore`：調整建置工具、套件更新等
  - `revert`：退回先前的 commit
- scope：選填，說明影響範圍，如：模組或檔案名稱。
- subject：簡短描述變更內容。

### Branching

- 主分支命名為 `main`，除非有特殊需求。
- 分支命名採用 `type/description` 格式。

### Merging

- 開發應用程式或服務時，應透過 Pull Request 合併至 `main`。
- PR 描述須包含：做了什麼 (What) 與 為什麼做 (Why)。

### 自動化與安全性

- 非研究型或實驗用專案，應開啟 Dependabot。
- 專案應包含基礎的 GitHub Actions (如：Linter 或 Unit Test)。

## Asset Management

* 靜態資產：存放於 `/assets`。
* 大型檔案：一律使用 Git LFS。
* 安全性：禁止將 Secrets (API Keys, .env) 推送至 Repo，只使用範例檔示意。
