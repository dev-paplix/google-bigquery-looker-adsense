# Looker Studio (Formerly Google Data Studio) Implementation Lab (Using Only Google Sheets)

A complete, step-by-step hands-on guide for **Looker Studio** (`lookerstudio.google.com`) using **Google Sheets** as your sole data source—no BigQuery, SQL database, GCP billing, or LookML required.

---

## Dashboard Wireframe & Visual Architecture

```
+---------------------------------------------------------------------------------------------------+
|  [Header Banner] RETAIL REVENUE & CUSTOMER INTELLIGENCE DASHBOARD   [Date Control] [Region Drop.] |
+---------------------------------------------------------------------------------------------------+
|  [Scorecard 1]            |  [Scorecard 2]            |  [Scorecard 3]            |  [Scorecard 4]|
|  Net Revenue (Completed)  |  Total Gross Profit       |  Gross Margin %           |  Total Orders |
+---------------------------+---------------------------+---------------------------+---------------+
|  [GEO CHART: Revenue by Country]                      |  [STACKED COLUMN CHART: Monthly Revenue]  |
|  - Geo Dimension: country_name                        |  - Dimension: order_date (Year-Month)     |
|  - Color Metric: Net Revenue                          |  - Breakdown: device_category             |
|  - Cross-Filtering: Enabled                           |  - Stacked Bars + Data Labels             |
+-------------------------------------------------------+-------------------------------------------+
|  [DONUT CHART: Share by Category]  |  [TABLE WITH HEATMAP: Segment & Sub-Category Performance]    |
|  - Dimension: category             |  - Dimensions: customer_segment, category, sub_category      |
|  - Inner Hole Radius: 55%          |  - Metrics: Net Revenue, Gross Profit, Units Sold            |
+------------------------------------+--------------------------------------------------------------+
```

---

## Phase 1: Import CSV Datasets into Google Sheets

