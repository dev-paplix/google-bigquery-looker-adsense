# Mastering Data Exploration, LookML & Visualization with Google Looker

A comprehensive, end-to-end technical manual aligned with the **1-Day Google Looker Workshop**. This curriculum covers enterprise BI architecture, semantic modeling with **LookML** (Projects, Models, Views, Joins, Symmetric Aggregates, Persistent Derived Tables, Parameters, and Liquid Variables), self-service **Explores**, **Table Calculations**, **Looker Dashboards**, **Looker Studio (Data Studio)** integration, and enterprise governance.

---

## Course Curriculum Roadmap & Visual Architecture

```
+---------------------------------------------------------------------------------------------------+
|                        1-DAY GOOGLE LOOKER & LOOKML WORKSHOP ROADMAP                              |
+---------------------------------------------------------------------------------------------------+
|  [09:00 - 10:30]  MODULE 1: Looker Architecture & Data Modeling Principles                        |
|                   - Semantic Layer vs. Traditional BI | Projects, Models, Views, Explores, Looks  |
+---------------------------------------------------------------------------------------------------+
|  [10:45 - 12:30]  MODULE 2: Exploring Data & Creating Powerful Visualizations                     |
|                   - Field Picker | Filtering & Pivoting | Table Calculations | Visualization Types |
+---------------------------------------------------------------------------------------------------+
|  [01:30 - 03:30]  MODULE 3: Interactive Dashboards & LookML Mastery (Part 1 & Part 2)             |
|                   - Dashboard Filters & Cross-Filtering | LookML Joins & Symmetric Aggregates      |
|                   - SQL & Native Derived Tables (PDTs) | LookML Parameters & Liquid Templating    |
+---------------------------------------------------------------------------------------------------+
|  [03:45 - 05:00]  MODULE 4: Looker Studio Integration, Governance & Enterprise Capstone           |
|                   - Connecting LookML to Looker Studio | Alerts, Schedules & Role-Based Security  |
+---------------------------------------------------------------------------------------------------+
```

### End-to-End Platform Architecture

Unlike traditional BI tools where every analyst writes separate SQL queries inside isolated reports, **Google Looker** centralizes business logic in a Git-backed **LookML Semantic Layer** that dynamically compiles optimized SQL against your cloud data warehouse:

```
+-----------------------------------------------------------------------------+
|                         CLOUD DATA WAREHOUSE (SQL)                          |
|        `ecommerce.orders` (100 rows)  <--->  `ecommerce.users` (30 rows)    |
+--------------------------------------+--------------------------------------+
                                       |
                        Live In-Database SQL Execution
                                       |
                                       v
+-----------------------------------------------------------------------------+
|                    GOOGLE LOOKER SEMANTIC LAYER (LookML)                    |
|  +-----------------------------------+-----------------------------------+  |
|  |         MODELS (.model.lkml)      |          VIEWS (.view.lkml)       |  |
|  |  - Database Connection            |  - Dimensions (Group-Bys, Tiers)  |  |
|  |  - Explore Definitions            |  - Measures (Sums, Counts, Avg)   |  |
|  |  - SQL Joins & Relationships      |  - Derived Tables & Liquid Logic  |  |
|  +-----------------------------------+-----------------------------------+  |
|                         Git Version Control (Dev / Prod)                    |
+-------------------+-------------------------------------+-------------------+
                    |                                     |
       Native Looker Query Engine            Looker BI Connector API
                    |                                     |
                    v                                     v
+---------------------------------------+ +-----------------------------------+
|      GOOGLE LOOKER PRESENTATION       | |     LOOKER STUDIO (DATA STUDIO)   |
|  - Self-Service Explores              | |  - Direct Explore Connection      |
|  - Pivot Tables & Table Calculations  | |  - Inherited LookML Dimensions    |
|  - Saved Looks & Native Dashboards    | |  - Custom Report Canvases         |
|  - Data Alerts & Scheduled Deliveries | |  - Multi-Source Report Blending   |
+---------------------------------------+ +-----------------------------------+
```

---

## Module 1: Looker Architecture & Data Modeling Principles

### 1.1 How Looker Generates SQL Dynamically
When a user selects fields in the Looker UI, Looker translates those selections into clean, dialect-specific SQL:

```
  [User Selects in UI]               [LookML Definitions]               [Compiled Warehouse SQL]
  +------------------+               +------------------+               +------------------------+
  | Dimension:       | ------------> | dimension:       | ------------> | SELECT                 |
  |   Country        |               |   country        |               |   orders.country,      |
  | Measure:         |               | measure:         |               |   SUM(orders.price)    |
  |   Total Revenue  |               |   total_revenue  |               | FROM ecommerce.orders  |
  | Filter:          |               | filter:          |               | WHERE status='Complete'|
  |   Status=Complete|               |   status         |               | GROUP BY 1             |
  +------------------+               +------------------+               +------------------------+
```

