# Data Dictionary
## Credit Card Portfolio Analytics & Risk Intelligence

| | |
|---|---|
| **Document Type** | Data Dictionary |
| **Scope** | All 9 tables in the production semantic model |
| **Version** | 1.1 |
| **Related Documents** | [Data Model.md](./14_Data_Model.md), [Architecture.md](./02_Architecture.md), [Data Sources.md](./04_Data_Sources.md), [Power Query Transformations.md](./08_Power_Query_Transformations.md) |

---

## 1. How to Use This Document

This dictionary is the column-level companion to [Data Model.md](./14_Data_Model.md), which covers grain, relationship rationale, and filter-propagation behavior at the table level. Use **Data Model.md** to understand *how* tables relate; use this document to understand *what each column means*.

> **Developer Tip:** When adding a new measure, cross-check the column's business definition here before writing the DAX — several columns (e.g., `FactRiskProfile[UtilizationPercent]` vs. `FactUtilization[UtilizationPercent]`) exist in more than one table at different grains, and using the wrong one silently changes the result.

## 2. Table Inventory

| Table | Type | Grain | Row Count |
|---|---|---|---:|
| DimCustomer | Dimension | One row per customer | 1,000 |
| DimCard | Dimension | One row per card product | 20 |
| DimMerchant | Dimension | One row per merchant | 500 |
| DimCategory | Dimension | One row per spend category | 12 |
| DimDate | Dimension | One row per calendar day | 1,096 |
| FactTransactions | Fact | One row per card transaction | 50,000 |
| FactPayments | Fact | One row per repayment event | 24,682 |
| FactUtilization | Fact | One row per customer-card monthly snapshot | 39,780 |
| FactRiskProfile | Fact | One row per customer monthly risk assessment | 36,000 |

---

## 3. DimCustomer — Customer Master

**Grain:** One row per customer. **Primary Key:** `CustomerID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| CustomerID | Integer (PK) | Unique customer identifier | 10001 |
| CustomerName | Text | Customer full name | Harini Mane |
| Gender | Text | Customer gender | Male, Female |
| Age | Integer | Customer age in years | 23–65 (typical range) |
| City | Text | Customer's city of residence | Pune, Indore |
| State | Text | Customer's state of residence | Maharashtra, Kerala, Delhi, etc. (16 states represented) |
| Occupation | Text | Stated occupation | Medical Practitioner, Software Engineer, Chartered Accountant |
| Industry | Text | Industry sector of employment | Healthcare, Technology, Finance, Banking, Education, Retail, Corporate Services, Infrastructure |
| EmploymentType | Text | Employment classification | Salaried, Self-Employed |
| Sector | Text | Public/private sector classification | Private, Government |
| MonthlySalary | Decimal | Self-reported monthly income (₹) | 129,084 |
| MaritalStatus | Text | Marital status | Single, Married |
| Dependents | Integer | Number of financial dependents | 0–4 (typical range) |
| CreditScore | Integer | Bureau credit score at onboarding | 630, 728 |
| CustomerSegment | Text | Value-tier segmentation used for retention and marketing | High Value, Mass Affluent, Premium, Standard |
| JoinDate | Date | Date the customer relationship began | 2020-08-25 |

**Business notes:** `CustomerSegment` is the primary axis used on the Customer Analytics page (see [Dashboard Guide.md](./06_Dashboard_Guide.md)); `Mass Affluent` is the largest segment at 42.3% of the base (see [KPIs & Business Metrics.md](./07_KPIs_and_Business_Metrics.md)).

> **Data Quality Note:** `CreditScore` appears twice in the model — once as an onboarding snapshot in `DimCustomer` and once per assessment month in `FactRiskProfile`. They are expected to diverge over time as a customer's bureau score changes; report authors should be explicit about which one a visual is using. See [Data Model.md §6](./14_Data_Model.md) for the full slowly-changing-attribute discussion.

---

## 4. DimCard — Card Product Catalog

**Grain:** One row per card product. **Primary Key:** `CardID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| CardID | Integer (PK) | Unique card product identifier | 200, 201 |
| CardName | Text | Commercial card product name | Swiggy HDFC, HDFC Millennia |
| BankName | Text | Issuing bank | HDFC, ICICI, SBI, Axis, Kotak, and 7 others (12 total) |
| CardCategory | Text | Product tier / category | Entry-level, Regular, Premium, Cashback, Co-branded, Customizable |
| CardNetwork | Text | Card payment network | Visa, Mastercard, RuPay, Amex |
| PrimaryBenefit | Text | Headline reward benefit of the product | Food & Dining, Cashback, Travel, Lounge Access, Shopping, Grocery, Rewards, Milestone Rewards, Entertainment, Utilities |
| CreditLimit | Integer | Standard credit limit assigned to the product (₹) | 150,000–200,000 |
| AnnualFee | Integer | Annual card fee (₹) | 500, 1,000 |
| MinAge | Integer | Minimum eligible age | 21 |
| MinCreditScore | Integer | Minimum bureau score required for issuance | 700, 720 |

