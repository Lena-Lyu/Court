---
fontsize: 10pt
geometry: margin=0.9in
header-includes: |
  \usepackage{float}
  \usepackage{booktabs}
  \usepackage{fancyhdr}
  \usepackage{graphicx}
  \usepackage{hyperref}
  \hypersetup{colorlinks=true,linkcolor=blue,citecolor=blue,urlcolor=blue,pdftitle={Why Storage-Time Compression Fails: An Information-Theoretic Analysis of Deferred Projection in LLM Memory Systems},pdfauthor={Your Name},pdfsubject={LLM Memory Systems, Information Retrieval}}
  \pagestyle{fancy}
  \fancyhf{}
  \fancyhead[L]{\small \textit{A Rate-Distortion Analysis of Storage-Time Compression}}
  \fancyhead[R]{\small \thepage}
  \renewcommand{\headrulewidth}{0.4pt}
  \usepackage{setspace}
  \onehalfspacing
---

\thispagestyle{empty}
\vspace*{1cm}

\begin{center}
{\LARGE \textbf{Why Storage-Time Compression Fails}}\\[0.3cm]
{\large \textbf{An Information-Theoretic Analysis of Deferred Projection in LLM Memory Systems}}\\[1.2cm]
{\large Lena Lyu}\\[0.3cm]
{\normalsize Independent Researcher}\\[0.5cm]
{\normalsize May 2026}\\[1.5cm]
\end{center}

\begin{abstract}
\noindent
Recent work---E-mem, Yamanaka et al., and MemPalace---has independently converged on a shared architectural critique: compressing information at ingestion time, before queries are known, destroys context that query-time reconstruction could recover. This paper provides a systematic information-theoretic analysis of \textbf{why} storage-time compression is structurally suboptimal. Applying rate-distortion theory, time-bounded epiplexity, and the information bottleneck method to the memory architecture design problem, we derive a \textbf{Projection Loss Conjecture} that decomposes the deferred-projection advantage into three interacting factors: query entropy $H(P_Q)$, document structural diversity $\Delta_{\text{struct}}$, and storage budget tightness $f(B/H)$. We prove a Necessity Lemma establishing that any valid lower bound must involve all three factors. A toy model (Appendix) validates the three-factor structure and the zero-product property. The analysis yields three testable predictions and explicit boundary conditions specifying when storage-time compression IS optimal. This paper does not propose a new architecture; it provides the theoretical framework that a rapidly converging but fragmented research area currently lacks.
\end{abstract}

\newpage

\section{Introduction}

LLM-based knowledge management systems---RAG pipelines, note-taking tools, conversation summarizers---share a common premise: information should be processed into structured form at the point of storage. Documents are chunked, embedded, summarized, or extracted into knowledge graphs at ingestion, before any query is known.

Within a single twelve-month window (late 2025 to mid-2026), multiple independent research groups have published pointed critiques of this default.

E-mem \cite{emem} identifies ``destructive de-contextualization'' caused by compressing sequential dependencies into rigid structures.

Yamanaka et al. \cite{stone} explicitly contrast the dominant ``extract then store'' paradigm with ``store then on-demand extract''---an approach we abbreviate as STONE ({\bf S}tore {\bf T}hen {\bf O}n-demand {\bf E}xtract) for the remainder of this paper.

MemPalace \cite{mempalace} demonstrates through verbatim storage benchmarks that zero-processing write paths achieve 96.6\% R@5 on LongMemEval (raw, using off-the-shelf all-MiniLM-L6-v2 embeddings on verbatim text with zero LLM calls and zero API calls). Their hybrid held-out result reaches 98.4\%---which the authors themselves describe as the ``honestly generalizable'' number. The team deliberately withholds 100\%, documenting that the remaining 0.6\% required manual inspection of error cases and constitutes ``teaching to the test''---a textbook disclosure against benchmark overfitting. Dey and Viradecha \cite{dey2026} corroborate that the 96.6\% result is driven primarily by off-the-shelf embeddings applied to raw text, with the spatial-metaphor architecture contributing negligibly---providing external validation for the claim that zero-processing write paths themselves carry the signal.

These works establish an \textit{empirical} case: storage-time compression causes measurable information loss, and query-time reconstruction can recover capability that compression destroys. What they do not provide is a \textit{theoretical} account of \textbf{why} the problem is structural rather than contingent. Without such an account, storage-time compression can be dismissed as an implementation detail to be optimized---make better embeddings, train better chunkers---rather than recognized as a structural property of the problem setting.

