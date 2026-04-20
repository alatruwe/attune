# Working notes on open problems

These are the live questions. Entries age out when a decision is made.

---

## [2026-04-11] Taxonomy operational constraint

The Analytical Observer produces the behavioral taxonomy offline. The Runtime Observer recognizes labels from that taxonomy per turn. The failure mode sitting between them is *taxonomy drift*: the analytical side can produce labels subtle enough that the runtime model can't reliably recognize them under latency constraint.

The obvious fix — make the runtime Observer bigger — is wrong. It breaks the latency budget that made the two-Observer decomposition work in the first place. The right fix is to constrain the analytical side's prompt: produce categories a small model could reliably recognize given the conversation plus a one-paragraph definition of each category. The taxonomy has to be operational, not purely descriptive.

This is a prompt-level concern for the Analytical Observer, but it's the kind of thing that has to be in the design before the analytical pipeline gets built. Descriptive taxonomies and operational taxonomies diverge fast, and the cost of discovering the divergence mid-Phase-3 is rebuilding the label set and re-running the analytical pass over the whole corpus.

Open: what's the right calibration loop? One approach is to hold out a small set of conversations, have the analytical side label them, then ask the runtime-tier model to recognize those labels given only the taxonomy definitions. Disagreement between the two is a signal that the label is under-specified or too fine-grained. Whether that calibration belongs in Phase 2B or Phase 3 is undecided.

---

## [2026-04-06] Surface affect vs substrate on the assistant side

Insight after reading Anthropic's [work on AI and emotions](https://www.anthropic.com/research/emotion-concepts-function?ss_ad_code=cct210706) ([full paper](https://transformer-circuits.pub/2026/emotions/index.html)): there's a split that matters for how Attune observes the model side of a conversation.

The assistant side has at least two layers that should be observed differently:

* **Surface layer** — politeness rituals, enthusiasm markers, hedges, apologies. Largely engineered conventions. Real signals about how the model was tuned, not about what the model is "doing." Treating them as direct evidence of internal state is a mistake the Observer shouldn't make.
* **Substrate layer** — what the model actually does: which questions it engages with, which it deflects, where its responses get more specific, where it backs off, where its reasoning compresses or expands. These are downstream of internal state in a way that surface affect isn't.

The behavioral signal taxonomy already separates these to some extent — the AI-specific signals category includes sycophancy markers, refusal calibration, instruction-following degradation, which are all substrate-layer. But the framing matters: when Attune watches the assistant side, it should weight substrate signals more heavily than surface affect, because the substrate is closer to what's actually happening and the surface is closer to what the model was trained to say.

This also affects the Analytical Observer's prompt. If the analytical side is going to interpret patterns on the assistant side, it needs to know not to take performed warmth at face value. Prompt-level concern, but it belongs in the design before Phase 2B starts.

---

## [2026-03-29] Dynamic graph visualization of behavioral signals

Imagine a force-directed graph rendered from the behavioral signal taxonomy, where nodes light up when their signal activates in the current turn. Edge weights shift as co-occurrence patterns update. Louvain communities re-partition — maybe "deference" and "hedging" were in separate clusters, but mid-conversation they start co-firing and the algorithm pulls them into the same community. You'd watch behavioral constellations form and dissolve in real time.

The practical question is granularity. A few options:

**Per-turn update** — after each exchange, recompute co-occurrence weights and re-cluster. Most responsive but could be jittery early in a conversation when data is sparse.

**Sliding window** — compute over the last N turns. The graph has a "memory horizon" and patterns emerge and fade. Smoother, more organic.

**Dual-layer** — a slow-moving "baseline" graph built from session history, with a fast-moving overlay showing current-turn activations. The baseline is the relationship substrate; the overlay is what's happening right now. Probably the most useful diagnostically — you could see when current behavior diverges from the established pattern.

The dual-layer approach maps cleanly onto the architecture distinction: labeled corpus stores what Attune has learned, runtime recognition is what's happening right now. Baseline graph is the understanding; overlay is the current turn. This is Phase 5 work, but the data it consumes is Phase 2 output — worth sketching now so the signal recording format doesn't foreclose the visualization later.

---

*Last updated: 2026-04-20*
