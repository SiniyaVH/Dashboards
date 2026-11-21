# DataCo SMART Supply Chain Analytics

**Analytical Documentation Report: Inventory Management, Demand Analysis and Logistics Optimization**

| Item | Detail |
|---|---|
| Domain | Supply Chain Management |
| Dataset | DataCo SMART Supply Chain for Big Data Analysis (Mendeley Data) |
| Scope | 172,765 order lines, 62,897 unique orders, 20,261 customers (1 Jan 2015 to 31 Jan 2018) |
| Tools | Python, NumPy, Pandas, Matplotlib, Seaborn, Plotly |
| Status | Descriptive and exploratory analysis; no predictive modelling |

> **Reading note.** All figures in this report are taken from the computed outputs and charts in the notebook. Where the notebook's written commentary disagrees with its own computed output, the computed output is used and the difference is listed in [Appendix A](#appendix-a-validation-notes-on-the-notebook).

---

# 1. Project Overview

## 1.1 Project Title

**DataCo SMART Supply Chain Analytics: Inventory Management, Demand Analysis and Logistics Optimization**

## 1.2 Business Problem

The primary business problem is poor supply chain and delivery performance, which is causing substantial profit leakage despite strong sales and customer demand. The company has strong market demand but suffers from severe order fulfilment and logistics inefficiencies, leading to excessive delivery delays, financial losses, and a high number of loss-making transactions.

The problem shows up in the data in three ways:

| Symptom | Evidence from the notebook |
|---|---|
| Strong demand and sales | $35.2M total sales, 367,591 units, 20,261 customers, average order value $560 |
| Excessive delivery delays | 54.69% of orders arrive later than scheduled (34,400 of 62,897 orders) |
| Financial leakage | $3.7M of loss on loss-making lines, against $7.5M of positive profit, leaving a net profit of $3.8M |
| High number of loss-making transactions | 32,295 of 172,765 order lines (18.69%) have negative profit |

## 1.3 Objective

To identify the key factors contributing to supply chain inefficiencies, delivery delays, and profit losses by analysing sales, demand, inventory, order fulfilment, and logistics performance.

### Supporting objectives

1. Clean, preprocess and transform the dataset for accurate analysis.
2. Establish overall performance through business KPIs.
3. Analyse demand across products, categories, segments and regions.
4. Classify products as fast-, medium- or slow-moving from historical quantities.
5. Evaluate sales and profitability, including the role of discounts and prices.
6. Compare scheduled and actual shipping durations.
7. Examine delivery delays by shipping mode and region.
8. Turn the patterns into data-driven recommendations.

## 1.4 Tools & Technologies

| Technology | Purpose in this project |
|---|---|
| Python | Overall analytical workflow |
| NumPy | Conditional logic (`np.select`, `np.where`) and numerical operations |
| Pandas | Cleaning, type conversion, feature creation, grouping and aggregation |
| Matplotlib | Static charts (distributions, KPI dashboard, bar charts) |
| Seaborn | Statistical charts (histograms, heatmaps, scatter plots) |
| Plotly | Interactive pie, bar, line and scatter charts |
| Jupyter Notebook | Development and documentation of the workflow |

## 1.5 Dataset Source

| Item | Detail |
|---|---|
| Name | DataCo SMART Supply Chain for Big Data Analysis |
| Source | Mendeley Data |
| Link | https://data.mendeley.com/datasets/8gx2fvg2k6/5 |
| Raw size | 180,519 rows and 53 columns |
| Cleaned size | 172,765 rows and 43 columns |
| Period | Orders from 1 January 2015 to 31 January 2018 |
| Unit of observation | One order line (`Order Item Id`); one order (`Order Id`) can have several lines |
| Coverage | 62,897 orders, 20,261 customers, 118 products, 50 categories, 11 departments, 5 markets, 23 regions, 164 countries |

---

# 2. Data Preparation

## 2.1 Phase 1 Workflow

The workflow below follows the order of the notebook.

### Step 1: Load and inspect

The file was read with `latin1` encoding. Shape, data types, descriptive statistics, unique counts and missing values were checked before any change was made.

### Step 2: Missing values and duplicates

| Check | Result |
|---|---|
| Product Description | 100% missing (empty column) |
| Order Zipcode | 86.24% missing |
| Customer Lname | 8 missing values (0.004%) |
| Customer Zipcode | 3 missing values (0.002%) |
| Duplicate rows | 0 |
| Order Item Id | 180,519 unique values (every row is a distinct line) |
| Order Id | 65,752 unique values (so one order can hold several lines) |

