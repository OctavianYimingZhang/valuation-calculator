---
name: valuation-calculator
description: >
  Archetype-driven public company valuation. Picks ONE primary method per company type
  (mid-cycle EPS × historical PE for cyclicals, forward EV/EBITDA for growth, unit math
  for commodities, P/FCF for turnarounds) and treats other methods as sanity checks —
  never averages across methods. Uses CFA-aligned DCF, multiples, SOTP, residual income,
  and comparable analysis. Accepts Bloomberg Terminal data from screenshots and fetches
  SEC EDGAR filings. Produces analyst-style valuation with explicit multiple re-rating
  thesis and catalyst-to-rerating map.
triggers:
  - stock valuation
  - company valuation
  - fair value calculation
  - target price
  - DCF analysis
  - valuation model
  - estimate intrinsic value
  - price target
  - what is the company worth
---

# Valuation Calculator Skill

## Core Philosophy (Read This First)

Analyst valuations DO NOT average across methods. They pick ONE primary method appropriate to the company's archetype, derive the target price from that method's inputs, and use other methods as brief sanity checks. The previous version of this skill ran 9 methods and output a "Probability-Weighted Fair Value" — that is an anti-pattern. Averaging across methods destroys the point of view.

**Five archetype mappings:**

| Archetype | Primary Method | Rationale |
|-----------|---------------|-----------|
| Semiconductor cyclical (DIOD, MCHP, TXN) | **Mid-cycle EPS × 10-year average P/E** | Earnings are cycle-dependent; the market anchors on through-cycle earnings × historical multiple |
| Growth foundry / specialty semi (TSEM, AMD) | **Forward EV/EBITDA × multiple-expansion thesis** | Multiple expands as growth thesis proves out |
| Mining / commodity (UUUU, UAMY, MP) | **lb/ton × commodity price × unit margin → EBITDA × peer EV/EBITDA** | Bottom-up unit math is the only honest valuation — see `references/commodity-math-template.md` |
| Turnaround (CSIQ, COMM) | **P/FCF or replacement value** | Earnings are noisy; FCF or liquidation value anchors the floor |
| High-growth SaaS / platform (DAVE, HOOD) | **EV/Revenue × Rule-of-40 check** | Market prices on growth until profitability stabilizes |
| Financial (JPM, WFC, banks) | **Residual Income or P/B × ROE** | Earnings quality depends on reserves; RI captures book-value creation |
| Pre-revenue / frontier (FCEL, BLDP) | **Scenario framing only — no target** | No denominator exists |

**Rules:**

1. **Pick the primary method FIRST**, before running any model. State it explicitly in the output: "Primary method: mid-cycle EPS × historical average P/E."
2. **Run the primary method fully** with every input named and sourced.
3. **Other methods are sanity checks** — max 2-line mention each, no independent "fair value" output.
4. **No "probability-weighted fair value" averaging** — the median of 9 methods is meaningless because the methods are not independent estimates of the same quantity.
5. **State the catalyst that re-rates the multiple** from current to target — this is what makes the target price actionable.
6. **For commodity companies, do unit math FIRST**, then multiples. See `references/commodity-math-template.md`.

## Purpose

Calculate a defensible target price for public companies using CFA-aligned methodologies
but with **analyst-style discipline**: one primary method per archetype, bottom-up unit economics
for commodities, explicit multiple-expansion thesis, peer quartile statistics, and a
catalyst-to-rerating map. This skill combines Bloomberg Terminal data (provided by user via
screenshots), SEC filings, and institutional-grade models to produce a valuation that
answers "what is this stock worth and WHY" — not a spreadsheet output.

## When to Use

Activate when the user:
- Provides Bloomberg Terminal financial data (screenshot or text) and asks for valuation
- Asks to calculate fair value, intrinsic value, or target price for a public company
- Wants DCF, multiples-based, or comparable company analysis
- Needs a comprehensive valuation report for an investment thesis

## Supporting Files

Load these reference files during analysis:
- `valuation-models.md` — Complete CFA-aligned formula reference (DDM, DCF, RI, multiples)
- `sec-filing-guide.md` — SEC EDGAR data retrieval and parsing instructions
- `industry-multiples.md` — Sector-specific benchmark multiples and peer selection

---

## VALUATION WORKFLOW

### Phase 1: Data Collection

#### 1.1 Parse User-Provided Bloomberg Data

Extract these metrics from the user's screenshot or text input. Record the currency
(usually USD) and whether figures are in millions or billions.

**Market Data:**
- Current market capitalization
- Current stock price
- Shares outstanding (basic and diluted)
- Enterprise Value (EV = Market Cap + Total Debt - Cash)

