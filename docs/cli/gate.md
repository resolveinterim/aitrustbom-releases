# `trustbom gate`

## Synopsis

```text
trustbom gate [--baseline <path>] [--fail-on <severity>] [--portfolio] [--github-rules]
```

## Purpose

Compares the current graph against the full fingerprint recorded as
`approved_graph_sha` in `trust/approvals.yml` (default path; override with
`--baseline`), and against the graph at the approved graph baseline commit derived
from default-branch first-parent history. When the default branch cannot be proven
(for example a shallow clone, or a detached pull-request checkout with neither
`origin/HEAD` nor a fetched `refs/remotes/origin/<base>` matching `GITHUB_BASE_REF`)
the gate still evaluates the hash, findings, waivers and rejections but reports no baseline diff. It reports, in plain language:

- how many conditions currently block the PR;
- the semantic differences (added/removed models, tools, MCP servers,
  classification changes) between the current graph and the approval; the
  PR comment's **What changed** section also shows the
  `path:line` evidence for each difference, says **No change** when the
  approved version matches, and summarises a **First record** when the
  default branch has no approved version yet. Fingerprints sit in a
  collapsed **Technical details** block. When the comment asks for
  `trustbom propose`, it gives the exact command (with `--system`, and the
  `--high-risk` and `--article-50` decisions for a first sign-off) and an
  `npm install --global trustbom@<version>` line pinned to the Action's
  version; local `gate` output omits the install line;
- each blocking condition and the concrete way to resolve it: correct the
  change, add a reasoned manual declaration, fix the finding, add a
  time-bounded waiver, resolve a rejection, or have Trust approve the graph.

The same assessment is appended to the local, Git-ignored `trust/card.md`
(without changing `card.json` or its hash), whose
[Approval section](scan.md#approval-status-in-cardmd) uses this command's
approval lookup, and printed as the PR-comment Markdown that the
GitHub Action posts. See [github-action.md](../github-action.md) for running
this automatically on every pull request.

## Options

| Option | Meaning |
| --- | --- |
| `--baseline <path>` | Repository-relative approval file path (default `trust/approvals.yml`). |
| `--fail-on <severity>` | Minimum finding severity that requires a waiver (default `medium`). |
| `--portfolio` | Evaluate each portfolio system against its own approval. |
| `--github-rules` | Also read GitHub (through `gh`) and report the enforcement level; a level below `enforced` is a finding and only a `violation` fails the gate. Needs `gh` and a token; off by default so the gate stays offline. See [`github-rules`](github-rules.md#check-the-enforcement-level). |

```powershell
trustbom gate --baseline trust/approvals.yml --fail-on medium
```

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | The gate passed (or an explicit draft break-glass soft pass). |
| `1` | Parser or usage error. |
| `2` | Operational or argument failure (for example an invalid `--fail-on`, or Git or input failures). |
| `3` | Policy failure (`CliExitCodes.GateFailed`). |

## License

Free, single-system or portfolio. `gate` never checks a license.

## Example

In a freshly scanned repository whose approval file is still the unsigned
template (real output, trimmed to the fixed sections):

```powershell
trustbom gate
```

```text
<!-- ai-trustbom-pr-comment -->
## AI TrustBOM

**Status:** Not approved — Needs information

**Gate:** bootstrap required

- \[medium\] model-id-undeclared: The anthropic package is present but no model ID was found in an allowlisted configuration file.
  **Next:** Declare the model 'anthropic' under models: in trust/manual.yml (provider, model\_id, region, reason), or list the path under not\_in\_scope: in trust/manual.yml if it is a false positive.
...
**Next:** Resolve the items above, then run `trustbom propose`.
...
### Why the gate failed

The gate is blocked because this repository has no approved version yet.
...
### How to resolve

#### 1. The approval file is still the unsigned template

trust/approvals.yml exists but is still the unsigned template. The first version must be reviewed by Trust before it counts as approved.
...
```

Exit code `3`. After the findings are resolved and the approval is merged to
the default branch, the same command reports:

```text
**Status:** Approved

**Gate:** passed

**Next:** None: the default branch holds Trust's approval of this graph.
...
### Gate result

The current version matches its approval and has no blocking policy conditions.
...
### Gate detail

No approval action is required.
```

On a feature branch that holds a matching proposal, the status is
**Proposed, not approved** until the proposal merges to the default branch.

Exit code `0`.

## Common errors

- **Gate `bootstrap required`**: no signed `trust/approvals.yml` entry exists yet.
  Run [`propose`](propose.md).
- **Unwaived findings at or above `--fail-on`**: fix the finding, add a reasoned
  manual declaration or a time-bounded waiver.
- **No baseline diff reported**: the default branch cannot be proven (shallow
  clone, or a detached checkout without `origin/HEAD`); run
  `git remote set-head origin -a`.
- **Non-empty `rejections`**: the gate fails until they are resolved.

## See also

- [`propose`](propose.md)
- [GitHub Action](../github-action.md)
- [Enforcement levels](../enforcement-levels.md)
- [Reducing review friction](../reducing-review-friction.md)
