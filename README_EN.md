# AI-Driven Software Pricing (Sitemap + Prompt Engineering) — Replication Guide

This repository documents a **reproducible** protocol to estimate **market price ranges** of software products by **competitive analogy**, using **official sitemaps** as a proxy for “functional scope” and **LLMs** (ChatGPT Thinking, Gemini 3 Pro, and DeepSeek‑V3.2) with a **standardized prompt**.

> **Workflow summary**: (1) collect official sitemap → (2) save as PDF with metadata → (3) run 1 chat per software×LLM (isolated sessions) → (4) extract features (PDF-only) + competitive benchmarking (web) → (5) human verification → (6) normalization (annual basis + currency) → (7) relative validation indicators + Wilcoxon.

---

## 1) Prerequisites

### Accounts and interfaces
- **ChatGPT Thinking** (PDF upload + web browsing if available).
- **Gemini 3 Pro** (web interface).
- **DeepSeek‑V3.2** (web interface).
- A browser that supports **Print → Save as PDF**.

### Analysis tools (optional, recommended)
- A spreadsheet (Excel/Google Sheets/LibreOffice) **or** Python/R for calculations and statistical tests.
- Python 3.10+ (if you replicate with code):
  - `pandas`, `numpy`, `scipy`

---

## 2) Recommended repository structure

A consistent folder layout makes auditing easier:

```
.
├─ prompts/
│  ├─ base_prompt_en.md
│  └─ base_prompt_pt.md   (optional)
├─ data/
│  ├─ sitemaps_pdfs/
│  ├─ sitemaps_metadata/
│  ├─ references_mkt/     (public or anonymized market references)
│  └─ currency_rates/     (FX rates used for conversions, if applicable)
├─ runs/
│  ├─ transcripts_anon/
│  ├─ screenshots/
│  └─ session_log.csv
├─ outputs/
│  ├─ raw_tables/         (CSV/ODS)
│  └─ consolidated/       (CSV/ODS)
└─ analysis/
   ├─ indicators.xlsx      (or .ods/.csv)
   └─ wilcoxon.ipynb       (or .R)
```

---

## 3) Sitemap collection and updates (download → PDF)

### 3.1 Sitemaps used in the paper (official URLs)
- INTEGRA (Rede INTEGRA): `https://redeintegra.mec.gov.br/sitemap.xml`
- Salesforce: `https://www.salesforce.com/br/sitemap.xml`
- HYPE Innovation: `https://www.hypeinnovation.com/sitemap.xml`
- IdeaScale: `https://ideascale.com/post-sitemap.xml`
- Viima / HYPE Boards: `https://www.viima.com/sitemap.xml`
- Qmarkets: `https://www.qmarkets.net/post-sitemap1.xml`

> **If you replicate with other products**, change only the variables in the prompt (Section 6) and keep the exact same workflow.

### 3.2 How to generate the PDF (baseline procedure)
For each sitemap URL:

1. Open the sitemap URL in your browser.
2. Use **Print → Save as PDF** (ensure the PDF includes the full listing of URLs/entries).
3. Name the file using:
   - `sitemap_<SOFTWARE>_YYYY-MM-DD.pdf`
   - Example: `sitemap_INTEGRA_2025-08-13.pdf`

### 3.3 Minimum metadata (mandatory)
For each PDF, record:
- `captured_at` (local date/time)
- `sitemap_url` (exact URL)
- `pdf_filename`
- `sha256` (file hash; recommended)
- `pages` (PDF page count)
- `approx_urls` (estimated number of URLs)

Store this in `data/sitemaps_metadata/metadata.csv` (or JSON).

> Tip: SHA‑256 helps demonstrate that *the same input artifact* was used across different rounds/LLMs.

---

## 4) Experimental setup

### 4.1 Isolation rules (1 chat per combination)
Create **independent chats** for each combination:

- 6 products × 3 LLMs = **18 runs**
- In **each chat**, upload **only 1 PDF** (the target product’s sitemap).

Use a standardized run ID, e.g.:
- `RUN_001_INTEGRA_CHATGPT`
- `RUN_002_INTEGRA_GEMINI`
- `RUN_003_INTEGRA_DEEPSEEK`
- ...
- `RUN_018_QMARKETS_DEEPSEEK`

Log IDs in `runs/session_log.csv`.

### 4.2 Inference parameters (capture as evidence)
Document (screenshots + logs) the parameters used:

**ChatGPT 5.1 Thinking**
- temperature = 0.2
- top_p = 1.0
- n = 1
- frequency penalty = 0.0
- presence penalty = 0.0

