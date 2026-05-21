# OEE DAX Measures Reference

All calculated measures used in the Manufacturing OEE Power BI Dashboard.

---

## Core KPI Measures

### Availability %
> When we planned to run the machine, how much time did it ACTUALLY run?

```dax
Availability % =
VAR TotalPlannedTime = SUM(FactProductionLog[PlannedProductionTime_Minutes])
VAR TotalDowntime    = SUM(FactProductionLog[DownTime_Minutes])
VAR TotalRunTime     = TotalPlannedTime - TotalDowntime
VAR Availability     = IF(
    TotalPlannedTime = 0,
    BLANK(),
    DIVIDE(TotalRunTime, TotalPlannedTime, 0)
)
RETURN Availability
```

---

### Performance %
> Compares how much time production should have taken at ideal speed vs. actual time.

```dax
Performance % =
VAR TotalRunTime  = SUM(FactProductionLog[ActualRunTime_Minutes]) * 60
VAR TotalUnits    = SUM(FactProductionLog[TotalUnitsProduced])
VAR IdealCycleTime = SUMX(
    FactProductionLog,
    FactProductionLog[TotalUnitsProduced] * FactProductionLog[IdealCycleTime_Seconds]
)
VAR Performance = IF(
    TotalRunTime = 0,
    BLANK(),
    DIVIDE(IdealCycleTime, TotalRunTime, 0)
)
RETURN Performance
```

---

### Quality %
> Percentage of produced units that are good and saleable.

```dax
Quality % =
VAR TotalUnits = SUM(FactProductionLog[TotalUnitsProduced])
VAR GoodUnits  = SUM(FactProductionLog[GoodUnitsProduced])
VAR Quality    = IF(
    TotalUnits = 0,
    BLANK(),
    DIVIDE(GoodUnits, TotalUnits, 0)
)
RETURN Quality
```

---

### Overall OEE
> Single composite metric: Availability × Performance × Quality.

```dax
Overall OEE =
VAR Availability = [Availability %]
VAR Performance  = [Performance %]
VAR Quality      = [Quality %]
VAR OEE          = Availability * Performance * Quality
RETURN IF(ISBLANK(OEE), BLANK(), OEE)
```

---

## Week-over-Week Comparison Measures

### OEE vs Last Week

```dax
OEE vs Last Week =
VAR CurrentOEE  = [Overall OEE]
VAR LastWeekOEE = CALCULATE(
    [Overall OEE],
    DATEADD(DimDate[FullDate], -7, DAY)
)
VAR Variance_ = CurrentOEE - LastWeekOEE
RETURN IF(ISBLANK(Variance_), BLANK(), Variance_)
```

### OEE vs Last Week Text (with arrow indicator)

```dax
OEE vs Last Week Text =
VAR Variance_   = [OEE vs Last Week]
VAR VariancePP  = Variance_ * 100
VAR Arrow       = IF(Variance_ > 0, UNICHAR(9650), UNICHAR(9660))
VAR Sign        = IF(Variance_ > 0, "+", "")
RETURN IF(
    ISBLANK(Variance_),
    "No comparison data",
    Arrow & " " & Sign & FORMAT(VariancePP, "0.00") & " pp vs Last Week"
)
```

### OEE vs Last Week Color

```dax
OEE vs Last Week color = IF([OEE vs Last Week] > 0, 1, 0)
```

> Same pattern applies for **Availability vs Last Week**, **Performance vs Last Week**, and **Quality vs Last Week** — replace the base measure reference accordingly.

---

## Six Big Loss Analysis

### Total Downtime Minutes

```dax
Total Downtime Minutes = SUM(FactDowntime[DurationMinutes])
```

### Cumulative Downtime %

```dax
Cumulative Downtime % =
VAR CurrentCategory = MAX(DimDowntimeReason[LossCategory])
VAR CurrentDowntime = [Total Downtime Minutes]
VAR AllCategories   = ADDCOLUMNS(
    SUMMARIZE(ALLSELECTED(FactDowntime), DimDowntimeReason[LossCategory]),
    "@Downtime", [Total Downtime Minutes]
)
VAR RankedCategories = ADDCOLUMNS(
    AllCategories,
    "@Rank", RANKX(AllCategories, [@Downtime], , DESC, DENSE)
)
VAR CurrentRank = MAXX(
    FILTER(RankedCategories, [LossCategory] = CurrentCategory),
    [@Rank]
)
VAR CumulativeSum = SUMX(
    FILTER(RankedCategories, [@Rank] <= CurrentRank),
    [@Downtime]
)
VAR TotalDowntime = SUMX(AllCategories, [@Downtime])
RETURN DIVIDE(CumulativeSum, TotalDowntime, 0)
```

---

## Top Downtime Equipment

### Max Downtime (hrs)

```dax
Max Downtime (hrs) = MAX(FactDowntime[DurationHours])
```

### Reason for Max Downtime

```dax
Reason for Max Downtime =
VAR MaxEvent = TOPN(1, FactDowntime, FactDowntime[DurationHours], DESC)
RETURN MAXX(MaxEvent, RELATED(DimDowntimeReason[ReasonDescription]))
```

---

## Equipment Performance Gauge Measures

```dax
Get_min_of_oee = MIN([Overall OEE], [OEE Target])

target_not_met_oee = IF([Overall OEE] < [OEE Target], [OEE Target] - [Overall OEE])

Achieved_oee = IF([Overall OEE] > [OEE Target], MAX([Overall OEE], [OEE Target]) - [Get_min_of_oee])
```

---

## KPI Metric Summary Measures

### KPI Selector Table

```dax
KPI Selector = DATATABLE(
    "Metric", STRING,
    {
        {"OEE Gap vs Target"},
        {"Worst Shift"},
        {"Lost Units"},
        {"Top Downtime Reason"},
        {"Availability Loss (hrs)"}
    }
)
```

### OEE Gap vs Target

```dax
OEE Gap (pp) = [Overall OEE] - [OEE Target]
```

### Worst Shift

```dax
Worst Shift =
VAR T = TOPN(1, VALUES(DimShift[ShiftName]), [Overall OEE], ASC)
RETURN MAXX(T, DimShift[ShiftName])
```

### Lost Units

```dax
Lost Units =
DIVIDE(
    SUM(FactProductionLog[DownTime_Minutes]) * 60,
    AVERAGE(FactProductionLog[IdealCycleTime_Seconds])
)
```

### Top Downtime Reason

```dax
Top Downtime Reason =
VAR TopEvent = TOPN(1, FactDowntime, FactDowntime[DurationHours], DESC)
RETURN MAXX(TopEvent, RELATED(DimDowntimeReason[ReasonDescription]))
```

### Availability Loss (hrs)

```dax
Availability Loss (hrs) = SUM(FactProductionLog[DownTime_Minutes]) / 60
```

---

## Shift Performance Colour Flag

```dax
overall_oee_75_color = IF([Overall OEE] >= 0.75, 1, 0)
```

> **Colour standard:** Green (#2ECC71) when OEE ≥ 75%, Red (#FF2C2C) below target.
