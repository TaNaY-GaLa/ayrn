# Technology Stack

This document explains the reasoning behind each technology choice, the alternatives considered, and the trade-offs accepted. It will be revisited as the platform scales — several choices here are deliberately optimized for early-stage velocity over long-term scale.

---

## Frontend: Next.js + React + Tailwind CSS

**Why:** Next.js gives us server-side rendering, file-based routing, and a mature deployment path (Vercel) with minimal setup overhead. React's ecosystem is the deepest available for building complex, stateful UIs — which agent monitoring and workspace dashboards will require. Tailwind lets us move fast on UI without maintaining a separate design system from day one.

**Alternatives considered:**
- **SvelteKit** — smaller bundle sizes and less boilerplate, but a smaller ecosystem and hiring pool. Rejected for team/ecosystem reasons, not technical ones.
- **Remix** — strong data-loading model, but Next.js has a more mature deployment story on Vercel, which we're already committed to.
- **Plain CSS / CSS-in-JS** — more control, but slower iteration speed early on. Tailwind's trade-off (utility class verbosity) is acceptable at this stage.

**Trade-offs:** Next.js brings some framework lock-in and opinionated conventions. We accept this for the productivity gain.

**Scalability:** Next.js scales well for content and dashboard-heavy applications; if AYRN's frontend needs evolve toward highly custom real-time visualizations (e.g. live multi-agent execution graphs), we may introduce a dedicated visualization library (D3, or a canvas-based renderer) without changing the framework.

**Future migration:** Low likelihood of migrating away from Next.js/React in the near term. Tailwind could be swapped for a component library (e.g. shadcn/ui, which is Tailwind-based) as design needs mature — not a full migration, an extension.

---

## Backend: FastAPI (Python)

**Why:** Python is the dominant language for AI/ML tooling, and FastAPI gives us async support, automatic OpenAPI documentation, and strong typing via Pydantic — important for a system where API contracts between the frontend, backend, and AI engine need to stay precise.

**Alternatives considered:**
- **Node.js (Express/NestJS)** — would unify the stack on JavaScript/TypeScript, but Python's AI/ML library ecosystem (and our own familiarity with it for agent orchestration) outweighs the benefit of a single language.
- **Django** — more batteries-included (admin panel, ORM), but heavier than needed for an API-first service, and less naturally async.
- **Go** — excellent performance and concurrency, but slower iteration speed for AI-adjacent code and a smaller ecosystem of agent/LLM tooling.

**Trade-offs:** Running Python for the backend and JavaScript for the frontend means two languages in the stack. We accept this because the backend's core job — orchestrating AI agents — is best served by Python's ecosystem.

**Scalability:** FastAPI's async support handles high I/O-bound concurrency well, which matches our workload (lots of waiting on model inference and tool calls). CPU-bound bottlenecks, if they emerge, would be addressed with worker processes rather than a framework change.

**Future migration:** Low likelihood of moving away from FastAPI. If a specific hot path needs more raw performance, we'd extract that path into a dedicated service rather than rewrite the whole backend.

---

## Authentication: Clerk (planned)

**Why:** Clerk provides drop-in auth (including SSO-ready infrastructure) without building session management, user storage, and auth UI from scratch — meaningful time savings pre-product-market-fit.

**Alternatives considered:**
- **Auth0** — comparable feature set, generally higher cost at scale.
- **Roll-your-own (NextAuth + custom backend)** — full control, but meaningfully more engineering time spent on a solved problem.

**Trade-offs:** Vendor dependency for a security-critical system component. Acceptable at this stage; revisited if enterprise customers require custom identity provider integration Clerk can't support.

**Scalability:** Clerk is built for production SaaS scale; not expected to be a bottleneck.

**Future migration:** If enterprise SSO/SAML requirements outgrow Clerk's offering, migration would be isolated to the auth layer given the API layer already abstracts authentication behind its own interface.

---

## Database: PostgreSQL

**Why:** PostgreSQL is a proven, relational system-of-record with strong consistency guarantees — appropriate for user accounts, workspaces, task records, and structured memory where correctness matters more than raw throughput.

**Alternatives considered:**
- **MongoDB** — more flexible schema, but we expect relational structure (users → workspaces → tasks → memory records) to dominate, which favors a relational model.
- **MySQL** — comparable to Postgres for our needs; Postgres was chosen for its stronger support for JSON columns and extensions (e.g. `pgvector`) that may reduce our dependency surface later.

