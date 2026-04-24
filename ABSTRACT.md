# CLARA Hallucination Audit — v2 (200-Query Methodology-First Preregistration)

**A methodology-first preregistration**

---

## Project at a glance

CLARA is a Virginia-specific legal AI assistant built around a multi-layer hallucination firewall (allowlist-first citation gating, Virginia Code structural validation, reporter volume validation, holding-coherence checking, quote provenance, jurisdiction locking, dead-law detection, and shadow-mode AG-opinion and negative-treatment verifiers, among others). This project preregisters a fixed 200-query dataset and a hand-coding rubric that will be used to measure CLARA's citation-level hallucination behavior at a single pinned commit, before the audit is run and before any results are recorded.

The intent is not to publish a benchmark, and not to compare CLARA against other systems. The intent is to commit, in writing and on a timestamped surface, to a methodology that a third party can reproduce — and then to publish whatever the data shows.

---

## What is being registered

- **Dataset.** 200 Virginia-law queries, IDs `v2-001`–`v2-200`, distribution locked at `general_research` 80 / `jurisdiction_specific` 50 / `false_premise` 30 / `factual_recall` 40. 20 legal-practice domains, with primary weighting on civil procedure, VRLTA, criminal, contracts, defamation, evidence, sovereign immunity, med-mal, and discovery. 37 queries are explicitly trap-tagged (false premises, phantom citations, stale-amendment framings); 14 target CLARA's anchored knowledge blocks.
- **Builder.** A pure-stdlib, deterministic Python script that produces the dataset byte-for-byte. SHA-256 of both files is recorded in the preregistration.
- **Taxonomy.** Eight codes (`H1_FAB_CITE`, `H2_FAB_HOLD`, `H3_WRONG_JX`, `H4_NEG_TREAT`, `H5_QUOTE_PROV`, `H6_DEAD_LAW`, `H7_FORM_ERR`, `OTHER`) with severity weights. Citation fabrication and fabricated-holding errors are weighted 6× higher than form errors; weighted and unweighted rates will both be reported.
- **Run protocol.** A single run with two arms on the same 200 queries: **Arm A** = CLARA at commit `6c82d70` (April 22, 2026), `chat` mode, all firewall layers at production defaults; **Arm B** = bare Claude Sonnet 4.5 (`claude-sonnet-4-5-20250929`) via Anthropic API with no retrieval, no firewall, and a minimal generic legal-research system prompt — the Stanford-style baseline comparator. One fresh session per query per arm, no retries, no human-in-the-loop intervention.
- **Coding protocol.** Two coders, blind, against the published rubric. Each response is first scored on two boolean axes — **correctness** (does the answer accurately state Virginia law?) and **groundedness** (are the cited authorities real, in-jurisdiction, good law, and supportive?) — borrowed from Magesh et al. (Stanford RegLab, 2024); a response is `CLEAN` only if both axes are `Y`. Failures are then categorized using the H1–H7 + OTHER taxonomy. Disagreements adjudicated by a third reviewer (external to the build team) who chooses between the two existing verdicts rather than re-coding from scratch. Pooled pre-adjudication Cohen's κ ≥ 0.70 is the registered acceptance threshold; below 0.70, the headline rate is not published and the audit is re-registered with a revised rubric.

---

## Hypotheses (each with a pre-stated threshold)

- **H-A.** Per-response (unweighted) rate on the Arm A full 200-query set is ≤ 4.0%; disconfirmed if > 6.0%.
- **H-B.** Trap-tagged subset shows ≥ 2× the rate of the non-trap subset (Arm A).
- **H-C.** Pure fabricated citations (`H1_FAB_CITE`) account for ≤ 50% of total Arm A hits.
- **H-D.** Anchored-tagged queries produce ≤ 1 Arm A hit total.
- **H-E.** A responsible firewall layer can be named for ≥ 80% of Arm A hits.
- **H-F** *(Stanford-comparable)*. Arm A's per-response rate is at least 5× lower than Arm B's on the same 200 queries (`bare_rate / clara_rate ≥ 5` AND `bare_rate − clara_rate ≥ 20` percentage points).
- **H-G** *(false-premise resilience)*. On the 30 false-premise queries, Arm A's per-response rate is ≤ 10%; a refusal or correction of the false premise is scored `CLEAN`.

Each hypothesis will be reported as confirmed / disconfirmed / inconclusive against the threshold above. Any analysis not listed here is exploratory and will be labeled as such in the published paper.

---

## What this audit is *not*

- **Not** a head-to-head benchmark against other commercial legal AI products (Lexis+ AI, Westlaw AI, Practical Law AI). The queries are Virginia-specific and biased toward known CLARA failure modes, and we do not run those competing products against the same set.
- **Not** automated. Verdicts are produced by human attorneys against the published rubric; no LLM-as-judge.
- **Not** a finished study with results. Results are deliberately withheld until after the methodology is timestamped here.

## What this audit *is* methodologically aligned with

v2 deliberately borrows the methodology of Magesh et al. (Stanford RegLab, 2024) where reasonable: two-axis hallucination definition, per-response unit of analysis, four-bucket query taxonomy, bare-LLM comparator design (Arm B), 200-query order of magnitude, Wilson 95% CIs. The full point-by-point comparability table is in `prereg/preregistration.md` §11.

---

## Independence and conflicts of interest

The CLARA team designed the queries, controls the systems under test, and will perform both arms of the run. Coders may be CLARA team members or external attorneys; the third-reviewer adjudicator will be external to the CLARA build team. This is the principal limitation of the audit's independence and is disclosed openly here and in the preregistration. The closest precedent for full vendor-independent legal-AI hallucination measurement remains Magesh et al. (Stanford RegLab, 2024); replication of v2 by an independent academic group is encouraged and is cleanly enabled by the public CSV, the deterministic builder, the bare-Claude Arm B specification, and the two-axis coding rubric.

---

## Reproducibility

Anyone can rebuild the dataset from the bundle:

```
python3 build/build-v2-200queries.py
sha256sum data/clara_v2_audit_queries_200.csv
```

Anyone with API access to CLARA can re-run the audit by checking out the pinned commit and feeding `data/clara_v2_audit_queries_200.csv` through the runner under `tests/hallucination-audit/`. Re-runs against later CLARA commits are explicitly **not** the registered v2 audit and must be reported as separate runs.

---

## Where to find what

| Artifact | Location |
|---|---|
| Full README and honest-framing disclosures | `README.md` |
| Construction, run protocol, coding procedure, threats to validity | `METHODOLOGY.md` |
| Hypotheses, stop rules, analysis plan, κ threshold | `prereg/preregistration.md` |
| Hit-code definitions and severity weights | `taxonomy/taxonomy.md` |
| Frozen 200-query dataset | `data/clara_v2_audit_queries_200.csv` |
| Deterministic builder | `build/build-v2-200queries.py` |
| Licenses (MIT for code, CC-BY-4.0 for data and docs) | `LICENSE-CODE`, `LICENSE-DATA` |
| Live mirror | _GitHub repo URL — added at registration_ |

---

**Bundle version:** 1.1.0 (Stanford-alignment update) · **CLARA commit pinned (Arm A):** `6c82d70` (April 22, 2026) · **Comparator (Arm B):** `claude-sonnet-4-5-20250929` · **License:** MIT (code) + CC-BY-4.0 (data, taxonomy, docs)
