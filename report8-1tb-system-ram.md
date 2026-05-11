# Report 8: The 1TB System RAM Path — LRDIMM Deep-Dive

**Date:** May 2026
**Configuration:** 1TB system RAM + Tesla P40(s) on HP Z840
**Total addressable memory:** ~1,048 GB (single P40) or ~1,072 GB (dual P40)

---

## Part 1: Why 1TB? What Does It Actually Unlock Beyond 512GB?

The 512GB configuration from Report 5 is already a frontier-class rig. It runs DeepSeek-V3 671B at Q4_K_M with 1M context — something no consumer GPU under $2,000 can match. So why go to 1TB?

Because **512GB is the ceiling of what RDIMMs can deliver**. To go further, you need **LRDIMMs** — Load-Reduced DIMMs — and that technology shift unlocks three genuinely new capability tiers:

### 1. Llama 3.1 405B at Q4_K_M + Context

The 405B dense model at Q4_K_M weighs ~236 GB. On a 512GB rig, it fits with room for ~256K context at f16 KV, but 1M context requires q8_0 KV and leaves no headroom for other tasks. At 1TB, the same model fits with **800+ GB of headroom** — enough for 1M context at f16 precision, or 2M context at q8_0, while leaving room for a second model.

### 2. DeepSeek-V3 at Q4_K_M + 2M Context

At 512GB, DeepSeek-V3 Q4_K_M + 1M context fits with 57 GB headroom. At 1TB, you can stretch to **2M tokens** (~137 GB KV at f16) with over 500 GB of headroom. This is enough context to ingest multiple full-length books, 100K+ lines of codebase, or entire legal case files.

### 3. Two Frontier Models Simultaneously

With 1TB, you can load **both** DeepSeek-V3 Q4_K_M (~410 GB) **and** Qwen3-Coder-480B Q4_K_M (~291 GB) at the same time — a combined 701 GB of weights, with 300+ GB remaining for KV cache on both. You can even load DeepSeek-V3 (~410 GB) + Llama 3.1 405B (~236 GB) = ~646 GB, leaving 400+ GB for context. This enables multi-model agentic workflows that are impossible on 512GB.

> **Bottom line:** 1TB is not a marginal upgrade. It is the threshold where you stop worrying about "which single frontier model fits" and start asking "how many frontier models can I run at once?"

---

## Part 2: The LRDIMM Requirement — Technology, Compatibility, and Tradeoffs

### What Is an LRDIMM?

**LRDIMM** (Load-Reduced DIMM) is a server memory technology that uses a buffer chip to reduce the electrical load on the memory controller. This allows the system to address higher-capacity modules — up to 128GB per DIMM on DDR4 — at the cost of slightly higher latency.

| Feature | RDIMM | LRDIMM |
|---------|-------|--------|
| Max per-module (DDR4) | 32 GB | 128 GB |
| Buffer chip | Register only | Register + data buffer |
| Latency vs. RDIMM | Baseline | ~1–2 ns higher (~5–10%) |
| Bandwidth | Full | ~95–98% of RDIMM |
| Price per GB | Lower | ~2–3× higher |
| Compatibility | Widespread | Requires LRDIMM-supporting platform |

### The Z840 LRDIMM Rules

From HP's official Z840 specifications:

1. **Dual CPUs required for 1TB.** A single-CPU Z840 maxes at 512GB (with 64GB LRDIMMs) or 256GB (with 32GB RDIMMs). You need both processors installed to use all 16 DIMM slots.
2. **Do not mix RDIMMs and LRDIMMs.** The system will not boot. If upgrading from 512GB RDIMMs, you must replace the entire memory population.
3. **Speed downclocking.** LRDIMMs will run at the speed of the slowest installed CPU or DIMM:
   - E5-2600 v3 (Haswell): max 2133 MT/s
   - E5-2600 v4 (Broadwell): max 2400 MT/s
   - PC4-2400 LRDIMMs will downclock to 2133 MT/s with v3 CPUs
4. **All DIMM slots must be populated for optimal bandwidth.** The Z840 has 8 memory channels (4 per CPU) with 2 slots per channel. For maximum quad-channel bandwidth per CPU, install DIMMs in all 16 slots.

> **LRDIMM latency impact:** For AI inference, the ~5–10% latency increase is negligible compared to the PCIe bottleneck of CPU-offloaded expert layers. The bandwidth reduction (~2–5%) is similarly minor. The real tradeoff is **cost per GB** — LRDIMMs are 2–3× more expensive than RDIMMs.

