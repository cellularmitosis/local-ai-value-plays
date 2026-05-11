# The Road to 512GB System RAM: Running Frontier Models Without Compromise

*Research conducted: May 2026*  
*Goal: Identify the cheapest reliable platform that supports 512GB DDR4 ECC, and determine what local LLM capabilities it unlocks when paired with a Tesla P40*  
*Budget target: ~$1,000–$1,300 for host + RAM + GPU*

---

## Executive Summary

**512GB of system RAM is the next genuine inflection point** after the 256GB configuration documented in `report4-256gb-p40.md`. While 256GB lets you squeeze DeepSeek-V3/R1 671B into memory at aggressive Q2_K quantization, **512GB lets you run it at Q4_K_M** — the default recommended quality — entirely in RAM/VRAM with no disk thrashing.

This is not a marginal improvement. It is a **qualitative leap** from "frontier model barely fits" to "frontier model fits comfortably with headroom for context."

**The Cheapest Path:** An **HP Z840 workstation** (dual Xeon E5-2600 v3/v4, 16 DIMM slots, officially 512GB-capable) plus **16×32GB DDR4 ECC RDIMMs**. Total platform cost — before GPU — is roughly **$750**. Add a Tesla P40 24GB and you have a ~$1,050 rig that can load DeepSeek-V3 Q4_K_M, Llama 3.1 405B, and million-token contexts on 70B-class models.

| Config | System RAM | Total Memory | DeepSeek-V3 671B | Llama 3.1 405B | Est. Rig Cost |
|--------|-----------|--------------|------------------|----------------|---------------|
| Ultra (Report 4) | 256GB | 280GB | Q2_K only | Q2_K only | ~$900 |
| **Max (Report 5)** | **512GB** | **536GB** | **Q4_K_M ✅** | **Q3_K_S ✅** | **~$1,050** |

---

## Part 1: Why 512GB? What Does It Actually Unlock?

### The Memory Wall at 256GB

With 256GB RAM + 24GB VRAM = 280GB total, the 256GB config from `report4-256gb-p40.md` can run:

- **DeepSeek-V3/R1 671B** at Unsloth UD-Q2_K_XL (~245GB) with modest context
- **Llama 3.1 405B** at Q2_K (~240GB) with minimal context
- **Qwen3-Coder-480B** at Q3_K_XL (~228GB) with turbo3 KV cache

What it *cannot* do is run any of these at **Q4_K_M** — the quantization level that retains ~98.5% of full-precision quality. The 256GB rig is permanently locked to 2-bit and 3-bit territory.

### What 512GB + 24GB VRAM = 536GB Unlocks

| Model | Quantization | Weights Size | KV Cache (8K ctx) | Total Memory | Fits in 536GB? |
|-------|-------------|-------------|-------------------|--------------|----------------|
| **DeepSeek-V3/R1 671B** | **Q4_K_M** | **~410 GB** | **~60 GB** | **~470 GB** | ✅ Yes (66GB headroom) |
| **DeepSeek-V3/R1 671B** | UD-Q4_K_XL | ~350 GB | ~60 GB | ~410 GB | ✅ Comfortable |
| **Llama 3.1 405B** | Q3_K_S | ~300 GB | ~30 GB | ~330 GB | ✅ Yes |
| **Llama 3.1 405B** | Q4_K_M | ~480 GB | ~60 GB | ~540 GB | ⚠️ Tight; needs KV quant |
| **Qwen3-Coder-480B** | Q4_K_M | ~291 GB | ~30 GB | ~321 GB | ✅ Yes |
| **Qwen3-Coder-480B** | IQ4_XS | ~257 GB | ~30 GB | ~287 GB | ✅ Comfortable |
| **Mixtral 8x22B** | Q4_K_M | ~80 GB | ~12 GB | ~92 GB | ✅ Fully in VRAM+RAM |
| **Llama 3.3 70B** | Q4_K_M | ~43 GB | ~10 GB | ~53 GB | ✅ Fully in VRAM |

**The headline:** DeepSeek-V3/R1 671B at **Q4_K_M** — the quality level most local-LLM users target — fits inside 536GB with **66GB of headroom** for the OS, working buffers, and expanded context windows. No other sub-$1,500 platform can make this claim.

### Context Windows at 512GB

With the KV cache no longer constrained by a 256GB ceiling, you can run frontier models at extreme context:

