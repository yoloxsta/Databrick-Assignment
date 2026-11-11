# Azure Databricks: Complete Learning & Lab Guide

## What is Azure Databricks?

Azure Databricks is a **unified, open analytics platform** for building, deploying, sharing, and maintaining enterprise-grade data, analytics, and AI solutions at scale. It's a managed Apache Spark-based analytics platform optimized for Microsoft Azure.

### Core Definition
- **Unified Platform**: Combines data engineering, data science, machine learning, and analytics in one place
- **Open Source Foundation**: Built on Apache Spark, created by the original creators of Spark
- **Lakehouse Architecture**: Merges the best of data warehouses and data lakes
- **Managed Service**: Azure handles infrastructure, you focus on code and data

### Key Technologies Created by Databricks
- **Delta Lake**: Open source storage layer bringing reliability to data lakes
- **MLflow**: Open source platform for the machine learning lifecycle
- **Apache Spark**: Unified analytics engine for large-scale data processing
- **Unity Catalog**: Unified governance solution for data and AI
- **Structured Streaming**: Scalable and fault-tolerant stream processing engine

---

## Why Use Azure Databricks?

### 1. **Lakehouse Architecture Benefits**
- **Single Source of Truth**: Data engineers, scientists, and analysts use the same data
- **Eliminates Data Silos**: No need to maintain separate data warehouses and data lakes
- **Reduced Complexity**: One platform instead of multiple disconnected systems
- **Cost Efficiency**: Store data in open formats (Parquet) on cheap object storage

### 2. **Performance & Scalability**
- **10-100x Performance Improvement**: Optimized Databricks Runtime vs. vanilla Spark
- **Auto-scaling**: Automatically scales compute resources up/down
- **Serverless Compute**: No infrastructure management required
- **Infinite Scale**: Handles petabytes of data

### 3. **Collaboration & Productivity**
- **Interactive Notebooks**: Collaborative coding environment supporting Python, SQL, Scala, R
- **Real-time Collaboration**: Multiple users can work simultaneously
- **Version Control Integration**: Git-based workflows
- **Visual Workflows**: Low-code/no-code pipeline creation

### 4. **Unified Governance**
- **Unity Catalog**: Centralized data governance across all workspaces
- **Fine-grained Access Control**: Row-level and column-level security
- **Audit Logging**: Complete visibility into data access
- **Data Lineage**: Track data from source to consumption

### 5. **AI & Machine Learning Integration**
- **Built-in ML Runtime**: Pre-installed libraries (scikit-learn, TensorFlow, PyTorch)
- **MLflow Integration**: Experiment tracking, model versioning, deployment
- **Feature Store**: Share and discover ML features
- **LLM Support**: Fine-tune and deploy large language models

### 6. **Enterprise Security**
- **Native Azure Integration**: Azure Active Directory (Entra ID) authentication
- **VNET Integration**: Deploy in your own virtual network
- **Private Links**: Secure connectivity to Azure services
- **Compliance**: SOC, HIPAA, GDPR compliant

### Common Use Cases

| Use Case | Description |
|----------|-------------|
| **ETL & Data Engineering** | Build reliable data pipelines with Auto Loader and Delta Lake |
| **Data Warehousing & BI** | Run SQL queries at scale, create dashboards |
| **Machine Learning** | Train, tune, and deploy ML models at scale |
| **Real-time Analytics** | Process streaming data with Structured Streaming |
| **Data Science** | Exploratory data analysis, statistical modeling |
| **AI Applications** | Build and deploy AI agents and LLM applications |
| **Data Sharing** | Secure data sharing within and across organizations |

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Azure Databricks Account                  │
│  (Control Plane - Managed by Databricks in Azure)           │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Web App UI  │  │   Job        │  │   Unity      │     │
│  │              │  │   Scheduler  │  │   Catalog    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                              │
                              │
        ┌─────────────────────┴─────────────────────┐
        │                                           │
        ▼                                           ▼
┌───────────────────────┐               ┌──────────────────────┐
│  Serverless Compute   │               │   Classic Compute    │
│  Plane (Databricks    │               │   Plane (Your Azure  │
│  Account)             │               │   Subscription)      │
│                       │               │                      │
│  • Serverless SQL     │               │  • Interactive       │
│    Warehouses         │               │    Clusters          │
│  • Model Serving      │               │  • Job Clusters      │
│  • Serverless Jobs    │               │  • All-purpose       │
│                       │               │    Clusters          │
└───────────────────────┘               └──────────────────────┘
        │                                           │
        │                                           │
        └─────────────────────┬─────────────────────┘
                              │
                              ▼
              ┌────────────────────────────┐
              │   Data Storage Layer       │
              │                            │
              │  • Azure Data Lake         │
              │    Storage Gen2            │
              │  • Unity Catalog Volumes   │
              │  • Delta Lake Tables       │
              └────────────────────────────┘
