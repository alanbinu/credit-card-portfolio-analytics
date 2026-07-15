# DAX Patterns

## Credit Card Portfolio Analytics & Risk Intelligence

| | |
|---|---|
| **Document Type** | Reusable DAX Design Pattern Reference |
| **Version** | 1.0 |
| **Related Documents** | [DAX Measures.md](./05_DAX_Measures.md), [Data Model.md](./14_Data_Model.md), [Performance Optimization.md](./10_Performance_Optimization.md) |

---

## 1. Why This Document Exists

[DAX Measures.md](./05_DAX_Measures.md) catalogs all 33 measures individually — what each one does, in business terms. Several of those measures share the same small set of underlying DAX techniques. Rather than re-explaining `DIVIDE` or context transition 33 times, this document explains each pattern once, generically, so the measure catalog can stay focused on business logic.

Read this document to understand *why* a formula is shaped the way it is; read [DAX Measures.md](./05_DAX_Measures.md) to understand *what a specific measure* computes.

## 2. Safe Division — `DIVIDE()`

| Aspect | Detail |
|---|---|
| Pattern | `DIVIDE(<numerator>, <denominator>, <alternate result>)` |
| Used in | Every ratio measure in the model — `Delinquency Rate %`, `Payment to Spend Ratio`, `EMI %`, `Average Cashback Per Transaction`, `Average Spend Per Customer` |
| Problem it solves | The `/` operator returns an error (or infinity) when the denominator is `0` or `BLANK()` — which happens routinely under an empty or highly filtered slicer selection. An error in one visual can break an entire report page. |
| Why it matters here | `FR-09` in [Business Requirements.md](./01_Business_Requirements.md) requires every ratio measure to degrade gracefully, returning `0` rather than an error, under empty filter context |

> **Best Practice:** `DIVIDE()` with an explicit alternate-result argument should be non-negotiable for any ratio measure added to this model in the future. A measure using the raw `/` operator should be treated as a defect, not a style preference — see the code-review recommendation in [DAX Measures.md §12](./05_DAX_Measures.md).

## 3. `VAR` / `RETURN` Staging

| Aspect | Detail |
|---|---|
| Pattern | `VAR <name> = <expression> RETURN <expression using name>` |
| Used in | `Current Risk Customers` — the model's most structurally complex measure |
| Problem it solves | Without staging, a multi-step calculation (find the latest month, *then* count within it) either needs to be written as deeply nested `CALCULATE` expressions, or risks re-evaluating the same sub-expression multiple times |
| Why it matters here | `VAR` evaluates its expression exactly once and treats the result as a constant for the rest of the measure — both a readability win and, for expensive sub-expressions, a performance win |

```dax
Current Risk Customers =
VAR LatestMonth =
    CALCULATE(
        MAX(FactRiskProfile[AssessmentMonth]),
        REMOVEFILTERS(DimRiskCategory)
    )
RETURN
    CALCULATE(
        DISTINCTCOUNT(FactRiskProfile[CustomerID]),
        FactRiskProfile[AssessmentMonth] = LatestMonth
    )
```

> **Developer Tip:** Naming a `VAR` after what it represents (`LatestMonth`, not `_x1`) is the difference between a measure a future developer can audit in thirty seconds and one they have to reverse-engineer. Every multi-step measure in this model follows this convention.

## 4. `REMOVEFILTERS()` — Deliberate Filter Removal

| Aspect | Detail |
|---|---|
| Pattern | `CALCULATE(<expression>, REMOVEFILTERS(<table or column>))` |
| Used in | `Current Risk Customers` |
| Problem it solves | Without it, selecting a risk category on a slicer would distort the "latest assessment month" calculation itself — e.g., filtering to "High Risk" might change which month is considered "latest" if High Risk assessments happen to stop earlier than other categories |
| Why it matters here | `REMOVEFILTERS(DimRiskCategory)` guarantees the *definition of "current"* stays stable regardless of what the user has sliced by, while the outer `CALCULATE` still applies the resulting month filter |

> **Architecture Note:** `REMOVEFILTERS` (and its predecessor, `ALL`) is one of the most common sources of subtle bugs in production DAX models — it is easy to remove more filter context than intended by targeting a table rather than a specific column, or vice versa. Every use of `REMOVEFILTERS` in this model targets a specific, named filter target rather than blanket-clearing a whole table.

