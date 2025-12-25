---
title: >-
  Python for Java Developers: Modern Data Pipeline Orchestration & Streaming -
  VII
date: 2025-12-24 08:45:50
categories:
- Python
tags:
- Python
---

## 🎯 **Chapater 7: Modern Data Pipeline Orchestration & Streaming**

### **Why This is CRITICAL Now:**

1. **Real Production Need**: You know PySpark, but how do you **orchestrate** and **operationalize** it?
2. **Market Gap**: Many data engineers know tools but not **production orchestration**
3. **Java Background**: You understand **scheduling, reliability, monitoring** from Java world
4. **Career Multiplier**: Data pipeline orchestration is **high-demand, high-salary**

---

## 📊 **From Batch Jobs to Production Data Pipelines**

### **The Evolution You Need:**
```
Java Developer (Traditional)        →       Data Engineer (Modern)
─────────────────────────────────────────────────────────────────────
Spring Batch Jobs                   →       Apache Airflow DAGs
Quartz Scheduler                    →       Prefect/ Dagster
Cron Jobs                          →       Event-driven pipelines
Custom Monitoring                   →       Built-in observability
Manual Recovery                    →       Auto-retry, alerting
```

## 🚀 **Chapter Outline: Orchestration & Streaming**

### **1. Apache Airflow: The Industry Standard**
```python
# Compare: Spring Batch vs Airflow
# Java: @Scheduled, JobLauncher, Step, Chunk-oriented processing
# Python: DAGs, Operators, Sensors, XComs

from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from airflow.sensors.filesystem import FileSensor
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from datetime import datetime, timedelta
from typing import Dict, List
import json

# DAG Definition (like Spring @Configuration)
default_args = {
    'owner': 'java_developer',
    'depends_on_past': False,
    'email': ['alerts@company.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'execution_timeout': timedelta(hours=2),
}

dag = DAG(
    'user_data_pipeline',
    default_args=default_args,
    description='Production data pipeline for user analytics',
    schedule_interval='0 2 * * *',  # Daily at 2 AM (like cron)
    start_date=datetime(2024, 1, 1),
    catchup=False,  # Don't backfill automatically
    tags=['production', 'analytics', 'users'],
    max_active_runs=1,
)

# Sensors (like waiting for dependencies)
wait_for_data = FileSensor(
    task_id='wait_for_user_data',
    filepath='/data/lake/raw/users/{{ ds }}.parquet',  # {{ ds }} = execution date
    poke_interval=300,  # Check every 5 minutes
    timeout=3600,  # Timeout after 1 hour
    mode='reschedule',
    dag=dag,
)

# Spark Job (your PySpark code!)
spark_job = SparkSubmitOperator(
    task_id='process_user_data',
    application='/opt/airflow/dags/spark_jobs/user_analytics.py',
    conn_id='spark_default',
    application_args=[
        '--input', '/data/lake/raw/users/{{ ds }}.parquet',
        '--output', '/data/lake/processed/users/{{ ds }}',
        '--date', '{{ ds }}'
    ],
    executor_memory='4g',
    driver_memory='2g',
    num_executors=4,
    dag=dag,
)

# Python function (business logic)
def validate_results(**context):
    """Validate processing results (like PostProcessor in Spring Batch)"""
    import pandas as pd
    from airflow.models import Variable
    
    date = context['ds']
    output_path = f'/data/lake/processed/users/{date}'
    
    # Read results
    df = pd.read_parquet(output_path)
    
    # Business rules validation
    assert df['user_id'].is_unique, "Duplicate user IDs found"
    assert df['age'].between(0, 120).all(), "Invalid age values"
    assert (df['signup_date'] <= pd.Timestamp.now()).all(), "Future signup dates"
    
    # Log metrics
    context['ti'].xcom_push(key='processed_count', value=len(df))
    context['ti'].xcom_push(key='avg_age', value=df['age'].mean())
    
    # Send to monitoring
    import requests
    requests.post(
        'http://monitoring:9090/metrics',
        json={'processed_users': len(df), 'pipeline': 'user_analytics'}
    )

validate_task = PythonOperator(
    task_id='validate_results',
    python_callable=validate_results,
    provide_context=True,
    dag=dag,
)

# Email notification (like Java's ApplicationEvent)
send_notification = EmailOperator(
    task_id='send_success_notification',
    to='data-team@company.com',
    subject='User Pipeline Completed - {{ ds }}',
    html_content='''
    <h3>User Data Pipeline Success</h3>
    <p>Date: {{ ds }}</p>
    <p>Processed: {{ ti.xcom_pull(task_ids="validate_results", key="processed_count") }} users</p>
    <p>Average Age: {{ ti.xcom_pull(task_ids="validate_results", key="avg_age") | round(2) }}</p>
    ''',
    dag=dag,
)

# Dependencies (like Spring Batch flow)
wait_for_data >> spark_job >> validate_task >> send_notification
```

