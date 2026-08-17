# Supply Chain Visibility system with Optimization analytics

An end-to-end Power BI project that unifies inventory, supplier, warehouse, transportation, and delivery data into a single, interactive, KPI-driven dashboard system for supply chain decision-making.

---

##  Problem Statement

Supply chain operations span multiple interconnected processes — inventory management, supplier performance, transportation, warehousing, and delivery. When each of these is monitored separately, operational inefficiencies stay hidden until they impact the bottom line. This project solves that by integrating all supply chain data into one Power BI platform, enabling real-time monitoring, performance analysis, and data-driven decision-making.

---

##  Objective

Build a single, interactive Power BI system that turns raw logistics data into actionable, KPI-driven decisions across every stage of the supply chain:

1. **Unified Visibility** – Integrate inventory, supplier, warehouse, transportation, and delivery data into one model.
2. **Inventory Optimization** – Track turnover ratios and flag slow-moving and dead stock.
3. **Delivery Efficiency** – Monitor on-time performance and identify regional delivery bottlenecks.
4. **Supplier Benchmarking** – Score and rank suppliers on reliability, quality, and lead time.
5. **Cost Control** – Compare transportation cost by route, carrier, and shipping mode.
6. **Warehouse Utilization** – Surface capacity and throughput gaps across all warehouses.

---

##  Dataset

**DataCo Global Supply Chain Dataset**
A real-world logistics dataset covering orders, products, customers, shipments, delivery status, sales performance, and geographic distribution — enabling end-to-end analysis of inventory, delivery, transportation, and fulfillment across multiple regions.

 **Dataset link:** [https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)

**Key tables used:** Orders Data, Product Data, Customer Data, Shipment Data, Delivery Status Data, Geographic Location Data

Additional supporting data:
- Supplier data loaded from a supplementary spreadsheet source
- Warehouse and inventory data derived to simulate stock, capacity, and reorder scenarios

---

##  Tools & Technologies

- Power BI Desktop
- Power Query Editor (M / ETL transformations)
- DAX (Data Analysis Expressions)
- PostgreSQL (Neon — cloud Postgres) as the primary data source
- Microsoft Excel / Google Sheets (supplementary source)
- GitHub (version control)

---

##  Data Modeling — Star Schema

The project uses a **star schema** with a single central fact table connected to ten dimension tables, optimized for fast, flexible Power BI reporting.

**Fact Table**
- `Fact_table` — the transactional grain: one row per order line, holding measures like sales, profit, quantity, discount, shipping dates, and foreign keys to every dimension.

**Dimension Tables**
| Dimension | Purpose |
|---|---|
| `Dim_Customer` | Customer identity, location, and segment |
| `Dim_Product` | Product name, price, status, category/department keys |
| `Dim_Category` | Product category reference |
| `Dim_Department` | Product department reference |
| `Dim_Shipping` | Shipping mode, delivery status, scheduled vs. real shipping days |
| `Dim_Location` | Market, order city/country/region/state |
| `Dim_Date` | Full calendar table (year, month, quarter, week, day, day name) |
| `Dim_Supplier` | Supplier lead time, quality score, reliability |
| `Dim_Warehouse` | Warehouse capacity, region, utilization |
| `Dim_Inventory` | Stock quantity, safety stock, reorder level, restock date |

### How the model was built — step by step

1. **Connect to source data** — imported the base dataset into Power BI via PostgreSQL (Neon), with a Google Sheets connection for supplier data.
2. **Land the data in Power Query** and rename the base import to `Fact_table`.
3. **Build each dimension by duplication** — for each dimension (Customer, Product, Category, Department, Shipping, Location, Department), duplicate `Fact_table`, keep only the relevant columns, and remove duplicate rows on the natural key (e.g. `Customer_id`, `Product_card_id`, `Category_id`).
4. **Generate surrogate keys where needed** — for composite dimensions like Shipping and Location, added an Index column (starting from 1) to create a clean `Shipping_id` / `Location_id`, then merged that key back into `Fact_table` via a left-outer merge query and expanded only the new ID column.
5. **Build the Date dimension with DAX**, spanning the full order-to-shipping date range:
```DAX
   Dim_Date = CALENDAR(
       MIN(Fact_table[order_date_(dateorders)]),
       MAX(Fact_table[shipping_date_(dateorders)])
   )
```
   Enriched with `Year`, `Month`, `Month Number`, `Quarter`, `Week`, `Day`, and `Day Name` calculated columns.
6. **Build supporting fact/dimension tables** for the operational layer:
   - `Dim_Warehouse` (capacity, region, utilization %) by selecting warehouse-related columns and removing duplicates on `Warehouse ID`.
   - `Dim_Inventory` (stock qty, safety stock, reorder level, last restock date) with a composite `Inventory ID` custom column:
