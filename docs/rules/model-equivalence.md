# Model equivalence (Standardized division)

If you are submitting Standardized, this page governs every optimisation decision you are about to
make. If you are submitting Serviced or RDI, augmentation is allowed with disclosure and this page
is background only.

--8<-- "precedence-notice.md"

!!! warning "Check your assumptions here"
    Endpoints **inherits** the MLPerf Inference model-equivalence rules, then deliberately diverges
    in three places. If you've submitted to MLPerf Inference before, your assumptions about
    cross-query KV reuse and dynamic approximate sparsity. Where the two conflict, the Endpoints
    rules are the source of truth for Endpoints submissions.

## The framing: disallowed-only

Endpoints states optimisation rules as a **single disallowed list**, not a list of permitted
techniques. Anything not listed as banned, and not in conflict with the model-equivalence rules, is
permitted.

The reason: submitters constantly ask "is X allowed?" about techniques that don't exist yet, like
new quantization formats or attention implementations. A fixed list of approved techniques would
need a rule change every time one appeared.

**Practical consequence:** you do not need permission for a technique that is not banned. You *do*
need to satisfy the accuracy gate and the disclosure requirements.

## Parameters the benchmark fixes

The reference implementation designates some configuration parameters as **fixed**. You must not
change those. This covers the sampling parameters — temperature, top-k, top-p, repetition penalty,
greedy-vs-stochastic, max output tokens, stop sequences — and anything else the benchmark
definition marks as fixed.

This was made explicit in 2026-09. It doesn't narrow what was already allowed; it closes the reading
that a parameter not otherwise mentioned in the rules was yours to tune.

## Always disallowed

- Wholesale weight replacement or supplements
- Pruning — discarding non-zero weight elements — **except** where mathematically equivalent to the
  dense reference
- Knowledge distillation to a different architecture
- Retraining, fine-tuning, LoRA, adapter layers, RLHF, or any gradient-based weight update, applied
  either to the canonical model **or** to any speculative-decoding drafter
- **Response caching**: returning a cached response verbatim to a matching request, bypassing the
  forward pass. Every request must execute the forward pass
- Coalescing identical queries to amortise work across them
- Modifying weights during the timed portion of a run (online learning)
- Benchmark detection — behaving differently when a benchmark workload is recognised
- **Input-based optimization**: encoding anything about the benchmark input dataset's *content* into
  the implementation: kernels, calibration outputs, embedding tables, compile-time constants
- Client-side dispatch manipulation — delaying, batching or reordering query dispatch to game TTFT
  or TPS/User
- Modifying the request/response stream outside the reference API specification
- Weight-quantization algorithms whose specification is similar in size to the weights they produce
- Hard-coding the total query count, or techniques that only work for fixed-length experiments

## Weights

All Standardized submissions start from the **canonical model weights** named in the benchmark
definition.

### Permitted: post-training quantization

PTQ is the canonical permitted transformation. AWQ, GPTQ, bitsandbytes, and arbitrary numerical
formats (INT8/INT4/FP8 and similar) are permitted provided all four hold:

1. Only the **published calibration set** is used
2. The method is publicly described well enough to be reproduced
3. It passes the accuracy gate
4. It is disclosed in the submission YAML

You may either derive a quantized checkpoint from the reference checkpoint using only the
MLPerf-provided calibration data, or use a publicly available checkpoint pre-approved by the Rules
Task Force or working group.

### The checkpoint must stay complete

!!! danger "You may not strip components from the derived checkpoint"
    A derived checkpoint must contain **every component the reference checkpoint ships**, including
    auxiliary prediction heads and speculative-decoding modules such as MTP or EAGLE-style heads.
    This holds even where the component does not participate in producing output, and even where you
    do not intend to use it.

    This is about provenance: reviewers check your derived checkpoint against the published model
    ID or checksum, and may want to re-run with the component enabled.

Declining to **use** a component at run time is a separate and permitted matter. Removing it from
the artifact is not.

