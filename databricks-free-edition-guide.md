# Databricks Free Edition - Complete Hands-On Project Guide

## Step-by-Step Demo Project from Zero to Production

---

## Project Overview

**Project**: NYC Taxi Analytics Platform  
**Objective**: Build complete data pipeline using Medallion Architecture  
**Time**: 2-3 hours  
**Platform**: Databricks Free Edition

### What You'll Build
```
┌─────────────────────────────────────────────────┐
│            COMPLETE DATA PLATFORM                │
├─────────────────────────────────────────────────┤
│                                                  │
│  Bronze Layer → Silver Layer → Gold Layer       │
│  (Raw Data)     (Cleansed)     (Business-Ready) │
│                                                  │
│  + ML Model for Tip Prediction                  │
│  + Interactive Dashboards                       │
│  + Scheduled Jobs                               │
│                                                  │
└─────────────────────────────────────────────────┘
```

### Skills You'll Learn

- ✅ Navigate Databricks workspace
- ✅ Create and use notebooks
- ✅ Work with Unity Catalog
- ✅ Build Delta Lake tables
- ✅ Create data visualizations
- ✅ Track ML experiments with MLflow
- ✅ Schedule production jobs

---

## Prerequisites

### What You Have

- ✅ Databricks Free Edition account created
- ✅ Workspace provisioned and accessible
- ✅ Internet connection

### What's Included in Free Edition

| Feature | Status | Notes |
|---------|--------|-------|
| Serverless Compute | ✅ Available | Auto-provisioned |
| Unity Catalog | ✅ Enabled | Default catalog created |
| Default Storage | ✅ Provided | No setup needed |
| Sample Data | ✅ Available | `samples` catalog |
| Python Support | ✅ Yes | Primary language |
| SQL Support | ✅ Yes | In notebooks |
| Scala/R Support | ❌ No | Free Edition limitation |
| Outbound Internet | ❌ Blocked | Must upload files via UI |

---

## STEP 1: Access Your Workspace (2 minutes)

### 1.1 Login to Your Workspace

**Action:**
1. Open your browser
2. Go to your Databricks workspace URL
   - Format: `https://<workspace-id>.cloud.databricks.com`
   - Or use the link from your welcome email

3. Login with your credentials (Google, GitHub, Microsoft, or Email)

### 1.2 Explore the UI

**Workspace Layout:**

```
┌────────────────────────────────────────────────────┐
│  DATABRICKS WORKSPACE UI                           │
├────────────────────────────────────────────────────┤
│                                                     │
│  Left Sidebar:                                      │
│  ├── 🏠 Home                                       │
│  ├── 📂 Workspace (your notebooks)                 │
│  ├── 📊 Compute (clusters)                         │
│  ├── 🗃️ Catalog (Unity Catalog)                   │
│  ├── 🔍 SQL (SQL queries)                          │
│  ├── 📈 Dashboards                                 │
│  ├── 🤖 Jobs (scheduled tasks)                     │
│  ├── 🧪 Experiments (MLflow)                       │
│  └── ⚙️ Settings                                   │
│                                                     │
│  Main Area:                                         │
│  └── Your workspace content                        │
│                                                     │
└────────────────────────────────────────────────────┘
```

### 1.3 Verify Serverless Compute

**Action:**
1. Click **Compute** in left sidebar
2. You should see serverless compute available
3. No need to create clusters - serverless auto-provisions

```
✓ Serverless compute is always ready
✓ No cluster creation needed
✓ Pay only for what you use (with free quota)
```

---

## STEP 2: Project Setup (5 minutes)

### 2.1 Create Project Folder

**Action:**
1. Click **Workspace** in left sidebar
2. Click your username folder (or Home)
3. Click **Create** → **Folder**
4. Name it: `NYC_Taxi_Project`
5. Click **Create Folder**

### 2.2 Create Folder Structure

Inside `NYC_Taxi_Project`, create these subfolders:

**Action:**
1. Right-click on `NYC_Taxi_Project`
2. Create these folders:

```
NYC_Taxi_Project/
├── 01_Bronze/
├── 02_Silver/
├── 03_Gold/
├── 04_ML_Models/
└── 05_Dashboards/
```

### 2.3 Verify Unity Catalog Access

**Action:**
1. Click **Catalog** in left sidebar
2. You should see:
   - `samples` catalog (sample datasets)
   - Your workspace catalog (named after your workspace)

```
Expected View:
──────────────────────────────────
hive_metastore (default)
├── default (schema)
└── ...

samples (read-only)
├── nyctaxi
│   └── trips (table)
├── tpch
└── ...

<your-workspace-name> (your catalog)
├── default (schema)
└── ...
```

