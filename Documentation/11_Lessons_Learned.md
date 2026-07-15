# Lessons Learned
## Credit Card Portfolio Analytics & Risk Intelligence

| | |
|---|---|
| **Document Type** | Project Retrospective |
| **Version** | 1.0 |
| **Related Documents** | [Power Query Transformations.md](./08_Power_Query_Transformations.md), [Architecture.md](./02_Architecture.md), [Project Roadmap.md](./12_Project_Roadmap.md) |

---

## 1. Purpose

This document captures the technical and process lessons learned during the design and build of this solution — what worked well, what was harder than expected, and what would be approached differently in a production engagement. It is written for the same audience as the rest of this documentation set: a hiring manager, BI architect, or Power BI developer evaluating the depth of thinking behind the delivered solution, not just the finished artifact.

---

## 2. What Worked Well

| Practice | Outcome |
|---|---|
| Designing the star schema before writing any DAX | Prevented an entire class of measure-duplication and inconsistent-filter-direction problems that typically surface late in a project when the model is retrofitted around visuals |
| Centralizing all 33 measures in a single calculation table | Made the metric layer auditable at a glance — any reviewer can open one table and see every certified business definition, rather than hunting through nine physical tables |
| Validating measures against known business plausibility (e.g., "can a payment-to-spend ratio exceed 100%?") | Caught a real data-quality defect (see [Power Query Transformations.md, Section 5.2](./08_Power_Query_Transformations.md)) before it reached a dashboard, rather than after a stakeholder questioned an implausible KPI |
| Fixing data-quality issues at the Power Query layer, not in DAX or visuals | Ensured the fix propagated automatically to every downstream consumer, and kept the DAX layer focused purely on business-logic aggregation rather than data cleansing |
| Scoping the one bidirectional relationship narrowly (`FactRiskProfile ↔ DimCustomer`) | Preserved predictable filter behavior across the rest of the model while still enabling the one interactive use case (customer-level risk segmentation) that required it |

## 3. What Was Harder Than Expected

| Challenge | Detail | Resolution |
|---|---|---|
| Point-in-time risk logic | A naive `DISTINCTCOUNT` over `FactRiskProfile[CustomerID]` counts the same customer multiple times across different assessment months, overstating "current" risk | Solved with the `VAR LatestMonth` + `REMOVEFILTERS(DimRiskCategory)` pattern in `Current Risk Customers` — see [DAX Measures.md, Section 5.1](./05_DAX_Measures.md) |
| Inconsistent categorical labeling in source data | The `"Aggressive User"` label in `FactRiskProfile[RiskCategory]` was inconsistent with the rest of the risk taxonomy and would have been visible directly to executive audiences if left uncorrected | Corrected once, at the source, in Power Query rather than patched per-visual — see [Power Query Transformations.md, Section 5.1](./08_Power_Query_Transformations.md) |
| Determining where to draw the bidirectional-relationship line | Making the entire model bidirectional would have simplified some early visual design but would have made filter propagation unpredictable at scale | Restricted bidirectional filtering to exactly one relationship, chosen because it was the one case where the business need (customer-level slicing of risk data) outweighed the cost to predictability |
| Keeping ratio measures safe under filtered/empty context | Several early KPI cards displayed blank or error states when a slicer combination returned no rows | Standardized every ratio measure on `DIVIDE(..., 0)` as a non-negotiable convention across the entire measure layer |

## 4. What Would Be Done Differently in a Production Engagement

| Area | Portfolio-Project Approach | Production Approach |
|---|---|---|
| Source connectivity | Static local file extracts | Direct, governed connections to source systems (core banking, risk engine, billing) via Azure SQL or Microsoft Fabric — see [Project Roadmap.md](./12_Project_Roadmap.md) |
| File paths | Hardcoded to a local development machine | Parameterized `SourceFolderPath` from day one — see [Technical Design.md, Section 6](./09_Technical_Design.md) |
| Security | No Row-Level Security | RLS scoped by region/state and by business-unit ownership, enforced before any stakeholder access is granted |
| Refresh | Manual, on-demand | Scheduled refresh via a data gateway, with incremental refresh on the largest fact tables |
| Data quality | Two defects identified manually during measure validation | Automated data-quality profiling (e.g., Power Query column-quality diagnostics, or a dedicated validation pipeline) run on every refresh, not just at initial build time |
| Risk scoring | Risk category and score are sourced as pre-calculated fields | A production engagement would validate the upstream risk-scoring methodology itself, and likely integrate a predictive model rather than treating `RiskScore`/`RiskCategory` as a black-box input |

## 5. Key Technical Takeaways

1. **A star schema is a discipline, not just a diagram.** The value came from consistently enforcing single-direction filtering as the default and treating every exception as a deliberate, documented decision — not from the shape of the model alone.
2. **Data-quality validation is part of measure design, not a separate QA step.** The payment-to-spend anomaly was only caught because the measure's *output* was checked against business plausibility, not because the input data was inspected in isolation.
3. **"Current" is a modeling decision, not a given.** Any metric describing present-state risk requires an explicit choice about how to handle historical assessments — the default `DISTINCTCOUNT` behavior in DAX will silently blend history unless a latest-period pattern is deliberately applied.
4. **Centralizing measures pays for itself immediately, even on a single-developer project.** With 33 measures, the ability to find, audit, and reuse a definition from one table — rather than re-deriving it per report page — was the single biggest driver of build speed after the initial model was in place.

## 6. Process Takeaways

- Documenting architectural decisions *as they were made* (e.g., why the bidirectional relationship was scoped narrowly) made this documentation set significantly easier to produce after the fact — a lesson to carry into future projects: capture rationale at decision time, not retroactively.
- Building four audience-specific pages, rather than one dense "everything" dashboard, forced clearer thinking about which KPIs actually belong to which business question — a discipline worth applying earlier in future projects, before any visual is built.

---

## Related Documents

- [Power Query Transformations.md](./08_Power_Query_Transformations.md)
- [Architecture.md](./02_Architecture.md)
- [Technical Design.md](./09_Technical_Design.md)
- [Project Roadmap.md](./12_Project_Roadmap.md)

---

## Version History

| Version | Date | Author | Change Description |
|---|---|---|---|
| 1.0 | 2025-12 | Alan Binu | Initial project retrospective |
