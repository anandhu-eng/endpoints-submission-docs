# Compliance checks

Every automated check, cross-walked from **checker rule ID** to the **rules clause** it enforces and
the **failure action** the rules assign. Use it to work out what a failed check actually means.

Run locally with [`submission-checker`](cli-checker.md); run server-side during **Week 0**.

!!! danger "Week 0 failures reject the submission"
    A submission that fails any automated check by the end of Week 0 is rejected. You correct and
    resubmit as a **new** submission — there is no in-place patching, and you lose your cohort slot.

## How to read the severity column

| Severity | Meaning |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } **Reject** | The rules assign *Reject submission* to this check |
| :material-alert:{ style="color:#ef6c00" } **Reject points** | Non-conforming points are rejected |
| :material-flag:{ style="color:#f9a825" } **Flag** | Flagged for reviewer attention — does not block automatically, but is objection material |
| :material-information:{ style="color:#1565c0" } **Warn** | Checker-level warning; becomes an error under `--strict` |

## Structure

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `path-exists` | §1 | Submission root directory exists | :material-alert-octagon:{ style="color:#c62828" } |
| `required-dir` | §1 | `results/` and `docs/` present | :material-alert-octagon:{ style="color:#c62828" } |
| `src-dir` | §2.2.1 | `src/` present with at least one implementation directory | :material-alert-octagon:{ style="color:#c62828" } |
| `src-readme` | §2.2.1 | Each `src/<implementation>/` has a `README.md` | :material-alert-octagon:{ style="color:#c62828" } |
| `system-results-dir` | §1 | At least one `results/<system>/` exists | :material-alert-octagon:{ style="color:#c62828" } |
| `benchmark-model-dir` | §1 | At least one model directory per system | :material-alert-octagon:{ style="color:#c62828" } |
| `point-dirs` | §1 | At least one `r<N>/` point directory per model | :material-alert-octagon:{ style="color:#c62828" } |
| `measurement-points-present` | §1 | Every `r<N>/` carries a `point.yaml` | :material-alert-octagon:{ style="color:#c62828" } |
| `result-summary-present` | §1 | `result_summary.json` exists for each point | :material-alert-octagon:{ style="color:#c62828" } |
| `shared-path-resolution` | §9.1 | `shared_src` / `shared_docs` resolve under the submission root | :material-alert-octagon:{ style="color:#c62828" } |
| `point-dirname-concurrency` | §1 | `r<N>/` name matches the declared concurrency | :material-information:{ style="color:#1565c0" } |

## System description

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `system-description-present` | §8.2 | Every point has a `system_desc.json` | :material-alert-octagon:{ style="color:#c62828" } |
| `system-description-valid` | §8.2 | Parses against the `SystemDescription` schema | :material-alert-octagon:{ style="color:#c62828" } |
| `system-description-consistency` | §9.1 | Every point of a curve describes the same system | :material-flag:{ style="color:#f9a825" } |
| `model-name-valid` | §3.2 | `model_name` is one of the round's supported models | :material-alert-octagon:{ style="color:#c62828" } |
| `model-name-consistency` | §8.2 | Matches the results directory name | :material-flag:{ style="color:#f9a825" } |
| `max-concurrency-declared` | §9.1 | `max_supported_concurrency` present and > 32 | :material-alert-octagon:{ style="color:#c62828" } |
| `tps-utilization` | §8.2 | Equals `system_tps / max(system_tps)` over the point's own curve | :material-flag:{ style="color:#f9a825" } |

