# Azure Databricks Demo Project: End-to-End Data Pipeline

## Project Overview

**Project Name**: NYC Taxi Data Analytics Platform

**Objective**: Build a complete data analytics solution that ingests NYC taxi trip data, processes it through a medallion architecture (Bronze → Silver → Gold), creates visualizations, and builds a machine learning model to predict high-tip trips.

**Business Scenario**: A taxi company wants to analyze trip patterns, optimize routes, and predict which trips are likely to result in higher tips to improve driver satisfaction and revenue.

**Duration**: 2-3 hours

**Difficulty Level**: Beginner to Intermediate

---

## Prerequisites

### What You Need

- [ ] Azure Databricks workspace (Free Edition or Trial)
- [ ] Basic understanding of Python or SQL
- [ ] Web browser (Chrome, Firefox, Edge)

### Dataset Information

We'll use the **NYC Taxi Trip Data** sample dataset available in Databricks:
- **Table**: `samples.nyctaxi.trips`
- **Records**: ~2.8 million trips
- **Columns**: Trip details (pickup/dropoff times, locations, fare, distance, etc.)

---

## Project Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    DATA INGESTION LAYER                      │
│                                                              │
│  Source: samples.nyctaxi.trips (Sample Dataset)             │
│  Method: Direct query from Unity Catalog                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    BRONZE LAYER (Raw Data)                   │
│                                                              │
│  • Copy raw trip data                                        │
│  • Add ingestion timestamp                                   │
│  • Preserve original schema                                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    SILVER LAYER (Cleansed Data)              │
│                                                              │
│  • Data quality checks                                       │
│  • Remove invalid trips (negative fare, zero distance)       │
│  • Standardize date formats                                  │
│  • Add derived columns (trip duration, speed)                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    GOLD LAYER (Business-Ready)               │
│                                                              │
│  • Aggregated metrics by location, time                      │
│  • KPI tables for dashboards                                 │
│  • Feature tables for ML models                              │
└─────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
        ┌──────────────────┐  ┌──────────────────┐
        │   DASHBOARDS &   │  │  ML MODEL        │
        │   VISUALIZATIONS │  │  (Tip Prediction)│
        └──────────────────┘  └──────────────────┘
```

---

## Step-by-Step Implementation

---

## STEP 1: Environment Setup

### 1.1 Create Project Folder Structure

**Time**: 2 minutes

1. Open your Databricks workspace
2. In the workspace browser (left sidebar), right-click on your username
3. Select **Create** → **Folder**
4. Name it: `NYC_Taxi_Project`
5. Inside this folder, create subfolders:
   - `01_Bronze`
   - `02_Silver`
   - `03_Gold`
   - `04_Dashboards`
   - `05_ML_Models`

### 1.2 Verify Sample Data Access

1. Click **Catalog** in the left sidebar
2. Navigate to: `samples` → `nyctaxi` → `trips`
3. Verify you can see the table schema and sample data

**Expected Output**:
```
Table: samples.nyctaxi.trips
Columns:
- tpep_pickup_datetime (timestamp)
- tpep_dropoff_datetime (timestamp)
- trip_distance (double)
- fare_amount (double)
- pickup_zip (int)
- dropoff_zip (int)
- ... (additional columns)
```

---

## STEP 2: Bronze Layer - Raw Data Ingestion

### 2.1 Create Bronze Notebook

**Time**: 10 minutes

1. Right-click on `01_Bronze` folder
2. Select **Create** → **Notebook**
3. Name it: `Bronze_Raw_Trip_Data`
4. Default language: **Python**

### 2.2 Copy Raw Data to Bronze Layer

Copy and paste the following code into your notebook:

```python
# CELL 1: Configuration
# Purpose: Set up variables for the project

# Define catalog and schema names
# Replace with your actual catalog/schema if different
catalog_name = "your_workspace_catalog"  # Check Catalog browser
schema_name = "default"

# Bronze table name
bronze_table = "taxi_trips_bronze"

# Full table path
bronze_full_path = f"{catalog_name}.{schema_name}.{bronze_table}"

print(f"Bronze table path: {bronze_full_path}")
```

**Note**: To find your catalog name:
1. Click **Catalog** in sidebar
2. Look for catalog named after your workspace
3. Copy that exact name

```python
# CELL 2: Load Source Data
# Purpose: Read sample NYC taxi data

# Read from Unity Catalog sample dataset
raw_trips_df = spark.read.table("samples.nyctaxi.trips")

