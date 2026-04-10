# SEC Filing Data Retrieval Guide

## 1. Finding Company Filings on EDGAR

### 1.1 Full-Text Search (Preferred)
```
URL: https://efts.sec.gov/LATEST/search-index?q={TICKER}&dateRange=custom&startdt=YYYY-MM-DD&enddt=YYYY-MM-DD&forms=10-K,10-Q

Alternative (EDGAR full-text search):
  https://efts.sec.gov/LATEST/search-index?q=%22{COMPANY_NAME}%22&forms=10-K
```

### 1.2 Company Filing Page
```
Step 1: Search by ticker or CIK
  https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&company={TICKER}&type=10-K&dateb=&owner=include&count=10

Step 2: From results, navigate to specific filing
Step 3: Find the full filing document (usually .htm or .xml)
```

### 1.3 XBRL Interactive Viewer
```
For structured financial data:
  https://www.sec.gov/cgi-bin/viewer?action=view&cik={CIK}&type=10-K
```

### 1.4 Using WebFetch
```
Fetch the EDGAR search page:
  WebFetch URL: https://efts.sec.gov/LATEST/search-index?q={TICKER}&forms=10-K,10-Q
  Prompt: "Find the most recent 10-K and 10-Q filing links for {COMPANY}"

Then fetch the actual filing for financial data.
```

---

## 2. Key Financial Statements to Extract

### 2.1 Income Statement (Consolidated Statements of Operations)

**Line Items Required:**
| Item | XBRL Tag | Use In Model |
|------|----------|-------------|
| Total Revenue / Net Sales | us-gaap:RevenueFromContractWithCustomerExcludingAssessedTax | Growth rates, P/S |
| Cost of Revenue / COGS | us-gaap:CostOfGoodsAndServicesSold | Gross margin |
| Gross Profit | (calculated) | Margin analysis |
| R&D Expense | us-gaap:ResearchAndDevelopmentExpense | OpEx structure |
| SG&A Expense | us-gaap:SellingGeneralAndAdministrativeExpense | OpEx structure |
| Total Operating Expenses | us-gaap:OperatingExpenses | EBIT calculation |
| Operating Income (EBIT) | us-gaap:OperatingIncomeLoss | EV/EBIT, FCFF |
| Interest Expense | us-gaap:InterestExpense | Cost of debt, FCFF |
| Income Before Tax | us-gaap:IncomeLossFromContinuingOperationsBeforeIncomeTaxes | Tax rate |
| Income Tax Expense | us-gaap:IncomeTaxExpenseBenefit | Effective tax rate |
| Net Income | us-gaap:NetIncomeLoss | EPS, P/E, RI |
| Basic EPS | us-gaap:EarningsPerShareBasic | Valuation |
| Diluted EPS | us-gaap:EarningsPerShareDiluted | Valuation |
| Diluted Shares | us-gaap:WeightedAverageNumberOfDilutedSharesOutstanding | Per-share calcs |
| D&A (often in notes) | us-gaap:DepreciationDepletionAndAmortization | EBITDA, FCFF |

**Derived Calculations:**
```
EBITDA = EBIT + D&A
Gross Margin = Gross Profit / Revenue
EBITDA Margin = EBITDA / Revenue
Net Margin = Net Income / Revenue
Effective Tax Rate = Tax Expense / Pre-tax Income
```

### 2.2 Balance Sheet (Consolidated Balance Sheets)

**Line Items Required:**
| Item | Use In Model |
|------|-------------|
| Cash and Cash Equivalents | EV, net debt |
| Short-term Investments | EV, liquidity |
| Accounts Receivable | NWC, DSO |
| Inventories | NWC, DIO |
| Total Current Assets | Current ratio, NWC |
| PP&E (net) | Asset intensity |
| Goodwill | Impairment risk |
| Intangible Assets | Asset quality |
| Total Assets | ROA, leverage |
| Accounts Payable | NWC, DPO |
| Short-term Debt | Net debt, liquidity |
| Current Portion of LT Debt | Net debt |
| Total Current Liabilities | Current ratio, NWC |
| Long-term Debt | Net debt, WACC |
| Total Liabilities | Leverage |
| Total Stockholders' Equity | ROE, P/B, RI |
| Shares Outstanding | Per-share calcs |

