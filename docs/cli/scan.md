# `trustbom scan`

## Synopsis

```text
trustbom scan [--out <dir>] [--path <glob>] [--portfolio]
```

## Purpose

Inspects the tracked lockfiles, manifests, allowlisted configuration, and
`trust/manual.yml`, then writes into `trust/`:

- `trust/card.json`: the deterministic system graph (the source of truth)
- `trust/card.md`: a human-readable system card citing file evidence
- `trust/questionnaire.yml`: fixed-question answers cited to the graph
- `trust/findings.sarif`: findings as SARIF 2.1.0 for code-scanning UIs

Commit only `card.json`. The other three files are derived from it, so
[`init`](init.md) adds them to `.gitignore`; the generated workflow attaches
them to each pull-request run instead.

When the new graph differs from the existing `card.json` only in `commit`,
`scanned_at` and `cli_version`, the scan leaves `card.json` untouched and
prints `No changes detected; trust/card.json is up to date.` in place of its
`Wrote` line, so a rescan after an unrelated commit leaves `git status` clean.
Those three fields therefore record the last graph change, not the last scan;
the fingerprint excludes them either way. Any other difference, a missing or
unreadable `card.json`, or a `--path` scan rewrites it. The three derived files
are always rewritten. `scan --portfolio` applies the same rule to each system.

The command prints the path of each file it wrote, a one-line summary and the
approval status (see below): the headline, each blocker with a `Next:` line
naming its fix, and one next step. Do not copy a fingerprint into an approval by
hand; [`trustbom propose`](propose.md) writes the proposal. Findings never fail
the scan itself: only the [gate](gate.md) enforces policy.

### Approval status in `card.md`

`card.md` has an **Approval** section above the classification. It uses the
same approval and default-branch lookup as [`gate`](gate.md), so the two agree:

| Status | When |
| --- | --- |
| **Approved** | The default branch holds an approval whose `approved_graph_sha` matches this graph. The card shows Trust's `high_risk` and `article_50`, the `approved_by` claim and `approved_at`, and drops the `draft (unapproved)` watermark. |
| **Proposed, not approved** | `trust/approvals.yml` in the working tree matches this graph, but no approval of it is found on the default branch (for example on the proposal branch before the merge, or when the default branch cannot be proven). The card shows the proposed values. |
| **Not approved (stale)** | An approval exists, but for a different fingerprint: the graph has changed since. |
| **Not approved** | No approval file, or only the unsigned template or an invalid file. The refinement is **Needs information** while findings, an invalid approval file or a rejection (shown as Trust's rejection) block a proposal, and **Ready for sign-off** when nothing does, so `trustbom propose` would succeed. |

The classification below it is headed **Classification: scanner draft (not a
Trust decision)**: `card.json` always keeps the scanner's values (for example
`high_risk: unknown`), because the approval binds to its hash. `card.md` is a
local view of repository state, not evidence; [`trustbom export`](export.md)
produces the buyer copy. `card.json`, the questionnaire and SARIF do not
depend on the approval status.

`card.json` records a coverage statement: the detector packs and versions
installed, the ecosystems recognised from tracked manifests and lockfiles (with or
without detector support), `not_in_scope` exclusions with their reasons and
the fixed method limits. `card.md`, the questionnaire (`evidence:` per answer
and a top-level `coverage:` list), the HTML/PDF pack and the portfolio pack
render the same statement, and each citation names its detector pack, rule
and, for a dependency, the package and resolved version. The statement says
what was examined; it never claims complete coverage or absence of AI use.
A change to the coverage statement alone changes `graph_sha` but is
informational in the gate; Trust re-approves the new fingerprint to publish
it in approved exports. Detector pack names and versions are shown but are
outside `graph_sha`, so upgrading the CLI does not by itself require
re-approval; only a changed ecosystem, exclusion or detected row does.

Shipped detector packs cover Python, JavaScript/TypeScript, .NET,
Java/Kotlin (Maven `pom.xml`, Gradle `build.gradle`, `build.gradle.kts` and
`gradle.lockfile`) and Go (`go.mod`, `go.sum`), plus provider HTTP endpoints.
Ruby, PHP and Rust manifests are still listed in the coverage statement as
recognised ecosystems **without** detector support. Orchestration frameworks
(LangChain, LlamaIndex, Semantic Kernel, the Vercel AI SDK, Spring AI and
LangChain4j) are detected as frameworks; a framework alone never invents a
provider or a model.

Dependencies that typically process your data outside the model call:
Pinecone, pgvector, Weaviate, Qdrant, LangSmith, Langfuse and Helicone,
raise a medium `data-processor-undeclared` finding naming the party. The
scanner never creates a subprocessor row from a dependency: declare the party
in `trust/subprocessors.yml` (or waive the finding) and the declared row, with
its DPA evidence, resolves it exactly like `subprocessor-undeclared`.

