# Streaming-Analytics-Dashboard

An end-to-end Power BI project built on a synthetic subscription-streaming dataset (~190K rows, 5 tables, Jan 2024 – Dec 2025). The report tracks subscription KPIs, marketing performance and content engagement, and automatically flags unusual marketing spend.

## Table of Contents
1. [Project Goal](#project-goal)
2. [Data](#data)
3. [Data Model](#data-model)
4. [DAX Measures](#dax-measures)
5. [Report Pages](#report-pages)
   - [Overview](#1-overview)
   - [Marketing Performance](#2-marketing-performance)
   - [Anomaly Detection](#3-anomaly-detection)
   - [Content & Engagement](#4-content--engagement)
6. [Key Finding: The Hidden Marketing Spend Anomaly](#key-finding-the-hidden-marketing-spend-anomaly)
7. [Tools](#tools)
8. [Repository Structure](#repository-structure)
9. [How to Open](#how-to-open)

## Project Goal
Build a realistic analytics workflow for a subscription business, from data generation to an interactive report:
- generate a synthetic dataset with built-in seasonality and a hidden marketing spend anomaly;
- design a clean data model with fact and dimension tables;
- define business KPIs (MRR, churn, CAC) as DAX measures;
- detect the hidden anomaly with a 7-day moving average and a deviation measure, instead of searching the raw data by hand.

## Data
The dataset is synthetic (no real customer data) and was loaded from Excel via Power Query.

| Table | Rows | Description |
|---|---|---|
| `Marketing_Spend` | 29,240 | Daily spend, impressions and clicks by channel and country |
| `Subscribers` | 3,000 | Subscriber profile: country, acquisition channel, plan, age group, status, churn date |
| `Transactions` | 28,346 | New subscriptions, renewals and refunds with amounts in USD |
| `Watch_Events` | 129,709 | Viewing sessions: content, minutes, device |
| `Content` | 120 | Titles with type, genre, popularity tier |

Period: 1 January 2024 – 31 December 2025. 5 marketing channels (Paid Social, Influencer, SEO, Email, Referral) and 8 countries.

## Data Model
Star-schema design with 3 fact tables and 3 dimensions:
- **Facts:** `Transactions`, `Watch_Events`, `Marketing_Spend`
- **Dimensions:** `Subscribers`, `Content`, `DateTable`
- **Helpers:** `FunnelStages` (a disconnected table that drives the funnel chart) and `_Measures` (a container table for all DAX measures)

Relationships (many-to-one, single direction):
- `Transactions[subscriber_id]` → `Subscribers[subscriber_id]`
- `Watch_Events[subscriber_id]` → `Subscribers[subscriber_id]`
- `Watch_Events[content_id]` → `Content[content_id]`
- date columns of the fact tables → `DateTable[Date]`

## DAX Measures

All measures live in the `_Measures` table.

| Measure | Definition |
|---|---|
| `Active Subscribers` | `CALCULATE(COUNTROWS(Subscribers), Subscribers[status] = "Active")` |
| `Churned Count` | `CALCULATE(COUNTROWS(Subscribers), Subscribers[status] = "Churned")` |
| `Churn Rate %` | `DIVIDE([Churned Count], COUNTROWS(Subscribers))` |
| `MRR` | `CALCULATE(SUM(Transactions[amount_usd]), Transactions[transaction_type] IN {"New Subscription", "Renewal"})` |
| `New Subscribers` | `CALCULATE(COUNTROWS(Subscribers), Transactions[transaction_type] = "New Subscription")` |
| `Total Spend` | `SUM(Marketing_Spend[spend_usd])` |
| `CAC` | `DIVIDE([Total Spend], [New Subscribers])` |
| `Spend MA7` | `AVERAGEX(DATESINPERIOD(DateTable[Date], MAX(DateTable[Date]), -7, DAY), [Total Spend])` |
| `Spend Anomaly %` | `DIVIDE([Total Spend] - [Spend MA7], [Spend MA7])` |
| `Total Impressions` / `Total Clicks` | `SUM` of the corresponding `Marketing_Spend` columns |
| `Funnel Value` | `SWITCH(SELECTEDVALUE(FunnelStages[FunnelStage]),"Impressions", [Total Impressions],"Clicks", [Total Clicks],"New Subscribers", [New Subscribers],BLANK())` |
| `Total Watch Minutes` | `SUM(Watch_Events[watch_minutes])` |
| `Avg Watch Minutes` | `DIVIDE(SUM(Watch_Events[watch_minutes]), DISTINCTCOUNT(Watch_Events[subscriber_id]))` |

## Report Pages

### 1. Overview
Headline KPIs (Active Subscribers, MRR, CAC, Churn Rate) and monthly MRR dynamics. Monthly MRR grows steadily from about $1K in January 2024 to nearly $20K in December 2025.

<img width="1101" height="552" alt="image" src="https://github.com/user-attachments/assets/ed72eb66-ca94-4cc3-bd4d-39d938251808" />

### 2. Marketing Performance
Spend by channel, the acquisition funnel (impressions → clicks → new subscribers) and CAC with new subscribers by country. Paid Social is the largest channel (about $5.9M of $11.0M total spend), followed by Influencer ($2.6M) and SEO ($1.6M).

<img width="1180" height="643" alt="image" src="https://github.com/user-attachments/assets/c66aa5ad-ad85-425e-8b70-32bb74026e3a" />

### 3. Anomaly Detection
Daily spend against its 7-day moving average, plus a table ranking days by `Spend Anomaly %` with conditional formatting. The page is filtered to **Paid Social in the USA**.

<img width="1146" height="641" alt="image" src="https://github.com/user-attachments/assets/0cf2603c-8d5b-4d01-967e-e38602ab2fd8" />

### 4. Content & Engagement
Top 10 titles by watch minutes and watch time by device: Mobile and Smart TV about 35% each, Desktop 15%, Tablet about 10%, Game Console 5%.

<img width="819" height="637" alt="image" src="https://github.com/user-attachments/assets/adca9072-4b7c-4596-acb5-90b9a1b2a356" />

## Key Finding: The Hidden Marketing Spend Anomaly

Daily Paid Social spend in the USA normally stays around $2–3K, with gradual seasonal increases in autumn and winter. The report exposes an abrupt jump starting on **10 March 2025**:

| Date | Total Spend | 7-day MA | Deviation |
|---|---|---|---|
| 11 Mar 2025 | 7,017.85 | 3,336.72 | **+110.3%** |
| 10 Mar 2025 | 5,337.19 | 2,667.68 | **+100.1%** |
| 12 Mar 2025 | 6,720.47 | 3,979.64 | +68.9% |
| 14 Mar 2025 | 8,621.74 | 5,580.53 | +54.5% |
| 13 Mar 2025 | 6,941.07 | 4,653.98 | +49.1% |

Spend more than doubled compared with the trailing weekly average within a single day, then stayed several times above its normal level. Measuring the deviation against a moving average makes the sudden jump stand out from slow seasonal growth. In a real business, such a pattern would trigger a check for budget errors, duplicated charges or an unplanned campaign.

## Tools

Power BI Desktop, DAX, Power Query, Data Modeling, Excel.

## Repository Structure

```
├── project.pbix
├── data/                  # synthetic dataset (Excel/CSV)
├── project.pdf             # PDF export of the report
└── README.md
```

## How to Open
1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).
2. Open `streaming-analytics-dashboard.pbix`.
3. In *Transform data → Data source settings*, point the source to the Excel file in the `data/` folder.
