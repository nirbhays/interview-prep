# 🎯 MONDAY.COM INTERVIEW - CORE TOPICS FOCUS SHEET
## Quick Reference for Last-Minute Review

---

## ⚡ TOP 10 MOST LIKELY QUESTIONS

### 1. SYSTEM DESIGN: "Design an ML recommendation system for Monday.com users"
**Key Points:**
- Multi-tenant isolation (separate data per customer)
- Feature store (user behavior, board usage, integration patterns)
- Model serving (real-time API + batch processing)
- A/B testing framework
- Monitoring (prediction quality, latency, drift)
- Scale: 150K+ customers, millions of users

**Architecture:**
```
User Request → API Gateway → Load Balancer → Model Serving (K8s)
                ↓
         Feature Store (Redis/DynamoDB)
                ↓
         Model Registry (MLflow)
                ↓
         Monitoring (Prometheus/Grafana)
```

---

### 2. KUBERNETES: "How would you deploy an ML model in Kubernetes?"

**Answer Structure:**
```yaml
# 1. Containerize model
Dockerfile:
- Base image with Python/ML libs
- Copy model files
- Expose port 8080
- Health check endpoint

# 2. Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  containers:
  - name: model-server
    image: model:v1.0
    resources:
      requests:
        memory: "2Gi"
        cpu: "1"
      limits:
        memory: "4Gi"
        cpu: "2"
    livenessProbe:
      httpGet:
        path: /health
        port: 8080

# 3. Service
kind: Service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080

# 4. HPA for autoscaling
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target: 70%
```

**Key Points:**
- Resource limits (prevent OOM)
- Health checks (liveness/readiness)
- Rolling updates (zero downtime)
- Autoscaling (handle traffic spikes)
- ConfigMaps for model config
- Secrets for API keys

---

### 3. CI/CD: "Design a CI/CD pipeline for ML models"

**Pipeline Stages:**
```
1. CODE STAGE
   ├── Git push/PR
   ├── Linting (pylint, black)
   ├── Unit tests (pytest)
   └── Code review

2. MODEL TRAINING
   ├── Trigger on data change
   ├── Distributed training (Kubeflow)
   ├── Hyperparameter tuning
   ├── Model validation
   └── Register in MLflow

3. BUILD & TEST
   ├── Docker build
   ├── Security scan (Trivy)
   ├── Integration tests
   └── Performance tests

4. DEPLOY TO STAGING
   ├── Deploy to staging K8s
   ├── Smoke tests
   ├── Load tests
   └── A/B test preparation

5. PRODUCTION DEPLOYMENT
   ├── Canary (10% traffic)
   ├── Monitor metrics (30 min)
   ├── Gradual rollout (50% → 100%)
   └── Automated rollback if issues

6. POST-DEPLOYMENT
   ├── Monitoring alerts
   ├── Model performance tracking
   └── Drift detection
```

**Tools:**
- GitLab CI / GitHub Actions
- ArgoCD for GitOps
- MLflow for model registry
- Kubernetes for orchestration

---

### 4. MODEL DRIFT: "How do you detect and handle model drift?"

**Detection Methods:**
1. **Data Drift** (Input distribution changes)
   - Track feature statistics (mean, std, percentiles)
   - KL divergence vs training data
   - Population Stability Index (PSI)

2. **Concept Drift** (Input-output relationship changes)
   - Monitor prediction accuracy over time
   - Compare with ground truth labels
   - Business metrics degradation

3. **Prediction Drift** (Output distribution changes)
   - Track prediction distribution
   - Sudden shifts in predicted classes

**Handling Strategy:**
```python
# Monitoring
if kl_divergence > threshold:
    alert("Data drift detected")
    trigger_retraining()

if accuracy < baseline - 5%:
    alert("Model performance degraded")
    initiate_investigation()

# Automated Retraining
def retraining_pipeline():
    # 1. Fetch recent data (last 90 days)
    # 2. Validate data quality
    # 3. Retrain model
    # 4. Evaluate on holdout set
    # 5. If better, deploy via CI/CD
    # 6. If worse, alert team
```

**Implementation:**
- Daily drift checks
- Weekly retraining schedule
- A/B test new model vs old
- Gradual rollout with monitoring

---