\textbf{This paper supplies that account.} We apply three established information-theoretic frameworks---rate-distortion theory \cite{shannon1959}, time-bounded epiplexity \cite{epiplexity}, and the information bottleneck method \cite{tishby1999}---to the memory architecture design problem. Our contribution is not in proving novel theorems, but in applying these frameworks to a domain where they explain a phenomenon that has been observed empirically but not yet understood theoretically.

\textbf{Contributions.} (1) A Projection Loss Conjecture that decomposes the deferred-projection advantage into three interacting factors (§3). (2) A Necessity Lemma proving that all three factors must appear in any valid lower bound (§3.4). (3) Three testable predictions with explicit boundary conditions (§4). (4) A toy model validating the three-factor structure and zero-product property (§Appendix).

\medskip
\noindent\textbf{Relation to a companion system release.}
The four design principles derived in §4 were not formulated in the abstract. They were extracted from a working prototype that the author has run continuously to support personal knowledge management workflows. This paper provides the theoretical accounting for that prototype's design choices; a separate release will describe the system itself, its operating behavior, and qualitative observations from sustained use. The information-theoretic argument presented here stands independently of the prototype's specific implementation: the predictions of §4 can be tested by any DPM-conformant system, and the boundary conditions of Table~2 apply equally to all instances of the class. Cross-validation between the theoretical predictions and the prototype's empirical behavior is left to the companion release and to future controlled experiments on standard benchmarks.

\section{Related Work}

\textbf{Critical memory architectures.} E-mem \cite{emem} proposes episodic context reconstruction with 7.75\% F1 improvement and 70\% token reduction on LoCoMo. STONE \cite{stone} advocates ``store then on-demand extract.'' MemPalace \cite{mempalace} provides verbatim storage benchmarks \cite{dey2026}. These works share the diagnosis; ours provides the theory.

\textbf{Late interaction methods.} ColBERT \cite{colbert} establishes that per-token embeddings with max-similarity scoring outperform single-vector representations. Late Chunking \cite{latechunking} defers chunk boundaries to query time (+6.5 nDCG@10 on NFCorpus). LatentMAS \cite{latentmas} avoids premature projection to text in multi-agent systems (14.6\% accuracy gain, 70--83\% token reduction). These methods converge on ``defer projection'' from different angles; we extend the principle to memory architecture with information-theoretic justification.

\textbf{Foundational theory.} Power sampling \cite{karan2025} demonstrates that capabilities exist latently in model weights. Epiplexity \cite{epiplexity} formalizes the information accessible to a time-bounded learner. The information bottleneck \cite{tishby1999} characterizes optimal compression under relevance constraints. We apply all three to our analysis.

\textbf{Memory benchmarks.} LongMemEval \cite{longmemeval} (ICLR 2025, 500 questions over multi-session chat) and LoCoMo \cite{locomo} (ACL 2024) are the standard evaluation frameworks. Our experimental predictions target these benchmarks.

\section{Information-Theoretic Analysis}

\subsection{Rate-Distortion Theory Applied to Memory Systems}

Shannon's rate-distortion function \cite{shannon1959} gives the minimum bits required to represent source $X$ with expected distortion $\leq D$:

\begin{equation}
R(D) = \inf_{P_{\hat{X}|X}: \mathbb{E}[d(X,\hat{X})] \leq D} I(X; \hat{X}) \tag{1}
\end{equation}

The critical implicit assumption is that the distortion measure $d$ is known to the encoder at compression time. For a memory system serving unknown future queries, this assumption fails. For each query $q \in \mathcal{Q}$, there exists an induced distortion $d_q$ that penalizes reconstructions differently depending on what information $q$ requires. A summary capturing algorithmic details but omitting performance characteristics is low-distortion for a correctness query but high-distortion for a latency query.

\begin{equation}
R_q(D) = \inf_{P_{\hat{X}|X}: \mathbb{E}[d_q(X,\hat{X})] \leq D} I(X; \hat{X}) \tag{2}
\end{equation}

When $d_{q_1} \neq d_{q_2}$, the optimal compression for $q_1$ is generally not optimal for $q_2$.