### **2. Modern Alternatives: Prefect & Dagster**
```python
# Compare: Spring Boot vs Prefect/Dagster
# Java: @Scheduled, @EventListener, @Transactional
# Python: @flow, @task, dependencies, versioning

# ========== PREFECT 2.0 ==========
from prefect import flow, task, get_run_logger
from prefect.task_runners import ConcurrentTaskRunner
from typing import List
import asyncio
import httpx

@task(retries=3, retry_delay_seconds=10)
def extract_users(date: str) -> List[Dict]:
    """Extract user data (like @Service)"""
    logger = get_run_logger()
    logger.info(f"Extracting users for {date}")
    
    # Call Java REST API
    async def fetch():
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"http://java-service:8080/api/users?date={date}"
            )
            return response.json()
    
    return asyncio.run(fetch())

@task(cache_key_fn=lambda *args, **kwargs: kwargs.get("date"))
def transform_users(users: List[Dict], date: str) -> pd.DataFrame:
    """Transform with caching (like @Cacheable)"""
    df = pd.DataFrame(users)
    
    # Data validation (like Bean Validation)
    from pydantic import BaseModel, validator
    from typing import Optional
    
    class User(BaseModel):
        id: int
        name: str
        email: str
        age: Optional[int] = None
        
        @validator('age')
        def validate_age(cls, v):
            if v and (v < 0 or v > 120):
                raise ValueError(f"Invalid age: {v}")
            return v
    
    # Validate each user
    validated_users = []
    for user in users:
        try:
            validated = User(**user)
            validated_users.append(validated.dict())
        except Exception as e:
            print(f"Invalid user {user.get('id')}: {e}")
    
    return pd.DataFrame(validated_users)

@task
def load_to_warehouse(df: pd.DataFrame, date: str):
    """Load to data warehouse"""
    # Write to BigQuery/Snowflake/etc.
    df.to_parquet(f"s3://data-warehouse/users/{date}.parquet")
    
    # Also update cache
    import redis
    r = redis.Redis(host='redis', port=6379)
    r.set(f"users:{date}:count", len(df))

@flow(
    name="user_analytics_pipeline",
    description="Daily user data pipeline",
    version="production",
    task_runner=ConcurrentTaskRunner(),
    retries=2,
    retry_delay_seconds=30,
)
def user_pipeline(date: str = None):
    """Main flow (like @SpringBootApplication)"""
    if not date:
        date = datetime.now().strftime("%Y-%m-%d")
    
    # Extract → Transform → Load
    raw_users = extract_users(date)
    transformed = transform_users(raw_users, date=date)
    load_to_warehouse(transformed, date=date)
    
    return len(transformed)

# ========== DAGSTER ==========
from dagster import job, op, repository, schedule, sensor, AssetMaterialization
from dagster_aws.s3 import s3_resource
from dagster_spark import spark_resource

@op(required_resource_keys={"spark", "s3"})
def process_users_spark(context):
    """Spark processing in Dagster"""
    spark = context.resources.spark.spark_session
    
    df = spark.read.parquet("s3://raw-data/users/")
    result = df.filter("active = true").groupBy("department").count()
    
    output_path = "s3://processed-data/users/"
    result.write.parquet(output_path)
    
    # Asset tracking (Dagster's killer feature)
    yield AssetMaterialization(
        asset_key="user_analytics",
        description="Processed user analytics",
        metadata={"row_count": result.count(), "path": output_path}
    )
    
    return output_path

@job(resource_defs={"spark": spark_resource, "s3": s3_resource})
def user_analytics_job():
    process_users_spark()

# Schedule
@schedule(cron_schedule="0 2 * * *", job=user_analytics_job)
def daily_schedule():
    return {}

# Sensor for file arrival
@sensor(job=user_analytics_job)
def file_sensor(context):
    """Wait for new files (like @EventListener)"""
    import boto3
    s3 = boto3.client('s3')
    
    files = s3.list_objects_v2(Bucket='raw-data', Prefix='users/')
    new_files = [f for f in files.get('Contents', []) 
                if f['LastModified'].date() == datetime.now().date()]
    
    if new_files:
        yield RunRequest(run_key=str(datetime.now()))
```

