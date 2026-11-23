# Review and correction guide
## Global Electricity Generation — Iran in Focus

This document does two jobs. First, it records the review of `Global Electricity Generation.pbix`. Second, it turns that path into practice steps so you can repeat them in Power BI Desktop and learn the pattern.

Do not change the original file until you have done each step yourself. Make a copy named `Global Electricity Generation - Practice.pbix` and practice on the copy.

The numbers below were calculated from the CSV itself for year 2024:

| Item | Value |
|---|---|
| Iran generation | 395.79 TWh |
| World generation (`World`) | 30,930.23 TWh |
| Iran share of world | 1.28% |
| Iran rank among countries with a 3-letter code | 12 of 196 countries with 2024 data |
| Top five countries | China, United States, India, Russia, Japan |

If after fixes the first-page cards and charts do not match these numbers, re-check that step.

---

## 1. What is already correct

Keep these. They are a good pattern.

- Two separate fact tables: `ElectricityTotal` (one total per country-year) and `ElectricityBySource` (same grain, with source columns).
- Two dimensions: `DimEntity` and `DimYear`.
- Four active relationships, all many-to-one, single direction:
  - `ElectricityTotal[Entity]` → `DimEntity[Entity]`
  - `ElectricityBySource[Entity]` → `DimEntity[Entity]`
  - `ElectricityTotal[Year]` → `DimYear[Year]`
  - `ElectricityBySource[Year]` → `DimYear[Year]`
- Separate measures for generation, world share, rank, and the two-file difference.
- First page defaults to Iran and year 2024.
- Comparison page places Iran next to China, United States, and Canada.

The star schema is started. Issues are mostly about measure output types, country vs region classification, slicer interactions, and the wide source table shape.

---

## 2. Review method

Every time you receive a report, run these four passes.

1. **Model view.** Read table names, relationship arrow direction, and the columns on both ends. Arrows should go from fact to dimension.
2. **Read each measure alone.** Is the output a number or text? What happens if the user selects no country? What if they select a region like Asia?
3. **Read each page with saved slicers.** See which year and country are pre-selected, and which visuals those filters reach.
4. **Hand-check one known number.** Here Iran in 2024 is that number. Cards, rank, share, and charts must match the table above.

---

## 3. Issues, ordered by impact

### 3.1 File opens on an empty page

There are four pages. The fourth, `Electricity Trends & Energy Mix`, has no visuals and was the last saved page. Opening the file lands there.

**Fix:** Delete this page, or fill it later. Before Save, click `Global Electricity Dashboard` so the next open starts there.

### 3.2 Two measures return text instead of numbers

`World Generation TWh` and `Selected Country Generation TWh` were built with `FORMAT`. `FORMAT` returns text. It may look fine on a card, but it is unusable in charts, sorting, and later calculations.

Put number format on the measure itself, not inside the formula.

Correct pattern:

```DAX
World Generation TWh =
CALCULATE (
    [Generation TWh],
    REMOVEFILTERS ( DimEntity ),
    DimEntity[Entity] = "World"
)
```

Format: `#,##0.00`

```DAX
Selected Country Generation TWh =
IF (
    HASONEVALUE ( DimEntity[Entity] ),
    [Generation TWh]
)
```

Format: `#,##0.00`

`Generation TWh` also has no format. Give it the same `#,##0.00`.

### 3.3 World share does not look like a percent

`Global Share %` divides correctly (selected generation / `World`, keeping year filters). It has no percent format. For Iran 2024 the real value is `0.0128`. The card shows about `0.01`, while the title says “Global Share %”. A reader will not read that as 1.28%.

Keep the formula. Set Format to `0.00%`. You should see `1.28%`.

### 3.4 Rank returns a number for regions, and skips Kosovo

`Global Rank` limits the ranking set to 3-letter codes, but does not check the value being ranked. If the slicer selects `Asia` or `World`, that region’s generation is larger than any country and rank becomes `1`. That is not a country rank.

Second exception: Kosovo is a country, but its code is `OWID_KOS`, not three letters. The current measure drops it. That does not change Iran’s 2024 rank (Kosovo is 6.77 TWh), but the rule is wrong.

A defensible rule for this file:

- Code is exactly 3 letters → country. Iran is `IRN`.
- Name equals `Kosovo` → country.
- Everything else is region, income group, union, or world: `World` with `OWID_WRL`, `Asia` with `OWID_ASI`, `European Union (27)` with `OWID_EU27`, and Ember/EI names with blank codes.

Build this flag on `DimEntity`, not inside every measure. See section 5.

### 3.5 Top 10 chart collapses when Iran is selected

The first-page entity slicer is on `Iran` and the year slicer on `2024`. Both also filter the bar chart. Top 10 then shows only Iran, because other countries are filtered out.

