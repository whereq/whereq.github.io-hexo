---
title: >-
  Python for Java Developers: Machine Learning Operations (MLOps) & Model
  Serving - VIII
date: 2025-12-24 10:10:00
categories:
- Python
tags:
- Python
---

## 🎯 **Chapater 8: Machine Learning Operations (MLOps) & Model Serving**

### **Why This is THE Natural Progression:**

1. **Logical Flow**: Data Pipeline → ML Pipeline → MLOps
2. **Career Superpower**: MLOps is **THE** hottest field right now
3. **Java Advantage**: Your production engineering skills give you an **edge** over pure data scientists
4. **Complete Picture**: You'll have the **full stack** from data to ML to production

---

## 📊 **From Data Pipelines to Production ML Systems**

### **The ML Hierarchy You're Building:**
```
Level 1: Data Processing (PySpark) ✓
Level 2: Data Orchestration (Airflow) ✓  
Level 3: ML Development (Python)
Level 4: ML Deployment (MLOps) ← WE'RE HERE
Level 5: ML Monitoring & Governance
```

## 🚀 **Chapter Outline: MLOps & Model Serving**

### **1. ML Pipeline Development: Scikit-learn vs Java ML**
```python
# Compare: Java MLlib vs Python Scikit-learn
# Java: Spark MLlib, Weka, Tribuo
# Python: Scikit-learn, XGBoost, LightGBM, TensorFlow

from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report, roc_auc_score
import mlflow
import pandas as pd
import numpy as np

class MLPipeline:
    """Complete ML pipeline (what Java developers need to know)"""
    
    def __init__(self):
        # MLflow tracking
        mlflow.set_tracking_uri("http://localhost:5000")
        mlflow.set_experiment("fraud_detection")
        
    def prepare_data(self) -> tuple:
        """Data preparation (your PySpark skills help here!)"""
        # Load data (could be from PySpark output)
        df = pd.read_parquet("/data/processed/transactions.parquet")
        
        # Feature engineering
        df['amount_log'] = np.log1p(df['amount'])
        df['hour_of_day'] = pd.to_datetime(df['timestamp']).dt.hour
        df['is_weekend'] = pd.to_datetime(df['timestamp']).dt.dayofweek >= 5
        
        # Features and target
        X = df.drop(['is_fraud', 'timestamp', 'transaction_id'], axis=1)
        y = df['is_fraud']
        
        return train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
    
    def create_pipeline(self):
        """Create ML pipeline (like Java MLlib Pipeline)"""
        # Define numeric and categorical columns
        numeric_features = ['amount', 'amount_log', 'hour_of_day']
        categorical_features = ['merchant_category', 'payment_method']
        
        # Preprocessing transformers
        numeric_transformer = Pipeline(steps=[
            ('scaler', StandardScaler())
        ])
        
        categorical_transformer = Pipeline(steps=[
            ('onehot', OneHotEncoder(handle_unknown='ignore'))
        ])
        
        # Column transformer (like Spark's VectorAssembler)
        preprocessor = ColumnTransformer(
            transformers=[
                ('num', numeric_transformer, numeric_features),
                ('cat', categorical_transformer, categorical_features)
            ])
        
        # Complete pipeline
        pipeline = Pipeline(steps=[
            ('preprocessor', preprocessor),
            ('classifier', RandomForestClassifier(
                n_estimators=100,
                max_depth=10,
                random_state=42
            ))
        ])
        
        return pipeline
    
    def train_with_mlflow(self, X_train, y_train):
        """Train with experiment tracking (like MLflow in Java)"""
        with mlflow.start_run():
            # Log parameters
            mlflow.log_param("model_type", "RandomForest")
            mlflow.log_param("n_estimators", 100)
            mlflow.log_param("max_depth", 10)
            
            # Create and train pipeline
            pipeline = self.create_pipeline()
            pipeline.fit(X_train, y_train)
            
            # Log model
            mlflow.sklearn.log_model(pipeline, "model")
            
            # Log metrics from cross-validation
            cv_scores = cross_val_score(pipeline, X_train, y_train, cv=5)
            mlflow.log_metric("cv_accuracy_mean", cv_scores.mean())
            mlflow.log_metric("cv_accuracy_std", cv_scores.std())
            
            return pipeline
    
    def evaluate(self, model, X_test, y_test):
        """Model evaluation"""
        predictions = model.predict(X_test)
        probabilities = model.predict_proba(X_test)[:, 1]
        
        # Calculate metrics
        report = classification_report(y_test, predictions, output_dict=True)
        roc_auc = roc_auc_score(y_test, probabilities)
        
        # Log to MLflow
        mlflow.log_metric("test_accuracy", report['accuracy'])
        mlflow.log_metric("test_precision", report['1']['precision'])
        mlflow.log_metric("test_recall", report['1']['recall'])
        mlflow.log_metric("test_roc_auc", roc_auc)
        
        return {
            'accuracy': report['accuracy'],
            'roc_auc': roc_auc,
            'predictions': predictions,
            'probabilities': probabilities
        }
```

