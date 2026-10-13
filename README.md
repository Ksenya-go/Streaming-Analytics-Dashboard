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



## How to Open
1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).
2. Open `streaming-analytics-dashboard.pbix`.
3. In *Transform data → Data source settings*, point the source to the Excel file in the `data/` folder.
