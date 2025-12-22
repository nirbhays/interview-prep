# 🚀 MONDAY.COM INTERVIEW - QUICK PRACTICE CHEAT SHEET
## Your 1-Page Last-Minute Reference

---

## ⚡ THE ESSENTIALS (MEMORIZE THESE)

### System Design Template (Use This Every Time)
```
1. CLARIFY (5 min)
   - Scale? Users? Data volume?
   - Read vs write heavy?
   - Latency requirements?
   - Multi-tenant? (YES for Monday.com)

2. HIGH-LEVEL (15 min)
   - Draw boxes: Client → LB → Services → DB
   - Mention: API Gateway, Cache, Queue

3. DEEP DIVE (20 min)
   - Choose 2-3 components
   - Talk about: Scaling, Failures, Monitoring

4. WRAP UP (5 min)
   - Trade-offs discussed
   - Mention what you'd improve
```

### Kubernetes Commands (Run Without Thinking)
```bash
# Deploy
kubectl apply -f deployment.yaml
kubectl get pods -w
kubectl describe pod <name>
kubectl logs -f <pod>

# Scale
kubectl scale deployment app --replicas=5
kubectl autoscale deployment app --min=2 --max=10 --cpu-percent=70

# Update
kubectl set image deployment/app app=image:v2
kubectl rollout status deployment/app
kubectl rollout undo deployment/app

# Debug
kubectl exec -it <pod> -- /bin/bash
kubectl port-forward <pod> 8080:8080
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Python Coding Patterns (15-Min Problems)
```python
# Pattern 1: Two Pointers
def two_sum(nums, target):
    left, right = 0, len(nums) - 1
    while left < right:
        curr = nums[left] + nums[right]
        if curr == target: return [left, right]
        elif curr < target: left += 1
        else: right -= 1

# Pattern 2: Hash Map
def group_anagrams(words):
    from collections import defaultdict
    groups = defaultdict(list)
    for word in words:
        key = ''.join(sorted(word))
        groups[key].append(word)
    return list(groups.values())

# Pattern 3: Sliding Window
def max_subarray_sum(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum
    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i-k] + arr[i]
        max_sum = max(max_sum, window_sum)
    return max_sum

# Pattern 4: BFS/DFS
from collections import deque
def bfs(graph, start):
    visited = set([start])
    queue = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return visited
```

---

## 🎯 TOP 5 SYSTEM DESIGN SCENARIOS

### 1. ML Recommendation System (Most Likely!)
```
Components:
├── Feature Store (Redis) - user behavior, board usage
├── Model Server (K8s) - serve predictions
├── A/B Testing - compare models
├── Monitoring - drift detection, latency
└── Feedback Loop - retrain on user actions

Scaling:
- Shard by customer_id (multi-tenant)
- Cache popular features
- Batch predictions for offline use
- Stream real-time events (Kafka)

Talk About:
✓ Multi-tenancy isolation
✓ Model versioning
✓ Canary deployments
✓ Feature engineering pipeline
```

### 2. Real-Time Analytics Dashboard
```
Pipeline:
Events → Kafka → Spark/Flink → Time-series DB → Dashboard
         ↓
      S3 (backup)

Key Decisions:
- Time-series DB: InfluxDB or Prometheus
- Pre-aggregate common queries
- WebSocket for real-time updates
- Cache last 1hr in Redis

Monday.com Context:
- Track board activity
- Automation execution stats
- Integration health
```

### 3. Notification System
```
Components:
├── Priority Queue (RabbitMQ/SQS)
├── Worker Pool (scale by queue depth)
├── Rate Limiter (per user/channel)
├── Delivery Tracking (retries, failures)
└── Template Engine

Channels:
- Email (batch low-priority)
- Push (real-time high-priority)
- In-app (WebSocket)
- Slack/Teams (integration)

Handle Failures:
- Dead letter queue
- Exponential backoff
- Circuit breaker
```

### 4. Search/Autocomplete Service
```
Architecture:
User input → API → Elasticsearch → Results
              ↓
         Cache (Redis)
         