```

### Key Architectural Components

#### 1. **Control Plane** (Managed by Databricks)
- Web application UI
- Notebook management
- Job scheduler
- Cluster management
- Security and access control

#### 2. **Compute Plane** (Where data is processed)
- **Serverless Compute Plane**: Managed by Databricks, instant startup
- **Classic Compute Plane**: Runs in your Azure subscription, more control

#### 3. **Storage Layer**
- **Unity Catalog**: Governed data storage
- **Delta Lake**: ACID transactions on data lakes
- **Volumes**: Managed storage for files

### Workspace Hierarchy

```
Account (Top level)
├── Workspaces (Collaboration environments)
│   ├── Notebooks (Code documents)
│   ├── Jobs (Scheduled tasks)
│   ├── Clusters (Compute resources)
│   └── Models (MLflow models)
├── Unity Catalog Metastore (Data governance)
│   ├── Catalogs (Top-level data container)
│   │   ├── Schemas (Databases)
│   │   │   ├── Tables
│   │   │   ├── Volumes
│   │   │   └── Functions
```

---

## How to Get Started: Step-by-Step Guide

### Option 1: Databricks Free Edition (Recommended for Learning)

**Best for**: Students, educators, hobbyists, personal learning

**Steps**:

1. **Sign up for Free Edition**
   - Go to: https://www.databricks.com/product/free-edition
   - Choose your signup method (Google, GitHub, Microsoft, or email)
   - Databricks creates a workspace automatically

2. **What You Get**:
   - Serverless compute included
   - Default storage provided
   - Unity Catalog enabled
   - Free quota (limited but sufficient for learning)
   - No credit card required

3. **Limitations**:
   - Python only (no Scala or R)
   - Limited compute quota
   - No outbound internet access (must upload files via UI)
   - Quota limits on compute hours

### Option 2: Azure Free Trial (14-Day Premium Access)

**Best for**: Organizations evaluating Azure Databricks for production

**Steps**:

1. **Prerequisites**:
   - Azure subscription (not a free trial subscription)
   - If you have Azure free account, upgrade to pay-as-you-go
   - Remove spending limit on your subscription

2. **Create Workspace**:
   ```
   1. Go to Azure Portal (portal.azure.com)
   2. Click "Create a resource" → "Analytics" → "Azure Databricks"
   3. Fill in details:
      - Workspace name: <your-workspace-name>
      - Subscription: <your-subscription>
      - Resource group: <new-or-existing>
      - Location: West US 2 (or preferred region)
      - Pricing Tier: Trial (Premium - 14-Days Free DBUs)
   4. Click "Review + Create" → "Create"
   5. Wait 2-5 minutes for deployment
   ```

3. **Access Your Workspace**:
   - Navigate to your Databricks resource in Azure Portal
   - Click "Launch Workspace"
   - You're automatically added as admin

---

## Hands-On Lab: Getting Started Tutorial

### Lab 1: Create Your First Notebook and Query Data

**Objective**: Learn to use notebooks and query sample data

**Time**: 15-20 minutes

**Steps**:

1. **Create a New Notebook**
   - In your workspace, click "New" → "Notebook"
   - Name it: `My First Notebook`
   - Default language: Python (Free Edition) or SQL

2. **Query Sample Data**

   **Option A: Using SQL**
   ```sql
   SELECT * FROM samples.nyctaxi.trips
   LIMIT 100
   ```

   **Option B: Using Python**
   ```python
   display(spark.read.table("samples.nyctaxi.trips"))
   ```

   **Option C: Using Scala**
   ```scala
   display(spark.read.table("samples.nyctaxi.trips"))
   ```

3. **Visualize the Data**
   - Run the cell (Shift + Enter)
   - Click "+" next to "Table" tab
   - Select "Visualization"
   - Choose chart type (Bar, Line, Scatter, etc.)
   - Configure X and Y axes
   - Click "Save"

4. **Practice Exercise**: Create a bar chart showing average fare amount by trip distance, grouped by pickup zip code

### Lab 2: Load and Transform Data with DataFrames

**Objective**: Learn PySpark DataFrame operations

**Time**: 30-45 minutes

**Complete Tutorial**: Follow the steps below:

#### Step 1: Setup Variables

```python
# Replace with your catalog, schema, and volume names
catalog = "your_catalog_name"  # Free Edition: use workspace catalog
schema = "default"              # Free Edition: use default schema
volume = "my_volume"            # Create if doesn't exist
table_name = "baby_names"

