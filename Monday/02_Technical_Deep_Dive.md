# 🎯 MONDAY.COM MLOps PLATFORM ENGINEER - TECHNICAL DEEP DIVE

---

# PART A: MONDAY.COM PLATFORM KNOWLEDGE

## A1. Verified Platform Facts

> **These are documented in official sources - safe to reference explicitly in interviews.**

### Platform API (GraphQL)
- **Endpoint:** `https://api.monday.com/v2`
- **Authentication:** Personal V2 API tokens in `Authorization` header
- **Permissions:** Mirror what the user can do in the UI
- **Docs:** https://developer.monday.com/api-reference/docs/authentication

### Rate Limiting System
| Limit Type | Details |
|------------|---------|
| **Complexity** | 5M points/minute (plan-dependent) |
| **Daily Calls** | Plan-dependent (Free: 200, Enterprise: 25K+) |
| **Minute** | Enterprise: 5K, Pro: 2.5K, Other: 1K |
| **Concurrency** | Enterprise: 250, Pro: 100, Other: 40 |
| **IP** | 5K requests per 10 seconds |

**Docs:** https://developer.monday.com/api-reference/docs/rate-limits

### API Versioning
- **Release Cycle:** Quarterly (Jan, Apr, Jul, Oct)
- **Active Versions:** 3 at any time (RC → Current → Maintenance)
- **How to Pin:** `API-Version` header (e.g., `2025-10`)
- **Docs:** https://developer.monday.com/api-reference/docs/api-versioning

### mondayDB (Data Infrastructure)
- **Purpose:** Custom data engine built for performance and scale
- **Key Design:** Separates **storage from compute** for horizontal scaling
- **Benefit:** Boards with 20K+ items load in seconds
- **Post:** https://monday.com/blog/product/introducing-mondaydb/

### Apps Framework
| Feature Type | Description |
|--------------|-------------|
| Board Views | Visualize board data |
| Item Views | Per-item custom views |
| Dashboard Widgets | Dashboard components |
| Integrations | External service connectors |
| AI Assistant | AI-powered app features |
| Custom Objects | Standalone views |

**Docs:** https://developer.monday.com/apps/docs/intro

### monday code (Hosting)
- **What:** Managed hosting for apps on monday.com infrastructure
- **Features:** Multi-region, secrets management, storage, queues, scheduler, logging, security scanning
- **Docs:** https://developer.monday.com/apps/docs/hosting-your-app-with-monday-code

---

## A2. Platform Architecture (Facts + Inferences)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MONDAY.COM PLATFORM ARCHITECTURE                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PRESENTATION LAYER                                                  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Web App • Mobile Apps • API Gateway                          │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                              │                                       │
│  APPLICATION LAYER                                                   │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  • GraphQL Platform API (documented)                          │  │
│  │  • Apps Framework (views/widgets/integrations/AI)             │  │
│  │  • (Inference) Event-driven patterns for automations          │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                              │                                       │
│  DATA LAYER                                                          │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  • mondayDB (documented)                                      │  │
│  │  • (Inference) Caching/search/indexing layers                 │  │
│  │  • (Inference) Analytics warehouse for BI + ML training       │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                              │                                       │
│  INFRASTRUCTURE LAYER                                                │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  • AWS (per JD)                                               │  │
│  │  • Kubernetes for container orchestration                     │  │
│  │  • Multi-region deployment                                    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## A3. ML/AI Use Cases at monday.com

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ML APPLICATIONS ECOSYSTEM                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. SMART AUTOMATIONS                                               │
│     • Automation Recommendations                                    │
│     • Trigger Prediction                                            │
│     • Anomaly Detection                                             │
│                                                                      │
│  2. INTELLIGENT SEARCH                                              │
│     • Semantic Search                                               │
│     • Auto-categorization                                           │
│     • Duplicate Detection                                           │
│                                                                      │
│  3. PROJECT ANALYTICS                                               │
│     • Completion Prediction                                         │
│     • Workload Balancing                                            │
│     • Bottleneck Detection                                          │
│                                                                      │
│  4. AI ASSISTANT + AGENTS + CONNECTORS                              │
│     • Summarization + Q&A (permission-aware)                        │
│     • NL Actions (create boards/items, update statuses)             │
│     • Agentic workflows (e.g., CRM)                                 │
│     • Connectors (Claude, Microsoft Copilot)                        │
│                                                                      │
│  5. USER ENGAGEMENT                                                 │
│     • Churn Prediction                                              │
│     • Feature Adoption                                              │
│     • Onboarding Optimization                                       │
│                                                                      │
│  6. SECURITY & TRUST                                                │
│     • Fraud Detection                                               │
│     • Bot Detection                                                 │
│     • Data Anomalies                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Deployment Patterns by Use Case

