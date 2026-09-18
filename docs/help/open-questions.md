# Open questions and WIP rules

What could still change under you, and what this documentation could not confirm. Read this before
committing accelerator time.

--8<-- "draft-rules-warning.md"

**Last reviewed:** 2026-09-19, against `endpoints_policies@v1.0_rules_dev` (a7ec3cc),
`endpoints-submission-cli@main` (f48ca84), `endpoints@main` (47cc5c8).

??? info "What changed at the 2026-09-19 review"
    The rules moved 31 commits. Three merged changes matter to a submitter:

    - **Accuracy is no longer one run per submission** — it is now required at every point §5.3
      lists. This also resolved the old §4.3-vs-§6.6 contradiction, which is dropped from this page.
    - **A steady-state window is now the official reporting basis** (new §4.4), which introduces
      **B8** and **B9** below.
    - **Agentic benchmarks** gained a metric and a chart, which introduces **C7** and **C8**.

    **B4** (seed sets) is partly resolved. The submission rules, the CLI and the reference client
    did not move.

## A. Registration and membership

| # | Question | Status |
|---|---|---|
| **A1** | Must a submitter's **organisation** be an MLCommons member, or is the individual's CLA enough? | The rules require the CLA from the individual and describe PRISM registration at "MLCommons Member Central using your organization email id". Membership is never stated as a precondition. **Unresolved — confirm with MLCommons.** |
| **A2** | Is there a separate "register for the round" step? | The eight-week advance registration is explicitly overridden and there are no fixed rounds. Reading: a PRISM account plus a scoped API key is the whole of registration. **Confirm.** |
| **A3** | What are the **PRISM and Member Central URLs**? | Neither appears in any rules document, CLI doc, or public MLCommons page. **Needed.** |
| **A4** | Who grants API-creation access, and how long does it take? | The rules say an email notification follows account creation. No turnaround is stated. **Needed** for planning. |

## B. Tooling and policy disagreements

Found while cross-reading the rules against the tooling. Each is documented at the relevant page;
none has been resolved by guesswork.

| # | Gap | Detail | This site follows |
|---|---|---|---|
| **B1** | **`system_info` config section** | The Submission Rules tell submitters to run the benchmark "with config.yml having `system_info` section (if you want to automatically capture the system description)". **No `system_info` exists anywhere in the reference client's schema**, and the CLI docs state plainly that `system_desc.json` is submitter-authored and not an endpoints artifact. Either the policy anticipates unshipped work, or the sentence is stale. | The tooling — you author `system_desc.json` by hand |
| **B2** | **Pareto updates** | The technical rules reference a post-submission update window and link "Submission Rules §8.1 Pareto Updates". **That section no longer exists** — §8.1 is now *Corrections*. The CLI confirms the removal: there is no `add-run`, and `update --run-ids` rejects additions. | The Submission Rules and the CLI — points are fixed at creation |
| **B3** | **Section-number drift** | The Submission Rules link "Endpoints Rules §7" for directory structure; it is §8.1. The checker's documentation cites §14/§15/§16 for metrics, accuracy and consistency; those sections do not exist. | Clause numbers as they actually are |
| **B4** | **Seed-set adoption cannot be checked** | *Partly resolved.* `seedset.yaml` merged on 2026-09-15 — one set, `id: A`, cohort `2026-10-C1` — and its values match the checker's bundled copy exactly, so the **values** are settled. But the checker's mirror still carries `cohorts: []` and a comment calling the upstream PR open, so `seed-set-adoption` still reports **SKIP**. The two files are also shaped differently: the published one nests under a `cohort:` key, the checker's is a bare `seed_sets:` list. | Bind set `A`, target `2026-10-C1`, and override the checker's file |
| **B5** | **TTFT percentile** | The public MLCommons benchmark page still advertises **TTFT P95**. The v1.0 rules require **P90** and state P95 was the v0.7 metric. | The rules — P90 |
| **B6** | **Result labels** | The rules use Available / Preview / RDI. The public page additionally shows "Verified / Provisional / Unverified". The relationship is unstated. | The rules — needs a mapping |
| **B7** | **File and field naming** | The rules refer to `system_desc_id.json` in places and to a `benchmark_model` field; the tooling uses `system_desc.json` and `model_name`. | The tooling spelling |
| **B8** | **Which metrics gate a steady-state window** | Three sources, three answers. §4.4's definition table says **TPOT at P50 and P90**. Its own next paragraph says **TTFT and TPOT at P50/P90**. The methodology document the section links to says **TTFT/TPOT at p50 and p95**. The percentile disagreement echoes **B5**. | Nothing — **you must ask** |
| **B9** | **The steady-state detector is not released** | §4.4 makes a detected steady-state window the official result, and §8.3 requires a `steady_state` block in every `point.yaml`. The methodology, the script (`steady_state_diagnostics.py`) and its documentation exist **only on an unmerged branch** of `mlcommons/endpoints` (`doc/alicheng-steady-state-design`); nothing matching is on `main` at 47cc5c8. The rules link a pinned commit on that branch. So a submitter cannot currently produce the field the rules require. | Nothing — **blocked until the tooling ships** |

