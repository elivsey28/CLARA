# Changelog

All notable changes to the CLARA Hallucination Audit v2 bundle are recorded here. The bundle follows semantic versioning at the bundle level (not at the level of individual files inside it).

The OSF preregistration freezes a specific tag. Cosmetic post-tag fixes (typos, broken links) are logged here without bumping the registered version. Any change to the dataset, the taxonomy, or the methodology requires a new bundle version and a new OSF registration that cites the prior one.

## [Unreleased]

_Reserved for post-v1.1.0 typo / link fixes that do not require a new OSF registration._

## [1.1.0] — Pending OSF submission (Stanford-alignment update)

This is the version registered with OSF. v1.0.0 was a pre-submission draft on the same GitHub repository; v1.1.0 supersedes it and is the binding registered protocol. The v1.0.0 tag remains on GitHub for historical reference.

The dataset (`data/clara_v2_audit_queries_200.csv`) and builder (`build/build-v2-200queries.py`) are **byte-identical to v1.0.0**. Only the methodology, taxonomy, preregistration, and framing documents changed.

### Added (alignment with Magesh et al., Stanford RegLab, 2024)

- **Two-arm run design.** Arm A = CLARA full stack at pinned commit; Arm B = bare Claude Sonnet 4.5 (`claude-sonnet-4-5-20250929`) via Anthropic API with no retrieval, no firewall, and a minimal generic legal-research system prompt. Both arms run on the same 200 queries. The Arm A vs Arm B difference is the Stanford-comparable architectural-lift headline. (`prereg/preregistration.md` §4, `METHODOLOGY.md` §2.1)
- **Two-axis hallucination definition.** Every response is first scored on `correct: Y/N` (substantive answer accurate?) AND `grounded: Y/N` (cited authorities real, in-jurisdiction, good law, supportive?). A response is `CLEAN` only if both axes are `Y`. The H1–H7 + OTHER taxonomy is now a sub-classification of the failure modes beneath those axes. (`prereg/preregistration.md` §5.0, `METHODOLOGY.md` §3.0, `taxonomy/taxonomy.md` "Two-axis hallucination definition")
- **Hypothesis H-F (Stanford-comparable).** Arm A's per-response rate is at least 5× lower than Arm B's on the same 200 queries, with both a ratio threshold (`bare_rate / clara_rate ≥ 5`) and a percentage-point-difference threshold (`bare_rate − clara_rate ≥ 20` pp). (`prereg/preregistration.md` §2)
- **Hypothesis H-G (false-premise resilience).** On the 30 false-premise queries, Arm A's per-response rate is ≤ 10%; a refusal or correction of the false premise is scored `CLEAN`. (`prereg/preregistration.md` §2)
- **Special false-premise scoring rule.** A substantive answer that accepts a false premise is scored `correct = N` even if the response contains no fabricated citation. (`prereg/preregistration.md` §5.0, `taxonomy/taxonomy.md`)
- **Stanford comparability section** (`prereg/preregistration.md` §11) — point-by-point table of what we matched, what we deviated, and why.
- **Per-response unit of analysis** clarified throughout. The headline rate is `n_responses_with_at_least_one_hit / 200`, not `total_hits / total_citations`. The severity-weighted rate remains as an Arm-A-only secondary metric.
- **Per-arm reporting** in §6.1 confirmatory analyses, including per-arm Wilson 95% CIs and a Newcombe 95% CI for the between-arm difference.
- **RQ5** added to research questions for the Stanford-comparable per-arm comparison.

### Changed

- RQ1 reframed as the per-response rate (Stanford-comparable) rather than the per-query rate (which was ambiguous about the unit of analysis).
- §6.1 confirmatory analyses reorganized to report per-arm rates side-by-side and to evaluate the new H-F and H-G hypotheses.
- §9 (Conflicts of interest) appended with explicit reference to Stanford 2024 as the closest precedent for vendor-independent measurement, and an explicit invitation for academic replication.
- README "What this is *not*" / new "What this *is* methodologically aligned with" sections rewritten to declare deliberate Stanford alignment rather than declaring v2 to be unrelated work.
- ABSTRACT updated with the two-arm run protocol bullet, the two-axis coding bullet, the new H-F and H-G hypotheses, the Stanford-alignment section, and the v1.1.0 footer.
- METHODOLOGY §2 restructured into 2.1.1 (Arm A) / 2.1.2 (Arm B) / 2.2.1 (Arm A protocol) / 2.2.2 (Arm B protocol).
- METHODOLOGY §3 restructured to add §3.0 two-axis coding before §3.1 coders, with the per-query coding form expanded to include `arm`, `correct`, `grounded` fields.
- METHODOLOGY §3.6 reported numbers expanded to per-arm rates plus the between-arm difference and ratio.
- Stale "500-query" reference in `taxonomy/taxonomy.md` corrected to "200-query".
- Methodology and bundle version bumped to 1.1.0.

### Pinned (unchanged from v1.0.0)

- CLARA commit `6c82d7001a241ec00d0f6bf458302d7431e9cc3c` (April 22, 2026) for Arm A.
- Anthropic model identifier `claude-sonnet-4-5-20250929` for Arm B (newly pinned in v1.1.0).
- 200-query frozen dataset (`data/clara_v2_audit_queries_200.csv`) — SHA-256 unchanged from v1.0.0.
- Deterministic builder (`build/build-v2-200queries.py`) — SHA-256 unchanged from v1.0.0.

## [1.0.0] — Pre-submission draft (superseded by 1.1.0)

Initial draft for OSF preregistration; pushed to GitHub as tag `v1.0.0` for review. Single-arm design (CLARA full stack only), no two-axis coding, five hypotheses (H-A through H-E). Replaced before OSF submission by v1.1.0 (Stanford-alignment update — see above). The v1.0.0 tag remains on GitHub for historical reference.

### Added

- `README.md` — bundle overview and honest-framing disclosures.
- `METHODOLOGY.md` — dataset construction, run protocol, hand-coding procedure, threats to validity.
- `prereg/preregistration.md` — research questions, hypotheses, stop rules, analysis plan.
- `taxonomy/taxonomy.md` — H1–H7 + OTHER definitions, severity weights, out-of-scope categories.
- `data/clara_v2_audit_queries_200.csv` — 200 frozen queries (80 / 50 / 30 / 40 distribution; 37 trap-tagged; 14 anchored-tagged; 20 domains).
- `build/build-v2-200queries.py` — deterministic stdlib-only Python builder for the dataset.
- `LICENSE-CODE` (MIT) and `LICENSE-DATA` (CC-BY-4.0).

### Pinned

- CLARA commit `6c82d7001a241ec00d0f6bf458302d7431e9cc3c` (April 22, 2026) as the system under test for the v2 audit.
