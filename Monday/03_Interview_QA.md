# 🎯 MONDAY.COM MLOps PLATFORM ENGINEER - INTERVIEW Q&A

---

# PART A: JOB DESCRIPTION SKILL MAPPING

## A1. Requirements Priority Matrix

| Requirement | Priority | Interview Weight |
|-------------|----------|------------------|
| 5+ years ML/DevOps experience | MUST | Resume screen |
| Production ML deployment at scale | MUST | Deep dive (30%) |
| ML infrastructure management | MUST | System design (40%) |
| CI/CD, observability, rollbacks | MUST | Deep dive (15%) |
| Cloud (AWS), GPUs, containers | MUST | System design |
| Cost monitoring & optimization | HIGH | Deep dive (10%) |
| Cross-team collaboration | HIGH | Behavioral (5%) |
| Fine-tuning, training data mgmt | BONUS | Optional depth |

---

## A2. What Interviewers Are Really Evaluating

| Question Type | Surface Evaluation | Deeper Evaluation |
|---------------|-------------------|-------------------|
| System Design | Technical breadth | Can they own a complex system? |
| MLOps Deep Dive | Tool knowledge | Production mindset |
| Cost Optimization | Knows spot/reserved | Cost-aware culture |
| Incident Scenario | Technical debugging | Ownership, learning |
| Coding | Code correctness | Production quality |

---

## A3. Green Flags vs Red Flags

### 🟢 Green Flags (Hire Signals)
- Asks clarifying questions before designing
- Discusses tradeoffs unprompted
- Mentions failure modes and mitigation
- Quantifies impact ("reduced latency from 500ms to 100ms")
- Acknowledges limitations ("this won't work if...")
- Shows business awareness ("this matters because users...")
- Demonstrates ownership ("I noticed X and fixed it")
- Mentions monitoring proactively
- Has opinions backed by experience

### 🔴 Red Flags (No-Hire Signals)
- Jumps into solution without requirements
- Only knows happy path, no failure scenarios
- Name-drops tools without understanding when to use them
- Can't explain WHY they made decisions
- Blames others for production issues
- No production experience (only training)
- Ignores cost considerations
- Can't simplify complex concepts
- Gets defensive when pushed

---

## A4. Expected Question Themes (from Community Research)

| Theme | Example Questions |
|-------|-------------------|
| **Platform Ownership** | "How would you design a platform to allow DS to deploy models themselves?" |
| **Feature Store** | "What considerations go into building a centralized feature store?" |
| **Scalability** | "How would your design handle a 10x increase in traffic?" |
| **High Availability** | "What happens if your model service goes down?" |
| **Cost Optimization** | "How would you architect to minimize cloud costs?" |
| **Multi-Tenancy** | "Trade-offs between multi-tenant vs single-tenant model deployments?" |
| **Rollback** | "What's your rollback strategy if a deployment causes errors?" |
| **Collaboration** | "Describe taking a model from research to production" |
| **Conflict Resolution** | "How would you handle a model that can't meet latency requirements?" |

---

# PART B: TOP 10 INTERVIEW QUESTIONS

---

## Question 1: Design an ML Model Serving Platform for Multi-Tenant SaaS

### Why Asked
- Tests end-to-end system design thinking
- Evaluates multi-tenancy understanding
- Assesses production mindset
- Validates scalability knowledge

### Evaluation Weights
| Signal | Weight |
|--------|--------|
| Requirements gathering | 15% |
| Architecture completeness | 30% |
| Multi-tenant isolation | 20% |
| Scalability approach | 15% |
| Tradeoff reasoning | 20% |

### Common Mistakes
❌ Jumping to solution without clarifying requirements  
❌ Ignoring tenant isolation (data leakage risk)  
❌ Single model focus (not platform thinking)  
❌ Forgetting cost implications  
❌ No monitoring or rollback strategy  

### Strong Sample Answer