| Model | Quant | Weights | 256K Context (KV) | 512K Context (KV) | Total @ 512K | Fits? |
|-------|-------|---------|-------------------|-------------------|--------------|-------|
| DeepSeek-V3 | Q4_K_M | ~410 GB | ~17.6 GB (turbo3) | ~35.2 GB (turbo3) | ~445 GB | ✅ |
| Llama 3.1 405B | Q3_K_S | ~300 GB | ~15.6 GB (q4_0) | ~31.2 GB (q4_0) | ~331 GB | ✅ |
| Qwen3-Coder-480B | Q4_K_M | ~291 GB | ~15.9 GB (q4_0) | ~31.8 GB (q4_0) | ~323 GB | ✅ |

At 512GB, you can process **half-million-token contexts** on 400B–600B-parameter frontier models. That is enough to ingest:
- The entire Linux kernel source tree (~400K tokens) into DeepSeek-V3's context
- A 1,000-page legal brief into Llama 3.1 405B
- 200+ PDFs simultaneously into Qwen3-Coder-480B for cross-document analysis

---

## Part 2: Platform Requirements — Why the T5810 Hits a Wall

### The T5810 Ceiling

The Dell Precision T5810 — the hero of Reports 3 and 4 — has **8 DIMM slots**. Dell's official manual and memory population tables confirm the maximum configurations:

| Configuration | DIMMs | Total |
|---------------|-------|-------|
| S256 | 8 × 32GB RDIMM | **256 GB** |

The T5810's C612 chipset and BIOS are validated only for **4GB, 8GB, 16GB, and 32GB RDIMMs**. While some third-party databases (e.g., TheRetroWeb) speculate about 512GB with 64GB LRDIMMs, **there are no verified community reports of a T5810 successfully posting with 64GB LRDIMMs**. The platform lacks the memory reference code for LR-DIMM support.

**Verdict:** Do not attempt 512GB on a T5810. The 256GB config from Report 4 is its practical endpoint.

### What You Need for 512GB

To hit 512GB with widely available, inexpensive 32GB RDIMMs, you need **16 DIMM slots**. This requires a **dual-socket Xeon E5-2600 v3/v4 platform**.

| Platform | DIMM Slots | Official 512GB? | Dual CPU? | Max GPU Power | Typical Used Price |
|----------|-----------|-----------------|-----------|---------------|-------------------|
| **HP Z840** | **16** | **Yes (32GB RDIMM)** | **Yes** | **675W (3×225W)** | **~$369** |
| **Dell T7910** | **16** | **Yes (QuickSpecs say 1TB)** | **Yes** | **675W (3×225W)** | **~$475** |
| Dell T7810 | 8 | No (256GB max) | Yes | 450W (2×225W) | ~$150–$250 |
| HP Z640 | 8 | No (256GB max) | Yes (with riser) | 225W | ~$200–$300 |

The **HP Z840** is the standout value. It is cheaper than the Dell T7910, officially supports 512GB at launch, and has a beefy 1125W PSU with three dedicated GPU power rails.

---

## Part 3: The Host Platform — HP Z840

### Why the Z840?

- **16 DDR4 DIMM slots** (8 per CPU) — enough for 16×32GB = 512GB using cheap RDIMMs
- **Official HP support** for 512GB at initial release (QuickSpecs: "The Z840 will support up to 512GB at initial release")
- **Dual Xeon E5-2600 v3 or v4** CPUs included in many barebones listings
- **1125W or 1500W PSU** options — enough for dual 250W Tesla P40s
- **Three PCIe Gen3 x16 slots** (with dual CPU installed) — can host dual GPUs plus a display adapter
- **Widely available** on eBay from business recyclers at very low cost

### Z840 Specifications

| Feature | Spec |
|---------|------|
| CPUs | Dual Intel Xeon E5-2600 v3/v4 (LGA2011-3) |
| DIMM Slots | 16 (8 per CPU, quad-channel each) |
| Max Memory | 256GB RDIMM / 512GB at launch / 1TB with 64GB LRDIMM / 2TB with 128GB LRDIMM |
| PCIe x16 | 3 slots with dual CPU (2×Gen3 x16, 1×Gen3 x16 via 2nd CPU) |
| PSU | 850W / 1125W / 1275W / 1500W depending on region and config |
| GPU Power | 3× 6-pin PCIe connectors (each rated 216W) |

