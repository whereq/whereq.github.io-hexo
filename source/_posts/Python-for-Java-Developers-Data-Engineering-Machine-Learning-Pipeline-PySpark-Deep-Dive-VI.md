---
title: >-
  Python for Java Developers: Data Engineering & Machine Learning Pipeline -
  PySpark Deep Dive - VI
date: 2025-12-23 23:23:52
categories:
- Python
tags:
- Python
---

# Python for Java Developers: Data Engineering & Machine Learning Pipeline - PySpark Deep Dive

## 🎯 **Chapater 6: PySpark - Where Your Java Spark Knowledge Pays Off**

### **The Critical Insight:**
Your **Java Spark experience** is NOT wasted! PySpark is just **Spark's Java API exposed to Python**. You already know 80% of it!

```python
# Java Spark vs PySpark - SAME CONCEPTS!
"""
Java: Dataset<Row> df = spark.read().json("data.json");
Python: df = spark.read.json("data.json")

Java: df.filter(col("age").gt(18));
Python: df.filter(col("age") > 18)

Java: df.groupBy("department").avg("salary");
Python: df.groupBy("department").avg("salary")

IT'S THE SAME API, JUST PYTHON SYNTAX!
"""
```

## 📊 **1. Spark Architecture: Java Knowledge Transfers**

### **Spark Internals (Same in Java & Python)**
```python
# Java developers: You ALREADY know this architecture!
# Let's map Java Spark concepts to PySpark

"""
┌─────────────────────────────────────────────────┐
│         SPARK ARCHITECTURE (SAME FOR ALL)       │
├─────────────────────────────────────────────────┤
│                                                 │
│    Driver Program (JVM)                         │
│    ├─ SparkContext (Java)                       │
│    ├─ RDD/Dataset API (Java)                    │
│    └─ Scheduler (Java)                          │
│                                                 │
│    Executors (JVMs on Workers)                  │
│    ├─ Task execution (Java)                     │
│    ├─ Memory management (Java)                  │
│    └─ Shuffle operations (Java)                 │
│                                                 │
│    Python Layer (Py4J Bridge)                   │
│    ├─ Python functions → Java objects           │
│    ├─ Data serialization (pickle)               │
│    └─ Result collection                         │
└─────────────────────────────────────────────────┘

KEY INSIGHT: PySpark = Python wrapper over Java Spark
Your Java Spark optimization knowledge APPLIES DIRECTLY!
"""
```

### **Setting Up PySpark Like a Pro**
```python
# Unlike Java's pom.xml, Python setup is simpler
# Install: pip install pyspark

# Production-grade Spark session (your Java config knowledge helps!)
from pyspark.sql import SparkSession
from pyspark import SparkConf

# Create config like Java SparkConf
conf = SparkConf() \
    .setAppName("ProductionApp") \
    .setMaster("spark://master:7077") \
    .set("spark.executor.memory", "4g") \
    .set("spark.driver.memory", "2g") \
    .set("spark.sql.shuffle.partitions", "200") \
    .set("spark.executor.cores", "2") \
    .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer") \
    .set("spark.kryo.registrator", "com.example.MyRegistrator") \
    .set("spark.sql.adaptive.enabled", "true")  # Your Java tuning skills matter!

# Create session
spark = SparkSession.builder \
    .config(conf=conf) \
    .enableHiveSupport() \
    .getOrCreate()

# Check Java version (yes, it's Java underneath!)
print(f"Java version: {spark._jvm.java.lang.System.getProperty('java.version')}")
print(f"Spark version: {spark.version}")
```

## 🔄 **2. DataFrames: Java Dataset API in Python**

### **Creating DataFrames (Like Java)**
```python
from pyspark.sql.types import *
from pyspark.sql import Row
import pandas as pd

# Method 1: From RDD (like Java)
data = [("Alice", 34), ("Bob", 45), ("Charlie", 29)]
rdd = spark.sparkContext.parallelize(data)
df_from_rdd = rdd.toDF(["name", "age"])

# Method 2: With schema (like Java's StructType)
schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("salary", DoubleType(), True),
    StructField("department", StringType(), True)
])

data = [(1, "Alice", 75000.0, "Engineering"),
        (2, "Bob", 65000.0, "Sales"),
        (3, "Charlie", 82000.0, "Engineering")]

df = spark.createDataFrame(data, schema=schema)
df.show()
df.printSchema()

# Method 3: From Pandas (Python-specific advantage!)
pandas_df = pd.DataFrame({
    'id': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie'],
    'salary': [75000, 65000, 82000]
})

spark_df = spark.createDataFrame(pandas_df)  # Easy conversion!
```

