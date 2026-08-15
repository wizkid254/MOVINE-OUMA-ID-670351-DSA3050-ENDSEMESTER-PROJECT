# MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT
# DSA3050 – Business Intelligence & Data Visualization
## Section A: Dataset Selection & Understanding (10 marks)



### 1. Source of the Dataset

The dataset is the **Global Superstore** dataset, a widely used retail transactions dataset originally distributed as a Tableau sample dataset and commonly hosted on Kaggle (e.g. "Global Superstore Dataset" by various uploaders). It is a real-world-style dataset built from an international superstore's order history.
![This is a image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-13%20235048.png?raw=true)


### 2. What the Dataset Represents

The dataset contains 51,290 individual order line items placed between 2011 and 2014 by a global retail superstore operating across 7 markets (US, Canada, LATAM, EU, EMEA, Africa, APAC), covering 147 countries, 1,094 states/provinces, and 3,636 cities. Each row represents one product line within a customer order, and includes:

- Order details: Order ID, Order Date, Ship Date, Ship Mode, Order Priority
- Customer details: Customer ID, Customer Name, Segment (Consumer / Corporate / Home Office)
- Product details: Product ID, Product Name, Category (3: Furniture, Office Supplies, Technology), Sub-Category (17 types)
- Geography: Country, State, City, Region, Market
- Financials: Sales, Profit, Discount, Quantity, Shipping Cost

Across the dataset there are 25,035 unique orders, 10,292 unique products, and 4,873 unique customers — giving enough granularity for meaningful drill-down by customer, product, and location.

### 3. Why I Selected It

*(Personalize this — some genuine reasons the data supports:)*
- It combines transactional (fact-level) data with rich categorical, geographic, and time dimensions, which is exactly the shape a star-schema BI project needs.
- It is not pre-aggregated — it's raw order-line data, so real cleaning/transformation work is required (unlike a summarized dataset, which the exam explicitly discourages).
- Profit is sometimes negative (12,544 of 51,290 rows, ~24%), which creates a genuine analytical problem worth investigating — not just a "which segment sold the most" surface question.
- The international, multi-market scope supports geographic and time-intelligence analysis (YoY growth, seasonal patterns, market comparisons).

### 4. Main Variables Available

| Type | Variables |
|---|---|
| Numerical | Sales, Profit, Discount, Quantity, Shipping Cost |
| Categorical | Category, Sub-Category, Segment, Market, Region, Ship Mode, Order Priority, Country, State, City |
| Date/Time | Order Date, Ship Date, Year |
| Identifiers | Order ID, Row ID, Customer ID, Product ID |



### 5. Business/Analytical Problem

Core problem: Despite generating over $14.9M in total sales and $1.47M in total profit across 2011–2014, a significant share of order lines (~24%) are unprofitable. The business needs to understand where, on what, and for whom profitability is being lost — and whether growth (sales volume, new markets, discounting strategy) is being pursued at the expense of margin.

This positions the project around: "Where is the business winning, where is it losing money, and what is driving the difference?" — which naturally supports the exam's Overview → Detailed → Diagnostic dashboard structure.

### 6. Analytical Questions the Power BI Solution Should Answer

1. Which markets, regions, and product categories generate the highest sales and profit — and are the two aligned, or are some high-sales areas actually low/negative profit?
2. How has performance trended over time (2011–2014), and is there a consistent year-over-year growth pattern or seasonality by quarter/month?
3. What is the relationship between discount level and profitability — at what discount thresholds does profit turn negative, and does this vary by category or segment?
4. Which customer segments (Consumer, Corporate, Home Office) and which top customers/products contribute disproportionately to sales vs. to losses?
5. Does shipping mode or order priority affect profitability (e.g. via shipping cost as a share of order value), and are there markets where shipping cost erodes margin significantly?
6.  Which sub-categories are consistently loss-making across multiple markets, indicating a structural/pricing issue rather than a one-off?

7.  # Section B: Data Cleaning & Transformation (Power Query)(20 marks)

## Overview

The raw Global Superstore** dataset (`data/Global_Superstore.xlsx`) contains 51,290
order-line records across 27 columns. Although the file loads without errors, it is
not analysis-ready: it contains a meaningless helper column, duplicate/ambiguous
geography fields, pre-baked derived columns that should be modelled properly instead,
inconsistent data types, and a flat structure with no dimension tables.

All transformation work was performed in **Power Query Editor** (Home → Transform
Data). Rather than simply importing the file and clicking *Close & Apply*, the raw
Fact query was used as a reference source to build five separate queries —
`FactSales`, `DimDate`, `DimCustomer`, `DimProduct`, and `DimLocation` — so that
cleaning, deduplication, and dimension-building could happen independently without
repeatedly re-importing the source file.

Below are the documented transformations, in the required Problem → Transformation → Reason → Result format. A minimum of 8 is required by
the brief; 11 are documented here across the Fact query and each dimension table.

---