# Create volume if it doesn't exist
spark.sql(f"CREATE VOLUME IF NOT EXISTS {catalog}.{schema}.{volume}")

# Set paths
path_volume = f"/Volumes/{catalog}/{schema}/{volume}"
path_table = f"{catalog}.{schema}"
```

#### Step 2: Load CSV Data

**Option A: Upload via UI** (Required for Free Edition)
1. Download: https://health.data.ny.gov/api/views/jxy9-yhdk/rows.csv
2. In workspace: Click "New" → "Add or upload data"
3. Select "Upload files to a volume"
4. Choose the rows.csv file
5. Select your volume destination

**Option B: Download via Code** (If outbound internet allowed)
```python
download_url = "https://health.data.ny.gov/api/views/jxy9-yhdk/rows.csv"
file_name = "rows.csv"
dbutils.fs.cp(download_url, f"{path_volume}/{file_name}")
```

#### Step 3: Create and Explore DataFrames

```python
# Load CSV into DataFrame
df = spark.read.csv(
    f"{path_volume}/rows.csv",
    header=True,
    inferSchema=True,
    sep=","
)

# Display data
display(df)

# View schema
df.printSchema()
```

#### Step 4: Transform Data

```python
from pyspark.sql.functions import desc

# Rename column
df = df.withColumnRenamed("First Name", "First_Name")

# Filter rows
filtered_df = df.filter(df["Count"] > 50)

# Select and order
top_names = df.select("First_Name", "Count") \
               .orderBy(desc("Count"))

display(top_names)

# Complex filter
subset = df.filter(
    (df["Year"] == 2009) & 
    (df["Count"] > 100) & 
    (df["Sex"] == "F")
).select("First_Name", "County", "Count") \
 .orderBy(desc("Count"))

display(subset)
```

#### Step 5: Save Results

```python
# Save as Delta table
df.write.mode("overwrite").saveAsTable(f"{path_table}.{table_name}")

# Save as JSON
df.write.format("json").mode("overwrite").save("/tmp/json_data")
```

### Lab 3: SQL Analytics and Dashboards

**Objective**: Learn Databricks SQL for analytics

**Time**: 20-30 minutes

**Steps**:

1. **Create a SQL Warehouse**
   - Click "SQL" in sidebar
   - Click "Create SQL Warehouse"
   - Choose "Serverless" (recommended)
   - Name it and click "Create"

2. **Write SQL Queries**

   ```sql
   -- Query 1: Total trips by year
   SELECT 
       YEAR(tpep_pickup_datetime) as year,
       COUNT(*) as total_trips,
       AVG(fare_amount) as avg_fare
   FROM samples.nyctaxi.trips
   GROUP BY YEAR(tpep_pickup_datetime)
   ORDER BY year

   -- Query 2: Top pickup locations
   SELECT 
       pickup_zip,
       COUNT(*) as trip_count,
       SUM(fare_amount) as total_revenue
   FROM samples.nyctaxi.trips
   GROUP BY pickup_zip
   ORDER BY trip_count DESC
   LIMIT 10
   ```

3. **Create a Dashboard**
   - Click "Dashboards" → "Create Dashboard"
   - Add visualizations from your queries
   - Arrange in a grid layout
   - Add text boxes for context
   - Share with team members

### Lab 4: Build an ETL Pipeline

**Objective**: Create a production data pipeline

**Time**: 45-60 minutes

**Steps**:

1. **Use Auto Loader for Incremental Ingestion**

   ```python
   # Auto Loader automatically detects new files
   df = spark.readStream.format("cloudFiles") \
       .option("cloudFiles.format", "csv") \
       .option("cloudFiles.schemaLocation", f"{path_volume}/schema") \
       .load(f"{path_volume}/input/")
   
   # Write to Delta table
   df.writeStream.format("delta") \
       .option("checkpointLocation", f"{path_volume}/checkpoint") \
       .toTable(f"{path_table}.raw_data")
   ```

2. **Create Delta Live Tables Pipeline**

   ```python
   import dlt
   
   @dlt.table
   def bronze_layer():
       return spark.readStream.format("cloudFiles") \
           .option("cloudFiles.format", "json") \
           .load("/path/to/raw")
   
   @dlt.table
   def silver_layer():
       return dlt.read_stream("bronze_layer") \
           .filter("status = 'valid'") \
           .withColumn("processed_date", current_date())
   
   @dlt.table
   def gold_layer():
       return dlt.read("silver_layer") \
           .groupBy("category") \
           .agg(sum("amount").alias("total_amount"))
   ```

3. **Schedule as a Job**
   - Click "Jobs" → "Create Job"
   - Add notebook or pipeline task
   - Set schedule (cron expression)
   - Configure alerts
   - Set retry policy

### Lab 5: Machine Learning with MLflow

**Objective**: Build and track ML models

**Time**: 45-60 minutes

**Steps**:

```python
import mlflow
import mlflow.sklearn
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# Load data
df = spark.read.table("samples.nyctaxi.trips").toPandas()

