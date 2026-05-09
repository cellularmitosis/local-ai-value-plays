# eBay GPU Survey by VRAM Capacity: Budget Local LLM Inference

*Research conducted: May 2026*  
*Budget cap: $600 USD*  
*Market: eBay.com (US) and international sellers with US-facing listings*

---

## Executive Summary

This survey examined recent eBay sold and completed-listing prices for NVIDIA GPUs categorized by VRAM capacity. The goal is to identify the best "bang for buck" upgrade path for running local Large Language Models (LLMs) via `llama.cpp` with CPU/VRAM balancing on a machine with 32GB of system RAM.

**Key Finding:** For pure VRAM-per-dollar, the used market favors older flagship cards (GTX 1080 Ti 11GB, RTX 3090 24GB). However, for LLM inference specifically, **Tensor Cores** (found on RTX 20-series and newer) significantly accelerate quantized models. The best-balanced value play is the **RTX 3060 12GB at ~$220 used**, which delivers modern tensor-core acceleration and enough VRAM to run 14B models fully on-card or 32B models with partial offload.

### At-a-Glance Price Table

| VRAM Tier | Typical Models | Price Range (Used) | $/GB | Tensor Cores? | Best For |
|-----------|---------------|-------------------|------|---------------|----------|
| 4GB | GTX 1050 Ti, GTX 1650 | $60 – $90 | ~$18 | No | Avoid for LLMs |
| 6GB | GTX 1060 6GB, RTX 2060 | $50 – $180 | ~$9–30 | Partial* | Entry-level 7B–8B models |
| 8GB | RTX 2060 Super, RTX 3060 Ti | $150 – $260 | ~$19–33 | Yes | 9B–14B models, fast 7B |
| 10GB | RTX 3080 10GB | $300 – $450 | ~$30–45 | Yes | High-speed 14B, partial 32B |
| 11GB | GTX 1080 Ti 11GB | $150 – $230 | ~$14–21 | No | High VRAM on budget, slower inference |
| 12GB | RTX 3060 12GB, RTX 4070 | $220 – $550 | ~$18–46 | Yes | **Sweet spot**: 14B native, 32B offload |
| 16GB | RTX 4060 Ti 16GB | $460 – $680 | ~$29–43 | Yes (Ada) | 20B native, 70B distill offload |
| 24GB | RTX 3090 | $580 – $850 | ~$24–35 | Yes | 35B native, 70B moderate offload |

\* RTX 2060 has Tensor Cores; GTX 1060 does not.

### Best Bang for Buck Rankings

1. **🥇 RTX 3060 12GB (~$220 used)** — Best balance of tensor-core speed, VRAM capacity, and price. Can run 14B models entirely in VRAM and 32B models with offload. This is the recommended upgrade from a GTX 1060 6GB.
2. **🥈 GTX 1060 6GB (~$55 used)** — The user's existing card. Surprisingly capable for 7B–8B models and MoE offloading. At $55, it's the cheapest entry point.
3. **🥉 RTX 3090 24GB (~$580+ used)** — The "endgame" card within budget. Unlock 35B models fully in VRAM. Occasional deals hit ~$580; most are $650–$800.
4. **GTX 1080 Ti 11GB (~$160 used)** — Excellent VRAM-per-dollar, but Pascal architecture lacks tensor cores. Inferior to RTX 3060 12GB for LLMs despite more VRAM.
5. **RTX 4060 Ti 16GB (~$460+ new)** — Best *new* card for LLMs under $600. Very efficient Ada architecture, but premium pricing.

---

## Detailed Findings

### Methodology Note
Searches targeted eBay sold/completed listings with "sold" filters where possible. Because eBay blocks automated access to sold-item pages, prices were triangulated from: (1) eBay search snippets showing completed sale prices, (2) auction closing prices, (3) third-party price aggregators tracking eBay historical data, and (4) "Buy It Now" prices from high-volume used GPU sellers. All prices are approximate and exclude shipping unless noted.

---

