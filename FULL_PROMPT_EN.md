# Standard Prompt (EN) — Sitemap-Driven Market Pricing by Competitive Analogy

Use this prompt **without any edits** (except the variables) in a **new chat** for each run (software × model).
**Operational step**: upload the sitemap PDF before pasting the prompt.

## Variables to replace
- `{{SOFTWARE_NAME}}` — target software name
- `{{COMPARISON_FOCUS}}` — category/segment (e.g., innovation management, idea management, etc.)
- `{{SITEMAP_URL}}` — official sitemap URL used to generate the PDF

---

## PROMPT (copy & paste)

Estimate the market value of **{{SOFTWARE_NAME}}** by positioning it and comparing it directly with other solutions and platforms available in the market, focusing on **{{COMPARISON_FOCUS}}**.

**Context:** The AI will act as a product analyst tasked with evaluating the features of **{{SOFTWARE_NAME}}** (publicly available through its sitemap) and conducting competitive benchmarking. The final goal is to determine an estimated market value range for **{{SOFTWARE_NAME}}**, based on the pricing and business models of competing products specialized in **{{COMPARISON_FOCUS}}**.

**AI workflow (Steps to follow):**

### 1) Feature analysis and extraction for {{SOFTWARE_NAME}}
**Action:** Analyze the sitemap (**{{SITEMAP_URL}}**) and identify the modules and features that **{{SOFTWARE_NAME}}** offers.  
**Output:** Produce a clear, structured list of **{{SOFTWARE_NAME}}** features to serve as the basis for comparison.

### 2) Market research focused on {{COMPARISON_FOCUS}} software
**Action:** Conduct market research to find commercial competing products (SaaS or licensed) that are direct/analogous competitors to **{{SOFTWARE_NAME}}** in the **{{COMPARISON_FOCUS}}** niche.  
**Search terms should be specific (examples):**
- "{{COMPARISON_FOCUS}} software price"
- "{{COMPARISON_FOCUS}} platform cost"
- "{{SOFTWARE_NAME}} competitors"
- "{{COMPARISON_FOCUS}} software pricing"
- "enterprise {{COMPARISON_FOCUS}} pricing comparison"

**Research focus:** Identify established vendors and products that truly operate in the **{{COMPARISON_FOCUS}}** market and document their pricing models.

### 3) Comparative analysis and pricing data collection
**Action:** For each **{{COMPARISON_FOCUS}}** competitor found, collect information from public sources (official websites, pricing pages, public documentation, reviews, and articles), including:
- Product and vendor name
- Pricing model (e.g., per user/month, modular pricing, annual plan, etc.)
- Prices or price ranges (any publicly available cost information)
- Key features (to validate comparability with **{{SOFTWARE_NAME}}**)

**Output:** A benchmarking table comparing **{{SOFTWARE_NAME}}** with the identified **{{COMPARISON_FOCUS}}** products.

### 4) Feature comparison table
**Action:** Build a detailed table comparing **{{SOFTWARE_NAME}}** features (**inferred only from the uploaded sitemap PDF**) with competitor features (from public documentation), highlighting strengths and gaps.  
**Output:** A clear, organized feature comparison table.

### 5) Market-analogy cost estimation
**Action:** Based on the comparative data, formulate a market value estimate for **{{SOFTWARE_NAME}}** using competitive analogy.
- **Annual price range:** determine a minimum and maximum range (e.g., “a platform with these capabilities would likely cost between X and Y per year”).
- **Justification:** clearly explain which competitors/anchors support the range and why the comparison is valid, considering feature scope.

---

## Objective scope (deliverables)
- Feature list for **{{SOFTWARE_NAME}}** (derived from the sitemap PDF)
- Competitive benchmarking table (competitors, pricing model, prices/ranges, and **sources/links**)
- Feature comparison table
- Estimated market value (annual range) with justification grounded in market analysis
- List of sources (links) used

## The AI MUST NOT
- Use Function Points (FPA).
- Include products that are not primarily focused on **{{COMPARISON_FOCUS}}**.
- Make assumptions about development or infrastructure costs (the focus is strictly the product’s **market value**).

