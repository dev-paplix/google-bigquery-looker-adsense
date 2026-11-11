# Looker Studio Visual Design & Dashboard Implementation Manual

A hands-on implementation guide for building an executive-grade, interactive dashboard in Google Looker Studio using blended BigQuery and Google AdSense monetization data.

---

## 1. Canvas & Theme Specifications

### 1.1 Canvas Setup
1. Open the Looker Studio report editor.
2. Select **Page** $\rightarrow$ **Current page settings** from the top menu.
3. In the right-hand panel, select **Style**.
4. Set **Canvas Size**:
   - **Width:** `1440 px`
   - **Height:** `1000 px`

### 1.2 Theme Selection
1. Click **Theme and layout** in the toolbar.
2. Choose a clean, high-contrast base theme:
   - **Constellation** (Dark UI) or **Edge** (Light Slate UI).
3. Under **Layout**:
   - Set **Header type** to `Slide in/out` or `Initially hidden` to optimize visual canvas height.

---

## 2. Layout Grid Blueprint

```
+----------------------------------------------------------------------------------------------------+
|  [Header Bar] CPG AD MONETIZATION & PUBLISHER YIELD HUB             [Category Filter] [Date Range] |
+----------------------------------------------------------------------------------------------------+
|  [Scorecard 1]         |  [Scorecard 2]         |  [Scorecard 3]         |  [Scorecard 4]          |
|  Total Gross Revenue   |  Total Impressions     |  Blended eCPM          |  Fill Rate %            |
+------------------------+------------------------+------------------------+-------------------------+
|  [Geo Bubble / Heatmap Chart]                   |  [Stacked Column Bar Chart]                      |
|  - Dimension: country_name                      |  - Dimension: category                           |
|  - Color: Total Gross Revenue                   |  - Metrics: Direct BQ Yield vs AdSense           |
|  - Cross-filtering: Enabled                     |  - Data Labels: Enabled                          |
+-------------------------------------------------+--------------------------------------------------+
|  [Donut Chart]                                  |  [Table with Heatmap Formatting]                 |
|  - Dimension: device_category                   |  - Dimensions: category, sub_category, ad_unit   |
|  - Metric: Total Gross Revenue                  |  - Heatmap Column: Blended Page RPM              |
|  - Label: Percentage Share                      |  - Summary Row: Enabled                          |
+-------------------------------------------------+--------------------------------------------------+
```

---

## 3. Header & Filter Controls

### 3.1 Banner Background
- Select the **Rectangle** tool from the toolbar.
- Draw a box at `X: 0`, `Y: 0`, `Width: 1440`, `Height: 70`.
- Style properties: Fill Color `#1E293B` (Dark Slate), Border: None.

### 3.2 Title Label
- Select the **Text** tool and draw inside the header bar.
- Value: `CPG Ad Monetization & Publisher Yield Hub`
- Style properties: Font `Roboto`, Size `22px`, Color `#FFFFFF`, Weight `Bold`.

### 3.3 Dynamic Category Dropdown
- Click **Add a control** $\rightarrow$ **Drop-down list**.
- Placement: `X: 950`, `Y: 15`, `Width: 200`, `Height: 40`.
- **Setup Tab:**
  - Data Source: `Blend_BQ_AdSense`
  - Control Field: `category`
- **Style Tab:**
  - Search option: Enabled.
  - Background: `#FFFFFF`, Corner Radius: `4px`.

### 3.4 Date Range Selector
- Click **Add a control** $\rightarrow$ **Date range control**.
- Placement: `X: 1180`, `Y: 15`, `Width: 230`, `Height: 40`.
- **Setup Tab:**
  - Default date range: **Custom** $\rightarrow$ **Fixed** (`2026-11-01` to `2026-11-03`).

---

## 4. Top KPI Scorecards

Position 4 uniform cards across `Y: 90`, each with `Width: 320` and `Height: 90`.

### 4.1 Card 1: Total Gross Revenue
- Click **Add a chart** $\rightarrow$ **Scorecard**.
- Placement: `X: 30`, `Y: 90`.
- **Setup Tab:**
  - Metric $\rightarrow$ **Create field**:
    - **Name:** `Total Gross Revenue`
    - **Formula:** `SUM(bq_earnings) + SUM(AdSense_Earnings)`
    - **Type:** Numeric $\rightarrow$ Currency $\rightarrow$ USD ($)
- **Style Tab:**
  - Alignment: Center. Font Size: `28px` Bold.
  - Background: `#FFFFFF` (or `#1E293B`), Border Radius: `6px`.