### **Transformations (Same as Java!)**
```python
from pyspark.sql.functions import *
from pyspark.sql.window import Window

# SELECT (like Java)
df.select("name", "salary").show()

# FILTER/WHERE (like Java)
df.filter(col("salary") > 70000).show()
df.where((col("department") == "Engineering") & (col("salary") > 70000)).show()

# GROUP BY (like Java)
df.groupBy("department") \
  .agg(
      avg("salary").alias("avg_salary"),
      count("*").alias("employee_count"),
      sum("salary").alias("total_salary")
  ).show()

# JOIN (same syntax as Java!)
employees = spark.createDataFrame([
    (1, "Alice", 100),
    (2, "Bob", 200),
    (3, "Charlie", 100)
], ["id", "name", "dept_id"])

departments = spark.createDataFrame([
    (100, "Engineering"),
    (200, "Sales")
], ["dept_id", "dept_name"])

# Inner join
joined = employees.join(departments, "dept_id", "inner")
joined.show()

# Window functions (like Java)
window_spec = Window.partitionBy("department").orderBy(col("salary").desc())

df.withColumn("rank", rank().over(window_spec)) \
  .withColumn("dense_rank", dense_rank().over(window_spec)) \
  .withColumn("row_number", row_number().over(window_spec)) \
  .show()
```

## 📈 **3. Performance Optimization: Java Skills Transfer**

### **Understanding PySpark Performance**
```python
# Java developers: You already know these concepts!
# Let's see them in Python

class PySparkOptimizer:
    """Optimization techniques that work in both Java and Python"""
    
    @staticmethod
    def optimize_dataframe(df):
        """Apply Java Spark optimizations to PySpark"""
        
        # 1. Partitioning (same as Java)
        df = df.repartition(200, "department")  # Like coalesce/repartition in Java
        
        # 2. Caching (same as Java)
        df.cache()  # persist() with storage level options available
        # df.persist(StorageLevel.MEMORY_AND_DISK)  # Java knowledge helps!
        
        # 3. Broadcast join for small tables (same as Java)
        from pyspark.sql.functions import broadcast
        small_df = df.filter(col("department") == "Engineering")
        result = df.join(broadcast(small_df), "id")
        
        # 4. Adaptive Query Execution (AQE) - Spark 3.0+ (same in Java/Python)
        # Already enabled in our config: spark.sql.adaptive.enabled=true
        
        # 5. Query planning (EXPLAIN - same as Java)
        df.explain("extended")  # Shows physical/logical plans
        
        return df
    
    @staticmethod
    def monitor_performance(spark):
        """Access Spark UI metrics (Java SparkContext underneath)"""
        # Spark UI is Java-based, accessible at http://localhost:4040
        # Metrics are the same
        
        # Get Java SparkContext
        sc = spark.sparkContext
        
        # Access Java metrics (same as Java API)
        print(f"Application ID: {sc.applicationId}")
        print(f"Application UI: {sc.uiWebUrl}")
        
        # Stage metrics
        status = sc.statusTracker()
        print(f"Active jobs: {len(status.getActiveJobsIds())}")
        print(f"Active stages: {len(status.getActiveStageIds())}")
```

