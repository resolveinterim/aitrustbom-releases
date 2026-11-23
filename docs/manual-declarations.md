# Complete `trust/manual.yml` example

For the other editable TrustBOM YAML files and their ownership, see the
[YAML input schema reference](yaml-input-reference.md).

`trust/manual.yml` records facts that repository detection cannot establish:
deployed models, tool capabilities, data classes, controls, disclosure copy
and provider data handling. The scanner combines these declarations with
detected evidence into `trust/card.json`. Manual values take precedence over
matching inferred values; they do not approve the resulting graph.

The example below includes **every currently supported top-level field**
and every declaration field. It describes a **fictional support assistant**.
Its deployment regions, retention periods, training policies and controls
are synthetic test facts, **not claims about the named providers**. Replace
them with facts Trust has reviewed for your actual deployment and contract.
Delete declarations you cannot support rather than copying reassuring values.

## Example file

Save this block as `trust/manual.yml` in the repository being scanned.
Paths inside YAML use repository-relative forward slashes, including on Windows.
Do not include credentials, customer messages or private prompt bodies.

```yaml
not_in_scope:
  - path: experiments/**
    reason: Disposable research prototypes are not shipped with the support assistant.
  - path: test/fixtures/**
    reason: Synthetic SDK and model examples are test inputs, not deployed integrations.

models:
  - provider: anthropic
    model_id: claude-sonnet-4-5
    region: eu-central-1
    endpoint: https://api.anthropic.com
    reason: Synthetic primary deployment; Trust reviewed the runtime configuration.
  - provider: openai
    model_id: gpt-5
    region: eu-west-1
    endpoint: https://api.openai.com/v1
    reason: Synthetic fallback deployment; Trust reviewed the separate runtime configuration.

tools:
  - name: search_help_center
    effect: read
    reason: Searches the approved help-center index without changing its contents.
  - name: save_reply_draft
    effect: write
    reason: Writes a draft to the support case but does not send it to the customer.
  - name: send_reply
    effect: send
    reason: Sends a customer-facing reply only after the support agent confirms it.
  - name: partner_connector
    effect: unknown
    reason: Connector is in scope but its capabilities still need Trust review.

data_classes:
  - class: support_message
    to_model: true
    reason: Redacted support-message text is included in the inference request.
  - class: help_center_content
    to_model: true
    reason: Relevant help-center passages are supplied as retrieval context.
  - class: billing_details
    to_model: false
    reason: Billing details remain in the case system and are excluded before inference.

prompt_bodies_stored:
  value: false
  reason: Application logs contain request identifiers and timing, not prompt bodies.

human_send_required:
  value: true
  reason: A support agent must review the draft and explicitly confirm sending.

kill_switch:
  value: true
  reason: The AI_DRAFTING_ENABLED feature flag disables AI drafting and model calls.

article_50_copy_path:
  path: docs/ai-disclosure.md
  reason: Trust maintains the customer-facing disclosure text at this tracked path.

data_handling:
  - provider: anthropic
    retention: 30
    training_use: none
    deletion_path: trust/deletion/anthropic.md
    evidence_path: trust/terms/anthropic.md
    reason: Synthetic enterprise terms and deletion procedure reviewed by Trust.
  - provider: openai
    retention: unknown
    training_use: unknown
    deletion_path: trust/deletion/openai.md
    evidence_path: trust/terms/openai.md
    reason: Synthetic review is incomplete; the evidence records the outstanding questions.
```

`unknown` is intentional in this example: a fully populated file does not
mean every fact is known or the graph is ready for approval. Tool effects
describe capabilities, not permission to use them without review.

## Referenced files

Create and track these files before scanning the example:

```text
docs/ai-disclosure.md
trust/deletion/anthropic.md
trust/deletion/openai.md
trust/terms/anthropic.md
trust/terms/openai.md
```

