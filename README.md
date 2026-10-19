# Berka Bank SQL Analysis

SQL analysis of a real, anonymised Czech bank dataset (PKDD'99 "Berka" dataset). Ten business questions on transactions, loans, cards and customer segments, answered in PostgreSQL, with a one-page Tableau Public summary and a customer-targeting recommendation.
---

## 1. Business context

A retail bank wants to understand how its customers use their accounts, where its lending risk sits, and which customers are the best candidates for a card offer. The data is a snapshot of one bank's clients, accounts, transactions, loans and cards from 1993 to 1998.

**Questions this project answers**

1. How have transaction volume and value changed over time?
2. Where does customer spend go, by transaction type and category?
3. How concentrated is activity among the biggest accounts?
4. How do balances differ across districts?
5. What drives loan default risk (size, duration)?
6. Do card holders behave differently from non-card holders?
7. How do age and gender relate to loan uptake and repayment?
8. Which accounts show unusual balance drops?
9. What do running balance trends look like by customer segment?
10. Which customer segment should the bank target for a card offer?

## 2. Dataset

- **Source:** Berka dataset (PKDD'99 Discovery Challenge), prepared by Petr Berka and Marta Sochorova. A real, anonymised dataset from a Czech bank.
- **Where to get it:** https://www.kaggle.com/datasets/marceloventura/the-berka-dataset.
- **Size:** 8 tables, about 1.09 million rows in total.
- **Period:** accounts opened 1993-01-01 to 1997-12-29; transactions from 1993-01-01 to 1998-12-31; loans from 1993-07-05 to 1998-12-08; cards issued from 1993-11-07 to 1998-12-29.
- **Currency:** Czech koruna (CZK).

## 3. Tools

| Tool | Used for |
|---|---|
| PostgreSQL | Database, schema, all analysis queries |
| Tableau Public | One-page summary dashboard |
| GitHub | Version control and publishing |

## 4. Repository structure

```
berka-bank-sql-analysis/
├── README.md
├── schema/
│   ├── 01_create_tables.sql
│   ├── 02_clean_views_and_profiling.sql
│   └── er_diagram.png
├── queries/
│   ├── q01_....sql
│   └── ... (one file per business question)
├── outputs/           CSV exports used by Tableau
└── dashboard/         screenshot and Tableau Public link
```

## 5. Schema



| Table | Rows | Purpose | Keys |
|---|---|---|---|
| `district` | 77 | Demographics and economic indicators per district | PK `district_id` |
| `account` | 4,500 | Account details: statement frequency, opening date | PK `account_id`; FK `district_id` |
| `client` | 5,369 | Client birth number (encodes birth date and gender) and district | PK `client_id`; FK `district_id` |
| `disp` | 5,369 | Links clients to accounts as owner or disponent | PK `disp_id`; FKs `client_id`, `account_id` |
| `card` | 892 | Payment cards issued, by type | PK `card_id`; FK `disp_id` |
| `loan` | 682 | Loans granted: amount, duration, monthly payment, status | PK `loan_id`; FK `account_id` |
| `permanent_order` | 6,471 | Standing payment orders | PK `order_id`; FK `account_id` |
| `trans` | 1,056,320 | Every transaction on every account | PK `trans_id`; FK `account_id` |

**Relationships in plain words:** a district has many accounts and clients. An account has one or more `disp` rows linking it to clients. A card is issued to a `disp` row. An account has at most one loan, many standing orders and many transactions.

**Table notes**

- The original `order` table is named `permanent_order` because `order` is a reserved word in SQL.
- The district columns `A1` to `A16` in the source files are renamed to readable names in `01_create_tables.sql`.

## 6. Data preparation and quality notes

Raw tables are loaded as-is, and a set of `*_clean` views handles all conversions once, so the analysis queries stay readable and the raw data is never modified.

**What the clean views do**

- Convert `YYMMDD` text dates into real dates (`TO_DATE('19' || date, 'YYYYMMDD')`). All data falls in 1993 to 1998.
- Translate Czech codes into English labels (transaction operations, categories, statement frequency, loan status).
- Derive client gender and birth date from `birth_number`. For women, 50 is added to the birth month.
- Derive client age as of 1 January 1999, just after the last transaction in the data.

**Data quality findings**

- **Referential integrity is clean.** Orphan checks on transactions, loans, dispositions and cards all returned zero problems.
- **Blank `operation`:** 183,114 transactions (about 17%). All of them are interest postings (`k_symbol = 'UROK'`), so these are not missing values. They appear as `NULL` in `operation_label`.
- **Counterparty fields:** `bank` and `account` in `trans` are empty for 782,812 rows (about 74%), 760,931 as NULL and 21,881 as empty strings. They only apply to transfers and are left out of `trans_clean`.
- **Blank `k_symbol`:** 535,314 transactions have a blank or single-space category and are treated as uncategorised.
- **`?` values:** one district (Jesenik) has `?` in the 1995 unemployment rate and 1995 crimes columns. These were set to NULL.
- **Negative balances:** 2,999 transaction rows (about 0.3%) across 288 accounts (about 6.4%) show a negative balance.
- **Ages:** clients range from 11 to 87 years old as of 1 January 1999 (2,724 men, 2,645 women), so some clients are minors.
- **Account growth:** accounts were opened between 1993 and 1997 while transactions run to the end of 1998, so early-period volume is low partly because fewer accounts existed.
- **Product penetration:** 682 of 4,500 accounts (about 15%) have a loan and 892 (about 20%) have a card.

**Assumptions**

- A "bad loan" is a loan with status `B` (finished, not paid) or `D` (running, client in debt). Statuses `A` and `C` are treated as good.
- Transaction direction is credit when the raw type is `PRIJEM`, otherwise debit. All amounts are positive numbers in both directions.
- TODO add any further assumptions you make while writing the queries.

## 7. Business questions and findings

Each question has its own SQL file in `queries/`.

| # | Question | SQL skills | Key finding |
|---|---|---|---|
| 1 | How have monthly transaction volume and value changed over time? | Aggregation, date handling, window functions | TODO |
| 2 | Which transaction types and categories drive the most spend? | Grouping, CASE statements | TODO |
| 3 | What are the top 10% of accounts by transaction value, and what share of total volume do they hold? | NTILE, CTEs | TODO |
| 4 | How does average account balance differ across districts? | Multi-table joins | TODO |
| 5 | What is the loan default rate by loan size band and duration? | Conditional aggregation | TODO |
| 6 | Do accounts with a card behave differently from those without? | LEFT JOIN, comparison logic | TODO |
| 7 | How do client age and gender relate to loan uptake and repayment? | Derived columns, joins | TODO |
| 8 | Which accounts show unusual month-over-month balance drops? | LAG(), anomaly flags | TODO |
| 9 | What is the running monthly balance trend per account segment? | Running totals, window frames | TODO |
| 10 | Which customer segments combine high balance, low loan exposure and no card, making them the best targets for a card offer? | Multi-CTE segmentation | TODO |

### Highlights

TODO: Pick your 2 to 3 strongest findings. For each one, write a short paragraph and add a chart or a screenshot of the result.

## 8. Dashboard

A one-page Tableau Public summary built from the exported query results.

- **Live dashboard:** TODO paste the Tableau Public link
- **Preview:**

TODO: add `![Dashboard preview](dashboard/dashboard.png)` after you save a screenshot.

## 9. Recommendation

TODO: Answer in 3 to 5 sentences. Which segment should the bank target with a card offer, how big is it, and why? State what the numbers show and what they cannot show (for example, that this is historical data and there is no direct measure of how customers would respond).

## 10. Limitations

- The data describes one Czech bank in the 1990s, so findings are illustrative and not representative of current retail banking.
- Customer attributes are limited to what the dataset records (birth number, district). There is no income, employment or product-pricing data.
- Loan outcomes are partly right-censored: many loans (statuses `C` and `D`) were still running when the data ends.
- The default definition (`B` or `D`) is an analytical assumption.

## 11. How to reproduce

1. Install PostgreSQL and pgAdmin 4, and create a database named `berka`.
2. Run `schema/01_create_tables.sql` PART 1 to create the tables.
3. Import the eight CSV files in pgAdmin (Import/Export Data: format `csv`, header on, delimiter `;`, quote `"`, escape `"`). Import `order.csv` into `permanent_order`.
4. Run PART 2 of `01_create_tables.sql` (row-count check, cleanup, foreign keys, indexes).
5. Run `schema/02_clean_views_and_profiling.sql`.
6. Run the files in `queries/` in order.

## 12. About

Built by **Yatharth Bisaria** as part of a data and analytics portfolio.

- LinkedIn: https://www.linkedin.com/in/yatharth-bisaria
- TODO add your portfolio site link once it is live

## 13. Credits

Dataset prepared by Petr Berka and Marta Sochorova for the PKDD'99 Discovery Challenge. Used here for educational and portfolio purposes.
