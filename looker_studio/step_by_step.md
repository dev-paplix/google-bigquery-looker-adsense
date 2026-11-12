# Step-by-Step Implementation Lab: Google Looker, LookML & Looker Studio

A hands-on, click-by-click technical lab manual designed to accompany the **1-Day Google Looker Workshop**. This workbook walks you through loading your retail CSV datasets (`retail_orders_dummy-v3.csv` and `retail_users_dummy-v3.csv`), building a governed semantic layer in **LookML**, exploring data with **Pivot Tables & Table Calculations**, designing an interactive **Looker Dashboard**, and connecting your governed Explore directly to **Looker Studio (formerly Google Data Studio)**.

---

## Lab Architecture & Data Schema Overview

Before starting, review the relational schema of the two CSV datasets you will be modeling and visualizing:

```
+---------------------------------------+          +---------------------------------------+
|       `ecommerce.retail_orders`       |          |       `ecommerce.retail_users`        |
|           (3,500 Rows)                |          |            (500 Rows)                 |
+---------------------------------------+          +---------------------------------------+
| PK: order_id                          |          | PK: user_id                           |
| FK: user_id  -------------------------+--------> |     signup_date                       |
|     order_timestamp / order_date      |   N : 1  |     country_code / country_name       |
|     country_code / country_name       |          |     region / state_province           |
|     region / state_province           |          |     customer_segment                  |
|     device_category                   |          |     acquisition_channel               |
|     category / sub_category           |          |     age_group                         |
|     product_name / quantity           |          |     newsletter_subscribed             |
|     unit_price_usd / discount_pct     |          +---------------------------------------+
|     gross_revenue_usd                 |
|     net_revenue_usd / cogs_usd        |
|     shipping_fee_usd                  |
|     order_status / payment_method     |
+---------------------------------------+
```

---

## Phase 1: Ingesting CSV Data & Connecting to Google Looker

### Step 1.1: Load CSV Files into Your Cloud Data Warehouse
*(Note: Google Looker requires a SQL database connection for LookML modeling. If you are using Looker Studio exclusively without a SQL warehouse, skip to **Phase 6** to upload the CSVs directly into Looker Studio).*

1. Open your SQL Data Warehouse console (e.g., Google BigQuery, Snowflake, or PostgreSQL).
2. Create a schema/dataset named `ecommerce`.
3. Upload **`retail_orders_dummy-v3.csv`** as a table named `retail_orders`:
   - Ensure `order_id`, `user_id`, `quantity` are typed as `INTEGER`.
   - Ensure `unit_price_usd`, `discount_pct`, `gross_revenue_usd`, `net_revenue_usd`, `cogs_usd`, `shipping_fee_usd` are typed as `FLOAT` / `NUMERIC`.
   - Ensure `order_date` is typed as `DATE` and `order_timestamp` as `TIMESTAMP`.
4. Upload **`retail_users_dummy-v3.csv`** as a table named `retail_users`:
   - Ensure `user_id` is typed as `INTEGER`, `signup_date` as `DATE`, and `newsletter_subscribed` as `BOOLEAN`.

### Step 1.2: Configure the Database Connection in Google Looker
1. Log into your **Google Looker** instance.
2. Click **Admin** (left sidebar) $\rightarrow$ **Database** $\rightarrow$ **Connections** $\rightarrow$ **Add Connection**.
3. Enter the connection details:
   - **Name:** `retail_warehouse_conn`
   - **Dialect:** Select your SQL warehouse dialect (e.g., *Google BigQuery Standard SQL*).
   - **Database / Dataset:** `ecommerce`
   - Upload your service account key or enter database credentials.
4. Click **Test These Settings** to verify green checkmarks, then click **Connect**.

---

## Phase 2: Initializing Your LookML Project & Views (LookML Part 1)

### Step 2.1: Create a New LookML Project
1. Look at the bottom-left corner of Looker and toggle **Development Mode** to **ON** (purple banner appears at the top).
2. In the top-left menu, go to **Develop** $\rightarrow$ **Projects**.
3. Click **New LookML Project** (top-right).
4. Configure the project:
   - **Project Name:** `retail_enterprise_bi`
   - **Starting Point:** **Generate Model from Database Schema**
   - **Connection:** `retail_warehouse_conn`
   - **Build Views From:** **All Tables** (or select `retail_orders` and `retail_users`).
