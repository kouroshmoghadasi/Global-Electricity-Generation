# Dashboard implementation guide
## Global Electricity Generation — Iran in Focus

Open `Global Electricity Generation - Final.pbix` (or the current publish file). The model and layout for all three pages were changed in that file. Do not open `... - Redesigned.pbix`; that copy was corrupted during packaging. The steps below explain the same work so you can repeat it on a copy and finish with Iran’s 2024 numbers.

| Check | Correct value |
|---|---|
| Iran generation | 395.79 |
| World generation | 30,930.23 |
| Iran share | 1.28% |
| Iran rank | 12 |

If a window titled `Global Electricity Generation1` is open, the model fixes (measures, `IsCountry`, UTF-8 CSV load) are in that window and not saved yet. The original file is locked because another Power BI instance has it open. In that window Cancel Save As, close the other windows for the same file, then from the `Generation1` window do Save As with a new name. If you closed that window, start again from step 1. Dashboard visuals are rebuilt from step 8.

---

## 1. Copy the file

Close the file. In the same folder make a copy and open the copy. Suggested name: `Global Electricity Generation - Practice.pbix`.

---

## 2. Load CSV with UTF-8

Home > Transform data.

In queries `ElectricityTotal` and `ElectricityBySource`, open Advanced Editor and change only these two expressions:

- `Encoding=1252` → `Encoding=65001`
- `QuoteStyle.None` → `QuoteStyle.Csv`

Do not change table names or file paths. Close & Apply.

After Refresh, row counts should stay the same: `ElectricityTotal` = 7,871 and `ElectricityBySource` = 10,582.

---

## 3. Country flag column

Select `DimEntity`. New column. Paste this formula and name the column `IsCountry`:

```DAX
IsCountry =
VAR CodeFromTotal =
    LOOKUPVALUE (
        ElectricityTotal[Code],
        ElectricityTotal[Entity], [Entity]
    )
VAR CodeFromMix =
    LOOKUPVALUE (
        ElectricityBySource[Code],
        ElectricityBySource[Entity], [Entity]
    )
VAR CodeToUse =
    IF ( NOT ISBLANK ( CodeFromTotal ), CodeFromTotal, CodeFromMix )
RETURN
    [Entity] = "Kosovo"
        || (
            NOT ISBLANK ( CodeToUse )
                && LEN ( CodeToUse ) = 3
        )
```

In Table view check: `Iran` and `Kosovo` = TRUE. `World`, `Asia`, and `High-income countries` = FALSE.

Iran’s code is `IRN`. Kosovo uses `OWID_KOS`, which is why it is handled separately. `World` uses `OWID_WRL` and is not a country.

---

## 4. Measures

Open each measure, replace the formula, and set Format from Measure tools. Do not use `FORMAT` inside the formula — it returns text and breaks charts and later calculations.

### Generation TWh

```DAX
Generation TWh =
SUM ( ElectricityTotal[Total electricity] )
```

Format: `#,##0.00`

### World Generation TWh

```DAX
World Generation TWh =
CALCULATE (
    [Generation TWh],
    REMOVEFILTERS ( DimEntity ),
    DimEntity[Entity] = "World"
)
```

Format: `#,##0.00`

`REMOVEFILTERS` clears the country slicer; the measure itself reads the `World` row. Year filters stay.

### Global Share %

The formula is already correct. Only set Format to `0.00%`.

```DAX
Global Share % =
DIVIDE (
    [Generation TWh],
    CALCULATE (
        [Generation TWh],
        REMOVEFILTERS ( DimEntity ),
        DimEntity[Entity] = "World"
    )
)
```

### Country Generation TWh

```DAX
Country Generation TWh =
IF (
    SELECTEDVALUE ( DimEntity[IsCountry] ) = TRUE,
    [Generation TWh]
)
```

Format: `#,##0.00`

### Selected Country Generation TWh

```DAX
Selected Country Generation TWh =
IF (
    HASONEVALUE ( DimEntity[Entity] ),
    [Generation TWh]
)
```

Format: `#,##0.00`

### Global Rank

Rank returns a number only when the selected entity is a country. Regions and `World` stay blank.

```DAX
Global Rank =
VAR CurrentIsCountry =
    SELECTEDVALUE ( DimEntity[IsCountry] )
VAR CurrentGeneration =
    [Generation TWh]
VAR CountriesToRank =
    FILTER (
        ALL ( DimEntity[Entity] ),
        CALCULATE ( SELECTEDVALUE ( DimEntity[IsCountry] ) ) = TRUE
    )
RETURN
    IF (
        CurrentIsCountry = TRUE && NOT ISBLANK ( CurrentGeneration ),
        RANKX (
            CountriesToRank,
            CALCULATE ( [Generation TWh] ),
            CurrentGeneration,
            DESC,
            DENSE
        )
    )
```

Format: `0`

### Generation in Latest Year TWh

Create this new measure on `ElectricityTotal`. The comparison page bar chart uses it so selecting multiple years does not sum those years together.

```DAX
Generation in Latest Year TWh =
VAR LatestYear =
    MAX ( DimYear[Year] )
RETURN
    CALCULATE (
        [Country Generation TWh],
        DimYear[Year] = LatestYear
    )
```

Format: `#,##0.00`

### Total Sources TWh and Source Difference TWh

Keep the formulas. Format both: `#,##0.00`.