For a disposable test, each can contain a clearly labelled synthetic
disclosure, review note or deletion procedure. For a real system, use
reviewed evidence you are permitted to keep in the repository. Do not put
confidential contracts into a public test repository. AI TrustBOM does not
fetch provider terms or verify that a contract establishes the declared facts.

`data_handling.evidence_path` and `deletion_path` are optional, but when
provided must reference Git-tracked repository-relative files. In a portfolio,
these paths must lie inside the declaring system's own path (for example
`apps/assist/trust/evidence.md`); a root-level file outside it is rejected
with `manual-invalid`. The graph
cites the manual declaration and these referenced paths. An Article 50 copy
path identifies disclosure text; it does **not** decide whether Article 50
applies.

## Field rules and alternatives

| Field | Supported values and meaning |
|---|---|
| `not_in_scope` | List of `{ path, reason }` entries. `path` is a supported repository-relative glob. Do not use a list of bare strings. Exclusions narrow detection; they are not a waiver or a way to hide shipped AI usage. |
| `models` | Each row requires `provider`, `model_id` and `reason`; `region` and `endpoint` are optional. Use the graph's provider identifier, such as `anthropic`, `openai`, `bedrock`, `vertex` or `azure-openai`, and the actual deployed model ID. Endpoint, when supplied, must be an absolute HTTP(S) URL, not a secret. An endpoint alone does not prove a deployment region. |
| `tools` | Each row requires `name`, `effect` and `reason`. Effect is exactly `unknown`, `read`, `write` or `send`. Name the same tool consistently so manual and inferred entries can be matched. |
| `data_classes` | Each row requires `class`, boolean `to_model` and `reason`. Class labels are descriptive strings, not a fixed taxonomy. `false` is an explicit declaration that this class is not sent to a model; omitting a class does not establish that fact. Rows do not select a particular model. |
| `prompt_bodies_stored` | `{ value: true/false, reason: ... }`. Declare application prompt-body storage; this is not a substitute for the provider-retention declaration. Omit the block if you cannot establish the fact. |
| `human_send_required` | `{ value: true/false, reason: ... }`. Whether a human must confirm sending. The scanner cannot verify your user interface or enforcement. |
| `kill_switch` | `{ value: true/false, reason: ... }`. Whether a disabling control exists. The declaration neither creates nor operates that control. |
| `article_50_copy_path` | `{ path: <repository-relative-file>, reason: ... }`. Points to disclosure copy, not a legal classification or approval. |
| `data_handling` | One row per provider with a model in the final graph. Requires `provider`, `retention`, `training_use` and `reason`; optional `deletion_path` and `evidence_path`. Provider matching is case-insensitive; use lower-case identifiers consistently. |
| `data_handling.retention` | `none`, `indefinite`, `unknown`, or an integer number of days from **1 to 36500**. `none` declares no retention; `unknown` means it has not been established. Do not use `0`, `"30 days"`, negative days or `null`. |
| `data_handling.training_use` | `none`, `opt_in`, `"yes"` or `unknown`. Quote `"yes"` to avoid YAML boolean interpretation. These are provider-training declarations, not assertions about your own company's training. |
| `reason` | Nonblank explanation required on every declaration, including exclusions, controls and paths. Reasons justify manual input; they are not automatically published verbatim as graph facts. |

All top-level sections are optional; collections may be `[]`. To state an
unknown control, omit its block rather than writing `value: unknown` or
inventing a boolean. A missing provider data-handling row renders as
unknown and raises a readiness gap; an explicit row with unknown values is
still a declaration included in the graph hash.

Do not repeat a model's `(provider, model_id)`, a tool name, a
`(class, to_model)` pair, an exclusion path or a data-handling provider.
Distinct models for the same provider share that provider's data-handling
row. YAML must be UTF-8 and no larger than 256 KiB; unknown keys, duplicate
keys, anchors, aliases and explicit YAML tags are rejected.

## What belongs elsewhere

