# 4. Run the measurement points

> Produces: one run folder per Pareto point, with accuracy results at the required points.

!!! note "Before you begin"
    - Completed [3. Plan your Pareto curve](plan-your-curve.md)
    - You have a list of legal concurrency levels
    - The endpoint under test is up and reachable
    - You know which seed set you are binding to — see [Seeds](#seeds-and-salting) below

## What you'll do

- Probe the endpoint
- Write one YAML config per measurement point
- Run each point at **fixed concurrency** for its region's minimum duration
- Run accuracy validations at the four mandatory region points
- Keep every run folder

This is the expensive step. Everything up to now took minutes. This takes days.

## The binding run constraints

| Constraint | Requirement |
|---|---|
| Load pattern | **`concurrency`** only. `max_throughput` and `poisson` are **not valid** for Pareto points |
| Steady-state duration | **600 s** in Ultra Low Concurrency; **1,200 s** in Low, Medium and High — measured over the steady-state window's issue time, not wall clock |
| Completed queries | At least one full pass over the dataset, and the total samples issued must be a whole-number multiple of the dataset size |
| Streaming | `stream_all_chunks: true` for every performance run |
| Sampling | Performance runs sample **with** replacement; accuracy runs **without** |
| Warmup | Optional, max 24 h per point, fully excluded from metrics, and fully documented |
| Consistency | Same model, endpoint config, software stack and seed set across **every** point |

!!! tip "Run longer than the minimum"
    Your official numbers now come from a **steady-state window** the tooling detects inside the
    run, not from the whole run. A run that only just clears its minimum can fail to produce a
    usable window, and then the published number falls back to the whole-run average — ramp-up and
    drain included, which understates what your system does. See
    [What your numbers are measured over](../reference/metrics-and-regions.md#what-your-numbers-are-measured-over).

!!! danger "Configuration consistency is checked across the whole curve"
    Every point must describe the same system, the same model and the same dataset. A curve
    assembled from points run against two different software versions will be flagged and can be
    rejected. Lock your software stack before the first point and don't change it until the last
    one is done.

## Steps

### 1. Probe the endpoint

```bash
uv run inference-endpoint probe \
  --endpoints http://your-endpoint:8000 \
  --model <your-model>
```

### 2. Generate and adapt a config template

```bash
uv run inference-endpoint init concurrency
```

A minimal concurrency config looks like this:

```yaml
name: point-c64
type: online
model_params:
  name: "<canonical model id>"
datasets:
  - name: perf
    type: performance
    path: "<performance dataset path>"
settings:
  runtime:
    scheduler_random_seed: <from your bound seed set>
    dataloader_random_seed: <from your bound seed set>
  load_pattern:
    type: concurrency        # the ONLY valid pattern for a Pareto point
    target_concurrency: 64
  timeouts:
    endpoint_response_idle_timeout_s: 300
endpoint_config:
  endpoints:
    - "http://your-endpoint:8000"
```

Validate the config before running it:

```bash
uv run inference-endpoint validate-yaml -c point-c64.yaml
```

### 3. Run each point

```bash
uv run inference-endpoint benchmark from-config --config point-c64.yaml
```

!!! tip "`from-config` has a narrow CLI surface"
    It accepts only `--config`, `--timeout` and `--mode`. There's no `--report-dir` override, so set
    `report_dir` in the YAML if you need to control where artifacts land.

Repeat for every concurrency level in your plan. Each writes its own run folder.

### 4. Run the accuracy validations

You need accuracy results at the four mandatory region points — Ultra Low, Low, Medium and High
Concurrency — plus one more if you submit Offline results. All of them use the **same** endpoint
configuration, model weights and software stack as the performance runs.

For a **single-turn** benchmark, each accuracy run goes at the same concurrency as its point, on
the same instance, **immediately after** that point's performance run. Don't batch them up at the
end: the rule ties each accuracy run to the performance run it follows. Every result has to pass.

For a **multi-turn** benchmark, only the average of the results has to pass, and the concurrency
can differ — these runs are expensive, so the rules give you room here.

The simplest way to get the ordering right is `--mode both` on each of those points' configs, so
one invocation writes the performance run and then the accuracy run into the same report
directory, with `accuracy/accuracy_results.json` alongside `performance/result_summary.json`.

!!! danger "Accuracy has no tolerance"
    Throughput results get reproducibility margins. Accuracy doesn't. It's a hard gate at automated
    compliance and throughout review. Miss the quality target and the submission is rejected.

### 5. Keep everything

Each run folder contains:

```
<report_dir>/
├── config.yaml                       # resolved config as run (secrets redacted)
├── report.txt                        # human-readable summary
├── events.jsonl                      # per-event log (the large one)
├── sample_idx_map.json
├── performance/result_summary.json   # written when the performance phase ran
├── accuracy/accuracy_results.json    # written when the accuracy phase ran
└── metrics/final_snapshot.json
```

Full detail in [Submission package layout](../reference/package-layout.md).

!!! warning "Warmup logs must be retained"
    All requests issued before `TEST_STARTED` are warmup and must not appear in any reported
    metric, but their logs must be **retained and available for reviewer inspection**. Reviewers
    may cross-check them against the performance dataset.

## Seeds and salting

Your submission binds to exactly **one seed set**, chosen when it first appears, and every point
must record that same set. The seeds drive the client's request-issue / sample-order RNG and the
per-query salt. Once bound, the set stays valid for that submission even after newer sets are
published. Seed rotation never forces an in-flight submission to be re-run.

The **salt** is a per-query unique value inserted between the shared system prompt and the per-query
user context. It is what makes blanket cross-query KV-cache reuse legal: the only prefix two queries
can share is the system prompt itself. Accuracy runs use the **un-salted** dataset so that model
output matches the canonical implementation exactly.

!!! warning "Warmup must not use performance-dataset samples"
    Warmup requests must not use any sample from the benchmark performance dataset — directly, as a
    subset, truncated, or derived. The accuracy dataset and other sources are fine. If your client
    does use the performance dataset during warmup, **salting must be enabled**, and note that the
    salting flag is **not on by default**.

!!! success "The v1.0 seed set is published"
    As of 2026-09-15 the policies repo carries `seedset.yaml`: one set, **`id: A`**, published for
    cohort **`2026-10-C1`**. Its three values are identical to the set the checker already ships,
    so a submission binding set `A` is using the right numbers.

    ```yaml
    seed_set: A
    target_cohort: 2026-10-C1
    ```

!!! warning "The checker's copy is stale, so adoption still reports SKIP"
    The checker's mirror carries `cohorts: []` and a comment describing the upstream PR as open —
    it predates the merge. With no cohort keys, `seed-set-adoption` reports **SKIP** instead of
    passing, so a clean report is not confirmation that your cohort is in the adoption window.

    Point the checker at the published file to get a real result:

    ```bash
    export MLPERF_ENDPOINTS_SEED_SETS=/path/to/seedset.yaml
    # or: submission-checker ... --seed-sets /path/to/seedset.yaml
    ```

    Note the two files are shaped differently — the published one nests under a `cohort:` key while
    the checker's is a bare `seed_sets:` list. If the override is rejected, reshape it. Tracked as
    **B4** in [Open questions](../help/open-questions.md).

## Verify

For each run folder, confirm the phase directory you expected exists and the run completed:

```bash
ls <report_dir>/performance/result_summary.json
python -c "import json;d=json.load(open('<report_dir>/performance/result_summary.json'));print(d['complete'], d['n_samples_completed'], d['duration_ns']/1e9)"
```

Check three things:

- `complete` is `true` — a `false` means the run drained out or was interrupted, and the point is
  not usable
- `duration_ns` meets your region's minimum in seconds
- `n_samples_completed` represents at least one dataset pass

Also confirm the run used the right pattern. `run_config` in `result_summary.json` should show the
concurrency load pattern at your target level.

## Next

→ [5. Author the disclosure files](author-disclosures.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#running-the-benchmark).