## 5. Context Transition via `CALCULATE()`

| Aspect | Detail |
|---|---|
| Pattern | `CALCULATE(<expression>, <filter arguments>)` |
| Used in | The majority of measures beyond simple `SUM`/`COUNTROWS` aggregations — `High Risk Customers`, `Delinquent Customers`, `EMI Transactions`, `Full Payment Customers`, `Minimum Payment Customers` |
| Problem it solves | A base aggregation (e.g., `[Total Transactions]`) needs to be re-evaluated under an *additional* filter condition (e.g., only rows where `EMIFlag = 1`) without duplicating the underlying logic |
| Why it matters here | Every conditional measure in this model layers a `CALCULATE` filter on top of a base measure rather than reimplementing the base logic, so the two measures can never drift out of sync — see the `[Current Risk Customers]` → `[High Risk Customers]` relationship in [DAX Measures.md §5.2](./05_DAX_Measures.md) |

## 6. Measure Reuse Over Re-implementation

| Aspect | Detail |
|---|---|
| Pattern | A measure references another measure by name — `[Total Spend] - [Total Payments]`, `CALCULATE([Current Risk Customers], ...)` — rather than repeating the underlying `SUM`/`CALCULATE` logic |
| Used in | `Net Portfolio Exposure`, `Payment to Spend Ratio`, `High Risk Customers`, `EMI %`, and most of the ratio and conditional measures |
| Problem it solves | If two measures each reimplement "what counts as a delinquent customer" independently, a future change to that business rule has to be applied in two places — and eventually won't be |
| Why it matters here | This is the single biggest structural reason the model can claim "one certified definition per metric" (NFR-04 in [Business Requirements.md](./01_Business_Requirements.md)) — base measures are the only place a business rule is defined; everything else composes them |

## 7. Time Intelligence Against a Marked Date Table

| Aspect | Detail |
|---|---|
| Pattern | `DATESMTD()`, `SAMEPERIODLASTYEAR()` applied against `DimDate[Date]` |
| Used in | `Spend MTD`, `Spend PY`, `Spend YoY %`, `Payments MTD`, `Delinquency Rate PY` |
| Precondition | `DimDate` must be marked as the model's official Date table, with a contiguous, unique `Date` column — confirmed by its 1,096-row, single-grain design (see [Data Dictionary.md §7](./03_Data_Dictionary.md)) |
| Why it matters here | Time-intelligence functions silently return incorrect results against a date column that has gaps or duplicates; they do not always throw an error, which makes this precondition easy to violate without noticing |

> **Warning:** If `DimDate` is ever regenerated or re-sourced, the contiguous-date and single-grain properties must be re-verified before trusting any time-intelligence measure — see the acceptance checklist in [Testing & Validation.md](./17_Testing_Validation.md).

## 8. Common Mistakes to Avoid

| Mistake | Symptom | Correct Pattern |
|---|---|---|
| Using `/` instead of `DIVIDE()` | Visual shows an error or infinity under an empty filter context | Section 2 |
| Re-implementing a base measure's filter logic inside a new measure | Two measures quietly diverge after one is edited | Section 6 |
| Applying `REMOVEFILTERS` to an entire table when only one column's filter needs clearing | Unintended filters get stripped, producing an inflated or incorrect result | Section 4 |
| Adding a new bidirectional relationship to "fix" a filter that isn't propagating | Unpredictable filter behavior elsewhere in the model | See [Data Model.md §5](./14_Data_Model.md); the fix belongs in the measure, not the relationship, in nearly every case |
| Writing implicit measures (dragging a raw column into a visual) | Business logic lives outside the auditable, centralized measure table | NFR-03 in [Business Requirements.md](./01_Business_Requirements.md) — explicit measures only |

## 9. Related Documents

- [DAX Measures.md](./05_DAX_Measures.md) — full measure-by-measure catalog
- [Data Model.md](./14_Data_Model.md) — relationship structure these patterns operate over
- [Performance Optimization.md](./10_Performance_Optimization.md) — formula-engine vs. storage-engine cost of these patterns

---

## Version History

| Version | Date | Author | Change Description |
|---|---|---|---|
| 1.0 | 2025-12 | Alan Binu | Initial DAX design pattern reference, factored out of the measure catalog |