### 2.4 Find Your Catalog Name

**Important**: Note your workspace catalog name

1. In Catalog, find the catalog with your workspace name
2. This is where you'll create tables
3. Write it down: `_______________`

```
Example:
If your workspace is: my-workspace-123456
Your catalog might be: my_workspace_123456
(Note: hyphens often converted to underscores)
```

---

## STEP 3: Bronze Layer - Raw Data Ingestion (15 minutes)

### 3.1 Create Bronze Notebook

**Action:**
1. Navigate to `NYC_Taxi_Project` → `01_Bronze`
2. Click **Create** → **Notebook**
3. Configure:
   - Name: `Bronze_Raw_Taxi_Data`
   - Default Language: **Python**
   - No cluster needed (serverless auto-attached)
4. Click **Create**

### 3.2 Add Notebook Code

**Cell 1: Setup and Configuration**

```python
# CELL 1: Configuration
# Purpose: Set up variables for the project

# Find your catalog name in Catalog browser
# Replace with your actual catalog name
catalog_name = "your_workspace_catalog"  # <-- UPDATE THIS
schema_name = "default"

# Bronze table name
bronze_table = "taxi_trips_bronze"

# Full table path
bronze_full_path = f"{catalog_name}.{schema_name}.{bronze_table}"

print(f"✓ Configuration set")
print(f"  Catalog: {catalog_name}")
print(f"  Schema: {schema_name}")
print(f"  Bronze Table: {bronze_full_path}")

# How to find your catalog name:
# 1. Click "Catalog" in left sidebar
# 2. Look for catalog named after your workspace
# 3. Copy that exact name (replace hyphens with underscores if needed)
```

**Run the cell:**
- Press `Shift + Enter`
- Or click the ▶ (play) button

**Note:** Update `catalog_name` with your actual catalog name from Step 2.4

**Cell 2: Explore Sample Data**

```python
# CELL 2: Explore Sample Data
# Purpose: Look at what data is available

# NYC Taxi sample dataset
print("=== NYC Taxi Sample Dataset ===\n")

# Read sample data
raw_trips_df = spark.read.table("samples.nyctaxi.trips")

# Show basic info
print(f"Total Records: {raw_trips_df.count():,}")
print(f"\nSchema:")
raw_trips_df.printSchema()

print(f"\nFirst 5 rows:")
display(raw_trips_df.limit(5))
```

**Expected Output:**
```
=== NYC Taxi Sample Dataset ===

Total Records: 2,XXX,XXX

Schema:
root
 |-- tpep_pickup_datetime: timestamp (nullable = true)
 |-- tpep_dropoff_datetime: timestamp (nullable = true)
 |-- trip_distance: double (nullable = true)
 |-- fare_amount: double (nullable = true)
 ...

[Table showing first 5 rows]
```

**Cell 3: Add Ingestion Metadata**

```python
# CELL 3: Add Ingestion Metadata
# Purpose: Add tracking columns for data lineage

from pyspark.sql.functions import current_timestamp, lit

# Add metadata columns
bronze_trips_df = raw_trips_df \
    .withColumn("ingestion_timestamp", current_timestamp()) \
    .withColumn("source_system", lit("samples.nyctaxi.trips")) \
    .withColumn("ingestion_date", current_timestamp().cast("date"))

print("✓ Added ingestion metadata")
print(f"\nNew columns added:")
print("  - ingestion_timestamp")
print("  - source_system")
print("  - ingestion_date")

print(f"\nTotal columns: {len(bronze_trips_df.columns)}")
```

**Cell 4: Write to Bronze Table**

```python
# CELL 4: Write to Bronze Table
# Purpose: Save raw data to Unity Catalog

# Write as Delta table
print("Writing Bronze table...")
bronze_trips_df.write \
    .format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable(bronze_full_path)

print(f"✓ Bronze table created: {bronze_full_path}")

# Verify
record_count = spark.read.table(bronze_full_path).count()
print(f"✓ Verification: {record_count:,} records saved")
```

**Expected Output:**
```
Writing Bronze table...
✓ Bronze table created: your_catalog.default.taxi_trips_bronze
✓ Verification: 2,XXX,XXX records saved
```

**Cell 5: Verify Bronze Table**

```python
# CELL 5: Verify Bronze Table
# Purpose: Confirm data is in Unity Catalog

# Read back
bronze_data = spark.read.table(bronze_full_path)

print("=== BRONZE LAYER SUMMARY ===\n")
print(f"Table: {bronze_full_path}")
print(f"Records: {bronze_data.count():,}")
print(f"Columns: {len(bronze_data.columns)}")

print("\nSchema:")
bronze_data.printSchema()

print("\nSample data:")
display(bronze_data.limit(5))
```

