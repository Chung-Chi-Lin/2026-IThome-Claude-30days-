# 【Day 11】新世代 tokenizer：同樣的中文為什麼變貴了

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

Day 7 講了 token 是怎麼切出來的，也留了一句話沒展開：**Claude 4.7 世代換了新的 tokenizer，同樣的文字會產生更多 token。**

今天把這件事講完整。這不是「中文比英文貴」那種跨語言的比較（那是 Day 7 的主題），而是**同一種語言、換一代模型，帳單自己漲了**——這件事更容易被忽略,因為你的 prompt 一個字都沒改,行為卻變了。

如果你正在把系統從舊世代模型（例如 Sonnet 4.6）遷移到新世代模型（Fable 5、Opus 5、Sonnet 5），今天這篇是遷移前一定要做的功課。

> 本篇 tokenizer 換代的規格與換算方式，於 **2026 年 8 月 14 日**對照 [Token counting 官方文件](https://platform.claude.com/docs/en/build-with-claude/token-counting) 查證。

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

## 一、換代分界線：Opus 4.7，不是 Sonnet 5

先把事實釘死，這是本系列 PROGRESS.md 裡明確記錄過的修正：**tokenizer 換代的分界是 Claude Opus 4.7**，不是很多人直覺以為的「新一代模型系列開始」那個時間點。

官方原文（token counting 文件）是這樣寫的：

> "Claude 4.7 and later models and Claude Mythos Preview use a newer tokenizer. The same input text produces approximately 30 percent more tokens than on earlier models."
>
> （Claude 4.7 及之後的模型，以及 Claude Mythos Preview，使用新的 tokenizer。同樣的輸入文字，產生的 token 數比早期模型**大約多 30%**。）

而 Fable 5 與 Mythos 5——本系列開場就介紹過的最新旗艦——用的正是**這一套從 Opus 4.7 開始引入的 tokenizer**：

> "Claude Fable 5 and Claude Mythos 5 use the tokenizer introduced with Claude Opus 4.7, which produces roughly 30 percent more tokens than models before Claude Opus 4.7 for the same text."

換句話說，這條分界線把模型切成兩群：

| 舊 tokenizer | 新 tokenizer |
| :--- | :--- |
| Sonnet 4.6 及更早、Opus 4.6 及更早 | Opus 4.7 及之後、Fable 5、Mythos 5、Mythos Preview |

**Sonnet 5 用的是哪一套？** 官方文件在列舉新 tokenizer 適用範圍時，明確寫的是「Claude 4.7 and later models」——這句話涵蓋了 4.7 之後所有世代，Sonnet 5 屬於這個範圍內。也就是說，**Sonnet 5 也是新 tokenizer**，只是它不是「換代的起點」，起點是 Opus 4.7。這正是 PROGRESS.md 裡記錄的修正：Day 11 原本的標題誤植成「Sonnet 5 換了 tokenizer」，事實上換代發生得更早。

## 二、大約多 30%，但這個數字不是固定倍率

官方用詞是「**approximately**（大約）」，而且兩處都補了同一句但書：

> "The exact increase depends on the content and workload shape."
>
> （確切的增幅取決於內容與工作負載的形狀。）

這句話很重要，它在提醒你：**30% 是一個帶方向感的量級，不是可以直接套用到任何內容的精確係數。** 程式碼、純英文、含大量專有名詞的技術文件，跟一般敘述性中文文字，增幅很可能不一樣。

官方接著給了唯一正確的驗證方式——**不要用舊模型量到的 token 數去估算新模型的成本或 context window 是否夠用**，而是直接拿你實際的內容，用新模型的 `model` 參數重新算一次。

## 三、實測方法：同一段文字，呼叫兩次

這個方法 Day 7 提過概念，這裡是官方針對「換代衝擊」給的具體做法——**呼叫兩次 token counting 端點，一次用舊模型、一次用新模型，比較兩次的 `input_tokens`**：

```python
import anthropic

client = anthropic.Anthropic()
sample_text = "（你實際會用到的中文內容，貼一段有代表性的）"

old_count = client.messages.count_tokens(
    model="claude-sonnet-4-6",       # 舊 tokenizer
    messages=[{"role": "user", "content": sample_text}],
)

new_count = client.messages.count_tokens(
    model="claude-fable-5",          # 新 tokenizer（Sonnet 5 / Opus 5 亦可）
    messages=[{"role": "user", "content": sample_text}],
)

increase = (new_count.input_tokens - old_count.input_tokens) / old_count.input_tokens
print(f"舊：{old_count.input_tokens} tokens")
print(f"新：{new_count.input_tokens} tokens")
print(f"增幅：{increase:.1%}")
```

拿你**真正會用到的內容**（不是一句打招呼的範例）跑這個腳本，你會得到屬於自己工作負載的實際增幅——這個數字才是你該拿來重新估算預算的依據，而不是官方給的那個「大約 30%」的通用參考值。

## 四、遷移時容易忽略的兩個地方

**① 帳單與計費，直接反映新 tokenizer 的計數。** 官方明講：如果你要把工作負載遷移到 Fable 5 或 Mythos 5，**同樣的內容大約會多消耗 30% 的 token**，計費也是按這個新的計數走。如果你只是把 model 參數從 `claude-sonnet-4-6` 換成 `claude-fable-5`，卻沿用舊模型時期估的預算，你的預算會系統性地偏低。

**② Context window 的可用空間，實際上變窄了。** Day 5 講過 1M context 大約能塞 55 萬英文單字，但那組換算是以英文為基準。換到新 tokenizer 之後，同樣一份中文文件會佔用比舊 tokenizer 更多的 token——這代表你原本以為「這份文件塞得進去」的估算，遷移後可能需要重新確認,尤其是本來就已經接近 context 上限的長文件情境。

## 五、連鎖影響：output 與快取門檻也會跟著動

Tokenizer 換代影響的不只是 input 帳單，還有兩個容易被忽略的連鎖反應。

**① Output 很可能也適用同一套換算邏輯。** 官方文件在講換代時，用字都是「input text produces more tokens」，重點放在計費用的 input 端；但 tokenizer 本身是雙向的——**同一套詞彙表也用來把模型生成的文字編碼成 token**。這代表如果你要求模型生成一段固定字數的中文回答，在新 tokenizer 下，這段回答很可能一樣會被切成比舊 tokenizer 更多的 token。

> **這裡要誠實標記**：官方文件並沒有針對「output 端增幅是否等於 input 端的 30%」給出明確數字，以上是基於 tokenizer 雙向編碼機制的合理推論，不是官方逐字保證。如果你的成本模型高度依賴 output 端的精確估算（例如 Day 2 提過的、output 單價是 input 的 5 倍，這塊估錯影響更大），建議用 Day 7 教的方法，實際跑一次舊模型與新模型的完整請求，比較 `output_tokens` 的差異，而不是假設它跟 input 端同步變動。

**② 快取的最小可快取長度門檻，用 token 數定義，換 tokenizer 後「用字數換算的門檻」跟著變。** Day 9 提過 Opus 5 的最小可快取長度是 512 tokens。在新 tokenizer 下，同樣一段中文內容被切成的 token 數比舊 tokenizer 多，這代表**你需要的原始字數反而變少，就能跨過 512 token 的門檻**——換句話說，遷移到新 tokenizer 之後，原本因為內容太短而沒被快取的中文片段，現在可能剛好夠格被快取了。這是換代帶來少數對中文使用者有利的副作用，值得在遷移後重新檢視一次你的快取策略是否需要調整斷點位置。

## 六、Before / After：遷移前的檢查清單

**❌ Before：直接切模型，沿用舊的成本估算**

```python
# 從舊模型切到新模型，其他什麼都沒動
response = client.messages.create(
    model="claude-fable-5",     # 從 claude-sonnet-4-6 改過來
    max_tokens=4096,
    messages=[{"role": "user", "content": long_document}],
)
# 沿用舊模型時期算好的預算表——這裡開始系統性地低估
```

**✅ After：遷移前先用新模型重新量過一輪**

```python
# 第一步：用有代表性的樣本，量出真實增幅
sample_docs = load_representative_samples(n=20)
increases = []
for doc in sample_docs:
    old = client.messages.count_tokens(model="claude-sonnet-4-6", messages=[{"role": "user", "content": doc}])
    new = client.messages.count_tokens(model="claude-fable-5", messages=[{"role": "user", "content": doc}])
    increases.append((new.input_tokens - old.input_tokens) / old.input_tokens)

avg_increase = sum(increases) / len(increases)
print(f"我的內容平均增幅：{avg_increase:.1%}")   # 拿這個數字重算預算，而不是套用「30%」

# 第二步：確認最長的文件在新 tokenizer 下還進得了 context window
longest_doc_tokens = client.messages.count_tokens(model="claude-fable-5", messages=[{"role": "user", "content": longest_document}]).input_tokens
assert longest_doc_tokens < 900_000  # 留一點餘裕，不要卡在上限邊緣
```

> Before 版本的風險不是「會壞掉」——遷移後系統通常還是能正常運作，這正是最危險的地方：**沒有任何錯誤訊息告訴你帳單估算錯了。** After 版本多做的兩件事——用有代表性的樣本量真實增幅、確認最長文件不會頂到 context 上限——都只是一次性的檢查成本，換到的是遷移後預算不會突然失準。

## 七、一個具體的數字感（僅供直覺參考，非精確公式）

用「大約多 30%」這個量級參考，感受一下換代對一份實際文件的影響：假設一份中文技術文件在舊 tokenizer 下量出來是 10,000 tokens，換到新 tokenizer 之後，同樣的內容大約會落在 13,000 tokens 上下。

如果你原本的 context 使用習慣是「文件塞到 80 萬 token 左右就是安全上限」，換代之後同一批文件實際佔用的 token 數會更高——原本 80 萬 token 能裝的文件量，遷移後可能只裝得下六成左右。這不是精確換算，只是幫你建立「換代不是無感的」這個直覺；**真正的數字，永遠以第三節的雙模型比較法實測為準。**

## 本篇自我挑戰

- **今日挑戰**：如果你手上有任何還在用 Sonnet 4.6 或更早模型的工作負載，挑一份有代表性的內容，跑一次本篇第三節的腳本，量出屬於你自己內容的實際增幅，跟官方的「大約 30%」比較看看差多少。

- **反思**：「同樣的輸入，換一個模型版本，成本就系統性地變了」——這件事平常很容易被忽略，因為多數人切模型時關注的是「品質有沒有變好」，很少人會回頭檢查「帳單的計算基礎有沒有跟著換」。你在做技術選型或版本升級時，有沒有類似「只看功能面、沒看計費面」的檢查清單漏洞？

## 總結

Tokenizer 換代這件事容易被忽略，正是因為**它不會讓程式報錯，只會讓帳單默默變化**。今天釐清了三件事：第一，**換代分界是 Opus 4.7**，不是 Sonnet 5 才開始，Fable 5 與 Mythos 5 沿用的是同一套從 Opus 4.7 引入的 tokenizer；第二，**大約多 30% 是官方給的量級參考，不是精確係數**，實際增幅因內容與工作負載形狀而異；第三，**唯一可靠的驗證方式是拿你自己的內容，跑兩次 token counting，親自比較差異**——不要沿用舊模型時期量到的數字去估新模型的成本或 context 空間。

如果你正準備從舊世代模型遷移到 Fable 5、Opus 5 或 Sonnet 5，把今天第五節的檢查清單存起來，遷移前先跑一遍。

**本日關鍵字回顧**

- **Tokenizer 換代分界**：Claude Opus 4.7 是新舊 tokenizer 的分界點，Sonnet 5 屬於新 tokenizer 世代但不是起點。
- **約多 30%**：官方給的量級參考數字，「exact increase depends on the content and workload shape」，非固定倍率。
- **雙模型比較法**：同一段文字用舊模型與新模型各呼叫一次 `count_tokens`，比較 `input_tokens` 差異，是官方建議的驗證方式。
- **遷移前檢查**：切換模型前應重新估算預算，並確認長文件在新 tokenizer 下仍在 context window 範圍內。

明天我們處理另一種會讓帳單默默膨脹的情境——不是換模型，而是**同一個對話越聊越長**。每一輪都要重新讀完整段歷史，這筆隱藏成本怎麼算、怎麼解。

**Day 12，長對話的成本陷阱與三種解法。**
