# MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT
# DSA3050 – Business Intelligence & Data Visualization
## Section A: Dataset Selection & Understanding (Draft)



### 1. Source of the Dataset

The dataset is the **Global Superstore** dataset, a widely used retail transactions dataset originally distributed as a Tableau sample dataset and commonly hosted on Kaggle (e.g. "Global Superstore Dataset" by various uploaders). It is a real-world-style dataset built from an international superstore's order history.
![image alt] (image-url)https://github.com/wizkid254/MOVINE-OUMA-ID-670351-DSA3050-ENDSEMESTER-PROJECT/blob/6cd2ba4c6d7789d8e253e8f7ee21fb7fc7438c8c/Screenshot%202026-08-13%20235048.png


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
6. *(Optional 6th)* Which sub-categories are consistently loss-making across multiple markets, indicating a structural/pricing issue rather than a one-off?

---