## Transformation Log

### 1. Removing a meaningless helper column

**Problem:** The column `记录数` (Chinese for "record count") contains the value `1`
for every single row in the dataset, adding no analytical value and being unreadable
to most report consumers.

**Transformation:** Home → Remove Columns → selected `记录数` and removed it.

Reason: A column that is constant across all 51,290 rows carries zero
information and only clutters the field list and model.

Result: The column list is reduced to genuinely useful fields; the fact table
is easier to navigate in the Fields pane.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-14%20121556.png?raw=true)

---

### 2. Removing a raw spreadsheet index column

Problem: `Row ID` is simply a sequential spreadsheet row number, not a business
key or attribute.

Transformation: Home → Remove Columns → `Row ID`.

Reason: Power BI's own row context and the `Order ID` field already identify
transactions; a meaningless numeric index adds no analytical or modelling value.

Result: One fewer redundant column in the Fact table.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-14%20121556.png?raw=true)
---

### 3. Resolving a duplicate, ambiguous geography column

Problem: Both `Market` and `Market2` describe geography, but at different and
unlabelled granularities — `Market2` groups `US` and `Canada` together under
"North America", while `Market` keeps them separate. Having two similarly-named
columns with no explanation of their difference is confusing for anyone using the
report.

Transformation: Renamed `Market2` to `Region Group` to describe its actual
purpose (a rolled-up geographic grouping), and retained `Market` as the primary
geography field used in the star schema.

Reason: Clear, self-explanatory field names are part of the modelling
requirement, and an unexplained duplicate field risks being used incorrectly by
report users (e.g. accidentally double-counting a country under two different
groupings).

Result: One clearly-purposed field (`Region Group`) instead of two ambiguous,
similarly-named columns.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-14%20121934.png?raw=true)
---

### 4. Removing and correctly re-deriving a pre-baked date attribute

Problem: `weeknum` is a pre-calculated ISO week number sitting directly in the
flat fact table — a derived attribute, not a raw fact, and in the wrong place in a
star-schema design.

Transformation: Removed `weeknum` from the Fact query entirely. Recreated the
same attribute properly inside the `DimDate` dimension table using
`Date.WeekOfYear([Date])`.
Reason: Derived date attributes belong on the date dimension, not repeated on
every transaction row — this avoids redundant storage and lets the attribute be
reused consistently for any date-based filtering, not just Order Date.

Result: A clean Fact table with no derived columns, and a fully-featured
`DimDate` table containing Week Number alongside Year, Quarter, Month, and Weekday
attributes.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-14%20122424.png?raw=true)
---

### 5. Correcting date/time data types

Problem: `Order Date` and `Ship Date` were stored with a redundant
`00:00:00.000` time component (i.e. typed as Date/Time rather than Date), even
though no order in this dataset carries meaningful time-of-day information.

Transformation: Transform → Data Type → changed both columns from Date/Time to
Date.

Reason: Carrying an unused time component increases storage and can cause
subtle relationship-matching issues when joining to a Date dimension keyed on pure
dates.

Result: Both columns are now clean `Date` types, ready to relate directly to
`DimDate[Date]`.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-14%20122620.png?raw=true)
---

### 6. Standardizing numeric data types

Problem: `Sales` was stored as a whole/rounded integer while `Profit`,
`Discount`, and `Shipping Cost` were stored as decimals — an inconsistent numeric
typing scheme across related currency fields.

Transformation: Transform → Data Type → changed `Sales` to Fixed Decimal
Number, matching the other currency fields.

Reason: Consistent numeric precision across all monetary fields avoids rounding
artefacts when these fields are combined in DAX measures (e.g. Profit Margin %).

Result: All currency-related fields share a consistent decimal type.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-14%20122805.png?raw=true)
---

### 7. Cleaning inconsistent text formatting

Problem: Categorical text fields (`Category`, `Sub-Category`, `Segment`,
`Ship Mode`, `Region`, `Market`, `Customer Name`, `Product Name`) were at risk of
inconsistent casing and stray leading/trailing whitespace, which would silently
split what should be a single category into multiple groups in visuals.

Transformation: Transform → Format → applied `Text.Trim` across the relevant
text columns, and verified capitalization consistency.

Reason: Even a single trailing space (`"Consumer "` vs `"Consumer"`) causes
Power BI to treat two identical business categories as separate groups, distorting
every chart and KPI that uses that field.

Result: Every text category now has exactly one consistent representation
throughout the model.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-14%20124012.png?raw=true)
---

### 8. Creating a conditional column to flag loss-making orders

Problem: `Profit` contains a large number of negative values (12,544 of the
51,290 rows, roughly 24% of all transactions) mixed in with positive values, with
no existing field to quickly identify or filter loss-making orders for diagnostic
analysis.

Transformation: Add Column → Custom Column:
```
Profitability Flag = if [Profit] < 0 then "Loss" else "Profit"
```