Put `Source Difference TWh` in a Display folder named `QA` and do not put it on the canvas. For Iran in shared years the difference is about zero. For groups like `High-income countries` the gap can reach about 1,345 TWh because the two CSV files are not identical for those rows.

---

## 5. Hide key columns

In both `ElectricityTotal` and `ElectricityBySource`, right-click `Entity`, `Code`, and `Year` and choose Hide in report view. Also hide `Total electricity`, because measure `[Generation TWh]` replaces it.

Leave relationships as they are. The four many-to-one single-direction relationships are correct:

- Both fact tables: `Entity` → `DimEntity[Entity]`
- Both fact tables: `Year` → `DimYear[Year]`

---

## 6. Model check before changing visuals

Build a temporary table or read the first-page cards. Year 2024, country Iran:

- Generation 395.79
- World 30,930.23
- Share 1.28%
- Rank 12
- If you select `World`, rank is blank

Delete the temporary table.

---

## 7. Pages

Delete the empty page `Electricity Trends & Energy Mix`. Before Save, select `Global Electricity Dashboard` so the file opens on that page next time.

If the country slicer is on Algeria or another country, select Iran again. Keep year 2024 on page 1.

---

## 8. First-page layout

Canvas is 1280 × 720. Set all three page backgrounds from Format > Canvas background to `#0B1220`.

Card / chart / slicer backgrounds: `#152033`. Chart titles: `#E2E8F0`, size 14, left-aligned, bold. Axis color: `#94A3B8`.

Place four cards in one row. y = 16, height 118, width 246.

| Order left→right | x | Measure | Card label | Value color |
|---|---|---|---|---|
| 1 | 16 | Selected Country Generation TWh | Selected country | `#F6C453` |
| 2 | 274 | Global Rank | Global rank | `#F8FAFC` |
| 3 | 532 | Global Share % | Share of world | `#F8FAFC` |
| 4 | 790 | World Generation TWh | World | `#F8FAFC` |

Iran’s card background is `#1C2A22` so it stands out from the other three. Others use `#152033`. Value size 28; label size 12 with color `#94A3B8`.

Also set format on the card itself. If the World card visual format is `0.00`, thousands separators will not show — use `#,##0.00`. Share card must be `0.00%`.

Slicers on the right column:

- Year: x 1048, y 16, width 216, height 188. Title: Year
- Country or region: x 1048, y 216, width 216, height 488. Title: Country or region

Charts:

- Top 10 countries: x 16, y 148, width 504, height 556. Bar color `#2DD4BF`
- Source mix: x 532, y 148, width 504, height 556

Set Top 10 category to `DimEntity[Entity]`. Value stays `Country Generation TWh`. Keep the Top 10 visual filter on that chart.

The country slicer should not filter this chart. Keep that: the selected country filters cards and the donut; Top 10 shows the ten countries for that year. If Iran is selected and the chart shows only one Iran bar, select the Entity slicer, Format > Edit interactions, and set None on the Top 10 chart. Year slicer should still Filter.

Donut colors by source:

| Source | Color |
|---|---|
| Coal | `#64748B` |
| Gas | `#F59E0B` |
| Oil | `#C2410C` |
| Nuclear | `#A78BFA` |
| Hydropower | `#38BDF8` |
| Wind | `#34D399` |
| Solar | `#FACC15` |
| Bioenergy | `#84CC16` |
| Other renewables | `#2DD4BF` |

With Iran and 2024, the donut total should be 395.79. If the country slicer is cleared, the donut sums World + regions + countries. Keep the slicer single-select with Iran selected.

---

## 9. Country Comparison page

Slicers on the left, width 200:

- Year from y 16, height 210
- Country from y 238, height 466

Turn off Single select on the country slicer so Canada, Iran, United States, and China can stay selected together.

Bar chart: x 232, y 16, width 508, height 688. Title: `Latest selected year`. Change the value from `Generation TWh` to `Generation in Latest Year TWh`. Bar color `#2DD4BF`.

Line chart: x 752, y 16, width 512, height 688. Title: `Trend across selected years`. Value stays `Country Generation TWh`. Line color `#38BDF8`.

Years 2020–2024 can stay selected. The bar shows only 2024; the line shows all five years.

---

## 10. Energy Mix page

Slicers like the comparison page. Country Single select off.

100% stacked chart: x 232, y 16, width 1032, height 400. Title: `Share of each source`.

In Values, `Other renewables` appears twice. Remove one. You should have 9 sources. Use the same donut colors here.

Table: x 232, y 428, width 1032, height 276. Title: `Source totals by country`.

Add `DimEntity[Entity]` to Values and make it the first column. Without it, the four selected countries collapse into one row.

Keep this page’s year on 2024.

---

## 11. Final checklist

- [ ] File opens on `Global Electricity Dashboard`
- [ ] Empty page is deleted
- [ ] Iran 2024: 395.79, 30,930.23, 1.28%, rank 12
- [ ] Selecting `World` blanks rank
- [ ] `Kosovo` is TRUE in `IsCountry`
- [ ] Top 10 still shows ten countries with Iran selected; first bar is China
- [ ] Iran donut has 9 colors and totals 395.79
- [ ] Other renewables appears once on the 100% chart
- [ ] Source mix table shows country names
- [ ] Comparison bars use only the latest selected year; line shows the trend across years
- [ ] No measure uses `FORMAT`
- [ ] `Source Difference TWh` is in the QA folder and not on the canvas
- [ ] Background is `#0B1220` and Iran’s card value is `#F6C453`

Ctrl+S.
