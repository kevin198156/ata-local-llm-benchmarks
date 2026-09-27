# Phase 0 Public Report — GTX 1660 Ti 6GB Local LLM Evaluation

日期：2026-09-24 ～ 2026-09-27  
狀態：**CLOSED**

這份文件是私人開發紀錄的 sanitized 公開版。原始私人／成人 UAT、完整聊天紀錄與本機私密資料不公開。

## 1. 問題不是「能不能跑」，而是「值不值得每天用」

硬體：

- Windows 11
- Intel Core i5-12400F
- NVIDIA GTX 1660 Ti 6GB
- 32GB RAM

主要 runtime：

- llama.cpp b11149
- 0.5.0-dev
- commit `d2e54583c`
- CUDA portable build
- localhost OpenAI-compatible API

Phase 0 的目標不是追求最高 benchmark，而是找出符合以下真實需求的模型：

- 台灣繁體中文自然且長聊不容易漂移
- 日常 Companion 有情感價值，不一直變人生導師
- fiction / RP 能維持角色控制權與指令
- 成人向工作流要看 actual generation，而不是模型自述
- 能處理 consent / character state
- 不知道的資訊要敢說不知道
- 速度與 RAM/VRAM 壓力要能接受
- 模型可替換，不把未來 Persona / Memory / History 綁死

## 2. 基準：8B 在 6GB VRAM 上很好跑，但品質才是真正 Gate

最早以 Qwen3-8B Generic 與 Qwen3-8B Abliterated Q4_K_M 做實機基準。

代表設定：

- context 4096
- 1 slot
- 6 CPU threads
- Q8_0 K/V cache
- flash attention on
- batch 512 / ubatch 128
- 約 26 層 GPU offload
- background desktop applications 保持運作

兩顆都完成 25 次請求，沒有 OOM、crash 或 HTTP failure。

| 指標 | Qwen3-8B Generic | Qwen3-8B Abliterated |
|---|---:|---:|
| 短冷 TTFT | 0.759 s | 0.772 s |
| 約 2970 tokens 冷 TTFT | 27.871 s | 27.909 s |
| 短熱 decode | 18.32 tok/s | 18.58 tok/s |
| 長熱 decode | 13.03 tok/s | 13.29 tok/s |
| 12 輪 decode | 17.89 tok/s | 17.27 tok/s |

技術面很漂亮，但 Human UAT 很快暴露出繁簡混用、補不存在的上下文、角色互動品質與 ownership 等問題。這成為整個專案第一個重要轉折：

> **技術跑得快，不代表是想要的 Companion。**

## 3. 8B → 14B：容量變大，不一定換來更好的日常體感

### Stheno 8B

`Sao10K/L3-8B-Stheno-v3.3-32K` Q4_K_M 的 4K minimal smoke：

- ready 約 3.05 s
- short cold TTFT 0.905 s
- decode 約 22.8 tok/s
- 26/33 layers offloaded
- VRAM ready 約 5.5GB
- technical smoke PASS

這證明 8B 級模型可以在 1660 Ti 上跑得相當輕快。不過 technical PASS 不是 Human UAT PASS；本專案後續沒有把它選為 finalist。其 CC-BY-NC-4.0 標示也意味著公開分享時必須注意非商業授權條件。

### qwen2.5-14B-roleplay-zh

14B Q4_K_M 採 20/49 layers hybrid：

| Prompt | TTFT | Decode |
|---|---:|---:|
| 短 | 1.57 s | 7.13 tok/s |
| 約 1K | 16.97 s | 6.11 tok/s |
| 約 3K | 53.80 s | 5.33 tok/s |

VRAM peak 約 4.94GB，但 RAM ready 約 21.4GB。沒有 OOM；問題是 context 變長後第一個 token 等待非常明顯。

這一輪讓「大一點是不是一定更值得」開始動搖。

### Spoomples Qwen3-14B v0.2

技術上約 6.8 tok/s。測試也抓到一個很實務的問題：同一模型在不同 system prompt / reasoning 設定下，可能從繁中正常回答變成英文拒答。縮短 system prompt 後可恢復繁中。

這是另一個重要提醒：

> **Template、reasoning parser、system prompt 與 runtime integration 會影響你以為的「模型品質」。**

## 4. 9B 成為真正的甜蜜點

