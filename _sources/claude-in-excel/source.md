좋습니다. 스킬별로 하나씩 제공하겠습니다. 각각 claude.ai/customize/skills → Create Skill에서 그대로 붙여넣으시면 됩니다.

분량이 많으니 2개씩 묶어서 순서대로 갑니다.

① audit-xls Copy
Name: audit-xls-copy

Description:

Audit a spreadsheet for formula accuracy, errors, and common mistakes. Scopes to a selected range, a single sheet, or the entire model (including financial-model integrity checks like BS balance, cash tie-out, and logic sanity). Triggers on "audit this sheet", "check my formulas", "find formula errors", "QA this spreadsheet", "sanity check this", "debug model", "model check", "model won't balance", "something's off in my model", "model review".
Instructions:

# Audit Spreadsheet

Audit formulas and data for accuracy and mistakes. Scope determines depth — from quick formula checks on a selection up to full financial-model integrity audits.

## Step 1: Determine scope

If the user already gave a scope, use it. Otherwise **ask them**:

> What scope do you want me to audit?
> - **selection** — just the currently selected range
> - **sheet** — the current active sheet only
> - **model** — the whole workbook, including financial-model integrity checks (BS balance, cash tie-out, roll-forwards, logic sanity)

The **model** scope is the deepest — use it for DCF, LBO, 3-statement, merger, comps, or any integrated financial model before sending to a client or IC.

---

## Step 2: Formula-level checks (ALL scopes)

Run these regardless of scope:

| Check | What to look for |
|---|---|
| Formula errors | `#REF!`, `#VALUE!`, `#N/A`, `#DIV/0!`, `#NAME?` |
| Hardcodes inside formulas | `=A1*1.05` — the `1.05` should be a cell reference |
| Inconsistent formulas | A formula that breaks the pattern of its neighbors in a row/column |
| Off-by-one ranges | `SUM`/`AVERAGE` that misses the first or last row |
| Pasted-over formulas | Cell that looks like a formula but is actually a hardcoded value |
| Circular references | Intentional or accidental |
| Broken cross-sheet links | References to cells that moved or were deleted |
| Unit/scale mismatches | Thousands mixed with millions, % stored as whole numbers |
| Hidden rows/tabs | Could contain overrides or stale calculations |

---

## Step 3: Model-integrity checks (MODEL scope only)

If scope is **model**, identify the model type (DCF / LBO / 3-statement / merger / comps / custom) and run the appropriate integrity checks below.

### 3a. Structural review

| Check | What to look for |
|---|---|
| Input/formula separation | Are inputs clearly separated from calculations? |
| Color convention | Blue=input, black=formula, green=link — or whatever the model uses, applied consistently? |
| Tab flow | Logical order (Assumptions → IS → BS → CF → Valuation)? |
| Date headers | Consistent across all tabs? |
| Units | Consistent (thousands vs millions vs actuals)? |

### 3b. Balance Sheet

| Check | Test |
|---|---|
| BS balances | Total Assets = Total Liabilities + Equity (every period) |
| RE rollforward | Prior RE + Net Income − Dividends = Current RE |
| Goodwill/intangibles | Flow from acquisition assumptions (if M&A) |

If BS doesn't balance, **quantify the gap per period and trace where it breaks** — nothing else matters until this is fixed.

### 3c. Cash Flow Statement

| Check | Test |
|---|---|
| Cash tie-out | CF Ending Cash = BS Cash (every period) |
| CF sums | CFO + CFI + CFF = Δ Cash |
| D&A match | D&A on CF = D&A on IS |
| CapEx match | CapEx on CF matches PP&E rollforward on BS |
| WC changes | Signs match BS movements (ΔAR, ΔAP, ΔInventory) |

### 3d. Income Statement

| Check | Test |
|---|---|
| Revenue build | Ties to segment/product detail |
| Tax | Tax expense = Pre-tax income × tax rate (allow for deferred tax adj) |
| Share count | Ties to dilution schedule (options, converts, buybacks) |

### 3e. Circular references

- Interest → debt balance → cash → interest is a common intentional circ in LBO/3-stmt models
- If intentional: verify iteration toggle exists and works
- If unintentional: trace the loop and flag how to break it

### 3f. Logic & reasonableness

| Check | Flag if |
|---|---|
| Growth rates | >100% revenue growth without explanation |
| Margins | Outside industry norms |
| Terminal value dominance | TV > ~75% of DCF EV (yellow flag) |
| Hockey-stick | Projections ramp unrealistically in out-years |
| Compounding | EBITDA compounds to absurd $ by Year 10 |
| Edge cases | Model breaks at 0% or negative growth, negative EBITDA, leverage goes negative |

### 3g. Model-type-specific bugs

**DCF:**
- Discount rate applied to wrong period (mid-year vs end-of-year)
- Terminal value not discounted back
- WACC uses book values instead of market values
- FCF includes interest expense (should be unlevered)
- Tax shield double-counted

**LBO:**
- Debt paydown doesn't match cash sweep mechanics
- PIK interest not accruing to principal
- Management rollover not reflected in returns
- Exit multiple applied to wrong EBITDA (LTM vs NTM)
- Fees/expenses not deducted from Day 1 equity

**Merger:**
- Accretion/dilution uses wrong share count (pre- vs post-deal)
- Synergies not phased in
- Purchase price allocation doesn't balance
- Foregone interest on cash not included
- Transaction fees not in sources & uses

**3-statement:**
- Working capital changes have wrong sign
- Depreciation doesn't match PP&E schedule
- Debt maturity schedule doesn't match principal payments
- Dividends exceed net income without explanation

---

## Step 4: Report

Output a findings table:

| # | Sheet | Cell/Range | Severity | Category | Issue | Suggested Fix |
|---|---|---|---|---|---|---|

**Severity:**
- **Critical** — wrong output (BS doesn't balance, formula broken, cash doesn't tie)
- **Warning** — risky (hardcodes, inconsistent formulas, edge-case failures)
- **Info** — style/best-practice (color coding, layout, naming)

For **model** scope, prepend a summary line:

> Model type: [DCF/LBO/3-stmt/...] — Overall: [Clean / Minor Issues / Major Issues] — [N] critical, [N] warnings, [N] info

**Don't change anything without asking** — report first, fix on request.

---

## Notes

- **BS balance first** — if it doesn't balance, everything downstream is suspect
- **Hardcoded overrides are the #1 source of silent bugs** — search aggressively
- **Sign convention errors** (positive vs negative for cash outflows) are extremely common
- If the model uses VBA macros, note any macro-driven calculations that can't be audited from formulas alone
② clean-data-xls Copy
Name: clean-data-xls-copy

Description:

Clean up messy spreadsheet data — trim whitespace, fix inconsistent casing, convert numbers-stored-as-text, standardize dates, remove duplicates, and flag mixed-type columns. Use when data is messy, inconsistent, or needs prep before analysis. Triggers on "clean this data", "clean up this sheet", "normalize this data", "fix formatting", "dedupe", "standardize this column", "this data is messy".
Instructions:

# Clean Data

Clean messy data in the active sheet or a specified range.

## Environment

