스킬 원본을 확인했습니다. 이제 실제 실행된 워크플로우와 스킬 정의서를 비교 분석하여 완전한 복원 문서를 작성하겠습니다.

📋 audit-xls 스킬 복원 문서
1. skill.md — 스킬 정의서 (원본 그대로)
---
name: audit-xls
description: Audit a spreadsheet for formula accuracy, errors, and common mistakes. Scopes to a selected range, a single sheet, or the entire model (including financial-model integrity checks like BS balance, cash tie-out, and logic sanity). Triggers on "audit this sheet", "check my formulas", "find formula errors", "QA this spreadsheet", "sanity check this", "debug model", "model check", "model won't balance", "something's off in my model", "model review".
---

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
2. 실제 실행 워크플로우 분석 — 스킬 vs 실행 비교
🔍 Step 1: 스코프 결정

스킬 정의	실제 실행
사용자가 스코프를 지정하지 않으면 selection/sheet/model 중 선택 요청	사용자가 "audit this [sheet or workbook]" + "fix any formula errors, broken references, or formatting errors"로 workbook 전체 스코프를 명시 → 질문 없이 바로 진행
model 스코프는 재무모델용	이 워크북은 재무모델이 아닌 재고관리 보고서이므로 model 스코프의 3a~3g 단계는 해당 없음 → 전체 워크북 대상 formula-level checks로 실행
판정: ✅ 적절 — 사용자 의도를 올바르게 해석하고 불필요한 질문 없이 진행

🔍 Step 2: Formula-level checks 실행 내역