| Use Case | Latency | Pattern | Infrastructure |
|----------|---------|---------|----------------|
| Automation Recommendations | <100ms | Real-time API | K8s + GPU/CPU |
| Semantic Search | <200ms | Real-time API | Elasticsearch + Vector DB |
| Completion Prediction | <500ms | Real-time API | CPU-only K8s |
| AI Content Generation | <3s | Real-time API | GPU-heavy (LLM) |
| Churn Prediction | Daily | Batch Pipeline | Spark/Airflow |
| Anomaly Detection | <1min | Streaming | Flink/Kafka Streams |

---

## A4. Multi-Tenancy Considerations for MLOps

| Aspect | Challenge | Platform Design |
|--------|-----------|-----------------|
| **Data Isolation** | Customer A's data must never influence B's predictions | Tenant-partitioned feature stores, isolated pipelines |
| **Model Variants** | Enterprise customers may need custom models | Multi-model serving with tenant-aware routing |
| **Resource Fairness** | One tenant's usage shouldn't degrade others | Request quotas, resource isolation, priority queues |
| **Compliance** | GDPR, SOC2, HIPAA | Data residency, audit logging, model lineage |
| **Cost Attribution** | Track ML cost per tenant | Metering, cost tagging, chargeback |
### Multi-Tenant Model Strategy Trade-offs

| Approach | Pros | Cons | When to Use |
|----------|------|------|-------------|
| **Shared Global Model** | Easy to maintain, one deployment | Less personalized, data mixing concerns | Generic features (sentiment, translation) |
| **Tenant-Specific Models** | Strong isolation, customization | N× deployment/monitoring overhead | Enterprise customers, sensitive data |
| **Hybrid (Fine-tuned)** | Personalization + shared base | Complex versioning | High-value customers with unique patterns |

---

## A5. Platform Enablement Philosophy

> **Key Interview Theme:** Can you build platforms that *enable* others?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PLATFORM ENABLEMENT LAYERS                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SELF-SERVICE LAYER (Data Scientists)                               │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  • One-command deployment to staging                          │  │
│  │  • Experiment tracking UI                                     │  │
│  │  • Feature store catalog + discovery                          │  │
│  │  • Model registry with approval workflow                      │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  GUARDRAILS LAYER (Enforced by Platform)                            │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  • Automated validation gates (accuracy, latency, bias)       │  │
│  │  • Cost budgets per model/team                                │  │
│  │  • Security scanning (container, dependencies)                │  │
│  │  • Production deployment requires platform review             │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ABSTRACTION LAYER (Hide Complexity)                                │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  • K8s complexity hidden behind simple configs                │  │
│  │  • Auto-scaling, GPU allocation handled automatically         │  │
│  │  • Unified logging, monitoring, alerting                      │  │
│  │  • Standard templates for common patterns                     │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Key Question to Expect:** "How would you design a platform to allow data scientists to deploy models by themselves?"

**Strong Answer Elements:**
- Self-service for experimentation and staging
- Guardrails for production (not blocking, enabling safely)
- Abstract infrastructure complexity
- Clear ownership boundaries (DS owns model quality, MLOps owns reliability)
---

# PART B: SYSTEM DESIGN FRAMEWORK

## B1. The D.E.C.I.D.E. Framework

