# Glossary

Terms used across this repository, defined precisely to avoid drift between documents. If a document uses one of these terms differently than defined here, the document is wrong and should be corrected to match this file.

**Agent**
A configured instance of the AI engine capable of executing a plan against a task: a model, a set of permitted tools, and a memory scope. Not the same as the AI engine itself — the engine is the infrastructure; an agent is a specific configuration of it.

**Task**
A single unit of work submitted by a user in natural language, decomposed by the Planning Engine into a plan. A task is scoped to one execution; see *Workflow* for the recurring/reusable case.

**Workflow**
A saved, reusable definition of a task (or sequence of tasks) that can be triggered repeatedly — manually or on a schedule — without being redefined from scratch each time. A workflow is what gets published to the Marketplace (Phase 8); a one-off task is not, unless saved as a workflow first.

**Plan**
The decomposed sequence (or graph) of steps the Planning Engine produces for a given task. Visible and editable by the user before execution, per requirements.md.

**Tool call**
A single invocation of an external capability (API call, file operation, etc.) made by the Tool Execution Layer on behalf of an agent during plan execution. Every tool call is logged; see architecture.md's Tool Execution Layer.

**Memory**
Durable, per-agent or per-user context that persists across sessions — distinct from a plan (which is scoped to one task) and from knowledge (which is organizational/shared, not agent-specific).

**Knowledge**
Structured and unstructured information ingested from external sources (documents, tickets, etc.) and made retrievable via the Knowledge Graph and vector search. Shared across agents within a workspace; not the same as an individual agent's memory.

**Workspace**
The tenancy boundary for users, agents, tasks, memory, and knowledge. Enterprise features (Phase 9) add role-based access control within a workspace.

**Approval checkpoint**
A point in plan execution where the system pauses and requires explicit human confirmation before proceeding, per requirement 8 (high-risk actions). The specific criteria for what triggers a checkpoint are not yet defined — see the open question in `architecture.md`.