# Display first 10 rows
print(f"Total records in source: {raw_trips_df.count():,}")
display(raw_trips_df.limit(10))
```

**Expected Output**: A table showing first 10 rows of taxi trip data

```python
# CELL 3: Add Ingestion Metadata
# Purpose: Add tracking columns for data lineage

from pyspark.sql.functions import current_timestamp, lit

# Add ingestion timestamp and source identifier
bronze_trips_df = raw_trips_df \
    .withColumn("ingestion_timestamp", current_timestamp()) \
    .withColumn("source_system", lit("samples.nyctaxi.trips")) \
    .withColumn("ingestion_date", current_timestamp().cast("date"))

print("Schema after adding metadata:")
bronze_trips_df.printSchema()
```

**Expected Output**: Schema with 3 new columns added

```python
# CELL 4: Write to Bronze Table
# Purpose: Save raw data to Bronze layer

# Write as Delta table with overwrite mode
bronze_trips_df.write \
    .format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable(bronze_full_path)

print(f"✓ Successfully wrote {bronze_trips_df.count():,} records to {bronze_full_path}")

# Verify the write
record_count = spark.read.table(bronze_full_path).count()
print(f"✓ Verification: {record_count:,} records in Bronze table")
```

**Expected Output**:
```
✓ Successfully wrote 2,XXX,XXX records to your_catalog.default.taxi_trips_bronze
✓ Verification: 2,XXX,XXX records in Bronze table
```

```python
# CELL 5: Preview Bronze Data
# Purpose: Verify data quality

# Read back and display
bronze_data = spark.read.table(bronze_full_path)

# Show summary statistics
print("=== Bronze Layer Summary ===")
print(f"Total Records: {bronze_data.count():,}")
print(f"Total Columns: {len(bronze_data.columns)}")
print(f"Data Size: ~{bronze_data.count() * len(bronze_data.columns) * 8 / 1024 / 1024:.2f} MB (estimated)")

display(bronze_data.limit(5))
```

### ✅ Checkpoint 1: Bronze Layer Complete

**What you've accomplished**:
- ✅ Created Bronze layer notebook
- ✅ Loaded raw taxi trip data
- ✅ Added ingestion metadata
- ✅ Saved to Delta table
- ✅ Verified data integrity

**Next**: Transform and clean data in Silver layer

---

## STEP 3: Silver Layer - Data Cleansing & Transformation

### 3.1 Create Silver Notebook

**Time**: 15 minutes

1. Right-click on `02_Silver` folder
2. Select **Create** → **Notebook**
3. Name it: `Silver_Cleansed_Trip_Data`
4. Default language: **Python**

### 3.2 Data Quality Checks and Cleansing

```python
# CELL 1: Configuration
# Purpose: Define table paths

catalog_name = "your_workspace_catalog"
schema_name = "default"

# Table names
bronze_table = f"{catalog_name}.{schema_name}.taxi_trips_bronze"
silver_table = f"{catalog_name}.{schema_name}.taxi_trips_silver"

print(f"Source (Bronze): {bronze_table}")
print(f"Target (Silver): {silver_table}")
```

```python
# CELL 2: Load Bronze Data
# Purpose: Read from Bronze layer

bronze_df = spark.read.table(bronze_table)

print(f"Records loaded from Bronze: {bronze_df.count():,}")
display(bronze_df.limit(5))
```

```python
# CELL 3: Data Quality Analysis
# Purpose: Identify data quality issues before cleansing

from pyspark.sql.functions import col, count, when, isnull

# Check for null values
print("=== NULL VALUE ANALYSIS ===")
null_counts = bronze_df.select([
    count(when(isnull(c), c)).alias(c) 
    for c in bronze_df.columns
])

display(null_counts)

# Check for invalid values
print("\n=== INVALID VALUE ANALYSIS ===")
invalid_stats = {
    "negative_fare": bronze_df.filter(col("fare_amount") < 0).count(),
    "zero_distance": bronze_df.filter(col("trip_distance") == 0).count(),
    "negative_distance": bronze_df.filter(col("trip_distance") < 0).count(),
    "null_pickup_time": bronze_df.filter(isnull("tpep_pickup_datetime")).count(),
    "null_dropoff_time": bronze_df.filter(isnull("tpep_dropoff_datetime")).count()
}

for issue, count in invalid_stats.items():
    print(f"{issue}: {count:,} records")
```

**Expected Output**:
```
=== NULL VALUE ANALYSIS ===
[Table showing null counts per column]

=== INVALID VALUE ANALYSIS ===
negative_fare: XXX records
zero_distance: XXX records
negative_distance: X records
null_pickup_time: X records
null_dropoff_time: X records
```

```python
# CELL 4: Apply Data Quality Rules
# Purpose: Remove invalid records

