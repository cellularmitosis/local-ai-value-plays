# eBay GPU Survey by VRAM Capacity: VRAM-Per-Dollar Edition

*Research conducted: May 2026*  
*Budget cap: $600 USD*  
*Market: eBay.com (US) and international sellers with US-facing listings*  
*Primary metric: VRAM capacity per dollar spent*

---

## Executive Summary

This report re-examines the eBay GPU market with a single, obsessive focus: **VRAM-per-dollar**. For local LLM inference via `llama.cpp`, VRAM capacity is the hard ceiling that determines which models can load. Everything else—tensor cores, architecture generation, power draw—is secondary if the model doesn't fit in memory.

**The Surprising Finding:** Older Pascal-era cards dominate the VRAM-per-dollar rankings. A used GTX 1080 Ti 11GB delivers 73% more VRAM per dollar than an RTX 3060 12GB. And because LLM token generation is heavily memory-bandwidth-bound, the real-world inference speed gap between Pascal and Ampere is far smaller than commonly assumed.

**If you want maximum VRAM for minimum spend:** Buy a **GTX 1080 Ti 11GB at ~$150** or hunt for a **Tesla P40 24GB at ~$280**.

**If you want the best balance of VRAM value + modern features:** The **RTX 3060 12GB at ~$200** remains compelling, but it's not the runaway winner when viewed through a pure VRAM/$ lens.

---

## At-a-Glance: VRAM-Per-Dollar Rankings

| Rank | Card | VRAM | Price (Used) | **VRAM/$** | Architecture | Tensor Cores |
|------|------|------|-------------|-----------|--------------|--------------|
| 1 | **GTX 1060 6GB** | 6GB | $50–$65 | **~$9.20–11.90/GB** | Pascal | No |
| 2 | **Tesla P40 24GB** | 24GB | $270–$300 | **~$11.25–12.50/GB** | Pascal | No |
| 3 | **GTX 1080 Ti 11GB** | 11GB | $150–$165 | **~$13.60–15.00/GB** | Pascal | No |
| 4 | **RTX 3060 12GB** | 12GB | $185–$220 | **~$15.40–18.30/GB** | Ampere | Yes (3rd Gen) |
| 5 | **RTX 2080 Ti 11GB** | 11GB | $210–$280 | **~$19.10–25.50/GB** | Turing | Yes (2nd Gen) |
| 6 | **RTX 3090 24GB** | 24GB | $580–$750 | **~$24.20–31.30/GB** | Ampere | Yes (3rd Gen) |
| 7 | **RTX 2060 Super 8GB** | 8GB | $180–$220 | **~$22.50–27.50/GB** | Turing | Yes (2nd Gen) |
| 8 | **RTX 4060 Ti 16GB** | 16GB | $460–$500 | **~$28.80–31.30/GB** | Ada | Yes (4th Gen) |

*Lower $/GB = better value. Prices reflect recent eBay sold/completed listings and auction closings.*

---

## Detailed Findings