### 3.3 Checkpoint

**Verify Bronze Table in Catalog:**
1. Click **Catalog** in left sidebar
2. Navigate to your catalog → `default` schema
3. You should see `taxi_trips_bronze` table
4. Click on it to see schema and sample data

```
✓ Bronze Layer Complete
  - Raw data saved
  - Ingestion metadata added
  - Table visible in Unity Catalog
```

---

## STEP 4: Silver Layer - Data Cleansing (20 minutes)

### 4.1 Create Silver Notebook

**Action:**
1. Navigate to `NYC_Taxi_Project` → `02_Silver`
2. Create notebook: `Silver_Cleansed_Taxi_Data`
3. Language: **Python**

### 4.2 Add Notebook Code

**Cell 1: Configuration**

```python
# CELL 1: Configuration
catalog_name = "your_workspace_catalog"  # <-- UPDATE THIS
schema_name = "default"

# Table names
bronze_table = f"{catalog_name}.{schema_name}.taxi_trips_bronze"
silver_table = f"{catalog_name}.{schema_name}.taxi_trips_silver"

print(f"Source (Bronze): {bronze_table}")
print(f"Target (Silver): {silver_table}")
```

**Cell 2: Load Bronze Data**

```python
# CELL 2: Load Bronze Data
bronze_df = spark.read.table(bronze_table)

print(f"✓ Loaded Bronze data: {bronze_df.count():,} records")
display(bronze_df.limit(5))
```

**Cell 3: Data Quality Analysis**

```python
# CELL 3: Data Quality Analysis
# Purpose: Identify data quality issues

from pyspark.sql.functions import col, count, when, isnull

print("=== DATA QUALITY ANALYSIS ===\n")

# Check for invalid values
print("Invalid Records Count:")
print(f"  Negative fare: {bronze_df.filter(col('fare_amount') < 0).count():,}")
print(f"  Zero distance: {bronze_df.filter(col('trip_distance') == 0).count():,}")
print(f"  Negative distance: {bronze_df.filter(col('trip_distance') < 0).count():,}")
print(f"  Null pickup time: {bronze_df.filter(isnull('tpep_pickup_datetime')).count():,}")

# Show statistics
print("\nNumeric Column Statistics:")
bronze_df.select("fare_amount", "trip_distance", "tip_amount").describe().display()
```

**Cell 4: Apply Data Cleansing**

```python
# CELL 4: Apply Data Quality Rules
# Purpose: Clean the data

from pyspark.sql.functions import col

print("=== APPLYING DATA QUALITY RULES ===\n")

# Start with Bronze data
silver_df = bronze_df

# Rule 1: Remove negative fare
silver_df = silver_df.filter(col("fare_amount") >= 0)
print("✓ Rule 1: Removed negative fare trips")

# Rule 2: Remove zero/negative distance
silver_df = silver_df.filter(col("trip_distance") > 0)
print("✓ Rule 2: Removed zero/negative distance trips")

# Rule 3: Remove null timestamps
silver_df = silver_df.filter(col("tpep_pickup_datetime").isNotNull())
silver_df = silver_df.filter(col("tpep_dropoff_datetime").isNotNull())
print("✓ Rule 3: Removed trips with null timestamps")

# Rule 4: Remove invalid time sequences
silver_df = silver_df.filter(col("tpep_dropoff_datetime") > col("tpep_pickup_datetime"))
print("✓ Rule 4: Removed trips with invalid time sequences")

print(f"\n✓ Total cleansed records: {silver_df.count():,}")
```

**Cell 5: Add Derived Columns**

