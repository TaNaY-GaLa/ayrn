# ADR 0001: Split language stack (Python backend, JavaScript frontend)

**Status:** Accepted
**Date:** 2026-07-30

## Context

AYRN needs a frontend for task submission and agent monitoring, and a backend that orchestrates AI agents, calls models, and executes tools. These could be built in a single language (e.g. Node.js end-to-end) or split by what each layer is best suited for.

## Decision

Frontend: Next.js/React/TypeScript. Backend and AI engine: Python/FastAPI. See `docs/tech-stack.md` for the full comparison against alternatives (Node.js unified stack, Django, Go).

## Consequences

- Two languages in the stack means two sets of tooling, two dependency ecosystems, and contributors who may need context-switching between them.
- In exchange, the backend gets direct access to Python's AI/ML ecosystem without a translation layer, which matters more for an agent-orchestration-heavy backend than stack unification does.
- Future contributors should not treat this as a settled-forever decision if the AI engine's bottleneck turns out to be something Python handles poorly (e.g. very high-concurrency tool execution) — see `docs/tech-stack.md`'s Future Migration notes.

## Alternatives Considered

Full detail in `docs/tech-stack.md`. Summary: Node.js unified stack (rejected — weaker AI/ML ecosystem), Go backend (rejected — smaller agent/LLM tooling ecosystem, slower iteration speed at this stage).