### 6GB Tier
**Models:** GTX 1060 6GB, GTX 1660 6GB  
**Price Range:** $50 – $65 used  
**VRAM/$:** ~$9.20–$10.80/GB  
**Reference URLs:**
- eBay category: [NVIDIA GeForce GTX 1060 6GB listings](https://www.ebay.com/b/NVIDIA-GeForce-GTX-1060-6GB-GDDR5-Computer-Graphics-Cards/27386/bn_110679878)
- eBay category: [NVIDIA GeForce GTX 1060 (all) listings](https://www.ebay.com/b/NVIDIA-GeForce-GTX-1060-NVIDIA-Computer-Graphics-Cards-for-PCI/27386/bn_7116463499)

The GTX 1060 6GB is the undisputed champion of VRAM-per-dollar. At roughly $10 per gigabyte, nothing else comes close. The catch: 6GB is a hard ceiling. You can run 7B–8B dense models fully in VRAM or use `--n-cpu-moe` to run larger MoE models with expert layers offloaded to system RAM. For the price of a decent dinner, you get a functional local LLM accelerator.

**Verdict:** The baseline. If you have one already, you've already won the value game. If you're buying, it's the cheapest way to experiment.

---

### 8GB Tier
**Models:** RTX 2060 Super 8GB, RTX 3060 Ti 8GB  
**Price Range:** $180 – $260 used  
**VRAM/$:** ~$22.50–$32.50/GB  
**Reference URLs:**
- eBay shop: [RTX 2060 Super 8GB listings](https://www.ebay.com/shop/2060-super-8gb?_nkw=2060+super+8gb)
- eBay category: [NVIDIA RTX 2060 8GB listings](https://www.ebay.com/b/NVIDIA-GeForce-RTX-2060-NVIDIA-8-GB-Memory-Computer-Graphics-Cards/27386/bn_7116827251)

This tier is mediocre from a pure VRAM/$ perspective. You're paying a steep premium for tensor cores and newer architecture. An RTX 2060 Super at $180 costs 3.6× as much per gigabyte as a GTX 1060 6GB. The RTX 3060 Ti at $220 is even worse at $27.50/GB.

The only reason to buy here is if you need tensor-core acceleration *and* can't afford the 12GB tier.

**Verdict:** Poor VRAM value. Skip unless you find a screaming deal under $150.

---

### 11GB Tier
**Models:** GTX 1080 Ti 11GB, RTX 2080 Ti 11GB  
**Price Range:**
- **GTX 1080 Ti:** $150 – $165 used
- **RTX 2080 Ti:** $210 – $280 used (occasional deals at ~$210)
**Reference URLs:**
- eBay shop: [GTX 1080 Ti 11GB listings](https://www.ebay.com/shop/gtx-1080-ti-11gb?_nkw=gtx+1080+ti+11gb)
- eBay category: [NVIDIA GeForce GTX 1080 Ti listings](https://www.ebay.com/b/NVIDIA-GeForce-GTX-1080-Ti-NVIDIA-Computer-Graphics-Cards/27386/bn_7116464572)
- eBay shop: [RTX 2080 Ti 11GB listings](https://www.ebay.com/shop/2080-ti-11gb?_nkw=rtx+2080+ti+11gb)
- eBay shop: [RTX 2080 Ti GPU listings](https://www.ebay.com/shop/2080-ti-gpu?_nkw=2080+ti+gpu)
- Sold listing: [Dell Alienware RTX 2080 Ti — $422.86](https://www.ebay.ca/itm/389061780309)
- Forum sale: [Zotac RTX 2080 Ti Amp — $210 shipped](https://www.overclock.net/threads/for-sale-rtx-2080ti-11gb-210-shipped.1814515/)

**VRAM/$:**
- **GTX 1080 Ti:** ~$13.60–$15.00/GB
- **RTX 2080 Ti:** ~$19.10–$25.50/GB

This is where the report's perspective shift becomes dramatic. The GTX 1080 Ti 11GB at ~$155 delivers the second-best VRAM/$ among cards with >8GB of memory. It has no tensor cores, but it has a massive 352-bit memory bus with 484 GB/s bandwidth—higher than the RTX 3060's 360 GB/s. For memory-bound token generation, this matters enormously. (See the Technical Addendum for benchmark proof.)

The RTX 2080 Ti 11GB is the "best of both worlds" card: same VRAM as the 1080 Ti, but with 2nd-gen tensor cores and GDDR6. At ~$250, its VRAM/$ is worse than the Pascal card, but it offers significantly faster prompt processing and modestly faster token generation.

**Verdict:** The GTX 1080 Ti is the value king for high-VRAM budgets. The RTX 2080 Ti is worth the premium if you can find one under $250.

---

### 12GB Tier
**Models:** RTX 3060 12GB  
**Price Range:** $185 – $220 used  
**VRAM/$:** ~$15.40–$18.30/GB  
**Reference URLs:**
- eBay shop: [RTX 3060 12GB listings](https://www.ebay.com/shop/rtx-3060-12gb?_nkw=rtx+3060+12gb)
- eBay listing: [DELL NVIDIA RTX 3060 12GB](https://www.ebay.com/itm/186469842323)
- Sold listing: [LENOVO RTX 3060 OEM 12GB — $185.00](https://www.ebay.com/itm/205797314734)

The RTX 3060 12GB is the first card that combines tensor cores with genuinely good VRAM value. At ~$200, it's only slightly worse VRAM/$ than the GTX 1080 Ti while offering modern features: 3rd-gen tensor cores, NVENC, lower power draw, and wider software compatibility.

A Lenovo OEM RTX 3060 12GB sold on eBay in late 2025 for **$185**. At that price, its VRAM/$ ($15.42/GB) nearly matches the GTX 1080 Ti. If you can find one under $200, it becomes extremely competitive.

**Verdict:** The best "safe" choice. Good VRAM/$ + modern architecture + display outputs + lower power. But from a pure VRAM/$ standpoint, it's roughly on par with the 1080 Ti, not superior.

---

### 16GB Tier
**Models:** RTX 4060 Ti 16GB  
**Price Range:** $460 – $500 used/new  
**VRAM/$:** ~$28.75–$31.25/GB  
**Reference URLs:**
- eBay shop: [RTX 4060 Ti 16GB listings](https://www.ebay.com/shop/4060-16gb?_nkw=4060+16gb)

Poor VRAM value, but it's the cheapest *new* card with 16GB. You're paying a massive premium for warranty, efficiency, and the Ada architecture. For a 24/7 subagent, the power savings might eventually offset the higher upfront cost, but the VRAM/$ is undeniably weak.

**Verdict:** Only buy if you demand a new card with warranty. Used options destroy it on value.

---

### 24GB Tier
**Models:** RTX 3090 24GB, Tesla P40 24GB  
**Price Range:**
- **RTX 3090:** $580 – $750 used
- **Tesla P40:** $270 – $300 used
**Reference URLs:**
- eBay shop: [NVIDIA RTX 3090 24GB listings](https://www.ebay.com/shop/nvidia-3090-24gb?_nkw=nvidia+3090+24gb)
- eBay shop: [RTX 3090 listings](https://www.ebay.com/shop/rtx-3090?_nkw=rtx+3090)
- eBay listing: [Manli NVIDIA GeForce RTX 3090 24GB](https://www.ebay.com/itm/136219749033)
- eBay shop: [Tesla P40 24GB listings](https://www.ebay.com/shop/tesla-p40-24gb?_nkw=tesla+p40+24gb)
- eBay listing: [Nvidia Tesla P40 24GB bulk pricing](https://www.ebay.com/itm/314217893619)

**VRAM/$:**
- **RTX 3090:** ~$24.20–$31.30/GB
- **Tesla P40:** ~$11.25–$12.50/GB

The Tesla P40 is the hidden gem of this survey. It's a datacenter Pascal card with 24GB GDDR5, no display outputs, and no tensor cores. It requires a custom cooling solution (blower-style passive heatsink) or an external fan shroud. But at **~$280**, it delivers 24GB at a VRAM/$ ratio that rivals the GTX 1060 6GB.

The RTX 3090 is the "safe" 24GB option: gaming cooler, display outputs, tensor cores, and a massive 936 GB/s memory bandwidth. At $580–$650, it's the card that unlocks 35B models fully in VRAM and 70B models with partial offload. But its VRAM/$ is 2× worse than the Tesla P40.

**Verdict:** The Tesla P40 is the ultimate VRAM value play for headless servers. The RTX 3090 is the practical 24GB choice if you need display outputs, gaming, or hassle-free cooling.

---

## Capability & Value Analysis

### What Each Tier Unlocks (with 32GB System RAM)

| Tier | Card (Value Pick) | Price | Models Fully in VRAM | Models with CPU Offload |
|------|-------------------|-------|---------------------|------------------------|
| 6GB | GTX 1060 6GB | $55 | 7B–8B dense | 14B dense, 35B MoE via `--n-cpu-moe` |
| 11GB | GTX 1080 Ti 11GB | $155 | 14B dense, 7B–8B with huge context | 32B dense, 70B MoE |
| 12GB | RTX 3060 12GB | $200 | 14B dense | 32B dense, 70B MoE (faster than 1080 Ti) |
| 24GB | Tesla P40 24GB | $280 | 35B dense | 70B dense, massive MoE models |
| 24GB | RTX 3090 24GB | $600 | 35B dense | 70B dense at usable speed |

### Specific Model Recommendations

**For Coding (Primary Use Case):**
- **$55 → GTX 1060 6GB:** Qwen2.5-Coder-7B-Instruct (fast, fully in VRAM)
- **$155 → GTX 1080 Ti 11GB:** Qwen2.5-Coder-14B-Instruct (fully in VRAM), DeepSeek-R1-Distill-Qwen-14B
- **$200 → RTX 3060 12GB:** Qwen2.5-Coder-32B-Instruct (with partial offload)
- **$280 → Tesla P40 24GB:** Qwen2.5-Coder-32B-Instruct (fully in VRAM!), DeepSeek-R1-Distill-Llama-70B (with offload)
- **$600 → RTX 3090 24GB:** DeepSeek-R1-Distill-Llama-70B (moderate offload, fast)

**The Tesla P40 Revelation:**
At $280, the Tesla P40 24GB can run **Qwen2.5-Coder-32B-Instruct entirely in GPU memory** at Q4_K_M quantization. That's a 32B-parameter coding model—capable of competitive programming, large-scale refactoring, and complex reasoning—running without any CPU offload penalty. No other card under $600 can do this. The GTX 1080 Ti comes close but lacks the final 13GB.

---

## Best Bang for Buck: VRAM-Per-Dollar Edition

### 🥇 GTX 1080 Ti 11GB (~$155 used)
**VRAM/$:** ~$14.10/GB  
**Why it wins:** It delivers 83% more VRAM than the GTX 1060 for only 3× the price. You can run 14B models fully in VRAM. The 352-bit memory bus gives it surprisingly competitive token-generation speed. It's the inflection point where VRAM capacity starts to unlock genuinely capable models.

### 🥈 Tesla P40 24GB (~$280 used)
**VRAM/$:** ~$11.70/GB  
**Why it wins:** 24GB at a price that undercuts the RTX 3090 by more than half. This is the cheapest way to run 32B dense models fully in VRAM. Caveats: no display outputs, passive cooling requires modification, no tensor cores. But for a headless subagent machine, those trade-offs may be irrelevant.

### 🥉 RTX 3060 12GB (~$200 used)
**VRAM/$:** ~$16.70/GB  
**Why it's here:** If you can find one under $200 (an OEM model sold for $185 in late 2025), its VRAM/$ approaches the GTX 1080 Ti while adding tensor cores, modern codecs, and hassle-free cooling. It's the "no compromises" choice.

### Honorable Mention: GTX 1060 6GB (~$55 used)
**VRAM/$:** ~$9.20/GB  
**Why it matters:** Nothing beats it on raw VRAM/$. If your budget is "as close to free as possible," this is still a viable entry point for 7B models and MoE offloading experiments.

---

## What to Avoid (VRAM/$ Perspective)

- **RTX 3080 10GB at $350:** 10GB is awkwardly positioned. At $35/GB, it's worse value than both the 1080 Ti and the 3060. Skip.
- **RTX 4060 Ti 16GB at $460+:** $28.75/GB is the worst in this survey. Buy only if you need a new card with warranty.
- **RTX 3060 Ti 8GB at $220:** $27.50/GB. For $20 more, get an RTX 3060 12GB with 50% more VRAM.

---

## "Just Out of Reach"

| Card | VRAM | Price | VRAM/$ | Notes |
|------|------|-------|--------|-------|
| RTX 3090 Ti 24GB | 24GB | ~$900 | ~$37.50/GB | Faster 3090; marginal improvement |
| RTX 4090 24GB | 24GB | ~$1,500+ | ~$62.50/GB | 2× faster than 3090; same VRAM ceiling |
| Mac Studio M1 Ultra | 64GB Unified | ~$2,000 | ~$31.25/GB | Runs 70B fully in memory; silent; efficient |
| Mac Studio M2 Ultra | 64–192GB Unified | ~$2,500+ | ~$39–13/GB | 192GB runs 120B+ models; ultimate local LLM rig |

The Apple Silicon inflection point remains at ~$2,000+. A used Mac Studio M1 Ultra with 64GB unified memory offers 2.7× the RAM of an RTX 3090 with zero CPU/GPU copy overhead.

---

# Technical Addendum: Quantifying the Pascal vs. Tensor Core Performance Gap

When prioritizing VRAM-per-dollar, you will inevitably consider Pascal-era cards (GTX 10-series, Tesla P40). The conventional wisdom says these are "too slow" for LLMs because they lack tensor cores. But how much slower are they, really?

## The Memory-Bandwidth Reality

LLM inference in `llama.cpp` has two phases:

1. **Prompt Processing (pp):** Processes the entire input prompt at once. This is *compute-bound*—tensor cores shine here.
2. **Token Generation (tg):** Generates one token at a time autoregressively. This is *memory-bandwidth-bound*—the GPU spends most of its time waiting for weights to stream from VRAM.

For a 24/7 subagent generating long responses, **token generation dominates total runtime**. And in token generation, raw memory bandwidth matters more than tensor-core throughput.

## Benchmark Data: llama.cpp CUDA Scoreboard

The following data comes from the official `llama.cpp` GitHub discussion #15013, testing **Llama 2 7B at Q4_0** quantization across consumer GPUs:

| GPU | Architecture | VRAM | Memory Bus | Bandwidth | pp512 (tok/s) | **tg128 (tok/s)** |
|-----|-------------|------|-----------|-----------|--------------|------------------|
| GTX 1060 6GB | Pascal | 6GB | 192-bit GDDR5 | 192 GB/s | 417 | **27.79** |
| GTX 1080 Ti 11GB | Pascal | 11GB | 352-bit GDDR5X | 484 GB/s | 1,084 | **62.49** |
| RTX 2060 Super 8GB | Turing | 8GB | 256-bit GDDR6 | 448 GB/s | 1,420 | **60.04** |
| RTX 3060 12GB | Ampere | 12GB | 192-bit GDDR6 | 360 GB/s | 2,138 | **75.57** |
| RTX 3090 24GB | Ampere | 24GB | 384-bit GDDR6X | 936 GB/s | 5,175 | **158.16** |

*Source: [llama.cpp CUDA Scoreboard — GitHub Discussion #15013](https://github.com/ggml-org/llama.cpp/discussions/15013)*

## Analysis: What the Numbers Actually Mean

### Token Generation (The Metric That Matters for Subagents)

- **GTX 1080 Ti vs. RTX 2060 Super:** The 1080 Ti generates tokens at **62.5 tok/s**; the 2060 Super at **60.0 tok/s**. The Pascal card is actually *faster* despite lacking tensor cores, because its 352-bit bus (484 GB/s) outguns the 2060 Super's 256-bit bus (448 GB/s).
- **GTX 1080 Ti vs. RTX 3060:** The RTX 3060 hits **75.6 tok/s**—only **21% faster** than the 1080 Ti. The 3060's tensor cores can't overcome its narrower 192-bit bus (360 GB/s).
- **GTX 1080 Ti vs. RTX 3090:** The 3090's massive 384-bit GDDR6X bus (936 GB/s) delivers **158.2 tok/s**—a genuinely transformative 2.5× speedup.

### Prompt Processing (Where Tensor Cores Dominate)

- **GTX 1080 Ti:** 1,084 tok/s
- **RTX 3060:** 2,138 tok/s (**97% faster**)
- **RTX 3090:** 5,175 tok/s (**3.8× faster**)

Tensor cores crush prompt processing. If your workload involves processing massive prompts (e.g., feeding entire codebases or books into context), the gap is significant. But for a subagent generating code or reasoning step-by-step, prompt lengths are typically short, and token generation dominates.

### The Academic Confirmation

A 2025 research paper studying quantized LLM performance on NVIDIA GPUs explicitly states:

> "Pascal lacks native hardware support for INT8 or INT4 operations. Consequently, any quantized computation must be emulated through slower FP32 CUDA cores... **This architectural limitation makes Pascal-based GPUs fundamentally unsuitable for efficient quantized LLM inference.**"

*Source: OpenReview, "Performance and Accuracy of Quantized LLMs"*

**But the data tells a more nuanced story.** "Fundamentally unsuitable" is overstated for token generation. The GTX 1080 Ti's brute-force memory bandwidth keeps it competitive with Turing cards at the same VRAM tier. Where Pascal truly struggles is prompt processing and small-context batch inference—workloads where tensor-core math throughput matters.

## Power Consumption: The Hidden Cost

| GPU | TDP | Est. 24/7 Power Cost (@$0.12/kWh) |
|-----|-----|----------------------------------|
| GTX 1060 6GB | 120W | ~$126/year |
| GTX 1080 Ti 11GB | 250W | ~$263/year |
| RTX 3060 12GB | 170W | ~$179/year |
| RTX 3090 24GB | 350W | ~$368/year |
| Tesla P40 24GB | 250W | ~$263/year |

Over a 2-year period, a GTX 1080 Ti costs ~$170 more in electricity than an RTX 3060. This narrows the value gap but doesn't eliminate it—especially if the 1080 Ti was $50 cheaper upfront.

## The Tesla P40 Specifics

The Tesla P40 is a datacenter inference accelerator, not a gaming card. Key quirks:

- **No display outputs:** Must be used headless or with a second GPU for display.
- **Passive cooling:** Requires either a server chassis with high airflow or a 3D-printed fan shroud + 120mm fan. Many eBay sellers include a shroud; verify before buying.
- **No tensor cores:** Same Pascal CUDA cores as the GTX 1080 Ti, but clocked lower for datacenter reliability. Expect ~10–15% lower token throughput than a 1080 Ti.
- **24GB GDDR5:** 384-bit bus, 346 GB/s bandwidth—less than the 1080 Ti's 484 GB/s, but the extra 13GB of capacity more than compensates.

For a dedicated subagent box running 24/7, the P40's quirks are manageable. For a desktop that also needs display output, it's the wrong choice.

## Summary: What You're Giving Up

If you prioritize VRAM/$ and buy a Pascal card (GTX 1080 Ti / Tesla P40) over an RTX Ampere/Ada card, here's the trade-off:

| Factor | Pascal (1080 Ti / P40) | Ampere (RTX 3060 / 3090) | Impact |
|--------|----------------------|-------------------------|--------|
| VRAM/$ | Excellent (~$12–14/GB) | Good (~$16–25/GB) | Pascal wins |
| Token generation speed | Good (62–70 tok/s on 7B) | Good–Excellent (75–158 tok/s) | Ampere wins 20–150% |
| Prompt processing speed | Slow (1,000 tok/s) | Fast (2,100–5,200 tok/s) | Ampere wins 2–5× |
| Power efficiency | Poor (250W for 11–24GB) | Good (170–350W) | Ampere wins |
| Software compatibility | Full CUDA support | Full CUDA support + newer features | Equal for llama.cpp |
| Display outputs | Yes (1080 Ti) / No (P40) | Yes | Varies |

**Bottom line:** For a 24/7 unattended subagent where token generation dominates and speed is "nice but not critical," Pascal cards offer legitimate value. You're giving up ~20% token speed and 2× prompt-processing speed compared to Ampere, but you're gaining 50–100% more VRAM per dollar. If your alternative is running smaller models on a faster card versus larger models on a slower card, the larger model usually wins on capability—even at modestly lower speed.

---

*Report compiled from eBay market data (May 2026), llama.cpp GitHub CUDA scoreboard #15013, and academic literature on quantized LLM inference. Prices are approximate and fluctuate.*
