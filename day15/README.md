# 【Day 15】Adaptive Thinking 是什麼？為什麼你不用再寫「think step by step」

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

如果你用 Claude 寫過 prompt 超過一年，大概率在某個角落還留著這句話：「Let's think step by step.」或是中文版的「請一步一步思考」。這句咒語曾經是提升推理品質最可靠的招數之一。

在新世代模型上，這句話已經不太需要了——不是因為它沒用，而是因為**模型現在會自己判斷該不該思考**。

這就是今天的主角：**Adaptive Thinking**。Day 14 講的 `effort` 是「模型願意花多少力氣」的旋鈕，今天要講的是那個力氣**具體花在哪裡**——模型會先在內部「打草稿」，把問題想過一輪、試幾種解法、檢查中間結果，再給你最終答案。這個打草稿的過程，新世代模型自己決定要不要做、做多深，不用你在 prompt 裡明講。

> 本篇 thinking 機制、各模型設定規則與錯誤訊息，於 **2026 年 8 月 14 日**對照 [Thinking](https://platform.claude.com/docs/en/build-with-claude/thinking) 與 [Troubleshooting thinking](https://platform.claude.com/docs/en/build-with-claude/thinking-troubleshooting) 官方文件查證。

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

## 一、Thinking 到底在做什麼

官方對 thinking 的定義很直白：一個「一次就要答對」的模型，沒有草稿紙、沒有檢查機會、沒有中途改變主意的餘地。對一道證明題、一個刁鑽的 bug、一個長時程的代理任務來說，**第一個想到的做法往往不是最好的做法。**

Thinking 拿掉了這個限制。啟用時，Claude 會先用自己的話把問題想過一輪——重述問題在問什麼、嘗試不同做法、檢查中間結果、放棄走不通的路——這段推理會出現在回應之前的 `thinking` 內容區塊裡，Claude 再依據這段推理生成最終答案。

**Thinking 是有成本的**：花在推理上的 token，就算最終沒有把 thinking 文字回傳給你，**一樣以 output token 計費**，而且一樣計入 `max_tokens` 的額度——這是 Day 2 提過的重點，今天再次確認。

## 二、Extended Thinking（舊）vs Adaptive Thinking（新）

這是理解今天主題最重要的一組對照：

**Extended Thinking（延伸思考）**：手動模式，你自己設定 `thinking: {"type": "enabled", "budget_tokens": N}`，明確告訴模型「你有 N 個 token 的思考預算」。這是舊世代的做法。

**Adaptive Thinking（自適應思考）**：`thinking: {"type": "adaptive"}`，**模型自己判斷這一次要不要思考、思考多深**，不需要你手動設定 token 預算。CLAUDE.md 早先就記錄過這個方向：新世代模型（Fable 5 / Opus 5 / Sonnet 5）已經沒有 extended thinking 這條路，思考深度改由 Day 14 的 `effort` 控制。

官方的換代時程講得很清楚：**Extended thinking 在 Claude 4.6 世代被標記為棄用（deprecated）**，用它送出的請求仍然會成功，只是不建議繼續用；**Claude 4.7 及之後的模型則直接不支援，送出 `enabled` 會被拒絕，回傳 400 錯誤。**

## 三、逐模型設定表：誰預設開、誰能關、誰拒絕什麼

官方的 troubleshooting 文件給了一份完整對照表，這是目前最精確的版本，比早期記錄更細：

| 模型 | 支援的 thinking 類型 | 預設狀態 | 會被拒絕（400）的值 |
| :--- | :--- | :---: | :--- |
| Fable 5 | 僅 Adaptive | 永遠開啟 | `enabled`、`disabled` |
| Mythos 5 | 僅 Adaptive | 永遠開啟 | `enabled`、`disabled` |
| Mythos Preview | Adaptive、Extended | 永遠開啟 | `disabled` |
| Opus 5 | 僅 Adaptive | 開啟 | `enabled`、`disabled`（僅在 effort `xhigh`/`max` 時） |
| Opus 4.8 / 4.7 | 僅 Adaptive | 關閉 | `enabled` |
| Sonnet 5 | 僅 Adaptive | 開啟 | `enabled` |
| Opus 4.6 / Sonnet 4.6 | Adaptive、Extended（棄用） | 關閉 | 無 |
| Opus 4.5 / Haiku 4.5 / Sonnet 4.5 | 僅 Extended | 關閉 | `adaptive` |

幾個重點看這張表就能一次搞懂：

**① 標「永遠開啟」的模型關不掉思考。** Fable 5 和 Mythos 5 連 `disabled` 都拒絕——這正是 CLAUDE.md 早先記錄的「Fable 5 拒絕 enabled、也拒絕 disabled」。

**② Opus 5 是唯一一個「條件式拒絕 disabled」的模型。** 官方註記很細：**Opus 5 只有在 effort 設為 `high` 或以下時，才接受 `thinking: {"type": "disabled"}`；一旦 effort 是 `xhigh` 或 `max`，這個組合會被拒絕。** 這跟 Day 14 提過的重點是同一件事，只是這裡看到了完整的條件——不是「Opus 5 在 xhigh/max 下永遠不能關 thinking」，而是「這個限制只在你同時嘗試傳 `disabled` 時才會觸發」。

**③ Haiku 4.5 是唯一停留在「僅 Extended」的現役主力模型。** 它拒絕 `adaptive`——這正是 CLAUDE.md 記錄的「Haiku 4.5 不支援 adaptive」。這也再次呼應 Day 1 提過的反直覺結論：**新世代模型的思考機制走的是 adaptive，Haiku 4.5 走的反而是舊的 extended 手動模式**，方向跟「越新的模型功能越新」的直覺是反的。

## 四、三句官方 400 錯誤訊息原文

CLAUDE.md 記錄過其中一句，這裡連同另外兩句一起對照官方原文核實：

**① 對不支援 `enabled` 的模型傳 `enabled`：**

```text
"thinking.type.enabled" is not supported for this model.
Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

**② 對「永遠開啟」的模型傳 `disabled`：**

```text
"thinking.type.disabled" is not supported for this model.
Thinking defaults to adaptive mode when not specified;
use "thinking.type.enabled" with "budget_tokens" for extended thinking.
```

官方特別提醒：這句錯誤訊息裡建議你改用 `enabled`，**但在 Fable 5 和 Mythos 5 上，`enabled` 一樣會被拒絕**——錯誤訊息的建議在這兩個模型上不適用，真正該做的是**完全不要傳 `thinking` 參數**，這些模型不需要任何設定就會思考。

**③ 對只支援 extended 的模型傳 `adaptive`：**

```text
adaptive thinking is not supported on this model
```

這種情況該做的是改用 `thinking: {"type": "enabled", "budget_tokens": N}`。

## 五、為什麼你不用再寫「think step by step」

回到今天的標題。「Let's think step by step」這類提示詞技巧，本質上是在**手動誘發**模型做更多中間推理——在舊世代模型上，這句話確實能讓模型多想一步，因為模型本身不會主動判斷「這題需不需要多想」。

Adaptive thinking 把這個判斷收回模型自己身上。官方的說法是：**這個決定發生在每一次請求層級**——同一段對話裡，有些回合會有 thinking 區塊，有些完全沒有，取決於 Claude 自己評估這次的問題夠不夠複雜。一個簡單的事實性問題可能得到直接回答、完全沒有 thinking 區塊；一道多步驟的數學題或棘手的除錯任務則會觸發更深的推理。

這代表兩件事：

**① 「think step by step」不再是必要的觸發器**，因為模型已經會自己判斷。但官方也留了一道後門——如果你發現模型思考得不夠深，**這件事仍然是可以用 prompt 引導的**，只是引導的方式從「命令式指示」變成「調整判斷門檻」。系統提示裡加上「這個任務需要多步驟推理，回答前請仔細思考」這類語句，可以把模型的觸發門檻往下調；反過來，加上「除非真的能明顯改善答案品質，否則不需要深入思考」則會讓模型更少觸發思考。

**② 不要假設每一輪對話都會有 thinking 區塊。** 官方明確提醒：**不要寫出「假設每個 assistant 回合都以 thinking 區塊開頭」的應用邏輯**——沒有 thinking 區塊是正常行為，不是異常。

## 六、思考也能穿插在工具呼叫之間：Interleaved Thinking

新世代模型還有一個自動獲得的能力：**Interleaved Thinking（交錯思考）**——Claude 可以在工具呼叫之間思考，針對每一次工具回傳的結果先反思一輪，再決定下一步該做什麼。

這個能力**在 adaptive thinking 下自動啟用，不需要額外的 beta header 或任何設定**。對代理型工作流特別有幫助：模型呼叫一個工具、看到結果、想一下這個結果代表什麼，再決定下一步是繼續呼叫工具還是直接回答——整段推理可以貫穿整個 assistant 回合，而不是只發生在最一開始的單次回應裡。

## 七、看不到 thinking 文字？檢查 `display` 設定

如果你發現回應裡的 `thinking` 區塊有 `signature` 欄位，但 `thinking` 文字內容是空字串——這不是壞掉，是**新世代模型的 `display` 預設是 `"omitted"`**，也就是預設不把思考文字回傳給你。

要看到摘要過的思考內容，需要明確設定：

```python
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    thinking={"type": "adaptive", "display": "summarized"},
    messages=[{"role": "user", "content": "..."}],
)
```

**這裡有個計費上的重要細節**：不管 `display` 設成 `"summarized"` 還是 `"omitted"`，**你被計費的 token 數量完全一樣**——都是模型內部實際產生的完整思考量。差別只在於你看不看得到那段文字，不影響帳單。如果你想知道這次請求裡，thinking 實際佔了多少 output token，可以讀 `usage.output_tokens_details.thinking_tokens` 這個欄位。

## 八、如果你手上還有舊的 `budget_tokens` 程式碼

如果你的專案是從舊世代模型遷移過來，程式碼裡可能還留著 extended thinking 的手動預算寫法：

**❌ Before：手動設定思考預算**

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8000,
    thinking={"type": "enabled", "budget_tokens": 4000},  # 手動預算，舊寫法
    messages=[{"role": "user", "content": "..."}],
)
```

