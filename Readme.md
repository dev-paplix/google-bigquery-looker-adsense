# Enterprise Ad Monetization Analytics: BigQuery & Looker Studio for Google AdSense

A production-grade, step-by-step technical guide for setting up a multi-dimensional dummy ad telemetry dataset in Google BigQuery, connecting it to Google Looker Studio, blending it with Google AdSense, and configuring multi-channel monetization metrics.

---

## 1. Architecture Overview

```
+------------------------------------+       +------------------------------------+
|          Google BigQuery           |       |           Google AdSense           |
| (Raw Pageviews, Direct Ad Revenue, |       |    (Ad Network Fill, Impressions,  |
|   Geo Telemetry, Device Splits)    |       |         Earnings, Clicks)          |
+-----------------+------------------+       +-----------------+------------------+
                  |                                            |
                  +---------------------+----------------------+
                                        |
                                        v
                       +----------------------------------+
                       |       Looker Studio Engine       |
                       |  - Left Outer Join on Date + Geo |
                       |  - Calculated Yield Metrics      |
                       |  - Interactive Dashboard Visuals |
                       +----------------------------------+
```

---

## 2. Step 1: BigQuery Setup & Rich Dummy Data Generation

This dataset provides multi-dimensional advertising telemetry containing geographical markers, editorial taxonomy, device categories, traffic volumes, and direct monetization logs.

