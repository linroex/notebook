# Anatomy of the `.claude/` 資料夾

> 依據〈Anatomy of the .claude/ Folder〉與 Claude Code 官方文件整理的中文筆記。

## 為什麼 `.claude/` 很重要

很多人把 `.claude/` 當成 Claude Code 自己生成、自己維護的黑盒子，但它其實是整個專案裡最核心的「行為控制面板」。

它決定了 Claude 在你的專案中：

- 讀哪些長期指令
- 會遵守哪些專案規則
- 能直接執行哪些工具與命令
- 有哪些可重用的 workflow / skills
- 是否有專用 subagent 可分工
- 如何保留部分跨 session 記憶

理解這個目錄之後，Claude Code 就不只是「會寫程式的聊天介面」，而更像是一個可配置的開發環境。

## 其實有兩個 `.claude`

Claude Code 的設定分成兩層：

### 1. 專案層級：`<repo>/.claude/`

放在 repo 裡，通常會進版控。這裡的內容屬於團隊共享設定，例如：

- 專案規則
- 團隊共用 commands / skills
- 權限設定
- hooks
- 專案專用 subagents

### 2. 使用者層級：`~/.claude/`

放在使用者家目錄，通常不會進版控。這裡的內容屬於個人偏好與本機狀態，例如：

- 全域 `CLAUDE.md`
- 個人 commands / skills
- 個人 settings
- 本機 session 記錄與 auto-memory

可以把它理解成：

- `repo/.claude/` = 團隊共同語境
- `~/.claude/` = 你自己的工作習慣

## `CLAUDE.md`：最高槓桿的檔案

如果只能先做好一件事，優先把 `CLAUDE.md` 寫好。

Claude Code 在 session 開始時，會把 `CLAUDE.md` 載入上下文，因此這份檔案等於你提供給 Claude 的長期操作說明。

適合寫進去的內容：

- 常用 build / test / lint 指令
- 架構決策與專案背景
- 命名、import、錯誤處理慣例
- 目錄結構與模組分工
- 非直覺但重要的限制

不適合塞進去的內容：

- linter / formatter 已經能表達的規則
- 大段理論性說明
- 太細碎、只對某些檔案有效的規範

### 實務建議

- 盡量控制在 200 行內
- 越短越聚焦，Claude 的遵循率通常越好
- 如果內容開始變肥，就拆到 `rules/` 或 `skills/`

## `CLAUDE.local.md`：個人但只限這個專案

如果某些偏好只屬於你，不該進團隊 repo，可以用：

- `CLAUDE.local.md`

它會和主要的 `CLAUDE.md` 一起被讀，但通常會被 gitignore，不會污染團隊設定。

適合放：

- 個人工作習慣
- 個人偏好的檢查流程
- 不想提交到 repo 的補充指令

## `rules/`：把大份規則拆小、按需載入

當 `CLAUDE.md` 開始膨脹時，`rules/` 就很重要。

你可以把不同主題拆成不同 Markdown 檔，例如：

- `testing.md`
- `api-design.md`
- `frontend.md`
- `db.md`

更關鍵的是，`rules/` 可以透過 frontmatter 做 **path-scoped 規則**，只在 Claude 處理特定檔案時載入。

例如：

```md
---
paths:
  - "src/api/**/*.ts"
---

# API Design Rules

- 所有 endpoint 都必須先做 schema 驗證
- 回傳格式統一為 `{ data }` 或 `{ error }`
```

這種做法的好處：

- 不必把所有規則塞進全域上下文
- 規則更模組化，維護責任更清楚
- Claude 只在需要時才讀取對應規範

## `settings.json`：真正會被強制執行的設定

`CLAUDE.md` 比較像「指導原則」，`settings.json` 則是 Claude Code 會實際執行的設定層。

它可以控制：

- permissions（允許 / 拒絕哪些命令）
- hooks（在某些事件後自動執行腳本）
- model
- env
- output style

### 核心差異

- `CLAUDE.md`：告訴 Claude「應該怎麼做」
- `settings.json`：系統層面限制「能不能做」

### 常見權限策略

**allow** 適合放：

- `npm run *`
- `npm test *`
- 常見唯讀 git 指令
- 一般檔案讀寫工具

**deny** 適合放：

- `rm -rf *`
- 直接打外網的高風險命令
- `.env`、`secrets/` 一類敏感檔案

這個分層很重要：把安全邊界交給 `settings.json`，把協作慣例交給 `CLAUDE.md`。

## `settings.local.json`：個人覆寫

如果你需要只在自己機器上放寬或改變設定，可以用：

- `.claude/settings.local.json`

這通常不會進版控，適合放：

