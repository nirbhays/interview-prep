# 🎯 MONDAY.COM MLOps PLATFORM ENGINEER - QUICK REFERENCE
> **Use this for last-minute review before your interview**

---

## ⚡ MONDAY.COM PLATFORM FACTS (Safe to Reference)

| Topic | Key Facts |
|-------|-----------|
| **Scale** | ~250,000 customers globally (Work OS platform) |
| **API** | GraphQL at `api.monday.com/v2` |
| **Auth** | Personal V2 API tokens in `Authorization` header |
| **Rate Limits** | Complexity points, daily/minute/concurrency/IP limits |
| **Versioning** | Quarterly releases; pin via `API-Version` header |
| **Data Layer** | **mondayDB** - internal data engine (storage ↔ compute separated) |
| **Apps** | Views, widgets, integrations, AI assistant features |
| **Hosting** | **monday code** - managed runtime with secrets, queues, scheduler |

### 🤖 Known AI Features (from product docs)
- **AI Blocks** - No-code AI components (categorize, summarize, translate, sentiment)
- **Monday Sidekick** - AI assistant for board interactions
- **Project Risk Analyzer** - Flags at-risk projects
- **Resource Optimizer** - Recommends right people for tasks
- **Digital Workers** - Sales forecasting, deal insights

**🔗 Official Sources:**
- API Docs: developer.monday.com/api-reference/docs
- Apps Framework: developer.monday.com/apps/docs/intro
- mondayDB: monday.com/blog/product/introducing-mondaydb/

---

## 🧠 THE D.E.C.I.D.E. FRAMEWORK (System Design)

| Step | Time | Focus |
|------|------|-------|
| **D**efine | 5 min | Requirements, constraints, **API limits if relevant** |
| **E**stimate | 2 min | Scale, data volume, compute needs |
| **C**reate | 10 min | High-level architecture, data flow |
| **I**terate | 15 min | Deep dive on 2-3 components |
| **D**iscuss | 5 min | Tradeoffs, alternatives, limitations |
| **E**nsure | 3 min | Reliability, monitoring, rollback |

---

## 📊 KEY METRICS TO KNOW

| Category | Metrics |
|----------|---------|
| **Latency** | p50, p95, p99 inference time |
| **ML-Specific** | PSI (drift), prediction distribution, feature stats |
| **Business** | CTR, conversion, engagement, retention |
| **Platform API** | Complexity consumed, daily call count, 429 rate |

**Drift Thresholds:**
- PSI > 0.2 → Investigate
- PSI > 0.25 → Alert + possible rollback

---

## 🚀 DEPLOYMENT STRATEGIES

| Strategy | Use When | Risk |
|----------|----------|------|
| **Canary** | Most model updates | Low - gradual rollout |
| **Blue-Green** | Need instant rollback | Medium - 2x resources |
| **Shadow** | Testing new models | Low - no user impact |
| **Feature Flag** | Kill switch needed | Low - instant toggle |

**Rollout Pattern:** 5% → 25% → 50% → 100%

**Auto-Rollback Triggers:**
- Error rate > 5%
- Latency > 2x baseline
- Business metrics drop > 5%

---

## 💰 COST OPTIMIZATION STRATEGIES

| Strategy | Savings | Risk |
|----------|---------|------|
| Spot instances (training) | 60-90% | Interruption |
| Reserved/Savings Plans | 30-60% | Commitment |
| Right-sizing | 20-50% | Performance |
| Scale to zero (dev) | ~80% | Cold start |
| Prediction caching | 30-50% inference reduction | Staleness |
| Request batching | 2-3x GPU efficiency | Added latency |

---

## 🏗️ QUICK ARCHITECTURE TEMPLATES

### ML Serving (draw this fast)
```
Client → API Gateway → Load Balancer → [Model Server Pods] → Response
                            ↓
                     Feature Store (Redis)
                            ↓
                     Model Registry (S3)
```

### Full ML Pipeline
```
Data Source → Ingestion → Feature Eng → Feature Store
                                             ↓
                          Training Pipeline ← Trigger
                                ↓
                          Model Registry
                                ↓
                          CI/CD → Staging → Production
                                                ↓
                                           Monitoring → Alert → Retrain
```

---

## 🎯 INTERVIEW FOCUS AREAS (from community research)

| Focus Area | What They're Testing | Key Phrases to Use |
|------------|---------------------|--------------------|
| **Platform Ownership** | Can you own E2E ML platform? | "enable data scientists to self-serve" |
| **Scalability** | Handle 10x traffic growth | "horizontal scaling", "auto-scaling" |
| **Reliability** | High availability design | "failure modes", "graceful degradation" |
| **Cost Awareness** | Treat cost as key metric | "cost per prediction", "spot instances" |
| **Trade-off Reasoning** | Justify decisions in context | "it depends on X", "if Y then..." |
| **Cross-Functional** | Bridge research ↔ production | "worked with DS to productionize" |

---

## ✅ INTERVIEW CHECKLIST

### Before Answering System Design:
- [ ] Clarify requirements (functional + non-functional)
- [ ] Ask about scale (users, requests/sec, data volume)
- [ ] Ask about constraints (latency SLA, budget, existing infra)
- [ ] If relevant: "Does this interact with the platform API?"

### Strong Answer Signals:
- [ ] Quantify with numbers ("p99 < 100ms", "99.9% availability")
- [ ] Discuss tradeoffs unprompted
- [ ] Mention failure modes and mitigation
- [ ] Include monitoring and rollback
- [ ] Show cost awareness

### Red Flags to Avoid:
- ❌ Jumping to solution without requirements
- ❌ Single approach without alternatives
- ❌ No mention of monitoring
- ❌ Vague scale ("large" without numbers)
- ❌ Only happy path, no failure handling

---

## 🔢 NUMBERS TO REMEMBER

| Metric | Target |
|--------|--------|
| Feature store latency (online) | < 10ms |
| Model inference (recommendations) | < 100ms p95 |
| Drift detection alert | Within 1-4 hours |
| Rollback time | < 5 minutes |
| GPU utilization target | > 70% |
| A/B test minimum duration | 1-2 weeks |

---

## 📝 MONDAY-SPECIFIC TALKING POINTS

When designing ML systems at monday.com, mention:

1. **API Complexity Budgeting** - "If the workflow reads/writes via the GraphQL API, we'd budget complexity points and use pagination"

2. **Version Pinning** - "We'd pin the API version header to prevent schema surprises during rollouts"

3. **Multi-Tenant Isolation** - "Feature store keys would include tenant_id to prevent data leakage across 250K+ customers"

4. **mondayDB Context** - "Since mondayDB separates storage from compute, we could leverage that for efficient feature extraction"

5. **AI Blocks Architecture** - "AI features are exposed as modular 'AI Blocks' that plug into workflows - the ML platform must ensure these scale reliably across workspaces"

6. **Product-Specific Models** - "Features like Project Risk Analyzer or Resource Optimizer likely use predictive models trained on monday's rich dataset of work patterns"

7. **Hybrid Model Strategy** - "Some features use in-house models (trained on monday data), others integrate external LLM APIs (summarization, translation) - platform handles both"

---

**→ For detailed content: See `02_Technical_Deep_Dive.md` and `03_Interview_QA.md`**