5. Click **Create Project**. Looker automatically generates a `.model.lkml` file and `.view.lkml` files inside the Looker IDE.

### Step 2.2: Configure the Orders View (`retail_orders.view.lkml`)
1. In the Looker IDE left file browser, open `views/retail_orders.view.lkml`.
2. Replace the boilerplate code with the governed dimensions, geographic map layers, tiers, and profit/revenue measures below:

```lkml
view: retail_orders {
  sql_table_name: `ecommerce.retail_orders` ;;

  # --- PRIMARY & FOREIGN KEYS ---
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

  # --- TIME DIMENSION GROUPS ---
  dimension_group: order {
    type: time
    timeframes: [raw, time, date, week, month, quarter, year, day_of_week]
    sql: ${TABLE}.order_timestamp ;;
  }

  # --- GEOGRAPHIC DIMENSIONS (Enables Interactive Maps) ---
  dimension: country_code {
    type: string
    map_layer_name: countries
    sql: ${TABLE}.country_code ;;
  }

  dimension: country_name {
    type: string
    map_layer_name: countries
    sql: ${TABLE}.country_name ;;
  }

  dimension: region {
    type: string
    sql: ${TABLE}.region ;;
  }

  dimension: state_province {
    type: string
    sql: ${TABLE}.state_province ;;
  }

  # --- PRODUCT & CHANNEL DIMENSIONS ---
  dimension: device_category {
    type: string
    sql: ${TABLE}.device_category ;;
  }

  dimension: category {
    type: string
    sql: ${TABLE}.category ;;
  }

  dimension: sub_category {
    type: string
    sql: ${TABLE}.sub_category ;;
  }

  dimension: product_name {
    type: string
    sql: ${TABLE}.product_name ;;
  }

  dimension: order_status {
    type: string
    sql: ${TABLE}.order_status ;;
  }

  dimension: payment_method {
    type: string
    sql: ${TABLE}.payment_method ;;
  }

  # --- CUSTOM COHORT TIERS ---
  dimension: order_size_tier {
    type: tier
    tiers: [0, 50, 150, 300, 500]
    style: integer
    sql: ${TABLE}.net_revenue_usd ;;
  }

  # --- GOVERNED MEASURES ---
  measure: total_orders {
    type: count
    drill_fields: [order_id, order_date, retail_users.customer_segment, category, product_name, total_net_revenue]
  }

  measure: total_units_sold {
    type: sum
    sql: ${TABLE}.quantity ;;
    value_format_name: decimal_0
  }

  measure: total_gross_revenue {
    type: sum
    sql: ${TABLE}.gross_revenue_usd ;;
    value_format_name: usd
  }

  measure: total_net_revenue {
    type: sum
    sql: ${TABLE}.net_revenue_usd ;;
    value_format_name: usd
  }

  measure: total_cogs {
    type: sum
    sql: ${TABLE}.cogs_usd ;;
    value_format_name: usd
  }

  measure: total_gross_profit {
    type: number
    sql: ${total_net_revenue} - ${total_cogs} ;;
    value_format_name: usd
  }

  measure: gross_margin_pct {
    type: number
    sql: SAFE_DIVIDE(${total_gross_profit}, ${total_net_revenue}) ;;
    value_format_name: percent_2
  }

  measure: average_order_value {
    type: average
    sql: ${TABLE}.net_revenue_usd ;;
    value_format_name: usd
  }

  measure: completed_net_revenue {
    type: sum
    sql: ${TABLE}.net_revenue_usd ;;
    filters: [order_status: "Completed"]
    value_format_name: usd
  }

  measure: return_rate_pct {
    type: number
    sql: SAFE_DIVIDE(COUNTIF(${order_status} = 'Returned'), ${total_orders}) ;;
    value_format_name: percent_2
  }
}
```
3. Click **Save Changes** (top-right).