- **If running inside Excel (Office Add-in / Office JS):** Use Office JS directly. Read via `range.values`, write helper-column formulas via `range.formulas = [["=TRIM(A2)"]]`. The in-place vs helper-column decision still applies.
- **If operating on a standalone .xlsx file:** Use Python/openpyxl.

## Workflow

### Step 1: Scope

- If a range is given (e.g. `A1:F200`), use it
- Otherwise use the full used range of the active sheet
- Profile each column: detect its dominant type (text / number / date) and identify outliers

### Step 2: Detect issues

| Issue | What to look for |
|---|---|
| Whitespace | leading/trailing spaces, double spaces |
| Casing | inconsistent casing in categorical columns (`usa` / `USA` / `Usa`) |
| Number-as-text | numeric values stored as text; stray `$`, `,`, `%` in number cells |
| Dates | mixed formats in the same column (`3/8/26`, `2026-03-08`, `March 8 2026`) |
| Duplicates | exact-duplicate rows and near-duplicates (case/whitespace differences) |
| Blanks | empty cells in otherwise-populated columns |
| Mixed types | a column that's 98% numbers but has 3 text entries |
| Encoding | mojibake, non-printing characters |
| Errors | `#REF!`, `#N/A`, `#VALUE!`, `#DIV/0!` |

### Step 3: Propose fixes

Show a summary table before changing anything:

| Column | Issue | Count | Proposed Fix |
|---|---|---|---|

### Step 4: Apply

- **Prefer formulas over hardcoded cleaned values** — where the cleaned output can be expressed as a formula (e.g. `=TRIM(A2)`, `=VALUE(SUBSTITUTE(B2,"$",""))`, `=UPPER(C2)`, `=DATEVALUE(D2)`), write the formula in an adjacent helper column rather than computing the result in Python and overwriting the original. This keeps the transformation transparent and auditable.
- Only overwrite in place with computed values when the user explicitly asks for it, or when no sensible formula equivalent exists (e.g. encoding/mojibake repair)
- For destructive operations (removing duplicates, filling blanks, overwriting originals), confirm with the user first
- After each category of fix (whitespace → casing → number conversion → dates → dedup), show the user a sample of what changed and get confirmation before moving to the next category
- Report a before/after summary of what changed
✅ 2개 완료 (audit-xls, clean-data-xls)

다음은 ③ comps-analysis 와 ④ lbo-model 입니다. 계속 진행할까요?
다음
③ comps-analysis Copy
Name: comps-analysis-copy

Description:

Build institutional-grade comparable company analyses with operating metrics, valuation multiples, and statistical benchmarking in Excel/spreadsheet format. Perfect for public company valuation (M&A, investment analysis), benchmarking performance vs. industry peers, pricing IPOs or funding rounds, identifying valuation outliers, supporting investment committee presentations, creating sector overview reports. Not ideal for private companies without comparable public peers, highly diversified conglomerates, distressed/bankrupt companies, pre-revenue startups, companies with unique business models.
Instructions:

# Comparable Company Analysis

## CRITICAL: Data Source Priority (READ FIRST)

**ALWAYS follow this data source hierarchy:**

1. **FIRST: Check for MCP data sources** - If S&P Kensho MCP, FactSet MCP, or Daloopa MCP are available, use them exclusively for financial and trading information
2. **DO NOT use web search** if the above MCP data sources are available
3. **ONLY if MCPs are unavailable:** Then use Bloomberg Terminal, SEC EDGAR filings, or other institutional sources
4. **NEVER use web search as a primary data source** - it lacks the accuracy, audit trails, and reliability required for institutional-grade analysis

## Overview
Build institutional-grade comparable company analyses that combine operating metrics, valuation multiples, and statistical benchmarking. The output is a structured Excel/spreadsheet that enables informed investment decisions through peer comparison.

**Reference Material:** An example comparable company analysis is provided in `examples/comps_example.xlsx`. Use examples for understanding structural hierarchy and rigor expected, NOT for exact reproduction.

**ALWAYS ask yourself first:**
1. "Do you have a preferred format or should I adapt the template style?"
2. "Who is the audience?" (Investment committee, board presentation, quick reference, detailed memo)
3. "What's the key question?" (Valuation, growth analysis, competitive positioning, efficiency)
4. "What's the context?" (M&A evaluation, investment decision, sector benchmarking, performance review)

## CRITICAL: Formulas Over Hardcodes + Step-by-Step Verification

**Environment — Office JS vs Python:**
- **If running inside Excel (Office Add-in / Office JS):** Use Office JS directly. Write formulas via `range.formulas = [["=E7/C7"]]`, not `range.values`. No separate recalc step — Excel handles it natively.
- **If generating a standalone .xlsx file:** Use Python/openpyxl. Write `cell.value = "=E7/C7"` (formula string).
- **Office JS merged cell pitfall:** Do NOT call `.merge()` then set `.values` on the merged range. Instead write the value to the top-left cell alone, then merge + format the full range.

**Verify step-by-step with the user:**
- After setting up the structure → show the user the header layout before filling data
- After entering raw inputs → show the user the input block and confirm sources/periods before building formulas
- After building operating metrics formulas → show the calculated margins and sanity-check
- After building valuation multiples → show the multiples and confirm they look reasonable before adding statistics
- Do NOT build the entire sheet end-to-end and then present it

---

## Section 1: Document Structure & Setup

### Header Block (Rows 1-3)
Row 1: [ANALYSIS TITLE] - COMPARABLE COMPANY ANALYSIS Row 2: [List of Companies with Tickers] Row 3: As of [Period] | All figures in [USD Millions/Billions] except per-share amounts and ratios

### Visual Convention Standards (OPTIONAL - User preferences override) **Font & Typography:** - Font family: Times New Roman (professional, readable, industry standard) - Font size: 11pt for data cells, 12pt for headers - Bold text: Section headers, company names, statistic labels **Color & Shading — Professional Blue/Grey Palette:** - **Section headers**: Dark blue (`#1F4E79`) background with white bold text - **Column headers**: Light blue (`#D9E1F2`) background with black bold text - **Data rows**: White background, black text for formulas, blue text for hardcoded inputs - **Statistics rows**: Light grey (`#F2F2F2`) background - **That's the whole palette**: dark blue + light blue + light grey + white **Formatting Conventions:** - Percentages: 1 decimal (12.3%) - Multiples: 1 decimal (13.5x) - Dollar amounts: No decimals, thousands separator (69,632) - Borders: No borders (clean, minimal appearance) - Alignment: All metrics center-aligned - Column widths: Uniform/even --- ## Section 2: Operating Statistics & Financial Metrics ### Core Columns 1. **Company** - Names with consistent formatting 2. **Revenue** - Size metric (LTM, quarterly, or annual) 3. **Revenue Growth** - Year-over-year percentage change 4. **Gross Profit** - Revenue minus COGS 5. **Gross Margin** - GP/Revenue 6. **EBITDA** - Earnings before interest, tax, depreciation, amortization 7. **EBITDA Margin** - EBITDA/Revenue ### Optional Additions (Choose based on industry/purpose) - Free Cash Flow / FCF Margin - Net Income / Net Margin - Operating Income - Rule of 40 (for SaaS) - CapEx metrics (for asset-heavy industries) ### Formula Examples ```excel Gross Margin (F7): =E7/C7 EBITDA Margin (H7): =G7/C7
Statistics Block (After company data)

