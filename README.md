# Attune

*Infrastructure for agent cognition about humans.*

Attune is an orchestrator-level analyst focused on the human-AI interaction. It sits inside the orchestration loop, observes the bidirectional stream between user and model, and shapes how the model responds — per user, not per population.

Session history stores *what was said*. Attune understands *what's happening*.

---

## Why

Current AI systems already do behavioral pattern-matching. They do it against a generic "user" abstracted from any actual person — tone calibration, formality adjustment, mirrored enthusiasm, inferred expertise level. It mostly works, and when it doesn't, people can sense it. The adjustments are aimed at a statistical user who isn't anyone in particular.

Attune is about per-user behavioral understanding, not generic pattern libraries. Signal thresholds are derived from *this* user's corpus. Interventions are selected based on *this* user's interaction patterns. The taxonomy of behavioral signals is universal; every value on top of it is specific.

Generic behavioral adaptation is a ceiling, not a floor. Attune is the substrate for the next layer up.

---

## Architecture at a glance

```
┌──────────────────────────────────────────────────────────────┐
│                        Offline                               │
│                                                              │
│  Analytical Observer ──→ Labeled corpus ──→ Structural       │
│  (big, batch, discovery)                    signals +        │
│                                             embeddings       │
└───────────────────────────────────────────┬──────────────────┘
                                            │
                                            ▼
┌──────────────────────────────────────────────────────────────┐
│                        Runtime                               │
│                                                              │
│   User ──→ Runtime Observer ──→ Intervention ──→ Receiver    │
│              (small, fast,       channel            ↑        │
│              recognition only)     │                │        │
│                                    │                │        │
│                                    ├─ context ──────┤        │
│                                    │  shaping       │        │
│                                    │                │        │
│                                    └─ activation ───┘        │
│                                       steering               │
└──────────────────────────────────────────────────────────────┘
```

Three components:

* **Analytical Observer** — offline, large model, batch. Reads the labeled corpus, discovers behavioral patterns, produces the taxonomy the runtime side recognizes against. Runs occasionally, on corpus updates or taxonomy refinements.
* **Runtime Observer** — online, small model, per-turn. Recognizes already-labeled patterns in the live conversation and routes to an intervention. Recognition is a smaller job than discovery, so the runtime model can be smaller than the analytical one.
* **Receiver** — the model the user is actually talking to. Swappable per experiment.

The Analytical and Runtime Observers never run simultaneously. They share the labeled corpus as their interface and nothing else. They can be different models from different labs.

Two intervention channels:

* **Context shaping** — inject guidance into the Receiver's prompt. Works with any Receiver.
* **Activation steering** — add steering vectors to the Receiver's residual stream during the forward pass. Requires an open-weight Receiver with hook access.

Both channels live inside the same per-turn loop. They act at different moments — context shaping before generation, activation steering during it — but both are Attune's hands on the turn.

---

## Current status

**Phase 2 Layer A in progress — April 2026.**

* Phase 1 (foundation, import pipeline, MCP server, auth): complete
* Phase 2 Layer A (computable signals, embeddings, exploration surface): in progress
* Phase 2 Layer B (interpretive Analytical Observer): research
* Phase 3 (per-turn intervention, first-proof experiments): planned
* Phase 4 (real-time observation loop): planned
* Phase 5 (impact measurement dashboard): planned
* Phase 6 (production deploy): planned

The full design principles for the project live in [DESIGN_PRINCIPLES.md](DESIGN_PRINCIPLES.md).

---

## Selected design decisions

See [DESIGN_DECISIONS.md](DESIGN_DECISIONS.md) for the full excerpts.

* **Phase 2 decomposition into computable + interpretive layers** — separating tractable work from open research.
* **The Observer is two Observers** — analytical (offline, discovery) and runtime (online, recognition) share the labeled corpus as their interface. Decomposing them dissolved a memory-budget tension and made each independently scalable.
* **Truncation strategy: multi-strategy with dynamic selection** — embed long conversations three ways and let clustering quality reveal which strategy matches which interaction type.
* **Data shape: semantic vectors plus structural columns, combined visually** — resisting the temptation to merge everything into one vector.

---

## Open questions

The hard problems Attune is working through — research questions, not engineering tasks. See [NOTES.md](NOTES.md) for the working notes.

These include:

* the split between surface affect and substrate signal on the assistant side of the conversation, and how the Analytical Observer should weight them
* what taxonomy granularity the analytical side should produce so the runtime side can actually recognize it
* what a real-time visualization of behavioral signal activation would look like as a conversation unfolds

---

## More

* [**The Cognitive Stack**](https://thecognitivestack.substack.com) — newsletter on AI systems, cognition, and the structures underneath both. Often draws from Attune's design work but ranges more broadly.
* **Contact** — [LinkedIn](https://www.linkedin.com/in/adeline-latruwe/) · [email](mailto:adeline.latruwe@gmail.com)

Code is in a private repository. Happy to walk through it in conversation.
