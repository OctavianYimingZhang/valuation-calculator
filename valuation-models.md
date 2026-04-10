# Valuation Models — Complete CFA-Aligned Formula Reference

## 1. Dividend Discount Models (DDM)

### 1.1 General DDM
```
V₀ = Σ(t=1 to ∞) [Dₜ / (1 + r)ᵗ]
```

### 1.2 Gordon Growth Model (Constant Growth)
```
V₀ = D₁ / (r - g)
   = D₀ × (1 + g) / (r - g)

Justified P/E = (Payout Ratio × (1 + g)) / (r - g)

Constraints: r > g, g must be sustainable
Best for: Mature, stable dividend payers (utilities, REITs)
```

### 1.3 Two-Stage DDM
```
V₀ = Σ(t=1 to n) [D₀(1+g₁)ᵗ / (1+r)ᵗ]  +  [Dₙ₊₁ / (r - g₂)] × [1/(1+r)ⁿ]

Where:
  g₁ = high growth rate (years 1 to n)
  g₂ = stable growth rate (year n+1 onward)
  n = length of high-growth period
```

### 1.4 H-Model (Linearly Declining Growth)
```
V₀ = D₀(1 + gL) / (r - gL)  +  D₀ × H × (gS - gL) / (r - gL)

Where:
  H = half-life of high-growth period (in years)
  gS = short-term (initial) growth rate
  gL = long-term (stable) growth rate
```

### 1.5 Three-Stage DDM
```
Stage 1: High growth g₁ for n₁ years
Stage 2: Growth linearly declines from g₁ to g₃ over n₂ years
Stage 3: Stable growth g₃ forever
V₀ = PV(Stage 1 dividends) + PV(Stage 2 dividends) + PV(Terminal Value)
```

---

## 2. Free Cash Flow Models

### 2.1 Free Cash Flow to Firm (FCFF)

**Definitions (multiple equivalent formulations):**
```
From Net Income:
  FCFF = NI + NCC + Int(1 - T) - FCInv - WCInv

From CFO:
  FCFF = CFO + Int(1 - T) - CapEx

From EBIT:
  FCFF = EBIT(1 - T) + D&A - CapEx - ΔNWC

From EBITDA:
  FCFF = EBITDA(1 - T) + D&A × T - CapEx - ΔNWC

Where:
  NI = Net Income
  NCC = Non-cash charges (depreciation, amortization, impairments)
  Int = Interest expense
  T = Effective tax rate
  FCInv = Fixed capital investment (CapEx)
  WCInv = Working capital investment (ΔNWC)
  CFO = Cash flow from operations
  ΔNWC = Change in net working capital
       = ΔCurrent Assets (excl cash) - ΔCurrent Liabilities (excl short-term debt)
```

**Firm Valuation:**
```
Firm Value = Σ(t=1 to n) [FCFFₜ / (1 + WACC)ᵗ]  +  TV_n / (1 + WACC)ⁿ

Equity Value = Firm Value - Market Value of Debt + Excess Cash
Per Share = Equity Value / Diluted Shares Outstanding
```

### 2.2 Free Cash Flow to Equity (FCFE)

**Definitions:**
```
From Net Income:
  FCFE = NI + NCC - FCInv - WCInv + Net Borrowing

From CFO:
  FCFE = CFO - CapEx + Net Borrowing

Where:
  Net Borrowing = New debt issued - Debt repaid
```

**Equity Valuation:**
```
Equity Value = Σ(t=1 to n) [FCFEₜ / (1 + re)ᵗ]  +  TV_equity_n / (1 + re)ⁿ
Per Share = Equity Value / Diluted Shares Outstanding
```

### 2.3 When to Use FCFF vs FCFE
```
Use FCFF when:
  - Capital structure is changing or unstable
  - FCFE is negative but FCFF is positive
  - Valuing the entire enterprise (for M&A)

Use FCFE when:
  - Capital structure is stable
  - Want to value equity directly
  - Company has stable debt levels
```

---

## 3. Cost of Capital

### 3.1 Cost of Equity — CAPM
```
re = Rf + β × (Rm - Rf)

Where:
  Rf = Risk-free rate (10Y US Treasury yield)
  β = Company beta (systematic risk)
  Rm = Expected market return
  (Rm - Rf) = Equity risk premium (ERP), typically 5.0-6.0%

Adjusted Beta (Bloomberg convention):
  β_adj = 0.33 + 0.67 × β_raw
```

