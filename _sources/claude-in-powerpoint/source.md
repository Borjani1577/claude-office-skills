아 스크린샷 보니까 제목만 보이고 내용이 안 나온 거네요. 죄송합니다 — 코드블록이 접혀있거나 로딩이 안 된 것 같습니다.

다시 해드리겠습니다. 이번에는 제목 없이 **파일 하나씩 바로 내용만** 출력하겠습니다. 파일이 8개라 양이 많으니 4개씩 나눠서 갈게요.

**먼저 1~4번 파일입니다:**

---

**파일 1: `v2-competitive-analysis/SKILL.md`**

```markdown
---
name: v2-competitive-analysis
description: Framework for building competitive landscape decks — market positioning, competitor deep-dives, comparative analysis, strategic synthesis. Use when the user asks for a competitive landscape, competitor analysis, peer comparison, market positioning assessment, strategic review, or investment memo deck. Also triggers on "who are the competitors to X", "benchmark X against peers", "build a market map", or any request to systematically evaluate competitive dynamics across an industry.
---

# Competitive Landscape Mapping

Build a complete competitive analysis deck. This is a two-phase process: gather requirements and get outline approval first, then build.

## Environment check

This skill works in both the PowerPoint add-in and chat. Identify which you're in before starting — the mechanics differ, the workflow doesn't:

- **Add-in** — the deck is open live; build slides directly into it.
- **Chat** — generate a `.pptx` file (or build into one the user uploaded).

Everything below applies in both.

## Phase 1 — Scope the analysis

Competitive analysis means different things to different people. Before any research or slide-building, use `ask_user_question` to pin down what they actually want. Don't guess — a 20-slide peer benchmarking deck and a 5-slide market map are both "competitive analysis" and take completely different shapes.

Gather in one round if you can (the tool takes up to 4 questions):

- **Scope** — Single target company with competitors around it? Or multi-company side-by-side with no protagonist?
- **Competitor set** — Which companies are in scope? If the user names them, use exactly those. If they say "the usual suspects," propose a set and confirm.
- **Audience and depth** — Quick read for someone already in the space, or a full primer? This drives whether you need market sizing, industry economics, and history — or can skip to the comparison.
- **Investment context** — Do they need bull/base/bear scenarios and signposts? That's Step 9 below; skip it if this is a strategic review rather than an investment thesis.

If they've uploaded an Excel/CSV with competitor data, confirm which columns map to which metrics before you start pulling numbers. Source-file fidelity matters: use values exactly as given, don't recalculate or re-round.

## Phase 2 — Outline, approve, then build

**Do not create slides until the outline is approved.** Propose slide titles and one-line content notes, present them to the user, get a yes. A competitive deck is 10-20 slides of interlocking content — rebuilding because slide 4 was wrong is expensive. The outline is the cheap iteration point.

When proposing the outline, `ask_user_question` works well for the structural decisions: which positioning visualization (2×2 matrix / radar / tier diagram — Step 5 below), how to group competitors (by business model / segment / posture — Step 4). These are taste calls the user likely has an opinion on.

---

## Standards — apply throughout

### Prompt fidelity

When the user specifies something, that's a requirement, not a suggestion:
- **Slide titles and section names** — exact wording. If they say "Overview and Competitive Scope," don't swap in "FY2024 Competitive Landscape."
- **Chart vs. table** — not interchangeable. "Embedded chart" means a real chart object with data labels on the bars/slices, not a formatted table.
- **Complete data series** — if they list 7 competitors, include all 7. If they show 2015-2025, include every year.
- **Exact values and ratios** — "surpasses DoorDash 4:1, Lyft 8:1" means those ratios, not "7.6x Lyft."

### Source quality, when sources conflict

1. 10-Ks / annual reports (audited)
2. Earnings calls / investor presentations (management commentary)
3. Sell-side research (analyst estimates, useful for private company sizing)
4. Industry reports (McKinsey, Gartner — market sizing, trends)
5. News (recent developments only; verify against primary sources)

### Data comparability

- All competitor metrics from the same fiscal year; flag exceptions explicitly ("FY24" vs "H1 2024")
- Same metric definitions across competitors
- Convert to USD for international; note the exchange rate and date
- Missing data shows as "-" or "N/A" with an "[E]" flag for estimates — never blank
- Every number has a citation: "[Company] [Document] ([Date])"

### Design

- **Slide titles are insights, not labels.** "Scale leaders pulling away from niche players" — not "Competitive Analysis."
- **Signposts are quantified.** "Margin below 40%" — not "margins decline."
- **Ratings show the actual.** "●●● $160B" — not just "●●●."
- **Charts are real chart objects** — not text tables dressed up to look like charts.

**Typography** — set explicitly, don't rely on defaults:
- Slide titles: 28-32pt bold
- Section headers: 18-20pt bold
- Body text: 14-16pt (never below 14pt)
- Table text: 14pt
- Sources/footnotes: 14pt, gray
- Same element type = same size throughout the deck

**Charts:**
- Legend inside the chart boundary, not floating over the plot area
- Right-side legend for pies (≤6 slices), bottom legend for line/bar (≤4 series)
- More than 6 series → split into multiple charts or use a table
- Pie charts show percentages on slices, not just in the legend

**Tables:**
- Light gray header row, bold
- Right-align numbers, left-align text
- Enough cell padding that text doesn't touch borders

**Color:** 2-3 colors max. Muted — navy, gray, one accent. Same color meanings throughout.

### What's strict vs. flexible

| Always | Case-by-case |
|---|---|
| Exact titles/sections when user specifies | Creative titles when they don't |
| Chart when user says chart; table when they say table | Visualization type when unspecified |
| Every competitor/data point they list | Number of competitors when unspecified |
| Exact values when specified | Rounding when precision unspecified |
| Titles fit without overflow | Number of competitor categories |
| No overlapping elements | Which dimensions to compare |

---

## Analysis workflow

### Step 0 — Industry-defining metrics

Before anything else: what 3-5 metrics does this industry actually run on? Use these consistently across every competitor.

| Industry | Key metrics |
|---|---|
| SaaS | ARR, NRR, CAC payback, LTV/CAC, Rule of 40 |
| Payments | GPV, take rate, attach rate, transaction margin |
| Marketplaces | GMV, take rate, buyer/seller ratio, repeat rate |
| Retail | Same-store sales, inventory turns, sales per sq ft |
| Logistics | Volume, cost per unit, on-time delivery %, capacity utilization |

Industry not listed — pick the metrics investors and operators benchmark on.

### Step 1 — Market context

Size, growth, drivers, headwinds. With sources.

Correct: "Embedded payments is $80-100B in 2024, growing 20-25% CAGR (McKinsey 2024)"
Wrong: "The market is large and growing rapidly"

### Step 2 — Industry economics

Map how value flows. Approach depends on industry structure:
- **Vertically structured** — value chain layers, typical margin at each
- **Platform/network** — ecosystem participants, value flows between them
- **Fragmented** — consolidation dynamics, margin differences by scale

### Step 3 — Target company profile

| Metric | Value |
|---|---|
| Revenue | $4.96B |
| Growth | +26% YoY |
| Gross Margin | 45% |
| Profitability | $373M Adj. EBITDA |
| Customers | 134K |
| Retention | 92% |
| Market Share | ~15% |

Multi-segment companies add a breakdown:

| Segment | Revenue | Rev YoY | Rev % | EBITDA | EBITDA YoY | Margin |
|---|---|---|---|---|---|---|
| Seg A | $25.1B | +26% | 57% | $6.5B | +31% | 26% |
| Seg B | $13.8B | +31% | 31% | $2.5B | +64% | 18% |
| Seg C | $5.1B | -2% | 12% | -$74M | -16% | -1% |
| Total | $44.0B | +18% | 100% | $6.5B* | - | 15% |

*Note corporate costs if applicable

### Step 4 — Competitor mapping

Group by whichever lens fits (this is a good `ask_user_question` decision if the user hasn't specified):
- By business model — platform / vertical / horizontal
- By segment — enterprise / SMB / consumer
- By posture — direct / adjacent / emerging
- By origin — incumbent / disruptor / new entrant

### Step 5 — Positioning visualization

| Type | When |
|---|---|
| 2×2 matrix | Two dominant competitive factors |
| Radar/spider | Multi-factor comparison |
| Tier diagram | Natural clustering into strategic groups |
| Value chain map | Vertical industries |
| Ecosystem map | Platform markets |

See `references/frameworks.md` for 2×2 axis pairs by industry.

### Step 6 — Competitor deep-dives

Two tables per competitor.

**Metrics:**

| Metric | Value |
|---|---|
| Revenue | $X.XB |
| Growth | +XX% YoY |
| Gross Margin | XX% |
| Market Cap | $X.XB |
| Profitability | $XXXM EBITDA |
| Customers | XXK |
| Retention | XX% |
| Market Share | ~XX% |

**Qualitative:**

| Category | Assessment |
|---|---|
| Business | What they do (1 sentence) |
| Strengths | 2-3 bullets |
| Weaknesses | 2-3 bullets |
| Strategy | Current priorities |

### Step 7 — Comparative analysis

| Dimension | Company A | Company B | Company C |
|---|---|---|---|
| Scale | ●●● $160B | ●●○ $45B | ●○○ $8B |
| Growth | ●●○ +26% | ●●● +35% | ●●○ +22% |
| Margins | ●●○ 7.5% | ●○○ 3.2% | ●●● 15% |

### Step 8 — Strategic context

M&A transactions (multiples, rationale), partnership trends, capital raising patterns, regulatory developments. See `references/schemas.md` for the M&A transaction table format.

### Step 9 — Synthesis

**Moat assessment** — rate each competitor Strong / Moderate / Weak on:

| Moat | What to assess |
|---|---|
| Network effects | User/supplier flywheel strength; cross-side vs same-side |
| Switching costs | Technical integration depth, contractual lock-in, behavioral habits |
| Scale economies | Unit cost advantages at volume; minimum efficient scale |
| Intangible assets | Brand, proprietary data, regulatory licenses, patents |

**Required synthesis elements:**
- Durable advantages (hard to replicate) — map to moat categories
- Structural vulnerabilities (hard to fix)
- Current state vs. trajectory

**For investment contexts** (skip if the Phase 1 scoping said no):

| Scenario | Probability | Key driver |
|---|---|---|
| Bull | 30% | Market share gains, margin expansion |
| Base | 50% | Current trajectory continues |
| Bear | 20% | Competitive pressure, margin compression |

---

## Quality checklist

Before finishing:

**Prompt fidelity**
- Slide titles match what the user specified, verbatim
- Charts where they said chart; tables where they said table
- Every competitor/year/data point they listed is present
- Exact values and formats as specified

**Data consistency**
- Source-file values extracted directly, not recalculated
- Same metric shows the same value on every slide it appears
- Same decimal precision as the source

**Layout**
- Titles fit without overflow
- No overlapping elements
- All text within containers, no clipping

**Content**
- Every number has a citation
- All metrics from the same fiscal period (or flagged)
- Slide titles state insights, not topics
- Charts are real chart objects

Run standard visual verification checks on every slide — this catches overlaps, overflow, and low-contrast text that don't show up when you're reading back the XML.
```

