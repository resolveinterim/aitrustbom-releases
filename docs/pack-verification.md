# Verifying a received AI TrustBOM pack

When a vendor sends you an AI TrustBOM pack (the `export/`, `export/portfolio/`,
or `export/site/` directory), you can check — offline, with no license and no
account — that the files are internally consistent with the `manifest.json`
shipped beside them and, with `--repo`, that the manifest's approval fields
match the vendor's Git history. Every approved pack ships a `manifest.json`
that lists the SHA-256 of every file plus the approved graph fingerprint,
approved commit, and approval digest for each system.
Each system also carries installed_detector_packs: the detector packs (name and
version) installed in the CLI that produced the pack, not the packs that matched
the repository. The manifest's cli_version is the full build version, and card.md$eolshows the same string. Packs written by earlier builds name the list
detector_packs; erify still reads them.

The manifest is an **integrity** record, not an approval record. Approval lives
in the vendor's `trust/approvals.yml`, merged under CODEOWNER review. Offline
hash checking is **self-consistency only**: anyone who can change a file can
also change the manifest digest that lists it, so the check does not
authenticate who produced the files or prove that the rendered content is what
the vendor approved. For origin, rely on the vendor's Git review evidence or an
independent signature or trusted delivery channel (see step 2).

## 1. Verify the files (free, offline)

Install the CLI with the install script for your platform (see
[installation](installing.md); no Node.js or .NET SDK
needed), or with npm, which needs Node.js:

```powershell
npm install --global trustbom
```

Then point `trustbom verify` at the pack directory you received:

```powershell
trustbom verify path/to/export
```

`verify` reads `manifest.json`, confirms every listed file is present and
matches its recorded SHA-256, and fails if any file was modified, added, or
removed relative to that manifest. It prints the offending path and exits non-zero on failure, and exits
`0` on success. It never touches the network.

If you also have the vendor's repository (or a checkout of it), add `--repo` to
confirm that each system's `approved_graph_sha`, `approved_graph_commit`, and approval
digest in the manifest match the approval file the manifest names. This checks
the manifest's graph and digest fields, not the rendered file content: a file
and its manifest digest changed together still pass the file check.

```powershell
trustbom verify path/to/export --repo path/to/vendor-repo
```

## 2. Optional: verify the Sigstore signature

A vendor may additionally publish a detached [Sigstore](https://www.sigstore.dev/)
bundle for `manifest.json`, named `manifest.json.sigstore`, signed in CI with
GitHub's OIDC identity (keyless — no long-lived private key). If you received
that file, verify it with [cosign](https://docs.sigstore.dev/):

```bash
cosign verify-blob \
  --bundle manifest.json.sigstore \
  --certificate-identity-regexp "^https://github.com/your-org/your-product/" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  manifest.json
```

Replace `your-org/your-product` with the vendor's repository slug. A successful
`cosign verify-blob` confirms that `manifest.json` was signed by a GitHub
Actions workflow in that repository; `trustbom verify` then confirms the pack
files match that manifest.

> Vendors with **private** repositories may choose to omit signing, because the
> public Sigstore transparency log would record the repository identity. This is
> an accepted trade-off: the offline `manifest.json` check in step 1 still shows
> files match the manifest, but without a signature it cannot show who wrote it.

## 3. For vendors: signing the manifest in the Action

Signing is optional and runs in your own release workflow (not the gate Action).
The CLI deliberately bundles no signing or Sigstore library, so you use cosign
directly. The job needs `id-token: write` to request the GitHub OIDC token:

```yaml
name: Sign approved pack

on:
  push:
    branches: [main]

permissions:
  contents: read
  id-token: write

jobs:
  sign:
    name: Sign manifest
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository history
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          fetch-depth: 0

      - name: Export the approved pack
        run: trustbom export

      - name: Install cosign
        uses: sigstore/cosign-installer@v3

      - name: Sign manifest.json
        run: >-
          cosign sign-blob --yes
          --bundle export/manifest.json.sigstore
          export/manifest.json
```

Publish `export/manifest.json.sigstore` alongside the pack. Buyers then follow
steps 1 and 2 above. `trustbom verify` ignores `manifest.json.sigstore` when
checking files, so a signed pack still verifies cleanly.

See the [`verify` reference](./cli/verify.md) for the
`trustbom verify` options and [approved-export.md](./approved-export.md) for how
approved packs are produced.