```
"Let me clarify requirements first. How many models? What's the latency 
SLA? Are models tenant-specific or shared?

Assuming ~tens of shared models, large multi-tenant scale, <100ms p95:

ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│  Client → API Gateway (auth, rate limit by tenant_id)           │
│       → Load Balancer                                           │
│       → Model Router (routes to correct model version)          │
│       → Model Server Pool (K8s deployment per model)            │
│       → Feature Store (tenant-partitioned keys)                 │
│       → Response                                                 │
└─────────────────────────────────────────────────────────────────┘

MULTI-TENANT ISOLATION:
• Feature store keys: tenant_id:user_id:feature_name
• Request context carries tenant_id (never cross-tenant data)
• Per-tenant rate limits to prevent noisy neighbors
• Audit logging with tenant attribution

PLATFORM-SPECIFIC (monday.com):
• If workflow uses GraphQL API: complexity budgeting + pagination
• Pin `API-Version` header to prevent schema surprises
• Respect rate limits and Retry-After headers

SCALING:
• HPA per model based on queue depth
• Separate GPU and CPU model pools
• Prediction caching with tenant+input hash as key

MONITORING:
• Latency, error rate, drift - segmented by tenant
• Alert if single tenant drives > 50% of errors

COST:
• Spot instances for 70% of GPU capacity
• Caching reduces inference volume by ~40%

TRADEOFFS:
• Chose shared models (simpler) over per-tenant (more customization)
• Would add tenant-specific routing for enterprise customers"
```

---

## Question 2: How Do You Handle Model Drift in Production?

### Why Asked
- Tests ML-specific production challenges
- Evaluates proactive monitoring
- Assesses incident response capability

### Evaluation Weights
| Signal | Weight |
|--------|--------|
| Understanding drift types | 20% |
| Detection strategy | 30% |
| Response automation | 25% |
| Business impact awareness | 25% |

### Common Mistakes
❌ Only mentioning data drift (ignoring prediction/concept drift)  
❌ Manual detection only  
❌ Retraining as only solution  
❌ Not connecting to business impact  
❌ No specific thresholds  

### Strong Sample Answer

```
"I approach drift detection across three dimensions:

1. DATA DRIFT (Input distribution changes)
   • Detection: Compare incoming features vs training baseline
   • Metrics: PSI, KL divergence per feature
   • Threshold: PSI > 0.2 → investigate, > 0.25 → alert
   • Implementation: Hourly batch job + Prometheus metrics

2. PREDICTION DRIFT (Output distribution changes)
   • Detection: Track prediction distribution over time
   • Metrics: Mean prediction, class distribution, confidence histogram
   • Example: If model suddenly predicts same item 50% of time, 
     something's wrong even if inputs look normal

3. CONCEPT DRIFT (Input-output relationship changes)
   • Detection: Monitor correlated business metrics
   • Metrics: CTR, conversion rate
   • Challenge: Need ground truth labels (often delayed)

RESPONSE STRATEGY:
┌─────────────────────────────────────────────────────────────────┐
│  Drift Detected                                                  │
│       │                                                          │
│       ▼                                                          │
│  Severity Assessment (PSI level, business impact)               │
│       │                                                          │
│       ├── LOW: Log, investigate next sprint                     │
│       ├── MEDIUM: Alert team, investigate within 24 hours       │
│       └── HIGH: Auto-rollback to previous model, page on-call   │
└─────────────────────────────────────────────────────────────────┘

REAL EXAMPLE:
We detected feature drift when a pipeline started returning nulls. 
PSI spiked, we auto-switched to fallback model while investigating. 
Root cause: upstream schema change. Added data validation that now 
catches similar issues before they reach the model."
```

---

## Question 3: Design a CI/CD Pipeline for ML Models

### Why Asked
- Core MLOps competency
- Tests understanding that ML CI/CD differs from software CI/CD
- Evaluates quality gates and rollback thinking

### Evaluation Weights
| Signal | Weight |
|--------|--------|
| Separation of code vs model pipelines | 20% |
| Quality gates | 25% |
| Deployment strategy | 25% |
| Rollback capability | 15% |
| Practical implementation | 15% |

### Common Mistakes
❌ Treating ML CI/CD same as regular software  
❌ No data/model validation gates  
❌ Big-bang deployments (no canary)  
❌ Manual rollback process  
❌ Not testing the model itself  