### 4GB Tier
**Models:** GTX 1050 Ti 4GB, GTX 1650 4GB  
**Price Range:** $60 – $90 used; ~$70 eBay refurbished  
**Reference URLs:**
- eBay category: [NVIDIA GeForce GTX 1050 Ti 4GB listings](https://www.ebay.com/b/NVIDIA-GeForce-GTX-1050-Ti-4GB-Computer-Graphics-Cards/27386/bn_110679085)
- eBay shop: [1050 Ti 4GB sold/completed](https://www.ebay.com/shop/1050-ti-4gb?_nkw=1050+ti+4gb)

**Observation:** This tier is effectively obsolete for modern LLMs. A 4GB card can barely run 3B–4B parameter models at Q4 quantization. Even 7B models require aggressive offload and provide poor token throughput. **Not recommended** for this project.

---

### 6GB Tier
**Models:** GTX 1060 6GB, RTX 2060 6GB, GTX 1660 6GB  
**Price Range:**
- **GTX 1060 6GB:** $50 – $65 pre-owned; ~$130 eBay refurbished
- **RTX 2060 6GB:** ~$150 – $180 used
**Reference URLs:**
- eBay category: [NVIDIA GeForce GTX 1060 6GB listings](https://www.ebay.com/b/NVIDIA-GeForce-GTX-1060-6GB-GDDR5-Computer-Graphics-Cards/27386/bn_110679878)
- eBay category: [NVIDIA GeForce GTX 1060 (all) listings](https://www.ebay.com/b/NVIDIA-GeForce-GTX-1060-NVIDIA-Computer-Graphics-Cards-for-PCI/27386/bn_7116463499)

**Observation:** The GTX 1060 6GB anchors the bottom of the market. At $50–$60, it is the cheapest viable GPU for local LLMs. However, it lacks tensor cores, so quantized inference is CPU-bound for large layers. The RTX 2060 6GB adds tensor cores and slightly faster memory but is 3× the price for the same VRAM capacity. If you already own a GTX 1060 6GB, spending $150+ for an RTX 2060 6GB is poor value—you're paying for speed, not capacity.

---

### 8GB Tier
**Models:** RTX 2060 Super 8GB, RTX 3060 Ti 8GB, RTX 4060 8GB  
**Price Range:**
- **RTX 2060 Super 8GB:** $150 – $200 used; ~$240 refurbished
- **RTX 3060 Ti 8GB:** $190 – $260 used (auction closings ~$140–$180; retail used ~$220)
- **RTX 4060 8GB:** ~$300+ new
**Reference URLs:**
- eBay shop: [RTX 2060 Super listings](https://www.ebay.com/shop/2060-super?_nkw=2060+super)
- eBay shop: [RTX 2060 Super 8GB listings](https://www.ebay.com/shop/2060-super-8gb?_nkw=2060+super+8gb)
- eBay category: [NVIDIA RTX 2060 8GB listings](https://www.ebay.com/b/NVIDIA-GeForce-RTX-2060-NVIDIA-8-GB-Memory-Computer-Graphics-Cards/27386/bn_7116827251)

**Observation:** This is the first tier where modern tensor-core acceleration meets reasonable VRAM. An 8GB card can run 7B–9B dense models fully in VRAM and 14B models with a few layers offloaded. The RTX 3060 Ti 8GB is the standout here—Ampere tensor cores are significantly faster than Turing (RTX 20-series) for INT8/FP16 operations. However, 8GB is still constraining for larger models; you will rely heavily on CPU offload for anything above 14B parameters.

---

### 10GB Tier
**Models:** RTX 3080 10GB  
**Price Range:** $300 – $450 used  
**Observation:** The RTX 3080 10GB is a high-performance gaming card with fast GDDR6X memory and excellent tensor-core throughput. However, its VRAM is oddly limiting for LLM work—10GB is only slightly more than 8GB, and the card carries a significant price premium over the RTX 3060 12GB. For LLMs specifically, a $220 RTX 3060 12GB is generally more useful than a $350 RTX 3080 10GB because the extra 2GB of VRAM matters more than raw compute for memory-bound inference.

---

### 11GB Tier
**Models:** GTX 1080 Ti 11GB  
**Price Range:** $150 – $230 used  
**Reference URLs:**
- eBay shop: [GTX 1080 Ti 11GB listings](https://www.ebay.com/shop/1080-ti-11gb?_nkw=1080+ti+11gb)
- eBay category: [NVIDIA GeForce GTX 1080 Ti listings](https://www.ebay.com/b/NVIDIA-GeForce-GTX-1080-Ti-NVIDIA-Computer-Graphics-Cards/27386/bn_7116464572)

**Observation:** The GTX 1080 Ti is a fascinating value anomaly. With 11GB of VRAM at ~$160, it offers the best dollars-per-gigabyte of any surveyed card (~$14.50/GB). For workloads that simply need VRAM capacity (e.g., running 13B–14B models fully in memory), it's compelling. **However**, the Pascal architecture lacks tensor cores entirely. In `llama.cpp`, Q4 quantized inference on Pascal runs via CUDA cores at roughly 1/3 to 1/4 the speed of an RTX card. For a 24/7 unattended subagent where speed is secondary, this *might* be acceptable, but an RTX 3060 12GB at $220 will outperform it in tokens-per-second despite only 1GB more VRAM.

---

### 12GB Tier
**Models:** RTX 3060 12GB, RTX 4070 12GB  
**Price Range:**
- **RTX 3060 12GB:** $190 – $300 used (auction closings ~$190; solid used units ~$230–$260)
- **RTX 4070 12GB:** $500 – $550 new
**Reference URLs:**
- eBay shop: [RTX 3060 12GB listings](https://www.ebay.com/shop/rtx-3060-12gb?_nkw=rtx+3060+12gb)
- eBay listing: [DELL NVIDIA RTX 3060 12GB](https://www.ebay.com/itm/186469842323)
- eBay listing: [EVGA NVIDIA GeForce RTX 3060 12GB](https://www.ebay.com/b/EVGA-NVIDIA-GeForce-RTX-3060-12GB-GDDR6-Graphics-Video-Cards/27386/bn_7117808167)
- Sold listing: [LENOVO RTX 3060 OEM 12GB — $185.00](https://www.ebay.com/itm/205797314734)

**Observation:** The RTX 3060 12GB is the **surprise champion** of this survey. It is the cheapest used card that combines tensor cores with 12GB of VRAM—enough to fit 14B parameter models entirely in GPU memory at Q4_K_M quantization. It can also run 32B models with partial CPU offload at usable speeds. Because RTX 3060 cards were produced in massive volume and many were used for mining, the used market is saturated with supply. The RTX 4070 12GB is much faster and more efficient but costs 2× as much; it's a better choice if buying new, but poor used value compared to the 3060.

---

### 16GB Tier
**Models:** RTX 4060 Ti 16GB  
**Price Range:** $460 – $680 new  
**Reference URLs:**
- eBay shop: [RTX 4060 Ti 16GB listings](https://www.ebay.com/shop/4060-16gb?_nkw=4060+16gb)

**Observation:** The RTX 4060 Ti 16GB was explicitly designed as a "content creation" card with extra VRAM for AI workloads. Its Ada Lovelace tensor cores are very efficient, and 16GB is enough to run 20B models fully in VRAM or 32B models with light offload. The downside is price: at $460+, it approaches the cost of a used RTX 3090 24GB. If you want a **new card with warranty**, this is the best option under $600. If you're willing to buy used, the RTX 3090 24GB is a far better value.

---

### 24GB Tier
**Models:** RTX 3090 24GB, RTX 4090 24GB  
**Price Range:**
- **RTX 3090 24GB:** $580 – $850 used (occasional deals at ~$580; typical range $650–$750)
- **RTX 4090 24GB:** $1,500+ (well above budget)
**Reference URLs:**
- eBay shop: [NVIDIA RTX 3090 24GB listings](https://www.ebay.com/shop/nvidia-3090-24gb?_nkw=nvidia+3090+24gb)
- eBay shop: [RTX 3090 listings](https://www.ebay.com/shop/rtx-3090?_nkw=rtx+3090)
- eBay listing: [Manli NVIDIA GeForce RTX 3090 24GB](https://www.ebay.com/itm/136219749033)

**Observation:** The RTX 3090 24GB is the legendary "poor man's AI card." It is the cheapest way to get 24GB of VRAM with full tensor-core acceleration. At Q4_K_M quantization, it can run 35B parameter models entirely in VRAM and 70B models with partial offload. The trade-offs are well-known: high power draw (350W TDP), significant heat output, and the risk of buying an ex-mining card. However, for 24/7 subagent workloads, these concerns are manageable if the card passes a stress test. **If you can stretch to ~$600–$650, this is the single largest capability leap in the entire survey.**

---

## Capability & Value Analysis: What Does Each Tier Unlock?

This analysis assumes **32GB of system RAM** and `llama.cpp` with quantization at **Q4_K_M** (the recommended quality/efficiency balance). Two offload strategies are considered:

1. **Dense models:** Use `--n-gpu-layers` to split transformer layers between GPU and CPU.
2. **MoE (Mixture of Experts) models:** Use `--n-cpu-moe` to offload *expert* layers to CPU while keeping attention/routing layers on GPU. This is dramatically more efficient because only a subset of experts activate per token.

### 6GB Tier (Your Current GTX 1060 6GB)
**Price:** ~$55 used  
**What it can run:**
- **Dense 7B–8B models fully in VRAM:** Llama 3.1 8B Instruct, Qwen2.5 7B Instruct, Gemma 2 9B (with slight offload). These are competent general-purpose assistants and adequate for simple coding tasks.
- **Dense 14B models with heavy CPU offload:** Qwen2.5 14B, DeepSeek-R1-Distill-Qwen-14B. Expect ~1–3 tokens/sec.
- **MoE 35B+ models with `--n-cpu-moe`:** As demonstrated in your reference video, models like **Qwen3.5-35B-A3B** (35B total params, ~3B active per token) can run by offloading expert layers to the 32GB of system RAM. This is the "killer app" for this tier—you get access to a 35B-class model for $55.

**Verdict:** Don't underestimate the GTX 1060 6GB. For 7B–8B workloads, it's fine. For larger MoE models, it works but inference is slow (~2–5 tok/s). At $55 replacement cost, it's the cheapest way to experiment.

---

### 8GB Tier (RTX 2060 Super / RTX 3060 Ti)
**Price:** $150 – $260 used  
**What it unlocks vs. 6GB:**
- **9B dense models fully in VRAM:** Stable, fast inference for Llama 3.1 8B and Qwen2.5 7B.
- **14B models with moderate offload:** Much better token rates than 6GB for 14B-class models because more layers fit on-card.
- **Better MoE performance:** More expert layers stay on GPU, improving tok/s on Mixtral 8x7B and Qwen3.5-35B.

**Specific model examples:**
- **Qwen2.5-Coder-14B-Instruct:** A significant step up from 7B coding models. Capable of generating multi-file project scaffolding, debugging complex code, and understanding larger context windows. Fits partially in 8GB.
- **Mixtral 8x7B MoE:** 47B total parameters, ~13B active. Excellent general reasoning. With `--n-cpu-moe`, this becomes viable on 8GB with acceptable speed.

**Verdict:** A meaningful upgrade from 6GB, primarily because you gain tensor cores (if moving from GTX to RTX) and enough room for 14B models. The RTX 3060 Ti 8GB at ~$220 is a solid choice, but the RTX 3060 12GB is only marginally more expensive and much more capable.

---

### 11GB Tier (GTX 1080 Ti)
**Price:** ~$160 used  
**What it unlocks vs. 8GB:**
- **14B models fully in VRAM:** Models like Qwen2.5 14B or Llama 3.1 8B with massive context windows fit entirely in GPU memory.
- **32B models with heavy offload:** More VRAM means more layers on GPU before hitting CPU.

**The Catch:** No tensor cores. Inference on 14B models will be slower on a GTX 1080 Ti than on an RTX 3060 Ti 8GB, despite having more VRAM. The 1080 Ti is only a good choice if you find one cheap (~$150) and primarily care about fitting models in memory, not speed.

---

### 12GB Tier (RTX 3060 12GB — Recommended)
**Price:** ~$220 used  
**What it unlocks vs. 8GB:**
- **14B models fully in VRAM:** Qwen2.5 14B, DeepSeek-R1-Distill-Qwen-14B run entirely on GPU at excellent speed.
- **32B models with partial offload:** **Qwen2.5-Coder-32B-Instruct** and **DeepSeek-R1-Distill-Qwen-32B** become accessible. These are genuinely powerful coding and reasoning models—capable of competitive programming tasks, large-scale refactoring, and deep reasoning chains.
- **Vision models:** 12GB is enough to run multimodal models like **LLaVA-1.6 34B** or **Qwen2-VL 7B** with comfortable headroom for image patches.

**Why this is the sweet spot:**
At $220, you triple your effective model capacity over the GTX 1060 6GB and gain tensor cores. The jump from 7B to 32B parameters is transformative for coding quality. A 32B model can reason about complex abstractions, maintain coherence across long files, and follow intricate instructions—capabilities that 7B models simply lack.

---

### 16GB Tier (RTX 4060 Ti 16GB)
**Price:** ~$460+ new  
**What it unlocks vs. 12GB:**
- **20B–22B models fully in VRAM:** **Codestral 22B**, **Qwen2.5 32B** (tight but possible with KV cache quantization).
- **32B models with light offload:** Much faster than 12GB because more layers stay on GPU.
- **70B distill models with moderate offload:** **DeepSeek-R1-Distill-Llama-70B**—a major leap in reasoning. This model rivals cloud APIs on math and logic tasks.
- **Large vision models:** **Qwen2-VL 72B** (with significant offload) or **InternVL2 26B** fully in VRAM for image analysis and OCR.

**Verdict:** The 16GB tier is the gateway to "serious" local inference. The 70B distill models are qualitatively better than 32B models at coding, math, and long-context reasoning. However, at $460+, it's only $120 less than a used RTX 3090 24GB. If buying used, skip this and go straight to 24GB.

---

### 24GB Tier (RTX 3090)
**Price:** ~$580–$750 used  
**What it unlocks vs. 16GB:**
- **35B models fully in VRAM:** **Qwen3.5-35B** or **Yi-1.5 34B** run entirely on GPU at high speed with large context windows.
- **70B models with partial offload:** At Q4_K_M, a 70B model needs ~44GB. With 24GB VRAM + 32GB RAM, you can offload ~40% of layers to CPU and still achieve usable throughput (3–6 tok/s depending on CPU).
- **Massive context windows:** 24GB allows 32K–64K context on 14B–32B models without CPU offload, ideal for processing entire technical books or large codebases.
- **MoE monsters:** Models like **DeepSeek-V2-Lite** (16B active, 236B total) or **Mixtral 8x22B** (141B total, ~39B active) become viable with `--n-cpu-moe`.

**The "Endgame" Perspective:**
An RTX 3090 24GB transforms a 32GB-RAM machine into a local AI workstation. You can run a 70B reasoning model overnight on complex tasks, process 50-page PDFs in a single context window, or host a 35B coding model as a fast local API. For a subagent that runs 24/7, this is the point where capability approaches mid-tier cloud APIs.

---

## Model Recommendations by Task

### For Coding Projects (Primary Use Case)
| Budget | Card | Model | Capability |
|--------|------|-------|------------|
| $0 (existing) | GTX 1060 6GB | Qwen2.5-Coder-7B-Instruct | Basic autocomplete, simple functions, debugging small snippets |
| $220 | RTX 3060 12GB | Qwen2.5-Coder-32B-Instruct | Multi-file projects, refactoring, competitive programming, test generation |
| $580 | RTX 3090 24GB | DeepSeek-R1-Distill-Llama-70B | Complex architecture design, deep reasoning, legacy code migration, large-scale debugging |

### For Image Analysis / Vision
| Budget | Card | Model | Capability |
|--------|------|-------|------------|
| $220 | RTX 3060 12GB | Qwen2-VL 7B / LLaVA-1.6 7B | Image captioning, OCR, chart interpretation, UI analysis |
| $580 | RTX 3090 24GB | Qwen2-VL 72B (with offload) | Detailed visual reasoning, multi-image comparison, technical diagram analysis |

### For Technical Book / Long-Document Processing
| Budget | Card | Model | Context | Capability |
|--------|------|-------|---------|------------|
| $55 | GTX 1060 6GB | Qwen2.5 7B Instruct | 32K–64K* | Summarize chapters, extract key concepts, Q&A |
| $220 | RTX 3060 12GB | Qwen2.5 32B Instruct | 64K–128K | Generate tutorials from textbooks, cross-reference chapters, create study guides |
| $580 | RTX 3090 24GB | Qwen2.5 72B / Llama 3.3 70B | 128K | Deep synthesis across entire books, generate course curricula, compare multiple sources |

\* Requires KV cache quantization to fit large contexts on small VRAM.

---

## "Just Out of Reach"

### Above the $600 Cap (Today's Market)
| Card | VRAM | Typical Price | What It Unlocks |
|------|------|--------------|-----------------|
| RTX 3090 Ti 24GB | 24GB | ~$900–$1,000 | Slightly faster 24GB; marginal improvement over RTX 3090 |
| RTX 4080 Super 16GB | 16GB | ~$900–$1,100 | Fastest 16GB card; excellent for 24/7 efficiency, but less VRAM than 3090 |
| RTX 4090 24GB | 24GB | ~$1,500+ | ~2× faster than 3090; same VRAM ceiling. The true consumer flagship. |
| Mac Studio M1 Ultra | 64GB Unified | ~$2,000+ (used) | **Inflection point.** 64GB unified memory runs 70B models fully in memory with zero offload latency. Massive context windows. Silent operation. |
| Mac Studio M2 Ultra | 64–192GB Unified | ~$2,500+ | The ultimate local LLM machine for non-technical users. 192GB runs 120B+ models. |

### The Apple Silicon Inflection Point
At roughly $2,000–$2,500 for a used Mac Studio M1/M2 Ultra, the value proposition shifts dramatically. Unified memory means no CPU/GPU copy penalties, and 64GB+ is enough to run 70B dense models entirely in memory. For a 24/7 subagent, a Mac Studio is quieter, more power-efficient, and simpler to maintain than a multi-GPU PC. **However**, CUDA ecosystem compatibility (e.g., some `llama.cpp` features, PyTorch workflows) is weaker than on NVIDIA. This is recommended as a **follow-up survey** topic.

---

## Final Recommendations

### If You Want to Spend Nothing
**Keep your GTX 1060 6GB.** Use it to run Qwen2.5-Coder-7B and experiment with `--n-cpu-moe` on 35B MoE models. It's a capable entry point.

### If You Want the Best Value Upgrade
**Buy an RTX 3060 12GB for ~$220.** This is the largest capability-per-dollar leap in the survey. You go from 7B/8B models to running 32B coding models with partial offload, or 14B models fully in VRAM with tensor-core speed.

### If You Want Maximum Capability Under $600
**Hunt for an RTX 3090 24GB at ~$580–$650.** Check eBay auctions, /r/hardwareswap, and local listings. A 24GB card is transformative—you unlock 35B native inference and usable 70B offload. Be prepared to verify the card's health (FurMark stress test, check for artifacting).

### If You Must Buy New
**Get an RTX 4060 Ti 16GB for ~$460–$500.** You lose VRAM versus a used 3090 but gain warranty, efficiency, and modern features. This is the safest choice if you don't trust the used market.

### What to Avoid
- **GTX 1080 Ti 11GB at $160+:** Tempting VRAM, but Pascal lacks tensor cores. An RTX 3060 12GB at $220 is better.
- **RTX 3080 10GB at $350+:** Fast, but 10GB is awkwardly positioned. The extra 2GB of a 3060 12GB is more useful than the 3080's raw speed for LLMs.
- **RTX 4060 8GB at $300+:** Current-gen but only 8GB. Outclassed by used 12GB and 16GB options.

---

## Follow-Up Survey Suggestion

As noted in the plan, a logical next step is to compare these discrete GPU options against **Apple Silicon Macs with unified memory** (Mac Studio M1/M2 Ultra, Mac mini M4 Pro with 64GB). At the $2,000+ price point, Apple Silicon offers 64GB–192GB of unified memory with no CPU/GPU copy overhead, potentially running 70B–120B models entirely in memory. This survey should examine whether the "value play" shifts from "cheap NVIDIA card + CPU offload" to "used Mac Studio" at higher budgets.

---

*Report compiled from eBay market data, llama.cpp documentation, and hardware benchmarking sources. Prices are approximate and subject to market fluctuation.*
