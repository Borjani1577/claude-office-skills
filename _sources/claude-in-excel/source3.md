스킬 원본을 확인했습니다. 이전 두 스킬 대비 상당히 간결한 구조입니다. 완전한 복원 문서를 작성하겠습니다.

---

# 📋 `clean-data-xls` 스킬 복원 문서

## 1. `skill.md` — 스킬 정의서 (원본 그대로)

```markdown
---
name: clean-data-xls
description: Clean up messy spreadsheet data — trim whitespace, fix inconsistent casing, convert numbers-stored-as-text, standardize dates, remove duplicates, and flag mixed-type columns. Use when data is messy, inconsistent, or needs prep before analysis. Triggers on "clean this data", "clean up this sheet", "normalize this data", "fix formatting", "dedupe", "standardize this column", "this data is messy".
---

# Clean Data

Clean messy data in the active sheet or a specified range.

## Environment

- **If running inside Excel (Office Add-in / Office JS):** Use Office JS directly (`Excel.run(async (context) => {...})`). Read via `range.values`, write helper-column formulas via `range.formulas = [["=TRIM(A2)"]]`. The in-place vs helper-column decision still applies.
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
| Encoding | mojibake (`Ã©`, `â€™`), non-printing characters |
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
```

---

## 2. 스킬 구조 분석

### 파일 구조

```
clean-data-xls/
└── skill.md          ← 단일 파일 (참조 문서 없음)
```

`audit-xls`나 `3-statement-model`과 달리 **참조 파일이 전혀 없는** 가장 가벼운 스킬입니다.

---

### 워크플로우 구조도

```
Step 1: Scope (범위 결정)
│
│  사용자가 범위 지정?
│  ├─ YES → 해당 범위 사용
│  └─ NO  → 활성 시트 전체 used range 사용
│
│  각 열 프로파일링: 지배적 타입(텍스트/숫자/날짜) + 이상치 식별
│
▼
Step 2: Detect issues (이슈 탐지)
│
│  9가지 체크 항목 순회:
│  ┌─────────────────────────────────────────────────┐
│  │ 1. Whitespace     — 앞뒤 공백, 이중 공백         │
│  │ 2. Casing         — 대소문자 불일치               │
│  │ 3. Number-as-text — 텍스트로 저장된 숫자          │
│  │ 4. Dates          — 같은 열 내 혼합 날짜 포맷     │
│  │ 5. Duplicates     — 완전 중복 + 유사 중복         │
│  │ 6. Blanks         — 채워져야 할 빈 셀            │
│  │ 7. Mixed types    — 98% 숫자인데 3개 텍스트 등    │
│  │ 8. Encoding       — 깨진 문자, 비인쇄 문자        │
│  │ 9. Errors         — #REF!, #N/A, #VALUE! 등     │
│  └─────────────────────────────────────────────────┘
│
▼
Step 3: Propose fixes (수정 제안)
│
│  변경 전 요약 테이블 출력:
│  | Column | Issue | Count | Proposed Fix |
│
│  ⏸️ 사용자 확인 대기 (아직 아무것도 변경하지 않음)
│
▼
Step 4: Apply (적용)
│
│  카테고리별 순차 적용 (각 단계마다 사용자 확인):
│
│  Whitespace ──→ 샘플 확인 ──→ 승인
│       │
│       ▼
│  Casing ──────→ 샘플 확인 ──→ 승인
│       │
│       ▼
│  Number conv ─→ 샘플 확인 ──→ 승인
│       │
│       ▼
│  Dates ───────→ 샘플 확인 ──→ 승인
│       │
│       ▼
│  Dedup ───────→ 샘플 확인 ──→ 승인 (파괴적 작업 — 추가 확인)
│       │
│       ▼
│  Before/After 요약 보고서 출력
```

---

## 3. 핵심 설계 원칙 분석

### 수식 우선 (Formula-first) 원칙

이 스킬의 가장 중요한 설계 철학입니다:

| 상황 | 접근법 |
|---|---|
| 공백 제거 | `=TRIM(A2)` 헬퍼 열 → 원본 유지 |
| 대소문자 통일 | `=UPPER(C2)` 또는 `=PROPER(C2)` 헬퍼 열 |
| 텍스트→숫자 변환 | `=VALUE(SUBSTITUTE(B2,"$",""))` 헬퍼 열 |
| 날짜 표준화 | `=DATEVALUE(D2)` 헬퍼 열 |
| 인코딩 수정 | ✖️ 수식으로 불가 → **유일하게** 직접 덮어쓰기 허용 |
| 중복 제거 | ⚠️ 파괴적 작업 → 반드시 사용자 확인 후 실행 |

