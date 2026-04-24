# CLARA Hallucination Audit — v2 (200-query methodology-first preregistration)

This repository accompanies an OSF.io preregistration of a fixed, 200-query dataset designed to measure citation-level hallucination behavior in CLARA, a Virginia-specific legal AI assistant. The dataset, the builder script that generated it, the hand-coding rubric, and the analysis plan are all frozen here so that a third party can reproduce the audit end-to-end.

## What this is

A **methodology-first preregistration**. We are publishing the queries, the coding rubric, and the analysis plan **before** running the v2 audit and recording the results. This is deliberately different from publishing a finished study with results attached; the point is for the methodology to be timestamped and immutable, so the eventual results cannot be retro-fit to the questions.

## What this is *not*

- **Not** a benchmark comparing CLARA against other commercial legal AI products. The 200 queries are Virginia-specific and were authored to probe failure modes we already suspect exist in CLARA's pipeline; they are not a fair fight against general-purpose models, and we do not run Lexis+ AI, Westlaw AI, or Practical Law AI against the same set.
- **Not** an industry-standard hallucination benchmark. The H1–H7 taxonomy beneath the two-axis definition (`taxonomy/taxonomy.md`) was developed in-house for this audit and weights citation fabrication more heavily than form errors. Other groups using different category definitions will get different numbers, and that is expected.
- **Not** automated. Final hits are determined by two human coders against the published rubric. No LLM-as-judge.

## What this *is* methodologically aligned with

v2 deliberately borrows the methodology of Magesh, Surani, Dahl, Suzgun, Manning, Ho, *Hallucination-Free? Assessing the Reliability of Leading AI Legal Research Tools* (Stanford RegLab, 2024), specifically: (a) the two-axis hallucination definition (correctness × groundedness); (b) the per-response unit of analysis; (c) the four-bucket query taxonomy (general / jurisdiction-specific / false-premise / factual-recall); (d) the bare-LLM comparator design (Arm B = bare Claude Sonnet 4.5, no retrieval, no firewall); (e) the 200-query order of magnitude; (f) Wilson 95% CIs. v2 deviates from Stanford in three principled ways: it is a single-vendor self-audit rather than vendor-independent academic work, the prompt set is Virginia-specific rather than general U.S. legal, and we report against a single underlying-LLM comparator rather than against multiple commercial systems. The full point-by-point comparability table is in `prereg/preregistration.md` §11.

## What's in the bundle

```
clara-audit-v2-200/
├── README.md                          # this file
├── LICENSE-CODE                       # MIT — covers build/ scripts
├── LICENSE-DATA                       # CC-BY-4.0 — covers data/ and taxonomy/
├── METHODOLOGY.md                     # how the dataset was built; how it will be coded
├── prereg/
│   └── preregistration.md             # hypotheses, sampling, stop rules, analysis plan
├── taxonomy/
│   └── taxonomy.md                    # H1–H7 + OTHER definitions and severity weights
├── data/
│   └── clara_v2_audit_queries_200.csv # the 200 queries (frozen)
├── build/
│   └── build-v2-200queries.py         # deterministic Python builder; no external deps
└── CHANGELOG.md                       # version history
```

## Dataset at a glance

- **200 queries**, IDs `v2-001` … `v2-200`, all in `chat` mode.
- **Schema:** `id, category, prompt, mode, domain, tags, expected_authorities, notes`
- **Category distribution (frozen):**

| category                | count |
|-------------------------|------:|
| `general_research`      |    80 |
| `jurisdiction_specific` |    50 |
| `factual_recall`        |    40 |
| `false_premise`         |    30 |
| **total**               | **200** |

- **20 legal domains** covered. Top-five by query count: `civil_procedure` (23), `vrlta` (18), `criminal` (18), `contracts` (16), `defamation` (16). Long tail covers ethics, plea-colloquy, contributory negligence, family law, trade secrets, bankruptcy, administrative law, and more.
- **37 trap-tagged queries** (target ≥20). Traps are queries with a known false premise, a high-risk caption family, or a deliberately stale-amendment framing — chosen specifically to probe known CLARA failure modes (caption-confusion fabrications, Rule 1:7 weekend/holiday extensions on amended statutes, stale med-mal cap, ghost-statute pattern, etc.).
- **14 anchored-tagged queries** target CLARA's anchored knowledge blocks (VRLTA emergency hearing, plea colloquy, contributory negligence + last-clear-chance, sovereign immunity Messina test, and the live med-mal cap window).
- **103 queries** name an `expected_authorities` set (a sentinel list of statutes or cases a competent answer should cite or distinguish). The remaining 97 are intentionally open-ended — coders judge correctness against the substantive answer rather than a fixed citation list.