### Defiant Fable 9B

Qwen3.5-9B Defiant Fable Q4_K_M 在既有 llama.cpp 上通過 compatibility 與 technical smoke。

早期 4K smoke：

- 12/33 layers offloaded
- visible decode 約 10.4–10.8 tok/s
- Qwen3.5 native template
- `enable_thinking=false` 後回答正常落在 visible content

後續 8K Human UAT 成為主力測試入口。

最後 Human UAT：

- Traditional Chinese：**PASS**，偶發簡體字 leakage 為 WARN
- Companion naturalness / emotional value：**STRONG**
- Adult actual generation：**PASS**
- Source-conditioned fiction：**PASS/WARN**
- RP ownership：**PASS**
- Clean consent / character-state tracking：**PASS**
- RP → normal chat mode recovery：**PASS**
- Unknown-info / hallucination control：**PASS**
- Long-session / mode switching：**PASS**
- 日常速度：約 **9 tok/s**

主要 warning：

- 偶爾過度分析／給建議
- source-conditioned writing 偶爾會新增人物、細節、心理或重貼原文
- 少量簡體字 leakage

最終定位：

> **Main Companion Champion**

### 27B HauhauCS：技術上可跑，不代表適合日常

Qwen3.5-27B Q4_K_M 做過實機嘗試。

14/65 layers offload 的 launcher smoke：

- ready 約 9.7 s
- short Traditional Chinese generation 可成功
- TTFT 5.62 s
- decode 約 **2.73 tok/s**
- VRAM peak 約 5.52 / 6.14GB
- 可用 RAM 最低只有約 **796 MiB**

沒有 OOM 或 crash，但 RAM reserve 明顯不足，因此 Human A/B 不開放，daily usability 判定 FAIL。

這一輪提供了最直接的答案：

> 27B 在這台機器上「可以啟動」，但對這個日常 Companion 需求不划算。

## 5. Creative / RP challengers：會寫，和會服從，是不同事

### DARK ROAST 9B

Technical smoke 與 8K launcher 均 PASS：

- 12/34 layers
- launcher decode 約 10.9 tok/s
- 繁中 visible output
- quick unknown-info / ownership check 正常

完整私人成人 UAT 原文不公開；本公開報告不把未收錄於 sanitized source 的細節補成結論。它沒有取代最終雙冠軍。

### Astrea R8 Chat 9B

Technical：

- 12/33 layers
- 4K Traditional Chinese gate PASS
- 約 9–11 tok/s

Human UAT：

- 容易自行新增背景、人物、象徵
- style continuation / instruction obedience 不夠精準
- Companion 感沒有明顯勝過 Defiant
- 偶有簡體字

結論：沒有取代 Defiant。

### Veiled Calla 12B

8K launcher：

- 12/49 layers
- decode 約 5.86 tok/s
- VRAM peak 約 4.29GB
- RAM 壓力明顯高於 9B

Human UAT sanitized conclusion：

- Companion naturalness：WARN / 偏弱
- Adult Freedom：部分 PASS
- Explicitness：FAIL
- Adult Writing：WARN
- Adult RP：FAIL
- Consent / character-state tracking：FAIL
- Instruction following：FAIL
- Creative-writing obedience：WARN
- 約 5.2–5.9 tok/s

結論：沒有取代 Defiant。

### Rocinante-X 12B

沒有下載或測試。

原因不是效能，而是衍生權重授權資訊不足。基礎模型的 Apache-2.0 不足以替衍生模型補上明確授權，因此標記 **SOURCE BLOCKED**。

這也是 Phase 0 的一個實務原則：

> **來源與 license gate 也是模型選拔的一部分。**

### Claude 4.6 HighIQ Heretic 9B

Technical / Traditional Chinese / short RP ownership 可用，8K launcher visible decode 約 8.9 tok/s。

Human UAT：

- Traditional Chinese：PASS
- Technical usability：PASS
- Short RP ownership：PASS
- Creative-writing obedience：WARN
- Adult actual generation：FAIL
- Explicit vocabulary：FAIL

結論：名稱帶有 Uncensored / Heretic 也不代表符合 actual-generation 需求。

### Deckard V8 Pro Writer 9B

IQ4_XS technical PASS：

- 12/33 layers
- 約 9.5–11.8 tok/s
- RP ownership short check PASS