[One blank row for visual separation] - Maximum: =MAX(B7:B9) - 75th Percentile: =QUARTILE(B7:B9,3) - Median: =MEDIAN(B7:B9) - 25th Percentile: =QUARTILE(B7:B9,1) - Minimum: =MIN(B7:B9)
Columns that NEED statistics: Revenue Growth %, Gross Margin %, EBITDA Margin %, EPS, EV/Revenue, EV/EBITDA, P/E, Dividend Yield %, Beta

Columns that DON'T need statistics: Revenue, EBITDA, Net Income (absolute size), Market Cap, Enterprise Value

Section 3: Valuation Multiples & Investment Metrics
Core Valuation Columns

Company - Same order as operating section
Market Cap - Current market valuation
Enterprise Value - Market Cap ± Net Debt/Cash
EV/Revenue - How much market pays per dollar of sales
EV/EBITDA - How much market pays per dollar of earnings
P/E Ratio - Price relative to net earnings
Optional Valuation Metrics

FCF Yield, PEG Ratio, Price/Book, ROE/ROA, Revenue/EBITDA CAGR, Debt/Equity
Cross-Reference Rule

CRITICAL: Valuation multiples MUST reference the operating metrics section. Never input the same raw data twice.

Statistics Block

Same structure as operating section: Max, 75th, Median, 25th, Min for every metric.

Section 4: Notes & Methodology Documentation
Required Components

Data Sources & Quality: Where data came from, period covered, how verified Key Definitions: EBITDA calculation method, FCF formula, special metrics explained Valuation Methodology: EV calculation, growth rates used, adjustments made Analysis Framework: Investment thesis, key metrics, how to interpret statistics

Section 5: Choosing the Right Metrics (Decision Framework)
The "5-10 Rule"

5 operating metrics - Revenue, Growth, 2-3 margins/efficiency metrics 5 valuation metrics - Market Cap, EV, 3 multiples = 10 total columns - Enough to tell the story

If you have more than 15 metrics, you're probably including noise.

Industry-Specific Metric Selection

Software/SaaS: Revenue Growth, Gross Margin, Rule of 40, ARR, Net Dollar Retention Manufacturing/Industrials: EBITDA Margin, Asset Turnover, CapEx/Revenue Financial Services: ROE, ROA, Efficiency Ratio, P/E, Net Interest Margin Retail/E-commerce: Revenue Growth, Gross Margin, Inventory Turnover

Section 6: Best Practices & Quality Checks
Sanity Checks

Gross margin > EBITDA margin > Net margin (always true by definition)
EV/Revenue: typically 0.5-20x; EV/EBITDA: typically 8-25x; P/E: typically 10-50x
Higher growth usually means higher multiples
Larger companies often have better margins (scale benefits)
Common Mistakes to Avoid

Mixing market cap and enterprise value in formulas
Using different time periods for numerator and denominator
Hardcoding numbers into formulas instead of cell references
Hard-coded inputs without cell comments citing the source
Including too many metrics without clear purpose
Including non-comparable companies (different business models)
Section 7: Workflow & Practical Tips
Step-by-Step Process

Set up structure — Create all headers, format cells, lock in units
Gather data — Pull from MCP sources or Bloomberg/SEC, input raw numbers in blue, document sources
Build formulas — Start with simple ratios, progress to multiples, add cross-checks
Add statistics — Copy formula structure, verify ranges, check quartile logic
Quality control — Run sanity checks, verify references, check for errors
Documentation — Complete notes section, add data sources, date-stamp
Section 8: Red Flags & Warning Signs
Data Quality Issues

Inconsistent time periods (mixing quarterly and annual)
Missing data without explanation
Significant differences between data sources (>10% variance)
Valuation Red Flags

Negative EBITDA companies valued on EBITDA multiples (use revenue multiples instead)
P/E ratios >100x without hypergrowth story
Margins that don't make sense for the industry
Comparability Issues

Different fiscal year ends
Mixing pure-play and conglomerates
Materially different business models labeled as "comps"
When in doubt, exclude the company. Better to have 3 perfect comps than 6 questionable ones.

Output Checklist
 All companies are truly comparable
 Data is from consistent time periods
 Units are clearly labeled
 Formulas reference cells, not hardcoded values
 All hard-coded input cells have comments with source or assumption
 Statistics include Max, 75th, Median, 25th, Min
 Notes section documents sources and methodology
 Blue = input, Black = formula formatting
 Sanity checks pass
 Date stamp is current
 Formula auditing shows no errors
--- ## ④ lbo-model Copy **Name:** `lbo-model-copy` **Description:**
Complete LBO (Leveraged Buyout) model templates in Excel for private equity transactions, deal materials, or investment committee presentations. Fills in formulas, validates calculations, and ensures professional formatting standards that adapt to any template structure.

