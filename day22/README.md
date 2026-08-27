# 【Day 22】Claude Code 省 token 設定：別讓它讀完整個專案

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

Day 21 結尾留了一個提醒：「Claude Code 讀專案讀得越多，帳單就跟著漲。」今天把這件事講透。

好消息是，**第二階段學過的每一個省錢原則，在 Claude Code 裡幾乎都有對應的實戰設定**——快取（Day 9）、壓縮（Day 12）、模型選型（Day 3、Day 4）、context 管理（Day 5）。今天不是學新東西，是把這些原則套進一個你每天都會用到的工具裡。

先講一個官方给的量級感受：**跨企業部署的統計，平均每位開發者每天大約花 13 美元，每月 150-250 美元，90% 的使用者每天低於 30 美元。** 這不是恐嚇你花費失控，是提醒你——這個工具的花費規模跟前面 13 天講的計價邏輯是同一套，值得認真管理。

> 本篇設定與策略，於 **2026 年 8 月 17 日**對照 [Manage costs effectively 官方文件](https://code.claude.com/docs/en/costs) 查證。

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
| Day 29 | Vibecoding 做出網站之後：AI 不會主動告訴你的那些事 | 門檻降低的是「做出來」，不是「做對」——怎麼問出你不知道要問的問題 |
| Day 30 | Claude 使用總整理：模型、成本、設定一次看懂 | 全系列濃縮成一份可以收藏的速查表 |

## 一、CLAUDE.md：每次啟動都要付的入場費

`CLAUDE.md` 是 Claude Code 的專案記憶檔——本系列自己的 `CLAUDE.md` 就是活生生的例子。官方提醒一個容易被忽略的成本結構：

> "Your CLAUDE.md file is loaded into context at session start. If it contains detailed instructions for specific workflows (like PR reviews or database migrations), those tokens are present even when you're doing unrelated work."

**`CLAUDE.md` 在每次啟動階段就整份載入 context**，就算你今天要做的事跟裡面某段特定流程完全無關，那段內容的 token 一樣付了。官方的具體建議：**把 `CLAUDE.md` 控制在 200 行以內，只放真正的基本原則。**

那專門情境的詳細流程（例如「PR 審查該檢查哪些項目」）該放哪？官方的答案是 **Skills**——skills 是**隨需載入**，只有真正被呼叫到才會進 context，不會像 `CLAUDE.md` 那樣每次啟動都攤開來付費。**把專門流程從 `CLAUDE.md` 搬到 skills，是最直接的一項瘦身動作。**

## 二、`/clear` 跟 `/compact`：兩種不同的歷史處理方式

Day 21 埋的伏筆在這裡兌現。這兩個指令對應的是完全不同的情境：

**`/clear`——切換到不相關任務時用。** 直接清空對話歷史，Day 12 提過的「每一輪都要重付整段歷史」問題，用 `/clear` 直接歸零。官方提醒：切換到不相關的工作卻沒有清空，**殘留的舊 context 會在你之後的每一則訊息裡持續耗費 token**。想之後找回這個對話？先用 `/rename` 幫它取個好認的名字，再 `/clear`，之後可以用 `/resume` 找回來。

**`/compact`——延續同一個任務，但歷史開始變長時用。** 這正是 Day 12 講的伺服器端摘要機制在 Claude Code 裡的實際入口。你甚至可以指定摘要時要保留什麼：

```text
/compact Focus on code samples and API usage
```

也可以直接寫進 `CLAUDE.md`，讓每次自動壓縮都套用同一套規則：

```markdown
# Compact instructions

When you are using compact, please focus on test output and code changes
```

**判斷原則很簡單**：任務還是同一個、只是聊得比較久 → `/compact`；已經換了完全不相關的任務 → `/clear`。混著用最常見的錯誤，是明明換了任務卻捨不得清空——省下的「重新解釋一次背景」的力氣，換來的是接下來每一輪都在多付根本用不到的舊 context。

## 三、選對模型：不是每個任務都要 Opus

Day 3、Day 4、Day 6 講過的選型邏輯，在 Claude Code 裡直接對應到日常操作：

> "Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning."

**多數程式任務用 Sonnet 就夠，Opus 留給複雜的架構決策或多步驟推理**——用 `/model` 隨時切換，或在 `/config` 裡設定預設模型。如果你的專案有 subagent 配置（處理特定子任務的專用代理），簡單的子任務可以直接指定 `model: haiku`，這正是 Day 4 講的「便宜模型的正確用法」在 Claude Code 裡的實作。

## 四、MCP 開銷：工具定義是預設延遲載入的

Day 9 提過工具定義本身要算 token，Claude Code 裡的 MCP 伺服器同樣適用這個邏輯，但官方有個好消息：

> "MCP tool definitions are deferred by default, so only tool names enter context until Claude uses a specific tool."

**MCP 的工具定義預設是延遲載入**——只有工具名稱會先進 context，完整的工具描述要等到真正被使用時才載入。想知道目前 context 被什麼佔用，執行 `/context` 就能看到明細。

官方還給了兩個實務建議：**優先用 CLI 工具而非 MCP**（`gh`、`aws`、`gcloud` 這類指令列工具不會有任何工具清單的 context 開銷，Claude 可以直接執行指令）；**用 `/mcp` 檢查目前設定的伺服器，停用沒在用的**——沒在用卻還留著的 MCP 伺服器，一樣佔著 context 空間。

## 五、把處理邏輯下放：Hooks 跟 Skills

這是最容易被忽略、但效益很大的一招。官方給了一個具體場景：與其讓 Claude 讀完一份一萬行的 log 檔案去找錯誤，不如用一個 **hook** 先做前置過濾：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "~/.claude/hooks/filter-test-output.sh" }]
      }
    ]
  }
}
```

```bash
#!/bin/bash
input=$(cat)
cmd=$(echo "$input" | jq -r '.tool_input.command')