A “top ten countries” chart should hear year and ignore the country slicer. Path: select the `Entity` slicer, then **Format > Edit interactions**, and set **None** (crossed circle) on the Top 10 chart. Iran cards, rank, share, and the donut should still be filtered by the country slicer. Only Top 10 is disconnected.

After that, with Iran and 2024 still selected, the ten bars should be China through the 10th country — not a single Iran bar. Read Iran’s rank 12 from the card.

The chart category currently uses `ElectricityTotal[Entity]`. That still works after the interaction fix. Cleaner pattern: use `DimEntity[Entity]` and add a visual filter `IsCountry = TRUE` so regions never enter Top N.

### 3.6 Donut double-counts World + region + country if country is cleared

The donut does not `Sum` source columns separately. With Iran 2024 it is correct. If the country selection is cleared, `World`, `Asia`, and individual countries are summed together and generation is multiplied. The source data is built that way: countries, regional totals, and world totals all exist.

Until you Unpivot, keep a visual filter so only one row stays, and keep the country slicer single-select. The real fix is in section 6: a tall table with a `Source` column, and a measure that sums only `IsCountry`.

### 3.7 On the Energy Mix 100% chart, Other renewables appears twice

In Values, `Other renewables` is listed twice. On a 100% chart that source’s share is doubled and the others shrink.

Click the chart. In Values remove one of the two `Other renewables`. You should have 9 sources, each once.

### 3.8 Energy Mix bottom table has no country labels

That table only sums source columns. With four countries selected, all four collapse into one row and you cannot tell whose numbers they are.

Add `DimEntity[Entity]` to Values and make it the first column.

### 3.9 Comparison slicer is single-select, but four countries are saved in the filter

On `Country Comparison` and `Energy Mix`, the entity slicer is Single select, while the saved filter has Canada, Iran, United States, and China. Comparison needs multiple countries.

Select the slicer. In Format, Selection, turn **Single select** off.

Years 2020–2024 are all selected on the comparison page. The bar chart sums `Generation TWh` across those five years. The chart title does not say that. Do one of these:

- Set that page’s year slicer to Single select and pick one year, e.g. 2024.
- Or change the chart title to `Electricity generation, sum of selected years`.

Energy Mix already has year 2024. Keep that so the mix is for one year.

### 3.10 Fact-table key columns are visible in the report

`Entity`, `Year`, and `Code` on both fact tables are still usable on the canvas. Top 10 uses `ElectricityTotal[Entity]`. If someone builds a slicer from a fact column, the star schema is bypassed.

In Data pane, right-click these columns and **Hide in report view**:

- From both fact tables: `Entity`, `Code`, `Year`
- After Unpivot, also hide the numeric source columns, because they should only be seen through measures

Hiding a column does not remove it from the model. Relationships stay.

### 3.11 Absolute file path and wrong CSV encoding

Queries read with a full path in this folder and `Encoding=1252` (Windows-1252). The files are UTF-8. Today country names have no non-Latin characters, so numbers are fine. Moving the folder or adding a name like `Côte d'Ivoire` will break the query or corrupt names.

In Advanced Editor for both queries, change `Encoding=1252` to `Encoding=65001`. That is UTF-8 in Power Query.

For the path, create a parameter so you change it in one place:

1. Home > **Manage parameters** > New
2. Name: `DataFolder`
3. Type: Text
4. Current Value: your project folder path
5. At the start of both queries, use `DataFolder` instead of a hard-coded path:

```powerquery
Source = Csv.Document(
    File.Contents(DataFolder & "\electricity-mix.csv"),
    [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
)
```

The total file is `electricity-mix.csv`. The by-source file is `electricity-mix (1).csv`. The names are opposite of their meaning. Keep the table names you already set correctly (`ElectricityTotal` and `ElectricityBySource`).

### 3.12 Do not confuse the two-file gap with “missing generation”

`Source Difference TWh` is the gap between the source sum and total generation. For Iran in shared years it is about zero. Keep this measure for QA and off the main canvas. Put it in a Display folder named `QA`.

Two real data gaps exist, not relationship bugs:

- For groups like `High-income countries`, the gap reaches about 1,345 TWh (year 1990). The two files define those rows differently.
- Iran starts in 1985 in the total file and in 1965 in the sources file. For 1965–1984 there is no total row. `BLANK() - number` in DAX behaves like `0 - number`, so those years look negative.
- `OPEC (EI)` and `USSR` exist only in the sources file.

---

## 4. Practice A — fixes that do not break current charts

Do this order on the practice copy. After each step, check what is written under it.

### Step 1. Landing page

Delete the empty page. Select `Global Electricity Dashboard`. Ctrl+S.

Close and reopen the file. It should open on that page, with Iran and 2024.