**Income Statement (historical + consensus estimates):**
- Total Revenue and YoY growth rate
- Gross Profit and Gross Margin %
- EBITDA and EBITDA Margin %
- EBIT (Operating Income)
- Net Income (adjusted and GAAP)
- Basic EPS and Adjusted EPS
- EPS YoY growth rate

**Cash Flow:**
- Operating Cash Flow (CFO)
- Capital Expenditures (CapEx)
- Free Cash Flow (FCF = CFO - CapEx)
- FCF Margin %
- Unlevered Free Cash Flow (UFCF)
- Cash Flow Per Share

**Balance Sheet:**
- Cash and Short-term Investments
- Total Debt (short-term + long-term)
- Net Debt
- Total Equity (Book Value)
- Book Value Per Share

**Existing Multiples (from Bloomberg):**
- Trailing P/E and Forward P/E
- EV/EBITDA
- P/B (Price-to-Book)
- P/S (Price-to-Sales)
- P/CF (Price-to-Cash-Flow)
- Price/FCF
- PEG Ratio
- Dividend Yield

**Historical Multiples (if available):**
- 5Y and 10Y average P/E
- Historical P/E range (min, mean+1SD, mean, mean-1SD, max)

**Consensus Estimates (forward years):**
- Revenue estimates (current year, next year, year after)
- EPS estimates (current year, next year, year after)
- EBITDA estimates
- Analyst count and estimate dispersion

**Profitability Metrics:**
- Return on Equity (ROE)
- Return on Invested Capital (ROIC)
- Return on Assets (ROA)

If any critical data is missing, note it and use SEC filings to fill gaps.

#### 1.2 Fetch SEC EDGAR Data

Follow the instructions in `sec-filing-guide.md` to retrieve financial statements.

**Required Filings:**
- Latest 10-K (annual report) — full-year financials
- Latest 10-Q (quarterly) — most recent quarter
- Proxy Statement (DEF 14A) — compensation, insider ownership

**Data to Extract:**
- 3-5 years of Income Statement data
- 3-5 years of Balance Sheet data
- 3-5 years of Cash Flow Statement data
- Segment revenue breakdown (for SOTP)
- Debt maturity schedule
- Share repurchase activity
- Capital expenditure guidance

#### 1.3 Calculate Derived Metrics

From raw data, compute:

```
Sustainable Growth Rate:
  g = ROE * (1 - Payout Ratio)

DuPont Decomposition:
  ROE = Net Profit Margin * Asset Turnover * Equity Multiplier
  ROE = (NI/Sales) * (Sales/Assets) * (Assets/Equity)

Extended DuPont (5-factor):
  ROE = EBIT Margin * Interest Burden * Tax Burden * Asset Turnover * Equity Multiplier

NOPAT:
  NOPAT = EBIT * (1 - Tax Rate)

Invested Capital:
  IC = Total Equity + Total Debt - Cash

ROIC:
  ROIC = NOPAT / Invested Capital

Implied EPS Growth:
  From consensus estimates for next 2-3 years

Revenue CAGR:
  CAGR = (Revenue_end / Revenue_start)^(1/n) - 1
```

---

### Phase 1b: Market Pricing Mechanism Pre-Check

Before running valuation models, reverse-engineer how the market actually prices this stock:

```
Step 1: Identify the market's current pricing method
  - What multiple does the stock trade most closely to? (P/E, EV/EBITDA, P/S, P/B)
  - Evidence: which metric best explains stock price movements around earnings?
  - Sell-side consensus: what methodology do most analysts use?

Step 2: Identify hidden assumptions in current price
  - What revenue growth rate is implied by current multiples?
  - What margin trajectory is priced in?
  - What terminal state does the market assume?

Step 3: Align valuation model selection with market reality
  - If market prices on P/S (growth company): lead with relative valuation, DCF secondary
  - If market prices on P/E (mature company): lead with earnings-based models
  - If market prices on P/B (financial/asset-heavy): lead with residual income
  - If market prices on EV/EBITDA (levered/capital-intensive): lead with EBITDA-based models
  - Run all applicable models regardless, but WEIGHT the output toward the market's method
```

This step ensures valuation output is RELEVANT to how the stock actually trades, not just academically correct.

### Phase 2: Valuation Engine

Run ALL applicable models. Skip models that are inappropriate for the company type
(e.g., skip DDM for non-dividend payers, skip P/E for negative-earnings companies).

#### 2.1 Relative Valuation (Multiples-Based)

For each multiple, calculate THREE implied values:

| Multiple | Formula | Best For |
|----------|---------|----------|
| **Trailing P/E** | Current Price / LTM EPS | Profitable, stable earnings |
| **Forward P/E** | Current Price / NTY EPS estimate | Growth companies |
| **EV/EBITDA** | Enterprise Value / LTM EBITDA | Capital-intensive, leveraged |
| **P/S** | Market Cap / LTM Revenue | Unprofitable or high-growth |
| **EV/Revenue** | Enterprise Value / LTM Revenue | Pre-profit comparison |
| **P/B** | Market Cap / Book Value | Asset-heavy (banks, semis) |
| **P/FCF** | Market Cap / LTM Free Cash Flow | Cash-generative |
| **P/CF** | Market Cap / Operating Cash Flow | All companies with positive CFO |
| **PEG** | Forward P/E / EPS Growth Rate | Growth-at-reasonable-price |

**For each multiple, derive fair value using three benchmarks:**

1. **Historical Average**: Company's own 5Y or 10Y average multiple
   ```
   Fair Value = Current Metric * Historical Average Multiple
   Per Share = Fair Value / Shares Outstanding
   ```

2. **Sector Median**: Industry median multiple from `industry-multiples.md`
   ```
   Fair Value = Current Metric * Sector Median Multiple
   ```

3. **Peer Group**: Median of 3-5 closest comparable companies
   ```
   Fair Value = Current Metric * Peer Median Multiple
   ```

**PEG Analysis (per TER report pattern):**
```
PEG Ratio = Forward P/E / Expected EPS Growth Rate (%)
Fair PEG = 1.0 (growth equals multiple)
Overvalued if PEG > 1.0, Undervalued if PEG < 1.0
Implied Fair P/E = EPS Growth Rate * Fair PEG
Implied Fair Price = Implied Fair P/E * Forward EPS
```

**EPS Growth x Historical PE Method (per TER report pattern):**
```
Step 1: Take Forward EPS estimate for Year+2 or Year+3
Step 2: Multiply by 10Y average P/E
Step 3: Result = Fair value based on normalized earnings power
Example (TER): FY27 EPS ~$8.34 * 29x avg PE = ~$242 fair value
```

#### 2.2 Discounted Cash Flow (DCF) Valuation

Run BOTH FCFF and FCFE models when data permits.

**FCFF Model (Firm Value):**

```
Step 1: Calculate Historical FCFF
  FCFF = EBIT * (1 - Tax Rate) + D&A - CapEx - Change in NWC
  Alternative: FCFF = CFO + Interest * (1 - Tax Rate) - CapEx

Step 2: Project FCFF for 5-10 years
  Use revenue growth rates from consensus + margin assumptions
  Stage 1 (Years 1-3): High growth phase — use analyst estimates
  Stage 2 (Years 4-7): Transition — linearly decline to terminal rate
  Stage 3 (Terminal): Stable growth at GDP rate (2-3%)

Step 3: Calculate WACC
  Cost of Equity: re = Rf + Beta * (Rm - Rf)
    Rf = 10-year US Treasury yield (current)
    Beta = Company beta (from Bloomberg or regression)
    Rm - Rf = Equity risk premium (5.0-6.0% typical)
    Adjusted Beta = 0.33 + 0.67 * Raw Beta

  Cost of Debt: rd = Interest Expense / Average Total Debt
    After-tax: rd * (1 - Tax Rate)

  Weights: Use market values
    E = Market Cap
    D = Market Value of Debt (approximate with book value)
    WACC = (E/(E+D)) * re + (D/(E+D)) * rd * (1 - Tax Rate)

Step 4: Calculate Terminal Value (TWO methods)
  Method A — Perpetual Growth:
    TV = FCFF_terminal * (1 + g) / (WACC - g)
    where g = long-term growth rate (2-3%)

  Method B — Exit Multiple:
    TV = Terminal Year EBITDA * Exit EV/EBITDA Multiple
    Use sector median or company historical average

Step 5: Discount to Present Value
  Firm Value = Sum of PV(FCFF) + PV(Terminal Value)
  Equity Value = Firm Value - Net Debt
  Per Share Value = Equity Value / Diluted Shares Outstanding
```

**FCFE Model (Equity Value Directly):**

```
FCFE = Net Income + D&A - CapEx - Change in NWC + Net Borrowing
Alternative: FCFE = CFO - CapEx + Net Borrowing

Discount at Cost of Equity (re), not WACC
Equity Value = Sum of PV(FCFE) + PV(Terminal Value of Equity)
Per Share = Equity Value / Diluted Shares
```

**DCF Sensitivity Table:**
Create a matrix varying:
- Rows: WACC (base -1%, base -0.5%, base, base +0.5%, base +1%)
- Columns: Terminal growth (1.5%, 2.0%, 2.5%, 3.0%, 3.5%)
- Cell values: Implied per-share fair value