1. **Create a New Google Sheet:**
   * Open [Google Sheets](https://sheets.new).
   * Click `Untitled spreadsheet` in the top-left corner and rename it to `Looker_Studio_Retail_Data`.
2. **Import `retail_orders_dummy-v3.csv` into Tab 1:**
   * Double-click `Sheet1` at the bottom-left and rename it to `Orders`.
   * In the top menu, click **File** $\rightarrow$ **Import** $\rightarrow$ **Upload**.
   * Drag or browse to select **`retail_orders_dummy-v3.csv`**.
   * In the Import file modal:
     * **Import location:** `Replace current sheet`
     * **Separator type:** `Detect automatically`
     * **Convert text to numbers, dates, and formulas:** `Yes`
   * Click **Import data**.
3. **Import `retail_users_dummy-v3.csv` into Tab 2:**
   * Click the **`+` (Add sheet)** button in the bottom-left corner.
   * Double-click `Sheet2` and rename it to `Users`.
   * Click **File** $\rightarrow$ **Import** $\rightarrow$ **Upload** $\rightarrow$ select **`retail_users_dummy-v3.csv`**.
   * Choose **Replace current sheet** and click **Import data**.

---

## Phase 2: Connect Both Google Sheet Tabs to Looker Studio

1. **Create a New Report:**
   * Go to [Looker Studio](https://lookerstudio.google.com/).
   * Click **Create** (top-left) $\rightarrow$ **Report**.
2. **Connect the `Orders` Worksheet:**
   * In the **Add data to report** panel, select **Google Sheets** under *Google Connectors*.
   * Click **Authorize** if prompted.
   * Under **Spreadsheet**, select **`Looker_Studio_Retail_Data`**.
   * Under **Worksheet**, select **`Orders`**.
   * Verify **Use first row as headers** and **Include hidden and filtered cells** are checked.
   * Click **Add** (bottom-right) $\rightarrow$ confirm **Add to Report**.
3. **Connect the `Users` Worksheet:**
   * In the top toolbar, click **Add data**.
   * Select **Google Sheets** $\rightarrow$ **`Looker_Studio_Retail_Data`** $\rightarrow$ Worksheet: **`Users`**.
   * Click **Add** $\rightarrow$ **Add to Report**.
4. **Verify Field Data Types (Critical for Maps & Currency):**
   * In the top menu, click **Resource** $\rightarrow$ **Manage added data sources**.
   * Click **Edit** next to `Orders`:
     * Find `country_name` $\rightarrow$ click the **Type** dropdown (`Text`) $\rightarrow$ change to **Geo** $\rightarrow$ **Country**.
     * Find `region` $\rightarrow$ ensure type is **Text** (or **Geo** $\rightarrow$ **Sub-continent**).
     * Find `net_revenue_usd`, `gross_revenue_usd`, `cogs_usd` $\rightarrow$ change Type to **Numeric** $\rightarrow$ **Currency** $\rightarrow$ **USD ($)**.
     * Find `order_date` $\rightarrow$ ensure Type is **Date (YYYYMMDD)**.
   * Click **Done** (top-right) $\rightarrow$ click **Close**.

---

## Phase 3: Blend the `Orders` and `Users` Sheets Visually

1. **Open the Blend Editor:**
   * In the top toolbar, click **Blend** (or go to **Resource** $\rightarrow$ **Manage blends** $\rightarrow$ **Add a blend**).
2. **Configure Table 1 (`Orders`):**
   * Click the pencil icon next to the table name and rename it `Orders_Table`.
   * **Join keys:** Drag `user_id` into the **Join keys** box.
   * **Dimensions:** Drag these fields into the **Dimensions** box:
     * `order_date`
     * `country_name`
     * `region`
     * `device_category`
     * `category`
     * `sub_category`
     * `order_status`
   * **Metrics:** Drag these fields into the **Metrics** box:
     * `net_revenue_usd` (Aggregation: `Sum`)
     * `cogs_usd` (Aggregation: `Sum`)
     * `quantity` (Aggregation: `Sum`)
     * `order_id` $\rightarrow$ click the pencil icon next to `order_id` $\rightarrow$ change aggregation to **Count Distinct** and rename it `Total Orders`.
3. **Configure Table 2 (`Users`):**
   * Click **Join another table** and select **`Users`**.
   * Rename it `Users_Table`.
   * **Join keys:** Drag `user_id` into the **Join keys** box.
   * **Dimensions:** Drag these fields into the **Dimensions** box:
     * `customer_segment`
     * `acquisition_channel`
     * `age_group`
4. **Configure Join Operator:**
   * Click the **Configure join** badge between the two tables.
   * Select **Left Outer** (keeps all orders and enriches them with user segment attributes).
   * Confirm the join condition is `user_id` = `user_id`.
   * Click **Save**.
5. **Save the Blended Data Source:**
   * At the top-left of the blend screen, set **Data blend name** to `Blended_Retail_Sheets`.
   * Click **Save** (bottom-right), then close the blend overlay.

---

## Phase 4: Build an Executive-Grade Looker Studio Dashboard

### Step 4.1: Canvas Setup, Theme & Header Controls
1. **Canvas Size & Theme:**
   * Click **Page** $\rightarrow$ **Current page settings** $\rightarrow$ **Style** $\rightarrow$ set **Width** to `1440 px` and **Height** to `1050 px`.
   * Click **Theme and layout** in the toolbar and choose **Edge** (clean light theme) or **Constellation** (dark theme).
2. **Add Title Banner:**
   * Click the **Text** (`A`) icon in the toolbar $\rightarrow$ draw a box at the top-left $\rightarrow$ type `Retail Revenue & Customer Intelligence` (Font: **Roboto**, Size: `22px`, Bold).
3. **Add Date Range Control:**
   * Click **Add a control** $\rightarrow$ **Date range control** $\rightarrow$ place at the top-right (`X: 1180, Y: 15`).
   * In the right-hand **Setup** panel, set **Default date range** to **Auto date range** (or **Fixed** across your dataset dates).
4. **Add Region Dropdown Control:**
   * Click **Add a control** $\rightarrow$ **Drop-down list** $\rightarrow$ place next to the date picker (`X: 960, Y: 15`).
   * Set **Data source** to `Blended_Retail_Sheets` and **Control field** to `region`.

---

### Step 4.2: Top KPI Scorecards (4 Cards Across Row 1)
1. **Scorecard 1 — Completed Net Revenue:**
   * Click **Add a chart** $\rightarrow$ **Scorecard** $\rightarrow$ place at `X: 30, Y: 80` (`Width: 320, Height: 95`).
   * **Data source:** `Blended_Retail_Sheets`.
   * **Metric:** Select `net_revenue_usd` $\rightarrow$ rename label to `Completed Net Revenue`.
   * **Filter (Optional):** Click **Add a filter** at the bottom of the Setup panel $\rightarrow$ **Create a filter** $\rightarrow$ `Include` `order_status` `Equal to (=)` `Completed` $\rightarrow$ click **Save**.
   * **Style Tab:** Center-align text, set metric font size to `28px`, add a subtle border and `6px` corner radius.
2. **Scorecard 2 — Total Gross Profit:**
   * Duplicate Scorecard 1 (`Ctrl+D` / `Cmd+D`) and place at `X: 380, Y: 80`.
   * Under **Metric**, click the metric chip $\rightarrow$ **Create field**:
     * **Name:** `Gross Profit`
     * **Formula:** `SUM(net_revenue_usd) - SUM(cogs_usd)`
     * **Data type:** `Currency (USD)` $\rightarrow$ click **Apply**.
3. **Scorecard 3 — Gross Margin %:**
   * Duplicate Scorecard 2 and place at `X: 730, Y: 80`.
   * Under **Metric**, click **Create field**:
     * **Name:** `Gross Margin %`
     * **Formula:** `(SUM(net_revenue_usd) - SUM(cogs_usd)) / SUM(net_revenue_usd)`
     * **Data type:** `Percent` $\rightarrow$ click **Apply**.
4. **Scorecard 4 — Total Orders:**
   * Duplicate Scorecard 3 and place at `X: 1080, Y: 80`.
   * Set **Metric** to `Total Orders` (`COUNT_DISTINCT(order_id)`).

---

### Step 4.3: Geographic Map & Stacked Column Chart (Row 2)
1. **Geographic Map (Left Half: `X: 30, Y: 200, Width: 670, Height: 370`):**
   * Click **Add a chart** $\rightarrow$ **Geo chart**.
   * **Data source:** `Blended_Retail_Sheets`.
   * **Geo dimension:** `country_name`.
   * **Color metric:** `net_revenue_usd`.
   * **Style Tab:** Set Max Color to Deep Emerald (`#0F5132`), Mid Color to Mint (`#20C997`), Min Color to Light Gray (`#E2E8F0`).
   * **Setup Tab (Bottom):** Check **Cross-filtering** so clicking a country filters the entire report.
2. **Stacked Column Chart (Right Half: `X: 730, Y: 200, Width: 670, Height: 370`):**
   * Click **Add a chart** $\rightarrow$ **Stacked column chart**.
   * **Data source:** `Blended_Retail_Sheets`.
   * **Dimension:** `order_date` $\rightarrow$ click the pencil icon next to `order_date` $\rightarrow$ change **Date & Time** granularity to **Year Month**.
   * **Breakdown dimension:** `device_category`.
   * **Metric:** `net_revenue_usd`.
   * **Sort:** By `order_date` **Ascending**.
   * **Style Tab:** Toggle **Show data labels** ON and customize series colors (`Desktop`: Royal Blue `#2563EB`, `Mobile`: Cyan `#06B6D4`, `Tablet`: Amber `#F59E0B`).

---

### Step 4.4: Donut Chart & Heatmap Data Table (Row 3)
1. **Product Category Donut Chart (Left: `X: 30, Y: 600, Width: 430, Height: 400`):**
   * Click **Add a chart** $\rightarrow$ **Donut chart**.
   * **Data source:** `Blended_Retail_Sheets`.
   * **Dimension:** `category`.
   * **Metric:** `net_revenue_usd`.
   * **Style Tab:** Set donut hole slider to `55%`, set slice labels to **Percentage**, and check **Cross-filtering** in the Setup tab.
2. **Segment & Sub-Category Heatmap Table (Right: `X: 480, Y: 600, Width: 920, Height: 400`):**
   * Click **Add a chart** $\rightarrow$ **Table with heatmap**.
   * **Data source:** `Blended_Retail_Sheets`.
   * **Dimensions:** `customer_segment`, `category`, `sub_category`.
   * **Metrics:**
     * `net_revenue_usd`
     * Calculated Metric: `Gross Profit` (`SUM(net_revenue_usd) - SUM(cogs_usd)`)
     * `quantity`
   * **Setup Tab:** Check **Show summary row** at the bottom.
   * **Style Tab:** Set heatmap color gradient on `net_revenue_usd` to Green (`#E8F5E9` $\rightarrow$ `#2E7D32`) and `Gross Profit` to Blue (`#EFF6FF` $\rightarrow$ `#1D4ED8`).

---

## Phase 5: Sharing & Automated Email Delivery

1. **View Mode Testing:** Click **View** (top-right) to test clicking slices on your Donut Chart, countries on your Geo Map, and filtering by Region.
2. **Schedule Automated Reports:** Click the dropdown arrow next to **Share** (top-right) $\rightarrow$ **Schedule delivery** to email PDF snapshots of your Google Sheets dashboard to stakeholders automatically.