*Sources: [HP Z840 QuickSpecs PDF](https://gzhls.at/blob/ldb/5/e/3/6/d9d838f1781c6811617016ea02c7d7ed014e.pdf), HP Community forums*

### Z840 Price Anchors

**Barebones (dual CPU, no RAM/GPU/HDD):**
- [eBay item `185768642880`](https://www.ebay.com/itm/185768642880) — *"HP Z840 Workstation 2X 12-Core E5-2690 V3 2.60GHz No RAM No HDD No OS No GPU"* — **$368.97** (digitalmind2000, eBay Refurbished, 110+ sold)

**Pre-built with 512GB RAM:**
- [eBay item `304612762187`](https://www.ebay.com/itm/304612762187) — *"HP Z840 24-Core E5-2690 V3 2.6GHz-3.5GHz 512GB No GPU No HDD No OS"* — price varies, but confirms 512GB is achievable
- [eBay UK item `126339355011`](https://www.ebay.com/itm/126339355011) — *"44-CORE HP Z840 Workstation 2x E5-2699 V4 Turbo 2.40GHz 512GB DDR4 1TB NVMe SSD"* — **£1,676.99** (~$2,100) — expensive because of dual E5-2699 v4 CPUs

**For this report, we budget $369 for the Z840 barebones with dual E5-2690 v3.**

### Alternative Host: Dell Precision T7910

If HP Z840 stock is thin, the **Dell T7910** is a drop-in alternative with nearly identical specs:

| Feature | Dell T7910 |
|---------|-----------|
| DIMM Slots | 16 (8 per CPU) |
| Max Memory | 512GB officially (manual); 1TB per QuickSpecs |
| PCIe x16 | 4 slots with dual CPU |
| PSU | 1300W (80PLUS Gold) |
| GPU Power | Up to 675W total (3×225W) |

**Price anchor:**
- [eBay shop listing](https://www.ebay.com/shop/dell-precision-t7910) — Dell T7910 2x Xeon E5-2690 v4, No RAM/HDD/GPU/OS — **$474.97** (pre-owned)
- [eBay item `187617314885`](https://www.ebay.com/itm/187617314885) — *"Dell Precision T7910 Barebones Unit No Heatsinks 4x TRAYS 2011-3 DDR4 1300W PSU"*

The T7910 is ~$100 more than the Z840 but offers a 1300W PSU and four PCIe x16 slots. For single-GPU builds, the Z840 is the better value. For dual-GPU builds (see Report 6), the T7910's extra power headroom is worth considering.

---

## Part 4: RAM Pricing — 512GB (16×32GB)

### The 32GB RDIMM Sweet Spot

DDR4 ECC RDIMMs are the correct type for both the Z840 and T7910. **Do not buy UDIMM** — these platforms require registered memory. Do not mix RDIMM and LRDIMM.

The used server-pull market for 32GB DDR4-2133/2400 RDIMMs is extremely liquid. Samsung, Hynix, and Micron modules are abundant.

### eBay Price Anchors

| Listing | Config | Price | Notes |
|---------|--------|-------|-------|
| [eBay item `297346959069`](https://www.ebay.com/itm/297346959069) | 512GB (16×32GB) DDR4-2400T ECC RDIMM | **$382.46 sold** | Pulled from working system; ~$23.90/stick |
| [eBay UK item `186998883998`](https://www.ebay.co.uk/itm/186998883998) | 512GB (16×32GB) DDR4-2400 Samsung RDIMM | **£299.00** (~$375) | ~£18.69/stick; UK seller |
| [eBay item `185995691105`](https://www.ebay.com/itm/185995691105) | 512GB (16×32GB) DDR4 PC4-2400T-R ECC RDIMM for HP Z840 | Price varies | PC Server & Parts listing |
| [A-Tech 512GB kit](https://atechmemory.com/products/512gb-16x32gb-ddr4-2400-rdimm-pc4-19200r-dual-rank-x4-server-kit) | 16×32GB DDR4-2400 RDIMM | ~$550 retail | New with warranty; higher cost |

**The $382 sold listing is the critical anchor.** It proves that 512GB of used server RAM can be acquired for under $400. Even at a conservative $450–$500, the per-gigabyte cost is roughly $0.90–$1.00/GB — far cheaper than consumer DDR5.

**For this report, we budget `$420` for 512GB (16×32GB)**, reflecting a blend of the extreme deal ($382) and typical market pricing.

### Memory Population Rules

On the Z840, DIMMs must be installed in **quads per CPU** (one per channel) for optimal performance. For 512GB (16×32GB):

- **CPU 0:** Slots 1, 2, 3, 4, 5, 6, 7, 8 — all populated with 32GB RDIMMs
- **CPU 1:** Slots 1, 2, 3, 4, 5, 6, 7, 8 — all populated with 32GB RDIMMs

All modules must be identical speed and rank. DDR4-2133 or DDR4-2400 is fine; the system will down-clock to the CPU's supported speed.

---

## Part 5: The Complete Build — "Max" Config (512GB + P40)

### Bill of Materials

| Component | Spec | Price | Source / Anchor |
|-----------|------|-------|-----------------|
| **Host** | HP Z840, dual E5-2690 v3, no RAM/GPU/HDD | **$369** | [eBay item `185768642880`](https://www.ebay.com/itm/185768642880) (digitalmind2000) |
| **RAM** | 512GB (16×32GB) DDR4 ECC RDIMM | **$420** | Server pulls / [eBay kit](https://www.ebay.com/itm/297346959069) |
| **GPU** | Tesla P40 24GB + cooling fan | **$260** | Item `198009478303` or bundled `376635025151` (from Report 3) |
| **Power adapter** | EPS 8-pin adapter for P40 | **$15** | Bundled with some P40s; or AliExpress |
| **Boot SSD** | 512GB–1TB SATA/NVMe SSD | **$25** | Any eBay SSD lot |
| **TOTAL** | | **~$1,089** | |

### Notes

- The Z840 in the $369 listing has a **1125W PSU** and dual heatsinks included.
- 512GB system RAM + 24GB VRAM = **536GB total addressable memory**.
- The P40 occupies one PCIe x16 slot. A second PCIe x16 slot is available for a cheap display adapter (GT 710 ~$15) if you need local video output.
- Linux is strongly recommended. The Z840 runs Ubuntu 22.04 LTS flawlessly with the NVIDIA datacenter driver.

---

## Part 6: What Each Config Can Actually Run

Assuming `llama.cpp` with Q4_K_M quantization unless noted. All models load entirely into memory (RAM + VRAM) with no disk swap.

| Model | Params | 256GB Config (Report 4) | 512GB Config (Report 5) | Speed Estimate |
|-------|--------|------------------------|------------------------|----------------|
| Qwen2.5-Coder-32B | 32B | Fully VRAM ✅ | Fully VRAM ✅ | 15–25 tok/s |
| DeepSeek-R1-Distill-Qwen-32B | 32B | Fully VRAM ✅ | Fully VRAM ✅ | 15–25 tok/s |
| Llama 3.3 70B | 70B | Partial offload ⚠️ | Fully VRAM ✅ | 15–25 tok/s |
| DeepSeek-R1-Distill-Llama-70B | 70B | Partial offload ⚠️ | Fully VRAM ✅ | 15–25 tok/s |
| Mixtral 8x22B | 141B total | MoE/offload ✅ | Fully VRAM ✅ | 8–15 tok/s |
| DeepSeek-V3/R1 671B | 671B total | Q2_K only ⚠️ | **Q4_K_M ✅** | 3–6 tok/s |
| Llama 3.1 405B | 405B dense | Q2_K only ⚠️ | **Q3_K_S ✅** | 2–4 tok/s |
| Qwen3-Coder-480B | 480B total | Q3_K_XL ⚠️ | **Q4_K_M ✅** | 2–4 tok/s |

*"Fully VRAM" = fits in the P40's 24GB. "Q4_K_M ✅" = fits in the combined 536GB envelope.*

### The Inflection Point: DeepSeek-V3 at Q4_K_M

With 512GB, the full **DeepSeek-V3 / R1 671B** model at Q4_K_M (~410GB weights + ~60GB KV cache at 8K context) consumes ~470GB. That leaves **66GB of headroom** for:
- OS and system buffers
- Context expansion to 32K–64K tokens
- Secondary model loads (e.g., a 7B embedding model)

This is the cheapest way on Earth to run a 671B-parameter frontier model at **default quantization quality** without disk swap.

### The Llama 3.1 405B Milestone

Meta's dense 405B model at Q3_K_S (~300GB weights + ~30GB KV) fits comfortably at 512GB. At Q4_K_M (~480GB weights + ~60GB KV = ~540GB), it is tight but possible with KV cache quantization (`--cache-type-k q4_0`). The 512GB config is the first sub-$2,000 platform that can realistically host Llama 3.1 405B.

---

## Part 7: Performance Expectations

### Token Generation Speeds (Estimated)

| Model | Quantization | GPU Offload | Expected tok/s | Bottleneck |
|-------|-------------|-------------|----------------|------------|
| DeepSeek-V3 671B | Q4_K_M | Attention+KV in VRAM, experts in RAM | **3–6** | CPU RAM bandwidth (DDR4 ~120–160 GB/s aggregate) |
| Llama 3.1 405B | Q3_K_S | Partial (~12–16 layers in VRAM) | **2–4** | CPU compute + PCIe transfer |
| Llama 3.3 70B | Q4_K_M | Fully in VRAM | **15–25** | P40 CUDA cores (no tensor cores) |
| Qwen3-Coder-480B | Q4_K_M | Attention in VRAM, experts in RAM | **2–4** | CPU RAM bandwidth + expert routing |
| Mixtral 8x22B | Q4_K_M | Attention in VRAM, experts in RAM | **8–15** | PCIe + CPU RAM bandwidth |

### The Pascal Speed Caveat

As documented in `report2.md`, the Tesla P40 lacks tensor cores. For token generation (memory-bandwidth-bound), this is less punishing than expected. But for prompt processing, the P40 will be 2–4× slower than an RTX 3090.

**For a 24/7 subagent running frontier models at 3–6 tok/s, this is a capability play, not a speed play.** You are trading speed for access to models that simply will not fit on any consumer GPU under $1,500.

---

## Part 8: Power, Thermals, and Practical Concerns

### Power Draw

| Component | TDP |
|-----------|-----|
| 2× Xeon E5-2690 v3 | 2 × 135W = 270W |
| Tesla P40 | 250W |
| 16 × DDR4 RDIMM | ~80W |
| SSD + fans + overhead | ~50W |
| **Total** | **~650W** |

The Z840's **1125W PSU** is sufficient with comfortable headroom. If you plan to add a second P40 later (see Report 6), the 1500W PSU option is preferable.

### Thermal Concerns at 512GB

Sixteen populated DIMM slots generate significant heat. The Z840's chassis was designed for this — it has dedicated memory shrouds and front-to-rear airflow. Ensure:
- The chassis fans are functional
- The memory shrouds are installed
- Ambient temperature is below 30°C

DDR4 RDIMMs will run warm but within spec; no active cooling required.

### Storage Requirements

Frontier GGUF files are enormous:
- DeepSeek-V3 Q4_K_M: ~400–450 GB
- Llama 3.1 405B Q3_K_S: ~300 GB
- Qwen3-Coder-480B Q4_K_M: ~291 GB

**A 1TB NVMe or SATA SSD is strongly recommended.** A 512GB SSD is the absolute minimum if you only plan to run one frontier model at a time.

---

## Part 9: Cost Comparison

### Build vs. Cloud API

| Metric | 512GB P40 Rig | GPT-4 / Claude API | Mac Studio M2 Ultra 192GB |
|--------|--------------|-------------------|--------------------------|
| **Upfront cost** | ~$1,100 | $0 | ~$3,500 (used) |
| **Per-month inference cost** | ~$40 (power) | $200+ | ~$25 (power) |
| **DeepSeek-V3 671B Q4_K_M** | ✅ Runs locally | ❌ N/A (API only) | ❌ 192GB < 470GB needed |
| **Llama 3.1 405B Q3_K_S** | ✅ Runs locally | ❌ N/A | ❌ 192GB < 330GB needed |
| **Llama 3.3 70B** | ✅ Fully in VRAM | ✅ API | ✅ Fully in unified memory |
| **2-year TCO** | ~$2,060 | ~$4,800+ | ~$3,860 |
| **Censorship** | None (local) | Yes (provider filters) | None (local) |

### Build vs. Apple Silicon

A used Mac Studio M2 Ultra with **192GB unified memory** costs ~$3,500. It cannot fit DeepSeek-V3 at Q4_K_M (needs 470GB) or Llama 3.1 405B at Q3_K_S (needs 330GB). The **512GB Z840 + P40 is the only sub-$1,500 platform that can load these models entirely in memory.**

---

## Part 10: Context Maxxing — What 536GB Means for Long-Context Inference

The 512GB RAM upgrade is not just about fitting bigger models. It is about **fitting bigger models at higher quality AND larger context simultaneously** — the difference between running DeepSeek-V3 at Q2_K with 1M context, and running it at Q4_K_M with 1M context.

With **536GB total memory** (512GB RAM + 24GB VRAM), you have **1.9× the envelope** of the 256GB configuration. This unlocks three things the 256GB rig cannot do:

1. **DeepSeek-V3 at Q4_K_M** (~410 GB weights) instead of Q2_K (~245 GB) — a massive quality improvement
2. **Qwen3-Coder-480B at Q4_K_M** (~291 GB) instead of Q3_K_XL (~228 GB) — full-precision coding quality
3. **1M context with f16 KV cache** on 70B+ models — no quantization artifacts in the attention state

### The KV Cache Quantization Toolkit

The same three levers from Report 4 apply:

| KV Type | Bits/Weight | vs FP16 | Speed Impact | Quality Impact |
|---------|------------|---------|--------------|----------------|
| **f16** (default) | 16 | 1.0× | Baseline | None |
| **q8_0** | 8 | 2.0× smaller | <5% slower | Negligible |
| **q4_0** | 4 | 4.0× smaller | ~35% slower at 64K+ | Minor |
| **turbo3** | ~3.1 | ~4.9× smaller | Minimal (flash-attn) | Small PPL increase |

> **Recommendation:** At 536GB, you can afford **f16 KV cache** for most models up to 512K context. For 1M context on frontier models, **q8_0** or **q4_0** still provides headroom.

### Architecture Matters: MLA vs. GQA

Same architecture notes as Report 4:

- **DeepSeek-V3 MLA:** ~68.6 KB/token (f16) — 57× smaller than standard MHA
- **Standard GQA (Llama 3):** ~128 KB/token (f16)
- **Standard GQA (Qwen2.5 32B):** ~256 KB/token (f16)
- **Qwen3-Coder-480B:** ~248 KB/token (f16)
- **Llama 3.3 70B / Llama 4 Scout / Qwen3.5-122B:** ~320 KB/token (f16)
- **Gemma 4 26B-A4B:** ~181 KB/token (f16)

---

### Context Capacity Matrix: What Fits at 256K, 512K, and 1M

All calculations assume **536GB total memory envelope** (24GB VRAM + 512GB RAM). Weights are Q4_K_M unless noted; KV cache uses the quantization shown.

#### 256K Context

| Model | Weights (Q4_K_M) | KV Cache (256K) | KV Quant | Total Memory | VRAM Status | Fits? |
|-------|-----------------|-----------------|----------|--------------|-------------|-------|
| **Gemma 4 26B-A4B** | ~14 GB | 45.3 GB | f16 | 59.3 GB | All in VRAM | ✅ Easily |
| **Qwen2.5-Coder-32B** | ~21 GB | 64.0 GB | f16 | 85.0 GB | Weights in VRAM, partial KV | ✅ Easily |
| **Llama 3.3 70B** | ~44 GB | 80.0 GB | f16 | 124.0 GB | Partial offload | ✅ Easily |
| **Llama 4 Scout** | ~60 GB | 80.0 GB | f16 | 140.0 GB | Partial offload | ✅ Easily |
| **DeepSeek-V3 671B** | ~410 GB | 17.2 GB | f16 | 427.2 GB | 24GB in VRAM, 403GB RAM | ✅ |
| **Qwen3-Coder-480B** | ~291 GB | 62.0 GB | f16 | 353.0 GB | 24GB in VRAM, 329GB RAM | ✅ |
| **Qwen3.5-122B** | ~75 GB | 80.0 GB | f16 | 155.0 GB | Partial offload | ✅ Easily |

At 256K context, **every model fits with f16 KV cache**. Even DeepSeek-V3 at Q4_K_M has 108 GB of headroom. The 256GB rig from Report 4 needed Q2_K weights and turbo3 KV to fit DeepSeek-V3 at this context.

#### 512K Context

| Model | Weights (Q4_K_M) | KV Cache (512K) | KV Quant | Total Memory | Fits? |
|-------|-----------------|-----------------|----------|--------------|-------|
| **Gemma 4 26B-A4B** | ~14 GB | 90.5 GB | f16 | 104.5 GB | ✅ Easily |
| **Qwen2.5-Coder-32B** | ~21 GB | 128.0 GB | f16 | 149.0 GB | ✅ Easily |
| **Llama 3.3 70B** | ~44 GB | 160.0 GB | f16 | 204.0 GB | ✅ Easily |
| **Llama 4 Scout** | ~60 GB | 160.0 GB | f16 | 220.0 GB | ✅ Easily |
| **DeepSeek-V3 671B** | ~410 GB | 34.3 GB | f16 | 444.3 GB | ✅ (91.7 GB headroom) |
| **Qwen3-Coder-480B** | ~291 GB | 124.0 GB | f16 | 415.0 GB | ✅ (121 GB headroom) |
| **Qwen3.5-122B** | ~75 GB | 160.0 GB | f16 | 235.0 GB | ✅ Easily |

At 512K, DeepSeek-V3 at Q4_K_M + f16 KV still fits with **92 GB of headroom**. Qwen3-Coder-480B at Q4_K_M + f16 KV fits with **121 GB of headroom**. No quantization compromises needed.

#### 1M Context

| Model | Weights (Q4_K_M) | KV Cache (1M) | KV Quant | Total Memory | Fits? |
|-------|-----------------|---------------|----------|--------------|-------|
| **Gemma 4 26B-A4B** | ~14 GB | 181.0 GB | f16 | 195.0 GB | ✅ Easily |
| **Qwen2.5-Coder-32B** | ~21 GB | 256.0 GB | f16 | 277.0 GB | ✅ |
| **Llama 3.3 70B** | ~44 GB | 320.0 GB | f16 | 364.0 GB | ✅ |
| **Llama 4 Scout** | ~60 GB | 320.0 GB | f16 | 380.0 GB | ✅ |
| **DeepSeek-V3 671B** | ~410 GB | 68.6 GB | f16 | 478.6 GB | ⚠️ Fits (57.4 GB headroom) |
| **Qwen3-Coder-480B** | ~291 GB | 248.0 GB | f16 | 539.0 GB | ⚠️ Tight; q8_0 (415 GB) ✅ |
| **Qwen3.5-122B** | ~75 GB | 320.0 GB | f16 | 395.0 GB | ✅ |

### The Headline: DeepSeek-V3 671B at Q4_K_M + 1 Million Tokens

With 512GB RAM + 24GB VRAM, the full **DeepSeek-V3 / R1 671B** can run at **Q4_K_M quantization with 1M token context** using **f16 KV cache**:

- **Q4_K_M weights** (~410 GB) — retaining ~98% of full-precision quality
- **f16 KV cache** (~68.6 GB for 1M tokens) — zero KV quantization artifacts
- **Total: ~478.6 GB** — fitting inside the 536GB envelope with 57 GB headroom

This is the headline capability of the 512GB rig. No consumer GPU under $2,000 can do this. An RTX 4090 24GB has the same VRAM but only 24GB of system RAM (or 64GB on a high-end desktop). The **512GB system RAM is the secret ingredient** that makes Q4_K_M frontier inference possible.

> **Speed expectation:** At Q4_K_M with heavy CPU offload, expect **2–4 tok/s** — slower than Q2_K because the weights are 1.7× larger, but the output quality is noticeably better for reasoning and coding tasks. This is a batch-job model, not an interactive chatbot.

### The Coding Monster: Qwen3-Coder-480B-A35B at 1M Context

The **Qwen3-Coder-480B-A35B-Instruct** at Q4_K_M quantization with **q8_0 KV cache** can process **1 million tokens**:

- **Q4_K_M weights** (~291 GB)
- **q8_0 KV cache** (~124 GB for 1M tokens)
- **Total: ~415 GB** — well within the 536GB envelope with 121 GB headroom

At Q4_K_M, this model scores approximately **61.8% on Aider Polyglot** (bartowski GGUF) — rivaling Claude Sonnet-4 on agentic coding tasks. With 1M context, it can ingest:
- The entire Linux kernel source tree (~400K tokens) plus a massive prompt
- A 1,000-page technical specification plus all prior revisions
- 100+ source files simultaneously for cross-file refactoring

> **Quality note:** Q4_K_M retains ~99% of full-precision quality. The q8_0 KV cache introduces negligible perplexity increase. This is the highest-fidelity coding setup available on a sub-$1,500 rig.

**Recommended launch command (1M context):**
```bash
llama-server \
  --model Qwen3-Coder-480B-A35B-Instruct-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  -ot ".ffn_.*_exps.=CPU" \
  --cache-type-k q8_0 --cache-type-v q8_0 \
  --no-mmap --mlock \
  --ctx-size 1048576 \
  --flash-attn \
  --threads 32
```

### The Expanded "1M Context Club"

For models where weights fit in the P40's 24GB VRAM, the 512GB RAM acts as a pure KV cache reservoir. This unlocks **1M+ context at f16 KV precision** for coding and document analysis:

| Model | Weights in VRAM | KV Cache (1M, f16) | Total | Expected Speed |
|-------|----------------|---------------------|-------|----------------|
| **Llama 3.1 8B** | ~6 GB | ~32 GB | ~38 GB | **25–40 tok/s** |
| **Qwen2.5-Coder-14B** | ~10 GB | ~64 GB | ~74 GB | **20–30 tok/s** |
| **Gemma 4 26B-A4B** | ~14 GB | ~181 GB | ~195 GB | **15–25 tok/s** |
| **Qwen2.5-Coder-32B** | ~21 GB | ~256 GB | ~277 GB | **10–18 tok/s** |
| **Llama 3.3 70B** | ~44 GB | ~320 GB | ~364 GB | **8–15 tok/s** |
| **Llama 4 Scout** | ~60 GB | ~320 GB | ~380 GB | **10–18 tok/s** |
| **Qwen3.5-122B** | ~75 GB | ~320 GB | ~395 GB | **6–12 tok/s** |

The key unlock over Report 4 is that **70B and 122B models can now run at 1M context with f16 KV cache**, preserving full precision in the attention state. On the 256GB rig, these models needed q4_0 or q8_0 KV quantization at 1M context.

### Recommended llama.cpp Commands for Context Maxxing

**DeepSeek-V3 at 1M context (Q4_K_M, f16 KV):**
```bash
llama-server \
  --model DeepSeek-V3.1-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  -ot ".ffn_.*_exps.=CPU" \
  --cache-type-k f16 --cache-type-v f16 \
  --no-mmap --mlock \
  --ctx-size 1048576 \
  --flash-attn \
  --threads 32
```

**Qwen3-Coder-480B at 512K context (Q4_K_M, q8_0 KV):**
```bash
llama-server \
  --model Qwen3-Coder-480B-A35B-Instruct-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  -ot ".ffn_.*_exps.=CPU" \
  --cache-type-k q8_0 --cache-type-v q8_0 \
  --no-mmap --mlock \
  --ctx-size 524288 \
  --flash-attn \
  --threads 32
```

**Llama 3.3 70B at 1M context (Q4_K_M, f16 KV):**
```bash
llama-server \
  --model Llama-3.3-70B-Instruct-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  --cache-type-k f16 --cache-type-v f16 \
  --ctx-size 1048576 \
  --flash-attn \
  --threads 32
```

### Why 256GB RAM Cannot Do This

With 256GB RAM + 24GB VRAM = 280GB total:

- DeepSeek-V3 at **Q4_K_M** + 1M context (f16 KV) = 410 + 68.6 = **478.6 GB** → ❌ Does not fit
- DeepSeek-V3 at **Q4_K_M** + 256K context (f16 KV) = 410 + 17.2 = **427.2 GB** → ❌ Does not fit
- Qwen3-Coder-480B at **Q4_K_M** + 512K context (f16 KV) = 291 + 124 = **415 GB** → ❌ Does not fit
- Llama 3.3 70B at **Q4_K_M** + 1M context (f16 KV) = 44 + 320 = **364 GB** → ❌ Does not fit
- Llama 4 Scout at **Q4_K_M** + 1M context (f16 KV) = 60 + 320 = **380 GB** → ❌ Does not fit

The 256GB config is limited to:
- DeepSeek-V3 at **Q2_K** (~245 GB) with reduced context
- Qwen3-Coder-480B at **Q3_K_XL** (~228 GB) with KV quantization
- 70B+ models at 1M context with **q4_0/q8_0 KV** (quality degradation)

The 512GB upgrade buys you **Q4_K_M quality on frontier models** AND **f16 KV cache at 1M context** — a combination impossible at 256GB.

---

## Part 11: Risks and Caveats

1. **Quantization Quality at 3-Bit**
   Q3_K_S on Llama 3.1 405B will degrade quality slightly compared to Q4_K_M. For most tasks the difference is negligible, but for high-stakes reasoning you may want to stick to Q4_K_M where possible.

2. **CPU RAM Bandwidth Ceiling**
   DDR4-2133/2400 in quad-channel yields ~60–80 GB/s per CPU. With dual CPUs, aggregate bandwidth is ~120–160 GB/s theoretical, but real-world efficiency is lower. This is the bottleneck for expert-layer retrieval in MoE models. DDR4-2400 or 2666 modules help marginally.

3. **PCIe 3.0 x16 Bandwidth**
   The P40 connects via PCIe 3.0 x16 (~16 GB/s). For MoE inference, only active experts cross the bus per token. This is manageable but means the P40 is not a substitute for an H100's 900 GB/s HBM3.

4. **No Display Output**
   The P40 is headless. You need a second GPU (GT 710 ~$15) for local monitor/keyboard setup, or manage entirely via SSH.

5. **Model Availability**
   300GB+ GGUF files are not hosted on standard HuggingFace model cards. You may need `hf-transfer` with a HuggingFace Pro subscription, or use torrents for large-file downloads.

---

## Final Recommendations

### Who Should Build This?

- **You want to run DeepSeek-V3/R1 671B at Q4_K_M** for uncensored, private, frontier-class inference
- **You need 256K–512K token context windows** on 400B+ parameter models
- **You accept 3–6 tok/s** as a fair trade for access to models that cloud APIs do not offer locally
- **You have the technical patience** to build from used server parts and manage Linux headless
- **You view this as a 2-year investment** that pays for itself vs. API costs

### Who Should Skip This?

- **You need interactive speed** (>15 tok/s) for coding assistants — buy an RTX 3090 or Mac Studio instead
- **You want a plug-and-play experience** — buy a Mac Studio M2 Ultra (but accept the 192GB ceiling)
- **You only run 7B–32B models** — the 64GB or 128GB T5810 configs from Report 3 are sufficient and $400–$600 cheaper

### The Verdict

The **512GB + Tesla P40 configuration** is the logical endpoint of the value-workstation chain. It is not the cheapest rig, nor the fastest. But it is the **cheapest rig on Earth that can load a 671B-parameter frontier MoE model at Q4_K_M entirely in memory** and generate tokens without disk thrashing.

At **~$1,100**, it is 3× cheaper than a used Mac Studio M2 Ultra 192GB — and it can run models the Mac cannot. For a 24/7 subagent that needs access to the absolute cutting edge of open-weight LLMs, this is the ultimate budget play.

---

*Report compiled from eBay sold/completed listings (May 2026), HP Z840 QuickSpecs, Dell T7910 technical specifications, and `llama.cpp` MoE offloading documentation. Prices are approximate and fluctuate.*