#### 2.3 Sum-of-the-Parts (SOTP) Valuation

Use when company has distinct business segments, geographic entities, or subsidiaries with different growth profiles, risk characteristics, or applicable valuation methodologies.

**Standard Segment SOTP:**
```
Step 1: Identify segments from SEC filings or Bloomberg
Step 2: For each segment:
  - Determine revenue and/or EBITDA
  - Select appropriate peer multiple (from industry-multiples.md)
  - Calculate segment value = Metric * Appropriate Multiple
Step 3: Sum all segment values
Step 4: Add: Net cash (or subtract net debt)
Step 5: Subtract: Minority interests, preferred equity
Step 6: Divide by diluted shares = Per share SOTP value

Run three scenarios:
  Conservative: Low-end multiples for each segment
  Base: Median multiples
  Optimistic: High-end multiples
```

**Geographic/Entity SOTP (for cross-border companies):**
When a company has publicly listed subsidiaries or operations in different jurisdictions:
```
Step 1: Identify entities (e.g., parent vs listed subsidiary, US vs international operations)
Step 2: For listed subsidiaries:
  - Use market cap as starting point
  - Apply ownership percentage: Parent's share = Subsidiary Market Cap * Ownership %
  - Apply geographic/political risk discount if warranted (e.g., 20-40% for China-listed subs)
Step 3: For non-listed segments:
  - Estimate revenue allocation by geography (from 10-K geographic disclosures)
  - Apply region-appropriate peer multiples
Step 4: Sum all entity values
Step 5: Apply conglomerate discount if appropriate (typically 10-20%)
```

**Per-Unit SOTP (for capacity-based businesses):**
When a company's value can be anchored to physical capacity or assets:
```
Step 1: Identify per-unit valuation anchors from comparable transactions:
  - EV/MW (power generation, data centers): Use recent M&A transactions as benchmarks
  - EV/GW (semiconductor capacity, solar manufacturing)
  - EV/subscriber or EV/user (platform businesses)
  - EV/store or EV/location (retail, healthcare facilities)
  - EV/boe or EV/oz (oil & gas, mining reserves)

Step 2: Apply per-unit value to company's capacity:
  Segment Value = Company Units * Per-Unit Value from Comparable Transaction

Step 3: Apply discount for:
  - Contracted vs uncontracted capacity (e.g., 40-60% discount for uncontracted)
  - Operational vs under-construction (e.g., 30-50% discount for under-construction)
  - Customer quality gap vs comparable (e.g., comparable had hyperscaler contracts)

Example (data center):
  AirTrunk acquisition: $161B for 1.8GW = ~$89.5B/GW
  Company has 0.66GW operational = 0.66 * $89.5B = ~$59B implied
  Apply 40% discount (no comparable customer contracts) = ~$35B
```

#### 2.4 Comparable Transaction Analysis

Use recent M&A transactions involving similar companies to derive valuation benchmarks.

```
Step 1: Identify 3-5 recent transactions (last 3 years) in the same sector
Step 2: For each transaction, calculate:
  - EV/Revenue at acquisition
  - EV/EBITDA at acquisition
  - EV/per-unit metric (MW, GW, subscriber, etc.)
  - Premium paid over pre-announcement price

Step 3: Build comparison table:
  | Transaction | Date | EV ($B) | EV/Rev | EV/EBITDA | EV/Unit | Premium |
  |-------------|------|---------|--------|-----------|---------|---------|

Step 4: Apply median transaction multiples to target company
Step 5: Adjust for size, growth rate, and profitability differences
```

#### 2.5 Bottom-Up Capacity-to-EBITDA Projection

For companies undergoing rapid capacity expansion (manufacturing, data centers, power generation, mining):

```
Step 1: Establish current capacity and utilization
  Current capacity: [XX] units (MW, GW, EH/s, etc.)
  Current utilization: [XX]%
  Current EBITDA from this segment: $[XX]M

Step 2: Map disclosed capacity expansion timeline
  | Quarter | Incremental Capacity | Cumulative | Source |
  |---------|---------------------|------------|--------|
  | Q1 2026 | +XX units           | XX total   | Mgmt guidance |
  | Q2 2026 | +XX units           | XX total   | Contract disclosure |
  | ...     | ...                 | ...        | ... |

Step 3: Calculate EBITDA at full capacity
  Growth multiple = Future Capacity / Current Capacity
  Future EBITDA = Current Segment EBITDA * Growth Multiple
  (Adjusted for: utilization ramp, pricing changes, operating leverage)

Step 4: Apply appropriate multiple to future EBITDA
  Fair Value = Future EBITDA * Target EV/EBITDA Multiple
  Discount to present if >2 years forward (use WACC)
```