### **3. Real-time Streaming: Beyond Kafka Streams**
```python
# Compare: Kafka Streams (Java) vs Python Streaming
# Java: Kafka Streams DSL, Processor API, State Stores
# Python: Faust, Bytewax, Quix, Spark Structured Streaming

import faust
from dataclasses import dataclass, asdict
import json
from datetime import datetime
from typing import Optional
import pandas as pd

@dataclass
class UserEvent:
    """Event schema (like Avro/Pojo)"""
    user_id: int
    event_type: str  # "login", "purchase", "view"
    amount: Optional[float] = None
    timestamp: str = None
    
    def __post_init__(self):
        if not self.timestamp:
            self.timestamp = datetime.now().isoformat()

# Faust App (Python alternative to Kafka Streams)
app = faust.App(
    'user-events-processor',
    broker='kafka://localhost:9092',
    store='rocksdb://',  # Local state store (like Kafka Streams)
    web_port=6066,  # Web UI
    version=1,
    autodiscover=True,
)

# Topics
raw_events_topic = app.topic('user-events-raw', value_type=UserEvent)
processed_events_topic = app.topic('user-events-processed')
fraud_alerts_topic = app.topic('fraud-alerts')

# Tables (like Kafka Streams KTables)
user_sessions = app.Table('user-sessions', default=int)
user_totals = app.Table('user-totals', default=float)

@app.agent(raw_events_topic)
async def process_user_events(events):
    """Process stream (like Kafka Streams Processor)"""
    async for event in events:
        # Update session count
        user_sessions[event.user_id] += 1
        
        # Update totals if purchase
        if event.event_type == 'purchase' and event.amount:
            user_totals[event.user_id] += event.amount
            
            # Fraud detection (stateful)
            if user_totals[event.user_id] > 10000:
                await fraud_alerts_topic.send(
                    value={
                        'user_id': event.user_id,
                        'total_spent': user_totals[event.user_id],
                        'alert_type': 'high_spending',
                        'timestamp': event.timestamp
                    }
                )
        
        # Enrich event
        enriched = {
            **asdict(event),
            'session_count': user_sessions[event.user_id],
            'total_spent': user_totals[event.user_id],
            'processed_at': datetime.now().isoformat()
        }
        
        # Send to output topic
        await processed_events_topic.send(value=enriched)

# Windowed aggregations (like Kafka Streams windows)
@app.timer(interval=60.0)
async def emit_minute_aggregates():
    """Emit minute aggregates (like windowed operations)"""
    aggregates = {}
    
    for user_id, total in user_totals.items():
        aggregates[user_id] = {
            'minute_total': total,
            'timestamp': datetime.now().isoformat()
        }
    
    # Send to another topic
    await app.topic('minute-aggregates').send(value=aggregates)

# ========== BYTEWAX (Rust-powered, like Flink) ==========
import bytewax.operators as op
from bytewax.connectors.kafka import KafkaSource, KafkaSink
from bytewax.dataflow import Dataflow
from bytewax.operators.window import EventClock, TumblingWindow

flow = Dataflow("user_analytics")

# Source (like Flink/Kafka Source)
source = op.input(
    "kafka_source",
    flow,
    KafkaSource(
        brokers=["localhost:9092"],
        topics=["user-events"],
        starting_offset="beginning",
    )
)

# Parse JSON
def parse_event(event):
    return json.loads(event.value.decode('utf-8'))

parsed = op.map("parse_json", source, parse_event)

# Key by user (like keyBy in Flink)
def key_by_user(event):
    return str(event['user_id']), event

keyed = op.key_on("key_by_user", parsed, key_by_user)

# Windowed aggregation
clock = EventClock(
    lambda event: datetime.fromisoformat(event['timestamp']),
    wait_for_system_duration=timedelta(seconds=10)
)

window = TumblingWindow(length=timedelta(minutes=5))

windowed = op.window.collect_window("windowed", keyed, clock, window)

# Aggregate in windows
def calculate_stats(events):
    amounts = [e.get('amount', 0) for e in events if e.get('amount')]
    return {
        'user_id': events[0]['user_id'],
        'window_start': events[0]['timestamp'],
        'count': len(events),
        'total_amount': sum(amounts),
        'avg_amount': sum(amounts) / len(amounts) if amounts else 0
    }

aggregated = op.map_value("calculate_stats", windowed, calculate_stats)

# Sink (like Flink Sink)
sink = op.output(
    "kafka_sink",
    aggregated,
    KafkaSink(
        brokers=["localhost:9092"],
        topic="user-aggregates"
    )
)
```

