# Change Log
## Credit Card Portfolio Analytics & Risk Intelligence

| | |
|---|---|
| **Document Type** | Project & Documentation Change Log |
| **Version** | 1.1 |
| **Related Documents** | All documents in the `Documentation/` folder |

---

## 1. Purpose

This change log tracks the version history of the Power BI solution and its accompanying enterprise documentation set. It follows [Semantic Versioning](https://semver.org/) conventions (`MAJOR.MINOR.PATCH`) adapted for a BI deliverable: **MAJOR** for architectural/model changes, **MINOR** for new dashboards, measures, or documentation, **PATCH** for corrections and clarifications.

---

## 2. Solution Change Log

| Version | Date | Change Type | Description |
|---|---|---|---|
| 1.0.0 | 2025-12 | Initial Release | Star-schema semantic model (5 dimensions, 4 facts, 11 relationships), 33 centralized DAX measures, 4 audience-specific report pages (Executive Overview, Spend Analytics, Risk Analytics, Customer Analytics) |
| 1.0.1 | 2025-12 | Data Quality Fix | Corrected `RiskCategory` label `"Aggressive User"` → `"Critical Risk"` at the Power Query source layer — see [Power Query Transformations.md, Section 5.1](./08_Power_Query_Transformations.md) |
| 1.0.2 | 2025-12 | Data Quality Fix | Traced and corrected a payment-to-spend ratio anomaly (values exceeding 100%) in `FactPayments` at the source — see [Power Query Transformations.md, Section 5.2](./08_Power_Query_Transformations.md) |

## 3. Documentation Change Log

| Version | Date | Document(s) | Change Type | Description |
|---|---|---|---|---|
| 1.0 → 1.1 | 2025-12 | Business Requirements.md | Minor Revision | Restructured with executive summary framing; added requirements traceability diagram, governance notes, and cross-references to the expanded documentation set |
| 1.0 → 1.1 | 2025-12 | Architecture.md | Minor Revision | Added alternatives-considered analysis, future architecture risks, and cross-references to the new Data Model and Data Lineage documents |
| 1.0 → 1.1 | 2025-12 | Data Dictionary.md | Minor Revision | Added implementation notes, known constraints, and cross-references to the new Data Model and Testing & Validation documents |
| 1.0 → 1.1 | 2025-12 | Data Sources.md | Minor Revision | Added refresh-flow diagram, source validation checklist, and cross-references to the new Data Lineage and Deployment Guide documents |
| 1.0 → 2.0 → 2.1 | 2025-12 | DAX Measures.md | Major Revision | Expanded to document all 33 measures individually with formula, explanation, business interpretation, dashboard usage, dependencies, and performance notes; labeled Repository-Verified vs. Inferred Implementation; later added enterprise recommendations, validation notes, and a cross-reference to the new DAX Patterns document |
| 1.0 → 2.0 → 2.1 | 2025-12 | Dashboard Guide.md | Major Revision | Expanded from page-level summaries to full per-page documentation (visuals, filters, navigation, drillthrough, business questions, insights, workflow, recommended actions); later added a report interaction lifecycle diagram and consumer/developer notes |
| 1.0 → 1.1 | 2025-12 | KPIs & Business Metrics.md | Minor Revision | Added a KPI relationship map, enterprise recommendations, and operational considerations |
| 1.0 → 1.1 | 2025-12 | Power Query Transformations.md | Minor Revision | Added a Power Query workflow diagram, engineering notes, best practices, and a warning on the exact-match limitation of the current remediation step |
| 1.0 → 1.1 | 2025-12 | Technical Design.md | Minor Revision | Expanded the storage-mode decision into a full alternatives-and-trade-offs analysis; added a security-posture interaction note for the planned RLS rollout; cross-referenced the new enterprise documents |
| 1.0 → 1.1 | 2025-12 | Performance Optimization.md | Minor Revision | Restructured around an Objectives/Philosophy/Engine Considerations framing; added Formula Engine vs. Storage Engine analysis, a diagnostic workflow diagram, and future risks |
| 1.0 | 2025-12 | Lessons Learned.md | Initial Release | Project retrospective covering what worked, what was harder than expected, and production-readiness gaps |
| 1.0 | 2025-12 | Project Roadmap.md | Initial Release | Near-term, mid-term, and long-term roadmap sequenced and traced to documented architectural gaps |
| 1.0 | 2025-12 | Change Log.md | Initial Release | This document established as the authoritative version history for the solution and documentation set |
| 1.0 | 2025-12 | Data Model.md | Initial Release | New document: grain reference, relationship inventory, filter-propagation rules, and slowly-changing-attribute notes, factored out of Architecture.md and Data Dictionary.md |
| 1.0 | 2025-12 | DAX Patterns.md | Initial Release | New document: reusable DAX design patterns (safe division, VAR staging, REMOVEFILTERS, context transition, measure reuse, time intelligence) factored out of DAX Measures.md |
| 1.0 | 2025-12 | Data Lineage.md | Initial Release | New document: end-to-end source-to-dashboard lineage with worked examples for a certified KPI and a data-quality correction |
| 1.0 | 2025-12 | Testing & Validation.md | Initial Release | New document: consolidated acceptance checklist covering measure, relationship, data-quality, grain, and business-requirement validation |
| 1.0 | 2025-12 | Deployment Guide.md | Initial Release | New document: first-time setup, refresh, target-state Power BI Service deployment, and troubleshooting |
| 1.0 | 2025-12 | Project Structure.md | Initial Release | New document: exhaustive repository file/folder inventory and documentation cross-reference map |

## 4. Versioning Conventions

| Component | Convention |
|---|---|
| Power BI solution (`.pbix`) | `MAJOR.MINOR.PATCH` — MAJOR for model/architecture changes, MINOR for new dashboards or measures, PATCH for data-quality or bug fixes |
| Documentation set | `MAJOR.MINOR` per document — MAJOR for structural rewrites, MINOR for content additions/corrections within the existing structure |

## 5. Change Classification Reference

| Change Type | Definition |
|---|---|
| **Initial Release** | First published version of the item |
| **Major Revision** | Structural change to scope, depth, or format |
| **Minor Revision** | Content addition without structural change |
| **Data Quality Fix** | Correction to a data defect at the source or transformation layer |
| **Patch** | Clarification, typo correction, or cross-reference fix |

---

## Related Documents

This Change Log cross-references every document in the `Documentation/` folder:

- [Business Requirements.md](./01_Business_Requirements.md)
- [Architecture.md](./02_Architecture.md)
- [Data Dictionary.md](./03_Data_Dictionary.md)
- [Data Sources.md](./04_Data_Sources.md)
- [DAX Measures.md](./05_DAX_Measures.md)
- [Dashboard Guide.md](./06_Dashboard_Guide.md)
- [KPIs & Business Metrics.md](./07_KPIs_and_Business_Metrics.md)
- [Power Query Transformations.md](./08_Power_Query_Transformations.md)
- [Technical Design.md](./09_Technical_Design.md)
- [Performance Optimization.md](./10_Performance_Optimization.md)
- [Lessons Learned.md](./11_Lessons_Learned.md)
- [Project Roadmap.md](./12_Project_Roadmap.md)
- [Data Model.md](./14_Data_Model.md)
- [DAX Patterns.md](./15_DAX_Patterns.md)
- [Data Lineage.md](./16_Data_Lineage.md)
- [Testing & Validation.md](./17_Testing_Validation.md)
- [Deployment Guide.md](./18_Deployment_Guide.md)
- [Project Structure.md](./19_Project_Structure.md)

---

## Version History (of this Change Log)

| Version | Date | Author | Change Description |
|---|---|---|---|
| 1.0 | 2025-12 | Alan Binu | Initial Change Log establishing solution and documentation version history |
| 1.1 | 2025-12 | Alan Binu | Reconciled the Documentation Change Log against each document's own Version History (added the 1.1/2.0/2.1 revisions that had gone unrecorded here), and added entries for the six new documents (Data Model, DAX Patterns, Data Lineage, Testing & Validation, Deployment Guide, Project Structure) |