#### 2.6 Davis Double Play Analysis

When both earnings growth AND multiple expansion could occur simultaneously:

```
Step 1: EPS Trajectory
  Current EPS: $[XX]
  Forward 2Y EPS (consensus): $[XX]
  EPS Growth: [XX]%

Step 2: Multiple Trajectory
  Current P/E: [XX]x
  Historical average P/E (10Y): [XX]x
  Sector median P/E: [XX]x
  "Normalized" P/E (post rate-cut environment): [XX]x

Step 3: Davis Double Play Calculation
  If EPS grows from $A to $B and P/E expands from Xx to Yx:
  Target Price = $B * Yx
  Total Return = (Target Price / Current Price - 1)
  Decomposition: [XX]% from EPS growth + [XX]% from PE expansion

Step 4: Davis Double Kill (downside scenario)
  If EPS disappoints (bear case) and P/E contracts (risk-off):
  Bear Price = Bear EPS * Trough P/E
```

#### 2.7 NAD Price Decomposition (Price Floor Analysis)

Decompose the current stock price into mutually exclusive "floors" — value layers that different investor types are paying for, each with specific evidence, reliability, and collapse conditions. This is NOT a valuation model — it is a reverse-engineering of what the market is currently paying for.

```
Step 1: Identify the Pricing Bridge
  Current Price = Floor 1 + Floor 2 + Floor 3 + ... - Discount Floor(s)
  Must approximately sum to current price (±15%)

Step 2: Define 2-4 Positive Floors + ≥1 Negative Floor

  For each floor, specify:
  | Field | Required |
  |-------|----------|
  | Floor Name | Descriptive label (e.g., "Core Business Value", "AI Growth Premium") |
  | Implied Value | Dollar amount per share |
  | Evidence | What supports this floor |
  | Reliability | High / Medium / Low |
  | Observation Signal | What to watch for changes |
  | Collapse Condition | What specific event/data breaks this floor |
  | Who Pays | Which investor type supports this layer |

Step 3: Map Downside Path
  - Which floor breaks first? (usually the most narrative-driven floor)
  - If it breaks, what price range results?
  - Which floor breaks next? (cascade analysis)

Step 4: Map Upside Path
  - Which floor thickens first? (usually the floor with nearest catalyst)
  - If it thickens, what price range results?
  - What would add a new floor? (new growth vector, M&A, etc.)
```

**Floor Types (common patterns):**
- **Foundation Floor**: Balance sheet value, net cash, liquidation value
- **Core Business Floor**: Stable earnings power at normalized multiples
- **Growth/Narrative Floor**: Premium for expected growth above market rate
- **Option Value Floor**: Upside from new products, markets, or transformations
- **Discount Floor (negative)**: Historical credibility issues, governance risk, policy risk

**Common Mistakes:**
- Floors that overlap (same value counted twice)
- No negative floor (every stock has discount factors)
- Bridge doesn't sum to approximately current price
- Floors defined as valuation methods rather than value layers

**Example (simplified):**
```
Current Price: $69
Floor 1 (Core Brokerage): $25 — stable transaction + NII revenue at peer multiples
Floor 2 (Growth Premium): $20 — 52% revenue growth commanding above-peer P/E
Floor 3 (Platform Optionality): $18 — crypto, prediction markets, international
Floor 4 (Narrative/AI): $10 — AI-driven platform story, Gold flywheel
Discount Floor: -$4 — MAU decline, crypto volatility, regulatory history
Bridge: $25 + $20 + $18 + $10 - $4 = $69 ✓
```

#### 2.9 Residual Income Valuation

Best for financial companies and asset-heavy businesses.

```
RI = Net Income - (Cost of Equity * Beginning Book Value)
   = (ROE - re) * Beginning Book Value

Single-Stage Model:
  V = Book Value + RI_1 / (re - g)

Multi-Stage:
  V = Book Value + Sum of PV(RI_t) for explicit period + PV(Terminal RI)

Justified P/B from RI:
  P/B = 1 + (ROE - re) / (re - g)
  If ROE > re: P/B > 1 (value creation)
  If ROE < re: P/B < 1 (value destruction)
```

#### 2.10 Dividend Discount Model (DDM)

Only for companies with consistent dividend payments.

```
Gordon Growth: V = D1 / (re - g)
  where D1 = D0 * (1 + g), g = sustainable growth rate

Two-Stage DDM:
  V = Sum[D0*(1+g1)^t / (1+re)^t] for t=1 to n
    + [Dn+1 / (re - g2)] * [1 / (1+re)^n]

H-Model (linearly declining growth):
  V = D0*(1+gL)/(re-gL) + D0*H*(gS-gL)/(re-gL)
  where H = half-life of high-growth period
```

