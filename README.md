# Inventory Aging & Dead-Stock Analysis — Power BI

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-2C2C2C)
![Power Query](https://img.shields.io/badge/Power%20Query-376A9E)
![Excel](https://img.shields.io/badge/Excel-217346?logo=microsoftexcel&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Data](https://img.shields.io/badge/Data-Synthetic-lightgrey)

> The question every COO dreads: **how much of our cash is frozen in stock that isn't moving?**
> This Power BI report answers it — how much, where, and whether it's getting worse.

**[▶ View the live dashboard](https://app.powerbi.com/view?r=eyJrIjoiNjI0NGJjOTAtYzFkMC00M2NmLThiN2MtMjNhZjNmZWIyN2RkIiwidCI6IjQ2YWNkMjk2LTczMGQtNDVlNy1iNWQ2LTMyY2M4NzE2ZmNjYiJ9)** · [PDF report](report/Inventory_Aging_Report.pdf) · [Power BI file (.pbix)](report/Inventory_Aging_Analysis.pbix)
·![Overview][assests/Overview.png]
---

## Dashboard preview

[View the Dashboard (PDF)](assests/1-Overview.pdf)

---

## The problem

Inventory sits on the balance sheet as an asset, but stock that stops moving is really
**frozen cash** — and it piles up quietly. An operations leader is left asking: how much of
our working capital is stuck, where is it concentrated, and why do we only find out at
write-off time? The answers are usually buried across ERP reports, not sitting in one view.

## How this dashboard helps

| The COO asks… | The dashboard answers with… |
|---|---|
| How big is the problem? | A dead-stock KPI, front and centre: **₹32.3 Cr — 44% of value** stuck 90+ days. |
| Where is it stuck? | A plant × storage matrix and treemap that pinpoint the worst locations (e.g. Chennai finished goods). |
| Is it getting better or worse? | Week-over-week variance and new / consumed flags that catch aging **early**. |
| What do I clear first? | A value-vs-age scatter that surfaces the expensive, old lots — the ones that release the most cash. |

## Business questions it answers

- How much inventory value sits in each aging bucket, and how much is dead stock (90+ days)?
- Which plants and storage locations hold the most stuck stock?
- How did inventory change week over week — what came in, what cleared, what kept aging?
- Which materials tie up the most money?

## Key insights (current snapshot)

- **≈ ₹72.8 Cr** total inventory across 4 plants and ~500 stock lots (107 distinct materials).
- **₹32.3 Cr (~44%)** is dead stock older than 90 days — the headline risk.
- **Chennai – Finished Goods Store** holds the largest single dead-stock pile (~₹8.3 Cr in 90+).
- Weekly movement: **30 lots** received, **30** consumed/dispatched, **470** carried over.

## Dashboard walkthrough

**Page 1 — Overview**
KPI cards (total value, dead stock, new vs consumed lots), an aging bar chart with 90+ in red,
value by material stage, value by plant (treemap), and a value-vs-age bubble scatter with a
90-day dead-stock line.

**Page 2 — Aging detail**

![Aging detail matrix](assets/aging-detail-matrix.png)

A plant → storage-location matrix showing **Last Week / This Week / Variance** for every aging
bucket, with red/green movement icons so worsening stock is obvious at a glance.

## Data & data model

The report runs on a single combined table stacking two weekly snapshots
(`Current Week` + `Last Week`), joined at lot level. A `Stock Status` flag
(`Carried Over` / `New this Week` / `Consumed this Week`) drives the movement analysis.

### Data dictionary

| Column | Meaning | Example |
|--------|---------|---------|
| Batch Number | Stock lot / batch (SAP `CHARG`) | `0000524188` |
| Item Code | Material number (`MATNR`), ranged by type | `10004521` |
| Item Description | Material short text (`MAKTX`) | `COPPER WINDING WIRE 1.20MM ETP` |
| UOM | Base unit of measure (`MEINS`) | `KG`, `EA`, `M` |
| Item Qty. | Quantity in stock | `1,250` |
| Value in INR | Stock value | `10,65,000` |
| Plant Location | Plant code + city (`WERKS`) | `1710 - Pune (MH)` |
| Storage Location | Storage location (`LGORT`) | `RM01 - Raw Material Store` |
| Aging Days | Days since goods receipt | `18` |
| Aging Group | Aging bucket | `0-30 / 31-60 / 61-90 / 90+` |
| Category Type | Material type (`MTART`) | `ROH / HALB / FERT` |
| Category | Material group | `Copper`, `Induction Motors` |
| Week Type | Snapshot week | `Current Week / Last Week` |
| Stock Status | Movement flag | `Carried Over / New / Consumed` |

## DAX highlights

```dax
Total Value = SUM ( 'Inventory'[Value in INR] )

Dead Stock Value =
CALCULATE (
    [Total Value],
    'Inventory'[Aging Group] = "90+ Days",
    'Inventory'[Week Type]  = "Current Week"
)

Dead Stock % = DIVIDE ( [Dead Stock Value], [Current Inventory Total] )

Variance = [Value This Week] - [Value Last Week]

New Lots This Week =
CALCULATE (
    DISTINCTCOUNT ( 'Inventory'[Batch Number] ),
    'Inventory'[Stock Status] = "New this Week",
    REMOVEFILTERS ( 'Inventory'[Week Type] )
)
```

## Tools

Power BI Desktop · DAX · Power Query · Excel · Python (synthetic data generation)

## Repository structure

```
.
├── README.md
├── data/
│   ├── inventory_stock_aging_combined_1000.csv   # both weeks (load this into Power BI)
│   ├── inventory_stock_aging_500_current.csv
│   └── inventory_stock_aging_last_week_500.csv
├── report/
│   ├── Inventory_Aging_Analysis.pbix             # data model + visuals
│   └── Inventory_Aging_Report.pdf                # static export for quick viewing
├── scripts/
│   └── generate_inventory.py                     # reproducible data generator
└── assets/
    ├── overview.png
    ├── aging-detail-matrix.png
    └── scatter-dead-stock.png
```

## How to explore

1. Open the [live dashboard](https://app.powerbi.com/view?r=eyJrIjoiNjI0NGJjOTAtYzFkMC00M2NmLThiN2MtMjNhZjNmZWIyN2RkIiwidCI6IjQ2YWNkMjk2LTczMGQtNDVlNy1iNWQ2LTMyY2M4NzE2ZmNjYiJ9), or the [PDF](assests/2-with_all_essentional_Variations.pdf).
2. To inspect the model and DAX, open `report/Inventory_Aging_Analysis.pbix` in Power BI Desktop.
3. To regenerate the data, run `python scripts/generate_inventory.py`.

## What I learned

Building this end to end taught me more than any tutorial. A few honest ones: DAX filter
context is unforgiving — renaming a column and a stray page filter both silently broke my
measures until I rebuilt them with `REMOVEFILTERS`. I also had my variance logic backwards at
first (reductions in stock were flagged red), and I learned to lead a dashboard with the
insight — a dead-stock headline — instead of a wall of numbers.

## Disclaimer

All data is **synthetic** and generated for demonstration. Company, plants, material codes,
and values are fictional and do not represent any real organization.

## Author

**[Your Name]** — [LinkedIn](https://linkedin.com/in/your-handle) · [GitHub](https://github.com/your-username)

*If you found this useful, a ⭐ on the repo is appreciated.*
