# 【Day 5】Claude context window 是什麼？1M token 到底能塞多少東西

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

**1M token。一百萬。**

聽到這個數字，大部分人的第一反應是：「太好了，那我把整個專案都丟進去。」

我也是。我第一次用 1M context 的時候，興高采烈地把一整個 repo 塞進 prompt，然後問了一個很具體的問題——結果拿到一個很籠統的答案，而且它引用的是一個跟問題無關的檔案。

我以為是我 prompt 寫得不好，改了三次。後來才發現，問題出在**我給它的東西太多了**。

這件事不是我的錯覺。Claude 官方文件在講 context window 時，寫了一句很直白的話：

> A larger context window allows the model to handle more complex and lengthy prompts, **but more context isn't automatically better**. As token count grows, accuracy and recall degrade, a phenomenon known as ***context rot***.
>
> （更大的 context window 讓模型能處理更複雜、更長的提示，**但更多的脈絡不會自動更好**。隨著 token 數量增長，準確度與召回率會下降，這個現象稱為「**脈絡腐化**」。）

官方不只承認這件事，還給它取了名字。

今天我們要弄清楚：**context window 裡到底裝了什麼**（比你以為的多）、**1M 實際上是多少東西**、以及**為什麼「塞好塞滿」是錯的策略**。

最後那句官方原話值得先記住：「**curating what's in context is just as important as how much space is available**」——**篩選放什麼，跟有多少空間一樣重要。**