\medskip
\noindent\textbf{Observation 1 (Query-Conditional Advantage).}
Let $P_Q$ be the query distribution. A storage-time system commits to a single $P_{\hat{X}|X}$ at ingestion, while a deferred system adapts per query:

\begin{equation}
\mathbb{E}_{q}\!\left[\inf_{P_{\hat{X}|X}^{(q)}: \mathbb{E}[d_q] \leq D} I(X; \hat{X}^{(q)})\right] \leq \inf_{P_{\hat{X}|X}: \mathbb{E}_q[\mathbb{E}[d_q]] \leq D} I(X; \hat{X}) \tag{3}
\end{equation}

with strict inequality when $d_{q_1} \neq d_{q_2}$ and documents contain information relevant to one query but not the other. \textit{Justification:} set inclusion---the per-query-adaptive feasible set contains the fixed scheme's feasible set.

Observation 1 is not a deep result---it is ``more flexibility at decision time cannot hurt'' restated in information-theoretic terms. Its value is making explicit \textit{what conditions} cause the inequality to be strict: diverse queries and multi-aspect documents. Under a storage budget $B$, the deferred system pays a storage premium ($H(X)$ vs. $B$) for query-time flexibility. Whether this trade-off favors deferred projection depends on query entropy $H(P_Q)$, document structural diversity, and budget tightness---the three factors of Conjecture 1.

\subsection{Time-Bounded Epiplexity}

Following Finzi et al. \cite{epiplexity}, we separate information in data $x$ for a time-bounded learner into structural epiplexity $S_T(x)$ (learnable within budget $T$) and time-bounded entropy $H_T(x)$ (noise-like to the bounded learner). Critical finding: deterministic transformations can shift information between these components:

\begin{equation}
S_T(\phi(x)) > S_T(x) \quad \text{is possible for deterministic } \phi \tag{4}
\end{equation}

Chain-of-thought prompting is canonical: explicit intermediate steps increase structural epiplexity without adding new information. Equation~(4) establishes that deterministic transformations \emph{can} increase $S_T$; it does not establish that all deterministic transformations preserve or increase it. We next identify the specific class of transformations---\textbf{content-reductive} ones (chunking, summarization, single-vector embedding)---that demonstrably reduce $S_T$ for at least some query-document pairs:

\begin{equation}
S_T(C(x)) < S_T(x) \tag{5}
\end{equation}