## Options

| Option | Meaning |
| --- | --- |
| `--out <dir>` | Repository-relative directory for generated artifacts (default `trust`). |
| `--path <glob>` | Repository-relative glob limiting the tracked-file inventory; useful in a monorepo where only one application should be covered. |
| `--portfolio` | Scan each system declared in `trust/portfolio.yml`. |

```powershell
trustbom scan --out trust --path "apps/service/**"
```

### Declaring what the scanner cannot see

See the [complete `trust/manual.yml` example](../manual-declarations.md) for
every supported field, required companion files and allowed values.

Detection is manifest- and pattern-based, not a runtime proxy: it cannot see
prompt bodies, human-in-the-loop UX, or an undeclared deployment region. Add
those facts to `trust/manual.yml` with a `reason` for anything that overrides
an inferred value, and list any deliberately excluded paths (for example
`experiments/**`) under `not_in_scope`. Re-run `trustbom scan` after editing
`trust/manual.yml`; invalid overlay content produces a `manual-invalid`
finding rather than a failed scan. Subprocessors, data handling and CI
evaluation results have their own declarations, described in
[manual declarations](../manual-declarations.md#declaring-subprocessors).

When an allowlisted tracked config or `.env.example` contains a known provider
API URL but no matching SDK, the scan adds an informational
`provider-endpoint-suspected` finding with the host and file/line. This is
**not** a model declaration. Trust can verify the deployment and add a
reasoned model declaration, for example:

```yaml
models:
  - provider: cohere
    model_id: command-r
    endpoint: https://api.cohere.ai
    reason: Trust verified the direct API deployment.
```

The next scan cites `trust/manual.yml` for the binding and clears the
suspicion for that confirmed host. Generic `transformers` only creates a
Hugging Face binding when an allowlisted model ID and Inference API/endpoint
are both present. Inferred Ollama models carry `self_hosted: true` in
`card.json`; the card labels this scanner classification as a draft for
Trust to verify. A self-hosted model does not require a subprocessor row by
default; Trust can still declare its external hosting provider explicitly.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success, even when findings are reported. |
| `1` | Parser or usage error (unknown option). The usage text is printed and nothing is written. |
| `2` | Operational failure, such as a Git or input failure. |

## License

Free, single-system or portfolio, any number of systems. `scan` never checks a
license.

## Example

From a repository with `app/requirements.txt` listing `anthropic==0.40.0`, no
model ID in configuration and no declarations (output from a real run;
`<repository>` stands for the absolute path):

```powershell
trustbom scan
```

```text
Wrote <repository>\trust\card.json
Wrote <repository>\trust\card.md
Wrote <repository>\trust\questionnaire.yml
Wrote <repository>\trust\findings.sarif
Found 0 model(s), 0 tool(s), 3 finding(s). Review <repository>\trust\card.md.
Status: Not approved — Needs information
  - [medium] model-id-undeclared: The anthropic package is present but no model ID was found in an allowlisted configuration file.
    Next: Declare the model 'anthropic' under models: in trust/manual.yml (provider, model_id, region, reason), or list the path under not_in_scope: in trust/manual.yml if it is a false positive.
  - [medium] no-models-detected: No AI models were detected in tracked repository inputs.
    Next: Declare the product's models under models: in trust/manual.yml; if it uses none, Trust adds a waiver in trust/approvals.yml.
  - [medium] subprocessor-undeclared: Provider 'anthropic' has no declared subprocessor.
    Next: Add 'anthropic' to trust/subprocessors.yml, or list the path under not_in_scope: in trust/manual.yml if it is a false positive.
Next: Resolve the items above, then run 'trustbom propose'.
```

Exit code `0`. The three findings are `model-id-undeclared`,
`no-models-detected` and `subprocessor-undeclared`; declaring the model in
`trust/manual.yml` and the party in `trust/subprocessors.yml` and rescanning
gives `Found 1 model(s), 0 tool(s), 0 finding(s).` and
`Status: Not approved — Ready for sign-off`.

## Common errors

- **`Unrecognized command or argument '--bogus'`**: the parser prints the
  closest option and the usage, writes nothing and exits `1`.
- A `manual-invalid` finding: the overlay in `trust/manual.yml` is invalid; the
  scan still completes and writes a draft.
- `scan` needs at least one commit; `init` does not.

## See also

- [Manual declarations](../manual-declarations.md) and the
  [YAML input reference](../yaml-input-reference.md)
- [`gate`](gate.md), the next command
- [Workflow overview](../workflow.md)
