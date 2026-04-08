1. dcf-model 스킬
파일 구조

dcf-model/ └── skill.md ← 단일 파일 (~54,000자, 가장 방대한 스킬)
skill.md 전체 구조

---
name: dcf-model
description: Real DCF (Discounted Cash Flow) model creation for equity valuation. Retrieves financial data from SEC filings and analyst reports, builds comprehensive cash flow projections with proper WACC calculations, performs sensitivity analysis, and outputs professional Excel models with executive summaries. Use when users need to value a company using DCF methodology, request intrinsic value analysis, or ask for detailed financial modeling with growth projections and terminal value calculations.
---

# DCF Model Builder

## Overview
Institutional-quality DCF models for equity valuation following investment banking standards.
Each analysis produces a detailed Excel model (with sensitivity analysis at bottom of DCF sheet).

## Tools
- Default to using all information provided by the user and MCP servers available for data sourcing.

## Critical Constraints - Read These First

### Environment: Office JS vs Python/openpyxl
- Office JS: Use range.formulas = [["=D19*(1+$B$8)"]] — never range.values for derived cells
- Python/openpyxl: Write ws["D15"] = "=D14*(1+Assumptions!$B$5)", then run recalc.py
- Office JS merged cell pitfall: Write value to top-left cell alone, then merge + format

### Formulas Over Hardcodes (NON-NEGOTIABLE)
- Every projection/margin/discount factor/PV/sensitivity cell MUST be live Excel formula
- Only permitted hardcodes: (1) raw historical inputs, (2) assumption drivers, (3) current market data
- "If you catch yourself computing something in Python and writing the result — STOP"

### Verify Step-by-Step With the User (DO NOT build end-to-end)
1. After data retrieval → confirm raw inputs
2. After revenue projections → confirm top line + growth rates
3. After FCF build → confirm FCF schedule logic
4. After WACC → confirm calculation + inputs
5. After terminal value + PV → confirm equity bridge
6. After sensitivity tables → final review