### 3.2 Cost of Equity — Build-Up Method
```
re = Rf + ERP + Size Premium + Company-Specific Premium

Used for: Private companies or when beta is unreliable
Size Premium: Small cap +2-4%, Micro cap +4-7%
```

### 3.3 Cost of Equity — Dividend Yield + Growth
```
re = (D₁ / P₀) + g
```

### 3.4 Cost of Debt
```
rd = Interest Expense / Average Total Debt
After-tax cost: rd × (1 - T)

Alternative: Use YTM on company's outstanding bonds
```

### 3.5 WACC
```
WACC = (E/(E+D)) × re + (D/(E+D)) × rd × (1 - T)

Where:
  E = Market value of equity (market cap)
  D = Market value of debt (≈ book value if no market data)
  re = Cost of equity
  rd = Cost of debt (pre-tax)
  T = Corporate tax rate

Key rules:
  - Use MARKET values for weights, not book values
  - E + D should equal total capital
  - WACC is the discount rate for FCFF
  - re is the discount rate for FCFE and DDM
```

---

## 4. Terminal Value

### 4.1 Perpetual Growth Method
```
TV = FCF_terminal × (1 + g) / (WACC - g)

Where:
  g = long-term sustainable growth rate
  Typical g: 2.0-3.0% (≈ long-term GDP growth)
  MUST have: WACC > g

Sanity check: TV should be 50-80% of total firm value
If TV > 85%, growth assumptions may be too aggressive
```

### 4.2 Exit Multiple Method
```
TV = Terminal Year EBITDA × Exit EV/EBITDA Multiple
  or
TV = Terminal Year Earnings × Exit P/E Multiple
  or
TV = Terminal Year Revenue × Exit EV/Revenue Multiple

Use: Historical average multiple or sector median
```

### 4.3 Present Value of Terminal Value
```
PV(TV) = TV / (1 + WACC)ⁿ

Where n = number of years in explicit forecast period
```

---

## 5. Relative Valuation Multiples

### 5.1 Price-to-Earnings (P/E)
```
Trailing P/E = Price / LTM EPS
Forward P/E = Price / NTY EPS estimate

Justified P/E (from Gordon Growth):
  P/E = Payout × (1 + g) / (r - g)

If 100% payout: P/E = 1 / (r - g)

Implied Price from P/E:
  Fair Price = Target P/E × Forward EPS
```

### 5.2 Enterprise Value / EBITDA
```
EV/EBITDA = (Market Cap + Debt - Cash) / EBITDA

Justified EV/EBITDA ≈ (1 - T) / (WACC - g)
  (simplified, assumes CapEx ≈ D&A)

Advantages:
  - Capital-structure neutral
  - Not affected by depreciation policy
  - Works for negative-earnings companies (if EBITDA positive)

Implied EV from multiple:
  Fair EV = Target EV/EBITDA × EBITDA
  Fair Equity = Fair EV - Net Debt
  Fair Price = Fair Equity / Shares
```

### 5.3 Price-to-Sales (P/S)
```
P/S = Market Cap / Revenue
EV/Revenue = Enterprise Value / Revenue (preferred)

Justified P/S = Net Margin × Payout × (1 + g) / (r - g)
             = P/E × Net Profit Margin

Implied Price:
  Fair Market Cap = Target P/S × Revenue
  Fair Price = Fair Market Cap / Shares
```

### 5.4 Price-to-Book (P/B)
```
P/B = Market Cap / Book Value of Equity

Justified P/B (from Residual Income):
  P/B = 1 + (ROE - r) / (r - g)

Interpretation:
  P/B > 1: Company creates value (ROE > cost of equity)
  P/B = 1: Company earns exactly its cost of equity
  P/B < 1: Company destroys value (ROE < cost of equity)
```

### 5.5 Price-to-Free Cash Flow (P/FCF)
```
P/FCF = Market Cap / Free Cash Flow

Justified P/FCF = (1 + g) / (r - g)  [Gordon Growth form]

FCF Yield = FCF / Market Cap = 1 / (P/FCF)
```

### 5.6 PEG Ratio
```
PEG = Forward P/E / Expected EPS Growth Rate (in %)

Interpretation:
  PEG = 1.0: Fairly valued (growth matches multiple)
  PEG < 1.0: Potentially undervalued
  PEG > 1.0: Potentially overvalued

Implied Fair P/E = EPS Growth Rate × Fair PEG (1.0)
Implied Fair Price = Implied Fair P/E × Forward EPS

Overvaluation % = (Actual PEG / Fair PEG - 1) × 100%
```