## C. Content MLCommons must still supply

None of this is available in any source we could find.

| # | Item | Why it blocks you |
|---|---|---|
| **C1** | The **v1.0 supported model list** and canonical model IDs | The rules defer to "the reference repository", published ≥ 6 weeks before a round. The client ships a ruleset named `mlperf-inference-v6.1`, which is not an Endpoints ruleset name. `model-name-valid` checks against this list |
| **C2** | **Per-benchmark accuracy targets and tolerances** | Marked `[WIP]` upstream. `accuracy-gate` is a hard reject and you cannot predict whether you pass |
| **C3** | **Dataset identities and download paths** for performance and accuracy runs | You cannot run without them |
| **C4** | **CoN client locations and scheduling procedure** | Deferred to a separate working-group publication that does not yet exist. CoN submitters cannot plan |
| **C5** | **The full submission-state list** | Only `REVIEW_PENDING`, `WITHDRAWN`, `FINALIZED` and `PUBLISHED` are documented |
| **C6** | **Preview Availability Tracker URL** and the public results/visualizer URL | Referenced by the rules; no URL given |
| **C7** | **What an "Offline" result is** | §5.3 requires an extra accuracy point "if Offline results are submitted". The word appears **once** in the entire rules document and is never defined — there is no Offline scenario, division or run mode anywhere else. You cannot tell whether this applies to you |
| **C8** | **Which benchmarks are agentic, and which are multi-turn** | The accuracy gate differs (every point vs mean-of-N), the primary chart differs (`tps_per_user` vs `e2e_avg_interactivity`), and agentic salting is enabled by unnamed "benchmark-specific flags". None of the three is mapped to a benchmark. Ties to **C1** |
| **C9** | **Super-pass size per benchmark** | The steady-state window is measured in super-passes, defaulting to one full dataset pass "unless the benchmark definition specifies a different super-pass size". No benchmark definition is published, so you cannot compute your own floor |

## D. Rules the working group has not ratified

Marked upstream as `[TENTATIVE]`, `[WIP]`, `[WG Open Item]` or `[WG Decision Required]`. These are
current policy where stated, but are the most likely to move.

### Likely to affect your run plan

| Area | What is unsettled |
|---|---|
| **Run requirements** | The entire run-requirements section is under active development. Minimum durations (600 s / 1,200 s), minimum query counts, dataset-subset rules, and the warmup model — submitter discretion plus mandatory disclosure, in place of a fixed warmup duration — are all pending ratification, though stated as locked and Task Force-approved for the v0.7 round |
| **Accuracy targets** | Per-benchmark tolerance values are `[WIP]` |
| **Steady-state reporting** | §4.4 carries its own pending-ratification note: whether the 4 super-pass floor rises, and whether a run where no steady state is found is declared **invalid** rather than merely reported-with-flags. The second matters most — it decides whether a run that fails detection is a failed run or just a weaker number. See also **B8** and **B9** |
| **Reproducibility margins** | The 10% and 5% throughput margins are proposals requiring ratification before they can be enforced |
| **Latency comparison method** | **None exists.** A reproducibility objection may not rest on latency alone, and a Preview-to-Available transition is not blocked on latency alone. The working group is weighing histogram-based methods, which would first require the per-metric histogram to become a required artifact |