Reason: This enables one-click slicing and drill-through into loss-making
transactions on the diagnostic dashboard page, without requiring a DAX filter
measure inside every single visual.

Result: A new categorical column, `Profitability Flag`, ready to be used
directly in slicers, visuals, and drill-through pages.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-14%20125033.png?raw=true)
---

### 9. Building a dedicated Customer dimension (deduplication + conflict check)

Problem: Customer attributes (`Customer ID`, `Customer Name`, `Segment`) were
repeated on every one of that customer's transaction rows inside the flat fact
table — a denormalized structure unsuitable for a star schema.

Transformation: Referenced the Fact query, kept only the three customer
columns, and used `Table.Distinct` keyed specifically on `Customer ID` to
deduplicate (rather than a blanket `Remove Duplicates` across all columns, which
would silently keep conflicting rows if a Customer ID were ever paired with more
than one Segment value).

Reason: Normalizing customer attributes into their own dimension table removes
repeated data from the fact table and is required for a valid star-schema
relationship (one customer row must relate to many fact rows).

Result: A clean `DimCustomer` table with one row per unique `Customer ID`,
related to `FactSales` as 1-to-many.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20202306.png?raw=true)
---

### 10. Building a dedicated Product dimension (deduplication)

Problem: Product attributes (`Product ID`, `Product Name`, `Category`,
`Sub-Category`) were similarly repeated across every transaction row for that
product.
Transformation: Referenced the Fact query, kept only the four product columns,
trimmed text fields, and applied `Table.Distinct` keyed on `Product ID`.

Reason: Same normalization rationale as the customer dimension — reduces
redundant storage and produces a valid dimension table for the star schema.

Result: A clean `DimProduct` table with one row per unique `Product ID`.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20202705.png?raw=true)
---

### 11. Building a Location dimension with a generated surrogate key

Problem: Geography (`Country`, `State`, `City`, `Region`, `Market`) existed only
as flat, repeated columns in the fact table, and — unlike Customer or Product —
there was no existing unique identifier for a "location," since city names such as
Springfield or Columbus repeat across different states and even countries.

Transformation: Referenced the Fact query, selected the five geography columns,
deduplicated, and added a custom surrogate key column:
```
Location Key = Text.Combine({[Country], [State], [City]}, "|")
```
The same concatenation logic was added to the Fact query so both tables share a
matching join key.

Reason: Without a generated key, `City` alone is not unique and would cause
many-to-many ambiguity or incorrect matches between locations in different states/
countries that happen to share a city name.

Result: A dedicated `DimLocation` table with a guaranteed-unique key, correctly
relatable to `FactSales[Location Key]` as 1-to-many.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20203017.png?raw=true)
---
I also created DimDate table
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20203411.png?raw=true)
## Additional Notes

- No duplicate rows were found in the raw fact-level data (`Table.Distinct` on
  the full row set returned zero removed rows), so no full-row deduplication was
  required at the fact level — only at the dimension level, where repetition is
  expected and intentional prior to deduplication.
  No null values were present in any column of the raw dataset, so no
  null-handling/imputation strategy was required for this dataset.
  
  
# Section C: Data Modelling(20 marks)


## Overview

The cleaned queries from Section B were assembled into a star schema rather than
left as a single flat table. A flat table was deliberately avoided because it would
force every customer, product, and location attribute to repeat on all 51,290
transaction rows, bloat the model, and make relationship-based filtering (slicers,
cross-filtering, drill-down) unreliable. A star schema instead separates
what happened (the fact table) from the descriptive context around it
(the dimension tables), which is both more storage-efficient and the standard,
recommended approach for Power BI reporting.

The final model:

```
                     DimDate
                        |
DimCustomer ---- FactSales ---- DimProduct
                        |
                  DimLocation
```

## Why `FactSales` Was Selected as the Fact Table

`FactSales` holds the transactional grain of the dataset — one row per order
line — and contains every numeric measure the report needs to aggregate:
`Sales`, `Profit`, `Quantity`, `Discount`, and `Shipping Cost`. A fact table should
sit at the centre of the model because it is the table that grows with business
activity (every new order adds a row here) and is the table all analytical measures
are calculated against. All descriptive attributes (who, what, where, when) were
deliberately removed from this table during Section B and pushed into dimension
tables, leaving `FactSales` with only:

- Foreign/join keys: `Order ID`, `Customer ID`, `Product ID`, `Location Key`,
  `Order Date`, `Ship Date`
- Measures: `Sales`, `Profit`, `Quantity`, `Discount`, `Shipping Cost`
- The `Profitability Flag` conditional column created in Section B

## Why Each Dimension Was Created