### **2. Feature Stores: Feast vs Java Feature Engineering**
```python
# Compare: Java custom feature computation vs Python feature stores
# Java: Manual feature computation, batch jobs
# Python: Feast, Hopsworks, Tecton

import feast
from datetime import datetime, timedelta
import pandas as pd

# Define feature store
fs = feast.FeatureStore(repo_path="feature_repo/")

# Register features (like defining database schema)
# feature_store/features.py
user_features = feast.FeatureView(
    name="user_features",
    entities=[feast.Entity(name="user_id")],
    features=[
        feast.Feature(name="total_transactions", dtype=feast.ValueType.INT64),
        feast.Feature(name="avg_transaction_amount", dtype=feast.ValueType.FLOAT),
        feast.Feature(name="fraud_rate", dtype=feast.ValueType.FLOAT),
        feast.Feature(name="last_transaction_days", dtype=feast.ValueType.INT64),
    ],
    batch_source=feast.FileSource(
        path="/data/features/user_features.parquet",
        timestamp_field="event_timestamp",
    ),
    ttl=timedelta(days=7)  # Feature freshness
)

class FeatureEngineering:
    """Modern feature engineering with feature store"""
    
    def compute_features(self, transactions_df: pd.DataFrame) -> pd.DataFrame:
        """Compute features (can be called from Java or Python)"""
        # Compute batch features
        user_features = transactions_df.groupby("user_id").agg({
            'transaction_id': 'count',
            'amount': ['mean', 'std'],
            'is_fraud': 'mean'
        }).reset_index()
        
        # Rename columns
        user_features.columns = [
            'user_id', 'total_transactions', 'avg_amount', 
            'std_amount', 'fraud_rate'
        ]
        
        # Add recency feature
        latest_transactions = transactions_df.groupby("user_id")['timestamp'].max()
        current_time = pd.Timestamp.now()
        user_features['last_transaction_days'] = (
            current_time - latest_transactions
        ).dt.days
        
        return user_features
    
    def write_to_feature_store(self, features_df: pd.DataFrame):
        """Write features to feature store"""
        # Add timestamp
        features_df['event_timestamp'] = pd.Timestamp.now()
        
        # Write to offline store (for training)
        fs.write_to_offline_store(
            feature_view_name="user_features",
            df=features_df,
            allow_registry_cache=False
        )
        
        # Materialize to online store (for serving)
        fs.materialize(
            start_date=datetime.now() - timedelta(days=1),
            end_date=datetime.now()
        )
    
    def get_features_for_training(self, user_ids: list) -> pd.DataFrame:
        """Get features for model training"""
        # Get historical features
        feature_vector = fs.get_historical_features(
            entity_df=pd.DataFrame({"user_id": user_ids}),
            feature_refs=[
                "user_features:total_transactions",
                "user_features:avg_transaction_amount",
                "user_features:fraud_rate",
                "user_features:last_transaction_days"
            ]
        ).to_df()
        
        return feature_vector
    
    def get_features_for_serving(self, user_id: int) -> dict:
        """Get latest features for real-time prediction"""
        # Online feature retrieval (low latency)
        feature_vector = fs.get_online_features(
            feature_refs=[
                "user_features:total_transactions",
                "user_features:avg_transaction_amount",
                "user_features:fraud_rate",
                "user_features:last_transaction_days"
            ],
            entity_rows=[{"user_id": user_id}]
        ).to_dict()
        
        return feature_vector
```