### 5.7 EPS × Historical PE Method
```
Fair Price = Forward EPS (Year+2) × Historical Average P/E

Example (from TER report):
  FY27E EPS = $8.34
  10Y Average P/E = 29x
  Base fair value = $8.34 × 29 = $242

With multiple expansion (rate cut scenario):
  If P/E expands to 40x: $8.34 × 40 = $334
```

---

## 6. Residual Income Model

### 6.1 Residual Income Definition
```
RI = Net Income - Equity Charge
   = NI - (re × Beginning Book Value)
   = (ROE - re) × Beginning Book Value
```

### 6.2 Single-Stage RI Model
```
V₀ = B₀ + RI₁ / (re - g)
   = B₀ + [(ROE - re) × B₀] / (re - g)

Where:
  B₀ = Current book value per share
  RI₁ = Next year residual income
  re = Required return on equity
  g = Growth rate of residual income
```

### 6.3 Economic Value Added (EVA)
```
EVA = NOPAT - (WACC × Invested Capital)

Where:
  NOPAT = EBIT × (1 - T)
  Invested Capital = Total Equity + Total Debt - Cash

Firm Value = Invested Capital + PV(Future EVA)
```

---

## 7. Growth Rate Analysis

### 7.1 Sustainable Growth Rate
```
g = ROE × (1 - Payout Ratio)
  = ROE × Retention Ratio
  = ROE × b

Where b = Retained Earnings / Net Income
```

### 7.2 DuPont Analysis (3-Factor)
```
ROE = Net Profit Margin × Asset Turnover × Equity Multiplier
    = (NI/Revenue) × (Revenue/Assets) × (Assets/Equity)
```

### 7.3 Extended DuPont (5-Factor)
```
ROE = Tax Burden × Interest Burden × EBIT Margin × Asset Turnover × Equity Multiplier
    = (NI/EBT) × (EBT/EBIT) × (EBIT/Revenue) × (Revenue/Assets) × (Assets/Equity)
```

### 7.4 PRAT Model
```
g = Profit Margin × Retention × Asset Turnover × Financial Leverage
  = (NI/Revenue) × b × (Revenue/Assets) × (Assets/Equity)
```

---

## 8. Sum-of-the-Parts (SOTP)

### 8.1 Framework
```
Step 1: Identify distinct business segments
Step 2: For each segment, determine:
  - Segment revenue and/or EBITDA
  - Appropriate comparable peer set
  - Appropriate valuation multiple

Step 3: Value each segment
  Segment Value = Segment Metric × Peer Multiple

Step 4: Aggregate
  Enterprise Value = Σ(Segment Values)
  Equity Value = Enterprise Value - Net Debt + Non-operating Assets
  Per Share = Equity Value / Diluted Shares

Step 5: Scenario analysis
  Conservative: Use low-end peer multiples
  Base: Use median peer multiples
  Optimistic: Use high-end peer multiples
```

### 8.2 SOTP Example (from CSIQ report pattern)
```
Segment A (China subsidiary): Market value × ownership % = $X
Segment B (Americas business): Revenue × PS multiple = $Y
Total = $X + $Y
Adjust for: Net debt, minority interests, holding discount
```

---

## 9. Sensitivity Analysis Framework

### 9.1 DCF Sensitivity Matrix
```
Create grid:
  Rows: WACC values (base ± 0.5%, ± 1.0%)
  Columns: Terminal growth rate (1.5% to 3.5% in 0.5% steps)
  Cells: Implied fair value per share

         g=1.5%  g=2.0%  g=2.5%  g=3.0%  g=3.5%
WACC-1%:  $XXX    $XXX    $XXX    $XXX    $XXX
WACC-0.5: $XXX    $XXX    $XXX    $XXX    $XXX
WACC base:$XXX    $XXX    $XXX    $XXX    $XXX
WACC+0.5: $XXX    $XXX    $XXX    $XXX    $XXX
WACC+1%:  $XXX    $XXX    $XXX    $XXX    $XXX
```

### 9.2 Multiple Sensitivity
```
Create grid:
  Rows: Revenue/EBITDA/EPS growth scenarios
  Columns: Multiple range (low, mid, high)
  Cells: Implied fair value per share
```

### 9.3 Scenario Weighting
```
Bull Case (25%): Optimistic growth + multiple expansion
Base Case (50%): Consensus + historical multiples
Bear Case (25%): Below consensus + multiple compression

Weighted Value = Σ(Scenario Value × Probability)
```