```python
# CELL 5: Add Derived Columns
# Purpose: Calculate business metrics

from pyspark.sql.functions import (
    col, unix_timestamp, round as spark_round,
    hour, dayofweek, month, year, when
)

print("=== ADDING DERIVED COLUMNS ===\n")

# Calculate trip duration (minutes)
silver_df = silver_df.withColumn(
    "trip_duration_minutes",
    spark_round(
        (unix_timestamp("tpep_dropoff_datetime") - unix_timestamp("tpep_pickup_datetime")) / 60,
        2
    )
)
print("✓ Added: trip_duration_minutes")

# Calculate average speed (mph)
silver_df = silver_df.withColumn(
    "avg_speed_mph",
    spark_round(
        when(col("trip_duration_minutes") > 0,
            col("trip_distance") / (col("trip_duration_minutes") / 60)
        ).otherwise(0),
        2
    )
)
print("✓ Added: avg_speed_mph")

# Extract time dimensions
silver_df = silver_df \
    .withColumn("pickup_hour", hour("tpep_pickup_datetime")) \
    .withColumn("pickup_day_of_week", dayofweek("tpep_pickup_datetime")) \
    .withColumn("pickup_month", month("tpep_pickup_datetime")) \
    .withColumn("pickup_year", year("tpep_pickup_datetime"))
print("✓ Added: pickup_hour, pickup_day_of_week, pickup_month, pickup_year")

# Calculate tip percentage
silver_df = silver_df.withColumn(
    "tip_percentage",
    spark_round(
        when(col("fare_amount") > 0,
            (col("tip_amount") / col("fare_amount")) * 100
        ).otherwise(0),
        2
    )
)
print("✓ Added: tip_percentage")

# Create high-tip flag (> 20% tip)
silver_df = silver_df.withColumn(
    "is_high_tip",
    when(col("tip_percentage") > 20, 1).otherwise(0)
)
print("✓ Added: is_high_tip")

print(f"\nTotal columns: {len(silver_df.columns)}")

# Show sample
print("\nSample data with derived columns:")
display(silver_df.select(
    "trip_distance", "fare_amount", "tip_amount",
    "trip_duration_minutes", "avg_speed_mph", 
    "pickup_hour", "tip_percentage", "is_high_tip"
).limit(5))
```

**Cell 6: Write Silver Table**

```python
# CELL 6: Write to Silver Table
# Purpose: Save cleansed data

print("Writing Silver table...")
silver_df.write \
    .format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable(silver_table)

print(f"✓ Silver table created: {silver_table}")

# Verify
record_count = spark.read.table(silver_table).count()
print(f"✓ Verification: {record_count:,} records saved")
```

**Cell 7: Silver Layer Summary**

```python
# CELL 7: Silver Layer Summary
silver_data = spark.read.table(silver_table)

print("=== SILVER LAYER SUMMARY ===\n")
print(f"Table: {silver_table}")
print(f"Records: {silver_data.count():,}")
print(f"Columns: {len(silver_data.columns)}")

print("\nKey Statistics:")
silver_data.select(
    "trip_distance", "fare_amount", "tip_amount",
    "trip_duration_minutes", "tip_percentage"
).describe().display()

print("\nHigh-Tip Distribution:")
silver_data.groupBy("is_high_tip").count().display()
```

### 4.3 Checkpoint

```
✓ Silver Layer Complete
  - Data quality rules applied
  - 7 derived columns added
  - Cleansed data saved
  - Table visible in Unity Catalog
```

---

## STEP 5: Gold Layer - Business Aggregations (25 minutes)

### 5.1 Create Gold Notebook

**Action:**
1. Navigate to `NYC_Taxi_Project` → `03_Gold`
2. Create notebook: `Gold_Business_Metrics`
3. Language: **Python**

### 5.2 Add Notebook Code

**Cell 1: Configuration**

```python
# CELL 1: Configuration
catalog_name = "your_workspace_catalog"  # <-- UPDATE THIS
schema_name = "default"

silver_table = f"{catalog_name}.{schema_name}.taxi_trips_silver"
print(f"Source (Silver): {silver_table}")
```

**Cell 2: Load Silver Data**

```python
# CELL 2: Load Silver Data
silver_df = spark.read.table(silver_table)
print(f"✓ Loaded Silver data: {silver_df.count():,} records")
```

**Cell 3: Create Hourly Metrics**

```python
# CELL 3: Create Hourly Metrics Table
# Purpose: Aggregate metrics by hour

from pyspark.sql.functions import col, count, sum, avg

hourly_metrics = silver_df.groupBy("pickup_hour") \
    .agg(
        count("*").alias("total_trips"),
        sum("fare_amount").alias("total_revenue"),
        avg("fare_amount").alias("avg_fare"),
        avg("trip_distance").alias("avg_distance"),
        avg("trip_duration_minutes").alias("avg_duration"),
        sum("tip_amount").alias("total_tips"),
        avg("tip_percentage").alias("avg_tip_percentage")
    ) \
    .orderBy("pickup_hour")

# Write to Gold
hourly_metrics.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_hourly_metrics")

print("✓ Created: gold_hourly_metrics")
print(f"Records: {hourly_metrics.count()} (24 hours)")
display(hourly_metrics)
```

**Cell 4: Create Location Metrics**

