# Portfolio across repositories

A **portfolio** is several AI systems under one Trust. Here each system lives in
its own repository with its own `trust/card.json` and approval, and a caller job
imports their approved cards into one buyer pack.

> **Not recommended.** This setup has not been run end to end. For systems in
> separate repositories, give each repository its own TrustBOM, as in the
> [tutorial](tutorial/README.md), and deliver each one's evidence separately.
> This page stays as a reference for the existing interface.

## Which setup do I need?

| Your systems | Use |
| --- | --- |
| Live in one repository (a monorepo) | [Portfolio in one repository](portfolio-one-repository.md). |
| Live in separate repositories | Each repository's own TrustBOM, as in the [tutorial](tutorial/README.md). |

The two setups are separate: a monorepo pack and an imported pack cannot be
combined. `--from` selects the imported systems for the buyer bundle; it is
**not** a union with local monorepo systems. Use separate calls to publish a
monorepo pack and an imported pack. The local portfolio's gate remains a
prerequisite. The [portfolio reference](portfolio-reference.md) lists every
input, token and interface.

## How it works

The imported mode requires an explicitly named `imports-artifact` uploaded by a
caller job in the **same run**. It contains a bounded `portfolio-import.json`
and its referenced sibling cards. The reusable workflow downloads that exact
named artifact; it does not discover or download cross-repository artifacts on
its own.

The caller prepares a directory (for example `prepared-sibling-bundle/`) in its
own trusted job, uploads it with the pinned
`actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`, and passes
that artifact's exact name as `imports-artifact` to a portfolio job that
`needs` the preparation job. Map the separate secret on that caller job:

```yaml
secrets:
  sibling-read-token: ${{ secrets.TRUSTBOM_SIBLING_READ_TOKEN }}
```

The directory must contain `portfolio-import.json` and every referenced
`artifact_path`, for example:

```json
{
  "organization": "Example company",
  "systems": [
    {
      "id": "assist",
      "repository": "example/assist",
      "source_commit": "<40-character derived approved graph commit>",
      "artifact_path": "assist/card.json",
      "graph_path": "trust/card.json",
      "approval_path": "trust/approvals.yml"
    }
  ]
}
```

This manifest identifies inputs for verification, **not** an approval record.
The verifier refuses wrong hashes, a source commit that differs from the derived
baseline, moved default-branch tips, missing files, partial collections and
inaccessible repositories. Never use untrusted network downloads as an implicit
preparation step.

## Token

Supply a separate `sibling-read-token` reusable-workflow secret: a read-only
token scoped to `contents: read` on **every** sibling source repository (for
example an appropriately scoped GitHub App installation token or fine-grained
PAT). The run's `github.token` is not substituted for it. The approved
default-branch step maps it to `TRUSTBOM_GITHUB_TOKEN` only when verifying
imports; the Action separately receives `GITHUB_TOKEN` for pull-request
publication, and neither token is passed to the scanner. A missing or
insufficient token fails closed.

Verification reads each source repository's current default-branch approval and
derived approved graph; the artifact's approval metadata alone is never
authority. The current verifier calls `api.github.com` and does not support a
custom GitHub Enterprise API URL. If the token cannot read the source
repositories, verification fails; do not substitute unverified artifacts.

## Licensing

Gating is free. Delivery (`pack`, `publish`) and answering (`answer`) use the
organization license, as in [one repository](portfolio-one-repository.md#licensing).