### **Serialization: Kryo vs Pickle**
```python
# Critical: PySpark has EXTRA serialization overhead!
# Java: Objects serialized between JVMs
# PySpark: Python → Pickle → JVM → Java Serialization → JVM → Pickle → Python

from pyspark import SparkContext
import pickle

class OptimizedSerialization:
    """Minimize serialization overhead"""
    
    def __init__(self):
        self.sc = SparkContext.getOrCreate()
    
    def avoid_serialization_overhead(self):
        """Techniques to reduce Python-Java serialization"""
        
        # BAD: Lambda with complex Python objects (heavy serialization)
        rdd = self.sc.parallelize(range(100))
        result = rdd.map(lambda x: complex_function(x))  # Serializes entire function
        
        # BETTER: Use built-in functions (executed in Java)
        result = rdd.map(lambda x: x * 2)  # Simple lambdas are okay
        
        # BEST: Use DataFrame API (stays in Java as much as possible)
        df = spark.range(100)
        result = df.select(col("id") * 2)  # Entirely in Java!
        
        # Use Pandas UDFs for vectorized operations
        from pyspark.sql.functions import pandas_udf
        from pyspark.sql.types import IntegerType
        
        @pandas_udf(IntegerType())
        def vectorized_multiply(s: pd.Series) -> pd.Series:
            return s * 2  # Executed in Python but vectorized
        
        df.select(vectorized_multiply(col("id"))).show()
    
    def custom_serialization(self):
        """Implement custom serializers (like Java)"""
        # Register custom Kryo serializer
        conf = SparkConf()
        conf.set("spark.kryo.registrator", "com.example.MyKryoRegistrator")
        
        # Or use PySpark's built-in optimization
        conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
        conf.set("spark.kryo.classesToRegister", "com.example.MyClass")
```

## 🏗️ **4. PySpark Architecture Patterns**

### **Pattern 1: Java-Style Spark Applications**
```python
# Writing PySpark like a Java developer (structured, maintainable)

from abc import ABC, abstractmethod
from typing import List, Dict, Any
from dataclasses import dataclass
import logging

@dataclass
class SparkConfig:
    """Configuration DTO (like Java POJO)"""
    app_name: str
    master: str = "local[*]"
    executor_memory: str = "2g"
    driver_memory: str = "1g"
    shuffle_partitions: int = 200

class BaseSparkJob(ABC):
    """Abstract base class for Spark jobs (Java-style OOP)"""
    
    def __init__(self, config: SparkConfig):
        self.config = config
        self.spark = self._create_spark_session()
        self.logger = logging.getLogger(self.__class__.__name__)
    
    def _create_spark_session(self):
        """Factory method for Spark session"""
        return SparkSession.builder \
            .appName(self.config.app_name) \
            .master(self.config.master) \
            .config("spark.executor.memory", self.config.executor_memory) \
            .config("spark.driver.memory", self.config.driver_memory) \
            .config("spark.sql.shuffle.partitions", self.config.shuffle_partitions) \
            .getOrCreate()
    
    @abstractmethod
    def extract(self):
        """Extract data (like ETL)"""
        pass
    
    @abstractmethod
    def transform(self, df):
        """Transform data"""
        pass
    
    @abstractmethod
    def load(self, df):
        """Load results"""
        pass
    
    def execute(self):
        """Template method pattern (common in Java)"""
        try:
            self.logger.info(f"Starting job: {self.config.app_name}")
            
            # ETL pipeline
            extracted = self.extract()
            transformed = self.transform(extracted)
            self.load(transformed)
            
            self.logger.info("Job completed successfully")
            return True
        except Exception as e:
            self.logger.error(f"Job failed: {e}")
            raise
        finally:
            self.spark.stop()


class UserAnalyticsJob(BaseSparkJob):
    """Concrete implementation (like Java service)"""
    
    def extract(self):
        self.logger.info("Extracting user data...")
        return self.spark.read \
            .format("parquet") \
            .load("/data/users/*.parquet")
    
    def transform(self, df):
        self.logger.info("Transforming user data...")
        
        # Complex transformations
        result = df \
            .filter(col("active") == True) \
            .groupBy("department", "country") \
            .agg(
                count("*").alias("user_count"),
                avg("salary").alias("avg_salary"),
                sum("revenue").alias("total_revenue")
            ) \
            .withColumn("revenue_per_user", col("total_revenue") / col("user_count"))
        
        return result
    
    def load(self, df):
        self.logger.info("Loading results...")
        df.write \
            .mode("overwrite") \
            .format("parquet") \
            .save("/output/user_analytics")


# Usage (like Java main method)
if __name__ == "__main__":
    config = SparkConfig(
        app_name="UserAnalytics",
        master="spark://cluster:7077",
        executor_memory="4g"
    )
    
    job = UserAnalyticsJob(config)
    job.execute()
```