```python
# CELL 4: Create Location Metrics Table
# Purpose: Analyze pickup locations

location_metrics = silver_df.groupBy("pickup_zip") \
    .agg(
        count("*").alias("total_trips"),
        sum("fare_amount").alias("total_revenue"),
        avg("fare_amount").alias("avg_fare"),
        avg("trip_distance").alias("avg_distance"),
        sum("tip_amount").alias("total_tips"),
        avg("tip_percentage").alias("avg_tip_percentage")
    ) \
    .orderBy(col("total_trips").desc())

# Write to Gold
location_metrics.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_location_metrics")

print("✓ Created: gold_location_metrics")
print(f"Records: {location_metrics.count()}")
display(location_metrics.limit(10))
```

**Cell 5: Create Daily Trends**

```python
# CELL 5: Create Daily Trends Table
# Purpose: Time series analysis

from pyspark.sql.functions import to_date

daily_metrics = silver_df \
    .withColumn("trip_date", to_date("tpep_pickup_datetime")) \
    .groupBy("trip_date") \
    .agg(
        count("*").alias("total_trips"),
        sum("fare_amount").alias("total_revenue"),
        avg("fare_amount").alias("avg_fare"),
        sum("tip_amount").alias("total_tips"),
        avg("tip_percentage").alias("avg_tip_percentage")
    ) \
    .orderBy("trip_date")

# Write to Gold
daily_metrics.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_daily_trends")

print("✓ Created: gold_daily_trends")
print(f"Records: {daily_metrics.count()}")
display(daily_metrics.limit(10))
```

**Cell 6: High-Tip Analysis**

```python
# CELL 6: High-Tip Pattern Analysis
# Purpose: Identify high-tip patterns

high_tip_analysis = silver_df.groupBy("pickup_hour", "is_high_tip") \
    .agg(
        count("*").alias("trip_count"),
        avg("trip_distance").alias("avg_distance"),
        avg("fare_amount").alias("avg_fare"),
        avg("tip_amount").alias("avg_tip"),
        avg("tip_percentage").alias("avg_tip_percentage")
    ) \
    .orderBy("pickup_hour", "is_high_tip")

# Write to Gold
high_tip_analysis.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_high_tip_patterns")

print("✓ Created: gold_high_tip_patterns")
display(high_tip_analysis)
```

**Cell 7: Executive KPIs**

```python
# CELL 7: Create Executive KPIs Table
# Purpose: High-level business metrics

from pyspark.sql.functions import lit

# Calculate KPIs
total_trips = silver_df.count()
total_revenue = silver_df.agg(sum("fare_amount")).collect()[0][0]
total_tips = silver_df.agg(sum("tip_amount")).collect()[0][0]
avg_fare = silver_df.agg(avg("fare_amount")).collect()[0][0]
avg_tip_pct = silver_df.agg(avg("tip_percentage")).collect()[0][0]
high_tip_trips = silver_df.filter(col("is_high_tip") == 1).count()

# Create KPI table
kpis = spark.createDataFrame([
    ("Total Trips", f"{total_trips:,}"),
    ("Total Revenue", f"${total_revenue:,.2f}"),
    ("Total Tips", f"${total_tips:,.2f}"),
    ("Average Fare", f"${avg_fare:.2f}"),
    ("Average Tip %", f"{avg_tip_pct:.2f}%"),
    ("High-Tip Trips", f"{high_tip_trips:,} ({(high_tip_trips/total_trips)*100:.1f}%)")
], ["Metric", "Value"])

# Write to Gold
kpis.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_executive_kpis")

print("=== EXECUTIVE KPIS ===\n")
display(kpis)
```

### 5.3 Checkpoint

**Verify in Catalog:**
1. Go to **Catalog** → your catalog → `default` schema
2. You should see 5 new tables:
   - `gold_hourly_metrics`
   - `gold_location_metrics`
   - `gold_daily_trends`
   - `gold_high_tip_patterns`
   - `gold_executive_kpis`

```
✓ Gold Layer Complete
  - 5 aggregated tables created
  - Business metrics calculated
  - Ready for dashboards
```

---

## STEP 6: Create Visualizations (20 minutes)

### 6.1 Create Visualization Notebook

**Action:**
1. Navigate to `NYC_Taxi_Project` → `03_Gold`
2. Create notebook: `Visualizations`
3. Language: **Python**

### 6.2 Add Visualization Code

**Cell 1: Load Gold Tables**

```python
# CELL 1: Load Gold Tables
catalog_name = "your_workspace_catalog"  # <-- UPDATE THIS
schema_name = "default"

hourly_df = spark.read.table(f"{catalog_name}.{schema_name}.gold_hourly_metrics")
location_df = spark.read.table(f"{catalog_name}.{schema_name}.gold_location_metrics")
daily_df = spark.read.table(f"{catalog_name}.{schema_name}.gold_daily_trends")

print("✓ Loaded Gold tables")
```