### 5. COST OPTIMIZATION: "How would you reduce GPU costs by 50%?"

**Strategy:**
1. **Autoscaling**
   - Scale to zero during off-hours
   - HPA based on request queue
   - Cluster Autoscaler for nodes

2. **Spot Instances**
   - 70% of capacity on Spot (70% savings)
   - Handle interruptions gracefully
   - Multiple instance types for availability

3. **Model Optimization**
   - Quantization (FP32 → FP16)
   - Model pruning
   - Knowledge distillation
   - Batch inference

4. **Smart Routing**
   - CPU for simple requests
   - GPU only when necessary
   - Request batching

5. **Reserved Capacity**
   - Savings Plans for baseline (30% discount)
   - Analyze usage patterns
   - Right-size instances

**Metrics to Track:**
- Cost per 1000 inferences
- GPU utilization %
- p95 latency
- Success rate

---

### 6. MONITORING: "What metrics would you monitor for an ML service?"

**Four Golden Signals + ML Specific:**

**1. LATENCY**
- p50, p95, p99 response time
- Breakdown: network, preprocessing, inference, postprocessing
- Target: <200ms for real-time

**2. TRAFFIC**
- Requests per second
- Requests by endpoint
- Geographic distribution

**3. ERRORS**
- HTTP error rate (4xx, 5xx)
- ML-specific errors (timeout, out of memory)
- Failed predictions

**4. SATURATION**
- CPU/memory/GPU utilization
- Queue length
- Connection pool usage

**ML-SPECIFIC:**
- **Model Quality**
  - Prediction accuracy (if labels available)
  - Prediction distribution
  - Feature drift scores
  - Business metrics (CTR, conversion)

- **Data Quality**
  - Missing features
  - Out-of-range values
  - Data pipeline health

**Alerting Rules:**
```
CRITICAL:
- Error rate > 5%
- p95 latency > 1 second
- Service unavailable

WARNING:
- Drift score > threshold
- Accuracy drop > 3%
- GPU utilization > 90%
```

---

### 7. A/B TESTING: "Design an A/B testing framework for ML models"

**Architecture:**
```
User Request
    ↓
API Gateway
    ↓
Traffic Splitter (Feature Flag Service)
    ├─── 90% → Model A (Control)
    └─── 10% → Model B (Treatment)
    ↓
Log Predictions & Outcomes
    ↓
Analytics Pipeline
    ↓
Statistical Analysis
```

**Implementation:**
```python
class ABTestFramework:
    def __init__(self):
        self.feature_flags = FeatureFlagService()
        self.metrics_logger = MetricsLogger()
    
    def route_request(self, user_id, context):
        # Consistent hashing for user assignment
        variant = self.feature_flags.get_variant(
            experiment_id="model_v2_test",
            user_id=user_id,
            traffic_split={"control": 90, "treatment": 10}
        )
        
        if variant == "treatment":
            prediction = model_v2.predict(context)
        else:
            prediction = model_v1.predict(context)
        
        # Log for analysis
        self.metrics_logger.log({
            "user_id": user_id,
            "variant": variant,
            "prediction": prediction,
            "timestamp": now()
        })
        
        return prediction
    
    def analyze_results(self, experiment_id):
        # After collecting data
        results_a = get_metrics("control")
        results_b = get_metrics("treatment")
        
        # Statistical significance test
        p_value = ttest(results_a, results_b)
        
        if p_value < 0.05 and results_b > results_a:
            return "WINNER: Treatment"
        else:
            return "INCONCLUSIVE or Control better"
```

**Key Considerations:**
- Minimum sample size calculation
- Statistical significance (p < 0.05)
- Multiple metrics (primary + guardrails)
- Duration (typically 1-2 weeks)
- Consistent user assignment

---

### 8. FEATURE STORE: "Design a feature store for ML systems"

**Requirements:**
1. **Low latency** for online serving (<10ms)
2. **Consistency** between training and serving
3. **Point-in-time correctness** for historical features
4. **Scalability** for millions of features