Features:
- Fuzzy matching
- Typo tolerance  
- Relevance ranking
- Personalization (user history)

Optimization:
- Index sharding
- Query caching
- Prefix trees for autocomplete
- Update index async
```

### 5. File Storage/Processing
```
Flow:
Upload → S3 → Event → Lambda/Worker → Process → Store metadata
         ↓
      Virus scan

Processing:
- Image: resize, thumbnail (ImageMagick)
- Video: transcode (FFmpeg)
- Document: extract text (Tika)

Scale:
- Pre-signed URLs (direct to S3)
- CDN for delivery (CloudFront)
- Queue for async processing
- Dedup by hash
```

---

## 🔧 MLOPS ESSENTIALS

### Model Lifecycle (Say This Confidently)
```
1. TRAIN
   - Experiment tracking (MLflow)
   - Hyperparameter tuning (Optuna)
   - Version control (DVC)

2. VALIDATE
   - Test set metrics (F1, AUC, etc.)
   - Cross-validation
   - Fairness checks

3. DEPLOY
   - Containerize (Docker)
   - Orchestrate (K8s)
   - A/B test (50/50 split)

4. MONITOR
   - Prediction drift (PSI score)
   - Data drift (KS test)
   - Latency (p95 < 100ms)
   - Error rate (< 1%)

5. RETRAIN
   - Trigger: drift detected OR weekly schedule
   - Auto-approve if metrics improve
   - Rollback on regression
```

### Drift Detection Code (Write This Fast)
```python
import numpy as np
from scipy import stats

def detect_drift(reference_data, current_data, threshold=0.1):
    """
    KS test for distribution drift
    Returns: True if drift detected
    """
    statistic, p_value = stats.ks_2samp(reference_data, current_data)
    return p_value < threshold

def psi_score(reference, current, buckets=10):
    """
    Population Stability Index
    PSI < 0.1: No drift
    PSI 0.1-0.2: Moderate drift
    PSI > 0.2: Significant drift
    """
    ref_counts, bins = np.histogram(reference, bins=buckets)
    curr_counts, _ = np.histogram(current, bins=bins)
    
    ref_pct = ref_counts / len(reference)
    curr_pct = curr_counts / len(current)
    
    # Avoid log(0)
    ref_pct = np.where(ref_pct == 0, 0.0001, ref_pct)
    curr_pct = np.where(curr_pct == 0, 0.0001, curr_pct)
    
    psi = np.sum((curr_pct - ref_pct) * np.log(curr_pct / ref_pct))
    return psi

# Example usage
if __name__ == "__main__":
    # Training data distribution
    train_data = np.random.normal(0, 1, 1000)
    
    # Production data (with drift)
    prod_data = np.random.normal(0.5, 1.2, 500)
    
    if detect_drift(train_data, prod_data):
        print("⚠️  Drift detected - retrain needed")
        psi = psi_score(train_data, prod_data)
        print(f"PSI: {psi:.3f}")
```

---

## 📊 KEY NUMBERS & METRICS

### Latency Targets
```
Real-time APIs:      p95 < 100ms, p99 < 500ms
Batch jobs:          Complete in < 1hr
Model inference:     < 50ms per prediction
Database queries:    < 10ms (indexed)
```

### Scale Estimates
```
Monday.com:
- 150K+ customers
- Millions of users
- Billions of data points

Assume:
- 10K requests/sec (peak)
- 1TB new data/day
- 99.9% uptime SLA
```

### Resource Sizing (K8s)
```
Small service:    0.5 CPU, 512MB RAM
Medium service:   1 CPU, 2GB RAM
ML model:         2 CPU, 4GB RAM
Database:         4 CPU, 16GB RAM
```

---

## 🗣️ BEHAVIORAL STAR FRAMEWORK

### Template (Use for Every Story)
```
SITUATION (15 sec)
"At [Company], we had [problem]..."

TASK (10 sec)
"My role was to [responsibility]..."

