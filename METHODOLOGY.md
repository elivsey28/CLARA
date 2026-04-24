# Methodology — CLARA Hallucination Audit v2 (200 queries)

This document explains, in enough detail for a third party to reproduce, **how the 200-query dataset was constructed**, **how CLARA will be queried**, and **how the resulting responses will be hand-coded**. It is the companion to `prereg/preregistration.md`, which states the hypotheses and analysis plan, and to `taxonomy/taxonomy.md`, which defines the codes a coder may apply.

The intent of this document is to make the methodology auditable, not to defend the choices. Where we made a judgment call, we say so.

---

## 1. Dataset construction

### 1.1 Source of queries

All 200 queries were authored by the CLARA team specifically for this audit. They were written without reference to the prior 374-query corpus (`clara-audit-dataset-374queries.csv`) so that v2 results cannot be inflated by overlap with previously seen prompts. The builder script (`build/build-v2-200queries.py`) is the canonical source: every query in `data/clara_v2_audit_queries_200.csv` corresponds to exactly one literal in the script, and the script's output has been byte-equal to the CSV since v1.0.0.

### 1.2 Schema

Each row has eight columns:

| column                 | type     | meaning                                                                 |
|------------------------|----------|-------------------------------------------------------------------------|
| `id`                   | string   | Stable identifier `v2-001` … `v2-200`. Order is locked at registration. |
| `category`             | enum     | `general_research`, `jurisdiction_specific`, `false_premise`, `factual_recall`. |
| `prompt`               | string   | The exact text sent to CLARA. No template substitution.                 |
| `mode`                 | enum     | All `chat` for v2 (DRAFT mode reserved for a later phase).              |
| `domain`               | string   | One of 20 legal-practice domain tags (see §1.4).                        |
| `tags`                 | JSON arr | Free-form descriptors (`doctrine`, `trap`, `anchored`, `recent-amendment`, etc.). |
| `expected_authorities` | JSON arr | Sentinel statutes/cases a competent answer should cite or distinguish. May be empty. |
| `notes`                | string   | Coder hint describing what a correct answer looks like.                 |

### 1.3 Category definitions

- **`general_research` (n = 80).** Open-ended doctrinal questions about Virginia law that a junior associate would ask before drafting. Designed to probe whether CLARA's normal research path produces clean citations under low-stress framing.
- **`jurisdiction_specific` (n = 50).** Questions where the answer turns on a Virginia-specific rule, statute, or local-court practice. These probe CLARA's "Virginia Rosetta Stone" and jurisdictional-locking layer.
- **`false_premise` (n = 30).** Questions that bake a factually wrong premise into the prompt (wrong cap amount, wrong rule number, wrong recent-amendment narrative, etc.). A correct CLARA response **must** correct the premise rather than answer it on its own terms. These directly probe the Nuclear Gatekeeper, the malpractice firewall, and the stale-law / dead-law detection paths.
- **`factual_recall` (n = 40).** Questions whose correct answer is a single, verifiable fact (a statute number, a deadline, a holding, a case caption). High signal-to-noise for citation-fabrication failure modes. A subset of these are deliberate caption-collision traps (e.g., asking for a case at a volume/page combination that does not exist).

The 80 / 50 / 30 / 40 distribution is locked in `prereg/preregistration.md` and reproduced by the builder via the row-by-row append order. Any change to the distribution before the audit closes invalidates the preregistration.

### 1.4 Domain coverage

20 domains with the following primary / secondary stratification (counts from the frozen CSV):

**Primary domains (≥10 queries each):**

| domain               | n  |
|----------------------|---:|
| `civil_procedure`    | 23 |
| `vrlta`              | 18 |
| `criminal`           | 18 |
| `contracts`          | 16 |
| `defamation`         | 16 |
| `evidence`           | 15 |
| `sovereign_immunity` | 14 |
| `med_mal`            | 14 |
| `discovery`          | 13 |

**Secondary domains (3–9 queries each):** `deadlines` (9), `appellate` (8), `employment` (6), `ethics` (5), `plea_colloquy` (4), `contributory_negligence` (4), `family_law` (4), `discovery_sanctions` (4), `trade_secrets` (3), `bankruptcy` (3), `administrative_law` (3).

The primary set is over-weighted toward domains where CLARA has anchored knowledge blocks (VRLTA, sovereign immunity, contributory negligence, plea colloquy, med-mal cap, discovery sanctions). This is intentional: those blocks are the part of the system most subject to drift and most likely to produce a confident, well-formatted, *wrong* answer if the underlying anchor moves.

### 1.5 Trap-tagged queries

37 of the 200 queries (target ≥20) carry a `trap` tag in their `tags` column. A trap query satisfies at least one of:

