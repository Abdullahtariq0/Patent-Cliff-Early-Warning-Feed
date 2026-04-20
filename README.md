# Patent Cliff Early Warning Feed

> **1,187 brand-name pharmaceutical drugs ranked by patent cliff urgency — enriched with CMS spend, exclusivity protection, patent complexity, safety signal, and a composite commercial score. Weekly refresh. Zero ETL.**

[![Snowflake Marketplace](https://img.shields.io/badge/Snowflake-Marketplace-29B5E8?style=flat-square&logo=snowflake&logoColor=white)](https://app.snowflake.com/marketplace)
[![Refresh](https://img.shields.io/badge/Refresh-Weekly%20Monday%2004%3A00%20UTC-0366d6?style=flat-square)](https://github.com)
[![Coverage](https://img.shields.io/badge/Coverage-US%20FDA%20Orange%20Book%20%2B%20CMS%20Part%20D-6f42c1?style=flat-square)](https://github.com)
[![Status](https://img.shields.io/badge/Status-Production%20Validated-2ea44f?style=flat-square)](https://github.com)
[![Built By](https://img.shields.io/badge/Built%20by-The%20Data%20Developers-black?style=flat-square)](https://thedatadevelopers.com)

---

## What You Get

One Snowflake table. One row per brand drug product. Everything you need to act on patent cliff exposure before competitors do.

`PCF_PRODUCT.PATENT_CLIFF_FEED` delivers 1,187 FDA-listed brand-name drug products ranked by a **composite cliff score (0–100)** built from four dimensions:

| Dimension | Weight | What it measures |
|-----------|--------|-----------------|
| CMS spend magnitude | 40% | How big is the revenue pool at risk? |
| Time urgency | 30% | How soon does the generic entry window open? |
| Patent complexity | 20% | How hard will it be for generics to enter? |
| Safety signal direction | 10% | Is post-market adverse event reporting rising or falling? |

Every product is also bucketed into a **cliff tier** for instant triage:

| Tier | Window | Description |
|------|--------|-------------|
| `IMMINENT` | ≤ 1 year | Act now — generic entry is imminent |
| `NEAR_TERM` | 1–2 years | Plan now — within the typical BD cycle |
| `WATCH` | 2–3 years | Monitor — approaching strategic horizon |
| `PIPELINE` | 3–5 years | Build into pipeline planning |
| `LONG_DATED` | > 5 years | Early awareness |

**701 products** are currently inside the 36-month cliff window.

---

## The Problem This Solves

Pharma companies spend **$80,000–$250,000 per year** on proprietary databases (Cortellis, Evaluate Pharma, Citeline) for patent cliff intelligence. These tools have three structural gaps that PCF Feed closes:

**1. No commercial prioritisation.** Existing tools do not cross-link patent expiry dates with actual Medicare spending data. A $4B cliff drug and a $40M cliff drug appear equivalent in their outputs. PCF Feed scores every drug against real CMS Part D spend — so your team focuses on what actually moves the needle.

**2. Fragmented data.** Patent timing, exclusivity protection, safety signals, and IP complexity live in separate vendor modules. Reconciling them takes analyst days per scan cycle. PCF Feed cross-links five data sources into a single query-ready row.

**3. Enterprise pricing locks out mid-market.** Mid-size biotech and emerging pharma cannot afford enterprise database subscriptions. PCF Feed delivers the same cliff intelligence at 10–20× lower cost via Snowflake Marketplace — available immediately, no implementation, no contract negotiation.

---

## What's in the Table

`PATENT_CLIFF_FEED` has 35 columns across six intelligence domains.

### Patent cliff timing

| Column | Type | Description |
|--------|------|-------------|
| `INGREDIENT` | TEXT | Normalised lowercase INN active ingredient name |
| `TRADE_NAME` | TEXT | Brand name in uppercase |
| `OB_APPLICANT` | TEXT | Orange Book sponsor name |
| `EARLIEST_PATENT_EXPIRY` | DATE | Date of earliest active patent expiry |
| `LATEST_PATENT_EXPIRY` | DATE | Date of latest active patent expiry |
| `PATENT_COUNT` | NUMBER | Total active Orange Book patents |
| `SUBSTANCE_PATENT_COUNT` | NUMBER | Drug substance (NCE) patents only |
| `PRODUCT_PATENT_COUNT` | NUMBER | Formulation / method patents |
| `DAYS_TO_EARLIEST_EXPIRY` | NUMBER | Days from today to earliest expiry. Negative = expired |
| `IN_36MO_CLIFF_WINDOW` | BOOLEAN | TRUE if generic entry within 3 years |
| `IN_60MO_CLIFF_WINDOW` | BOOLEAN | TRUE if generic entry within 5 years |
| `EFFECTIVE_GENERIC_ENTRY_DATE` | DATE | MAX(patent expiry, exclusivity expiry) — the **true** generic entry floor |
| `DAYS_TO_EFFECTIVE_ENTRY` | NUMBER | Days to effective generic entry |
| `LATEST_EXCLUSIVITY_DATE` | DATE | Latest Orange Book exclusivity expiry |
| `EXCLUSIVITY_CODES` | TEXT | Pipe-delimited exclusivity codes (e.g. `M-295 \| PED \| I-926`) |
| `CLIFF_TIER` | TEXT | IMMINENT / NEAR_TERM / WATCH / PIPELINE / LONG_DATED |

> **Why `EFFECTIVE_GENERIC_ENTRY_DATE` matters:** The earliest patent expiry is not the generic entry date. Exclusivity protections — paediatric (PED), orphan (ODE), NCE, and others — can extend the exclusivity window by months or years beyond patent expiry. `EFFECTIVE_GENERIC_ENTRY_DATE` is the later of all patent and exclusivity dates, giving you the true floor that generics cannot enter before.

### CMS commercial magnitude

| Column | Type | Description |
|--------|------|-------------|
| `SPEND_2023` | FLOAT | CMS Part D Medicare spend 2023 (USD) |
| `CLAIMS_2023` | FLOAT | CMS Part D Medicare claims 2023 |
| `BENES_2023` | FLOAT | Medicare beneficiaries 2023 |
| `SPEND_2022–2019` | FLOAT | Annual spend each year 2019–2022 |
| `TOTAL_SPEND_5YR` | FLOAT | Total 5-year Medicare spend 2019–2023 |
| `SPEND_CAGR_19_23` | FLOAT | Spend per unit CAGR 2019–2023. Positive = growing market |
| `SPEND_TREND` | TEXT | GROWING / DECLINING / STABLE / NO_CMS_DATA |

> **Coverage note:** CMS Part D covers Medicare Part D (age 65+) prescription drug spending. Actual total market size is typically 2–4× the CMS figure for most drugs when commercial insurance and Medicaid are included.

### Drug scientific profile

| Column | Type | Description |
|--------|------|-------------|
| `ATC_LEVEL1` | TEXT | WHO ATC Level 1 therapeutic class |
| `ATC_LEVEL2` | TEXT | WHO ATC Level 2 sub-class |
| `MECHANISM_OF_ACTION` | TEXT | Pharmacological mechanism (pipe-delimited if multiple) |
| `TARGET_NAMES` | TEXT | Molecular targets (pipe-delimited) |
| `ORPHAN_FLAG` | BOOLEAN | TRUE if orphan designation exists |
| `BLACK_BOX_WARNING` | NUMBER | 1 if FDA black box warning exists |
| `FIRST_APPROVAL_YEAR` | NUMBER | Year of first regulatory approval |

### Patent owner and complexity

| Column | Type | Description |
|--------|------|-------------|
| `PRIMARY_ASSIGNEE` | TEXT | Patent holder organisation (modal across all patents) |
| `AVG_CLAIMS_PER_PATENT` | FLOAT | Average patent claim count. Higher = more litigation surface |
| `MAX_CLAIMS_SINGLE_PATENT` | NUMBER | Highest claim count single patent. Proxy for broadest protection |
| `PV_MATCH_PCT` | FLOAT | % of Orange Book patents matched to PatentsView. Below 50% = IP complexity flag |

### Post-market safety signal

| Column | Type | Description |
|--------|------|-------------|
| `FAERS_TOTAL_CASES` | NUMBER | Total adverse event cases in FDA FAERS 2018–2025 |
| `FAERS_DEATH_CASES` | NUMBER | Cases with death outcome |
| `FAERS_HOSP_CASES` | NUMBER | Cases with hospitalisation outcome |
| `FAERS_SERIOUS_PCT` | FLOAT | % of cases with serious outcome |
| `SAFETY_TREND` | TEXT | RISING / DECLINING / STABLE / NO_FAERS_DATA |

### Composite score and metadata

| Column | Type | Description |
|--------|------|-------------|
| `CLIFF_COMMERCIAL_RANK` | FLOAT | Composite score 0–100 |
| `LAST_REFRESHED_AT` | TIMESTAMP | Timestamp of most recent table rebuild |
| `SNAPSHOT_DATE` | DATE | Snapshot date for time-series tracking |

---

## Who Uses This

### Pharma BD and M&A teams
Monitor the competitive patent landscape for out-licensing opportunities and acquisition timing. When a competitor's highest-revenue drug hits `CLIFF_TIER = 'IMMINENT'`, the window for lifecycle management deal-making is open. Filter by `ATC_LEVEL1` to scope to your therapeutic areas and `PRIMARY_ASSIGNEE` to target specific companies.

> *"Two to three analyst days per BD scan cycle replaced by a 60-second query."*

### Market access and pricing directors
Plan generic entry defence 18–24 months in advance. Set up an `OB_APPLICANT` filter to watchlist your own portfolio and track `DAYS_TO_EFFECTIVE_ENTRY` week-over-week as the table refreshes automatically. `EXCLUSIVITY_CODES` reveals remaining defence mechanisms. `SPEND_CAGR_19_23` informs rebate negotiation positioning with payers.

### IP and patent strategy teams
Map competitor patent portfolios for freedom-to-operate analysis. `AVG_CLAIMS_PER_PATENT` and `MAX_CLAIMS_SINGLE_PATENT` indicate litigation cost for Para IV challenges. `PV_MATCH_PCT < 50%` on a high-spend drug flags incomplete citation coverage warranting deeper FTO review. Group by `PRIMARY_ASSIGNEE` to see total portfolio-level cliff exposure per company — multiple drugs from one assignee hitting `NEAR_TERM` simultaneously is an M&A signal.

### Generic manufacturers
Prioritise ANDA filing pipelines by revenue opportunity, time-to-market, and litigation risk. `TOTAL_SPEND_5YR` is the 5-year Medicare revenue pool for modelling Day-1 generic market share. `EFFECTIVE_GENERIC_ENTRY_DATE` — not just the earliest patent — is the true entry floor. `PATENT_COUNT` and `AVG_CLAIMS_PER_PATENT` together score Para IV litigation risk before you commit development resources.

### Life sciences equity analysts
Quantify patent cliff risk in pharma company revenue bases for buy/sell recommendations. Sum `SPEND_2023` across all `IMMINENT` and `NEAR_TERM` products for a target company to model revenue at risk. `ORPHAN_FLAG = TRUE` drugs command premium pricing and may qualify for extended exclusivity — an upside offset to the cliff headline. `SAFETY_TREND = 'RISING'` on large-spend drugs is a potential earnings surprise risk that standard patent cliff tools miss entirely.

---



---

## Validated Anchor Results

These known outputs confirm data quality. Use them to spot-check after querying.

| Drug | Trade name | Cliff score | Tier | Days to expiry | 2023 spend |
|------|-----------|------------|------|----------------|-----------|
| linagliptin | TRADJENTA | 97.8 | IMMINENT | 37 | $2.59B |
| enzalutamide | XTANDI | ~93 | NEAR_TERM | — | — |

| Metric | Confirmed value |
|--------|----------------|
| Total products | 1,187 |
| In 36-month cliff window | 701 (59.1%) |
| Has CMS spend data | 707 (59.6%) |
| Has ChEMBL enrichment | 947 (79.7%) |
| Has PatentsView coverage | 1,147 (96.6%) |
| Zero duplicate (APPL_NO, PRODUCT_NO) pairs | Confirmed |

---

## Common Questions

**Why is `EFFECTIVE_GENERIC_ENTRY_DATE` later than `EARLIEST_PATENT_EXPIRY` for many drugs?**
Orange Book exclusivity protections — paediatric extensions (PED), NCE exclusivity, orphan drug exclusivity (ODE), and others — run independently of patents and can extend the period before generic entry by months or years. `EFFECTIVE_GENERIC_ENTRY_DATE` is the true floor: the later of all patent expiries and all exclusivity expiries. Always use this column for generic entry modelling, not the raw patent date.

**What does a high `AVG_CLAIMS_PER_PATENT` mean for generic entry?**
More patent claims = more litigation surface = more expensive Para IV challenges. Drugs with high average claim counts are the ones where generic manufacturers historically face the most sustained patent litigation before entry. This column is a proxy for regulatory and litigation cost, not just legal complexity.

**Why does `PV_MATCH_PCT` matter?**
PatentsView covers US patents from roughly 1976 onward and is the source of `PRIMARY_ASSIGNEE` and `AVG_CLAIMS_PER_PATENT`. If a drug's Orange Book patents match at less than 50%, some of those patents may be older, continuation patents, or foreign-origin filings with limited PatentsView coverage — meaning the IP picture is incomplete and warrants direct Orange Book or PAIR investigation.

**The `SPEND_2023` is lower than published market size for this drug. Why?**
CMS Part D covers Medicare prescription drug spending (age 65+). For most branded drugs, the total market including commercial insurance and Medicaid is 2–4× the CMS figure. PCF Feed uses CMS as a consistent, fully public commercial sizing anchor. Apply a market multiplier appropriate to the drug's indication and patient demographics to estimate total market revenue at risk.

**Is `SAFETY_TREND = 'RISING'` a red flag for generic entry?**
Not necessarily for generic manufacturers — a rising safety signal does not automatically indicate market withdrawal. It is more relevant for BD teams evaluating whether a branded drug may face regulatory action before its patent cliff, and for equity analysts modelling downside scenarios. A rising safety signal on a drug with high `FAERS_SERIOUS_PCT` warrants deeper pharmacovigilance investigation.

---

## Data Coverage

| Dimension | Coverage |
|-----------|----------|
| Geographic scope | United States (FDA Orange Book, CMS Medicare Part D) |
| Patent data | US FDA Orange Book active RX patents |
| Commercial spend | CMS Medicare Part D 2019–2023 |
| Drug enrichment | Global (ChEMBL 34 — WHO ATC + MoA) |
| Safety signal | Global FAERS reports (248 reporter countries) |
| Refresh cadence | Weekly — every Monday 04:00 UTC |
---
