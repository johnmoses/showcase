# John Moses — Full-Stack AI Engineer · Co-Founder @ ShopStack360

> I build AI into products end-to-end — from LLM pipelines and agent systems to the backends, mobile apps, and infrastructure that run them in production. Built a GPT from scratch in PyTorch. Fine-tuned BERT/T5/Mistral across 6 domains. Designed autonomous multi-agent systems before LangChain existed.

📹 **[Watch Portfolio Loom (90s)](https://www.loom.com/share/16bbb05b07b641509eb2a3a2d68f45a0)**
&nbsp;·&nbsp; 🐙 **[github.com/johnmoses](https://github.com/johnmoses)**
&nbsp;·&nbsp; 🌐 **[shopstack360.com](https://shopstack360.com)**

---

## Production Systems

### Life Reports — Church Attendance Platform
> 10,000+ users in week one · FastAPI + Flutter + Next.js · AWS Lightsail · $20/month

A full-stack attendance reporting platform for hierarchical church organizations (Fellowship → Zone → District → Region → State). Built offline-first — works without internet, syncs when connected. Security: guest, scoped event worker, and 3-tier admin roles enforced via FastAPI RBAC at the dependency injection layer.

**AI highlights:**
- **Unified Chat** — LangGraph supervisor + 6 specialist agents in one compiled StateGraph. Single interface for report submission and data querying in natural language. Supervisor classifies intent, report agent does multi-turn field collection, 5 query agents (insights, list, forecast, comparison, missing) each render to text / WhatsApp / PDF / Excel. Redis-backed session with in-process fallback.
- **Voice capture pipeline** — Whisper (real-time transcription) → GPT-4o-mini (structured field extraction via function calling) → regex fallback running in parallel. Offline-first with server-side recovery on reconnect. Handles Nigerian multilingual input (Hausa, Yoruba, Igbo, Efik, Idoma)
- **Attendance anomaly detection** — flags consecutive member absences and center-level attendance drops across all centers automatically
- **Outreach intelligence** — engagement scoring, churn prediction, and message variant generation by tone
- **Statistical trend analysis** — Redis-cached insights with cold-start handling for new centers

**DevOps highlights:**
- ADR-driven deployment decisions documented before any infrastructure was touched
- Docker Compose with per-service memory limits, PostgreSQL tuned for a $20 Lightsail instance
- GitHub Actions CI/CD, Nginx reverse proxy, fail2ban hardening, kernel-level SYN flood protection
- Sliding window rate limiter with real IP extraction behind proxy

**Code excerpts:**
- [Voice capture pipeline](snippets/voice_pipeline.md) — hybrid inference, multilingual NLP, offline resilience
- [Anomaly detection agent](snippets/anomaly_detection_agent.md) — autonomous absence + drop detection
- [Offline-first sync](snippets/offline_sync.md) — Flutter + Next.js dual-layer queue
- [Production infrastructure](snippets/devops_infrastructure.md) — Docker, Lightsail, hardening, rate limiting
- [Outreach AI](snippets/outreach_ai.md) — engagement scoring, churn prediction, priority queue

**Pre-AI-era foundation (built without AI assistance):**
- [Custom transformer fine-tuning](snippets/transformer_finetuning.md) — BERT intent classifier (44 classes, 87.5% accuracy) + T5 text-to-SQL (ROUGE-L 0.961 PEFT/LoRA variant, 118K samples) · 3 models published to HuggingFace · all runs tracked in Weights & Biases
- [Conversational agent](snippets/conversational_agent.md) — multi-turn dialog state machine with entity extraction and autonomous DB writes via WebSocket (agentic pattern before LangChain existed)

---

### ShopStack360 — Universal Commerce Operating System
> Co-Founder & Lead Engineer (team of 5) · FastAPI + Flutter + Next.js · PostgreSQL + Redis · Docker · Railway

A full-stack B2B/B2C commerce platform connecting manufacturers, distributors, retailers, and shoppers across Nigeria and West Africa. Incubated in Abuja with 20+ live retailers, scaling to the Lagos industrial corridor.

**AI highlights:**
- **5-agent autonomous fleet** — Smart Reorder, Supplier Intelligence, Price Optimization, Inventory Advisor, Product Recommendations — orchestrated via LangGraph with an eval harness (pass rates persisted to Redis) and an intervention tracker (human override rate + trend per agent per 7-day window)
- **Multi-provider AI strategy** — pluggable provider architecture (rule-based, LangChain hybrid, AWS Bedrock) activated by feature flags; zero downstream changes when provider switches
- **Production MCP server** — wraps the agent fleet as typed tools (stdio + HTTP transport); open-source [mcp-foundations](https://github.com/johnmoses/mcp-foundations) repo (5 progressive apps) demonstrating MCP protocol from basic FastMCP through full-stack agent bridges
- **Arbitrage engine** — 3-layer price scanner (internal + market scouts + external), landed cost calculator (product + shipping + customs + last-mile), opportunity scorer (0–100 composite)
- **Borderless logistics AI** — route optimization, rider matching, demand forecasting (hourly heatmaps), dynamic surge pricing
- **Social intelligence signals** — tier-specific feed items generated from live transaction data; agent health signals surface in business-tier feeds

**Cloud & infrastructure:**
- Lightsail is the stable core — 7 services (backend, AI, web, DB, Redis, MCP, autoheal) run on Docker Compose with self-managed equivalents for every AWS capability; the platform never depends on a managed service being available
- ECS is a detachable burst layer — containers are ECS-compatible by design (health endpoints, stateless, stdout logging), but ECS only attaches for heavy workloads (model retraining, batch ML inference, large arbitrage scans); if ECS has a cold start, task failure, or cost spike, Lightsail handles the same workload in degraded-but-running mode
- Lambda sits between them as a pure scheduler — fires the right target (ECS task or Lightsail endpoint) based on availability; owns no business logic
- Bedrock, RDS, and S3 follow the same pattern: each has a Lightsail-side fallback (local inference, self-managed Postgres, local artifacts); switching is a feature flag, not a rewrite
- 3-layer observability: infrastructure metrics + model quality (eval harness) + data drift detection (TFDV) as retraining signal
- 24h auto-retraining loop — reloads recommendation engine in-place on success, keeps previous version on failure

**Full-stack highlights:**
- 3-provider payment routing (Paga / Paystack / Flutterwave) — intelligent routing saves 45% on processing costs
- Escrow with milestone releases for B2B, wallet system for instant settlements
- 7-level ambassador engine with anti-fraud detection and dormancy downgrade
- Community commerce: group buy lifecycle with per-participant escrow and trust scoring

*Private repo — architecture overview.*

---

## Public AI Work

### Foundations (Theory → Code)
| Repo | What it covers |
|------|---------------|
| [llm-foundations](https://github.com/johnmoses/llm-foundations) | LLM internals, fine-tuning, prompt engineering, inference optimization |
| [rag-foundations](https://github.com/johnmoses/rag-foundations) | RAG pipelines, vector stores, chunking strategies, retrieval evaluation |
| [ai-agents-foundations](https://github.com/johnmoses/ai-agents-foundations) | Agent loops, tool use, multi-agent coordination, autonomous control |
| [mcp-foundations](https://github.com/johnmoses/mcp-foundations) | Model Context Protocol — 5 progressive apps, stdio + HTTP transport, full-stack agent bridges |

### Applied AI Systems
| Repo | What it covers |
|------|---------------|
| [ultra-learning](https://github.com/johnmoses/ultra-learning) | AI-powered adaptive learning system — Flask API (LLM + RAG + MCP) · Next.js web · React Native mobile |
| [sure-health](https://github.com/johnmoses/sure-health) | Multi-party AI healthcare platform — Flask API (agents + LLM + RAG) · Next.js web · React Native mobile · 90%+ FHIR compliance |
| [easy-finance](https://github.com/johnmoses/easy-finance) | AI-powered financial platform — FastAPI (wealth + community + security services) · Next.js web · React Native mobile |
| [zero-to-hero-python-ai](https://github.com/johnmoses/zero-to-hero-python-ai) | End-to-end ML/AI roadmap — CNNs, RNNs, Transformers, LLMs, Agentic AI |

### Specializations
| Repo | Credential | Score |
|------|-----------|-------|
| [coursera-genai-agents-specialization](https://github.com/johnmoses/coursera-genai-agents-specialization) | Building GenAI Applications and Agents (6 courses) | **All 100%** |
| [coursera-nlp-specialization](https://github.com/johnmoses/coursera-nlp-specialization) | Coursera NLP Specialization | C1–C3 100%, C4 97.5% |
| [coursera-mlops-specialization](https://github.com/johnmoses/coursera-mlops-specialization) | Coursera MLOps for Production | C1–C3 100%, C4 98% |
| [coursera-ai-4-medicine-specialization](https://github.com/johnmoses/coursera-ai-4-medicine-specialization) | Coursera AI for Medicine (3 courses) | **All 100%** |

### DevOps
| Repo | What it covers |
|------|---------------|
| [jenkins-docker-compose-flask](https://github.com/johnmoses/jenkins-docker-compose-flask) | Jenkins CI/CD with Docker Compose |
| [jenkins-docker-flask](https://github.com/johnmoses/jenkins-docker-flask) | Jenkins + Docker pipeline |

---

## Stack

**AI/ML:** LLMs · SLMs · RAG · Agentic AI · MCP · LangGraph · AutoGen · CrewAI · Transformer Fine-Tuning (BERT, T5, GPT-2, Mistral) · PEFT/LoRA · QLoRA · Collaborative Filtering · Time-Series Forecasting · NLP · Computer Vision · Weights & Biases

**Backend:** FastAPI · Flask · PostgreSQL · Redis · pgvector · Qdrant · SQLite · REST · WebSockets · SSE

**Frontend:** Next.js · TypeScript · Tailwind CSS · React

**Mobile:** Flutter · Dart · React Native · Offline-first (SQLite + sync queue)

**DevOps & MLOps:** Docker · GitHub Actions · AWS Lightsail · ECS · CloudWatch · Railway · Nginx · fail2ban · TFDV · TFX

---

## Contact

📧 johnmosesng@gmail.com &nbsp;·&nbsp; 🐙 [github.com/johnmoses](https://github.com/johnmoses) &nbsp;·&nbsp; 📹 [Loom Portfolio](https://www.loom.com/share/16bbb05b07b641509eb2a3a2d68f45a0) &nbsp;·&nbsp; 🌐 [shopstack360.com](https://shopstack360.com)

---

## Credentials

30 courses across 5 institutions — 22 at 100%, 7 above 95%.

| Area | Credential | Score |
|------|-----------|-------|
| GenAI & Agents | Building GenAI Applications and Agents Specialization (6 courses) | All 100% |
| AI for Medicine | AI for Medicine Specialization (3 courses) — DenseNet/U-Net, Cox PH, GradCAM | All 100% |
| MLOps | MLOps for Production Specialization (4 courses) | C1–C3 100%, C4 98% |
| Deep Learning | Deep Learning Specialization (5 courses) | C3 & C5 100%, others 97%+ |
| NLP | Natural Language Processing Specialization (4 courses) | C1–C3 100%, C4 97.5% |
| Mathematics | Mathematics for ML: Linear Algebra · Multivariate Calculus · PCA · Data Science Math Skills | All 98%+ |
| Finance/ML | Investment Management with Python & ML Specialization (4 courses) | 3 courses 100%, 1 at 97%+ |
| Generative AI | Generative AI with LLMs | — |