### **3. Model Serving: REST APIs vs Java Microservices**
```python
# Compare: Java Spring Boot REST API vs Python FastAPI for ML
# Java: Spring Boot, REST, gRPC, serialization
# Python: FastAPI, MLflow Serving, BentoML, Seldon Core

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
import numpy as np
import joblib
from typing import List, Optional
import pandas as pd
from prometheus_client import Counter, Histogram, generate_latest
import asyncio
import redis

app = FastAPI(title="ML Model Service", version="1.0.0")

# Metrics (like Micrometer)
PREDICTION_COUNT = Counter('model_predictions_total', 'Total predictions')
PREDICTION_LATENCY = Histogram('prediction_latency_seconds', 'Prediction latency')

# Request/Response models (like Java DTOs)
class PredictionRequest(BaseModel):
    """Input for prediction (like Java Request DTO)"""
    user_id: int = Field(..., gt=0)
    amount: float = Field(..., gt=0)
    merchant_category: str
    payment_method: str
    timestamp: str
    
    class Config:
        json_schema_extra = {
            "example": {
                "user_id": 12345,
                "amount": 150.75,
                "merchant_category": "electronics",
                "payment_method": "credit_card",
                "timestamp": "2024-01-15T14:30:00Z"
            }
        }

class PredictionResponse(BaseModel):
    """Output from prediction (like Java Response DTO)"""
    prediction: bool
    probability: float = Field(..., ge=0, le=1)
    model_version: str
    processing_time_ms: float

# Model loading (could be from MLflow)
class ModelService:
    """Model service with caching and fallback"""
    
    def __init__(self):
        self.model = self._load_model()
        self.cache = redis.Redis(host='redis', port=6379, decode_responses=True)
        self.feature_store = FeatureEngineering()
        
    def _load_model(self):
        """Load model (from MLflow, S3, etc.)"""
        try:
            # Try MLflow first
            import mlflow.pyfunc
            model = mlflow.pyfunc.load_model("models:/fraud_model/production")
        except:
            # Fallback to local
            model = joblib.load("/models/fraud_model.pkl")
        
        return model
    
    @PREDICTION_LATENCY.time()
    async def predict(self, request: PredictionRequest) -> PredictionResponse:
        """Make prediction with caching"""
        start_time = asyncio.get_event_loop().time()
        
        # Check cache first
        cache_key = f"pred:{request.user_id}:{request.amount}:{request.merchant_category}"
        cached = self.cache.get(cache_key)
        
        if cached:
            cached_data = eval(cached)
            return PredictionResponse(
                prediction=cached_data['prediction'],
                probability=cached_data['probability'],
                model_version="cached",
                processing_time_ms=0.1
            )
        
        # Get features from feature store
        user_features = self.feature_store.get_features_for_serving(request.user_id)
        
        # Prepare input data
        input_data = pd.DataFrame([{
            'user_id': request.user_id,
            'amount': request.amount,
            'merchant_category': request.merchant_category,
            'payment_method': request.payment_method,
            **user_features  # Merge with feature store features
        }])
        
        # Make prediction
        try:
            probability = self.model.predict_proba(input_data)[0, 1]
            prediction = probability > 0.5
            
            # Cache result (TTL: 5 minutes for dynamic data)
            self.cache.setex(
                cache_key, 300,
                str({'prediction': bool(prediction), 'probability': float(probability)})
            )
            
            # Update metrics
            PREDICTION_COUNT.inc()
            
            processing_time = (asyncio.get_event_loop().time() - start_time) * 1000
            
            return PredictionResponse(
                prediction=bool(prediction),
                probability=float(probability),
                model_version="1.0.0",
                processing_time_ms=processing_time
            )
            
        except Exception as e:
            raise HTTPException(status_code=500, detail=f"Prediction failed: {str(e)}")

# FastAPI endpoints
@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    """Predict endpoint (like @PostMapping in Spring)"""
    service = ModelService()
    return await service.predict(request)

@app.get("/health")
async def health():
    """Health check (like Spring Actuator)"""
    return {"status": "healthy", "timestamp": pd.Timestamp.now().isoformat()}

@app.get("/metrics")
async def metrics():
    """Prometheus metrics endpoint"""
    return generate_latest()

# Batch prediction endpoint
@app.post("/batch_predict")
async def batch_predict(requests: List[PredictionRequest]):
    """Batch prediction (optimized)"""
    service = ModelService()
    
    # Process in parallel
    tasks = [service.predict(req) for req in requests]
    results = await asyncio.gather(*tasks)
    
    return {
        "predictions": results,
        "count": len(results),
        "batch_id": f"batch_{pd.Timestamp.now().timestamp()}"
    }

# Model management endpoints
@app.post("/models/{model_name}/deploy")
async def deploy_model(model_name: str, version: str = "latest"):
    """Deploy new model version (like CI/CD for models)"""
    # Download from MLflow
    import mlflow.pyfunc
    model = mlflow.pyfunc.load_model(f"models:/{model_name}/{version}")
    
    # Save to model registry
    joblib.dump(model, f"/models/{model_name}.pkl")
    
    # Update load balancer/API gateway
    # (Integration with Kubernetes, Istio, etc.)
    
    return {"status": "deployed", "model": model_name, "version": version}
```