**Cell 2: Trips by Hour**

```python
# CELL 2: Trips by Hour
# Purpose: Show trip patterns by hour

display(hourly_df.select("pickup_hour", "total_trips"))
```

**Create Visualization:**
1. Run the cell
2. Click **+** next to **Table** → **Visualization**
3. Configure:
   - Visualization Type: **Bar**
   - X Column: `pickup_hour`
   - Y Column: `total_trips`
   - Aggregation: None
4. Click **Save**

**Cell 3: Revenue by Hour**

```python
# CELL 3: Revenue by Hour
display(hourly_df.select("pickup_hour", "total_revenue", "total_tips"))
```

**Visualization:** Bar chart
- X: `pickup_hour`
- Y: `total_revenue`

**Cell 4: Top 10 Locations**

```python
# CELL 4: Top 10 Pickup Locations
display(location_df.select("pickup_zip", "total_trips", "total_revenue").limit(10))
```

**Visualization:** Bar chart
- X: `pickup_zip`
- Y: `total_trips`

**Cell 5: Daily Trends**

```python
# CELL 5: Daily Trip Trends
display(daily_df.select("trip_date", "total_trips", "total_revenue"))
```

**Visualization:** Line chart
- X: `trip_date`
- Y: `total_trips`
- Add series: `total_revenue`

**Cell 6: Average Fare by Hour**

```python
# CELL 6: Average Fare by Hour
display(hourly_df.select("pickup_hour", "avg_fare", "avg_tip_percentage"))
```

**Visualization:** Combination chart
- Bar: `avg_fare`
- Line: `avg_tip_percentage`

### 6.3 Create Dashboard

**Action:**
1. In left sidebar, click **Dashboards**
2. Click **Create Dashboard**
3. Name it: `NYC Taxi Analytics Dashboard`
4. Click **Create**

**Add Widgets:**
1. Click **Add Widget**
2. Select visualization from your notebooks
3. Arrange in layout

**Suggested Layout:**
```
┌──────────────────────────────────────────┐
│  EXECUTIVE KPIS (Text Widgets)           │
│  Total Trips | Revenue | Avg Fare | Tips │
└──────────────────────────────────────────┘

┌─────────────────┬────────────────────────┐
│ Trips by Hour   │ Revenue by Hour        │
│ (Bar Chart)     │ (Bar Chart)            │
└─────────────────┴────────────────────────┘

┌──────────────────────────────────────────┐
│ Daily Trends (Line Chart)                │
└──────────────────────────────────────────┘

┌─────────────────┬────────────────────────┐
│ Top Locations   │ High-Tip Patterns      │
│ (Bar Chart)     │ (Grouped Bar)          │
└─────────────────┴────────────────────────┘
```

---

## STEP 7: Build Machine Learning Model (30 minutes)

### 7.1 Create ML Notebook

**Action:**
1. Navigate to `NYC_Taxi_Project` → `04_ML_Models`
2. Create notebook: `Tip_Prediction_Model`
3. Language: **Python**

### 7.2 Add ML Code

**Cell 1: Import Libraries**

```python
# CELL 1: Import Libraries
import mlflow
import mlflow.sklearn
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score
import pandas as pd

print("✓ Libraries imported")
```

**Cell 2: Load and Sample Data**

```python
# CELL 2: Load Data
catalog_name = "your_workspace_catalog"  # <-- UPDATE THIS
silver_table = f"{catalog_name}.default.taxi_trips_silver"

# Load silver data
silver_df = spark.read.table(silver_table)

# Sample for faster training (Free Edition has limited compute)
sample_df = silver_df.sample(withReplacement=False, fraction=0.1, seed=42)

# Convert to Pandas
pdf = sample_df.toPandas()

print(f"Sample size: {len(pdf):,} records")
```

**Cell 3: Prepare Features**

```python
# CELL 3: Feature Engineering
# Select features for model
feature_columns = [
    "trip_distance",
    "fare_amount",
    "pickup_hour",
    "pickup_day_of_week",
    "pickup_month",
    "trip_duration_minutes",
    "avg_speed_mph"
]

# Target variable
target_column = "is_high_tip"

# Filter valid data
ml_data = pdf[feature_columns + [target_column]].dropna()

print(f"Records for ML: {len(ml_data):,}")

# Prepare features and target
X = ml_data[feature_columns]
y = ml_data[target_column]

print(f"\nFeatures: {X.shape}")
print(f"Target distribution:")
print(y.value_counts(normalize=True))
```

**Cell 4: Split Data**

