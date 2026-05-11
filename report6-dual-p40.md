# The Road to Dual Tesla P40s: 48GB VRAM on a Budget

*Research conducted: May 2026*  
*Goal: Determine whether running two Tesla P40 24GB cards in a single workstation is practical, what it costs, and what capabilities 48GB of pooled VRAM actually unlocks*  
*Budget target: ~$1,200–$1,600 for a complete dual-GPU rig*

---

## Executive Summary

**Yes, dual Tesla P40s are possible.** Two P40s in one workstation yield **48GB of pooled VRAM** — enough to run 70B-parameter models with minimal CPU offload, host two 32B models simultaneously, or split MoE expert layers across both cards. It is the cheapest way to get >24GB of VRAM with any semblance of a warranty (the cards themselves are used, but the workstation chassis is typically refurbished).

**However, it is not for everyone.** Dual P40s introduce power-cabling complexity, thermal management challenges, and the same Pascal-era limitations (no tensor cores, no display outputs) that define the single-P40 experience. The performance gain over a single P40 is substantial for models that straddle the 24GB line, but dual P40s do not magically make dense 70B models fast.

**The honest bottom line:** If you already have a single P40 rig and want more VRAM, a second P40 is the cheapest incremental upgrade (~$260). If you are building from scratch, consider whether a single P40 + 512GB system RAM (Report 5) meets your needs before chasing the complexity of dual GPUs.

| Config | VRAM | Total Rig Cost | Best For |
|--------|------|---------------|----------|
| Single P40 + 256GB RAM | 24GB | ~$900 | 32B native; 70B offload; frontier MoE at Q2_K |
| Single P40 + 512GB RAM | 24GB | ~$1,100 | 70B native; frontier dense/MoE at Q4_K_M |
| **Dual P40 + 256GB RAM** | **48GB** | **~$1,300** | **70B with light offload; dual 32B; better MoE split** |
| **Dual P40 + 512GB RAM** | **48GB** | **~$1,500** | **The ultimate budget frontier rig** |
| RTX 3090 24GB + PC | 24GB | ~$900–$1,100 | Faster inference, tensor cores, display output, simpler build |

---

## Part 1: Is Dual P40 Actually Possible?

### The Short Answer

Yes. `llama.cpp` supports multi-GPU inference natively via CUDA. The Tesla P40 is a standard PCIe 3.0 x16 card. Any workstation or server with **two free PCIe x16 slots**, sufficient power delivery, and adequate cooling can run dual P40s.

### What You Actually Need

| Requirement | Spec | Why It Matters |
|-------------|------|----------------|
| **PCIe x16 slots** | 2× full-length, full-height | P40 is a dual-slot card. You need physical space + electrical x16 |
| **Power supply** | ≥1000W recommended | 2×250W GPUs + 2×135W CPUs + overhead = ~800W sustained |
| **GPU power cables** | EPS 8-pin (not PCIe 8-pin) | P40 uses CPU-style EPS connectors. Standard GPU cables will not fit. |
| **Above 4G Decoding** | Enabled in BIOS | Required for each GPU's 24GB BAR space to be mapped |
| **CPU lanes** | 2 CPUs preferred | Dual-CPU workstations route x16 lanes to multiple slots without PLX switching |
| **Cooling** | Blower fans mandatory | P40s are passively cooled. In a tower case, each needs a high-static-pressure fan. |

### The Power Cable Gotcha (Read This Carefully)

The Tesla P40 requires **8-pin EPS 12V power** — the same connector used for CPU power on motherboards. **This is not the same as an 8-pin PCIe power connector.** The pinouts are different, and forcing a standard GPU cable into the P40's EPS socket can damage the card.