**Instructions:** ```markdown # LBO Model Template Completion ## TEMPLATE REQUIREMENT **This skill uses templates for LBO models. Always check for an attached template file first.** Before starting any LBO model: 1. **If a template file is attached/provided**: Use that template's structure exactly 2. **If no template is attached**: Ask the user if they have one, or use the standard template 3. **If using the standard template**: Copy `examples/LBO_Model.xlsx` as your starting point **IMPORTANT**: When a file like `LBO_Model.xlsx` is attached, you MUST use it as your template - do not build from scratch. --- ## CRITICAL INSTRUCTIONS - READ FIRST ### Environment: Office JS vs Python **If running inside Excel (Office Add-in / Office JS):** - Use Office JS directly — do NOT use Python/openpyxl - Write formulas via `range.formulas = [["=B5*B6"]]` — formulas recalculate natively - Use `range.format.font.color` / `range.format.fill.color` for color conventions - **Merged cell pitfall:** Do NOT call `.merge()` then set `.values` on the merged range. Write value to top-left cell alone, then merge + format the full range. **If generating a standalone .xlsx file:** - Use Python/openpyxl, write formula strings, then run `recalc.py` before delivery ### Core Principles * **Every calculation must be an Excel formula** - NEVER compute values and hardcode results * **Use the template structure** - Follow the organization in the template * **Use proper cell references** - All formulas should reference appropriate cells * **Maintain sign convention consistency** - Follow template convention throughout * **Work section by section, verify with user at each step** ### Formula Color Conventions * **Blue (0000FF)**: Hardcoded inputs - typed numbers that don't reference other cells * **Black (000000)**: Formulas with calculations * **Purple (800080)**: Links to cells on the **same tab** * **Green (008000)**: Links to cells on **different tabs** ### Fill Color Palette — Professional Blues & Greys * **Section headers**: Dark blue `#1F4E79` with white bold text * **Column headers**: Light blue `#D9E1F2` with black bold text * **Input cells**: Light grey `#F2F2F2` with blue font * **Formula/calculated cells**: White, no fill * **Key outputs** (IRR, MOIC): Medium blue `#BDD7EE` with black bold text ### Number Formatting Standards * **Currency**: `$#,##0;($#,##0);"-"` or `$#,##0.0` * **Percentages**: `0.0%` (one decimal) * **Multiples**: `0.0"x"` (one decimal) * **MOIC/Detailed Ratios**: `0.00"x"` (two decimals) * **All numeric cells**: Right-aligned --- ## TEMPLATE ANALYSIS PHASE - DO THIS FIRST Before filling any formulas: 1. **Map the structure** - Identify all sections and how they relate 2. **Understand the timeline** - Which columns represent which periods 3. **Identify input vs formula cells** - Respect template conventions 4. **Read existing labels carefully** - Labels tell you what calculation is expected 5. **Check for existing formulas** - Don't overwrite working formulas 6. **Note template-specific conventions** - Sign conventions, subtotal structures --- ## FILLING FORMULAS - GENERAL APPROACH ### Step 1: Check the Template * Does the cell already have a formula? Verify and move on. * Is there a comment or note indicating expected calculation? * Does the row/column label make the calculation obvious? * Do neighboring cells show a pattern? ### Step 2: Check the User's Instructions * Did the user specify a particular calculation method? * Are there stated assumptions? ### Step 3: Apply Standard Practice * Use standard LBO modeling conventions * Document any assumptions you make --- ## COMMON PROBLEM AREAS ### Balancing Sections * When Sources = Uses, one item is typically the "plug" (balancing figure) * Identify which item is the plug and calculate it as the difference ### Tax Calculations * Tax formulas should only reference relevant income line and tax rate * Consider whether losses create tax shields ### Interest and Circular References * Use **Beginning Balance** (not average or ending) to break circular references * Pattern: Interest → Cash Flow → Paydown → Ending Balance ### Debt Paydown / Cash Sweeps * Respect the priority waterfall when multiple tranches exist * Balances cannot go negative - use MAX or MIN functions ### Returns Calculations (IRR/MOIC) * Cash flows must have correct signs: Investment = negative, Proceeds = positive * MOIC = Total Proceeds / Total Investment ### Sensitivity Tables * **Use ODD dimensions** (5×5 or 7×7) — never 4×4 or 6×6 * **Center cell = base case.** Build axis values symmetrically around actual assumptions * **Highlight the center cell** — medium-blue fill (`#BDD7EE`) + bold font * Each cell should show a DIFFERENT value — if all same, formulas aren't varying correctly * Use mixed references (`$A5` for row input, `B$4` for column input) --- ## VERIFICATION CHECKLIST - RUN AFTER COMPLETION ### Section Balancing - [ ] Sources/Uses balance exactly - [ ] Plug items calculated correctly ### Income/Operating Projections - [ ] Revenue builds correctly from drivers - [ ] All cost items calculated appropriately - [ ] Subtotals and totals sum correctly - [ ] Margins are reasonable ### Debt/Financing Schedules - [ ] Beginning balances tie to sources or prior period - [ ] Interest calculated on beginning balance - [ ] Paydowns respect cash availability and priority - [ ] Ending balances cannot be negative - [ ] Totals sum tranches correctly ### Returns/Output Analysis - [ ] Cash flow signs are correct (negative for investment, positive for proceeds) - [ ] IRR/MOIC formulas reference complete ranges - [ ] Results are reasonable ### Sensitivity Tables - [ ] Grid dimensions are ODD (5×5 or 7×7) - [ ] Row/column axis values symmetric around base case - [ ] Center cell output equals model's actual IRR/MOIC - [ ] Center cell highlighted (medium-blue fill, bold font) - [ ] Each data cell contains a formula (not hardcoded) - [ ] Each data cell shows a DIFFERENT value - [ ] Values move in expected directions ### Formatting - [ ] Blue inputs, black formulas, purple same-tab links, green cross-tab links - [ ] All numbers right-aligned - [ ] Appropriate number formats applied - [ ] No error values (#REF!, #DIV/0!, #VALUE!, #NAME?) --- ## COMMON ERRORS TO AVOID | Error | What Goes Wrong | How to Fix | |-------|-----------------|------------| | Hardcoding calculated values | Model doesn't update when inputs change | Always use formulas | | Wrong cell references after copying | Formulas point to wrong cells | Verify all links, use $ anchoring | | Circular reference errors | Model can't calculate | Use beginning balances | | Sections don't balance | Totals that should match don't | Ensure one item is the plug | | Negative balances where impossible | Paying more than available | Use MAX(0, ...) or MIN | | IRR/return errors | Wrong signs or incomplete ranges | Check signs and ranges | | Sensitivity table shows same value | Formula not varying | Check mixed references ($A5, B$4) | --- ## WORKING WITH THE USER — SECTION-BY-SECTION CHECKPOINTS * **After Sources & Uses** → show the balanced table, confirm the plug is correct * **After Operating Model / Projections** → show the projected P&L, confirm growth rates and margins * **After Debt Schedule** → show beginning/ending balances and interest, confirm waterfall logic * **After Returns (IRR/MOIC)** → show cash flow series and outputs, confirm signs and ranges * **After Sensitivity Tables** → show that each cell varies, confirm base case lands where expected * **If errors are found during verification**, fix them before moving to next section * **Never present a completed model without having checked in at each section**
✅ 4개 완료 (audit-xls, clean-data-xls, comps-analysis, lbo-model)

다음은 ⑤ dcf-model 과 ⑥ 3-statement-model 입니다. 계속할까요?
다음
DCF 모델은 원본이 ~50,000자로 매우 깁니다. 핵심을 유지하면서 제공하겠습니다.

Working
⑤ dcf-model Copy
Name: dcf-model-copy

Description:

Real DCF (Discounted Cash Flow) model creation for equity valuation. Retrieves financial data from SEC filings and analyst reports, builds comprehensive cash flow projections with proper WACC calculations, performs sensitivity analysis, and outputs professional Excel models with executive summaries. Use when users need to value a company using DCF methodology, request intrinsic value analysis, or ask for detailed financial modeling with growth projections and terminal value calculations.
Instructions:

# DCF Model Builder

## Overview

This skill creates institutional-quality DCF models for equity valuation following investment banking standards. Each analysis produces a detailed Excel model with sensitivity analysis included at the bottom of the DCF sheet.

## Tools

- Default to using all of the information provided by the user and MCP servers available for data sourcing.

## Critical Constraints - Read These First

**Environment: Office JS vs Python/openpyxl:**
- **If running inside Excel (Office Add-in / Office JS):** Use Office JS directly. Write formulas via `range.formulas = [["=D19*(1+$B$8)"]]`. No separate recalc step needed.
- **If generating a standalone .xlsx file:** Use Python/openpyxl, then run `recalc.py` before delivery.
- **Office JS merged cell pitfall:** Write value to top-left cell alone, then merge and format the full range.

**Formulas Over Hardcodes (NON-NEGOTIABLE):**
- Every projection, margin, discount factor, PV, and sensitivity cell MUST be a live Excel formula
- The only hardcoded numbers permitted are: (1) raw historical inputs, (2) assumption drivers, (3) current market data
- If you catch yourself computing something in Python and writing the result — STOP.

**Verify Step-by-Step With the User (DO NOT build end-to-end):**
- After data retrieval → show raw inputs block, confirm before projecting
- After revenue projections → show projected top line, confirm before margin build
- After FCF build → show full FCF schedule, confirm before WACC
- After WACC → show calculation and inputs, confirm before discounting
- After terminal value + PV → show equity bridge, confirm before sensitivity tables