| Dimension | Purpose | Key |
|---|---|---|
| DimCustomer | Describes *who* placed the order — Customer Name and Segment — so sales/profit can be sliced by customer segment without repeating that text on every transaction row. | `Customer ID` |
| DimProduct | Describes *what* was sold — Product Name, Category, Sub-Category — enabling category-level analysis and drill-down from Category → Sub-Category → Product. | `Product ID` |
|DimLocation | Describes *where* the order shipped — Country, State, City, Region, Market — enabling geographic analysis (maps, regional comparisons). Required a generated surrogate key (see Modelling Challenges below) since no natural unique location identifier existed in the raw data. | `Location Key` (generated) |
| DimDate | Describes when the order happened, as a continuous calendar table independent of the transaction data. Required for correct time-intelligence DAX (e.g. `SAMEPERIODLASTYEAR`, `YoY Growth %`) — these functions fail or behave unreliably against a date column that has gaps (dates with no orders), which `Order Date` alone would have. | `Date` |

Each dimension answers a different analytical question a manager might ask
("by customer," "by product," "by region," "by time period"), which is exactly the
separation a star schema is designed to support.

## Relationships, Cardinality, and Filter Direction

| Relationship | Cardinality | Cross-Filter Direction |
|---|---|---|
| `DimCustomer[Customer ID]` → `FactSales[Customer ID]` | One-to-Many | Single (dimension → fact) |
| `DimProduct[Product ID]` → `FactSales[Product ID]` | One-to-Many | Single (dimension → fact) |
| `DimLocation[Location Key]` → `FactSales[Location Key]` | One-to-Many | Single (dimension → fact) |
| `DimDate[Date]` → `FactSales[Order Date]` | One-to-Many | Single (dimension → fact) |

Cardinality decision: every relationship is one-to-many, because one row in
a dimension table (one customer, one product, one location, one date) can relate to
many rows in the fact table (many orders by that customer, of that product, from
that location, on that date). This is the natural cardinality of a star schema and
matches how the data was normalized in Section B — each dimension was explicitly
deduplicated down to one row per key before being related.

Filter direction decision: all relationships use single-direction filtering,
flowing from each dimension into the fact table. This was chosen deliberately over
bidirectional filtering because:
- It matches how the report is actually used — users filter the fact table by
  customer/product/location/date, not the other way around.
- Bidirectional filtering across multiple dimensions simultaneously connected to one
  fact table can create ambiguous filter paths, where Power BI cannot
  determine a single, predictable way to apply a filter. Since this model already
  has four dimensions on one fact table, keeping every relationship single-direction
  avoids this risk entirely.
- No genuine many-to-many business scenario exists in this dataset (e.g. no
  promotions-to-products bridge table) that would justify the added complexity of
  bidirectional or many-to-many relationships.

## The Date Table

`DimDate` was built as a continuous calendar (padded slightly beyond the actual
`Order Date`/`Ship Date` range to avoid edge-case gaps) and explicitly marked as an
official Date Table in Power BI (Modeling → Mark as Date Table, using the
`Date` column). This is required for Power BI's time-intelligence DAX functions
(`SAMEPERIODLASTYEAR`, `DATESYTD`, etc.) to behave correctly — without an official,
continuous, gap-free date table, these functions can silently return incorrect
results.

## Data Types and Naming

- All key/join columns (`Customer ID`, `Product ID`, `Location Key`, `Date`) were
  set to consistent, matching data types on both sides of each relationship (text
  for ID keys, Date for the date relationship) — mismatched types are a common
  cause of relationships silently failing to match rows.
- All queries and tables were renamed to clear business names (`FactSales`,
  `DimCustomer`, `DimProduct`, `DimLocation`, `DimDate`) rather than default names
  like `Query1`.
- Foreign/join key columns inside `FactSales` (`Customer ID`, `Product ID`,
  `Location Key`) were hidden from Report View (right-click → Hide in Report
  View) so report users only ever browse and filter using the clean fields
  exposed on the dimension tables, not raw keys.

## Modelling Challenges Encountered

1. No natural key for location. Unlike Customer and Product, the raw data had
   no single column that uniquely identified a location — `City` alone repeats
   across different states and countries (e.g. multiple "Springfield" entries).
   This was resolved by generating a surrogate key,
   `Location Key = Text.Combine({[Country], [State], [City]}, "|")`, added
   identically to both `DimLocation` and `FactSales` in Power Query so the two
   tables could be related on a guaranteed-unique field.

2. Two date fields competing for the date relationship. The fact table
   contains both `Order Date` and `Ship Date`, but a table can only have one
   *active* relationship to a given Date table at a time. `Order Date` was chosen
   as the active relationship because most analysis in this report (sales trends,
   YoY growth, seasonality) is order-driven rather than fulfilment-driven. An
   inactive second relationship was added from `DimDate[Date]` to
   `FactSales[Ship Date]`, which can be activated inside a specific DAX measure
   using `USERELATIONSHIP()` for any shipping-time-specific analysis (e.g. average
   shipping delay), without disturbing the default order-based time intelligence.