**Derived Calculations:**
```
Net Working Capital = Current Assets (excl cash) - Current Liabilities (excl ST debt)
Net Debt = Total Debt - Cash - Short-term Investments
Book Value Per Share = Total Equity / Shares Outstanding
Debt-to-Equity = Total Debt / Total Equity
Equity Multiplier = Total Assets / Total Equity
```

### 2.3 Cash Flow Statement (Consolidated Statements of Cash Flows)

**Line Items Required:**
| Item | Use In Model |
|------|-------------|
| Net Income | Starting point for CFO |
| D&A | EBITDA, FCFF |
| Stock-Based Compensation | Non-cash charge |
| Changes in Working Capital | ΔNWC for DCF |
| Cash from Operations (CFO) | FCFF, FCFE, P/CF |
| Capital Expenditures | FCF, FCFF, FCFE |
| Acquisitions | Growth vs organic |
| Cash from Investing | Capital allocation |
| Debt Issuance | Net borrowing for FCFE |
| Debt Repayment | Net borrowing for FCFE |
| Share Repurchases | Capital return |
| Dividends Paid | Payout ratio, DDM |
| Cash from Financing | Capital structure |

**Derived Calculations:**
```
Free Cash Flow = CFO - CapEx
FCFF = CFO + Interest × (1 - T) - CapEx
FCFE = CFO - CapEx + Net Borrowing
Net Borrowing = Debt Issued - Debt Repaid
FCF Margin = FCF / Revenue
FCF Yield = FCF / Market Cap
CapEx Intensity = CapEx / Revenue
```

---

## 3. Segment Data (for SOTP)

### 3.1 Where to Find
- 10-K: "Note X — Segment Information" in Notes to Financial Statements
- Look for: Revenue by segment, Operating income by segment, Assets by segment

### 3.2 What to Extract
```
For each segment:
  - Segment revenue (and growth rate)
  - Segment operating income / EBIT
  - Segment EBITDA (if disclosed, or estimate from segment D&A)
  - Segment assets
  - Geographic breakdown (if available)
```

---

## 4. Additional Data Points

### 4.1 From Proxy Statement (DEF 14A)
- Executive compensation structure
- Insider ownership percentages
- Related-party transactions
- Share repurchase authorization

### 4.2 From 10-K Management Discussion
- Revenue guidance or outlook
- Margin trajectory expectations
- Capital expenditure plans
- Debt maturity schedule
- Risk factors (competitive, regulatory, macro)

### 4.3 From Earnings Call Transcripts
- Forward guidance (revenue, EPS, EBITDA)
- Capital allocation priorities
- Industry/end-market commentary
- Management confidence signals

---

## 5. Data Quality Checks

Before using extracted data:

1. **Verify fiscal year**: Some companies have non-calendar fiscal years
   (e.g., TER fiscal year ends 12/31, but some end 6/30 or 9/30)

2. **Match periods**: Ensure income statement and cash flow cover same period

3. **Restatements**: Check for any restated figures in recent filings

4. **Non-GAAP adjustments**: Note differences between GAAP and adjusted figures
   (stock-based comp, restructuring charges, one-time items)

5. **Currency**: Confirm all figures are in USD (or convert consistently)

6. **Share count**: Use diluted shares outstanding for per-share calculations

7. **Segment consistency**: Verify segment definitions haven't changed between periods

---

## 6. Useful SEC EDGAR URLs

```
Company search:
  https://www.sec.gov/cgi-bin/browse-edgar?company={NAME}&CIK=&type=10-K&dateb=&owner=include&count=40&search_text=&action=getcompany

Full-text search:
  https://efts.sec.gov/LATEST/search-index?q={QUERY}&forms=10-K,10-Q,DEF+14A

XBRL financial data:
  https://data.sec.gov/api/xbrl/companyfacts/CIK{CIK_PADDED}.json

Recent filings feed:
  https://data.sec.gov/submissions/CIK{CIK_PADDED}.json
```