```python
# CELL 4: Split Data
X_train, X_test, y_train, y_test = train_test_split(
    X, y, 
    test_size=0.2, 
    random_state=42,
    stratify=y
)

print(f"Training set: {X_train.shape[0]:,} records")
print(f"Test set: {X_test.shape[0]:,} records")
```

**Cell 5: Train Model with MLflow**

```python
# CELL 5: Train Model with MLflow Tracking
# Set experiment
mlflow.set_experiment("/demo/tip_prediction")

# Start MLflow run
with mlflow.start_run(run_name="RandomForest_Tip_Prediction"):
    
    # Model parameters
    params = {
        "n_estimators": 100,
        "max_depth": 10,
        "min_samples_split": 5,
        "random_state": 42
    }
    
    # Log parameters
    mlflow.log_params(params)
    
    # Train model
    print("Training Random Forest Classifier...")
    rf_model = RandomForestClassifier(**params)
    rf_model.fit(X_train, y_train)
    print("✓ Model trained")
    
    # Predictions
    y_pred = rf_model.predict(X_test)
    
    # Calculate metrics
    accuracy = accuracy_score(y_test, y_pred)
    precision = precision_score(y_test, y_pred)
    recall = recall_score(y_test, y_pred)
    f1 = f1_score(y_test, y_pred)
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("precision", precision)
    mlflow.log_metric("recall", recall)
    mlflow.log_metric("f1_score", f1)
    
    # Log model
    mlflow.sklearn.log_model(rf_model, "random_forest_model")
    
    # Print results
    print("\n=== MODEL PERFORMANCE ===")
    print(f"Accuracy:  {accuracy:.4f}")
    print(f"Precision: {precision:.4f}")
    print(f"Recall:    {recall:.4f}")
    print(f"F1 Score:  {f1:.4f}")
    
    # Feature importance
    feature_importance = pd.DataFrame({
        'feature': feature_columns,
        'importance': rf_model.feature_importances_
    }).sort_values('importance', ascending=False)
    
    print("\n=== FEATURE IMPORTANCE ===")
    print(feature_importance)
    
    run_id = mlflow.active_run().info.run_id
    print(f"\n✓ MLflow Run ID: {run_id}")
```

**Cell 6: Register Model**

```python
# CELL 6: Register Model
# Register in Model Registry
model_name = "Taxi_Tip_Prediction"
model_uri = f"runs:/{run_id}/random_forest_model"

# Register
model_version = mlflow.register_model(model_uri, model_name)

print(f"✓ Model registered")
print(f"  Name: {model_name}")
print(f"  Version: {model_version.version}")
```

**Cell 7: Test Prediction Function**

```python
# CELL 7: Create Prediction Function
import numpy as np
from sklearn.preprocessing import StandardScaler

# Load model
loaded_model = mlflow.sklearn.load_model(f"models:/{model_name}/{model_version.version}")

# Create prediction function
def predict_high_tip(trip_distance, fare_amount, pickup_hour,
                     pickup_day_of_week, pickup_month,
                     trip_duration_minutes, avg_speed_mph):
    """
    Predict if a trip will result in a high tip (>20%)
    """
    # Create feature array
    features = np.array([[
        trip_distance, fare_amount, pickup_hour,
        pickup_day_of_week, pickup_month,
        trip_duration_minutes, avg_speed_mph
    ]])
    
    # Predict
    prediction = loaded_model.predict(features)[0]
    probability = loaded_model.predict_proba(features)[0][1]
    
    return prediction, probability

# Test
pred, prob = predict_high_tip(
    trip_distance=5.2,
    fare_amount=25.50,
    pickup_hour=14,
    pickup_day_of_week=5,
    pickup_month=6,
    trip_duration_minutes=18.5,
    avg_speed_mph=16.9
)

print("=== TEST PREDICTION ===")
print(f"Prediction: {'High Tip' if pred == 1 else 'Normal Tip'}")
print(f"Probability: {prob:.2%}")
```

### 7.3 View MLflow UI

**Action:**
1. Click **Experiments** in left sidebar
2. Find your experiment: `/demo/tip_prediction`
3. Click to see run details
4. View:
   - Parameters
   - Metrics
   - Model artifacts
   - Feature importance

---

## STEP 8: Create Scheduled Job (15 minutes)

### 8.1 Create Job

**Action:**
1. Click **Jobs** in left sidebar
2. Click **Create Job**
3. Configure:

**Job Name:** `NYC Taxi Daily Pipeline`

**Task 1: Bronze Layer**
- Type: Notebook
- Path: `/NYC_Taxi_Project/01_Bronze/Bronze_Raw_Taxi_Data`
- Compute: Serverless