### Step 3: Remove redundant columns

A pairwise check (`Series.equals`) found seven columns that exactly duplicate another column.

| Dropped | Identical to (kept) |
|---|---|
| Benefit per order | Order Profit Per Order |
| Sales per customer | Order Item Total |
| Category Id | Product Category Id |
| Customer Email | Customer Password (constant; dropped separately) |
| Customer Id | Order Customer Id (renamed Customer Id) |
| Order Item Cardprod Id | Product Card Id (dropped separately) |
| Order Item Product Price | Product Price |

Ten further columns with no analytical value or privacy risk were then dropped: Customer Password, Customer Zipcode, Order Zipcode, Product Description, Product Image, Product Card Id, Customer City, Customer Country, Latitude and Longitude.

### Step 4: Feature creation and renaming

| New or changed field | Definition |
|---|---|
| Customer Name | Customer first and last name combined; the two source columns were dropped |
| Order Date, Shipping date | Converted to datetime |
| Product Status, Late_delivery_risk | Converted to boolean |
| Actual Shipping Days | Shipping date minus order date, in days |
| Scheduled Shipping days | Renamed from `Days for shipment (scheduled)` |
| Delivery Status | Rebuilt with `np.select`: Advanced Shipping (scheduled > actual), Late Delivery (scheduled < actual), Shipping On Time |
| Delivery Risk | 1 if Late Delivery, otherwise 0 |
| Delay | Actual minus scheduled shipping days |
| Is delay | True when Delay is greater than 0 |
| Profitability | Profit, Loss or Break-even from the sign of `Order Profit Per Order` |
| Order Year, Order Month, Month_Name, Order Day, Order Hour | Calendar fields from Order Date |
| Discount | Renamed from `Order Item Discount` (a currency amount, not a rate) |

### Step 5: Filter

7,754 rows with delivery status "Shipping canceled" were removed because they have no completed shipment to analyse. This leaves **172,765 rows**.

### Step 6: Validate

The cleaned data has no missing values in the retained columns, 20 numeric and 19 text columns, and 43 columns in total. Average daily demand was computed as 367,591 units over 1,127 distinct order dates, which gives **326.17 units per order day**.

## 2.2 Facing Issues & Challenges

| Issue | Why it matters | How it was handled |
|---|---|---|
| Several columns are 86% to 100% empty | Imputing them would invent data | Dropped; Order Zipcode and Product Description have no analytical use here |
| Seven duplicate columns | Repeated fields inflate the column count and risk double use | Detected by a pairwise comparison and dropped |
| Orders versus order lines | 180,519 lines belong to 65,752 orders; counting rows as orders overstates volume | Order-level KPIs use `Order Id` unique counts; line-level KPIs say "records" |
| Calculated and supplied shipping days differ | 4,657 rows (2.6%) differ by 1 day because the calculation counts whole days from timestamps | The calculated field was kept and the supplied column dropped; this can move a few lines between "on time" and "late" |
| Delivery status labels differ from the supplied ones | The supplied labels include cancelled shipments and use a different rule | Status was rebuilt from scheduled and actual days so it is consistent with the Delay field |
| Constant column | `Product Status` is False on every row after conversion, so it carries no information | Left in place, not used |
| Heavy right tail in sales, discount and profit | Averages are pulled by a few extreme lines (profit ranges from âˆ’4,274.98 to +911.80) | Medians and distributions were reviewed alongside means |
| Metric defined inconsistently | A "delivery risk %" was computed by dividing late **lines** by unique **orders**, which gives values above 100% | Replaced in this report by the order-level late delivery % (see Appendix A) |
| Sparse final months | October 2017 to January 2018 show far fewer orders than earlier months | Flagged as a data completeness question before any forecasting |

---

# 3. Analysis Performed & Key Findings

## 3.1 Key Business Questions