### **Pattern 2: Functional-Style PySpark (Pythonic)**
```python
# More Pythonic approach using functional programming

from typing import Callable
from functools import reduce
from pyspark.sql import DataFrame

# Define transformations as pure functions
def filter_active_users(df: DataFrame) -> DataFrame:
    return df.filter(col("active") == True)

def add_derived_columns(df: DataFrame) -> DataFrame:
    return df.withColumn("years_of_service", 
                        year(current_date()) - year(col("hire_date"))) \
             .withColumn("salary_band",
                        when(col("salary") < 50000, "Low")
                         .when(col("salary") < 100000, "Medium")
                         .otherwise("High"))

def aggregate_by_department(df: DataFrame) -> DataFrame:
    return df.groupBy("department") \
             .agg(
                 count("*").alias("count"),
                 avg("salary").alias("avg_salary"),
                 collect_list("name").alias("employees")
             )

# Pipeline builder
class SparkPipeline:
    """Functional pipeline (like Java Streams)"""
    
    def __init__(self):
        self.transformations: List[Callable[[DataFrame], DataFrame]] = []
    
    def add_transformation(self, func: Callable[[DataFrame], DataFrame]):
        self.transformations.append(func)
        return self
    
    def execute(self, df: DataFrame) -> DataFrame:
        # Chain transformations
        return reduce(lambda d, f: f(d), self.transformations, df)

# Build and execute pipeline
pipeline = SparkPipeline() \
    .add_transformation(filter_active_users) \
    .add_transformation(add_derived_columns) \
    .add_transformation(aggregate_by_department)

result = pipeline.execute(users_df)
```

## 🔧 **5. Testing PySpark (Like Java Unit Tests)**

### **Testing Strategies**
```python
# Java: JUnit, Mockito, Spark Testing Base
# PySpark: pytest, chispa, pyspark-test

import pytest
from pyspark_test import assert_pyspark_df_equal
import chispa

class TestUserAnalyticsJob:
    """Unit tests for PySpark jobs"""
    
    @pytest.fixture(scope="session")
    def spark(self):
        """Test Spark session (like @BeforeAll)"""
        return SparkSession.builder \
            .master("local[2]") \
            .appName("test") \
            .getOrCreate()
    
    @pytest.fixture
    def sample_data(self, spark):
        """Test data (like @BeforeEach)"""
        data = [
            (1, "Alice", 75000, "Engineering", True),
            (2, "Bob", 65000, "Sales", False),
            (3, "Charlie", 82000, "Engineering", True)
        ]
        
        schema = ["id", "name", "salary", "department", "active"]
        return spark.createDataFrame(data, schema)
    
    def test_filter_active_users(self, sample_data):
        """Test transformation (like @Test)"""
        from my_job import filter_active_users
        
        result = filter_active_users(sample_data)
        
        # Assertions
        assert result.count() == 2
        assert result.filter(col("active") == False).count() == 0
        
        # Compare DataFrames
        expected_data = [
            (1, "Alice", 75000, "Engineering", True),
            (3, "Charlie", 82000, "Engineering", True)
        ]
        expected = sample_data.sparkSession.createDataFrame(expected_data, 
                                                          sample_data.schema)
        
        # Use chispa for DataFrame comparison
        chispa.assert_df_equality(result, expected)
    
    def test_aggregate_by_department(self, sample_data):
        """Test aggregation"""
        from my_job import aggregate_by_department
        
        filtered = sample_data.filter(col("active") == True)
        result = aggregate_by_department(filtered)
        
        # Check aggregation results
        engineering_dept = result.filter(col("department") == "Engineering").first()
        assert engineering_dept["count"] == 2
        assert engineering_dept["avg_salary"] == 78500.0
    
    @pytest.mark.integration
    def test_end_to_end(self, spark, tmp_path):
        """Integration test (like SpringBootTest)"""
        # Create test data
        test_data_path = tmp_path / "test_data.parquet"
        df = spark.createDataFrame([(1, "Alice"), (2, "Bob")], ["id", "name"])
        df.write.parquet(str(test_data_path))
        
        # Run job
        job = UserAnalyticsJob(SparkConfig("TestJob", master="local[*]"))
        
        # Mock methods for testing
        job.extract = lambda: spark.read.parquet(str(test_data_path))
        
        # Execute
        result = job.execute()
        
        assert result is True
```

