# Global Electricity Generation — Iran in Focus

Power BI portfolio project: where the world’s electricity is generated, which sources produce it, and where Iran sits in that picture.

## Story

1. **Global Electricity Dashboard** — world scale, selected country, rank, share of world, Top 10, source mix  
2. **Country Comparison** — compare countries in the latest selected year and across a year range  
3. **Energy Mix** — volume and share by source for selected countries  
4. **About & Data** — purpose, data source, file map, how the next user should use the model  

Core question:

> Where is the world’s electricity generated, from which sources, and where does Iran stand?

## Open the report

Download / clone this repository, then open:

`Global Electricity Generation - Publish.pbix`

in **Power BI Desktop**.

If the CSV paths do not match your machine:

1. Home → Transform data  
2. Update paths for both queries  
3. Keep encoding **UTF-8 (`65001`)**  
4. Close & Apply → Refresh  

Quick check for **Iran, 2024**:

| Metric | Approx. value |
|---|---|
| Generation | 395.79 TWh |
| World | 30,930.23 TWh |
| Share of world | 1.28% |
| Rank (countries) | 12 |

## Data source

- **Our World in Data — Electricity Mix**  
  https://ourworldindata.org/electricity-mix  
- Unit: **TWh**  
- Years in the current files: about **1900–2025**

| File | Model table | Contents |
|---|---|---|
| `electricity-mix.csv` | `ElectricityTotal` | Entity, Code, Year, Total electricity |
| `electricity-mix (1).csv` | `ElectricityBySource` | Entity, Code, Year + Coal, Gas, Oil, Nuclear, Hydropower, Wind, Solar, Bioenergy, Other renewables |

## Model notes

- Star schema: `DimEntity` and `DimYear` filter both fact tables  
- `IsCountry` separates countries from regions / World (needed for rank and Top N)  
- Do not sum World or regional aggregates together with countries  
- Use a normal stacked column for **volume**, and a 100% stacked column for **share**

## What this project demonstrates

- Power Query load and typing  
- Relationships and dimension tables  
- DAX measures (generation, world share, rank, latest-year comparison)  
- Multi-page report design and dark theme formatting  
- Documentation for the next user  

## License / attribution

Report built for learning and portfolio use.  
Underlying electricity data © Our World in Data contributors / original providers — see the OWID Electricity Mix page for citation details.

Persian project notes: `درباره پروژه و منبع داده.md`
