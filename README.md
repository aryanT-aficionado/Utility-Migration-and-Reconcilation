# Utility Billing Migration & Reconciliation

A fully working simulation of a customer billing data migration between a
legacy relational database and a new billing platform. Covers data generation, profiling, migration (with realistic error injection), validation, reconciliation.

## Why Migration happens-
- Old System can't support the new business needs
    - Can't handle new product type
    - Can't ahndle new product
    - Too slow to process the query
    - Support for some of the file format stops, making it security risk
    - Databricks/BI new version supports certain type of files

## Why Profile before migrating-
- To understand data
- You cant fix what you dont measure 
- You cant validate what you cant baseline

Its like forensic evidence taken before crime scene is disturbed

1. Structural Profiling - How many rows and counts, Data types of columns, Nulls, percentage of row have values(completeness rate)
2. Semantic Profiling- 
    - Numeric values in Range(transfer cant be -500, or 99,999)
    - Logically Consistant (Dates are logically correct[Feb cant have 30 days])
    - Categorical(Fuel have CV or petrol or just related values)
    - Format Consistancy (Phone no in format/dates in format DD/MM/YYYY)
3. Relational Profiling - Relationship between tables
    Billing System Belongs => Account Belongs => Customer
    Payments are linked => Billing Cycle

## Data Reonciliation
lets say we pack 47 boxes => transfer => 45 boxes arrives (where does 2 boxes go, **Why** didnt arrive, **What** was on them, **What** to do)

- **Count Reconciliation:** did same number of records make it.(Same record exist in both system)
- **Record Reconciliation:** Which specific record missing. (Values in those records match)
- **Value-level Reconciliation:** Did the values survive the migration.(Do the total adds up)


## Key findings 
- **Billing cycle amount discrepancies:** 5,030 cycles with total deviation of €-643.91
- **Payment amount discrepancies:** 4,400 payments with total deviation of €-1,117.07
- **Balance drift:** 510 accounts with total drift of €155.08
- **Data quality issues:** 228 customers with null first names (introduced as part of migration simulation)

## What does the number tells
1. Missing records: ~2% of customers, accounts, cycles, and payments are missing from the target. This mimics real migration errors.

2. Amount discrepancies: €-643.91 total difference in billing cycles, with a max single discrepancy of €15.00. The negative sign means the target has slightly lower amounts—consistent with the deliberate balance alterations.

3. Null injection: 228 customers have missing first names (you introduced 5% null addresses; here it’s first names—still a data quality issue).

4. Balance drift: 510 accounts show balance mismatches, with a total drift of €155.08.

All date formats are clean, so your date validation logic worked perfectly.

## Technical Decisions
1. Why SQLite? Portable, no server setup, behaves like Oracle/PostgreSQL
for SQL logic. The queries translate directly to production systems.

2. Why inject errors deliberately? A migration reconciliation tool that
only works on clean data isn't useful. The whole point is finding what
broke. Error injection makes the validation logic meaningful.

3. Date format as the primary transform — this mirrors real migration
challenges. Legacy billing systems (like older Hansen Peace implementations) store dates in locale formats. New platforms standardise to ISO 8601.
A single character difference in format breaks every date comparison downstream.

4. Reconciliation at account level, not just row level — individual record
checks catch corruption. Account-level aggregation catches drop patterns
(e.g., all records for certain account IDs missing).

5. Separation of profiling → migration → validation — these are genuinely
separate phases in a real migration programme. Running profiling before
migration defines your baseline. Running validation after gives you
before/after comparison. That's the discipline this project demonstrates.

