# Model Results — Sanitized Summary

> 這不是 leaderboard。不同候選使用不同 quant、GPU offload、context 與測試階段；表格的用途是保留「為什麼繼續／停止」的決策證據。

## Summary

| Candidate | Size / Quant | Representative local result | Human / quality result | Final status |
|---|---|---|---|---|
| Qwen3-8B Generic | 8B / Q4_K_M | 12-turn decode ~17.9 tok/s | 繁簡混用、曾補不存在的過往脈絡 | Baseline only |
| Qwen3-8B Abliterated | 8B / Q4_K_M | 12-turn decode ~17.3 tok/s | Technical usable；後續 Human UAT 未符合核心需求 | Rejected |
| Stheno v3.3 32K | 8B / Q4_K_M | 4K smoke ~22.8 tok/s | Technical PASS；未成為 finalist | Not selected |
| qwen2.5-14B-roleplay-zh | 14B / Q4_K_M | ~7.1 → 5.3 tok/s；3K TTFT ~54s | Speed / latency WARN | Not selected |
| Spoomples Qwen3-14B v0.2 | 14B / Q4_K_M | ~6.8 tok/s | Prompt / launcher integration 對語言行為有明顯影響 | Not selected |
| **Defiant Fable** | **9B / Q4_K_M** | 日常約 ~9 tok/s | 繁中、Companion、ownership、clean consent/state、mode recovery、unknown-info、長聊通過 | **Main Companion Champion** |
| HauhauCS Aggressive | 27B / Q4_K_M | ~2.73 tok/s；最低 available RAM ~796 MiB | 技術可啟動，但 daily usability 不合格 | Experimental / not daily |
| DARK ROAST | 9B / Q4_K_M | 8K launcher ~10.9 tok/s | Technical / UAT-ready；未取代 final champions | Not selected |
| Astrea R8 Chat | 9B / Q4_K_M | ~9–11 tok/s | Human UAT：authorial takeover / instruction obedience 問題 | Rejected as champion |
| Veiled Calla | 12B / Q4_K_M | ~5.2–5.9 tok/s | Companion 弱、explicitness FAIL、adult RP / consent-state FAIL | Rejected |
| Rocinante-X v1 | 12B | Not downloaded | 衍生權重授權資訊不足 | SOURCE BLOCKED |
| Claude 4.6 HighIQ Heretic | 9B / Q4_K_M | 8K visible decode ~8.9 tok/s | 繁中 PASS；adult actual generation / explicit vocabulary FAIL | Rejected |
| Deckard V8 Pro Writer | 9B / IQ4_XS | ~9.5–11.8 tok/s | Traditional Chinese hard gate FAIL | Rejected |
| **Holodeck Lounge** | **9B / Q4_K_M** | ~9–11 tok/s | fiction workflow 強；clean consent PASS；hallucination FAIL；high-intensity consent robustness WARN | **Adult / Fiction Specialist Champion** |

## Notes by candidate

### Qwen3-8B Generic / Abliterated

兩顆都是最早的硬體基準。它們證明 GTX 1660 Ti 6GB 可以有效 CUDA offload，且 8B Q4 有舒服的生成速度。

但 Human UAT 很快證明：

- Generic / Abliterated 的「跑得動」沒有解決自然繁中、情境合理性與 ownership。
- Abliterated 標籤本身沒有提供「更適合這個使用者」的保證。

### Stheno 8B

