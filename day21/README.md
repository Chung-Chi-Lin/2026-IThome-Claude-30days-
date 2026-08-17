# 【Day 21】Claude Code 是什麼？安裝與第一次使用完整教學

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

第三階段講的都是「怎麼跟 Claude 溝通」的原理——effort、thinking、prompt 結構。第四階段開始動手，第一站是多數工程師接觸 Claude 的第一個入口：**Claude Code**，一個在終端機裡運作的 AI 編碼助理。

Claude Code 不是「聊天視窗裡貼程式碼」的體驗。它會直接讀你的專案檔案、執行指令、修改程式碼、操作 Git——前面 20 天學的 effort、context 管理、prompt 技巧，在這裡都會變成你每天實際會用到的工具。

今天的目標很單純：**裝起來，跑完第一個任務。** 明天（Day 22）再處理「怎麼用得省」。

> 本篇安裝指令與操作流程，於 **2026 年 8 月 17 日**對照 [Claude Code Quickstart 官方文件](https://code.claude.com/docs/en/quickstart) 查證。

## 目錄

| 天數 | 主題 | 描述 |
| :--- | :--- | :--- |
| Day 1 | Claude 模型怎麼選？2026 最新四階模型完整比較 | Fable 5 / Opus 5 / Sonnet 5 / Haiku 4.5 的定位、規格與適用場景 |
| Day 2 | Claude 的計價邏輯：搞懂 input / output 為什麼差 5 倍 | 不背數字，理解計價結構，建立可長期沿用的成本直覺 |
| Day 3 | 不知道用哪個模型？官方建議「從 Opus 5 開始」背後的思維 | 為什麼預設起手不是最便宜、也不是最強的那個 |
| Day 4 | Claude Haiku 4.5 適合做什麼？便宜模型的正確用法 | 便宜模型不是次等品，是專用工具 |
| Day 5 | Claude context window 是什麼？1M token 到底能塞多少東西 | 用實際檔案量換算，破除「塞越多越好」的迷思 |
| Day 6 | Claude 模型選擇決策表：一張圖判斷你該用哪一個 | 把前五天濃縮成一張可以貼在螢幕旁的決策流程 |
| Day 7 | Token 是什麼？為什麼你的 Claude 帳單比想像中貴 | 從 tokenizer 原理理解中文為什麼特別燒錢 |
| Day 8 | Claude 省 token 的 5 個實用技巧（一般使用者也適用） | 不寫程式也能立刻套用的五個習慣 |
| Day 9 | Prompt Caching 是什麼？讓重複內容只算 10% 費用 | 快取寫入與命中的計價邏輯，以及什麼時候會虧 |
| Day 10 | Claude Batch API 教學：非即時任務直接省一半費用 | 用時間換金錢，非同步任務的正確打開方式 |
| Day 11 | 新世代 tokenizer：同樣的中文為什麼變貴了 | Claude 4.7 世代換了 tokenizer，這對中文使用者的實際影響 |
| Day 12 | 對話越長越燒錢？Claude 長對話的成本陷阱與解法 | 每一輪都重算全部歷史——以及三種切斷成本累積的做法 |
| Day 13 | Claude 用量怎麼監控？成本失控前的預警機制 | 從 usage 欄位到 Console 儀表板，把帳單變成可觀測系統 |
| Day 14 | Claude effort 參數是什麼？五個檔位該怎麼設 | `low` / `medium` / `high` / `xhigh` / `max` 的取捨與實測建議 |
| Day 15 | Adaptive Thinking 是什麼？為什麼你不用再寫「think step by step」 | 模型自己決定何時思考，舊 prompt 技巧為何失效 |
| Day 16 | Claude 回答變淺了？檢查這兩個隱藏設定 | 排查思路：先看 effort，再看 thinking 設定 |
| Day 17 | Claude Prompt 寫法教學：官方最佳實踐的骨架 | 一個可以套用在 90% 情境的 prompt 結構 |
| Day 18 | 用 XML 標籤讓 Claude 輸出更穩定（結構化輸出教學） | 為什麼 Claude 特別吃 XML，以及怎麼設計標籤 |
| Day 19 | System Prompt 怎麼寫？角色設定的正確姿勢 | system 與 user 的分工，以及「你是一位專家」為什麼沒用 |
| Day 20 | Claude 幻覺怎麼防？降低錯誤輸出的實用做法 | 引用來源、允許說不知道、把驗證寫進流程 |
| Day 21 | Claude Code 是什麼？安裝與第一次使用完整教學 | 從安裝到跑完第一個任務，含常見卡關點 |
| Day 22 | Claude Code 省 token 設定：別讓它讀完整個專案 | `CLAUDE.md`、忽略規則與 context 控制的實戰配置 |
| Day 23 | MCP 是什麼？把外部工具接進 Claude 的原理與實作 | Model Context Protocol 的設計哲學與一個可跑的範例 |
| Day 24 | 前端如何呼叫 Claude API？Messages 端點入門 | 第一支 API 請求，以及為什麼不該在瀏覽器直接呼叫 |
| Day 25 | Claude 串流輸出（Streaming）：打造即時回應體驗 | SSE 事件流解析與前端逐字渲染 |
| Day 26 | Claude API 錯誤處理與重試：正式環境該注意什麼 | 429 / 529 的正確退避策略與冪等性設計 |
| Day 27 | 模型分流（Model Routing）是什麼？別再一支模型用到底 | 依任務難度動態選模型的判斷邏輯 |
| Day 28 | LLM 成本優化架構：小模型前置分流 + 大模型收尾 | 一套可落地的分層架構與失敗處理 |
| Day 29 | 從「會用」到「用得對、用得省」：我 30 天的踩坑與心法 | 誠實記錄過程中判斷錯誤的地方 |
| Day 30 | Claude 使用總整理：模型、成本、設定一次看懂 | 全系列濃縮成一份可以收藏的速查表 |

## 一、Claude Code 是什麼

一句話：**Claude Code 是一個在終端機裡運作、能直接讀寫你專案檔案、執行指令的 AI 編碼代理。** 跟網頁版聊天最大的不同是——你不用複製貼上程式碼，Claude Code 自己會去讀專案裡的檔案、理解結構，然後直接動手改。

官方特別提醒一點：**Claude Code 讀取你的專案檔案是隨需進行的，你不用手動把檔案內容貼給它。** 這句話乍看理所當然，但它其實隱含了 Day 5、Day 22 會講到的 context 管理議題——Claude Code 自己決定要讀哪些檔案，讀多讀少，直接影響你的 token 花費。

終端機之外，官方也列出了其他介面：網頁版（claude.ai/code）、桌面應用、VS Code 與 JetBrains 外掛、Slack，以及 CI/CD 情境下的 GitHub Actions 與 GitLab 整合。今天這篇聚焦在最基礎、最通用的終端機 CLI。

## 二、安裝：三種作業系統各自的一鍵指令

官方建議的**原生安裝（Native Install）**是目前的推薦做法，會自動在背景更新到最新版：

**macOS / Linux / WSL：**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell：**

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD：**

```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

官方特別提醒一個 Windows 使用者常見的搞混點：**PowerShell 跟 CMD 的指令不能混用。** 如果你在 PowerShell 裡看到 `The token '&&' is not a valid statement separator`，代表你其實在 PowerShell 裡卻用了 CMD 的指令；如果看到 `'irm' is not recognized`，代表你在 CMD 裡卻用了 PowerShell 的指令。判斷方法很簡單：提示字元開頭有 `PS C:\` 的是 PowerShell，只有 `C:\` 的是 CMD。

如果你偏好套件管理工具，官方也支援：

```bash
# Homebrew（macOS）
brew install --cask claude-code

# WinGet（Windows）
winget install Anthropic.ClaudeCode
```

**這裡有個容易忽略的差異**：官方原生安裝會**自動在背景更新**；但透過 Homebrew 或 WinGet 安裝的版本**不會自動更新**，需要你自己定期執行 `brew upgrade claude-code` 或 `winget upgrade Anthropic.ClaudeCode`。如果你發現自己的 Claude Code 功能好像比教學文章裡少，先檢查是不是很久沒手動升級。

裝完之後確認安裝成功：

```bash
claude --version
```

## 三、登入：三種帳號類型

安裝完成後，在任意目錄下執行 `claude` 就會進入互動模式，第一次使用會提示你登入：

```bash
claude
```

官方列出的可用帳號類型：

- **Claude Pro / Max / Team / Enterprise**（訂閱制，官方標記為推薦）
- **Claude Console**（API 存底儲值制）——第一次用 Console 帳號登入時，系統會自動幫你建立一個叫「Claude Code」的專屬 Workspace，方便你集中追蹤這部分的花費（呼應 Day 13 提過的 Workspace 費用管理概念）
- **企業雲端服務**（Amazon Bedrock、Google Cloud、Microsoft Foundry）

登入成功後憑證會存在本機，之後不需要重複登入。要切換帳號時，在互動階段裡輸入 `/login` 即可。

## 四、第一次使用：從認識專案到動手改程式碼

進到你的專案目錄，啟動 Claude Code：

```bash
cd /path/to/your/project
claude
```

官方建議的起手式，是先讓 Claude 認識你的專案，而不是直接下達修改指令：

```text
what does this project do?
what technologies does this project use?
where is the main entry point?
```

接著可以試著讓它做一個小改動：

```text
add a hello world function to the main file
```

Claude Code 會依序**找到適當的檔案 → 顯示提議的變更 → 依你的權限模式決定要不要先徵求同意 → 執行修改**。

**權限模式是這裡的關鍵設定**：預設模式下，每次改動前都會先問你要不要核准；按 `Shift+Tab` 可以在不同模式間切換——`acceptEdits` 會自動核准檔案編輯，`plan` 則是讓 Claude 只提議方案、不真的動手改。部分帳號還有 `auto` 模式，會在背景做風險檢查，只有反覆被擋下時才轉回詢問你。**第一次使用建議留在預設模式**，親眼看過幾次它怎麼改程式碼，有了信任基礎後再考慮切換到更自動化的模式。

## 五、Git 操作也能用自然語言

Claude Code 把 Git 操作變成對話，這是它跟傳統 CLI 工具很不一樣的地方：

```text
what files have I changed?
commit my changes with a descriptive message
create a new branch called feature/quickstart
show me the last 5 commits
help me resolve merge conflicts
```

不需要背 Git 指令的語法，用你平常描述需求的方式說就好。

## 六、幾個最基礎、天天會用到的指令

| 指令 | 用途 | 範例 |
| :--- | :--- | :--- |
| `claude` | 啟動互動模式 | `claude` |
| `claude "task"` | 執行一次性任務 | `claude "fix the build error"` |
| `claude -p "query"` | 執行一次性查詢後直接退出 | `claude -p "explain this function"` |
| `claude -c` | 繼續當前目錄最近一次的對話 | `claude -c` |
| `claude -r` | 恢復先前的對話 | `claude -r` |
| `/clear` | 清除對話歷史 | 在互動階段內輸入 |
| `/help` | 顯示可用指令 | 在互動階段內輸入 |

**`/clear` 這個指令現在先記住，Day 22 會講到它在省 token 上的關鍵角色**——這是本篇留給明天的第一個伏筆。

## 七、新手容易忽略的三個小技巧

官方給了幾個上手階段就值得養成的習慣：

**① 講具體，不要講模糊。** 「fix the bug」跟「fix the login bug where users see a blank screen after entering wrong credentials」，後者能讓 Claude 直接命中問題，前者常常需要來回幾輪才能對齊。這正是 Day 17 骨架裡「明確直接的指令」原則的實戰版本。

**② 複雜任務先拆成步驟。** 官方範例：

```text
1. create a new database table for user profiles
2. create an API endpoint to get and update user profiles
3. build a webpage that allows users to see and edit their information
```

**③ 讓 Claude 先探索，再動手。** 對一個不熟悉的程式庫，先問「analyze the database schema」這類理解性問題，讓 Claude 建立好背景知識，再交辦實際任務——這比讓它一邊探索一邊改動更穩妥，也更省（Day 22 會展開這個邏輯）。

## 八、不只是終端機：其他官方支援的介面

今天的教學聚焦終端機 CLI，但官方列出的介面其實不只這一種：**網頁版**（claude.ai/code）、**桌面應用**、**VS Code 與 JetBrains IDE 外掛**、**Slack**，以及 CI/CD 情境下的 **GitHub Actions 與 GitLab** 整合。這些介面底層都是同一套 Claude Code，差別在於你怎麼觸發它、它跑在哪個環境裡。

如果你的團隊已經習慣在 IDE 裡工作，VS Code 或 JetBrains 外掛可能比獨立開一個終端機視窗更順手——修改建議會直接疊加在你熟悉的編輯器介面上。如果你想讓 Claude Code 參與 CI/CD 流程（例如自動審查 PR），GitHub Actions／GitLab 整合是對應的入口。今天學到的核心概念——權限模式、`/clear`、Git 對話式操作——在這些介面上是共通的，差別只在於啟動方式跟顯示介面。

## Before / After：從第一次卡關到順手

**❌ Before：模糊指令 + 全自動模式，第一次用就想全開**

```text
$ claude
> 幫我把這個專案改好
```

**✅ After：具體指令 + 預設權限模式，循序漸進**

```text
$ cd my-project && claude
> what does this project do?
（先讓 Claude 建立背景知識）

> there's a bug where users can submit empty forms on the registration page - fix it
（具體描述問題，而不是「改好」這種無法執行的指令）

（在預設權限模式下，逐一核可它提出的修改，直到你熟悉它的做事風格）
```

> Before 版本的問題是雙重的：指令太模糊，Claude 得自己猜你要什麼；權限模式太寬鬆，你看不到它做了什麼決定。After 版本把 Day 17 的「明確指令」原則帶進 Claude Code，並且第一次使用時留在預設的核可模式——**先建立起「它會怎麼做決定」的信任感，之後再考慮要不要開更自動化的模式。**

## 九、裝不起來怎麼辦

如果一鍵安裝指令失敗，官方文件把常見錯誤整理成三種：**`syntax error near unexpected token '<'`**（通常代表網路環境擋掉了安裝腳本、下載到的是錯誤頁面而非真正的安裝檔）、**403 錯誤**（常見於公司網路的防火牆或代理伺服器封鎖）、以及其他 curl 相關錯誤。這三種情況官方都有對應的疑難排解頁面與替代安裝方式（例如改用 Homebrew 或 WinGet，見第二節）。

如果你在公司網路環境下安裝失敗，優先懷疑是防火牆或代理伺服器的問題，而不是急著懷疑自己的指令打錯——一鍵安裝腳本本質上就是對外發出一個下載請求，任何攔截外部連線的網路政策都可能讓它失敗。

## 本篇自我挑戰

- **今日挑戰**：如果你還沒裝過 Claude Code，今天就照著本篇的步驟裝起來，在一個你熟悉的小專案裡跑完「認識專案 → 做一個小改動 → 用自然語言 commit」這三步。如果已經裝過，檢查一下你目前用的是 Homebrew／WinGet 版本還是原生安裝——確認是不是很久沒手動升級了。

- **反思**：Claude Code 把「跟 AI 對話」跟「操作開發環境」合在一起，這跟過去「複製貼上程式碼到聊天視窗」的模式差很多。這種轉變會不會也改變你檢查程式碼的習慣——當修改是自動發生的，你會花多少心力去確認每一個變更，而不是看到「測試通過」就放心？

## 總結

今天把 Claude Code 從安裝到跑完第一個任務走了一遍：**原生安裝指令依作業系統而異，且會自動更新（Homebrew/WinGet 版本不會）**；**三種帳號類型各自的登入方式**；**權限模式決定 Claude 動手前要不要先問你**，新手建議先留在預設模式建立信任;以及幾個從第一天就該養成的習慣——**具體指令、拆解步驟、先探索再動手**。

明天接著處理一個裝完之後很快就會遇到的問題：**Claude Code 讀專案讀得越多，帳單就跟著漲。** 怎麼設定才能不讓它讀完整個專案。

**Day 22，Claude Code 省 token 的實戰設定。**