!!! question "Residency is undecided"
    Whether a component present in the checkpoint must also be **resident in accelerator memory**
    during measurement is an open working-group item. The memory freed by not loading it converts
    directly into KV-cache capacity and therefore throughput, so this is a real and currently
    undisclosed advantage. Tracked upstream as `[CKPT-RESIDENCY]` and here as **B-open** in
    [Open questions](../help/open-questions.md).

## Speculative decoding

Permitted for any benchmark whose definition **designates a drafter**. If the benchmark does not
designate one, speculative decoding isn't available for that benchmark at all. A drafter shipped
with the model but not designated may not be used.

The drafter is **frozen**. Disallowed:

- Fine-tuning, LoRA, adapters, RLHF, or any gradient update to the drafter
- Continued pre-training or retraining of the drafter
- **Swapping** the drafter, including a different checkpoint of the same family, or a smaller one
- Replacing the algorithm with a different one (e.g. EAGLE for Medusa)
- Approximate methods that alter the output distribution — outputs must be token-for-token identical
  to what the target would generate without speculation
- Approximating, skipping or replacing the verification step, including verifying with a secondary
  drafter

**Permitted:** PTQ on the drafter weights, under the same four conditions as the main model.

**Across the curve:** all points must use the **same drafter** — same head, same algorithm. Different
*configurations* of that drafter are fine across points, including disabling speculation entirely at
some points. Declare the per-point configuration in `point.yaml`; report any dynamic variation
within a point as a distribution.

## KV cache: the big difference

!!! success "Cross-query KV reuse is permitted in Endpoints"
    This intentionally diverges from MLPerf Inference, which prohibits it. Endpoints targets
    agentic-style workloads where a shared system prompt across queries is normal, and prohibiting
    reuse would force submitters to cripple production serving stacks.

Sharing KV state across independent requests, including prefix and prompt caching of the shared
system prompt, is allowed as a serving optimisation. No bit-for-bit output identity against an
un-cached run is required, and no cross-user partitioning is required.

**The salt is what makes this sound.** The performance dataset injects a per-query salt between the
shared system prompt and the per-query user context. The only prefix two queries can share is
therefore the system prompt itself; KV state derived from user context cannot be reused across
queries with different contexts.

Salt requirements: ≥ 64 bits of entropy per query, generated from a seeded pseudo-random sequence
whose seed is declared in the run configuration, inserted **between** system prompt and user
context, and generated at request-construction time. **Never store it in the dataset on disk**, or it
would lose uniqueness across replays.

!!! danger "Pre-tokenizing clients must salt the token stream"
    If your client sends `input_tokens` rather than text, the **token stream the system under test
    receives** must contain the salt between system-prompt tokens and user-context tokens. Applying
    the salt to a `prompt` text field while sending the original `input_tokens` is **non-compliant**
    because the system never sees the text. Either salt the text and re-tokenize, or reserve a salt-marker
    token ID and emit it inline.

**Accuracy runs use the un-salted dataset**, so model output matches the canonical implementation
exactly. You are not required to disable KV reuse for accuracy runs.

!!! note "Agentic benchmarks salt differently"
    They use their own benchmark-specific random salting to control KV reuse, switched on by
    benchmark-specific flags rather than by the mechanism above. Those flags **must be enabled** for
    a submission. Which flags, for which benchmarks, is not yet published — see
    [Open questions](../help/open-questions.md).

Still disallowed: response caching that bypasses the forward pass, and KV-cache compression methods
that are neither in the reference implementation nor disclosed. Reduced-precision KV (INT8/INT4/FP8)
must have its precision and quantization method disclosed. Paged/virtual KV implementations such as
PagedAttention need no disclosure beyond the software-stack listing.

!!! warning "Not portable to MLPerf Inference"
    Endpoints submissions relying on this delta are **not** portable to standard MLPerf Inference
    without disabling cross-query KV reuse. Treat the two rule sets as incompatible for code paths
    that depend on it.

## Sparsity and approximate computation

