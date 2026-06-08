# Court → Agent → A2A: A Complete Design Philosophy

> Three layers sharing one worldview: Court (memory/epistemology kernel) → Agent (acting entity on the kernel) → A2A (boundary protocol between agents).

---

## Part 1: The Origin Story — Why This Architecture

The design emerged from a single question: **Why can't AI remember?**

Following it produced a chain of conclusions. Understanding this chain means understanding why every component exists.

1. LLM inference is fundamentally **amnesic** — each forward pass is a fresh start. What we call "memory" lives in the harness outside the model, not in the weights. **Continuity is the harness's job, not the model's.**

2. A completely amnesic cycle traps itself in loops. Breaking the loop requires leaking a small piece of information across iterations — not full memory, just **the signal that makes this iteration different from the last.**

3. But carrying full memory isn't pure upside. Memory is both asset and liability: it rots, locks frames, creates learned helplessness.

4. **Memory should be "held lightly": strong enough to act, weak enough to abandon.** The measure of a memory system is how cheap and safe it is to start from zero.

5. Breaking memory-locked cycles requires an **orthogonal signal** — not more information in the same dimension, but a new goal in a different dimension.

6. LLMs are **conditional distributions**, not databases. Your input coordinates where in the model's capability space you land. What matters isn't your intelligence — it's the pressure/friction you apply.

7. Tracing "what is memory" to its root reveals: **the real enemy isn't forgetting, it's loss of provenance.** Poisoning, hallucination, and decay are the same disease — a belief stripped of its source, treated as fact.

8. Hence Court: **don't build a warehouse that remembers everything about you — build a courtroom.** Store raw evidence with provenance. Adjudicate on demand. Never archive verdicts as truth.

9. Writing memory deeper (into weights/latent space) is more dangerous; writing externally (recomputable caches) is safer. This forms a **reversibility gradient.** A deeper law emerges: **legibility and emergent power antagonize each other at every level.** The universal governance: when provenance is unavailable, operational falsification substitutes for conceptual legibility.

10. Generalizing: multi-agent communication is the same problem moved to the boundary between agents. Court is the single-agent version of the A2A protocol.

---

## Part 2: Five Invariants

These hold regardless of whether you're building Court, an agent, or A2A. They are the foundation.

1. **Provenance is load-bearing; losing provenance is the root of all evil.** Poisoning = forged provenance. Hallucination = lost provenance. Decay = expired provenance. Any mechanism is only as safe as it carries "where did this come from, how much should I trust it."

2. **Raw evidence is the only authority, trusted only as "record" not "truth."** Weights are the chef (capability); raw evidence is the ingredients (facts). Hallucination is the chef substituting pantry stock for ingredients that should be fetched fresh.

3. **Beliefs are formed on demand, discarded after use — build a courtroom, not a warehouse.** Verdicts are reached at the moment of use, carrying confidence scores and provenance, never archived as truth. Explore in superposition; only convict claims that collapse onto anchored, provable evidence.

4. **Writing deeper is more dangerous.** Reversibility gradient: raw text > KV cache > latent space > weights. Trustworthy persistent beliefs must never be written into weights. **Compress "patterns" (intelligence), externalize "facts" (truth).**

5. **When provenance is unavailable, use operational falsification in place of conceptual legibility.** This is the only universal governance across memory, non-symbolic reasoning, and A2A.

---

## Part 3: Court — The Memory/Epistemology Kernel

### Principles

1. **Raw evidence is the only authority.** Trust only provenance-carrying original observations (trusted as records, not truth). Everything else is recomputable cache.

2. **Store as raw as possible; defer digestion to query time.** The digestion spectrum: grep → RAG → GraphRAG → LLM-wiki → weights. The more "interpretation" frozen at write time, the more susceptible to rot and poisoning. Never freeze conclusions at write time. Never "resolve contradictions."

3. **Beliefs formed on demand, discarded after use.** No persistent beliefs. Run "path integral adjudication" when needed: corroboration amplitude across independent evidence paths = confidence. Isolated evidence (no independent corroboration) destructively interferes, falls below threshold.