```
┌─────────────────────────────────────────────────────────────────────┐
│                    D.E.C.I.D.E. FRAMEWORK                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  D - DEFINE Requirements                              (5 minutes)   │
│      ├─ Functional: What does the system do?                        │
│      ├─ Non-functional: Scale, latency, availability                │
│      ├─ Constraints: Budget, timeline, existing infrastructure      │
│      └─ Platform API constraints (if relevant):                     │
│         • Does workflow read/write via GraphQL API?                 │
│         • What complexity budget and rate limits apply?             │
│         • Which API version should we pin?                          │
│                                                                      │
│  E - ESTIMATE Scale                                   (2 minutes)   │
│      ├─ Users/requests per second                                   │
│      ├─ Data volume (training, inference)                           │
│      └─ Storage and compute requirements                            │
│                                                                      │
│  C - CREATE High-Level Design                         (10 minutes)  │
│      ├─ Components and responsibilities                             │
│      ├─ Data flow diagram                                           │
│      └─ API contracts between components                            │
│                                                                      │
│  I - ITERATE on Deep Dive                             (15 minutes)  │
│      ├─ Pick 2-3 components to detail                               │
│      ├─ Database schemas, algorithms                                │
│      └─ Scaling strategies                                          │
│                                                                      │
│  D - DISCUSS Tradeoffs                                (5 minutes)   │
│      ├─ Why this approach vs alternatives?                          │
│      ├─ What are the limitations?                                   │
│      └─ What would you do differently with more resources?          │
│                                                                      │
│  E - ENSURE Reliability                               (3 minutes)   │
│      ├─ Failure modes and mitigation                                │
│      ├─ Monitoring and alerting                                     │
│      └─ Rollback and recovery strategies                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## B2. End-to-End ML Platform Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    END-TO-END ML PLATFORM ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PLATFORM API LAYER                                                          │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │  • GraphQL API at api.monday.com/v2                                    │ │
│  │  • Rate limits: complexity points, daily/minute/concurrency/IP         │ │
│  │  • Quarterly versioning (pin via API-Version header)                   │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                      │                                       │
│                                      ▼                                       │
│  DATA LAYER                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │  Event Stream    │  │  mondayDB + Lake │  │  Feature Store   │          │
│  │  (inferred)      │─▶│  (mondayDB is    │─▶│  (Redis + S3)    │          │
│  │                  │  │   documented)    │  │                  │          │
│  │  • User events   │  │  • Core data     │  │  • Online: ms    │          │
│  │  • Board ops     │  │  • Warehouse     │  │  • Offline: hrs  │          │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘          │
│                                      │                                       │
│                                      ▼                                       │
│  TRAINING LAYER                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │  Experiment      │  │  Training        │  │  Model           │          │
│  │  Tracking        │◀─│  Pipeline        │─▶│  Registry        │          │
│  │  (MLflow)        │  │  (Kubeflow)      │  │  (MLflow)        │          │
│  │                  │  │                  │  │                  │          │
│  │  • Parameters    │  │  • GPU jobs      │  │  • Versions      │          │
│  │  • Metrics       │  │  • Distributed   │  │  • Metadata      │          │
│  │  • Artifacts     │  │  • HPO           │  │  • Approval      │          │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘          │
│                                      │                                       │
│                                      ▼                                       │
│  SERVING LAYER                                                               │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │  API Gateway     │  │  Model Servers   │  │  Prediction      │          │
│  │  (Kong)          │─▶│  (K8s + Triton)  │─▶│  Cache (Redis)   │          │
│  │                  │  │                  │  │                  │          │
│  │  • Auth          │  │  • Auto-scale    │  │  • TTL: 1hr      │          │
│  │  • Rate limit    │  │  • GPU/CPU pools │  │  • Hit: ~40%     │          │
│  │  • A/B routing   │  │                  │  │                  │          │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘          │
│                                      │                                       │
│                                      ▼                                       │
│  OBSERVABILITY LAYER                                                         │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │  Metrics         │  │  Logging         │  │  Alerting        │          │
│  │  (Prometheus)    │  │  (ELK)           │  │  (PagerDuty)     │          │
│  │                  │  │                  │  │                  │          │
│  │  • Latency       │  │  • Structured    │  │  • Drift         │          │
│  │  • Drift         │  │  • Searchable    │  │  • Errors        │          │
│  │  • Business      │  │  • 30d retention │  │  • SLA breach    │          │
│  │  • API limits    │  │                  │  │  • API limits    │          │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## B3. Batch vs Real-Time Inference

### Decision Matrix

| Factor | Batch | Real-Time |
|--------|-------|-----------|
| Latency | Hours OK | Milliseconds needed |
| Freshness | Daily is fine | Must reflect now |
| Cost | Lower (scheduled) | Higher (always-on) |
| Complexity | Lower | Higher |
| Scalability | Easier (pre-compute) | Harder (on-demand) |

### Use Case Examples

**BATCH (Offline):**
- Weekly email recommendations
- Churn prediction scores (daily)
- Usage analytics reports
- Model retraining

```
S3 (Data) → Spark Job → Model → S3 (Predictions) → DB → Product
```

**REAL-TIME (Online):**
- Automation suggestions
- Smart search results
- Anomaly detection alerts
- AI assistant responses

```
User Request → API → Feature Store → Model Server → Response
```

### Platform API Constraint (monday-specific)

If your workflow reads/writes via the GraphQL API:

| Consideration | Implementation |
|---------------|----------------|
| Complexity budget | Track points; 5M/min cap (plan-dependent) |
| Pagination | Use cursors, avoid deep nesting |
| Version pinning | Pass `API-Version: 2025-10` header |
| Retry handling | Respect `Retry-After` header |

**HYBRID Pattern:**
- Batch: Pre-compute top 100 candidates per user (nightly)
- Real-time: Re-rank top 100 based on current context
- Benefits: Lower latency, lower cost, fresher results

---

## B4. Feature Store Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FEATURE STORE ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FEATURE COMPUTATION                                                 │
│  ┌─────────────────────────┐    ┌─────────────────────────┐        │
│  │  BATCH FEATURES         │    │  STREAMING FEATURES     │        │
│  │  (Spark Jobs - Daily)   │    │  (Flink/Kafka Streams)  │        │
│  │                         │    │                         │        │
│  │  • user_30d_activity    │    │  • user_session_cnt     │        │
│  │  • board_usage_stats    │    │  • recent_item_views    │        │
│  │  • historical_aggs      │    │  • real_time_clicks     │        │
│  └───────────┬─────────────┘    └───────────┬─────────────┘        │
│              └──────────────┬───────────────┘                       │
│                             ▼                                        │
│  FEATURE STORAGE                                                     │
│  ┌─────────────────────────┐    ┌─────────────────────────┐        │
│  │  ONLINE STORE           │    │  OFFLINE STORE          │        │
│  │  (Redis Cluster)        │    │  (S3 + Parquet)         │        │
│  │                         │    │                         │        │
│  │  • Latency: <5ms        │    │  • For training         │        │
│  │  • Key: entity_id       │    │  • Point-in-time joins  │        │
│  │  • TTL: 24 hours        │    │  • Partitioned by date  │        │
│  └─────────────────────────┘    └─────────────────────────┘        │
│                             │                                        │
│                             ▼                                        │
│  FEATURE REGISTRY                                                    │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  • Feature definitions (schema, types)                         │ │
│  │  • Ownership and documentation                                 │ │
│  │  • Lineage (data source → feature → model)                     │ │
│  │  • Version history + access controls                           │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Access Patterns

| Pattern | Description | Store |
|---------|-------------|-------|
| Real-time serving | Get features for user_123 at inference | Redis (online) |
| Training data | Historical features with point-in-time join | S3 (offline) |
| Backfill | Compute new feature for all historical data | Both |

---

## B5. CI/CD for ML Systems

### Three Interconnected Pipelines

```
┌─────────────────────────────────────────────────────────────────────┐
│  PIPELINE 1: CODE CI/CD (Same as regular software)                  │
├─────────────────────────────────────────────────────────────────────┤
│  Git Push → Lint → Unit Test → Build Docker → Security Scan → Push │
│                                                                      │
│  Runs: Every commit                                                  │
│  Tools: GitHub Actions, pytest, Trivy                                │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  PIPELINE 2: TRAINING CI/CD (ML-specific)                           │
├─────────────────────────────────────────────────────────────────────┤
│  Trigger (schedule/data change/drift)                               │
│       │                                                              │
│       ▼                                                              │
│  Data Validation (Great Expectations)                               │
│       │ GATE: Schema matches, no null spike, distributions OK       │
│       ▼                                                              │
│  Training (Kubeflow)                                                │
│       │                                                              │
│       ▼                                                              │
│  Model Validation                                                    │
│       │ GATE: Accuracy > baseline, latency < SLA, no bias           │
│       ▼                                                              │
│  Register in Model Registry (MLflow)                                │
│                                                                      │
│  Runs: Daily schedule + drift trigger                                │
│  Tools: Kubeflow, MLflow, Great Expectations                         │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  PIPELINE 3: DEPLOYMENT CI/CD                                       │
├─────────────────────────────────────────────────────────────────────┤
│  Model Promotion → Deploy Staging → Integration Tests               │
│       │ GATE: Shadow mode metrics OK                                │
│       ▼                                                              │
│  Canary (5% traffic)                                                │
│       │ GATE: 30-min soak, error rate < 1%, latency p99 < SLA       │
│       ▼                                                              │
│  Gradual Rollout (5% → 25% → 50% → 100%)                           │
│       │ GATE: Business metrics stable at each stage                 │
│       ▼                                                              │
│  Full Production                                                     │
│                                                                      │
│  Rollback: Auto if error > 5% or latency > 2x baseline             │
│  Tools: ArgoCD, Prometheus, custom canary controller                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## B6. Monitoring & Observability