---

**파일 2: `v2-competitive-analysis/references/schemas.md`**

```markdown
# Schemas Reference

Additional table formats not shown in main SKILL.md.

## M&A Transaction Table

| Acquirer | Target | Date | Deal Value | Multiple | Rationale |
|----------|--------|------|------------|----------|-----------|
| Company A | Company B | MMM YYYY | $X.XB | X.Xx EV/Rev | [Strategic logic] |

State multiple methodology: "X.Xx EV/Revenue" or "X.Xx EV/EBITDA"

## Scenario Analysis Table

| Scenario | Probability | Valuation | Key Assumptions |
|----------|-------------|-----------|-----------------|
| Bull | XX% | $XXB | [Specific, quantified] |
| Base | XX% | $XXB | [Specific, quantified] |
| Bear | XX% | $XXB | [Specific, quantified] |

## Slide Structure

┌─────────────────────────────────────────────────────────────┐
│ [Insight headline, not topic]                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                     [Main Content]                          │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ Source: [Citation] ([Date])                                 │
└─────────────────────────────────────────────────────────────┘
```

---

**파일 3: `v2-competitive-analysis/references/frameworks.md`**

```markdown
# Frameworks Reference

## 2x2 Matrix: Common Axis Pairs by Industry

*Technology/SaaS:* Product breadth × Customer segment, Integration depth × Geographic reach

*Consumer/Retail:* Price point × Product range, Online × Offline presence

*Financial Services:* Product complexity × Customer sophistication, Scale × Specialization

*Healthcare:* Care setting × Payer mix, Technology enablement × Service breadth

*Industrial:* Customization × Scale, Geographic scope × Vertical focus
```

