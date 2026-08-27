# 【Day 14】Claude effort 參數是什麼？五個檔位該怎麼設

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

`effort` 這個參數，前面幾天已經被提過好幾次——Day 9 講快取時提過「改變 effort 會讓快取失效」，Day 12 也繞著它打轉。今天終於輪到它自己當主角。

先破除一個常見誤解：**`effort` 不是「模型有多聰明」的開關，是「模型願意花多少 token 把事情做好」的旋鈕。** 同一個模型，同一個問題，`effort` 調低，答案品質不會歸零，但模型會更傾向直接給結論，少繞彎、少反覆檢查、少解釋。調高，則反過來。

今天要把五個檔位講清楚：它們各自的行為、什麼場景該用哪一個、以及一個很多人會踩到的雷——**改變 effort 會讓 Day 9 的快取整個失效**，如果你在同一段對話裡動態調整 effort，帳單可能因此不減反增。

> 本篇 effort 參數的規格、檔位定義與範例，於 **2026 年 8 月 14 日**對照 [Effort 官方文件](https://platform.claude.com/docs/en/build-with-claude/effort) 與 [Steering thinking 官方文件](https://platform.claude.com/docs/en/build-with-claude/thinking-steering-and-cost) 查證。

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

## 一、五個檔位，由低到高

參數位置是 `output_config.effort`，**不是 top-level**——這是 CLAUDE.md 第三節就標記過的重點，官方範例長這樣：

```python
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    messages=[{"role": "user", "content": "分析微服務與單體架構的取捨"}],
    output_config={"effort": "medium"},
)
```

五個檔位由低到高：

| 檔位 | 說明 | 典型場景 |
| :--- | :--- | :--- |
| `low` | 最節省，效率最高，但能力會有一定程度的折損 | 簡單任務，追求最快速度與最低成本，例如 subagent |
| `medium` | 平衡取向，適度節省 token | 需要在速度、成本、效果間取得平衡的代理任務 |
| `high`（**預設**） | 高能力表現 | 複雜推理、困難的程式問題、代理任務 |
| `xhigh` | 為長時程工作延伸的能力 | 超過 30 分鐘、token 預算上看百萬等級的長時程代理與程式任務 |
| `max` | 絕對最大能力，不限制 token 花費 | 需要最深度推理與最徹底分析的任務 |

**沒有 `minimal` 這個檔位**——CLAUDE.md 已經標記過這是先前第三方來源的錯誤說法，這裡再次確認：官方文件裡從頭到尾就是這五個。

官方特別註明一句容易被忽略的行為定義：

> "Setting `effort` to `"high"` produces exactly the same behavior as omitting the `effort` parameter entirely."

**`high` 是預設值，設 `high` 等同不傳這個參數。**

## 二、`xhigh` 比較新，不是每個支援 `max` 的模型都有它

官方原文：「`xhigh` is a newer level; some models that support `max` don't support `xhigh`.」——這句話對應到 CLAUDE.md 已記錄的重點：**不能反過來假設「支援 max 就一定支援 xhigh」。**

各檔位的支援模型範圍（節錄）：

- **`max`**：Fable 5、Mythos 5、Opus 5、Opus 4.8、Mythos Preview、Opus 4.7、Opus 4.6、Sonnet 5、Sonnet 4.6
- **`xhigh`**：Fable 5、Mythos 5、Opus 5、Opus 4.8、Opus 4.7、Sonnet 5
- **五個檔位全支援**：Opus 5

而 **effort 支援清單裡沒有 Haiku 4.5**——CLAUDE.md 已標記的重點在官方文件裡逐字確認：effort 頁面列出的支援模型是 `claude-fable-5`、`claude-mythos-5`、`claude-mythos-preview`、`claude-opus-5`、`claude-opus-4-8`、`claude-opus-4-7`、`claude-opus-4-6`、`claude-sonnet-5`、`claude-sonnet-4-6`、`claude-opus-4-5-20251101`——**這份清單裡確實沒有任何一款 Haiku。** 如果你的應用會動態切換模型，切到 Haiku 4.5 時傳入 `output_config.effort` 不會產生預期效果，這是設計分流邏輯時要記住的邊界。

## 三、effort 管的不只是「想不想」，是「整個回應花多少 token」

這是 effort 跟單純的「thinking 開關」最大的不同。官方講得很直接：

> "The effort parameter affects **all tokens** in the response, including: Text responses and explanations / Tool calls and function arguments / Thinking (when active)."

這帶來兩個實際好處：

**① 不需要先啟用 thinking，effort 照樣有效。** 就算你完全沒開 thinking，`effort` 調低一樣會讓 Claude 的文字回應更精簡直接。

**② 連工具呼叫的效率都會被影響。** 低 effort 時，Claude 傾向：把多個操作合併成更少次工具呼叫、減少工具呼叫的總次數、不多做開場白直接動手、完成後只給簡短確認訊息。高 effort 時則相反：可能呼叫更多次工具、先說明計畫再動手、給更詳盡的變更摘要、程式碼註解也更完整。

這代表 Day 9 提過的「effort 會影響 output token 消耗」不是只影響那段可見的文字回答，而是**整個請求生命週期的 token 花費**——包含 Day 2 提過的「thinking tokens 算 output」那塊，也包含工具使用的部分。

## 四、Effort 是行為訊號，不是硬預算

官方特別用一個 Note 澄清這件事：

> "Effort is a behavioral signal, not a strict token budget. At lower effort levels, Claude will still think on sufficiently difficult problems, but it will think less than it would at higher effort levels for the same problem."

換句話說，**`low` 不代表「絕對不思考」**，遇到真的困難的問題，Claude 在 `low` 底下還是會思考，只是思考的量會比同一個問題在 `high` 底下少。這跟 Day 15 要講的 adaptive thinking 機制是綁在一起的——effort 是調整「模型多願意思考」的旋鈕，不是「思考開／關」的二元開關。

真正硬性限制成本的，是 `max_tokens`——它是這次請求輸出的硬上限，thinking 加上文字回應合計不能超過它。**`effort` 和 `max_tokens` 是兩個不同層次的控制**：`max_tokens` 是絕對上限，`effort` 是這個上限之下，模型願意花多少力氣的軟性引導。如果你把 `effort` 設到 `xhigh` 或 `max`，官方建議把 `max_tokens` 設大一點（例如從 64k 起跳再調整），讓模型有足夠空間思考與行動，否則容易撞上 `stop_reason: "max_tokens"` 被硬性截斷。

## 五、Opus 5 的特殊限制：`xhigh` / `max` 下關不掉 thinking

CLAUDE.md 已經標記過這個重點，這裡用官方原文再次確認：

> "On Claude Opus 5, thinking cannot be disabled at `xhigh` or `max` effort: requests that set `thinking: {"type": "disabled"}` at those levels return a 400 error."

如果你的應用邏輯是「預設關閉 thinking，只有特定情境才開」，切記**不要把這個邏輯跟 `xhigh` / `max` 的 effort 設定綁在一起用在 Opus 5 上**，否則會直接收到 400 錯誤。

另外一個 Opus 5 特有的行為：**effort 控制的是思考量，不是肉眼可見的回應長度。** 官方明講在 Opus 5 上「changing effort does not reliably shorten responses」——如果你的目標是讓回答變短，該做的是 Day 8 提過的「明確要求輸出長度」，而不是指望調降 effort 就能讓文字變少。

## 六、跨請求改變 effort，會讓快取直接失效

Day 9 提過這個結論，這裡官方給了一組實測數字，直接示範發生了什麼事。三次連續請求，前兩次維持同樣的 thinking 設定與 effort，第三次把 effort 從預設的 `high` 改成 `medium`：

```text
第一次請求（建立快取）
cache_creation_input_tokens: 3546, cache_read_input_tokens: 0

第二次請求（相同設定，預期命中快取）
cache_creation_input_tokens: 0, cache_read_input_tokens: 3546   ← 命中！

第三次請求（effort 從 high 改成 medium，預期快取失效）
cache_creation_input_tokens: 3546, cache_read_input_tokens: 0   ← 沒命中，整段重寫
```

第三次的 `cache_read_input_tokens` 掉回 0，`cache_creation_input_tokens` 又要重寫一次——**改變 effort 這個動作本身，讓原本已經建立好的快取整個作廢。** 官方的解釋是：effort 的值會被渲染進實際送給模型的 prompt 裡，所以它跟改變 thinking 設定一樣，會讓快取比對的前綴不一致。

有個容易被誤解的細節要澄清：**把 `effort` 明確設成模型的預設值（例如明確傳 `"high"`），效果等同不傳，不會打斷快取。** 只有「傳了一個跟之前不一樣的值」才會造成快取失效。

## 七、Before / After：在同一段對話裡動態調整 effort 的代價

**❌ Before：每一輪都依任務難度即興調整 effort**

```python
messages = []
for turn in conversation_turns:
    difficulty = estimate_difficulty(turn)
    effort = "low" if difficulty < 3 else "high"   # 每輪都可能不一樣

    messages.append({"role": "user", "content": turn})
    response = client.messages.create(
        model="claude-opus-5",
        max_tokens=4096,
        messages=messages,
        output_config={"effort": effort},   # 快取在這裡反覆被打斷
    )
    messages.append({"role": "assistant", "content": response.content})
```

**✅ After：整段對話固定一個 effort，用 prompt 語句做細部調整**

```python
FIXED_EFFORT = "high"   # 整個對話期間維持不變，保住快取

messages = []
for turn in conversation_turns:
    content = turn
    if is_trivial(turn):
        content += "\n\n(這題很直接，請直接回答，不用深入分析。)"  # 用語句微調，不動 effort

    messages.append({"role": "user", "content": content})
    response = client.messages.create(
        model="claude-opus-5",
        max_tokens=4096,
        messages=messages,
        output_config={"effort": FIXED_EFFORT},
    )
    messages.append({"role": "assistant", "content": response.content})
```

> Before 版本每一輪都可能打斷快取，等於 Day 9 講的快取策略在這種對話模式下完全發揮不了作用。After 版本把 effort 固定在整段對話期間不變，改用**附加在使用者訊息裡的自然語言指引**去微調模型在單輪的思考深度——這正是 Day 15 會展開的「per-message steering」技巧,官方明講這種寫法「leaves earlier cache breakpoints intact」，不會動到已經建立的快取前綴。
>
> 官方給的最佳實踐排序也是這個邏輯：**先用 effort 設定整個工作負載的預設平衡點，如果同一個 effort 下某些請求的觸發行為還是不符合需求，才用 prompt 語句做微調**——而不是把 effort 本身當成逐輪調整的旋鈕。

## 八、不寫程式的人：在 Claude Code 裡調 effort

前面七節都在講 `output_config.effort` 這個 API 參數。但如果你是在 Claude Code 裡工作，**這個旋鈕一樣在你手上，而且更好轉。**

**`/effort` 指令**有三種用法：

```text
/effort              → 開一個互動滑桿讓你選
/effort medium       → 直接設定成指定檔位
/effort auto         → 重置回這個模型的預設值
```

也可以在 `/model` 選單裡用**左右方向鍵**調整 effort 滑桿，或啟動時用 `--effort` 指定單一 session。

**怎麼確認現在跑在哪一檔？** 看 session 標題列——它會顯示在模型名稱旁邊，例如「with low effort」。啟動時和每次變更時，底部也會短暫顯示。**這比 API 情境好排查得多，你不用翻程式碼就知道現在是什麼設定。**

**各模型支援的檔位不一樣，設錯會自動降級：**

| 模型 | 可用檔位 |
| :--- | :--- |
| Fable 5、Opus 5、Sonnet 5、Opus 4.8、Opus 4.7 | `low` / `medium` / `high` / `xhigh` / `max` |
| Opus 4.6、Sonnet 4.6 | `low` / `medium` / `high` / `max`（沒有 `xhigh`） |

官方說明：**如果你設了目前模型不支援的檔位，Claude Code 會自動退到「不高於你設定值的最高可用檔位」**——例如在 Opus 4.6 上設 `xhigh`，實際會跑 `high`。這呼應第二節講的「支援 `max` 不代表支援 `xhigh`」，只是 Claude Code 幫你做了優雅降級，不會直接報錯。

**預設值也有一個例外**：大多數模型預設是 `high`，但 **Opus 4.7 預設是 `xhigh`**。

### 一個特例：`ultrathink` 關鍵字

這個發現很有意思，而且它是 Day 15 那個論點的**唯一例外**。

Day 15 會講「你不用再寫 think step by step 了」。但 Claude Code 認得一個關鍵字——**在 prompt 裡任何位置寫 `ultrathink`，可以要求模型在這一輪做更深的推理，而且不改變你的 session effort 設定。**

官方特別澄清了一件事：

> Claude Code passes other phrases such as "think", "think hard", and "think more" through as ordinary prompt text and doesn't recognize them as keywords.
>
> （「think」「think hard」「think more」這類詞會被當成普通的 prompt 文字傳過去，**不會被辨識為關鍵字**。）

換句話說：**那些你習慣寫的「請仔細思考」咒語，在 Claude Code 裡真的只是普通文字**（雖然它們仍可能透過一般的 prompt 引導產生效果，見 Day 15）；只有 `ultrathink` 這一個字是 Claude Code 明確辨識的開關。

什麼時候用它？**當你不想為了一題困難的問題，把整個 session 的 effort 都調高**——調高 effort 會打斷快取（第六節），而 `ultrathink` 只影響這一輪。

## 九、effort 跟換模型，是兩個不同層級的旋鈕

容易被混在一起的兩件事：**調 effort** 跟 **換模型**（Day 3、Day 4、Day 6 講過的選型邏輯），都能達到「省成本」的效果，但作用的層級不一樣。

**換模型**是換一整套能力範圍——例如從 Opus 5 換成 Haiku 4.5，換的是模型本身的推理能力上限、知識廣度、適合的任務類型。**調 effort**是在**同一個模型**的能力範圍內，決定這次要不要用滿。用 Day 6 的決策樹比喻：先用那張表決定「該用哪個模型」，決定好模型之後，才輪到 `effort` 決定「這個模型這次要多認真」。

實務上的判斷順序：**如果一個任務用最便宜的模型、開最高 effort 都做不好，那是換模型的訊號，不是調 effort 的訊號。** 反過來，如果任務用某個模型能做好，只是想再省一點成本或延遲，那才是 effort 該出場的時候。把兩個旋鈕的職責分清楚，能避免陷入「一直調 effort 卻怎麼調都做不好」的死胡同——那種情況通常代表你該回頭看 Day 6 的決策樹，換一個模型，而不是繼續在 effort 的五個檔位裡打轉。

## 本篇自我挑戰

- **今日挑戰**：檢查你目前所有呼叫 Claude 的地方，有沒有任何一處是「同一段對話裡動態切換 effort」的寫法。如果有，改成本篇第七節的 After 版本，比較看看快取命中率的變化。

- **反思**：`effort` 這個名字取得很巧妙——它暗示的是「模型願意付出多少努力」，而不是「模型有多聰明」。這個命名選擇會不會也在暗示一件事：**很多時候我們以為的「品質差距」，其實是「努力程度差距」？** 你在管理自己的工作或團隊時，有沒有類似「調低努力程度、品質沒有等比例下降」的經驗？

## 總結

`effort` 是貫穿整個「設定與調校」階段的核心參數，今天釐清了五件事：**五個檔位由低到高是 `low` / `medium` / `high` / `xhigh` / `max`，沒有 `minimal`**；**預設是 `high`，等同不傳**；**它影響的是整個回應的所有 token（文字、工具呼叫、thinking），不只是思考量**；**它是行為訊號而非硬預算，真正的硬上限是 `max_tokens`**；以及今天最重要的實務提醒——**改變 effort 會讓快取失效，同一段對話期間應該固定 effort，用 prompt 語句做細部調整。**

Opus 5 有兩個特別的地方要記住：`xhigh`/`max` 下無法關閉 thinking（會 400），以及改變 effort 不保證能縮短肉眼可見的回應長度。

**本日關鍵字回顧**

- **`output_config.effort`**：控制 Claude 整體回應 token 花費的參數，五個檔位 `low`/`medium`/`high`/`xhigh`/`max`。
- **行為訊號，非硬預算**：effort 調整的是模型的思考傾向，真正的硬上限來自 `max_tokens`。
- **Effort 影響全部 token**：涵蓋文字回應、工具呼叫與 thinking，不需要額外啟用 thinking 才有效。
- **快取失效**：跨請求改變 effort 值會讓對話快取失效，官方建議整段對話固定 effort，改用 prompt 語句微調。
- **Opus 5 特例**：`xhigh`/`max` 下無法關閉 thinking（回 400），且 effort 不保證縮短可見回應長度。
- **`/effort` 指令**：Claude Code 端的對應操作，可開滑桿、直接指定檔位或 `auto` 重置；不支援的檔位會自動降到最高可用值。
- **`ultrathink` 關鍵字**：寫在 prompt 任何位置可要求該輪深入推理而不改動 session 設定；「think」「think hard」則**不被辨識**為關鍵字。

明天接著講跟 effort 綁在一起的另一半——**Adaptive Thinking**。為什麼你不用再寫「think step by step」了，模型自己怎麼決定要不要思考、思考多深。

**Day 15，Adaptive Thinking 完整原理。**