1. The premise is verifiably false and a correct response must refuse / correct it.
2. The query asks for a citation at a known phantom volume/page or for a known phantom statute section.
3. The query references a "recent amendment" that does not exist.
4. The query names a caption family with high collision risk (e.g., `Thompson v. Commonwealth`, `Williams v. Commonwealth`) at a non-existent volume/page.

The 37 / 200 ratio is set so that even if every non-trap query produces a clean response, the audit can still produce a meaningful denominator-weighted hallucination rate from the trap subset alone. **The trap rate and the non-trap rate will be reported separately** in the final paper to avoid headline inflation.

### 1.6 Anchored-tagged queries

14 queries carry an `anchored` tag, meaning their correct answer touches an anchored CLARA knowledge block. These are scored against the anchor's own ground truth (e.g., the D2 VRLTA § 55.1-1244(F) emergency-hearing provision, the Messina four-factor test for sovereign immunity, the live med-mal cap window). A drift between CLARA's response and the anchor is recorded as `H6_DEAD_LAW` if the anchor moved, or `H2_FAB_HOLD` if the anchor is correct and CLARA misstates it.

### 1.7 Builder properties

`build/build-v2-200queries.py`:

- Pure-stdlib Python 3.10+. No `pip install` step.
- Deterministic output: row order, ID assignment, JSON encoding of array columns (`json.dumps(..., ensure_ascii=False)`), and CSV quoting (`csv.QUOTE_ALL`) are all fixed.
- Includes a self-check: warns on stderr if `len(ROWS) != 200`; prints category, domain, and trap counts at the end of every run.
- Re-running the script overwrites the CSV in place with byte-identical output for the same input ROWS list.

A reproducer should be able to:

```
git checkout v1.0.0
python3 build/build-v2-200queries.py
sha256sum data/clara_v2_audit_queries_200.csv
```

and obtain the same SHA-256 as the registered artifact.

---

## 2. Run procedure (the audit itself)

### 2.1 Systems under test (two-arm design)

The v2 audit runs both arms on the same 200 queries from `data/clara_v2_audit_queries_200.csv` and codes them under the same rubric (see §3.0). Arm B is the Stanford-style bare-LLM comparator, included so that v2's headline can be reported as the percentage-point reduction CLARA's defense stack contributes over the underlying LLM alone (see `prereg/preregistration.md` §6.1.2 and H-F).

#### 2.1.1 Arm A — CLARA full stack

- **CLARA commit:** `6c82d7001a241ec00d0f6bf458302d7431e9cc3c` (April 22, 2026). Re-runs against later commits are explicitly **not** the registered v2 audit and must be reported as separate runs.
- **Mode:** `chat` (Sonar Mode = "Analyze", Strategic Mode = off, privacy mode = off, jurisdiction = Virginia).
- **All firewall layers ENABLED** at default production settings, including but not limited to: Hallucination Firewall (Virginia Code structural validator, reporter volume validator V2, allowlist-first citation gate, citation normalization, RAG provenance tracker, ghost-statute blocklist, KNOWN_FABRICATED_CITATIONS list), Holding Coherence Validator, Quote Provenance Layer, Drift Detector, Jurisdiction Validator, Discovery Sanctions Firewall, AG Opinion Shadow-Mode verifier, Negative Treatment Service (Shadow).
- **No human-in-the-loop intervention** during the run. Verification Interceptor pop-ups, if any, are recorded as raw response artifacts; coders do not click through.

#### 2.1.2 Arm B — bare Claude Sonnet 4.5 (Stanford-style baseline comparator)

- **Model:** `claude-sonnet-4-5-20250929` via Anthropic API direct call. This is the same underlying model CLARA uses, so any rate difference between the two arms is attributable to CLARA's architecture (retrieval, prompting, firewall) rather than to a different base model.
- **No retrieval, no tools, no CLARA system prompt, no firewall, no post-generation auditing.**
- **Minimal generic system prompt** (recorded verbatim, used identically for all 200 queries):

  > You are a legal research assistant. Answer the user's question about Virginia law accurately, citing primary authorities (statutes, cases, rules) where appropriate.

- **Decoding:** `temperature = 0`, `max_tokens` matched to CLARA's chat mode default (4096 unless capped by the API for the model). Deterministic decoding so the run is reproducible.
- **One fresh API call per query.** No retries. No conversational context across queries.

The intent of Arm B is to isolate the architectural contribution of CLARA's defense stack. It is **not** a benchmark of bare Claude Sonnet 4.5's general legal-research ability — Anthropic does not market the bare model as a legal-research product, and the per-arm comparison should be read accordingly.

### 2.2 Per-query protocol

For each row in `data/clara_v2_audit_queries_200.csv`, in `id` order, executed independently for each arm:

#### 2.2.1 Arm A (CLARA full stack)

1. Send the `prompt` to a fresh CLARA chat session (no shared session state across queries).
2. Capture: full SSE stream concatenated to a single response string, all firewall verdicts emitted by the post-generation pipeline, latency, and the model identifier returned in the trace.
3. Persist a JSON record per query under `runs/v2-<timestamp>/arm-a/<id>.json` containing `{id, prompt, response_text, firewall_verdicts, model, latency_ms, raw_trace}`.

#### 2.2.2 Arm B (bare Claude Sonnet 4.5)

1. Open a fresh Anthropic API client per query (no retained client state, no message history).
2. Send the system prompt from §2.1.2 followed by the `prompt` as a single user message.
3. Capture: full assistant message text, finish reason, model identifier from the API response, latency.
4. Persist a JSON record per query under `runs/v2-<timestamp>/arm-b/<id>.json` containing `{id, prompt, system_prompt, response_text, model, finish_reason, latency_ms, raw_request, raw_response}`.

No retries in either arm. If a query errors, the error is logged and the query is marked `ERROR` for the coding pass; coders score errors as `OTHER` only if the error itself contains a fabricated citation in its message.

### 2.3 Substitution rule

**No query may be substituted, rewritten, or removed after registration.** If a query becomes ambiguous in light of a Virginia legislative change between registration and run, it is scored against the law as it stood **at the registration date** (April 22, 2026). The fact that the law later changed is logged in `notes` for the final paper but does not change the verdict.

---

## 3. Hand-coding procedure

### 3.0 Two-axis coding (Stanford-aligned)

For every response in both arms, each coder records two boolean axes **before** assigning any taxonomy code (Stanford 2024 hallucination definition; see `prereg/preregistration.md` §5.0):

| field      | values  | meaning                                                                                  |
|------------|---------|------------------------------------------------------------------------------------------|
| `correct`  | `Y`/`N` | Does the substantive answer accurately state Virginia law on the question asked?         |
| `grounded` | `Y`/`N` | Are the cited authorities (a) real, (b) from the represented jurisdiction, (c) good law, and (d) actually supportive of the cited proposition? |

A response is `CLEAN` only if `correct = Y` AND `grounded = Y`. A response with either axis = N is hallucinated and receives one or more taxonomy codes (`H1`–`H7`, `OTHER`) describing the failure mode. The two-axis rate (Stanford-comparable) is reported alongside the taxonomy-coded rate (CLARA-specific failure-mode breakdown).

**Special rule for `false_premise` category queries:** the correct behavior is refusal or correction of the false premise. A substantive answer that accepts the false premise is scored `correct = N` even if the response contains no fabricated citation; the failure is on the correctness axis, not the grounding axis.

### 3.1 Coders

Two coders, both attorneys licensed in Virginia or supervised by one. Coders are **blind to each other's verdicts** until both have completed all 400 responses (200 per arm). Where the response text does not itself reveal arm identity, responses are presented in randomized arm-blinded order; CLARA-specific formatting tokens (e.g., `[CITATION UNVERIFIED — DO NOT USE]`) cannot be blinded and coders are instructed to score on substance, not format.

### 3.2 Per-query coding form

For each of the 400 responses (200 per arm), each coder records:

| field                    | values                                                                 |
|--------------------------|------------------------------------------------------------------------|
| `id`                     | matches dataset ID                                                     |
| `arm`                    | `A` (CLARA) or `B` (bare Claude)                                       |
| `correct`                | `Y` or `N` (per §3.0)                                                  |
| `grounded`               | `Y` or `N` (per §3.0)                                                  |
| `category_code`          | one of `H1_FAB_CITE`, `H2_FAB_HOLD`, `H3_WRONG_JX`, `H4_NEG_TREAT`, `H5_QUOTE_PROV`, `H6_DEAD_LAW`, `H7_FORM_ERR`, `OTHER`, or `CLEAN` |
| `responsible_layer`      | which firewall layer should have caught the hit (multi-select); `n/a` for Arm B (no firewall) and for `CLEAN` responses |
| `verification_method`    | how the coder confirmed (CourtListener exact-cite search, Va. LIS, Westlaw, Lexis, official Va. Code, anchor block reference) |
| `verification_artifact`  | URL or screenshot path supporting the verdict                          |
| `notes`                  | free text                                                              |

A response may receive **multiple** category codes if it contains multiple distinct hits (e.g., one fabricated citation and one quote provenance miss). Each is logged as a separate row in the coding sheet under the same `id` + `arm` pair.

### 3.3 Hit definitions

Defined in `taxonomy/taxonomy.md`. Reproduced here at headline level only:

