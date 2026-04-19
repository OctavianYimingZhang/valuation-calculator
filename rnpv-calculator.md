# Risk-Adjusted NPV (rNPV) Calculator

For **pipeline biotech**, **pre-commercial pharma**, **oil & gas 2P reserves**, and **mining projects under NI 43-101**, the primary valuation method is **risk-adjusted NPV** — not DCF, not multiples. This file codifies the method.

---

## Why rNPV Instead of DCF

A binary-outcome asset (drug approved / not approved, reserves proved / not proved, mine permitted / not permitted) has a distribution, not a point value. Standard DCF discounts expected cash flows at a single WACC, which double-counts risk when the underlying outcome is binary.

**rNPV separates:**
- Technical/regulatory success probability (PoS) — applied as a multiplier
- Commercial discount rate — applied to the cash flows *conditional on success*

This lets the analyst reason separately about "does it work" vs "what is it worth if it works."

---

## The Formula

```
rNPV = Σ PoS_cumulative × FCF_conditional_on_success × 1/(1+r)^t − Investment_costs_unconditional

where:
  PoS_cumulative = product of stage-success probabilities from current stage through launch
  FCF_conditional = free cash flow if the asset reaches commercial launch
  r = commercial discount rate (10-15% for approved pharma, 12-20% for commodity)
  Investment_costs = trial/development costs that are incurred REGARDLESS of success
```

---

## Stage-Specific Probabilities (Industry Benchmarks)

### Pharma / Biotech (BIO/IQVIA 2020-2024 aggregate)

| From stage | Probability to advance | Cumulative PoS to approval |
|-----------|------------------------|-----------------------------|
| Preclinical → Phase I | 10% | 10% × 63.2% × 30.7% × 58.1% × 85.3% = **~1.0%** |
| Phase I → Phase II | 63.2% | 63.2% × 30.7% × 58.1% × 85.3% = ~9.6% |
| Phase II → Phase III | 30.7% | 30.7% × 58.1% × 85.3% = ~15.2% |
| Phase III → NDA/BLA | 58.1% | 58.1% × 85.3% = **~49.6%** |
| NDA/BLA → Approval | 85.3% | **~85%** |

**Therapeutic-area adjustments:**
- Oncology is lower (Phase II→III ~28%, Phase III→NDA ~35%) — overall cumulative from Ph1 ~4%
- Hematology / Rare disease is higher (Phase II→III ~48%) — overall cumulative from Ph1 ~18%
- Cardiovascular, CNS, Psychiatry are lower than overall
- Anti-infectives, endocrine are higher

**Use**: Check the BIO Clinical Development Success Rates study or equivalent IQVIA/Evaluate Pharma data for the specific therapeutic area. Do not use the "everything is 10%" heuristic.

### Oil & Gas (SPE PRMS reserves classification)

| Category | Probability of commercial |
|----------|---------------------------|
| Proved (1P) | 90% |
| Proved + Probable (2P) | 50% |
| Proved + Probable + Possible (3P) | 10% |

**Use**: Apply PoS to the volumetric × price × margin calculation. For 2P, multiply the EBITDA stream by 0.5.

### Mining (NI 43-101 / JORC)

| Category | Probability of commercial |
|----------|---------------------------|
| Measured Reserves | 90% |
| Indicated Reserves | 65% |
| Inferred Resources | 20% |
| Exploration Target | 5-10% |

**Stage gates beyond reserves**:
- Pre-PEA (no study) → PEA complete: +0% (PEA is the study itself)
- PEA → Prefeasibility (PFS): ~50% project advancement
- PFS → Feasibility (FS): ~70%
- FS → Permitted/Construction: ~80%
- Permitted → Commercial Production: ~90%

**Use**: Multiply reserve PoS × stage-gate PoS for total project PoS.

---

## Discount Rate Selection (Conditional-on-Success)

### Pharma / Biotech

| Stage | Typical rate |
|-------|-------------|
| Preclinical | 40% |
| Phase I | 25-35% |
| Phase II | 20-25% |
| Phase III | 15-20% |
| NDA/BLA (pre-approval) | 12-15% |
| Approved (launch) | 10-12% |

### Mining / E&P
| Stage | Typical rate |
|-------|-------------|
| Exploration | 15-20% real |
| PEA/PFS | 10-12% real |
| Feasibility/Production | 8-10% real |

Note: mining/E&P typically use **real** discount rates against real cash flows (no inflation). Pharma uses **nominal** against nominal.

---

## Worked Example: Phase II Biotech (single oncology asset)

**Assumptions:**
- Phase II → approval cumulative PoS: 15.2% × (0.70 oncology adjustment) = ~10.6%
- Peak sales forecast: $1.5B (5 years after launch, 50% US share of OCP market, mature year 5)
- Net margin at maturity: 35% (royalty accounting)
- Patent cliff: 13 years post-launch
- Discount rate (conditional on success, Ph2 asset): 22%
- Trial costs (Ph2 ongoing → FDA filing): $150M over 4 years, incurred REGARDLESS of success

**Cash flow schedule (conditional on success):**
```
Year 0-4: -$37.5M/yr trial cost (unconditional — always incurred)
Year 5 (launch): +$50M (launch year, 3% of peak)
Year 6: +$300M (20% of peak)
Year 7: +$750M (50% of peak)
Year 8: +$1,200M (80% of peak)
Year 9+: +$1,500M (100% of peak) until Year 18 patent expiry
Year 19+: +$225M (15% of peak, generic residual)
```

