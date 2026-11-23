# About the project and data source
## Global Electricity Generation — Iran in Focus

This page is for the next user: what the report is for, where the data comes from, which file maps to which table, and how to read the report.

---

## 1) Project purpose

This dashboard was built for three reasons:

1. **Power BI portfolio** — show a star-schema model, DAX measures, and a multi-page report on real energy data.
2. **Learning** — practice cleaning data, relationships, country vs region filters, and volume vs percentage share by source.
3. **Quick topic overview** — see world electricity generation, the source mix, and where Iran sits in that global picture.

This report is not a substitute for a scientific paper or an official dataset. For serious research, always go back to the original Our World in Data source.

---

## 2) Report story (how to read it)

Suggested page order:

1. **Global Electricity Dashboard** — world scale, selected country, rank, share of world, Top 10, and source mix.
2. **Country Comparison** — compare countries in one year and over a multi-year range.
3. **Energy Mix** — share and volume by source for selected countries.
4. **About / Data** — this text: purpose, source, files, limitations.

The question the report answers:

> Where is the world’s electricity generated, from which sources, and where does Iran stand?

---

## 3) Data source

The primary source is **Our World in Data — Electricity Mix**.

- Website: [Our World in Data — Electricity Mix](https://ourworldindata.org/electricity-mix)
- Generation unit in this project: **TWh** (terawatt-hours)
- Year coverage in the current files: about **1900–2025**
- The data includes both **countries** and **regions / groups** (e.g. World, Asia, OECD) and sometimes Ember/EI rows

From that page you typically get two related exports:

| Topic | Use in the model |
|---|---|
| Total electricity generation | `ElectricityTotal` table |
| Electricity generation by source | `ElectricityBySource` table |

Both can be downloaded as CSV from Our World in Data.

---

## 4) Data files in this folder

Raw file names are a bit confusing — do not rely on the file name; look at the columns and the model tables.

### `electricity-mix.csv` → `ElectricityTotal` table

| Column | Meaning |
|---|---|
| Entity | Country or region name |
| Code | Code; for countries usually 3 letters (e.g. `IRN`). For some regions blank or like `OWID_WRL` |
| Year | Year |
| Total electricity | Total electricity generation (TWh) |

About **7,871** rows in the current version.

### `electricity-mix (1).csv` → `ElectricityBySource` table

| Column | Meaning |
|---|---|
| Entity | Country or region name |
| Code | Code |
| Year | Year |
| Other renewables | Other renewables |
| Bioenergy | Bioenergy |
| Solar | Solar |
| Wind | Wind |
| Hydropower | Hydropower |
| Nuclear | Nuclear |
| Gas | Gas |
| Oil | Oil |
| Coal | Coal |

About **10,582** rows in the current version.

### Report file

For day-to-day work, use the latest corrected version (e.g. `Global Electricity Generation - Publish.pbix`). Do not reopen broken `Redesigned` / older broken copies as your starting point.

---

## 5) Data model (short)

```
DimEntity  ←── ElectricityTotal
DimEntity  ←── ElectricityBySource
DimYear    ←── ElectricityTotal
DimYear    ←── ElectricityBySource
```

- `DimEntity` and `DimYear` are dimension tables.
- The two fact tables keep total generation and generation-by-source separate.
- The `IsCountry` column on `DimEntity` separates countries from regions (3-letter code + Kosovo exception).

Note for the next user: rows like `World` or `Asia` are regional aggregates. If you sum them with countries, you double-count. For rank and Top N, use countries only.

---

## 6) Important measures

| Measure | Meaning |
|---|---|
| Generation TWh | Total generation from the Total table |
| World Generation TWh | Generation for the World row in the same years |
| Global Share % | Share of World |
| Global Rank | Rank among countries |
| Country Generation TWh | Generation only when the entity is a country |
| Generation in Latest Year TWh | Country generation in the latest year selected on the slicer |
| Total Sources TWh | Sum of source columns |
| Source Difference TWh | Difference between Total and source sum (QA; keep in a QA folder) |

Set number format on the measure Format property — not with `FORMAT` inside DAX.

---

## 7) How to open and refresh the report

1. Open the final `.pbix` in Power BI Desktop.
2. If CSV paths changed, fix the file paths in Transform data.
3. Keep encoding as UTF-8 (`65001`).
4. Home > Refresh.
5. On the first page, check year 2024 and a sample country (e.g. Iran):
   - Iran generation about `395.79`
   - World about `30,930.23`
   - Share about `1.28%`
   - Rank about `12`

---

## 8) Limitations (portfolio honesty)

- Some older years are incomplete for all countries.
- Some group definitions (e.g. income groups) may not match between the two files; that is why the source-difference measure exists.
- The name `electricity-mix (1).csv` means “second downloaded file,” not “the total file.”
- This dashboard is for learning, portfolio, and a quick overview. For research citations, use the Our World in Data page and metadata.

---

## 9) Ready text for an About page inside Power BI

Put the text below in a new page with several Text boxes.

### Box 1 — Title
Global Electricity Generation — Iran in Focus  
About & Data Source

### Box 2 — Purpose
This report explores where the world’s electricity is generated, which sources produce it, and where Iran sits in that picture. It is built as a Power BI portfolio and learning project: star-schema model, DAX measures, and a clear analytical story.

### Box 3 — Source
Data source: Our World in Data — Electricity Mix  
Website: https://ourworldindata.org/electricity-mix  
Unit: TWh  
The same source family provides total generation and generation by source. Both were loaded as CSV files into this model.

### Box 4 — Files
electricity-mix.csv → table ElectricityTotal  
Columns: Entity, Code, Year, Total electricity  

electricity-mix (1).csv → table ElectricityBySource  
Columns: Entity, Code, Year, Coal, Gas, Oil, Nuclear, Hydropower, Wind, Solar, Bioenergy, Other renewables  

### Box 5 — Notes for the next user
Use DimEntity and DimYear for slicers.  
Do not sum World or regions together with countries.  
Prefer Country Generation TWh and IsCountry for rankings.  
Use a regular stacked column for volume and a 100% stacked column for share.  
Refresh the CSVs after updating file paths.

---

## 10) Building the About page in Power BI Desktop

1. At the bottom of the report, create a new page named `About & Data`.
2. Set the page background dark like the others (`#0B1220`).
3. Insert > Text box — create five boxes and paste the section 9 text.
4. Make the page title white and large.
5. Order the pages so About is last.
6. Ctrl+S.

If you later replace the CSV files, update sections 3 and 4 with the new names and dates.