**Business notes:** `CardCategory = "Entry-level"` products generate the largest single share of portfolio spend (₹89.85M) despite the lowest credit limits — see [KPIs & Business Metrics.md](./07_KPIs_and_Business_Metrics.md).

> **Implementation Note:** `DimCard[CreditLimit]` is the *product-level standard* limit; `FactUtilization[CreditLimit]` is the *customer-assigned* limit at a point in time and may differ from the product default due to individual underwriting. Measures computing portfolio-wide credit exposure should reference `FactUtilization[CreditLimit]`, not `DimCard[CreditLimit]` — see [DAX Measures.md §4.4](./05_DAX_Measures.md).

---

## 5. DimMerchant — Merchant Network

**Grain:** One row per merchant. **Primary Key:** `MerchantID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| MerchantID | Integer (PK) | Unique merchant identifier | 5001, 5002 |
| MerchantName | Text | Merchant display name | BookMyShow 46, Amazon Seller 97 |
| CategoryID | Integer (FK → DimCategory) | Spend category the merchant belongs to | 110, 102 |
| MerchantCity | Text | City where the merchant operates | Mumbai, Hyderabad |
| MerchantType | Text | Channel through which the transaction occurs | Online, POS |
| MerchantSize | Text | Relative merchant size classification | Small, Medium, Large |

---

## 6. DimCategory — Spend Category Taxonomy

**Grain:** One row per spend category. **Primary Key:** `CategoryID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| CategoryID | Integer (PK) | Unique category identifier | 100, 101 |
| CategoryName | Text | Business-facing category name | Swiggy, Zomato, and 10 others (12 total) |

---

## 7. DimDate — Calendar Dimension

**Grain:** One row per calendar day, spanning the full reporting horizon. **Primary Key:** `DateID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| DateID | Integer (PK) | Date key in YYYYMMDD format | 20230101 |
| Date | Date | Calendar date | 2023-01-01 |
| Year | Integer | Calendar year | 2023 |
| Quarter | Integer | Calendar quarter (1–4) | 1 |
| Month | Integer | Calendar month number (1–12) | 1 |
| MonthName | Text | Month name | January |
| Day | Integer | Day of month | 1 |
| DayOfWeek | Integer | Numeric day-of-week indicator | 7 |
| DayName | Text | Day name | Sunday |
| IsWeekend | Integer (Boolean flag) | 1 if the date falls on a weekend, else 0 | 1, 0 |

**Business notes:** `DimDate` spans 1,096 days (~3 calendar years), enabling year-over-year and quarter-over-quarter time intelligence across all fact tables it relates to (Transactions, Payments).

> **Best Practice:** `DimDate` is marked as the model's official Date Table in Power BI. This is a prerequisite — not an optional convenience — for the `SAMEPERIODLASTYEAR`, `DATESMTD`, and related time-intelligence functions documented in [DAX Patterns.md §7](./15_DAX_Patterns.md). Any table used for time intelligence must have a contiguous, unique, single-grain date column, which `DimDate` satisfies by design.

---

## 8. FactTransactions — Card Spend Events

**Grain:** One row per individual card transaction. **Primary Key:** `TransactionID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| TransactionID | Integer (PK) | Unique transaction identifier | 7000001 |
| DateID | Integer (FK → DimDate) | Transaction date key | 20230127 |
| CustomerID | Integer (FK → DimCustomer) | Customer who made the transaction | 10001 |
| CardID | Integer (FK → DimCard) | Card product used | 215 |
| MerchantID | Integer (FK → DimMerchant) | Merchant where the transaction occurred | 5039 |
| CategoryID | Integer (FK → DimCategory) | Spend category | 101 |
| TransactionAmount | Decimal | Transaction value (₹) | 970, 9,825 |
| TransactionType | Text | Transaction channel | Online, POS |
| EMIFlag | Integer (Boolean flag) | 1 if the transaction was converted to EMI, else 0 | 0, 1 |
| CashbackEarned | Decimal | Cashback value earned on this transaction (₹) | 9.70, 98.25 |
| RewardPoints | Integer | Reward points earned on this transaction | 0, and positive integers |