### Step 2.3: Configure the Users View (`retail_users.view.lkml`)
1. Open `views/retail_users.view.lkml` in the IDE.
2. Replace its contents with:

```lkml
view: retail_users {
  sql_table_name: `ecommerce.retail_users` ;;

  dimension: user_id {
    primary_key: yes
    type: number
    sql: ${TABLE}.user_id ;;
  }

  dimension_group: signup {
    type: time
    timeframes: [raw, date, week, month, quarter, year]
    convert_tz: no
    datatype: date
    sql: ${TABLE}.signup_date ;;
  }

  dimension: customer_segment {
    type: string
    sql: ${TABLE}.customer_segment ;;
  }

  dimension: acquisition_channel {
    type: string
    sql: ${TABLE}.acquisition_channel ;;
  }

  dimension: age_group {
    type: string
    sql: ${TABLE}.age_group ;;
  }

  dimension: newsletter_subscribed {
    type: yesno
    sql: ${TABLE}.newsletter_subscribed ;;
  }

  measure: total_customers {
    type: count
    drill_fields: [user_id, customer_segment, acquisition_channel, country_name, signup_date]
  }
}
```
3. Click **Save Changes**.

---

## Phase 3: Derived Tables, Liquid Parameters & Model Joins (LookML Part 2)

### Step 3.1: Create a Parameterized Derived Table (`customer_ltv_facts.view.lkml`)
1. In the Looker IDE file browser, click the **`+`** icon at the top $\rightarrow$ **Create View**.
2. Name the file `customer_ltv_facts.view.lkml` and click **Create**.
3. Paste the following SQL Derived Table using a **LookML Parameter** and **Liquid Templating** (`{% parameter ... %}`):

```lkml
view: customer_ltv_facts {
  parameter: min_order_threshold {
    type: number
    default_value: "3"
    description: "Dynamic filter parameter: Minimum number of completed orders per customer."
  }

  derived_table: {
    sql:
      SELECT
        user_id,
        COUNT(DISTINCT order_id) AS lifetime_completed_orders,
        SUM(net_revenue_usd) AS lifetime_net_revenue,
        SUM(net_revenue_usd - cogs_usd) AS lifetime_gross_profit,
        MIN(order_date) AS first_purchase_date,
        MAX(order_date) AS latest_purchase_date
      FROM `ecommerce.retail_orders`
      WHERE order_status = 'Completed'
      GROUP BY 1
      HAVING COUNT(DISTINCT order_id) >= {% parameter min_order_threshold %}
    ;;
  }

  dimension: user_id {
    primary_key: yes
    hidden: yes
    type: number
    sql: ${TABLE}.user_id ;;
  }

  dimension: lifetime_completed_orders {
    type: number
    sql: ${TABLE}.lifetime_completed_orders ;;
  }

  dimension: lifetime_net_revenue {
    type: number
    sql: ${TABLE}.lifetime_net_revenue ;;
    value_format_name: usd
  }

  dimension: buyer_frequency_tier {
    type: string
    sql: CASE
           WHEN ${lifetime_completed_orders} >= 10 THEN 'Champions (10+ Orders)'
           WHEN ${lifetime_completed_orders} >= 5 THEN 'Loyalists (5-9 Orders)'
           ELSE 'Repeat Buyers (3-4 Orders)'
         END ;;
  }

  measure: average_customer_ltv {
    type: average
    sql: ${TABLE}.lifetime_net_revenue ;;
    value_format_name: usd
  }
}
```
4. Click **Save Changes**.

### Step 3.2: Join All Views in the Model File (`retail_enterprise_bi.model.lkml`)
1. Open `models/retail_enterprise_bi.model.lkml`.
2. Define the Explore and its `many_to_one` joins:

```lkml
connection: "retail_warehouse_conn"

include: "/views/*.view.lkml"

explore: retail_orders {
  label: "Enterprise Retail Orders & Customer LTV"
  description: "3,500 orders joined to 500 user profiles and dynamic LTV cohorts."

  join: retail_users {
    type: left_outer
    relationship: many_to_one
    sql_on: ${retail_orders.user_id} = ${retail_users.user_id} ;;
  }

  join: customer_ltv_facts {
    type: left_outer
    relationship: many_to_one
    sql_on: ${retail_orders.user_id} = ${customer_ltv_facts.user_id} ;;
  }
}
```
3. Click **Save Changes**.
4. Click **Validate LookML** (top-right) $\rightarrow$ ensure zero errors $\rightarrow$ click **Commit Changes & Push** $\rightarrow$ click **Deploy to Production**.

