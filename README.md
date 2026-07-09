# John Moses — AI Engineer (Full-Stack)

> I build AI into products end-to-end — from LLM pipelines and agent systems to the backends, mobile apps, and infrastructure that run them in production.

📹 **[Watch Portfolio Loom (90s)](https://www.loom.com/share/16bbb05b07b641509eb2a3a2d68f45a0)**
&nbsp;·&nbsp; 🐙 **[github.com/johnmoses](https://github.com/johnmoses)**

---

## Production Systems

### Life Reports — Church Attendance Platform
> 10,000+ users · FastAPI + Flutter + Next.js · AWS Lightsail · $20/month

A full-stack attendance reporting platform for hierarchical church organizations (Fellowship → Zone → District → Region → State). Built offline-first — works without internet, syncs when connected.

**AI highlights:**
- **Voice capture pipeline** — chunked MediaRecorder with real-time regex extraction per chunk for instant UX, then a final LLM pass at session end for accuracy. Handles Nigerian multilingual input (Hausa, Yoruba, Igbo, Efik, Idoma)
- **Attendance anomaly detection** — flags consecutive member absences (≥3) and center-level drops (≥30%) across all centers automatically
- **Outreach intelligence** — engagement scoring (attendance 35% + recency 30% + interactions 20% + stage 15%), churn prediction, and message variant generation by tone
- **Statistical trend analysis** — Redis-cached insights with cold-start handling for new centers

**DevOps highlights:**
- ADR-driven deployment decisions documented before any infrastructure was touched
- Docker Compose with per-service memory limits (384MB), PostgreSQL tuned for a $20 Lightsail instance
- GitHub Actions CI/CD, Nginx reverse proxy, fail2ban hardening, kernel-level SYN flood protection
- Sliding window rate limiter (5000 req/min general, 1000 req/min auth) with real IP extraction behind proxy

**Code excerpts:**
- [Voice capture pipeline](snippets/voice_pipeline.md) — hybrid inference, multilingual NLP, offline resilience
- [Anomaly detection agent](snippets/anomaly_detection_agent.md) — autonomous absence + drop detection
- [Offline-first sync](snippets/offline_sync.md) — Flutter + Next.js dual-layer queue
- [Production infrastructure](snippets/devops_infrastructure.md) — Docker, Lightsail, hardening, rate limiting
- [Outreach AI](snippets/outreach_ai.md) — engagement scoring, churn prediction, priority queue

**Pre-AI-era foundation (built without AI assistance):**
- [Custom transformer fine-tuning](snippets/transformer_finetuning.md) — BERT intent classifier (44 classes, 87.5% accuracy) + T5 text-to-SQL (ROUGE-L 0.961) + PEFT/LoRA on consumer hardware
- [Conversational agent](snippets/conversational_agent.md) — multi-turn dialog state machine with entity extraction and autonomous DB writes (agentic pattern before LangChain existed)

---

### ShopStack360 — Universal Commerce Operating System
> FastAPI + Flutter + Next.js · PostgreSQL + Redis · Docker · Railway

A full-stack B2B/B2C commerce platform connecting manufacturers, distributors, retailers, and shoppers across Nigeria and West Africa. Incubated in Abuja with 20+ live retailers, scaling to the Lagos industrial corridor.

**AI highlights:**
- **5 agentic AI modules** — Smart Reorder, Supplier Intelligence, Price Optimization, Inventory Advisor, Product Recommendations (collaborative filtering, auto-retrains every 24h)
- **Arbitrage engine** — 3-layer price scanner (internal + market scouts + external), landed cost calculator (product + shipping + customs + last-mile), opportunity scorer (0–100 composite)
- **Borderless logistics AI** — route optimization (Haversine), rider matching (composite scoring), demand forecasting (hourly heatmaps), dynamic surge pricing
- **Social intelligence signals** — tier-specific feed items generated from live transaction data for 3 user tiers (Connect / Trade / Supply)

**Full-stack highlights:**
- 3-provider payment routing (Paga / Paystack / Flutterwave) — intelligent routing saves 45% on processing costs
- Escrow with milestone releases for B2B, wallet system for instant settlements
- 7-level ambassador engine with anti-fraud detection and dormancy downgrade
- Recursive CTE for hierarchical center filtering across unlimited org depth

*Private repo — architecture overview.*

---

## Public AI Work

### Foundations (Theory → Code)
| Repo | What it covers |
|------|---------------|
| [llm-foundations](https://github.com/johnmoses/llm-foundations) | LLM internals, fine-tuning, prompt engineering, inference optimization |
| [rag-foundations](https://github.com/johnmoses/rag-foundations) | RAG pipelines, vector stores, chunking strategies, retrieval evaluation |
| [ai-agents-foundations](https://github.com/johnmoses/ai-agents-foundations) | Agent loops, tool use, multi-agent coordination, autonomous control |
| [mcp-foundations](https://github.com/johnmoses/mcp-foundations) | Model Context Protocol — building and consuming MCP servers |

### Applied AI Systems
| Repo | What it covers |
|------|---------------|
| [ultra-learning](https://github.com/johnmoses/ultra-learning) | AI-powered adaptive learning system — Flask API (LLM + RAG + MCP) · Next.js web · React Native mobile |
| [sure-health](https://github.com/johnmoses/sure-health) | Multi-party AI healthcare platform — Flask API (agents + LLM + RAG) · Next.js web · React Native mobile |
| [easy-finance](https://github.com/johnmoses/easy-finance) | AI-powered financial platform — FastAPI (wealth + community + security services) · Next.js web · React Native mobile |
| [zero-to-hero-python-ai](https://github.com/johnmoses/zero-to-hero-python-ai) | End-to-end ML/AI roadmap — CNNs, RNNs, Transformers, LLMs, Agentic AI |

### Specializations
| Repo | Credential |
|------|-----------|
| [coursera-nlp-specialization](https://github.com/johnmoses/coursera-nlp-specialization) | Coursera NLP Specialization |
| [coursera-mlops-specialization](https://github.com/johnmoses/coursera-mlops-specialization) | Coursera MLOps for Production |
| [coursera-ai-4-medicine-specialization](https://github.com/johnmoses/coursera-ai-4-medicine-specialization) | Coursera AI for Medicine |

### DevOps
| Repo | What it covers |
|------|---------------|
| [jenkins-docker-compose-flask](https://github.com/johnmoses/jenkins-docker-compose-flask) | Jenkins CI/CD with Docker Compose |
| [jenkins-docker-flask](https://github.com/johnmoses/jenkins-docker-flask) | Jenkins + Docker pipeline |

---

## Stack

**AI/ML:** LLMs · RAG · AI Agents · MCP · Transformer Fine-Tuning (BERT, T5, PEFT/LoRA) · Collaborative Filtering · Time-Series Forecasting · NLP · Computer Vision

**Backend:** FastAPI · Flask · PostgreSQL · Redis · SQLite · REST · WebSockets

**Frontend:** Next.js · TypeScript · Tailwind CSS · React

**Mobile:** Flutter · Dart · React Native · Offline-first (SQLite + sync queue)

**DevOps:** Docker · GitHub Actions · AWS Lightsail · Railway · Nginx · fail2ban

---

## Contact

📧 johnmosesng@gmail.com &nbsp;·&nbsp; 🐙 [github.com/johnmoses](https://github.com/johnmoses) &nbsp;·&nbsp; 📹 [Loom Portfolio](https://www.loom.com/share/16bbb05b07b641509eb2a3a2d68f45a0)