**rNPV calculation:**
```
PV of trial costs (unconditional, 10% WACC since must pay):
= -$37.5M × [(1-(1.10)^-5)/0.10] = -$142M

PV of post-launch cash flows (at 22% conditional rate):
Year 5: $50M / 1.22^5 = $18.6M
Year 6: $300M × 0.35 net margin = $105M / 1.22^6 = $31.8M
Year 7: $262.5M / 1.22^7 = $65.6M
Year 8: $420M / 1.22^8 = $85.6M
Year 9-18: $525M × [(1-1.22^-10)/0.22] / 1.22^8 = $373M
Year 19-25 (generic): $78.75M × [(1-1.22^-7)/0.22] / 1.22^18 = $7M

Total PV of success cash flows: $18.6 + 31.8 + 65.6 + 85.6 + 373 + 7 = $582M

rNPV = 10.6% × $582M − $142M = $61.7M − $142M = -$80M
```

**Interpretation**: Negative rNPV on this asset alone at Phase II means the market is right to not price in this asset at full value. You would need **cumulative PoS ≥24%** (i.e., approval rate improvements or platform-value optionality) to justify investing based on this single asset.

**Sum-of-parts**: Most Phase II biotechs have multiple pipeline assets. Sum the rNPV of each asset to arrive at total pipeline value. Add cash and subtract net debt.

---

## Worked Example: E&P 2P Reserves

**Assumptions:**
- 2P reserves: 180 million boe
- WTI price deck: $75/bbl flat (or use curves)
- Recovery factor: already baked into 2P (no further adjustment)
- Opex: $25/boe
- Tax & royalty: $15/boe
- Capex to develop all 2P: $2.5B over 8 years
- Production profile: ramp to 45 Mboe/day by year 4, decline 10%/yr after year 6
- Discount rate: 10% real
- PoS (2P to commercial): 50%

**Per-boe gross margin**: $75 - $25 opex - $15 tax/royalty = $35 net cash margin

**Production-weighted cash flow** (10-year):
```
Y1: 10 Mbpd × 365 = 3.65M boe × $35 = $128M
Y2: 20 Mbpd × 365 = 7.3M × $35 = $255M
Y3: 35 Mbpd × 365 = 12.8M × $35 = $448M
Y4: 45 Mbpd × 365 = 16.4M × $35 = $575M
Y5: 45 Mbpd × 365 = 16.4M × $35 = $575M
Y6: 45 Mbpd × 365 = 16.4M × $35 = $575M
Y7: 40.5 × 365 = 14.8M × $35 = $517M
Y8: 36.4 × 365 = 13.3M × $35 = $464M
Y9: 32.8 × 365 = 12.0M × $35 = $419M
Y10: 29.5 × 365 = 10.8M × $35 = $377M
```

**Terminal value** (Y11 onwards, at 10% decline continuing):
```
Y11 cash flow = $339M; continued decline approx PV-weighted = $1.4B at Y11 (~$530M PV @ Y0)
```

**Capex schedule** (NEGATIVE):
```
Y1-Y4: -$500M/yr (fully developed by Y4)
```

**PV of operating cash flows** (10% real):
```
Σ Y1-Y10 PV = $128M/1.1 + $255M/1.1² + $448M/1.1³ + $575M/1.1⁴ + $575M/1.1⁵ 
             + $575M/1.1⁶ + $517M/1.1⁷ + $464M/1.1⁸ + $419M/1.1⁹ + $377M/1.1¹⁰
            ≈ $116 + $211 + $337 + $393 + $357 + $325 + $265 + $217 + $178 + $145
            = $2,544M
Plus terminal: $530M  → Operating PV $3,074M
```

**PV of capex** (discounted by year when spent):
```
Σ Capex PV = $500M/1.1 + $500M/1.1² + $500M/1.1³ + $500M/1.1⁴
            ≈ $455 + $413 + $376 + $342 = $1,586M
```

**rNPV = 50% × ($3,074M − $1,586M) = 50% × $1,488M = $744M**

Compared to a naive DCF at 15% WACC (to capture reserve risk) which might yield ~$1.1B, the rNPV approach gives a more honest $744M.

---

## When rNPV Is the Wrong Tool

- **Diversified pharma with ≥15 marketed products**: Too many assets to model individually; use EV/EBITDA multiples.
- **Mature diversified miner (e.g., Rio Tinto, BHP)**: 100+ mines; use EV/EBITDA and NAV replacement.
- **Single-asset E&P with only 1P reserves and producing**: Already commercial; use NAV at 10% WACC, not rNPV.
- **Exploration-stage** (no reserves yet): Too speculative for a point estimate; use scenario framing only (match `frontier` archetype in valuation-calculator).

---

## Integration with `valuation-calculator` SKILL.md

If the target company has the `pipeline_pharma`, `pipeline_biotech`, `commodity_with_reserves`, or `mining_project_stage` archetype, the primary method in Phase 2 is **rNPV** (not DCF, not multiples).

Phase 3 output must include:
1. Per-asset rNPV table (asset name, stage, PoS, peak sales, discount rate, rNPV)
2. Sum-of-parts company value (Σ rNPV + cash − debt)
3. Sensitivity on PoS (±20%) and peak sales (±30%)
4. Named comparable transactions or deals if available (e.g., "Alnylam Phase II oncology assets traded at $300M upfront + $800M in milestones per asset in 2024-2025")

---

## References

- BIO Clinical Development Success Rates 2011-2020 (2021 update)
- SPE PRMS 2018 (Petroleum Resources Management System)
- CIM / NI 43-101 (Canadian mining reserve standards)
- Damodaran on rNPV for biotech (Stern NYU)
- Industry report: IQVIA Global Use of Medicines (annual)