### **4. Data Quality & Testing Pipeline**
```python
# Compare: Java Unit Tests vs Data Pipeline Tests
# Java: JUnit, Mockito, TestContainers
# Python: pytest, Great Expectations, dbt tests

import pytest
from great_expectations.core import ExpectationSuite
from great_expectations.dataset import SparkDFDataset
import dbt
from dbt.contracts.results import TestResult

class DataPipelineTests:
    """Testing data pipelines (like integration tests)"""
    
    def test_data_quality(self, spark):
        """Test data quality (like @SpringBootTest)"""
        # Load data
        df = spark.read.parquet("/data/users/*.parquet")
        
        # Convert to Great Expectations dataset
        ge_df = SparkDFDataset(df)
        
        # Define expectations (like assertions)
        expectations = {
            "user_id_unique": ge_df.expect_column_values_to_be_unique("user_id"),
            "email_format": ge_df.expect_column_values_to_match_regex(
                "email", r"^[^@]+@[^@]+\.[^@]+$"
            ),
            "age_range": ge_df.expect_column_values_to_be_between(
                "age", min_value=0, max_value=120
            ),
            "no_null_emails": ge_df.expect_column_values_to_not_be_null("email"),
            "row_count": ge_df.expect_table_row_count_to_be_between(
                min_value=1000, max_value=1000000
            )
        }
        
        # Validate
        validation_results = []
        for name, result in expectations.items():
            assert result.success, f"Failed: {name} - {result.result}"
            validation_results.append(result)
        
        return validation_results
    
    def test_pipeline_integration(self):
        """End-to-end test (like @SpringBootTest)"""
        # Use testcontainers for integration testing
        from testcontainers.kafka import KafkaContainer
        from testcontainers.postgres import PostgresContainer
        
        with KafkaContainer() as kafka:
            with PostgresContainer("postgres:15") as postgres:
                # Run pipeline with test containers
                pipeline = UserPipeline(
                    kafka_broker=kafka.get_bootstrap_server(),
                    database_url=postgres.get_connection_url()
                )
                
                # Process test data
                result = pipeline.process(test_data)
                
                # Verify results
                assert result.success
                assert result.processed_count > 0
                
                # Verify in database
                import psycopg2
                conn = psycopg2.connect(postgres.get_connection_url())
                cursor = conn.cursor()
                cursor.execute("SELECT COUNT(*) FROM users")
                count = cursor.fetchone()[0]
                assert count == len(test_data)
    
    @pytest.mark.performance
    def test_pipeline_performance(self):
        """Performance test (like JMH in Java)"""
        import time
        import pandas as pd
        
        # Generate test data
        test_size = 1000000
        test_data = pd.DataFrame({
            'user_id': range(test_size),
            'name': [f'User_{i}' for i in range(test_size)],
            'email': [f'user_{i}@example.com' for i in range(test_size)],
            'age': [i % 100 for i in range(test_size)]
        })
        
        # Run pipeline
        start_time = time.time()
        pipeline = UserPipeline()
        result = pipeline.process(test_data)
        end_time = time.time()
        
        # Assert performance SLAs
        processing_time = end_time - start_time
        assert processing_time < 300  # Less than 5 minutes
        
        throughput = test_size / processing_time
        assert throughput > 1000  # More than 1000 records/second
        
        return {
            'processing_time': processing_time,
            'throughput': throughput,
            'memory_usage': result.memory_usage
        }

# dbt Tests (SQL-based testing)
"""
-- tests/unique_user_id.sql
SELECT user_id, COUNT(*) as count
FROM {{ ref('users') }}
GROUP BY user_id
HAVING count > 1

-- tests/valid_email.sql  
SELECT email
FROM {{ ref('users') }}
WHERE email NOT LIKE '%@%.%'

-- Run: dbt test
"""
```