---

## Phase 4: Exploring Data, Pivots & Table Calculations

### Step 4.1: Build a Pivoted Revenue Exploration
1. In the top navigation bar, click **Explore** $\rightarrow$ **Enterprise Retail Orders & Customer LTV**.
2. **Apply a Filter:**
   - Expand **Retail Orders** $\rightarrow$ hover over **Order Status** $\rightarrow$ click the **Filter** icon.
   - Set **Order Status** `is equal to` `Completed`.
3. **Select Dimensions & Measures:**
   - Click **Order Month** (under `Order Date` dimension group).
   - Hover over **Device Category** $\rightarrow$ click the **Pivot** icon.
   - Under Measures, click **Completed Net Revenue** and **Gross Margin %**.
4. Click **Run** (top-right).

### Step 4.2: Add an Ad-Hoc Table Calculation
1. In the Explore field picker, click **Custom Fields** $\rightarrow$ **Add** $\rightarrow$ **Table Calculation**.
2. **Name:** `Channel Share %`
3. **Formula:**
   ```looker
   ${retail_orders.completed_net_revenue} / sum(pivot_row(${retail_orders.completed_net_revenue}))
   ```
4. **Format:** `Percent (1)` $\rightarrow$ click **Save**.
5. Expand the **Visualization** bar $\rightarrow$ select **Column** chart $\rightarrow$ click **Edit** $\rightarrow$ set **Series Positioning** to **Stacked**.
6. Click the **Gear icon** (top-right) $\rightarrow$ **Save** $\rightarrow$ **As a Look** $\rightarrow$ name it `Monthly Net Revenue by Device`.

---

## Phase 5: Building an Interactive Google Looker Dashboard

### Step 5.1: Create the Dashboard & Add Top KPI Tiles
1. Go to **Folders** $\rightarrow$ **My Folder** $\rightarrow$ click **New** $\rightarrow$ **Dashboard**.
2. Name it `Executive Retail Performance & LTV Dashboard` and click **Create Dashboard**.
3. Click **Edit Dashboard** (top-right).
4. **Add 4 Single Value KPI Tiles across Row 1:**
   - Click **Add** $\rightarrow$ **Visualization** $\rightarrow$ select **Enterprise Retail Orders & Customer LTV**.
   - Select **Completed Net Revenue** $\rightarrow$ choose **Single Value** visualization $\rightarrow$ title it `Completed Net Revenue` $\rightarrow$ click **Save**.
   - Repeat to create 3 more Single Value tiles:
     - **Tile 2:** `Total Gross Profit`
     - **Tile 3:** `Gross Margin %`
     - **Tile 4:** `Average Customer LTV` (from `Customer Ltv Facts`).
   - Drag and resize all 4 tiles side-by-side across the top row.

### Step 5.2: Add Geo Map, Donut Chart & Heatmap Table
1. **Geographic Map (Row 2, Left):**
   - Click **Add** $\rightarrow$ **Visualization** $\rightarrow$ select **Country Name** and **Completed Net Revenue**.
   - Choose **Map** visualization $\rightarrow$ under **Points/Regions**, set color gradient from Light Mint to Deep Emerald Green $\rightarrow$ click **Save**.
2. **Category Donut Chart (Row 2, Right):**
   - Click **Add** $\rightarrow$ **Visualization** $\rightarrow$ select **Category** and **Completed Net Revenue**.
   - Choose **Pie** visualization $\rightarrow$ click **Edit** $\rightarrow$ **Plot** $\rightarrow$ set **Inner Radius** to `55%` $\rightarrow$ click **Save**.
