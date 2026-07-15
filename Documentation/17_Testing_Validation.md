# Testing & Validation

## Credit Card Portfolio Analytics & Risk Intelligence

| | |
|---|---|
| **Document Type** | Testing & Validation Reference |
| **Version** | 1.0 |
| **Related Documents** | [Business Requirements.md](./01_Business_Requirements.md), [DAX Measures.md](./05_DAX_Measures.md), [Data Model.md](./14_Data_Model.md), [Data Dictionary.md](./03_Data_Dictionary.md) |

---

## 1. Validation Philosophy

This document defines how the model's correctness is validated, and consolidates the validation notes that appear individually throughout the other documentation into a single checklist a reviewer can execute against the delivered `.pbix`.

## 2. Measure Validation

> **Validation Note:** Every measure in [DAX Measures.md](./05_DAX_Measures.md) should be spot-checked against a manual pivot-table calculation on a filtered subset of the source data before being marked production-ready. The two Repository-Verified measures (`Current Risk Customers`, `Delinquency Rate %`) were validated this way during original development; the remaining Inferred Implementation measures should receive the same pass before being relied upon in a live decision-making context.

| Check | Method | Applies To |
|---|---|---|
| Formula correctness | Manual pivot-table cross-check against a filtered data subset | Every measure |
| Safe-division behavior | Filter a ratio measure's denominator to an empty set; confirm it returns `0`, not an error | All ratio measures — see [DAX Patterns.md §2](./15_DAX_Patterns.md) |
| Latest-month logic | Confirm `Current Risk Customers` resolves to the single most recent `AssessmentMonth`, unaffected by a `RiskCategory` slicer selection | `Current Risk Customers`, `High Risk Customers` |
| Base-measure reuse | Confirm derived measures (`Delinquency Rate %`, `Net Portfolio Exposure`) update consistently when their base measures are edited, rather than drifting | All composed measures — see [DAX Patterns.md §6](./15_DAX_Patterns.md) |

## 3. Relationship Validation

| Check | Method |
|---|---|
| Cross-filter direction | Confirm each of the 11 fact-to-dimension relationships is Single, except `FactRiskProfile ↔ DimCustomer`, which is Bidirectional — see [Data Model.md §4](./14_Data_Model.md) |
| Bidirectional isolation | Slice by a `DimCustomer` attribute (e.g., `State`) and confirm the filter reaches the Risk Analytics page's risk-category breakdown, but does **not** unexpectedly alter `Total Spend` or `Total Payments` on other pages |
| Cardinality | Confirm every fact-to-dimension relationship is one-to-many (dimension to fact), with no many-to-many relationships present |
| `FactPayments[DateID]` semantics | Confirm report authors are aware `DateID` represents the reporting period, not necessarily `PaymentDate` — see [Data Dictionary.md §9](./03_Data_Dictionary.md) |

> **Important:** When Row-Level Security is introduced (see [Project Roadmap.md](./12_Project_Roadmap.md)), its interaction with the bidirectional `FactRiskProfile ↔ DimCustomer` relationship must be explicitly tested — bidirectional relationships combined with RLS are a documented source of unexpected data leakage in Power BI. This is a required test case before RLS ships to production, not an optional one.

## 4. Data Quality Validation

| Check | Method |
|---|---|
| Risk category relabeling | Confirm no `RiskCategory` value of `"Aggressive User"` remains anywhere downstream of Power Query; all instances should read `"Critical Risk"` | 
| Payment-to-spend anomaly | Confirm no `FactPayments` row has a payment-to-spend ratio exceeding 100% after the correction documented in [Power Query Transformations.md §5.2](./08_Power_Query_Transformations.md) |
| Null / blank handling | Spot-check that ratio measures and slicers behave predictably against rows with blank or missing categorical values |
| `UtilizationPercent` divergence | Confirm any divergence between `FactUtilization[UtilizationPercent]` and `FactRiskProfile[UtilizationPercent]` for the same customer-month is within an expected tolerance, not a systemic defect — see [Data Model.md §6](./14_Data_Model.md) |