| Separate file | Purpose |
|---|---|
| `trust/system.yml` | System name, description, customer-facing name and optional disclosure-copy path. |
| `trust/subprocessors.yml` | Provider/processor legal-party details, purpose, location and tracked DPA paths. A model declaration does not satisfy missing subprocessor evidence. A row whose provider matches no detected or declared model is excluded from the graph and questionnaire without a finding, so check provider spelling. |
| `trust/evals.yml` | Names and artifact paths for CI evaluation results. `manual.yml` does not run or fabricate evaluations. |
| `trust/approvals.yml` | Human classifications, reviewed graph hash, waivers and rejections; the approved graph commit is derived from default-branch history, never supplied here. |
| `trust/portfolio.yml` | Portfolio membership and each system's input paths. Each system has its own manual file; root manual declarations are not automatically shared. |

There is no supported manual `self_hosted`, `high_risk`, `article_50`,
`approved_graph_sha`, `mcp_servers` or arbitrary `controls` field. Self-hosted
classification can come from supported detection; explicit human legal
classifications belong in approvals. Declare a tool through `tools` rather
than inventing a separate MCP declaration schema.

## Declaring subprocessors

Commit a repository-root `trust/subprocessors.yml` and the DPA files it
references. Each row uses the graph provider identifier for matching; `party`
is the processor's legal name (for example, `provider: bedrock` with
`party: AWS`):

```yaml
subprocessors:
- provider: bedrock
  party: AWS
  purpose: model inference
  location: Ireland
  dpa_path: trust/dpas/aws.pdf
```

The file is limited to 256 KiB and 100 unique providers. Every `dpa_path`
must be a tracked, repository-relative regular file; paths, YAML syntax,
duplicate keys, and links are validated. A row without a matching model
does not appear on the system card. A detected non-self-hosted provider
without a matching row produces a medium `subprocessor-undeclared` finding;
a detected data-processing dependency without a matching row produces a
medium `data-processor-undeclared` finding; invalid declarations produce
`subprocessors-invalid` (error). A declared row whose `provider` names the
detected party resolves either finding and makes the row visible. Scan still
writes a draft graph, but the gate applies its configured threshold.

The hashed `card.json` stores cited subprocessor rows so approved exports
and buyer CSV answers cannot silently change when the declaration changes.
Q4 includes the model's inference region when known and labels the
processor's location separately, citing both sources; processor location
alone does **not** prove an inference region. Q3 (customer-data training)
remains `unknown`: a DPA index does not establish training use. Trust declares
training use explicitly in `data_handling`, which answers `Q10`. A portfolio
shares the root declaration, includes only matching rows in each system's
approved graph, and lists the approved rows once on its cover. This does not
host or sign legal agreements.

## Declaring data handling

Declare, per AI provider, how long prompts and customer data are retained,
whether that data is used to train provider models, and how deletion is
requested. The section lives in `trust/manual.yml`:

```yaml
data_handling:
  - provider: anthropic
    retention: 30
    training_use: none
    deletion_path: trust/deletion-anthropic.md
    evidence_path: trust/terms/anthropic-dpa.md
    reason: Trust reviewed the enterprise agreement on 2026-10-19.
```

`provider` must match a provider that has a model in the graph; a declaration
for any other provider is dropped and reported as a `data-handling-invalid`
error. `retention` is `none`, `indefinite`, `unknown`, or a whole number of
days from 1 to 36500. `training_use` is `none`, `opt_in`, `yes` or `unknown`
(quote `"yes"` so YAML does not read it as a boolean). `deletion_path` and
`evidence_path` are optional and must be tracked, repository-relative files.
`reason` is required for each row and is validated but never stored in
`card.json`, exactly like the `reason` on a model or tool declaration.

Only declared rows are stored in the graph, always with `source: manual`. A
provider with a model but no declaration is **not** stored; `card.md`, the
questionnaire, the HTML/PDF pack and the portfolio cover render it as
`unknown`, and `trustbom readiness` raises a `disclosure:data-handling` gap.
"unknown" means Trust has not declared the value: it is never a statement
that data is retained, used for training, or safe. AI TrustBOM does not read,
fetch or verify provider terms.