3. Potential one-to-many key conflicts in dimensions. Before finalizing
   `DimCustomer` and `DimProduct`, a check was run to confirm that no `Customer ID`
   or `Product ID` was ever associated with more than one `Segment` or
   `Product Name` respectively (see Section B, Transformation Log). Confirming this
   before deduplication was necessary to avoid silently breaking the one-to-many
   cardinality these relationships depend on.

## Model View
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20204059.png?raw=true)

# Section D: DAX & Business Calculations(20 marks)

## Overview

Fourteen DAX measures were created across the three required complexity levels —
Core Measures, Calculated Business Measures, and Advanced DAX — to turn the star
schema built in Section C into a genuine analytical solution rather than a set of
raw fields. All measures reference `FactSales` and, where time intelligence is
needed, the marked Date Table `DimDate`.

All measures below were created via **Model view (or Report view) → New Measure**,
and organized into a dedicated `_Measures` display folder / measure table for
readability in the Fields pane (optional but recommended: create a blank query
table named `_Measures` in Power Query with no columns, load it, and assign
measures to it).

---

## Level 1 — Core Measures

```DAX
Total Sales = SUM(FactSales[Sales])

Total Profit = SUM(FactSales[Profit])

Total Orders = DISTINCTCOUNT(FactSales[Order ID])

Total Quantity = SUM(FactSales[Quantity])

Average Discount = AVERAGE(FactSales[Discount])
```

These form the foundational building blocks every other measure is derived from.
`Total Orders` uses `DISTINCTCOUNT` rather than `COUNTROWS` because a single order
can span multiple line items (rows) in this dataset — counting rows would
overstate the number of actual orders.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20204605.png?raw=true)
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20205051.png?raw=true)

## Level 2 — Calculated Business Measures

```DAX
Profit Margin % =
DIVIDE([Total Profit], [Total Sales], 0)

Average Order Value =
DIVIDE([Total Sales], [Total Orders], 0)

Shipping Cost Ratio % =
DIVIDE(SUM(FactSales[Shipping Cost]), [Total Sales], 0)

Loss-Making Orders =
CALCULATE([Total Orders], FactSales[Profit] < 0)
```

`DIVIDE()` is used instead of the `/` operator throughout, since it safely returns
the specified fallback value (0) instead of throwing a divide-by-zero error when a
filter context has no sales (e.g. a slicer selection with no matching rows).
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20205416.png?raw=true)
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20205854.png?raw=true)

## Level 3 — Advanced DAX

```DAX
Previous Year Sales =
CALCULATE(
    [Total Sales],
    SAMEPERIODLASTYEAR(DimDate[Date])
)

YoY Sales Growth % =
DIVIDE(
    [Total Sales] - [Previous Year Sales],
    [Previous Year Sales],
    0
)

Sales Rank by Sub-Category =
RANKX(
    ALL(DimProduct[Sub-Category]),
    [Total Sales],
    ,
    DESC
)

High Discount Profit Impact =
VAR HighDiscountThreshold = 0.3
RETURN
    CALCULATE(
        [Total Profit],
        FactSales[Discount] >= HighDiscountThreshold
    )

Customer Segment Contribution % =
DIVIDE(
    [Total Sales],
    CALCULATE([Total Sales], ALL(DimCustomer[Segment])),
    0
)

Profit Category =
SWITCH(
    TRUE(),
    [Profit Margin %] >= 0.15, "High",
    [Profit Margin %] >= 0, "Low",
    "Loss"
)

Shipping Cost by Ship Date =
CALCULATE(
    SUM(FactSales[Shipping Cost]),
    USERELATIONSHIP(DimDate[Date], FactSales[Ship Date])
)
```

`Shipping Cost by Ship Date` demonstrates active use of the **inactive**
`DimDate`-to-`Ship Date` relationship established in Section C — it is only ever
"switched on" inside this specific measure via `USERELATIONSHIP()`, without
affecting the default Order-Date-based time intelligence used everywhere else in
the report.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20210547.png?raw=true)
---

## Documentation of the Six Most Important Measures

### 1. `Profit Margin %`

- What it calculates: Profit as a percentage of Sales — `[Total Profit] ÷ [Total Sales]`.
- Why it's useful: Sales volume alone is misleading; a category or region can
  generate high sales but still be unprofitable. This measure is the single most
  important profitability indicator in the report.
- Main DAX functions used: `DIVIDE()`.
- Filter context effect: Because it references `[Total Profit]` and
  `[Total Sales]` (both implicit `SUM` aggregations), the measure automatically
  recalculates for whatever filter context is applied — a single Category, a single
  Market, a date range from a slicer, or a cross-filter from clicking a bar chart.
  No explicit `CALCULATE()` is needed because the base measures already respond to
  row/visual-level filter context.
  Where used: KPI card on Page 1 (Executive Overview), and as a conditional
  colour/format trigger on the Category and Sub-Category bar charts on Page 2.

### 2. `YoY Sales Growth %`

- **What it calculates:** The percentage change in Total Sales compared to the same
  period one year earlier.