### **4. Model Monitoring & Drift Detection**
```python
# Compare: Java application monitoring vs ML model monitoring
# Java: Micrometer, Prometheus, custom metrics
# Python: Evidently, Alibi Detect, MLflow, custom drift detection

import evidently
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset, ClassificationPreset
from evidently.metrics import *
import pandas as pd
from datetime import datetime, timedelta
import numpy as np

class ModelMonitoring:
    """Production model monitoring (like Java monitoring but for ML)"""
    
    def __init__(self, model_name: str):
        self.model_name = model_name
        self.reference_data = None
        self.drift_detected = False
        
    def set_reference_data(self, data: pd.DataFrame):
        """Set reference data (baseline)"""
        self.reference_data = data
        
    def detect_data_drift(self, current_data: pd.DataFrame) -> dict:
        """Detect data drift between reference and current"""
        if self.reference_data is None:
            raise ValueError("Reference data not set")
        
        # Generate drift report
        data_drift_report = Report(metrics=[
            DataDriftPreset(),
            DatasetSummaryMetric(),
            ColumnSummaryMetric(column_name="amount"),
            ColumnSummaryMetric(column_name="probability"),
        ])
        
        data_drift_report.run(
            reference_data=self.reference_data,
            current_data=current_data
        )
        
        # Extract results
        results = data_drift_report.as_dict()
        
        # Check for drift
        drift_metrics = results['metrics'][0]['result']  # DataDriftPreset results
        n_drifted_features = drift_metrics.get('number_of_drifted_features', 0)
        dataset_drift = drift_metrics.get('dataset_drift', False)
        
        self.drift_detected = dataset_drift or n_drifted_features > 0
        
        return {
            'drift_detected': self.drift_detected,
            'n_drifted_features': n_drifted_features,
            'dataset_drift': dataset_drift,
            'report': results
        }
    
    def detect_concept_drift(self, y_true: list, y_pred: list) -> dict:
        """Detect concept drift (model performance degradation)"""
        from sklearn.metrics import accuracy_score, precision_score, recall_score
        
        # Calculate metrics
        accuracy = accuracy_score(y_true, y_pred)
        precision = precision_score(y_true, y_pred, zero_division=0)
        recall = recall_score(y_true, y_pred, zero_division=0)
        
        # Compare with baseline (would come from training)
        baseline_accuracy = 0.95  # From model training
        
        # Detect significant drop
        accuracy_drop = baseline_accuracy - accuracy
        concept_drift = accuracy_drop > 0.05  # 5% drop threshold
        
        return {
            'concept_drift': concept_drift,
            'accuracy': accuracy,
            'accuracy_drop': accuracy_drop,
            'precision': precision,
            'recall': recall,
            'baseline_accuracy': baseline_accuracy
        }
    
    def monitor_predictions(self, predictions: list, probabilities: list):
        """Monitor prediction distribution"""
        import matplotlib.pyplot as plt
        import io
        import base64
        
        # Create monitoring dashboard
        fig, axes = plt.subplots(2, 2, figsize=(12, 10))
        
        # 1. Prediction distribution
        axes[0, 0].hist(probabilities, bins=20, alpha=0.7)
        axes[0, 0].set_title('Prediction Probability Distribution')
        axes[0, 0].set_xlabel('Probability')
        axes[0, 0].set_ylabel('Count')
        
        # 2. Time series of predictions
        axes[0, 1].plot(range(len(predictions)), predictions, 'b-', alpha=0.7)
        axes[0, 1].set_title('Predictions Over Time')
        axes[0, 1].set_xlabel('Time')
        axes[0, 1].set_ylabel('Prediction')
        
        # 3. Confidence intervals
        confidence_intervals = np.percentile(probabilities, [5, 95])
        axes[1, 0].bar(['5th', '95th'], confidence_intervals)
        axes[1, 0].set_title('Confidence Intervals')
        axes[1, 0].set_ylabel('Probability')
        
        # 4. Drift detection over time
        axes[1, 1].plot([0, 1], [0.9, 0.85], 'r-', label='Accuracy')
        axes[1, 1].set_title('Model Performance Over Time')
        axes[1, 1].set_xlabel('Time')
        axes[1, 1].set_ylabel('Metric')
        axes[1, 1].legend()
        
        plt.tight_layout()
        
        # Convert plot to base64 for web display
        buf = io.BytesIO()
        plt.savefig(buf, format='png')
        buf.seek(0)
        img_str = base64.b64encode(buf.read()).decode('utf-8')
        plt.close()
        
        return {
            'plot': f"data:image/png;base64,{img_str}",
            'statistics': {
                'mean_probability': np.mean(probabilities),
                'std_probability': np.std(probabilities),
                'prediction_rate': np.mean(predictions),
                'total_predictions': len(predictions)
            }
        }
    
    def alert_on_drift(self, drift_results: dict):
        """Send alerts when drift detected"""
        if drift_results.get('drift_detected') or drift_results.get('concept_drift'):
            # Send to alerting system (PagerDuty, Slack, etc.)
            import requests
            
            alert_message = {
                'severity': 'warning',
                'model': self.model_name,
                'type': 'drift_detected',
                'timestamp': datetime.now().isoformat(),
                'details': drift_results
            }
            
            # Send to Slack
            requests.post(
                'https://hooks.slack.com/services/...',
                json={'text': f"🚨 Model drift detected for {self.model_name}"}
            )
            
            # Send to PagerDuty
            requests.post(
                'https://events.pagerduty.com/v2/enqueue',
                json={
                    'routing_key': 'ml-monitoring',
                    'event_action': 'trigger',
                    'payload': {
                        'summary': f'Model drift: {self.model_name}',
                        'severity': 'warning',
                        'source': 'ml-monitoring'
                    }
                }
            )
```