- **HP Z840:** The PSU provides **three 6-pin PCIe auxiliary connectors** (not EPS). Each is rated for 216W. To power a P40, you need a **6-pin PCIe female → EPS 8-pin male** adapter cable. These are not standard consumer parts. The HP community has documented similar adaptations for Tesla K80s, which share the EPS connector requirement. Budget **$15–$30 per adapter** and verify polarity before powering on.
  - HP forum reference: [Tesla K80 in Z840 — EPS connector discovery](https://h30434.www3.hp.com/t5/Business-PCs-Workstations-and-Point-of-Sale-Systems/HP-Z840-2x-Xeon-E5-2667v3-2x-Tesla-K80-1x-Quadro-K4200/td-p/9134314)

- **Dell T7910:** The 1300W PSU provides dedicated GPU power cables. Dell's workstation GPU cables are typically wired for standard PCIe connectors, but the T7910 supports up to three 225W Quadro cards. You will likely need **custom PCIe 8-pin → EPS 8-pin adapters** or repurpose CPU EPS cables with proper pinout verification.

- **Rackmount servers:** The cleanest solution. Many 2U/4U server chassis (e.g., Penguin Computing, Supermicro) include native EPS cabling for Tesla cards. This is how the hyperscalers ran P40s — and why the tinycomputers.io 4×P40 build used a rackmount server.

**Recommendation:** If you buy P40s that include a "GPU power adapter" (some eBay bundles do), verify whether the adapter is PCIe-to-EPS or motherboard-specific. For HP Z840 builds, you may need to crimp custom cables or commission adapters from a vendor like [ModDIY](https://www.moddiy.com) or eBay sellers specializing in workstation GPU cables.

---

## Part 2: The Ideal Platforms

### Option 1: HP Z840 (Best Value Tower)

The HP Z840 is the same hero platform from Report 5. For dual P40s, it offers:

- **Three PCIe Gen3 x16 slots** with dual CPUs installed — room for two dual-slot P40s plus a display adapter
- **1125W or 1500W PSU** — the 1125W unit is adequate; the 1500W unit provides headroom for 24/7 operation
- **Three 6-pin PCIe power connectors** — enough for two P40s with one connector to spare (if using splitters or adapters)
- **16 DIMM slots** — scales to 512GB system RAM

| Spec | HP Z840 |
|------|---------|
| CPUs | Dual Xeon E5-2600 v3/v4 |
| DIMM Slots | 16 |
| PCIe x16 (dual CPU) | 3× Gen3 x16 |
| PSU Options | 850W / 1125W / 1275W / 1500W |
| GPU Power Rails | 3× 6-pin PCIe (216W each) |
| Official GPU Support | Up to 3× 225W cards |

**Price anchor:**
- [eBay item `185768642880`](https://www.ebay.com/itm/185768642880) — *"HP Z840 Workstation 2X 12-Core E5-2690 V3 2.60GHz No RAM No HDD No OS No GPU"* — **$368.97** (digitalmind2000)

**Power math for dual P40s:**
- 2× P40 = 500W
- 2× E5-2690 v3 = 270W
- 16× RDIMM + SSD + fans = ~130W
- **Total sustained:** ~900W
- **1125W PSU headroom:** 225W — tight but viable for a well-ventilated chassis
- **1500W PSU headroom:** 600W — ideal for 24/7 operation

If your Z840 ships with the 1125W PSU, it will work. If you plan to run both GPUs at 100% load simultaneously for extended periods, upgrading to the 1500W PSU (HP P/N 860477-001 or 792340-001) is worth the ~$100–$150.

### Option 2: Dell Precision T7910 (More PCIe, More Power)

The T7910 is the premium alternative:

- **Four PCIe x16 slots** with dual CPU — ultimate expansion flexibility
- **1300W 80PLUS Gold PSU** — more headroom than the Z840's base 1125W
- **Dell's GPU power ecosystem** — better documented for custom Tesla cabling

| Spec | Dell T7910 |
|------|-----------|
| CPUs | Dual Xeon E5-2600 v3/v4 |
| DIMM Slots | 16 |
| PCIe x16 (dual CPU) | 4× Gen3 x16 |
| PSU | 1300W (80PLUS Gold) |
| Official GPU Support | Up to 3× 225W cards (675W total) |

**Price anchors:**
- [eBay shop listing](https://www.ebay.com/shop/dell-precision-t7910) — Dell T7910 2x Xeon E5-2690 v4, No RAM/HDD/GPU/OS — **$474.97**
- [eBay item `187617314885`](https://www.ebay.com/itm/187617314885) — *"Dell Precision T7910 Barebones Unit No Heatsinks 4x TRAYS 2011-3 DDR4 1300W PSU"*

**Verdict:** The T7910 is ~$100 more than the Z840 but offers a larger PSU and an extra PCIe slot. For dual-GPU builds, the 1300W PSU is a meaningful upgrade. For single-GPU builds, the Z840 is the better value.

### Option 3: Rackmount Server (The "Proper" Way)

If you have the space and noise tolerance, a used 2U or 4U rackmount server is the native habitat for Tesla P40s:

- **Supermicro 2U/4U GPU servers** — designed for passively cooled Tesla cards with front-to-rear airflow
- **Dell PowerEdge C4130** — supports up to 4× Tesla cards with redundant 2000W PSUs
- **Penguin Computing Relion series** — the platform used in the [tinycomputers.io 4×P40 build](https://tinycomputers.io/posts/repurposing-enterprise-gpus-the-tesla-p40-home-lab-story.html)

**Trade-offs:**
- Louder than tower workstations (high-RPM 40mm fans)
- Require rack rails or shelf
- Often lack display outputs — fully headless operation
- Can be cheaper per-slot than towers ($300–$600 for a 2U chassis with dual E5 v4)

---

## Part 3: llama.cpp Multi-GPU Support

### How It Works

`llama.cpp` automatically distributes model layers across all visible CUDA devices. You do not need to manually shard models. When you load a GGUF, `llama.cpp` queries each GPU's available VRAM and assigns layers greedily from the lowest-numbered device upward.

For two P40s:
```bash
export CUDA_VISIBLE_DEVICES=0,1
./llama-server \
  --model DeepSeek-R1-Distill-Llama-70B-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  --ctx-size 8192 \
  --host 0.0.0.0 --port 8080
```

The first ~24GB of layers load onto GPU 0; the next ~24GB load onto GPU 1. Any remaining layers spill to system RAM (CPU offload).

### Manual Layer Pinning (Advanced)

For optimal performance, you can manually assign specific layer ranges to specific GPUs using environment variables or `llama.cpp` build flags. This is useful if you want to keep attention layers on the faster card (if cards differ) or reserve one GPU for a secondary workload.

### MoE Model Behavior

For Mixture-of-Experts models, `llama.cpp` with `--n-cpu-moe` offloads *expert* layers to CPU/RAM while keeping attention/routing layers on GPU. With dual P40s, you can distribute expert layers across both GPUs using tensor-split arguments, reducing the CPU-RAM bandwidth bottleneck.

```bash
./llama-server \
  --model DeepSeek-V3.1-UD-Q2_K_XL.gguf \
  --tensor-split 0.5,0.5 \
  --n-gpu-layers 999 \
  -ot ".ffn_.*_exps.=CPU" \
  --no-mmap --mlock \
  --ctx-size 8192
```

---

## Part 4: What Does 48GB Actually Unlock?

### The Density vs. Speed Trade-Off

Before listing capabilities, a critical reality check from real-world benchmarking:

A [four-P40 build documented by tinycomputers.io](https://tinycomputers.io/posts/repurposing-enterprise-gpus-the-tesla-p40-home-lab-story.html) (96GB VRAM total) achieved the following on Ollama/llama.cpp:

| Model | Architecture | 4× P40 (96GB) | 4× T4 (AWS) |
|-------|-------------|---------------|-------------|
| Llama 3.2 3B | Dense | 94.3 tok/s | 101.5 tok/s |
| Qwen 2.5 7B | Dense | 52.7 tok/s | 40.3 tok/s |
| Llama 3.1 8B | Dense | 47.8 tok/s | 29.2 tok/s |
| gpt-oss 120B | MoE (5.1B active) | **28.1 tok/s** | 20.6 tok/s |
| Llama 3.1 70B | Dense | **0.033 tok/s** | 2.0 tok/s |

**The lesson:** Dense 70B models are a wall for Pascal. The 4×P40 rig produced **one token every 30 seconds** on Llama 3.1 70B — unusable. The reason: no tensor cores + massive cross-GPU PCIe traffic for dense layers.

With **dual P40 (48GB)**, a 70B model would offload ~5GB of weights to CPU/RAM. This is far better than the 4×P40 result (which tried to fit the whole model across all GPUs with catastrophic overhead), but still slower than an RTX 3090 24GB with heavy CPU offload.

### What Dual P40 Does Well

| Capability | Single P40 (24GB) | Dual P40 (48GB) | Notes |
|------------|-------------------|-----------------|-------|
| **70B Q4_K_M** | Offload ~29GB to CPU (~2–4 tok/s) | Offload ~5GB to CPU (~4–6 tok/s) | Dual P40 reduces CPU bottleneck significantly |
| **Two 32B models simultaneously** | ❌ Impossible | ✅ ~15–25 tok/s each | Run Qwen2.5-Coder-32B + DeepSeek-R1-Distill-Qwen-32B side by side |
| **32B + massive context** | KV cache spills to CPU | KV cache stays in VRAM | 128K–256K context on 32B models at full speed |
| **MoE expert split** | Experts on CPU or single GPU | Experts distributed across 2 GPUs | Reduces PCIe traffic for active experts |
| **70B with Q8_0** | ❌ Does not fit | ~50GB; fits with KV quant | Higher quality 70B inference |

### The Real-World Sweet Spots for Dual P40

1. **Llama 3.3 70B / DeepSeek-R1-Distill-Llama-70B at Q4_K_M**
   - Weights: ~43GB. KV cache (8K): ~10GB. Total: ~53GB.
   - Single P40: Offloads ~29GB of weights to CPU. Speed: ~2–4 tok/s.
   - Dual P40: Offloads only ~5GB. Speed: ~4–6 tok/s.
   - This is the single biggest quality-of-life improvement — the model is finally "mostly GPU-resident."

2. **Two Independent 32B Models**
   - Qwen2.5-Coder-32B on GPU 0, DeepSeek-R1-Distill-Qwen-32B on GPU 1.
   - Each gets 24GB — enough for full weights + comfortable KV cache.
   - Ideal for a multi-model subagent pipeline (e.g., one model generates, another reviews).

3. **Mixtral 8x22B MoE**
   - Q4_K_M weights: ~80GB. Active experts per token: ~39B params.
   - Single P40: Heavy CPU offload; ~3–5 tok/s.
   - Dual P40: Attention layers on GPU 0, expert layers split across both GPUs; ~8–12 tok/s.

4. **Qwen3.5-35B-A3B MoE at 256K Context**
   - Weights: ~20GB. KV cache (256K, f16): ~17.9GB. Total: ~38GB.
   - Single P40: Tight; KV cache spills to CPU.
   - Dual P40: Entire model + context fits in 48GB VRAM. No CPU offload. ~12–18 tok/s.

---

## Part 5: Build Configurations

### Config A: Dual P40 + 256GB RAM (The Upgrade Path)

For those who already built the 128GB or 256GB T5810/P40 rig and want more VRAM without replacing the host:

**Problem:** The T5810 cannot support dual P40s. It has only two PCIe x16 slots but its PSU (685W/825W max) and slot power limit (225W per slot) make dual 250W P40s risky. The motherboard also has only one GPU power header.

**Solution:** Upgrade the host to an HP Z840 or Dell T7910, migrate your RAM and P40, then add a second P40.

| Component | Spec | Price |
|-----------|------|-------|
| **Host** | HP Z840 barebones (dual E5-2690 v3) | **$369** |
| **RAM** | 256GB (8×32GB) DDR4 ECC RDIMM | **$220** |
| **GPU 1** | Tesla P40 24GB (existing) | **$0** |
| **GPU 2** | Tesla P40 24GB + cooling fan | **$260** |
| **Power adapters** | 2× 6-pin PCIe → EPS 8-pin adapters | **$30–$60** |
| **Boot SSD** | 512GB SATA SSD | **$25** |
| **TOTAL** | | **~$904 (existing rig) / ~$1,164 (from scratch)** |

### Config B: Dual P40 + 512GB RAM (The Ultimate Budget Rig)

This is the logical combination of Report 5 and Report 6.

| Component | Spec | Price | Source |
|-----------|------|-------|--------|
| **Host** | HP Z840 barebones (dual E5-2690 v3) | **$369** | [eBay item `185768642880`](https://www.ebay.com/itm/185768642880) |
| **RAM** | 512GB (16×32GB) DDR4 ECC RDIMM | **$420** | Server pulls / [eBay kit](https://www.ebay.com/itm/297346959069) |
| **GPU 1** | Tesla P40 24GB + cooling fan | **$260** | [eBay item `198009478303`](https://www.ebay.com/itm/198009478303) |
| **GPU 2** | Tesla P40 24GB + cooling fan | **$260** | [eBay item `198009478303`](https://www.ebay.com/itm/198009478303) |
| **Power adapters** | 2× 6-pin PCIe → EPS 8-pin adapters | **$40** | Custom / eBay workstation cable vendors |
| **Display GPU** | GT 710 1GB (for local monitor) | **$15** | Any eBay listing |
| **Boot SSD** | 1TB NVMe or SATA SSD | **$40** | Any eBay SSD lot |
| **TOTAL** | | **~$1,404** | |

### Notes

- **Cooling:** Each P40 needs its own blower fan + shroud. Ensure the Z840's front intake is unobstructed. The stock Z840 fans are loud under load; expect server-grade noise.
- **Power:** If your Z840 has the 1125W PSU, dual P40s + dual E5-2690 v3 will run at ~80% sustained load. This is within spec but warm. The 1500W PSU is strongly recommended for 24/7 dual-GPU operation.
- **Above 4G Decoding:** Must be enabled in BIOS. Without it, only one P40's BAR space will map.
- **Headless operation:** The P40s have no display outputs. Use the GT 710 for local setup, then manage via SSH.

---

## Part 6: Performance Expectations

### Token Generation Speeds (Estimated, Dual P40)

| Model | Quantization | GPU Offload | Expected tok/s | Bottleneck |
|-------|-------------|-------------|----------------|------------|
| Llama 3.1 8B | Q4_K_M | Fully in VRAM (split across both GPUs) | 45–55 | Pascal CUDA cores |
| Qwen2.5-Coder-32B | Q4_K_M | Fully in VRAM (fits on one GPU; second GPU idle or running second model) | 15–25 | Pascal CUDA cores |
| Llama 3.3 70B | Q4_K_M | ~43GB on GPUs, ~5GB on CPU | **4–6** | CPU offload latency |
| DeepSeek-R1-Distill-Llama-70B | Q4_K_M | ~43GB on GPUs, ~5GB on CPU | **4–6** | CPU offload latency |
| Mixtral 8x22B | Q4_K_M | Attention on GPU 0, experts split across both GPUs | 8–12 | PCIe + CPU RAM bandwidth |
| gpt-oss 120B (MoE) | MXFP4 | Split across both GPUs | 12–18 (est.) | Memory bandwidth |

### Comparison to Single RTX 3090 24GB

| Model | Dual P40 (48GB) | RTX 3090 24GB + 64GB RAM | Notes |
|-------|-----------------|--------------------------|-------|
| 7B Q4 | 45–55 tok/s | 90–105 tok/s | RTX 3090 wins 2× on small models (tensor cores) |
| 32B Q4 | 15–25 tok/s | 15–22 tok/s | Roughly tied |
| 70B Q4 | **4–6 tok/s** | **2–4 tok/s** | Dual P40 wins by reducing CPU offload |
| MoE 120B | 12–18 tok/s | ❌ Cannot run | Dual P40 uniquely wins |

**Pattern:** The RTX 3090 dominates on small models thanks to tensor cores and faster GDDR6X memory. But for 70B models and large MoE architectures, dual P40s win because **capacity beats speed** when the alternative is heavy CPU offload.

---

## Part 7: Power, Thermals, and Noise

### Power Draw (Dual P40 + Z840)

| Component | TDP |
|-----------|-----|
| 2× Xeon E5-2690 v3 | 270W |
| 2× Tesla P40 | 500W |
| 16× DDR4 RDIMM | 80W |
| SSD + fans + GT 710 | 50W |
| **Total** | **~900W** |

At $0.12/kWh, 24/7 operation costs roughly **$95/month** (~$1,140/year). This is significant. For comparison, a single RTX 3090 rig draws ~450W (~$47/month), and a Mac Studio M1 Max draws ~100W (~$10/month).

### Thermal Concerns

- **P40s:** Each card needs a dedicated 120mm blower fan (Delta BFB1012VH or similar) pushing air through the passive heatsink. Without this, the card will thermal-throttle at 85°C and eventually shut down.
- **Z840 chassis:** Designed for 225W Quadro cards. Two 250W P40s exceed the thermal design by a small margin. Ensure:
  - Front intake vents are unobstructed
  - The memory shrouds are installed (they direct airflow)
  - Ambient room temp is <25°C if possible

### Noise

A dual-P40 Z840 is **not a desktop PC.** Under load, it will produce **50–55 dBA** — comparable to a server closet. If you plan to run this in a home office, expect to hear it. Basement, garage, or closet placement is recommended.

---

## Part 8: Cost Comparison

### Dual P40 vs. Alternatives

| Metric | Dual P40 + Z840 + 512GB | RTX 3090 24GB + PC + 128GB | Mac Studio M2 Ultra 192GB |
|--------|------------------------|---------------------------|--------------------------|
| **Upfront cost** | ~$1,400 | ~$1,000 | ~$3,500 |
| **VRAM / Unified Memory** | 48GB VRAM + 512GB RAM | 24GB VRAM + 128GB RAM | 192GB unified |
| **70B Q4_K_M speed** | ~4–6 tok/s | ~2–4 tok/s | ~8–12 tok/s |
| **Max model size** | 70B mostly native; 405B/671B with CPU RAM | 35B native; 70B heavy offload | 70B native; 405B/671B ❌ |
| **Tensor cores** | ❌ No | ✅ Yes | ✅ Yes (Neural Engine) |
| **Display outputs** | ❌ No (need 2nd GPU) | ✅ Yes | ✅ Yes |
| **Noise** | Loud | Moderate | Whisper quiet |
| **Power (24/7)** | ~900W | ~450W | ~100W |
| **2-year TCO** | ~$3,680 | ~$2,120 | ~$3,860 |

### The Economic Argument

If your goal is **maximum model capacity per dollar**, dual P40s are compelling. If your goal is **speed, efficiency, and sanity**, an RTX 3090 or Mac Studio is a better choice. The dual P40 rig pays for itself only if you genuinely need 48GB+ of VRAM and cannot afford an RTX 4090 24GB ($1,500+) or Mac Studio M2 Ultra ($3,500+).

---

## Part 10: Context Maxxing — What 48GB VRAM + 512GB RAM Means for Long-Context Inference

With **560GB total memory** (512GB RAM + 48GB VRAM across two P40s), the dual-P40 configuration is not just "more total memory" — it is **more GPU-resident memory**. The 24GB difference between single and dual P40 is the difference between spilling 70B model weights to CPU RAM and keeping them entirely in VRAM.

### The Dual-VRAM Advantage

For autoregressive inference, memory lives in two tiers:
1. **GPU VRAM** — 48 GB total, ~400+ GB/s bandwidth, zero PCIe latency
2. **CPU RAM** — 512 GB, ~120 GB/s bandwidth, PCIe 3.0 x16 bottleneck (~16 GB/s)

On a **single P40**, a 70B model at Q4_K_M (~44 GB weights) spills ~20 GB of weights to CPU RAM. Every token requires fetching those 20 GB over PCIe — a catastrophic bottleneck that drops dense 70B inference to **4–6 tok/s** even with perfect cooling.

On **dual P40s**, the same 70B model fits **entirely in VRAM** (44 GB < 48 GB). Only the KV cache spills to CPU RAM. Since KV cache is accessed sequentially and CPU RAM bandwidth is ~120 GB/s, the bottleneck is dramatically reduced. Dense 70B inference improves to **6–10 tok/s** — a 1.5–2× speedup purely from keeping weights in VRAM.

### The KV Cache Quantization Toolkit

Same toolkit as Reports 4 and 5:

| KV Type | Bits/Weight | vs FP16 | Speed Impact | Quality Impact |
|---------|------------|---------|--------------|----------------|
| **f16** (default) | 16 | 1.0× | Baseline | None |
| **q8_0** | 8 | 2.0× smaller | <5% slower | Negligible |
| **q4_0** | 4 | 4.0× smaller | ~35% slower at 64K+ | Minor |
| **turbo3** | ~3.1 | ~4.9× smaller | Minimal (flash-attn) | Small PPL increase |

### Architecture Matters

Same per-token KV cache rates as Reports 4 and 5:

- **DeepSeek-V3 MLA:** ~68.6 KB/token (f16)
- **Standard GQA (Llama 3):** ~128 KB/token (f16)
- **Standard GQA (Qwen2.5 32B):** ~256 KB/token (f16)
- **Qwen3-Coder-480B:** ~248 KB/token (f16)
- **Llama 3.3 70B / Llama 4 Scout / Qwen3.5-122B:** ~320 KB/token (f16)
- **Gemma 4 26B-A4B:** ~181 KB/token (f16)

---

### Context Capacity Matrix: What Fits at 256K, 512K, and 1M

All calculations assume **560GB total memory envelope** (48GB VRAM + 512GB RAM). Weights are Q4_K_M unless noted.

#### 256K Context

| Model | Weights (Q4_K_M) | KV Cache (256K) | KV Quant | Total Memory | Weights in VRAM? | Fits? |
|-------|-----------------|-----------------|----------|--------------|------------------|-------|
| **Gemma 4 26B-A4B** | ~14 GB | 45.3 GB | f16 | 59.3 GB | ✅ All 14 GB | ✅ |
| **Qwen2.5-Coder-32B** | ~21 GB | 64.0 GB | f16 | 85.0 GB | ✅ All 21 GB | ✅ |
| **Llama 3.3 70B** | ~44 GB | 80.0 GB | f16 | 124.0 GB | ✅ All 44 GB | ✅ |
| **Llama 4 Scout** | ~60 GB | 80.0 GB | f16 | 140.0 GB | ✅ All 60 GB | ✅ |
| **DeepSeek-V3 671B** | ~410 GB | 17.2 GB | f16 | 427.2 GB | ❌ 48GB only | ✅ |
| **Qwen3-Coder-480B** | ~291 GB | 62.0 GB | f16 | 353.0 GB | ❌ 48GB only | ✅ |
| **Qwen3.5-122B** | ~75 GB | 80.0 GB | f16 | 155.0 GB | ✅ All 75 GB | ✅ |

**Key insight:** Every model up to **122B parameters** has its **weights fully resident in VRAM** at 256K context. On a single P40, only models up to ~24B parameters achieve this. For 70B–122B models, this eliminates the weight-spill bottleneck entirely.

#### 512K Context

| Model | Weights (Q4_K_M) | KV Cache (512K) | KV Quant | Total Memory | Weights in VRAM? | Fits? |
|-------|-----------------|-----------------|----------|--------------|------------------|-------|
| **Gemma 4 26B-A4B** | ~14 GB | 90.5 GB | f16 | 104.5 GB | ✅ All 14 GB | ✅ |
| **Qwen2.5-Coder-32B** | ~21 GB | 128.0 GB | f16 | 149.0 GB | ✅ All 21 GB | ✅ |
| **Llama 3.3 70B** | ~44 GB | 160.0 GB | f16 | 204.0 GB | ✅ All 44 GB | ✅ |
| **Llama 4 Scout** | ~60 GB | 160.0 GB | f16 | 220.0 GB | ✅ All 60 GB | ✅ |
| **DeepSeek-V3 671B** | ~410 GB | 34.3 GB | f16 | 444.3 GB | ❌ 48GB only | ✅ |
| **Qwen3-Coder-480B** | ~291 GB | 124.0 GB | f16 | 415.0 GB | ❌ 48GB only | ✅ |
| **Qwen3.5-122B** | ~75 GB | 160.0 GB | f16 | 235.0 GB | ✅ All 75 GB | ✅ |

At 512K context, **DeepSeek-V3 at Q4_K_M + f16 KV fits with 115 GB headroom** — comfortably inside the 560GB envelope. Qwen3-Coder-480B at Q4_K_M + f16 KV fits with 145 GB headroom.

#### 1M Context

| Model | Weights (Q4_K_M) | KV Cache (1M) | KV Quant | Total Memory | Fits? |
|-------|-----------------|---------------|----------|--------------|-------|
| **Gemma 4 26B-A4B** | ~14 GB | 181.0 GB | f16 | 195.0 GB | ✅ |
| **Qwen2.5-Coder-32B** | ~21 GB | 256.0 GB | f16 | 277.0 GB | ✅ |
| **Llama 3.3 70B** | ~44 GB | 320.0 GB | f16 | 364.0 GB | ✅ |
| **Llama 4 Scout** | ~60 GB | 320.0 GB | f16 | 380.0 GB | ✅ |
| **DeepSeek-V3 671B** | ~410 GB | 68.6 GB | f16 | 478.6 GB | ✅ (81.4 GB headroom) |
| **Qwen3-Coder-480B** | ~291 GB | 248.0 GB | f16 | 539.0 GB | ⚠️ Tight; q8_0 (415 GB) ✅ |
| **Qwen3.5-122B** | ~75 GB | 320.0 GB | f16 | 395.0 GB | ✅ |

At 1M context, DeepSeek-V3 at Q4_K_M + f16 KV fits with **81 GB headroom** — even more comfortable than the 512GB single-P40 config (which had 57 GB). Qwen3-Coder-480B at Q4_K_M needs q8_0 KV at 1M (total 415 GB), same as the single-P40 512GB config.

### The VRAM-Resident Context Club

The dual-P40's 48GB VRAM enables a unique tier of inference: **models that fit entirely in GPU memory** with substantial context. This eliminates CPU-RAM bandwidth as a bottleneck for autoregressive generation.

| Model | Weights | Context | KV Quant | Total | Per-GPU | Fully VRAM? | Expected Speed |
|-------|---------|---------|----------|-------|---------|-------------|----------------|
| **Llama 3.1 8B** | ~6 GB | 1M | q4_0 | ~38 GB | ~19 GB | ✅ Yes | **30–45 tok/s** |
| **Qwen2.5-Coder-14B** | ~10 GB | 512K | q4_0 | ~42 GB | ~21 GB | ✅ Yes | **25–35 tok/s** |
| **Gemma 4 26B-A4B** | ~14 GB | 256K | q4_0 | ~25 GB | ~12.5 GB | ✅ Yes | **20–30 tok/s** |
| **Qwen2.5-Coder-32B** | ~21 GB | 256K | q4_0 | ~37 GB | ~18.5 GB | ✅ Yes | **15–25 tok/s** |
| **Llama 3.3 70B** | ~44 GB | 8K | turbo3 | ~45 GB | ~22.5 GB | ✅ Yes | **12–18 tok/s** |
| **Llama 3.3 70B** | ~44 GB | 128K | q4_0 | ~64 GB | ~32 GB | ❌ KV spills | **6–10 tok/s** |

> **Per-GPU calculation:** Total memory divided by 2 (tensor-split). Each P40 has 24GB, so "Fully VRAM" means per-GPU usage ≤ 24GB.

**The headline:** A 70B model with **weights fully in VRAM** at 8K context generates at **12–18 tok/s** — faster than a single RTX 4090 running the same model at 8K (which also fits in 24GB). The dual P40s' combined CUDA core count (~7,680 cores) matches a single RTX 3090, and the 48GB capacity means no weight spill.

### Two-Model Simultaneous Context

The dual-P40 configuration can run **two independent llama-server instances** simultaneously, each assigned to one GPU:

| Setup | GPU 1 | GPU 2 | Use Case |
|-------|-------|-------|----------|
| **Coder + Reasoner** | Qwen2.5-Coder-32B @ 128K | Llama 3.3 70B @ 32K | Coding assistant + general reasoning agent |
| **Small + Large** | Llama 3.1 8B @ 1M | Qwen3-Coder-480B @ 256K | Fast retrieval + frontier coding |
| **Redundant 70B** | Llama 3.3 70B @ 64K | Llama 3.3 70B @ 64K | A/B testing or load balancing |
| **MoE + Dense** | DeepSeek-V3 @ 32K | Llama 4 Scout @ 32K | MoE reasoning + dense retrieval |

**Launch two instances:**
```bash
# GPU 0 — Qwen2.5-Coder-32B at 128K
CUDA_VISIBLE_DEVICES=0 llama-server \
  --model Qwen2.5-Coder-32B-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  --cache-type-k q4_0 --cache-type-v q4_0 \
  --ctx-size 131072 \
  --port 8080 &

# GPU 1 — Llama 3.3 70B at 32K
CUDA_VISIBLE_DEVICES=1 llama-server \
  --model Llama-3.3-70B-Instruct-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  --cache-type-k q4_0 --cache-type-v q4_0 \
  --ctx-size 32768 \
  --port 8081 &
```

> **Note:** Running two frontier models simultaneously will consume ~300–400W of GPU power alone. Ensure your Z840's 1125W PSU has sufficient headroom for CPUs, fans, and drives.

### Recommended llama.cpp Commands for Context Maxxing

**DeepSeek-V3 at 1M context (Q4_K_M, f16 KV, tensor-split):**
```bash
llama-server \
  --model DeepSeek-V3.1-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  --tensor-split 24,24 \
  -ot ".ffn_.*_exps.=CPU" \
  --cache-type-k f16 --cache-type-v f16 \
  --no-mmap --mlock \
  --ctx-size 1048576 \
  --flash-attn \
  --threads 32
```

**Llama 3.3 70B at 128K context (weights fully in VRAM, q4_0 KV):**
```bash
llama-server \
  --model Llama-3.3-70B-Instruct-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  --tensor-split 24,24 \
  --cache-type-k q4_0 --cache-type-v q4_0 \
  --no-mmap --mlock \
  --ctx-size 131072 \
  --flash-attn \
  --threads 32
```

**Qwen2.5-Coder-32B at 256K context (fully in VRAM, q4_0 KV):**
```bash
llama-server \
  --model Qwen2.5-Coder-32B-Q4_K_M.gguf \
  --n-gpu-layers 999 \
  --tensor-split 24,24 \
  --cache-type-k q4_0 --cache-type-v q4_0 \
  --no-mmap --mlock \
  --ctx-size 262144 \
  --flash-attn \
  --threads 32
```

### Why Single P40 Cannot Do This

With 24GB VRAM on a single P40:

- **Llama 3.3 70B weights** (~44 GB) spill **20 GB to CPU RAM** — every token fetches 20 GB over PCIe
- **Qwen3.5-122B weights** (~75 GB) spill **51 GB to CPU RAM** — catastrophic bottleneck
- **Two-model simultaneous** is impossible — only one GPU, one model at a time
- **Qwen2.5-Coder-32B at 256K**: weights ~21 GB fit, but KV ~64 GB (f16) spills **41 GB to RAM**

The dual-P40 upgrade buys you:
1. **Weights fully in VRAM** for all models up to 122B parameters
2. **1.5–2× speedup** on dense 70B inference from eliminating weight spill
3. **Two-model parallelism** for multi-agent workflows
4. **More KV cache in VRAM** for 32B models at 256K+ context

---

## Part 11: Risks and Caveats

1. **Power Cable Complexity**
   The EPS 8-pin requirement is the single biggest gotcha. Do not assume standard GPU power adapters will work. Verify pinouts with a multimeter before first boot.

2. **Pascal Performance Ceiling**
   Dual P40s will never match an RTX 3090 or 4090 on token generation speed for models that fit in 24GB. The P40's advantage is purely capacity.

3. **Dense 70B Models Are Still Slow**
   Even with 48GB VRAM, Llama 3.3 70B at Q4_K_M will generate at only 4–6 tok/s. If you need interactive 70B inference, buy an RTX 4090 or Mac Studio.

4. **Thermal Throttling**
   Two passively cooled 250W GPUs in a tower case is a thermal engineering challenge. If you cannot maintain <75°C on both cards, you will lose performance.

5. **Used GPU Risk**
   Both P40s may be ex-mining or ex-datacenter cards. Buy from sellers with return policies. Run `nvidia-smi` and a 30-minute `gpu-burn` stress test on arrival.

6. **No NVLink**
   The P40 predates NVLink. Inter-GPU communication goes over PCIe 3.0 x16 (~16 GB/s). For inference this is acceptable; for training it would be a serious bottleneck.

---

## Final Recommendations

### Who Should Build Dual P40s?

- **You already own one P40** and want the cheapest path to 48GB VRAM (~$260 incremental cost)
- **You need to run two large models simultaneously** (e.g., 32B coder + 32B reasoning)
- **You want 70B models with minimal CPU offload** and can tolerate 4–6 tok/s
- **You are running MoE inference** where splitting experts across two GPUs reduces bottlenecks
- **You have the technical skills** to manage power adapters, Linux headless setup, and thermal monitoring

### Who Should Skip Dual P40s?

- **You want interactive speed** on 70B models — buy an RTX 4090 24GB or Mac Studio M2 Ultra
- **You value silence** — a dual-P40 Z840 is objectively loud
- **You are building from scratch and only run 7B–32B models** — a single P40 or RTX 3060 12GB is sufficient
- **You lack the patience for custom cabling** — the EPS adapter requirement is a real hurdle

### The Verdict

Dual Tesla P40s are the **poor man's multi-GPU inference cluster.** For ~$1,400 all-in (with 512GB RAM), you get 48GB of VRAM and enough system memory to run frontier models that consumer GPUs simply cannot touch. But the complexity — power adapters, cooling, noise, Pascal slowness — means this is a **tinkerer's project**, not a consumer product.

If you are willing to wrestle with the build, dual P40s deliver unmatched VRAM-per-dollar. If you just want local AI that works, buy a Mac Studio.

---

*Report compiled from eBay sold/completed listings (May 2026), HP Z840 QuickSpecs, Dell T7910 technical specifications, the tinycomputers.io 4×P40 build log, `llama.cpp` multi-GPU documentation, and HP community forum posts. Prices are approximate and fluctuate.*