### **5. Monitoring & Observability for Pipelines**
```python
# Compare: Spring Boot Actuator vs Pipeline Monitoring
# Java: Micrometer, Prometheus, Grafana, ELK
# Python: OpenTelemetry, Prometheus, Airflow metrics, custom dashboards

import prometheus_client
from prometheus_client import Counter, Histogram, Gauge
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
import logging
from pythonjsonlogger import jsonlogger

class PipelineMonitoring:
    """Production monitoring for data pipelines"""
    
    def __init__(self, pipeline_name: str):
        self.pipeline_name = pipeline_name
        
        # Metrics (like Micrometer)
        self.records_processed = Counter(
            f'{pipeline_name}_records_processed_total',
            'Total records processed'
        )
        
        self.processing_duration = Histogram(
            f'{pipeline_name}_processing_duration_seconds',
            'Processing duration histogram',
            buckets=[0.1, 0.5, 1, 5, 10, 30, 60]
        )
        
        self.pipeline_errors = Counter(
            f'{pipeline_name}_errors_total',
            'Total pipeline errors',
            ['error_type']
        )
        
        self.active_pipelines = Gauge(
            'active_pipelines',
            'Number of active pipelines'
        )
        
        # Tracing (OpenTelemetry)
        trace.set_tracer_provider(TracerProvider())
        self.tracer = trace.get_tracer(__name__)
        
        # Jaeger exporter
        jaeger_exporter = JaegerExporter(
            agent_host_name="localhost",
            agent_port=6831,
        )
        span_processor = BatchSpanProcessor(jaeger_exporter)
        trace.get_tracer_provider().add_span_processor(span_processor)
        
        # Structured logging
        logger = logging.getLogger(pipeline_name)
        log_handler = logging.StreamHandler()
        formatter = jsonlogger.JsonFormatter(
            '%(asctime)s %(levelname)s %(name)s %(message)s'
        )
        log_handler.setFormatter(formatter)
        logger.addHandler(log_handler)
        self.logger = logger
    
    def instrument_pipeline(self, func):
        """Decorator to instrument pipeline functions"""
        def wrapper(*args, **kwargs):
            # Start span
            with self.tracer.start_as_current_span(func.__name__) as span:
                span.set_attribute("pipeline", self.pipeline_name)
                
                # Update gauge
                self.active_pipelines.inc()
                
                # Time the execution
                with self.processing_duration.time():
                    try:
                        self.logger.info(
                            "Starting pipeline execution",
                            extra={'pipeline': self.pipeline_name}
                        )
                        
                        # Execute pipeline
                        result = func(*args, **kwargs)
                        
                        # Update metrics
                        if hasattr(result, 'processed_count'):
                            self.records_processed.inc(result.processed_count)
                        
                        self.logger.info(
                            "Pipeline completed successfully",
                            extra={
                                'pipeline': self.pipeline_name,
                                'duration': result.duration if hasattr(result, 'duration') else None
                            }
                        )
                        
                        return result
                        
                    except Exception as e:
                        # Log error
                        self.pipeline_errors.labels(error_type=type(e).__name__).inc()
                        self.logger.error(
                            "Pipeline failed",
                            extra={
                                'pipeline': self.pipeline_name,
                                'error': str(e),
                                'error_type': type(e).__name__
                            }
                        )
                        
                        # Add error to span
                        span.record_exception(e)
                        span.set_status(trace.Status(trace.StatusCode.ERROR))
                        
                        raise
                    finally:
                        self.active_pipelines.dec()
        
        return wrapper
    
    def create_grafana_dashboard(self):
        """Generate Grafana dashboard JSON"""
        dashboard = {
            "dashboard": {
                "title": f"{self.pipeline_name} Monitoring",
                "panels": [
                    {
                        "title": "Records Processed",
                        "targets": [{
                            "expr": f"rate({self.pipeline_name}_records_processed_total[5m])",
                            "legendFormat": "{{job}}"
                        }]
                    },
                    {
                        "title": "Processing Duration",
                        "targets": [{
                            "expr": f"histogram_quantile(0.95, rate({self.pipeline_name}_processing_duration_seconds_bucket[5m]))",
                            "legendFormat": "95th percentile"
                        }]
                    },
                    {
                        "title": "Pipeline Errors",
                        "targets": [{
                            "expr": f"rate({self.pipeline_name}_errors_total[5m])",
                            "legendFormat": "{{error_type}}"
                        }]
                    }
                ]
            }
        }
        
        return dashboard

# Airflow integration
def airflow_metrics_callback(context):
    """Send metrics from Airflow task"""
    from airflow.models import TaskInstance
    
    ti: TaskInstance = context['ti']
    
    # Prometheus metrics
    records_processed = ti.xcom_pull(key='processed_count')
    if records_processed:
        Counter('airflow_records_processed_total').inc(records_processed)
    
    # Duration
    duration = (ti.end_date - ti.start_date).total_seconds()
    Histogram('airflow_task_duration_seconds').observe(duration)
    
    # Send to monitoring system
    import requests
    requests.post('http://monitoring:9090/metrics', json={
        'task_id': ti.task_id,
        'dag_id': ti.dag_id,
        'duration': duration,
        'records_processed': records_processed,
        'state': ti.state
    })
```