## Regions and curve structure

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `region-basis` | §5.4 | Reports the derived `C_min` and how many points it came from | :material-information:{ style="color:#1565c0" } |
| `region-computation` | §5.5 | `(C_max, C_min)` is a valid input to the reference algorithm | :material-alert-octagon:{ style="color:#c62828" } |
| `concurrency-in-range` | §9.1 | Each concurrency falls in a valid region, margin included | :material-flag:{ style="color:#f9a825" } |
| `region-declared` | §8.3 | Declared `region` is one of the permitted values | :material-alert:{ style="color:#ef6c00" } |
| `region-placement` | §8.3 | Declared region matches the computed one | :material-information:{ style="color:#1565c0" } |
| `ultra-low-concurrency-coverage` | §5.4, §9.1 | At least one point at concurrency ≤ 32 | :material-alert-octagon:{ style="color:#c62828" } |
| `low-concurrency-coverage` | §9.1 | At least one point in Low Concurrency | :material-alert-octagon:{ style="color:#c62828" } |
| `med-concurrency-coverage` | §9.1 | At least one point in Medium Concurrency | :material-alert-octagon:{ style="color:#c62828" } |
| `high-concurrency-coverage` | §9.1 | At least one point in High Concurrency | :material-alert-octagon:{ style="color:#c62828" } |
| `point-count` | §5.3, §9.1 | 7–32 measurement points | :material-alert-octagon:{ style="color:#c62828" } |
| `point-cap` | §5.6, §9.1 | Does not exceed 32 points | :material-alert:{ style="color:#ef6c00" } |

!!! warning "The 10% margin does not satisfy High Concurrency"
    A point in `C_max + 1 … ceil(1.10 × C_max)` is in its own region. It passes
    `concurrency-in-range` but does **not** count toward `high-concurrency-coverage`.

## Measurement points

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `point-config-valid` | §8.3 | `point.yaml` parses against the `PointConfig` schema | :material-alert-octagon:{ style="color:#c62828" } |
| `point-disclosure-complete` | §8.3 | Every required disclosure field is present | :material-alert-octagon:{ style="color:#c62828" } |
| `load-pattern` | §6.1 | `load_pattern` is `concurrency` with a positive level. §6.1 now says "the benchmark-defined fixed-concurrency load pattern" rather than naming `ConcurrencyScheduler`; the checker still tests the config value | :material-alert:{ style="color:#ef6c00" } |
| `streaming-config` | §6.5, §9.1 | `stream_all_chunks` is `True` | :material-flag:{ style="color:#f9a825" } |
| `point-duration` | §6.2 | Meets its region's minimum steady-state duration | :material-flag:{ style="color:#f9a825" } |
| `min-query-count` | §6.4 | `n_samples_completed` meets the dataset minimum | :material-flag:{ style="color:#f9a825" } |
| `warmup-present` | §6.3.3 | Warmup declaration present | :material-flag:{ style="color:#f9a825" } |
| `warmup-logs-retained` | §6.3.2 | Log retention declared | :material-information:{ style="color:#1565c0" } |
| `warmup-salt` | §6.3.3 | Warns when the warmup salt is enabled | :material-information:{ style="color:#1565c0" } |
| `config-consistency-dataset` | §9.1 | All points use the same dataset | :material-flag:{ style="color:#f9a825" } |

## Seed binding

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `seed-set-consistency` | §9.1 | Every point records the same seed set | :material-alert-octagon:{ style="color:#c62828" } |
| `seed-set-membership` | §9.1 | The bound set is one MLCommons published | :material-alert-octagon:{ style="color:#c62828" } |
| `seed-runtime-match` | §2.1.1 | The RNG seeds equal the bound set's values | :material-alert-octagon:{ style="color:#c62828" } |
| `target-cohort` | §4.6 | `target_cohort` matches `YYYY-MM-C0` / `YYYY-MM-C1` | :material-alert-octagon:{ style="color:#c62828" } |
| `seed-set-adoption` | §4.6 | Set published for the target cohort or the three before it | :material-alert-octagon:{ style="color:#c62828" } |
| `seed-config-legacy` | §4.6 | v0.7 fallback — seeds equal 42 when no `seed_set` is declared | :material-information:{ style="color:#1565c0" } |
| `seed-set-registry` | §4.6 | Warns when the seed-set file cannot be read | :material-information:{ style="color:#1565c0" } |