from pyspark.sql.functions import col, when, unix_timestamp, round

# Start with Bronze data
silver_df = bronze_df

# Apply cleansing rules
print("=== APPLYING DATA QUALITY RULES ===")

# Rule 1: Remove trips with negative fare
silver_df = silver_df.filter(col("fare_amount") >= 0)
print("✓ Rule 1: Removed negative fare trips")

# Rule 2: Remove trips with zero or negative distance
silver_df = silver_df.filter(col("trip_distance") > 0)
print("✓ Rule 2: Removed zero/negative distance trips")

# Rule 3: Remove trips with null pickup/dropoff times
silver_df = silver_df.filter(col("tpep_pickup_datetime").isNotNull())
silver_df = silver_df.filter(col("tpep_dropoff_datetime").isNotNull())
print("✓ Rule 3: Removed trips with null timestamps")

# Rule 4: Remove trips where dropoff is before pickup
silver_df = silver_df.filter(
    col("tpep_dropoff_datetime") > col("tpep_pickup_datetime")
)
print("✓ Rule 4: Removed trips with invalid time sequences")

print(f"\nRecords after cleansing: {silver_df.count():,}")
```

```python
# CELL 5: Add Derived Columns
# Purpose: Calculate business metrics

from pyspark.sql.functions import (
    col, unix_timestamp, round, hour, dayofweek, month, year
)

# Calculate trip duration in minutes
silver_df = silver_df.withColumn(
    "trip_duration_minutes",
    round(
        (unix_timestamp("tpep_dropoff_datetime") - unix_timestamp("tpep_pickup_datetime")) / 60,
        2
    )
)

# Calculate average speed (mph)
silver_df = silver_df.withColumn(
    "avg_speed_mph",
    round(
        when(col("trip_duration_minutes") > 0,
            col("trip_distance") / (col("trip_duration_minutes") / 60)
        ).otherwise(0),
        2
    )
)

# Extract time dimensions
silver_df = silver_df \
    .withColumn("pickup_hour", hour("tpep_pickup_datetime")) \
    .withColumn("pickup_day_of_week", dayofweek("tpep_pickup_datetime")) \
    .withColumn("pickup_month", month("tpep_pickup_datetime")) \
    .withColumn("pickup_year", year("tpep_pickup_datetime"))

# Calculate tip percentage
silver_df = silver_df.withColumn(
    "tip_percentage",
    round(
        when(col("fare_amount") > 0,
            (col("tip_amount") / col("fare_amount")) * 100
        ).otherwise(0),
        2
    )
)

# Create high-tip flag (tip > 20% of fare)
silver_df = silver_df.withColumn(
    "is_high_tip",
    when(col("tip_percentage") > 20, 1).otherwise(0)
)

print("✓ Derived columns added:")
print("  - trip_duration_minutes")
print("  - avg_speed_mph")
print("  - pickup_hour")
print("  - pickup_day_of_week")
print("  - pickup_month")
print("  - pickup_year")
print("  - tip_percentage")
print("  - is_high_tip")

display(silver_df.limit(5))
```

```python
# CELL 6: Data Quality Summary
# Purpose: Generate cleansing report

print("=== SILVER LAYER QUALITY SUMMARY ===\n")

total_records = silver_df.count()
print(f"Total Cleansed Records: {total_records:,}")
print(f"Total Columns: {len(silver_df.columns)}")

print("\n=== KEY METRICS STATISTICS ===")
silver_df.select(
    "trip_distance",
    "fare_amount",
    "tip_amount",
    "trip_duration_minutes",
    "avg_speed_mph"
).describe().display()

print("\n=== HIGH TIP ANALYSIS ===")
high_tip_stats = silver_df.groupBy("is_high_tip").count()
high_tip_stats.display()
```

```python
# CELL 7: Write to Silver Table
# Purpose: Save cleansed data

# Write as Delta table
silver_df.write \
    .format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable(silver_table)

print(f"✓ Successfully wrote {silver_df.count():,} records to {silver_table}")

# Verify
record_count = spark.read.table(silver_table).count()
print(f"✓ Verification: {record_count:,} records in Silver table")
```

### ✅ Checkpoint 2: Silver Layer Complete

**What you've accomplished**:
- ✅ Loaded Bronze data
- ✅ Analyzed data quality issues
- ✅ Applied 4 cleansing rules
- ✅ Added 8 derived columns
- ✅ Saved cleansed data to Silver layer

**Data Quality Improvements**:
- Removed invalid fare amounts
- Removed invalid distances
- Removed invalid timestamps
- Added business metrics

---

## STEP 4: Gold Layer - Business Aggregations

### 4.1 Create Gold Notebook

**Time**: 20 minutes

1. Right-click on `03_Gold` folder
2. Select **Create** → **Notebook**
3. Name it: `Gold_Business_Metrics`
4. Default language: **Python**

### 4.2 Create Aggregated Tables

```python
# CELL 1: Configuration
catalog_name = "your_workspace_catalog"
schema_name = "default"

