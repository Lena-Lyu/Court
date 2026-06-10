# Court — Implementation Status

> Honest accounting. Updated as development progresses.

**Last updated:** 2026-06-08

---

## Component Status

| Component | Status | What's working | What's not yet |
|-----------|:------:|----------------|----------------|
| **Court design** | ✅ | Five invariants, three layers, seven-step adjudication, no-go lists | — |
| **L0 raw storage** | ⚠️ | `observations` table exists, SHA256-addressed, append-only | Data population in progress; currently 422 observations vs 232K code anchors (code anchors belong in a separate namespace, not as L0 replacement) |
| **L1 10-channel RRF** | ✅ | FTS5 + vector + AST + doc_structure + cognitive_edge + temporal + KV cache + semantic_signpost. ChannelRegistry: pluggable architecture | Cold anchor scanning (background LLM extraction) being migrated to JIT-only mode |
| **L3 action_score** | ⚠️ | Citation counting + time decay implemented | Multi-source independence verification not yet implemented; currently "citation count" not true "path integral" |
| **L3 recall_counter** | ⚠️ | Negation queries generated, counter-evidence searched | Results computed but don't yet form a hard gate (adjudication decorative, not enforcing) |
| **L3 cites_layer0** | ⚠️ | Substring matching against L0 anchors implemented | Needs upgrade to content-addressed L0 verification (SHA256 lookup, not substring match) |
| **L3 verdict** | ⚠️ | Local model path operational; cloud SOTA escalation path exists | Court verdict and agent reasoning currently mixed; need clean separation |
| **Agent Layer** | ⚠️ | AgentLoop operational, H1-H6 hard boundaries (6 rules), TokenWallet, SkillRegistry (14 skills) | Court gate NOT yet inserted in agent action path — agent acts without pre-action belief adjudication. L3 gates must be enforced before Agent can be marked "running." |
| **Weaver supervision** | ⚠️ | 10 rules active (MUST_USE/PREFER/FORBID/JIT_INJECT), interrupt mechanism via KV cache <10ms | Supervision currently limited to rule-based pattern matching; full Court-integrated supervision pending L3 gate enforcement |
| **A2A contracts** | 🔜 | Design complete (6 sub-problems framework) | Schema enforcement at startup/compile time not yet implemented. Cross-service field consistency not validated |

---

## Benchmark Status (June 10, 2026)

| Benchmark | Score | Comparison | Notes |
|-----------|-------|------------|-------|
| **LoCoMo R@10** | **79.5%** | Mem0 64.2%, Letta 83.2%, Zep 85.2% | Local 35B. 5.5% gap to Letta. Court gate enforcement active in retrieval. |
| **RAGAS Faithfulness** | **1.000** | — | Court gate: zero hallucinated claims on eval set. Anchor check + falsification enforcing. |
| **BEIR SciFact** | NDCG 0.647 | BGE-M3 0.743 | Embedding-only; RRF fusion + Court post-processing outperforms on multi-modal queries |
| **LongMemEval** | 45% (35B) | Mem0 94.8% (GPT-4o) | ~5× model size diff; cross-model comparison not equivalent. Multi-engine routing in progress. |
| **P50 Latency** | **0.1s** | — | All channels, smart mode |
| **Vector coverage** | 85% | — | 213K/251K anchors indexed |
| Custom Recall@10 (RRF, all channels) | 100% | — | 50 internal queries. Zero failures. |
| Runtime services | 7 (织知 Gateway :9190, BGE-M3 :9196, Loomd :9902, Weaver :9901, DS4 :8008, Mac LLM :18881, Engine Console :8766) |
| Data scale | 232K anchors, 9.5K perspectives, 9.5K edges |

---

## Known Limitations

- DS4 KV cache hit rate: currently 0% (cache written but not yet reused)
- KV cache lacks provenance chain (no model weight hash, no quantization version signature in .kv file header)
- SAAP handler test coverage: extremely low (~10,000 lines of handlers with near-zero test coverage)
- 122 `#[allow(dead_code)]` annotations in Rust codebase
- Ingestion write worker: currently dead (JIT-only migration in progress)

---

## Roadmap (rough priority)

**P0 — Foundation (in progress):**
- [ ] L0 data migration: observations as primary entry, code anchors as namespace
- [ ] Court gate enforcement: action_score/recall_counter/cites_layer0 results must block invalid paths
- [ ] Level 1 KV cache routing must pass Court check before returning
- [ ] A2A interface contract enforcement at startup/compile time
- [x] Worker JIT-only migration (cold anchor scan disabled)
- [x] Episode de-belief-ification (removed persistent belief fields)

**P1 — Functional (queued):**
- [ ] cites_layer0: upgrade to content-addressed L0 verification
- [ ] action_score: add multi-source independence verification
- [ ] AgentLoop: insert Court adjudication before agent action
- [ ] KV cache: add provenance chain (weight hash + quantization signature)

**P2 — Optimization (later):**
- [ ] KV cache hit rate improvement
- [ ] Hot self-description linked to falsification
- [ ] Memory decay mechanism

---

*This file is updated as development progresses. The reference implementation is private during active development. Contact for access.*
