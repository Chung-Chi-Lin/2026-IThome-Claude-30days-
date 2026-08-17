# 【Day 16】Claude 回答變淺了？檢查這兩個隱藏設定

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

「Claude 最近怎麼變笨了？」——如果你在工程師社群待過，這句話大概每隔幾個月就會出現一次。多數時候，模型本身沒有變，變的是**你的請求裡某個設定，跟你以為的不一樣。**

Day 14 和 Day 15 分別講了 `effort` 和 thinking 的完整機制，今天把它們合起來，變成一份排查清單。目標是：下次你或你的同事覺得「Claude 回答變淺了」，不用重新學一次原理，照著這份清單一項一項對，通常幾分鐘內就能找到原因。

先講結論：**九成的「變淺」問題，根源在 effort 或 thinking 設定被意外改動，而不是模型本身的能力變化。** 今天就是那份對照表。

> 本篇排查邏輯，於 **2026 年 8 月 14 日**對照 [Troubleshooting thinking](https://platform.claude.com/docs/en/build-with-claude/thinking-troubleshooting) 官方文件查證，症狀分類與修法均引用官方原文。

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

## 排查前先分清楚：「變淺」還是「變短」

這是動手查之前最重要的第一步，因為兩者的原因完全不同。

**變淺**：答案的推理深度不夠，遺漏了明顯該考慮的面向，感覺像是「隨便回一下」。這通常跟 effort 或 thinking 有關，是今天的主題。

**變短**：答案的推理沒問題，就是篇幅比你期待的短。Day 14 提過一個容易搞混的重點——**在 Opus 5 上，調整 effort 不保證能改變肉眼可見的回應長度**，effort 控制的是思考量,不是輸出的字數。如果你要的是「變短」而不是「變淺」，該做的是 Day 8 教的「明確要求輸出長度」，不是往下調 effort。

搞清楚是哪一種，才知道要往下查哪個方向。以下清單假設你要查的是「變淺」。

## 檢查清單第一項：effort 是不是被意外調低了

**最常見的原因，也是最該先查的一項。** 回頭檢查你的請求裡 `output_config.effort` 實際傳的值：

- **是不是分流邏輯（Day 27 的主題）誤判了任務難度**，把一個複雜任務路由到了 `low` 或 `medium`？
- **是不是繼承了別的請求的設定**，例如某個共用的請求模板原本是給簡單任務用的 `low`，卻被套用到了複雜任務上？
- **如果完全沒傳 `effort`**，預設是 `high`——這通常不是問題來源，除非你的模型或情境特別需要 `xhigh`。

官方在 Day 14 提過的排查方向是這句話：「**Claude is thinking too rarely or too shallowly: raise effort**」——遇到思考不夠深的狀況，第一個該調的旋鈕就是 effort，不是急著去改 prompt 或懷疑模型能力下降。

## 檢查清單第二項：thinking 有沒有被停用

第二步查 thinking 本身的設定：

- **你的請求裡是否明確傳了 `thinking: {"type": "disabled"}`？** 如果模型支援關閉（例如 Opus 5 在 effort `high` 以下時），這個設定會讓模型完全跳過推理階段直接回答。
- **在 Opus 5 上，如果你同時傳了 `disabled` 又把 effort 設到 `xhigh` 或 `max`，這個組合本身會被拒絕、回傳 400 錯誤**——如果你看到的不是「變淺」而是直接報錯，這就是原因，回頭調整其中一項。

## 檢查清單第三項：沒有 thinking 區塊，是正常現象還是異常

如果你發現某些回應完全沒有 `thinking` 內容區塊，先別急著當成 bug。官方明講：

> "This is normal in adaptive mode: Claude skips thinking on requests it judges simple enough to answer directly."

**在 adaptive thinking 下，模型自己判斷這次要不要思考，跳過思考是正常行為，不是異常。** 真正該檢查的是：這次真的是「模型判斷正確、這題確實不需要多想」，還是「模型誤判了這題的難度」？如果是後者，做法跟第一項一樣——調高 effort，或用 Day 15 提過的系統提示引導語句調整觸發門檻。

## 檢查清單第四項：`thinking` 欄位是空的，但不是沒思考

這是很容易誤判的一項——你打開回應，看到 `thinking` 區塊確實存在，但裡面的 `thinking` 文字是空字串，只有 `signature` 欄位有內容。**這不代表模型沒有思考，是 `display` 預設是 `"omitted"`，思考文字本來就不會回傳。**

如果你是因為「看不到思考過程」而誤以為「模型沒有深入思考」，這其實是可觀測性的問題，不是能力的問題。解法是 Day 15 提過的：

```python
thinking={"type": "adaptive", "display": "summarized"}
```

## 檢查清單第五項：System Prompt 裡是不是有話在暗示模型別想太多

這一項容易被忽略，因為問題不在參數，在你寫的文字裡。如果你的 system prompt 裡有類似「直接回答，不要解釋」「回答要精簡」這類指令，這些話**同時會影響 thinking 的觸發門檻**——不只是縮短可見輸出，也可能連帶讓模型更少啟動深度推理。

官方在 Opus 5 上還提到一個更具體的異常症狀：**如果 thinking 被停用，尤其在工具使用密集的情境（例如搜尋類任務），system prompt 裡「不要思考」「不要推理」這類規則反而會提高模型把工具呼叫或內部標籤洩漏到可見文字輸出裡的機率**——也就是說，你看到的不是單純「答案變淺」，而是輸出裡混進了不該出現的內容（例如殘留的 `<thinking>` 標籤或該被執行卻只是被寫成文字的工具呼叫）。官方給的建議是：**重新開啟 thinking（也就是預設狀態），改用較低的 effort 控制成本，而不是硬把 thinking 關掉。**

## 檢查清單第六項：`stop_reason` 是不是 `max_tokens`

如果答案讀起來像是「講到一半突然斷掉」，先查 `stop_reason` 欄位是不是 `max_tokens`。Day 5 提過這個欄位，這裡是它跟 thinking 的交集：**thinking 的 token 也計入 `max_tokens`**，如果模型這次思考得比較深，思考本身就可能把預算用完，留給正式回答的空間就被壓縮甚至歸零。

官方給的兩個修法方向：

- **調高 `max_tokens`**，讓思考和回答都有足夠空間。
- **調低 effort**，讓模型少想一點，把預算留給正式回答。

哪一個對，取決於這次「被截斷的內容」到底需不需要那麼多思考——如果需要，調高上限；如果是想太多，調低 effort。

## 檢查清單第七項：快取命中率突然下降，是不是連帶影響了什麼

這一項不是直接造成「變淺」，但常常跟前面幾項一起出現，值得順手檢查。Day 9、Day 14 都提過：**改變 thinking 設定或 effort 值，會讓對話快取失效**。如果你的排查過程中動了這兩個設定去做 A/B 測試，記得這個動作本身會讓 `cache_read_input_tokens` 掉回 0——這不是「變淺」的原因，但很容易在你比對前後測試結果時造成誤判（例如誤以為某次測試比較貴是因為思考變多了，其實只是快取沒命中）。

## 一個真實案例：三個原因疊在一起

實務上「變淺」很少只有單一原因，通常是好幾項疊加。分享一個排查過程，方便你對照自己的情境：

某個客服摘要功能，團隊反映「這週的摘要品質明顯比上週差」。照著清單查下去：

**第一項就中了**——分流邏輯（Day 27 的主題）最近改過，把「客訴」類的工單也一併路由到了原本只給「一般諮詢」用的 `low` effort 設定，因為兩種工單被合併進同一個分類規則。**第五項也中了**——同一次改動裡，system prompt 加了一句「回答要精簡，不要長篇大論」，這句話同時抑制了 thinking 的觸發門檻。

單獨看，`low` effort 或那句系統提示都不是不合理的設定，問題是**兩者疊加在同一批本來就需要多想一點的客訴工單上**，效果被放大了。修法是把客訴類工單的 effort 調回 `medium`，並把系統提示改成「回答要精簡，但遇到複雜情況仍要完整說明」——精簡跟深度不衝突，只是原本的用詞把兩件事綁在了一起。

這個案例想說明的是：**照清單查，不要查到第一項有問題就停下來**，很多時候真正的原因是好幾個小設定疊加的結果。

## Before / After：把排查邏輯寫成程式碼

**❌ Before：完全靠感覺猜原因**

```python
# 「Claude 最近變笨了」——直接換更貴的模型試試看
response = client.messages.create(
    model="claude-fable-5",   # 沒查原因，先換模型再說
    max_tokens=4096,
    messages=[{"role": "user", "content": user_question}],
)
```

**✅ After：照清單依序排查**

```python
def diagnose_shallow_response(request_config):
    checks = []

    effort = request_config.get("output_config", {}).get("effort", "high")
    if effort in ("low", "medium"):
        checks.append(f"effort 目前是 {effort}，若任務偏複雜，先試著調高")

    thinking = request_config.get("thinking", {})
    if thinking.get("type") == "disabled":
        checks.append("thinking 被明確停用，確認是否為刻意設計")

    if thinking.get("display", "omitted") == "omitted":
        checks.append("display 是 omitted，看不到思考文字不代表沒有思考")

    system = request_config.get("system", "")
    if any(kw in system for kw in ["不要解釋", "直接回答", "不要思考", "不要推理"]):
        checks.append("system prompt 可能在抑制 thinking 觸發，檢查用詞是否過於強硬")

    return checks

# 用實際的請求設定跑一次，先找出可疑項，而不是直接換模型
issues = diagnose_shallow_response(my_request_config)
print(issues)
```

> Before 版本的問題是：換模型可能真的會讓答案「看起來」變好，但你永遠不知道原因是什麼，下次同樣的狀況還會再發生。After 版本把 Day 14、Day 15 學到的規則變成一份可以重複使用的檢查清單——大部分「變淺」的案例，答案就藏在這五、六個檢查點裡，不需要真的换一顆更貴的模型。

## 本篇自我挑戰

- **今日挑戰**：如果你手上有任何「這個 prompt 最近感覺變弱了」的案例，照著今天的清單，從 effort 開始依序查一遍，記錄下真正的原因是哪一項。

- **反思**：「Claude 變笨了」這句話背後，其實預設了一個沒有根據的假設——模型的能力會無故退化。但更常見的情況是**我們自己的請求設定變了，而我們沒意識到**。你有沒有在其他系統上也做過類似「先怪工具、後查自己設定」的反射動作？

## 總結

今天沒有新的技術原理，是把 Day 14 和 Day 15 的知識整理成一份可執行的排查清單：**先分清楚是「變淺」還是「變短」**，兩者的解法完全不同；**依序檢查 effort、thinking 是否停用、有沒有 thinking 區塊、`display` 設定、system prompt 用詞、`stop_reason`**，多數案例的原因就藏在這幾項裡。

最後一個提醒：排查過程本身如果動了 effort 或 thinking 設定去做測試，記得這會讓快取失效——不要把「這次比較貴」誤判成「思考變多了」，先確認是不是快取沒命中。

**本日關鍵字回顧**

- **變淺 vs 變短**：前者是推理深度不足（查 effort/thinking），後者是可見輸出篇幅問題（查 prompt 指令），原因與解法不同。
- **Thinking 跳過是正常行為**：adaptive 模式下，模型判斷簡單問題會直接跳過思考，不代表故障。
- **`display: "omitted"`**：思考文字預設不回傳，看不到不代表沒思考，計費不受影響。
- **System prompt 抑制效應**：過度強硬的「不要思考」類指令，可能連帶提高 Opus 5 工具呼叫洩漏到文字輸出的風險。
- **`stop_reason: "max_tokens"`**：thinking 佔用預算導致回答被截斷的常見症狀，修法是調高上限或調低 effort。

明天進入第三階段的下一個主題——把過去兩天學到的 effort 與 thinking 放一邊，回到最根本的問題：**一份好的 prompt，骨架長什麼樣子？**

**Day 17，官方最佳實踐的 prompt 骨架。**