### **5. Model Registry & Versioning**
```python
# Compare: Java artifact repositories vs ML model registries
# Java: Maven Central, JFrog, Nexus
# Python: MLflow Model Registry, DVC, Weights & Biases

import mlflow
from mlflow.tracking import MlflowClient
from datetime import datetime
import json

class ModelRegistry:
    """Model versioning and lifecycle management"""
    
    def __init__(self, tracking_uri: str = "http://localhost:5000"):
        self.client = MlflowClient(tracking_uri=tracking_uri)
        
    def register_model(self, model_name: str, run_id: str):
        """Register model in MLflow Registry"""
        # Create model if it doesn't exist
        try:
            self.client.create_registered_model(model_name)
        except mlflow.exceptions.RestException:
            pass  # Model already exists
        
        # Register model version
        model_uri = f"runs:/{run_id}/model"
        mv = self.client.create_model_version(
            name=model_name,
            source=model_uri,
            run_id=run_id
        )
        
        print(f"Registered model version: {mv.version}")
        return mv
    
    def promote_to_staging(self, model_name: str, version: int):
        """Promote model to staging"""
        self.client.transition_model_version_stage(
            name=model_name,
            version=version,
            stage="Staging"
        )
        
        # Add description
        self.client.update_model_version(
            name=model_name,
            version=version,
            description="Promoted to staging for testing"
        )
    
    def promote_to_production(self, model_name: str, version: int):
        """Promote model to production"""
        # First, validate the model
        validation_result = self._validate_model(model_name, version)
        
        if validation_result['passed']:
            # Archive current production model
            current_prod = self.get_production_model(model_name)
            if current_prod:
                self.client.transition_model_version_stage(
                    name=model_name,
                    version=current_prod.version,
                    stage="Archived"
                )
            
            # Promote new model
            self.client.transition_model_version_stage(
                name=model_name,
                version=version,
                stage="Production"
            )
            
            # Log deployment
            self.client.set_model_version_tag(
                name=model_name,
                version=version,
                key="deployed_at",
                value=datetime.now().isoformat()
            )
            
            return True
        else:
            raise ValueError(f"Model validation failed: {validation_result['errors']}")
    
    def _validate_model(self, model_name: str, version: int) -> dict:
        """Validate model before promotion"""
        # Load model
        model = mlflow.pyfunc.load_model(
            model_uri=f"models:/{model_name}/{version}"
        )
        
        # Run validation tests
        validation_tests = {
            'loads_successfully': True,  # Already passed
            'has_predict_method': hasattr(model, 'predict'),
            'has_predict_proba_method': hasattr(model, 'predict_proba'),
            # Add custom validation logic here
        }
        
        passed = all(validation_tests.values())
        
        return {
            'passed': passed,
            'tests': validation_tests,
            'errors': [k for k, v in validation_tests.items() if not v]
        }
    
    def get_production_model(self, model_name: str):
        """Get current production model"""
        model_versions = self.client.search_model_versions(
            f"name='{model_name}' and status='READY'"
        )
        
        for mv in model_versions:
            if mv.current_stage == "Production":
                return mv
        
        return None
    
    def rollback_model(self, model_name: str, target_version: int):
        """Rollback to previous model version"""
        current_prod = self.get_production_model(model_name)
        
        if current_prod:
            # Archive current production
            self.client.transition_model_version_stage(
                name=model_name,
                version=current_prod.version,
                stage="Archived"
            )
        
        # Promote target version
        self.client.transition_model_version_stage(
            name=model_name,
            version=target_version,
            stage="Production"
        )
        
        # Add rollback tag
        self.client.set_model_version_tag(
            name=model_name,
            version=target_version,
            key="rollback",
            value=datetime.now().isoformat()
        )
    
    def get_model_lineage(self, model_name: str):
        """Get model lineage (training data, code version, etc.)"""
        prod_model = self.get_production_model(model_name)
        
        if prod_model:
            # Get run info
            run = self.client.get_run(prod_model.run_id)
            
            lineage = {
                'model_name': model_name,
                'version': prod_model.version,
                'stage': prod_model.current_stage,
                'created_at': prod_model.creation_timestamp,
                'run_id': prod_model.run_id,
                'experiment_id': run.info.experiment_id,
                'training_metrics': run.data.metrics,
                'training_params': run.data.params,
                'training_tags': run.data.tags,
                'artifact_uri': run.info.artifact_uri,
                'git_commit': run.data.tags.get('mlflow.source.git.commit', 'unknown')
            }
            
            return lineage
        
        return None
```

