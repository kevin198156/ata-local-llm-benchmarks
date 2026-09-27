# ata-local-llm-benchmarks

在一般消費級硬體上實測本地 LLM：效能、繁體中文、Human UAT、角色扮演／小說工作流，以及「真的會不會每天想用」。

> **Phase 0 結論（2026-09-27）**  
> Main Companion：**Defiant Fable 9B**  
> Adult / Fiction Specialist：**Holodeck Lounge 9B**

這不是世界模型排行榜，也不是安全認證。這是一個特定使用者、特定硬體與特定工作流下的 **sanitized 實機紀錄**。

## 測試環境

| 項目 | 環境 |
|---|---|
| OS | Windows 11 |
| CPU | Intel Core i5-12400F |
| GPU | NVIDIA GeForce GTX 1660 Ti 6GB |
| RAM | 32GB |
| Runtime | llama.cpp b11149 / 0.5.0-dev / commit `d2e54583c` |
| 主要格式 | GGUF，候選以 Q4_K_M 為主；個別模型使用其他 quant |
| 主要目標 | 台灣繁中、日常 Companion、fiction/RP、長聊、狀態追蹤、幻覺控制與實際可用速度 |

早期 8B 基準測試在 4K context 下進行；後期 9B finalists 的 Human UAT 主要使用 8K context。不同模型依顯存與 RAM 狀態採不同 GPU offload，因此效能數字是「這台機器上的使用紀錄」，不是跨模型嚴格同條件跑分。

## 為什麼做這個實驗？

起點不是要做 benchmark，而是想知道：

> 一張 GTX 1660 Ti 6GB，能不能跑出一個繁體中文自然、願意長聊、能做 fiction/RP，而且真的會每天想打開的私人 AI？

實驗一路從 8B、9B、12B、14B 到 27B。過程中最重要的發現是：**「Uncensored」、「中文能力」、「成人 actual generation」、「小說文體跟隨」、「RP ownership」、「幻覺控制」和「日常 Companion 感」是不同能力。**

因此最後沒有硬選唯一全能模型，而是依實際工作流分工。

## 最終角色

| Role | Model | Phase 0 結論 |
|---|---|---|
| Main Companion | **Defiant Fable 9B** | 繁中、日常陪伴、RP ownership、clean consent/state tracking、mode recovery、unknown-info control、長聊與約 9 tok/s 綜合表現最適合作為預設模型 |
| Adult / Fiction Specialist | **Holodeck Lounge 9B** | source-conditioned fiction 改寫／續寫的 Human UAT 滿意度最高；繁中、adult actual generation、explicitness、RP ownership、clean consent 與約 9–11 tok/s 通過，但 factual hallucination 與高強度沉浸式 consent robustness 有警告 |

完整結案請看 [PHASE0_REPORT.md](PHASE0_REPORT.md)。

## 文件

- [Phase 0 技術報告](PHASE0_REPORT.md)
- [Human UAT 方法](docs/methodology.md)
- [模型結果整理](docs/model-results.md)
- [實裝心得 / Lessons Learned](docs/lessons-learned.md)

## Human UAT 比單一跑分更重要

本專案後期主要看這些項目：

- Traditional Chinese stability
- Companion naturalness / emotional value
- Adult actual generation
- Explicitness
- Source-conditioned fiction
- Writing obedience
- RP ownership
- Consent / character-state tracking
- Clean consent test
- High-intensity immersed RP robustness
- Mode recovery
- Unknown-info / hallucination control
- Long-session stability
- Instruction switching
- Repetition / trope loop
- Speed / daily usability

詳細定義與抽象測試方法見 [docs/methodology.md](docs/methodology.md)。

## 主要結論

1. **Uncensored 標籤不是能力證明。** 實際生成才算。
2. **模型自述邊界不可靠。** 要看 actual behavior。
3. **從零生成與拿原文續寫是不同能力。**
4. **小說需要的想像力，在 factual QA 可能變成 hallucination。**
5. **Clean consent 與高強度沉浸式 RP robustness 要分開測。**
6. **Human UAT 的考卷會演進，finalists 應用同一份新版 protocol 重測。**
7. **在 GTX 1660 Ti 6GB + 32GB RAM 上，9B Q4 是這次實驗最實際的甜蜜點。** 27B 可以技術上跑起來，但速度與 RAM 壓力不適合此專案的日常目標。
8. **不需要唯一 Winner。** Daily Companion 與 Fiction Specialist 可以是不同模型。

## 一般讀者版本

如果你想先看「一個普通使用者怎麼從 GTX 1660 Ti 一路把本地 AI 測到能用」的開發故事，可看方格子文章：

**[【本地 AI 實驗】一個普通使用者，拿 GTX 1660 Ti 把自己的私人 AI 一路測到能用](https://vocus.cc/article/6ab8ac02fd8978000146aec8)**

這個 GitHub Repo 則保留較技術、可重現的 sanitized 方法與結果。

## Privacy / Sanitization

這個公開 Repo **不包含**：

- raw 成人／私人 UAT transcript
- 私人 Story / Persona / User Memory
- API key、token、`.env`
- GGUF 權重與 runtime binary
- 私人 Git history

成人相關評測只公布抽象 protocol 與 PASS / WARN / FAIL 結論，不發布露骨原文。

## Scope

Phase 0 已結案。本 Repo 目前只公開 Phase 0 的 sanitized 評測與方法，不代表後續私人 Companion runtime 的完整實作。

最後更新：2026-09-27