> 本篇規格與官方引文皆於 **2026 年 8 月 8 日**對照 [Context windows 官方文件](https://platform.claude.com/docs/en/build-with-claude/context-windows) 查證。

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

## 一、Context window 是「工作桌面」，不是「硬碟」

先把定義講清楚，因為這裡最容易搞混。

官方的說法是：context window 指的是模型生成回應時**能參照的所有文字，包括回應本身**。它跟模型訓練用的那個龐大語料庫是兩回事——**它是模型的「工作記憶」。**

我用一個比喻。

**Context window 是你的辦公桌桌面，不是你的檔案櫃。**

檔案櫃（訓練資料）裡有幾萬份文件，但你現在手邊能直接翻閱的，只有攤在桌上的那幾份。桌子越大，能同時攤開的文件越多——這是好事。

但你有沒有過這種經驗：桌上堆了三十份文件，你要找其中一份的某一頁，反而找了老半天？桌子大不代表效率高。**攤太多，反而看不見要看的那份。**

這就是 context rot 的直覺版本。

## 二、桌上其實比你以為的擠

第二個容易誤會的地方：**什麼東西會佔用 context window？**

官方列得很清楚——**請求裡的每一樣東西都算**：

| 佔用 context 的項目 | 常被忽略的程度 |
| :--- | :--- |
| System prompt | 低 |
| `messages` 裡的每一則訊息 | 低 |
| 工具執行結果（tool results） | **高** |
| 圖片與文件 | 中 |
| **工具定義（`tools` 參數）** | **高** |
| **Claude 這一輪生成的輸出，含 thinking** | **極高** |

最後兩項是重點。

**工具定義會一直佔著位子。** Day 2 算過，十個工具的 schema 輕鬆破千 token，而且每一次請求都在那裡。它不只是錢的問題，它同時在吃你的桌面空間。

**輸出本身也算。** 這一點違反直覺——很多人以為 context window 是「輸入的上限」。不是，它是**輸入加輸出的總和上限**。模型思考的 thinking token 也在裡面。

還有一個進階細節值得知道：**上一輪的 thinking 區塊會不會留在 context 裡，各模型行為不同。** 官方明載，Opus 4.5 之後的 Opus、Sonnet 4.6 之後的 Sonnet、Fable 5、Mythos 5 會**保留**先前的 thinking 區塊，它們會像一般輸入一樣持續佔用空間並計費；而較早的 Opus / Sonnet 以及**所有 Haiku 模型**則會自動剝除。

## 三、1M 到底是多少東西？

官方給的換算是這樣：

| Context 大小 | 約略英文單字 | 約略字元數 |
| :--- | :--- | :--- |
| **1M tokens** | ~555,000 字 | ~2,500,000 字元 |
| **200k tokens** | ~150,000 字 | ~680,000 字元 |

55 萬英文單字大概是什麼概念？一本《哈利波特：鳳凰會的密令》大約 25 萬字——**所以 1M context 大概是兩本厚小說。**

但這裡有個**中文使用者一定要知道的但書**：

> 上面那組換算是**以英文為基準**的。中文的 token 效率跟英文差很多，同樣的字數會吃掉更多 token。而且更麻煩的是，Claude 4.7 世代換了新的 tokenizer，官方明載**同樣的文字會產生大約多 30% 的 token**。
>
> 所以請**不要**直接拿「555,000 字」去估算你的中文文件。這個坑我用兩整天來講——Day 7 講 tokenizer 原理，Day 11 講換代之後對中文的實際衝擊。今天你只要記住：**中文的實際容量比這張表看起來的少。**

另外兩個實用上限：

- **1M context 的模型，單次請求最多 600 張圖片或 PDF 頁面**；200k 的模型是 100 張。
- **1M 是預設值，不需要 beta header，而且照標準價計費**——沒有長文本溢價。90 萬 token 的請求跟 9 千 token 的請求，每個 token 一樣價錢。

## 四、一個殘酷的事實：快取不會幫你省空間

這是我覺得最多人誤會、而且代價最大的一點。

很多人的心智模型是：「把大文件放進快取，它就不佔位子了。」

官方文件用一句話直接打掉這個想法：

> Cached prompt prefixes still occupy the context window: prompt caching **changes what you pay for those tokens, not whether they count**.
>
> （被快取的前綴仍然佔用 context window：提示快取改變的是你為這些 token 付多少錢，**而不是它們算不算數**。）

請把這句話讀兩遍。

**快取解決的是「錢」的問題，不是「空間」的問題。**

你把一份 500k token 的文件放進快取，第二次呼叫時你只付 0.1× 的價格——很棒。但那 500k **依然佔著 context window 的一半**，而且依然參與 context rot。

> 這也是為什麼 Day 2 的「省錢地圖」跟今天的「空間管理」必須分開看。它們是**兩個獨立的維度**：
>
> - **錢**：快取、批次、換模型 → 降低單價
> - **空間**：篩選、壓縮、清理 → 降低數量
>
> 一個 token 可以很便宜，同時很佔位子。優化其中一個不會自動優化另一個。

## 五、塞爆的時候會發生什麼？

兩種情況，行為不一樣：

**情況一：光是輸入就超過上限**

所有模型都一樣——回一個 400 錯誤：

```text
prompt is too long
```

**情況二：輸入沒超過，但輸入 + `max_tokens` 超過**

Claude 4.5 之後的模型會**接受這個請求**，讓它跑。如果生成過程真的撞到上限，才停下來並回報：

```json
{ "stop_reason": "model_context_window_exceeded" }
```

這個設計比直接拒絕好——你至少拿得到已經生成的部分。但代價是**你必須自己檢查 `stop_reason`**。如果你的程式碼只讀 `content` 不看 `stop_reason`，你會拿到一個被截斷的答案，而且完全不知道它被截斷了。

（這又是 Day 4 講的「靜默失敗」的一種。Day 26 講錯誤處理時會回到這裡。）

想事先避免？官方提供 **token counting API**，可以在送出前先估算這個請求會用掉多少。

## 六、實務上該怎麼做

**❌ Before：能塞就塞，讓模型自己找**

```python
# 把整個 repo 讀進來
all_files = "\n\n".join(read_file(p) for p in glob("src/**/*.py"))

response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    messages=[{
        "role": "user",
        "content": f"{all_files}\n\n請問使用者登入失敗時的錯誤處理在哪裡？"
    }],
)
```

**✅ After：先篩選，只給相關的**

```python
# 先用便宜的方式縮小範圍（grep、檔名規則，甚至用 Haiku 4.5 做前置篩選）
candidates = [p for p in glob("src/**/*.py") if "auth" in p or "login" in p]
relevant = "\n\n".join(f"# {p}\n{read_file(p)}" for p in candidates)

response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    messages=[{
        "role": "user",
        "content": f"{relevant}\n\n請問使用者登入失敗時的錯誤處理在哪裡？"
    }],
)
```

> Before 版本的問題不是「會爆掉」——1M 塞得下，它不會報錯。**它的問題是會安靜地變笨。**
>
> 模型要在幾百個檔案裡找出跟登入有關的那幾行，這件事本身就消耗了它的注意力。而 context rot 的意思正是：東西越多，該被找到的東西越可能被漏掉。你會拿到一個看起來合理、但引用錯檔案的答案——就像我開頭遇到的那樣。
>
> After 版本多做的事只有一件：**在送進模型之前，先用便宜的方法篩一輪**。`grep` 不用錢，檔名規則不用錢，就算用 Haiku 4.5 做前置篩選也只要 1× 的成本。
>
> 這是 Day 4 「路由前置判斷」的另一種形態——**用便宜的東西保護貴的東西。**

如果你的對話真的會長到逼近上限（例如長時間的代理任務），官方也提供了工具：

- **Compaction（伺服器端壓縮，beta）**：自動在伺服器端摘要較早的對話，讓對話能越過 context 上限繼續。支援 Claude 4.6 之後的模型。
- **Context editing**：更細緻的控制，包含清除舊的工具結果、清除 thinking 區塊。

（長對話的成本累積是 Day 12 的完整主題。）

## 七、送出前先量一下：token counting API

上面講了這麼多「不要塞太多」，但你怎麼知道自己塞了多少？

用眼睛估是不準的，尤其中文。官方提供了一支專門的端點，可以在**真正送出之前**先算這個請求會用掉多少 input token——而且官方明載它 **free to use**（不收費）。

（它有獨立的每分鐘請求數上限，依帳號用量層級而定，從 2,000 RPM 起跳。這個額度跟你正常發訊息的額度是**分開計算**的，用它不會吃掉你的主要額度。）

```python
count = client.messages.count_tokens(
    model="claude-opus-5",
    system=SYSTEM_PROMPT,
    tools=MY_TOOLS,                    # 工具定義也算，記得帶進來
    messages=[{"role": "user", "content": long_document}],
)
print(count.input_tokens)              # 送出前就知道
```

這支 API 最實用的兩個場景：

**① 建立你自己的換算基準。** 拿你**實際**會處理的文件跑一次，你就有了「我的資料，每 1000 字大約幾個 token」這個數字。這比任何通用換算表都準——因為它算的是你的內容、你的語言、你的模型。

**② 設一道閘門。** 在送出之前檢查，超過門檻就先做篩選或摘要：

```python
if count.input_tokens > 150_000:
    context = summarize_or_filter(context)   # 先瘦身，別直接硬送
```

> 為什麼門檻要設在遠低於上限的地方？因為**你要防的不是 400 錯誤，是 context rot。**
>
> 撞到 1M 上限會報錯，那反而是好事——至少你知道出事了。真正麻煩的是塞到 60 萬 token、沒有任何錯誤、模型安靜地開始漏東西。**沒有警報的那一段，才是危險區。**
>
> 門檻設多少沒有標準答案，取決於你的任務。但有一個實用的做法：拿 Day 3 那份測試案例，在不同 context 長度下各跑一次，看品質從哪裡開始掉——**那個轉折點就是你的門檻。**

## 本篇自我挑戰

- **今日挑戰**：找一個你最近餵給 Claude 的長 prompt，做一件事——**刪掉一半，然後重問一次同樣的問題**。

  刪的原則是：只留下「回答這個問題絕對必要」的部分。刪完之後比對兩次的答案。

  我自己做這個實驗時的結果是：有一半的情況答案品質**一樣**（那代表我原本浪費了一半的錢），有三成的情況**變好了**（那就是 context rot），只有兩成真的變差（那才是我真正需要的資訊）。你的比例是多少？

- **反思**：為什麼「塞好塞滿」這麼有吸引力？我的答案是——**因為篩選需要我先理解問題，而全部丟進去不用。** 把整個 repo 丟進去，等於把「找出哪些檔案相關」這個工作外包給模型。但這件事恰恰是我們自己做比較快、比較準、也比較便宜的。你有沒有其他「把本來該自己想的事外包給工具」的習慣？

## 總結

Context window 是**工作桌面，不是硬碟**——它是模型的工作記憶，而不是它的知識庫。

桌上的東西比你以為的多：system prompt、所有訊息、工具結果、圖片文件、**工具定義**，以及**模型自己這一輪的輸出與 thinking**。它是輸入加輸出的**總和**上限，不只是輸入。

1M 大約是 55 萬英文單字，但**中文使用者不能直接套用這個數字**——中文的 token 效率不同，而且 4.7 世代換了 tokenizer 之後同樣文字約多 30% token。

最重要的一件事：**快取不會幫你省空間。** 官方明講，它改變的是你為那些 token 付多少錢，而不是它們算不算數。錢和空間是兩個獨立的維度。

而官方自己的結論是最好的收尾：**篩選放什麼，跟有多少空間一樣重要。** 1M 不是讓你不用思考的許可證，它只是把「該放什麼進去」這個問題的重要性放大了。

**本日關鍵字回顧**

- **Context window**：模型生成回應時能參照的全部文字，包含輸入與輸出，是「工作記憶」而非知識庫。
- **Context rot（脈絡腐化）**：官方用詞。隨 token 數量增長，模型的準確度與召回率會下降。
- **快取與空間無關**：Prompt caching 改變 token 的計價，不改變它們是否佔用 context window。
- **`model_context_window_exceeded`**：4.5 之後模型在生成中撞到上限時的 `stop_reason`，需主動檢查，否則會拿到靜默截斷的結果。
- **Compaction（壓縮）**：伺服器端自動摘要早期對話的 beta 功能，讓對話能越過 context 上限延續。
- **Token counting API**：送出前預估 input token 數的免費端點，可用來建立自己的換算基準並設置閘門。

明天是第一階段的收尾。我們把前五天的判斷依據——四階模型的規格、計價的 1:5 結構、由上往下的選型流程、Haiku 的四道判準、context 的取捨——全部收斂成**一張可以貼在螢幕旁邊的決策表**。

**Day 6，一張圖判斷你該用哪一個模型。**