## 5. Grain and Cross-Check Validation

| Check | Method |
|---|---|
| Row counts match documented grain | Confirm each table's loaded row count matches the inventory in [Data Dictionary.md §2](./03_Data_Dictionary.md) (e.g., `FactTransactions` = 50,000 rows) |
| No duplicate primary keys | Confirm each table's primary key column has no duplicate values post-load |
| `DimDate` contiguity | Confirm `DimDate[Date]` is contiguous and unique across its full range — a precondition for every time-intelligence measure (`Spend MTD`, `Spend PY`, etc.) — see [DAX Patterns.md §7](./15_DAX_Patterns.md) |
| Grain mismatch awareness | Confirm any measure or visual that appears to join `FactUtilization` and `FactRiskProfile` is not silently assuming equivalent grain — see [Data Model.md §3](./14_Data_Model.md) |

## 6. Dashboard Validation

| Check | Method |
|---|---|
| Slicer synchronization | Confirm State, Card, Risk, and Date slicers apply consistently across all four report pages (NFR-02 in [Business Requirements.md](./01_Business_Requirements.md)) |
| Cross-page navigation | Confirm every navigation and drillthrough path documented in [Dashboard Guide.md](./06_Dashboard_Guide.md) resolves to the intended page/filter state |
| Visual performance | Confirm visual refresh remains interactive (sub-3-second) against the full ~150,000 combined fact rows (NFR-01) — see [Performance Optimization.md](./10_Performance_Optimization.md) |

## 7. Business / Requirements Validation

Each functional requirement in [Business Requirements.md §6](./01_Business_Requirements.md) is independently verifiable against the delivered model. The table below is the acceptance checklist referenced there.

| Requirement | Acceptance Check |
|---|---|
| FR-01 — Portfolio-wide spend, payments, exposure at any point-in-time | `Total Spend`, `Total Payments`, `Net Portfolio Exposure` respond correctly to a Date slicer selection |
| FR-02 — Current at-risk customers, latest assessment only | `Current Risk Customers` unaffected by stale/historical assessment months |
| FR-03 — Delinquency rate as % of total customer base | `Delinquency Rate %` denominator is `Total Customers`, not a filtered subset |
| FR-04 — Utilization as early-warning indicator | `Avg Utilization %` trends ahead of `Delinquency Rate %` in the historical data |
| FR-05 — Payment-to-spend ratio | `Payment to Spend Ratio` returns a sane percentage, never above what the anomaly correction allows |
| FR-08 — Synchronized filtering across pages | See Section 6, Slicer synchronization |
| FR-09 — Ratio measures degrade gracefully | See Section 2, Safe-division behavior |
| FR-10 — Data-quality fixes at transformation layer, not display | See Section 4 |

## 8. Regression Testing

> **Best Practice:** Any change to a certified measure's DAX, a relationship's cross-filter direction, or a Power Query transformation step should trigger a re-run of the relevant checks in Sections 2–5 before release, not just a visual smoke-test of the affected dashboard page. Log the change and its validation outcome in [Change Log.md](./13_Change_Log.md).

## 9. Related Documents

- [Business Requirements.md](./01_Business_Requirements.md) — source of the FR/NFR acceptance criteria
- [DAX Measures.md](./05_DAX_Measures.md)
- [DAX Patterns.md](./15_DAX_Patterns.md)
- [Data Model.md](./14_Data_Model.md)
- [Data Dictionary.md](./03_Data_Dictionary.md)
- [Change Log.md](./13_Change_Log.md)

---

## Version History

| Version | Date | Author | Change Description |
|---|---|---|---|
| 1.0 | 2025-12 | Alan Binu | Initial testing and validation reference, consolidating validation notes from across the documentation set into a single acceptance checklist |