silver_table = f"{catalog_name}.{schema_name}.taxi_trips_silver"

print(f"Source (Silver): {silver_table}")
```

```python
# CELL 2: Load Silver Data
silver_df = spark.read.table(silver_table)

print(f"Records loaded from Silver: {silver_df.count():,}")
```

```python
# CELL 3: Create Hourly Metrics Table
# Purpose: Aggregate metrics by hour of day

from pyspark.sql.functions import col, count, sum, avg, max, min

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

# Write to Gold table
hourly_metrics.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_hourly_metrics")

print("✓ Created gold_hourly_metrics table")
print(f"Records: {hourly_metrics.count()}")
display(hourly_metrics)
```

**Expected Output**: A table with 24 rows (0-23 hours) showing trip patterns by hour

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

# Write to Gold table
location_metrics.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_location_metrics")

print("✓ Created gold_location_metrics table")
print(f"Records: {location_metrics.count()}")
display(location_metrics.limit(10))
```

```python
# CELL 5: Create Daily Trend Table
# Purpose: Track daily performance over time

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

# Write to Gold table
daily_metrics.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_daily_trends")

print("✓ Created gold_daily_trends table")
print(f"Records: {daily_metrics.count()}")
display(daily_metrics.limit(10))
```

```python
# CELL 6: Create High-Tip Analysis Table
# Purpose: Identify patterns in high-tip trips

high_tip_analysis = silver_df.groupBy("pickup_hour", "is_high_tip") \
    .agg(
        count("*").alias("trip_count"),
        avg("trip_distance").alias("avg_distance"),
        avg("fare_amount").alias("avg_fare"),
        avg("tip_amount").alias("avg_tip"),
        avg("tip_percentage").alias("avg_tip_percentage")
    ) \
    .orderBy("pickup_hour", "is_high_tip")

# Write to Gold table
high_tip_analysis.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_high_tip_patterns")

print("✓ Created gold_high_tip_patterns table")
display(high_tip_analysis)
```

```python
# CELL 7: Create Summary KPIs
# Purpose: Executive-level metrics

from pyspark.sql.functions import lit

# Calculate overall KPIs
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

# Write to Gold table
kpis.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable(f"{catalog_name}.{schema_name}.gold_executive_kpis")

print("=== EXECUTIVE KPIS ===")
display(kpis)
```

### ✅ Checkpoint 3: Gold Layer Complete

**What you've accomplished**:
- ✅ Created 5 aggregated tables
- ✅ Hourly metrics (24 rows)
- ✅ Location metrics (by zip code)
- ✅ Daily trends (time series)
- ✅ High-tip pattern analysis
- ✅ Executive KPIs summary

---

## STEP 5: Create Visualizations and Dashboard

### 5.1 Create Visualization Notebook

**Time**: 15 minutes

1. Right-click on `04_Dashboards` folder
2. Create notebook: `Dashboard_Visualizations`

```python
# CELL 1: Setup
catalog_name = "your_workspace_catalog"
schema_name = "default"

# Load Gold tables
hourly_df = spark.read.table(f"{catalog_name}.{schema_name}.gold_hourly_metrics")
location_df = spark.read.table(f"{catalog_name}.{schema_name}.gold_location_metrics")
daily_df = spark.read.table(f"{catalog_name}.{schema_name}.gold_daily_trends")
```

```python
# CELL 2: Trips by Hour Visualization
# Create bar chart
display(hourly_df.select("pickup_hour", "total_trips"))
```

**Instructions for visualization**:
1. Run the cell
2. Click **+** next to Table → **Visualization**
3. Select **Bar** chart
4. X Column: `pickup_hour`
5. Y Column: `total_trips`
6. Aggregation: None (already aggregated)
7. Click **Save**

```python
# CELL 3: Revenue by Hour
display(hourly_df.select("pickup_hour", "total_revenue"))
```

**Visualization**: Bar chart with X=`pickup_hour`, Y=`total_revenue`

```python
# CELL 4: Top 10 Locations by Trips
display(location_df.select("pickup_zip", "total_trips").limit(10))
```

**Visualization**: Bar chart with X=`pickup_zip`, Y=`total_trips`