### 1.2 Git Workflow: Development Mode vs. Production Mode
Every LookML project is backed by a Git repository, allowing multiple developers to model safely without breaking live dashboards:

```
  [Production Branch] ---> Live Dashboards & Explores used by Business Teams
         |
         +---> Toggle "Development Mode ON" ---> [Personal Dev Branch]
                                                       |
                                              Edit .view & .model files
                                                       |
                                              Run "Validate LookML"
                                                       |
                                              Commit Changes & Push
                                                       |
         +<--- "Deploy to Production" <----------------+
```

---

## Module 2: LookML Fundamentals — Projects, Models & Views (Part 1)

### 2.1 Anatomy of a LookML View (`orders.view.lkml`)
A **View** corresponds to a database table (or derived table) and defines **Dimensions** (attributes) and **Measures** (aggregations).

```lkml
view: orders {
  sql_table_name: `ecommerce.orders` ;;

  # 1. PRIMARY KEY (Essential for Symmetric Aggregates across joins)
  dimension: order_id {
    primary_key: yes
    type: number
    sql: ${TABLE}.order_id ;;
  }

  dimension: user_id {
    hidden: yes
    type: number
    sql: ${TABLE}.user_id ;;
  }

  # 2. DIMENSION GROUP (Automatically generates multiple date/time granularities)
  dimension_group: created {
    type: time
    timeframes: [raw, time, date, week, month, quarter, year, day_of_week]
    sql: ${TABLE}.created_at ;;
  }

  # 3. GEOGRAPHIC DIMENSION (Enables native Looker Maps)
  dimension: country {
    type: string
    map_layer_name: countries
    sql: ${TABLE}.country ;;
  }

  dimension: status {
    type: string
    sql: ${TABLE}.status ;;
  }

  dimension: sales_channel {
    type: string
    sql: ${TABLE}.sales_channel ;;
  }

  dimension: product_category {
    type: string
    sql: ${TABLE}.product_category ;;
  }

  # 4. TIER DIMENSION (Buckets continuous numeric values into discrete cohorts)
  dimension: order_value_tier {
    type: tier
    tiers: [0, 100, 250, 500]
    style: integer
    sql: ${TABLE}.sale_price ;;
  }

  # 5. CASE DIMENSION (Custom business logic categorization)
  dimension: fulfillment_health {
    type: string
    case: {
      when: {
        sql: ${status} = 'Complete' ;;
        label: "Fulfilled"
      }
      when: {
        sql: ${status} IN ('Returned', 'Cancelled') ;;
        label: "Lost / Reversed"
      }
      else: "Pending"
    }
  }

  # 6. GOVERNED MEASURES (Aggregations with Drill-Down Fields)
  measure: order_count {
    type: count
    drill_fields: [order_id, created_date, users.full_name, product_category, total_revenue]
  }

  measure: total_revenue {
    type: sum
    sql: ${TABLE}.sale_price ;;
    value_format_name: usd
  }

  measure: average_order_value {
    type: average
    sql: ${TABLE}.sale_price ;;
    value_format_name: usd
  }

  # 7. FILTERED MEASURE (Embeds business rules directly into the metric)
  measure: completed_revenue {
    type: sum
    sql: ${TABLE}.sale_price ;;
    filters: [status: "Complete"]
    value_format_name: usd
  }

  measure: return_rate {
    type: number
    sql: SAFE_DIVIDE(COUNTIF(${status} = 'Returned'), ${order_count}) ;;
    value_format_name: percent_2
  }
}
```

### 2.2 Joins, Cardinality & Symmetric Aggregates (`retail_analytics.model.lkml`)
In a **Model** file, you define **Explores** and specify how views join together. Defining the exact `relationship` (`many_to_one`, `one_to_one`, `one_to_many`) activates Looker's **Symmetric Aggregates**, preventing double-counting when joining tables of different granularities:

```
  +-------------------+        many_to_one        +-------------------+
  |      orders       | ------------------------> |       users       |
  | (PK: order_id)    |  orders.user_id=users.id  | (PK: user_id)     |
  +-------------------+                           +-------------------+
            |
            |                  many_to_one        +-------------------+
            +-----------------------------------> | user_order_facts  |
                         orders.user_id=facts.id  | (Derived Table)   |
                                                  +-------------------+
```

