# Azure Databricks Lab with Docker

## Complete Guide to Learning Databricks Concepts Using Docker

---

## Overview

This guide provides step-by-step instructions for setting up a local Databricks-like environment using Docker. Perfect for learning Spark, Delta Lake, and MLflow without cloud costs.

### What You'll Learn

| Topic | Databricks Feature | Docker Equivalent |
|-------|-------------------|-------------------|
| **Data Processing** | Apache Spark | Local Spark Cluster |
| **Storage** | Delta Lake | Delta Lake (open source) |
| **ML Tracking** | MLflow | MLflow Server |
| **Notebooks** | Databricks Notebooks | Jupyter Notebooks |
| **Storage** | DBFS | Local filesystem |
| **Catalog** | Unity Catalog | Local Hive Metastore |

### Advantages of Docker Lab

```
✓ No cloud costs
✓ Learn offline
✓ Full control
✓ Safe to experiment
✓ Easy to reset
✓ Fast iteration
```

---

## Prerequisites

### System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| **RAM** | 8 GB | 16 GB |
| **CPU Cores** | 4 | 8 |
| **Disk Space** | 20 GB | 50 GB |
| **OS** | Windows/Mac/Linux | Windows/Mac/Linux |

### Software Required

- **Docker Desktop**: [Download](https://www.docker.com/products/docker-desktop/)
- **Git**: [Download](https://git-scm.com/downloads)
- **VS Code** (optional): [Download](https://code.visualstudio.com/)

### Verify Docker Installation

```powershell
# Check Docker version
docker --version

# Check Docker Compose
docker-compose --version

# Test Docker
docker run hello-world
```

---

## Option 1: Simple Spark + Jupyter Setup

### Step 1: Create Project Structure

```powershell
# Create project directory
mkdir C:\databricks-docker
cd C:\databricks-docker

# Create subdirectories
mkdir notebooks
mkdir data
mkdir spark-warehouse
mkdir mlruns
```

### Step 2: Create docker-compose.yml

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  spark:
    image: jupyter/pyspark-notebook:latest
    container_name: pyspark-lab
    ports:
      - "8888:8888"   # Jupyter
      - "4040:4040"   # Spark UI
      - "4041:4041"   # Spark UI (secondary)
    environment:
      - JUPYTER_ENABLE_LAB=yes
    volumes:
      - ./notebooks:/home/jovyan/work
      - ./data:/data
      - ./spark-warehouse:/spark-warehouse
      - ./mlruns:/mlruns
    command: start-notebook.sh --NotebookApp.token=''
```

### Step 3: Start the Container

```powershell
# Start container
docker-compose up -d

# Check logs
docker-compose logs

# Check running containers
docker ps
```

### Step 4: Access Jupyter

```
Open browser: http://localhost:8888

No password required (token disabled for lab)
```

### Step 5: Test Spark Installation

Create a new notebook and run:

```python
# Test Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("TestSpark") \
    .getOrCreate()

# Create test data
data = [(1, "Alice"), (2, "Bob"), (3, "Charlie")]
df = spark.createDataFrame(data, ["id", "name"])

df.show()

# Expected output:
# +---+-------+
# | id|   name|
# +---+-------+
# |  1|  Alice|
# |  2|    Bob|
# |  3|Charlie|
# +---+-------+
```

---

## Option 2: Complete Stack (Spark + Delta Lake + MLflow)

### Step 1: Create Dockerfile

Create `Dockerfile`:

```dockerfile
FROM jupyter/pyspark-notebook:spark-3.5.0

USER root

# Install Java (required for Spark)
RUN apt-get update && \
    apt-get install -y openjdk-11-jre-headless && \
    apt-get clean

# Install Python packages
RUN pip install --no-cache-dir \
    delta-spark==2.4.0 \
    mlflow==2.9.2 \
    pandas==2.1.3 \
    numpy==1.26.2 \
    scikit-learn==1.3.2 \
    matplotlib==3.8.2 \
    seaborn==0.13.0 \
    plotly==5.18.0 \
    pyarrow==14.0.1

USER $NB_USER

WORKDIR /home/jovyan/work

EXPOSE 8888 4040 5000

CMD ["start-notebook.sh", "--NotebookApp.token=''"]
```

### Step 2: Create docker-compose.yml

```yaml
version: '3.8'

services:
  jupyter:
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: databricks-lab
    ports:
      - "8888:8888"   # Jupyter
      - "4040:4040"   # Spark UI
      - "5000:5000"   # MLflow
    environment:
      - JUPYTER_ENABLE_LAB=yes
      - SPARK_MASTER=local[*]
    volumes:
      - ./notebooks:/home/jovyan/work
      - ./data:/data
      - ./spark-warehouse:/spark-warehouse
      - ./mlruns:/mlruns
    networks:
      - databricks-net

  minio:
    image: minio/minio:latest
    container_name: minio
    ports:
      - "9000:9000"   # MinIO API
      - "9001:9001"   # MinIO Console
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
    command: server /data --console-address ":9001"
    volumes:
      - minio-data:/data
    networks:
      - databricks-net

  postgres:
    image: postgres:13
    container_name: postgres
    environment:
      - POSTGRES_USER=mlflow
      - POSTGRES_PASSWORD=mlflow
      - POSTGRES_DB=mlflow
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - databricks-net

  mlflow:
    image: ghcr.io/mlflow/mlflow:v2.9.2
    container_name: mlflow
    ports:
      - "5001:5000"
    environment:
      - MLFLOW_BACKEND_STORE_URI=postgresql://mlflow:mlflow@postgres:5432/mlflow
      - MLFLOW_ARTIFACT_ROOT=/mlflow/artifacts
    command: mlflow server --host 0.0.0.0 --port 5000
    volumes:
      - mlflow-artifacts:/mlflow/artifacts
    depends_on:
      - postgres
      - minio
    networks:
      - databricks-net

networks:
  databricks-net:
    driver: bridge

volumes:
  minio-data:
  postgres-data:
  mlflow-artifacts:
```

### Step 3: Build and Start

```powershell
# Build custom image
docker-compose build

# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f
```

### Step 4: Access Services

```
Jupyter Lab:    http://localhost:8888
Spark UI:       http://localhost:4040 (when Spark job running)
MLflow UI:      http://localhost:5001
MinIO Console:  http://localhost:9001
  Username: minioadmin
  Password: minioadmin
```

### Step 5: Stop Services

```powershell
# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

---

## Hands-On Labs

### Lab 1: Spark Basics

**Create**: `notebooks/01_spark_basics.ipynb`

```python
# CELL 1: Initialize Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("SparkBasics") \
    .master("local[*]") \
    .getOrCreate()

print(f"Spark Version: {spark.version}")
print(f"Spark Master: {spark.conf.get('spark.master')}")
```

```python
# CELL 2: Create DataFrame from Data
data = [
    (1, "Alice", 25, 50000),
    (2, "Bob", 30, 60000),
    (3, "Charlie", 35, 70000),
    (4, "David", 28, 55000),
    (5, "Eve", 32, 65000)
]

columns = ["id", "name", "age", "salary"]

df = spark.createDataFrame(data, columns)

print("DataFrame Schema:")
df.printSchema()

print("\nDataFrame Data:")
df.show()
```

```python
# CELL 3: DataFrame Operations
# Select
df.select("name", "salary").show()

# Filter
df.filter(df.age > 30).show()

# Group and Aggregate
df.groupBy("age").count().show()

# Sort
df.orderBy("salary", ascending=False).show()

# Add new column
from pyspark.sql.functions import col, lit

df_with_bonus = df.withColumn("bonus", col("salary") * 0.1)
df_with_bonus.show()
```

```python
# CELL 4: Read/Write CSV
# Write to CSV
df.write.mode("overwrite").csv("/data/employees.csv", header=True)

# Read from CSV
df_from_csv = spark.read.csv("/data/employees.csv", header=True, inferSchema=True)
df_from_csv.show()
```

---

### Lab 2: Delta Lake Operations

**Create**: `notebooks/02_delta_lake.ipynb`

```python
# CELL 1: Setup Delta Lake
from pyspark.sql import SparkSession
from delta import *

builder = SparkSession.builder \
    .appName("DeltaLake") \
    .master("local[*]") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .config("spark.jars.packages", "io.delta:delta-core_2.12:2.4.0")

spark = configure_spark_with_delta_pip(builder).getOrCreate()

print(f"Spark Version: {spark.version}")
print("Delta Lake configured successfully!")
```

```python
# CELL 2: Create Delta Table
# Create sample data
data = [
    (1, "Alice", 1000, "2024-01-01"),
    (2, "Bob", 1500, "2024-01-02"),
    (3, "Charlie", 2000, "2024-01-03")
]

df = spark.createDataFrame(data, ["id", "name", "amount", "date"])

# Write as Delta table
df.write.format("delta").mode("overwrite").save("/data/delta/transactions")

print("Delta table created!")
```

```python
# CELL 3: Read Delta Table
# Read Delta table
delta_df = spark.read.format("delta").load("/data/delta/transactions")
delta_df.show()

# Read specific version (Time Travel)
# delta_df_version_0 = spark.read.format("delta").option("versionAsOf", 0).load("/data/delta/transactions")
```

```python
# CELL 4: Delta Table Operations
from delta.tables import DeltaTable

# Load Delta table
delta_table = DeltaTable.forPath(spark, "/data/delta/transactions")

# Update data
delta_table.update(
    condition = "name = 'Alice'",
    set = { "amount": "2000" }
)

print("After update:")
delta_table.toDF().show()

# Delete data
delta_table.delete("name = 'Bob'")

print("After delete:")
delta_table.toDF().show()
```

```python
# CELL 5: Merge (Upsert) Operation
# New data to merge
new_data = spark.createDataFrame([
    (1, "Alice", 3000, "2024-01-04"),  # Update existing
    (4, "David", 2500, "2024-01-04")   # Insert new
], ["id", "name", "amount", "date"])

# Merge
delta_table.alias("target").merge(
    new_data.alias("source"),
    "target.id = source.id"
).whenMatchedUpdateAll().whenNotMatchedInsertAll().execute()

print("After merge:")
delta_table.toDF().show()
```

```python
# CELL 6: Time Travel
# View history
history = delta_table.history()
history.select("version", "timestamp", "operation", "operationParameters").show()

# Read previous version
print("\nVersion 0 (original):")
spark.read.format("delta").option("versionAsOf", 0).load("/data/delta/transactions").show()

print("\nVersion 1 (after update):")
spark.read.format("delta").option("versionAsOf", 1).load("/data/delta/transactions").show()
```

```python
# CELL 7: Delta Table Optimization
# Create larger dataset
large_data = [(i, f"User_{i}", i * 100, "2024-01-01") for i in range(1000)]
large_df = spark.createDataFrame(large_data, ["id", "name", "amount", "date"])

# Write as Delta
large_df.write.format("delta").mode("overwrite").save("/data/delta/large_table")

# Optimize
from pyspark.sql.functions import lit
spark.sql("OPTIMIZE '/data/delta/large_table'")

print("Table optimized!")

# Z-Order (if you have columns frequently queried)
# spark.sql("OPTIMIZE '/data/delta/large_table' ZORDER BY (name)")
```

---

### Lab 3: MLflow Tracking

**Create**: `notebooks/03_mlflow_tracking.ipynb`

```python
# CELL 1: Setup MLflow
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, precision_score, recall_score
import pandas as pd
import numpy as np

# Set MLflow tracking URI
mlflow.set_tracking_uri("http://localhost:5001")

# Create experiment
mlflow.set_experiment("/demo/iris-classification")

print(f"MLflow Version: {mlflow.__version__}")
print(f"Tracking URI: {mlflow.get_tracking_uri()}")
```

```python
# CELL 2: Generate Sample Data
from sklearn.datasets import load_iris

# Load iris dataset
iris = load_iris()
X = pd.DataFrame(iris.data, columns=iris.feature_names)
y = iris.target

# Split data
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

print(f"Training set: {X_train.shape}")
print(f"Test set: {X_test.shape}")
```

```python
# CELL 3: Train Model with MLflow Tracking
# Start MLflow run
with mlflow.start_run(run_name="RandomForest_Baseline"):
    
    # Model parameters
    n_estimators = 100
    max_depth = 5
    
    # Log parameters
    mlflow.log_param("n_estimators", n_estimators)
    mlflow.log_param("max_depth", max_depth)
    mlflow.log_param("model_type", "RandomForest")
    
    # Train model
    rf = RandomForestClassifier(
        n_estimators=n_estimators,
        max_depth=max_depth,
        random_state=42
    )
    rf.fit(X_train, y_train)
    
    # Predictions
    y_pred = rf.predict(X_test)
    
    # Calculate metrics
    accuracy = accuracy_score(y_test, y_pred)
    precision = precision_score(y_test, y_pred, average='weighted')
    recall = recall_score(y_test, y_pred, average='weighted')
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("precision", precision)
    mlflow.log_metric("recall", recall)
    
    # Log model
    mlflow.sklearn.log_model(rf, "model")
    
    # Log feature importances
    for i, importance in enumerate(rf.feature_importances_):
        mlflow.log_metric(f"feature_importance_{i}", importance)
    
    print(f"Accuracy: {accuracy:.4f}")
    print(f"Precision: {precision:.4f}")
    print(f"Recall: {recall:.4f}")
    print(f"Run ID: {mlflow.active_run().info.run_id}")
```

```python
# CELL 4: Hyperparameter Tuning with MLflow
from sklearn.model_selection import GridSearchCV

# Define parameter grid
param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [3, 5, 7, 10]
}

# Grid search
rf = RandomForestClassifier(random_state=42)
grid_search = GridSearchCV(rf, param_grid, cv=5, scoring='accuracy')

# Log each combination as separate run
for n_est in param_grid['n_estimators']:
    for depth in param_grid['max_depth']:
        with mlflow.start_run(run_name=f"RF_n{n_est}_d{depth}"):
            # Log params
            mlflow.log_param("n_estimators", n_est)
            mlflow.log_param("max_depth", depth)
            
            # Train
            rf = RandomForestClassifier(
                n_estimators=n_est,
                max_depth=depth,
                random_state=42
            )
            rf.fit(X_train, y_train)
            
            # Evaluate
            y_pred = rf.predict(X_test)
            accuracy = accuracy_score(y_test, y_pred)
            
            # Log metrics
            mlflow.log_metric("accuracy", accuracy)
            
            # Log model
            mlflow.sklearn.log_model(rf, "model")
            
            print(f"n_estimators={n_est}, max_depth={depth}, accuracy={accuracy:.4f}")
```

```python
# CELL 5: Register Best Model
# Get best run
runs = mlflow.search_runs(
    experiment_ids=mlflow.get_experiment_by_name("/demo/iris-classification").experiment_id,
    order_by=["metrics.accuracy DESC"]
)

best_run = runs.iloc[0]
best_run_id = best_run.run_id
best_accuracy = best_run['metrics.accuracy']

print(f"Best Run ID: {best_run_id}")
print(f"Best Accuracy: {best_accuracy:.4f}")

# Register model
model_uri = f"runs:/{best_run_id}/model"
model_name = "IrisClassifier"

mlflow.register_model(model_uri, model_name)

print(f"\nModel registered as: {model_name}")
```

```python
# CELL 6: Load and Use Registered Model
# Load model
loaded_model = mlflow.sklearn.load_model(f"models:/{model_name}/1")

# Make predictions
sample = X_test.iloc[0:5]
predictions = loaded_model.predict(sample)

print("Sample Predictions:")
for i, (features, pred) in enumerate(zip(sample.values, predictions)):
    print(f"Sample {i}: Features={features}, Predicted Class={pred}")
```

```python
# CELL 7: View MLflow UI Link
print("View MLflow UI at: http://localhost:5001")
print("\nIn the UI, you can:")
print("  - View all experiment runs")
print("  - Compare metrics across runs")
print("  - View model artifacts")
print("  - Download trained models")
print("  - Transition model stages")
```

---

### Lab 4: Complete Pipeline (Bronze-Silver-Gold)

**Create**: `notebooks/04_complete_pipeline.ipynb`

```python
# CELL 1: Setup
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count, sum, avg, max, min, to_date, year, month
from delta import *

# Configure Spark with Delta Lake
builder = SparkSession.builder \
    .appName("CompletePipeline") \
    .master("local[*]") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")

spark = configure_spark_with_delta_pip(builder).getOrCreate()

print("Spark + Delta Lake ready!")
```

```python
# CELL 2: Generate Raw Data
import random
from datetime import datetime, timedelta

# Generate synthetic taxi trip data
num_trips = 10000
base_date = datetime(2024, 1, 1)

data = []
for i in range(num_trips):
    trip_date = base_date + timedelta(days=random.randint(0, 30))
    data.append((
        i,  # trip_id
        trip_date.strftime("%Y-%m-%d %H:%M:%S"),  # pickup_datetime
        (trip_date + timedelta(minutes=random.randint(5, 60))).strftime("%Y-%m-%d %H:%M:%S"),  # dropoff_datetime
        round(random.uniform(1, 20), 2),  # trip_distance
        round(random.uniform(5, 100), 2),  # fare_amount
        round(random.uniform(0, 20), 2),  # tip_amount
        random.randint(10000, 99999),  # pickup_zip
        random.randint(10000, 99999)   # dropoff_zip
    ))

raw_df = spark.createDataFrame(
    data,
    ["trip_id", "pickup_datetime", "dropoff_datetime", "trip_distance", 
     "fare_amount", "tip_amount", "pickup_zip", "dropoff_zip"]
)

print(f"Generated {raw_df.count()} raw trips")
raw_df.show(5)
```

```python
# CELL 3: Bronze Layer - Raw Data
# Add ingestion metadata
from pyspark.sql.functions import current_timestamp, lit

bronze_df = raw_df \
    .withColumn("ingestion_timestamp", current_timestamp()) \
    .withColumn("source_system", lit("synthetic_data_generator"))

# Write to Bronze layer
bronze_df.write.format("delta").mode("overwrite").save("/data/bronze/taxi_trips")

print("✓ Bronze layer created")
print(f"Records: {bronze_df.count()}")
```

```python
# CELL 4: Silver Layer - Cleansed Data
from pyspark.sql.functions import col, unix_timestamp, round as spark_round

# Read from Bronze
bronze_data = spark.read.format("delta").load("/data/bronze/taxi_trips")

# Data quality checks
silver_df = bronze_data \
    .filter(col("fare_amount") > 0) \
    .filter(col("trip_distance") > 0) \
    .filter(col("pickup_datetime") < col("dropoff_datetime"))

# Add derived columns
silver_df = silver_df \
    .withColumn(
        "trip_duration_minutes",
        spark_round(
            (unix_timestamp("dropoff_datetime") - unix_timestamp("pickup_datetime")) / 60,
            2
        )
    ) \
    .withColumn(
        "tip_percentage",
        spark_round((col("tip_amount") / col("fare_amount")) * 100, 2)
    ) \
    .withColumn(
        "is_high_tip",
        (col("tip_percentage") > 20).cast("int")
    )

# Write to Silver layer
silver_df.write.format("delta").mode("overwrite").save("/data/silver/taxi_trips")

print("✓ Silver layer created")
print(f"Records: {silver_df.count()}")
silver_df.show(5)
```

```python
# CELL 5: Gold Layer - Business Metrics
# Daily metrics
daily_metrics = silver_df \
    .withColumn("trip_date", to_date("pickup_datetime")) \
    .groupBy("trip_date") \
    .agg(
        count("*").alias("total_trips"),
        sum("fare_amount").alias("total_revenue"),
        avg("fare_amount").alias("avg_fare"),
        sum("tip_amount").alias("total_tips"),
        avg("tip_percentage").alias("avg_tip_percentage")
    ) \
    .orderBy("trip_date")

daily_metrics.write.format("delta").mode("overwrite").save("/data/gold/daily_metrics")

print("✓ Gold layer (daily metrics) created")
daily_metrics.show(10)
```

```python
# CELL 6: Gold Layer - Location Metrics
location_metrics = silver_df \
    .groupBy("pickup_zip") \
    .agg(
        count("*").alias("total_trips"),
        sum("fare_amount").alias("total_revenue"),
        avg("fare_amount").alias("avg_fare"),
        sum("tip_amount").alias("total_tips")
    ) \
    .orderBy(col("total_trips").desc())

location_metrics.write.format("delta").mode("overwrite").save("/data/gold/location_metrics")

print("✓ Gold layer (location metrics) created")
location_metrics.show(10)
```

```python
# CELL 7: Create Visualizations
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Convert to Pandas for visualization
daily_pd = daily_metrics.toPandas()

# Create visualization
fig, axes = plt.subplots(2, 2, figsize=(15, 10))

# Plot 1: Daily trips
axes[0, 0].plot(daily_pd['trip_date'], daily_pd['total_trips'], marker='o')
axes[0, 0].set_title('Daily Trip Count')
axes[0, 0].set_xlabel('Date')
axes[0, 0].set_ylabel('Trips')
axes[0, 0].tick_params(axis='x', rotation=45)

# Plot 2: Daily revenue
axes[0, 1].bar(daily_pd['trip_date'], daily_pd['total_revenue'])
axes[0, 1].set_title('Daily Revenue')
axes[0, 1].set_xlabel('Date')
axes[0, 1].set_ylabel('Revenue ($)')
axes[0, 1].tick_params(axis='x', rotation=45)

# Plot 3: Average fare
axes[1, 0].plot(daily_pd['trip_date'], daily_pd['avg_fare'], marker='s', color='green')
axes[1, 0].set_title('Average Fare')
axes[1, 0].set_xlabel('Date')
axes[1, 0].set_ylabel('Avg Fare ($)')
axes[1, 0].tick_params(axis='x', rotation=45)

# Plot 4: Average tip percentage
axes[1, 1].plot(daily_pd['trip_date'], daily_pd['avg_tip_percentage'], marker='^', color='orange')
axes[1, 1].set_title('Average Tip Percentage')
axes[1, 1].set_xlabel('Date')
axes[1, 1].set_ylabel('Tip %')
axes[1, 1].tick_params(axis='x', rotation=45)

plt.tight_layout()
plt.savefig('/data/daily_metrics_visualization.png', dpi=150, bbox_inches='tight')
plt.show()

print("\n✓ Visualization saved to /data/daily_metrics_visualization.png")
```

```python
# CELL 8: Summary Statistics
print("=" * 60)
print("PIPELINE SUMMARY")
print("=" * 60)

print(f"\nBronze Layer:")
print(f"  Records: {spark.read.format('delta').load('/data/bronze/taxi_trips').count()}")

print(f"\nSilver Layer:")
print(f"  Records: {spark.read.format('delta').load('/data/silver/taxi_trips').count()}")

print(f"\nGold Layer:")
print(f"  Daily Metrics: {spark.read.format('delta').load('/data/gold/daily_metrics').count()} days")
print(f"  Location Metrics: {spark.read.format('delta').load('/data/gold/location_metrics').count()} locations")

print("\n✓ Pipeline complete!")
```

---

## Lab 5: PySpark vs Databricks Comparison

**Create**: `notebooks/05_pyspark_vs_databricks.ipynb`

```python
# CELL 1: Feature Comparison
comparison = """
╔═══════════════════════════════════════════════════════════════╗
║           PySpark (Docker) vs Azure Databricks               ║
╠═══════════════════════════════════════════════════════════════╣
║ Feature              │ PySpark Local  │ Azure Databricks     ║
╠═══════════════════════════════════════════════════════════════╣
║ Cost                 │ Free           │ Pay per DBU          ║
║ Setup                │ Manual         │ Managed              ║
║ Cluster Management   │ Manual         │ Automatic            ║
║ Scalability          │ Single machine │ Unlimited            ║
║ Collaboration        │ Single user    │ Multi-user           ║
║ Delta Lake           ✓ Supported      ✓ Native               ║
║ MLflow               ✓ Supported      ✓ Integrated           ║
║ Unity Catalog        ✗ Not available  ✓ Available            ║
║ Jobs/Scheduling      ✗ Manual         ✓ Built-in             ║
║ Dashboard            ✗ Not available  ✓ AI/BI Dashboards     ║
║ Security             ✗ Basic          ✓ Enterprise           ║
╚═══════════════════════════════════════════════════════════════╝
"""
print(comparison)
```

```python
# CELL 2: Code Translation Guide
code_comparison = {
    "Initialize Spark": {
        "Databricks": "spark  # Already available",
        "PySpark Local": """spark = SparkSession.builder \\
    .appName("MyApp") \\
    .getOrCreate()"""
    },
    
    "Read Table": {
        "Databricks": 'df = spark.read.table("catalog.schema.table")',
        "PySpark Local": 'df = spark.read.format("delta").load("/path/to/table")'
    },
    
    "Display Data": {
        "Databricks": "display(df)  # Rich UI",
        "PySpark Local": "df.show()  # Basic text"
    },
    
    "Create Widget": {
        "Databricks": 'dbutils.widgets.text("param", "default")',
        "PySpark Local": "# Not available - use environment variables or config files"
    },
    
    "Mount Storage": {
        "Databricks": 'dbutils.fs.mount(source, mount_point)',
        "PySpark Local": "# Direct filesystem access"
    },
    
    "Secret Management": {
        "Databricks": 'dbutils.secrets.get(scope, key)',
        "PySpark Local": "# Use environment variables or external secret managers"
    },
    
    "Run SQL": {
        "Databricks": 'spark.sql("SELECT * FROM table")',
        "PySpark Local": 'spark.sql("SELECT * FROM table")  # Same"'
    }
}

for feature, code in code_comparison.items():
    print(f"\n{feature}:")
    print(f"  Databricks: {code['Databricks']}")
    print(f"  PySpark:    {code['PySpark Local']}")
```

```python
# CELL 3: What You Can Learn with Docker
learnable_topics = """
Skills You Can Learn with Docker:
═══════════════════════════════════════════════════════════

✓ CORE SKILLS (Fully Learnable)
  ├── PySpark DataFrame operations
  ├── Spark SQL
  ├── Delta Lake operations
  ├── MLflow tracking
  ├── Basic ML workflows
  ├── Data transformations
  └── File I/O operations

✓ INTERMEDIATE SKILLS (Partially Learnable)
  ├── Data pipeline design
  ├── Medallion architecture
  ├── Data quality checks
  ├── Performance optimization
  └── Notebook organization

✗ ADVANCED SKILLS (Require Real Databricks)
  ├── Unity Catalog
  ├── Multi-user collaboration
  ├── Databricks Jobs
  ├── Cluster management
  ├── AI/BI Dashboards
  ├── Databricks SQL
  └── Model Serving

RECOMMENDATION:
──────────────────────────────────
Start with Docker for fundamentals,
then move to Databricks Free Edition
or Azure Trial for advanced features.
"""
print(learnable_topics)
```

---

## Quick Commands Reference

### Docker Commands

```powershell
# Start containers
docker-compose up -d

# View running containers
docker-compose ps

# View logs
docker-compose logs -f
docker-compose logs jupyter

# Stop containers
docker-compose stop

# Stop and remove containers
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v

# Rebuild images
docker-compose build --no-cache

# Execute command in container
docker-compose exec jupyter bash

# Copy file from container
docker cp pyspark-lab:/home/jovyan/work/notebook.ipynb ./local_copy.ipynb

# Copy file to container
docker cp ./local_file.csv pyspark-lab:/data/
```

### Jupyter Commands (Inside Container)

```bash
# List running notebooks
jupyter notebook list

# Generate config
jupyter notebook --generate-config

# Set password
jupyter notebook password
```

### Spark Commands (Inside Notebook)

```python
# Check Spark UI
print("Spark UI: http://localhost:4040")

# List tables
spark.catalog.listTables()

# Clear cache
spark.catalog.clearCache()

# Stop Spark
spark.stop()
```

---

## Troubleshooting

### Issue 1: Port Already in Use

```powershell
# Check what's using the port
netstat -ano | findstr :8888

# Kill the process
taskkill /PID <process_id> /F

# Or change port in docker-compose.yml
ports:
  - "8889:8888"  # Use different host port
```

### Issue 2: Insufficient Memory

```powershell
# Check Docker memory allocation
# Docker Desktop → Settings → Resources

# Reduce Spark memory in notebooks
spark = SparkSession.builder \
    .config("spark.driver.memory", "2g") \
    .config("spark.executor.memory", "2g") \
    .getOrCreate()
```

### Issue 3: Delta Lake Not Found

```python
# Make sure Delta Lake is installed
!pip install delta-spark

# Or specify version
!pip install delta-spark==2.4.0
```

### Issue 4: Cannot Connect to MLflow

```python
# Check MLflow is running
# In terminal:
!curl http://localhost:5000/health

# Or use correct URI
mlflow.set_tracking_uri("http://mlflow:5000")  # Use service name
# OR
mlflow.set_tracking_uri("http://localhost:5001")  # Use host port
```

### Issue 5: Slow Performance

```python
# Enable Spark caching
df.cache()

# Use parquet instead of CSV
df.write.parquet("/data/file.parquet")

# Increase parallelism
spark.conf.set("spark.sql.shuffle.partitions", "8")

# Check Spark UI for bottlenecks
# http://localhost:4040
```

---

## Next Steps

### Path 1: Continue with Docker

1. Complete all 5 labs
2. Experiment with larger datasets
3. Try different ML algorithms
4. Optimize Spark performance
5. Build your own pipelines

### Path 2: Move to Real Databricks

1. **Databricks Free Edition**: Sign up for free
2. **Azure Trial**: 14-day free Premium DBUs
3. Apply your Docker knowledge
4. Learn Unity Catalog
5. Deploy production pipelines

### Recommended Learning Path

```
Week 1-2: Docker Labs
├── Spark basics
├── Delta Lake
└── MLflow

Week 3-4: Databricks Free Edition
├── Unity Catalog
├── Collaboration
└── Jobs

Week 5-6: Real Projects
├── Production pipeline
├── ML deployment
└── Dashboards
```

---

## Resources

### Official Documentation

- [Apache Spark](https://spark.apache.org/docs/latest/)
- [Delta Lake](https://docs.delta.io/latest/index.html)
- [MLflow](https://mlflow.org/docs/latest/index.html)
- [PySpark API](https://spark.apache.org/docs/latest/api/python/)

### Docker Images

- [Jupyter PySpark](https://hub.docker.com/r/jupyter/pyspark-notebook)
- [Bitnami Spark](https://hub.docker.com/r/bitnami/spark)
- [MinIO](https://hub.docker.com/r/minio/minio)

### Free Databricks

- [Databricks Free Edition](https://www.databricks.com/product/free-edition)
- [Azure Databricks Trial](https://azure.microsoft.com/free/databricks/)

---

**Document Version**: 1.0  
**Last Updated**: November 2026

**Related Guides**:
- Azure Databricks Learning Guide
- Demo Project
- Workspace Deployment Guide
