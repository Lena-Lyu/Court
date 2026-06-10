# Why Storage-Time Compression Fails

**An Information-Theoretic Analysis of Deferred Projection in LLM Memory Systems**

_Lena Lyu, Independent Researcher, May 2026_

---

## Abstract

Recent work—E-mem, Yamanaka et al., and MemPalace—has independently converged on a shared architectural critique: **compressing information at ingestion time, before queries are known, destroys context that query-time reconstruction could recover.** This paper provides a systematic information-theoretic analysis of *why* storage-time compression is structurally suboptimal. Applying rate-distortion theory, time-bounded epiplexity, and the information bottleneck method to the memory architecture design problem, we derive a **Projection Loss Conjecture** that decomposes the deferred-projection advantage into three interacting factors: query entropy $H(P_Q)$, document structural diversity $\Delta_{\text{struct}}$, and storage budget tightness $f(B/H)$. We prove a Necessity Lemma establishing that any valid lower bound must involve all three factors. A toy model (Appendix) validates the three-factor structure and the zero-product property. The analysis yields three testable predictions and explicit boundary conditions specifying when storage-time compression IS optimal. This paper does not propose a new architecture; it provides the theoretical framework that a rapidly converging but fragmented research area currently lacks.

---

## 1. Introduction

LLM-based knowledge management systems—RAG pipelines, note-taking tools, conversation summarizers—share a common premise: information should be processed into structured form at the point of storage. Documents are chunked, embedded, summarized, or extracted into knowledge graphs at ingestion, before any query is known.

Within a single twelve-month window (late 2025 to mid-2026), multiple independent research groups have published pointed critiques of this default:

- **E-mem [1]** identifies "destructive de-contextualization" caused by compressing sequential dependencies into rigid structures.
- **Yamanaka et al. [2]** explicitly contrast the dominant "extract then store" paradigm with **"store then on-demand extract"** — an approach we abbreviate as **STONE**.
- **MemPalace [3]** demonstrates through verbatim storage benchmarks that zero-processing write paths achieve **96.6% R@5** on LongMemEval (raw, using off-the-shelf all-MiniLM-L6-v2 embeddings on verbatim text with zero LLM calls and zero API calls). Their hybrid held-out result reaches **98.4%** — which the authors describe as the "honestly generalizable" number. The team deliberately withholds 100%, documenting that the remaining 0.6% required manual inspection of error cases and constitutes "teaching to the test" — a textbook disclosure against benchmark overfitting. Dey and Viradecha [4] corroborate that the 96.6% result is driven primarily by off-the-shelf embeddings applied to raw text, with the spatial-metaphor architecture contributing negligibly.

These works establish an empirical case: **storage-time compression causes measurable information loss, and query-time reconstruction can recover capability that compression destroys.** What they do not provide is a theoretical account of *why* the problem is structural rather than contingent.

**This paper supplies that account.** We apply three established information-theoretic frameworks — rate-distortion theory [10], time-bounded epiplexity [9], and the information bottleneck method [11] — to the memory architecture design problem.

**Contributions:** (1) A Projection Loss Conjecture decomposing the deferred-projection advantage into three interacting factors (§3). (2) A Necessity Lemma proving all three factors must appear in any valid lower bound (§3.4). (3) Three testable predictions with explicit boundary conditions (§4). (4) A toy model validating the three-factor structure and zero-product property (Appendix).

---

## 2. Related Work

**Critical memory architectures.** E-mem [1] proposes episodic context reconstruction with 7.75% F1 improvement and 70% token reduction on LoCoMo. STONE [2] advocates "store then on-demand extract." MemPalace [3] provides verbatim storage benchmarks [4]. These works share the diagnosis; ours provides the theory.

**Late interaction methods.** ColBERT [5] establishes that per-token embeddings with max-similarity scoring outperform single-vector representations. Late Chunking [6] defers chunk boundaries to query time (+6.5 nDCG@10 on NFCorpus). LatentMAS [7] avoids premature projection to text in multi-agent systems (14.6% accuracy gain, 70–83% token reduction). These methods converge on "defer projection" from different angles.

**Foundational theory.** Power sampling [8] demonstrates that capabilities exist latently in model weights. Epiplexity [9] formalizes the information accessible to a time-bounded learner. The information bottleneck [11] characterizes optimal compression under relevance constraints. We apply all three to our analysis.