```lkml
connection: "enterprise_warehouse"

include: "/views/*.view.lkml"

explore: orders {
  label: "Retail Orders & Customer Intelligence"
  description: "Unified Explore joining order transactions, customer profiles, and LTV cohorts."

  # Always filter out cancelled orders unless explicitly overridden
  sql_always_where: ${orders.status} != 'Cancelled' ;;

  join: users {
    type: left_outer
    relationship: many_to_one
    sql_on: ${orders.user_id} = ${users.user_id} ;;
  }

  join: user_order_facts {
    type: left_outer
    relationship: many_to_one
    sql_on: ${orders.user_id} = ${user_order_facts.user_id} ;;
  }
}
```

---

## Module 3: Advanced LookML — Derived Tables, Parameters & Liquid (Part 2)

### 3.1 Ephemeral vs. Persistent Derived Tables (PDTs)

```
  +-----------------------------------+-----------------------------------+
  |     EPHEMERAL DERIVED TABLE       |   PERSISTENT DERIVED TABLE (PDT)  |
  +-----------------------------------+-----------------------------------+
  | - Compiled as a SQL CTE (`WITH`)  | - Materialized as a physical table|
  | - Runs fresh on every Explore hit | - Rebuilt on a `datagroup` trigger|
  | - Ideal for dynamic Liquid inputs | - Ideal for heavy aggregations    |
  +-----------------------------------+-----------------------------------+
```

### 3.2 Dynamic SQL Derived Table with LookML Parameters & Liquid
**Parameters** create filter-only UI controls in Explores and Dashboards. **Liquid Variables** (`{% parameter ... %}`) inject the user's selected parameter value directly into the underlying SQL query at runtime.

```lkml
view: user_order_facts {
  # Interactive UI input for analysts
  parameter: min_completed_orders {
    type: number
    default_value: "2"
    description: "Minimum number of completed orders required to qualify as a repeat buyer."
  }

  derived_table: {
    sql:
      SELECT
        user_id,
        COUNT(DISTINCT order_id) AS lifetime_orders,
        SUM(sale_price) AS lifetime_revenue,
        MIN(created_at) AS first_order_at,
        MAX(created_at) AS latest_order_at
      FROM `ecommerce.orders`
      WHERE status = 'Complete'
      GROUP BY 1
      HAVING COUNT(DISTINCT order_id) >= {% parameter min_completed_orders %}
    ;;
  }

  dimension: user_id {
    primary_key: yes
    hidden: yes
    type: number
    sql: ${TABLE}.user_id ;;
  }

  dimension: lifetime_orders {
    type: number
    sql: ${TABLE}.lifetime_orders ;;
  }

  dimension: lifetime_revenue {
    type: number
    sql: ${TABLE}.lifetime_revenue ;;
    value_format_name: usd
  }

  dimension: repeat_buyer_status {
    type: string
    sql: CASE
           WHEN ${lifetime_orders} >= 4 THEN 'Super Loyal (4+ Orders)'
           WHEN ${lifetime_orders} >= 2 THEN 'Repeat Buyer (2-3 Orders)'
           ELSE 'Single Order'
         END ;;
  }

  measure: average_customer_ltv {
    type: average
    sql: ${TABLE}.lifetime_revenue ;;
    value_format_name: usd
  }
}
```

---

## Module 4: Exploring Data, Pivot Tables & Table Calculations

### 4.1 Looker Explore Workspace Layout

```
+---------------------------------------------------------------------------------------------------+
|  EXPLORE: Retail Orders & Customer Intelligence                                    [RUN]  [GEAR]  |
+---------------------------+-----------------------------------------------------------------------+
|  FIELD PICKER             |  FILTERS                                                              |
|  [Search Fields...]       |  [Orders Status: is equal to Complete]  [+ Filter]                    |
|                           +-----------------------------------------------------------------------+
|  > ORDERS                 |  VISUALIZATION  (Expand / Collapse)                                   |
|    Dimensions:            |  [Column] [Bar] [Pie] [Line] [Map] [Single Value] [Table]    [Edit]   |
|    - Created Month        |  +-----------------------------------------------------------------+  |
|    - Country              |  |      |||          |||          |||          |||                 |  |
|    - Product Category     |  |      |||   |||    |||   |||    |||   |||    |||   |||           |  |
|    - Sales Channel [Pivot]|  +-----------------------------------------------------------------+  |
|    Measures:              +-----------------------------------------------------------------------+
|    - Order Count          |  DATA TABLE (Pivoted by Sales Channel)                                |
|    - Completed Revenue    |  Created Month | Online Store | Mobile App | Marketplace | % Share    |
|    - Average Order Value  |  2026-08       | $2,089.49    | $1,149.49  | $679.00     | 100.0%     |
|                           |  2026-09       | $2,602.00    | $1,422.50  | $840.00     | 100.0%     |
|  > USERS                  |  2026-10       | $2,664.50    | $1,182.00  | $276.00     | 100.0%     |
|    - Customer Segment     |  2026-11       | $3,318.00    | $1,431.00  | $1,065.00   | 100.0%     |
+---------------------------+-----------------------------------------------------------------------+
```