## Systems under test (two-arm design)

The audit runs both arms on the same 200 queries.

**Arm A — CLARA full stack:**
- **Project:** CLARA (Virginia-specific legal AI assistant)
- **CLARA commit pinned for the v2 audit:** [`6c82d70`](https://github.com/<REPLACE-WITH-CLARA-MONOREPO-URL>/commit/6c82d7001a241ec00d0f6bf458302d7431e9cc3c) (April 22, 2026)
- **Firewall layers active at this commit:** Hallucination Firewall (incl. Virginia Code structural validator, reporter volume validator with V2 verdict pipeline, allowlist-first citation gate, citation normalization layer, RAG provenance tracker), Holding Coherence Validator, Quote Provenance Layer, Drift Detector, Jurisdiction Validator, Discovery Sanctions Firewall, AG Opinion Shadow-Mode verifier, Negative Treatment Service (Shadow Mode).
- **Routing:** `chat` mode, **Sonar = "Analyze"**, **Strategic toggle off**, default jurisdiction = Virginia, no privacy-mode bypass. (These exact settings are also recorded in `prereg/preregistration.md` §4.1 and `METHODOLOGY.md` §2.1.1.)

**Arm B — bare Claude Sonnet 4.5 (Stanford-style baseline comparator):**
- **Model:** `claude-sonnet-4-5-20250929` via Anthropic API direct call. Same underlying model CLARA uses, so any rate difference between arms is attributable to CLARA's architecture (retrieval, prompting, firewall) rather than to a different base model.
- **No retrieval, no tools, no CLARA system prompt, no firewall.** Minimal generic system prompt only: *"You are a legal research assistant. Answer the user's question about Virginia law accurately, citing primary authorities (statutes, cases, rules) where appropriate."*
- **Decoding:** `temperature = 0`, `max_tokens` matched to CLARA's chat default. Deterministic.
- **Purpose:** Isolate the architectural contribution of CLARA's defense stack. Arm A vs Arm B percentage-point reduction is the Stanford-comparable headline (see `prereg/preregistration.md` H-F).

The full audit will be re-runnable by anyone with API access (CLARA monorepo + Anthropic API key) by checking out the pinned commit, configuring the runner under `tests/hallucination-audit/`, and feeding the v2 CSV through it for both arms.

## Reproducing the dataset (not the audit)

The CSV is fully reproducible from the builder script. From this bundle:

```
python3 build/build-v2-200queries.py
diff data/clara_v2_audit_queries_200.csv ./clara_v2_audit_queries_200.csv
```

The diff should be empty. The builder has no external dependencies (stdlib only) and is deterministic — query order, IDs, JSON serialization of the `tags` and `expected_authorities` columns, and CSV quoting are all fixed.

## Honest framing

Two pre-emptive disclosures we want on the record:

1. **The taxonomy is ours.** `H1_FAB_CITE` (fabricated cite), `H2_FAB_HOLD` (real cite, fabricated holding), `H3_WRONG_JX`, `H4_NEG_TREAT`, `H5_QUOTE_PROV`, `H6_DEAD_LAW`, `H7_FORM_ERR`, and `OTHER` are defined in `taxonomy/taxonomy.md` with severity weights. Other audits use different definitions; comparing rates across taxonomies is not apples-to-apples.
2. **The queries are ours.** They were authored fresh for this audit (they do not duplicate the prior 374-query corpus) and they are biased toward Virginia practice and toward known CLARA failure modes. This is on purpose — a Virginia-specific assistant should be evaluated on Virginia-specific queries — but it means the headline rate from this audit should be read as "rate on this stress test," not as an open-domain LLM hallucination rate.

The OSF preregistration locks both decisions in writing before the run, which is the credibility lift this exercise is designed to produce.

## License

- **Code** (`build/`): MIT — see `LICENSE-CODE`.
- **Data and methodology** (`data/`, `taxonomy/`, `prereg/`, this README): CC-BY-4.0 — see `LICENSE-DATA`. Reuse with attribution.

## Citation

If you use this dataset, please cite the OSF registration DOI (added once issued) and the GitHub release tag (`v1.1.0`).