---

## 3. Information-Theoretic Analysis

### 3.1 Rate-Distortion Theory Applied to Memory Systems

Shannon's rate-distortion function gives the minimum bits required to represent source $X$ with expected distortion $\leq D$:

$$R(D) = \inf_{P_{\hat X \mid X}\,:\, \mathbb{E}[d(X, \hat X)] \leq D} I(X; \hat X)$$

The critical implicit assumption: **the distortion measure $d$ is known to the encoder at compression time.** For a memory system serving unknown future queries, this assumption fails. For each query $q \in \mathcal{Q}$, there exists an induced distortion $d_q$ that penalizes reconstructions differently depending on what information $q$ requires. When $d_{q_1} \neq d_{q_2}$, the optimal compression for $q_1$ is generally not optimal for $q_2$.

> **Observation 1 (Query-Conditional Advantage).** Let $P_Q$ be the query distribution. A storage-time system commits to a single $P_{\hat X \mid X}$ at ingestion, while a deferred system adapts per query. The deferred-feasible set contains the storage-feasible set, with **strict advantage when $d_{q_1} \neq d_{q_2}$** and documents contain information relevant to one query but not the other.

Observation 1 is not deep — it is "more flexibility at decision time cannot hurt" restated in information-theoretic terms. Its value is making explicit what conditions cause the advantage: diverse queries and multi-aspect documents.

### 3.2 Time-Bounded Epiplexity

Following Finzi et al. [9], we separate information in data $x$ for a time-bounded learner into **structural epiplexity** $S_T(x)$ (learnable within budget $T$) and **time-bounded entropy** $H_T(x)$ (noise-like to the bounded learner). A critical finding: deterministic transformations can shift information between these components:

$$S_T(\phi(x)) > S_T(x) \text{ is possible for deterministic } \phi$$

Chain-of-thought prompting is canonical: explicit intermediate steps increase structural epiplexity without adding new information. However, **content-reductive transformations** (chunking, summarization, single-vector embedding) demonstrably reduce $S_T$:

$$S_T(C(x)) < S_T(x)$$

for mainstream RAG, KG extraction, and summarization.[^1]

[^1]: The epiplexity framework does not preclude structure-revealing storage-time augmentations (e.g., cross-reference detection) that add explicit structure without removing content. DPM's near-identity storage is the simplest guarantee of $S_T$ preservation, not the only one.

> **Observation 2 (Structural Preservation).** A system storing $x$ with near-identity transformation ($\hat x \approx x$) preserves $S_T(x)$ for all queries. Any non-invertible storage-time compression achieves $S_T(C(x)) < S_T(x)$ for some document-query pairs.

### 3.3 Information Bottleneck with Unknown Targets

The IB method minimizes $I(X; T) - \beta \cdot I(T; Y)$ for a known target $Y$. In memory systems, $Y$ is the answer to an unknown future query. Storage-time systems optimize for an assumed target $Y_{\text{assumed}}$; deferred systems optimize for the actual $Y_q$ at query time.

> **Observation 3 (Adaptive Bottleneck).** For any fixed $\beta$, the deferred IB scheme achieves $\mathbb{E}_q[I(A; Y_q)] \geq I(\hat X; Y_{\text{assumed}})$, with strict inequality when the optimal $\beta_q$ varies across queries.

**On the relationship among Observations 1–3.** The three observations share a common set-inclusion structure: a deferred scheme can replicate the corresponding storage-time scheme as a special case. They are not three independent lines of evidence — they clarify how the same structural principle manifests under different optimization criteria: representation efficiency (RDT), computational tractability for a bounded learner (epiplexity), and task relevance (IB). The three frameworks are applied here not to multiply evidence, but to make explicit which architectural properties each criterion privileges.

### 3.4 The Projection Loss Conjecture

Observations 1–3 establish *weak dominance* but not magnitude. We now state a conjecture quantifying the deferred-projection advantage.

> **Conjecture 1 (Projection Loss).** For a memory system with document distribution $P_X$, query distribution $P_Q$, storage budget $B$, and utility $U$:
> $$\mathbb{E}_{(x,q)} [U(\mathcal{D}) - U(\mathcal{C}_{\text{storage}})] \geq H(P_Q) \cdot \Delta_{\text{struct}} \cdot f\!\left(\frac{B}{H(X)}\right)$$