**Architecture:**
```
┌─────────────────────────────────────────────┐
│           Feature Store                      │
├─────────────────────────────────────────────┤
│                                              │
│  ONLINE SERVING (Redis/DynamoDB)             │
│  - Key: user_id                              │
│  - Value: {feature1: val, feature2: val}     │
│  - TTL: 24 hours                             │
│  - Latency: 1-5ms                            │
│                                              │
│  OFFLINE STORAGE (S3/Data Lake)              │
│  - Partitioned by date                       │
│  - Parquet format                            │
│  - For training/batch inference              │
│                                              │
│  FEATURE ENGINEERING PIPELINE                │
│  - Airflow/Kubeflow for orchestration        │
│  - Spark for batch computation               │
│  - Kafka for streaming features              │
│                                              │
│  REGISTRY & METADATA                         │
│  - Feature definitions                       │
│  - Lineage tracking                          │
│  - Version control                           │
│                                              │
└─────────────────────────────────────────────┘
```

**Example Features for Monday.com:**
```python
User Features:
- user_active_boards_7d
- user_items_created_30d
- user_integrations_count
- user_automation_usage_score

Board Features:
- board_update_frequency
- board_collaborator_count
- board_item_count
- board_activity_trend

Context Features:
- time_of_day
- day_of_week
- user_timezone
```

**Implementation:**
- **Feast** (open-source feature store)
- **AWS SageMaker Feature Store**
- **Custom solution** with Redis + S3

---

### 9. CODING: "Implement an LRU cache for ML predictions"

```python
from collections import OrderedDict
from datetime import datetime, timedelta
import threading

class MLPredictionCache:
    """
    LRU cache with TTL for ML model predictions
    Thread-safe implementation
    """
    def __init__(self, capacity=1000, ttl_seconds=3600):
        self.capacity = capacity
        self.ttl = timedelta(seconds=ttl_seconds)
        self.cache = OrderedDict()
        self.timestamps = {}
        self.lock = threading.Lock()
        self.hits = 0
        self.misses = 0
    
    def _is_expired(self, key):
        """Check if cache entry has expired"""
        if key not in self.timestamps:
            return True
        return datetime.now() - self.timestamps[key] > self.ttl
    
    def get(self, key):
        """
        Get prediction from cache
        Returns None if not found or expired
        """
        with self.lock:
            if key not in self.cache or self._is_expired(key):
                self.misses += 1
                if key in self.cache:
                    del self.cache[key]
                    del self.timestamps[key]
                return None
            
            # Move to end (most recently used)
            self.cache.move_to_end(key)
            self.hits += 1
            return self.cache[key]
    
    def put(self, key, value):
        """
        Add prediction to cache
        Evicts LRU item if at capacity
        """
        with self.lock:
            if key in self.cache:
                self.cache.move_to_end(key)
            else:
                if len(self.cache) >= self.capacity:
                    # Evict least recently used
                    oldest_key = next(iter(self.cache))
                    del self.cache[oldest_key]
                    del self.timestamps[oldest_key]
            
            self.cache[key] = value
            self.timestamps[key] = datetime.now()
    
    def get_stats(self):
        """Get cache statistics"""
        total = self.hits + self.misses
        hit_rate = self.hits / total if total > 0 else 0
        return {
            "hits": self.hits,
            "misses": self.misses,
            "hit_rate": f"{hit_rate:.2%}",
            "size": len(self.cache)
        }
    
    def clear(self):
        """Clear all cache entries"""
        with self.lock:
            self.cache.clear()
            self.timestamps.clear()
            self.hits = 0
            self.misses = 0

# Usage example
cache = MLPredictionCache(capacity=10000, ttl_seconds=3600)

# Cache key from input features
def get_cache_key(features):
    """Generate cache key from features"""
    import hashlib
    import json
    feature_str = json.dumps(features, sort_keys=True)
    return hashlib.md5(feature_str.encode()).hexdigest()

# In model serving
def predict_with_cache(model, features):
    cache_key = get_cache_key(features)
    
    # Try cache first
    cached_result = cache.get(cache_key)
    if cached_result is not None:
        return cached_result
    
    # Cache miss - run model
    prediction = model.predict(features)
    
    # Store in cache
    cache.put(cache_key, prediction)
    
    return prediction

# Monitor cache performance
print(cache.get_stats())
# Output: {"hits": 8500, "misses": 1500, "hit_rate": "85.00%", "size": 1500}
```

