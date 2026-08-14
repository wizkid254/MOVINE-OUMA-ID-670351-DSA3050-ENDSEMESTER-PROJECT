# MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT
# DSA3050 – Business Intelligence & Data Visualization
## Section A: Dataset Selection & Understanding (10 marks)



### 1. Source of the Dataset

The dataset is the **Global Superstore** dataset, a widely used retail transactions dataset originally distributed as a Tableau sample dataset and commonly hosted on Kaggle (e.g. "Global Superstore Dataset" by various uploaders). It is a real-world-style dataset built from an international superstore's order history.



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

7.  # Section B: Data Cleaning & Transformation (Power Query)

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

---

### 2. Removing a raw spreadsheet index column

Problem: `Row ID` is simply a sequential spreadsheet row number, not a business
key or attribute.

Transformation: Home → Remove Columns → `Row ID`.

Reason: Power BI's own row context and the `Order ID` field already identify
transactions; a meaningless numeric index adds no analytical or modelling value.

Result: One fewer redundant column in the Fact table.

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

---

## Additional Notes

- **No duplicate rows** were found in the raw fact-level data (`Table.Distinct` on
  the full row set returned zero removed rows), so no full-row deduplication was
  required at the fact level — only at the dimension level, where repetition is
  expected and intentional prior to deduplication.
- **No null values** were present in any column of the raw dataset, so no
  null-handling/imputation strategy was required for this dataset.
  
  (




---