체크 항목	스킬 정의	실제 실행 여부	실행 방법
Formula errors (#REF!, #VALUE! 등)	✅ 필수	✅ 실행	execute_office_js로 852행 전체 스캔, 에러 문자열 필터링
Hardcodes inside formulas	✅ 필수	✅ 실행	모든 셀이 하드코딩이라는 더 심각한 발견 — formulas 속성 로드 후 =로 시작하는 수식 0건 확인
Inconsistent formulas	✅ 필수	⚠️ 해당없음	수식이 0개이므로 일관성 검사 대상 자체가 없음
Off-by-one ranges	✅ 필수	⚠️ 해당없음	SUM/AVERAGE 수식이 없으므로 범위 오류 검사 불가
Pasted-over formulas	✅ 필수	✅ 실행	소계/합계 행이 모두 정적값 → "수식이어야 할 셀이 하드코딩" 발견
Circular references	✅ 필수	✅ 암묵적 확인	수식 0개 → 순환참조 불가능
Broken cross-sheet links	✅ 필수	✅ 실행	3개 시트 간 교차값 검증 (월별/분기별 총계 일치 확인)
Unit/scale mismatches	✅ 필수	✅ 실행	수량 소계(kg 환산)와 개별 행(팩/통/봉 단위) 불일치 발견 → Info로 보고
Hidden rows/tabs	✅ 필수	✅ 실행	visibility 속성 + columnHidden 속성 확인 → 숨겨진 시트/열 없음
판정: ✅ 전체 항목 커버 — 수식이 없는 상황에서 해당없는 항목도 왜 해당없는지 논리적으로 확인

🔍 Step 3: Model-integrity checks

스킬 정의: model 스코프일 때만 실행 실제 실행: 재무모델이 아니므로 3a~3g 전체 SKIP

판정: ✅ 적절 — 재고관리 보고서에 BS balance/CF tie-out 검사는 불필요

🔍 Step 4: 보고서 출력

스킬 정의	실제 실행
테이블 형식 (#, Sheet, Cell/Range, Severity, Category, Issue, Suggested Fix)	✅ 정확히 동일한 7열 테이블로 출력
Severity: Critical / Warning / Info	✅ 3단계 모두 사용 (0 Critical, 2 Warning, 1 Info)
model 스코프면 요약 라인 추가	model 스코프 아니지만 유사한 요약 라인 추가 ("Overall: Minor Issues")
"Don't change anything without asking"	✅ 보고 후 "Would you like me to fix any of these issues?" 로 마무리
판정: ✅ 완벽 준수

3. 실제 실행된 도구 호출 순서 (Tool Call Trace)
이 부분이 핵심입니다 — 스킬이 실제로 어떤 순서로 어떤 도구를 호출했는지 복원합니다:

Phase 1: 전체 시트 데이터 읽기 (병렬) ├─ get_cell_ranges("입출고현황", "A1:Q20") — 헤더/구조 파악 ├─ get_cell_ranges("월별차트", "A1:J17") — 전체 시트 읽기 └─ get_cell_ranges("요약", "A1:H21") — 전체 시트 읽기 Phase 2: 수식 존재 여부 확인 ├─ get_cell_ranges("입출고현황", "E18:L18", "A18:D18") — 소계 행 수식 확인 ├─ get_cell_ranges("월별차트", "C17:J17", "I5:J16") — 합계/Q1 수식 확인 ├─ get_cell_ranges("요약", subtotal rows) — 소계/총계 수식 확인 └─ execute_office_js: .formulas 속성 로드 — ⚠️ 결과: 수식 0건, 전부 하드코딩 Phase 3: 입출고현황 전체 스캔 (852행) └─ execute_office_js: 200행 청크로 분할 스캔 ├─ 소계 행 위치 파악 (B열 "소계" 패턴 매칭) → 70개 소계 행 발견 ├─ 금액 계산 검증 (H=E*G, L=I*K) → 개별 행 전부 정확 ├─ 에러 값 스캔 (#REF! 등) → 에러 0건 └─ 수식 존재 스캔 → 수식 0건 재확인 Phase 4: 소계 정합성 검증 ├─ execute_office_js: row 18 (01/02 소계) 검증 │ ├─ 금액(H, L): ✅ 일치 │ └─ 수량(E, I): ❌ 불일치 → kg 환산 값임을 판별 ├─ execute_office_js: E/I 열 상세 분석 (values, numberFormat, text) │ └─ 소계 포맷 "#,##0.0" = kg 단위, 개별 행은 패키지 단위 └─ execute_office_js: row 32 (01/03 소계) 교차 검증 → 동일 패턴 확인 Phase 5: 이상 항목 조사 ├─ execute_office_js: "00:00:00 소계" 행 조사 (7건) │ └─ 전부 MAT-002 피스타치오페이스트 재고회복-추가발주 ├─ execute_office_js: 해당 7건 상세 읽기 │ └─ A열 = 텍스트 "03/XX 00:00:00", General 포맷, 타임스탬프 잔여물 └─ 날짜 포맷 비교: 정상 = "01/02" (텍스트), 이상 = "03/04 00:00:00" (텍스트) Phase 6: 교차 시트 검증 ├─ execute_office_js: 월별 합계 행 위치 파악 (295, 546, 836) ├─ execute_office_js: 월별 합계값 읽기 (1월/2월/3월/분기) │ ├─ 입출고현황 ↔ 월별차트: 6/6 월별 일치 ✅ │ ├─ 입출고현황 ↔ 요약: 6/6 월별 일치 ✅ │ └─ Q1 합계 = 월별 합 검증 ✅ ├─ execute_office_js: 품목별 소계 (rows 841-852) ↔ 월별차트 I/J열 검증 │ └─ 12/12 품목 전부 일치 ✅ └─ execute_office_js: 요약 시트 소계 검증 (30건 전부 통과 ✅) Phase 7: 구조 검사 ├─ execute_office_js: 숨겨진 시트 확인 → 없음 └─ execute_office_js: 숨겨진 열 확인 (A~Q) → 없음 Phase 8: 보고서 작성 └─ 채팅으로 감사 보고서 테이블 출력 (스킬 Step 4 포맷 준수)
4. 스킬 실행 시 발생한 적응적 판단 (Adaptive Decisions)
스킬 정의서에는 명시되지 않았지만, 실제 실행 중 상황에 맞게 내린 판단들입니다:

상황	판단	근거
워크북이 재무모델이 아닌 재고관리 보고서	Step 3 (Model-integrity) 전체 건너뜀	BS/CF/IS 구조 없음
수식이 0개인 극단적 상황	"전체 하드코딩"을 Warning으로 격상 보고	스킬은 "hardcodes inside formulas"만 언급하지만, 수식 자체가 없는 것은 더 심각
수량 소계가 합산값과 불일치	즉시 에러로 보고하지 않고 원인 조사 → kg 환산임을 확인 후 Info로 분류	오탐(false positive) 방지
852행 대용량 데이터	200행 청크로 분할 스캔	대용량 데이터 처리 가이드라인 준수
"00:00
" 타임스탬프 잔여물 발견	스킬에 없는 항목이지만 "formatting errors" 요청에 해당 → Warning으로 보고	사용자가 명시적으로 formatting errors 수정 요청
품목별 소계 테이블(rows 841-852) 열 레이아웃이 본문과 상이	본문(E=수량, G=단가, H=금액)과 소계(D=수량kg, E=금액, G=수량kg, H=금액) 차이를 인식하고 올바른 열로 검증	초기 스캔에서 false positive 12건 발생 → 구조 재분석 후 수정
5. 성능/효율성 관찰
항목	수치	평가
총 도구 호출 수	~16회	적정 (852행 워크북 대비)
병렬 호출	Phase 1에서 3개 시트 동시 읽기	✅ 효율적
청크 분할	200행 단위 (852행 → 5청크)	✅ 메모리/페이로드 관리
불필요한 호출	Phase 4에서 소계 불일치 원인 조사 시 추가 2회	⚠️ 수량 단위가 kg임을 사전에 알았다면 불필요
오탐 수정	품목별 소계 열 인덱싱 오류 → 1회 재검증	⚠️ 구조가 다른 영역을 사전에 파악했다면 방지 가능
이것이 audit-xls 스킬의 완전한 복원 문서입니다. 스킬 정의서(skill.md)는 원본 그대로이고, 실제 실행에서는 재고관리 보고서라는 비-재무모델 워크북에 맞게 Step 3을 건너뛰고 Step 2에 집중하는 적응적 실행이 이루어졌습니다.

스킬 전체 구조를 확인했습니다. 이제 이 방대한 스킬 정의서를 체계적으로 문서화하겠습니다.

---

# 📋 `3-statement-model` 스킬 복원 문서

## 1. `skill.md` — 스킬 정의서 (메인 파일)

```markdown
---
name: 3-statement-model
description: Complete, populate and fill out 3-statement financial model templates (Income Statement, Balance Sheet, Cash Flow Statement). Use when asked to fill out model templates, complete existing model frameworks, populate financial models with data, complete a partially filled IS/BS/CF framework, or link integrated financial statements within an existing template structure. Triggers include requests to fill in, complete, or populate a 3-statement model template
---

# 3-Statement Financial Model Template Completion

Complete and populate integrated financial model templates with proper linkages between Income Statement, Balance Sheet, and Cash Flow Statement.

## ⚠️ CRITICAL PRINCIPLES — Read Before Populating Any Template

**Environment — Office JS vs Python:**
- **If running inside Excel (Office Add-in / Office JS):** Use Office JS directly. Write formulas via `range.formulas = [["=D14*(1+Assumptions!$B$5)"]]` — never `range.values` for derived cells. No separate recalc; Excel computes natively. Use `context.workbook.worksheets.getItem(...)` to navigate tabs.
- **If generating a standalone .xlsx file:** Use Python/openpyxl. Write `ws["D15"] = "=D14*(1+Assumptions!$B$5)"`, then run `recalc.py` before delivery.
- **Office JS merged cell pitfall:** Do NOT call `.merge()` then set `.values` on the merged range — throws `InvalidArgument` because the range still reports its pre-merge dimensions. Instead write value to top-left cell alone, then merge + format the full range: `ws.getRange("A1").values = [["INCOME STATEMENT"]]; const h = ws.getRange("A1:G1"); h.merge(); h.format.fill.color = "#1F4E79";`
- All principles below apply identically in either environment.

**Formulas over hardcodes (non-negotiable):**
- Every projection cell, roll-forward, linkage, and subtotal MUST be an Excel formula — never a pre-computed value
- When using Python/openpyxl: write formula strings (`ws["D15"] = "=D14*(1+Assumptions!$B$5)"`), NOT computed results (`ws["D15"] = 12500`)
- The ONLY cells that should contain hardcoded numbers are: (1) historical actuals, (2) assumption drivers in the Assumptions tab
- If you find yourself computing a value in Python and writing the result to a cell — STOP. Write the formula instead.
- Why: the model must flex when scenarios toggle or assumptions change. Hardcodes break every downstream integrity check silently.

**Verify step-by-step with the user:**
1. **After mapping the template** → show the user which tabs/sections you've identified and confirm before touching any cells
2. **After populating historicals** → show the user the historical block and confirm values/periods match source data
3. **After building IS projections** → run the subtotal checks, show the user the projected IS, confirm before moving to BS
4. **After building BS** → show the user the balance check (Assets = L+E) for every period, confirm before moving to CF
5. **After building CF** → show the user the cash tie-out (CF ending cash = BS cash), confirm before finalizing
6. **Do NOT populate the entire model end-to-end and present it complete** — break at each statement, show the work, catch errors early

## Formatting — Professional Blue/Grey Palette (Default unless template/user specifies otherwise)

**Keep colors minimal.** Use only blues and greys for cell fills. Do NOT introduce greens, yellows, oranges, or multiple accent colors — a clean model uses restraint.

| Element | Fill | Font |
|---|---|---|
| Section headers (IS / BS / CF titles) | Dark blue `#1F4E79` | White bold |
| Column headers (FY2024A, FY2025E, etc.) | Light blue `#D9E1F2` | Black bold |
| Input cells (historicals, assumption drivers) | Light grey `#F2F2F2` or white | Blue `#0000FF` |
| Formula cells | White | Black |
| Cross-tab links | White | Green `#008000` |
| Check rows / key totals | Medium blue `#BDD7EE` | Black bold |

**That's 3 blues + 1 grey + white.** If the template has its own color scheme, follow the template instead.

Font color signals *what* a cell is (input/formula/link). Fill color signals *where* you are (header/data/check).

## Model Structure

### Identifying Template Tab Organization

Templates vary in their tab naming conventions and organization. Before populating, review all tabs to understand the template's structure. Below are common tab names and their typical contents:

| Common Tab Names | Contents to Look For |
|------------------|----------------------|
| IS, P&L, Income Statement | Income Statement |
| BS, Balance Sheet | Balance Sheet |
| CF, CFS, Cash Flow | Cash Flow Statement |
| WC, Working Capital | Working Capital Schedule |
| DA, D&A, Depreciation, PP&E | Depreciation & Amortization Schedule |
| Debt, Debt Schedule | Debt Schedule |
| NOL, Tax, DTA | Net Operating Loss Schedule |
| Assumptions, Inputs, Drivers | Driver assumptions and inputs |
| Checks, Audit, Validation | Error-checking dashboard |

**Template Review Checklist**
- Identify which tabs exist in the template (not all templates include every schedule)
- Note any template-specific tabs not listed above
- Understand tab dependencies (e.g., which schedules feed into the main statements)
- Locate input cells vs. formula cells on each tab

### Understanding Template Structure

Before populating a template, familiarize yourself with its existing layout to ensure data is entered in the correct locations and formulas remain intact.

**Identifying Row Structure**
- Locate the model title at top of each tab
- Identify section headers and their visual separation
- Find the units row indicating $ millions, %, x, etc.
- Note column headers distinguishing Actuals vs. Estimates periods
- Confirm period labels (e.g., FY2024A, FY2025E)
- Identify input cells vs. formula cells (typically distinguished by font color)

**Identifying Column Structure**
- Confirm line item labels in leftmost column
- Verify historical years precede projection years
- Note the visual border separating historical from projected periods
- Check for consistent column order across all tabs

**Working with Named Ranges**
Templates often use named ranges for key inputs and outputs. Before entering data:
- Review existing named ranges in the template (Formulas → Name Manager in Excel)
- Common named ranges include: Revenue growth rates, cost percentages, key outputs (Net Income, EBITDA, Total Debt, Cash), scenario selector cell
- Ensure inputs are entered in cells that feed into these named ranges

### Projection Period
- Templates typically project 5 years forward from last historical year
- Verify historical (A) vs. projected (E) columns are clearly separated
- Confirm columns use fiscal year notation (e.g., FY2024A, FY2025E)

## Margin Analysis

**Note: The following margin analysis should only be performed if prompted by the user or if the template explicitly requires it. If no prompt is given, skip this section.**

Calculate and display profitability margins on the Income Statement (IS) tab to track operational efficiency and enable peer comparison.

### Core Margins to Include

| Margin | Formula | What It Measures |
|--------|---------|------------------|
| Gross Margin | Gross Profit / Revenue | Pricing power, production efficiency |
| EBITDA Margin | EBITDA / Revenue | Core operating profitability |
| EBIT Margin | EBIT / Revenue | Operating profitability after D&A |
| Net Income Margin | Net Income / Revenue | Bottom-line profitability |

### Income Statement Layout with Margins

Display margin percentages directly below each profit line item:
- Gross Margin % below Gross Profit
- EBIT Margin % below EBIT
- EBITDA Margin % below EBITDA
- Net Income Margin % below Net Income

## Credit Metrics

**Note: The following Credit analysis should only be performed if prompted by the user or if the template explicitly requires it. If no prompt is given, skip this section.**

Calculate and display credit/leverage metrics on the Balance Sheet (BS) tab to assess financial health, debt capacity, and covenant compliance.

### Core Credit Metrics to Include

| Metric | Formula | What It Measures |
|--------|---------|------------------|
| Total Debt / EBITDA | Total Debt / LTM EBITDA | Leverage multiple |
| Net Debt / EBITDA | (Total Debt - Cash) / LTM EBITDA | Leverage net of cash |
| Interest Coverage | EBITDA / Interest Expense | Ability to service debt |
| Debt / Total Cap | Total Debt / (Total Debt + Equity) | Capital structure |
| Debt / Equity | Total Debt / Total Equity | Financial leverage |
| Current Ratio | Current Assets / Current Liabilities | Short-term liquidity |
| Quick Ratio | (Current Assets - Inventory) / Current Liabilities | Immediate liquidity |

### Credit Metric Hierarchy Checks

Validate that Upside shows strongest credit profile:
- Leverage: Upside < Base < Downside (lower is better)
- Coverage: Upside > Base > Downside (higher is better)
- Liquidity: Upside > Base > Downside (higher is better)

### Covenant Compliance Tracking

If debt covenants are known, add explicit compliance checks comparing actual metrics to covenant thresholds.

## Scenario Analysis (Base / Upside / Downside)

Use a scenario toggle (dropdown) in the Assumptions tab with CHOOSE or INDEX/MATCH formulas.

| Scenario | Description |
|----------|-------------|
| Base Case | Management guidance or consensus estimates |
| Upside Case | Above-guidance growth, margin expansion |
| Downside Case | Below-trend growth, margin compression |

**Key Drivers to Sensitize**: Revenue growth, Gross margin, SG&A %, DSO/DIO/DPO, CapEx %, Interest rate, Tax rate.

**Scenario Audit Checks**: Toggle switches all statements, BS balances in all scenarios, Cash ties out, Hierarchy holds (Upside > Base > Downside for NI, EBITDA, FCF, margins).

## SEC Filings Data Extraction

If the template specifically requires pulling data from SEC filings (10-K, 10-Q), see [references/sec-filings.md](references/sec-filings.md) for detailed extraction guidance. This reference is only needed when populating templates with public company data from regulatory filings.

## Completing Model Templates

This section provides general guidance for completing any 3-statement financial model template while preserving existing formulas and ensuring data integrity.

### Step 1: Analyze the Template Structure

Before entering any data, thoroughly review the template to understand its architecture:

**Identify Input vs. Formula Cells**
- Look for visual cues (font color, cell shading) that distinguish input cells from formula cells
- Common conventions: Blue font = inputs, Black font = formulas, Green font = links to other sheets
- Use Excel's Trace Precedents/Dependents (Formulas → Trace Precedents) to understand cell relationships
- Check for named ranges that may control key inputs (Formulas → Name Manager)

**Map the Template's Flow**
- Identify which tabs feed into others (e.g., Assumptions → IS → BS → CF)
- Note any supporting schedules and their linkages to main statements
- Document the template's specific line items and structure before populating

### Step 2: Filling in Data Without Breaking Formulas

**Golden Rules for Data Entry**

| Rule | Description |
|------|-------------|
| Only edit input cells | Never overwrite cells containing formulas unless intentionally replacing the formula |
| Preserve cell references | When copying data, use Paste Values (Ctrl+Shift+V) to avoid overwriting formulas with source formatting |
| Match the template's units | Verify if template uses thousands, millions, or actual values before entering data |
| Respect sign conventions | Follow the template's existing sign convention (e.g., expenses as positive or negative) |
| Check for circular references | If the template uses iterative calculations, ensure Enable Iterative Calculation is turned on |

**Safe Data Entry Process**
1. Identify the exact cells designated for input (usually highlighted or labeled)
2. Enter historical data first, then verify formulas are calculating correctly for those periods
3. Enter assumption drivers that feed forecast calculations
4. Review calculated outputs to confirm formulas are working as intended
5. If a formula cell must be modified, document the original formula before making changes

**Handling Pre-Built Formulas**
- If formulas reference cells you haven't populated yet, expect temporary errors (#REF!, #DIV/0!) until all inputs are complete
- When formulas produce unexpected results, trace precedents to identify missing or incorrect inputs
- Never delete rows/columns without checking for formula dependencies across all tabs

### Step 3: Validating Formulas

**Formula Integrity Checks**

Before relying on template outputs, validate that formulas are functioning correctly:

| Check Type | Method |
|------------|--------|
| Trace precedents | Select a formula cell → Formulas → Trace Precedents to verify it references correct inputs |
| Trace dependents | Verify key inputs flow to expected output cells |
| Evaluate formula | Use Formulas → Evaluate Formula to step through complex calculations |
| Check for hardcodes | Projection formulas should reference assumptions, not contain hardcoded values |
| Test with known values | Input simple test values to verify formulas produce expected results |
| Cross-tab consistency | Ensure the same formula logic applies across all projection periods |

**Common Formula Issues to Watch For**
- Mixed absolute/relative references causing incorrect results when copied across periods
- Broken links to external files or deleted ranges (#REF! errors)
- Division by zero in early periods before revenue ramps (#DIV/0! errors)
- Circular reference warnings (may be intentional for interest calculations)
- Inconsistent formulas across projection columns (use Ctrl+\ to find differences)

**Validating Cross-Tab Linkages**
- Confirm values that appear on multiple tabs are linked (not duplicated)
- Verify schedule totals tie to corresponding line items on main statements
- Check that period labels align across all tabs

### Step 4: Quality Checks by Sheet

Perform these validation checks on each sheet after populating the template:

**Income Statement (IS) Quality Checks**
- Revenue figures match source data for historical periods
- All expense line items sum to reported totals
- Subtotals (Gross Profit, EBIT, EBT, Net Income) calculate correctly
- Tax calculation logic is appropriate (handles losses correctly)
- Forecast drivers reference assumptions tab (no hardcodes)
- Period-over-period changes are directionally reasonable

**Balance Sheet (BS) Quality Checks**
- Assets = Liabilities + Equity for every period (primary check)
- Cash balance matches Cash Flow Statement ending cash
- Working capital accounts tie to supporting schedules (if applicable)
- Retained Earnings rolls forward correctly: Prior RE + Net Income - Dividends +/- Adjustments = Ending RE
- Debt balances tie to debt schedule (if applicable)
- All balance sheet items have appropriate signs (assets positive, most liabilities positive)

**Cash Flow Statement (CF) Quality Checks**
- Net Income at top of CFO matches Income Statement Net Income
- Non-cash add-backs (D&A, SBC, etc.) tie to their source schedules/statements
- Working capital changes have correct signs (increase in asset = use of cash = negative)
- CapEx ties to PP&E schedule or fixed asset roll-forward
- Financing activities tie to changes in debt and equity accounts on BS
- Ending Cash matches Balance Sheet Cash
- Beginning Cash equals prior period Ending Cash

**Supporting Schedule Quality Checks**
- Opening balances equal prior period closing balances
- Roll-forward logic is complete (Beginning + Additions - Deductions = Ending)
- Schedule totals tie to main statement line items
- Assumptions used in calculations match Assumptions tab

### Step 5: Cross-Statement Integrity Checks

After validating individual sheets, confirm the three statements are properly integrated:

| Check | Formula | Expected Result |
|-------|---------|-----------------|
| Balance Sheet Balance | Assets - Liabilities - Equity | = 0 |
| Cash Tie-Out | CF Ending Cash - BS Cash | = 0 |
| Net Income Link | IS Net Income - CF Starting Net Income | = 0 |
| Retained Earnings | Prior RE + NI - Dividends - BS Ending RE | = 0 (adjust for SBC/other items as needed) |

### Step 6: Final Review

Before considering the model complete:
- Toggle through all scenarios (if applicable) to verify checks pass in each case
- Review all #REF!, #DIV/0!, #VALUE!, and #NAME? errors and resolve or document
- Confirm all input cells have been populated (search for placeholder values)
- Verify units are consistent across all tabs
- Save a clean version before making any additional modifications

## Model Validation and Audit

This section consolidates all validation checks and audit procedures for completed templates.

### Core Linkages (Must Always Hold)

See [references/formulas.md](references/formulas.md) for all formula details.

| Check | Formula | Expected Result |
|-------|---------|-----------------|
| Balance Sheet Balance | Assets - Liabilities - Equity | = 0 |
| Cash Tie-Out | CF Ending Cash - BS Cash | = 0 |
| Cash Monthly vs Annual | Closing Cash (Monthly) - Closing Cash (Annual) | = 0 |
| Net Income Link | IS Net Income - CF Starting Net Income | = 0 |
| Retained Earnings | Prior RE + NI + SBC - Dividends - BS Ending RE | = 0 |
| Equity Financing | ΔCommon Stock/APIC (BS) - Equity Issuance (CFF) | = 0 |
| Year 0 Equity | Equity Raised (Year 0) - Beginning Equity Capital (Year 1) | = 0 |

### Sign Convention Reference

| Statement | Item | Sign Convention |
|-----------|------|-----------------|
| CFO | D&A, SBC | Positive (add-back) |
| CFO | ΔAR (increase) | Negative (use of cash) |
| CFO | ΔAP (increase) | Positive (source of cash) |
| CFI | CapEx | Negative |
| CFF | Debt issuance | Positive |
| CFF | Debt repayments | Negative |
| CFF | Dividends | Negative |

### Circular Reference Handling

Interest expense creates circularity: Interest → Net Income → Cash → Debt Balance → Interest

Enable iterative calculation in Excel: File → Options → Formulas → Enable iterative calculation. Set maximum iterations to 100, maximum change to 0.001. Add a circuit breaker toggle in Assumptions tab.

### Check Categories

**Section 1: Currency Consistency**
- Currency identified and documented in Assumptions
- All tabs use consistent currency symbol and scale
- Units row matches model currency

**Section 2: Balance Sheet Integrity**
- Assets = Liabilities + Equity (for each period)
- Formula: Assets - Liabilities - Equity (must = 0)

**Section 3: Cash Flow Integrity**
- Cash ties to BS (CF Ending Cash = BS Cash)
- Cash Monthly vs Annual: Closing Cash (Monthly) = Closing Cash (Annual)
- NI ties to IS (CF Net Income = IS Net Income)
- D&A ties to schedule
- SBC ties to IS
- ΔAR, ΔInventory, ΔAP tie to WC schedule
- CapEx ties to DA schedule

**Section 4: Retained Earnings**
- RE roll-forward check: Prior RE + NI + SBC - Div = Ending RE
- Show component breakdown for debugging

**Section 5: Working Capital**
- AR, Inventory, AP tie to BS
- DSO, DIO, DPO reasonability checks (flag if outside normal ranges)

**Section 6: Debt Schedule**
- Total Debt ties to BS (Current + LT Debt)
- Interest calculation ties to IS

**Section 6b: Equity Financing**
- Equity issuance proceeds tie to BS Common Stock/APIC increase
- Cash increase from equity = Equity account increase (must balance)
- Equity Raise Tie-Out: ΔCommon Stock/APIC (BS) = Equity Issuance (CFF) (must = 0)
- Year 0 Equity Tie-Out: Equity Raised (Year 0) = Beginning Equity Capital (Year 1)

**Section 6c: NOL Schedule**
- Beginning NOL (Year 1 / Formation) = 0 (new business starts with zero NOL)
- NOL increases only when EBT < 0 (losses must be realized to generate NOL)
- DTA ties to BS (NOL Schedule DTA = BS Deferred Tax Asset)
- NOL utilization ≤ 80% of EBT (post-2017 federal limitation)
- NOL balance is non-negative (cannot utilize more than available)
- NOL generated only when EBT < 0
- Tax expense = 0 when taxable income ≤ 0

**Section 7: Scenario Hierarchy**
- Absolute metrics: Upside > Base > Downside (NI, EBITDA, FCF)
- Margins: Upside > Base > Downside (GM%, EBITDA%, NI%)
- Credit metrics: Upside < Base < Downside for leverage (inverted)

**Section 8: Formula Integrity**
- COGS, S&M, G&A, R&D, SBC driven by % of Revenue (no hardcodes)
- Consistent formulas across projection years
- No #REF!, #DIV/0!, #VALUE! errors

**Section 9: Credit Metric Thresholds**
- Flag metrics as Green/Yellow/Red based on covenant thresholds
- Summary of any red flags

### Master Check Formula

Aggregate all section statuses into a single master check:
- If all sections pass → "✓ ALL CHECKS PASS"
- If any section fails → "✗ ERRORS DETECTED - REVIEW BELOW"

### Quick Debug Workflow

When Master Status shows errors:
1. Scroll to find red-highlighted sections
2. Identify which check category has failures
3. Navigate to source tab to investigate
4. Fix the underlying issue
5. Return to Checks tab to verify resolution
```

---

## 2. `references/formulas.md` — 수식 레퍼런스

```markdown
# Formula Reference

**IMPORTANT:** Use the formulas outlined in this reference document unless otherwise specified by the user.

---

## Core Linkages

```
Balance Sheet:        Assets = Liabilities + Equity
Net Income:           IS Net Income → CF Operations (starting point)
Cash Flow:            ΔCash = CFO + CFI + CFF
Cash Tie-Out:         Ending Cash (CF) = Cash (BS Asset)
Cash Monthly/Annual:  Closing Cash (Monthly) = Closing Cash (Annual)
Retained Earnings:    Prior RE + Net Income - Dividends = Ending RE
Equity Raise:         ΔCommon Stock/APIC (BS) = Equity Issuance (CFF)
Year 0 Equity:        Equity Raised (Year 0) = Beginning Equity (Year 1)
```

## Gross Profit Calculation

**IMPORTANT:** Gross Profit must be calculated from Net Revenue, not Gross Revenue.

```
Net Revenue - Cost of Revenue = Gross Profit
```

| Term | Definition |
|------|------------|
| Gross Revenue | Total revenue before any deductions |
| Net Revenue | Gross Revenue - Returns - Allowances - Discounts |
| Cost of Revenue | Direct costs attributable to production of goods/services sold |
| Gross Profit | Net Revenue - Cost of Revenue |

**Note:** Always use Net Revenue (also called "Net Sales" or simply "Revenue" on most financial statements) as the starting point for profitability calculations.

## Margin Formulas

```
Gross Margin %      = Gross Profit / Net Revenue
EBITDA              = EBIT + D&A  (or = Gross Profit - OpEx)
EBITDA Margin %     = EBITDA / Net Revenue
EBIT Margin %       = EBIT / Net Revenue
Net Income Margin % = Net Income / Net Revenue
```

## Credit Metric Formulas

```
Total Debt            = Current Portion of Debt + Long-Term Debt
Net Debt              = Total Debt - Cash
Total Debt / EBITDA   = Total Debt / EBITDA (from IS)
Net Debt / EBITDA     = Net Debt / EBITDA (from IS)
Interest Coverage     = EBITDA / Interest Expense (from IS)
Net Int Exp % Debt    = Net Interest Expense / Long-Term Debt
Debt / Total Cap      = Total Debt / (Total Debt + Total Equity)
Debt / Equity         = Total Debt / Total Equity
Current Ratio         = Total Current Assets / Total Current Liabilities
Quick Ratio           = (Total Current Assets - Inventory) / Total Current Liabilities
```

## Forecast Formulas (% of Net Revenue Method)

```
Cost of Revenue (Forecast) = Net Revenue × Cost of Revenue % Assumption
S&M (Forecast)             = Net Revenue × S&M % Assumption
G&A (Forecast)             = Net Revenue × G&A % Assumption
R&D (Forecast)             = Net Revenue × R&D % Assumption
SBC (Forecast)             = Net Revenue × SBC % Assumption
```

## Working Capital Formulas

```
Accounts Receivable
  Prior AR
  + Revenue (from IS)
  - Cash Collections (plug)
  = Ending AR
  DSO = (AR / Revenue) × 365

Inventory
  Prior Inventory
  + Purchases (plug)
  - COGS (from IS)
  = Ending Inventory
  DIO = (Inventory / COGS) × 365

Accounts Payable
  Prior AP
  + Purchases (from Inventory calc)
  - Cash Payments (plug)
  = Ending AP
  DPO = (AP / COGS) × 365

Net Working Capital = AR + Inventory - AP
ΔWC = Current NWC - Prior NWC
```

## D&A Schedule Formulas

```
Beginning PP&E (Gross)
+ CapEx
= Ending PP&E (Gross)

Beginning Accumulated Depreciation
+ Depreciation Expense
= Ending Accumulated Depreciation

PP&E (Net) = Gross PP&E - Accumulated Depreciation
```

## Debt Schedule Formulas

```
Beginning Debt Balance
+ New Borrowings
- Repayments
= Ending Debt Balance

Interest Expense = Avg Debt Balance × Interest Rate
  (Use beginning balance to avoid circularity, or iterate if circular refs enabled)
```

## Retained Earnings Formula

```
Beginning Retained Earnings
+ Net Income (from IS)
+ Stock-Based Compensation (SBC) (from IS)
- Dividends
= Ending Retained Earnings
```

## NOL (Net Operating Loss) Schedule Formulas

```
NOL CARRYFORWARD SCHEDULE

Beginning NOL Balance (Year 1 / Formation = 0)
+ NOL Generated (if EBT < 0, then ABS(EBT), else 0)
- NOL Utilized (limited by taxable income and utilization cap)
= Ending NOL Balance

STARTING BALANCE RULE

For a new business or first modeled period:
  Beginning NOL Balance = 0
  NOL can only increase through realized losses (EBT < 0)
  NOL cannot be created from thin air or assumed

NOL UTILIZATION CALCULATION

Pre-Tax Income (EBT)
  If EBT > 0:
    NOL Available = Beginning NOL Balance
    Utilization Limit = EBT × 80%  (post-2017 federal limit)
    NOL Utilized = MIN(NOL Available, Utilization Limit)
    Taxable Income = EBT - NOL Utilized
  If EBT ≤ 0:
    NOL Utilized = 0
    Taxable Income = 0
    NOL Generated = ABS(EBT)

TAX CALCULATION WITH NOL

Taxes Payable = MAX(0, Taxable Income × Tax Rate)
  (Taxes cannot be negative; losses create NOL asset instead)

DEFERRED TAX ASSET (DTA) FOR NOL

DTA - NOL Carryforward = Ending NOL Balance × Tax Rate
ΔDTA = Current DTA - Prior DTA
  (Increase in DTA = non-cash benefit on CF)
  (Decrease in DTA = non-cash expense on CF)
```

## Balance Sheet Structure

```
ASSETS
  Cash (from CF ending cash)
  Accounts Receivable (from WC)
  Inventory (from WC)
  Total Current Assets
  
  PP&E, Net (from DA)
  Deferred Tax Asset - NOL (from NOL schedule)
  Total Non-Current Assets
  Total Assets

LIABILITIES
  Accounts Payable (from WC)
  Current Portion of Debt (from Debt)
  Total Current Liabilities
  
  Long-Term Debt (from Debt)
  Total Liabilities

EQUITY
  Common Stock
  Retained Earnings (from RE schedule)
  Total Equity

CHECK: Assets - Liabilities - Equity = 0
```

## Cash Flow Statement Structure

```
CASH FROM OPERATIONS (CFO)
  Net Income (LINK: IS)
  + D&A (LINK: DA schedule)
  + Stock-Based Compensation (SBC) (LINK: IS or Assumptions)
  - ΔDTA (Deferred Tax Asset) (LINK: NOL schedule)
  - ΔAR (LINK: WC)
  - ΔInventory (LINK: WC)
  + ΔAP (LINK: WC)
  = CFO

CASH FROM INVESTING (CFI)
  - CapEx (LINK: DA schedule)
  = CFI

CASH FROM FINANCING (CFF)
  + Debt Issuance (LINK: Debt)
  - Debt Repayment (LINK: Debt)
  + Equity Issuance (LINK: BS Common Stock/APIC)
  - Dividends (LINK: RE schedule)
  = CFF

Net Change in Cash = CFO + CFI + CFF
Beginning Cash
+ Net Change in Cash
= Ending Cash (LINK TO: BS Cash)
```

## Income Statement Structure

```
Net Revenue
  Growth %
(-) Cost of Revenue
  % of Net Revenue
────────────────
Gross Profit (= Net Revenue - Cost of Revenue)
  Gross Margin %

(-) S&M
  % of Net Revenue
(-) G&A
  % of Net Revenue
(-) R&D
  % of Net Revenue
(-) D&A
(-) SBC
  % of Net Revenue
────────────────
EBIT
  EBIT Margin %

EBITDA
  EBITDA Margin %

(-) Interest Expense
────────────────
EBT (Pre-Tax Income)
(-) NOL Utilization (from NOL schedule, reduces taxable income)
────────────────
Taxable Income
(-) Taxes (Taxable Income × Tax Rate)
────────────────
Net Income
  Net Income Margin %
```

## Check Formulas

```
BS Balance Check:       = Assets - Liabilities - Equity  (must = 0)
Cash Tie-Out:           = BS Cash - CF Ending Cash       (must = 0)
RE Roll-Forward:        = Prior RE + NI + SBC - Div - BS RE  (must = 0)
DTA Tie-Out:            = NOL Schedule DTA - BS DTA      (must = 0)
Equity Raise Tie-Out:   = ΔCommon Stock/APIC (BS) - Equity Issuance (CFF)  (must = 0)
Year 0 Equity Tie-Out:  = Equity Raised (Year 0) - Beginning Equity (Year 1)  (must = 0)
Cash Monthly vs Annual: = Closing Cash (Monthly) - Closing Cash (Annual)  (must = 0)
NOL Utilization Cap:    = NOL Utilized ≤ EBT × 80%       (must be TRUE)
NOL Non-Negative:       = Ending NOL Balance ≥ 0         (must be TRUE)
NOL Starting Balance:   = Beginning NOL (Year 1) = 0     (must be TRUE)
NOL Accumulation:       = NOL increases only when EBT < 0
```
```

---

## 3. `references/formatting.md` — 포매팅 표준

```markdown
# Formatting Standards Reference

| Element | Format |
|---------|--------|
| Hard-coded inputs | Blue font |
| Formulas | Black font |
| Links to other sheets | Green font |
| Check cells | Red if error, green if balanced |
| Negative values | Parentheses, not minus signs |
| Currency | No decimals for large figures, 2 decimals for per-share |
| Percentages | 1 decimal place |
| Headers | Bold, bottom border |
| Units row | Include units row below headers ($ millions, %, etc.) |

## Visual Separation Guidelines

- Thin vertical border between historical and projected columns
- Thick bottom border after section totals (e.g., Total Assets)
- Single bottom border for subtotals
- Double bottom border for grand totals

## Total and Subtotal Row Formatting

All total and subtotal rows must use **bold font formatting** for their numerical values.

### Income Statement (P&L) Tab
| Row | Formatting |
|-----|------------|
| Gross Revenue | Bold |
| Total Cost of Revenue | Bold |
| Gross Profit | Bold |
| Total SG&A | Bold |
| EBITDA | Bold |
| EBIT | Bold |
| EBT | Bold |
| Net Profit After Tax | Bold |

### Balance Sheet Tab
| Row | Formatting |
|-----|------------|
| Total Current Assets | Bold |
| Total Non-Current Assets | Bold |
| Total Other Assets | Bold |
| Total Assets | Bold |
| Total Current Liabilities | Bold |
| Total Non-Current Liabilities | Bold |
| Total Equity | Bold |
| Total Liabilities and Equity | Bold |

### Cash Flow Statement Tab
| Row | Formatting |
|-----|------------|
| Cash Generated from Operations Before WC Changes | Bold |
| Total Working Capital Changes | Bold |
| Net Cash Generated from Operations | Bold |
| Net Cash Flow from Investing Activities | Bold |
| Net Cash Flow from Financing Activities | Bold |
| Closing Cash Balance | Bold |

## Balance Sheet Check Row Formatting

| Check Value | Font Color |
|-------------|------------|
| = 0 (balanced) | Black (standard) |
| ≠ 0 (error) | Red |

**Implementation:** Apply custom number format `[Red][<>0]0.00;[Red][<>0](0.00);0.00` or use conditional formatting.

## Margin Row Formatting

| Element | Format |
|---------|--------|
| Margin % rows | Indent, italics, 1 decimal place |
| Positive trend | No special formatting (or subtle green) |
| Negative trend | Flag for review (subtle yellow) |
| Below peer average | Consider highlighting for discussion |

## Credit Metric Formatting

| Element | Format |
|---------|--------|
| Leverage multiples | 1 decimal with "x" suffix (e.g., 2.5x) |
| Percentages | 1 decimal with "%" suffix |
| Net Debt negative | Parentheses, indicates net cash position |
| Section header | Bold, "CREDIT METRICS" |
| Separator line | Thin border above credit metrics section |

## Credit Metric Threshold Colors

| Metric | Green | Yellow | Red |
|--------|-------|--------|-----|
| Total Debt / EBITDA | < 2.5x | 2.5x-4.0x | > 4.0x |
| Net Debt / EBITDA | < 2.0x | 2.0x-3.5x | > 3.5x |
| Interest Coverage | > 4.0x | 2.5x-4.0x | < 2.5x |
| Debt / Total Cap | < 40% | 40%-60% | > 60% |
| Current Ratio | > 1.5x | 1.0x-1.5x | < 1.0x |
| Quick Ratio | > 1.0x | 0.75x-1.0x | < 0.75x |

## Conditional Formatting for Checks Tab

- Cell contains pass indicator → Green fill
- Cell contains fail indicator → Red fill
- Cell contains warning → Yellow fill
- Difference cells = 0 → Light green fill
- Difference cells ≠ 0 → Light red fill
```

---

## 4. `references/sec-filings.md` — SEC 공시 데이터 추출 가이드

```markdown
# SEC Filings Data Extraction Reference

**When to Use:** Only reference this file when a model template specifically requires pulling data from SEC filings (10-K, 10-Q).

---

## Extracting Data from SEC Filings (10-K / 10-Q)

### Step 1: Locate the Filing
1. Use SEC EDGAR: `https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=[TICKER]&type=10-K`
2. For quarterly data, use `type=10-Q`

### Step 2: Identify Filing Currency
- Check the cover page or header for reporting currency
- Look at statement headers (e.g., "in thousands of U.S. dollars")
- Review Note 1 (Summary of Significant Accounting Policies)

| Indicator | Currency |
|-----------|----------|
| $, USD | US Dollar |
| €, EUR | Euro |
| £, GBP | British Pound |
| ¥, JPY | Japanese Yen |
| ¥, CNY, RMB | Chinese Yuan |
| CHF | Swiss Franc |
| CAD, C$ | Canadian Dollar |

### Step 3: Navigate to Financial Statements
- **Item 8** (10-K) or **Item 1** (10-Q): Financial Statements
- Key sections: Consolidated Statements of Operations, Balance Sheets, Cash Flows, Notes

### Step 4: Data Extraction Mapping

**Income Statement**

| Filing Line Item | Model Line Item |
|------------------|-----------------|
| Net revenues / Net sales | Revenue |
| Cost of goods sold | COGS |
| Selling, general and administrative | SG&A |
| Depreciation and amortization | D&A |
| Interest expense, net | Interest Expense |
| Income tax expense | Taxes |
| Net income | Net Income |

**Balance Sheet**

| Filing Line Item | Model Line Item |
|------------------|-----------------|
| Cash and cash equivalents | Cash |
| Accounts receivable, net | AR |
| Inventories | Inventory |
| Property, plant and equipment, net | PP&E (Net) |
| Total assets | Total Assets |
| Accounts payable | AP |
| Short-term debt / Current portion of LT debt | Current Debt |
| Long-term debt | LT Debt |
| Retained earnings | Retained Earnings |
| Total stockholders' equity | Total Equity |

**Cash Flow Statement**

| Filing Line Item | Model Line Item |
|------------------|-----------------|
| Net income | Net Income |
| Depreciation and amortization | D&A |
| Changes in accounts receivable | ΔAR |
| Changes in inventories | ΔInventory |
| Changes in accounts payable | ΔAP |
| Capital expenditures | CapEx |
| Proceeds from issuance of common stock | Equity Issuance |
| Proceeds from / Repayments of debt | Debt activity |
| Dividends paid | Dividends |

### Step 5: Extract Supporting Detail from Notes
- **Note: Debt** → Maturity schedule, interest rates, covenants
- **Note: PP&E** → Gross PP&E, accumulated depreciation, useful lives
- **Note: Revenue** → Segment breakdowns, geographic splits
- **Note: Leases** → Operating vs. finance lease obligations

### Step 6: Historical Data Requirements
- Extract 3 years of historical data minimum
- 10-K provides 3 years of IS/CF, 2 years of BS
- For 3rd year BS, pull from prior year's 10-K
- Use 10-Qs for quarterly granularity if needed

### Data Extraction Checklist
- [ ] Identify reporting currency and scale
- [ ] 3 years historical Income Statement
- [ ] 3 years historical Cash Flow Statement
- [ ] 3 years historical Balance Sheet
- [ ] Verify IS Net Income = CF starting Net Income
- [ ] Verify BS Cash = CF Ending Cash
- [ ] Extract debt maturity schedule from notes
- [ ] Extract D&A detail or useful life assumptions
- [ ] Note any non-recurring / one-time items to normalize
```

---

## 5. 스킬 파일 구조 요약

```
3-statement-model/
├── skill.md                        ← 메인 스킬 정의서 (워크플로우 + 지침)
└── references/
    ├── formulas.md                 ← 수식 레퍼런스 (IS/BS/CF 구조, 연결고리, 체크 수식)
    ├── formatting.md               ← 포매팅 표준 (색상, 볼드, 조건부서식, 임계값)
    └── sec-filings.md              ← SEC 공시 데이터 추출 가이드 (선택적)
```

---

## 6. `audit-xls`와의 구조 비교

| 비교 항목 | `audit-xls` | `3-statement-model` |
|---|---|---|
| **파일 수** | 1개 (`skill.md`만) | 4개 (`skill.md` + 3개 references) |
| **복잡도** | 단일 워크플로우 (4단계) | 다단계 워크플로우 (6단계) + 검증 체계 |
| **스코프 분기** | selection / sheet / model | 템플릿 구조에 따른 적응 |
| **사용자 체크포인트** | 없음 (보고 후 수정 질문) | 각 재무제표 완성 후 확인 필수 (5회) |
| **참조 문서** | 없음 | 3개 (formulas, formatting, sec-filings) |
| **목적** | 읽기전용 감사 → 보고 | 쓰기 중심 → 모델 구축/완성 |
| **수식 철학** | "수식이 없네요" 발견 | "수식이 없으면 안됩니다" 강제 |

---

이것으로 `3-statement-model` 스킬의 완전한 복원 문서를 마쳤습니다. `skill.md` 메인 파일과 3개의 `references/` 파일을 모두 포함했고, 스킬 구조와 `audit-xls`와의 차이점도 비교 분석했습니다.