**Key Points:**
- OrderedDict for LRU ordering
- Thread-safe with locks
- TTL for cache invalidation
- Metrics for monitoring
- O(1) get and put operations

---

### 10. SCALABILITY: "How would you scale Monday.com's ML inference from 1K to 100K RPS?"

**Scaling Strategy:**

**1. HORIZONTAL SCALING (Kubernetes)**
```yaml
# Deployment with auto-scaling
HPA:
  minReplicas: 10
  maxReplicas: 100
  targetCPUUtilization: 70%
  
# Use custom metrics
metrics:
- type: Pods
  pods:
    metric:
      name: requests_per_second
    target:
      type: AverageValue
      averageValue: 1000

# Cluster Autoscaler
- Add nodes automatically
- Multiple AZs for availability
```

**2. CACHING**
```
Level 1: CDN (CloudFront) - static responses
Level 2: Redis Cluster - frequently requested predictions
Level 3: In-memory cache - per-pod cache

Expected hit rate: 60-80%
Reduces backend load by 70%
```

**3. LOAD BALANCING**
```
Internet
    ↓
AWS ALB / NLB
    ↓
Ingress Controller
    ↓
K8s Service (with session affinity)
    ↓
Pods (100 replicas)

Strategies:
- Round robin
- Least connections
- Consistent hashing (for cache efficiency)
```

**4. DATABASE OPTIMIZATION**
```
Features Storage:
- Read replicas (5+)
- Partition by user_id
- Use Redis for hot data
- DynamoDB for scalability

Model Storage:
- S3 with CloudFront
- Model caching on pods
- Lazy loading
```

**5. BATCH PROCESSING**
```
For requests that can wait:
- Queue requests (SQS/Kafka)
- Batch inference (32-64 samples)
- Process in background workers
- Better GPU utilization
```

**6. REQUEST OPTIMIZATION**
```
# Batch requests from client
POST /predictions/batch
{
  "requests": [
    {"id": "1", "features": {...}},
    {"id": "2", "features": {...}},
    ...
  ]
}

# Server processes in single inference call
predictions = model.predict_batch(all_features)

# Reduces network overhead
# Better GPU utilization
# Lower latency per request
```

**7. MONITORING AT SCALE**
```
Metrics:
- RPS per pod
- p95 latency
- Error rate
- CPU/Memory/GPU usage
- Cache hit rate

Alerts:
- Latency > 500ms for 5 minutes
- Error rate > 1%
- Pod failure rate > 5%

Auto-remediation:
- Restart unhealthy pods
- Scale up if latency high
- Circuit breaker for dependencies
```

**Cost Estimate:**
```
100K RPS = 8.64B requests/day

With 80% cache hit rate:
- Backend: 20K RPS
- ~200 pods with 100 RPS each
- Mix of spot (70%) and on-demand (30%)

Estimated cost: $15K-20K/month
Cost per 1M requests: $0.20
```

---

## 🔧 QUICK TECHNICAL ANSWERS

### "Difference between Deployment and StatefulSet?"

**Deployment:**
- For stateless applications
- Pods are interchangeable
- Random pod names (app-5f6g7h8)
- No persistent storage guarantee
- **Use for**: ML inference servers, web apps

**StatefulSet:**
- For stateful applications
- Pods have stable identity
- Ordered pod names (app-0, app-1, app-2)
- Persistent volumes follow pods
- **Use for**: Databases, model training jobs with checkpoints

---

### "How do you handle secrets in Kubernetes?"

```yaml
# 1. Create secret
kubectl create secret generic ml-api-keys \
  --from-literal=api-key=abc123 \
  --from-literal=db-password=xyz789

# 2. Use in pod
containers:
- name: model-server
  env:
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: ml-api-keys
        key: api-key

# 3. Or mount as file
volumes:
- name: secrets
  secret:
    secretName: ml-api-keys
volumeMounts:
- name: secrets
  mountPath: /etc/secrets
  readOnly: true
```

**Best Practices:**
- Never commit secrets to Git
- Use external secret management (AWS Secrets Manager, HashiCorp Vault)
- Rotate secrets regularly
- Encrypt secrets at rest
- Use RBAC to limit access

---

### "Blue-Green vs Canary Deployment?"