### **6. Complete MLOps Pipeline with Airflow**
```python
# Complete MLOps pipeline integrating everything
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from airflow.providers.docker.operators.docker import DockerOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'mlops_engineer',
    'depends_on_past': False,
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'mlops_pipeline',
    default_args=default_args,
    description='End-to-end MLOps pipeline',
    schedule_interval='0 3 * * 0',  # Weekly on Sunday at 3 AM
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['mlops', 'retraining', 'production'],
)

def extract_training_data():
    """Extract training data"""
    from pyspark.sql import SparkSession
    
    spark = SparkSession.builder.getOrCreate()
    
    # Extract raw data
    transactions = spark.read.parquet("/data/transactions/*.parquet")
    users = spark.read.parquet("/data/users/*.parquet")
    
    # Join and prepare
    training_data = transactions.join(users, "user_id", "left")
    training_data.write.mode("overwrite").parquet("/data/training/latest.parquet")
    
    return "/data/training/latest.parquet"

def train_model(**context):
    """Train model with MLflow"""
    import mlflow
    import pandas as pd
    from sklearn.model_selection import train_test_split
    
    # Get data path from previous task
    data_path = context['ti'].xcom_pull(task_ids='extract_training_data')
    df = pd.read_parquet(data_path)
    
    # Start MLflow run
    with mlflow.start_run(run_name="weekly_retraining") as run:
        # Log parameters
        mlflow.log_param("training_date", datetime.now().isoformat())
        mlflow.log_param("data_size", len(df))
        
        # Train model
        # ... (training logic from earlier)
        
        # Log model
        mlflow.sklearn.log_model(model, "model")
        
        # Register model
        client = mlflow.tracking.MlflowClient()
        client.create_model_version(
            name="fraud_detection",
            source=f"runs:/{run.info.run_id}/model",
            run_id=run.info.run_id
        )
        
        return run.info.run_id

def validate_model(**context):
    """Validate new model"""
    run_id = context['ti'].xcom_pull(task_ids='train_model')
    
    # Load model
    import mlflow.pyfunc
    model = mlflow.pyfunc.load_model(f"runs:/{run_id}/model")
    
    # Load validation data
    val_data = pd.read_parquet("/data/validation/latest.parquet")
    
    # Validate
    predictions = model.predict(val_data.drop('is_fraud', axis=1))
    accuracy = (predictions == val_data['is_fraud']).mean()
    
    # Check if model meets criteria
    if accuracy >= 0.95:  # 95% accuracy threshold
        context['ti'].xcom_push(key='validation_passed', value=True)
        context['ti'].xcom_push(key='model_accuracy', value=accuracy)
    else:
        context['ti'].xcom_push(key='validation_passed', value=False)
        
    return accuracy >= 0.95

def deploy_model(**context):
    """Deploy validated model"""
    validation_passed = context['ti'].xcom_pull(
        task_ids='validate_model', 
        key='validation_passed'
    )
    
    if validation_passed:
        run_id = context['ti'].xcom_pull(task_ids='train_model')
        
        # Get latest model version
        client = mlflow.tracking.MlflowClient()
        model_versions = client.search_model_versions(f"run_id='{run_id}'")
        latest_version = max(model_versions, key=lambda x: x.version)
        
        # Promote to staging
        client.transition_model_version_stage(
            name="fraud_detection",
            version=latest_version.version,
            stage="Staging"
        )
        
        # Run integration tests
        # ... (test API endpoints, etc.)
        
        # Promote to production
        client.transition_model_version_stage(
            name="fraud_detection",
            version=latest_version.version,
            stage="Production"
        )
        
        print(f"Deployed model version {latest_version.version} to production")
        
        # Trigger model serving update
        import requests
        requests.post(
            "http://model-serving:8000/models/fraud_detection/deploy",
            json={"version": latest_version.version}
        )

# Define tasks
extract_task = PythonOperator(
    task_id='extract_training_data',
    python_callable=extract_training_data,
    dag=dag,
)

train_task = PythonOperator(
    task_id='train_model',
    python_callable=train_model,
    dag=dag,
)

validate_task = PythonOperator(
    task_id='validate_model',
    python_callable=validate_model,
    dag=dag,
)

deploy_task = PythonOperator(
    task_id='deploy_model',
    python_callable=deploy_model,
    dag=dag,
)

# Dependencies
extract_task >> train_task >> validate_task >> deploy_task
```