### 2.1 Navigate to BigQuery Studio
1. Open the [Google Cloud Console BigQuery Studio](https://console.cloud.google.com/bigquery).
2. Verify that your designated Google Cloud Platform (GCP) project is selected from the project dropdown in the top navigation bar.

### 2.2 Create the Dataset
1. In the **Explorer** panel on the left, locate your project ID.
2. Click the three vertical action dots next to your project ID and select **Create dataset**.
3. Configure the dataset settings:
   - **Dataset ID:** `cpg_monetization`
   - **Data location:** `US (multiple regions in United States)` or your regional preference.
4. Click **Create dataset**.

### 2.3 Populate Multi-Dimension Dummy Data
Click **+ (Compose new query)**, paste the following SQL script into the query editor, and click **Run**:

```sql
CREATE OR REPLACE TABLE `cpg_monetization.ad_telemetry_dummy` AS
SELECT
  PARSE_DATE('%Y-%m-%d', date_str) AS date,
  country_code,
  country_name,
  device_category,
  category,
  sub_category,
  ad_unit_name,
  ad_requests,
  impressions,
  clicks,
  page_views,
  ROUND(earnings, 2) AS bq_earnings
FROM UNNEST([
  -- 2026-11-01
  STRUCT('2026-11-01' AS date_str, 'US' AS country_code, 'United States' AS country_name, 'Mobile' AS device_category, 'Beauty' AS category, 'Anti-Aging' AS sub_category, 'Top_Leaderboard' AS ad_unit_name, 18500 AS ad_requests, 15400 AS impressions, 245 AS clicks, 19200 AS page_views, 62.50 AS earnings),
  STRUCT('2026-11-01', 'US', 'United States', 'Desktop', 'Wellness', 'Gut Health', 'Sidebar_Sticky', 12000, 10200, 180, 13100, 48.20),
  STRUCT('2026-11-01', 'GB', 'United Kingdom', 'Mobile', 'Beauty', 'Skincare Routines', 'In_Article_1', 9800, 8400, 135, 10500, 38.60),
  STRUCT('2026-11-01', 'CA', 'Canada', 'Desktop', 'Home Care', 'Eco Cleaners', 'In_Article_2', 6200, 5100, 68, 7000, 19.40),
  STRUCT('2026-11-01', 'AU', 'Australia', 'Mobile', 'Wellness', 'Supplements', 'Top_Leaderboard', 7500, 6300, 95, 8200, 27.80),
  STRUCT('2026-11-01', 'DE', 'Germany', 'Desktop', 'Beauty', 'Anti-Aging', 'Sidebar_Sticky', 5900, 4800, 62, 6500, 21.30),

  -- 2026-11-02
  STRUCT('2026-11-02', 'US', 'United States', 'Mobile', 'Beauty', 'Anti-Aging', 'Top_Leaderboard', 19800, 16800, 280, 20900, 71.40),
  STRUCT('2026-11-02', 'US', 'United States', 'Desktop', 'Wellness', 'Gut Health', 'Sidebar_Sticky', 13100, 11400, 210, 14300, 54.10),
  STRUCT('2026-11-02', 'GB', 'United Kingdom', 'Mobile', 'Beauty', 'Skincare Routines', 'In_Article_1', 10400, 9100, 152, 11200, 42.10),
  STRUCT('2026-11-02', 'CA', 'Canada', 'Desktop', 'Home Care', 'Eco Cleaners', 'In_Article_2', 5800, 4700, 59, 6400, 17.50),
  STRUCT('2026-11-02', 'AU', 'Australia', 'Mobile', 'Wellness', 'Supplements', 'Top_Leaderboard', 8100, 7000, 112, 9000, 31.50),
  STRUCT('2026-11-02', 'DE', 'Germany', 'Desktop', 'Beauty', 'Anti-Aging', 'Sidebar_Sticky', 6400, 5300, 71, 7100, 24.80),

  -- 2026-11-03
  STRUCT('2026-11-03', 'US', 'United States', 'Mobile', 'Beauty', 'Anti-Aging', 'Top_Leaderboard', 21500, 18200, 310, 22400, 80.20),
  STRUCT('2026-11-03', 'US', 'United States', 'Desktop', 'Wellness', 'Gut Health', 'Sidebar_Sticky', 12500, 10800, 195, 13800, 51.00),
  STRUCT('2026-11-03', 'GB', 'United Kingdom', 'Mobile', 'Beauty', 'Skincare Routines', 'In_Article_1', 11200, 9800, 168, 12100, 46.70),
  STRUCT('2026-11-03', 'CA', 'Canada', 'Desktop', 'Home Care', 'Eco Cleaners', 'In_Article_2', 6500, 5400, 74, 7300, 20.80),
  STRUCT('2026-11-03', 'AU', 'Australia', 'Mobile', 'Wellness', 'Supplements', 'Top_Leaderboard', 8600, 7400, 120, 9600, 33.90),
  STRUCT('2026-11-03', 'DE', 'Germany', 'Desktop', 'Beauty', 'Anti-Aging', 'Sidebar_Sticky', 6100, 5000, 66, 6800, 22.50)
]);
```

---

## 3. Step 2: Looker Studio Ingestion & Field Formatting

1. Navigate to [Google Looker Studio](https://lookerstudio.google.com/).
2. Click **Create** $\rightarrow$ **Report**.
3. Under the **Connect to data** prompt:
   - Select the native **BigQuery** connector.
   - Choose **My Projects** $\rightarrow$ select your GCP project.
   - Select Dataset: `cpg_monetization`.
   - Select Table: `ad_telemetry_dummy`.
   - Click **Add** $\rightarrow$ **Add to Report**.
4. Confirm Field Data Types:
   - Go to **Resource** $\rightarrow$ **Manage added data sources** $\rightarrow$ click **Edit** next to `ad_telemetry_dummy`.
   - Verify `date` is recognized as **Date (YYYYMMDD)**.
   - Change `country_name` or `country_code` type to **Geo** $\rightarrow$ **Country**.
   - Change `bq_earnings` type to **Numeric** $\rightarrow$ **Currency** $\rightarrow$ **USD ($)**.
   - Click **Done**.

---

## 4. Step 3: Google AdSense Connection

### Option A: Live Production AdSense Connector
1. In Looker Studio, navigate to **Resource** $\rightarrow$ **Manage added data sources** $\rightarrow$ **Add a Data Source**.
2. Select the native **AdSense** connector.
3. Authorize Google account credentials, select your ad account, and click **Connect** $\rightarrow$ **Add to Report**.

### Option B: Synthetic AdSense Benchmark Ledger (Testing Alternative)
If developing offline without active AdSense property permissions, create a Google Sheet containing benchmark ad exchange earnings:

| Date | Country_Code | AdSense_Impressions | AdSense_Clicks | AdSense_Earnings |
| :--- | :--- | :--- | :--- | :--- |
| 2026-11-01 | US | 12000 | 190 | 45.00 |
| 2026-11-01 | GB | 7000 | 95 | 28.50 |
| 2026-11-01 | CA | 4000 | 45 | 14.00 |
| 2026-11-02 | US | 13500 | 215 | 52.20 |
| 2026-11-02 | GB | 7800 | 110 | 31.00 |
| 2026-11-02 | CA | 3800 | 40 | 13.10 |
| 2026-11-03 | US | 14200 | 230 | 58.00 |
| 2026-11-03 | GB | 8100 | 125 | 34.50 |
| 2026-11-03 | CA | 4300 | 50 | 15.60 |

Add this sheet to Looker Studio via the **Google Sheets** connector.

---

## 5. Step 4: Multi-Channel Data Blending

1. Go to **Resource** $\rightarrow$ **Manage blends** $\rightarrow$ **Add a blend**.
2. **Table 1 (`BigQuery_Core`):**
   - Source: `ad_telemetry_dummy`
   - **Join Keys:** `date`, `country_code`
   - **Dimensions:** `country_name`, `device_category`, `category`, `sub_category`, `ad_unit_name`
   - **Metrics:** `ad_requests`, `impressions`, `clicks`, `page_views`, `bq_earnings`
3. **Table 2 (`AdSense_Channel`):**
   - Source: `AdSense` (or AdSense Benchmark Sheet)
   - **Join Keys:** `Date`, `Country_Code`
   - **Metrics:** `AdSense_Earnings`, `AdSense_Impressions`, `AdSense_Clicks`
4. **Join Configuration:**
   - Click **Configure join**.
   - Operator: **Left Outer**.
   - Conditions:
     - `date` = `Date`
     - `country_code` = `Country_Code`
   - Click **Save**.
5. Set the blend name in the top left to `Blend_BQ_AdSense`, click **Save**, and close the blend window.

---

## 6. Step 5: Core Yield Metric Formulas

Copy and paste these exact formulas into Looker Studio when creating calculated fields:

- **Total Combined Earnings:**
  ```sql
  SUM(bq_earnings) + SUM(AdSense_Earnings)
  ```

- **Total Blended Impressions:**
  ```sql
  SUM(impressions) + SUM(AdSense_Impressions)
  ```

- **Blended eCPM:**
  ```sql
  ((SUM(bq_earnings) + SUM(AdSense_Earnings)) / (SUM(impressions) + SUM(AdSense_Impressions))) * 1000
  ```

- **Blended Page RPM:**
  ```sql
  ((SUM(bq_earnings) + SUM(AdSense_Earnings)) / SUM(page_views)) * 1000
  ```

- **Fill Rate %:**
  ```sql
  (SUM(impressions) / SUM(ad_requests)) * 100
  ```

- **Blended CTR %:**
  ```sql
  ((SUM(clicks) + SUM(AdSense_Clicks)) / (SUM(impressions) + SUM(AdSense_Impressions))) * 100
  ```