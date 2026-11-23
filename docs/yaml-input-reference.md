# User-editable YAML schema reference

This reference documents the YAML files you can author or review in
AI TrustBOM: fields, types, required values, defaults, limits and checks
against Git or other files. It describes the **implemented loader contracts**,
not a proposed format. It is human-readable schema documentation, not a
published JSON Schema or an editor-validation integration.

## Which files should you edit?

| File | Authoring and ownership |
|---|---|
| [`trust/system.yml`](#system-identity) | Engineer/Trust authors the system identity; init creates a starter. |
| [`trust/manual.yml`](#manual-declarations) | Engineer/Trust authors facts detection cannot establish. |
| [`trust/portfolio.yml`](#portfolio-membership) | Engineer/Trust declares portfolio membership; Growth init creates a starter. |
| [`trust/subprocessors.yml`](#subprocessors) | Trust maintains processor declarations and references to reviewed DPAs. |
| [`trust/evals.yml`](#evaluation-artifacts) | Engineer declares evaluation artifacts already produced by CI. |
| [`trust/question-map.yml`](#buyer-question-aliases) | Engineer/Trust maintains aliases to canonical buyer questions. |
| [`trust/answer-register.yml`](#reviewed-answer-register) | Trust records reviewed, supplied answer versions and evidence dependencies. |
| [`trust/answer-reuse.yml`](#reviewed-workbook-mapping) | `answer --suggest-only` generates a proposal; Trust reviews the mappings before merging them. |
| [`trust/approvals.yml`](#trust-decisions-and-approval) | `propose` authors a proposal. Trust may edit human decisions, waivers, rejections and break-glass. No field is automation-maintained. |

**Approvals are not entirely machine-owned.** Anyone with write access can
propose a change, but independent Trust/CODEOWNER review and merge establish
the approval boundary. Never treat an identity
string in YAML as authenticated approval.

Do not hand-edit generated `trust/card.json`, `trust/card.md`,
`trust/questionnaire.yml`, `trust/findings.sarif` or buyer export files as a
way to change source facts. `questionnaire.yml` is YAML, but it is an
**output**, not another configuration input. Detector packs and framework
mappings are shipped implementation resources, not customer-configurable
YAML inputs. GitHub workflows and detected SDK/MCP configuration have their
own formats and are outside this reference.

## Common conventions

- Use a top-level YAML mapping, UTF-8, exact field names and real YAML
  booleans `true`/`false`. These inputs have a **256 KiB (262144-byte)**
  declaration limit. Unknown/duplicate keys, YAML anchors, aliases and
  explicit tags are rejected.
- A required string must be nonblank. Optional means the field may be
  omitted; it does not promise that arbitrary `null`, empty collections or
  unknown enum values are valid. Use explicit values instead of relying
  on YAML scalar coercion. Quote enum strings `"yes"`/`"no"`.
- Use repository-relative paths with `/`, even on Windows. Do not use
  absolute paths, traversal (`..`), credentials, or symlinks as evidence.
  Only exclusion `path` entries are globs; evidence paths are file paths.
  Match the spelling/case of tracked paths for Linux portability.
- Track declarations with Git. Several loaders ignore an untracked
  declaration; others report it as invalid. An untracked file is not a
  reliable configuration input. Commit reviewed inputs and generated
  graphs before the approval/merge journey.
- Where timestamps are required, use ISO 8601 with an explicit offset,
  such as `2026-10-20T12:00:00Z` or `2026-10-20T13:00:00+01:00`.
  Date-only waiver expiry uses `yyyy-MM-dd`.

The examples use synthetic facts. Hashes/commit IDs shown as repeated hex
characters satisfy the **shape** only; replace them with real values from
the approved Git history before running history-dependent commands.
File existence, tracking, provider matching, branch membership and historical
approval cannot be established by syntax validation alone.

## System identity

**Default:** `trust/system.yml`. In a portfolio, use the file explicitly
selected by that system's `system_yml`.

```yaml
name: Support Assistant
description: Drafts support replies for an agent to review.
customer_facing_name: Example Support Assistant
article_50_copy_path: docs/ai-disclosure.md
```

| Field | Type | Required / default | Constraints |
|---|---|---|---|
| `name` | string | Required when the file is supplied | Nonblank system name. |
| `description` | string | Optional; absent/blank becomes no description | Description, not evidence of deployed AI usage. |
| `customer_facing_name` | string | Optional; absent/blank becomes no separate name | Display name. |
| `article_50_copy_path` | string | Optional; absent/blank becomes no path | Repository-relative disclosure-copy path, not a legal applicability decision. |

A reasoned `manual.yml.article_50_copy_path` takes precedence over the
system-level path. Neither field determines the human `article_50` decision.
An invalid identity file produces a `manual-invalid` finding during scan.

## Manual declarations

**Default:** `trust/manual.yml`. In a portfolio, use the system's explicitly
selected `manual_yml`. The
[complete example and detailed field rules](manual-declarations.md) cover all
supported declaration fields.

```yaml
not_in_scope:
  - path: experiments/**
    reason: Synthetic prototypes are not part of the shipped service.
models:
  - provider: anthropic
    model_id: claude-sonnet-4-5
    region: eu-central-1
    endpoint: https://api.anthropic.com
    reason: Synthetic deployed model reviewed for this test.
tools:
  - name: send_reply
    effect: send
    reason: Sends the reply only after human confirmation.
data_classes:
  - class: support_message
    to_model: true
    reason: Redacted support text is included in inference requests.
prompt_bodies_stored:
  value: false
  reason: Application logs record request IDs only.
human_send_required:
  value: true
  reason: An agent must confirm sending.
kill_switch:
  value: true
  reason: A feature flag disables model calls.
article_50_copy_path:
  path: docs/ai-disclosure.md
  reason: Trust maintains the disclosure copy.
data_handling:
  - provider: anthropic
    retention: 30
    training_use: none
    deletion_path: trust/deletion/anthropic.md
    evidence_path: trust/terms/anthropic.md
    reason: Synthetic contractual terms reviewed for this test only.
```

All top-level fields are optional. Missing collections default to empty;
omitted controls remain undeclared, not `false`.

| Field | Structure and required row fields |
|---|---|
| `not_in_scope` | List of mappings with string `path` (repository glob) and string `reason`. Not a list of bare path strings. Paths must be unique. |
| `models` | List of mappings with required strings `provider`, `model_id`, `reason`; optional strings `region`, `endpoint`. Unique `(provider ignoring case, model_id)` pairs. |
| `tools` | List with required strings `name`, `effect`, `reason`. Effect: `unknown`, `read`, `write`, `send`. Unique tool names. |
| `data_classes` | List with required string `class`, boolean `to_model`, string `reason`. Unique `(class, to_model)` pairs. No per-model selector. |
| `prompt_bodies_stored`, `human_send_required`, `kill_switch` | Optional mappings, each requiring boolean `value` and string `reason`. Use omission, not `value: unknown`, for an undeclared control. |
| `article_50_copy_path` | Optional mapping requiring string `path` (repository-relative) and string `reason`. |
| `data_handling` | List requiring string `provider`, scalar `retention`, string `training_use`, string `reason`. Optional strings `deletion_path`, `evidence_path`. Unique provider ignoring case. |

`endpoint` must be an absolute HTTP(S) URL **without credentials, query
or fragment**. Provider identifiers should match detected graph providers
(`anthropic`, `openai`, `azure-openai`, `bedrock`, `vertex`, etc.).

`retention` accepts `none`, `indefinite`, `unknown` or a whole number of
days from **1 through 36500**. `training_use` accepts `none`, `opt_in`,
`"yes"` or `unknown`. These declarations concern provider behavior, not
your own company's training. Every data-handling provider must have a model in the
final graph. Optional evidence/deletion paths must be Git-tracked files.
Absent data handling renders as unknown and raises an advisory readiness gap.

Manual declarations override matching inferred facts but do not approve the
graph. Invalid YAML/manual fields produce scan findings; invalid/unmatched
data-handling declarations are reported as well. Scan success is not proof
of valid input or a passing gate.

## Portfolio membership

**Path:** `trust/portfolio.yml`, selected with portfolio commands such as
`scan --portfolio` and `gate --portfolio`. The canonical path is not
configurable by the Growth workflow.

```yaml
org: Example Vendor
systems:
  - id: assist
    path: apps/assist
    system_yml: apps/assist/trust/system.yml
    manual_yml: apps/assist/trust/manual.yml
  - id: search
    path: apps/search
    system_yml: apps/search/trust/system.yml
    manual_yml: apps/search/trust/manual.yml
```

| Field | Type | Required / default | Constraints |
|---|---|---|---|
| `org` | string | Required | Nonblank organization display name. |
| `systems` | list of mappings | Required | At least one entry. |
| `systems[].id` | string | Required | Matches `^[a-z0-9][a-z0-9-]{0,62}$` (1-63 characters); unique ignoring case. |
| `systems[].path` | string | Required | Repository-relative directory containing tracked files; no globs or `.`/`..` segments. System paths must not overlap. |
| `systems[].system_yml` | string | Optional; omitted/blank means no selected identity file | If supplied, must be tracked and inside that system's path. It is not implicitly discovered. |
| `systems[].manual_yml` | string | Optional; omitted/blank means no selected manual file | If supplied, must be tracked and inside that system's path. It is not implicitly discovered. |

All paths above are **repository-root relative**, not relative to
`portfolio.yml`. Scan writes each system's graph under
`trust/systems/<id>/`; its approval lives there too. Root manual/eval
declarations are not shared among systems. Root subprocessors are shared,
then filtered to each graph's matching providers/processors.
Manual evidence/deletion paths still use repository-root-relative spelling,
but must belong to the selected system's tracked-file scope. For example,
use `apps/assist/trust/terms/anthropic.md` in assist's manual file, not a
shared root `trust/terms/anthropic.md`.

Invalid membership, missing tracked inputs, duplicate IDs or overlapping
paths are operational errors, not silently skipped systems. See
[Growth setup](growth-workflow.md).

## Subprocessors

**Path:** repository-root `trust/subprocessors.yml` for single-system and
portfolio scans.

```yaml
subprocessors:
  - provider: anthropic
    party: Example AI Processor
    purpose: Synthetic model inference
    location: Ireland
    dpa_path: trust/dpas/anthropic.md
```

| Field | Type | Required / default | Constraints |
|---|---|---|---|
| `subprocessors` | list of mappings | Required if file exists | **1-100** entries. Omit the optional file rather than supplying an empty list. |
| `provider` | string | Required per row | Nonblank plain text; normalized to lower case; unique ignoring case. Matches the graph's provider/processor identifier, not necessarily its legal name. |
| `party` | string | Required per row | Nonblank plain-text legal-party name. |
| `purpose` | string | Required per row | Nonblank plain-text processing purpose. |
| `location` | string | Required per row | Nonblank plain-text processor location; does not establish model inference region. |
| `dpa_path` | string | Required per row | Normalized repository-relative, Git-tracked regular file; no glob, traversal, empty segments, symlink or reparse point. |

For example, a Bedrock deployment uses `provider: bedrock` with the
reviewed legal party in `party`. Matching declared processors are included in
the graph; unrelated rows are not evidence of model use. This index does not
host/sign agreements or establish training policy.

Malformed declarations produce `subprocessors-invalid` error findings;
missing processor declarations can produce `subprocessor-undeclared` or
`data-processor-undeclared`. A scan can still exit successfully with findings.

## Evaluation artifacts

**Default:** `trust/evals.yml`. For a portfolio, use
`<system-path>/trust/evals.yml`; artifact paths remain repository-root
relative and must stay inside that system's path.

```yaml
evals:
  - name: refund
    artifact: evals/out/summary.json
    threshold: 95
  - name: regression
    artifact: evals/out/results.xml
```

| Field | Type | Required / default | Constraints |
|---|---|---|---|
| `evals` | list of mappings | Required if file supplied | **1-100** entries. |
| `evals[].name` | string | Required | Nonblank; unique ignoring case. |
| `evals[].artifact` | string | Required | Normalized repository-relative regular-file path ending in `.json` or `.xml`; no globs/traversal/links. Artifact may be untracked CI output. |
| `evals[].threshold` | number | Optional; no threshold when omitted | Percentage **0-100**, inclusive. Displayed alongside the result, not used to override artifact pass/fail. |

Produce artifacts **before** scanning. Each artifact is limited to **1 MiB**.
The supported JSON object has required boolean `pass`, optional numeric
`score` (0-100) and optional string `timestamp` (explicit UTC offset);
unknown/duplicate JSON fields are invalid:

```json
{"pass":true,"score":97.2,"timestamp":"2026-10-20T12:00:00Z"}
```

JUnit XML accepts unnamespaced `testsuite`/`testsuites`; pass/fail is
derived from counts, with no numeric score. A missing/unzoned JUnit timestamp
cannot establish a run time. Neither scanning nor the Action runs evaluations.

Invalid declarations produce `evals-invalid`; missing artifacts produce
`eval-artifact-missing`, and unsafe/malformed artifacts
`eval-artifact-invalid`. The latter two are low-severity findings.

## Buyer-question aliases

**Path:** `trust/question-map.yml`, used by CSV questionnaire answering.

```yaml
maps:
  - match: 'where (is|are) .* models hosted'
    canonical: Q4_inference_region
  - match: 'prompt retention policy'
    canonical: Q9_prompt_retention
```

| Field | Type | Required / default | Constraints |
|---|---|---|---|
| `maps` | list of mappings | Required if file supplied | May be empty; built-in matching remains available. |
| `maps[].match` | string | Required | Nonblank .NET regular expression, at most **512 characters**, supported by the non-backtracking engine. |
| `maps[].canonical` | string | Required | Exact canonical ID from the table below. |

Matching is case-insensitive and culture-invariant, with a **250 ms**
regex timeout. Input question text is Unicode-compatibility normalized,
lowercased, and punctuation/symbol/whitespace runs become spaces. Write
patterns for that normalized text; single-quoted YAML preserves regex
backslashes. Unsupported constructs such as backreferences/lookarounds
must not be used with the non-backtracking engine.

| Canonical ID | Meaning |
|---|---|
| `Q1_do_you_use_ai` | AI use |
| `Q2_models_and_vendors` | Models and providers |
| `Q3_is_customer_data_used_to_train` | Customer-data training, including undeclared training by your own company |
| `Q4_inference_region` | Inference region |
| `Q5_human_oversight` | Human oversight |
| `Q6_high_risk_ai_act` | Human high-risk decision |
| `Q7_prompt_logging` | Prompt-body storage |
| `Q8_customer_kill_switch` | Disabling AI |
| `Q9_prompt_retention` | Provider retention |
| `Q10_provider_training_use` | Provider training use |
| `Q11_deletion_path` | Deletion-request route |

Aliases select existing answers; they do not author new claims. Different
canonical IDs matching the same question are ambiguous and remain unmatched.
Invalid YAML/regex/IDs cause an explicit answering error (exit `2`), rather
than silently falling back. See [questionnaire mapping](questionnaire-mapping.md).

## Reviewed answer register

**Default:** `trust/answer-register.yml`. Select another repository-relative
file with `drift --register` or `readiness --register`. This is a
Trust-maintained record, not a buyer-text database.

```yaml
schema_version: 1
answers:
  - answer_id: support-region
    question_id: Q4_inference_region
    version: 1
    reviewed_by: secops
    reviewed_at: 2026-10-20T12:00:00Z
    supplied_at: 2026-10-20T12:30:00Z
    dependencies:
      - system_id: default
        source_commit: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
        approval_commit: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
        source_graph_sha: cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
        fields:
          - collection: models
            key: anthropic/claude-sonnet-4-5
        citations:
          - trust/manual.yml
```

| Field | Type | Required / default | Constraints |
|---|---|---|---|
| `schema_version` | integer | Required | Exactly `1`. |
| `answers` | list of mappings | Required | **1-500** entries. |
| `answer_id` | identifier string | Required per answer | 1-128 characters, matches `^[A-Za-z0-9][A-Za-z0-9._:-]{0,127}$`. |
| `question_id` | identifier string | Required per answer | Same identifier rule; use a supported canonical ID for reusable answers. Parsing an arbitrary identifier does not make it a supported question. |
| `version` | integer | Required per answer | Positive; `(answer_id, version)` pairs unique. |
| `reviewed_by` | identifier string | Required per answer | Same identifier rule, e.g. `secops`, not an email address or `@login`. Not an authenticated identity claim. |
| `reviewed_at` | timestamp string | Required per answer | Explicit UTC offset; maximum 64 characters. |
| `supplied_at` | timestamp string | Required per answer | Same rule; cannot precede `reviewed_at`. |
| `dependencies` | list of mappings | Required per answer | **1-50**, unique `system_id` per answer. |
| `dependencies[].system_id` | string | Required | `default` for the root system, otherwise portfolio ID; matches `^[a-z0-9][a-z0-9-]{0,62}$`. |
| `dependencies[].source_commit` | string | Required | Lower-case **40- or 64-character** Git object ID of the approved graph commit. |
| `dependencies[].approval_commit` | string | Required | Same Git ID shape; the historical commit that carries its approval, not a newly invented commit. |
| `dependencies[].source_graph_sha` | string | Required | Lower-case **64-character** SHA-256 matching that graph. |
| `dependencies[].fields` | list of mappings | Required | **1-100** unique `(collection, key)` pairs. |
| `fields[].collection` | string | Required | One of the semantic collections below. |
| `fields[].key` | string | Required | Nonblank plain text, at most **256 characters**; exact semantic row/field key. No wildcard expansion. |
| `dependencies[].citations` | list of path strings | Required | **1-100** repository-relative tracked paths, at most **512 characters** each; duplicates are deduplicated. |

Accepted semantic collections are `system`, `models`, `tools`, `mcp_servers`,
`data_flows`, `controls`, `eval_gates`, `eval_results`, `subprocessors`,
`data_handling`, `findings`, `classification` and `coverage`.
Keys match the graph differ's exact keys, **not JSONPath**:

| Collection | Key examples / form |
|---|---|
| `system` | `name`, `description`, `customer_facing_name`, `article_50_copy_path` |
| `models` | `<provider>/<model_id>` |
| `tools`, `controls`, `eval_results` | Row/control name, e.g. `human_send_required` |
| `mcp_servers` | `<name>/<command-or-url>` |
| `data_flows` | `<class>/to_model=true` or `<class>/to_model=false` |
| `eval_gates` | `<name>/<path>` |
| `subprocessors`, `data_handling` | Provider identifier |
| `findings` | `<id>/<subject-or-repository-path-or-repository>`; use the exact reported key |
| `classification` | `high_risk`, `article_50` |
| `coverage` | `ecosystems`, `ecosystem_manifests`, `not_in_scope`, `method_limits` |

Do not invent dependency keys: the loader accepts a nonblank key, but that
alone does not prove it identifies the intended fact. Register every fact
the supplied answer depends on. Historical graph hashes, approval authority
and reachability are separately verified by drift/reuse; shape-valid fake
hashes cannot pass those checks.

Keep supplied versions immutable; append a newly reviewed/supplied version
instead of rewriting history. Invalid/unverifiable input is an explicit
error; verified affected answers are reported for review, not automatically
reapproved. See [the drift walkthrough](questionnaire-mapping.md#buyer-answer-drift).

## Reviewed workbook mapping

**Default:** `trust/answer-reuse.yml`. `answer --reuse-map` can select another
repository-relative path. Prefer generating the initial proposal:

```text
trustbom answer --workbook rfi/buyer.xlsx --sheet Questions --id-column id --question-column question --answer-column answer --suggest-only
```

Example shape (the hash must be the actual workbook SHA-256):

```yaml
schema_version: 1
workbook_sha256: dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd
sheet: Questions
id_column: id
question_column: question
answer_column: answer
mappings:
  - row_id: region
    answer_id: support-region
    version: 1
    question_id: Q4_inference_region
    rationale: Exact supported question matched to a reviewed supplied answer.
    status: suggested
```

| Field | Type | Required / default | Constraints |
|---|---|---|---|
| `schema_version` | integer | Generated as `1`; parser defaults omitted field to `1` | Only `1` supported; include it explicitly. |
| `workbook_sha256` | string | Required | Lower-case 64-character SHA-256 of the input workbook bytes. |
| `sheet` | string | Required | Nonblank, at most **128 characters**. Exact selected sheet. |
| `id_column`, `question_column`, `answer_column` | string | Each required | Nonblank, at most **128 characters** each; three distinct, case-sensitive header names matching the selected columns. |
| `mappings` | list of mappings | Required | **0-5000** entries. |
| `mappings[].row_id` | string | Required | Nonblank, at most **128 characters**, unique; must name a workbook row ID. |
| `mappings[].answer_id` | string | Required | 1-128 characters; `^[A-Za-z0-9][A-Za-z0-9._-]*$`. |
| `mappings[].version` | integer | Required | Positive; references the reviewed register version. |
| `mappings[].question_id` | string | Required | Same identifier rule as mapping `answer_id`; must agree with the referenced candidate to reuse it. |
| `mappings[].rationale` | string | Required | Nonblank, at most **512 characters**. |
| `mappings[].status` | string | Required | Exactly `suggested` or `reviewed`. |

Trust reviews suggestions, changes only accepted rows to `reviewed`, and
merges the mapping through independent Git review. The current mapping must
match the default-branch file (line-ending normalization allowed) before
reuse. Editing a local status alone is insufficient. Workbook hash, sheet
and columns must still match; source answer versions must remain verified,
unaffected and unambiguously matched.

There are **no `reviewed_by` or `reviewed_at` fields in this mapping**;
those belong to the answer register. Git review provides mapping-review
provenance. A map contains answer references, not arbitrary answer text.
Re-running suggestions can replace previous mapping/output, so review the
new proposal rather than assuming an old review carries over.

## Trust decisions and approval

**Default:** `trust/approvals.yml`; portfolios use
`trust/systems/<id>/approvals.yml`. Gate/propose `--baseline`
and export `--from-approvals` support alternate paths where documented.

Prefer `propose` to copying hashes/timestamps manually. Trust still
reviews its proposal and can edit the fields below. `init` creates a
comment-only **unsigned template**, not a valid approval; `propose`
replaces the template's comments with the proposal.

```yaml
approved_graph_sha: cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
approved_by: secops
approved_at: 2026-10-20T12:00:00Z
high_risk: no
article_50: yes
waivers: []
rejections: []
```

| Field | Type | Required / default | Ownership and constraints |
|---|---|---|---|
| `approved_graph_sha` | string | Required | Proposed/reviewed full lower-case **64-character** graph SHA-256; never the short display fingerprint. |
| `approved_by` | string | Required | Human/proposed reviewer label, **1-128 characters** after trimming; not authenticated by the CLI. |
| `approved_at` | timestamp string | Required | Explicit offset, maximum **64 characters**; ISO timestamp with seconds and optional fractional seconds. Recorded, not enforced: a future value is not rejected, and the merged approval in Git history is the authority. |
| `high_risk` | string | Required | `yes` or `no`; explicit human decision, not scanner inference. `unset` is no longer accepted. |
| `article_50` | string | Required | `yes` or `no`; explicit human decision. `unset`, `applies` and `not_applicable` are no longer accepted (`applies` is `yes`, `not_applicable` is `no`); rendered output keeps the buyer wording. `trustbom propose --high-risk yes\|no --article-50 yes\|no` replaces legacy values. |
| `waivers` | list of mappings | Optional; defaults to `[]` | Human-owned, time-limited finding exceptions; see below. |
| `rejections` | list of mappings | Optional; defaults to `[]` | Human-owned blockers; see below. |
| `break_glass` | mapping | Optional; absent means no exception | Temporary draft-only gate exception; see below. |
| `approved_commit` | not supported | Unknown field | Removed. The approved graph commit is derived from default-branch first-parent history and never stored; the file is rejected if it contains this key. |

The parser normalizes classification values to lower case. A matching hash
does not waive findings, clear rejections or authorize export without a
valid default-branch approval with a derived graph baseline.

### Waivers, rejections and break-glass

These are separate optional policy shapes, **not recommendations to add
exceptions**. For example, replace the empty `waivers` list only after
reviewing an actual finding:

```yaml
waivers:
  - finding_id: region-unknown
    reason: Synthetic exception pending deployment-region confirmation.
    until: 2026-11-01
    by: secops
```

Each waiver requires `finding_id` (identifier, max 128), `reason` (nonblank,
max **2048**), `until` (valid `yyyy-MM-dd` date) and `by` (nonblank, max
**128**). Finding IDs are lowercased and must match
`^[a-z0-9][a-z0-9._:-]*$`; IDs must be unique. Expired waivers do not cover
findings. A waiver does not approve a changed graph.

```yaml
rejections:
  - id: contract-review
    reason: Synthetic provider agreement review is still outstanding.
    by: secops
```

Each rejection requires `id` (same identifier rule/128-character limit,
unique), `reason` (nonblank, max 2048) and `by` (nonblank, max 128).
A nonempty rejection list blocks ordinary approval even if the graph hash
matches; remove a resolved rejection only through review.

```yaml
break_glass:
  by: secops
  expires: 2026-10-21T12:00:00Z
  ticket: INC-123
  allows:
    - graph_delta
```

All break-glass fields are required when the block is supplied: nonblank
`by` and `ticket` (max **128** each), explicit-offset `expires` timestamp,
and a nonempty list of unique condition identifiers (same normalized
identifier rule/128-character limit). Recognized policy conditions are
`graph_delta`, `findings`, `finding:<id>`, `rejections` and
`rejection:<id>`. An arbitrary shape-valid identifier does not cover a
different policy condition. Expired exceptions do not apply.

Break-glass can soften covered gate conditions **only as a draft**.
It never creates a publishable approval or authorizes exporting
that draft. A valid historical approved snapshot remains a separate matter.
See [approval and export rules](approved-export.md) and the
[review workflow](reducing-review-friction.md).

## Validation and change workflow

AI TrustBOM currently validates these contracts through its loaders, not
through a customer `validate-yaml` command or a published JSON Schema.

| Input | How to exercise validation |
|---|---|
| System/manual/subprocessors/evals | Track inputs, run `scan`, inspect SARIF/card findings, then `gate`. Findings do not make scan fail by themselves. |
| Portfolio | Track membership and selected inputs, run `scan --portfolio` and `gate --portfolio`; invalid membership is an explicit error. |
| Question aliases | Run `answer --questions <csv>` with a licensed, valid approved baseline. Input/regex errors differ from unmatched rows, but both can exit `2`. |
| Answer register | Run `drift` (paid) or `readiness --register <path>` (free, advisory); historical references must be available. |
| Reuse mapping | Generate suggestions, review/merge the map, then rerun workbook answering without `--suggest-only` (paid); mismatches fail or remain unresolved. |
| Approval | Use `propose --dry-run`, review the proposal, then `propose`; approval publication additionally needs merge to the default branch. |

Preserve unknowns instead of inventing facts. Commit input changes, rescan
and review semantic changes before updating approval. An exit `0` from a
parser, scan, advisory report or proposal command is never a substitute for
independent Trust review.

### Implementation references

These loaders define current behavior and should be checked when extending
this reference:

| Contract | Implementation |
|---|---|
| System/manual | `ManualConfigurationLoader` |
| Portfolio | `PortfolioConfigurationLoader` |
| Subprocessors | `SubprocessorLoader` |
| Evals | `EvalArtifactLoader` |
| Question aliases | `QuestionnaireAnswerService` |
| Answer register | `AnswerDriftRegisterLoader` |
| Reuse map | `AnswerReuseMapLoader` and types |
| Approval | `ApprovalFileLoader` |