[Source model](https://huggingface.co/Sao10K/L3-8B-Stheno-v3.3-32K)  
GGUF used from mradermacher conversion.

- technical smoke PASS
- 4K short decode ~22.8 tok/s
- 26/33 layers offload
- CC-BY-NC-4.0

這是一個「速度很好，但 technical pass 不等於 final fit」的例子。

### qwen2.5-14B-roleplay-zh

[GGUF source](https://huggingface.co/TouchNight/qwen2.5-14B-roleplay-zh-Q4_K_M-GGUF)

20-layer hybrid：

- short ~7.13 tok/s
- ~1K context ~6.11 tok/s
- ~3K context ~5.33 tok/s
- 3K TTFT ~53.8s

使用者後來接受「5–7 tok/s 不是絕對不能用」，但這顆沒有在品質／長 session 上成為最後方案。

### Spoomples Qwen3-14B v0.2

[GGUF source](https://huggingface.co/mradermacher/spoomples-qwen3-14b-v0.2-GGUF)

最有價值的不是排名，而是 integration bug lesson：

- 某版 launcher system prompt 讓它輸出英文拒答；
- 換成更短的繁中 system prompt 後恢復繁中；
- technical smoke ~6.8 tok/s。

因此模型評測必須先排除 template / parser / prompt integration 問題。

### Defiant Fable 9B

[GGUF source](https://huggingface.co/DavidAU/Qwen3.5-9B-The-Defiant-Fable-Uncensored-Heretic-NEO-IMATRIX-MAX-MTP-GGUF)

Final Human UAT：

| Dimension | Result |
|---|---|
| Traditional Chinese | PASS / occasional Simplified leakage WARN |
| Companion naturalness | STRONG |
| Emotional value | STRONG |
| Adult actual generation | PASS |
| Source-conditioned fiction | PASS/WARN |
| Writing obedience | WARN |
| RP ownership | PASS |
| Clean consent / state tracking | PASS |
| Mode recovery | PASS |
| Unknown-info control | PASS |
| Long-session / mode switching | PASS |
| Daily speed | ~9 tok/s |

Final role：**Main Companion Champion**。

### HauhauCS Qwen3.5-27B Aggressive

[Model source](https://huggingface.co/HauhauCS/Qwen3.5-27B-Uncensored-HauhauCS-Aggressive)

- Q4_K_M ~16.5GB file
- 14/65 layers offload
- ~2.73 tok/s
- VRAM peak ~5.52GB
- available RAM bottomed near ~796 MiB

No OOM，但日常使用的 RAM 安全餘裕與生成速度不合格。

### DARK ROAST 9B

[GGUF source](https://huggingface.co/mradermacher/Qwen3.5-9B-The-Defiant-Fable-DARK-ROAST-Uncensored-Heretic-NEO-IMATRIX-MAX-MTP-GGUF)

- technical smoke PASS
- 8K launcher decode ~10.9 tok/s
- quick Traditional Chinese / unknown-info / ownership checks usable

完整私人 UAT 不以 raw transcript 公開；在可公開的 Source of Truth 中，它沒有成為 final champion。

### Astrea R8 Chat 9B

[Model source](https://huggingface.co/Altworld/Astrea-R8-Chat-9B)

Technical Chinese gate PASS；Human UAT 的主要問題是：

- 容易自行新增背景、人物、象徵；
- style continuation / instruction obedience 不夠精準；
- Companion 沒有明顯勝過 Defiant；
- 偶有簡體 leakage。

### Veiled Calla 12B

[Model source](https://huggingface.co/soob3123/Veiled-Calla-12B)

Human UAT sanitized result：

| Dimension | Result |
|---|---|
| Companion naturalness | WARN / weak |
| Adult Freedom | partial PASS |
| Explicitness | FAIL |
| Adult Writing | WARN |
| Adult RP | FAIL |
| Consent / character-state | FAIL |
| Instruction following | FAIL |
| Creative-writing obedience | WARN |
| Speed | ~5.2–5.9 tok/s |

### Rocinante-X 12B v1

沒有下載。

模型／GGUF 頁面沒有提供足以確認衍生權重授權的資訊；base model 的 license 不自動等於 derivative license。

Result：**SOURCE BLOCKED**。

### Claude 4.6 HighIQ Heretic 9B

Human UAT：

- Traditional Chinese PASS
- technical usability PASS
- short RP ownership PASS
- creative-writing obedience WARN
- adult actual generation FAIL
- explicit vocabulary FAIL

再次證明 model naming / “uncensored” 宣稱不取代 actual generation test。

### Deckard V8 Pro Writer 9B

[Model source](https://huggingface.co/DavidAU/Qwen3.5-9B-The-Deckard-V8-Pro-Writer-Uncensored-Heretic)

Technical performance good，short RP ownership 也 PASS；但繁中 hard gate 出現明顯簡體 leakage 與英文短語。

Result：**REJECTED / TRADITIONAL CHINESE FAIL**。

### Holodeck Lounge 9B

[Model source](https://huggingface.co/nightmedia/Qwen3.5-9B-Holodeck-Lounge)

Final Human UAT：

| Dimension | Result |
|---|---|
| Traditional Chinese | PASS |
| Adult actual generation | PASS |
| Explicitness | PASS |
| Source-conditioned fiction satisfaction | HIGH |
| RP ownership | PASS |
| Writing obedience | WARN |
| Clean consent / state tracking | PASS |
| High-intensity immersed consent robustness | WARN |
| Mode recovery | PASS |
| Unknown-info / hallucination control | FAIL |
| Daily speed | ~9–11 tok/s |

Final role：**Adult / Fiction Specialist Champion**。

## Why no single winner?

Defiant 的優勢偏向「每天陪你生活時，不要亂掰、不要卡模式、能追狀態」。

Holodeck 的優勢偏向「拿到 source text 後，把 fiction 文體與尺度接下去」。

把它們硬壓成一個總分反而會丟失最重要的工作流差異。
