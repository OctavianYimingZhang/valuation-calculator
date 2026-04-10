# Valuation Calculator

A comprehensive public company valuation skill for [Claude Code](https://claude.ai/claude-code). Runs multiple CFA-curriculum-aligned valuation models from Bloomberg Terminal data and SEC EDGAR filings.

## What It Does

When triggered, this skill:

1. **Parses** Bloomberg Terminal data from screenshots (market cap, revenue, EBITDA, EPS, FCF, margins, multiples)
2. **Fetches** SEC EDGAR filings (10-K, 10-Q) for income statement, balance sheet, cash flow statement
3. **Runs all applicable valuation models** and produces a comprehensive report

## Valuation Models

| Model | When Applied |
|-------|-------------|
| **DCF (FCFF)** | All companies with forecastable cash flows |
| **DCF (FCFE)** | Companies with stable capital structure |
| **Forward P/E** | Profitable companies |
| **EV/EBITDA** | Capital-intensive, leveraged companies |
| **P/S / EV/Revenue** | High-growth or unprofitable companies |
| **P/B** | Asset-heavy companies (banks, semiconductors) |
| **P/FCF** | Cash-generative companies |
| **PEG Ratio** | Growth-at-reasonable-price check |
| **EPS x Historical PE** | Quick fair value with PE bands |
| **Sum-of-the-Parts (SOTP)** | Multi-segment conglomerates |
| **Residual Income** | Financial institutions, asset-heavy |
| **Dividend Discount (DDM)** | Consistent dividend payers |
| **Comparable Company Analysis** | All companies (peer benchmarking) |
| **Scenario Analysis** | Bull / Base / Bear with probability weighting |

## Files

```
valuation-calculator/
├── SKILL.md                 # Main skill — workflow, triggers, output format
├── valuation-models.md      # Complete CFA-aligned formula reference
├── sec-filing-guide.md      # SEC EDGAR data retrieval instructions
├── industry-multiples.md    # Sector-specific benchmark multiples (9 sectors)
├── LICENSE                  # MIT License
└── README.md
```

## Installation

### Claude Code (CLI / Desktop / Web)

Copy all `.md` files into your Claude Code skills directory:

```bash
# Clone the repo
git clone https://github.com/OctavianYimingZhang/valuation-calculator.git

# Copy to Claude Code skills directory
mkdir -p ~/.claude/skills/valuation-calculator
cp valuation-calculator/*.md ~/.claude/skills/valuation-calculator/
```

The skill will auto-register and appear in your available skills list.

## Usage

Provide Bloomberg Terminal data (screenshot or text) and ask for a valuation:

```
/valuation-calculator

# or just ask naturally:
"Calculate the fair value of TER based on this Bloomberg data"
"Run a DCF valuation on GDOT"
"What is the intrinsic value of this company?"
```

### Example Input

Provide a Bloomberg screenshot containing:
- Market cap, revenue, EBITDA, EPS, FCF (historical + estimates)
- Margins (gross, EBITDA, net, FCF)
- Existing multiples (P/E, EV/EBITDA, P/S, P/B, P/FCF)
- Consensus estimates (revenue, EPS for forward years)

### Example Output

The skill produces a structured report:

```
================================================================
VALUATION REPORT: TER — Teradyne, Inc.
================================================================

1. EXECUTIVE SUMMARY
   Fair Value Range: $180 — $330
   Base Case: $242
   Upside: +22% from current price

2. VALUATION SUMMARY TABLE (all models side by side)
3. SCENARIO ANALYSIS (bull 25% / base 50% / bear 25%)
4. DCF MODEL DETAILS (with sensitivity table)
5. MULTIPLES ANALYSIS (current vs historical vs sector)
6. COMPARABLE COMPANY TABLE
7. KEY RISKS
8. POSITION SIZING GUIDANCE
================================================================
```

## Formula Reference

The `valuation-models.md` file contains complete formulas for:

- Gordon Growth Model, Two-Stage DDM, H-Model, Three-Stage DDM
- FCFF and FCFE (multiple equivalent formulations)
- WACC, CAPM, Build-Up Method
- Terminal Value (perpetual growth + exit multiple)
- All justified multiples (P/E, P/B, P/S, EV/EBITDA, P/FCF)
- Residual Income and EVA
- DuPont Analysis (3-factor and 5-factor)
- Sustainable Growth Rate

## Industry Coverage

The `industry-multiples.md` provides benchmark multiples for:

- Semiconductors & Equipment
- Fintech / Digital Financial Services
- Solar / Clean Energy
- Mining / Resources / Uranium
- Pre-Revenue / Early-Stage Technology
- Technology / SaaS / Data Infrastructure
- Healthcare / Biotech
- Aerospace / Defense / Space
- Consumer / Retail Banking

## Sources

This skill's methodology is derived from:

- **CFA Level 2 Volume 5: Equity Valuation** — DDM, DCF, Residual Income, Multiples
- **CFA Level 3 Volumes 1-4** — Asset Allocation, Portfolio Construction, Performance Measurement
- **30+ proprietary research reports** covering TER, GDOT, CSIQ, DAVE, UAMY, FCEL, IREN, INOD, PGY, ASTS, ACHR, and more

## License

MIT