### Sensitivity Tables
- Use ODD number of rows/columns (5×5 or 7×7)
- Center cell = base case (must equal model's actual implied share price)
- Highlight center cell with #BDD7EE + bold
- Populate ALL cells with full DCF recalculation formulas (75 total for 3 tables)
- NO placeholder text, NO linear approximations

### Cell Comments
- Add AS each hardcoded value is created
- Format: "Source: [System/Document], [Date], [Reference], [URL]"
- Never defer to end or write "TODO: add source"

### Model Layout Planning
- Define ALL section row positions BEFORE writing formulas
- Write ALL headers → section dividers → THEN formulas

## DCF Process Workflow

### Step 1: Data Retrieval and Validation
- Priority: MCP Servers → User-Provided Data → Web Search/Fetch
- Validation: net debt vs net cash, diluted shares, historical margins, growth rates, tax rate

### Step 2: Historical Analysis (3-5 years)
- Revenue growth trends (CAGR), margin progression, capital intensity, WC efficiency, return metrics

### Step 3: Build Revenue Projections
- Revenue(Year N) = Revenue(Year N-1) × (1 + Growth Rate)
- Three-scenario approach: Bear/Base/Bull
- Growth rates: Year 1-2 higher → Year 3-4 moderate → Year 5+ approaching terminal

### Step 4: Operating Expense Modeling
- S&M, R&D, G&A — ALL percentages based on REVENUE, not gross profit
- Model operating leverage: % should decline as revenue scales

### Step 5: Free Cash Flow Calculation
- EBIT → (-)Taxes → NOPAT → (+)D&A → (-)CapEx → (-)ΔNWC → Unlevered FCF

### Step 6: Cost of Capital (WACC) Research
- CAPM: Cost of Equity = Risk-Free Rate + Beta × ERP
- After-Tax Cost of Debt = Pre-Tax × (1 - Tax Rate)
- WACC = (Ke × We) + (Kd × Wd)

### Step 7: Discount Rate Application
- Mid-year convention: periods 0.5, 1.5, 2.5...
- Discount Factor = 1 / (1 + WACC)^Period

### Step 8: Terminal Value Calculation
- Perpetuity Growth: TV = Terminal FCF / (WACC - g)
- Exit Multiple: TV = Final Year EBITDA × Exit Multiple
- Sanity check: TV should be 50-70% of EV

### Step 9: Enterprise to Equity Value Bridge
- Sum PV FCFs + PV Terminal Value = EV
- EV - Net Debt = Equity Value
- Equity Value / Diluted Shares = Implied Price per Share

### Step 10: Sensitivity Analysis
Three tables: (1) WACC vs Terminal Growth, (2) Revenue Growth vs EBIT Margin, (3) Beta vs Risk-Free Rate

## <correct_patterns> section
- Scenario Block Selection: INDEX/consolidation column approach (not scattered IF statements)
- Revenue Projection: Reference consolidation column
- FCF Formula: Use consolidation columns with INDEX
- Cell Comment Format: exact source with date/URL
- Assumption Table Structure: 3 elements per block (header + column headers + data rows)
- Row Planning Process: headers first → dividers → formulas
- Sensitivity Table: 5×5 grid, symmetric axes, center = base case, formula in every cell

## <common_mistakes> section
- WRONG: Linear approximations in sensitivity tables
- WRONG: Placeholder text instead of formulas
- WRONG: Missing cell comments
- WRONG: Formula row references off (write formulas before headers)
- WRONG: Single row per assumption across scenarios
- WRONG: No borders
- WRONG: Wrong font colors
- WRONG: OpEx based on Gross Profit instead of Revenue
- TOP 5 ERRORS: row references, comments, sensitivity, scenario references, borders

## Excel Model Structure
### Sheet Architecture: 2 sheets — DCF + WACC
### Formatting Standards
- Font Colors: Blue=#0000FF (inputs), Black (formulas), Green=#008000 (cross-sheet)
- Fill Colors: Dark blue #1F4E79 (headers), Light blue #D9E1F2 (sub-headers), Light grey #F2F2F2 (inputs), White (formulas), Medium blue #BDD7EE (outputs)
- Borders: Thick 1.5pt (major sections), Medium 1pt (sub-sections), Thin 0.5pt (data tables)
- Number Formats: Years as text, % as 0.0%, Currency $#,##0, Negatives in parentheses

### DCF Sheet Detailed Structure (Sections 1-5 + Sensitivity)
[Detailed row-by-row layout with formula patterns]

## Quality Rubric
1. Realistic assumptions, 2. Proper WACC, 3. Comprehensive sensitivity, 4. Clear terminal value, 5. Professional structure, 6. Transparent documentation
핵심 특징

항목	값
분량	~54,000자 (전체 스킬 중 최대)
워크플로우 단계	10단계
사용자 체크포인트	6회 (각 주요 섹션 완성 후)
시트 구조	2개 (DCF + WACC)
감도분석 테이블	3개 × 5×5 = 75 수식 셀
시나리오	Bear/Base/Bull 3-case
고유 섹션	<correct_patterns> + <common_mistakes> 패턴/안티패턴 분리

Used a tool
2. lbo-model 스킬
파일 구조

lbo-model/ ├── skill.md ← 메인 스킬 정의서 (~12,000자) └── examples/ └── LBO_Model.xlsx ← 표준 템플릿 (바이너리)
skill.md 전체 구조

---
name: lbo-model
description: This skill should be used when completing LBO (Leveraged Buyout) model templates in Excel for private equity transactions, deal materials, or investment committee presentations. The skill fills in formulas, validates calculations, and ensures professional formatting standards that adapt to any template structure.
---

## TEMPLATE REQUIREMENT
- If template attached → use that template's structure exactly
- If no template → ask user, or use standard examples/LBO_Model.xlsx
- NEVER build from scratch when template is provided

## CRITICAL INSTRUCTIONS - READ FIRST

### Environment: Office JS vs Python
- Office JS: range.formulas = [["=B5*B6"]], no recalc needed
- Python/openpyxl: ws["D20"] = "=B5*B6", run recalc.py before delivery
- Merged cell pitfall: value to top-left first, then merge + format

### Core Principles
- Every calculation must be an Excel formula — NEVER compute in Python and hardcode
- Use the template structure — follow existing organization
- Work section by section, verify with user at each step

### Formula Color Conventions (4-color system)
- Blue (0000FF): Hardcoded inputs
- Black (000000): Formulas with calculations
- Purple (800080): Links to cells on SAME tab (direct references, no calculation)
- Green (008000): Links to cells on DIFFERENT tabs

### Fill Color Palette — Professional Blues & Greys
- Section headers: Dark blue #1F4E79 (white bold text)
- Column headers: Light blue #D9E1F2 (black bold text)
- Input cells: Light grey #F2F2F2
- Formula cells: White
- Key outputs (IRR, MOIC): Medium blue #BDD7EE (black bold)

### Number Formatting Standards
- Currency: $#,##0;($#,##0);"-"
- Percentages: 0.0%
- Multiples: 0.0"x"
- MOIC/Detailed Ratios: 0.00"x"

## TEMPLATE ANALYSIS PHASE
1. Map the structure — identify sections and relationships
2. Understand the timeline — columns = periods, pro forma column
3. Identify input vs formula cells — respect template conventions
4. Read existing labels — they specify expected calculations
5. Check for existing formulas — don't overwrite working formulas
6. Note template-specific conventions — sign conventions, subtotal structures

## FILLING FORMULAS - GENERAL APPROACH
### Step 1: Check the Template (existing formula? comment? label? pattern?)
### Step 2: Check the User's Instructions
### Step 3: Apply Standard Practice

## COMMON PROBLEM AREAS
- Balancing Sections: Sources = Uses, one item is "plug"
- Tax Calculations: reference income line + tax rate only
- Interest and Circular References: use Beginning Balance to break circularity
- Debt Paydown / Cash Sweeps: priority waterfall, MAX/MIN to prevent negative
- Returns Calculations (IRR/MOIC): correct signs, consecutive periods
- Sensitivity Tables: ODD dimensions, center cell = base case, #BDD7EE highlight

## VERIFICATION CHECKLIST
- [ ] Section balancing (Sources = Uses)
- [ ] Income projections (subtotals, margins, links)
- [ ] Balance Sheet (Assets = L+E, beginning = prior ending)
- [ ] Cash Flow (correct signs, ending cash ties)
- [ ] Supporting Schedules (roll-forwards balance)
- [ ] Debt Schedule (beginning balance, interest, paydown priority)
- [ ] Returns (IRR/MOIC correct signs and ranges)
- [ ] Sensitivity Tables (ODD grid, center = base, all cells have formulas)
- [ ] Formatting (blue/black/purple/green, number formats, no errors)
- [ ] Logical Sanity Checks (magnitude, trends, reasonableness)

## COMMON ERRORS TO AVOID
| Hardcoding | Wrong cell refs | Circular refs | Sections unbalanced |
| Negative balances | IRR/return errors | Sensitivity same value | Roll-forwards don't tie |
| Inconsistent signs |

## WORKING WITH THE USER — SECTION-BY-SECTION CHECKPOINTS
1. After Sources & Uses → confirm plug, get sign-off
2. After Operating Model → confirm growth rates, margins
3. After Debt Schedule → confirm waterfall logic
4. After Returns (IRR/MOIC) → confirm cash flow signs
5. After Sensitivity Tables → confirm base case lands correctly
핵심 특징

항목	값
분량	~12,000자
고유 요소	템플릿 필수 (examples/LBO_Model.xlsx)
색상 체계	4색 (Blue/Black/Purple/Green) — 유일하게 Purple 사용
사용자 체크포인트	5회 (섹션별)
감도분석	ODD 그리드, center = base case
순환참조 처리	Beginning Balance로 break
검증 체크리스트	10개 카테고리, ~40개 항목

Used a tool
3. comps-analysis 스킬
파일 구조

comps-analysis/ ├── skill.md ← 메인 스킬 정의서 (~18,000자) └── examples/ └── comps_example.xlsx ← 예시 파일 (바이너리)
skill.md 전체 구조

---
name: comps-analysis
description: Build institutional-grade comparable company analyses with operating metrics, valuation multiples, and statistical benchmarking in Excel/spreadsheet format.
  Perfect for: Public company valuation, benchmarking, IPO pricing, outlier identification, IC presentations, sector overviews
  Not ideal for: Private companies without public peers, conglomerates, distressed/bankrupt, pre-revenue startups, unique business models
---

# Comparable Company Analysis

## ⚠️ CRITICAL: Data Source Priority (READ FIRST)
1. FIRST: Check for MCP data sources (S&P Kensho, FactSet, Daloopa)
2. DO NOT use web search if MCPs available
3. ONLY if MCPs unavailable: Bloomberg, SEC EDGAR
4. NEVER use web search as primary data source

## Overview
Institutional-grade comps combining operating metrics, valuation multiples, statistical benchmarking.

## Reference Material & Contextualization
- examples/comps_example.xlsx for structural hierarchy understanding
- DO use for: structure, rigor level, principles
- DO NOT use for: exact reproduction, copying without context
- ALWAYS ask: format preference? audience? key question? context?
- Adapt for: industry, sector, company familiarity, decision type

## ⚠️ Formulas Over Hardcodes + Step-by-Step Verification
- Office JS: range.formulas, not range.values
- Merged cell pitfall: value to top-left first
- Step-by-step: structure → raw inputs → operating metrics → valuation multiples → statistics

## Section 1: Document Structure & Setup
### Header Block (Rows 1-3): Title, Companies list, Date/Units
### Visual Convention Standards (OPTIONAL - user prefs override)
- Font: Times New Roman 11pt (data), 12pt (headers)
- Color Palette: Dark blue #1F4E79/#17365D (headers), Light blue #D9E1F2 (column headers), White (data), Light grey #F2F2F2 (statistics)
- Decimal precision: % 1 decimal, multiples 1 decimal, $ no decimals
- No borders (clean minimal appearance)
- All metrics center-aligned
- Uniform column widths + consistent row heights

## Section 2: Operating Statistics & Financial Metrics
### Core Columns: Company, Revenue, Revenue Growth, Gross Profit, Gross Margin, EBITDA, EBITDA Margin
### Optional: FCF, FCF Margin, Net Income, Operating Income, CapEx, Rule of 40, FCF Conversion
### Statistics Block: Maximum, 75th Percentile, Median, 25th Percentile, Minimum
- Statistics for comparable metrics (ratios, margins, multiples) — NOT size metrics (absolute $)
- One blank row between data and statistics — NO "SECTOR STATISTICS" header

## Section 3: Valuation Multiples & Investment Metrics
### Core: Company, Market Cap, Enterprise Value, EV/Revenue, EV/EBITDA, P/E
### Optional: FCF Yield, PEG Ratio, Price/Book, ROE/ROA, CAGR, Asset Turnover, Debt/Equity
### Cross-Reference Rule: Multiples MUST reference operating metrics section
### Statistics Block: Same structure (Max, 75th, Med, 25th, Min)

## Section 4: Notes & Methodology Documentation
- Data Sources & Quality, Key Definitions, Valuation Methodology, Analysis Framework

## Section 5: Choosing the Right Metrics (Decision Framework)
- "Which is undervalued?" → EV/Rev, EV/EBITDA, P/E
- "Which is most efficient?" → margins
- "Which is growing fastest?" → growth rates
- "Which generates most cash?" → FCF metrics
### Industry-Specific: SaaS, Manufacturing, Financial Services, Retail
### The "5-10 Rule": 5 operating + 5 valuation = 10 total

## Section 6: Best Practices & Quality Checks
- Cell comments on ALL hard-coded inputs (source OR assumption)
- Sanity: Gross > EBITDA > Net margin
- Multiple ranges: EV/Rev 0.5-20x, EV/EBITDA 8-25x, P/E 10-50x
### Common Mistakes: mixing market cap/EV, inconsistent periods, hardcodes without comments

## Section 6 (Advanced): Dynamic Headers, Quartile Analysis, Industry Modifications

## Section 7: Workflow & Practical Tips
1. Set up structure (30min) → 2. Gather data (60-90min) → 3. Build formulas (30min) → 4. Add statistics (15min) → 5. Quality control (30min) → 6. Documentation (15min)

## Section 8: Example Template Layout (Simple ASCII art grid)

## Section 9: Industry-Specific Additions (Optional)
SaaS, Financial Services, E-commerce, Healthcare, Manufacturing

## Section 10: Red Flags & Warning Signs
Data quality, valuation, comparability issues

## Section 11: Formulas Reference Guide
Statistical + Financial + Cross-Sheet + Formatting formulas

## Key Principles Summary (7 points)

## Output Checklist (~15 items)
핵심 특징

항목	값
분량	~18,000자
고유 요소	데이터 소스 우선순위 (MCP > Bloomberg > Web), "5-10 Rule"
산업별 메트릭	SaaS, 금융, 제조, 소매, 헬스케어 5개
통계 블록	Max, 75th, Median, 25th, Min (5-point)
포맷 특이사항	Times New Roman, No borders (다른 스킬과 반대), center-aligned
사용자 체크포인트	4회 (structure → inputs → metrics → multiples)
Section 번호 중복	Section 6이 2개 (Best Practices + Advanced)

Used a tool
4. skill-creator 스킬
파일 구조

skill-creator/ └── (description only — no skill.md or reference files exposed)
정의

---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.
---
참고: read_skill 호출 시 description만 반환됨 — 내부 워크플로우 파일이 노출되지 않는 메타 스킬입니다. 스킬을 만들고/수정하고/테스트하는 "스킬을 위한 스킬"로, 실제 실행 시 내부 로직으로 동작합니다.

기능 범위

기능	설명
스킬 생성	새로운 skill.md + 참조 파일 작성
스킬 수정	기존 스킬 개선/최적화
Eval 실행	스킬 성능 테스트
벤치마크	분산 분석으로 성능 측정
Description 최적화	트리거 정확도 향상을 위한 description 튜닝
5. mcp-builder 스킬
파일 구조

mcp-builder/ └── (description only — no skill.md or reference files exposed)
정의

---
name: mcp-builder
description: Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. Use when building MCP servers to integrate external APIs or services, whether in Python (FastMCP) or Node/TypeScript (MCP SDK).
---
참고: skill-creator와 마찬가지로 description만 노출되는 메타 스킬입니다. MCP 서버 개발 가이드로, Excel 도메인이 아닌 개발자 도구입니다.

기능 범위

기능	설명
Python MCP 서버	FastMCP 프레임워크 기반
Node/TypeScript MCP 서버	MCP SDK 기반
외부 API 통합	LLM이 외부 서비스와 상호작용하도록 도구 설계

Used a tool
이제 전체 8개 스킬 비교 분석 시트를 생성하겠습니다.

Create skill comparison analysis sheet
워크북 포커스 문제가 발생했습니다. 다시 시도하겠습니다.

Create skill comparison sheet (no activate)
Border style 이슈입니다. 수정하겠습니다.

Create skill comparison sheet (fixed borders), used a tool
모든 작업이 완료되었습니다.

📊 최종 요약
완성된 산출물

채팅 문서 (이전 3개 + 이번 5개 = 전체 8개)

#	스킬	문서 포함 내용	상태
1	audit-xls	skill.md 원본 + 8단계 워크플로우 + 적응적 판단 분석	✅ (이전)
2	3-statement-model	skill.md + formulas.md + formatting.md + sec-filings.md 4파일 전체	✅ (이전)
3	clean-data-xls	skill.md 단일파일 + 4단계 워크플로우	✅ (이전)
4	dcf-model	skill.md 전체 구조 (~54K자, 10단계, correct_patterns/common_mistakes)	✅
5	lbo-model	skill.md + template 구조 (4색 체계, 6단계, 검증체크리스트 40항목)	✅
6	comps-analysis	skill.md + example 구조 (11 sections, 5-10 Rule, 산업별 메트릭)	✅
7	skill-creator	description only (메타 스킬 — 내부 로직 비공개)	✅
8	mcp-builder	description only (메타 스킬 — 내부 로직 비공개)	✅
시트 산출물


스킬 비교 분석
 시트에 5개 Section으로 구성된 전체 비교표:

Section 1: 기본 정보 (8개 스킬 × 8개 항목)
Section 2: 기술적 특성 (8개 스킬 × 10개 항목)
Section 3: 트리거 키워드
Section 4: 공통 설계 원칙 (8개)
Section 5: 복잡도 스펙트럼 (순위 1~8)
핵심 발견사항

dcf-model이 압도적 최대 스킬 (~54K자) — correct_patterns/common_mistakes를 인라인으로 전부 포함
lbo-model만 4색 체계 (Purple 추가: 같은 탭 내 직접 참조)
comps-analysis만 No borders 정책 (다른 재무 스킬은 모두 borders 사용)
skill-creator와 mcp-builder는 메타 스킬 — description만 노출, 내부 워크플로우 비공개
6개 재무 스킬 공통: Formula-first, 단계별 확인, Blue/Grey 팔레트, Merged Cell 주의
lbo-model 표준템플릿 만들어줘 시트 만들어서
comps_example 파일도 여기에 만들어줘 시트 추가해서
대규모 작업이므로 계획을 먼저 세우겠습니다. 이전에 분석한 스킬 문서를 기반으로 두 개의 시트를 만들어야 합니다.

계획:

LBO_Model 시트 — lbo-model 스킬의 표준 템플릿 구조:

Sources & Uses
Operating Model (P&L 프로젝션)
Debt Schedule
Returns Analysis (IRR/MOIC)
Sensitivity Tables
Comps_Example 시트 — comps-analysis 스킬의 예시 파일 구조:

Operating Statistics & Financial Metrics (with statistics block)
Valuation Multiples & Investment Metrics (with statistics block)
Notes & Methodology
두 시트 모두 수식 포함, 색상 컨벤션 적용, 숫자 포맷 적용으로 실제 사용 가능한 완성된 템플릿으로 만들겠습니다. 진행할까요, 아니면 조정할 부분이 있나요?