**Sensitivity Tables:**
- **Use ODD number of rows and columns** (standard: 5×5, sometimes 7×7)
- **Center cell = base case.** Build axis values so middle row/column exactly equal actual assumptions
- **Highlight the center cell** with medium-blue fill (`#BDD7EE`) + bold font
- Populate ALL cells (typically 3 tables × 25 cells = 75) with full DCF recalculation formulas
- NO placeholder text, NO linear approximations, NO manual steps required

**Cell Comments:**
- Add cell comments AS each hardcoded value is created
- Format: "Source: [System/Document], [Date], [Reference], [URL if applicable]"
- Every blue input must have a comment before moving to next section

---

## DCF Process Workflow

### Step 1: Data Retrieval and Validation

Fetch data from MCP servers, user provided data, and the web.

**Data Sources Priority:**
1. MCP Servers (if configured)
2. User-Provided Data
3. Web Search/Fetch — current prices, beta, debt and cash

**Validation Checklist:**
- Verify net debt vs net cash (critical for valuation)
- Confirm diluted shares outstanding
- Validate historical margins are consistent with business model
- Cross-check revenue growth rates with industry benchmarks
- Verify tax rate is reasonable (typically 21-28%)

### Step 2: Historical Analysis (3-5 years)

Analyze and document:
- Revenue growth trends (calculate CAGR, identify drivers)
- Margin progression (gross margin, EBIT margin, FCF margin)
- Capital intensity (D&A and CapEx as % of revenue)
- Working capital efficiency (NWC changes as % of revenue growth)

### Step 3: Build Revenue Projections

**Three-scenario approach:**
- Bear Case: Conservative growth
- Base Case: Most likely scenario
- Bull Case: Optimistic growth

**Formula structure:**
- Revenue(Year N) = Revenue(Year N-1) × (1 + Growth Rate)
- Growth Rate Framework: Year 1-2 higher, Year 3-4 moderating, Year 5+ approaching terminal

**Consolidation Column Pattern (using INDEX):**
- Case selector cell (e.g., B6) contains 1=Bear, 2=Base, 3=Bull
- `=INDEX(B10:D10, 1, $B$6)` — centralizes scenario logic
- Projection formulas reference consolidation column (clean cell references)

### Step 4: Operating Expense Modeling

Operating expenses based on REVENUE (not gross profit):
- Sales & Marketing: typically 15-40% of revenue
- Research & Development: typically 10-30%
- General & Administrative: typically 8-15%

Model operating leverage: % should decline as revenue scales.

### Step 5: Free Cash Flow Calculation
EBIT (-) Taxes (EBIT × Tax Rate) = NOPAT (+) D&A (% of revenue) (-) CapEx (% of revenue, typically 4-8%) (-) Δ NWC (% of revenue change) = Unlevered Free Cash Flow

### Step 6: Cost of Capital (WACC) **CAPM Methodology:**
Cost of Equity = Risk-Free Rate + Beta × Equity Risk Premium After-Tax Cost of Debt = Pre-Tax Cost of Debt × (1 - Tax Rate) WACC = (Cost of Equity × Equity Weight) + (After-Tax Cost of Debt × Debt Weight)

Typical WACC Ranges: - Large Cap, Stable: 7-9% - Growth Companies: 9-12% - High Growth/Risk: 12-15% ### Step 7: Discount Rate Application **Mid-Year Convention:** - Discount Period: 0.5, 1.5, 2.5, 3.5, 4.5 - Discount Factor = 1 / (1 + WACC)^Period - PV of FCF = Unlevered FCF × Discount Factor ### Step 8: Terminal Value Calculation **Perpetuity Growth Method (Preferred):**
Terminal FCF = Final Year FCF × (1 + Terminal Growth Rate) Terminal Value = Terminal FCF / (WACC - Terminal Growth Rate)

Terminal Growth Rate: 2.0-3.5% (do not exceed risk-free rate or long-term GDP growth) **Terminal Value Sanity Check:** - Should represent 50-70% of Enterprise Value - If >75%, model may be over-reliant on terminal assumptions ### Step 9: Enterprise to Equity Value Bridge
(+) Sum of PV of Projected FCFs (+) PV of Terminal Value = Enterprise Value (-) Net Debt [or + Net Cash] = Equity Value ÷ Diluted Shares Outstanding = Implied Price per Share vs. Current Stock Price → Implied Return

### Step 10: Sensitivity Analysis Build **three sensitivity tables** at bottom of DCF sheet: 1. **WACC vs Terminal Growth** — enterprise value sensitivity 2. **Revenue Growth vs EBIT Margin** — impact of top-line growth and operating leverage 3. **Beta vs Risk-Free Rate** — sensitivity to cost of equity components Each cell must contain a full DCF recalculation formula for that assumption combination. Use mixed references (`$A88` for row input, `B$87` for column input). --- ## Excel Model Structure ### Sheet Architecture — Two sheets: 1. **DCF** — Main valuation model with sensitivity analysis at bottom 2. **WACC** — Cost of capital calculation ### Formatting Standards **Font Colors (MANDATORY):** - Blue text (#0000FF): ALL hardcoded inputs - Black text (#000000): ALL formulas and calculations - Green text (#008000): Links to other sheets **Fill Colors — Professional Blue/Grey Palette:** - Section headers: Dark blue (`#1F4E79`) with white bold text - Sub-headers/column headers: Light blue (`#D9E1F2`) with black bold text - Input cells: Light grey (`#F2F2F2`) with blue font - Calculated cells: White with black font - Output/summary rows: Medium blue (`#BDD7EE`) with black bold font **Number Formats:** - Years: Format as text strings ("2024" not "2,024") - Percentages: `0.0%` - Currency: `$#,##0` for millions; `$#,##0.00` for per-share - Zeros: Use "-" format - Negative numbers: Parentheses `(#,##0)` **Border Standards (REQUIRED):** - Thick borders (1.5pt) around major sections - Medium borders (1pt) between sub-sections - Thin borders (0.5pt) around data tables ### DCF Sheet Layout **Section 1: Header** — Company name, ticker, date, case selector **Section 2: Market Data** — Stock price, shares, market cap, net debt (NOT case dependent) **Section 3: Scenario Assumptions** — Separate Bear/Base/Bull blocks with column headers showing years **Section 4: Historical & Projected Financials** — Revenue, costs, EBIT, taxes, NOPAT **Section 5: Free Cash Flow Build** — NOPAT + D&A - CapEx - ΔNWC = UFCF **Section 6: Discount Factors & PV** — Mid-year discount factors, PV of FCFs **Section 7: Terminal Value** — Perpetuity growth + exit multiple methods **Section 8: Valuation Summary** — EV → Equity bridge → Implied price per share **Section 9: Sensitivity Tables** — 3 tables × 5×5 grid at bottom --- ## Common Mistakes to Avoid 1. **Formula row references off** → Define ALL row positions BEFORE writing formulas 2. **Missing cell comments** → Add comments AS cells are created, not at end 3. **Simplified sensitivity tables** → Populate all cells with full DCF recalc formulas 4. **Scenario block references wrong** → Ensure consolidation column pulls from correct blocks 5. **No borders** → Add professional section borders 6. **OpEx based on gross profit** → Always base on REVENUE 7. **Terminal growth > WACC** → Creates infinite value, must be lower 8. **Wrong discount period** → Use mid-year convention (0.5, 1.5, 2.5...) ## Quality Rubric 1. Realistic revenue and margin assumptions based on historical performance 2. Appropriate WACC calculation with proper CAPM methodology 3. Comprehensive sensitivity analysis showing valuation ranges 4. Clear terminal value calculation with supporting rationale 5. Professional model structure enabling scenario analysis 6. Transparent documentation of all key assumptions
⑥ 3-statement-model Copy
Name: 3-statement-model-copy