| # | Business question | Analysis used |
|---|---|---|
| 1 | How large is the business, and how profitable is it? | KPI dashboard |
| 2 | How many transactions lose money, and where? | Profit distribution; loss by category and shipping mode |
| 3 | Do discounts or prices explain losses? | Discount versus profit scatter; correlation matrix |
| 4 | Which products and categories have high or low demand? | Top and bottom products; category demand; movement classification |
| 5 | Where is demand geographically concentrated? | Market, region and country demand |
| 6 | How does demand change over time? | Monthly demand by year |
| 7 | How do scheduled and actual shipping times compare? | Scheduled versus actual by shipping mode |
| 8 | Which shipping modes and regions are late most often? | Late % by mode; regional late %; delay heatmap |
| 9 | How much profit sits on delayed orders? | Profit at Risk by mode and region |

## 3.2 Key Performance Indicators

| KPI | Value | Definition |
|---|---|---|
| Total sales | $35.2M | Sum of `Sales` |
| Unique orders | 62,897 | Unique `Order Id` |
| Order lines | 172,765 | Rows after cleaning |
| Customers | 20,261 | Unique `Customer Id` |
| Units sold | 367,591 | Sum of `Order Item Quantity`
 |
| Average order value | $560 | Mean of sales summed per order |
| Average sales per line | $204 | Mean of `Sales` |
| Positive profit | $7.5M | Sum of profit on profitable lines |
| Total loss | $3.7M | Sum of the absolute value of negative profit |
| Net profit | $3.8M | Positive profit minus loss (about 10.8% of sales) |
| Loss-making lines | 32,295 (18.69%) | Lines with negative profit |
| Loss-making orders | 4,401 (7.0%) | Orders whose highest line profit is negative |
| Late orders | 34,400 (54.69%) | Orders with at least one line delayed |
| On-time orders | 21.26% | Delay equal to 0 |
| Early orders | 24.05% | Delay below 0 |
| Average delay (late lines) | 1.65 days | Mean of Delay where Delay > 0 |
| 90th percentile delay | 3.00 days | Same group |
| Average daily demand | 326.17 units | Units divided by distinct order dates |

## 3.3 Important Findings with Visualizations

### Finding 1: Delivery lateness is the largest operational problem

More than half of all deliveries miss their schedule. On line records, **54.71% are late** (94,523 lines), 24.07% are early (41,592) and only 21.21% are on time (36,650).

**Visual:** a donut chart of delivery status, with the late slice pulled out, and a KPI dashboard of 12 cards.

### Finding 2: The premium shipping modes are the ones failing their promise

| Shipping mode | Share of lines | Scheduled days | Actual days | Late orders |
|---|---|---|---|---|
| First Class | 15.3% | 1 | 2 | **100.00%** |
| Second Class | 19.6% | 2 | 4 | **79.99%** |
| Standard Class | 59.7% | 4 | 4 (range 2 to 6) | 39.85% |
| Same Day | 5.4% | 0 | 0 | 0.00% |

First Class is promised in 1 day and always takes 2. Second Class is promised in 2 days and averages 4. Standard Class meets its average promise but spreads from 2 to 6 days against a 4-day promise, which is why about 40% of its orders are late. Same Day never misses.

Overall, the average scheduled duration is 2.93 days against about 3.5 days actual, a gap of roughly 0.55 days.

**Visuals:** grouped bar of scheduled versus actual days by mode; bar chart of late % by mode.

### Finding 3: Delay is systemic across the network, not regional

Late delivery % by region sits in a narrow band, from 50.83% (Canada) to 58.66% (Central Africa). Most regions are between 52% and 56%.

| Region group | Late delivery % |
|---|---|
| Highest | Central Africa 58.66%, Southeast Asia 55.97%, Eastern Europe 55.96%, South of USA 55.93% |
| Typical | Western Europe 55.79%, Central America 54.71%, Central Asia 54.06% |
| Lowest | Canada 50.83%, Southern Africa 52.27%, West Africa 52.48% |

The average-delay heatmap by region and mode shows the same story. Second Class averages roughly 1.9 to 2.2 days late in every region (highest in Central Asia at 2.22), First Class about 1 day late everywhere, and Same Day and Standard Class close to zero. The driver is the **shipping mode**, not the geography.

**Visuals:** bar chart of late % by region; region-by-mode heatmap of average delay.

### Finding 4: Loss-making lines are spread evenly, and are not explained by shipping mode or discount

| Shipping mode | Loss-making lines | Loss rate (derived) |
|---|---|---|
| Standard Class | 19,439 | 18.8% |
| Second Class | 6,265 | 18.5% |
| First Class | 4,832 | 18.2% |
| Same Day | 1,759 | 18.9% |

