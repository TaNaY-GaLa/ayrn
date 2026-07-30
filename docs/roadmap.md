# Roadmap

This roadmap is organized into sequential phases. Phases are scoped by capability, not by calendar date — each phase begins once its dependencies are in place. Dates will be added once Phase 1 is underway and velocity is known.

---

## Phase 1 — Research

**Goal:** Validate architectural assumptions before writing production code.

- Survey existing agent frameworks (LangGraph, AutoGen, CrewAI, OpenAI Assistants, etc.) and document what to borrow vs. avoid
- Prototype memory retrieval approaches (vector search vs. hybrid vs. graph-based)
- Define evaluation criteria for "is this agent behaving reliably"
- Identify the minimum viable agent loop (plan → act → observe → revise)

**Deliverables:** Research notes, framework comparison, initial technical spikes (not part of the production codebase).

---

## Phase 2 — Architecture

**Goal:** Lock the system architecture before building on it.

- Finalize service boundaries (frontend, backend, AI engine, memory layer, tool layer)
- Define the API contract between frontend and backend
- Define the internal contract between the backend and the AI engine
- Choose data models for agents, tasks, memory records, and tool calls

**Deliverables:** `architecture.md` (this repo), API contract draft, data model draft.

---

## Phase 3 — Core Agent Engine

**Goal:** A single agent that can reason and act, without memory or multi-agent coordination yet.

- Implement the plan → act → observe → revise loop
- Minimal tool-calling interface, hardcoded to 1-2 tools, no permissioning or approval gates — just enough for the loop to act on the world (formalized in Phase 6)
- Task execution logging for auditability
- Minimal frontend to submit a task and watch it execute

**Deliverables:** Working single-agent execution engine, internal demo.

---

## Phase 4 — Memory

**Goal:** Agents retain and use context across sessions.

- Integrate vector database (Qdrant) for semantic memory
- Design memory write/read policies (what gets stored, what gets retrieved, when)
- Session-to-session continuity for a single agent
- Memory inspection UI (read-only)

**Deliverables:** Persistent memory layer, demonstrable cross-session recall.

---

## Phase 5 — Reasoning

**Goal:** Improve planning quality for multi-step tasks.

- Long-horizon task decomposition
- Plan revision when a step fails or new information arrives
- Human-readable, editable plan representation
- Basic self-evaluation of task progress

**Deliverables:** Planning engine capable of handling tasks with 5+ dependent steps.

---

## Phase 6 — Tool Calling

**Goal:** Formalize the minimal tool-calling built in Phase 3 into a permissioned, auditable framework. This phase is not "add tool calling" — it's "make tool calling safe to run unattended."

- Formal tool registration and permissioning model
- Structured tool call logging (input, output, latency, success/failure)
- Human approval checkpoints for high-risk tool calls
- First-party integrations (e.g. web search, file operations)

**Deliverables:** Tool execution framework with audit trail and approval gates.

---

## Phase 7 — Knowledge Engine

**Goal:** Ground agents in structured and unstructured organizational knowledge.

- Knowledge graph construction from ingested documents
- RAG pipeline combining vector search and graph traversal
- Source attribution for every retrieved fact
- Knowledge freshness/staleness tracking

**Deliverables:** Queryable knowledge layer usable by any agent.

---

## Phase 8 — Marketplace

**Goal:** Let users share and reuse agents and workflows.

- Agent packaging format (config + prompt + tool permissions)
- Public/private agent listing
- Versioning for shared agents
- Usage analytics for published agents

**Deliverables:** Marketplace MVP, first set of published agents.

---

## Phase 9 — Enterprise

**Goal:** Production readiness for team and organizational use.

- Team workspaces with role-based access control
- Multi-agent orchestration across a workspace
- Compliance and audit logging suitable for enterprise review
- Cloud and local (on-prem) deployment options
- SLA-grade monitoring and alerting for agent runs

**Deliverables:** Enterprise-ready deployment of AYRN.

---

## Notes

- Phases are sequential in dependency but may overlap in execution (e.g. Phase 6 tool infrastructure will begin before Phase 5 is fully complete).
- Phases 4 (Memory), 5 (Reasoning), and 6 (Tool Calling) are ordered by priority, not by hard dependency — none of the three strictly requires another to be finished first. Team capacity permitting, these can run partially in parallel rather than strictly sequentially.
- This roadmap will be revisited at the end of each phase and adjusted based on what was learned.
