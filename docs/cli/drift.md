# `trustbom drift`

## Synopsis

```text
trustbom drift [--register <path>]
```

## Purpose

Reports which explicitly registered buyer answers may be stale against current
graph evidence. It compares explicitly supplied, reviewed answer versions in a
tracked YAML register with the current graph. See
[buyer-answer drift](../questionnaire-mapping.md#buyer-answer-drift) for what the
register stores and how versions are appended.

## Options

| Option | Meaning |
| --- | --- |
| `--register <path>` | Tracked YAML register of reviewed, supplied answer versions (default `trust/answer-register.yml`). |

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | The report was verified (even when answers are affected). |
| `1` | Parser or usage error. |
| `2` | A register or source cannot be verified. |
| `4` | Entitlement failure. |

## Licence

**Paid.** `drift` checks the licence first.

## Example

```powershell
trustbom drift
```

With a usable licence and a tracked `trust/answer-register.yml` it prints the
internal report. Without one it prints nothing but the diagnostic; a build with
licensing not configured printed (real output, exit `4`):

```text
license-invalid: Licensing is not configured for this build: the license service URL and lease signing keys are embedded at build time and cannot be supplied at runtime.
```

A released build without a key prints `license-missing` and instructions.

## Common errors

- **Exit `4`**: a `license-*` diagnostic; see [`license`](license.md#license-enforcement).
- **Exit `2`**: the register or its approved source commit cannot be verified.
  Previously supplied versions are immutable: append a newly reviewed version
  bound to its new approved graph.

## See also

- [Map a buyer questionnaire](../questionnaire-mapping.md)
- [`answer`](answer.md) and [`readiness`](readiness.md)
- [YAML input reference](../yaml-input-reference.md)
