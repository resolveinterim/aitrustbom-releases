# `trustbom changelog`

## Synopsis

```text
trustbom changelog [--from <sha> --to <sha>] [--system <id>] [--fail-on <severity>]
```

## Purpose

Prints a Git-derived Markdown comparison of the last two historically approved
graph baselines, or compares an approved baseline (`--from`) with a committed
target (`--to`). An unapproved target is labelled **internal draft - unapproved
- not for customer use**. It writes no export artifact, changes no approval and
does not use the uncommitted checkout. See [Review pack history](../pack-history.md)
for the baseline rules and limits.

## Options

| Option | Meaning |
| --- | --- |
| `--from <sha>` | Historically approved baseline commit SHA; use with `--to`. |
| `--to <sha>` | Committed target graph SHA; use with `--from`. |
| `--system <id>` | Portfolio system id; omit for `trust/card.json`. |
| `--fail-on <severity>` | Historical approval finding threshold, matching the approval policy (default `medium`). |

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | The Markdown comparison was printed. |
| `1` | Parser or usage error. |
| `2` | No Markdown: an unavailable default branch, missing or shallow history, an unverified baseline, a malformed or tampered graph, fewer than two verified baselines in implicit mode, or a comparison with more than 500 semantic changes. |

## Licence

Free. `changelog` is not a paid command.

## Example

In a repository with a single approved baseline (real output):

```powershell
trustbom changelog
```

```text
Changelog failed: Fewer than two historically approved graph baselines exist for trust/card.json on the default branch.
```

Exit code `2`. With two approved baselines the command prints Markdown listing
semantic additions, removals and changes, for example `+ Model Anthropic claude-haiku-4`.

## Common errors

- **Fewer than two approved baselines**: use `--from` and `--to`.
- **Shallow checkout**: fetch the relevant Git history; there is no fallback to
  a draft.

## See also

- [Review pack history](../pack-history.md)
- [`gate`](gate.md), [`export`](export.md)