---

**파일 4: `v2-deck-refresh/SKILL.md`**

```markdown
---
name: v2-deck-refresh
description: Updates a presentation with new numbers — quarterly refreshes, earnings updates, comp rolls, rebased market data. Use whenever the user asks to "update the deck with Q4 numbers", "refresh the comps", "roll this forward", "swap in the new earnings", "change all the $485M to $512M", or any request to swap figures across an existing deck without rebuilding it.
---

# Deck Refresh

Update numbers across the deck. The deck is the source of truth for formatting; you're only changing values.

## Environment check

This skill works in both the PowerPoint add-in and chat. Identify which you're in before starting — the edit mechanism differs, the intent doesn't:

- **Add-in** — the deck is open live; edit text runs, table cells, and chart data directly.
- **Chat** — the deck is an uploaded file; edit it by regenerating the affected slides with the new values and writing the result back.

Either way: smallest possible change, existing formatting stays intact.

This is a four-phase process and the third phase is an approval gate. Don't edit until the user has seen the plan.

## Phase 1 — Get the data

Use `ask_user_question` to find out how the new numbers are arriving:

- **Pasted mapping** — user types or pastes "revenue $485M → $512M, EBITDA $120M → $135M." The clearest case.
- **Uploaded Excel** — old/new columns, or a fresh output sheet the user wants pulled from. Read it, confirm which column is which before you trust it.
- **Just the new values** — "Q4 revenue was $512M, margins were 22%." You figure out what each one replaces. Workable, but confirm the mapping before you touch anything — a "$512M" that you map to revenue but the user meant for gross profit is a quiet disaster.

Also ask about **derived numbers**: if revenue moves, does the user want growth rates and share percentages recalculated, or left alone? Most decks have "+15% YoY" baked in somewhere that's now stale. Whether to touch those is a judgment call the user should make, not you.

## Phase 2 — Read everything, find everything

Read every slide. For each old value, find every instance — including the ones that don't look the same:

| Variant | Example |
|---|---|
| Scale | `$485M`, `$0.485B`, `$485,000,000` |
| Precision | `$485M`, `$485.0M`, `~$485M` |
| Unit style | `$485M`, `$485MM`, `$485 million`, `485M` |
| Embedded | "revenue grew to $485M", "a $485M business", axis labels |

A deck that says `$485M` on slide 3, `485` on slide 8's chart axis, and `$485.0 million` in a footnote on slide 15 has three instances of the same number. Find-replace misses two of them. You shouldn't.

**Where numbers hide:**
- Text boxes (obvious)
- Table cells
- Chart data labels and axis labels
- Chart source data — the numbers driving the bars, not just the labels on them
- Footnotes, source lines, small print
- Speaker notes, if the user cares about those

Build a list: for each old value, every location it appears, the exact text it appears as, and what it'll become. This list is the plan.

## Phase 3 — Present the plan, get approval

**This is a destructive operation on a deck someone spent time on.** Show the full change list before editing a single thing. Format it so it's scannable:

$485M → $512M (Revenue)
  Slide 3  — Title box: "Revenue grew to $485M"
  Slide 8  — Chart axis label: "485"
  Slide 15 — Footnote: "$485.0 million in FY24 revenue"

$120M → $135M (Adj. EBITDA)
  Slide 3  — Table cell
  Slide 11 — Body text: "$120M of Adj. EBITDA"

FLAGGED — possibly derived, not in your mapping:
  Slide 3  — "+15% YoY" (growth rate — stale if base year didn't change?)
  Slide 7  — "12% market share" (was this computed from $485M / market size?)

The flagged section matters. You're not just executing a find-replace — you're catching the second-order effects the user would've missed at 11pm. If the mapping says `$485M → $512M` and slide 3 also has `+15% YoY` right next to it, that growth rate is probably wrong now. Flag it; don't silently fix it, don't silently leave it.

Use `ask_user_question` for the approval: proceed as shown, proceed but skip the flagged items, or let them revise the mapping first.

## Phase 4 — Execute, preserve, report

For each change, make the smallest edit that accomplishes it. How that happens depends on your environment:

- **Add-in** — edit the specific run, cell, or chart series directly in the live deck.
- **Chat** — regenerate the affected slide with the new value in place, preserving every other element exactly as it was, and write it back to the file.

Either way, the standard is the same:

- **Text in a shape** — change the value, leave font/size/color/bold state exactly as they were. If `$485M` is 14pt navy bold inside a sentence, `$512M` is 14pt navy bold inside the same sentence.
- **Table cell** — change the cell, leave the table alone.
- **Chart data** — update the underlying series values so the bars/lines actually move. Editing just the label without the data leaves a chart that lies.

Don't reformat anything you didn't need to touch. The deck's existing style is correct by definition; you're a surgeon, not a renovator.

After the last edit, report what actually happened:

Updated 11 values across 8 slides.

Changed:
  [the list from Phase 3, now past-tense]

Still flagged — did NOT change:
  Slide 3 — "+15% YoY" (derived; confirm separately)
  Slide 7 — "12% market share"

Run standard visual verification checks on every edited slide. A number that got longer (`$485M` → `$1,205M`) might now overflow its text box or push a table column width. Catch it before the user does.

## What you're not doing

- **Not rebuilding slides** — if a slide's narrative no longer makes sense with the new numbers ("margins compressed" but margins went up), flag it, don't rewrite it.
- **Not recalculating unless asked** — derived numbers are the user's call. Your Phase 1 question covers this.
- **Not touching formatting** — if the deck uses `$MM` and the user's mapping says `$M`, match the deck, not the mapping. Values change; style stays.
```

