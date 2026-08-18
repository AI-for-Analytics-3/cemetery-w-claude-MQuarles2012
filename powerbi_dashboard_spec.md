# Power BI Dashboard Spec — Mortality in Nashville's City Cemetery, 1846–1979

## Data source

Import `nashville_cemetery_powerbi_export.csv` as the single table backing this dashboard. It is the fully cleaned dataset from the analysis notebook (19,745 rows), with these columns already built in so no re-cleaning is needed in Power BI:

| Column | Purpose |
|---|---|
| `Cause of Death Updated` | Standardized cause of death (use this, not the raw `Cause of Death/Burial`) |
| `decade` | Text like `1840's`, sorts correctly alphabetically |
| `age group` | child / minors / young adults / middle aged / older adults / unknown |
| `age group order` | 1–6 numeric helper; set **Sort by Column** on `age group` to this, or Power BI will alphabetize it (child, middle aged, minors...) instead of by age |
| `is_spike_year` | True for 1849, 1850, 1864, 1865, 1866, 1873, 1880 — the four mortality spikes identified in the analysis |

**Before building visuals:** on `age group`, right-click the column → Sort by Column → `age group order`.

## Suggested measures (DAX)

```
Total Burials = COUNTROWS('nashville_cemetery_powerbi_export')

Known-Cause Burials =
CALCULATE([Total Burials], 'nashville_cemetery_powerbi_export'[Cause of Death Updated] <> "Unknown")

% of Known-Cause Burials =
DIVIDE([Total Burials], [Known-Cause Burials])

Spike Year Burials =
CALCULATE([Total Burials], 'nashville_cemetery_powerbi_export'[is_spike_year] = TRUE)

% Age Unknown =
DIVIDE(
    CALCULATE([Total Burials], 'nashville_cemetery_powerbi_export'[age group] = "unknown"),
    [Total Burials]
)
```

## Page 1 — Overview

- **KPI cards**: Total Burials · Date range (min/max `Burial Year`) · Leading cause of death (top of `Cause of Death Updated`, excluding Unknown)
- **Line chart**: `Burial Year` (axis) × count of burials (value). Add `is_spike_year` as a legend or conditional-format rule to highlight the four spike years directly on the line.
- **Bar chart**: Top 10 `Cause of Death Updated` by count, excluding "Unknown" — filter the visual to exclude Unknown rather than filtering the page.

## Page 2 — Age & Demographics

- **Bar chart**: count of burials by `age group` (sorted via `age group order`). Color the `unknown` bar gray/muted to visually separate it from the five real brackets, matching the distinction made in the analysis.
- **100% stacked bar chart**: `decade` (axis) × `age group` (stack, legend), values as percentage of total. This reproduces the age-composition-by-decade view from the analysis and is the best single chart for showing how the population buried here shifted over time.
- **Slicer**: `Sex`, `Race` — lets the historical society filter demographics interactively.

## Page 3 — Civil War Deep Dive

- **100% stacked bar chart**: `Burial Year` filtered to 1860–1869 only (axis) × `age group` (stack). This is the year-by-year version — the chart that actually shows the compositional shift between 1864 (child-heavy) and 1866 (adult-heavy).
- **Card/callout**: annotate 1864 and 1866 directly with their child-share percentages (55.6% and 35.4%) — Power BI doesn't auto-annotate, so this is either a text box positioned on the canvas or a reference line.
- **Table**: raw counts by year and age group for 1860–1869, for anyone who wants the underlying numbers rather than the visual.

## Page 4 — Cause of Death Explorer

- **Treemap or matrix**: `Cause of Death Updated` sized/valued by count, drillable.
- **Slicers**: `decade`, `age group`, `Sex` — lets a viewer ask their own questions (e.g., "what did children die of in the 1850's?") rather than being limited to the four spike/age findings already surfaced.
- Exclude "Unknown" by default via a page-level filter, with a toggle/bookmark to bring it back in if someone wants to see the data-completeness picture too.

## Notes

- `Age` is missing for close to half the dataset even after the infant/stillborn reclassification described in the notebook — any age-based visual should either show the `unknown` bucket alongside the real ones (as done above) or explicitly caption that it covers only the known-age subset, to avoid overstating precision the source data doesn't have.
- The four spike years are a small share of records; if slicing by `is_spike_year`, expect thin bars for individual causes and consider a minimum-count filter to avoid every one-off cause of death cluttering the visual.
