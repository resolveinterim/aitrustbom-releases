# Publish the trust page with GitHub Pages

`trustbom publish` renders a self-contained static site under `export/site/`
(see [approved-export.md](approved-export.md)). Because it is a plain directory
of HTML, a PDF, an integrity `manifest.json` and per-system downloads — inline
styles only, no scripts and no network access — you can host it anywhere that
serves static files. This guide shows the simplest secure route: GitHub Pages.

## How the trust boundary stays in Git

Keep the approval boundary where the rest of the product keeps it — in Git, not
in a hosted UI. The recommended flow is:

1. Trust runs `trustbom publish` locally (or on the default branch) once the
   graph is approved and merged to the default branch. Verification is free and offline, but
   **publishing is a paid command.**
2. Commit the generated `export/site/` directory on the default branch, so the
   exact bytes a buyer sees are reviewable and versioned like code.
3. A Pages workflow verifies the committed directory offline and deploys it.

The workflow never approves anything and needs no AI TrustBOM license: it only
runs the free, offline `trustbom verify` and serves bytes that were already
approved and committed.

## Workflow

Add a workflow such as `.github/workflows/trust-page.yml`. Pin each third-party
action to a full commit SHA, exactly as the generated AI TrustBOM Action does
(see [github-action.md](github-action.md)); the tag comments below show which
version each SHA must match.

```yaml
name: Publish AI TrustBOM trust page

on:
  push:
    branches: [main]
    paths:
      - 'export/site/**'
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Check out repository
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: Set up Node.js
        uses: actions/setup-node@v4 # pin to a commit SHA
        with:
          node-version: 'lts/*'

      - name: Verify the approved trust page offline
        run: |
          npm install --global trustbom
          trustbom verify export/site

      - name: Configure Pages
        uses: actions/configure-pages@v5 # pin to a commit SHA

      - name: Upload the trust page
        uses: actions/upload-pages-artifact@v3 # pin to a commit SHA
        with:
          path: export/site

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4 # pin to a commit SHA
```

- `permissions` grants only what Pages needs: `pages: write` to publish and
  `id-token: write` so `deploy-pages` can mint the OIDC token Pages requires.
  `contents: read` is enough because the approved bytes are already committed.
- The `path: export/site` upload serves the published directory as the site
  root, so `index.html`, `verify.html`, each `systems/<system>.html` page and
  the `downloads/` artifacts resolve with their relative links intact.
- The `trustbom verify export/site` step is an offline integrity gate: if any
  published file was modified, added or removed after approval, the run fails
  before anything is deployed.
- Enable Pages for the repository with **Build and deployment → Source → GitHub
  Actions**. The deployed URL appears as the job's `page_url`, for example
  `https://your-org.github.io/your-repo/`.

## Verification stays available to buyers

Hosting the directory does not replace verification. Because `manifest.json`
and the downloads ship inside the site, a buyer can download the directory and
run `trustbom verify .` offline, or confirm an optional Sigstore signature with
`cosign verify-blob` and
`--certificate-oidc-issuer "https://token.actions.githubusercontent.com"`, as
described in [pack-verification.md](pack-verification.md). The manifest is an
integrity record, not an approval: approval stays in `trust/approvals.yml`,
reviewed by an independent CODEOWNER.

A `trustbom publish --draft` directory carries no `manifest.json` and is
watermarked **Draft - unapproved - not for buyer use**; never deploy a draft to
a buyer-facing site.