### Metrics to Track

| Category | Metrics |
|----------|---------|
| **System** | CPU, memory, GPU utilization, network I/O |
| **ML-Specific** | Inference latency, prediction distribution, feature drift, confidence scores |
| **Business** | CTR, conversion, revenue impact, engagement |
| **Platform API** | Complexity consumed, daily call count, 429 rate |

### Drift Detection

| Drift Type | Detection Method | Threshold |
|------------|------------------|-----------|
| **Data Drift** | PSI, KL divergence per feature | PSI > 0.2 → investigate, > 0.25 → alert |
| **Prediction Drift** | Track output distribution over time | Shift > 10% |
| **Concept Drift** | Monitor business metrics | CTR/conversion drop |

### Alerting Matrix

| Severity | Condition | Action |
|----------|-----------|--------|
| CRITICAL | Error rate > 5% | Page on-call |
| CRITICAL | Latency p99 > 2s | Page on-call |
| CRITICAL | Model unavailable | Page on-call |
| HIGH | Accuracy drop > 5% | Slack + auto-rollback |
| HIGH | Data drift PSI > 0.25 | Slack + investigate |
| HIGH | API daily limit > 80% | Slack + throttle jobs |
| MEDIUM | GPU util < 30% | Slack (cost alert) |
| MEDIUM | API 429s spike | Slack + backoff check |
| LOW | Cache hit rate < 50% | Daily report |