### **6. Production Deployment Patterns**
```python
# Compare: Java WAR/JAR deployment vs Data Pipeline deployment
# Java: Tomcat, Docker, Kubernetes, Helm
# Python: Docker, Kubernetes, Helm, GitOps

# Dockerfile for Data Pipeline
"""
# Base image with Java AND Python (polyglot!)
FROM openjdk:11-jre-slim as java-base
FROM python:3.11-slim as python-base

# Multi-stage build
FROM python-base as builder

# Install system dependencies
RUN apt-get update && apt-get install -y \
    openjdk-11-jre-headless \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Copy Spark (Java)
COPY --from=java-base /usr/local/openjdk-11 /usr/local/openjdk-11
ENV JAVA_HOME=/usr/local/openjdk-11
ENV PATH=$JAVA_HOME/bin:$PATH

# Copy Python dependencies
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# Copy application code
COPY . /app
WORKDIR /app

# Entrypoint
ENTRYPOINT ["python", "main.py"]
"""

# Kubernetes Deployment
"""
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-pipeline
spec:
  replicas: 3
  selector:
    matchLabels:
      app: data-pipeline
  template:
    metadata:
      labels:
        app: data-pipeline
    spec:
      containers:
      - name: pipeline
        image: data-pipeline:latest
        env:
        - name: SPARK_MASTER
          value: "k8s://https://kubernetes.default.svc"
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: access-key
        resources:
          requests:
            memory: "2Gi"
            cpu: "500m"
          limits:
            memory: "4Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
"""

# Helm Chart (like Maven for Kubernetes)
"""
# Chart.yaml
apiVersion: v2
name: data-pipeline
version: 1.0.0
dependencies:
  - name: airflow
    version: 1.0.0
    repository: https://airflow.apache.org
  - name: spark
    version: 1.0.0
    repository: https://spark.apache.org

# values.yaml
airflow:
  enabled: true
  executor: KubernetesExecutor
  image:
    repository: apache/airflow
    tag: 2.7.0

spark:
  enabled: true
  mode: cluster
  executor:
    instances: 10
    memory: 4g
    cores: 2
"""

# GitOps with ArgoCD
"""
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: data-pipeline
spec:
  project: default
  source:
    repoURL: https://github.com/company/data-pipeline
    targetRevision: main
    path: kubernetes
  destination:
    server: https://kubernetes.default.svc
    namespace: data-pipeline
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
"""
```