### 4.2 Essential Table Calculation Formulas
Table Calculations allow business analysts to create Excel-like post-query formulas directly in an Explore without modifying LookML:

1. **Share of Row Total (Across Pivots):**
   ```looker
   ${orders.completed_revenue} / sum(pivot_row(${orders.completed_revenue}))
   ```
2. **Month-over-Month (MoM) Revenue Growth %:**
   ```looker
   (${orders.completed_revenue} - offset(${orders.completed_revenue}, 1)) / offset(${orders.completed_revenue}, 1)
   ```
3. **Running Cumulative Revenue:**
   ```looker
   running_total(${orders.completed_revenue})
   ```

---

## Module 5: Interactive Dashboards, Looker Studio & Governance

### 5.1 Dashboard Layout Wireframe (Native Looker & Looker Studio)

```
+---------------------------------------------------------------------------------------------------+
|  EXECUTIVE RETAIL REVENUE & CUSTOMER INTELLIGENCE         [Date Filter] [Region] [Segment Filter] |
+---------------------------------------------------------------------------------------------------+
|  [KPI TILE 1]               |  [KPI TILE 2]               |  [KPI TILE 3]               |  [KPI 4]|
|  Completed Revenue          |  Total Fulfilled Orders     |  Average Order Value        |  LTV    |
|  $21,310.97                 |  91 Orders                  |  $234.19                    |  $845.20|
+-----------------------------+-----------------------------+-----------------------------+---------+
|  [STACKED COLUMN CHART: Monthly Revenue by Channel]       |  [GEOGRAPHIC MAP: Revenue by Country] |
|  - X-Axis: Created Month | Stack: Sales Channel           |  - Region: Country | Color: Revenue   |
|  - Series Colors: Blue (Online), Cyan (App), Amber (Mkt)  |  - Interactive Cross-Filtering ON     |
+-----------------------------------------------------------+---------------------------------------+
|  [DONUT CHART: Product Category]   |  [HEATMAP DATA TABLE: Customer Segment & Loyalty Breakdown]  |
|  - Inner Radius: 55%               |  - Dimensions: Segment, Loyalty Tier, Repeat Buyer Status  |
|  - Labels: Category + % Share      |  - Conditional Formatting Scale on Completed Revenue         |
+------------------------------------+--------------------------------------------------------------+
```

### 5.2 Choosing the Right Visualization Type

| Analytical Goal | Best Looker Visualization | Key Configuration Setting |
| :--- | :--- | :--- |
| **Headline KPIs & Targets** | **Single Value** (`123`) | Enable *Comparison* to show delta vs. prior period |
| **Time-Series & Category Split** | **Column / Bar Chart** | Set *Series Positioning* to **Stacked** or **Grouped** |
| **Part-to-Whole Proportions** | **Pie / Donut Chart** | Set *Inner Radius* to `50%–60%` for clean Donut styling |
| **Regional Performance** | **Map (Interactive / Region)** | Requires `map_layer_name: countries` in LookML |
| **Multi-Metric Drill-Downs** | **Table (`Table-Next`)** | Enable *Show Totals* and *Conditional Formatting* scales |
| **Two-Metric Correlation** | **Scatter / Bubble Plot** | Plot `Order Count` (X) vs. `Average Order Value` (Y) |

### 5.3 Enterprise Governance, Alerts & Scheduling
* **Cross-Filtering:** Toggle **Cross-filtering ON** in Dashboard Settings so clicking any country on the map or slice in the donut chart filters all remaining tiles instantaneously.
* **Data Alerts:** Set automated threshold triggers on Single Value KPIs (e.g., notify `#revenue-ops` on Slack if `Return Rate` exceeds `8.0%`).
* **Scheduled Deliveries:** Automate recurring dashboard exports (`PDF`, `PNG`, or `CSV` data tables) delivered via Email, Slack, or S3/GCS webhooks.
* **Role-Based Access Control (RBAC):** Combine **Permission Sets** (what actions a user can perform, such as *Explore* vs. *Develop*) with **Model Sets** (which LookML models they can query) inside **Admin $\rightarrow$ Roles**.