### **Mocking Spark (Like Mockito)**
```python
from unittest.mock import Mock, MagicMock, patch
import pytest

def test_spark_job_with_mocks():
    """Mocking Spark components (like Mockito in Java)"""
    
    # Mock SparkSession
    mock_spark = Mock()
    mock_df = Mock()
    
    # Configure mocks
    mock_spark.read.format.return_value.load.return_value = mock_df
    mock_df.filter.return_value = mock_df
    mock_df.groupBy.return_value.agg.return_value = mock_df
    mock_df.write.mode.return_value.format.return_value.save.return_value = None
    
    # Patch the SparkSession creation
    with patch('pyspark.sql.SparkSession.builder.getOrCreate', 
               return_value=mock_spark):
        
        # Create and execute job
        job = UserAnalyticsJob(SparkConfig("Test"))
        job.execute()
        
        # Verify interactions (like Mockito.verify)
        mock_spark.read.format.assert_called_with("parquet")
        mock_df.filter.assert_called()
        mock_df.write.mode.assert_called_with("overwrite")
```

## 🚀 **6. Production Deployment Patterns**

### **Pattern 1: Spark Submit (Like Java JARs)**
```bash
# Java: spark-submit --class com.example.Main --master yarn app.jar
# PySpark: spark-submit --master yarn app.py

# Submit Python script
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --num-executors 10 \
  --executor-memory 4g \
  --driver-memory 2g \
  --conf spark.sql.shuffle.partitions=200 \
  my_spark_job.py

# With dependencies (like Java uber-jar)
spark-submit \
  --master yarn \
  --py-files dependencies.zip \
  --files config.json \
  main.py

# Submit with virtual environment (Python-specific)
spark-submit \
  --master yarn \
  --archives venv.tar.gz#venv \
  --conf spark.pyspark.python=./venv/bin/python \
  job.py
```

### **Pattern 2: Packaging as Python Package**
```python
# Structure like Java Maven project
"""
my-spark-project/
├── pyproject.toml           # Like pom.xml
├── src/
│   └── myspark/
│       ├── __init__.py
│       ├── jobs/
│       │   ├── __init__.py
│       │   ├── etl_job.py
│       │   └── analytics_job.py
│       └── utils/
│           ├── __init__.py
│           └── spark_utils.py
├── tests/
│   └── test_jobs.py
└── scripts/
    └── run_job.py
"""

# pyproject.toml (like pom.xml)
"""
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[project]
name = "my-spark-project"
version = "1.0.0"
dependencies = [
    "pyspark>=3.4.0",
    "pandas>=2.0.0",
]

[project.scripts]
run-etl = "myspark.scripts.run_job:main"
"""

# Entry point (like Java main class)
# scripts/run_job.py
def main():
    import argparse
    from myspark.jobs.etl_job import ETLJob
    
    parser = argparse.ArgumentParser()
    parser.add_argument("--input", required=True)
    parser.add_argument("--output", required=True)
    args = parser.parse_args()
    
    job = ETLJob(input_path=args.input, output_path=args.output)
    job.run()

if __name__ == "__main__":
    main()
```

## 📊 **7. Real-World Example: Java-PySpark Integration**

### **Scenario: Migrating Java Spark Job to PySpark**
```java
// Original Java Spark Job
public class UserAnalyticsJava {
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
            .appName("UserAnalytics")
            .getOrCreate();
        
        Dataset<Row> users = spark.read()
            .parquet("/data/users/*.parquet");
        
        Dataset<Row> result = users
            .filter(col("active").equalTo(true))
            .groupBy(col("department"))
            .agg(
                avg(col("salary")).as("avg_salary"),
                count("*").as("count")
            );
        
        result.write()
            .mode(SaveMode.Overwrite)
            .parquet("/output/analytics");
    }
}
```

```python
# PySpark equivalent (YOUR EXISTING KNOWLEDGE APPLIES!)
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg, count

class UserAnalyticsPython:
    def __init__(self):
        self.spark = SparkSession.builder \
            .appName("UserAnalytics") \
            .getOrCreate()
    
    def run(self):
        # SAME LOGIC, Python syntax!
        users = self.spark.read.parquet("/data/users/*.parquet")
        
        result = users \
            .filter(col("active") == True) \
            .groupBy("department") \
            .agg(
                avg("salary").alias("avg_salary"),
                count("*").alias("count")
            )
        
        result.write \
            .mode("overwrite") \
            .parquet("/output/analytics")
        
        return result

# The transformation is 1:1!
# Your Java optimization knowledge still applies:
# - Partitioning strategies ✓
# - Memory tuning ✓  
# - Shuffle optimization ✓
# - Serialization settings ✓
```