# Prepare features
feature_cols = ["trip_distance", "passenger_count"]
X = df[feature_cols]
y = df["high_tip_flag"]  # Binary target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Start MLflow experiment
with mlflow.start_run():
    # Train model
    rf = RandomForestClassifier(n_estimators=100, max_depth=10)
    rf.fit(X_train, y_train)
    
    # Predict
    predictions = rf.predict(X_test)
    accuracy = accuracy_score(y_test, predictions)
    
    # Log parameters
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 10)
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    
    # Log model
    mlflow.sklearn.log_model(rf, "random_forest_model")
    
    print(f"Accuracy: {accuracy}")

# Register model
model_uri = f"runs:/{mlflow.active_run().info.run_id}/random_forest_model"
mlflow.register_model(model_uri, "tip_prediction_model")
```

---

## Learning Roadmap

### Beginner (Weeks 1-2)
- [ ] Complete Free Edition signup
- [ ] Navigate the workspace UI
- [ ] Create and run notebooks
- [ ] Query sample datasets
- [ ] Create basic visualizations
- [ ] Learn PySpark DataFrame basics

### Intermediate (Weeks 3-4)
- [ ] Understand Delta Lake format
- [ ] Build ETL pipelines
- [ ] Use Auto Loader for data ingestion
- [ ] Create scheduled jobs
- [ ] Learn SQL warehouse concepts
- [ ] Implement basic data transformations

### Advanced (Weeks 5-8)
- [ ] Master Delta Live Tables
- [ ] Implement streaming pipelines
- [ ] Build ML models with MLflow
- [ ] Configure Unity Catalog
- [ ] Set up CI/CD with Databricks
- [ ] Optimize performance
- [ ] Implement advanced security

### Expert (Months 3-6)
- [ ] Architect multi-workspace solutions
- [ ] Implement data mesh architecture
- [ ] Build real-time streaming solutions
- [ ] Deploy ML models at scale
- [ ] Create custom connectors
- [ ] Optimize cost management
- [ ] Lead data platform migrations

---

## Best Practices

### 1. Data Organization
```
Bronze Layer (Raw)
├── Raw data as ingested
├── Append-only tables
└── Minimal transformations

Silver Layer (Cleansed)
├── Cleaned and validated data
├── Business logic applied
└── Deduplicated records

