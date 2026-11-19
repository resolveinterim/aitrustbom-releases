# 5. Publish

**Goal:** render the approved graph as a CycloneDX ML-BOM and a deployable trust
page, check readiness, and verify the result the way a buyer would.

**You will need:** the approved repository from [page 3](3-approve.md) and the
license from [Get a license](get-a-license.md). `export` and `publish` are paid;
`readiness` and `verify` are free
([which commands are paid](../licensing.md#free-and-paid-commands)). This page
does not depend on [page 4](4-answer.md).

## Try it

1. **Check readiness.** [`trustbom readiness`](../cli/readiness.md#example)
   writes a free, offline checklist of what remains unevidenced for a
   framework (`nist-ai-rmf`, `iso-42001` or `eu-ai-act`):

   **PowerShell / bash:**

   ```sh
   trustbom readiness --framework nist-ai-rmf
   ```

   ```text
   Wrote <repo>/export/readiness/readiness.md
   Wrote <repo>/export/readiness/readiness.json
   Readiness gaps: <count>; assessed: <count>; excluded: <count>
   Wrote <repo>/export/readiness/frameworks/nist-ai-rmf.md
   Wrote <repo>/export/readiness/frameworks/nist-ai-rmf.json
   ```

   The gaps are controls with no evidence in the repository; counts vary with
   the checkout. Each `card.json` also carries a coverage statement of what the
   scan could and could not see; it is not a claim of absence.

2. **Export the CycloneDX ML-BOM.**
   [`trustbom export --format cyclonedx`](../cli/export.md#options) renders
   the same approved graph:

   **PowerShell / bash:**

   ```sh
   trustbom export --format cyclonedx
   ```

   ```text
   Wrote <repo>/export/card.md
   Wrote <repo>/export/questionnaire.yml
   Wrote <repo>/export/trustbom.cdx.json
   Wrote <repo>/export/manifest.json
   Approved graph fingerprint: <12 lowercase hex characters>
   ```

   An export replaces `export/`; if you ran page 4, it also reports removing
   `answers.csv` and `unmatched.csv`.

3. **Publish the trust page.** [`trustbom publish`](../cli/publish.md#example)
   builds a static site that a buyer can open and verify:

   **PowerShell / bash:**

   ```sh
   trustbom publish
   ```

   ```text
   Publishing 1 system page(s) to export/site/ (approved).
   ...
   Wrote <repo>/export/site/index.html
   Wrote <repo>/export/site/index.pdf
   Status: approved
   ```

   `export/site` holds `index.html`, `verify.html`, `index.pdf`,
   `manifest.json` and `downloads/<system>/` (`card.json`,
   `trustbom.cdx.json`, `questionnaire.yml`).

4. **Verify as the buyer.** [`trustbom verify`](../cli/verify.md#example) is
   free and offline. It succeeds only if every file still matches
   `manifest.json`; `--repo .` also confirms the approved fingerprint against
   `trust/approvals.yml` and the default branch:

   **PowerShell / bash:**

   ```sh
   trustbom verify export/site
   trustbom verify export/site --repo .
   ```

   ```text
   Verified 6 file(s) plus manifest.json across 1 system(s) against manifest.json.
   Repository check passed: approved fingerprints and approved graph commits match the default-branch approvals.
   Pack verification: OK
   ```

> **In production.** Host `export/site` on GitHub Pages from the Action, so
> the page changes only when an approval merges: see the
> [trust page guide](../trust-page.md). A buyer who received the files should
> follow [verifying a received pack](../pack-verification.md), including the
> optional Sigstore signature check. Licensing in CI, owner binding and the
> exit codes are in [licensing](../licensing.md#continuous-integration).

## Checkpoint

- `readiness` exited `0` and wrote `export/readiness/`.
- `export/trustbom.cdx.json` exists and `publish` reported `Status: approved`.
- Both `verify` commands exited `0` and ended with `Pack verification: OK`.

Next: [6. Automate](6-automate.md).