### Likely to affect your classification

| Area | What is unsettled |
|---|---|
| **Custom SKUs** `[CUSTOM-SKU]` | How to classify hardware in production at hyperscalers but not orderable by any comparable customer. It fails Available criterion 2 and may have no GA commitment, so cannot be Preview — but it is not a prototype. Options: classify as RDI (current default, with the 221-day cooling-off), or create a new "Production" tier. **Defaults to RDI until decided** |
| **RDI comparability** `[RDI-COMP]` | Whether RDI results should be directly comparable to Available and Preview on the same charts. Currently all three are plotted together |
| **Serviced requirements** `[SERVICED-REQ]` | Which system-description fields are required versus optional for APIs; how to handle API versioning when the underlying model or stack changes without notice; whether Serviced results should carry a permanent reproducibility disclaimer |

### Likely to affect your optimisation choices

| Area | What is unsettled |
|---|---|
| **Checkpoint residency** `[CKPT-RESIDENCY]` | Whether a component present in the canonical checkpoint — an MTP or EAGLE head — must also be **resident in accelerator memory** during measurement. The rules require it in the *artifact*; residency is undecided. Memory freed by not loading it converts directly into KV-cache capacity and therefore throughput, so this is a real and currently undisclosed advantage. Options: require residency, require disclosure of the loaded component set, or leave unconstrained |
| **Token counting** `[TOK-COUNT]` | Whether stakeholders accept that published numbers may differ from serving-stack-reported numbers, given the reference-chat-template tokenization rule. Resolution needed before v1.0 publishes side-by-side charts |
| **Iteration coalescing** | Whether the server returning multiple generated tokens in a single network message is allowed. **Until resolved: disclose any token-coalescing behaviour and conservatively assume `stream_all_chunks = true` semantics** |
| **Standardized CoN techniques** | A comprehensive list of allowed techniques and optimizations for the Standardized CoN scenario is still under development |
| **Partial Unicode at chunk boundaries** | An open edge case in the tokenizer rules |

### Likely to affect review

| Area | What is unsettled |
|---|---|
| **Late objections** | The entire late-objection policy is `[WIP — pending WG approval]`. The grounds, process and time limits are a current proposal |
| **Dispute resolution** | Panel composition, deadlines, non-participation consequences and the 8-week backstop require ratification. Still unspecified: quorum and voting among neutral members, confidentiality provisions, and whether a standing roster of pre-cleared neutrals should be maintained |
| **Audit process** | To be defined in a separate document. Current proposal: up to 2 audits per quarter, chair-selected. Selection criteria and procedures are not finalized |
| **Audit votes** | Cadence, quorum and voting rule are undecided — and because the vote cadence determines how long a result stays challengeable, this blocks the late-concern window too |
| **Audit nominations** | Whether nominated audits count against the 2-per-quarter capacity, and whether a nominating member bears any audit cost. A nomination route with no capacity guarantee may defer indefinitely |
| **Review chair membership** | Whether the review chair must be an MLCommons member |

## How to use this page

- **Before planning a submission** — read sections A and C. If anything there blocks you,
  [ask](support.md) before scheduling hardware.
- **Before tuning** — read *Likely to affect your optimisation choices*. Building a submission on a
  technique whose status is open is a risk you should take knowingly.
- **Before claiming Available** — read *Likely to affect your classification*.

!!! tip "When in doubt, disclose"
    Several open items resolve the same way in practice: where a rule is unsettled, disclosing what
    you did turns a potential compliance failure into a reviewable choice.