```python
# CELL 5: Daily Trend Line Chart
display(daily_df.select("trip_date", "total_trips", "total_revenue"))
```

**Visualization**: Line chart with:
- X: `trip_date`
- Y: `total_trips`
- Add second series: `total_revenue`

```python
# CELL 6: Average Fare by Hour
display(hourly_df.select("pickup_hour", "avg_fare", "avg_tip_percentage"))
```

**Visualization**: Combination chart (Bar + Line)
- Bar: `avg_fare`
- Line: `avg_tip_percentage`

### 5.2 Create Dashboard

**Time**: 10 minutes

1. In the left sidebar, click **Dashboards**
2. Click **Create Dashboard**
3. Name it: `NYC Taxi Analytics Dashboard`
4. Click **Create**

5. **Add Widgets**:
   - Click **Add Widget**
   - Select your visualizations from the notebook
   - Arrange in a grid layout:
     - Row 1: Executive KPIs (large text widgets)
     - Row 2: Trips by Hour | Revenue by Hour
     - Row 3: Daily Trends (full width)
     - Row 4: Top Locations | High-Tip Patterns

6. **Add Text Widgets** for context:
   - Add titles and descriptions
   - Include data refresh date

7. **Save Dashboard**

**Expected Dashboard Layout**:
```
┌─────────────────────────────────────────────────────┐
│          NYC TAXI ANALYTICS DASHBOARD               │
│                                                     │
│  [KPI: Total Trips] [KPI: Revenue] [KPI: Tips]    │
└─────────────────────────────────────────────────────┘
┌───────────────────┬─────────────────────────────────┐
│  Trips by Hour    │     Revenue by Hour             │
│  [Bar Chart]      │     [Bar Chart]                 │
└───────────────────┴─────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│              Daily Trip Trends                       │
│              [Line Chart]                            │
└─────────────────────────────────────────────────────┘
┌───────────────────┬─────────────────────────────────┐
│  Top Locations    │    High-Tip Patterns            │
│  [Bar Chart]      │    [Grouped Bar]                │
└───────────────────┴─────────────────────────────────┘
```

---

## STEP 6: Build Machine Learning Model

### 6.1 Create ML Notebook

**Time**: 25 minutes

1. Right-click on `05_ML_Models` folder
2. Create notebook: `Tip_Prediction_Model`

```python
# CELL 1: Import Libraries
import mlflow
import mlflow.sklearn
from mlflow.models.signature import infer_signature
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, confusion_matrix
from sklearn.metrics import classification_report
import pandas as pd
import numpy as np

print("✓ Libraries imported successfully")
```

```python
# CELL 2: Load Silver Data
catalog_name = "your_workspace_catalog"
schema_name = "default"
silver_table = f"{catalog_name}.{schema_name}.taxi_trips_silver"

# Load and convert to Pandas for ML
silver_df = spark.read.table(silver_table)

# Sample data for faster training (adjust as needed)
sample_df = silver_df.sample(withReplacement=False, fraction=0.1, seed=42)

# Convert to Pandas
pdf = sample_df.toPandas()

print(f"Total records for ML: {len(pdf):,}")
print(f"Columns: {pdf.shape[1]}")
```

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

print(f"Records after removing nulls: {len(ml_data):,}")

# Prepare features and target
X = ml_data[feature_columns]
y = ml_data[target_column]

print(f"\nFeature shape: {X.shape}")
print(f"Target distribution:")
print(y.value_counts(normalize=True))
```

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
print(f"\nClass distribution in training:")
print(y_train.value_counts(normalize=True))
```

```python
# CELL 5: Scale Features
scaler = StandardScaler()

X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

print("✓ Features scaled successfully")
```