**Blue-Green:**
```
Blue (v1) ─── 100% traffic
Green (v2) ─ deployed, 0% traffic

Test Green → Switch 100% to Green → Rollback to Blue if issues

Pros: Instant rollback, easy
Cons: Double infrastructure, all-or-nothing
```

**Canary:**
```
v1 ─── 90% traffic
v2 ─── 10% traffic (canary)

Monitor → 50% → 100% (gradual)

Pros: Lower risk, gradual rollout
Cons: More complex, longer deployment
```

**For ML Models: Prefer Canary**
- Models can fail in subtle ways
- Need time to collect metrics
- Gradual rollout reduces blast radius

---

### "How to monitor model accuracy in production?"

**Challenges:**
- Ground truth labels not immediately available
- Labeling is expensive

**Solutions:**
1. **Proxy Metrics** (Real-time)
   - Click-through rate
   - User engagement
   - Business conversions
   - User feedback (thumbs up/down)

2. **Human Labeling** (Batch)
   - Sample predictions daily
   - Send to labeling service
   - Compare with model output
   - Track accuracy over time

3. **Delayed Ground Truth**
   - For some domains, truth emerges later
   - Example: Did user complete project on time?
   - Join predictions with outcomes weekly

4. **A/B Testing**
   - Compare new model vs baseline
   - Use business metrics as proxy
   - Statistical significance testing

**Implementation:**
```python
# Log predictions
logger.log({
    "prediction_id": uuid,
    "model_version": "v2.3",
    "input_features": features,
    "prediction": result,
    "timestamp": now,
    "user_id": user_id
})

# Later, join with ground truth
accuracy = calculate_accuracy(
    predictions=get_predictions(last_7_days),
    ground_truth=get_labels(last_7_days)
)

if accuracy < threshold:
    alert("Model accuracy degraded!")
```

---

## 📊 KEY NUMBERS TO REMEMBER

### Latency Targets
- Real-time API: <200ms p95
- Batch processing: <1 hour
- Feature serving: <10ms
- Model loading: <30 seconds

### Availability
- Production SLA: 99.9% (8.76 hours downtime/year)
- Monday.com likely aims for 99.95%+

### Scale
- 100K RPS = 8.64 billion requests/day
- 1M users → ~1TB of features
- Model size: 100MB-5GB typically

### Cost
- GPU instance: $3-8/hour (varies)
- Spot discount: 70%
- Reserved: 30-50% discount
- Target: <$0.01 per 1000 requests

---

## 🎯 LAST-MINUTE CHECKLIST

- [ ] Review system design framework
- [ ] Practice one coding problem
- [ ] Know your STAR stories
- [ ] Understand Monday.com product
- [ ] Prepare 3 questions to ask
- [ ] Test video setup
- [ ] Have paper and pen ready
- [ ] Water nearby
- [ ] Quiet space confirmed

---

## 💡 INTERVIEW TIPS

1. **Think Out Loud**
   - Explain your reasoning
   - Don't go silent
   - Ask for hints if stuck

2. **Clarify Requirements**
   - Better to ask than assume
   - Shows thoughtful approach
   - Prevents wrong solution

3. **Start Simple**
   - MVP first
   - Then optimize
   - Discuss trade-offs

4. **Use Real Numbers**
   - Makes answers concrete
   - Shows practical experience
   - Easier to reason about

5. **Admit When You Don't Know**
   - "I'm not familiar with X, but here's how I'd approach it..."
   - Shows honesty
   - Demonstrates problem-solving

6. **Be Product-Minded**
   - Consider user impact
   - Discuss business value
   - Not just technical correctness

---

## ⚡ IF YOU FORGET EVERYTHING, REMEMBER THIS:

**System Design:**
1. Clarify requirements
2. Start with high-level architecture
3. Discuss trade-offs
4. Add monitoring

**Coding:**
1. Understand the problem
2. Explain your approach
3. Write clean code
4. Test with examples

**MLOps:**
1. Model lifecycle: Train → Deploy → Monitor
2. Always include monitoring and rollback
3. Scale horizontally with Kubernetes
4. Optimize costs

**Communication:**
1. Think out loud
2. Ask questions
3. Be honest
4. Stay calm

---

**YOU'VE GOT THIS! 🚀**