---

나머지 5~8번 이어서 갑니다:

**파일 5: `v2-ib-check-deck/SKILL.md`**

```markdown
---
name: v2-ib-check-deck
description: Investment banking presentation quality checker. Reviews a pitch deck or client-ready presentation for (1) number consistency across slides, (2) data-narrative alignment, (3) language polish against IB standards, (4) visual and formatting QC. Use whenever the user asks to review, check, QC, proof, or do a final pass on a deck, pitch, or client materials — including requests like "check my numbers", "reconcile figures across slides", "is this client-ready", or "what am I missing before I send this out".
---

# IB Deck Checker

Perform comprehensive QC on the presentation across four dimensions. Read every slide, then report findings.

## Environment check

This skill works in both the PowerPoint add-in and chat. Identify which you're in before starting:

- **Add-in** — read from the live open deck.
- **Chat** — read from the uploaded `.pptx` file.

This is read-and-report only — no edits — so the workflow is identical in both.

## Workflow

### Read the deck

Pull text from every slide, keeping track of which slide each line came from. You'll need slide-level attribution for every finding ("$500M appears on slides 3 and 8, but slide 15 shows $485M"). A deck with 30 slides is too much to hold in working memory reliably — write the extracted text to a file so the number-checking script can process it.

The script expects markdown-ish input with slide markers. Format as:

## Slide 1
[slide 1 text content]

## Slide 2
[slide 2 text content]

### 1. Number consistency

Run the extraction script on what you collected:

python scripts/extract_numbers.py /tmp/deck_content.md --check

It normalizes units ($500M vs $500MM vs $500,000,000 → same number), categorizes values (revenue, EBITDA, multiples, margins), and flags when the same metric category shows conflicting values on different slides. This is the part most likely to catch something a human missed on the fifth read-through.

Beyond what the script flags, verify:
- Calculations are correct (totals sum, percentages add up, growth rates match the endpoints)
- Unit style is consistent — the deck should pick one of $M or $MM and stick with it
- Time periods are aligned — FY vs LTM vs quarterly, explicitly labeled

### 2. Data-narrative alignment

Map claims to the data that's supposed to support them. This is where decks go wrong quietly — someone edits the chart on slide 7 and forgets the narrative on slide 4.

- Trend statements ("declining margins") → does the chart actually go that direction?
- Market position claims ("#1 player") → revenue and share data support it?
- Plausibility — "#1 in a $100B market" with $200M revenue is 0.2% share; that's not #1

### 3. Language polish

IB decks have a register. Scan for anything that breaks it: casual phrasing ("pretty good", "a lot of"), contractions, exclamation points, vague quantifiers without numbers, inconsistent terminology for the same concept.

See `references/ib-terminology.md` for replacement patterns.

### 4. Visual and formatting QC

Run standard visual verification checks on each slide. You're looking for: missing chart source citations, missing axis labels, typography inconsistencies, number formatting drift (1,000 vs 1K within the same deck), date format drift, footnote and disclaimer gaps.

Visual verification catches overlaps, overflow, and contrast issues that don't show up in text extraction. Don't skip it — a chart with no source citation looks the same as a properly sourced one in the text dump.

## Output

Use `references/report-format.md` as the structure. Categorize by severity:

- **Critical** — number mismatches, factual errors, data contradicting narrative. These block client delivery.
- **Important** — language, missing sources, terminology drift. Should fix.
- **Minor** — font sizes, spacing, date formats. Polish.

Lead with criticals. If there aren't any, say so explicitly — "no number inconsistencies found" is a finding, not an absence of one.
```