Description:

Complete, populate and fill out 3-statement financial model templates (Income Statement, Balance Sheet, Cash Flow Statement). Use when asked to fill out model templates, complete existing model frameworks, populate financial models with data, complete a partially filled IS/BS/CF framework, or link integrated financial statements within an existing template structure.
Instructions:

# 3-Statement Financial Model Template Completion

Complete and populate integrated financial model templates with proper linkages between Income Statement, Balance Sheet, and Cash Flow Statement.

## CRITICAL PRINCIPLES — Read Before Populating Any Template

**Environment — Office JS vs Python:**
- **If running inside Excel (Office Add-in / Office JS):** Use Office JS directly. Write formulas via `range.formulas`. No separate recalc needed.
- **If generating a standalone .xlsx file:** Use Python/openpyxl, then run `recalc.py` before delivery.
- **Office JS merged cell pitfall:** Write value to top-left cell alone, then merge + format the full range.

**Formulas over hardcodes (non-negotiable):**
- Every projection cell, roll-forward, linkage, and subtotal MUST be an Excel formula
- The ONLY cells that should contain hardcoded numbers are: (1) historical actuals, (2) assumption drivers

**Verify step-by-step with the user:**
1. After mapping the template → show which tabs/sections identified, confirm
2. After populating historicals → show historical block, confirm values match source
3. After building IS projections → run subtotal checks, confirm before BS
4. After building BS → show balance check (Assets = L+E) every period, confirm before CF
5. After building CF → show cash tie-out (CF ending cash = BS cash), confirm before finalizing

## Formatting — Professional Blue/Grey Palette

| Element | Fill | Font |
|---|---|---|
| Section headers | Dark blue `#1F4E79` | White bold |
| Column headers | Light blue `#D9E1F2` | Black bold |
| Input cells | Light grey `#F2F2F2` or white | Blue `#0000FF` |
| Formula cells | White | Black |
| Cross-tab links | White | Green `#008000` |
| Check rows / key totals | Medium blue `#BDD7EE` | Black bold |

---

## Model Structure

### Identifying Template Tab Organization

| Common Tab Names | Contents |
|---|---|
| IS, P&L, Income Statement | Income Statement |
| BS, Balance Sheet | Balance Sheet |
| CF, CFS, Cash Flow | Cash Flow Statement |
| WC, Working Capital | Working Capital Schedule |
| DA, D&A, Depreciation, PP&E | Depreciation & Amortization Schedule |
| Debt, Debt Schedule | Debt Schedule |
| NOL, Tax, DTA | Net Operating Loss Schedule |
| Assumptions, Inputs, Drivers | Driver assumptions and inputs |
| Checks, Audit, Validation | Error-checking dashboard |

### Template Review Checklist
- Identify which tabs exist (not all templates include every schedule)
- Note template-specific tabs not listed above
- Understand tab dependencies (Assumptions → IS → BS → CF)
- Locate input cells vs. formula cells

---

## Completing Model Templates

### Step 1: Analyze the Template Structure

**Identify Input vs. Formula Cells:**
- Blue font = inputs, Black font = formulas, Green font = links to other sheets
- Use Trace Precedents/Dependents to understand cell relationships

**Map the Template's Flow:**
- Which tabs feed into others
- Note supporting schedules and their linkages
- Document template's specific line items

### Step 2: Filling in Data Without Breaking Formulas

| Rule | Description |
|---|---|
| Only edit input cells | Never overwrite cells containing formulas |
| Preserve cell references | Use Paste Values to avoid overwriting formulas |
| Match the template's units | Verify thousands, millions, or actual values |
| Respect sign conventions | Follow existing convention |
| Check for circular references | Enable Iterative Calculation if needed |

### Step 3: Validating Formulas

| Check Type | Method |
|---|---|
| Trace precedents | Verify formula references correct inputs |
| Trace dependents | Verify key inputs flow to expected outputs |
| Check for hardcodes | Projection formulas should reference assumptions |
| Cross-tab consistency | Same formula logic across all projection periods |

### Step 4: Quality Checks by Sheet

**Income Statement:**
- Revenue matches source data for historical periods
- All expense line items sum to reported totals
- Subtotals (Gross Profit, EBIT, EBT, Net Income) calculate correctly
- Tax calculation handles losses correctly
- Forecast drivers reference assumptions tab (no hardcodes)

**Balance Sheet:**
- Assets = Liabilities + Equity for every period (PRIMARY CHECK)
- Cash balance matches CF ending cash
- RE rolls forward: Prior RE + Net Income - Dividends = Ending RE
- Debt balances tie to debt schedule

**Cash Flow Statement:**
- Net Income at top of CFO matches IS Net Income
- Non-cash add-backs (D&A, SBC) tie to source schedules
- WC changes have correct signs (increase in asset = negative)
- CapEx ties to PP&E schedule
- Ending Cash matches BS Cash
- Beginning Cash = prior period Ending Cash

### Step 5: Cross-Statement Integrity Checks

| Check | Formula | Expected |
|---|---|---|
| Balance Sheet Balance | Assets - Liabilities - Equity | = 0 |
| Cash Tie-Out | CF Ending Cash - BS Cash | = 0 |
| Net Income Link | IS Net Income - CF Starting Net Income | = 0 |
| Retained Earnings | Prior RE + NI - Dividends - BS Ending RE | = 0 |

### Step 6: Final Review

- Toggle through all scenarios to verify checks pass in each case
- Review all #REF!, #DIV/0!, #VALUE!, #NAME? errors
- Confirm all input cells have been populated
- Verify units are consistent across all tabs

---

## Scenario Analysis (Base / Upside / Downside)

Use a scenario toggle (dropdown) in the Assumptions tab with CHOOSE or INDEX/MATCH formulas.

| Scenario | Description |
|---|---|
| Base Case | Management guidance or consensus estimates |
| Upside Case | Above-guidance growth, margin expansion |
| Downside Case | Below-trend growth, margin compression |

**Key Drivers to Sensitize:** Revenue growth, Gross margin, SG&A %, DSO/DIO/DPO, CapEx %, Interest rate, Tax rate

**Scenario Audit Checks:** Toggle switches all statements, BS balances in all scenarios, Cash ties out, Hierarchy holds (Upside > Base > Downside for NI, EBITDA, FCF, margins)

---

## Model Validation and Audit

### Core Linkages (Must Always Hold)

| Check | Expected |
|---|---|
| BS Balance | Assets - Liabilities - Equity = 0 |
| Cash Tie-Out | CF Ending Cash = BS Cash |
| NI Link | IS Net Income = CF Starting Net Income |
| RE Roll-forward | Prior RE + NI + SBC - Dividends = Ending RE |
| Equity Financing | ΔCommon Stock/APIC (BS) = Equity Issuance (CFF) |