**Gemini 3 Pro (default/balanced)**
- temperature = 1.0
- top_p = 0.95
- n = 1
- frequency penalty = 0.0
- presence penalty = 0.0

**DeepSeek‑V3.2 (default/balanced)**
- temperature = 0.7
- top_p = 0.9–0.95
- n = 1
- frequency penalty = 0.0
- presence penalty = 0.0

> Note: web UIs may not expose all internal details (system prompt, caching, exact context limits). Capture everything that is **observable**.

---

## 5) Protocol execution (LLM) — step by step

### 5.1 Stage A — Feature extraction (“PDF-only mode”)
Goal: infer features **strictly** from the sitemap PDF content.

1. Start a new chat (no prior context).
2. Upload the target product’s sitemap PDF.
3. Paste the **standardized prompt** (Section 6) and fill:
   - `{{SOFTWARE_NAME}}`
   - `{{COMPARISON_FOCUS}}`
   - `{{SITEMAP_URL}}`
4. **Operational rule**: in this stage, ensure the model is not using external sources; if the UI allows it, work in a “file-only” setting.

**Expected outputs (minimum):**
- A structured list of inferred features/modules derived from the sitemap.

### 5.2 Stage B — Competitive benchmarking (with web browsing, when applicable)
Goal: identify comparable competitors and collect pricing ranges/models from public sources.

1. Enable web browsing (if offered).
2. Reinforce the requirement to:
   - cite URLs;
   - state pricing model (per user, per instance, enterprise, etc.);
   - prioritize **official sources**, using aggregators only when primary sources are unavailable.

**Expected outputs (minimum):**
- Benchmarking table (competitor, vendor, pricing model, prices/ranges, source).
- Feature comparison table (sitemap-derived vs public docs).
- Final estimated annual range for the target product (with anchors and rationale).
- List of all consulted links.

### 5.3 Artifact capture (mandatory)
For each run (RUN):

- Export the chat transcript (or copy into `runs/transcripts_anon/`).
- Save screenshots of:
  - the displayed model/version;
  - parameters (if visible);
  - response blocks containing sources (URLs).
- Fill `runs/session_log.csv` with:
  - `run_id`, `software`, `llm`, `date_time`, `pdf_filename`, `model_version_displayed`
  - `web_enabled` (yes/no)
  - `notes` (discrepancies, UI failures, etc.)

---

## 6) Standardized prompt (template)

Save this as `prompts/base_prompt_en.md` and use it **without edits**, replacing only variables.

```text
Estimate the market value of the software {{SOFTWARE_NAME}} by positioning it and comparing it directly with other solutions and platforms available in the market, focusing on {{COMPARISON_FOCUS}}.

Context: The AI will act as a product analyst tasked with evaluating the features of {{SOFTWARE_NAME}} (publicly available through its sitemap) and conducting competitive benchmarking. The final goal is to determine an estimated market value range for {{SOFTWARE_NAME}}, based on the pricing and business models of competing products specialized in {{COMPARISON_FOCUS}}.

AI workflow (Steps to follow):

1) Analyze and Extract Features of {{SOFTWARE_NAME}}:
Action: Analyze the sitemap ({{SITEMAP_URL}}) and identify the modules and features that {{SOFTWARE_NAME}} offers.
Output: Provide a clear, structured list of features to serve as the basis for comparison.

2) Market Research Focused on {{COMPARISON_FOCUS}} Software:
Action: Search for commercial competing products (SaaS or licensed) that are direct/analogous competitors.
Search terms (examples):
- "{{COMPARISON_FOCUS}} software price"
- "{{COMPARISON_FOCUS}} platform cost"
- "{{SOFTWARE_NAME}} competitors"
- "{{COMPARISON_FOCUS}} software pricing"
- "enterprise {{COMPARISON_FOCUS}} pricing comparison"

3) Comparative Analysis and Pricing Data Collection:
Action: For each competitor, collect (from public sources) the name, pricing model, prices/ranges, and key features.
Output: A benchmarking table comparing {{SOFTWARE_NAME}} with the identified products.

4) Feature Comparison Table:
Action: Compare {{SOFTWARE_NAME}} features (from the sitemap only) with competitors (from public documentation).
Output: A clear, organized feature comparison table.

5) Market-Analogy Cost Estimation:
Action: Provide an annual estimate (minimum and maximum range) and justify it with comparative anchors.

Scope of deliverables:
- Feature list (from the sitemap)
- Competitive benchmarking table (with sources)
- Feature comparison table
- Estimated market value range (annual) with justification
- List of links used

The AI MUST NOT:
- Use Function Points (FPA).
- Include products that are not primarily focused on {{COMPARISON_FOCUS}}.
- Make assumptions about development or infrastructure costs.
```