for mainstream RAG, KG extraction, and summarization. Storage-time compression permanently reduces the structural epiplexity available for query-conditional extraction.\footnote{The epiplexity framework does not preclude \textit{structure-revealing} storage-time augmentations (e.g., cross-reference detection) that add explicit structure without removing content. DPM's near-identity storage is the simplest guarantee of $S_T$ preservation, not the only one.}

\medskip
\noindent\textbf{Observation 2 (Structural Preservation).}
A system storing $x$ with near-identity transformation ($\hat{x} \approx x$) preserves $S_T(x)$ for all queries. Any non-invertible storage-time compression achieves $S_T(C(x)) < S_T(x)$ for some document-query pairs.

\subsection{Information Bottleneck with Unknown Targets}

The IB method \cite{tishby1999} minimizes $I(X;T) - \beta \cdot I(T;Y)$ for a known target $Y$. In memory systems, $Y$ is the answer to an unknown future query. Storage-time systems optimize for an assumed target $Y_{\text{assumed}}$; deferred systems optimize for the actual $Y_q$ at query time.

\medskip
\noindent\textbf{Observation 3 (Adaptive Bottleneck).}
For any fixed $\beta$, the deferred IB scheme achieves $\mathbb{E}_q[I(A; Y_q)] \geq I(\hat{X}; Y_{\text{assumed}})$, with strict inequality when the optimal $\beta_q$ varies across queries. \textit{Justification:} the deferred scheme can replicate the storage-time scheme by setting $\beta_q = \beta$.

\medskip
\noindent\textbf{On the relationship among Observations 1--3.}
The three observations share a common set-inclusion structure: in each framework, a deferred scheme can replicate the corresponding storage-time scheme as a special case, so the deferred-feasible set contains the storage-feasible set. They are not three independent lines of evidence for the same claim---they clarify how the same structural principle manifests under different optimization criteria: representation efficiency (RDT), computational tractability for a bounded learner (epiplexity), and task relevance (IB). The three frameworks are applied here not to multiply evidence, but to make explicit which architectural properties each criterion privileges. The three frameworks differ in \emph{what is being optimized}---rate under a distortion criterion (RDT), structural epiplexity for a bounded learner, or predictive information under an IB tradeoff---not in the dominance mechanism. Their joint application clarifies which architectural property each criterion privileges (representation size, computational tractability, task relevance, respectively), rather than constituting three independent lines of evidence for a single conclusion.

\subsection{The Projection Loss Conjecture}

Observations 1--3 establish \textit{weak dominance} but not magnitude. We now state a conjecture quantifying the deferred-projection advantage.

\medskip
\noindent\textbf{Conjecture 1 (Projection Loss).}
For a memory system with document distribution $P_X$, query distribution $P_Q$, storage budget $B$, and utility $U$, let $\mathcal{C}_{\text{storage}}$ be any storage-time compression within budget $B$, and $\mathcal{D}$ a deferred projection scheme storing documents near-losslessly. Then:

\begin{equation}
\mathbb{E}_{(x,q)}\left[U(\mathcal{D}) - U(\mathcal{C}_{\text{storage}})\right] \geq H(P_Q) \cdot \Delta_{\text{struct}} \cdot f\!\left(\frac{B}{H(X)}\right) \tag{6}
\end{equation}

where:
\begin{itemize}
\item $H(P_Q)$ is query distribution entropy (nats). $H=0$ when all queries are identical.
\item $\Delta_{\text{struct}} = \mathbb{E}_{x}\!\left[\mathbb{E}_{q_1, q_2 \sim P_Q}\!\left[1 - \text{sim}\!\left(C^*_{q_1}(x),\, C^*_{q_2}(x)\right)\right]\right]$ is the \textbf{structural diversity} of the collection---the expected pairwise dissimilarity, across query pairs, of the optimal extractions $C^*_q(x) = \arg\min_{C \in \mathcal{C}} d_q(x, C)$ from the same document. $\text{sim}(\cdot,\cdot)$ is any bounded pairwise similarity on the representation space $\mathcal{C}$ (e.g., cosine, normalized edit similarity, or LLM-as-judge consistency rating). $\Delta_{\text{struct}} = 0$ when every document requires the same extraction regardless of query; $\Delta_{\text{struct}}$ approaches its maximum when different queries demand near-disjoint extractions. The pairwise formulation avoids requiring $\mathcal{C}$ to admit a well-defined mean and accommodates non-vector representations (text, graphs) on equal footing.
\item $f(B/H(X))$ is a budget sensitivity function, monotonically decreasing, with $f(0)=1$, $f(\infty)=0$, $f' < 0$. The hypothesized form is $f(x)=1-e^{-x}$ (diminishing marginal returns); alternatives include $f(x)=1/(1+x)$ and linear threshold forms, distinguishable empirically.
\end{itemize}

\medskip
\noindent\textbf{On the units of Equation (6).} The left-hand side is in task-specific utility units; the right-hand side combines nats with a similarity-metric variance and a dimensionless budget term. Equation (6) should therefore be read as a \emph{structural} relation rather than a numerical inequality: it asserts that the projection-loss advantage is governed by the three factors named on the right and vanishes whenever any of them vanishes (Lemma~1). Operationalization for any specific deployment requires calibrating the unit reconciliation between $H(P_Q) \cdot \Delta_{\text{struct}}$ and $U$ as an empirical choice, not a theoretical constant. The testable content of Conjecture~1 is the directional dependence captured in the three predictions of §4, which hold under this conceptual reading regardless of the specific calibration.

Conjecture~1 is a \textbf{conjecture}, not a theorem. A complete proof requires specifying the structural representation space, the similarity metric, and the mapping from structural fidelity to utility---specifications that depend on the specific LLM and task domain.

\subsection{Necessity Lemma}

While Conjecture 1 remains unproven in its full quantitative form, we can establish the structural necessity of the three factors.

\medskip
\noindent\textbf{Lemma 1 (Necessity).}
The projection loss $L = \mathbb{E}_{(x,q)}[U(\mathcal{D}) - U(\mathcal{C}_{\text{storage}})]$ satisfies:
\begin{enumerate}
\item $L = 0$ when $H(P_Q) = 0$ (all queries identical---storage-time can tailor compression)
\item $L = 0$ when $\Delta_{\text{struct}} = 0$ (uniform documents---single scheme suffices)
\item $L = 0$ when $B \geq H(X)$ (lossless storage feasible---no information discarded)
\end{enumerate}

\textit{Proof sketch.} (i) A single query $q_0$ with $\Pr[q_0]=1$: optimize $P_{\hat{X}|X}$ for $d_{q_0}$. (ii) $\Delta=0 \implies$ pairwise similarity $\text{sim}(C^*_{q_1}, C^*_{q_2}) = 1$ for all query pairs, so a single extraction scheme serves all queries equally, yielding zero utility gap. (iii) $B \geq H(X)$ enables $\hat{X} \approx X$, preserving all information. $\square$

Consequently, any lower bound on $L$ must involve all three factors. The multiplicative form $H \cdot \Delta \cdot f$ is the minimal structure satisfying the zero-product property: $0 \cdot \Delta \cdot f = H \cdot 0 \cdot f = H \cdot \Delta \cdot 0 = 0$. Additive forms ($H+\Delta+f$) would remain positive when one factor is zero, contradicting Lemma 1. Alternatives such as $\min(H,\Delta,f)$ satisfy the zero-product property but lose the amplification intuition; empirical discrimination between functional forms is left to future work.

\begin{figure}[H]
\centering
\includegraphics[width=0.92\textwidth]{figures/fig1_concept.png}
\caption{Storage-time compression (left) locks in a lossy representation before the query is known. Deferred projection (right) preserves raw content and applies query-conditional extraction.}
\label{fig:concept}
\end{figure}

\begin{figure}[H]
\centering
\includegraphics[width=0.85\textwidth]{figures/fig5_factors.png}
\caption{The three factors of Projection Loss. The product form satisfies the zero-product property: if any factor vanishes, the total advantage vanishes (Lemma 1).}
\label{fig:factors}
\end{figure}
\end{figure}

\section{Architectural Implications}

The analysis yields four design principles for deferred-projection memory systems:

\begin{enumerate}
\item \textbf{Storage Preservation.} Store original text verbatim; no chunking for semantic boundaries, no embedding-only representation, no summarization (Observation 2).
\item \textbf{Query-Conditional Extraction.} Retrieve and project based on observed query, not at ingestion (Observation 1).
\item \textbf{Graded Relevance.} Return continuous confidence scores $c_i \in [0,1]$ rather than binary relevant/irrelevant judgments, enabling evidence weighting (Observation 3).
\item \textbf{Lazy Relationship Inference.} Infer document relationships at query time, conditioned on the specific query, rather than pre-committing to fixed schemas (Equation 4).
\end{enumerate}

These principles collectively define the \textbf{Deferred Projection Memory (DPM)} architectural class, of which E-mem, STONE, and MemPalace are instances at varying levels of conformance. Table~\ref{tab:arch} compares DPM against dominant paradigms.

\begin{table}[H]
\centering
\caption{Architectural comparison.}
\label{tab:arch}
\small
\begin{tabular}{@{}p{2.2cm}p{2.5cm}p{2.5cm}p{2.5cm}@{}}
\toprule
\textbf{Property} & \textbf{RAG} & \textbf{Knowledge Graphs} & \textbf{DPM} \\
\midrule
Storage & Chunked + embedded & Pre-extracted triples & Verbatim \\
Write cost & 50--200 ms/doc & 100--500 ms/doc & 10--50 ms/doc \\
Relevance & Binary (top-k) & Binary (edge exists) & Continuous $c_i \in [0,1]$ \\
Causality & Coincidental co-retrieval & Fixed at ingestion & Query-conditional \\
Retention & Lossy & Lossy & Lossless \\
\bottomrule
\end{tabular}
\end{table}

\begin{table}[H]
\centering
\caption{When deferred projection provides advantage vs. when alternatives are preferable.}
\label{tab:boundary}
\small
\begin{tabular}{@{}p{2.8cm}p{3.2cm}p{3.2cm}@{}}
\toprule
\textbf{Condition} & \textbf{Favors Deferred Projection} & \textbf{Favors Alternatives} \\
\midrule
Query diversity & High & Low (homogeneous, predictable) \\
Document heterogeneity & High (mixed formats) & Low (uniform type) \\
Storage budget & Constrained & Abundant (all fits in context) \\
Latency & $>$2s acceptable & Sub-500ms required \\
Cross-doc reasoning & Required (causal chains) & Not needed \\
\bottomrule
\end{tabular}
\end{table}

\section{Empirical Predictions}

Conjecture 1 yields three testable predictions. Each is a statistical hypothesis that does not depend on the conjecture's formal status.

\textbf{Prediction 1 (Query Distribution).} The relative advantage of deferred projection increases with query entropy $H(P_Q)$. Testing: compare $\Delta\text{AQ}$ between homogeneous (single query type) and heterogeneous (mixed types) query sets.

\textbf{Prediction 2 (Structural Diversity).} The advantage is larger for document collections with high $\Delta_{\text{struct}}$---mixed code, documentation, discussions---than for uniform collections.

\textbf{Prediction 3 (Budget Sensitivity).} The advantage increases as storage budget tightens. Under abundant budget (entire collection fits in context window), deferred projection and direct injection perform comparably ($f(\infty)=0$). Under tight constraints (artificially limited context), the advantage is maximized.

\textbf{Falsification criteria.} The framework is falsifiable: (a) if no query-type interaction is observed, the query-conditionality premise is disconfirmed; (b) if direct injection matches deferred projection on quality at comparable cost even under tight budget, retrieval-based projection provides no practical benefit; (c) if BM25 lexical matching achieves comparable causal chain completeness, deferred projection's semantic components add cost without value.

\textbf{Power analysis.} For the minimum viable experiment (150 documents, $n=25$ per query category), detecting $\delta = 0.5$ on a 5-point LLM-as-judge scale (assumed $\sigma \approx 0.8$) requires $n \approx 42$ per group at $\alpha=0.05$, $1-\beta=0.80$; $n=25$ yields power $\approx 0.55$. The experiment is powered only for large effects ($\delta \geq 0.75$). Medium but non-significant effects should be interpreted as inconclusive.

\section{Discussion}

\textbf{Limitations.} (1) The rate-distortion framing assumes well-defined $d_q$, but distortion in LLM systems is mediated by complex reasoning. (2) $\Delta_{\text{struct}}$ remains unoperationalized---plausible proxies include embedding-space variance, edit distances between extracted fact sets, and LLM-as-judge similarity. (3) Observations 1--3 establish weak dominance, not magnitude. (4) The product decomposition in Conjecture~1 is the simplest form consistent with Lemma~1 but not the only one; interaction terms, regime-dependent scaling, or alternative parameterizations may better capture empirical behavior.

\textbf{Relationship to Long-Context Evolution.} Expanding context windows are the most serious challenge to deferred projection's relevance. A simple cost analysis suggests that retrieval becomes preferable for collections beyond a small threshold---on the order of tens of documents at 2026 frontier-model pricing---above which direct injection's per-query token cost dominates retrieval's amortized cost. A more precise breakeven depends on assumed retrieval call cost, average document length, top-$k$ value, and per-token model pricing, and is sensitive to which model and provider are assumed. Even when context windows make retrieval optional, the attention efficiency argument remains: LLM attention is a finite resource, and filtering to query-relevant content concentrates that resource where it matters.

\textbf{Open problems.} (1) Efficient estimation of the parametric-external knowledge boundary. (2) Compute-aware budget allocation across retrieval, assembly, and reasoning. (3) Longitudinal validation: do deferred-projection users make better decisions over months?

\section*{Acknowledgments}
This manuscript was prepared with LLM assistance for drafting, literature cross-verification, and editorial refinement. All conceptual contributions---the framing of storage-time compression as an information-theoretic question, the formulation of Conjecture~1, the Necessity Lemma, the toy model construction, the four architectural principles, and the experimental predictions---are the author's. The analysis was motivated by a working prototype used daily for personal knowledge management. Source for the companion prototype will be released at: [URL forthcoming]. This research received no specific funding.

\begin{thebibliography}{13}

\bibitem{emem} K.~Wang et al., ``E-mem: Multi-agent based Episodic Context Reconstruction for LLM Agent Memory,'' arXiv:2601.21714, 2026.

\bibitem{stone} H.~Yamanaka et al., ``Revolutionizing Long-Term Memory in AI: New Horizons with High-Capacity and High-Speed Storage,'' arXiv:2602.16192, 2026.

\bibitem{mempalace} MemPalace, ``Local-first AI Memory with Verbatim Storage,'' github.com/mempalace/mempalace, 2025--2026.

\bibitem{dey2026} R.~Dey and P.~Viradecha, ``Spatial Metaphors for LLM Memory: A Critical Analysis of the MemPalace Architecture,'' arXiv:2604.21284, 2026.

\bibitem{colbert} O.~Khattab and M.~Zaharia, ``ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT,'' \textit{SIGIR}, 2020.

\bibitem{latechunking} M.~G\"{u}nther et al., ``Late Chunking: Contextual Chunk Embeddings Using Long-Context Embedding Models,'' arXiv:2409.04701, 2024.

\bibitem{latentmas} J.~Zou et al., ``LatentMAS: Latent Collaboration in Multi-Agent Systems,'' arXiv:2511.20639, 2025.

\bibitem{karan2025} A.~Karan and Y.~Du, ``Reasoning with Sampling: Your Base Model is Smarter Than You Think,'' arXiv:2510.14901, 2025.

\bibitem{epiplexity} M.~Finzi et al., ``From Entropy to Epiplexity: Rethinking Information for Computationally Bounded Intelligence,'' arXiv:2601.03220, 2026.

\bibitem{shannon1959} C.~E.~Shannon, ``Coding Theorems for a Discrete Source with a Fidelity Criterion,'' \textit{IRE National Convention Record}, 1959.

\bibitem{tishby1999} N.~Tishby, F.~C.~Pereira, and W.~Bialek, ``The Information Bottleneck Method,'' \textit{Allerton Conf.}, 1999.

\bibitem{longmemeval} X.~Wu et al., ``LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory,'' \textit{ICLR}, 2025.

\bibitem{locomo} A.~Maharana et al., ``LoCoMo: Long-Context Memory Benchmark,'' \textit{ACL}, 2024.

\end{thebibliography}

\newpage

\section*{Appendix: A Toy Model of Projection Loss}

We construct a fully-specified combinatorial model where the projection loss admits closed-form analysis, validating the three-factor structure and zero-product property.

\subsection*{Model Definition}
Let there be $n$ atomic facts, each $x_i \sim \text{Bernoulli}(1/2)$. A document $x \in \{0,1\}^n$ is the complete vector. There are $T$ query types; type $t$ requests fact subset $S_t \subset \{1,\ldots,n\}$ of size $k$. $P_Q$ is uniform: $H(P_Q) = \log T$. Storage budget: $m < n$ bits. Utility: fraction of requested facts correctly retrieved. Incidence matrix $C \in \{0,1\}^{n \times T}$ where $C_{i,t}=1$ if query $t$ needs fact $i$; row sums $r_i = \sum_t C_{i,t}$.

\subsection*{Optimal Strategies}
Storage-time stores the $m$ facts with largest $r_i$: $\mathbb{E}[U_{\text{storage}}] = \frac{1}{Tk}\sum_{i \in M} r_i$. Deferred stores all $n$ bits: $\mathbb{E}[U_{\text{deferred}}] = 1$. Loss: $L = 1 - \frac{1}{Tk}\sum_{i \in M} r_i$.

\subsection*{Three Canonical Cases}
\textbf{Case I} ($T=1$, $H=0$): all queries identical. Storage-time stores exactly those $k$ facts. $L=0$. \textbf{Case II} ($r_i$ constant, $\Delta=0$): every fact equally important. $L = 1 - m/n$, depending only on budget. \textbf{Case III} (disjoint queries, $\Delta=1$, $m \ll Tk$): $L \geq 1 - \lfloor m/k \rfloor / T \to 1$ for large $T$. Loss maximized when all three factors are at extremes.

\subsection*{Lower Bound}
The projection loss satisfies $L \geq \Delta \cdot \max(0, 1 - m/(n(1-\Delta)))$, satisfying the zero-product property. The query entropy $H = \log T$ enters through $\Delta$: larger $T$ enables larger $\Delta$, but the relationship is mediated by incidence matrix structure.

\subsection*{Relationship to Conjecture 1}
The toy model validates: (i) all three factors are necessary, (ii) the zero-product property holds, (iii) effect directions match ($H\!\uparrow$, $\Delta\!\uparrow$, $m/n\!\downarrow$ all increase $L$). It does \textbf{not} produce exact multiplicative factorization $L = H \cdot \Delta \cdot f$---$T$ and $\Delta$ interact through the incidence matrix in ways that resist clean separation. The multiplicative form in Conjecture 1 is the simplest functional form consistent with all three structural properties, not a claim of exact factorization.

