# Competitive Landscape — AI Agent Memory (June 2026)

> Court's position in the rapidly converging field of agent memory infrastructure.

---

## The Convergence

The field is converging on a shared insight: **memory reliability is a provenance problem, not a storage problem.** Multiple independent teams have reached similar conclusions in 2025–2026. Court is distinguished not by any single mechanism (others have discovered individual pieces) but by the **completeness of the framework** — five invariants connecting memory → agent → multi-agent communication into a single coherent design.

---

## Head-to-Head Comparison

| System | Provenance-first? | Falsification? | Beliefs ephemeral? | A2A framework? | Hard supervision? |
|--------|:---:|:---:|:---:|:---:|:---:|
| **Court** | ✅ | ✅ | ✅ | ✅ (6 sub-problems) | ✅ (Weaver) |
| [Eywa](https://arxiv.org/abs/2605.30771) | ✅ | ❌ | ❌ | ❌ | ❌ |
| [TierMem](https://arxiv.org/abs/2602.17913) | ✅ | ❌ | ❌ | ❌ | ❌ |
| [Harness-1](https://arxiv.org/abs/2606.02373) | ⚠️ | ❌ | ❌ | ❌ | ❌ |
| [Mem0](https://github.com/mem0ai/mem0) | ⚠️ | ❌ | ❌ | ❌ | ❌ |
| Anthropic Dreaming | ❌ | ❌ | ❌ | ❌ | ❌ |
| Palantir AIP | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| [HOM Local](https://github.com/wallidsaydi-creator/hom-local) | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## Detailed Analysis

### Eywa (arXiv, May 2026)
**"Evidence before belief."** The closest independent work to Court's philosophy.

- Immutable source evidence stored before deriving canonical facts
- Zero LLM in retrieval path (deterministic multi-route read)
- 90.19% accuracy on LoCoMo C1-C4
- **Key difference from Court:** Validates against source but does NOT attempt falsification (searching for counter-evidence). Stores canonical facts rather than discarding beliefs after use. No A2A framework. No hard supervision layer.

### TierMem (ICLR 2025)
**Provenance-aware tiered memory.** Addresses the write-before-query barrier.

- Tier-1: fast summary index with provenance pointers → Tier-2: immutable raw-log store
- Lightweight router: answer vs. escalate
- Reduces tokens by 54.1%, latency by 60.7% vs. raw-only (accuracy 0.851 vs. 0.873)
- **Key difference from Court:** Provenance pointers but no falsification mechanism. Summaries are LLM-generated at write time (Court defers all digestion to query time).

### Harness-1 (arXiv, June 2026)
**State externalization for search agents.** UIUC + Berkeley + Chroma.

- 20B parameter model + external harness beats GPT-5.4 on search tasks
- Separates bookkeeping (harness) from semantic decisions (model)
- +11.4 points over next best open-source searcher
- **Key difference from Court:** Focused on search agent state management, not general memory epistemology. No falsification, no provenance chain for beliefs, no A2A framework.

### Mem0 (Open Source, 52K GitHub stars)
**Memory SDK for AI apps.** The most widely deployed memory solution.

- Vector DB semantic retrieval + graph DB relationship reasoning
- Adaptive decay, claims 26% higher accuracy, 91% faster, 90% less tokens vs. OpenAI built-in memory
- **Key difference from Court:** Digests conversations into "facts" at write time — exactly the pattern Court identifies as the root cause of provenance loss. Warehouse, not courtroom.

### Anthropic Dreaming (May 2026)
**Offline memory consolidation.** Anthropic's flagship memory feature.

- Reviews up to 100 past session transcripts
- Extracts patterns, cleans stale entries, restructures memory
- Non-destructive (original stays), takes tens of minutes
- **Key difference from Court:** Digests at write time (between sessions), creates persistent memory store. Does not preserve raw evidence provenance. Court specifically identifies Dreaming-style consolidation as the anti-pattern: "resolving contradictions at write time, freezing conclusions."

### Palantir AIP
**Enterprise ontology-driven agent platform.**

- Provenance-aware deletion (Data Lifetime service propagates deletions downstream through lineage engine)
- OAG (Ontology-Augmented Generation) instead of RAG — deterministic logic, not embedding similarity
- Audit.3: standardized audit schema for every agent action
- **Key difference from Court:** Pre-defined ontology (engineers hand-craft schemas). Court takes the opposite approach: no pre-defined ontology, evidence structure emerges from raw observations. Palantir's provenance answers "which system did this data come from?" — Court's provenance answers "which evidence supports this belief, and does counter-evidence exist?"

---

## Five Claims That Remain Unique to Court

As of June 2026, no other system simultaneously claims all five:

1. **Provenance as epistemological first principle.** Anthropic's own data validated this (structured sourcing took accuracy from 21% → 95%) but they didn't elevate it to design philosophy. Eywa and TierMem implement provenance mechanisms but don't center their entire framework on it.

2. **Courtroom, not warehouse.** All competing systems (Mem0, Supermemory, Dreaming, Eywa) store derived facts or canonical memories persistently. Only Court insists that beliefs must be formed on demand and discarded after use — verdicts logged as observations, never archived as truth.

3. **Dual hallucination defense.** Anchor check (find claims without L0 roots) + falsification search (find claims with fake roots). Eywa has anchor-like validation but no falsification. Anthropic discovered adversarial review independently (+6% accuracy) but abandoned it as too expensive (+72% latency) — Court's local-model falsification path addresses this cost.

4. **Reversibility gradient.** The explicit framework ranking storage trustworthiness (raw text > KV cache > latent space > weights) appears in no other system's design documents. MemoraX AI (March 2026, $10M+ funding) is actively pursuing the opposite: baking memory into model weights via RL.

5. **A2A as six measurable sub-problems.** Anthropic's Multi-Agent Orchestration solves parallel task dispatch but doesn't address "is what was transmitted trustworthy?" or "does the receiver understand my intent?" Court's A2A framework decomposes communication quality into individually optimizable engineering problems — a framework distinct from A2A/MCP transport protocols (which deliberately avoid these questions).

---

## What This Means

Court's core insights are being independently validated by top institutions (UIUC/Berkeley, Palantir, Anthropic). Individual mechanisms (provenance pointers, state externalization, adversarial review) are appearing across the field. But the **unified framework** — five invariants spanning memory → agent → multi-agent communication — remains unique.

The convergence validates the direction. The completeness of the framework is the differentiator.

---

*Sources: Eywa (arXiv:2605.30771), TierMem (arXiv:2602.17913), Harness-1 (arXiv:2606.02373), MemQ (arXiv:2605.08374), Anthropic Code with Claude (May 6, 2026), Palantir AIP developer documentation, Mem0 changelog (2026 Q1-Q2)*