### Strong Sample Answer

```
"ML CI/CD requires three interconnected pipelines:

PIPELINE 1: CODE CI/CD
┌─────────────────────────────────────────────────────────────────┐
│  Git Push → Lint → Unit Test → Build Docker → Security Scan    │
│       → Push to Registry                                        │
│                                                                  │
│  Runs: Every commit | Tools: GitHub Actions, pytest, Trivy     │
└─────────────────────────────────────────────────────────────────┘

PIPELINE 2: TRAINING CI/CD
┌─────────────────────────────────────────────────────────────────┐
│  Trigger (schedule/data change/manual)                          │
│       ▼                                                          │
│  Data Validation (Great Expectations)                           │
│       │ GATE: Schema OK, no null spike, distributions OK        │
│       ▼                                                          │
│  Training (Kubeflow)                                            │
│       ▼                                                          │
│  Model Validation                                                │
│       │ GATE: Accuracy > baseline, latency < SLA, no bias       │
│       ▼                                                          │
│  Register in Model Registry (MLflow)                            │
│                                                                  │
│  Runs: Daily + drift trigger | Tools: Kubeflow, MLflow          │
└─────────────────────────────────────────────────────────────────┘

PIPELINE 3: DEPLOYMENT CI/CD
┌─────────────────────────────────────────────────────────────────┐
│  Promotion Request → Deploy Staging → Integration Tests         │
│       │ GATE: Shadow mode metrics OK                            │
│       ▼                                                          │
│  Canary (5%) → 30-min soak                                      │
│       │ GATE: Error rate < 1%, latency p99 < SLA                │
│       ▼                                                          │
│  Gradual Rollout (5% → 25% → 50% → 100%)                       │
│       │ GATE: Business metrics stable                           │
│       ▼                                                          │
│  Full Production                                                 │
│                                                                  │
│  Rollback: Auto if error > 5% or latency > 2x baseline          │
└─────────────────────────────────────────────────────────────────┘

KEY PRINCIPLE: Each stage has clear gates. Fail fast, never ship 
a degraded model to 100% of users."
```

---

## Question 4: How Would You Reduce ML Infrastructure Costs by 50%?

### Why Asked
- Cost awareness is critical
- Tests ability to quantify and prioritize
- Evaluates tradeoff reasoning

### Evaluation Weights
| Signal | Weight |
|--------|--------|
| Specific strategies | 25% |
| Quantification | 25% |
| Tradeoff awareness | 25% |
| Implementation experience | 25% |

### Common Mistakes
❌ Vague answers ("optimize things")  
❌ Single strategy only  
❌ Ignoring tradeoffs  
❌ No measurement approach  
❌ Not considering reliability impact  

### Strong Sample Answer

```
"I'd approach systematically, prioritizing by impact and effort:

PHASE 1: QUICK WINS (2 weeks, 20-30% savings)
┌─────────────────────────────────────────────────────────────────┐
│  1. RIGHT-SIZING                                                 │
│     Action: Downsize p3.8xlarge → p3.2xlarge where GPU util <50%│
│     Savings: ~40% on those instances | Risk: Low                │
│                                                                  │
│  2. SCALE TO ZERO (Dev/Staging)                                 │
│     Action: HPA minReplicas=0, KEDA for scale-up                │
│     Savings: ~80% dev/staging costs | Risk: Low (30s cold start)│
│                                                                  │
│  3. RESERVED CAPACITY (Baseline production)                     │
│     Action: 1-year Savings Plan for baseline                    │
│     Savings: 30-40% vs on-demand | Risk: Medium (commitment)    │
└─────────────────────────────────────────────────────────────────┘

PHASE 2: MEDIUM EFFORT (1-2 months, 15-20% more)
┌─────────────────────────────────────────────────────────────────┐
│  4. SPOT FOR TRAINING                                           │
│     Action: 80% spot with checkpointing                         │
│     Savings: 60-70% training | Risk: Medium                     │
│                                                                  │
│  5. PREDICTION CACHING                                          │
│     Action: Redis cache, input hash key, 1hr TTL                │
│     Savings: 30-50% inference reduction | Risk: Medium (stale)  │
│                                                                  │
│  6. REQUEST BATCHING                                            │
│     Action: Batch similar requests for GPU efficiency           │
│     Savings: 2-3x throughput/GPU | Risk: Low (added latency)    │
└─────────────────────────────────────────────────────────────────┘

PHASE 3: LONGER TERM (2-3 months, 10-15% more)
┌─────────────────────────────────────────────────────────────────┐
│  7. MODEL OPTIMIZATION                                          │
│     Action: Quantization (FP16), pruning                        │
│     Savings: Smaller GPUs | Risk: Medium-High (accuracy)        │
│                                                                  │
│  8. SMART ROUTING                                               │
│     Action: CPU for simple, GPU only for complex                │
│     Savings: 30% GPU fleet | Risk: Medium                       │
└─────────────────────────────────────────────────────────────────┘

MEASUREMENT:
• Dashboard: Cost per 1000 predictions (by model)
• Alert: Cost anomaly detection (>20% spike)
• Weekly review: Top drivers, optimization opportunities"
```