---

**파일 6: `v2-ib-check-deck/references/report-format.md`**

```markdown
# Deck Check Report Format

## Report Template

# Deck Check Report: [Presentation Name]

## Summary
- Total issues: X
- Critical: X (number mismatches, factual errors)
- Important: X (narrative-data alignment, language)
- Minor: X (formatting)

## Critical Issues

### Number Consistency
1. **[Issue name]** (Slides X, Y)
   - Slide X: [value]
   - Slide Y: [value]
   - Action: [recommendation]

### Data-Narrative Alignment
1. **[Issue name]** (Slides X, Y)
   - Claim: "[quoted text]"
   - Data shows: [contradiction]
   - Action: [recommendation]

## Important Issues

### Language Polish
1. **[Issue type]** (Slide X)
   - Current: "[quoted text]"
   - Suggested: "[replacement]"

## Minor Issues

### Formatting
1. **[Issue type]** (Slide X)
   - [Description and fix]

## Final Checklist
- [ ] Numbers reconciled
- [ ] Narrative matches data
- [ ] Language meets IB standards
- [ ] Charts sourced
- [ ] Formatting consistent

## Issue Severity Classification

**Critical** (must fix before client delivery):
- Number mismatches across slides
- Calculation errors
- Factual inaccuracies (names, titles, dates)
- Data contradicting narrative

**Important** (should fix):
- Casual/informal language
- Vague claims without specificity
- Terminology inconsistency
- Missing chart sources

**Minor** (polish items):
- Font/color inconsistencies
- Date format variations
- Spacing/alignment issues
- Orphaned text
```