The operative test is whether a technique **changes the tokens the model emits for any input**.

### Permitted with no disclosure

Mathematically equivalent to the reference:

- Sparse operations producing the same outputs as the dense original; skipping exactly-zero blocks
- Fused, streaming and online softmax — FlashAttention-style running max/sum, log-sum-exp
  rearrangement, max subtraction
- **Vocabulary softmax elision under greedy decoding** — `argmax(softmax(x)) == argmax(x)`
- Softmax elision in speculative verification where the target is greedy

Bit-exact identity is not required; ordinary floating-point reassociation is expected.

### Required where the architecture is itself sparse

Where the canonical architecture specifies sparse attention, **implementing that sparsity is
required for equivalence**. The reference point is the canonical model's own computation, not a
dense idealisation. Likewise, an MoE model touching only routed weights per token *is* the canonical
computation; that most expert weights go untouched is not grounds for objection.

### Permitted under the accuracy gate: the second difference

Sparsity derived **at run time** from live attention scores or activations is permitted even where
it does not preserve outputs exactly. This is Endpoints-specific.

Covers attention-score thresholding (skipping a key/value block whose local max score falls far
below the running maximum) and activation thresholding.

Conditions, **all** of which apply:

- Must pass the accuracy quality target on the un-salted accuracy dataset
- Must **not** be calibrated on the benchmark performance or accuracy dataset — use the published
  calibration set or a data-independent procedure. Calibrating on benchmark inputs is input-based
  optimization and is banned
- Must be disclosed: method and source, threshold parameter, calibration procedure, per-point value,
  and a schedule or distribution where it varies

### Disallowed

- **Structural attention-pattern changes** — sink tokens not in the canonical architecture, a
  different attention mask, a fixed or precomputed sparsity pattern imposed on a canonically dense
  layer, or substituting a different structural sparse pattern
- **Normalization substitution** — replacing softmax with linear or ReLU attention
- **Weight-side sparsity to reach a hardware sparse path** — imposing 2:4 or block-structured
  patterns on weights. Dense weights *may* be dispatched through a sparse kernel; the weights
  themselves must stay dense
- **Static removal of weights** — dropping experts, layers, or anything that participates in the
  forward pass for some inputs, even rarely-routed experts, **whether or not accuracy passes**
- Softmax elision where the reference sampling configuration is stochastic, or where the response
  returns logprobs or probabilities

Neither the run-time sparsity ratio nor the hardware's sparsity granularity is itself a compliance
criterion. Whether a technique runs in software or on sparse tensor cores is not a criterion either.

## Pre- and post-processing

**Input:** tokenization must produce the same token IDs as the reference tokenizer. An alternative
tokenizer implementation requires demonstrating token-for-token equivalence on the accuracy dataset.
The chat template, system prompt, turn formatting and special tokens must match the benchmark
specification. Truncation must produce the same result.

!!! warning "Chat-template flags count too"
    Since 2026-09 the rule names the template's **flags** explicitly — `enable_thinking`,
    `clear_thinking`, `preserve_thinking` and their equivalents. These have to match the benchmark
    specification. Flipping a thinking flag changes the effective input to the model, so it is a
    template modification, not a serving-side setting.

**Output:** detokenize with the reference detokenizer. Halt on the same stop tokens and EOS
conditions. Match the benchmark's sampling configuration exactly: greedy means greedy. With
`stream_all_chunks = true`, every output token must be dispatched as generated; **buffering is not
permitted**.

## The accuracy gate is necessary, not sufficient

You have to pass the accuracy quality target to be model equivalent, and it's what decides whether
an approximation the rules allow is acceptable in your particular submission.

!!! danger "It is not a general override"
    A submission that hits the quality target while breaking one of the rules above (pruning
    weights, removing experts, fine-tuning a drafter) **is not model equivalent**. Accuracy doesn't buy
    forgiveness for a banned technique.

--8<-- "draft-rules-warning.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (a7ec3cc), 2026-09-19.*
