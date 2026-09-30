# Hi there 👋

I'm Shubhay — a CS & Engineering undergrad at **UCLA** who spends most of his time on applied deep learning: sequence models for financial microstructure, multimodal architectures for immunotherapy, and agent systems that do real paperwork.

I care about a specific thing: **results that survive contact with reality.** A backtest that only works at zero transaction cost isn't alpha. A classifier that scores well because of sequence leakage isn't a classifier. Most of the engineering below is the unglamorous work of proving a result isn't an artifact.

- 🔭 Currently building cross-asset lead–lag models and immunopeptidomics classifiers
- 🌱 Currently learning market microstructure, execution modeling, and equivariant architectures
- 💬 Ask me about sparse attention, ranking losses, cost-curve backtesting, or MHC antigen presentation
- ⚡ Fun fact: I argued federal policy competitively for four years, which turns out to be excellent training for defending a research claim

---

## 🧪 Projects

### 1. Cross-Asset Lead–Lag Discovery in Crypto Markets
**xLSTM · sparse cross-asset attention · listwise ranking · submitted to ICAIF '26 (Milan)**

Crypto markets price information into different assets at different times. This project asks whether a deep model can *forecast* the cross-section of returns while simultaneously *recovering* which assets lead which — using the attention matrix itself as a directed, per-bar graph of asset leadership.

The central problem is that a graph-shaped attention pattern doesn't prove inter-asset lead–lag. It can just as easily encode an asset's own short-horizon reversal, and no standard accuracy metric distinguishes the two.

**The core idea: a control model that isolates the claim.**

I built **SelfLagNet** — architecturally identical to the main model, but with the cross-asset leader index removed, so it can only attend to each asset's *own* lag bank. A model earns the label "genuinely lead–lag" only when its per-bar forecasting skill separates from SelfLagNet's on the same bars. That separation is the result; raw accuracy is not.

<details>
<summary><b>Technical detail</b></summary>

**Data pipeline**
- Fixed 20-asset Binance USDT cross-section (BTC, ETH, SOL, LINK, ARB, OP, PEPE, WIF, …)
- Three frequencies: 5-minute, hourly, daily
- Feature tensor `X[T, N, 8]` with availability masking and per-asset entry indices
- First half of 2024 reserved as holdout; normalization statistics computed on the train window only
- Seeded end to end (`numpy` + `torch`), resumable, leakage-safe by construction

**Architectures**
| Model | Encoder | Attention |
|---|---|---|
| **xLSTM** | exponentially-gated recurrent | sparse, over all assets' current states |
| **LeadLagNet** | causal temporal-convolutional | sparse, over a bank of explicit historical lags |
| **SelfLagNet** | identical to its parent | confined to the asset's own lag bank *(control)* |

</details>

---

### 2. Multimodal Neoantigen Classification for Cancer Vaccines
**Transformer fusion · bidirectional cross-attention · ESM-2 · presented at ICDLSBE 2026 (Japan)**

Cancer immunotherapy depends on T-cells recognizing malignant cells via peptides presented by the MHC. Finding tumor-specific peptides is bottlenecked by mass-spectrometry immunopeptidomics — low antigen abundance, confounding post-translational modifications.

Existing tools (NetMHCpan, MARIA, ImmunoStruct) predict *binding affinity* or immunogenicity. They operate in a contextual vacuum: they can't tell you whether a presented peptide came from a malignant cell. This model predicts **malignancy context** directly, across both presentation pathways.

**Why Class II matters.** Nearly every computational tool is skewed toward MHC Class I — shorter peptides, tighter structural constraints, cheaper to model. But advanced tumors downregulate Class I expression to evade exactly those therapies. Covering Class II reaches the tumors that escape.

<details>
<summary><b>Technical detail</b></summary>

**Data**
- 43,678 Class I samples across 27 HLA alleles; 9,633 Class II samples across 54 alleles
- Sourced from mass-spectrometry-confirmed complexes in IEDB and PCI-DB
- Length-stratified to physical binding-pocket constraints: 8–12 aa (Class I), 13–25 aa (Class II)
- **Peptide-grouped splitting** to eliminate algorithmic leakage — without it, near-identical sequences straddle the train/test boundary and every metric inflates

**Architecture** — three independent streams fused by cross-attention:

| Stream | Representation |
|---|---|
| Biophysical | standardized vector of 8 biochemical properties |
| Sequence | BLOSUM62 substitution scores **or** pre-trained ESM-2 embeddings |
| Allele | explicit amino-acid configuration of the HLA binding-groove pseudosequence |