---

## 7) Human verification (manual checking)

Human verification is **part of the protocol** and must not be skipped.

### 7.1 What to verify
For each RUN:

1. **Features**: confirm items are supported by the sitemap and/or relevant official pages.
2. **Competitors**: confirm competitors are truly comparable within the target focus.
3. **Pricing**: confirm:
   - the price exists on the cited page;
   - the licensing model is correct (per user/instance/enterprise);
   - billing periodicity (monthly/annual) is correctly interpreted.
4. **Sources**: reject non-primary links when an official source exists.

### 7.2 What to do when discrepancies are found
- Log the discrepancy in `runs/session_log.csv` (`notes` field).
- **Discard or rerun** the session.
- When rerunning, apply only *minimal* reinforcements (e.g., “official sources only” and “must list URLs”).

---

## 8) Normalization and relative validation indicators (AI vs MKT)

### 8.1 Standardization rules (common basis)
Before comparing estimates to market references:

1. **Range → midpoint**  
   - `AI_mid = (AI_min + AI_max) / 2`
2. **Monthly → annual** (when applicable)  
   - `annual = monthly * 12`
3. **Currency conversion** (when needed)  
   - Record the FX rate and date (store in `data/currency_rates/`)

### 8.2 Indicators (per software×LLM)
Define `AI` (estimated value, normalized) and `MKT` (reference value, normalized) in the **same currency and time basis**.

Compute:

- Difference: `diff = AI - MKT`
- Absolute difference: `abs_diff = |AI - MKT|`
- Relative deviation: `rel_diff = (AI - MKT) / MKT`
- Absolute Percentage Error: `APE(%) = (|AI - MKT| / MKT) * 100`

Export these to `outputs/consolidated/validation_indicators.csv`.

### 8.3 Suggested spreadsheet columns
- `software`, `llm`, `run_id`
- `ai_min`, `ai_max`, `ai_mid`, `ai_currency`, `ai_basis` (per user/instance/enterprise)
- `mkt_value`, `mkt_currency`, `mkt_basis`, `mkt_source_type`
- `conversion_notes` (monthly→annual, FX, etc.)
- `diff`, `abs_diff`, `rel_diff`, `ape_percent`

---

## 9) Statistical test (Wilcoxon signed-rank) — AI vs MKT

Goal: test whether there is a systematic difference between paired `AI` and `MKT` values (small n).

- Null hypothesis: `median(AI - MKT) = 0`
- Alternative (two-sided): `median(AI - MKT) ≠ 0`
- Significance level: `α = 0.05`

### 9.1 Python example (SciPy)

Create `analysis/wilcoxon.py`:

```python
import pandas as pd
from scipy.stats import wilcoxon

df = pd.read_csv("outputs/consolidated/validation_indicators.csv")

pairs = df.dropna(subset=["ai_mid", "mkt_value"]).copy()
diff = pairs["ai_mid"] - pairs["mkt_value"]

stat, p = wilcoxon(diff, alternative="two-sided", zero_method="wilcox")

print(f"W = {stat:.4f} | p-value = {p:.6f} | n = {len(diff)}")
```

### 9.2 Interpretation
- If `p < 0.05`: reject H0 (evidence of systematic difference).
- If `p ≥ 0.05`: fail to reject H0 (insufficient evidence of systematic difference).

---

## 10) Technical replication (robustness check)

For each software×LLM, run a **second round**:

- New chat
- Same PDF
- Same prompt
- Same browsing rule (PDF-only vs web)

Compare:
- Similarity of feature lists
- Stability of estimated ranges (order of magnitude)
- Overlap of cited public sources

Log divergences and how they were handled (re-check sitemap/pages; rerun/discard).

---

## 11) Quick checklist (before publishing results)

- [ ] PDFs saved with naming convention and metadata (date/time + URL + hash)
- [ ] 1 chat per software×LLM (no prior context)
- [ ] Prompt used without edits (only variables changed)
- [ ] Evidence archived (transcript + links + screenshots)
- [ ] Human verification recorded (OK/error + keep/discard decision)
- [ ] Normalization applied (midpoint, annualization, FX documented)
- [ ] Relative indicators computed
- [ ] Wilcoxon run on valid pairs

---

## 12) License and ethics

This repository should contain:
- only **non-confidential artifacts**;
- **anonymized** transcripts when needed;
- public market references or anonymized values when restrictions apply.