### Sign Convention Reference

| Statement | Item | Sign |
|---|---|---|
| CFO | D&A, SBC | Positive (add-back) |
| CFO | ΔAR (increase) | Negative (use of cash) |
| CFO | ΔAP (increase) | Positive (source of cash) |
| CFI | CapEx | Negative |
| CFF | Debt issuance | Positive |
| CFF | Debt repayments | Negative |
| CFF | Dividends | Negative |

### Circular Reference Handling

Interest expense creates circularity: Interest → Net Income → Cash → Debt Balance → Interest

Enable iterative calculation: File → Options → Formulas → Enable iterative calculation. Set maximum iterations to 100, maximum change to 0.001. Add a circuit breaker toggle in Assumptions tab.

### Check Categories

1. **Currency Consistency** — All tabs use consistent currency and scale
2. **Balance Sheet Integrity** — Assets = L + E for each period
3. **Cash Flow Integrity** — Cash ties to BS, NI ties to IS, D&A ties to schedule
4. **Retained Earnings** — RE roll-forward check
5. **Working Capital** — AR, Inventory, AP tie to BS; DSO/DIO/DPO reasonability
6. **Debt Schedule** — Total Debt ties to BS, Interest ties to IS
7. **Equity Financing** — Equity issuance ties to BS changes
8. **NOL Schedule** — Beginning NOL = 0, NOL only when EBT < 0, utilization ≤ 80%
9. **Scenario Hierarchy** — Upside > Base > Downside for absolute metrics
10. **Formula Integrity** — Consistent formulas across projection years, no errors

### Master Check Formula
- All sections pass → "✓ ALL CHECKS PASS"
- Any section fails → "✗ ERRORS DETECTED - REVIEW BELOW"

### Quick Debug Workflow
1. Find red-highlighted sections
2. Identify which check category has failures
3. Navigate to source tab to investigate
4. Fix the underlying issue
5. Return to Checks tab to verify resolution
✅ 6개 완료 (audit-xls, clean-data-xls, comps-analysis, lbo-model, dcf-model, 3-statement-model)

나머지 ⑦ skill-creator 와 ⑧ mcp-builder는 원본이 description만 있어서 복사할 상세 내용이 없습니다.

전체 요약:

#	스킬	상태
①	audit-xls-copy	✅ 제공 완료
②	clean-data-xls-copy	✅ 제공 완료
③	comps-analysis-copy	✅ 제공 완료
④	lbo-model-copy	✅ 제공 완료
⑤	dcf-model-copy	✅ 제공 완료
⑥	3-statement-model-copy	✅ 제공 완료
⑦	skill-creator	⚠️ description만 존재
⑧	mcp-builder	⚠️ description만 존재
각 스킬의 Name, Description, Instructions 를 claude.ai/customize/skills → Create Skill에서 그대로 붙여넣으시면 됩니다! 추가로 개선하거나 수정할 부분이 있으면 말씀해주세요.

좋은 질문입니다! 스킬 안에는 Instructions(마크다운) 외에 추가 참조 파일들이 포함되어 있습니다. 앞서 제공한 코드블록에는 메인 Instructions만 있었고, 아래 파일들이 빠져있었습니다:

추가 파일 목록
스킬	추가 파일	유형
3-statement-model	references/formulas.md	텍스트 (수식 레퍼런스)
3-statement-model	references/sec-filings.md	텍스트 (SEC 데이터 추출 가이드)
3-statement-model	references/formatting.md	텍스트 (포맷 표준)
lbo-model	examples/LBO_Model.xlsx	바이너리 (엑셀 템플릿)
comps-analysis	examples/comps_example.xlsx	바이너리 (엑셀 예제)
텍스트 파일 3개 (그대로 복사 가능)
📄 3-statement-model — references/formulas.md

# Formula Reference

**IMPORTANT:** Use the formulas outlined in this reference document unless otherwise specified by the user.

---

## Core Linkages

Balance Sheet:        Assets = Liabilities + Equity
Net Income:           IS Net Income → CF Operations (starting point)
Cash Flow:            ΔCash = CFO + CFI + CFF
Cash Tie-Out:         Ending Cash (CF) = Cash (BS Asset)
Cash Monthly/Annual:  Closing Cash (Monthly) = Closing Cash (Annual)
Retained Earnings:    Prior RE + Net Income - Dividends = Ending RE
Equity Raise:         ΔCommon Stock/APIC (BS) = Equity Issuance (CFF)
Year 0 Equity:        Equity Raised (Year 0) = Beginning Equity (Year 1)

## Gross Profit Calculation

Net Revenue - Cost of Revenue = Gross Profit

| Term | Definition |
|------|------------|
| Gross Revenue | Total revenue before any deductions |
| Net Revenue | Gross Revenue - Returns - Allowances - Discounts |
| Cost of Revenue | Direct costs attributable to production |
| Gross Profit | Net Revenue - Cost of Revenue |

## Margin Formulas

Gross Margin %      = Gross Profit / Net Revenue
EBITDA              = EBIT + D&A  (or = Gross Profit - OpEx)
EBITDA Margin %     = EBITDA / Net Revenue
EBIT Margin %       = EBIT / Net Revenue
Net Income Margin % = Net Income / Net Revenue

## Credit Metric Formulas

Total Debt            = Current Portion of Debt + Long-Term Debt
Net Debt              = Total Debt - Cash
Total Debt / EBITDA   = Total Debt / EBITDA (from IS)
Net Debt / EBITDA     = Net Debt / EBITDA (from IS)
Interest Coverage     = EBITDA / Interest Expense (from IS)
Debt / Total Cap      = Total Debt / (Total Debt + Total Equity)
Debt / Equity         = Total Debt / Total Equity
Current Ratio         = Total Current Assets / Total Current Liabilities
Quick Ratio           = (Total Current Assets - Inventory) / Total Current Liabilities

## Forecast Formulas (% of Net Revenue Method)

Cost of Revenue (Forecast) = Net Revenue × Cost of Revenue % Assumption
S&M (Forecast)             = Net Revenue × S&M % Assumption
G&A (Forecast)             = Net Revenue × G&A % Assumption
R&D (Forecast)             = Net Revenue × R&D % Assumption
SBC (Forecast)             = Net Revenue × SBC % Assumption

## Working Capital Formulas

Accounts Receivable:
  Prior AR + Revenue (from IS) - Cash Collections (plug) = Ending AR
  DSO = (AR / Revenue) × 365

Inventory:
  Prior Inventory + Purchases (plug) - COGS (from IS) = Ending Inventory
  DIO = (Inventory / COGS) × 365

Accounts Payable:
  Prior AP + Purchases (from Inventory calc) - Cash Payments (plug) = Ending AP
  DPO = (AP / COGS) × 365

Net Working Capital = AR + Inventory - AP
ΔWC = Current NWC - Prior NWC

## D&A Schedule Formulas

Beginning PP&E (Gross) + CapEx = Ending PP&E (Gross)
Beginning Accumulated Depreciation + Depreciation Expense = Ending Accumulated Depreciation
PP&E (Net) = Gross PP&E - Accumulated Depreciation

## Debt Schedule Formulas

