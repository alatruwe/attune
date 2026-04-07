# Design Principles

## §1 — What Attune is

Attune is an **orchestrator-level analyst**. It sits inside the orchestration loop, observes conversation traffic in both directions (user → model, model → user), tracks behavioral patterns, and informs context-building decisions.

Session history stores *what was said*. Attune understands *what's happening*.

**Attune is not:**
- A memory system
- A tool or plugin
- A RAG pipeline
- A vector database wrapper
- The MCP server that exposes it

**Attune is:**
- An observer embedded in the runtime
- A behavioral pattern tracker
- Invisible infrastructure that makes orchestrators smarter about context

## §2 — Invisible by design

Attune is invisible *to the user*. No UI elements, no "Attune detected something" messages, no opt-in prompts.

The user experiences better conversations, not a product. Behaviors Attune enables — checking understanding, adjusting tone, catching divergence — should feel conversational, not mechanical.

**The test:** if something feels like a feature, it's wrong. If it feels like the model just *gets it*, it's right.

This principle applies to UX, not to internal intervention methods. Attune may intervene in significant ways under the hood (shaping context, reordering information, surfacing patterns to the receiver model). The principle is that those interventions don't surface as features the user has to think about. Internally, they should be inspectable, deliberate, and well-instrumented.

## §3 — Context-dependent, not static

User behavioral patterns are not stored as fixed profiles, labels, or scores. They are inferred per-interaction and can shift within a single conversation, across sessions, and across domains.

Attune tracks patterns as **dynamic behavioral embeddings**, not categories.

## §4 — Embedded, not bolted on

Attune lives inside the orchestration loop. It is not an external service the orchestrator calls when it needs context help.

The relationship substrate — the system's awareness of what's happening *between* user and model — is embedded in the runtime, not bolted on.

If Attune requires a separate API call to function during a conversation, the architecture is wrong.

_Note: Attune does expose an MCP server as an **access layer** for external clients like Claude.ai and Claude Code. This is a thin interface to Attune, not Attune itself._

## §5 — Emergent observation

Attune does not watch for a predefined list of signals. It observes the full bidirectional conversation stream and **learns what matters** from the patterns that emerge.

The design principle is the same one that makes transformer attention work: you don't hardcode what to attend to. The system observes enough traffic and discovers which patterns are signal — including patterns a human designer wouldn't have thought to specify.

Examples of the *kinds* of things that might emerge as meaningful (illustrative, not prescriptive):
- Moments where user and model mental models diverge
- Shifts in how a user frames problems within a single conversation
- Interaction dynamics that correlate with conversation quality or breakdown

**Attune does NOT:**
- Maintain a checklist of signals to watch for
- Classify interactions into predefined categories
- Reduce user behavior to a static profile or score

## §6 — User control

Attune does not decide what to forget. User control over their data is not optional.

All internal state transitions must be inspectable for debugging, even if the user never sees them during normal operation.