- Loss rates are almost identical across modes, even though lateness ranges from 0% to 100%.
- Mean discount is 20.66 on loss lines and 20.68 on profitable lines. There is no gap.
- The correlation matrix agrees: discount rate versus profit is âˆ’0.02, and actual or scheduled shipping days versus profit are 0.00.
- Large categories all lose on about 18.6% to 18.9% of lines. The highest rates are in small categories: Men's Clothing 24.24% (198 lines), Health and Beauty 23.41% (346), Video Games 21.77% (804) and Golf Shoes 21.71% (502).
- The most loss-making lines by count are in the biggest categories: Cleats 4,389, Men's Footwear 3,993, Women's Apparel 3,745, Indoor/Outdoor Games 3,471 and Fishing 3,047.
- Profit per line is heavily skewed: mean 22.03, median 31.52, standard deviation 104.36, with a long negative tail down to âˆ’4,274.98.

**Interpretation.** On this data, late delivery and loss-making are two separate problems that both exist at scale. The data does **not** show that delays cause the losses. What drives the loss tail is still unexplained and needs cost-level detail.

**Visuals:** profit-per-order histogram with box plot; discount versus profit scatter; sales versus profit scatter by segment.

### Finding 5: Profit at Risk is large but mostly follows volume

**$4.11M, or 54.71% of all positive profit**, sits on delayed lines. By mode it is roughly $1.8M for Standard Class, and $1.2M each for Second Class and First Class. By region it is highest in Western Europe and Central America (about $0.65M each), then South America (about $0.33M). The ranking mirrors order volume, which fits the finding that delay is spread evenly.

Profit at Risk is an exposure measure (positive profit on late lines), not realised loss.

**Visuals:** bar charts of Profit at Risk by shipping mode and by region.

### Finding 6: Demand is concentrated in a few products and categories

| Rank | Product | Units |
|---|---|---|
| 1 | Perfect Fitness Perfect Rip Deck | 70,575 |
| 2 | Nike Men's Dri-FIT Victory Golf Polo | 60,232 |
| 3 | O'Brien Men's Neoprene Life Vest | 55,333 |

| Rank | Category | Units | Revenue |
|---|---|---|---|
| 1 | Cleats | 70,611 | $3.82M |
| 2 | Women's Apparel | 60,232 | $2.71M |
| 3 | Indoor/Outdoor Games | 55,333 | $2.49M |
| 4 | Cardio Equipment | 35,937 | $3.17M |
| 5 | Shop By Sport | 31,329 | $1.13M |

- Revenue and volume do not line up. **Fishing** is the top revenue category ($5.96M) on only 16,595 units, while Cleats leads on volume.
- The lowest-demand products sell between 8 and 33 units: Bushnell Pro X7 Jolt Slope Rangefinder (8), Bowflex SelectTech 1090 Dumbbells (10), SOLE E25 Elliptical (10), SOLE E35 Elliptical (15).
- Splitting the 118 products into thirds by quantity gives 33.1% fast, 33.1% medium and 33.9% slow. This is a relative ranking by construction, not evidence of balanced inventory.
- 54.9% of lines are single-unit purchases.

**Visuals:** top 10 and bottom 10 product bars; category demand and revenue bars; movement donut.

### Finding 7: Demand is geographically concentrated

| Market | Share of units |
|---|---|
| LATAM | 29.4% |
| Europe | 27.4% |
| Pacific Asia | 21.9% |
| USCA | 14.7% |
| Africa | 6.7% |

- Top regions: Central America (about 59.5k units), Western Europe (about 53.9k) and South America (about 31k). Central Asia is lowest (about 1.2k).
- Top countries by units: United States (about 52.0k), Mexico (27.5k), France (26.4k), Germany (19.0k), Brazil (16.5k), Australia (16.1k), United Kingdom (14.6k).
- By customer segment, Consumer is 51.8% of units (190,522), Corporate 30.4%, Home Office 17.8%.

**Visuals:** market donut; region and country bars; segment donut.

### Finding 8: Monthly demand is stable until a sharp drop in late 2017

Demand runs at about 10,000 to 11,500 units a month in 2015 and 2016, with no strong seasonality. In 2017 it follows the same pattern to September (about 9,900), then falls to roughly 2,000 to 2,400 units in October, November and December. January 2018 shows about 2,000 units and is the only 2018 month.