ACTION (30 sec) ⭐ MOST IMPORTANT
"I took these steps:
1. First, I...
2. Then I...
3. Finally, I..."

RESULT (15 sec)
"This led to [quantified outcome]:
- Reduced X by Y%
- Improved Z
- Team recognized..."

LEARNING (10 sec) - Optional
"What I learned was..."
```

### Quick Stories to Prepare (2-3 min each)
1. **Technical Challenge** - Solved scaling issue
2. **Leadership** - Led project or mentored
3. **Conflict** - Disagreed with approach, resolved it
4. **Failure** - Mistake and how you recovered
5. **Innovation** - Introduced new tech/process

---

## 💡 MONDAY.COM SPECIFIC

### What They Care About
- **Multi-tenancy** - Isolate customer data
- **Scalability** - Handle growth
- **AI/Automation** - Their new focus
- **Developer experience** - Clean APIs
- **Reliability** - Enterprise customers

### Questions to Ask Them
```
Technical:
"What's the biggest scaling challenge you're facing?"
"How do you handle multi-tenant data isolation?"
"What's your ML/AI roadmap?"

Team:
"What does success look like in the first 6 months?"
"How do teams collaborate on cross-cutting features?"
"What's the on-call rotation like?"

Culture:
"What's unique about Monday.com's engineering culture?"
"How do you balance innovation with stability?"
```

---

## 🎯 DAY-BEFORE CHECKLIST

### Tonight
- [ ] Review this cheat sheet (30 min)
- [ ] Skim Core_Topics_Focus_Sheet.md (15 min)
- [ ] Practice 1 system design problem out loud (30 min)
- [ ] Review your STAR stories (20 min)
- [ ] Prepare 3 questions to ask them
- [ ] Get 8 hours of sleep!

### Morning Of
- [ ] Eat breakfast
- [ ] Test your camera/mic
- [ ] Have water nearby
- [ ] Pull up this cheat sheet
- [ ] Take 3 deep breaths

### During Interview
- [ ] **Think out loud** - Don't go silent
- [ ] **Ask clarifying questions** - Show you think about requirements
- [ ] **Draw diagrams** - Visual > words
- [ ] **Discuss trade-offs** - Show maturity
- [ ] **Admit what you don't know** - Then propose how you'd learn

---

## 🔥 QUICK CONFIDENCE BOOSTERS

### If Stuck on System Design
```
"Let me think about this systematically:
1. What are the functional requirements?
2. What's the scale we need to handle?
3. Let me start with a simple design and iterate..."
```

### If Asked Something You Don't Know
```
"I haven't worked with [X] directly, but here's how 
I'd approach it based on [similar thing]:
1. [Reasonable assumption]
2. [Logical next step]
3. [How I'd validate]"
```

### To Buy Time
```
"That's a great question. Let me think through 
the trade-offs here..."

"Let me make sure I understand correctly - 
you're asking about [rephrase]?"

"Before I answer, can I clarify [aspect]?"
```

---

## 🎓 FINAL TIPS

### DO ✅
- Start simple, then add complexity
- Mention monitoring/observability
- Talk about failure scenarios
- Use real numbers (even estimates)
- Connect to Monday.com's context

### DON'T ❌
- Jump to code without design
- Ignore the interviewer's hints
- Over-engineer simple problems
- Forget to ask questions
- Panic if you don't know something

---

## 📞 EMERGENCY REMINDERS

**Breathe** - You know this stuff

**They want you to succeed** - Interviewer is your ally

**Communication > Perfection** - Explain your thinking

**You've prepared** - Trust your preparation

---

## 🚀 YOU'VE GOT THIS!

Remember: They're evaluating:
1. **Can you think through problems?** (System design)
2. **Can you code?** (Coding round)
3. **Can you collaborate?** (Behavioral)
4. **Do you fit the culture?** (All rounds)

Show enthusiasm for Monday.com's mission, be yourself, and demonstrate your skills. **Good luck!** 🎉