- 個人額外允許的工具
- 個人模型偏好
- 本機專屬設定

## `skills/`：可重用的工作流程封裝

`.claude/skills/` 裡的每個 skill 都是一個小型可重用工作包。

和單純 command 不同的是，skill 可以帶：

- `SKILL.md`
- 補充文件
- 模板
- 腳本
- 其他資源

Claude 可以根據描述自動判斷要不要套用 skill，也可以由你手動觸發。

適合拿來封裝：

- 安全檢查流程
- PR review checklist
- 特定框架的固定處理方式
- 某種常見文件輸出工作

如果 workflow 很常重複，而且有固定步驟，做成 skill 幾乎一定值得。

## `commands/`：顯式觸發的快捷流程

`commands/` 和 skills 很像，但思維上比較偏向：

- 你主動輸入 `/name` 才執行

例如：

- `/review`
- `/fix-issue 123`
- `/deploy staging`

它適合：

- 明確、單次、你想手動控制的流程
- 需要帶參數的固定 prompt

官方文件也提到，現在很多新 workflow 其實更適合直接做成 skill，因為 skill 能附帶支援檔案，擴充性更高。

## `agents/`：讓專門工種在獨立上下文工作

當任務夠複雜，可以建立 subagent。

每個 agent 都可以有：

- 自己的 system prompt
- 自己可用的 tools
- 自己的 model
- 自己的記憶

這樣做的價值是：

- 把大型任務拆出去
- 避免主對話被大量中間推理污染
- 用更便宜或更專門的模型做局部任務
- 透過工具白名單縮小風險面

例如：

- `code-reviewer`
- `security-auditor`
- `docs-writer`

這比讓主 agent 什麼都做，通常更乾淨也更可控。

## 記憶與本機狀態

除了 repo 裡的 `.claude/`，家目錄下的 `~/.claude/` 也很重要。

常見用途包括：

- 全域 `CLAUDE.md`
- 全域 `settings.json`
- 個人 commands / skills
- `projects/` 下的 auto-memory 與 session 資料
- `.claude.json` 這類偏本機狀態的設定

這也是為什麼 Claude 有時候會「記得」某些你沒重新交代的事：

- 可能是來自 auto-memory
- 可能是來自全域設定
- 也可能是之前專案 session 中累積的本機狀態

## 一個務實的導入順序

如果是從零開始配置 Claude Code，我會建議這個順序：

### 第一步：先把 `CLAUDE.md` 寫好

先寫最小可用版本，只放真正高頻、真正重要的事。

### 第二步：補上 `settings.json`

先定好基本安全邊界：

- 允許哪些低風險命令
- 阻擋哪些高風險命令
- 保護哪些敏感檔案

### 第三步：把高頻流程做成 commands 或 skills

例如：

- code review
- issue 修復
- release checklist
- security scan

### 第四步：等規則變多，再拆 `rules/`

當 `CLAUDE.md` 太長、多人協作變複雜，再拆成 topic-based / path-based 規則。

### 第五步：真的有分工需求，再上 agents

當你已經出現明確的專門任務，才開始定義 subagents。

## 我認為最值得記住的幾個觀念

### 1. `.claude/` 不是裝飾，是基礎設施

它不是「有空再整理」的附加物，而是 AI 開發流程的一部分。配置得好，會直接影響 Claude 的穩定度與可預測性。

### 2. `CLAUDE.md` 要短，而且要有用

最常見的錯誤不是沒有寫，而是寫太多。長篇規章不只難維護，也會稀釋真正重要的指令。

### 3. 規則要分層

- 慣例 → `CLAUDE.md`
- 條件式規則 → `rules/`
- 強制與安全邊界 → `settings.json`
- 可重用 workflow → `skills/`
- 專職分工 → `agents/`

把每種責任放對位置，整體才會清楚。

### 4. 先小後大

不用一開始就把 `.claude/` 做成完整平台。通常只要先把 `CLAUDE.md` + `settings.json` 做好，就已經能解決大多數問題。

## 這篇文章帶給我的實際啟發

這篇的重點不只是介紹目錄結構，而是提醒我們：**要把 Claude 當成可配置的協作系統，而不是一次性 prompt 機器。**

當你開始把工作習慣、專案規則、權限控制、重複流程都制度化後，Claude 的輸出品質才會逐漸穩定。

換句話說，真正的生產力提升，不只是模型更強，而是你有沒有把「怎麼讓模型在你的環境裡持續表現好」這件事工程化。

## 參考來源

- 原文：<https://blog.dailydoseofds.com/p/anatomy-of-the-claude-folder>
- 官方文件：<https://code.claude.com/docs/en/claude-directory>