By year, order lines are 59,954 (2015), 59,926 (2016), 50,848 (2017) and 2,037 (2018). A drop this abrupt looks more like incomplete data capture than a business change, and needs confirming before any forecast is built.

**Visual:** line chart of monthly demand, one line per year.

### Finding 9: Correlation analysis

| Pair | r | Reading |
|---|---|---|
| Sales and Product Price | 0.79 | Price drives line sales |
| Order Quantity and Product Price | âˆ’0.48 | Dearer items are bought in smaller quantities |
| Actual and Scheduled Shipping Days | 0.53 | Longer promises take longer, as expected |
| Sales and Profit | 0.13 | Higher sales do not mean proportionally higher profit |
| Sales and Quantity | 0.11 | Weak |
| Discount Rate with every other variable | âˆ’0.02 to 0.00 | No linear relationship |
| Shipping days with Sales, Profit, Price | 0.00 to 0.01 | Logistics and financial performance move independently |

**Visual:** annotated correlation heatmap.

---

# 4. Limitations & Assumptions

## 4.1 Analytical Limitations

| # | Limitation | Effect on the findings |
|---|---|---|
| 1 | **No stock or replenishment data.** The file holds no warehouse balances, reorder points or stock-outs | "Inventory" analysis is movement of historical sales quantity, not true inventory turnover |
| 2 | **Demand equals fulfilled orders.** Lost sales and unmet demand are not recorded | Demand is likely understated for stocked-out items |
| 3 | **No cost structure.** Product cost, carrier cost, penalties and refunds are absent | Root causes of loss lines cannot be separated into pricing, cost or mix |
| 4 | **No carrier, warehouse or route fields** | Delay causes can be located (mode) but not explained |
| 5 | **Truncated final months.** October 2017 to January 2018 look incomplete | Trend and seasonality conclusions are unreliable for 2017 onward |
| 6 | **Association only.** Analysis is descriptive, with no hypothesis tests or models | No claim of cause can be made |
| 7 | **Scheduled days are a fixed promise per mode** (0, 1, 2 or 4 days) | Lateness largely reflects how the promise is set, not only how the network performs |
| 8 | **Whole-day shipping calculation** | About 2.6% of lines may sit one day off the supplied value |
| 9 | **Historical, single-company data (2015 to early 2018)** | May not reflect current operations |
| 10 | **Order-status mix.** About 54% of lines (93,658) are pending payment, pending, processing, on hold or in payment review, and only 59,491 are COMPLETE | Profit on
 non-completed orders may still change |

## 4.2 Key Assumptions

| # | Assumption |
|---|---|
| 1 | Each row is one order line and is treated as an independent record for totals |
| 2 | `Order Profit Per Order` is correct as recorded |
| 3 | Quantity sold is a valid proxy for demand |
| 4 | An order is "late" if any of its lines has actual days above scheduled days |
| 5 | Shipping canceled rows carry no delivery performance and can be excluded |
| 6 | A line is "loss-making" when profit is below zero; zero profit is break-even |
| 7 | Fast, medium and slow movers are equal-sized thirds of products ranked by units |
| 8 | Delay and profit are compared as associations; no causal assumption is made |
| 9 | Dollar values are in the dataset's currency units (shown as $) |

---

# 5. Recommendations & Conclusion

## 5.1 Actionable Recommendations

| Priority | Recommendation | Supporting evidence |
|---|---|---|
| 1 | **Audit First Class and Second Class carriers, or reset the promise.** Either meet the 1-day and 2-day commitments or publish realistic dates (2 and 4 days) | First Class 100% late; Second Class 79.99% late; both pay for speed |
| 2 | **Reduce variability in Standard Class** (target a tighter 3 to 5 day window) and re-test the 4-day promise | 39.85% late on 103,153 lines; actual ranges 2 to 6 days |
| 3 | **Fix lateness at the mode and process level first, not region by region** | Late % is 50.8% to 58.7% in every region |
| 4 | **Use Same Day as the operational benchmark** and document what it does differently | 0% late across 9,293 lines |
| 5 | **Run a cost-level loss investigation** (price versus product cost, shipping charge, refunds) starting with the extreme negative-profit tail | Losses are ~18% in every mode and category; discount and shipping days do not explain them |
| 6 | **Prioritise replenishment for the top SKUs and categories**, with safety stock for Perfect Rip Deck, Dri-FIT Polo and Life Vest | 70,575, 60,232 and 55,333 units |
| 7 | **Review the lowest-demand products before discontinuing them**, checking margin and price point | 8 to 33 units each, often premium items |
| 8 | **Verify the Q4 2017 data drop** before using this data for forecasting | Demand falls from about 9.9k to about 2.0k units in a month |
| 9 | **Build an SLA scorecard** by mode and region with late %, average delay and 90th percentile delay | KPIs already exist in the notebook |
| 10 | **Standardise definitions** so orders, lines and risk percentages are never mixed | See Appendix A |

