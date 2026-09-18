# Before you begin

Work through this list top to bottom. If you can't tick everything in the first section, stop here
rather than after step 4, which is where the accelerator time gets spent.

## You can't submit without these

- [ ] The **individual** who will submit has signed the MLCommons CLA
- [ ] A **PRISM API token** in `mlc_…` format, scoped to *MLPerf Endpoints* — see [step 1](register.md)
- [ ] You have chosen a **division**: Standardized, Serviced or RDI — see [Divisions and scenarios](../understand/divisions-and-scenarios.md)
- [ ] You have chosen a **scenario**: CoP or CoN
- [ ] Your benchmark **model is on the round's supported list** — see the caveat below
- [ ] You can reach the endpoint under test from wherever the client will run
- [ ] Python **3.12+** for the reference client; Python 3.10+ for the submission CLI
- [ ] [`gh` CLI](https://cli.github.com/) installed and authenticated

## What this actually costs

- [ ] **Accelerator time.** Minimum 7 points: one at 600 s of steady state, six at 1,200 s, plus
      warmup — and **at least four accuracy runs**, one at each mandatory region point. You need
      exclusive access to the system for all of it, and a point that fails has to be re-run.
- [ ] **Disk.** `events.jsonl` is the bulk of each run folder — around 46 MB for a 60-second,
      16-concurrency run, scaling with sample count. A 600-second point reaches several hundred MB.
- [ ] **Upload bandwidth.** Every run folder is archived and uploaded before the submission is
      assembled.
- [ ] **Someone available for six weeks.** Once review starts, you have **3 business days** to
      respond to any objection. After 10 business days of no response your submission is withdrawn.
      This is an easy way to lose a submission after all the hardware time is already spent.

## Decisions to make now, not later

- [ ] **`C_max`** — your maximum supported concurrency. It sets your region boundaries and therefore
      which concurrency levels are legal. Getting it wrong means re-running. [Step 3](plan-your-curve.md).
- [ ] **Publication status** — Available, Preview or RDI. Preview commits you to achieving
      availability within **180 days** and re-submitting, or the result is invalidated. See
      [Publication status](../rules/publication-status.md).
- [ ] **Provisional publication?** Publishing before review completes, with a "peer review pending"
      tag. You **can't change this after submitting**, so decide now.
- [ ] **Embargo date?** Up to 60 days after review completes for confidential submissions.
- [ ] **Who signs off on disclosure.** Standardized requires publishing your configuration, launch
      scripts and integration code. Get that cleared internally before you run, not after.

## Read these before running anything

- [ ] [Requirements you must meet](../rules/requirements.md) — the binding constraints
- [ ] [Model equivalence](../rules/model-equivalence.md) — if you are submitting Standardized, this
      governs every optimisation decision you are about to make
- [ ] [Why submissions get rejected](../rules/rejection-reasons.md) — the failure modes
- [ ] [Open questions and WIP rules](../help/open-questions.md) — what could still move under you

--8<-- "draft-rules-warning.md"

## Verify

You are ready to start when this command prints a table rather than an authentication error:

```bash
endpoints-submission-cli runs list
```

An empty table is correct, since you have no runs yet. An error means your token isn't set;
go to [step 1](register.md).

## Next

→ [1. Register and get a token](register.md)