- `H1_FAB_CITE` — case/statute/rule does not exist (weight 3.0)
- `H2_FAB_HOLD` — real cite, holding not supported by source (weight 3.0)
- `H3_WRONG_JX` — real cite, wrong jurisdiction (weight 2.0)
- `H4_NEG_TREAT` — real cite, overruled / abrogated, presented as good law (weight 2.0)
- `H5_QUOTE_PROV` — quoted text does not appear in source as quoted (weight 1.5)
- `H6_DEAD_LAW` — repealed / renumbered statute presented as live (weight 2.0)
- `H7_FORM_ERR` — wrong volume / pin / year on a real authority (weight 0.5)
- `OTHER` — fits no above category; triggers taxonomy review (provisional 1.0)

The taxonomy file is itself frozen at registration. Any new code added during coding requires the taxonomy version to bump, the change to be logged in `CHANGELOG.md`, and the audit results to be reported under both the old and new versions.

### 3.4 Out-of-scope (not hallucinations)

Restated from `taxonomy/taxonomy.md` for emphasis:

- Tone / stylistic disagreements.
- Strategic disagreements ("I would have argued X").
- Correct refusals.
- `[CITATION UNVERIFIED — DO NOT USE]` blocks — these are **successful** firewall catches.
- `[MISSING_FACT: …]` placeholders in DRAFT mode — correct behavior. (v2 is chat-mode-only, so this is unlikely to arise.)

### 3.5 Adjudication

After both coders finish independently, a third reviewer compares the two coding sheets row by row. For each disagreement:

1. The third reviewer **does not** re-code from scratch; they choose between the two existing verdicts based on the recorded `verification_artifact`.
2. Coders may submit a one-paragraph rebuttal before the third reviewer decides.
3. The final verdict is the third reviewer's choice and is the value used in the published rates.

**Inter-rater reliability is reported regardless of adjudication outcome.** Cohen's κ is computed on the pre-adjudication verdicts (CLEAN vs. each H-code as a multi-class label). κ ≥ 0.70 is the pre-registered acceptance threshold; κ below 0.70 invalidates the headline rate and triggers a methodology review before any number is published.

### 3.6 Reported numbers

**For each arm independently:**

- `per_response_rate` (Stanford-comparable headline) = `n_responses_with_at_least_one_hit / 200`, where a "hit" is `correct = N` OR `grounded = N`. Reported with a Wilson 95% CI.
- `correctness_failure_rate` = `n_responses_with_correct=N / 200`
- `groundedness_failure_rate` = `n_responses_with_grounded=N / 200`
- `joint_failure_rate` = `n_responses_with_both_axes_failing / 200`
- `false_premise_rate` = rate on the 30 false-premise category queries only

**For Arm A only:**

- `severity_weighted_rate = Σ (n_hits_i × weight_i) / 200` (CLARA-internal metric, not reported for Arm B)
- `trap_rate` and `non_trap_rate` reported separately
- Per-firewall-layer accountability table listing, for each layer, the count of Arm A hits attributed to that layer. This becomes the public remediation backlog.

**Arm A vs Arm B (the Stanford-comparable architectural-lift summary):**

- `difference = bare_rate − clara_rate` (percentage points), with a Newcombe 95% CI
- `ratio = bare_rate / clara_rate`
- These two together evaluate H-F.

---

## 4. Threats to validity (acknowledged up front)

1. **Construct validity.** Our taxonomy weights citation fabrication 6× higher than form errors. A reader using a flat 1.0 weight will get a different headline number from the same data. We publish both the unweighted and weighted rates to make the comparison explicit.
2. **Selection bias.** Queries were authored by the people who built CLARA. We mitigate by (a) tagging trap queries explicitly, (b) reporting trap and non-trap rates separately, and (c) freezing the dataset at registration so we cannot quietly drop a query that fires.
3. **Coder bias.** Coders may be sympathetic to or hostile to the system. Mitigation is two-coder blind coding plus third-reviewer adjudication, plus pre-registered κ ≥ 0.70 threshold.
4. **Pipeline drift.** CLARA is under active development; the pinned commit may not match what users see in production a week later. We pin the commit explicitly and report all results against that pin.
5. **Generalization.** Results from this dataset do not generalize beyond Virginia practice and beyond the failure modes we chose to probe. We say so in `README.md` and we say so again here.

---

## 5. Versioning

This methodology is `v1.1.0` (Stanford-alignment update; supersedes pre-submission draft v1.0.0 — see `CHANGELOG.md`), frozen at OSF registration. Material changes (new queries, new taxonomy codes, changes to the run protocol) require a new methodology version, a new dataset version, and a new OSF registration that cites this one. Cosmetic edits (typos, link corrections) are tracked in `CHANGELOG.md` without bumping the version.