Gold Layer (Business-Ready)
├── Aggregated metrics
├── Business-specific views
└── Optimized for queries
```

### 2. Compute Management
- Use **serverless** for SQL queries and light workloads
- Use **job clusters** for production pipelines
- Use **interactive clusters** for development only
- Enable **auto-termination** (10-30 minutes)
- Use **spot instances** for cost savings (non-critical workloads)

### 3. Code Organization
- Use **Git integration** for version control
- Separate **notebooks by purpose** (ingestion, transform, analyze)
- Create **modular functions** for reusable code
- Use **Databricks Repos** for CI/CD

### 4. Security
- Enable **Unity Catalog** from day 1
- Use **service principals** for automated jobs
- Implement **row-level security** for multi-tenant data
- Regular **access reviews**
- Enable **audit logging**

### 5. Cost Optimization
- Monitor **DBU consumption** regularly
- Use **spot instances** where possible
- Right-size **cluster configurations**
- Implement **auto-scaling** policies
- Use **Delta cache** for frequently accessed data

---

## Key Concepts to Master

### 1. Delta Lake
- **ACID Transactions**: Reliable data operations
- **Time Travel**: Query historical data versions
- **Schema Evolution**: Handle changing data structures
- **MERGE Operation**: Upsert data efficiently
- **Compaction**: Optimize file sizes

### 2. Unity Catalog
- **Three-level namespace**: catalog.schema.table
- **Fine-grained permissions**: Table, row, column level
- **Data lineage**: Track data flow
- **Audit logs**: Monitor access
- **Centralized governance**: One metastore for all workspaces

### 3. Compute Types
| Type | Use Case | Startup Time | Cost |
|------|----------|--------------|------|
| Serverless | SQL, notebooks, jobs | Seconds | Higher per-hour |
| Interactive | Development | 5-10 min | Running cost |
| Job clusters | Production pipelines | 5-10 min | Only when running |

### 4. File Formats
- **Parquet**: Columnar, compressed, efficient queries
- **Delta Lake**: Parquet + ACID + features
- **JSON**: Semi-structured, human-readable
- **CSV**: Simple, widely compatible

---

## Common Pitfalls to Avoid

1. **Not Using Unity Catalog**
   - Start with Unity Catalog from day 1
   - Hard to retrofit governance later

2. **Over-engineering Clusters**
   - Start small, scale as needed
   - Use auto-scaling

3. **Ignoring Cost Management**
   - Set budgets and alerts early
   - Monitor DBU consumption weekly

4. **Not Using Delta Lake**
   - Always use Delta format for tables
   - Provides reliability features

5. **Skipping Tests**
   - Implement data quality checks
   - Use assertions in pipelines

6. **Poor Secret Management**
   - Use Databricks secret scopes
   - Never hardcode credentials

---

## Additional Resources

### Official Documentation
- [Azure Databricks Documentation](https://learn.microsoft.com/en-us/azure/databricks/)
- [Databricks Academy](https://www.databricks.com/training)
- [Databricks Community](https://community.databricks.com/)

### Free Learning Paths
1. [Query and visualize data](https://learn.microsoft.com/en-us/azure/databricks/getting-started/quick-start)
2. [Load and transform data with DataFrames](https://learn.microsoft.com/azure/databricks/getting-started/dataframes)
3. [Build ETL pipelines](https://learn.microsoft.com/en-us/azure/databricks/getting-started/data-pipeline-get-started)
4. [Build ML models](https://learn.microsoft.com/en-us/azure/databricks/getting-started/ml-get-started)
5. [Query LLMs and build agents](https://learn.microsoft.com/en-us/azure/databricks/getting-started/gen-ai-llm-agent)

### Certification Paths
- **Databricks Certified Data Engineer Associate**
- **Databricks Certified Data Engineer Professional**
- **Databricks Certified Machine Learning Professional**

### Sample Datasets
- `samples.nyctaxi.trips` - NYC Taxi data
- `samples.tpch` - TPC-H benchmark data
- `samples.datasets` - Various sample datasets

---

## Quick Reference Commands

### Notebook Shortcuts
- **Shift + Enter**: Run cell and move to next
- **Ctrl + Enter**: Run cell and stay
- **Ctrl + Shift + Enter**: Run all cells
- **Esc + A**: Insert cell above
- **Esc + B**: Insert cell below
- **Esc + DD**: Delete cell

### Common PySpark Commands
```python
# Read data
df = spark.read.format("delta").load("/path/to/table")
df = spark.read.csv("/path/to/file", header=True)
df = spark.read.table("catalog.schema.table")

# Display
display(df)
df.show(10)
df.printSchema()

# Transform
df = df.filter(col("status") == "active")
df = df.select("col1", "col2")
df = df.groupBy("category").agg(sum("amount"))

# Write
df.write.format("delta").save("/path/to/table")
df.write.saveAsTable("catalog.schema.table")
df.write.mode("overwrite").saveAsTable("table_name")

# SQL
spark.sql("SELECT * FROM table_name")
df.createOrReplaceTempView("temp_table")
```

---

## Next Steps

1. **Today**: Sign up for Databricks Free Edition
2. **This Week**: Complete Labs 1-3
3. **Next Week**: Build your first ETL pipeline (Lab 4)
4. **Month 1**: Complete beginner and intermediate learning
5. **Month 2-3**: Work on real projects, consider certification

---

**Document Version**: 1.0  
**Last Updated**: November 2026  
**Sources**: Microsoft Learn, Databricks Documentation

**Content Attribution**: Content was rephrased for compliance with licensing restrictions. Original sources available at:
- [Azure Databricks Documentation](https://learn.microsoft.com/en-us/azure/databricks/)
- [Databricks Product Page](https://databricks.com/product/azure)