if [[ "$cmd" =~ ^(npm test|pytest|go test) ]]; then
  filtered_cmd="$cmd 2>&1 | grep -A 5 -E '(FAIL|ERROR|error:)' | head -100"
  echo "{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\",\"updatedInput\":{\"command\":\"$filtered_cmd\"}}}"
else
  echo "{}"
fi
```

這段 hook 在每次執行測試指令**之前**攔截，只留下失敗訊息，把原本可能上萬 token 的完整輸出，壓縮成幾百 token 的重點——**這正是 Day 8 技巧二「先篩選，再貼上」的自動化版本**，只是這次篩選的是 Claude 自己執行工具的輸出，不是你手動貼的內容。

**Skills** 則是另一個方向：與其讓 Claude 自己讀好幾個檔案去搞懂你的專案架構，不如寫一個「codebase-overview」skill 直接描述架構、目錄慣例、命名規則——**呼叫 skill 一次拿到的資訊，比讓它自己 grep 探索省得多。**

## 六、快取生命週期：訂閱制跟 API 金鑰不一樣

這是一個容易讓長時間 session 突然變貴的細節。官方明講：**快取的存活時間，訂閱帳號是 1 小時，用 usage credits 時降到 5 分鐘，API 金鑰或雲端供應商預設是 5 分鐘。**

> "Cache misses: your first message after a break longer than the cache lifetime misses the cache and reprocesses your full context."

**休息超過快取存活時間之後傳的第一則訊息，會整段快取失效、重新處理全部 context**——這也是為什麼「開著視窗放一整天，中午休息完回來問一句話，帳單卻很有感」的體感由來：不是那句話很貴，是那句話**觸發了整段舊 context 的重新計費**。如果你是訂閱帳號並且在用 usage credits，可以設定環境變數 `ENABLE_PROMPT_CACHING_1H=1` 保留 1 小時的存活時間，而不是降到 5 分鐘。

## 七、哪些操作會打斷快取（這張表值得貼在螢幕旁）

Day 9 講過快取的原理是**前綴匹配**：只要前面那一段一模一樣就命中，中間有任何一個字不同，後面全部要重算。Claude Code 幫你把請求排成「系統提示 → 專案脈絡 → 對話歷史」三層，讓最不常變的排最前面——**但有些操作會直接改動前面那幾層，一改就是整段重算。**

官方把這些動作列得很完整：

| 會打斷快取（下一輪變慢也變貴） | 不會打斷（可以放心做） |
| :--- | :--- |
| `/model` 換模型（每個模型有各自的快取） | 編輯你 repo 裡的檔案 |
| `/effort` 改檔位 | 切換權限模式（Shift+Tab） |
| 開啟 fast mode | 呼叫 skill 或 slash command |
| 接上或斷開 MCP 伺服器 | `/recap` |
| 啟用／停用 plugin | **`/rewind`** |
| 用 deny 規則整個封鎖某個工具 | 開 subagent |
| `/compact` | |
| 升級 Claude Code 後 resume 舊 session | |

官方給的操作原則很簡單：

> "Pick your model and effort level at the top of a session, then save `/compact` for natural breaks between tasks. **The fewer changes you make mid-task, the higher your cache hit rate.**"

**開場就把模型跟 effort 選定，中途不要改。** 這句話呼應 Day 14 提過的「同一段對話裡固定 effort」——當時是從 API 角度講的，在 Claude Code 裡是同一件事，只是換成 `/model` 和 `/effort` 兩個指令。

**三個容易踩到的細節：**

**① 走錯路想回頭時，`/rewind` 比 `/compact` 便宜。** rewind 是把對話截斷回到之前某一輪，而**那個前綴早就快取過了，直接命中**；`/compact` 是生一份新摘要，等於建立全新前綴，得從頭寫入。官方原文：「Rewinding truncates back to a prefix that is already cached, rather than building a new one as compaction does.」

**② 中途編輯 `CLAUDE.md` 不會打斷快取——但也不會生效。** 這點對本篇特別重要，因為前面整篇都在教你怎麼寫 `CLAUDE.md`。官方明講：專案根目錄與使用者層級的 `CLAUDE.md` **在 session 開始時讀取一次就留在記憶體裡**，你中途改它，Claude 仍然用開場時載入的那個版本。**新內容要等下一次 `/clear`、`/compact` 或重開才會套用。**（例外是子目錄裡的巢狀 `CLAUDE.md`，那些是 Claude 第一次讀到對應檔案時才載入，在載入前修改是會生效的。）

**③ 快取範圍是「同一台機器 + 同一個目錄」。** 兩個不同目錄開的 session 不共用快取——**包含同一個 repo 的不同 git worktree**，因為系統提示裡帶了工作目錄路徑，路徑不同前綴就不同。如果你習慣開多個 worktree 平行作業，要知道它們各自在養自己的快取。

（反過來，**同一個目錄下平行開的多個 session 會互相讀到對方的快取**，這是少數可以「共用」的情況。）

## 八、寫具體 prompt，這條原則到哪裡都成立

Day 17 的黃金法則在 Claude Code 裡一樣適用：

```text
❌ 「improve this codebase」→ 觸發大範圍掃描
✅ 「add input validation to the login function in auth.ts」→ 精準定位，最少檔案讀取
```

模糊的請求會讓 Claude 為了搞懂你要什麼而擴大搜尋範圍，具體的請求能讓它幾乎不用探索就知道該改哪裡。

## 九、追蹤花費：`/usage` 跟 `/insights`

養成定期檢查的習慣，比事後回頭查原因有效率得多。`/usage` 顯示目前 session 的詳細花費，包含依模型拆分的 token 用量；如果你是訂閱帳號，還能看到扣抵方案額度的分項統計，包含依 skill、subagent、外掛、個別 MCP 伺服器拆分的佔比——**這能直接告訴你,是哪一個 MCP 伺服器或哪一類操作在吃掉你大部分的額度**，比憑印象猜測準確得多。

`/insights` 則是另一種角度的報告——它分析的不是「花了多少 token」，而是「你平常怎麼工作」：常見的摩擦點（例如指令被誤解、產生有 bug 的程式碼）、以及怎麼用得更有效率的建議。這份報告會寫成 HTML 檔存在本機，適合每隔一段時間跑一次，回顧自己的使用模式有沒有值得調整的地方。

## Before / After：一個典型的長時間開發 session

**❌ Before：開一整天不清、什麼都塞進 CLAUDE.md**

```text
$ claude
（早上開始，一路工作到下午，中間換了三個完全不相關的功能，從沒 /clear 過）
（CLAUDE.md 裡塞了 500 行，包含各種特殊流程的詳細步驟）
```

**✅ After：任務邊界清楚、專門流程搬去 skills**

```text
$ claude
> what does this project do?
（開始第一個功能）
...
/rename 功能A-開發
/clear
（開始不相關的第二個功能，先清空）
...
/compact 保留程式碼變更與測試結果
（同一功能內對話變長，用 compact 而不是整個清掉）
```

搭配一份精簡到 200 行內的 `CLAUDE.md`，加上把「PR 審查標準」「資料庫遷移流程」這類專門知識搬進對應的 skills。

> Before 版本每一句話都在幫整個對話史付費，而且每次啟動都要吞下一份過胖的 `CLAUDE.md`。After 版本用 `/clear` 劃清任務邊界、用 `/compact` 處理同任務內的歷史增長、把專門流程移出常駐 context——三個動作，分別對應 Day 12 的「切斷成本累積」跟今天的「入場費瘦身」。

## 本篇自我挑戰

- **今日挑戰**：執行 `/context` 看看你目前 session 的 context 都被什麼佔走了，再執行 `/usage` 看這個 session 的實際花費。檢查你的 `CLAUDE.md` 有沒有超過 200 行，如果有，找出哪些段落其實只在特定情境下用得到，考慮搬去 skills。

- **反思**：「開著同一個對話视窗做完全不相關的事」是一個很容易養成的懶惰習慣——因為 `/clear` 感覺像是要重新來過。但今天學到的是，**不清空的代價是每一句話都在幫你不需要的舊 context 付錢。** 你有沒有其他因為「重新開始感覺麻煩」而持續累積不必要成本的習慣？

## 總結

Claude Code 的省錢邏輯，本質上是第二階段全部原則的實戰應用：**`CLAUDE.md` 控制在 200 行內、專門流程搬去 skills**（減少每次啟動的入場費）；**`/clear` 處理任務切換、`/compact` 處理同任務內的歷史增長**（呼應 Day 12）；**選對模型、MCP 延遲載入、CLI 優先於 MCP**（呼應 Day 3、Day 4、Day 9）；**用 hooks 前置過濾工具輸出、用 skills 取代自行探索**（呼應 Day 8 的「先篩選再貼上」）；以及一個容易被忽略的細節——**快取存活時間因帳號類型而異，長時間 session 的第一句話特別容易觸發重新計費。**

**本日關鍵字回顧**

- **`CLAUDE.md` 200 行原則**：每次啟動整份載入 context，專門流程應搬去隨需載入的 skills。
- **`/clear` vs `/compact`**：前者用於切換不相關任務、後者用於同任務內歷史過長，混用是常見的浪費來源。
- **MCP 延遲載入**：工具定義預設只先進工具名稱，真正使用時才載入完整描述；`/context` 可查看目前佔用明細。
- **Hooks 前置過濾**：在工具執行前攔截並精簡輸出，把上萬 token 的原始內容壓縮成重點。
- **快取存活時間差異**：訂閱帳號 1 小時、API 金鑰預設 5 分鐘，休息超過此時間後的第一句話會觸發整段重新計費。
- **打斷快取的動作**：`/model`、`/effort`、fast mode、MCP 增減、plugin 開關、`/compact` 都會重算前綴；編輯檔案、切權限模式、`/rewind` 則不會。
- **`CLAUDE.md` 的載入時機**：session 開始時讀一次即駐留記憶體，中途編輯不會生效也不會打斷快取，須 `/clear`、`/compact` 或重開才套用。

明天離開 Claude Code，回到更底層的問題——**Claude Code 能操作外部工具（檔案、Git、MCP 伺服器），這個「連接外部世界」的能力，背後的協定原理是什麼？**

**Day 23，MCP 是什麼？把外部工具接進 Claude 的原理與實作。**