3. **Segment & Sub-Category Heatmap Table (Row 3, Full Width):**
   - Click **Add** $\rightarrow$ **Visualization** $\rightarrow$ select **Customer Segment**, **Category**, **Sub Category**, **Total Orders**, **Completed Net Revenue**, and **Return Rate %**.
   - Choose **Table** $\rightarrow$ click **Edit** $\rightarrow$ **Formatting** $\rightarrow$ **Add Rule** $\rightarrow$ apply a green color scale to `Completed Net Revenue` and a red color scale to `Return Rate %` $\rightarrow$ click **Save**.

### Step 5.3: Configure Dashboard Filters & Cross-Filtering
1. In **Edit Dashboard** mode, click **Filters** (top toolbar) $\rightarrow$ **Add Filter**:
   - **Title:** `Order Date` | **Field:** `Order Date` | **Control:** `Date Range` | Map to all tiles.
2. Add a second filter:
   - **Title:** `Region` | **Field:** `Region` | **Control:** `Dropdown` | Map to all tiles.
3. Click **Settings** (top toolbar) $\rightarrow$ toggle **Cross-filtering** to **ON**.
4. Click **Save** (top-right) to exit Edit Mode. Clicking any country on the map or slice in the donut chart now dynamically filters the entire dashboard.

---

## Phase 6: Connecting to Looker Studio (Formerly Google Data Studio)

You can connect your governed Looker Explore directly to **Looker Studio** (Option A) or upload the raw CSVs directly into Looker Studio for standalone reporting (Option B).

### Option A: Connect Governed LookML Explore to Looker Studio
1. Open [Looker Studio](https://lookerstudio.google.com/) and click **Create** $\rightarrow$ **Report**.
2. In the **Add data to report** panel, search for and select the **Looker** connector.
3. Enter your **Looker Instance URL** (e.g., `https://yourcompany.cloud.looker.com`) and authorize your account.
4. Select **Model:** `retail_enterprise_bi` $\rightarrow$ **Explore:** `Enterprise Retail Orders & Customer LTV` $\rightarrow$ click **Add**.
5. All your LookML dimensions, measures, and joins appear immediately in the right-hand Looker Studio Data panel without needing manual SQL or report-level blends.

### Option B: Direct CSV Upload & Blending in Looker Studio
1. In a new Looker Studio report, click **Add data** $\rightarrow$ select **File Upload**.
2. Upload **`retail_orders_dummy-v3.csv`** and click **Add**.
3. Go to **Resource** $\rightarrow$ **Manage added data sources** $\rightarrow$ **Add a Data Source** $\rightarrow$ **File Upload** $\rightarrow$ upload **`retail_users_dummy-v3.csv`**.
4. Go to **Resource** $\rightarrow$ **Manage blends** $\rightarrow$ **Add a blend**:
   - **Table 1 (`retail_orders`):** Join Key = `user_id` | Dimensions = `order_date`, `country_name`, `region`, `device_category`, `category`, `sub_category`, `order_status` | Metrics = `net_revenue_usd`, `cogs_usd`, `quantity`.
   - **Table 2 (`retail_users`):** Join Key = `user_id` | Dimensions = `customer_segment`, `acquisition_channel`, `age_group`.
   - **Join Operator:** `Left Outer` on `user_id = user_id` $\rightarrow$ name blend `Retail_Orders_Users_Blend` $\rightarrow$ click **Save**.
5. Build your canvas visuals using `Retail_Orders_Users_Blend`:
   - **Scorecards:** `SUM(net_revenue_usd)`, `SUM(net_revenue_usd) - SUM(cogs_usd)` (Gross Profit), and `COUNT_DISTINCT(order_id)`.
   - **Geo Chart:** Geo Dimension = `country_name`, Color Metric = `SUM(net_revenue_usd)`.
   - **Stacked Column Chart:** Dimension = `order_date` (grouped by Year-Month), Breakdown Dimension = `device_category`, Metric = `SUM(net_revenue_usd)`.
   - **Donut Chart:** Dimension = `category`, Metric = `SUM(net_revenue_usd)`, Hole Size = `55%`.
   - **Table with Heatmap:** Dimensions = `customer_segment`, `sub_category`; Metrics = `SUM(net_revenue_usd)` and `SUM(quantity)`.