---

## Question 5: Design a Feature Store for Real-Time and Batch ML

### Why Asked
- Feature stores are critical ML infrastructure
- Tests online vs offline patterns
- Evaluates consistency thinking

### Evaluation Weights
| Signal | Weight |
|--------|--------|
| Dual storage architecture | 25% |
| Training/serving consistency | 25% |
| Performance requirements | 20% |
| Point-in-time correctness | 20% |
| Feature governance | 10% |

### Common Mistakes
❌ Single storage solution  
❌ Ignoring training/serving skew  
❌ No point-in-time correctness  
❌ Ignoring feature versioning  
❌ Overlooking operational complexity  

### Strong Sample Answer

```
"Requirements first: How many features? What entities? Latency SLA?

Assuming: 500 features, 10M users, <10ms online latency.

ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│  COMPUTATION                                                     │
│  ┌───────────────────┐    ┌───────────────────┐                │
│  │ Batch (Spark)     │    │ Streaming (Flink) │                │
│  │ • user_30d_stats  │    │ • session_count   │                │
│  │ • historical_aggs │    │ • recent_views    │                │
│  └─────────┬─────────┘    └─────────┬─────────┘                │
│            └──────────┬─────────────┘                           │
│                       ▼                                          │
│  STORAGE                                                         │
│  ┌───────────────────┐    ┌───────────────────┐                │
│  │ Online (Redis)    │    │ Offline (S3)      │                │
│  │ • <5ms latency    │    │ • For training    │                │
│  │ • Key: entity_id  │    │ • Point-in-time   │                │
│  │ • TTL: 24hrs      │    │ • Partitioned     │                │
│  └───────────────────┘    └───────────────────┘                │
│                       │                                          │
│                       ▼                                          │
│  REGISTRY                                                        │
│  • Schema definitions    • Lineage tracking                     │
│  • Ownership/docs        • Version history                      │
└─────────────────────────────────────────────────────────────────┘

TRAINING/SERVING CONSISTENCY:
• Same feature computation code for both paths
• Point-in-time joins prevent future leakage
• Feature versioning ensures reproducibility

MULTI-TENANT (monday.com):
• Keys include tenant_id: tenant:user:feature
• Isolation prevents cross-tenant data access"
```

---

## Question 6: How Do You Set Up GPU Autoscaling for Inference?

### Strong Sample Answer

