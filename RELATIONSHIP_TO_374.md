# Relationship to the Five-Phase 374-Query Audit

This bundle (the **v2 200-query preregistration**, bundle version `1.1.0`) is one of two CLARA hallucination-audit deposits on OSF. The other is the **five-phase 374-query registered report** (`clara-audit-374`). They are **complementary, not redundant**, and a reviewer who reads only one is missing half the picture.

This document explains how the two studies relate, what each one is designed to do that the other cannot, and why both are necessary.

---

## At a glance

| Dimension                       | This bundle (v2 200-query, v1.1.0)                                      | 374-query bundle (v1.0.0)                                                     |
|---------------------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| **Type**                        | Preregistration (methodology-first)                                     | Registered report (post-hoc archival of completed work)                       |
| **Status at OSF submission**    | Methodology timestamped before the run; no results yet                  | Five-phase audit complete; results published with the bundle                  |
| **Run design**                  | Two arms on the same 200 queries: Arm A = CLARA full stack, Arm B = bare Claude Sonnet 4.5 (Stanford-style baseline comparator) | Three arms preserved per prompt: CLARA post-fix, CLARA pre-correction, bare-LLM baseline |
| **Hypotheses**                  | H-A through H-G (per-response rate, trap×, H1 share, anchored, layer attribution, Stanford-comparable bare-LLM lift, false-premise resilience), pre-stated thresholds | None pre-stated; results read out post-hoc                                    |
| **Coding unit**                 | Per-response, two-axis (correctness × groundedness), Stanford 2024-aligned | Per-citation primary; per-response computed                                   |
| **κ acceptance threshold**      | Pooled pre-adjudication κ ≥ 0.70 is a hard publication gate              | κ not formally computed; documented as a methodology limitation               |
| **Adjudicator**                 | External to CLARA build team (committed in §9 of the prereg)             | Internal                                                                       |
| **Stanford alignment**          | Deliberate; comparability table in `prereg/preregistration.md` §11      | Partial; the v1.1.0 v2 design generalizes the bare-LLM-arm idea from the 374Q's third arm |

---

## What each study answers

### What the 374-query bundle does that this one cannot

- **Reports actual CLARA performance on a substantial query set with a published headline number** (~0.43% per-citation, ~3.6% per-response).
- **Provides three-arm side-by-side data** — every prompt has CLARA post-fix, CLARA pre-correction, and bare-LLM baseline responses captured in one CSV. A reader can re-derive any rate they want from the raw data. (v2 borrows the bare-LLM idea but uses two arms on a fresh prompt set; the third arm — pre-correction — only makes sense for a multi-phase find-and-fix audit and is not part of v2.)
- **Documents the find-and-fix cycle** — five architectural fixes, ~14 net-new blocklist entries, and five real-case false-positive corpus closures, all shipped during the audit window. This is what an internal QA pass looks like; it is not what an independent benchmark looks like.
- **Provides the per-firewall-layer accountability table** — `reports/firewall-effectiveness.clara-2026-04-21.md` lists, for each of the ~10 firewall layers, what fraction of fabrication attempts that layer caught. This is the public remediation backlog.

### What this bundle (v2, v1.1.0) does that the 374-query bundle cannot

- **Locks the methodology before the results exist.** The hypotheses (H-A through H-G), the κ ≥ 0.70 acceptance threshold, the substitution rule (no query may be added/removed/edited after registration), the two-arm SUT specification (Arm A pinned commit + Arm B pinned model snapshot), and the analysis plan are all timestamped on OSF before any v2 query is run. Nothing can be retro-fit.
- **Pre-commits to publishing whatever the data shows in both arms.** §8 of `prereg/preregistration.md` is a written commitment to publish disconfirming results — including the case in which CLARA's defense stack provides no meaningful lift over bare Claude (H-F disconfirmation). The 374-query audit is also published warts-and-all (Finding F-1, the inverse-coherence canary failures, etc.), but it was not pre-committed in writing.
- **Names an external adjudicator.** §9 of the preregistration commits the third-reviewer adjudicator to be external to the CLARA build team. The 374-query audit's adjudication was internal.
- **Pre-stated κ threshold.** If pre-adjudication pooled κ falls below 0.70, the v2 headline rate is **not** published. This is the single strongest credibility lever in either bundle and is the answer to "your coders work for you — how do we know they were consistent?"
- **Stanford-aligned methodology.** v2 deliberately borrows the methodology of Magesh et al. (Stanford RegLab, 2024) where reasonable: per-response unit of analysis, two-axis (correctness × groundedness) coding, four-bucket query taxonomy, bare-LLM comparator design, Wilson 95% CIs. The full point-by-point comparability table is in `prereg/preregistration.md` §11. The 374-query audit predates this alignment work.

---

## Why we need both

A skeptical reader can attack each study from the opposite direction:

- The 374-query audit has a favorable headline (~0.43% per-citation; ~3.6% per-response) but is not preregistered, the coders work for the build team, and the methodology bundles the find-and-fix cycle into the headline number. A skeptic can reasonably ask: "Would a fresh prompt set, scored with a Stanford-aligned rubric and an external adjudicator, produce the same number? And does CLARA's defense stack actually do anything compared to bare Claude?"
- The v2 200-query preregistration answers exactly those questions, but **only after it runs**. A skeptic reading v2 alone, before the results land, sees only a methodology document — no number to anchor against.

Together they answer both halves:

1. The 374-query audit shows what a real five-phase find-and-fix cycle produces, with all three arms preserved so any rate can be recomputed.
2. The v2 200-query preregistration commits, in writing and on a timestamped surface, to a single-shot, externally-adjudicated, κ-gated, two-arm, Stanford-aligned re-measurement on a fresh prompt set — and to publishing the result regardless of which way it cuts.

If you read only the 374-query bundle, you are looking at internal QA. If you read only this bundle, you are looking at a methodology document with no data. Reading both is the audit.

---

## Pointers

- **374-query bundle, OSF DOI:** _to be added once issued_
- **374-query bundle, GitHub:** [`DeeCarp-dev/CLARA_374Q`](https://github.com/DeeCarp-dev/CLARA_374Q) (tag `v1.0.0`)
- **374-query bundle, headline document:** `RESULTS.md` inside the 374 bundle (canonical numbers)
- **374-query bundle, methodology disclosure:** `METHODOLOGY.md` §1.2 (iterative remediation) and §3.5 (κ not formally computed)

---

**Both bundles use the same eight-code H-taxonomy** (`H1_FAB_CITE` … `H7_FORM_ERR`, `OTHER`; identical category definitions across the two bundles). v2 adds the two-axis (correctness × groundedness) coding layer **above** that taxonomy per the Stanford 2024 alignment (see `taxonomy/taxonomy.md` and `prereg/preregistration.md` §5.0); the 374-query bundle uses the H-codes directly without the upper two-axis layer. Hits in v2 will therefore translate cleanly back to the 374-query per-citation rate via the H-code label, with the additional per-response correctness/groundedness rate as a v2-specific Stanford-comparable metric.