- Why it's useful: Raw sales totals don't reveal trend direction; this measure
  answers "are we growing or declining?" at a glance, which is central to the
  Executive Overview story.
- Main DAX functions used: `CALCULATE()`, `SAMEPERIODLASTYEAR()`, `DIVIDE()`.
- Filter context effect: `SAMEPERIODLASTYEAR()` shifts the date filter context
  applied by whatever is currently selected (a specific Year, Quarter, or Month
  from a slicer) back by exactly one year, then `CALCULATE()` re-evaluates
  `[Total Sales]` inside that shifted context. This relies entirely on `DimDate`
  being marked as an official, continuous Date Table — without it, the date shift
  would be unreliable.
- **Where used:** KPI card and trend line chart on Page 1.

### 3. `Sales Rank by Sub-Category`

- What it calculates: The rank position of the currently-filtered
  Sub-Category by Total Sales, from 1 (highest) downward.
- Why it's useful: Turns a raw sales table into an ordered, at-a-glance
  leaderboard — instantly showing which sub-categories matter most, used in Page 2's
  product analysis.
- Main DAX functions used: `RANKX()`, `ALL()`.
- Filter context effect: `ALL(DimProduct[Sub-Category])` deliberately removes
  any existing filter on Sub-Category so that `RANKX` can rank every
  Sub-Category against each other, rather than being restricted to whatever a
  slicer has already filtered down to — this is essential, since ranking only
  makes sense against the full comparison set.
- Where used: Table/bar chart on Page 2 (Detailed Product Analysis), sorted by
  this measure to show top and bottom performing sub-categories.

### 4. `Loss-Making Orders`

- What it calculates: A count of distinct orders where `Profit` is negative.
- Why it's useful: Directly supports the diagnostic question "how much of our
  business is actually losing money?" — pairs with the `Profitability Flag` column
  created in Power Query (Section B) for slicing.
- Main DAX functions used: `CALCULATE()`, boolean filter argument.
- Filter context effect: `CALCULATE()` layers an additional filter
  (`FactSales[Profit] < 0`) on top of whatever filter context already exists from
  slicers or visual interactions, so this measure always reflects "loss-making
  orders within the current view" rather than the loss count across the entire
  dataset.
- Where used: KPI card and breakdown chart by Sub-Category/Market on 
  (Diagnostic Analysis).

### 5. `Shipping Cost Ratio %`

- What it calculates: Total Shipping Cost as a percentage of Total Sales.
- Why it's useful: Reveals whether fulfilment costs are quietly eroding
  margin — a question raw Sales and Profit totals don't answer on their own,
  supporting the "why is profit lower than expected here?" diagnostic angle.
- Main DAX functions used: `SUM()`, `DIVIDE()`.
- Filter context effect: Recalculates per whichever Ship Mode, Region, or
  Market is selected, since both the numerator and denominator are simple
  aggregations that respond to the active filter context automatically.
- Where used: Bar/column chart comparing Ship Modes and Regions .

### 6. `Profit Category`

- What it calculates: Classifies the current filter context into "High",
  "Low", or "Loss" profitability tiers based on `Profit Margin %` thresholds.
- Why it's useful: Converts a continuous percentage into a simple, glanceable
  categorical label — useful for conditional formatting and quickly spotting
  problem areas without reading exact percentages.
- Main DAX functions used: `SWITCH()`, `TRUE()` pattern, dependent on the
  `Profit Margin %` measure.
- Filter context effect: Since it's built entirely on top of `[Profit Margin %]`
  (itself filter-context-sensitive), this measure automatically re-classifies for
  every Category, Sub-Category, Region, or any other filter applied to the visual
  it sits in — no additional `CALCULATE()` is required because the dependency
  chain already carries the filter context through.
- Where used: Conditional formatting / colour rules on the Category and
  Sub-Category visuals across Page 1 and Page 3, and as a legend/label field on the
  diagnostic breakdown chart.

# Section E: Professional Power BI Dashboards(20 marks)

## Overview

The report consists of **three pages**, each with a distinct analytical purpose,
designed to move the reader progressively from a high-level summary to a
root-cause investigation:

```
Page 1: Executive Overview  →  Page 2: Detailed Analysis  →  Page 3: Diagnostic Analysis
     "What happened?"              "Where/who?"                "Why? What needs attention?"
```

A single consistent theme, colour palette, and font set was applied across all
three pages (**View → Themes**), and one colour was locked to one meaning
throughout the entire report — for example, `Technology` is always the same shade
on every page, so the reader never has to re-learn the colour key when moving
between pages.

---

## Page 1: Executive Overview

**Purpose:** Allow a manager to understand overall performance within a few
seconds, with no drilling required.

