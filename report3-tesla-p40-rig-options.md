# Tesla P40 Value Rig Options: Three Builds for MoE LLM Inference

*Research conducted: May 2026*  
*Goal: Maximize VRAM-per-dollar for local LLM inference via `llama.cpp` MoE offloading*  
*Philosophy: Extreme value using used enterprise cast-offs, similar to an OptiPlex 9020 + GTX 1060 build*

---

## Executive Summary

This report specs three complete rigs built around the **NVIDIA Tesla P40 24GB** — the undisputed king of VRAM-per-dollar (~$11–12/GB). The P40 is a datacenter Pascal card with no display outputs and passive cooling, but its 24GB of GDDR5 unlocks 32B dense models fully in VRAM and massive MoE models (e.g., DeepSeek-V3 671B) when paired with enough system RAM and `--n-cpu-moe` offloading.

All three configurations use a **Dell Precision T5810** as the host — a single-socket Xeon E5 v3/v4 workstation with 8 DDR4 DIMM slots, PCIe 3.0 x16, and "Above 4G Decoding" support. It is the cheapest compatible host that can scale to 128GB+ of system RAM.

| Config | System RAM | Total Rig Cost | What It Can Run |
|--------|-----------|----------------|-----------------|
| **Budget** | 32GB | **~$450** | 7B–14B dense native; 32B dense offload; 35B–70B MoE |
| **Mid** | 64GB | **~$520** | 32B dense native; 70B MoE; lighter DeepSeek-V3 offload |
| **Max** | 128GB | **~$680** | 70B dense offload; **DeepSeek-V3 671B MoE**; massive context |

*Prices include GPU, host, RAM, cooling mod, power adapter, and a boot SSD. All anchor listings are active or recently completed eBay listings (May 2026).*

---

## The Core GPU: Tesla P40 24GB

### Why the P40?
- **24GB VRAM** at roughly **$11–12 per GB** — half the price/GB of a used RTX 3090.
- At Q4_K_M quantization, 24GB fits **Qwen2.5-Coder-32B** entirely in GPU memory with no CPU offload.
- For MoE models, the 24GB holds attention layers + KV cache while expert layers live in system RAM.

### What You Get
| Spec | Value |
|------|-------|
| Architecture | Pascal (GP102) |
| CUDA Cores | 3,840 |
| VRAM | 24GB GDDR5, 384-bit bus |
| TDP | 250W |
| Power Connectors | **Two 8-pin EPS** (NOT standard PCIe 6+2) |
| Display Outputs | None (headless only) |
| PCIe | 3.0 x16 |

### What You Must Add
1. **Active cooling** — the P40 is passively cooled. You need a blower fan + 3D-printed shroud, or buy a unit that already has one.
2. **EPS power adapter** — workstation PSUs usually output PCIe-style power for GPUs. The P40 requires EPS 8-pin.