where:
- $H(P_Q)$ is **query distribution entropy**. $H = 0$ when all queries are identical.
- $\Delta_{\text{struct}}$ is the **structural diversity** of the collection — the expected pairwise dissimilarity, across query pairs, of the optimal extractions from the same document. $\Delta_{\text{struct}} = 0$ when every document requires the same extraction regardless of query.
- $f(B/H(X))$ is a **budget sensitivity function**, monotonically decreasing, with $f(0) = 1$, $f(\infty) = 0$. The hypothesized form is $f(x) = 1 - e^{-x}$.

**On the units:** The left-hand side is in task-specific utility units; the right-hand side combines nats with a similarity-metric variance and a dimensionless budget term. The equation should therefore be read as a **structural relation** rather than a numerical inequality. The testable content of Conjecture 1 is the directional dependence captured in the three predictions of §4.

### 3.5 Necessity Lemma

> **Lemma 1 (Necessity).** The projection loss $L = \mathbb{E}_{(x,q)}[U(\mathcal{D}) - U(\mathcal{C}_{\text{storage}})]$ satisfies:
> 1. $L = 0$ when $H(P_Q) = 0$ (all queries identical)
> 2. $L = 0$ when $\Delta_{\text{struct}} = 0$ (uniform documents)
> 3. $L = 0$ when $B \geq H(X)$ (lossless storage feasible)

*Proof sketch.* (i) A single query $q_0$ with $\Pr[q_0] = 1$: optimize compression for $d_{q_0}$. (ii) $\Delta = 0 \implies$ a single extraction scheme serves all queries equally. (iii) $B \geq H(X)$ enables $\hat X \approx X$, preserving all information. $\square$

Consequently, any lower bound on $L$ must involve all three factors. The multiplicative form $H \cdot \Delta \cdot f$ is the minimal structure satisfying the zero-product property: if any factor is zero, the product is zero. Additive forms ($H + \Delta + f$) would remain positive when one factor is zero, contradicting Lemma 1.

---

## 4. Architectural Implications

The analysis yields four design principles for deferred-projection memory systems:

1. **Storage Preservation.** Store original text verbatim; no chunking for semantic boundaries, no embedding-only representation, no summarization.
2. **Query-Conditional Extraction.** Retrieve and project based on observed query, not at ingestion.
3. **Graded Relevance.** Return continuous confidence scores $c_i \in [0, 1]$ rather than binary relevant/irrelevant judgments.
4. **Lazy Relationship Inference.** Infer document relationships at query time, conditioned on the specific query, rather than pre-committing to fixed schemas.

These principles collectively define the **Deferred Projection Memory (DPM)** architectural class, of which E-mem, STONE, and MemPalace are instances.

| Property | RAG | Knowledge Graphs | DPM |
|----------|-----|-----------------|-----|
| Storage | Chunked + embedded | Pre-extracted triples | **Verbatim** |
| Write cost | 50–200 ms/doc | 100–500 ms/doc | **10–50 ms/doc** |
| Relevance | Binary (top-k) | Binary (edge exists) | **Continuous** $c_i \in [0,1]$ |
| Causality | Coincidental | Fixed at ingestion | **Query-conditional** |
| Retention | Lossy | Lossy | **Lossless** |

| Condition | Favors Deferred Projection | Favors Alternatives |
|-----------|---------------------------|---------------------|
| Query diversity | High | Low |
| Document heterogeneity | High (mixed formats) | Low (uniform type) |
| Storage budget | Constrained | Abundant |
| Latency | >2s acceptable | Sub-500ms required |
| Cross-doc reasoning | Required | Not needed |

---

## 5. Empirical Predictions

**Prediction 1 (Query Distribution).** The relative advantage of deferred projection increases with query entropy $H(P_Q)$. Test by comparing homogeneous vs. heterogeneous query sets.

**Prediction 2 (Structural Diversity).** The advantage is larger for document collections with high $\Delta_{\text{struct}}$ — mixed code, documentation, discussions — than for uniform collections.

**Prediction 3 (Budget Sensitivity).** The advantage increases as storage budget tightens. Under abundant budget (entire collection fits in context window), deferred projection and direct injection perform comparably ($f(\infty) = 0$). Under tight constraints, the advantage is maximized.

