# What is MLPerf Endpoints?

MLPerf Endpoints measures how well an **inference endpoint serves generative AI models**. A
separate client sends load to your model-serving API over HTTP. What gets tested is the endpoint
itself, not a framework integration.

## It measures a curve, not a single number

MLPerf Inference reports latency or throughput at one operating point. Real serving systems don't
have one operating point — they have a tradeoff between total throughput and per-user speed, and
you choose where to sit on it. Endpoints measures that whole tradeoff as a Pareto curve across
concurrency levels.

Four dimensions are captured at each measurement point:

| Metric | Field | What it is |
|---|---|---|
| System throughput | `system_tps` | Total output tokens per second across all concurrent users |
| Interactivity | `tps_per_user` | Per-user output rate, derived as `1000 / tpot_p90_ms` |
| First-token latency | `ttft_p90_ms` | P90 milliseconds from query issue to the first non-empty text fragment |
| Load | `concurrency` | Target number of in-flight queries at that point |

The main published chart plots `system_tps` against `tps_per_user`: total capacity against
per-user experience. Full definitions in [Metrics and regions](../reference/metrics-and-regions.md).

!!! warning "P90, not P95"
    v1.0 requires `ttft_p90_ms`. The v0.7 rules used P95, and some public MLCommons material still
    shows P95. Only P90 is plotted in the v1.0 publication chart. You may report additional
    percentiles in your `point.yaml`; they will not be charted.

## Why a step function

The official curve is a **step function**. Each submitted point defines a discrete step; between
points the curve holds at the last measured value. No interpolation, no curve fitting, no
smoothing.

A smooth line would suggest operating points you never actually measured, and the gaps between
points are where problems like memory pressure and latency spikes tend to show up. Tools can overlay
a smoothed curve for readability, but it has to be labelled *interpolated (not official)* and can't
replace the step function.

## What a submission actually is

One submission is one **Pareto curve**: one system, one benchmark model, one dataset. It contains
a **minimum of 7 and a maximum of 32** measurement points, structured `1 + 3 + 3`:

- **1** point in the Ultra Low Concurrency region (concurrency 1–32),
- **1 each** in the Low, Medium and High Concurrency regions,
- **3** anywhere in those three regions, at your discretion.

Region boundaries differ between submitters. They're calculated from your own minimum and maximum
concurrency, which is covered in [Plan your Pareto curve](../workflow/plan-your-curve.md). Pick your
maximum badly and you'll have to re-run points.

Each point is a sustained run: **600 seconds** of steady state at ultra-low concurrency, **1,200
seconds** elsewhere, not counting warmup.

## How tokens are counted

Token counts don't come from your serving stack. The reference client rebuilds the full response —
visible output, reasoning traces and tool calls — formats it with the model's official chat
template, and counts it **once** using the reference tokenizer from the model's Hugging Face
repository.

This means every submitter is measured the same way, no matter how their system batches or streams
output. It also means your published `system_tps` may differ from the number your serving framework
reports, which is expected. You can use any tokenizer internally; it doesn't affect your score.

## Accuracy is a hard gate

Each benchmark has a quality target, and you have to hit it at several points on the curve — not
once for the whole submission. Accuracy runs are required at the four mandatory region points
(Ultra Low, Low, Medium and High Concurrency), plus one more if you submit Offline results. Every
accuracy run uses the same endpoint configuration, weights and software stack as your performance
runs.

How those runs are judged depends on the benchmark:

| Benchmark type | The gate |
|---|---|
| Single-turn | **Every** result must meet the target. Each run uses the same concurrency as its point, on the same instance, immediately after that point's performance run |
| Multi-turn | The **average** of the results must meet the target. Individual results may fall short, and the concurrency may differ |

Throughput results are allowed some variation between runs. Accuracy isn't: miss the target and
the submission is rejected.

!!! warning "This changed recently"
    Until 2026-09, the rules asked for one accuracy run per submission. If you planned your
    hardware time against that, re-plan — it's now at least four.

**Next:** [How submission works](how-submission-works.md)

--8<-- "precedence-notice.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (a7ec3cc), 2026-09-19.*
