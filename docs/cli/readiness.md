# `trustbom readiness`

## Synopsis

```text
trustbom readiness [--question <id>]... [--portfolio] [--register <path>]
    [--fail-on <severity>] [--max-approval-age-days <n>]
    [--max-answer-review-age-days <n>] [--framework <id>]...
```

## Purpose

Writes `export/readiness/readiness.md` and `export/readiness/readiness.json`. It
checks the built-in canonical disclosure set, separates evidence coverage,
approval validity, unresolved findings, and freshness, and also inspects tracked
reviewed-answer history when `trust/answer-register.yml` is present. The report
is advisory only: it distinguishes unknown/unsupported evidence from cited
not-applicable declarations, reports scoped gaps and exclusions without a
generic compliance score, and does not change graph hashes, approval bytes,
gate exit semantics, or buyer-facing approval status. A provider with a model
but no `data_handling` declaration raises a `disclosure:data-handling` gap.

Repeated `--framework <id>` options additionally write a free, offline
framework-alignment overlay per requested framework to
`export/readiness/frameworks/<id>.md` and `<id>.json`. Supported ids are
`nist-ai-rmf`, `iso-42001`, and `eu-ai-act`. Each overlay maps versioned
framework items to graph evidence and declarations with a per-item status
(`evidenced`, `declared`, `organizational`, `not-applicable`, or `not-covered`)
and states explicitly that it is a coverage view, not a certification or a
compliance score. The overlay is read-only: it is derived from the same graph
and never changes graph hashes, approval bytes, or gate semantics. An unknown
framework id is rejected before any file is written. The same framework
alignment section appears in approved `publish` and `pack` output.

## Options

| Option | Meaning |
| --- | --- |
| `--question <id>` | Canonical question identifier to assess. Repeat to narrow the built-in readiness set. |
| `--register <path>` | Tracked YAML register of reviewed answer versions to assess for drift and freshness (default `trust/answer-register.yml`). |
| `--portfolio` | Assess each approved portfolio system independently. |
| `--fail-on <severity>` | Minimum finding severity that must be resolved for buyer-readiness (default `medium`). |
| `--max-approval-age-days <n>` | Maximum approval age, in days, before approval freshness becomes a readiness gap (default `90`). |
| `--max-answer-review-age-days <n>` | Maximum reviewed-answer age, in days, before answer freshness becomes a readiness gap (default `90`). |
| `--framework <id>` | Framework mapping id to assess (`nist-ai-rmf`, `iso-42001`, `eu-ai-act`). Repeat to write an advisory alignment under `export/readiness/frameworks`. |

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | The report was written (gaps do not fail the command). |
| `1` | Parser or usage error. |
| `2` | Operational or argument failure, including an unknown framework id (rejected before any file is written). |

## Licence

Free. `readiness` never checks a licence; framework overlays are free and offline.

## Example

In a repository with the undeclared anthropic model from the [`scan`](scan.md)
example (real output; `<repository>` is the absolute path):

```powershell
trustbom readiness
```

```text
Wrote <repository>\export\readiness\readiness.md
Wrote <repository>\export\readiness\readiness.json
Readiness gaps: 15; assessed: 17; excluded: 2
```

Exit code `0`. Add `--framework nist-ai-rmf` to also write
`export/readiness/frameworks/nist-ai-rmf.md` and `.json`.

## Common errors

- **Unknown `--framework` id**: rejected with exit `2`; use `nist-ai-rmf`,
  `iso-42001` or `eu-ai-act`.
- **Many gaps on a fresh repository**: expected; a gap means evidence or a
  declaration is missing, never a compliance score.

## See also

- [Manual declarations](../manual-declarations.md) to close gaps
- [`drift`](drift.md) and [Map a buyer questionnaire](../questionnaire-mapping.md)
- [`gate`](gate.md)