**Business notes:** `EMIFlag` drives the `EMI Transactions` and `EMI %` measures; used to gauge the share of spend converted to installment credit — a secondary risk and revenue signal (see [DAX Measures.md](./05_DAX_Measures.md)).

> **Architecture Note:** `FactTransactions` is the largest and most heavily-referenced fact table in the model (5 dimension relationships). It is the table most sensitive to relationship-direction mistakes — see the common-mistakes table in [Data Model.md §7](./14_Data_Model.md).

---

## 9. FactPayments — Repayment / Billing Events

**Grain:** One row per billing/repayment cycle event per customer-card. **Primary Key:** `PaymentID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| PaymentID | Text (PK) | Unique payment record identifier | PAY000001 |
| CustomerID | Integer (FK → DimCustomer) | Customer being billed | 10001 |
| CardID | Integer (FK → DimCard) | Card product billed | 215 |
| DateID | Integer (FK → DimDate) | Reporting date key associated with the payment record | 20230305 |
| BillAmount | Decimal | Total amount billed for the cycle (₹) | 10,795.00 |
| PaymentAmount | Decimal | Amount actually paid by the customer (₹) | 3,735.83 |
| DueDate | Date | Payment due date | 2023-02-20 |
| PaymentDate | Date | Date payment was received | 2023-03-05 |
| DaysPastDate | Integer | Days between due date and payment date | 13 |
| PaymentStatus | Text | Categorical repayment outcome for the cycle | Paid in Full, Partial Payment, Minimum Payment, Delinquent |

**Business notes:** `PaymentStatus = "Delinquent"` underpins the `Delinquent Customers` and `Delinquency Rate %` measures. Roughly 1 in 4 customers clears `PaymentAmount = BillAmount` in full every cycle; the remainder carry a running balance (see [KPIs & Business Metrics.md](./07_KPIs_and_Business_Metrics.md)). A payment-to-spend anomaly (ratio exceeding 100%) was identified and corrected during validation — see [Power Query Transformations.md](./08_Power_Query_Transformations.md).

> **Validation Note:** `DateID` in `FactPayments` represents the reporting period the payment record is associated with, not necessarily `PaymentDate` itself — the two can fall in different months for a late payment. Time-based visuals should confirm which date field they intend to slice on. See [Testing & Validation.md §3](./17_Testing_Validation.md).

---

## 10. FactUtilization — Monthly Credit Utilization Snapshot

**Grain:** One row per customer-card, per calendar month. **Primary Key:** `UtilizationID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| UtilizationID | Integer (PK) | Unique snapshot identifier | 9000001 |
| CustomerID | Integer (FK → DimCustomer) | Customer being assessed | 10001 |
| CardID | Integer (FK → DimCard) | Card product assessed | 215 |
| SnapshotMonth | Text (YYYY-MM) | Month of the utilization snapshot | 2023-01 |
| CreditLimit | Integer | Assigned credit limit at time of snapshot (₹) | 220,000 |
| OutstandingBalance | Decimal | Outstanding balance at time of snapshot (₹) | 61,317 |
| UtilizationPercent | Decimal | `OutstandingBalance / CreditLimit × 100` | 27.87 |