The financial benefit of these actions is not quantified. Any estimate should come from a simulation that re-sets promised days or removes late shipments and recalculates the results, rather than from the Profit at Risk figure, which measures exposure and not realised loss.

## 5.2 Suggestions for Future Work

- Add cost, carrier and warehouse fields so that loss and delay causes can be separated.
- Test the delay and profit relationship with a statistical test and a regression or tree model.
- Build a delay prediction model using mode, region, product and order timing.
- Add a demand forecast once the late-2017 gap is resolved.
- Extend inventory analysis with stock levels, lead times and reorder points.

## 5.3 Conclusion

The DataCo data describes a business with strong, stable demand and healthy gross volume ($35.2M sales, $3.8M net profit) but a serious service problem: 54.69% of orders are delivered later than promised. The failure is concentrated in the premium services. First Class and Second Class miss their promise on 100% and 80% of orders, and Standard Class misses on about 40% because its delivery times are too variable. Because the pattern is the same in every region, the fix belongs in shipping-mode operations and promise setting.

Loss-making is a separate, equally large issue. Roughly one line in five loses money, at a nearly identical rate across shipping modes and categories, and discount and shipping time show no relationship with profit. The notebook therefore supports the business problem as two parallel inefficiencies, delivery performance and transaction profitability, and does not prove that one causes the other.

The most valuable next step is better data: cost, carrier and stock fields, and a clean 2017 to 2018 record. With those, the project can move from describing where the problems are to explaining and predicting them.

---

# 6. Project Resources

## 6.1 Links & References

| Resource | Description |
|---|---|
| Dataset | DataCo SMART Supply Chain for Big Data Analysis, Mendeley Data: https://data.mendeley.com/datasets/8gx2fvg2k6/5 |
| Analysis notebook | `DataCo_SMART_Supply_Chain_Analytics.ipynb` (cleaning, feature engineering, EDA, inventory, logistics and demand analysis) |
| Companion report | `DataCo_Supply_Chain_Analytics_Project_Report.pdf` |
| Libraries | Python, NumPy, Pandas, Matplotlib, Seaborn, Plotly |

---

# Appendix A: Validation Notes on the Notebook

While preparing this report, the notebook's written commentary was compared with its computed outputs. Please review these items, because several are visible in the notebook narrative.

| # | Notebook commentary | Computed output | Used in this report |
|---|---|---|---|
| 1 | "172,765 orders" | 172,765 is the **line** count; unique orders are 62,897 | 62,897 orders, 172,765 lines |
| 2 | "Total profit $7.5M, 21.3% of sales" | $7.5M is positive profit only; losses are $3.7M and net profit is $3.8M | Net profit $3.8M (about 10.8%) |
| 3 | "32,295 loss-making records, roughly 18.7% of total orders" | 32,295 is lines (18.69%); loss-making **orders** are 4,401 | Both, labelled correctly |
| 4 | "$2.1M in losses due to delays" | No cell computes this figure; Profit at Risk is $4.11M of positive profit | Not used |
| 5 | Delivery risk "276.1% / 220.19% / 109.01%" | Late lines divided by unique orders, so values above 100% are not valid percentages | Order-level late % (100.00 / 79.99 / 39.85 / 0.00) |
| 6 | Regional "variance" 0.42 to 0.62 | The chart shows late % from 50.83 to 58.66 | Chart values |
| 7 | Top product "about 76.6k units" | Computed 70,575 units | 70,575 |
| 8 | Country chart described as "sales exceeding 4.5M" | The chart plots demand in units (United States about 52.0k) | Units |
| 9 | "Delay losses are about 28% of profit" | Depends on the unsupported $2.1M figure | Not used |
| 10 | The notebook framing that discounts or shipping drive loss lines | Loss rate is flat across modes; mean discount is equal on loss and profit lines | Reported as no observed relationship |