**Visuals included:**
| Visual | Field(s) | Why this visual |
|---|---|---|
| KPI Cards | `Total Sales`, `Total Profit`, `Profit Margin %`, `Total Orders`, `YoY Sales Growth %` | Cards give an instant read of the five headline numbers before any interaction |
| Line chart | `Total Sales` & `Total Profit` by `DimDate[Year-Quarter]` | A line chart is the correct choice for trend-over-time data — shows growth/decline pattern at a glance |
| Filled/shape map | `Total Sales` by `DimLocation[Country]` | Geography is inherently spatial; a map communicates regional concentration faster than a table of country names |
| Bar chart | `Total Sales` by `DimProduct[Category]` | A small number of categories (3) is ideal for a simple bar chart comparison |
| Slicers | `DimDate[Year]`, `DimLocation[Market]`, `DimCustomer[Segment]` | Lets the manager immediately narrow the view to their area of interest without leaving the page |

**Layout notes:** Kept to 5 visual elements plus slicers — deliberately not
crowded, generous white space, KPI cards aligned in a single row across the top
so they read left-to-right like a scoreboard.
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20211020.png?raw=true)
---

## Page 2: Detailed Analysis — Product , Customer Analysis & Geographical Insight

**Purpose:** A deeper look at *which* products and *which* customers are driving
the numbers seen on Page 1.

**Visuals included:**
| Visual | Field(s) | Why this visual |
|---|---|---|
| Treemap | `Total Sales` by `Category` → `Sub-Category` | Treemaps communicate both hierarchy and relative size in one view — ideal for showing which sub-categories dominate within each category |
| Table/bar chart | Top 10 customers by `SUM(FactSales[Sales])`, using a Top N visual-level filter on `DimCustomer[Customer Name]` | A ranked list directly answers "who are our most valuable customers" — built directly from the fact table's `Sales` field with a Top N filter, no separate ranking measure required |
| Scatter plot | `Average Discount` (x-axis) vs `Profit Margin %` (y-axis), bubble size = `Total Sales`, by `Sub-Category` | A scatter plot is the correct visual for showing the *relationship* between two continuous variables — reveals whether heavier discounting correlates with lower margin |
| Donut/stacked bar chart | `Total Sales` by `DimCustomer[Segment]` | A small number of segments (3) suits a simple part-to-whole visual |

**Interactivity used on this page:**
- **Drill-down** enabled on the treemap and bar chart: `Category → Sub-Category → Product Name`, so a manager can click into "Furniture" and see exactly which sub-category or product is underperforming.
- **Cross-filtering:** clicking a Segment in the donut chart filters every other visual on the page (Power BI's default behaviour, verified to work cleanly with no conflicting bidirectional relationships from Section C).
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20211519.png?raw=true)
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20211902.png?raw=true)

## Page 3: Advanced/Diagnostic Analysis

**Purpose:** Investigate **why** certain areas underperform, not just what
happened — directly using the `Profitability Flag` column (Section B) and the
diagnostic DAX measures (Section D).

![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20212249.png?raw=true)
![This is an image](https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/main/Screenshot%202026-08-15%20212644.png?raw=true)

- 
# Key Insights, Recommendations & Storytelling

## Overview

This section summarizes what the dashboard reveals, translates those findings
into concrete business recommendations, and explains how the three report pages
work together to tell a single, coherent story — moving from what happened to
what should be done about it. All figures below were calculated directly from
the cleaned dataset and should match the KPI cards and visuals in the final
`.pbix` file.

---

## Key Insights

### 1. Overall performance is healthy but margin is thin
Across the full 2011–2014 period, the business generated $12.64M in total
sales and $1.47M in profit, a Profit Margin of 11.6%. Margin has
improved gradually year over year (11.0% in 2011 → 11.7% in 2014), alongside
consistent sales growth each year — but an 11–12% margin leaves little room
for error, meaning the loss-making pockets identified below matter more than
their dollar size alone suggests.

### 2. Furniture is the weakest-performing category — and Tables specifically lose money
`Technology` (13.99% margin) and `Office Supplies` (13.69% margin) are both
solidly profitable, but `Furniture` trails badly at just 6.94% margin —
roughly half the profitability of the other two categories on similar sales
volume. Drilling in, the Tables sub-category is the single worst performer
in the entire dataset: -8.47% margin, losing a combined $64,083 despite
$757,034 in sales. This loss is not isolated to one region — Tables lose money
in the EU (-$20,998), APAC (-$20,129), US (-$17,725), and LATAM (-$12,306)
markets alike, while only turning a small profit in Canada, EMEA, and Africa.

### 3. Heavy discounting is strongly associated with losses
There is a clear, near-linear relationship between discount level and
profitability:

| Discount Band | Margin % | Orders |
|---|---|---|
| 0% | 25.3% | 15,211 |
| 0–10% | 17.2% | 3,028 |
| 10–20% | 9.9% | 4,363 |
| 20–30% | -5.5% | 843 |
| 30–50% | -32.4% | 3,674 |
| 50%+ | -111.0% | 2,369 |

