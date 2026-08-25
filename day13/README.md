# 【Day 13】Claude 用量怎麼監控？成本失控前的預警機制

## 聯繫我

如果有任何問題或建議，歡迎隨時聯繫我：

- [Email](mailto:z0925955648@gmail.com)

## 前言

第二階段走到這裡，我們講了計價結構（Day 2）、tokenizer 原理（Day 7、11）、快取與批次（Day 9、10）、長對話的成本累積（Day 12）。這些都是「怎麼讓每一筆花費更划算」的技巧。

今天要處理一個不一樣的問題：**你怎麼知道自己正在花多少錢、還剩多少配額、什麼時候會被擋下來？**

Day 6 結尾埋過一句話：「帳單超出預期時，先確認錢花在哪。」今天就是兌現這句話——把 Claude 的用量管理，從「月底看帳單嚇一跳」，變成一個你能主動監控、事先設好防線的系統。

> 本篇關於用量層級、費用上限、rate limit 標頭的規格，於 **2026 年 8 月 14 日**對照 [Rate limits 官方文件](https://platform.claude.com/docs/en/api/rate-limits) 查證。

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

## 一、兩種上限，先搞清楚差在哪

官方文件把限制分成兩種，很多人會混著講，但它們管的是不同的事：

- **Spend limits（費用上限）**：組織每個月最多能在 API 上花多少錢。
- **Rate limits（速率限制）**：組織在特定時間內最多能發出多少請求。

**費用上限**依你的用量層級（usage tier）而定：

| 用量層級 | 每月費用上限 |
| :--- | :--- |
| Start | $500 美元 |
| Build | $1,000 美元 |
| Scale | $200,000 美元 |
| Custom | 無固定上限，與帳號團隊另行約定 |

超過這個上限，API 用量會**暫停到下個月**，除非你申請調高限制。你也可以在自己的層級上限之下，**自訂更低的費用上限**來做更保守的控制——這個設定在 Console 的 Billing 頁面。

## 二、用量層級：新帳號從哪裡開始、怎麼升級

官方講得很明白：**層級是自動判定的**，根據你的用量歷史與帳號狀況，隨著你持續使用 API，會自動往更高的層級移動。

**新帳號或用量歷史還很少的組織，會先落在 Evaluation 層級**，限制比 Start/Build/Scale 這幾個標準層級都低——這是官方用來防止濫用與詐騙的機制，隨著帳號累積使用歷史會自動提升。

如果標準層級的限制不夠用，可以在 Console 的 Rate limits 頁面申請調高；急件也可以直接聯繫官方支援。

## 三、Rate limits 的三個計量單位：RPM、ITPM、OTPM

Messages API 的速率限制用三個指標衡量，**依模型類別分別計算**：

- **RPM**：每分鐘請求數
- **ITPM**：每分鐘輸入 token 數
- **OTPM**：每分鐘輸出 token 數

超過任何一項，會收到 **429 錯誤**，回應裡的 `retry-after` 標頭會告訴你該等多久再重試。

**有一個對快取使用者非常有利的規則，很容易被忽略：多數模型的 ITPM，只計算「未命中快取」的 input token。**

官方原文：

> "For most Claude models, only uncached input tokens count toward your ITPM rate limits."

具體拆解各欄位是否計入 ITPM：

| Usage 欄位 | 是否計入 ITPM |
| :--- | :---: |
| `input_tokens`（最後一個快取斷點之後的內容） | ✓ 計入 |
| `cache_creation_input_tokens`（寫入快取） | ✓ 計入 |
| `cache_read_input_tokens`（讀取快取） | ✗ **不計入**（多數模型） |

官方舉的例子很直白：**如果你的 ITPM 上限是 200 萬，快取命中率 80%，你實際上每分鐘能處理的總 input token 量可以到 1000 萬**（200 萬未快取 + 800 萬走快取，快取的部分不佔用額度）。這代表 **Day 9 的快取策略，不只省錢，還能實質拉高你的有效吞吐量**——不用申請更高的 rate limit，光靠提高快取命中率就能達到類似效果。

（有個例外：**Claude Haiku 3.5**（已退役機型，僅 Bedrock / Google Cloud 保留）的 `cache_read_input_tokens` **會**計入 ITPM，這是唯一的特例。）

**OTPM 則是即時依實際產生的輸出 token 計算**，`max_tokens` 這個參數本身不影響 OTPM 額度計算——也就是說，**把 `max_tokens` 設高一點，不會對你的 rate limit 造成任何額外負擔**，這也呼應了 Day 5 提過的：`max_tokens` 應該當保險絲用，不用怕設太高。

## 四、標準層級的實際數字（節錄）

以 Opus 5 為例，三個標準層級的差異：

| 層級 | RPM | ITPM | OTPM |
| :--- | :---: | :---: | :---: |
| Start | 1,000 | 2,000,000 | 400,000 |
| Build | 5,000 | 5,000,000 | 1,000,000 |
| Scale | 10,000 | 10,000,000 | 2,000,000 |

（完整的各模型對照表請查官方 Rate limits 頁面，每個模型類別的數字不同，且部分舊世代模型的額度是跨版本共用的合併額度。）

**Batch API 的速率限制是獨立的一組**，跟 Messages API 分開計算——用批次不會侵蝕你即時呼叫的額度，反之亦然。這也是 Day 10 沒展開的一個細節：批次除了折扣，還有自己專屬的吞吐量配額。

## 五、怎麼監控：Console 的 Usage 頁面與回應標頭

**① Console 的 Usage 頁面**：提供兩張獨立的速率限制圖表——Input Tokens 和 Output Tokens——各自顯示每小時最高用量、目前的額度上限，以及（針對 input）**快取命中率**。官方明講這個頁面就是拿來「看你還有多少成長空間、什麼時候接近尖峰、該申請多高的額度、以及怎麼調整快取策略」的工具。

**② 每次 API 回應都帶著即時額度資訊**，透過標頭回傳：

| 標頭 | 說明 |
| :--- | :--- |
| `retry-after` | 需要等待幾秒才能重試 |
| `anthropic-ratelimit-requests-remaining` | 距離觸發限制前，還剩多少請求次數 |
| `anthropic-ratelimit-input-tokens-remaining` | 還剩多少 input token 額度 |
| `anthropic-ratelimit-output-tokens-remaining` | 還剩多少 output token 額度 |
| `anthropic-ratelimit-*-reset` | 額度何時完全恢復（RFC 3339 格式時間） |

這些標頭在**每一次**呼叫都會回傳，代表你可以把「檢查剩餘額度」直接內建在你的應用程式邏輯裡，而不需要另外呼叫一支查詢 API：

```python
response = client.messages.with_raw_response.create(
    model="claude-opus-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
)

remaining_input = response.headers.get("anthropic-ratelimit-input-tokens-remaining")
remaining_output = response.headers.get("anthropic-ratelimit-output-tokens-remaining")

if int(remaining_input) < 100_000:
    alert_team("Input token 額度即將見底")
```

**③ Rate Limits API**：如果你想用程式化的方式讀取組織與各 Workspace 目前設定的限制（而不是等回應標頭），官方也提供了專門的端點可以查詢。

## 六、Before / After：從「月底才知道」到「即時預警」

**❌ Before：完全不監控，等帳單信來才知道**

```python
def call_claude(prompt):
    return client.messages.create(
        model="claude-opus-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}],
    )
# 沒有任何地方在追蹤用量，直到月底收到帳單才知道花了多少
```

**✅ After：把回應標頭接進監控，設好預警門檻**

```python
def call_claude(prompt):
    response = client.messages.with_raw_response.create(
        model="claude-opus-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}],
    )

    remaining = int(response.headers.get("anthropic-ratelimit-input-tokens-remaining", 0))
    limit = int(response.headers.get("anthropic-ratelimit-input-tokens-limit", 1))

    if remaining / limit < 0.2:   # 剩餘額度低於 20%
        send_alert(f"ITPM 額度僅剩 {remaining}/{limit}")

    return response.parse()
```

> Before 版本的問題不是「一定會出事」——多數時候一切正常。**它的問題是出事的時候你毫無準備**，可能是流量突然暴增撞上 rate limit，也可能是某個功能悄悄把成本推高，等你發現時已經是下個月的帳單。After 版本只是把官方本來就免費提供的標頭資訊，接進你自己的告警系統——不需要額外的 API 呼叫，純粹是把已經有的資料用起來。

## 七、把三道防線串起來：一份可以照抄的設定清單

前面六節分別講了費用上限、用量層級、rate limit 的三個指標、以及監控管道。實務上這些東西該怎麼一次設好，而不是東拼西湊？給一份可以照抄的順序：

**第一步，設自訂費用上限，比你的層級上限低一截。** 如果你的組織在 Build 層級（上限 $1,000），不要等到真的花滿 $1,000 才有感覺——在 Billing 頁面把自訂上限設在你能接受的「意外情境最大損失」，例如 $600，留一段緩衝讓你有時間反應。

**第二步，決定你的預警門檻，而不是只設「到頂才擋」的硬上限。** Rate limit 的硬上限（撞到才會收到 429）跟你想要的「提早收到警訊」是兩回事。用第五節示範的程式碼邏輯，在剩餘額度低於某個比例（例如 20%）時就先發告警，而不是等真的被 429 擋下來才知道。

**第三步，把快取命中率排進你定期會看的指標。** 因為 Day 9 講過快取命中不太計入 ITPM，命中率下降不只讓你多花錢，還會讓你更容易撞到速率限制。Console 的 Usage 頁面本身就有這張圖，養成習慣定期看一次，比事後回頭查原因有效率得多。

**第四步，區分「即時呼叫」與「批次呼叫」的監控。** Day 10 提過批次有自己獨立的速率限制，如果你的系統同時有即時互動與批次任務，兩邊的額度不會互相排擠，但也代表**你需要分開監控**，不能只看一個總量儀表板就以為涵蓋了全部。

這四步沒有一步需要額外的付費工具——全部建立在官方免費提供的標頭、Console 頁面與費用設定介面上，差別只在於「有沒有花時間把它們串起來」。

## 本篇自我挑戰

- **今日挑戰**：找一支你正在使用的 Claude API 呼叫，加上讀取 `anthropic-ratelimit-input-tokens-remaining` 的邏輯，印出目前剩餘額度的百分比。如果你是網頁版使用者，去 Console 的 Usage 頁面看一次快取命中率圖表——這是你目前完全沒注意過的指標。

- **反思**：多數人管理成本的方式是「先做，事後看帳單反省」。今天講的機制讓你可以「邊做邊看」，甚至「做之前先防」。你在其他系統（雲端資源、訂閱服務）上，有沒有類似「應該設預警但一直沒設」的地方？是什麼原因讓你一直拖著沒做？

## 總結

用量監控不是為了在出事之後追查原因，是為了**在出事之前就先知道**。今天釐清了三層防線：**費用上限**（Spend limits）是最後一道硬防線，超過就直接暫停；**速率限制**（RPM/ITPM/OTPM）是短期的流量閘門，撞到會收到 429 但有 `retry-after` 可以重試；**回應標頭與 Console 儀表板**則是讓你在撞到限制之前，就能主動觀察趨勢。

今天最值得記住的一個細節：**快取命中的 token 多數不計入 ITPM**——這代表 Day 9 學到的快取策略，除了省錢，還在幫你的有效吞吐量默默加分。把這件事跟 Rate Limits 一起看，你會發現前面幾天的技巧其實環環相扣，不是七個獨立的招式。

**本日關鍵字回顧**

- **Spend limits vs Rate limits**：前者是每月最高花費，後者是單位時間內的最高請求／token 數。
- **Evaluation 層級**：新組織或用量歷史不足時的起始層級，額度低於標準的 Start/Build/Scale，會隨用量歷史自動提升。
- **RPM / ITPM / OTPM**：速率限制的三個獨立計量單位，依模型類別分別計算。
- **Cache-aware ITPM**：多數模型的快取命中 token（`cache_read_input_tokens`）不計入 ITPM 額度，是拉高有效吞吐量的關鍵細節。
- **`anthropic-ratelimit-*` 標頭**：每次 API 回應都附帶的即時額度資訊，可直接用於自建監控告警。

第二階段到這裡告一段落。七天下來，我們把「省 token」從一句口號拆成了七個可操作的維度：token 本身怎麼算、五個立即可用的習慣、快取、批次、tokenizer 換代、長對話陷阱、以及今天的監控機制。

明天進入第三階段——**設定與調校**。第一站是那個貫穿全系列、卻始終沒有完整展開的參數：`effort`。五個檔位到底該怎麼選，什麼時候該調、什麼時候不該調。

**Day 14，effort 參數完整攻略。**