!!! warning "`seed-set-adoption` reports SKIP, not PASS"
    The checker's bundled seed-set file predates the upstream merge and carries no cohort keys, so
    the adoption test cannot run. Override it with `--seed-sets FILE` or
    `$MLPERF_ENDPOINTS_SEED_SETS` pointing at the published `seedset.yaml`. See
    [step 4](../workflow/run-the-points.md#seeds-and-salting).

!!! note "Amendments are not re-tested for adoption"
    For an amendment, every new or replacement point must match the **original** submission's bound
    seed set. The four-cohort adoption test is not reapplied using the amendment's later cohort.

## Metrics

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `result-file-valid` | §8.3 | `result_summary.json` parses against `PointSummary` | :material-alert-octagon:{ style="color:#c62828" } |
| `metric-consistency-duration` | §9.1 | `duration_ns > 0` | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-accounting` | §9.1 | `completed + failed == issued` | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-output-tokens` | §9.1 | `total_output_tokens ≥ 0` | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-system-tps` | §9.1 | Stored `system_tps` matches the derived value | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-tpot-p90` | §9.1 | Reported TPOT P90 present, finite, strictly positive | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-tps-per-user` | §9.1 | Stored `tps_per_user` matches `1000 / tpot_p90_ms` | :material-flag:{ style="color:#f9a825" } |

## Accuracy

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `accuracy-present` | §6.6, §9.1 | At least one model carries accuracy results | :material-alert-octagon:{ style="color:#c62828" } |
| `accuracy-valid` | §6.6 | `accuracy_results.json` parses correctly | :material-alert-octagon:{ style="color:#c62828" } |
| `accuracy-sample-count` | §6.6 | Issued sample count meets the model's minimum | :material-alert-octagon:{ style="color:#c62828" } |
| `accuracy-gate` | §9.1 | Score meets the benchmark quality target | :material-alert-octagon:{ style="color:#c62828" } |

**Accuracy has no variability allowance** at any stage.

!!! warning "The checker is behind the rules here"
    Since 2026-09 the rules require accuracy results at **every point listed in §5.3** — the four
    mandatory region points, plus an Offline point if applicable — judged per-point for single-turn
    benchmarks and as a mean for multi-turn (§4.3, §6.6, and the §9.1 **Accuracy** row). The
    shipped checker still tests `accuracy-present` as *at least one model carries accuracy
    results*. Passing the checker is therefore no longer proof that you meet the accuracy rule.
    Count your accuracy runs yourself.

## Rules §9.1 rows with no shipped check

§9.1 gained two rows in 2026-09 that the released checker does not yet implement. A clean checker
report does not cover them, and a reviewer can still raise them.

| §9.1 row | What it requires | Status in the checker |
|---|---|---|
| **Accuracy** | Results present at every point §5.3 requires, passing the single-turn or multi-turn gate in §4.3 | `accuracy-present` still tests only *at least one model carries accuracy results* |
| **Agentic metric consistency** | Reported agentic metrics are derivable from their §4 definitions | Not implemented — `e2e_avg_interactivity` is new in §4.1 |

Both are **Reject submission** / **Flag** actions in the rules, so the gap is on the checker's side,
not a relaxation.

## What automation does not check

Manual reviewers focus on what the checker cannot see. These are not rule IDs — they are objection
grounds. See [Why submissions get rejected](../rules/rejection-reasons.md#rejections-that-come-from-judgement-not-checks).

## Clause-numbering note

!!! note "Rule IDs cite some clause numbers that do not exist"
    The checker's own documentation cites §14, §15 and §16 for the metrics, accuracy and consistency
    families. Those sections are not present in the current rules document — the corresponding
    content is in §6.6, §8.5 and §9.1. The clause column above maps to the rules as they actually
    are. Tracked as **B3** in [Open questions](../help/open-questions.md).

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (a7ec3cc) and
`mlcommons/endpoints-submission-cli@main` (f48ca84), 2026-09-19.*