**Task 2: Silver Layer** (depends on Task 1)
- Type: Notebook
- Path: `/NYC_Taxi_Project/02_Silver/Silver_Cleansed_Taxi_Data`
- Compute: Serverless
- Dependencies: Task 1

**Task 3: Gold Layer** (depends on Task 2)
- Type: Notebook
- Path: `/NYC_Taxi_Project/03_Gold/Gold_Business_Metrics`
- Compute: Serverless
- Dependencies: Task 2

### 8.2 Configure Schedule

**Action:**
1. Click **Add Schedule**
2. Configure:
   - Frequency: Daily
   - Time: 2:00 AM UTC
   - Timezone: Select your timezone

### 8.3 Configure Alerts

**Action:**
1. Add email for notifications
2. Configure retry policy:
   - Retries: 3
   - Retry interval: 5 minutes

### 8.4 Run the Job

**Action:**
1. Click **Run Now**
2. Monitor in **Run History** tab
3. Check each task's logs
4. Verify successful completion

---

## STEP 9: Project Summary

### 9.1 What You Built

```
┌─────────────────────────────────────────────────┐
│            COMPLETE DATA PLATFORM                │
├─────────────────────────────────────────────────┤
│                                                  │
│  DATA LAYERS                                     │
│  ├── Bronze: taxi_trips_bronze                  │
│  ├── Silver: taxi_trips_silver                  │
│  └── Gold: 5 aggregated tables                  │
│                                                  │
│  ANALYTICS                                       │
│  ├── Interactive Dashboards                     │
│  └── 6 Visualizations                           │
│                                                  │
│  MACHINE LEARNING                                │
│  ├── MLflow Experiment                          │
│  ├── Registered Model                           │
│  └── Prediction Function                        │
│                                                  │
│  AUTOMATION                                      │
│  └── Scheduled Daily Job                        │
│                                                  │
└─────────────────────────────────────────────────┘
```

### 9.2 Tables Created

| Layer | Table | Records | Purpose |
|-------|-------|---------|---------|
| Bronze | `taxi_trips_bronze` | ~2.8M | Raw data with metadata |
| Silver | `taxi_trips_silver` | ~2.7M | Cleansed + enriched |
| Gold | `gold_hourly_metrics` | 24 | Hourly aggregations |
| Gold | `gold_location_metrics` | ~200 | Location aggregations |
| Gold | `gold_daily_trends` | ~30 | Daily time series |
| Gold | `gold_high_tip_patterns` | 48 | High-tip analysis |
| Gold | `gold_executive_kpis` | 6 | Executive metrics |

### 9.3 Skills Learned

- ✅ Navigate Databricks workspace
- ✅ Create and use notebooks
- ✅ Work with Unity Catalog
- ✅ Build Delta Lake tables
- ✅ Implement Medallion Architecture
- ✅ Create data visualizations
- ✅ Build dashboards
- ✅ Use MLflow for experiment tracking
- ✅ Register ML models
- ✅ Create scheduled jobs

---

## Troubleshooting

### Issue 1: "Catalog not found"

**Solution:**
```python
# Check available catalogs
spark.sql("SHOW CATALOGS").display()

# Use the correct catalog name
# Check Catalog browser for exact name
```

### Issue 2: "Permission denied"

**Solution:**
```
Free Edition gives you full permissions on your workspace catalog.
Make sure you're using YOUR catalog, not someone else's.
```

### Issue 3: "Out of quota"

**Solution:**
```
Free Edition has compute quota limits.
Wait if quota is exhausted.
Try smaller data samples.
```

### Issue 4: "Cannot upload file"

**Solution:**
```
Free Edition blocks outbound internet.
Upload files via UI:
1. Click "New" → "Add or upload data"
2. Upload to volume
```

---

## Next Steps

### Continue Learning

1. **Explore More Sample Data**
   - `samples.tpch` - TPC-H benchmark data
   - `samples.datasets` - Various datasets

2. **Try Advanced Features**
   - Delta Live Tables (DLT)
   - Structured Streaming
   - Feature Store

3. **Build Your Own Project**
   - Use your own data
   - Create custom schemas
   - Build production pipelines

### Resources

- [Databricks Documentation](https://learn.microsoft.com/azure/databricks/)
- [Free Edition Limitations](https://learn.microsoft.com/azure/databricks/getting-started/free-edition)
- [Community Forum](https://community.databricks.com/)

---

**Congratulations! 🎉**

You've successfully built a complete data analytics platform on Databricks Free Edition including:
- Bronze-Silver-Gold architecture
- Interactive dashboards
- ML model with MLflow
- Scheduled production job

**Project Completion Time:** ~2.5 hours

**Document Version:** 2.0  
**Last Updated:** November 2026
