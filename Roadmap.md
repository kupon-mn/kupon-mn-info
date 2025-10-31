# 5-Week Roadmap (MVP → Checkout)

## Week 1 — Groundwork + Catalog Foundation

### Platform
- Monorepo: `apps/*` (services), `infra/*` (Helm). Dev/stage namespaces.
- CI/CD: lint+tests → Docker → Helm deploy to **dev**.
- Observability: OpenTelemetry, Prometheus, Grafana (HTTP RPS, p95).
- Secrets via External Secrets (cloud secret manager).

### Catalog Service (FastAPI, async)
- Endpoints: `POST /products`, `GET /products/:id`, `GET /products?query=...`
- Postgres (+ `pgvector`) for product embeddings; seed **300–1,000** items.
- **BFF**: `GET /bff/product-cards?ids=[]`.

### Acceptance
- Push to `main` auto-deploys to dev; Swagger works; p95 `GET /products/:id` < **80ms** (dev).

---

## Week 2 — AI Chat Orchestrator + Retrieval

### Chat Service (FastAPI)
- `POST /chat/message`, `GET /chat/session/:id`; tools: `catalog.search`, `catalog.get_cards`.
- LLM integration with strict timeouts, retries, and cost/latency logging.

### State & Caching
- **Redis-Session** for last N chat turns; **Redis-Cache** for product cards (TTL 5–30m).

### UI
- Minimal chat UI with inline product cards; BFF hydrates details.

### Acceptance
- Query “**50% off spa near me**” → **3–6 cards** + rationale; ≤ **2** LLM calls/turn; p95 E2E < **1.2s**.

---

## Week 3 — Ranking, Guardrails, Media Stub

### Quality
- Hybrid rank (BM25 + vector), optional LLM rerank **top-20** only.
- Rate-limit `/chat`, strip PII unless consented; error fallbacks (keyword search).

### Telemetry
- CTR on cards, add-to-view; distributed traces across **BFF → Chat → Catalog**.

### Media Service (stub)
- Signed URLs to object storage; placeholder image fallback for missing assets.

### Acceptance
- CTR visible in Grafana; ≥ **95%** image coverage (with placeholders) on seed data.

---

## Week 4 — Inventory + Reservation (Skeleton)

### Inventory Service
- **Postgres**: `stock`, `reserved`; atomic reserve API.
- **Redis-Locks** for fast decrement + reservation **TTL 5 min** with expiry worker.

### BFF
- `POST /reserve` returns `reservation_id`.

### Acceptance
- **1000** concurrent reserve attempts with **100** stock → **0 oversells**.

---

## Week 5 — Orders + Payments + Hardening

### Orders Service
- **Postgres**: `orders`, `order_items`; idempotent create.

### Payments
- Stripe (idempotency keys); **saga**: reserve → auth → confirm/cancel; async events.

### Scale/Sec
- HPA on RPS/CPU; alerts on p95 regressions; JWT auth, basic WAF rules.

### Acceptance
- E2E “browse → chat reco → reserve → pay → confirm” happy path; failure flows release stock.

---

# Microservices & Data Architecture

## Services (bounded contexts)
1. **API Gateway / BFF** (FastAPI)
   - Concern: client-facing composition, not business logic.
   - Reads from downstream services; caches product cards.
2. **Catalog Service** (FastAPI + Postgres + `pgvector`)
   - Owns products, categories, attributes, embeddings, search.
3. **Chat Orchestrator** (FastAPI + LLM + Redis-Session)
   - Tool-calling into Catalog; keeps short session history; logs prompts/results (optional).
4. **Media Service** (FastAPI + Object Storage)
   - Manages asset ingestion and signed URLs; later: async AI image generation worker.
5. **Inventory Service** (FastAPI + Postgres + Redis-Locks)
   - Owns stock levels & reservations; provides atomic reserve/release.
6. **Orders Service** (FastAPI + Postgres)
   - Owns orders and state machine; integrates with Payments.
7. **Payments Adapter** (FastAPI)
   - Thin facade for Stripe (or provider); emits events to Orders.

> Optional (later): **Users/Identity** (authn/z, profiles), **Merchants/Import**, **Analytics API**.

## Databases: per-service **OLTP** + one **General DB** (analytics)

### Per-service ownership (recommended for writes)
- Each service has its **own Postgres database** (separate DBs in one cluster *or* separate clusters). No cross-service SQL. Inter-service data flows happen via APIs or events.
- **Rationale:** clear ownership, safer schema evolution, fault isolation, least-privilege access.

### General “big” DB (read-only analytics & cross-domain queries)
- Use a **reporting/warehouse** (e.g., Postgres replica, ClickHouse, Redshift/BigQuery).
- Feed it via **CDC** (e.g., logical replication/Debezium) or nightly jobs.
- Consumers: BI dashboards, multi-service reporting, experimentation, ad-hoc joins.
- **Strict rule:** Services do **not** write here; product flows never depend on it synchronously.

## Redis tiers (at least one, add more if needed)
- **Redis-Session:** chat session turns, web session crumbs (evictable).
- **Redis-Cache:** product cards, search results, feature flags (TTL’d).
- **Redis-Locks / Counters:** inventory decrements, idempotency keys (short TTL).

> Start with a single managed Redis with logical separation; evolve to multiple clusters when traffic or isolation demands it.

## Eventing & Integration
- **Event bus** (SNS/SQS, Kafka, or Redis streams) for:
  - `order.created`, `payment.authorized`, `inventory.reserved`, `image.missing`.
- **Sagas & retries:** idempotent handlers, DLQs, circuit breakers to LLM and payments.

---

# Storage Matrix (Quick Reference)

| Service   | Primary Store                 | Secondary / Cache              | Notes                                   |
|----------:|-------------------------------|--------------------------------|-----------------------------------------|
| BFF       | —                             | Redis-Cache                    | Composition only.                       |
| Catalog   | Postgres (+`pgvector`)        | Redis-Cache                    | Owns product truth + embeddings.        |
| Chat      | (Optional) Postgres (logs)    | Redis-Session                  | Keep prompt/response audit if needed.   |
| Media     | Object storage (S3/GCS)       | —                              | Signed URL flow.                        |
| Inventory | Postgres                      | Redis-Locks / short counters   | Atomic reserve; TTL expiry worker.      |
| Orders    | Postgres                      | Redis (idempotency keys)       | State machine + events.                 |
| Payments  | —                             | —                              | Stateless adapter; provider vault holds tokens. |
| Analytics | Warehouse (General DB)        | —                              | Read-only; fed by CDC/ETL.              |

---

# Guardrails & Best Practices
- **No cross-service DB queries.** If you must “join,” do it in BFF using service APIs or by reading **Analytics DB** for non-critical/reporting.
- **Migration path:** Start with separate DBs **in one Postgres cluster** (one per service). When scaling, split into dedicated clusters with logical replication to the **General DB**.
- **Backups & DR:** PITR on per-service Postgres; warehouse recoverable from CDC + object storage snapshots.
- **Security:** Per-service DB credentials (least privilege); rotate via secret manager; audit LLM inputs (strip PII).

---

# Repo & Infra Skeleton

```
repo/
  apps/
    bff/
    catalog/
    chat/
    media/
    inventory/
    orders/
    payments/
  infra/
    helm/
      catalog/
      chat/
      ...
    k8s/
      namespaces.yaml
      ingress.yaml
  docs/
    architecture.md
    api/
```

> Each service: FastAPI, uvicorn, async SQLAlchemy, alembic migrations, pytest, OpenAPI via `/docs`.