### Step 2. Measure formats

For each measure: click the measure, Ribbon **Measure tools**, Format box.

| Measure | Format |
|---|---|
| `Generation TWh` | `#,##0.00` |
| `World Generation TWh` | replace formula with section 3.2 pattern; Format `#,##0.00` |
| `Selected Country Generation TWh` | section 3.2 formula; Format `#,##0.00` |
| `Global Share %` | keep formula; Format `0.00%` |
| `Global Rank` | `0` |
| `Country Generation TWh` | `#,##0.00` |
| `Total Sources TWh` | `#,##0.00` |
| `Source Difference TWh` | `#,##0.00` and Display folder `QA` |

Check 2024 with Iran slicer:

- Country card: `395.79`
- World card: `30,930.23`
- Share card: `1.28%`

If share shows `0.01`, percent format was not applied.

### Step 3. IsCountry column

In Data pane select `DimEntity`. **New column**:

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

This column is True/False. In Table view on `DimEntity` you should see:

- `Iran` = TRUE
- `Kosovo` = TRUE
- `World`, `Asia`, `High-income countries`, `EU (Ember)` = FALSE

Then update the rank measure. Both the ranking set and the current value must be countries:

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

Also wire `Country Generation TWh` to the same flag so it does not depend on code length alone:

```DAX
Country Generation TWh =
IF (
    SELECTEDVALUE ( DimEntity[IsCountry] ) = TRUE,
    [Generation TWh]
)
```

Check:

- Iran 2024 rank `12`
- If the slicer is on `World`, rank card is blank
- Kosovo gets a numeric rank in a year with data, not blank

### Step 4. Top 10 interaction

Per section 3.5, disconnect the first-page entity slicer from the Top 10 chart. Year should still affect that chart.

Check: year 2024, country Iran. Iran card stays `395.79`. Top 10 shows ten countries; first bar is China.

### Step 5. Energy Mix chart and table

Remove one of the two `Other renewables` fields from the 100% chart. Add `DimEntity[Entity]` to the bottom table.

Set the entity slicer on this page and the comparison page to Multi-select (Single select off).

Check: on the Iran 2024 100% chart, shares sum to 100% and Other renewables is one slice, not two.

### Step 6. Hide keys and UTF-8

Hide `Entity`, `Code`, and `Year` on both fact tables. If Top 10 used `ElectricityTotal[Entity]` and lost its category after hiding, set category to `DimEntity[Entity]` and add visual filter `IsCountry = TRUE`.

Set Encoding to `65001` and Refresh. `ElectricityTotal` should stay 7,871 rows and `ElectricityBySource` 10,582. If counts change, revert Encoding and check the file path.

---

## 5. Why the source table should be tall

`ElectricityBySource` is wide: one column per source. That is why the donut and 100% chart have nine separate fields, one of them duplicated, and you cannot build a “Source” slicer.

The star-schema shape for this is:

- One row per entity, year, and source
- A small `DimSource` dimension
- One measure: sum of `Generation TWh`

Before Unpivot, take another practice copy. This step empties visuals bound to Coal, Gas, and the other source columns. You will rebuild the donut, 100% chart, and table.

---

## 6. Practice B — Unpivot and rebuild source visuals

### Step 1. Unpivot in Power Query

1. Open Transform data.
2. Select query `ElectricityBySource`.
3. Select `Entity`, `Code`, and `Year` with Ctrl.
4. Right-click one of those three headers > **Unpivot Other Columns**.
5. Rename `Attribute` to `Source`.
6. Rename `Value` to `Generation TWh`.
7. Do not set `Generation TWh` to Whole Number — use **Decimal Number**. Keep `Year` as whole number.
8. Close & Apply.

If you prefer Advanced Editor, after Promote Headers and type changes the query should include:

```powerquery
Unpivoted =
    Table.UnpivotOtherColumns(
        #"Changed Type",
        {"Entity", "Code", "Year"},
        "Source",
        "Generation TWh"
    )
```

Set the query output to `Unpivoted`.

Row count grows because each of the 9 sources becomes its own row. Remove blank source rows so the sum matches the previous `SUM`:

```powerquery
RemovedBlanks =
    Table.SelectRows(
        Unpivoted,
        each [#"Generation TWh"] <> null
    )
```

`SUM` in DAX already ignores blanks, so keeping blanks does not change totals. Removing them makes the model smaller.

### Step 2. Source dimension

The simplest source dimension is a calculated table. `SortOrder` is the order you want on charts, fossil to renewable:

```DAX
DimSource =
DATATABLE (
    "Source", STRING,
    "SortOrder", INTEGER,
    "SourceGroup", STRING,
    {
        { "Coal", 1, "Fossil" },
        { "Gas", 2, "Fossil" },
        { "Oil", 3, "Fossil" },
        { "Nuclear", 4, "Nuclear" },
        { "Hydropower", 5, "Renewable" },
        { "Wind", 6, "Renewable" },
        { "Solar", 7, "Renewable" },
        { "Bioenergy", 8, "Renewable" },
        { "Other renewables", 9, "Renewable" }
    }
)
```

Select `Source` on `DimSource`. Column tools > **Sort by column** > `SortOrder`.

New relationship:

- From `ElectricityBySource[Source]`
- To `DimSource[Source]`
- Cardinality: Many to one
- Cross filter direction: Single
- Active: on

Do not touch the existing Entity and Year relationships.

### Step 3. One measure instead of nine columns

The old `Total Sources TWh` that summed nine `SUM`s will no longer find those columns. Replace it with:

```DAX
Total Sources TWh =
SUM ( ElectricityBySource[Generation TWh] )
```

For charts that must not mix regions and World:

```DAX
Country Source TWh =
CALCULATE (
    [Total Sources TWh],
    DimEntity[IsCountry] = TRUE
)
```

`Source Difference TWh` stays `[Generation TWh] - [Total Sources TWh]`. Keep it in the `QA` folder.

### Step 4. Rebuild the visuals

**First-page donut**

- Legend: `DimSource[Source]`
- Values: `Country Source TWh`
- Year and country slicers still filter it

With Iran and 2024, slice totals should equal 395.79, because Iran’s two-file gap is zero.

**Energy Mix 100% chart**

- Axis: `DimEntity[Entity]`
- Legend: `DimSource[Source]`
- Values: `Total Sources TWh` (this page already has specific countries selected; if the slicer is cleared, `Country Source TWh` is safer)
- Each source appears once in Legend. There is no way for Other renewables to duplicate.

**Same-page table**

- `DimEntity[Entity]`
- `DimSource[Source]`
- `Total Sources TWh`

### Step 5. Post-Unpivot check

Build a temporary table, then delete it.

- Year 2024, entity Iran, `Generation TWh` = 395.79
- Same filter, `Total Sources TWh` = 395.79
- `Source Difference TWh` ≈ 0
- Rank = 12
- Share = 1.28%
- Iran donut has 9 slices; largest are gas then oil/gas — not a duplicated slice

You do not need to memorize per-source numbers. Donut total matching the generation card is enough.

---

## 7. Fill page four if you want it

You deleted the empty page. If you want an “Iran trends and energy mix” topic, create a new page with that name and put only these four things:

1. Year slicer, range, default 2015–2024. Single select off.
2. Entity slicer on Single select, default Iran. All other visuals on this page filter from it.
3. Line chart: axis `DimYear[Year]`, value `Generation TWh`. This is Iran’s total generation trend.
4. Stacked column: axis `DimYear[Year]`, Legend `DimSource[Source]`, value `Total Sources TWh`.

Write a one-line page title, e.g. `Iran, generation and source mix`.

Do not put the World card here. That card belongs on page 1 and is separated from the country slicer with `REMOVEFILTERS`.

---

## 8. End-of-work checklist

- [ ] File opens on the dashboard page, not an empty page
- [ ] Iran 2024: generation 395.79, world 30,930.23, share 1.28%, rank 12
- [ ] Selecting `World` blanks rank
- [ ] `Kosovo` is TRUE in `IsCountry`
- [ ] Top 10 still shows ten countries with Iran selected and respects year
- [ ] Other renewables appears once on the 100% chart
- [ ] Source mix table shows country names
- [ ] Comparison slicer keeps multiple countries at once
- [ ] If the comparison bar sums multiple years, the title says so — or the year slicer is single-select
- [ ] No measure uses `FORMAT`; format is in Measure tools
- [ ] Fact-table key columns are hidden in Report view
- [ ] Queries load with UTF-8 (`65001`)
- [ ] After Unpivot, donut and 100% chart use `DimSource[Source]`
- [ ] Iran 2024 source total matches Iran 2024 total generation
- [ ] `Source Difference TWh` is not on the main canvas and is in the QA folder

---

## 9. Pattern to keep from this exercise

1. Compute with `SUM` and `DIVIDE`. Leave display shape to Format.
2. “Is this row a country or a region?” is one column on the dimension. Measures filter that column.
3. A measure that must show World clears entity filters with `REMOVEFILTERS` and sets `Entity = "World"` itself.
4. A rank that only means something for countries limits both the `RANKX` set and the current value to countries.
5. A slicer that sets page context must not filter a chart meant to show other countries. Disconnect it with Edit interactions.
6. If a topic (here energy source) should be an axis or Legend, it is one column in the data — not nine side-by-side columns.
7. Hand-check one known number before you call the work done. Here that number is Iran in 2024.