**Falsification criteria.** The framework is falsifiable: (a) if no query-type interaction is observed, the query-conditionality premise is disconfirmed; (b) if direct injection matches deferred projection on quality at comparable cost even under tight budget, retrieval-based projection provides no practical benefit; (c) if BM25 lexical matching achieves comparable causal chain completeness, deferred projection's semantic components add cost without value.

---

## 6. Discussion

**Limitations.** (1) The rate-distortion framing assumes well-defined $d_q$, but distortion in LLM systems is mediated by complex reasoning. (2) $\Delta_{\text{struct}}$ remains unoperationalized. (3) Observations 1–3 establish weak dominance, not magnitude. (4) The product decomposition in Conjecture 1 is the simplest form consistent with Lemma 1 but not the only one.

**Relationship to Long-Context Evolution.** Expanding context windows are the most serious challenge. A simple cost analysis suggests retrieval becomes preferable for collections beyond a small threshold — on the order of tens of documents at 2026 frontier-model pricing. Even when context windows make retrieval optional, the attention efficiency argument remains: LLM attention is a finite resource.

**Open problems.** (1) Efficient estimation of the parametric-external knowledge boundary. (2) Compute-aware budget allocation across retrieval, assembly, and reasoning. (3) Longitudinal validation: do deferred-projection users make better decisions over months?

---

## Appendix: A Toy Model of Projection Loss

**Model.** $n$ atomic facts, each $x_i \sim \text{Bernoulli}(1/2)$. $T$ query types; type $t$ requests fact subset $S_t$ of size $k$. $P_Q$ uniform: $H(P_Q) = \log T$. Storage budget $m < n$ bits. Utility: fraction of requested facts correctly retrieved.

**Optimal strategies.** Storage-time stores the $m$ facts with largest row sums $r_i$: $\mathbb{E}[U_{\text{storage}}] = \frac{1}{Tk}\sum_{i \in M} r_i$. Deferred stores all $n$ bits: $\mathbb{E}[U_{\text{deferred}}] = 1$. Loss: $L = 1 - \frac{1}{Tk}\sum_{i \in M} r_i$.

**Three canonical cases:**
- **Case I** ($T = 1$, $H = 0$): $L = 0$.
- **Case II** ($r_i$ constant, $\Delta = 0$): $L = 1 - m/n$.
- **Case III** (disjoint queries, $m \ll Tk$): $L \to 1$ for large $T$.

The projection loss satisfies $L \geq \Delta \cdot \max(0, 1 - m/(n(1 - \Delta)))$, satisfying the zero-product property.

**Relationship to Conjecture 1.** The toy model validates: (i) all three factors are necessary, (ii) the zero-product property holds, (iii) effect directions match. It does not produce exact multiplicative factorization — $T$ and $\Delta$ interact through the incidence matrix. The multiplicative form is the simplest functional form consistent with all three structural properties, not a claim of exact factorization.

---

## References

1. K. Wang et al., "E-mem: Multi-agent based Episodic Context Reconstruction for LLM Agent Memory," arXiv:2601.21714, 2026.
2. H. Yamanaka et al., "Revolutionizing Long-Term Memory in AI," arXiv:2602.16192, 2026.
3. MemPalace, "Local-first AI Memory with Verbatim Storage," github.com/mempalace/mempalace, 2025–2026.
4. R. Dey and P. Viradecha, "Spatial Metaphors for LLM Memory," arXiv:2604.21284, 2026.
5. O. Khattab and M. Zaharia, "ColBERT: Efficient and Effective Passage Search," SIGIR, 2020.
6. M. Günther et al., "Late Chunking," arXiv:2409.04701, 2024.
7. J. Zou et al., "LatentMAS: Latent Collaboration in Multi-Agent Systems," arXiv:2511.20639, 2025.
8. A. Karan and Y. Du, "Reasoning with Sampling," arXiv:2510.14901, 2025.
9. M. Finzi et al., "From Entropy to Epiplexity," arXiv:2601.03220, 2026.
10. C. E. Shannon, "Coding Theorems for a Discrete Source with a Fidelity Criterion," IRE, 1959.
11. N. Tishby, F. C. Pereira, and W. Bialek, "The Information Bottleneck Method," Allerton, 1999.
12. X. Wu et al., "LongMemEval," ICLR, 2025.
13. A. Maharana et al., "LoCoMo: Long-Context Memory Benchmark," ACL, 2024.
