# `trustbom pack`

## Synopsis

```text
trustbom pack [--from <dir>] [--fail-on <severity>]
```

## Purpose

Renders independently approved monorepo or GitHub-verified sibling graphs into
one portfolio pack for buyers: a cover card, a questionnaire and per-system
cards, from independently approved default-branch graphs or verified sibling
artifacts (`--from`). It refuses changed membership or unapproved systems.
Portfolio approval is per system. A portfolio pack carries a `manifest.json`
that a buyer checks with [`verify`](verify.md).

For sibling imports, set a read-only `TRUSTBOM_GITHUB_TOKEN` with access to
each source repository and prepare `portfolio-import.json` as described in the
[portfolio across repositories guide](../portfolio-across-repositories.md). Unverified artifact content is
never an approved buyer claim.

## Options

| Option | Meaning |
| --- | --- |
| `--from <dir>` | Caller-prepared sibling artifacts directory containing `portfolio-import.json`. |
| `--fail-on <severity>` | Minimum finding severity that requires a waiver (default `medium`). |

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Parser or usage error. |
| `2` | Refused or operational failure (changed membership, an unapproved system, unverified artifacts, Git or input failures). |
| `4` | Entitlement failure. |

## License

**Paid**, including `--from`.

## Example

```powershell
trustbom pack
```

With a usable license and every system approved on its default branch it
writes the portfolio pack. Without one nothing is written; a build with
licensing not configured printed (real output, exit `4`):

```text
license-invalid: Licensing is not configured for this build: the license service URL and lease signing keys are embedded at build time and cannot be supplied at runtime.
```

A released build without a key prints `license-missing` and instructions.

## Common errors

- **Exit `4`**: a `license-*` diagnostic; see [`license`](license.md#license-enforcement).
- **A system is unapproved or membership changed**: approve each system with
  [`propose --system <id>`](propose.md) and merge it.
- **Sibling artifacts not verified**: the repository's current default-branch
  approval must be reachable with `TRUSTBOM_GITHUB_TOKEN`.

## See also

- [Portfolio in one repository](../portfolio-one-repository.md)
- [`publish`](publish.md), [`answer`](answer.md), [`verify`](verify.md)