---

### Phase 3: Scenario Analysis (Prose, Not Probability-Weighted)

Produce bull / base / bear scenarios as **one sentence each in prose**. Do NOT assign probability weights and do NOT compute a probability-weighted fair value. Analysts commit to a base case target price — they do not hide behind a weighted average.

**Bull Case (one sentence):**
"[Specific data-driven upside path — e.g., 'If 2027E EPS reaches $8.34 on 85% utilization and multiple expands to 32x on AI rerating, target becomes $267']"

**Base Case (one sentence — this is the primary target):**
"[Primary target derivation — e.g., 'Base case applies 10-year average P/E of 29x to 2027E consensus EPS of $7.28, yielding $211 target']"

**Bear Case (one sentence):**
"[Specific downside path — e.g., 'If gross margin recovery stalls below 32% and cyclical EPS stays near $2.50, trough P/E of 18x implies $45']"

**Rules:**
- Do NOT assign numeric probabilities unless the user explicitly asks
- Do NOT average Bull/Base/Bear into a single "weighted fair value"
- The **Base Case target IS the target price** — bull and bear are context for risk assessment
- Each scenario must name the specific data thresholds that trigger it (margin %, revenue growth %, multiple level)
- For turnarounds or high-uncertainty names, state "目标价区间 $X-$Y 取决于 [specific trigger]" instead of a single number

---

### Phase 4: Comparable Company Analysis (with Quartile Statistics)

#### 4.1 Peer Selection

Select 3-5 peers based on:
- Same industry / sub-sector
- Similar market cap range (0.5x to 2x target)
- Similar business model and revenue mix
- Similar growth profile

#### 4.2 Comparison Table with Quartile Stats

Build a comparison table with **a statistics block at the bottom** showing Max / 75th Percentile / Median / 25th Percentile / Min for each ratio column. This is a stronger analyst format than the old "Median only" version — it lets the reader see quartile bands and locate the target company within the peer distribution.

(Pattern stolen from `anthropics/financial-services-plugins` comps-analysis skill.)

**Rule about which columns deserve stats:** Multiples, margins, and growth rates SHOULD have stats. Absolute size columns (market cap, revenue) should NOT — statistics on absolute size are meaningless. Every ratio should be X / Revenue or X / [something already on the sheet].

```
| Company | Market Cap | 2026E Revenue | Rev Growth | GM% | EBITDA% | FCF% | Fwd P/E | EV/EBITDA | P/S | ROIC |
|---------|-----------|--------------|-----------|-----|---------|------|---------|-----------|-----|------|
| DIOD    | $3.8B     | $1,680M      | 13%       | 34% | 14.5%   | 9.3% | 22x     | 14.4x     | 2.3x| 6%   |
| ON Semi | $28B      | $7,100M      | 8%        | 45% | 30%     | 18%  | 18x     | 11x       | 3.9x| 12%  |
| Vishay  | $2.5B     | $2,900M      | 5%        | 25% | 14%     | 7%   | 15x     | 9x        | 0.9x| 5%   |
| MCHP    | $32B      | $5,800M      | 6%        | 60% | 38%     | 25%  | 20x     | 14x       | 5.5x| 9%   |
| **Max** | —         | —            | **13%**   | **60%** | **38%** | **25%**| **22x** | **14.4x** | **5.5x** | **12%** |
| **75th** | —        | —            | **12%**   | **54%** | **32%** | **22%**| **20.5x**| **14.2x**| **4.7x** | **10.5%** |
| **Median** | —      | —            | **7%**    | **40%** | **22%** | **14%**| **19x** | **12.5x** | **3.1x** | **7.5%** |
| **25th** | —        | —            | **6%**    | **30%** | **14%** | **8%** | **17x** | **10x**   | **1.6x** | **5.5%** |
| **Min** | —         | —            | **5%**    | **25%** | **14%** | **7%** | **15x** | **9x**    | **0.9x** | **5%**  |
```

**How to read this:**
- "DIOD trades at 14.4x EV/EBITDA — 75th percentile of peers (median 12.5x)" → priced at premium
- "DIOD's 13% revenue growth matches the MAX of the peer set" → growth is why it's premium
- "DIOD's 6% ROIC is between 25th and median" → returns are below average for the premium

#### 4.3 Premium/Discount Interpretation

Instead of "X trades at Y premium", use the quartile framing:

```
For each ratio:
  - Where does target sit in the peer distribution? (Max / 75th / Median / 25th / Min)
  - If above 75th: trading at premium-company level — what justifies the premium?
  - If between 25th and median: trading at a discount — is this a value opportunity or a trap?
  - If below 25th: priced as a distressed name — what fundamental issue is the market pricing in?
```