**✅ After：改用 adaptive thinking，交給模型自己判斷**

```python
response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=8000,
    thinking={"type": "adaptive"},          # 不用手動設定 token 數
    output_config={"effort": "high"},       # 改用 effort 控制思考深度
    messages=[{"role": "user", "content": "..."}],
)
```

> 舊寫法要你自己猜「這個任務大概需要幾千個 token 的思考預算」，猜多了浪費、猜少了不夠。新寫法把這個猜測的工作交還給模型——**你只需要用 `effort` 表達整體的深度傾向，不用再精算一個具體的 token 數字。** 如果你的模型仍停留在只支援 extended thinking 的世代（例如 Haiku 4.5），這個遷移暫時不適用，繼續用 `budget_tokens` 即可。

## 本篇自我挑戰

- **今日挑戰**：找一個你 prompt 裡還留著「請一步一步思考」或類似字句的地方，如果你用的是新世代模型（Fable 5 / Opus 5 / Sonnet 5），試著把這句話拿掉，比較拿掉前後的回答品質與 `thinking_tokens` 用量差異——你可能會發現拿掉之後結果幾乎沒變，因為模型本來就會自己判斷。

- **反思**：「think step by step」曾經是最可靠的 prompt 技巧之一，現在卻可能是多餘甚至無效的。這提醒我們：**任何依賴模型特定行為的技巧，都有過期的風險。** 你手上還有沒有其他曾經很有效、但現在可能已經過時的 prompt 慣用語？多久檢查一次自己的 prompt 庫，可能是個值得思考的問題。