## 🎯 **Why This Chapter is ESSENTIAL:**

### **The MLOps Market Reality:**
```python
# Job Market Demand (2024):
ml_engineer_demand = "High"
data_engineer_demand = "Very High"
mlops_engineer_demand = "EXTREMELY HIGH (shortage!)"

# Salary Comparison (US):
average_salaries = {
    "Data Engineer": 130000,
    "ML Engineer": 150000,
    "MLOps Engineer": 180000,  # +38% premium!
    "Senior MLOps Engineer": 220000
}

# Your Competitive Edge:
"""
As a Java Developer learning MLOps:
1. Production experience ✓ (deployment, monitoring, scaling)
2. Engineering rigor ✓ (testing, CI/CD, architecture)
3. Now adding: ML pipelines, model serving, monitoring
4. Result: You can build ENTIRE ML platforms, not just models
"""
```

### **Complete Skill Stack After This Chapter:**
```
✅ Java Backend Development (Spring Boot, Microservices)
✅ Python Data Processing (PySpark, Pandas)
✅ Data Pipeline Orchestration (Airflow, Prefect)
✅ Machine Learning Development (Scikit-learn, MLflow)
✅ MLOps & Model Serving (FastAPI, Docker, Kubernetes)
✅ Monitoring & Observability (Prometheus, Grafana)

→ You're now a FULL-STACK DATA PLATFORM ENGINEER!
```