### Price Anchor: P40 with Fan & Power Adapter
**Recommended listing:** [eBay item `376635025151`](https://www.ebay.com/itm/376635025151) — *"Nvidia Tesla P40 24Gb with 8-pin GPU power adapter, shroud & fan"* (US seller, pre-owned). Bundles the card + cooling mod + power adapter in one shipment. Typically **~$280–$330**.

**Alternative (lowest cost):** [eBay item `198009478303`](https://www.ebay.com/itm/198009478303) — *"NVIDIA Tesla P40 24GB DDR5 GPU Accelerator Card With cooling turbine fan"* (China seller, open-box). Listed at **$215.16 + ~$15 shipping = ~$230**. Condition is "excellent, new condition with no wear."

**Another alternative:** [eBay item `315168134779`](https://www.ebay.com/itm/315168134779) — *"Nvidia Tesla P40 24GB GPU GDDR5 PCIE Graphics Accelerator Card with Cooling Fan"* (xdcomputers, China). **$315.00 or Best Offer**, free SpeedPAK shipping, 10 sold.

**For this report, we budget `$260` for a P40 with cooling fan included.**

---

## The Host Machine: Dell Precision T5810

### Why the T5810?
- **Cheapest DDR4 workstation** with 8 DIMM slots and a PSU that can feed a 250W GPU.
- Officially supports **256GB DDR4 ECC RDIMM** (8 × 32GB).
- Has **PCIe 3.0 x16** slot and BIOS support for "Above 4G Decoding" (required for >16GB BAR space).
- Widely available on eBay from business recyclers.

### T5810 Specs
| Feature | Spec |
|---------|------|
| CPU | Intel Xeon E5-1600/2600 v3 or v4 (LGA2011-3) |
| DIMM Slots | 8 |
| Max RAM | 256GB DDR4 ECC RDIMM |
| PCIe x16 | 2 slots (Gen 3) |
| PSU | 425W / 685W / 825W options |
| GPU Power | Proprietary motherboard 8-pin → riser cable |

### ⚠️ Critical: PSU Wattage
The T5810 shipped with **425W**, **685W**, or **825W** PSUs. The 425W unit is insufficient for a 250W P40 + 120W Xeon + overhead. **Only buy the 685W or 825W variant.**

### T5810 Price Anchors

**Barebones (no CPU/RAM/HDD/GPU):**
- eBay item / shop listing — *"NO GPU/RAM/HDD/SSD - 425W"* — **$69.99 + $23.05 shipping** (~$93). ⚠️ 425W PSU; not suitable for P40 without PSU swap.
- eBay item — *"Dell Precision 5810 Workstation LGA 2011-V3 Barebones PC (No HDD, CPU, RAM)"* — **$91.82** ([gtatechsource](https://www.ebay.com/usr/gtatechsource), Canada).

**With CPU + minimal RAM (best value):**
- [eBay item `358165683242`](https://www.ebay.com/itm/358165683242) — *"Dell Precision T5810 Xeon E5 1620 v4 3.0GHz 8GB RAM No Drive No OS No GPU"* — **sold for $104.99** (AZ Warehouse, Feb 2026).
- eBay search / shop — *"Dell Precision T5810 | Xeon E5-1603 v3 | 8GB DDR4 | NO GPU | NO HDD"* — **$86.72** ([calgarycomputerwholesale](https://www.ebay.com/usr/calgarycomputerwholesale), Canada).

**With CPU + 32GB RAM (plug-and-play for Config 1):**
- eBay item / shop — *"Dell Precision T5810 12-Core 2.50GHz E5-2680 v3 32GB RAM No GPU/ HDD/ OS"* — **$174.94 or Best Offer** ([digitalmind2000](https://www.ebay.com/usr/digitalmind2000), 201 sold, eBay Refurbished).
- eBay item / shop — *"Dell Precision T5810 E5-2660 v3 10C 20T 32GB RAM No HD"* — **$135.00 Buy It Now** ([funtechtoy](https://www.ebay.com/usr/funtechtoy), US seller).

---

## System RAM: DDR4 ECC RDIMM

Both the T5810 and HP Z440 require **ECC Registered (RDIMM)** memory. Do not buy desktop UDIMM — it will not post.

The sweet spot for value is **used server pulls** from decommissioned datacenter gear. Samsung, Hynix, and Micron 32GB DDR4-2400/2666 RDIMMs are abundant.

### RAM Price Anchors
| Capacity | Config | Price | Anchor |
|----------|--------|-------|--------|
| 32GB | 2×16GB RDIMM | ~$45 | [eBay server pull lots](https://www.ebay.com/sch/i.html?_nkw=32gb+ddr4+ecc+rdimm) |
| 32GB | 1×32GB RDIMM | ~$55 | [atechcomponents](https://www.ebay.com/usr/atechcomponents) (eBay shop, 131K feedback) |
| 64GB | 2×32GB RDIMM | ~$110 | 2 × $55 server pulls |
| 128GB | 4×32GB RDIMM | ~$220 | 4 × $55 server pulls |

**Specific anchors:**
- [eBay item `364858630914`](https://www.ebay.com/itm/364858630914) — *"Hynix HMA84GR7DJR4N-XN 32GB DDR4-3200 ECC REG DIMM"* — **~$55** (TM Space, US seller).
- eBay shop — atechcomponents *"32GB DDR4 ECC RDIMM"* listings — **$55–$60 per stick**, thousands sold.
- eBay item / shop — *"128GB DDR4-2400 ECC RDIMM Micron MTA18ASF2G72PDZ 8×16GB"* — **$329** (if you prefer 16GB sticks).

For this report, we assume **$55 per 32GB RDIMM stick** for 64GB and 128GB configs.

---

## P40 Cooling Mod (If Not Pre-Installed)

If you buy a bare P40 without a fan, you need:
1. A **3D-printed blower shroud** that clips onto the P40's passive heatsink.
2. A **high-static-pressure blower fan** (e.g., Delta BFB1012VH, 97mm, ~22+ CFM).

### Cooling Price Anchors
- [eBay item `226829835848`](https://www.ebay.com/itm/226829835848) — *"New Cooling Turbine Fan Kit for NVIDIA Tesla P40 P100 M40 K80 Graphics Card"* — **$19.99 + shipping** (China seller).
- [eBay item `153175788721`](https://www.ebay.com/itm/153175788721) — *"NVIDIA Tesla P40 K80 P100 V100 FHHL M40 cooling fan shroud duct"* — **~$25** (Australia seller, 127 sold).
- [eBay item `155462951531`](https://www.ebay.com/itm/155462951531) — *"Nvidia Tesla Cooling FAN AND SHROUD for M40,P40,P100,V100 - USA"* — **$49.99** (US seller, 50 sold).

Because our GPU budget assumes a P40 **with fan included**, we do not add a separate cooling line item. If you buy a bare card, add **$20–$50**.

---

## P40 Power Adapter for Workstations

The P40 requires **8-pin EPS** power. Workstation GPU power cables are usually wired for **PCIe 6+2 pin**, which has a different pinout. **Do not force a standard GPU cable into the P40.**

For the Dell T5810, the motherboard has an 8-pin GPU power header that feeds the PCIe riser. You need a cable that goes from the Dell motherboard 8-pin (or riser) to the P40's EPS 8-pin. Pre-made adapters are sold on eBay and [AliExpress](https://www.aliexpress.com) for Tesla cards.

### Power Adapter Price Anchors
- eBay item `376635025151` — P40 bundled **with 8-pin GPU power adapter** (included in ~$280–$330).
- [eBay item `397278799706`](https://www.ebay.com/itm/397278799706) — *"4PCS Server GPU Power Cable For Dell R720 R730 to Tesla P40 P100 EPS 12V Adapter"* — sold for **$75.99** (4-pack = **~$19 each**).
- [eBay item `185007673667`](https://www.ebay.com/itm/185007673667) — *"10pin to 8pin EPS CPUs Cable for HP DL380 G8 G9 and Nvidia K80/M40/M60/P40/P100"* — **£12.99 (~$17)**.
- AliExpress — *"PCI-E GPU Adapter Cable 2x8Pin Female to 8-Pin Male for Nvidia Tesla P40"* — **~$8–$15** shipped.

For this report, we budget **$15** for a power adapter.

---

## Configuration 1: Budget — 32GB System RAM

**Target:** Run 7B–14B dense models fully in VRAM, 32B dense with offload, and 35B–70B MoE models via `--n-cpu-moe`.

### Bill of Materials
| Component | Spec | Price | eBay Anchor |
|-----------|------|-------|-------------|
| **Host** | Dell T5810, E5-2680 v3, 32GB RAM, no GPU/HDD/OS | **$174.94** | digitalmind2000 — [T5810 12C 32GB](https://www.ebay.com/sch/i.html?_nkw=dell+precision+t5810) (201 sold) |
| **GPU** | Tesla P40 24GB + cooling fan | **$260.00** | [Item `198009478303`](https://www.ebay.com/itm/198009478303) (~$230) or bundled item `376635025151` (~$300) |
| **Power adapter** | EPS adapter for P40 | **$15.00** | [Item `397278799706`](https://www.ebay.com/itm/397278799706) (~$19/ea) or AliExpress |
| **Boot SSD** | 128–256GB SATA SSD (new or used) | **$15.00** | Any eBay SSD lot |
| **TOTAL** | | **~$465** | |

### Notes
- The T5810 in this listing has a **685W or 825W PSU** (digitalmind2000's eBay Refurbished units typically ship with the larger PSU).
- 32GB system RAM is the floor for MoE offloading. You will rely heavily on `--no-mmap` and `--mlock` to keep weights in RAM.
- Example command for Qwen3-35B-A3B MoE:
  ```bash
  llama-server --model qwen3-35b-a3b.Q4_K_M.gguf --n-gpu-layers 999 \
    --n-cpu-moe 41 --no-mmap --mlock --ctx-size 64000
  ```

---

## Configuration 2: Mid — 64GB System RAM

**Target:** Run 32B dense fully in VRAM, 70B MoE comfortably, and experiment with lighter DeepSeek-V3-class offload.

### Bill of Materials
| Component | Spec | Price | eBay Anchor |
|-----------|------|-------|-------------|
| **Host** | Dell T5810 barebones (no CPU/RAM/HDD) | **$92.00** | [eBay listing ~$69.99 + $23 ship](https://www.ebay.com/sch/i.html?_nkw=dell+precision+t5810+no+ram+no+hdd) (685W+ PSU) |
| **CPU** | Xeon E5-1620 v3 or E5-2680 v3 | **$20.00** | [eBay CPU lots](https://www.ebay.com/sch/i.html?_nkw=xeon+e5-1620+v3) (~$15–$30) |
| **RAM** | 64GB (2×32GB) DDR4 ECC RDIMM | **$110.00** | atechcomponents / server pulls (~$55/stick) |
| **GPU** | Tesla P40 24GB + cooling fan | **$260.00** | Item `198009478303` or `315168134779` |
| **Power adapter** | EPS adapter for P40 | **$15.00** | Item `397278799706` or AliExpress |
| **Boot SSD** | 256GB SATA SSD | **$20.00** | [eBay used SSD](https://www.ebay.com/sch/i.html?_nkw=256gb+sata+ssd) |
| **TOTAL** | | **~$517** | |

### Alternative (Simpler)
Instead of building from barebones, buy the **$174.94 T5810 with 32GB** (Config 1 host) and add **one 32GB RDIMM stick (~$55)**. Total host cost = **~$230**, total rig = **~$520**.

### Notes
- 64GB is the practical sweet spot for 32B dense + large context, or 70B MoE with moderate CPU offload.
- You can now run **DeepSeek-R1-Distill-Qwen-32B** entirely in the P40's 24GB VRAM at Q4_K_M.

---

## Configuration 3: Max — 128GB System RAM

**Target:** Run **DeepSeek-V3 671B MoE** (or similar frontier-class MoE) via `--n-cpu-moe`, with all expert layers pinned to system RAM.

### Bill of Materials
| Component | Spec | Price | eBay Anchor |
|-----------|------|-------|-------------|
| **Host** | Dell T5810 barebones (no CPU/RAM/HDD) | **$92.00** | [eBay ~$69.99 + $23 ship](https://www.ebay.com/sch/i.html?_nkw=dell+precision+t5810+no+ram+no+hdd) |
| **CPU** | Xeon E5-2680 v3 (12C/24T) | **$40.00** | eBay CPU lots |
| **RAM** | 128GB (4×32GB) DDR4 ECC RDIMM | **$220.00** | [Server pulls](https://www.ebay.com/sch/i.html?_nkw=32gb+ddr4+ecc+rdimm) (~$55/stick) |
| **GPU** | Tesla P40 24GB + cooling fan | **$260.00** | Item `198009478303` or bundled `376635025151` |
| **Power adapter** | EPS adapter for P40 | **$15.00** | Item `397278799706` or AliExpress |
| **Boot SSD** | 256GB SATA SSD | **$20.00** | eBay used SSD |
| **Case fan** | 120mm high-static-pressure fan (if needed) | **$10.00** | Arctic P12 / Noctua NF-P12 |
| **TOTAL** | | **~$657** | |

### Alternative Host
There is a rare listing for a T5810 with 128GB pre-installed:
- **[Dell Precision T5810 E5-2660 v3 10C 20T 128GB RAM No HD — $200.00](https://www.ebay.com/sch/i.html?_nkw=Dell+Precision+T5810+E5-2660+v3+128GB+RAM+No+HD&fashion=1&_fsrp=0&_sop=12)** ([funtechtoy](https://www.ebay.com/usr/funtechtoy), US seller, 22 watchers). If available, this drops the total to **~$485** — an exceptional value. Availability fluctuates.

### Notes
- 128GB system RAM + 24GB VRAM = **152GB total memory**. DeepSeek-V3 671B at Q4_K_M needs roughly **~400GB** total, so you still need quantization or partial offload. At Q2_K or with aggressive offload, it becomes feasible.
- For a 671B MoE, only a fraction of parameters are active per token (~37B). The P40 handles attention + KV cache; the CPU handles expert routing + RAM-resident expert weights.
- Use `--no-mmap --mlock --cache-type-k turbo4 --cache-type-v turbo3` to maximize context window on the 24GB VRAM.

---

## What Each Config Can Actually Run

Assuming `llama.cpp` with Q4_K_M quantization unless noted:

| Model | Params | Budget (32GB) | Mid (64GB) | Max (128GB) |
|-------|--------|---------------|------------|-------------|
| Qwen2.5-Coder-7B | 7B | Fully VRAM ✅ | Fully VRAM ✅ | Fully VRAM ✅ |
| Llama 3.1 8B Instruct | 8B | Fully VRAM ✅ | Fully VRAM ✅ | Fully VRAM ✅ |
| Qwen2.5-Coder-14B | 14B | Fully VRAM ✅ | Fully VRAM ✅ | Fully VRAM ✅ |
| Qwen2.5-Coder-32B | 32B | Partial offload ⚠️ | Fully VRAM ✅ | Fully VRAM ✅ |
| DeepSeek-R1-Distill-Qwen-32B | 32B | Partial offload ⚠️ | Fully VRAM ✅ | Fully VRAM ✅ |
| DeepSeek-R1-Distill-Llama-70B | 70B | Too large ❌ | MoE/offload ⚠️ | Partial offload ⚠️ |
| Qwen3-35B-A3B (MoE) | 35B total | `--n-cpu-moe` ✅ | `--n-cpu-moe` ✅ | `--n-cpu-moe` ✅ |
| DeepSeek-V3 (MoE) | 671B total | Not feasible ❌ | Tight / Q2_K ❌ | `--n-cpu-moe` ✅ |

*"Fully VRAM" = fits in the P40's 24GB with no CPU offload.*  
*"Partial offload" = some layers in system RAM, 3–8 tok/s depending on CPU.*  
*"MoE" = `--n-cpu-moe` keeps expert weights in RAM, only active experts cross PCIe per token.*

---

## Important Build Notes

### 1. Above 4G Decoding
You **must** enable "Above 4G Decoding" in the T5810 BIOS. Without it, the system cannot map the P40's full 24GB BAR space and will fail to initialize. This is under BIOS → Integrated Devices → MMIO Above 4GB.

### 2. Headless Operation
The P40 has **no display outputs**. You will need:
- An IPMI/iDRAC connection (T5810 has optional iDRAC), or
- A cheap display adapter (e.g., GT 710, ~$15) in a second PCIe slot for local monitor/keyboard setup, or
- Install OS with another GPU, then swap to P40.

### 3. Linux Recommended
The P40 runs best on Linux with the **NVIDIA Data Center Driver** (535.x or compatible). Windows can work but Tesla drivers are more painful to install and WDDM mode can limit VRAM availability.

### 4. Power Cable Safety
**Do not** shave down a standard PCIe 6+2 pin connector and force it into the P40's EPS socket. The pinouts are different and you risk frying the card. Use a proper adapter or a bundled cable.

### 5. Cooling
The P40's passive heatsink is designed for server chassis with high front-to-rear airflow. In a tower workstation, you **must** have a blower fan attached. Without it, the card will thermal-throttle within minutes. Target <75°C under load.

### 6. RAM Population
The T5810 has 8 DIMM slots. For best performance, populate slots in the order defined in the Dell manual (typically starting furthest from the CPU). All configs here use ≤4 sticks, so slot placement is flexible.

---

## Cost Comparison: Build vs. Cloud API

| Rig Config | Upfront Cost | Power Draw* | 2-Year TCO |
|------------|-------------|-------------|------------|
| Budget (32GB) | ~$465 | ~300W | ~$465 + $315 = **~$780** |
| Mid (64GB) | ~$517 | ~320W | ~$517 + $336 = **~$853** |
| Max (128GB) | ~$657 | ~350W | ~$657 + $368 = **~$1,025** |

*Assuming 24/7 operation at ~$0.12/kWh. Power costs are rough estimates.*

By comparison, a **year** of ChatGPT Plus + Copilot Pro + Claude Pro can easily exceed **$600/year**. The Max rig pays for itself in under two years — and you own the hardware, can run uncensored local models, and pay no per-token fees.

---

## Alternative Host: HP Z440

If Dell T5810 stock is thin, the **HP Z440** is a drop-in alternative with nearly identical specs:
- 8 DDR4 DIMM slots, up to 128GB–256GB depending on BIOS and DIMM density.
- Xeon E5-1600/2600 v3 or v4.
- 700W PSU option (required for P40).
- eBay anchor: **$114.99** — *"HP Z440 Gaming PC PC 6-Core 3.60GHz E5-1650 v4 - No RAM HDD GPU or OS"* ([global_data_center_supply](https://www.ebay.com/usr/global_data_center_supply), **101 sold**).

The HP Z440 uses **proprietary PCIe power cables** from the motherboard. You will need an HP-specific adapter (or the bundled P40 + adapter listing) to convert to EPS. The T5810 is slightly preferred because its GPU power ecosystem is better documented in the homelab community.

---

## Final Recommendations

- **If you want the cheapest possible entry:** Buy the **$230 P40 with fan** (item `198009478303`) + the **$175 T5810 with 32GB** (digitalmind2000). Total **~$450**. You get 24GB VRAM and 32GB system RAM — enough to experiment with MoE offloading.

- **If you want the best balance:** Build the **Mid (64GB)** config. 64GB system RAM is the inflection point where 32B models run fully in VRAM and 70B MoE models become comfortable. Total **~$520**.

- **If you want to chase DeepSeek-V3:** Build the **Max (128GB)** config. At **~$680**, it is the cheapest way to run a 671B-parameter frontier model at home. No consumer GPU under $1,000 can match this capability.

---

*Report compiled from eBay sold/completed listings (May 2026), Dell/HP technical specifications, and `llama.cpp` MoE offloading documentation. Prices fluctuate; shop used listings aggressively for the best deals.*