4. **When corroboration is insufficient, falsification is mandatory (orthogonal).** Same index, inverted question: don't just search for what supports a claim — actively search for what would refute it.

5. **Anchor check catches hallucination.** Every belief must be traceable to L0. No anchor = likely hallucination, marked low confidence, never silently adopted. Anchor check catches "rootless" hallucinations; falsification catches "fake-rooted" ones.

6. **Forgetting is a first-class operation, and equals "not promoted to belief" — not "deleted data."** Default to forgetting, exception to remembering. The exception is only: raw observation + provenance pointer.

7. **Observable, challengeable.** Unauditable memory = uncorrectable memory. Every belief carries confidence score + live provenance chain.

8. **"Starting from zero" must be cheap enough to be a daily operation.** This is the only thing that stops a position from becoming a prison.

### Three-Layer Architecture

**L0 — Raw Evidence Layer:** Append-only, content-addressed (id = SHA256(content)), never deleted (tombstone only), carries source/method/timestamp/trust. Stores **observations**, not **conclusions** ("User said: 'I switched to spaces'" — not "User prefers spaces").

**L1 — Recomputable Cache:** FTS5 (BM25) always-on + local vector index (lazy-built) + lightweight structures (NER, timestamps). Each entry carries "who derived it." **These are signposts, not truth; freely deletable, fully rebuildable.**

**L3 — On-Demand Adjudication (the Courtroom):** Local small model handles high-volume dirty work (query expansion, evidence reading, evidence packet assembly, falsification search). Cloud SOTA invoked only once for high-stakes cases, on condensed evidence packets. Produces belief with confidence + citations, used then discarded. Only "I reached conclusion X" is logged back as a new L0 observation.

### Court No-Go List

