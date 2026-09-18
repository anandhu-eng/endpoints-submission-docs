# Requirements you must meet

Every binding constraint, grouped by what it constrains, with a note on how to tell whether you
comply. Rules marked :material-alert-octagon:{ style="color:#c62828" } have a failure action of
**reject the submission**.

--8<-- "precedence-notice.md"

## Curve structure

| Requirement | How to check |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } **≥ 7 measurement points** | `submission-checker check` — rule `point-count` |
| **≤ 32 measurement points**, including any later additions | rule `point-cap` |
| :material-alert-octagon:{ style="color:#c62828" } **≥ 1 point at concurrency 1–32** (Ultra Low) | rule `ultra-low-concurrency-coverage` |
| :material-alert-octagon:{ style="color:#c62828" } **≥ 1 point in each of Low, Medium, High Concurrency** | rules `low-` / `med-` / `high-concurrency-coverage` |
| :material-alert-octagon:{ style="color:#c62828" } **`C_max` declared and > 32** | rule `max-concurrency-declared` |
| Every concurrency falls in a valid region | rule `concurrency-in-range` |

Regions are computed in log-2 space from your own `C_min` and `C_max`. See
[Plan your Pareto curve](../workflow/plan-your-curve.md) and
[Metrics and regions](../reference/metrics-and-regions.md).

## Per-point run requirements

| Requirement | Value | How to check |
|---|---|---|
| Load pattern | The benchmark's **fixed-concurrency** pattern only — `max_throughput` and `poisson` are invalid | rule `load-pattern` |
| Steady-state duration | **600 s** Ultra Low; **1,200 s** Low / Medium / High | rule `point-duration` |
| Completed queries | At least one pass over the dataset | rule `min-query-count` |
| Samples issued | A whole-number multiple of the dataset size | — |
| Streaming | `stream_all_chunks = true` | rule `streaming-config` |
| Sampling order | Performance: with replacement. Accuracy: without replacement | — |

!!! warning "Section 6 is pending ratification"
    The duration and query-count values are marked as example values subject to working-group
    ratification, though they are stated as locked and Task Force-approved for the v0.7 round.
    Check [Open questions](../help/open-questions.md) before planning around them.

## Warmup

Warmup is optional, capped at 24 hours per point, and entirely up to you, but it is
heavily constrained and heavily disclosed, because warmup state materially affects the measurement.

| Requirement | Detail |
|---|---|
| Excluded from metrics | Everything issued before `TEST_STARTED` |
| **No performance-dataset samples** | Direct use, subsets, truncations, or anything derived from them. If the client does use the performance dataset, **salting must be enabled** — and it is not on by default |
| Logs retained | Must be available for reviewer inspection; reviewers may cross-check against the dataset |
| Declared per point | `duration_s`, `requests_issued`, `requests_completed`, `data_source`, `concurrency`, `initialization_steps` |

Incomplete or ambiguous warmup documentation is explicit grounds for a Methodology objection.

## Accuracy

| Requirement | Detail |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } Accuracy results at every required point | The four mandatory region points, plus one Offline point if you submit Offline results |
| :material-alert-octagon:{ style="color:#c62828" } The results pass the quality target | **Single-turn:** every result passes. **Multi-turn:** the average of them passes |
| Placement (single-turn) | Same concurrency as the point, same instance, immediately after that point's performance run |
| Same configuration | Identical endpoint config, weights and software stack as the performance runs |
| Dataset | The **un-salted** accuracy dataset |
| Tolerance | **None.** Accuracy is a hard gate at compliance and throughout review |

!!! question "The targets are not published"
    Per-benchmark accuracy tolerance values are marked `[WIP]` upstream. They are not in any source
    this documentation could verify. Ask MLCommons. Tracked as **C2** in
    [Open questions](../help/open-questions.md).

## Seeds

| Requirement | Detail |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } One seed set per submission | Every point records the **same** set |
| Adoption window | The set must have been published for your `target_cohort` or one of the three preceding cohorts |
| Binding lifetime | Once bound, the set stays valid for that submission even after newer sets publish |
| Runtime match | The client's RNG seeds must equal the bound set's values |
| Salt entropy | At least 64 bits per query, generated at request-construction time, never stored in the dataset |
| Salt placement | **Between** the system prompt and the per-query user context |

For amendments, new or replacement points must match the original submission's bound set — the
four-cohort adoption test is not reapplied.

## Consistency across the curve

Same model, endpoint configuration, software stack, dataset and seed set at **every** point. Every
point's `system_desc.json` must describe the same system. Freeze the stack before the first run.

## Disclosure by division

| | Standardized | Serviced | RDI |
|---|---|---|---|
| Source code to reproduce | Required | N/A (remote optional) | Optional |
| Rack / node hardware | Required | Optional | Required |
| Primary accelerator | Required | Required | Required |
| Parallelism mapping (TP/EP/PP/DP) | Required | Optional | Optional |
| Software stack versions | Required | From public docs and API metadata | Required |
| Model name and version | Required | Required, as advertised | Required |
| Pricing and rate limits | — | **Required** | — |

Software disclosure must name, at minimum: the serving framework with version and commit or release
tag, the accelerator compute library and build, the driver version, and the operating system.

## Reproducibility

| Division | By a third party | On a third-party system | On a public endpoint |
|---|---|---|---|
| Standardized — CoP | Required | Required | N/A |
| Standardized — CoN | Required | Required | N/A |
| Serviced — CoN | Required | Optional | **Required** |
| RDI | Optional | Optional | Optional |

Variability margins during review: **10%** when an independent party re-runs, **5%** on the exact
same system. Both apply to `system_tps` and `tps_per_user` only, **not** to any latency percentile.

## Tokenization

Official output token counts are produced by the **client-side reference tokenizer**, applied once
to the reconstructed assistant message through the model's official reference chat template. Not by
your serving stack, and not as a sum of per-chunk counts.

| Content | Counted? |
|---|---|
| Visible output | Yes |
| Tool-call content | Yes — reassembled by ascending tool-call index |
| Reasoning / thinking | Yes |
| Chat-template framing | Only payload-specific framing; empty-assistant framing is subtracted out |

!!! danger "Padding and duplication invalidate the point"
    Each received fragment must be assigned to exactly one response field and must not contribute
    twice. Deliberately duplicating substantially identical content across fields, or padding any
    field to inflate reported token counts, is prohibited and **invalidates the affected
    measurement point**.

    Emitting whitespace, control characters or punctuation just to stop the TTFT clock, rather than
    as a genuine part of the response, is also not allowed.

## See also

- [Model equivalence](model-equivalence.md) — if you are submitting Standardized
- [Publication status](publication-status.md) — the Available / Preview / RDI tests
- [Compliance checks](../reference/compliance-checks.md) — rule ID to clause cross-walk

--8<-- "draft-rules-warning.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (a7ec3cc), 2026-09-19.*