Each stream passes through its own Transformer block; a **bidirectional cross-attention mechanism** then couples them, directly simulating the peptide–groove interaction interface. Separate models are trained per MHC class. AdamW, cross-entropy, early stopping on validation plateau.

**Results (held-out test set)**

| Configuration | ROC AUC |
|---|---|
| Class I — ESM-2 embeddings | **0.925** |
| Class I — BLOSUM62 | 0.910 |
| Class II — BLOSUM62 | **0.908** |
| PCA clustering baseline | ~0.531 (near chance) |

</details>

---

### 3. Constructa — AI Construction Foreman
**Multi-agent orchestration · Fetch.ai ASI:One · Browserbase · Three.js — 2× sponsor award, Cal Hacks AI**

Construction projects run an average of **one year over schedule and 30% over budget**. In heavily regulated states the bottleneck isn't labor or land — it's a dense, overlapping stack of permits, inspections, and paperwork that no single human foreman can track in real time.

Constructa is an AI foreman for that stack: pick a parcel on a national map, describe what you want to build in one sentence, and get back an explorable 3D model plus the exact regulatory sequence your building type requires — with six specialist agents running alongside as live chat tabs.

<details>
<summary><b>Technical detail</b></summary>

**The architectural idea: ASI:One as universal router and fallback.**

Every agent interaction attempts its job with the best specialized tool first. If that tool fails, is unavailable, or returns low confidence, the request falls through to ASI:One, which interprets intent and routes to the best-matched agent on the Agentverse marketplace. The result is a system with **no dead ends** — a filing agent that drops mid-demo, a scorer that returns nothing for an obscure district, an RFI outside any agent's training all recover gracefully instead of breaking the user's flow.

**Six specialist agents**
`daily briefing` · `RFI resolution` · `compliance watchdog` · `site photo classification` · `subcontractor coordination` · `permit & exemption research`

**Map system** — a three-tier drill built on real geography:
1. US states, shaded by aggregate construction-consensus score
2. Congressional-district boundary polygons fetched from a national GeoJSON index (batched, cached) with a density heatmap clipped to the state outline
3. District deep-dive → live Browserbase scraping with geotargeted proxies → ASI:One factor-scored buy/hold/avoid guide

Covers all 50 states and 435 congressional districts. Scoring decomposes into three weighted sub-scores plus a `buildablePct` surfaced per district, grounded by a vector RAG index over district research documents so similar districts retrieve similar prior rationale.

**Stack**

| Layer | Choice |
|---|---|
| Framework | TanStack Start (React, SSR, file routing) |
| Data | TanStack Query |
| 3D | Three.js via React Three Fiber + drei |
| Map | MapLibre GL + deck.gl |
| Animation | GSAP + ScrollTrigger |
| Web API | Hono, mounted inside TanStack Start at `/api/*` |
| State / cache | Redis |
| LLM | Anthropic Claude |
| Agent service | Python FastAPI (Deepgram voice + Fetch.ai uAgent watchdog) |
| Auth | Clerk |
| Observability | Arize on every agent and classifier call |

**Generated artifacts, not just chat.** 3D building models are authored as Three.js code by an agent from a natural-language prompt — not pre-baked video. Cal/OSHA DOSH 41-1 forms and RFIs are auto-filled from inferred project context. Redis is the single shared state layer connecting the map, the project workspace, and every agent's memory.

</details>

---

### Earlier: MicroNet — Exoplanet Detection in Microlensing Events
**Wavelet transforms · CNN classification · CSEF Grand Award (Physics & Astronomy)**

A wavelet-transform frequency-image framework over 600+ KMTNet light curves, achieving a **311% improvement** in planetary signal extraction over baseline methods and **93% classification accuracy at ~30,000× the speed of MCMC fitting** — cutting analysis runtimes by up to six years of compute.

---

## 🧰 Toolbox

```
Languages     Python · TypeScript / JavaScript · C++
Deep Learning PyTorch · TensorFlow · xLSTM · Transformers · cross-attention
              sparse attention · ESM-2 · temporal convolutional networks
Quant         walk-forward validation · Diebold–Mariano testing · listwise ranking
              long-short portfolio construction · transaction cost modeling
Web           React · Next.js · TanStack · Node.js · Hono · FastAPI · Three.js
Infra         Redis · MongoDB · Docker · Browserbase · Arize · Fetch.ai ASI:One
Hardware      PCB design · embedded microcontrollers · semiconductor fabrication
```

## 📫 Reach me

Open to collaboration on quantitative finance research, computational immunology, and agent infrastructure.

- 💼 [LinkedIn](https://www.linkedin.com/in/shubhay-choubey-17b78a272/)