```
"For GPU autoscaling, I use a combination of HPA with custom metrics:

CONFIGURATION:
┌─────────────────────────────────────────────────────────────────┐
│  Metrics to track:                                               │
│  • Primary: Inference queue depth (from Prometheus)             │
│  • Secondary: GPU utilization (from DCGM exporter)              │
│                                                                  │
│  Scaling rules:                                                  │
│  • Scale UP: Queue depth > 100 OR GPU util > 80% for 2 min     │
│  • Scale DOWN: Queue depth < 20 AND GPU util < 30% for 10 min  │
│                                                                  │
│  Constraints:                                                    │
│  • Min replicas: 2 (availability)                               │
│  • Max replicas: 20 (cost control)                              │
│  • Cool-down: 5 min (prevent thrashing)                         │
└─────────────────────────────────────────────────────────────────┘

NODE POOL STRATEGY:
• GPU nodes (p3.2xlarge): For inference
• CPU nodes: For preprocessing, API gateway
• Spot instances: 70% of GPU fleet
• On-demand: 30% baseline for reliability

COST OPTIMIZATION:
• Scale to zero in dev/staging (KEDA)
• Right-size based on actual GPU memory usage
• Batch requests to maximize GPU efficiency"
```

---

## Question 7: Walk Me Through a Production Incident

### Strong Sample Answer

```
"INCIDENT: Model latency spiked 10x during Monday morning traffic.

DETECTION (9:02 AM):
• PagerDuty alert: p99 latency > 2s (SLA: 200ms)
• Grafana showed spike correlating with traffic increase

TRIAGE (9:05 AM):
• Checked: Pod health (OK), GPU util (100% - maxed)
• Root cause: HPA wasn't scaling fast enough for traffic spike

MITIGATION (9:10 AM):
• Manually scaled to 10 replicas
• Enabled request queue with backpressure
• Latency recovered to normal

ROOT CAUSE ANALYSIS:
• HPA was configured with 5-minute evaluation window
• Monday 9 AM traffic spikes in 2 minutes
• Insufficient headroom for burst

PREVENTION:
1. Reduced HPA evaluation window to 1 minute
2. Added scheduled scaling for known traffic patterns
3. Implemented predictive scaling based on historical data
4. Added pre-warming for Monday mornings

RESULT:
No similar incidents in 6 months. Built runbook for traffic 
spikes that's now used across ML platform."
```

---

## Question 8: How Do You Ensure Reproducibility in ML?

### Strong Sample Answer

```
"Reproducibility requires versioning four things:

1. DATA VERSIONING
   • Tool: DVC or LakeFS
   • What: Dataset hash, point-in-time snapshots
   • Why: Same data → same model

2. CODE VERSIONING
   • Tool: Git with tagged releases
   • What: Training code, feature engineering, preprocessing
   • Why: Same code → same transformations

3. ENVIRONMENT VERSIONING
   • Tool: Docker with pinned dependencies
   • What: Python version, library versions, CUDA version
   • Why: Same environment → same execution

4. EXPERIMENT TRACKING
   • Tool: MLflow or W&B
   • What: Parameters, metrics, artifacts, random seeds
   • Why: Complete record of what produced the model

IMPLEMENTATION:
┌─────────────────────────────────────────────────────────────────┐
│  Every model in registry has:                                    │
│  • Git commit SHA                                                │
│  • Dataset version (DVC hash)                                    │
│  • Docker image digest                                           │
│  • MLflow run ID                                                 │
│  • Random seed used                                              │
│                                                                  │
│  To reproduce: Pull artifacts, run `mlflow run --run-id=XXX`    │
└─────────────────────────────────────────────────────────────────┘"
```

---

## Question 9: Compare SageMaker vs Self-Managed Kubernetes

### Strong Sample Answer

```
"The choice depends on team size, customization needs, and budget:

┌─────────────────────────────────────────────────────────────────┐
│  Factor          │ SageMaker          │ Self-Managed K8s       │
│  ────────────────┼────────────────────┼─────────────────────── │
│  Setup time      │ Hours              │ Weeks                  │
│  Operational     │ Low (managed)      │ High (need expertise)  │
│  Customization   │ Limited            │ Full control           │
│  Vendor lock-in  │ High               │ Low                    │
│  Cost at scale   │ Higher             │ Lower (optimizable)    │
│  Multi-cloud     │ No                 │ Yes                    │
└─────────────────────────────────────────────────────────────────┘

MY RECOMMENDATION:
• Small team (<5), quick start: SageMaker
• Large team, custom needs: Self-managed K8s
• Hybrid: SageMaker for training, K8s for serving

AT MONDAY.COM:
Given the JD mentions AWS + containers + GPUs + cost optimization,
I'd lean toward self-managed K8s with:
• EKS for orchestration
• Custom serving layer (more control over multi-tenancy)
• SageMaker for specific use cases (HPO, distributed training)"
```

