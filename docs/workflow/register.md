# 1. Register and get a token

> Produces: a PRISM API key in `mlc_…` format, scoped to MLPerf Endpoints.

!!! note "Before you begin"
    - You have read [Membership, PRISM and eligibility](../understand/eligibility.md)
    - You have an organisation email address
    - You know whether your organisation needs MLCommons membership — see the warning below

## What you'll do

- Sign the MLCommons CLA as the individual who will submit
- Create a PRISM account at MLCommons Member Central
- Mint an API key with Service Scope **MLPerf Endpoints**
- Put the key in your environment

!!! warning "Start this first"
    Every other step is work you control. This one waits on MLCommons granting API-creation access
    and emailing you. No turnaround time is published. Do it before you schedule accelerator time,
    not after.

## Steps

### 1. Sign the CLA

The **individual making the submission** must have signed the relevant MLCommons CLA. It is not
required that everyone in your organisation has signed. All code you submit is made under the CLA
and will be Apache-2-compatible.

### 2. Create your PRISM account

Create your account at **MLCommons Member Central** using your **organisation email address**, not
a personal one. Once the account exists you are granted API-creation access and notified by email.

!!! question "URL not published"
    The Member Central and PRISM URLs do not appear in any rules document, CLI documentation, or
    public MLCommons page. Ask via the [support channels](../help/support.md). Tracked as **A3** in
    [Open questions](../help/open-questions.md).

### 3. Create the API key

Sign in, open the **API Keys** dashboard, and create a key with **Service Scope: MLPerf Endpoints**.
The key looks like `mlc_…`.

### 4. Put the key in your environment

=== "Shell profile (recommended)"

    ```bash
    # Add to ~/.zshrc, ~/.bashrc or equivalent
    export PRISM_USER_API_TOKEN=mlc_your_token_here
    ```

=== "Per command"

    ```bash
    endpoints-submission-cli runs list --token mlc_your_token_here
    ```

Both forms work on every command. `--token` wins when both are set.

!!! danger "Treat the token as a credential"
    It authenticates every action against your organisation's submissions, including withdrawal. Do
    not commit it, do not paste it into an issue, and note that report directories written by the
    reference client contain a **sanitized** `config.yaml` with secrets replaced by `<redacted>`,
    which means a config copied back out of a report directory will not work until you restore them.

## Verify

```bash
endpoints-submission-cli runs list
```

- **A table (possibly empty)** — authentication works. Proceed.
- **An authentication error** — the token is missing, mistyped, or not scoped to MLPerf Endpoints.
- **A connection error** — check `MLPERF_API_BASE_URL` is unset; it defaults to
  `https://api.mlcommons.org` and should only be overridden for dev or staging.

## Next

→ [2. Install the tools](install-tools.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#authentication).