- ❌ Digesting conversations into "facts" at write time, storing as truth (Mem0/Supermemory's pitfall)
- ❌ "Resolving contradictions" at write time, freezing conclusions (Anthropic Dreaming's pitfall)
- ❌ Baking beliefs into weights / online fine-tuning (unfalsifiable, poisoning leaves no trace)
- ❌ Equal-weight treatment of all memories, no decay
- ❌ Treating negative conclusions as unfalsifiable facts (learned helplessness)
- ❌ Trusting unreadable, provenance-free caches (including transported KV cache — it must carry an evidence chain: source token hash + model/quantization version signature)
- ❌ Using semantic similarity for structurally-precise problems (find one row in 10,000 Excel rows: use SQL/exact match, not embedding)
- ❌ Letting ingestion become an unexamined trust black hole

---

## Part 4: Agent Layer — Above Court

Agent = a single entity that **acts**, using Court as its memory/epistemology. The key is not mixing up the division of labor.

### Principles

1. **Intelligence lives in the model; truth lives in Court.** Weights (chef) handle reasoning, language, skills — they can be unauditable, because they are capability, not belief. Court (pantry) handles concrete, current, challengeable facts. When an agent works, facts are retrieved from Court and adjudicated — not answered directly from weights.

2. **Before acting, run the beliefs you'll depend on through the courtroom.** Pre-action: adjudicate + falsify key beliefs. Only escalate to SOTA adjudication for high-stakes actions. This ties "action reliability" to "belief reliability."

3. **Agent persistent state lives in Court (external, auditable, resettable), never folded into weights.** This means an agent can always start from zero while its capabilities (weights) remain intact.

4. **Memory overload is a real risk.** Don't inject all history equally into context — that locks frames and creates learned helplessness. Inject only "the signal that changes this decision" — not the full log.

5. **KV cache is a safe internal lever** (recomputable, outside weights): usable for cognitive snapshots/rollback (cheap save points), fork exploration (share high-cognition prefix, each branch benefits). But **merging must go through text** (collapse into auditable output, then re-prefill). Never directly average two KV caches. Persistent KV must carry evidence chain.

### Agent Layer No-Go

- ❌ Agent writing its own conclusions back as "authoritative facts" (its conclusions are also just provenance-carrying observations, subject to re-adjudication)
- ❌ Agent self-corroborating without external ground (vacuum debate = vacuum spherical chicken)
- ❌ Continuously fine-tuning agent experience into weights (rot + poisoning leaves no trace + can't cleanly start from zero)

---

## Part 5: A2A Layer — Agent-to-Agent Boundaries

**Core insight: A2A communication is not an "information gap" problem.** It's a collection of six sub-problems — and team theory (Witsenhausen's counterexample) has proven that even with perfectly aligned goals and full information sharing, coordination can be intractable when decision information structures are decentralized. This is not rhetoric; it's a theorem-backed engineering/math problem.

### Six Sub-Problems (each measurable, optimizable)

1. **Conditioning/elicitation:** The receiver is a conditional distribution. Message quality = how well it conditions the receiver to its high-capability region. Measurable: fix receiver, measure output quality expectation and variance under different messages.

2. **Provenance preservation:** Claims transmitted across boundaries must carry their provenance. Receiver **re-adjudicates** rather than blindly inherits. Otherwise one agent's hallucination becomes the downstream premise, cascading and amplifying (compound hallucination). This is Court generalized to the boundary.

3. **Correlation/independence:** Same-source model "consensus" is echo, not corroboration. Measurable: inter-agent error correlation ρ. Second-opinion value ∝ (1−ρ). **Design for independence** (heterogeneous models, heterogeneous evidence, adversarial roles); discount correlated opinions in aggregation.

4. **Goal under-specification:** Natural language is underdetermined for "what success looks like." Receiver fills gaps with own priors; literal satisfaction violates intent (spec gaming). Solution: replace natural language intent with **verifiable acceptance criteria** (tests, schemas, formal constraints), or add "restate and confirm" handshake.

5. **Compound reliability:** n-step pipeline success rate ≈ ∏pᵢ (multiplicative, not average. 95% × 10 steps = 59%, 85% = 20% — regardless of model strength). Solution: insert **verification gates**, checkpoints, rollback at coupling points. This is an optimization problem: where to insert gates to maximize end-to-end reliability at minimum cost.

6. **Channel representation:** Legibility-bandwidth tradeoff. Dense latent states/KV (high bandwidth, powerful, but unauditable, cascading disaster) vs. structured sparse messages with provenance (low bandwidth, auditable, falsifiable). **Choose by stakes.**

### Measurable Definition of a Good A2A Message

A good A2A message simultaneously satisfies three **measurable** properties:
1. **Conditions the receiver into its capability region** (high expectation, low variance)
2. **Carries verifiable success criteria** (replaces "do you understand me?" with "does it pass this criterion?")
3. **Demands provenance-carrying output** (so the receiver's claims can be re-adjudicated, not inherited)

### A2A No-Go

- ❌ Treating "transport protocols" (A2A/MCP-style) as having solved the problem — they solve wiring (transport + discovery + task lifecycle) and **deliberately avoid** "is what was transmitted trustworthy? does the other party understand my intent?"
- ❌ Transmitting bare conclusions across boundaries (must transmit provenance-carrying claims)
- ❌ Treating same-source agent mutual corroboration as independent (pseudo-independence)
- ❌ Blindly stacking agent count/chain length (p^n decay; multi-agent often **worse** than single-agent; MAST research: 79% of failures from specification ambiguity and coordination breakdown)
- ❌ Using unauditable dense channels (latent states) for critical beliefs outside low-stakes high-frequency scenarios

---

## Summary: One-Page Cheat Sheet

**Five invariants:** provenance is load-bearing / raw evidence as only authority / form-and-discard, be a courtroom / deeper is more dangerous (compress patterns, externalize facts) / when provenance fails, falsification replaces legibility.

**Cross-layer no-gos:** don't freeze conclusions as facts; don't bake into weights; don't treat all memories equally without decay; don't self-corroborate without external ground; don't trust anything without provenance (data, caches, conclusions, KV); don't use semantic similarity for structurally-precise problems; don't let ingestion become a black hole; don't mistake "wiring protocols" for epistemology.

**One recurring law:** legibility and power antagonize each other at every layer; the only universal antidote — when provenance is unavailable, falsification against reality substitutes.