## 🎯 **Why This Chapter is Essential:**

### **The Data Engineering Hierarchy of Needs:**
```python
# Level 1: Write code (✓ You have this from Part 5.1)
# Level 2: Schedule & orchestrate (← WE'RE HERE)
# Level 3: Monitor & alert
# Level 4: Scale & optimize
# Level 5: Governance & security

# Without orchestration, your PySpark code is just a script!
# With orchestration, it's a production data pipeline.
```

### **Career Impact:**
```python
# Job Titles You Can Now Target:
job_titles = [
    "Data Engineer",           # $130k
    "Senior Data Engineer",    # $160k  
    "Data Platform Engineer",  # $170k
    "MLOps Engineer",          # $180k
    "Data Architect",          # $200k
]

# Skills You'll Have:
skills = [
    "PySpark (from Part 5.1)",
    "Airflow/Prefect Orchestration",
    "Real-time Streaming",
    "Data Pipeline Testing",
    "Production Monitoring",
    "Kubernetes Deployment",
]
```

## 📚 **Practical Project: Build a Complete Data Platform**

```python
# Project: User Analytics Platform
# Architecture:
"""
┌─────────────────────────────────────────────────────┐
│             USER ANALYTICS PLATFORM                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────┐    ┌──────────┐    ┌────────────┐    │
│  │  Java    │    │  Kafka   │    │   Python   │    │
│  │  Web App │────▶  Events  │────▶  Streaming │    │
│  │ (Spring) │    │          │    │  (Faust)   │    │
│  └──────────┘    └──────────┘    └────────────┘    │
│         │                │                │         │
│         ▼                ▼                ▼         │
│  ┌──────────┐    ┌──────────┐    ┌────────────┐    │
│  │PostgreSQL│    │   S3     │    │   Airflow  │    │
│  │ (OLTP)   │    │(Data Lake)│    │  (Batch)   │    │
│  └──────────┘    └──────────┘    └────────────┘    │
│         │                │                │         │
│         └────────────────┼────────────────┘         │
│                          ▼                          │
│                   ┌────────────┐                    │
│                   │   PySpark  │                    │
│                   │ (Analytics)│                    │
│                   └────────────┘                    │
│                          │                          │
│                          ▼                          │
│                   ┌────────────┐                    │
│                   │  BigQuery  │                    │
│                   │(Data Mart) │                    │
│                   └────────────┘                    │
│                          │                          │
│                          ▼                          │
│                   ┌────────────┐                    │
│                   │   FastAPI  │                    │
│                   │   (API)    │                    │
│                   └────────────┘                    │
│                          │                          │
│                          ▼                          │
│                   ┌────────────┐                    │
│                   │ React App  │                    │
│                   │ (Dashboard)│                    │
│                   └────────────┘                    │
└─────────────────────────────────────────────────────┘
"""

# Your role: Build everything from Kafka to Dashboard!
# Java handles transactions, Python handles data.
```

## 🚀 **Learning Outcomes:**

After this chapter, you'll be able to:
1. **Orchestrate complex data pipelines** with Airflow/Prefect
2. **Build real-time streaming applications** with Python
3. **Test and validate data pipelines** like software
4. **Monitor production pipelines** with observability
5. **Deploy data pipelines** to Kubernetes
6. **Integrate Java and Python systems** seamlessly

## 💡 **The Big Picture:**

**You're transitioning from:**
```
Java Backend Developer 
    ↓
Data Engineer (PySpark + Orchestration + Streaming)
    ↓
Data Platform Engineer (Full pipeline ownership)
```

**This is the inflection point** where your value multiplies because you can now **build complete data platforms**, not just write PySpark code.

---