**Trade-offs:** Relational schemas require more upfront design discipline than a document store. Acceptable given the structured nature of our core data.

**Scalability:** Postgres scales well vertically and via read replicas for the volumes expected in early phases. Sharding would be considered only if workspace/task volume grows well beyond what a single well-tuned instance can handle.

**Future migration:** `pgvector` is a possible path to consolidate vector search into Postgres itself, reducing our dependency on a separate vector database — worth revisiting once Qdrant usage patterns are better understood.

---

## Vector Database: Qdrant (planned)

**Why:** Qdrant is open-source, self-hostable, and performant for the similarity search needed for semantic memory and retrieval — important given our long-term goal of supporting local/on-prem deployment.

**Alternatives considered:**
- **Pinecone** — strong managed offering, but closed-source and cloud-only, which conflicts with our local deployment goal.
- **Weaviate** — comparable open-source option; Qdrant was chosen for its simpler operational model and Rust-based performance characteristics.
- **pgvector (Postgres extension)** — simplest operationally (no separate service), but less mature for large-scale, high-dimensional vector search. May be revisited for smaller deployments.

**Trade-offs:** An additional service to operate and scale. Accepted because semantic memory is core to the product, not an add-on.

**Scalability:** Qdrant supports horizontal scaling via sharding and replication; sufficient headroom for anticipated growth.

**Future migration:** Unlikely to migrate away; more likely to run Qdrant alongside `pgvector` for different use cases (heavy semantic search vs. lightweight in-database lookups).

---

## AI: OpenAI APIs (current), Ollama (future)

**Why:** OpenAI's APIs provide the strongest available combination of reasoning quality, tool-calling support, and reliability for the agent engine's initial development. Building against a hosted API lets us focus early effort on orchestration, memory, and planning rather than model infrastructure.

**Alternatives considered:**
- **Anthropic, Google Gemini APIs** — comparable hosted options; the architecture is being kept model-agnostic at the AI engine's interface layer so any of these (or multiple, per use case) can be integrated without a rearchitecture.
- **Self-hosted open-weight models (from day one)** — rejected for now due to the operational overhead of serving and fine-tuning models before the core product is validated.

**Trade-offs:** Dependency on a third-party model provider for cost and availability. Mitigated by designing the AI engine's model interface to be provider-agnostic from the start.

**Scalability:** Hosted APIs scale with usage-based pricing; cost management becomes a first-class concern as usage grows, tracked as part of the Phase 9 (Enterprise) monitoring work.

**Cost:** Worth stating plainly rather than deferring entirely to Phase 9: an agent that plans, acts, observes, and revises makes several model calls per task, not one — a single delegated task can plausibly cost 10-50x a single chat turn depending on plan length and revision count. This should inform early decisions in Phase 3 (e.g. using a cheaper model for planning/revision steps and a stronger model only for the steps that need it), not be left as a monitoring concern to address after the engine is built.

**Future migration:** Ollama integration is planned specifically for local/offline deployment and privacy-sensitive enterprise use cases — not a replacement for OpenAI, but an additional supported backend.

---

## Deployment: Docker, Vercel, AWS (future)

**Why:** Docker gives us environment consistency across local development, CI, and production from day one. Vercel is the natural fit for the Next.js frontend given first-party support. AWS is the planned target for backend/AI engine/database hosting once we need more control than a PaaS provides.

**Alternatives considered:**
- **Fully PaaS (Render, Railway) for backend too** — simpler initially, but less control over networking and scaling for AI workloads long-term. AWS is the more durable choice even though it adds operational complexity later.
- **Kubernetes from day one** — premature; adds operational overhead not justified until service count and scale require it.

**Trade-offs:** Splitting frontend (Vercel) and backend (AWS, future) hosting means managing two deployment surfaces. Accepted for the fit-for-purpose benefit of each.

**Scalability:** AWS provides the headroom needed for the AI engine and tool execution layer as usage grows; container orchestration (ECS or EKS) will be evaluated once service count justifies it.

**Future migration:** Kubernetes adoption is plausible once the number of independently-scaling services grows beyond what simpler container orchestration can manage cleanly.

---

## Version Control: Git + GitHub

**Why:** Industry standard, integrates with essentially every CI/CD and deployment tool we're likely to use.

**Alternatives considered:** GitLab, Bitbucket — comparable; GitHub chosen for ecosystem familiarity and integration breadth.

**Trade-offs:** None significant at this stage.

**Future migration:** Not anticipated.
