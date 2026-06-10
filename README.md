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

**Theoretical foundation:** [Why Storage-Time Compression Fails](docs/dpm-theory.pdf) — an information-theoretic analysis applying rate-distortion theory, time-bounded epiplexity, and the information bottleneck method to explain why storing raw evidence and deferring extraction to query time is structurally optimal, not just a preference.

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

## Status (June 11, 2026)

**What works:**
- L1 retrieval: 10-channel RRF fusion, zero-LLM, pluggable ChannelRegistry. P50 latency **0.1s**. Vector coverage: **96.3%** (nearly complete). **Zero anchor loss** in RRF fusion.
- L3 core mechanisms: `action_score`, `recall_counter` (falsification search), `cites_layer0` (anchor check) enforce Court gates. Court falsification: **100%** success in smart mode.
- **Adversarial safety: 80%** (target >70%). Unique to Court — no equivalent in Mem0/Letta/Zep.
- **A2A success: 100%** (20/20) — structured agent-to-agent communication with provenance chains.
- Agent full-stack operational: session + bootstrap + prefix cache. Successfully modifies files in agent tasks.
- Agent runtime (AgentLoop) and Weaver supervision (10 hard rules) operational.

**Benchmark results (June 11, 2026):**

| Benchmark | Score | Comparison | Notes |
|-----------|-------|------------|-------|
| **自建 1000 条 (全通道)** | R@10 **49.1%** | FTS5-only 38.5% (+28%) | Top 100: 68%. Long-tail 900: ~47%. Long-tail coverage is current bottleneck. |
| **LoCoMo (向量)** | MRR **0.521** | Zep R@10 85.2% | ⚠️ MRR ≠ R@10 — different metric, not directly comparable. |
| **BEIR SciFact (BGE-M3 直调)** | NDCG **0.647** | BGE-M3 0.743 | Embedding-only benchmark. RRF fusion + Court post-processing stronger on multi-modal. |
| **LongMemEval** | **45%** (35B) | Mem0 94.8% (GPT-4o) | ~5× model size diff. Multi-engine routing in progress for larger-model runs. |
| **对抗安全** | **80%** | — | Unique. Court falsification catches adversarial false claims. |
| **Court 证伪** | **100%** (smart) | — | Unique. Falsification search + anchor check enforcement. |
| **RRF 融合** | **0 锚点丢失** | — | Previously lost anchors during fusion — resolved. |
| **P50 Latency** | **0.1s** | — | All channels, smart mode. |
| **A2A Success** | **100%** | — | 20/20 agent-to-agent tasks with provenance chains. |

**What's in progress:**
- Court gate enforcement on Agent action path: gates active in retrieval pipeline, agent-loop integration ~70%.
- L0 data: `observations` table expanding (currently 422 entries). Code anchors (232K) as separate namespace.
- LongMemEval: local 35B ceiling investigation. Multi-engine routing (DS4 / 5090 / Cloud) for larger-model runs.
- A2A interface contracts: schema enforcement at startup/compile time, in design phase.

**Reference implementation:** private during active development. 814 tests passing, 7 runtime services. I lead with what I can demonstrate and explain line-by-line.

---

## Who This Is For

Engineers building agent infrastructure who know RAG isn't enough. Researchers thinking about memory epistemology. Teams burned by agent degradation in long tasks. Anyone who's tried vibe coding and watched their project turn into garbage.

→ [Read the full design](DESIGN.md) · [Competitive analysis](COMPETITIVE_LANDSCAPE.md) · [Honest status](STATUS.md)

---

## License

- Code: MIT
- Documentation: CC-BY-SA-4.0

*Court is a design philosophy with a reference implementation. Not a product. Not yet.*
