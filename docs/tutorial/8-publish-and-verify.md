# 8. Publish and verify

**Why:** a customer should not have to take our word for it. Publishing
builds a trust page and a set of downloads with a list of their checksums, so
the customer can check that nothing was changed after Trust approved it.

## Try it

1. **Publish.** [`trustbom publish`](../cli/publish.md#example) builds a static
   site in `export/site/` from the approved record:

   **PowerShell / bash:**

   ```sh
   trustbom publish
   ```

   Output:

   ```text
   Publishing 1 system page(s) to export/site/ (approved).
   The static trust page will expose:
     export/site/index.html, export/site/verify.html, export/site/index.pdf
     downloads: export/site/downloads/abingdon-taxi-solutions/card.json, trustbom.cdx.json, questionnaire.yml
   Exposes 3 evidence download file(s) across 1 system(s).
   export/site/manifest.json lists the SHA-256 of every file; buyers run 'trustbom verify export/site' offline.
   Includes export/answers.csv as user-provided answers (as supplied, not graph-approved); review before sending.
   Wrote <repo>/export/site/index.html
   Wrote <repo>/export/site/index.pdf
   Status: approved
   ```

   Open `export/site/index.html` in a browser. It shows the approved record,
   with a PDF copy and three downloads: the record (`card.json`), a CycloneDX
   file for AI-BOM tools (`trustbom.cdx.json`) and the standard questionnaire
   (`questionnaire.yml`). `verify.html` tells a customer how to check the
   files. The page also lists your answers from page 7, in a
   section marked as supplied by you rather than approved, because a person
   may have edited them after `answer` wrote them.

2. **Verify as the customer.** [`trustbom verify`](../cli/verify.md#example) is
   free and works offline. It passes only if every file still matches the
   list of checksums:

   **PowerShell / bash:**

   ```sh
   trustbom verify export/site
   ```

   Output:

   ```text
   Verified 6 file(s) plus manifest.json across 1 system(s) against manifest.json.
   Pack verification: OK
   ```

   The six files are `index.html`, `verify.html`, `index.pdf` and the three
   downloads; your answers are inside `index.html`. Change any file in
   `export/site/` and run it again: it reports
   `Pack verification: FAILED` and names the modified file. Run
   `trustbom publish` again to rebuild the site.

3. **Verify against the repository.** With access to the repository, `--repo`
   also checks that the published record is the one approved on `main`:

   **PowerShell / bash:**

   ```sh
   trustbom verify export/site --repo .
   ```

   Output:

   ```text
   Verified 6 file(s) plus manifest.json across 1 system(s) against manifest.json.
   Repository check passed: approved fingerprints and approved graph commits match the default-branch approvals.
   Pack verification: OK
   ```

## What you now have

Evidence you can send or host: an approved record, a CycloneDX file, the
questionnaire answers and a trust page, which the customer can check without
trusting us or contacting anyone. [Verifying a received pack](../pack-verification.md)
is the page to send them.

> **In production.** Host `export/site/` with GitHub Pages so that the page
> changes only when a sign-off merges: see the [trust page guide](../trust-page.md).
> The [readiness checklist](../cli/readiness.md#example) lists what is still
> unevidenced for a framework such as the EU AI Act.

Next: [9. Enforce, then the next change](9-enforce.md).