但 Traditional Chinese A/B/C 出現明顯簡體字 leakage 與英文短語，因此繁中 hard gate FAIL。後續成人／長聊測試直接停止。

結論：

> **Hard gate 失敗就停止，不為了「也許後面很好」浪費測試成本。**

## 6. 最終補測：Defiant vs Holodeck

到了最後，Human UAT protocol 已經比專案起點完整很多，因此 finalists 重新接受同一批核心測試，而不是沿用早期不同考卷。

### Defiant Fable 9B

最終維持：

> **Main Companion Champion**

理由不是它每個單項都最強，而是日常 Companion 所需的綜合可靠度較好：繁中、長聊、ownership、clean consent/state、模式恢復與 unknown-info control 都通過。

### Holodeck Lounge 9B

4K technical：

- 12/33 layers
- load-to-ready 4.11 s
- Chinese / RP / writing decode 約 9–9.5 tok/s

8K launcher：

- load-to-ready 3.09 s
- decode 11.15 tok/s（兩個短 smoke）
- VRAM ready/peak 約 3.21GB

Final Human UAT：

- Traditional Chinese：**PASS**
- Adult actual generation：**PASS**
- Explicitness：**PASS**
- RP ownership：**PASS**
- Clean consent / character-state tracking：**PASS**
- Mode recovery：**PASS**
- Source-conditioned adult / fiction workflow：Human satisfaction **HIGH**
- Writing obedience：**WARN**
- High-intensity immersed RP consent robustness：**WARN**
- Unknown-info / hallucination control：**FAIL**

Unknown-info 測試中特別有價值的一點：Holodeck 能對部分虛構項目保持保留，但也曾對一個不存在的遊戲自信補出開發商、年份、類型與玩法；另一個虛構作品測試則補出未支持的作者背景。

這讓它的定位非常清楚：

> **在 fiction 裡，想像力是優點；在 factual QA 裡，同一傾向可能變成 hallucination。**

因此最終定位：

> **Adult / Fiction Specialist Champion**

## 7. Final Championship

| Role | Champion | 為什麼 |
|---|---|---|
| Main Companion | **Defiant Fable 9B** | 綜合日常陪伴、繁中、ownership、clean consent/state、mode recovery、unknown-info control、長聊較可靠 |
| Adult / Fiction Specialist | **Holodeck Lounge 9B** | source-conditioned fiction/成人小說工作流的 Human UAT 滿意度最高，但 factual hallucination 與高強度 consent robustness 不適合當唯一通用助手 |

沒有選「唯一總冠軍」。

這不是逃避排名，而是承認不同工作流需要不同能力。

## 8. GTX 1660 Ti 6GB 的實際答案

這次測試的結論不是「1660 Ti 可以跑所有模型」，而是：

- 8B 很輕快，技術速度甚至可超過 17–22 tok/s；品質仍需 Human UAT。
- 12B / 14B 可以 hybrid 跑，但速度與 context TTFT 開始明顯下降。
- 27B technically feasible，但 RAM reserve 與約 2.7 tok/s 不適合作為此專案日常方案。
- **9B Q4 在這台 GTX 1660 Ti 6GB + 32GB RAM 上，形成最佳品質／速度／日用性折衷。**

這是這台機器、這組 runtime、這個使用需求的結論，不是普遍硬體定律。

## 9. Caveats

- 不同模型的 GPU layers、quant、template 與 context 不完全相同，效能數字不是嚴格 apples-to-apples leaderboard。
- 部分數據樣本很小，主要目的為個人 daily-use gate。
- 背景桌面程式沒有全部關閉，不是獨占 GPU 的實驗室測量。
- Human UAT 有主觀成分，而且 protocol 在過程中演進。
- 因此 finalists 最後重新補測，而不是直接比較不同時期、不同題目的結果。
- Adult/consent 測試結果不等於任何形式的安全保證。
- Raw private/adult transcript 不公開，因此外部讀者能重現的是 protocol，不是逐字對話。

## 10. Next

Phase 0 到此結案。後續私人專案優先處理 Story Continuity / Human-gated memory，再評估 Hermes、Telegram 與 Web Search。

這個公開 Repo 則專注保留 sanitized benchmark、Human UAT 方法與實裝心得。
