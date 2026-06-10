# Court — Provenance-First Memory for AI Agents

> **Not a warehouse. A courtroom.**
>
> Evidence is stored raw. Beliefs are formed on demand, discarded after use. Nothing is trusted without provenance. Everything can be reset to zero.

[中文版](README.zh.md)

---

## The Problem

AI agents forget. Not because models aren't smart enough — because **memory systems mistake storage for truth.**

Current approaches (Mem0, Supermemory, Anthropic's Dreaming) all do the same thing: digest conversations into "facts" at write time, store them as persistent beliefs, and trust them until they rot. This creates three failures that look different but share one root cause:

| Failure | What it looks like | Root cause |
|---------|-------------------|------------|
| **Hallucination** | Model invents things | Belief lost its source |
| **Poisoning** | Bad data corrupts the system | Belief has a forged source |
| **Decay** | Old info stays "true" forever | Belief outlived its source |

**The root cause is always: loss of provenance.** A belief detached from where it came from, treated as fact instead of claim.

---

## The Insight

LLMs are stateless functions — every inference is a fresh start. "Memory" isn't in the model weights; it's in the harness surrounding the model. The question isn't "how do we store more." It's **"how do we store so that every belief can be verified, falsified, and discarded when wrong."**

Think of it this way: a warehouse stores everything and trusts it forever. A courtroom stores raw evidence, forms judgments on demand, and **never archives a verdict as truth.** Court is the courtroom.

---

## The Framework: Five Invariants

Every component follows five principles:

1. **Provenance is load-bearing.** Poisoning = forged provenance. Hallucination = lost provenance. Decay = expired provenance.

2. **Raw evidence is the only authority.** Original observations, verbatim. Trusted as "records," never as "truth."

3. **Beliefs formed on demand, discarded after use.** No persistent beliefs. Every claim adjudicated at query time with confidence scores and source citations.

4. **The deeper you write, the more dangerous.** Reversibility gradient: raw text > KV cache > latent space > weights. Compress skills (what the model can do), externalize facts (what it claims is true).

5. **When provenance is unavailable, operational falsification substitutes.** Can't read what's inside? Try to prove it wrong. Same index, inverted question: "does evidence contradict this?"

---

## Three Layers

```
L0 — Raw Evidence (SQLite, append-only, SHA256-addressed)
    "User said: 'I prefer Rust'" — not "User prefers Rust"

L1 — Recomputable Index (10-channel RRF fusion, zero-LLM retrieval)
    Pointers only — "signposts, not truth." Can be deleted, rebuilt from L0.

L3 — On-Demand Adjudication (the courtroom)
    1. Multi-channel recall (milliseconds, zero LLM)
    2. Action score: corroboration × source trust × freshness × consistency
    3. Falsification search: same index, inverted question
    4. Anchor check: every claim MUST cite L0 — unverifiable = flagged
    5. Verdict: local model for routine, cloud SOTA only for high-stakes
    6. Disposal: belief logged as new observation, then discarded
```

---

## Agent Layer + Weaver Supervision

**Agent rule:** before acting, run your key beliefs through the courtroom.

**Weaver:** an orthogonal supervision layer with hard boundaries. Uses KV cache to enable real-time monitoring (<10ms response) — a second model watches the working agent and **interrupts when it catches sloppy work, misunderstanding, or silent deviation from the goal** — before garbage compounds.

---

## Where This Fits (June 2026)

The field is converging on provenance-first memory. Multiple teams have independently arrived at similar conclusions:

- **TierMem** (arXiv, Feb 2026) — provenance-aware tiered memory with source pointers. Reduces tokens 54% while keeping accuracy.
- **Eywa** (arXiv, May 2026) — "evidence before belief," immutable source → verified facts, zero-LLM retrieval.
- **Harness-1** (arXiv, Jun 2026) — external state harness lets a 20B model outperform GPT-5.4 on search.
- **Anthropic** — structured sourcing lifted SQL accuracy from 21% → 95%. Dreaming consolidates memory across sessions.
- **Mem0** — market leader (94.8% LongMemEval), ADD-only extraction.
- **Palantir AIP** — provenance-aware deletion, enterprise ontology-driven agent memory.

I arrived at the same direction independently — from a different starting point (physics PhD, not CS), through building and breaking things rather than reading papers. What I find least explored, and most interesting, is **active falsification** (searching for counter-evidence, not just checking source anchors) and **orthogonal real-time supervision** (a second model watching the working agent, with hard interrupt capability). I don't claim priority. I'm sharing a set of trade-offs I thought through, and a working core that demonstrates them.

---

## Status (June 2026)

**What works:**
- L1 retrieval: 10-channel RRF fusion, zero-LLM, pluggable ChannelRegistry. Baseline (pure FTS5, no vector): Recall@5 90%, P50 85ms. Vector channel (BGE-M3) being integrated — expecting 94-96% recall.
- L3 core mechanisms: `action_score`, `recall_counter` (falsification search), `cites_layer0` (anchor check) implemented and called within search pipeline.
- Agent runtime (AgentLoop) and Weaver supervision (10 hard rules) operational.

**What's in progress:**
- Court gate enforcement: core functions compute scores, but results don't yet block invalid paths. This is the top priority — currently ~30% of the way there.
- L0 data: `observations` table exists, needs more data (currently 422 entries vs 232K code anchors — code anchors belong in a separate namespace).
- Standard benchmarks: LoCoMo 44.6% (vs Zep 85.2% — early stage, major gap). BEIR 0.647 (vs BGE-M3 0.743). LongMemEval pending. Targeting completion end of June.
- A2A interface contracts: schema enforcement at startup/compile time, in design phase.

**Reference implementation:** private during active development. The system exists and runs — 814 tests passing, 7 runtime services. I lead with what I can demonstrate and explain line-by-line, not with line counts.

---

## Who This Is For

Engineers building agent infrastructure who know RAG isn't enough. Researchers thinking about memory epistemology. Teams burned by agent degradation in long tasks. Anyone who's tried vibe coding and watched their project turn into garbage.

→ [Read the full design](DESIGN.md) · [Competitive analysis](COMPETITIVE_LANDSCAPE.md) · [Honest status](STATUS.md)

---

## License

- Code: MIT
- Documentation: CC-BY-SA-4.0

*Court is a design philosophy with a reference implementation. Not a product. Not yet.*
