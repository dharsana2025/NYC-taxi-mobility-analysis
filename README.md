# 🚕 NYC Taxi Mobility Analysis — Q1 2026

> End-to-end Big Data analytics pipeline analyzing 63 million NYC TLC trip records to model platform economics, operational efficiency, and revenue optimization opportunities across Uber and Lyft.

---

## 📌 Overview

NYC ride-hailing platforms face SLA breaches during peak hours, over-reliance on airport revenue zones, and shared ride matching inefficiencies. This project builds a scalable PySpark pipeline, models $2.05B in gross bookings, and delivers a 4-page interactive Power BI dashboard with actionable business recommendations.

---

## 📊 Key Results

| Metric | Value |
|---|---|
| Total Trips Processed | 63 Million |
| Gross Bookings Modelled | $2.05 Billion |
| Net Platform Revenue | $391.16 Million |
| SLA Breach Rate | 8% (~5M trips) |
| Platform Profit / Trip | $6.22 |
| Avg Passenger Spend | $26.65 |
| Avg Driver Pay | $20.42 |

---

## 🏆 Market Structure

| Provider | Market Share | Trips | SLA Breach Rate |
|---|---|---|---|
| 🟣 Uber | 71% | 45M | 9% |
| 🟠 Lyft | 29% | 18M | 6% |

> Despite a smaller fleet, **Lyft demonstrates better operational efficiency** than Uber.

---

## ⚠️ Peak Failure Windows

- **Weekday 7–9 AM** — wait times spike to 6+ minutes
- **Weekend 10 PM+** — wait times climb past 6 minutes
- Both windows drive the majority of SLA breaches despite high trip volumes

---

## 📋 Dashboard Pages

| Page | Content |
|---|---|
| 📍 Overview | Total trips, gross bookings, SLA breach %, avg wait time |
| 💰 Revenue Analysis | Platform revenue flow, top zones by revenue, booking trend |
| ⚙️ Operations & Efficiency | Wait time by hour, shared ride matching, accessibility map |
| 💡 Key Insights & Recommendations | Market structure, economic model, action plan |

---

## 💡 Recommendations

| # | Action | Expected Impact |
|---|---|---|
| 1 | Target driver incentives at 7–9 AM weekdays and 10 PM+ weekends | Reduce SLA breach from 8% |
| 2 | Fix shared ride matching algorithm | Increase revenue per route without adding trips |
| 3 | Diversify revenue beyond JFK + LaGuardia | Reduce regulatory concentration risk |
| 4 | Quantify cost of 5M SLA breached trips | Build data-driven investment case |

---

## 🔬 Methodology

1. **Data Pipeline** — PySpark ingestion of NYC TLC public dataset with quality checks and zone-level standardisation
2. **Revenue Modelling** — decomposed gross bookings into fares, fees, tips, tax, and driver pay
3. **Operational Analysis** — SLA breach rate, avg wait time by hour, shared ride match vs request ratio
4. **Demand Analysis** — zone-level trip volume, weekday vs weekend patterns, top revenue zones
5. **Visualisation** — 4-page interactive Power BI dashboard with 15+ KPIs and slicers

---

## 🛠️ Tech Stack

![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=flat&logo=apachespark&logoColor=white)
![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=flat&logo=databricks&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)

---
