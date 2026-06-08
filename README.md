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

## Competitive Landscape (June 2026)

| System | Provenance-first? | Falsification? | Beliefs ephemeral? | A2A framework? | Hard supervision? |
|--------|:---:|:---:|:---:|:---:|:---:|
| **Court** | ✅ | ✅ | ✅ | ✅ (6 sub-problems) | ✅ (Weaver) |
| Eywa (arXiv 2026.5) | ✅ | ❌ | ❌ | ❌ | ❌ |
| TierMem (ICLR 2025) | ✅ | ❌ | ❌ | ❌ | ❌ |
| Mem0 | ⚠️ | ❌ | ❌ | ❌ | ❌ |
| Anthropic Dreaming | ❌ | ❌ | ❌ | ❌ | ❌ |
| Harness-1 (arXiv 2026.6) | ⚠️ | ❌ | ❌ | ❌ | ❌ |
| Palantir AIP | ✅ | ❌ | ❌ | ❌ | ⚠️ |

**Five claims that remain unique to Court:**
1. Provenance as epistemological first principle
2. Court, not warehouse — beliefs ephemeral, verdicts not archived as truth
3. Dual hallucination defense — anchor check + falsification
4. Reversibility gradient — raw text > KV cache > latent space > weights
5. A2A as six measurable sub-problems — not better protocols, better *messages*

---

## Status (June 2026)

| Component | Status | Notes |
|-----------|--------|-------|
| Court design | ✅ Complete | Five invariants, three layers, seven-step adjudication |
| L0 raw storage | ⚠️ Partial | Data population in progress |
| L1 10-channel RRF | ✅ Running | Pluggable ChannelRegistry |
| L3 adjudication | ⚠️ Partial | Core functions implemented; gate enforcement under development |
| Agent + Weaver | ✅ Running | AgentLoop + 10 supervision rules active |
| A2A contracts | 🔜 Design | Schema enforcement at startup/compile time |
| E2E benchmark | 🔜 End of June | — |

**Reference implementation exists:** ~58K Rust + ~25K Python, 814 tests, 7 services. Private during active development.

---

## Who This Is For

Engineers building agent infrastructure who know RAG isn't enough. Researchers thinking about memory epistemology. Teams burned by agent degradation in long tasks. Anyone who's tried vibe coding and watched their project turn into garbage.

→ [Read the full design](DESIGN.md) · [Competitive analysis](COMPETITIVE_LANDSCAPE.md) · [Honest status](STATUS.md)

---

## License

- Code: MIT
- Documentation: CC-BY-SA-4.0

*Court is a design philosophy with a reference implementation. Not a product. Not yet.*
