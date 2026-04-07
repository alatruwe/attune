# Attune

*Infrastructure for agent cognition about humans.*

Attune is an orchestrator-level analyst focused on the human-AI interaction.  
It sits inside the orchestration loop, observes the bidirectional stream between user and model, and tracks behavioral patterns that inform how context gets built for the next turn.

Session history stores *what was said*. Attune understands *what's happening*.

---

## Why

Most of the work on AI systems right now is about making models better at answering.  
Almost none of it is about making the system around the model better at noticing what's going on.  
You end up repeating yourself or explaining yourself, no matter how capable the model gets.


Attune is built on the idea that interaction quality is its own signal layer, distinct from content.  
An orchestrator-level observer dedicated to watching that layer can make AI conversations meaningfully better, by making the system around the model more attuned to the human in the loop.

---

## Architecture at a glance

```
┌──────────────────────────────────────────────────┐
│                  Orchestrator                    │
│                                                  │
│   User ───→ [Attune observes] ───→ Model         │
│   User ←─── [Attune observes] ←─── Model         │
│                    │                             │
│                    ▼                             │
│           Behavioral Pattern                     │
│              Tracker                             │
│                    │                             │
│                    ▼                             │
│           Context-Building                       │
│             Decisions                            │
└──────────────────────────────────────────────────┘
```

Two distinct model roles:

- **Observer model** — Attune's brain. Analyzes conversations and discovers behavioral patterns. Runs locally.
- **Receiver model** — The model the user is actually talking to. Where Attune's context recommendations get injected.

The two are always separate concerns, even when the same model serves both roles.

---

## Current status

**Phase 2 Layer A in progress** — March 2026.

- Phase 0 (foundation, MCP server, network access): complete
- Phase 1 (chat import pipeline): complete
- Phase 2 Layer A (embeddings, structural signals, exploration surface): in progress
- Phase 2 Layer B (interpretive Observer): research phase
- Phase 3 (context-building recommendations): planned
- Phase 4 (real-time observation loop): planned
- Phase 5 (impact measurement dashboard): planned
- Phase 6 (NAS deployment): planned

Self-hosted on a Synology NAS via Docker Compose, SQLite backend, Cloudflare tunnel for external access.

The full design principles for the project live in [DESIGN_PRINCIPLES.md](./DESIGN_PRINCIPLES.md).

---

## Selected design decisions

See [DESIGN_DECISIONS.md](./DESIGN_DECISIONS.md) for the full excerpts.

- **Phase 2 decomposition into computable + interpretive layers** — separating tractable work from open research
- **Truncation strategy: multi-strategy with dynamic selection** — embedding the same conversation three ways and letting clustering quality reveal which strategy matches which interaction type
- **Data shape: semantic vectors plus structural columns, combined visually** — resisting the temptation to merge everything into one vector

---

## Open questions

The hard problems Attune is working through — not the engineering tasks, the research questions. See [NOTES.md](./NOTES.md) for the working notes.

These include: 
- what a behavioral signal actually *is* when you're committed to emergent observation rather than predefined categories
- how to evaluate pattern emergence without labeled ground truth
- what a real-time visualization of behavioral signal activation would look like as a conversation unfolds.

---

## More

- [**The Cognitive Stack**](https://thecognitivestack.substack.com) — newsletter on AI systems, cognition, and the structures underneath both. Often draws from Attune's design work but ranges more broadly.
- **Contact** — [LinkedIn](https://www.linkedin.com/in/adeline-latruwe/) · [email](mailto:adeline.latruwe@gmail.com)

Code is in a private repository. Happy to walk through it in conversation.