---

## Question 10: How Do You Work with Data Scientists?

### Strong Sample Answer

```
"I see MLOps as an enablement role. My approach:

1. SHARED STANDARDS
   • Created ML readiness checklist with DS team
   • Before production: validate data quality, latency requirements,
     monitoring needs, fallback behavior
   • Reduced production issues by 60%

2. SELF-SERVICE PLATFORM
   • Built internal tools for model deployment
   • DS can deploy to staging with one command
   • Production requires platform team approval (guardrails)
   • Reduced deployment time from 2 weeks to 2 hours

3. CLEAR INTERFACES
   • DS owns: Model architecture, training code, feature selection
   • MLOps owns: Infrastructure, deployment, monitoring, scaling
   • Shared: Feature store, experiment tracking, model registry

4. FEEDBACK LOOPS
   • Weekly sync on production model health
   • Quarterly retro on platform improvements
   • Shared on-call for ML-related incidents

RESULT:
DS team can iterate faster while maintaining production standards.
They focus on model quality; I ensure reliability and scale."
```

---

# PART C: CODING EXERCISES

## C1. Rate Limiter (Sliding Window)

```python
import time
from collections import deque
from threading import Lock

class SlidingWindowRateLimiter:
    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = deque()
        self.lock = Lock()
    
    def allow_request(self) -> bool:
        with self.lock:
            now = time.time()
            # Remove expired timestamps
            while self.requests and self.requests[0] < now - self.window_seconds:
                self.requests.popleft()
            
            if len(self.requests) < self.max_requests:
                self.requests.append(now)
                return True
            return False

# Usage
limiter = SlidingWindowRateLimiter(max_requests=100, window_seconds=60)
if limiter.allow_request():
    # Process request
    pass
```

---

## C2. LRU Cache with TTL

```python
import time
from collections import OrderedDict
from threading import Lock

class LRUCacheWithTTL:
    def __init__(self, capacity: int, ttl_seconds: int):
        self.capacity = capacity
        self.ttl = ttl_seconds
        self.cache = OrderedDict()  # key -> (value, timestamp)
        self.lock = Lock()
    
    def get(self, key: str):
        with self.lock:
            if key not in self.cache:
                return None
            
            value, timestamp = self.cache[key]
            if time.time() - timestamp > self.ttl:
                del self.cache[key]
                return None
            
            # Move to end (most recently used)
            self.cache.move_to_end(key)
            return value
    
    def put(self, key: str, value) -> None:
        with self.lock:
            if key in self.cache:
                del self.cache[key]
            elif len(self.cache) >= self.capacity:
                self.cache.popitem(last=False)  # Remove oldest
            
            self.cache[key] = (value, time.time())

# Usage for prediction caching
cache = LRUCacheWithTTL(capacity=10000, ttl_seconds=3600)
```

---

## C3. PSI Drift Detector

```python
import numpy as np
from typing import List

def calculate_psi(baseline: List[float], current: List[float], 
                  buckets: int = 10) -> float:
    """
    Calculate Population Stability Index (PSI).
    PSI < 0.1: No significant change
    PSI 0.1-0.2: Moderate change
    PSI > 0.2: Significant change
    """
    # Create bins from baseline
    _, bin_edges = np.histogram(baseline, bins=buckets)
    
    # Calculate proportions
    baseline_counts, _ = np.histogram(baseline, bins=bin_edges)
    current_counts, _ = np.histogram(current, bins=bin_edges)
    
    baseline_pct = baseline_counts / len(baseline)
    current_pct = current_counts / len(current)
    
    # Avoid division by zero
    baseline_pct = np.clip(baseline_pct, 0.0001, None)
    current_pct = np.clip(current_pct, 0.0001, None)
    
    # PSI formula
    psi = np.sum((current_pct - baseline_pct) * 
                 np.log(current_pct / baseline_pct))
    return psi

# Usage
baseline_data = [...]  # Training distribution
current_data = [...]   # Production distribution
psi = calculate_psi(baseline_data, current_data)
if psi > 0.2:
    alert("Significant drift detected!")
```