**Business notes:** `UtilizationPercent` is the model's primary early-warning indicator — high- and critical-risk customers both run utilization near 90%, visible before delinquency shows up in `FactPayments` (see [KPIs & Business Metrics.md](./07_KPIs_and_Business_Metrics.md)).

---

## 11. FactRiskProfile — Monthly Risk Assessment

**Grain:** One row per customer, per calendar month. **Primary Key:** `RiskProfileID`.

| Column | Data Type | Description | Example Values |
|---|---|---|---|
| RiskProfileID | Integer (PK) | Unique risk assessment identifier | 4000001 |
| CustomerID | Integer (FK → DimCustomer) | Customer being assessed | 10001 |
| AssessmentMonth | Text (YYYY-MM) | Month of the risk assessment | 2023-01 |
| CreditScore | Integer | Bureau credit score at time of assessment | 717 |
| UtilizationPercent | Decimal | Utilization percentage at time of assessment | 27.87 |
| LatePaymentsLast6Months | Integer | Count of late payments in the trailing 6 months | 0 |
| RiskScore | Integer | Composite numeric risk score | 11 |
| RiskCategory | Text | Categorical risk classification (post-remediation) | Low Risk, Medium Risk, High Risk, **Critical Risk** |

**Business notes:** The source system originally labeled the highest-risk tier `"Aggressive User"`, inconsistent with the `Low/Medium/High Risk` naming convention. This is corrected to `"Critical Risk"` once, at the source, in Power Query — every downstream visual and measure inherits the corrected label automatically (see [Power Query Transformations.md](./08_Power_Query_Transformations.md)). `FactRiskProfile → DimCustomer` is the model's single bidirectional relationship (see [Architecture.md](./02_Architecture.md)).

> **Known Constraint:** `FactRiskProfile[UtilizationPercent]` is duplicated conceptually from `FactUtilization[UtilizationPercent]` at the same customer-card-month grain, but sourced independently by the upstream risk-scoring process. Minor discrepancies between the two are expected and should not be treated as a data-quality defect unless the divergence is materially large — see [Testing & Validation.md §5](./17_Testing_Validation.md).

---

## 12. Calculation Table (DAX Measures)

A disconnected table (not related to any dimension or fact table) hosting all 33 certified business measures. Full detail in [DAX Measures.md](./05_DAX_Measures.md) and pattern-level explanation in [DAX Patterns.md](./15_DAX_Patterns.md).

---

## 13. Cross-Table Key Summary

| Key | Appears In (as PK) | Appears In (as FK) |
|---|---|---|
| CustomerID | DimCustomer | FactTransactions, FactPayments, FactUtilization, FactRiskProfile |
| CardID | DimCard | FactTransactions, FactPayments, FactUtilization |
| MerchantID | DimMerchant | FactTransactions |
| CategoryID | DimCategory | FactTransactions, DimMerchant |
| DateID | DimDate | FactTransactions, FactPayments |

## 14. Data Dictionary Maintenance

> **Best Practice:** Any schema change (new column, renamed field, changed data type) in the source extracts must be reflected in this document and in [Data Model.md](./14_Data_Model.md) in the same change cycle. Treat this dictionary as a contract between the data engineering team producing the extracts and the BI team consuming them — see [Change Log.md](./13_Change_Log.md) for how schema changes are tracked.

---

## Related Documents

- [Data Model.md](./14_Data_Model.md)
- [Architecture.md](./02_Architecture.md)
- [Data Sources.md](./04_Data_Sources.md)
- [Power Query Transformations.md](./08_Power_Query_Transformations.md)
- [DAX Measures.md](./05_DAX_Measures.md)
- [Testing & Validation.md](./17_Testing_Validation.md)

---

## Version History

| Version | Date | Author | Change Description |
|---|---|---|---|
| 1.0 | 2025-12 | Alan Binu | Initial data dictionary covering all 9 production tables |
| 1.1 | 2025-12 | Alan Binu | Added implementation notes, known constraints, and cross-references to the new Data Model and Testing & Validation documents |