## 📚 **Practical Project: Build a Fraud Detection System**

```python
# End-to-end Fraud Detection System
# Architecture that uses BOTH Java and Python:

"""
┌─────────────────────────────────────────────────────┐
│         PRODUCTION FRAUD DETECTION SYSTEM           │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Java Transaction Service (Spring Boot)             │
│  ├─ Processes payments                              │
│  ├─ Validates business rules                        │
│  └─ Sends to Kafka for fraud check                  │
│                                                     │
│  Kafka (Event Bus)                                  │
│  ├─ Real-time transaction stream                    │
│  └─ Connects Java and Python systems                │
│                                                     │
│  Python Fraud Detection (FastAPI + ML)              │
│  ├─ Real-time prediction (100ms latency)            │
│  ├─ Model serving with feature store                │
│  └─ Returns fraud score to Java service             │
│                                                     │
│  Python Batch Processing (Airflow + PySpark)        │
│  ├─ Weekly model retraining                         │
│  ├─ Feature engineering                             │
│  └─ Model evaluation & deployment                   │
│                                                     │
│  Monitoring (Prometheus + Grafana)                  │
│  ├─ Model performance metrics                       │
│  ├─ Data drift detection                            │
│  └─ Business impact tracking                        │
└─────────────────────────────────────────────────────┘
"""

# Your Role: Build the ENTIRE Python side, integrate with Java
```

## 🚀 **Learning Path for Part 5.3:**

1. **Week 1**: ML Pipeline Development (Scikit-learn, MLflow)
2. **Week 2**: Feature Stores & Engineering (Feast)
3. **Week 3**: Model Serving APIs (FastAPI, MLflow Serving)
4. **Week 4**: Model Monitoring & Drift Detection
5. **Week 5**: Model Registry & Versioning
6. **Week 6**: Complete MLOps Pipeline Integration
7. **Week 7-8**: Build Complete Fraud Detection System

## 💡 **The Ultimate Career Move:**

**You're positioning yourself as:**
```
Java Backend Developer (Traditional)
    ↓
Data Engineer (Modern)
    ↓
MLOps Engineer (Cutting-edge, High-demand)
    ↓
AI Platform Engineer (Future-proof)
```

**This is where you become IRREPLACEABLE** because:
1. **Few people** understand both Java backend AND ML systems
2. **Even fewer** can productionize ML models properly
3. **Almost none** can build complete ML platforms end-to-end

---