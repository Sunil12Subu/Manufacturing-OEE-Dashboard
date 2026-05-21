# Manufacturing OEE Dashboard — Power BI

> End-to-end Power BI solution for tracking **Overall Equipment Effectiveness (OEE)** across a multi-shift manufacturing environment. Built on a star-schema data model with 10 tables, covering availability, performance, quality, downtime analysis, and shift-level KPIs.

---

## Overview

Manufacturing loses money every minute without knowing *where* and *why*. This dashboard gives plant managers, production leads, and continuous improvement teams a single source of truth for equipment health — combining real-time KPIs, week-over-week trend tracking, and root cause drill-through in one report.

**Key questions this dashboard answers:**

- Are machines available when planned?
- Are they running at the right speed?
- Are we producing good-quality output?
- Which loss category is costing us the most time?
- Which equipment and shift needs immediate attention?

---

## Dashboard Highlights

| Section | What it shows |
|---|---|
| **OEE KPI Cards** | Availability %, Performance %, Quality %, Overall OEE — each with ▲/▼ week-over-week comparison |
| **OEE 30-Day Trend** | Daily OEE line chart to spot stability, drops, and variance patterns |
| **Six Big Loss Analysis** | Pareto + cumulative % chart of downtime loss categories |
| **Top Downtime Equipment** | Worst single downtime event: which machine, how long, and why |
| **Equipment Performance Gauge** | Target vs achieved OEE per equipment, colour-coded by threshold |
| **KPI Metric Summary** | Switchable summary panel: OEE gap, worst shift, lost units, top downtime reason, availability loss |
| **Shift Performance** | OEE by shift, flagged green (≥ 75%) or red (below target) |

---

## Tech Stack

[![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-0078D4?style=flat&logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/dax/)
[![Excel](https://img.shields.io/badge/Excel-217346?style=flat&logo=microsoft-excel&logoColor=white)](https://microsoft.com/excel)
[![Data Modelling](https://img.shields.io/badge/Star%20Schema-grey?style=flat)](https://en.wikipedia.org/wiki/Star_schema)

---

## Data Model

The model follows a **Star Schema** with one central fact table and supporting dimensions.

```
DimDate          ──┐
DimTime          ──┤
DimShift         ──┼──► FactProductionLog
DimEquipment     ──┤
DimProduct       ──┘

DimDate          ──┐
DimShift         ──┼──► FactDowntime ◄── DimDowntimeReason
DimEquipment     ──┘

DimDate          ──┐
DimEquipment     ──┼──► FactQualityDefects ◄── DimDefectType
```

### Tables

| Table | Type | Description |
|---|---|---|
| `FactProductionLog` | Fact | Planned time, run time, downtime, units produced per shift |
| `FactDowntime` | Fact | Individual downtime events with duration and reason |
| `FactQualityDefects` | Fact | Defect records by equipment, product, and defect type |
| `DimDate` | Dimension | Full date calendar with fiscal year and week attributes |
| `DimTime` | Dimension | Hour-level time dimension |
| `DimShift` | Dimension | Shift names (Morning / Afternoon / Night) |
| `DimEquipment` | Dimension | Equipment metadata |
| `DimProduct` | Dimension | Product catalogue |
| `DimDowntimeReason` | Dimension | Downtime categories (Six Big Losses) and reason descriptions |
| `DimDefectType` | Dimension | Defect classification |

---

## Project Structure

```
manufacturing-oee-dashboard/
├── OEE_Dashboard.pbix              # Power BI report file
├── data/
│   └── Manufacturing_OEE_Data.xlsx # Source data — all 10 model tables
├── dax/
│   └── OEE_DAX_Measures.md         # All DAX measures with explanations
├── docs/
│   └── OEE_Dashboard_Documentation.html  # Full KPI definitions & colour standards
└── README.md
```

---

## KPI Definitions

### Availability
Measures how often machines were available for production instead of being stopped.

```
Availability % = (Planned Time − Downtime) ÷ Planned Time
```

### Performance
Compares ideal production time (at max speed) vs. actual run time.

```
Performance % = Σ(Units × Ideal Cycle Time) ÷ (Actual Run Time in seconds)
```

### Quality
Percentage of produced units that are good and saleable.

```
Quality % = Good Units ÷ Total Units Produced
```

### Overall OEE

```
OEE = Availability × Performance × Quality
```

> **World-class OEE target: ≥ 85%**
> Dashboard threshold for colour flags: **75%** (configurable via `[OEE Target]` measure)

All DAX formulas with full variable-by-variable explanations are in [`dax/OEE_DAX_Measures.md`](dax/OEE_DAX_Measures.md).

---

## Colour Standards

| Element | Colour | Code |
|---|---|---|
| On target / positive trend | 🟢 Green | `#2ECC71` |
| Below target / negative trend | 🔴 Red | `#FF2C2C` |
| Bar and line charts | 🔵 Blue | `#2C5DBE` |

---

## Target Audience

This dashboard is designed for decision-makers with operational authority:

- **Plant Manager** — overall equipment health and strategic trend
- **Production Manager** — shift-level and equipment-level KPIs
- **Operations Head** — daily review and escalation triggers
- **Shift Supervisor** — real-time shift performance flag
- **CI / Lean Team** — Six Big Loss root cause and improvement prioritisation

---

## How to Use

1. Clone or download this repository
2. Open `OEE_Dashboard.pbix` in Power BI Desktop (June 2024 or later recommended)
3. Data is embedded from `data/Manufacturing_OEE_Data.xlsx` — update the source path if needed via **Transform Data → Data Source Settings**
4. Publish to Power BI Service for shared access and scheduled refresh
5. Refer to `docs/OEE_Dashboard_Documentation.html` for full KPI reference and DAX formula documentation

---

## Selected Insights from Sample Data

- The **Six Big Loss Pareto** reveals which loss category contributes the top 80% of cumulative downtime — enabling focused maintenance planning
- Week-over-week arrows on each KPI card surface directional changes without requiring users to build their own comparisons
- The **Worst Shift** metric in the KPI Summary panel identifies which team needs immediate support or investigation
- Equipment gauge visuals make target vs. actual gaps immediately visible at a glance

---

## Related Projects

- [IMDB Top-100 EDA](https://github.com/Sunil12Subu/Analysis-on-IMDB-dataset) — Python EDA with Pandas & Matplotlib
- [Google Play Store Case Study](https://github.com/Sunil12Subu/Case-study-on-Google-play-store-) — App ratings analysis
- [Bank Telemarketing Case Study](https://github.com/Sunil12Subu/Case-study-on-Bank-Telemarketing) — Campaign response modelling

---

*Part of a data analytics portfolio by [Sunil Subramaniam Sreedhar](https://linkedin.com/in/sunil-subramaniam-sreedhar) — Senior BI Developer & Data Analyst based in Dortmund, Germany.*