Every order discounted 20% or more turns a loss on average, and orders
discounted 50%+ lose more than the sale value itself. Roughly 6,900 orders
(about 27% of all orders) fall into these loss-making discount bands.

### 4. Losses are broad-based, not a handful of outliers
24.5% of all orders (12,544 of 51,290) are loss-making, totaling
-$920,646 in losses — an amount equivalent to nearly two-thirds of the
company's entire realized profit. This is too large and widespread to be
explained by a few unusual transactions; it points to a systemic discounting
and/or product-mix problem rather than isolated bad orders.

### 5. Fulfilment speed carries a real cost, but "Standard" shipping is comparatively efficient
Shipping cost as a share of sales varies meaningfully by ship mode: Same Day
(17.4%) and First Class (16.8%) are markedly more expensive relative to
sales than Standard Class (8.1%). Since Standard Class also carries the
largest share of total sales ($7.58M of $12.64M), the business is not overly
reliant on its most expensive fulfilment option — but faster shipping tiers
are eating a disproportionate share of revenue where they are used.

### 6. Geography shows strong performers and one clear underperformer
Margin by market ranges widely: Canada leads at 26.6% (on a small sales
base), followed by EU (12.7%) and US (12.5%). EMEA is the clear
laggard at just 5.5% margin — less than half the margin of every other major
market — despite not being the smallest market by sales ($806K, ahead of
Africa and Canada). EMEA warrants specific investigation as a market where
something other than pure sales performance is compressing profitability.

### 7. Customer segments are similarly profitable — this is not where the problem lies
`Consumer` (11.5%), `Corporate` (11.5%), and `Home Office` (12.0%) margins are
close to identical. This is itself a useful insight: segment is not a
meaningful driver of the profitability gap seen elsewhere in the data, so
diagnostic effort is better spent on category, discount, and geography than on
re-targeting customer segments.

---

## Recommendations

1. Review or restructure discount authorization above 20%.
   Since every discount band at 20% or higher averages a net loss, and the
   30%+ bands lose disproportionately more than the sale amount, discounting
   policy should require explicit approval (or be disallowed by default)
   beyond the 20% threshold — particularly the 50%+ band, which is actively
   destroying value on every order it touches.

 2.  Investigate the Tables sub-category specifically, not Furniture broadly.
   Since losses are concentrated in Tables and consistent across four separate
   markets, this looks like a product-level issue (cost structure, supplier
   pricing, or a category-wide discounting pattern specific to Tables) rather
   than a regional or seasonal one. A targeted cost/pricing review of Tables
   is likely to yield more improvement than a blanket Furniture strategy.

3. Conduct a focused review of the EMEA market.
   EMEA's margin is less than half that of comparable markets despite
   meaningful sales volume, which rules out "too small to matter." Its
   discount practices, cost base, and product mix should be compared directly
   against the EU and US for divergence.

4. Protect Standard Class as the default shipping option and reconsider Same Day/First Class pricing.
   Given Same Day and First Class shipping consume roughly double the sales
   share in shipping cost compared to Standard, either the shipping surcharge
   passed to premium-shipping customers should be revisited, or these options
   should be positioned/priced to better reflect their true cost.

5. Do not prioritize segment-based interventions.
   Because Consumer, Corporate, and Home Office margins are nearly identical,
   resources aimed at "fixing" a specific segment's profitability are unlikely
   to move the needle — the data points toward discount policy, product mix,
   and geography as the higher-leverage areas.

---

## Storytelling: How the Three Pages Build the Case

The dashboard is deliberately sequenced so a reader is led from headline
performance to a specific, actionable finding, rather than being handed a wall
of disconnected charts:

Page 1 (Executive Overview) — "What happened?"
Establishes that the business is growing and modestly profitable overall
(11.6% margin, steady YoY growth), giving the reader a baseline before any
problem is introduced. The Category bar chart and geographic map plant the
first visual clue — Furniture and EMEA already look weaker here, without yet
explaining why.

Page 2 (Detailed Analysis,Geographical insight) — "Where, and for whom?"**
Narrows the focus onto product and customer dimensions. The
Category → Sub-Category treemap drill-down isolates Tables as the specific
sub-category dragging Furniture down, while the Segment breakdown shows
segment is not where the story lives — actively ruling out a plausible but
incorrect explanation, which strengthens the credibility of the diagnosis that
follows.

Page 3 (Diagnostic Analysis) — "Why, and what needs attention?"
Directly tests the two remaining hypotheses raised by Pages 1–2: discounting
and fulfilment cost. The discount-vs-margin combo chart shows the loss
threshold appears right around 20% discount, and the loss-by-sub-category/
market breakdown confirms Tables losses are consistent across regions rather
than a single-market anomaly. The Shipping Cost Ratio chart closes the loop by
showing that fulfilment cost, while real, is a secondary factor compared to
discounting. The page ends with a dynamic, data-driven summary statement and a
drill-through path straight to the underlying loss-making orders — moving the
reader from insight directly to an auditable next action.


---








---