```python
# CELL 6: Train Model with MLflow Tracking
# Set experiment
mlflow.set_experiment("/Users/me/taxi_tip_prediction")

# Start MLflow run
with mlflow.start_run(run_name="RandomForest_Tip_Prediction"):
    
    # Model parameters
    params = {
        "n_estimators": 100,
        "max_depth": 10,
        "min_samples_split": 5,
        "min_samples_leaf": 2,
        "random_state": 42
    }
    
    # Train model
    print("Training Random Forest Classifier...")
    rf_model = RandomForestClassifier(**params)
    rf_model.fit(X_train_scaled, y_train)
    print("✓ Model trained successfully")
    
    # Make predictions
    y_pred = rf_model.predict(X_test_scaled)
    y_pred_proba = rf_model.predict_proba(X_test_scaled)[:, 1]
    
    # Calculate metrics
    accuracy = accuracy_score(y_test, y_pred)
    precision = precision_score(y_test, y_pred)
    recall = recall_score(y_test, y_pred)
    f1 = f1_score(y_test, y_pred)
    
    # Log parameters
    mlflow.log_params(params)
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("precision", precision)
    mlflow.log_metric("recall", recall)
    mlflow.log_metric("f1_score", f1)
    
    # Log model
    signature = infer_signature(X_train, y_pred)
    mlflow.sklearn.log_model(
        rf_model, 
        "random_forest_model",
        signature=signature,
        input_example=X_train[:5]
    )
    
    # Print results
    print("\n=== MODEL PERFORMANCE ===")
    print(f"Accuracy: {accuracy:.4f}")
    print(f"Precision: {precision:.4f}")
    print(f"Recall: {recall:.4f}")
    print(f"F1 Score: {f1:.4f}")
    
    # Feature importance
    feature_importance = pd.DataFrame({
        'feature': feature_columns,
        'importance': rf_model.feature_importances_
    }).sort_values('importance', ascending=False)
    
    print("\n=== FEATURE IMPORTANCE ===")
    print(feature_importance)
    
    # Log feature importance as artifact
    mlflow.log_dict(feature_importance.to_dict(), "feature_importance.json")
    
    run_id = mlflow.active_run().info.run_id
    print(f"\n✓ MLflow Run ID: {run_id}")
```

**Expected Output**:
```
Training Random Forest Classifier...
✓ Model trained successfully

=== MODEL PERFORMANCE ===
Accuracy: 0.7XXX
Precision: 0.7XXX
Recall: 0.6XXX
F1 Score: 0.6XXX

=== FEATURE IMPORTANCE ===
            feature  importance
1      fare_amount    0.XXX
0     trip_distance    0.XXX
...

✓ MLflow Run ID: abc123...
```

```python
# CELL 7: Confusion Matrix Visualization
import matplotlib.pyplot as plt
import seaborn as sns

# Create confusion matrix
cm = confusion_matrix(y_test, y_pred)

# Plot
plt.figure(figsize=(8, 6))
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')
plt.title('Confusion Matrix - Tip Prediction Model')
plt.ylabel('Actual')
plt.xlabel('Predicted')
plt.show()

# Classification report
print("\n=== CLASSIFICATION REPORT ===")
print(classification_report(y_test, y_pred, target_names=['Normal Tip', 'High Tip']))
```

```python
# CELL 8: Register Model in MLflow Model Registry
# Register the model
model_name = "Taxi_Tip_Prediction_Model"
model_uri = f"runs:/{run_id}/random_forest_model"

# Register
model_version = mlflow.register_model(model_uri, model_name)

print(f"✓ Model registered successfully")
print(f"Model Name: {model_name}")
print(f"Version: {model_version.version}")
```

```python
# CELL 9: Load and Test Registered Model
# Load model from registry
loaded_model = mlflow.sklearn.load_model(f"models:/{model_name}/{model_version.version}")

# Test prediction
sample_predictions = loaded_model.predict(X_test_scaled[:5])

print("=== SAMPLE PREDICTIONS ===")
for i in range(5):
    print(f"Actual: {y_test.iloc[i]}, Predicted: {sample_predictions[i]}")
```

```python
# CELL 10: Create Prediction Function
def predict_high_tip(trip_distance, fare_amount, pickup_hour, 
                     pickup_day_of_week, pickup_month,
                     trip_duration_minutes, avg_speed_mph):
    """
    Predict if a trip will result in a high tip (>= 20%)
    
    Returns: prediction (0 or 1) and probability
    """
    # Create feature array
    features = np.array([[
        trip_distance, fare_amount, pickup_hour,
        pickup_day_of_week, pickup_month,
        trip_duration_minutes, avg_speed_mph
    ]])
    
    # Scale
    features_scaled = scaler.transform(features)
    
    # Predict
    prediction = loaded_model.predict(features_scaled)[0]
    probability = loaded_model.predict_proba(features_scaled)[0][1]
    
    return prediction, probability

# Test the function
pred, prob = predict_high_tip(
    trip_distance=5.2,
    fare_amount=25.50,
    pickup_hour=14,
    pickup_day_of_week=5,
    pickup_month=6,
    trip_duration_minutes=18.5,
    avg_speed_mph=16.9
)

print(f"Prediction: {'High Tip' if pred == 1 else 'Normal Tip'}")
print(f"Probability: {prob:.2%}")
```

### ✅ Checkpoint 4: ML Model Complete

**What you've accomplished**:
- ✅ Prepared ML features
- ✅ Trained Random Forest classifier
- ✅ Tracked experiments with MLflow
- ✅ Evaluated model performance
- ✅ Registered model in Model Registry
- ✅ Created prediction function