Declared rows are part of the graph hash, so adding, changing or removing one
is a blocking semantic delta: the gate reports `data handling for <provider>`
and the change must be approved before it can be exported. A graph with no
declaration at all hashes exactly as it did before this feature, so existing
approvals stay valid.

The questionnaire renders three canonical questions from these rows:
`Q9_prompt_retention`, `Q10_provider_training_use` and `Q11_deletion_path`.
Existing reviewed answers to other questions are unaffected.

## Showing CI evaluation results

Commit `trust/evals.yml` with the names and repository-relative paths of
artifacts your CI already produces:

```yaml
evals:
  - name: refund
    artifact: evals/out/summary.json
    threshold: 95
```

A JSON summary at that path can contain
`{"pass":true,"score":97.2,"timestamp":"2026-09-30T11:30:00Z"}`.
`pass` must be a boolean; `score` is an optional percentage from 0 to 100,
and `timestamp` must include a UTC offset. Alternatively, declare a JUnit
`.xml` artifact: `pass` is based on its tests, failures, errors, and skipped
counts; JUnit supplies no numeric score. A JUnit run timestamp is optional;
a timestamp without a timezone cannot establish the run time and appears
as `unknown`.
The optional declaration `threshold` is a 0–100 percentage shown alongside
the artifact result; it does not override the artifact's pass/fail value.

Run `trustbom scan` **after** CI has produced the artifact. Only `evals.yml`
must be git-tracked; the artifact may be an untracked CI output, but must be a
repository-local regular file (at most 1 MiB). The declaration has a 256 KiB
limit and at most 100 entries. The resulting `card.json` contains cited
`eval_results[]`, separate from workflow-presence-only `eval_gates[]`; the
Markdown card links to the declared repository path and shows the last run
from the artifact, or `unknown` if absent. Exported cards retain the path
link but do not embed CI artifacts; keep them available for Trust to review.
An absent artifact yields a `low`-severity `eval-artifact-missing` finding,
not a fabricated result. An unreadable/unsafe/malformed artifact yields
`eval-artifact-invalid`; invalid declarations yield `evals-invalid`. Scanning
still writes a draft card, and the gate applies its usual finding threshold
(default `medium`). Neither scanning nor the Action runs evaluations.
For `scan --portfolio`, put each declaration at
`<system-path>/trust/evals.yml` and its artifacts inside that system's path;
root `trust/evals.yml` is not shared between systems.

## Scan, review and approve

Scanning reads Git-tracked files only. Do not Git-ignore `trust/` (or the
files in it): ignored declarations are untracked, are never read, and a
declared model is silently missing from the graph. Track every declaration
and evidence file before scanning.

From the test repository root, after creating `trust/system.yml` and the
referenced files:

```powershell
git add trust/manual.yml docs/ai-disclosure.md trust/deletion trust/terms
git commit -m "Declare reviewed AI deployment facts"
trustbom scan
trustbom readiness --framework nist-ai-rmf --framework iso-42001 --framework eu-ai-act
trustbom propose --dry-run --high-risk no --article-50 no
```

Inspect `trust/card.json`, `trust/card.md`, `trust/questionnaire.yml` and
`trust/findings.sarif`. Manual facts should be attributed to `manual` and
cite their declaration paths/lines. A successful scan does not mean valid
approval: malformed declarations produce findings, and `gate` applies its
finding threshold and approval policy. Missing provider matches and invalid
data-handling declarations must be resolved, not silently ignored.

Changing declared graph facts, including retention/training/deletion facts,
can change the graph fingerprint and require fresh Trust review. After
resolving findings, use `trustbom propose` to propose the approval, obtain
independent CODEOWNER review, and merge to the default branch. This
file, its reasons and a passing local scan are not themselves approval.
Approved export/publication additionally require an active license.

See the [`scan`](cli/scan.md) reference and
[approved export rules](approved-export.md).