### 변경 방식 안전 등급

| 등급 | 설명 | 예시 |
|---|---|---|
| 🟢 안전 | 헬퍼 열에 수식 작성 (원본 보존) | TRIM, UPPER, VALUE, DATEVALUE |
| 🟡 주의 | 원본 직접 수정 (수식 불가한 경우) | 인코딩 수정, 비인쇄 문자 제거 |
| 🔴 파괴적 | 행 삭제/데이터 손실 가능 | 중복 제거, 빈 셀 채우기, 원본 덮어쓰기 |

---

## 4. 3개 스킬 비교 분석

| 비교 항목 | `audit-xls` | `3-statement-model` | `clean-data-xls` |
|---|---|---|---|
| **파일 수** | 1개 | 4개 | 1개 |
| **총 분량** | ~3,500자 | ~25,000자+ | ~1,500자 |
| **워크플로우 단계** | 4단계 | 6단계 + 검증체계 | 4단계 |
| **참조 문서** | 없음 | 3개 (formulas, formatting, sec-filings) | 없음 |
| **목적** | 읽기전용 감사 | 쓰기 — 모델 구축 | 쓰기 — 데이터 정제 |
| **사용자 체크포인트** | 결과 보고 후 1회 | 재무제표마다 5회 | 카테고리마다 N회 |
| **수식 철학** | "수식이 없네요" 발견 | "수식 필수" 강제 | "수식으로 정제" 선호 |
| **파괴적 작업** | 없음 (보고만) | 있음 (셀 쓰기) | 있음 (중복 제거 등) |
| **스코프 분기** | selection/sheet/model | 템플릿 구조 적응 | range/used range |
| **도메인 지식** | 재무모델 구조 이해 | IS/BS/CF 회계 지식 | 범용 데이터 품질 |
| **트리거 키워드** | "audit", "check formulas", "sanity check" | "fill out template", "complete model" | "clean", "normalize", "dedupe", "messy" |

---

## 5. 스킬 설계 패턴 비교

### 공통 패턴 (3개 스킬 모두)

```
1. Scope 결정 → 2. 데이터 읽기/분석 → 3. 결과 보고 → 4. 사용자 확인 → 5. (선택적) 수정
```

### 안전장치 패턴

| 패턴 | audit-xls | 3-statement-model | clean-data-xls |
|---|---|---|---|
| **변경 전 보고** | ✅ 전체 보고 후 수정 질문 | ✅ 각 단계마다 확인 | ✅ 제안 테이블 먼저 |
| **파괴적 작업 이중 확인** | N/A (읽기전용) | ⚠️ 수식 덮어쓰기 시 | ✅ 명시적 규정 |
| **원본 보존** | N/A | 히스토리컬은 하드코딩 보존 | 헬퍼 열로 원본 보존 |
| **롤백 가능성** | N/A | 수식이므로 입력값만 변경 | 헬퍼 열 삭제로 롤백 |

### 환경 분기 패턴

3개 스킬 중 2개(`3-statement-model`, `clean-data-xls`)가 환경 분기를 명시합니다:

```
실행 환경 확인
├─ Excel Add-in (Office JS) → range.formulas / range.values 직접 사용
└─ Python (openpyxl)        → ws["A1"] = "=FORMULA" 문자열 작성
```

`audit-xls`는 환경 분기가 없음 — 읽기전용이므로 환경에 무관하게 동일한 로직 적용.

---

## 6. 스킬 복잡도 스펙트럼

```
간결 ◄──────────────────────────────────────────────────► 복잡

clean-data-xls          audit-xls          3-statement-model
   (~1,500자)            (~3,500자)           (~25,000자+)
   1 파일                1 파일               4 파일
   범용 데이터            도메인 특화            고도 도메인 특화
   4단계 선형             4단계 분기형           6단계 + 검증 체계
```

---

이것으로 `clean-data-xls` 스킬의 완전한 복원 문서를 마쳤습니다. 3개 스킬 전체를 비교 분석한 결과:

- **`clean-data-xls`**: 가장 가볍고 범용적 — 단일 파일, 어떤 데이터셋에나 적용 가능
- **`audit-xls`**: 중간 복잡도 — 재무모델 도메인 지식 필요하지만 읽기전용
- **`3-statement-model`**: 가장 복잡 — 4개 파일, 회계 전문 지식, 모델 구축까지 수행