---

## STEP 7: Schedule Production Pipeline

### 7.1 Create Job

**Time**: 10 minutes

1. In the left sidebar, click **Jobs**
2. Click **Create Job**
3. Configure the job:

**Job Settings**:
```
Job Name: NYC Taxi Daily Pipeline

Task 1: Bronze Layer
  - Type: Notebook
  - Path: /NYC_Taxi_Project/01_Bronze/Bronze_Raw_Trip_Data
  - Cluster: Create new cluster or use existing
  - Timeout: 30 minutes

Task 2: Silver Layer (depends on Task 1)
  - Type: Notebook
  - Path: /NYC_Taxi_Project/02_Silver/Silver_Cleansed_Trip_Data
  - Cluster: Same as Task 1
  - Timeout: 30 minutes

Task 3: Gold Layer (depends on Task 2)
  - Type: Notebook
  - Path: /NYC_Taxi_Project/03_Gold/Gold_Business_Metrics
  - Cluster: Same as Task 1
  - Timeout: 30 minutes
```

4. **Schedule**:
   - Click **Add Schedule**
   - Frequency: Daily
   - Time: 2:00 AM UTC
   - Timezone: Select your timezone

5. **Alerts**:
   - Add email for failure notifications
   - Configure retry policy (3 retries with 5-minute intervals)

6. Click **Create**

### 7.2 Run the Job

1. In the job details page, click **Run Now**
2. Monitor the run in the **Run History** tab
3. Check each task's logs for completion
4. Verify all tasks completed successfully

---

## STEP 8: Project Summary and Documentation

### 8.1 Create Project README

Create a notebook called `README_Project_Summary` in your project root:

```markdown
# NYC Taxi Data Analytics Platform

## Project Overview
End-to-end data pipeline analyzing NYC taxi trip data to optimize operations and predict high-tip trips.

## Architecture
- **Bronze Layer**: Raw data with ingestion metadata
- **Silver Layer**: Cleansed data with derived metrics
- **Gold Layer**: Business aggregations and KPIs

## Tables Created

### Bronze Layer
- `taxi_trips_bronze`: Raw trip data with metadata

### Silver Layer
- `taxi_trips_silver`: Cleansed and enriched trip data

### Gold Layer
- `gold_hourly_metrics`: Metrics by hour of day
- `gold_location_metrics`: Metrics by pickup location
- `gold_daily_trends`: Daily time series metrics
- `gold_high_tip_patterns`: High-tip analysis
- `gold_executive_kpis`: Executive summary KPIs

## ML Model
- **Model**: Random Forest Classifier
- **Purpose**: Predict high-tip trips
- **Features**: Distance, fare, time, duration, speed
- **Accuracy**: ~XX%
- **Registry**: Taxi_Tip_Prediction_Model

## Dashboards
- NYC Taxi Analytics Dashboard
  - Hourly patterns
  - Location analysis
  - Daily trends
  - Revenue metrics

## Scheduled Jobs
- **Job**: NYC Taxi Daily Pipeline
- **Schedule**: Daily at 2:00 AM UTC
- **Tasks**: Bronze → Silver → Gold

## Key Insights
1. Peak hours: XX:00 - XX:00
2. Top locations: ZIP codes XXXXX, XXXXX
3. Average tip percentage: XX%
4. High-tip rate: XX% of trips

## Next Steps
- Add real-time streaming
- Implement anomaly detection
- Build recommendation engine
- Integrate external data sources
```

### 8.2 Project Statistics

```python
# Run this in any notebook to see project stats
from pyspark.sql import SparkSession

# Count records in each layer
bronze_count = spark.read.table("your_catalog.default.taxi_trips_bronze").count()
silver_count = spark.read.table("your_catalog.default.taxi_trips_silver").count()

print("=== PROJECT STATISTICS ===")
print(f"Bronze Layer: {bronze_count:,} records")
print(f"Silver Layer: {silver_count:,} records")
print(f"Gold Tables: 5 tables created")
print(f"ML Model: 1 model registered")
print(f"Dashboards: 1 dashboard created")
```

---

## Troubleshooting Guide

### Common Issues and Solutions

**Issue 1: "Table not found" error**
```
Solution: 
- Verify catalog name in Catalog browser
- Ensure Unity Catalog is enabled
- Check permissions on catalog/schema
```

**Issue 2: "Out of memory" error**
```
Solution:
- Reduce sample size in ML training
- Use smaller fraction for sample()
- Increase cluster memory
```

