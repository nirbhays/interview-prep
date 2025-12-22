# 🎓 MONDAY.COM INTERVIEW - PRACTICE QUESTIONS & EXERCISES
## Hands-on Practice for DevOps, MLOps, System Design & Coding

---

## 📋 TABLE OF CONTENTS
1. [System Design Practice Questions](#system-design)
2. [Coding Exercises (15-min format)](#coding-exercises)
3. [DevOps/Kubernetes Scenarios](#devops-scenarios)
4. [MLOps Specific Questions](#mlops-questions)
5. [Behavioral Questions (STAR Method)](#behavioral-questions)

---

## 🏗️ SYSTEM DESIGN PRACTICE QUESTIONS

### Question 1: Design Monday.com's Automation Recommendation System
**Scenario:** Design a system that recommends automations to Monday.com users based on their board structure and usage patterns.

**Requirements:**
- 150,000 customers
- Millions of users
- Real-time recommendations (when user opens automation menu)
- Also batch recommendations (weekly emails)
- Multi-tenant isolation
- 99.9% availability

**Things to Cover:**
1. Data collection (user actions, board structure, usage patterns)
2. Feature engineering pipeline
3. Model training infrastructure
4. Real-time serving architecture
5. Batch processing for emails
6. Monitoring and metrics
7. A/B testing framework
8. Cost considerations

**Expected Solution Outline:**
```
DATA LAYER:
- Event tracking (Kafka/Kinesis)
- Data lake (S3/Snowflake)
- Feature store (Redis + S3)

ML LAYER:
- Training pipeline (Airflow + SageMaker)
- Model registry (MLflow)
- Versioning and experimentation

SERVING LAYER:
- Real-time API (K8s + FastAPI)
- Caching (Redis)
- Load balancing

MONITORING:
- Prometheus + Grafana
- Model drift detection
- Business metrics

BATCH LAYER:
- Spark jobs for batch predictions
- Email service integration
```

---

### Question 2: Design a Model Serving Platform for Multiple ML Models
**Scenario:** Monday.com wants to serve 20+ different ML models (NLP, recommendations, anomaly detection, etc.) from a unified platform.

**Requirements:**
- Support multiple frameworks (PyTorch, TensorFlow, scikit-learn)
- Different latency requirements (50ms to 5 seconds)
- Autoscaling per model
- Version management
- A/B testing capabilities
- Cost-effective

**Key Challenges:**
- Resource allocation (GPU vs CPU)
- Request routing
- Model loading and caching
- Monitoring across models
- Deployment coordination

---

### Question 3: Design a Feature Store for Monday.com
**Scenario:** Build a feature store that serves features for both real-time inference and batch training.

**Requirements:**
- Real-time serving (<10ms latency)
- Historical features for training
- Point-in-time correctness
- Feature versioning
- Discovery and documentation

---

### Question 4: Design a Data Pipeline for Model Training
**Scenario:** Design an end-to-end data pipeline that processes user activity logs and prepares training data for ML models.

**Scale:**
- 1TB of logs per day
- Need to aggregate features per user
- Support both streaming and batch processing
- Data quality validation

---

### Question 5: Design an Anomaly Detection System
**Scenario:** Detect unusual activity in Monday.com (security threats, bot traffic, data corruption).

**Requirements:**
- Real-time detection (<1 minute)
- Handle 100K events/second
- Low false positive rate
- Alerting and investigation workflow
- Scalable and cost-effective

---

## 💻 CODING EXERCISES (15-minute format)

### Exercise 1: Rate Limiter (Easy-Medium)
**Problem:**
Implement a rate limiter that allows N requests per minute per user.

```python
"""
Implement a RateLimiter class with the following methods:
1. is_allowed(user_id: str) -> bool
2. reset(user_id: str) -> None

Requirements:
- Sliding window algorithm
- Thread-safe
- Memory efficient

Example:
limiter = RateLimiter(max_requests=5, window_seconds=60)
limiter.is_allowed("user1") # True (1st request)
limiter.is_allowed("user1") # True (2nd request)
... (5 times)
limiter.is_allowed("user1") # False (exceeded limit)
# After 60 seconds
limiter.is_allowed("user1") # True (window reset)
"""

class RateLimiter:
    def __init__(self, max_requests: int, window_seconds: int):
        # Your implementation
        pass
    
    def is_allowed(self, user_id: str) -> bool:
        # Your implementation
        pass
    
    def reset(self, user_id: str) -> None:
        # Your implementation
        pass

# Test cases
def test_rate_limiter():
    limiter = RateLimiter(max_requests=3, window_seconds=60)
    
    # Within limit
    assert limiter.is_allowed("user1") == True
    assert limiter.is_allowed("user1") == True
    assert limiter.is_allowed("user1") == True
    
    # Exceeded
    assert limiter.is_allowed("user1") == False
    
    # Different user
    assert limiter.is_allowed("user2") == True
    
    # Reset
    limiter.reset("user1")
    assert limiter.is_allowed("user1") == True
```

**Solution Approach:**
1. Use dictionary to store user_id → list of timestamps
2. On each request, remove timestamps older than window
3. Check if remaining count < max_requests
4. Add current timestamp if allowed
5. Use threading.Lock for thread safety

---

### Exercise 2: LRU Cache with TTL (Medium)
**Problem:**
Implement an LRU cache that also supports TTL (Time To Live).

```python
"""
Implement a cache for ML model predictions:
- Capacity limit (LRU eviction)
- TTL (time-based expiration)
- Thread-safe
- O(1) get and put

Example:
cache = MLCache(capacity=2, ttl_seconds=5)
cache.put("input1", "output1")
cache.get("input1")  # "output1"
# After 6 seconds
cache.get("input1")  # None (expired)
"""

from datetime import datetime, timedelta
from collections import OrderedDict
import threading

class MLCache:
    def __init__(self, capacity: int, ttl_seconds: int):
        # Your implementation
        pass
    
    def get(self, key: str):
        # Your implementation
        pass
    
    def put(self, key: str, value):
        # Your implementation
        pass
    
    def stats(self) -> dict:
        # Return cache statistics
        pass
```

---

### Exercise 3: Log Aggregator (Medium)
**Problem:**
Parse and aggregate log data.

```python
"""
Given a list of log entries, aggregate by user and event type.

Input:
logs = [
    {"timestamp": "2024-01-01T10:00:00", "user_id": "u1", "event": "click"},
    {"timestamp": "2024-01-01T10:01:00", "user_id": "u1", "event": "click"},
    {"timestamp": "2024-01-01T10:02:00", "user_id": "u2", "event": "view"},
    {"timestamp": "2024-01-01T10:03:00", "user_id": "u1", "event": "purchase"},
]

Output:
{
    "u1": {"click": 2, "purchase": 1},
    "u2": {"view": 1}
}

Additional: Add time window filtering (e.g., last N minutes)
"""

def aggregate_logs(logs, window_minutes=None):
    # Your implementation
    pass

# Test
logs = [...]
result = aggregate_logs(logs)
print(result)
```

---

### Exercise 4: Feature Extractor (Medium)
**Problem:**
Extract features from user activity data.

```python
"""
Given user activity data, compute features for ML model.

Input:
activities = [
    {"user_id": "u1", "action": "create_item", "timestamp": "2024-01-01T10:00:00"},
    {"user_id": "u1", "action": "update_item", "timestamp": "2024-01-01T11:00:00"},
    {"user_id": "u2", "action": "create_board", "timestamp": "2024-01-01T12:00:00"},
]

Output features for each user:
{
    "u1": {
        "total_actions": 2,
        "unique_action_types": 2,
        "actions_last_hour": 1,
        "actions_last_24h": 2,
        "most_common_action": "create_item"
    },
    "u2": {...}
}
"""

from datetime import datetime, timedelta
from collections import Counter

def extract_features(activities, reference_time=None):
    # Your implementation
    # Consider: time windows, action types, frequencies
    pass
```

---

### Exercise 5: Model Version Router (Medium-Hard)
**Problem:**
Route requests to different model versions based on rules.

```python
"""
Implement a router that directs requests to model versions based on:
- User ID (for A/B testing)
- Feature flags
- Model health status

Example:
router = ModelRouter()
router.add_version("v1", weight=0.9, health=True)
router.add_version("v2", weight=0.1, health=True)

# Route based on consistent hashing
version = router.route(user_id="user123")  # "v1" or "v2"

# If v2 becomes unhealthy, all traffic goes to v1
router.set_health("v2", False)
version = router.route(user_id="user456")  # Always "v1"
"""

import hashlib

class ModelRouter:
    def __init__(self):
        # Your implementation
        pass
    
    def add_version(self, version_id: str, weight: float, health: bool = True):
        # Your implementation
        pass
    
    def route(self, user_id: str) -> str:
        # Consistent hashing + weights
        # Your implementation
        pass
    
    def set_health(self, version_id: str, is_healthy: bool):
        # Your implementation
        pass
```

---

## 🔧 DEVOPS/KUBERNETES SCENARIOS

### Scenario 1: Debug High Memory Usage
**Problem:**
A Kubernetes pod is getting OOMKilled. How do you debug and fix?

**Steps:**
1. Check pod logs: `kubectl logs pod-name`
2. Describe pod: `kubectl describe pod pod-name`
3. Check events for OOMKilled
4. Identify memory limits vs requests
5. Check application metrics
6. Profile memory usage
7. Increase limits or optimize code

**Fix:**
```yaml
# Before
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"

# After analysis, increase limits
resources:
  requests:
    memory: "512Mi"
  limits:
    memory: "1Gi"

# Or add memory-efficient settings
env:
- name: PYTHONOPTIMIZE
  value: "1"
- name: MODEL_BATCH_SIZE
  value: "16"  # Reduced from 32
```

---

### Scenario 2: Deploy Model with Zero Downtime
**Problem:**
Deploy a new model version without service interruption.

**Solution:**
```yaml
# Use RollingUpdate strategy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-server
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods over replicas
      maxUnavailable: 0  # Always keep all pods running
  template:
    spec:
      containers:
      - name: model
        image: model:v2.0
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30  # Wait for model to load
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
```

**Steps:**
```bash
# 1. Update image
kubectl set image deployment/model-server model=model:v2.0

# 2. Watch rollout
kubectl rollout status deployment/model-server

# 3. If issues, rollback
kubectl rollout undo deployment/model-server

# 4. Check rollout history
kubectl rollout history deployment/model-server
```

---

### Scenario 3: Setup Autoscaling Based on Custom Metrics
**Problem:**
Scale model server based on request queue length, not just CPU.

**Solution:**
```yaml
# 1. Expose custom metric
# In application code:
from prometheus_client import Gauge
queue_length = Gauge('model_queue_length', 'Requests waiting in queue')

# Update metric
queue_length.set(current_queue_size)

# 2. Configure Prometheus to scrape
# 3. Setup HPA with custom metric
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: model-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: model-server
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Pods
    pods:
      metric:
        name: model_queue_length
      target:
        type: AverageValue
        averageValue: "10"  # Scale if avg queue > 10
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 1
        periodSeconds: 60
```

---

### Scenario 4: Secret Management
**Problem:**
Store and use API keys, database passwords securely in Kubernetes.

**Solution:**
```bash
# Option 1: Kubernetes Secrets
kubectl create secret generic ml-secrets \
  --from-literal=api-key=abc123 \
  --from-literal=db-password=xyz789

# Option 2: From file
echo -n 'abc123' > api-key.txt
kubectl create secret generic ml-secrets --from-file=api-key=api-key.txt

# Option 3: External Secrets (AWS Secrets Manager)
# Install External Secrets Operator
# Then create ExternalSecret resource
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: ml-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-store
    kind: SecretStore
  target:
    name: ml-secrets
  data:
  - secretKey: api-key
    remoteRef:
      key: prod/ml/api-key

# Use in pod
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: model-server
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: ml-secrets
          key: api-key
    # Or mount as file
    volumeMounts:
    - name: secrets
      mountPath: /secrets
      readOnly: true
  volumes:
  - name: secrets
    secret:
      secretName: ml-secrets
```

---

### Scenario 5: CI/CD Pipeline Failure
**Problem:**
Your model deployment pipeline fails at the security scan stage.

**Debug Steps:**
```bash
# 1. Check pipeline logs
# Look for CVE vulnerabilities

# 2. Scan image locally
trivy image model:v2.0

# 3. Common issues:
#    - Outdated base image
#    - Vulnerable dependencies
#    - Hardcoded secrets

# 4. Fix Dockerfile
FROM python:3.11-slim  # Use recent version
RUN apt-get update && apt-get upgrade -y  # Update packages
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 5. Update dependencies
pip list --outdated
pip install --upgrade package-name

# 6. Remove hardcoded secrets
# Use environment variables instead

# 7. Re-scan
trivy image model:v2.1

# 8. If approved, push and deploy
```

---

## 🤖 MLOPS SPECIFIC QUESTIONS

### Question 1: Model Drift Detection Implementation
**Problem:**
Implement a simple drift detector that monitors feature distributions.

```python
"""
Detect if current data distribution differs from training distribution.

Use KL divergence or PSI (Population Stability Index) metric.
"""

import numpy as np
from scipy.stats import entropy

class DriftDetector:
    def __init__(self, reference_data, threshold=0.1):
        """
        reference_data: Training data distribution
        threshold: Drift alert threshold
        """
        self.reference_dist = self._compute_distribution(reference_data)
        self.threshold = threshold
    
    def _compute_distribution(self, data, bins=10):
        """Compute histogram distribution"""
        # Your implementation
        pass
    
    def detect_drift(self, current_data):
        """
        Returns:
            (is_drift: bool, score: float)
        """
        current_dist = self._compute_distribution(current_data)
        
        # Calculate KL divergence
        kl_div = entropy(current_dist, self.reference_dist)
        
        is_drift = kl_div > self.threshold
        
        return is_drift, kl_div
    
    def update_reference(self, new_reference_data):
        """Update reference distribution (after retraining)"""
        self.reference_dist = self._compute_distribution(new_reference_data)

# Usage
training_data = np.random.normal(0, 1, 10000)
detector = DriftDetector(training_data, threshold=0.1)

# Production data (no drift)
prod_data_good = np.random.normal(0, 1, 1000)
is_drift, score = detector.detect_drift(prod_data_good)
print(f"Drift: {is_drift}, Score: {score:.4f}")

# Production data (with drift)
prod_data_drift = np.random.normal(2, 1, 1000)  # Mean shifted
is_drift, score = detector.detect_drift(prod_data_drift)
print(f"Drift: {is_drift}, Score: {score:.4f}")
```

---

### Question 2: A/B Test Analysis
**Problem:**
Analyze A/B test results and determine winner.

```python
"""
Given metrics from model A and model B, determine if there's
a statistically significant difference.
"""

import numpy as np
from scipy import stats

def analyze_ab_test(metrics_a, metrics_b, metric_name="conversion_rate"):
    """
    metrics_a: List of metric values for model A
    metrics_b: List of metric values for model B
    
    Returns:
        {
            "winner": "A" or "B" or "No significant difference",
            "p_value": float,
            "confidence": float,
            "improvement": float (percentage)
        }
    """
    # Calculate statistics
    mean_a = np.mean(metrics_a)
    mean_b = np.mean(metrics_b)
    
    # T-test
    t_stat, p_value = stats.ttest_ind(metrics_a, metrics_b)
    
    # Determine winner
    if p_value < 0.05:  # 95% confidence
        if mean_b > mean_a:
            winner = "B"
            improvement = ((mean_b - mean_a) / mean_a) * 100
        else:
            winner = "A"
            improvement = ((mean_a - mean_b) / mean_b) * 100
    else:
        winner = "No significant difference"
        improvement = 0
    
    return {
        "winner": winner,
        "p_value": p_value,
        "confidence": (1 - p_value) * 100,
        "mean_a": mean_a,
        "mean_b": mean_b,
        "improvement": improvement
    }

# Example usage
# Model A metrics (control)
metrics_a = np.random.normal(0.15, 0.05, 1000)  # 15% conversion

# Model B metrics (treatment)
metrics_b = np.random.normal(0.18, 0.05, 1000)  # 18% conversion

result = analyze_ab_test(metrics_a, metrics_b)
print(result)
# Output: {"winner": "B", "p_value": 0.001, "improvement": 20%, ...}
```

---

### Question 3: Model Registry Design
**Problem:**
Design a simple model registry that tracks model versions.

```python
"""
Implement a model registry that:
- Stores model metadata
- Tracks versions
- Allows promoting models to production
- Records metrics
"""

from datetime import datetime
from dataclasses import dataclass
from typing import Dict, List, Optional
import json

@dataclass
class ModelMetadata:
    model_id: str
    version: str
    framework: str
    metrics: Dict[str, float]
    created_at: datetime
    created_by: str
    stage: str  # "development", "staging", "production"
    tags: Dict[str, str]

class ModelRegistry:
    def __init__(self):
        self.models: Dict[str, List[ModelMetadata]] = {}
    
    def register_model(
        self,
        model_id: str,
        version: str,
        framework: str,
        metrics: Dict[str, float],
        created_by: str,
        tags: Optional[Dict[str, str]] = None
    ) -> ModelMetadata:
        """Register a new model version"""
        # Your implementation
        pass
    
    def get_model(self, model_id: str, version: str) -> Optional[ModelMetadata]:
        """Get specific model version"""
        # Your implementation
        pass
    
    def get_production_model(self, model_id: str) -> Optional[ModelMetadata]:
        """Get current production model"""
        # Your implementation
        pass
    
    def promote_to_production(self, model_id: str, version: str):
        """Promote model version to production"""
        # Demote current production model
        # Promote new version
        # Your implementation
        pass
    
    def list_versions(self, model_id: str) -> List[ModelMetadata]:
        """List all versions of a model"""
        # Your implementation
        pass
    
    def search_models(self, tags: Dict[str, str]) -> List[ModelMetadata]:
        """Search models by tags"""
        # Your implementation
        pass

# Usage
registry = ModelRegistry()

# Register model
metadata = registry.register_model(
    model_id="recommendation_model",
    version="v1.0",
    framework="pytorch",
    metrics={"accuracy": 0.85, "f1": 0.82},
    created_by="user@monday.com",
    tags={"team": "ml", "use_case": "recommendations"}
)

# Get production model
prod_model = registry.get_production_model("recommendation_model")

# Promote to production
registry.promote_to_production("recommendation_model", "v1.0")
```

---

## 🎭 BEHAVIORAL QUESTIONS (STAR Method)

### Question 1: "Tell me about a time you improved system reliability"
**Use:** High-latency incident story from STAR templates

**Key Points to Mention:**
- Situation: Production issue affecting customers
- Task: Quick resolution needed
- Action: Systematic debugging, rollback, long-term fixes
- Result: Met SLA, prevented future issues

---

### Question 2: "Describe a time you had to make a trade-off decision"
**Example Framework:**
```
SITUATION:
"We needed to deploy a new model but faced a trade-off between
accuracy and latency."

TASK:
"As MLOps engineer, I needed to decide the right balance for
our product requirements."

ACTION:
"1. Gathered requirements from product team
 2. Benchmarked different model sizes
 3. Measured business impact of latency vs accuracy
 4. Created comparison matrix
 5. Presented options to stakeholders
 6. Decided on medium model with 2% accuracy drop but 50% faster"

RESULT:
"User satisfaction increased due to faster responses.
The 2% accuracy drop had minimal business impact.
Saved 40% on infrastructure costs."
```

---

### Question 3: "How do you prioritize when everything is urgent?"
**Framework:**
```
1. Assess impact and urgency (Eisenhower Matrix)
2. Communicate with stakeholders
3. Negotiate deadlines if needed
4. Delegate when possible
5. Focus on customer-facing issues first
6. Set clear expectations
```

**Example from STAR templates:**
Use "Conflicting Priorities" story

---

## 🎯 PRACTICE SCHEDULE

### Day 1: System Design
- [ ] Practice Question 1 (Recommendation System)
- [ ] Practice Question 2 (Model Serving Platform)
- [ ] Time yourself: 45 minutes each

### Day 2: Coding
- [ ] Exercise 1: Rate Limiter (15 min)
- [ ] Exercise 2: LRU Cache (15 min)
- [ ] Exercise 3: Log Aggregator (15 min)

### Day 3: DevOps Scenarios
- [ ] Scenario 1: Debug OOMKilled
- [ ] Scenario 2: Zero-downtime deploy
- [ ] Scenario 3: Custom autoscaling

### Day 4: MLOps
- [ ] Question 1: Drift detection
- [ ] Question 2: A/B test analysis
- [ ] Question 3: Model registry

### Day 5: Mock Interview
- [ ] Full system design (45 min)
- [ ] Coding exercise (15 min)
- [ ] Behavioral questions (30 min)

### Day 6: Review and Polish
- [ ] Review mistakes
- [ ] Practice weak areas
- [ ] Prepare questions for interviewer

---

## 📊 SELF-ASSESSMENT CHECKLIST

After practicing, rate yourself (1-5):

**System Design:**
- [ ] Can explain trade-offs clearly
- [ ] Consider scalability from start
- [ ] Include monitoring in designs
- [ ] Discuss cost implications
- [ ] Handle follow-up questions

**Coding:**
- [ ] Write syntactically correct code
- [ ] Consider edge cases
- [ ] Explain time/space complexity
- [ ] Test with examples
- [ ] Complete within 15 minutes

**DevOps:**
- [ ] Know Kubernetes concepts
- [ ] Can debug production issues
- [ ] Understand CI/CD pipelines
- [ ] Familiar with monitoring tools

**MLOps:**
- [ ] Explain ML lifecycle
- [ ] Know model deployment patterns
- [ ] Understand drift and monitoring
- [ ] Can discuss A/B testing

**Communication:**
- [ ] Think out loud
- [ ] Ask clarifying questions
- [ ] Explain reasoning
- [ ] Stay calm under pressure

---

## 💪 CONFIDENCE BUILDERS

**You Know:**
✅ Kubernetes and container orchestration
✅ CI/CD pipelines
✅ ML model deployment
✅ System design principles
✅ Python programming
✅ Monitoring and observability
✅ Cost optimization strategies

**You Have:**
✅ Production ML experience
✅ Problem-solving skills
✅ Ownership mindset
✅ Collaboration experience
✅ Learning agility

**You Can:**
✅ Design scalable systems
✅ Debug production issues
✅ Write clean code
✅ Communicate effectively
✅ Learn quickly

---

**Practice, practice, practice! You've got this! 🚀**
