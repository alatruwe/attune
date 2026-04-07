# Working notes on open problems

---

## [2026-04-06] The two-layer problem on the model side

Insight after reading Anthropic's recent [work on AI and emotions](https://www.anthropic.com/research/emotion-concepts-function?ss_ad_code=cct210706): there's a split that matters for Attune. ([full paper](https://transformer-circuits.pub/2026/emotions/index.html))

The assistant side of the conversation has at least two layers that should be observed differently:

- **Surface layer** — politeness rituals, enthusiasm markers, hedges, apologies. These are largely engineered conventions. They're real signals about how the model was tuned, not about what the model is "doing." Treating them as direct evidence of internal state is a mistake the Observer shouldn't make.
- **Substrate layer** — what the model actually does: which questions it engages with, which it deflects, where its responses get more specific, where it backs off, where its reasoning compresses or expands. These are downstream of internal state in a way that surface affect isn't.

The behavioral signal taxonomy already separates these to some extent (Category 25 "AI-Specific Signals" includes sycophancy markers, refusal calibration, instruction-following degradation — substrate-layer things). But the framing matters: when Attune watches the assistant side of a conversation, it should weight substrate signals more heavily than surface affect, because the substrate is closer to what's actually happening and the surface is closer to what the model was trained to say.

This also affects Phase 2 Layer B. If the Observer LLM is going to interpret patterns on the assistant side, it needs to know not to take performed warmth at face value. That's a prompt-level concern, but it's the kind of thing that should be in the design before Layer B starts.

---

## [2026-03-29] Exploration surface ≠ observability dashboard

I was confusing Phase 2 and Phase 5 because both involve "looking at conversation data." They're different questions:

- **Phase 2 exploration surface:** What do my conversation patterns look like? (Understanding the data)
- **Phase 5 impact dashboard:** Is Attune making my conversations better? (Measuring the system)

The exploration surface is permanent infrastructure — I need it every time I re-import, add new model conversations, or change embedding parameters. It's not a one-time verification step. Moved it from Phase 5 to Phase 2 in the implementation plan.

The key design choice: semantic vectors define spatial position in the scatter plot, structural signals become visual properties (color, size) and interactive filters. They don't get merged into one vector. The combination happens in the visualization, not in the math. This avoids normalization and weighting decisions entirely.

---

## [2026-03-29] Truncation strategy as behavioral signal

When a conversation exceeds the embedding model's token window, you have to cut something. The question is *what* to cut.

Different conversation types distribute their semantic weight differently. Technical conversations where I'm aligning with Claude on an approach front-load the important content — the problem framing, the constraints, the initial negotiation. Cutting the tail loses execution detail but preserves the interaction that mattered. But conversations about self-discovery or open-ended exploration back-load the substance — the breakthrough or realization comes late. Cutting the tail loses the point.

This means the optimal truncation strategy is a property of the conversation type, not a universal setting. And that means the strategy *choice* is itself a signal about the dyad's interaction pattern. If my technical conversations consistently cluster better with head truncation and my reflective conversations cluster better with tail truncation, that's a structural insight about how I interact differently across domains.

The practical upshot: embed long conversations three ways (head, tail, bookend), compare clustering quality across strategies, and let the pattern emerge. By Phase 4, when a live conversation exceeds the token window, the strategy selection is informed by everything the batch analysis learned.

This connects to the larger Attune principle — don't hardcode what matters, let patterns emerge. Even the embedding pipeline's parameters become observation targets.

---

## [2026-03-26] Structural signals as Phase 2 starting point

Phase 1's SQLite data already contains enough to compute a meaningful subset of behavioral signals without an Observer LLM. Turn length, response latency, tool use frequency, message length trajectories — these are all extractable with SQL and basic Python from the messages table.

This means Phase 2 doesn't have to start with the hard research question ("what does the Observer produce?"). It can start with the computable foundation and build the interpretive layer on top.

The conversation-level embedding layer adds semantic grounding — without it, structural clusters are uninterpretable at scale. You can't remember what 400+ conversations were about just from turn lengths.

The brain-map visualization that prompted this line of thinking lives in Phase 5 (observability dashboard). But the data it needs is Phase 2 output. Building the computable layer now means Phase 5 has something real to render when the time comes.

---

## Hard Problem 1: What is a behavioral signal?

The design principles say Attune "observes the full bidirectional conversation stream and learns what matters from the patterns that emerge." That's the right philosophy. It's also the hardest thing to implement, because I need to operationalize "learns what matters" into actual code.

When the Observer model looks at a conversation, what is it actually producing? A summary? A list of observations? Embedding vectors? Structured annotations? I don't know yet, and that's fine architecturally, but it means the first real implementation decision in Phase 2 — "define the observation interface" — is actually a **research question disguised as a task**.

The output format of the Observer determines everything downstream: how signals get stored, how patterns emerge from them, how recommendations get built. Get this wrong and I rebuild from Phase 2 forward.

### The tension

**Too structured** (predefined signal categories) → violates the emergent observation principle. I'm just building a classifier with extra steps.

**Too open** (free-text observations from an LLM) → can't compute over the results. I just have a pile of paragraphs. No clustering, no pattern math, no embeddings to compare.

### Where the sweet spot probably is

The Observer produces **embeddings plus short natural-language annotations**. The pattern layer operates on the embeddings (computable, clusterable, comparable). The annotations are for human inspection (debuggable, interpretable). This gives both sides: emergence from the embedding space, legibility from the annotations.

### Where this stands

Partially resolved. Phase 2 was decomposed into Layer A (computable signals and embeddings — deterministic, no LLM, building now) and Layer B (the interpretive Observer — still the open question). Layer A gives the foundation a clear path forward without requiring an answer to "what does the Observer produce?" That question moves entirely to Layer B, where it belongs.

The structural side is no longer blocked on the research question. The research question itself is unchanged.

### Questions to research
- What embedding models produce useful representations of conversational dynamics (not just topic similarity)?
- Can a small model (Observer-tier) produce meaningful behavioral annotations, or does it need to be a stronger model?
- Is there prior work on embedding interaction patterns rather than content?
- What does the observation interface contract look like if embeddings are the primary output?

---

## Hard Problem 2: Pattern emergence without labeled data

Once I have signals, I need patterns to emerge from them. But I have no ground truth. Nobody has labeled a dataset of "here's a conversation where the user's mental model diverged from the model's" or "here's the moment cognitive load spiked." I'm doing unsupervised discovery on a data type that doesn't have established benchmarks.

### The evaluation problem

If Attune clusters my conversations and says "I notice you tend to ask shorter questions after long model responses" — is that a genuine behavioral signal or a statistical regularity that means nothing?

I'll have to use my own judgment as ground truth. That works for a personal tool, but "works for me" isn't a reproducible claim.

The observability dashboard (Phase 5) helps eventually — it can show whether Attune's patterns correlate with measurable conversation quality changes. But in Phase 2 I'm flying without instruments. The dashboard doesn't exist yet when I need to evaluate whether the patterns are real.

### How to mitigate
- Start with my own conversations as the test set. I know my patterns well enough to tell if Attune is finding real things.
- Look for patterns that are **surprising but recognizable** — things I didn't expect but can confirm once named. That's a stronger signal than patterns that are obvious or patterns that seem random.
- Don't chase precision early. If the Observer surfaces 10 candidate patterns and 3 are real, that's a starting point.
- Build the observability dashboard as early as possible once the core loop is working. Phase 5 says "after the bones are in place" — but maybe the structural metrics (message count, abandonment, duration) could be wired up alongside Phase 2 as a lightweight feedback signal.

---

## Hard Problem 3: Dynamic graph visualization of behavioral signals

Imagine the force-directed graph is rendered, and as a conversation unfolds, nodes light up when their signal activates. Edge weights shift as co-occurrence patterns update. Louvain communities re-partition — maybe "deference" and "hedging" were in separate clusters, but mid-conversation they start co-firing and the algorithm pulls them into the same community. You'd literally watch behavioral constellations form and dissolve in real time.

The practical question is granularity. A few options:

**Per-turn update** — after each exchange, recompute co-occurrence weights and re-cluster. This is the most responsive but could be jittery early in a conversation when you have sparse data.

**Sliding window** — compute over the last N turns. The graph has a "memory horizon" and you see patterns emerge and fade. This would look smoother and more organic.

**Dual-layer** — a slow-moving "baseline" graph built from session history, with a fast-moving overlay showing current-turn activations. The baseline is the relationship substrate; the overlay is what's happening right now. This is probably the most useful diagnostically — you could see when current behavior diverges from the established pattern.

The dual-layer approach maps cleanly onto the existing architecture distinction: session stores what was said, Attune understands what's happening. The baseline graph is the understanding; the overlay is what's being said.

---

_Last updated: 2026-04-06_