**Issue 3: MLflow experiment not found**
```
Solution:
- Check experiment path
- Create experiment manually: mlflow.create_experiment("/path")
- Verify permissions
```

**Issue 4: Slow query performance**
```
Solution:
- Add ZORDER on frequently filtered columns
- Run OPTIMIZE on Delta tables
- Use caching: df.cache()
```

---

## Cost Management Tips

1. **Use Spot Instances**: Up to 60% cost savings for non-critical workloads
2. **Auto-termination**: Set clusters to terminate after 10-20 minutes of inactivity
3. **Job Clusters**: Use job clusters for scheduled tasks (cheaper than interactive)
4. **Serverless**: Use serverless for short SQL queries
5. **Right-size Clusters**: Start small, scale as needed

---

## Next Steps for Production

### 1. Add Data Quality Monitoring
```python
# Create data quality checks
from delta.tables import DeltaTable

# Add constraints
DeltaTable.forName(spark, "taxi_trips_silver") \
    .alter().addConstraint("fare_positive", "fare_amount >= 0")
```

### 2. Implement Incremental Processing
```python
# Use Auto Loader for incremental loads
df = spark.readStream.format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .load("/path/to/data")
```

### 3. Add Data Lineage
- Enable Unity Catalog lineage tracking
- Visualize data flow in Catalog Explorer

### 4. Set Up Alerts
- Configure job failure alerts
- Set up data quality alerts
- Monitor cost thresholds

---

## Appendix A: Complete Code Repository

All notebooks are available in your workspace:
```
NYC_Taxi_Project/
├── 01_Bronze/
│   └── Bronze_Raw_Trip_Data.py
├── 02_Silver/
│   └── Silver_Cleansed_Trip_Data.py
├── 03_Gold/
│   └── Gold_Business_Metrics.py
├── 04_Dashboards/
│   └── Dashboard_Visualizations.py
├── 05_ML_Models/
│   └── Tip_Prediction_Model.py
└── README_Project_Summary.md
```

## Appendix B: SQL Alternative

If you prefer SQL over Python, here's the SQL equivalent for key operations:

```sql
-- Bronze Layer
CREATE TABLE taxi_trips_bronze AS
SELECT 
    *,
    CURRENT_TIMESTAMP() as ingestion_timestamp,
    'samples.nyctaxi.trips' as source_system
FROM samples.nyctaxi.trips;

-- Silver Layer
CREATE TABLE taxi_trips_silver AS
SELECT 
    *,
    ROUND((UNIX_TIMESTAMP(tpep_dropoff_datetime) - UNIX_TIMESTAMP(tpep_pickup_datetime)) / 60, 2) as trip_duration_minutes,
    HOUR(tpep_pickup_datetime) as pickup_hour,
    DAYOFWEEK(tpep_pickup_datetime) as pickup_day_of_week
FROM taxi_trips_bronze
WHERE fare_amount >= 0
  AND trip_distance > 0
  AND tpep_dropoff_datetime > tpep_pickup_datetime;

-- Gold Layer
CREATE TABLE gold_hourly_metrics AS
SELECT 
    pickup_hour,
    COUNT(*) as total_trips,
    SUM(fare_amount) as total_revenue,
    AVG(fare_amount) as avg_fare
FROM taxi_trips_silver
GROUP BY pickup_hour
ORDER BY pickup_hour;
```

---

## Congratulations! 🎉

You've successfully completed a production-ready data analytics platform on Azure Databricks including:

✅ **Data Pipeline**: Bronze → Silver → Gold architecture  
✅ **Data Quality**: Cleansing rules and validation  
✅ **Business Intelligence**: Dashboards and visualizations  
✅ **Machine Learning**: Predictive model with MLflow  
✅ **Automation**: Scheduled jobs for production  
✅ **Documentation**: Complete project README

### Skills Mastered
- Delta Lake table management
- PySpark DataFrame operations
- Data quality implementation
- Aggregation and KPI creation
- MLflow experiment tracking
- Model registration and deployment
- Dashboard creation
- Job scheduling
- Unity Catalog usage

### Time to Build
- **Total**: 2-3 hours
- **Bronze**: 10 minutes
- **Silver**: 15 minutes
- **Gold**: 20 minutes
- **Dashboards**: 25 minutes
- **ML Model**: 25 minutes
- **Scheduling**: 10 minutes

---

**Project Completion Date**: ________________  
**Workspace URL**: ________________  
**Model Version**: ________________

For questions or issues, refer to the [Azure Databricks Documentation](https://learn.microsoft.com/en-us/azure/databricks/) or [Databricks Community](https://community.databricks.com/).
