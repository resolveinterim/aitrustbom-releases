# `trustbom verify`

## Synopsis

```text
trustbom verify [<path>] [--repo <path>]
```

## Purpose

Checks a received pack. `trustbom verify <path>` is **free and fully offline**:
buyers have no license and need none. It reads `manifest.json` from the pack
directory (or a `manifest.json` path directly; the default path is `export`) and
checks that every listed file is present and matches its recorded SHA-256 and
that no undeclared file was added. Modifying, adding, or removing any file makes
verification fail with the offending path and a nonzero exit code; an
untouched pack passes without network access. The success line counts the
listed files and says "plus manifest.json" because the manifest does not hash
itself.

Only approved `export`, `pack`, and `publish` output carries a manifest; a
draft export never writes one, so there is nothing that falsely claims
approval. The manifest is **not** itself an approval: approval stays in
`trust/approvals.yml`, reviewed by an independent CODEOWNER.

Given the repository that produced the pack (or a checkout of it) via `--repo`, verification
additionally confirms that each system's `approved_graph_sha`,
`approved_graph_commit`, and approval digest in the manifest match the approval file
the manifest names, with the baseline re-derived from the repository's default-branch history. See
[pack-verification.md](../pack-verification.md) for buyer instructions and the
optional Sigstore signature.

## Options

| Option or argument | Meaning |
| --- | --- |
| `<path>` | Pack directory or `manifest.json` to verify (default `export`). |
| `--repo <path>` | Optional path to the repository that produced the pack (or a checkout of it) to confirm each system's approved fingerprint and derived approved graph commit match the default-branch approval it names. |

```powershell
trustbom verify
trustbom verify export/portfolio
trustbom verify export --repo .
```

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | The pack verified. |
| `1` | Parser or usage error. |
| `2` | Verification failed (a file changed, was added or removed, or the manifest is missing) or an operational error. |

## Licence

Free and offline. `verify` never checks a licence or the network.

## Example

On a directory that has no manifest (real output; `<repository>` is the
absolute path):

```powershell
trustbom verify
```

```text
Pack verification: FAILED
  - The pack has no manifest.json at '<repository>\export'.
```

Exit code `2`. On an untouched approved pack the success line has the form
`Verified <n> file(s) plus manifest.json across <m> system(s) against manifest.json.`
and the exit code is `0`.

## Common errors

- **No `manifest.json`**: a draft export never writes one; verify an approved
  `export`, `pack` or `publish` output instead.
- **A listed file changed or is missing, or an undeclared file was added**: the
  offending path is printed and the exit code is nonzero.
- **`--repo` mismatch**: the manifest's fingerprint, graph commit or approval
  digest does not match the default-branch approval it names.

## See also

- [Verifying a received pack](../pack-verification.md)
- [`export`](export.md), [`publish`](publish.md), [`pack`](pack.md)