---

## C4. Circuit Breaker

```python
import time
from enum import Enum
from threading import Lock

class CircuitState(Enum):
    CLOSED = "closed"      # Normal operation
    OPEN = "open"          # Failing, reject requests
    HALF_OPEN = "half_open"  # Testing recovery

class CircuitBreaker:
    def __init__(self, failure_threshold: int = 5, 
                 recovery_timeout: int = 30):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failures = 0
        self.last_failure_time = 0
        self.state = CircuitState.CLOSED
        self.lock = Lock()
    
    def call(self, func, fallback, *args, **kwargs):
        with self.lock:
            if self.state == CircuitState.OPEN:
                if time.time() - self.last_failure_time > self.recovery_timeout:
                    self.state = CircuitState.HALF_OPEN
                else:
                    return fallback(*args, **kwargs)
        
        try:
            result = func(*args, **kwargs)
            with self.lock:
                self.failures = 0
                self.state = CircuitState.CLOSED
            return result
        except Exception as e:
            with self.lock:
                self.failures += 1
                self.last_failure_time = time.time()
                if self.failures >= self.failure_threshold:
                    self.state = CircuitState.OPEN
            return fallback(*args, **kwargs)

# Usage
breaker = CircuitBreaker(failure_threshold=5, recovery_timeout=30)
result = breaker.call(
    func=ml_model.predict,
    fallback=rule_based_fallback,
    input_data=features
)
```

---

# PART D: BEHAVIORAL & COLLABORATION QUESTIONS

> **Key Theme:** MLOps interviews at product companies often resemble DevOps/SRE interviews and heavily test cross-functional collaboration.

---

## Question 11: Tell Me About Taking a Model from Research to Production

### Why Asked
- Tests cross-functional collaboration
- Evaluates understanding of the research → production gap
- Assesses communication skills

### Strong Sample Answer

```
"I worked with a data scientist on a recommendation model for our platform.

THE GAP:
• DS had a Jupyter notebook with 85% accuracy
• But: 2-second latency, no error handling, hardcoded paths

MY APPROACH:
1. REQUIREMENTS ALIGNMENT
   • Met with DS to understand what 'good' means
   • Agreed on: <100ms latency, 80% accuracy acceptable if faster
   • Defined fallback behavior together

2. TECHNICAL TRANSLATION
   • Converted notebook to modular Python package
   • Added input validation, error handling
   • Optimized: batch inference, caching → 50ms latency

3. PRODUCTION HARDENING
   • Containerized with pinned dependencies
   • Added monitoring (latency, accuracy, drift)
   • Built canary deployment pipeline

4. KNOWLEDGE TRANSFER
   • Created runbook with DS
   • Documented how to retrain and redeploy
   • DS can now push to staging independently

RESULT:
• Model launched in 3 weeks (vs. typical 6-8 weeks)
• Still running with 99.5% availability after 1 year
• Became template for 5 subsequent model launches"
```

---

## Question 12: How Do You Handle a Model That Can't Meet Latency Requirements?

### Why Asked
- Tests trade-off reasoning
- Evaluates problem-solving approach
- Assesses communication with stakeholders

### Strong Sample Answer