*Sources: [HP Z840 Service Manual](https://h10032.www1.hp.com/ctg/Manual/c04823811.pdf), [HP Z840 Workstation Technical Specifications](https://www.bhphotovideo.com/lit_files/113010.pdf)*

---

## Part 3: The Host Platform — HP Z840 (Dual-CPU Required)

The same platform from Reports 5 and 6, but with a critical requirement:

### Dual CPUs Are Mandatory for 1TB

| Config | Max Memory | CPU Requirement | Notes |
|--------|-----------|-----------------|-------|
| Single CPU | 256 GB (RDIMM) / 512 GB (LRDIMM) | 1× E5-2600 v3/v4 | 8 DIMM slots active |
| **Dual CPU** | **512 GB (RDIMM) / 1 TB (LRDIMM)** | **2× E5-2600 v3/v4** | **16 DIMM slots active** |

**Implication:** If your Z840 barebones comes with a single CPU, you need a second one. Dual E5-2680 v4 (14c/28t each, 2.4 GHz base) costs ~$60–$80 on eBay. This is a minor incremental cost compared to the RAM.

### Z840 Specs Recap (from Report 5)

| Spec | Detail |
|------|--------|
| DIMM slots | 16 (8 per CPU) |
| Max memory | 1 TB with 64GB LRDIMMs / 2 TB with 128GB LRDIMMs |
| PCIe x16 | 3 slots (ideal for dual P40 + display GPU) |
| PSU | 1125W (supports dual 250W P40s + CPUs + drives) |
| CPUs | Dual Xeon E5-2600 v3/v4, up to 22c/44t each |

### Dell T7910 Alternative

The Dell T7910 also supports 1TB with 16×64GB LRDIMMs and offers **4× PCIe x16 slots** — useful if you ever want to add a third P40 (72GB VRAM). However, T7910 barebones cost ~$100–$200 more than the Z840.

*eBay anchor:* [Dell T7910 barebones ~$450–$550](https://www.ebay.com/sch/i.html?_nkw=dell+t7910+barebones)

---

## Part 4: RAM Pricing — 64GB LRDIMM eBay Survey

Pricing 64GB DDR4 LRDIMMs is more complex than 32GB RDIMMs because the market is thinner and counterfeit/scam listings are more common.

### Price Anchors (May 2026)

| Module | Speed | Condition | Price | Seller / Notes |
|--------|-------|-----------|-------|----------------|
| **Micron 64GB PC4-25600 (DDR4-3200)** | 3200 MT/s | Used | ~$59 | [Cloud Storage Corp](https://www.ebay.com/itm/186732551535) — reputable server reseller |
| **HP 64GB PC4-23400 (DDR4-2933)** | 2933 MT/s | New pulls | ~$110 | [Columbus seller](https://www.ebay.com/itm/326347210675) — 12+ sold |
| **Samsung 64GB PC4-19200 (DDR4-2400)** | 2400 MT/s | Open box | ~$88–$140 | Various sellers |
| **Hynix 64GB PC4-2400T (DDR4-2400)** | 2400 MT/s | Pre-owned | ~$150 | [eBay shop listing](https://www.ebay.com/shop/64gb-ddr4-ecc) |
| **256GB kit (4×64GB) DDR4-2666** | 2666 MT/s | Used | ~$409 (~$102/ea) | [German seller](https://www.ebay.com/itm/356189580634) |
| **512GB kit (8×64GB) DDR4-3200** | 3200 MT/s | New | ~$87 suspiciously low | Likely scam/placeholder — **avoid** |

> **Scam warning:** Listings under ~$40 for a 64GB LRDIMM are almost always counterfeit, auction bait, or "photo shows kit, price is per module" scams. Stick to sellers with 500+ feedback and 99%+ ratings.

### Realistic 1TB Pricing

| Source | Price per Module | 16-Module Total | Notes |
|--------|-----------------|-----------------|-------|
| **Low-end** (used pulls, generic) | ~$60 | **~$960** | Requires careful seller vetting |
| **Mid-range** (reputable reseller) | ~$80 | **~$1,280** | Samsung/Hynix/Micron pulls, tested |
| **High-end** (OEM-branded, new) | ~$110 | **~$1,760** | HP/Dell part numbers, warranty |

**Recommended budget:** ~$1,200 for 16×64GB from a reputable server reseller. This is **3× the cost** of the 512GB (16×32GB RDIMM) kit from Report 5 (~$382).

### LRDIMM Speed Selection

The Z840 with E5-2600 v4 CPUs runs LRDIMMs at up to 2400 MT/s. Faster modules (2666, 2933, 3200) will downclock to 2400 but are often cheaper because they're pulled from newer servers. **Buy the cheapest compatible speed** — there is no performance benefit to paying extra for 2933/3200 modules that will downclock anyway.

**Recommended spec:** PC4-2400T or PC4-2133P 64GB LRDIMM, quad-rank (4Rx4) or octal-rank (8Rx4). Speed: 2133 or 2400 MT/s.

---

## Part 5: The Complete Build — "Ultra" Config (1TB + P40)

### Build Option A: 1TB + Single P40 (~$1,850)

| Component | Spec | Price | Notes |
|-----------|------|-------|-------|
| **Workstation** | HP Z840 barebones (dual CPU) | ~$369 | [eBay search](https://www.ebay.com/sch/i.html?_nkw=hp+z840+barebones) |
| **RAM** | 16×64GB DDR4-2400 LRDIMM | ~$1,200 | Reputable server reseller |
| **GPU** | Tesla P40 24GB | ~$260 | [eBay search](https://www.ebay.com/sch/i.html?_nkw=tesla+p40+24gb) |
| **Display GPU** | GT 710 1GB | ~$15 | For local monitor setup |
| **Storage** | 1TB NVMe SSD | ~$50 | WD SN570 or similar |
| **Cables** | EPS 8-pin adapter | ~$10 | If needed for P40 power |
| **Total** | | **~$1,904** | |

### Build Option B: 1TB + Dual P40 (~$2,150)

| Component | Spec | Price | Notes |
|-----------|------|-------|-------|
| **Workstation** | HP Z840 barebones (dual CPU) | ~$369 | |
| **RAM** | 16×64GB DDR4-2400 LRDIMM | ~$1,200 | |
| **GPUs** | 2× Tesla P40 24GB | ~$520 | |
| **Display GPU** | GT 710 1GB | ~$15 | |
| **Storage** | 1TB NVMe SSD | ~$50 | |
| **Cables** | 2× EPS 8-pin adapters | ~$20 | Report 6 power caveat applies |
| **Total** | | **~$2,174** | |

> **CPU note:** If your Z840 barebones ships with only one CPU, add ~$60–$80 for a matching second E5-2680 v4 (or equivalent). The Z840 uses the same CPU family for both sockets.

---

## Part 6: What 1TB Actually Unlocks — The Capability Matrix

With **~1,048 GB total** (1TB RAM + 24GB VRAM) or **~1,072 GB** (1TB RAM + 48GB VRAM), the 1TB rig breaks through three walls that 512GB cannot:

### Wall 1: Llama 3.1 405B at Q4_K_M

| Context | Weights (Q4_K_M) | KV Cache | KV Quant | Total | Fits in 512GB? | Fits in 1TB? |
|---------|-----------------|----------|----------|-------|----------------|--------------|
| 8K | ~236 GB | ~4.0 GB | f16 | ~240 GB | ✅ Easily | ✅ Easily |
| 32K | ~236 GB | ~15.8 GB | f16 | ~252 GB | ✅ Easily | ✅ Easily |
| 128K | ~236 GB | ~63.0 GB | f16 | ~299 GB | ✅ | ✅ Easily |
| 256K | ~236 GB | ~126.0 GB | f16 | ~362 GB | ✅ | ✅ Easily |
| 512K | ~236 GB | ~252.0 GB | f16 | ~488 GB | ⚠️ Tight | ✅ Easily |
| 1M | ~236 GB | ~504.0 GB | f16 | ~740 GB | ❌ | ✅ |
| 2M | ~236 GB | ~1008.0 GB | f16 | ~1244 GB | ❌ | ❌ |
| 2M | ~236 GB | ~504.0 GB | q8_0 | ~740 GB | ❌ | ✅ |

At 512GB, Llama 3.1 405B at Q4_K_M fits with 256K context at f16 KV, but 512K context needs q8_0. At 1TB, it runs at **1M context with f16 KV** or **2M context with q8_0 KV**.

### Wall 2: DeepSeek-V3 at Q4_K_M + 2M Context

| Context | Weights (Q4_K_M) | KV Cache (MLA) | KV Quant | Total | Fits in 512GB? | Fits in 1TB? |
|---------|-----------------|----------------|----------|-------|----------------|--------------|
| 1M | ~410 GB | ~68.6 GB | f16 | ~478.6 GB | ✅ | ✅ |
| **2M** | **~410 GB** | **~137.2 GB** | **f16** | **~547.2 GB** | **❌** | **✅** |
| 2M | ~410 GB | ~68.6 GB | q8_0 | ~478.6 GB | ✅ | ✅ |

The 512GB rig tops out at 1M context for DeepSeek-V3 Q4_K_M. The 1TB rig stretches to **2M context** — enough for:
- The entire King James Bible (~800K tokens) plus extensive commentary
- A full novel trilogy (~600K–1M tokens) plus prompts
- 200K+ lines of code across 500+ files

### Wall 3: Two Frontier Models Simultaneously

| Model Combo | Weights | Context | KV Cache | Total | Fits in 1TB? |
|-------------|---------|---------|----------|-------|--------------|
| DeepSeek-V3 Q4_K_M + Qwen3-Coder-480B Q4_K_M | ~701 GB | 8K each | ~5.2 GB | ~706 GB | ✅ |
| DeepSeek-V3 Q4_K_M + Llama 3.1 405B Q4_K_M | ~646 GB | 8K each | ~8.1 GB | ~654 GB | ✅ |
| DeepSeek-V3 Q4_K_M (256K) + Qwen3-Coder-480B (64K) | ~701 GB | ~95 GB | — | ~796 GB | ✅ |

This is the **multi-model agent tier** — loading two frontier models and routing queries between them. No consumer platform under $5,000 can do this.

---

## Part 7: Performance Expectations

### LRDIMM Latency Impact

LRDIMMs add ~1–2 ns of latency compared to RDIMMs. At DDR4-2400, a single CAS latency cycle is ~0.42 ns (1/2400MHz × 2). A typical LRDIMM has CL 17–19 vs. CL 15–17 for RDIMMs — approximately **2–4 additional clock cycles**.

**Real-world impact on AI inference:**
- **Expert-layer retrieval (MoE):** ~1–3% slower due to increased RAM latency
- **Dense model CPU offload:** Minimal impact — PCIe 3.0 x16 is the bottleneck, not RAM latency
- **KV cache access from RAM:** ~2–5% slower for large-context generation
- **Overall token generation:** <5% difference vs. RDIMMs at the same capacity

> **Verdict:** The LRDIMM latency penalty is real but minor. The 2× capacity gain far outweighs the <5% speed cost.

### Token Generation Speeds (Estimated)

| Model | Quantization | GPU | Expected tok/s | Notes |
|-------|-------------|-----|----------------|-------|
| DeepSeek-V3 671B | Q4_K_M | Single P40 | **2–4** | Heavier CPU offload than Q2_K |
| DeepSeek-V3 671B | Q4_K_M | Dual P40 | **3–5** | More layers in VRAM |
| Llama 3.1 405B | Q4_K_M | Single P40 | **1.5–3** | Weights mostly in RAM |
| Llama 3.1 405B | Q4_K_M | Dual P40 | **2–4** | Weights still mostly in RAM |
| Llama 3.3 70B | Q4_K_M | Single P40 | **8–15** | Fits in 24GB VRAM |
| Llama 3.3 70B | Q4_K_M | Dual P40 | **12–20** | Weights fully in VRAM |
| Qwen3-Coder-480B | Q4_K_M | Single P40 | **2–4** | MoE, similar to DeepSeek-V3 |
| Qwen3-Coder-480B | Q4_K_M | Dual P40 | **3–5** | More expert layers in VRAM |

> **Important:** These are *slower* than the 512GB Q2_K numbers from Report 4 because Q4_K_M weights are ~1.7× larger, increasing CPU offload traffic. You are trading speed for quality.

---

## Part 8: Context Maxxing — What 1TB+ Means for Long-Context Inference

### Context Capacity Matrix: 256K, 512K, 1M, and 2M

All calculations assume **~1,048 GB total** (1TB RAM + 24GB VRAM, single P40). Weights are Q4_K_M unless noted.

#### 256K Context

| Model | Weights (Q4_K_M) | KV Cache (256K) | KV Quant | Total Memory | Fits? |
|-------|-----------------|-----------------|----------|--------------|-------|
| **Gemma 4 26B-A4B** | ~14 GB | 45.3 GB | f16 | 59.3 GB | ✅ Easily |
| **Qwen2.5-Coder-32B** | ~21 GB | 64.0 GB | f16 | 85.0 GB | ✅ Easily |
| **Llama 3.3 70B** | ~44 GB | 80.0 GB | f16 | 124.0 GB | ✅ Easily |
| **Llama 4 Scout** | ~60 GB | 80.0 GB | f16 | 140.0 GB | ✅ Easily |
| **DeepSeek-V3 671B** | ~410 GB | 17.2 GB | f16 | 427.2 GB | ✅ Easily |
| **Qwen3-Coder-480B** | ~291 GB | 62.0 GB | f16 | 353.0 GB | ✅ Easily |
| **Qwen3.5-122B** | ~75 GB | 80.0 GB | f16 | 155.0 GB | ✅ Easily |
| **Llama 3.1 405B** | ~236 GB | 126.0 GB | f16 | 362.0 GB | ✅ Easily |

At 256K, **every model including Llama 3.1 405B fits with f16 KV** and hundreds of GB of headroom.

#### 512K Context

| Model | Weights (Q4_K_M) | KV Cache (512K) | KV Quant | Total Memory | Fits? |
|-------|-----------------|-----------------|----------|--------------|-------|
| **DeepSeek-V3 671B** | ~410 GB | 34.3 GB | f16 | 444.3 GB | ✅ Easily |
| **Qwen3-Coder-480B** | ~291 GB | 124.0 GB | f16 | 415.0 GB | ✅ Easily |
| **Llama 3.1 405B** | ~236 GB | 252.0 GB | f16 | 488.0 GB | ✅ Easily |
| **Llama 3.3 70B** | ~44 GB | 160.0 GB | f16 | 204.0 GB | ✅ Easily |
| **Qwen3.5-122B** | ~75 GB | 160.0 GB | f16 | 235.0 GB | ✅ Easily |

At 512K, Llama 3.1 405B at Q4_K_M + f16 KV fits with **~560 GB headroom**. This is the headline unlock of 1TB.

#### 1M Context

| Model | Weights (Q4_K_M) | KV Cache (1M) | KV Quant | Total Memory | Fits? |
|-------|-----------------|---------------|----------|--------------|-------|
| **DeepSeek-V3 671B** | ~410 GB | 68.6 GB | f16 | 478.6 GB | ✅ Easily |
| **Qwen3-Coder-480B** | ~291 GB | 248.0 GB | f16 | 539.0 GB | ✅ |
| **Llama 3.1 405B** | ~236 GB | 504.0 GB | f16 | 740.0 GB | ✅ |
| **Llama 3.3 70B** | ~44 GB | 320.0 GB | f16 | 364.0 GB | ✅ Easily |
| **Llama 4 Scout** | ~60 GB | 320.0 GB | f16 | 380.0 GB | ✅ Easily |
| **Qwen3.5-122B** | ~75 GB | 320.0 GB | f16 | 395.0 GB | ✅ Easily |

At 1M context, **all models fit with f16 KV**. Even Llama 3.1 405B — the largest dense open-weight model — fits with **~300 GB of headroom**.

#### 2M Context

| Model | Weights (Q4_K_M) | KV Cache (2M) | KV Quant | Total Memory | Fits? |
|-------|-----------------|---------------|----------|--------------|-------|
| **DeepSeek-V3 671B** | ~410 GB | 137.2 GB | f16 | 547.2 GB | ✅ |
| **Qwen3-Coder-480B** | ~291 GB | 496.0 GB | f16 | 787.0 GB | ✅ |
| **Llama 3.1 405B** | ~236 GB | 1008.0 GB | f16 | ~1244 GB | ❌ |
| **Llama 3.1 405B** | ~236 GB | 504.0 GB | q8_0 | ~740 GB | ✅ |
| **Llama 3.3 70B** | ~44 GB | 640.0 GB | f16 | 684.0 GB | ✅ |
| **Llama 4 Scout** | ~60 GB | 640.0 GB | f16 | 700.0 GB | ✅ |
| **Qwen3.5-122B** | ~75 GB | 640.0 GB | f16 | 715.0 GB | ✅ |

At 2M context, **DeepSeek-V3 at Q4_K_M + f16 KV fits with 500 GB headroom**. Llama 3.1 405B needs q8_0 KV (~740 GB total) but still fits comfortably.

### The "2M Context Club"

With 1TB, the following models can process **2 million tokens** of context:

| Model | Weights | KV Cache (2M, q8_0) | Total | Use Case |
|-------|---------|---------------------|-------|----------|
| **DeepSeek-V3 671B** | ~410 GB | 68.6 GB | ~478.6 GB | Multi-book analysis, legal discovery |
| **Qwen3-Coder-480B** | ~291 GB | 248.0 GB | ~539.0 GB | Entire large codebases (Linux kernel × 5) |
| **Llama 3.1 405B** | ~236 GB | 504.0 GB | ~740 GB | Full academic libraries, multi-year document archives |
| **Llama 3.3 70B** | ~44 GB | 320.0 GB | ~364.0 GB | Interactive long-document chat |
| **Llama 4 Scout** | ~60 GB | 320.0 GB | ~380.0 GB | Multi-novel creative writing |
| **Qwen3.5-122B** | ~75 GB | 320.0 GB | ~395.0 GB | Research synthesis across 100+ papers |

> **What is 2M tokens?** Approximately 1.5 million words — enough for:
> - The entire Lord of the Rings trilogy (~470K words) plus The Silmarillion plus The Hobbit, with room for prompts
> - The complete works of Shakespeare (~900K words) plus extensive commentary
> - 500+ average-length research papers
> - A 5,000-page legal case file

### The Multi-Model Tier

With 1TB, you can load **two frontier models simultaneously** and run inference on both:

| Model A | Model B | Weights Total | Context | KV Total | Memory Total | Fits? |
|---------|---------|--------------|---------|----------|--------------|-------|
| DeepSeek-V3 Q4_K_M | Qwen3-Coder-480B Q4_K_M | ~701 GB | 8K each | ~10 GB | ~711 GB | ✅ |
| DeepSeek-V3 Q4_K_M | Llama 3.1 405B Q4_K_M | ~646 GB | 8K each | ~12 GB | ~658 GB | ✅ |
| DeepSeek-V3 Q4_K_M (128K) | Qwen3-Coder-480B (64K) | ~701 GB | ~130 GB | — | ~831 GB | ✅ |

**Use case:** A coding agent that uses Qwen3-Coder-480B for code generation and DeepSeek-V3 for reasoning/planning, with both models resident in memory. No model swapping latency.

### Recommended llama.cpp Commands for 1TB Context Maxxing

**DeepSeek-V3 at 2M context (Q4_K_M, f16 KV):**
```bash
llama-server \
  --model DeepSeek-V3.1-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  -ot ".ffn_.*_exps.=CPU" \
  --cache-type-k f16 --cache-type-v f16 \
  --no-mmap --mlock \
  --ctx-size 2097152 \
  --flash-attn \
  --threads 32
```

**Llama 3.1 405B at 1M context (Q4_K_M, f16 KV):**
```bash
llama-server \
  --model Llama-3.1-405B-Instruct-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  --cache-type-k f16 --cache-type-v f16 \
  --no-mmap --mlock \
  --ctx-size 1048576 \
  --flash-attn \
  --threads 32
```

**Two models simultaneously (DeepSeek-V3 + Qwen3-Coder-480B):**
```bash
# Terminal 1 — DeepSeek-V3 on CPU+GPU
llama-server \
  --model DeepSeek-V3.1-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  -ot ".ffn_.*_exps.=CPU" \
  --ctx-size 8192 \
  --port 8080 &

# Terminal 2 — Qwen3-Coder-480B on CPU+GPU
llama-server \
  --model Qwen3-Coder-480B-A35B-Instruct-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  -ot ".ffn_.*_exps.=CPU" \
  --ctx-size 8192 \
  --port 8081 &
```

### Why 512GB Cannot Do This

With 512GB RAM + 24GB VRAM = 536GB total:

- **Llama 3.1 405B at Q4_K_M** (~236 GB) + 1M context (~504 GB f16) = **740 GB** → ❌ Does not fit in 512GB
- **DeepSeek-V3 at Q4_K_M** (~410 GB) + 2M context (~137 GB f16) = **547 GB** → ❌ Does not fit
- **Two frontier models simultaneously:** DeepSeek-V3 (~410 GB) + Qwen3-Coder-480B (~291 GB) = **701 GB** → ❌ Does not fit
- **DeepSeek-V3 Q4_K_M + Llama 3.1 405B Q4_K_M simultaneously:** ~646 GB weights alone → ❌ Does not fit

The 512GB rig is a **single-frontier-model platform**. The 1TB rig is a **multi-frontier-model platform**.

---

## Part 9: Risks and Caveats

1. **LRDIMM Cost**
   1TB of LRDIMM costs ~3× more than 512GB of RDIMM. This is not a linear cost scaling — you are paying a premium for the LRDIMM buffer technology. If budget is tight, 512GB RDIMM delivers 80% of the capability at 60% of the cost.

2. **Single-CPU Z840 Limitation**
   Many Z840 barebones on eBay ship with a single CPU. You need dual CPUs for 1TB. Verify the listing or budget ~$60–$80 for a second E5-2680 v4. Also, both CPUs must be identical (same model, same stepping) for optimal performance.

3. **LRDIMM Latency Penalty**
   ~5–10% higher latency than RDIMMs. For latency-sensitive applications (real-time trading, competitive gaming), this matters. For batch AI inference, it does not.

4. **No RDIMM/LRDIMM Mixing**
   If upgrading from a 512GB RDIMM config, you must sell or store all 16×32GB RDIMMs and buy a full set of 16×64GB LRDIMMs. You cannot mix and match.

5. **Counterfeit LRDIMMs**
   The 64GB LRDIMM market has more counterfeit listings than 32GB RDIMMs. Stick to sellers with 500+ feedback, 99%+ ratings, and explicit return policies. Test all modules with MemTest86+ on arrival.

6. **Model Availability**
   400GB+ GGUF files for Llama 3.1 405B and DeepSeek-V3 Q4_K_M are not available on standard HuggingFace model cards. You may need `hf-transfer` with a Pro subscription, or use alternative hosting. Downloading a 500GB file requires a stable connection and significant time.

7. **Power and Thermals**
   A 1TB + dual P40 Z840 draws ~700–800W under load. The 1125W PSU has sufficient headroom, but ensure your wall circuit can handle the sustained draw. Also, two passively cooled P40s in a tower case require excellent airflow — add case fans if necessary.

---

## Final Recommendations

### Who Should Build This?

- **You need Llama 3.1 405B at Q4_K_M with 128K+ context** — 1TB is the minimum viable platform
- **You want 2M token context** on DeepSeek-V3 or Qwen3-Coder-480B
- **You run multi-model agentic workflows** (two frontier models simultaneously)
- **You have a 512GB rig and want to upgrade** without changing the host platform
- **You view this as a 3-year investment** and accept the LRDIMM cost premium

### Who Should Skip This?

- **You only run 7B–70B models** — 512GB is sufficient and $700+ cheaper
- **You need interactive speed** on 405B models — even 1TB + dual P40 generates at only 2–4 tok/s for 405B
- **Budget is under $1,500** — buy the 512GB config from Report 5 instead
- **You want plug-and-play** — this is still a tinkerer's project, just with more RAM

### Build vs. Cloud

Running Llama 3.1 405B at Q4_K_M on a 1TB rig costs ~$1,900 upfront. Cloud API access to a 405B-class model (Claude Opus, GPT-4) costs ~$15–$75 per million tokens. At 2M tokens per query and daily usage, the rig pays for itself in **3–6 months** vs. cloud API costs — and you own the weights, the data stays local, and there are no rate limits.

### The Verdict

The **1TB + Tesla P40 configuration** is the ultimate expression of the value-workstation approach. It is not cheap (~$1,900), not fast (2–4 tok/s on 405B), and not plug-and-play. But it is the **cheapest platform on Earth that can load Llama 3.1 405B at Q4_K_M entirely in memory** and process million-token contexts without disk thrashing.

For researchers, legal analysts, and AI agents that need to reason across entire libraries of text, the 1TB rig is the last stop before enterprise-grade $10,000+ server hardware.

---

*Report compiled from eBay sold/completed listings (May 2026), HP Z840 Service Manual (c04823811.pdf), HP Z840 Workstation Technical Specifications, and llama.cpp context-sizing documentation. Prices are approximate and fluctuate. LRDIMM listings under $40 per module should be treated as suspicious — verify seller reputation before purchase.*
