---
name: valuation-calculator
description: >
  Comprehensive public company valuation using DCF, multiples, comparable analysis,
  residual income, sum-of-the-parts, and scenario modeling. Accepts Bloomberg Terminal
  data from screenshots and fetches SEC EDGAR filings for full financial analysis.
  Produces institutional-grade valuation reports in English.
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

## Purpose

Calculate comprehensive fair value for public companies using multiple CFA-curriculum-aligned
valuation methodologies. This skill combines Bloomberg Terminal data (provided by user via
screenshots), SEC filings, and institutional-grade models to produce a multi-method valuation
with scenario analysis.

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

### Phase 3: Scenario Analysis

#### 3.1 Define Scenarios

**Bull Case (default 25% — adjust based on context):**
- Revenue growth at high end of analyst range or above
- Margin expansion (operating leverage, mix shift)
- Multiple expansion (sector re-rating, catalyst)
- Apply highest reasonable multiples from peer/historical range

**Base Case (default 50% — adjust based on context):**
- Revenue growth at consensus median
- Margins stable or per management guidance
- Multiples at historical average or sector median

**Bear Case (default 25% — adjust based on context):**
- Revenue growth at low end or miss
- Margin compression (competition, cost inflation)
- Multiple contraction (macro headwinds, sector rotation)
- Apply lowest reasonable multiples

**Probability Adjustment Guidelines:**
- Default weights (25/50/25) may be adjusted when evidence supports asymmetry
- For turnaround stories with high uncertainty: consider 30/40/30
- For high-conviction catalysts with clear timeline: consider 20/50/30 or 15/55/30
- For speculative/pre-revenue companies: consider 20/30/50 (heavier bear weight)
- Always state the rationale for any non-default probability weights

#### 3.2 Calculate Probability-Weighted Fair Value

```
Weighted Fair Value = (Bull Price * Bull Prob)
                    + (Base Price * Base Prob)
                    + (Bear Price * Bear Prob)

Upside/Downside from Current Price:
  Upside = (Weighted Fair Value / Current Price - 1) * 100%
```

---

### Phase 4: Comparable Company Analysis

#### 4.1 Peer Selection

Select 3-5 peers based on:
- Same industry / sub-sector
- Similar market cap range (0.5x to 2x target)
- Similar business model and revenue mix
- Similar growth profile

#### 4.2 Comparison Table

Build a comparison table with these columns:

| Metric | Target | Peer 1 | Peer 2 | Peer 3 | Median |
|--------|--------|--------|--------|--------|--------|
| Market Cap | | | | | |
| Revenue | | | | | |
| Revenue Growth % | | | | | |
| Gross Margin % | | | | | |
| EBITDA Margin % | | | | | |
| Net Margin % | | | | | |
| Forward P/E | | | | | |
| EV/EBITDA | | | | | |
| P/S | | | | | |
| P/FCF | | | | | |
| PEG | | | | | |
| ROE % | | | | | |
| ROIC % | | | | | |
| Net Debt/EBITDA | | | | | |

#### 4.3 Premium/Discount Analysis

```
For each multiple:
  Target Premium = (Target Multiple / Peer Median - 1) * 100%
  If premium > 0: Company trades at premium — assess if justified by growth/quality
  If premium < 0: Company trades at discount — assess if undervalued or deserved
```

---

### Phase 5: Output Report

Generate the valuation report in this exact structure:

```
================================================================
VALUATION REPORT: [TICKER] — [COMPANY NAME]
Date: [Current Date]
Current Price: $[XX.XX] | Market Cap: $[XX.X]B
================================================================

1. EXECUTIVE SUMMARY
   - Fair Value Range: $[Low] — $[High]
   - Base Case Fair Value: $[XX.XX]
   - Probability-Weighted Fair Value: $[XX.XX]
   - Upside/Downside: [+/-XX%] from current price
   - Valuation Verdict: [Undervalued / Fairly Valued / Overvalued]

2. VALUATION SUMMARY TABLE

   | Model                  | Fair Value/Share | vs Current |
   |------------------------|-----------------|------------|
   | DCF (FCFF) — Base      | $XX.XX          | +/-XX%     |
   | DCF (FCFE) — Base      | $XX.XX          | +/-XX%     |
   | Forward P/E (hist avg) | $XX.XX          | +/-XX%     |
   | Forward P/E (sector)   | $XX.XX          | +/-XX%     |
   | EV/EBITDA (hist avg)   | $XX.XX          | +/-XX%     |
   | P/S (sector)           | $XX.XX          | +/-XX%     |
   | P/FCF                  | $XX.XX          | +/-XX%     |
   | PEG-implied            | $XX.XX          | +/-XX%     |
   | EPS x Hist PE          | $XX.XX          | +/-XX%     |
   | SOTP (if applicable)   | $XX.XX          | +/-XX%     |
   | Residual Income        | $XX.XX          | +/-XX%     |
   | NAD Floor Analysis     | See breakdown   | See floors |
   | **Median of All**      | **$XX.XX**      | **+/-XX%** |

3. SCENARIO ANALYSIS

   | Scenario | Probability | Fair Value | Upside/Down |
   |----------|-------------|------------|-------------|
   | Bull     | XX%         | $XX.XX     | +XX%        |
   | Base     | XX%         | $XX.XX     | +/-XX%      |
   | Bear     | XX%         | $XX.XX     | -XX%        |
   | Weighted |             | $XX.XX     | +/-XX%      |

4. DCF MODEL DETAILS
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

## MODEL SELECTION LOGIC

Apply these rules to determine which models to run:

```
IF company pays dividends consistently:
  RUN DDM (Gordon Growth or Two-Stage)

IF company has positive earnings:
  RUN P/E analysis (trailing + forward)
  RUN PEG analysis
  RUN EPS x Historical PE method

IF company has positive EBITDA:
  RUN EV/EBITDA analysis
  RUN DCF (FCFF) model

IF company has positive free cash flow:
  RUN P/FCF analysis
  RUN DCF (FCFE) model

IF company is pre-revenue or pre-profit:
  RUN P/S or EV/Revenue only
  RUN backlog-based valuation if applicable
  NOTE: High uncertainty — widen scenario ranges

IF company has multiple distinct segments:
  RUN SOTP valuation

IF company is asset-heavy or financial:
  RUN Residual Income model
  RUN P/B analysis

ALWAYS RUN:
  - At least one multiples-based method
  - Comparable company analysis
  - Scenario analysis (bull/base/bear)
```

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

## OUTPUT LANGUAGE

All output MUST be in English. Financial data labels, commentary, analysis, and
conclusions must be written in English regardless of the source language of input data.