```
"First, I'd diagnose whether it's a model issue or infrastructure issue.

DIAGNOSTIC STEPS:
1. Profile the model: Where is time spent? (preprocessing, inference, postprocessing)
2. Check infrastructure: Is it CPU-bound? Memory-bound? Network latency?
3. Baseline: What latency is achievable with optimized setup?

SOLUTION OPTIONS (discuss trade-offs with DS):
┌─────────────────────────────────────────────────────────────────┐
│  OPTION              │ IMPACT        │ TRADE-OFF               │
│  ────────────────────┼───────────────┼─────────────────────────│
│  Model distillation  │ 2-3x speedup  │ Slight accuracy drop    │
│  Quantization (FP16) │ 1.5-2x        │ Minimal accuracy impact │
│  Async processing    │ UX change     │ User waits differently  │
│  Caching             │ 3-5x for hits │ Stale predictions       │
│  Simpler model       │ Varies        │ May need retraining     │
│  More hardware       │ Linear        │ Cost increase           │
└─────────────────────────────────────────────────────────────────┘

COMMUNICATION:
• Present options to DS with data: 'If we quantize, we lose 0.5% accuracy 
  but gain 40% latency improvement'
• Involve PM: 'Is 150ms acceptable? Or must we hit 100ms?'
• Document decision rationale for future reference

I never just say 'it can't be done' - I present options with trade-offs."
```

---

## Question 13: How Do You Handle a Flawed Model in Production?

### Why Asked
- Tests incident response
- Evaluates ownership and accountability
- Assesses learning mindset

### Strong Sample Answer

```
"When we detected a flawed model giving nonsense suggestions:

IMMEDIATE (First 10 minutes):
• Acknowledged alert, assessed severity (affecting 30% of users)
• Initiated rollback to previous model version (took 3 minutes)
• Posted status update to internal channel

INVESTIGATION (Next 2 hours):
• Compared model versions - new model had training data issue
• A data pipeline bug introduced corrupted records
• Model validated on hold-out set but hold-out was also corrupted

ROOT CAUSE:
• Insufficient data validation before training
• Hold-out set created from same pipeline (not independent)

PREVENTION:
1. Added Great Expectations checks on training data
2. Created separate, curated golden test set
3. Added shadow mode comparison (new vs old) before any rollout
4. Blameless postmortem shared with team

OUTCOME:
• Similar issues now caught in CI before reaching production
• Reduced production incidents by 70% over next quarter
• Postmortem process adopted by other teams"
```

---

## Question 14: Describe Your Approach to Platform Documentation

### Strong Sample Answer

```
"I believe documentation is a product, not an afterthought.

MY DOCUMENTATION STACK:

1. RUNBOOKS (For on-call)
   • Step-by-step troubleshooting for common issues
   • 'If you see X, do Y' format
   • Updated after every incident

2. GETTING STARTED GUIDES (For new DS)
   • 'Deploy your first model in 30 minutes'
   • Includes working example code
   • Video walkthrough for complex flows

3. ARCHITECTURE DOCS (For engineers)
   • Decision records (why we chose X over Y)
   • Component diagrams with data flows
   • Updated when architecture changes

4. API DOCS (Auto-generated)
   • OpenAPI specs for all platform APIs
   • Examples for every endpoint

KEY PRINCIPLES:
• Docs live with code (same repo, same PR)
• Broken docs = broken build (link checking in CI)
• Quarterly 'doc day' to refresh and prune
• Measure: Track doc page views, feedback scores"
```

---

# PART E: FINAL CHECKLIST

## Before the Interview

- [ ] Review monday.com platform facts (GraphQL, rate limits, mondayDB)
- [ ] Know AI features: AI Blocks, Sidekick, Risk Analyzer, Resource Optimizer
- [ ] Practice DECIDE framework out loud
- [ ] Know key numbers (latency targets, drift thresholds)
- [ ] Prepare 2-3 production incident stories
- [ ] Prepare 1-2 collaboration stories (DS ↔ MLOps)
- [ ] Review cost optimization strategies with numbers

## During the Interview

- [ ] Ask clarifying questions first
- [ ] State assumptions explicitly
- [ ] Draw diagrams (even simple boxes and arrows)
- [ ] Mention tradeoffs without being asked
- [ ] Include monitoring and rollback in designs
- [ ] Quantify with specific numbers

## Questions to Ask Them

1. "What ML models are currently in production?"
2. "What's the biggest MLOps challenge the team faces?"
3. "How do data scientists and MLOps engineers collaborate?"
4. "What does the current deployment pipeline look like?"
5. "What's the scale of inference requests?"

---

**Good luck! 🚀**
