# 5. Author the disclosure files

> Produces: `system_desc.json` and `point.yaml` in every run folder, plus the shared `src/` and
> `docs/` content.

!!! note "Before you begin"
    - Completed [4. Run the measurement points](run-the-points.md)
    - You have one run folder per point
    - Your disclosure content is cleared for publication

!!! danger "No tool generates these files"
    `system_desc.json` and `point.yaml` are **hand-authored by you** and dropped into each run
    folder before upload. The reference client does not write them. The submission CLI copies
    `point.yaml` into the bundle **exactly as written**. It doesn't derive it from `config.yaml` and does
    not fill in missing fields. Whatever you write is what gets submitted and what gets checked.

## What you'll do

- Write `point.yaml` for every measurement point
- Write `system_desc.json` for every measurement point
- Write the shared `src/<implementation>/README.md`
- Write the shared `docs/` disclosure content

## Where the files go

Each run folder needs both files at its top level:

```
<run-folder>/
├── system_desc.json                  # §8.2 — you author this
├── point.yaml                        # §8.3 — you author this
├── performance/result_summary.json   # written by the client
├── accuracy/accuracy_results.json    # written by the client
├── config.yaml                       # written by the client (optional as of v1.0)
├── src/<implementation>/             # merged into the bundle's shared src/
└── documentation/                    # merged into the bundle's shared docs/
```

## Steps

### 1. Write `point.yaml` for each point

This is the §8.3 disclosure the checker validates. It must declare, at minimum:

| Field | Notes |
|---|---|
| `concurrency` | The target level for this point |
| `region` | Which region it satisfies |
| `runtime_settings` | Load pattern, `min_duration_ms`, `min_sample_count`, `stream_all_chunks` |
| `dataset` / `dataset_name` / `dataset_type` / `dataset_link` | Identity and role of the dataset |
| `warmup` | `duration_s`, `requests_issued`, `requests_completed`, `data_source`, `concurrency`, `initialization_steps` |
| `division` | Standardized, Serviced or RDI |
| `max_supported_concurrency` | Your `C_max` |
| `model_name`, `model_precision`, `link_to_model` | Model identity and lowest weight precision |
| `link_to_model_transformation` | Calibration / quantization write-up, if any |
| `seed_set`, `target_cohort` | The set you bound to, and the cohort you target |
| `shared_src`, `shared_docs` | Must resolve to directories under the submission root |

Full field list: [`point.yaml` reference](../reference/point-yaml.md).

!!! warning "`dataset_type` does real work"
    The bundle builder must know whether a run is an accuracy or a performance run and **will not
    guess**. It reads `datasets[].type` from `config.yaml` first, then falls back to `point.yaml`'s
    `dataset_type`, but only when that value is exactly `Accuracy` or `Performance`. A value of
    `Accuracy + Performance` describes the *dataset*, not the run, and the build fails naming the
    run. The strictness is intentional: defaulting to "performance" used to file accuracy runs in
    the wrong place and quietly drop their results.

### 2. Write `system_desc.json` for each point

The §8.2 hardware and software description. Since policies PR #119 there's no per-system file:
**every Pareto point carries its own copy**, and the checker verifies all points of a curve describe
the same system.

Key fields: `division`, `system_name`, `shortened_system_name` (≤ 20 characters),
`system_availability_status`, node and accelerator topology, `serving_framework`,
`inference_backend`, `driver`, `container_link`, `model_name`, `max_supported_concurrency`,
`endpoint_url`, the parallelism mapping (`tensor_parallel`, `expert_parallel`, `pipeline_parallel`,
`data_parallel`, `disaggregated`), `batch`, `config_summary` and `tps_utilization`.

Full field list and a copyable template: [`system_desc.json` reference](../reference/system-desc-json.md).

!!! tip "`tps_utilization` is computed, not chosen"
    It is `reported_system_tps / max(reported_system_tps across the curve)`. The checker recomputes
    it against your own curve. You cannot fill this in until every point has run.

### 3. Write the shared `src/` content

`src/<implementation>/` (for example `vllm/`, `trtllm/`, `sglang/`) holds the endpoint interface
code, infrastructure and cluster setup, and client harness. **A `README.md` is required** in each
implementation directory, explaining how to build and launch the system under test and reproduce a
point.

This content is shared across the whole submission and written **once**. It isn't duplicated per
Pareto point. Adding or withdrawing a point must not require any change under `src/` or `docs/`.

### 4. Write the shared `docs/` content

- `software_disclosure.md` — serving framework with version and commit or release tag, accelerator
  compute library and build, driver version, operating system.
- `calibration.adoc` — required if you applied any weight transformation. Either describe the recipe
  in enough detail for an external team to reproduce it, or provide the scripts that implement it.
- Anything else a reviewer needs to follow your setup.

!!! warning "Disclosure obligations differ by division"
    Standardized requires full hardware, software and parallelism disclosure. Serviced requires the
    advertised model name and version, endpoint URL, pricing model and rates, and rate limits, but
    but full rack hardware disclosure is optional. See
    [Requirements you must meet](../rules/requirements.md).

## Verify

The fastest check is the checker itself, which is [step 6](validate.md). Before that, confirm
mechanically that nothing is missing:

```bash
for d in run-folders/*/; do
  for f in system_desc.json point.yaml; do
    [ -f "$d$f" ] || echo "MISSING: $d$f"
  done
done
```

Then confirm your `shared_src` and `shared_docs` values name directories that will exist under the
assembled submission root. A point whose pointers don't resolve is incomplete and the submission
is rejected.

## Next

→ [6. Validate locally](validate.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#disclosure-files).