**Sector-specific metrics to include:**
- SaaS / platforms: Add Rule-of-40 (revenue growth % + FCF margin %)
- Banks / financials: Add ROTCE, efficiency ratio, NIM
- Industrials / capacity-driven: Add capacity utilization, maintenance capex ratio
- Mining: Add EV/reserves, AISC ($/lb), replacement cost per ton

---

### Phase 5: Output Report

Generate the valuation report in this exact structure:

```
================================================================
VALUATION REPORT: [TICKER] — [COMPANY NAME]
Date: [Current Date]
Current Price: $[XX.XX] | Market Cap: $[XX.X]B
================================================================

1. HEADLINE (Analyst-Style, NOT a Probability-Weighted Average)
   - PRIMARY METHOD: [e.g., "Mid-cycle EPS × 10-year average P/E"]
   - WHY THIS METHOD: [one sentence tied to archetype]
   - TARGET PRICE: $[XX] (from primary method derivation)
   - UPSIDE FROM CURRENT: [+/-XX]%
   - MULTIPLE RE-RATING CATALYST: [single specific event]

2. PRIMARY METHOD DERIVATION (Detailed)
   - Every input named with source
   - For cyclicals: 10Y average P/E × 2Y forward EPS
   - For growth: forward EV/EBITDA × multiple-expansion thesis
   - For commodity: unit math → EBITDA × peer EV/EBITDA (commodity-math-template.md Phase 1-5)
   - For turnarounds: P/FCF or replacement value
   - Show the arithmetic: "$X × Yx = $Z per share"

3. SANITY CHECKS (Each ≤2 Lines — NOT Independent Fair Values)

   | Sanity Check        | Value   | vs Primary | Interpretation |
   |---------------------|---------|-----------|---------------|
   | DCF (FCFF)          | $XX.XX  | [tight/wide] | [one phrase] |
   | EV/EBITDA (hist)    | $XX.XX  | [tight/wide] | [one phrase] |
   | P/FCF               | $XX.XX  | [tight/wide] | [one phrase] |
   | NAD Floor Analysis  | See below | — | — |

   DO NOT compute a "Median of All" row. DO NOT average.

4. SCENARIOS (Prose, One Sentence Each, No Probability Weights)
   - Bull: [trigger → resulting price, one sentence]
   - Base: [primary target, one sentence]
   - Bear: [trigger → resulting price, one sentence]

5. DCF MODEL DETAILS (If DCF is Sanity Check or Primary)
   - Key Assumptions: Revenue growth, margins, WACC, terminal growth
   - Projected cash flows table (5-10 years)
   - Terminal value calculation (both methods)
   - Sensitivity table (WACC vs growth)

5. MULTIPLES ANALYSIS
   - Current vs historical vs sector for each multiple
   - Premium/discount assessment

6. COMPARABLE COMPANY TABLE
   - Full peer comparison matrix

7. KEY RISKS TO VALUATION
   - Upside risks (catalysts that could push value higher)
   - Downside risks (factors that could impair value)

8. POSITION SIZING GUIDANCE
   - Based on conviction and risk/reward:
     High conviction (>30% upside, strong catalyst): 10-15%
     Medium conviction (15-30% upside): 5-10%
     Speculative (<15% upside or high uncertainty): 2-5%
================================================================
```

---

### Phase 6: Historical DCF / Model Backtest (Optional but Recommended)

Run the PRIMARY valuation method for each of the last 3 years and compare the target price vs the actual price at that point. This is a sanity check analysts do by hand — if the model would have called past moves correctly, the approach is credible; if it was consistently off, you need to re-examine assumptions.

(Pattern stolen from `halessi/DCF` — historical_DCF wrapper.)

```
For each year Y in [2023, 2024, 2025]:
  1. Use only data available as of the end of year Y (no look-ahead bias)
  2. Apply the primary method with year-Y-ending inputs:
     - Mid-cycle EPS approach: use trailing 10Y average P/E as of year Y, multiply by Y+2 EPS estimate
     - DCF: use year Y WACC, year Y consensus estimates, year Y terminal g
     - EV/EBITDA: use year Y forward EBITDA × year Y peer median multiple
  3. Record: target_price_Y, actual_price_at_end_of_Y, error %
```

**Interpretation:**
- Consistent <15% error → model is credible, trust the current target
- Errors >30% → either the model doesn't fit, or the prior assumptions were off
- Directional accuracy (up vs down) matters more than exact number

**Output:**