### **Hybrid Approach: Java + PySpark**
```python
# Best of both worlds: Java for heavy lifting, Python for data science

from pyspark.sql import SparkSession
import jpype
import jpype.imports

# Option 1: Call Java libraries from PySpark
class JavaIntegration:
    """Use existing Java code in PySpark"""
    
    def __init__(self):
        # Start JVM (your Java code runs here!)
        jpype.startJVM(classpath=['/path/to/java.jar'])
        
        # Import Java classes
        from java.util import ArrayList
        from com.example import JavaProcessor
        
        self.java_processor = JavaProcessor()
    
    def process_with_java(self, data):
        """Use Java for complex business logic"""
        # Convert Python data to Java
        java_list = ArrayList()
        for item in data:
            java_list.add(item)
        
        # Process in Java (fast, optimized)
        result = self.java_processor.process(java_list)
        
        # Convert back to Python
        return list(result)
    
    def py_spark_pipeline(self):
        """PySpark pipeline using Java libraries"""
        spark = SparkSession.builder.getOrCreate()
        
        # Read data with PySpark
        df = spark.read.parquet("/data/*.parquet")
        
        # Use Java UDF for complex logic
        from pyspark.sql.functions import udf
        from pyspark.sql.types import StringType
        
        @udf(StringType())
        def java_udf(value):
            # Call Java code from Python UDF
            return self.java_processor.transform(value)
        
        result = df.withColumn("processed", java_udf(col("raw_data")))
        return result

# Option 2: PySpark calls REST API of Java service
import requests
import json
from pyspark.sql.functions import udf
from pyspark.sql.types import MapType, StringType

class JavaServiceIntegration:
    """Call Java microservices from PySpark"""
    
    @staticmethod
    @udf(MapType(StringType(), StringType()))
    def call_java_service(data):
        """UDF that calls Java REST API"""
        response = requests.post(
            "http://java-service:8080/api/process",
            json={"data": data},
            headers={"Content-Type": "application/json"}
        )
        return response.json()
    
    def enrich_data(self, df):
        """Enrich PySpark data with Java service"""
        return df.withColumn("enriched", self.call_java_service(col("raw_data")))
```

## 🎯 **Key Takeaways for Java Developers:**

### **What Transfers Directly:**
1. **Spark Architecture**: Same JVM-based architecture
2. **Optimization Techniques**: Partitioning, caching, broadcast joins
3. **Performance Tuning**: Memory, serialization, shuffle settings
4. **Deployment Patterns**: spark-submit, cluster modes
5. **Monitoring**: Spark UI, metrics, logging

### **What's Different:**
1. **Syntax**: Python vs Java syntax
2. **Serialization**: Extra Python-Java serialization layer
3. **Libraries**: Python data science libraries (Pandas, NumPy)
4. **Development Speed**: Faster prototyping in Python

### **Performance Comparison:**
```python
# Rule of thumb for Java developers:
"""
Java Spark: 100% performance, more verbose
PySpark: 90% performance, 50% less code

For ETL: Consider PySpark (faster development)
For CPU-intensive: Consider Java Spark (better performance)
For ML: Definitely PySpark (ML libraries)
"""
```

## 📚 **Homework: Convert Your Java Spark Job**

Take an existing Java Spark job and:
1. **Rewrite it in PySpark** (80% will be direct translation)
2. **Add Python-specific optimizations** (Pandas UDFs, vectorization)
3. **Compare performance** (you'll be surprised how close it is!)
4. **Measure development time** (Python is faster to write)

## 🚀 **Next in Part 5.2: Airflow & Pipeline Orchestration**

We'll cover:
- **Apache Airflow** vs Java schedulers (Quartz, Spring Batch)
- **Building data pipelines** that integrate Java and Python
- **Monitoring and alerting** for data pipelines
- **Real-time streaming** with Python (alternative to Kafka Streams)

---