## 總結

Adaptive Thinking 把「該不該多想一下」這個判斷，從使用者手上交還給模型自己。今天釐清了幾件事：**thinking 的成本以 output token 計費，且計入 `max_tokens`**；**extended thinking 在 4.6 世代被標記棄用、4.7 之後直接不支援**；**逐模型的預設與拒絕規則差異很大**，尤其 Opus 5 的「僅在 `xhigh`/`max` 時拒絕 disabled」是最容易被誤解的條件；以及**「think step by step」這類手動觸發語句已經不是必要條件**，但如果模型觸發不如預期，仍然可以用 prompt 引導這個判斷門檻。

`display` 預設 `"omitted"` 也是很多人第一次遇到「thinking 欄位是空的」時會卡住的地方——記得計費不受 `display` 影響，只有可見度不同。

**本日關鍵字回顧**

- **Thinking（思考）**：模型在正式回答前，於 `thinking` 區塊進行的中間推理過程，以 output token 計費。
- **Adaptive Thinking**：`thinking: {"type": "adaptive"}`，模型自行判斷每次請求是否思考、思考多深，新世代模型的主要模式。
- **Extended Thinking**：舊世代手動模式，`thinking: {"type": "enabled", "budget_tokens": N}`，4.6 世代起棄用，4.7 之後不支援。
- **逐請求判斷**：是否思考的決定發生在每一次請求層級，同一段對話裡有些回合會有 thinking 區塊、有些不會，屬正常行為。
- **Interleaved Thinking**：adaptive thinking 下自動啟用，讓模型能在工具呼叫之間穿插思考，無需額外設定。
- **`display` 設定**：預設 `"omitted"` 不回傳思考文字，設為 `"summarized"` 才看得到，但計費不受影響。

明天把 Day 14 和今天的內容合起來，處理一個實務上很常見的求救訊號——**「Claude 的回答怎麼突然變淺了？」排查思路就從這兩個隱藏設定開始查起。**

**Day 16，回答變淺的排查指南。**