```
HISTORICAL BACKTEST
| Year | Target | Actual | Error % | Notes |
|------|--------|--------|---------|-------|
| 2023 | $X.XX  | $Y.YY  | +/-Z%   | Model under-weighted cyclical recovery |
| 2024 | $X.XX  | $Y.YY  | +/-Z%   | 2024 missed due to inventory correction (non-recurring) |
| 2025 | $X.XX  | $Y.YY  | +/-Z%   | Model validated once margin bottomed |
| Mean absolute error: Z% |
```

---

## MODEL SELECTION LOGIC (Primary Method → Others are Sanity Checks)

The primary method is chosen by ARCHETYPE (see Core Philosophy at the top). Other models run only as sanity checks, max 2 lines each in the output — never as independent fair-value estimates.

```
Archetype → Primary Method:

mature semiconductor cyclical → Mid-cycle EPS × 10Y average P/E
  Sanity checks: DCF (FCFF), EV/EBITDA at historical

growth foundry / specialty semi → Forward EV/EBITDA × multiple-expansion thesis
  Sanity checks: P/S if rev growth >30%, DCF for downside floor

mining / commodity → Unit math (lb/ton × price × margin) → EBITDA × peer EV/EBITDA
  Sanity checks: replacement cost, NAV on reserves
  MANDATORY: commodity-math-template.md Phase 1-5

turnaround → P/FCF or replacement value
  Sanity checks: NAD floor analysis for downside

high-growth SaaS / platform → EV/Revenue with Rule-of-40 check
  Sanity checks: DCF once profitability stabilizes, P/S vs peers

bank / financial → Residual Income or P/B × ROE
  Sanity checks: P/E on normalized earnings, P/TBV

pre-revenue / frontier → Scenario framing only, NO target price
  Sanity checks: None — cannot valuation-gate without denominator

dividend-paying utility / REIT → DDM (Gordon Growth or Two-Stage)
  Sanity checks: P/FFO for REITs, P/E for utilities
```

**Rules:**
- ALWAYS run the primary method fully with every input named and sourced
- Sanity checks are for showing "the answer is in the same zip code across methods" — they do NOT replace the primary
- Comparable company analysis ALWAYS runs (with quartile stats, per Phase 4)
- Scenario analysis ALWAYS runs (as prose, per Phase 3)
- Historical backtest runs when 3+ years of data exist

**Special cases:**
- Multi-segment company → additionally run SOTP
- Currency-cross-border company → additionally run geographic SOTP with political risk discount
- Pre-revenue / binary catalyst → run scenario framing only, do NOT output a single target price

---

## SPECIAL CONSIDERATIONS BY SECTOR

### Semiconductors / Hardware (TER, AEHR, TowerSemi)
- Use P/E and EV/EBITDA as primary multiples
- Cyclicality adjustment: use mid-cycle earnings for P/E
- Test intensity / TAM analysis for growth projections
- Customer concentration risk assessment

### Fintech / Financial Services (GDOT, DAVE, PGY, SOFI)
- Use P/S for high-growth phase, P/E once profitable
- For banks: P/B and Residual Income are primary
- Loan loss provisions and credit quality assessment
- Regulatory risk premium in discount rate

### Clean Energy / Solar (CSIQ, FCEL, T1Energy)
- SOTP is critical (multiple business lines)
- Policy/subsidy sensitivity analysis
- Use EV/Revenue or P/S (many unprofitable)
- Balance sheet leverage matters significantly

### Mining / Resources (UAMY, UUUU)
- Use P/E at mid-cycle commodity prices
- NAV (Net Asset Value) based on reserves
- Commodity price sensitivity table
- EV/EBITDA with commodity price scenarios

### Pre-Revenue / Early-Stage (ACHR, ASTS)
- Backlog-based valuation (total orders * probability of conversion)
- Comparable transaction multiples
- TAM penetration analysis
- Heavy scenario weighting — bear case may be near-zero

### Technology / SaaS / Platform (IREN, INOD, COMM)
- P/S and EV/Revenue primary for high-growth
- Rule of 40 check (Revenue Growth % + EBITDA Margin % >= 40)
- Transition to P/E as profitability emerges
- Recurring revenue premium

---

## DATA QUALITY CHECKS

Before running models, verify:

1. **Consistency**: Revenue in income statement matches cash flow statement
2. **Share count**: Diluted shares used for per-share calculations
3. **Currency**: All figures in same currency
4. **Time alignment**: LTM vs fiscal year vs calendar year
5. **Adjustments**: Identify and handle one-time items (restructuring, impairments)
6. **Negative values**: Cannot use P/E if EPS < 0; cannot use P/FCF if FCF < 0
7. **Extreme multiples**: Flag if any multiple > 100x or < 0x — likely distorted

---