---

## B7. Non-Functional Requirements Checklist

### Cost Optimization
- [ ] Spot instances for training (60-90% savings)
- [ ] Autoscaling (scale to zero in dev)
- [ ] Right-size based on utilization
- [ ] Model optimization (quantization, pruning)
- [ ] Prediction caching
- [ ] Cost per inference tracking
- [ ] Budget alerts

### Scalability
- [ ] Horizontal scaling (HPA)
- [ ] GPU upgrade path for training
- [ ] Feature store sharding
- [ ] CDN for static artifacts
- [ ] Async for non-critical predictions
- [ ] Request batching
- [ ] Multi-region deployment

### Reliability
- [ ] Circuit breaker (fallback to rule-based)
- [ ] Graceful degradation
- [ ] Health checks (liveness, readiness)
- [ ] Automatic rollback
- [ ] Multi-AZ deployment
- [ ] DR plan (RTO, RPO)
- [ ] Chaos engineering

### Security
- [ ] Encryption (rest + transit)
- [ ] IAM (least privilege)
- [ ] Secrets management
- [ ] Network isolation (VPC)
- [ ] Container scanning
- [ ] Audit logging
- [ ] Input validation

### Compliance
- [ ] GDPR: Right to explanation, data deletion
- [ ] SOC2: Audit trails, access controls
- [ ] Model lineage tracking
- [ ] Data retention policies
- [ ] PII handling
- [ ] Model governance approvals

---

**→ For interview questions and sample answers: See `03_Interview_QA.md`**
