# Selected Design Decisions

Excerpts from Attune's internal `DECISIONS.md` log.

---

## Phase 2 decomposition: computable layer + interpretive layer

**Context.** Phase 2, the "Behavioral Observation Engine," was originally treated as a single block of work centered on an Observer model. Design review identified that it actually contained two distinct concerns with very different difficulty levels and dependencies.

**Decision.** Decompose Phase 2 into two layers:

* **Layer A — Computable signals and embeddings.** Structural signals extracted from existing conversation data (turn length asymmetry, response latency, tool use patterns, message length trajectories, time-of-day patterns) plus conversation-level semantic embeddings. Deterministic computation. No LLM required. Can be built and validated independently.
* **Layer B — Interpretive Observer.** An LLM that analyzes conversations and discovers behavioral patterns beyond what structural computation can find. Layer B consumes Layer A's output as pre-computed features. It builds on Layer A; it doesn't replace it.

Layer A is implementable now. Layer B remains an open research question.

**Alternatives considered.** Treating Phase 2 as a single block and waiting until the Observer LLM question was resolved — this would have blocked progress on the computable foundation that Phases 3–5 depend on. Skipping Layer A and going straight to an LLM Observer — this would have meant doing expensive inference to discover patterns that are trivially computable from structural data.

**Consequences.** Layer A tasks become concrete and assignable. Layer B tasks stay marked as research. The embedding model decision becomes a Layer A decision. The Observer LLM selection becomes a Layer B decision and stays open without blocking anything.

---

## The Observer is two Observers

**Context.** Early architecture treated "the Observer" as one component — a model that analyzes conversations and drives per-turn intervention. That framing created a tension that wouldn't resolve: the analytical work (discovering behavioral patterns across a corpus) and the runtime work (recognizing those patterns in a live turn) have incompatible requirements. Discovery wants a large, expensive model with broad context; recognition needs a fast, cheap model that fits in the per-turn latency budget. Trying to satisfy both with one model meant choosing between an analytical capacity too shallow for discovery and a runtime cost too high for real-time use.

**Decision.** Decompose the Observer into two components with different lifecycles:

* **Analytical Observer** — offline, large, batch. Reads the corpus, identifies behavioral patterns, produces a labeled dataset. Could be a frontier model via API for the first pass. Runs occasionally: on corpus updates, on taxonomy refinements. Job: discovery and interpretation.
* **Runtime Observer** — online, small, fast. In the per-turn loop, recognizes which already-labeled patterns are present in the current conversation and routes to an intervention. Job: recognition against a known taxonomy, not discovery.

They never run simultaneously. They share the labeled corpus as their interface and nothing else. They can be different models from different labs.

**The insight.** Recognition is a much smaller job than discovery. Once the hard interpretive work has been done offline and compressed into a labeled taxonomy, a 7B–14B dense model is plausibly enough for the runtime side. The memory-budget tension that blocked progress on the single-Observer framing dissolves: the Analytical Observer has no per-turn latency constraint, the Runtime Observer has no discovery burden.

**Alternatives considered.** A single mid-sized Observer for both roles — would either underperform on discovery or miss per-turn latency targets, and would force a single model choice when the two jobs want different tradeoffs. A retrieval-augmented runtime Observer (small model + RAG over analytical outputs) — closer to the decomposed design but collapses "what Attune has learned" and "what Attune has done" into the same retrieval surface, which the intervention log needs to keep separate.

**Consequences.** The analytical pipeline and the runtime loop become independently specifiable. Model selection is per-role, not project-wide. The labeled corpus becomes a durable, model-agnostic asset — the transferable artifact — while steering vectors become per-Receiver, per-experiment. A new failure mode enters the picture: *taxonomy drift*, where the analytical side produces labels too subtle for the runtime side to reliably recognize. The fix is not a bigger runtime model; it's constraining the analytical prompt to produce categories a small model can recognize given the conversation and a short definition.

---

## Truncation strategy: multi-strategy with dynamic selection

**Context.** The embedding model has an 8192-token context window. Approximately 13% of conversations in the corpus exceed this limit — and they are likely the most substantive ones (long debugging sessions, deep design discussions, extended exploration). Truncation is unavoidable. The question is what to cut.

**Decision.** Conversations within the token window are embedded once. Conversations exceeding the window are embedded *three times*, once per strategy:

* **Head** — first 8192 tokens. Captures problem framing, initial alignment, opening dynamic.
* **Tail** — last 8192 tokens. Captures conclusions, breakthroughs, final state.
* **Bookend** — first 4096 + last 4096 tokens. Captures opening and closing, loses the middle.

Storage holds all three variants. Clustering runs three times, substituting each strategy's embeddings for the long conversations. The strategy that produces the most coherent clusters for a given conversation type is the strategy that preserves the most behavioral signal for that type.

**The insight.** No single truncation strategy is optimal for all conversations because conversations distribute their semantic weight differently. Convergent conversations (alignment at the top, execution follows) lose less from head truncation. Emergent conversations (exploration leading to a breakthrough) lose less from tail truncation. The way a conversation distributes its semantic weight tells you something about the interaction pattern of the dyad — which means the truncation strategy *itself* becomes a behavioral signal, not just an engineering parameter.

By the time real-time observation arrives in a later phase, the batch analysis has already established which strategy works for which conversation type. When a live conversation exceeds the window, the strategy choice is informed by everything prior analysis has learned. The truncation decision becomes a context-building input, not a default.

**Alternatives considered.** A single fixed strategy — simpler but loses signal on whichever conversation type the strategy doesn't fit. Summary-based truncation (LLM summarizes before embedding) — introduces an LLM dependency into what should be a computable pipeline, creates two different embedding populations, adds a failure mode.

---

## Data shape: semantic vectors + structural columns, combined visually

**Context.** Phase 2 Layer A produces two kinds of data per conversation: semantic embeddings from the embedding model, and structural measurements (turn count, duration, character asymmetry, tool use, response latency). These need to support exploration and insight extraction, both independently and together.

**Decision.** Semantic data lives as vectors in the vector store. Structural data lives as named columns in SQLite. The two are not merged into a single combined vector. They are combined in the exploration surface *visually*: semantic vectors define spatial position in the scatter plot, structural measurements become visual properties (color, size) and interactive filters.

**Alternatives considered.** Merge structural signals into a vector, concatenate with the semantic vector, cluster on the combined space — this requires normalization (structural metrics have wildly different scales), introduces weighting decisions (should semantic or structural features dominate?), and makes structural signals less interpretable because they lose their names inside the vector. Structural signals as a separate embedding space with its own projection — adds a parallel visualization that's harder to cross-reference with the semantic view.

**Consequences.** No normalization, no weighting, no merging. The exploration surface shows semantic clusters with structural overlays. Structural data stays readable and queryable as named columns. If structural similarity computation is needed later — "which conversations behave the same regardless of topic?" — the columns can be converted to a normalized vector at that point. Nothing is lost by storing them as columns now, and the false precision of premature merging is avoided.