---

**파일 7: `v2-ib-check-deck/references/ib-terminology.md`**

```markdown
# IB Terminology Reference

## Casual to Professional Replacements

| Casual/Informal | IB Standard |
|-----------------|-------------|
| "a lot of growth" | "significant growth" or "X% growth" |
| "pretty good margins" | "attractive margins" or "margins of X%" |
| "they bought the company" | "the company was acquired" |
| "big deal" | "transformative transaction" |
| "cheap valuation" | "attractive valuation" or "valuation discount" |
| "expensive" | "premium valuation" |
| "make more money" | "enhance profitability" or "drive margin expansion" |
| "getting bigger" | "pursuing growth" or "expanding operations" |
| "cut costs" | "implement cost optimization" or "drive operational efficiencies" |
| "good fit" | "strategic fit" or "compelling strategic rationale" |
| "help with" | "support" or "facilitate" |
| "a bunch of" | "multiple" or "numerous" |
| "kind of" / "sort of" | [remove or be specific] |
| "really" / "very" | [remove or quantify] |
| "tons of" | "substantial" or quantify |
| "huge" | "significant" or quantify |
| "pretty much" | [remove or be precise] |
| "basically" | [remove or clarify] |

## Language Patterns to Avoid

- **Contractions**: Don't → Do not, won't → will not
- **Exclamation points**: Generally inappropriate for IB materials
- **First-person**: "We think..." → "Management believes..." or passive voice
- **Superlatives without evidence**: "best-in-class" requires supporting data
- **Vague quantifiers**: "some", "many", "several" → specific numbers

## Preferred Phrasing Patterns

**Growth narratives**:
- "Demonstrated track record of X% revenue CAGR"
- "Consistent margin expansion over [period]"
- "Proven ability to generate organic growth"

**Market position**:
- "#X player in [specific segment]"
- "Leading provider of [specific offering]"
- "Differentiated positioning through [specific attribute]"

**Strategic rationale**:
- "Compelling strategic fit driven by..."
- "Attractive value creation opportunity through..."
- "Synergy potential of $Xm from [specific sources]"
```