### 4.2 Card 2: Total Impressions
- Duplicate Card 1 $\rightarrow$ Placement: `X: 380`, `Y: 90`.
- **Setup Tab:**
  - Metric $\rightarrow$ **Create field**:
    - **Name:** `Total Impressions`
    - **Formula:** `SUM(impressions) + SUM(AdSense_Impressions)`
    - **Type:** Numeric $\rightarrow$ Number.
- **Style Tab:**
  - Check **Compact numbers** (displays as e.g. `85.4K`).

### 4.3 Card 3: Blended eCPM
- Duplicate Card 2 $\rightarrow$ Placement: `X: 730`, `Y: 90`.
- **Setup Tab:**
  - Metric $\rightarrow$ **Create field**:
    - **Name:** `Blended eCPM`
    - **Formula:** `((SUM(bq_earnings) + SUM(AdSense_Earnings)) / (SUM(impressions) + SUM(AdSense_Impressions))) * 1000`
    - **Type:** Numeric $\rightarrow$ Currency $\rightarrow$ USD ($).

### 4.4 Card 4: Inventory Fill Rate %
- Duplicate Card 3 $\rightarrow$ Placement: `X: 1080`, `Y: 90`.
- **Setup Tab:**
  - Metric $\rightarrow$ **Create field**:
    - **Name:** `Fill Rate %`
    - **Formula:** `(SUM(impressions) / SUM(ad_requests)) * 100`
    - **Type:** Numeric $\rightarrow$ Percent.

---

## 5. Mid-Fold Charts

### 5.1 Geographic Bubble / Heatmap Chart
- Click **Add a chart** $\rightarrow$ **Geo chart** (or **Bubble Map**).
- Placement: `X: 30`, `Y: 200`, `Width: 670`, `Height: 360`.
- **Setup Tab:**
  - Data Source: `Blend_BQ_AdSense`
  - Geo Dimension: `country_name`
  - Color Metric: `Total Gross Revenue`
- **Style Tab:**
  - Min Color: `#D1E7DD` (Light Green).
  - Max Color: `#0F5132` (Deep Emerald).
  - Check **Cross-filtering** (selecting a country filters the entire dashboard to that region).

### 5.2 Stacked Column Bar Chart
- Click **Add a chart** $\rightarrow$ **Stacked column chart**.
- Placement: `X: 730`, `Y: 200`, `Width: 670`, `Height: 360`.
- **Setup Tab:**
  - Dimension: `category`
  - Metrics:
    - `bq_earnings` (Rename label to `Direct BQ Yield`)
    - `AdSense_Earnings` (Rename label to `AdSense Network`)
  - Sort: `Total Gross Revenue` descending.
- **Style Tab:**
  - Series 1 Color: Royal Blue (`#2563EB`).
  - Series 2 Color: Amber (`#F59E0B`).
  - Check **Show data labels**.
  - Check **Stacked bars**.

---

## 6. Lower-Fold Visuals

### 6.1 Device Category Donut Chart
- Click **Add a chart** $\rightarrow$ **Donut chart**.
- Placement: `X: 30`, `Y: 580`, `Width: 420`, `Height: 380`.
- **Setup Tab:**
  - Dimension: `device_category`
  - Metric: `Total Gross Revenue`
  - Sort: `Total Gross Revenue` descending.
- **Style Tab:**
  - Hole size: `60%`.
  - Slice Colors: Cyan (`#06B6D4`) for Mobile, Indigo (`#4F46E5`) for Desktop.
  - Slice label: **Percentage**.
  - Check **Cross-filtering**.

### 6.2 Heatmap Table (Multi-Dimension Yield Analysis)
- Click **Add a chart** $\rightarrow$ **Table with heatmap**.
- Placement: `X: 470`, `Y: 580`, `Width: 930`, `Height: 380`.
- **Setup Tab:**
  - Dimensions: `category`, `sub_category`, `ad_unit_name`
  - Metrics:
    1. `page_views`
    2. `Total Impressions`
    3. `Total Gross Revenue`
    4. Calculated Field `Blended Page RPM`:
       ```sql
       ((SUM(bq_earnings) + SUM(AdSense_Earnings)) / SUM(page_views)) * 1000
       ```
    5. Calculated Field `Blended CTR %`:
       ```sql
       ((SUM(clicks) + SUM(AdSense_Clicks)) / (SUM(impressions) + SUM(AdSense_Impressions))) * 100
       ```
  - Check **Show summary row**.
- **Style Tab:**
  - Table Header: Fill Color `#1E293B`, Text `#FFFFFF`, Bold.
  - Metric Column Style: Set Heatmap gradient color for `Blended Page RPM` (`#E8F5E9` to `#2E7D32`).
  - Rows per page: `10`.