Beginning Debt Balance + New Borrowings - Repayments = Ending Debt Balance
Interest Expense = Avg Debt Balance × Interest Rate
  (Use beginning balance to avoid circularity)

## Retained Earnings Formula

Beginning RE + Net Income + SBC - Dividends = Ending RE

## NOL Schedule Formulas

Beginning NOL Balance (Year 1 = 0)
+ NOL Generated (if EBT < 0, then ABS(EBT), else 0)
- NOL Utilized (limited by taxable income and utilization cap)
= Ending NOL Balance

NOL Utilization: MIN(NOL Available, EBT × 80%)
Tax Expense: MAX(0, Taxable Income × Tax Rate)
DTA - NOL = Ending NOL Balance × Tax Rate

## Balance Sheet Structure

ASSETS: Cash + AR + Inventory = Total Current Assets
        PP&E Net + DTA = Total Non-Current Assets
        Total Assets

LIABILITIES: AP + Current Debt = Total Current Liabilities
             LT Debt = Total Liabilities

EQUITY: Common Stock + Retained Earnings = Total Equity

CHECK: Assets - Liabilities - Equity = 0

## Cash Flow Statement Structure

CFO: Net Income + D&A + SBC - ΔDTA - ΔAR - ΔInventory + ΔAP
CFI: -CapEx
CFF: +Debt Issuance - Debt Repayment + Equity Issuance - Dividends

Net Change = CFO + CFI + CFF
Beginning Cash + Net Change = Ending Cash (→ BS Cash)

## Income Statement Structure

Net Revenue
(-) Cost of Revenue → Gross Profit
(-) S&M, G&A, R&D, D&A, SBC → EBIT
EBITDA = EBIT + D&A
(-) Interest → EBT
(-) NOL Utilization → Taxable Income
(-) Taxes → Net Income

## Check Formulas

BS Balance:     Assets - Liabilities - Equity = 0
Cash Tie-Out:   BS Cash - CF Ending Cash = 0
RE Roll-Forward: Prior RE + NI + SBC - Div - BS RE = 0
DTA Tie-Out:    NOL Schedule DTA - BS DTA = 0
Equity Raise:   ΔCommon Stock/APIC - Equity Issuance (CFF) = 0
NOL Cap:        NOL Utilized ≤ EBT × 80%
NOL Non-Neg:    Ending NOL ≥ 0
📄 3-statement-model — references/sec-filings.md

# SEC Filings Data Extraction Reference

**When to Use:** Only when a model template requires pulling data from SEC filings (10-K, 10-Q).

## Step 1: Locate the Filing
SEC EDGAR: https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=[TICKER]&type=10-K

## Step 2: Identify Filing Currency
Check cover page or statement headers for reporting currency and scale.

| Indicator | Currency |
|-----------|----------|
| $, USD | US Dollar |
| €, EUR | Euro |
| £, GBP | British Pound |
| ¥, JPY | Japanese Yen |
| ¥, CNY, RMB | Chinese Yuan |

## Step 3: Navigate to Financial Statements
- Item 8 (10-K) or Item 1 (10-Q)
- Consolidated Statements of Operations (IS)
- Consolidated Balance Sheets (BS)
- Consolidated Statements of Cash Flows (CF)
- Notes to Financial Statements

## Step 4: Data Extraction Mapping

**Income Statement:**
| Filing Line Item | Model Line Item |
|------------------|-----------------|
| Net revenues | Revenue |
| Cost of goods sold | COGS |
| SG&A | SG&A |
| D&A | D&A |
| Interest expense, net | Interest Expense |
| Income tax expense | Taxes |
| Net income | Net Income |

**Balance Sheet:**
| Filing Line Item | Model Line Item |
|------------------|-----------------|
| Cash and equivalents | Cash |
| Accounts receivable | AR |
| Inventories | Inventory |
| PP&E, net | PP&E |
| Total assets | Total Assets |
| Accounts payable | AP |
| Short-term debt | Current Debt |
| Long-term debt | LT Debt |
| Retained earnings | Retained Earnings |
| Total equity | Total Equity |

**Cash Flow Statement:**
| Filing Line Item | Model Line Item |
|------------------|-----------------|
| Net income | Net Income |
| D&A | D&A |
| Changes in AR | ΔAR |
| Changes in inventory | ΔInventory |
| Changes in AP | ΔAP |
| Capital expenditures | CapEx |
| Equity issuance | Equity Issuance |
| Debt activity | Debt activity |
| Dividends | Dividends |

## Step 5: Notes Detail
- Debt → Maturity schedule, rates, covenants
- PP&E → Gross PP&E, accumulated depreciation, useful lives
- Revenue → Segment breakdowns
- Leases → Operating vs. finance

## Step 6: Historical Data Requirements
- 10-K: 3 years IS/CF, 2 years BS
- Prior year 10-K for 3rd year BS
- 10-Q for quarterly granularity

## Handling Variations
| Variation | How to Handle |
|-----------|---------------|
| D&A embedded in COGS/SG&A | Pull from Cash Flow Statement |
| Material "Other" items | Check notes for breakdown |
| Restatements | Use restated figures |
| Fiscal ≠ calendar year | Label with fiscal year end |
📄 3-statement-model — references/formatting.md

# Formatting Standards Reference

| Element | Format |
|---------|--------|
| Hard-coded inputs | Blue font |
| Formulas | Black font |
| Links to other sheets | Green font |
| Check cells | Red if error, green if balanced |
| Negative values | Parentheses, not minus signs |
| Currency | No decimals for large, 2 decimals for per-share |
| Percentages | 1 decimal place |
| Headers | Bold, bottom border |
| Units row | Include below headers ($ millions, %, etc.) |

## Visual Separation
- Thin vertical border between historical and projected
- Thick bottom border after section totals
- Single bottom border for subtotals
- Double bottom border for grand totals

## Bold Formatting for Totals

**IS:** Gross Revenue, Total COGS, Gross Profit, Total SG&A, EBITDA, EBIT, EBT, Net Profit
**BS:** Total Current/Non-Current Assets, Total Assets, Total Current/Non-Current Liabilities, Total Equity, Total L+E
**CF:** Cash from Ops before WC, Total WC Changes, Net CFO, Net CFI, Net CFF, Closing Cash

## Balance Sheet Check Row
| Check = 0 | Black (standard) |
| Check ≠ 0 | Red |

Format: [Red][<>0]0.00;[Red][<>0](0.00);0.00

## Margin Row Formatting
- Indent, italics, 1 decimal place

## Credit Metric Formatting
- Leverage multiples: 1 decimal + "x"
- Percentages: 1 decimal + "%"
- Net Debt negative: Parentheses (net cash)

## Credit Metric Thresholds
| Metric | Green | Yellow | Red |
|--------|-------|--------|-----|
| Debt/EBITDA | <2.5x | 2.5-4.0x | >4.0x |
| Net Debt/EBITDA | <2.0x | 2.0-3.5x | >3.5x |
| Interest Coverage | >4.0x | 2.5-4.0x | <2.5x |
| Debt/Total Cap | <40% | 40-60% | >60% |
| Current Ratio | >1.5x | 1.0-1.5x | <1.0x |
| Quick Ratio | >1.0x | 0.75-1.0x | <0.75x |