---

**파일 8: `v2-ib-check-deck/scripts/extract_numbers.py`**

```python
#!/usr/bin/env python3
"""
Extract numerical values from presentation content for consistency checking.

Usage:
    python extract_numbers.py presentation-content.md
    python extract_numbers.py presentation-content.md --output numbers.json

This script parses markdown-formatted presentation content (from markitdown)
and extracts all numerical values with their context and slide references.
"""

import argparse
import json
import re
import sys
from collections import defaultdict
from dataclasses import dataclass, asdict
from pathlib import Path
from typing import Optional


@dataclass
class NumberInstance:
    """A numerical value found in the presentation."""
    value: str           # Original string representation
    normalized: float    # Normalized numeric value
    unit: str           # Detected unit (M, B, K, %, bps, x, etc.)
    slide: int          # Slide number (0 if unknown)
    context: str        # Surrounding text for context
    line_number: int    # Line number in source file
    category: str       # Detected category (revenue, margin, multiple, etc.)


def normalize_number(value_str: str, unit: str) -> float:
    """Convert a number string with unit to a normalized float value."""
    clean = re.sub(r'[,\s]', '', value_str)

    try:
        base_value = float(clean)
    except ValueError:
        return 0.0

    multipliers = {
        'T': 1e12,
        'B': 1e9,
        'bn': 1e9,
        'billion': 1e9,
        'M': 1e6,
        'mm': 1e6,
        'mn': 1e6,
        'million': 1e6,
        'K': 1e3,
        'k': 1e3,
        'thousand': 1e3,
    }

    for unit_key in sorted(multipliers.keys(), key=len, reverse=True):
        if unit_key.lower() in unit.lower():
            return base_value * multipliers[unit_key]

    return base_value


def detect_category(context: str, unit: str) -> str:
    """Detect the category of a number based on context and unit."""
    context_lower = context.lower()

    if any(term in context_lower for term in ['revenue', 'sales', 'top line', 'topline']):
        return 'revenue'

    if 'ebitda' in context_lower:
        if any(term in context_lower for term in ['margin', '%', 'percent']):
            return 'ebitda_margin'
        return 'ebitda'

    if any(term in context_lower for term in ['margin', 'profit']):
        return 'margin'

    if any(term in context_lower for term in ['growth', 'cagr', 'yoy', 'y/y']):
        return 'growth'

    if any(term in context_lower for term in ['multiple', 'ev/', 'p/e', 'ev/ebitda', 'ev/revenue']):
        return 'multiple'

    if any(term in context_lower for term in ['enterprise value', 'ev ', 'market cap']):
        return 'valuation'

    if unit in ['%', 'bps', 'percent']:
        return 'percentage'

    if unit == 'x':
        return 'multiple'

    return 'other'


def extract_numbers(content: str) -> list[NumberInstance]:
    """Extract all numbers from presentation content."""
    numbers = []
    current_slide = 0

    slide_pattern = re.compile(r'^#+\s*Slide\s*(\d+)|^<!-- Slide (\d+)')

    number_pattern = re.compile(
        r'(?P<currency>[$\u20ac\u00a3\u00a5])?'
        r'(?P<number>[\d,]+(?:\.\d+)?)'
        r'\s*'
        r'(?P<unit>%|bps|x|'
        r'[Tt]rillion|[Bb]illion|[Mm]illion|[Tt]housand|'
        r'[TBMKtbmk]n?|mm|MM)?'
        r'(?!\d)'
    )

    lines = content.split('\n')

    for line_num, line in enumerate(lines, 1):
        slide_match = slide_pattern.match(line)
        if slide_match:
            current_slide = int(slide_match.group(1) or slide_match.group(2))
            continue

        for match in number_pattern.finditer(line):
            value_str = match.group('number')
            currency = match.group('currency') or ''
            unit = match.group('unit') or ''

            if len(value_str.replace(',', '').replace('.', '')) < 2 and not unit:
                continue

            try:
                num_val = float(value_str.replace(',', ''))
                if 1900 <= num_val <= 2099 and not unit and not currency:
                    continue
            except ValueError:
                pass

            full_value = f"{currency}{value_str}{unit}"

            start = max(0, match.start() - 50)
            end = min(len(line), match.end() + 50)
            context = line[start:end].strip()

            if currency:
                if not unit:
                    unit = 'USD'
                else:
                    unit = f"USD_{unit}"

            normalized = normalize_number(value_str, unit)
            category = detect_category(context, unit)

            numbers.append(NumberInstance(
                value=full_value,
                normalized=normalized,
                unit=unit or 'none',
                slide=current_slide,
                context=context,
                line_number=line_num,
                category=category
            ))

    return numbers


def find_inconsistencies(numbers: list[NumberInstance]) -> list[dict]:
    """Find potential inconsistencies in extracted numbers."""
    inconsistencies = []

    by_category = defaultdict(list)
    for num in numbers:
        if num.category != 'other':
            by_category[num.category].append(num)

    for category, instances in by_category.items():
        if len(instances) < 2:
            continue

        value_groups = []
        for inst in instances:
            placed = False
            for group in value_groups:
                ref_value = group[0].normalized
                if ref_value > 0:
                    diff_pct = abs(inst.normalized - ref_value) / ref_value
                    if diff_pct < 0.05:
                        group.append(inst)
                        placed = True
                        break
            if not placed:
                value_groups.append([inst])

        if len(value_groups) > 1:
            value_groups.sort(key=len, reverse=True)

            main_group = value_groups[0]
            for other_group in value_groups[1:]:
                inconsistencies.append({
                    'category': category,
                    'expected': {
                        'value': main_group[0].value,
                        'slides': sorted(set(n.slide for n in main_group)),
                        'count': len(main_group)
                    },
                    'found': {
                        'value': other_group[0].value,
                        'slides': sorted(set(n.slide for n in other_group)),
                        'count': len(other_group)
                    },
                    'severity': 'high' if category in ['revenue', 'ebitda', 'valuation'] else 'medium'
                })

    return inconsistencies


def main():
    parser = argparse.ArgumentParser(
        description='Extract numbers from presentation content for consistency checking'
    )
    parser.add_argument('input_file', help='Markdown file with presentation content')
    parser.add_argument('--output', '-o', help='Output JSON file (default: stdout)')
    parser.add_argument('--check', '-c', action='store_true',
                       help='Check for inconsistencies and report')

    args = parser.parse_args()

    input_path = Path(args.input_file)
    if not input_path.exists():
        print(f"Error: File not found: {args.input_file}", file=sys.stderr)
        sys.exit(1)

    content = input_path.read_text()

    numbers = extract_numbers(content)

    output = {
        'total_numbers': len(numbers),
        'by_category': defaultdict(list),
        'numbers': [asdict(n) for n in numbers]
    }

    for num in numbers:
        output['by_category'][num.category].append({
            'value': num.value,
            'slide': num.slide,
            'context': num.context[:100]
        })

    output['by_category'] = dict(output['by_category'])

    if args.check:
        inconsistencies = find_inconsistencies(numbers)
        output['inconsistencies'] = inconsistencies

        if inconsistencies:
            print("\n=== POTENTIAL INCONSISTENCIES DETECTED ===\n", file=sys.stderr)
            for inc in inconsistencies:
                print(f"Category: {inc['category'].upper()}", file=sys.stderr)
                print(f"  Expected: {inc['expected']['value']} (Slides: {inc['expected']['slides']}, Count: {inc['expected']['count']})", file=sys.stderr)
                print(f"  Found:    {inc['found']['value']} (Slides: {inc['found']['slides']}, Count: {inc['found']['count']})", file=sys.stderr)
                print(f"  Severity: {inc['severity']}", file=sys.stderr)
                print(file=sys.stderr)

    json_output = json.dumps(output, indent=2)

    if args.output:
        Path(args.output).write_text(json_output)
        print(f"Output written to {args.output}", file=sys.stderr)
    else:
        print(json_output)


if __name__ == '__main__':
    main()
```

---

**전체 8개 파일 완료!** 각 코드블록을 복사해서 아래 폴더 구조로 저장하시면 됩니다:

```
v2-competitive-analysis/
├── SKILL.md
└── references/
    ├── schemas.md
    └── frameworks.md

v2-deck-refresh/
└── SKILL.md

v2-ib-check-deck/
├── SKILL.md
├── references/
│   ├── report-format.md
│   └── ib-terminology.md
└── scripts/
    └── extract_numbers.py
```