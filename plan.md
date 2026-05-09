# Plan: eBay GPU Survey by VRAM Capacity

## Objective
Research sold eBay prices for NVIDIA GPUs (GTX and RTX) categorized by VRAM capacity, to determine the best "bang for buck" upgrade path for running local LLMs via llama.cpp's CPU/VRAM MoE offload (`--n-cpu-moe` or equivalent layer offload) on a machine with 32GB system RAM.

## Budget & Scope
- **Budget cap:** $600 for this initial survey.
- **Market:** eBay.com (US), sold/completed listings only.
- **Condition:** Working / tested cards only. Exclude "for parts or not working" listings.
- **Timeframe:** Recent sales where possible (last 30-90 days).
- **Follow-up note:** A future survey may expand beyond $600 and include Apple Silicon machines (e.g., Mac Studio M1/M2 Unified Memory) to identify the inflection point where value shifts from "old discrete GPU" to "Apple Silicon unified memory."

## Research Methodology

### Search Strategy
We will perform eBay sold-item searches using eBay's `LH_Sold=1&LH_Complete=1` URL filters. Because eBay pages are JS-heavy, we will:
1. Attempt direct FetchURL on parameterized eBay search URLs.
2. Fall back to web search (Google/Bing) queries targeting eBay sold results if direct fetching fails.
3. Searches will be model-agnostic where possible, but we will also search specific popular SKUs known for each VRAM tier to ensure coverage.

### VRAM Tiers to Survey
| Tier | Example Search Terms | Rationale |
|------|---------------------|-----------|
| 4GB | `"nvidia 4gb gpu"`, `"gtx 1050 ti 4gb"` | Baseline / ultra-budget tier |
| 6GB | `"nvidia 6gb gpu"`, `"gtx 1060 6gb"`, `"rtx 2060 6gb"` | User's existing tier; anchor for comparison |
| 8GB | `"nvidia 8gb gpu"`, `"rtx 2060 super"`, `"rtx 3060 ti"`, `"rtx 4060"` | Common mid-range sweet spot |
| 10GB | `"rtx 3080 10gb"` | High-end Ampere entry |
| 11GB | `"gtx 1080 ti"` | Older high-VRAM flagship |
| 12GB | `"rtx 3060 12gb"`, `"rtx 4070"` | Modern mid-range with generous VRAM |
| 16GB | `"rtx 4060 ti 16gb"` | Modern VRAM-optimized card for LLMs |
| 20GB | `"rtx 3080 20gb"` (if any sold) | Rare modded/SKU variant |
| 24GB | `"rtx 3090"`, `"rtx 4090"` | High-end / near-budget-ceiling |

For each tier, we will capture:
- Card model(s) observed
- Price range (low, high, approximate median from 1-2 pages of results)
- Notable outliers or bundles

### Capability Analysis (Post-Price Research)
For each VRAM tier, assuming a host machine with **32GB system RAM**, we will map:
1. **Dense models** that can run partially offloaded to GPU (via `--n-gpu-layers` / `--n-cpu-moe`).
2. **MoE models** that can run with CPU+VRAM balancing.
3. **Specific recommended models** for:
   - Coding / software engineering tasks (e.g., DeepSeek-Coder-V2, Qwen2.5-Coder, Llama 3.3 70B distill)
   - Image analysis / vision (e.g., LLaVA, Qwen-VL)
   - Technical book / long-document processing (context window importance)
4. **Speed vs. capability trade-off**, framed for 24/7 unattended subagent workloads where throughput matters less than model capability.

### Report Structure
The final report (`report.md`) will contain:
1. **Executive Summary:** At-a-glance price table by VRAM tier + "best bang for buck" recommendation.
2. **Detailed Findings:** Per-tier price ranges, specific models observed, and notes on condition/auction vs. buy-it-now.
3. **Capability & Value Analysis:** What stepping up to each VRAM tier unlocks in terms of runnable models, with concrete examples ("stepping up to 12GB unlocks X model which excels at Y task").
4. **"Just Out of Reach" Section:** What the next tier above $600 looks like (e.g., RTX 3090 24GB at ~$700-800? RTX 4090?) and what capabilities it unlocks.
5. **Follow-up Note:** Brief mention of Apple Silicon unified-memory survey as a future value comparison.

## Open Questions / Assumptions
- eBay search results may be incomplete via FetchURL; we will note if data is sparse and rely on web search for triangulation.
- We assume "working" cards are those not explicitly listed as "for parts/not working" in the sold listing title.
- We assume the user's use of `--n-cpu-moe` maps to llama.cpp-style layer offload for MoE architectures; we will verify the specific flag behavior during research if needed.

## Approval Gate
**This plan is pending user approval.** No eBay searches or data collection will begin until the user explicitly approves this plan.
