# Approved and draft export

`trustbom export` writes the last approved customer-facing pack from the default branch:

```text
export/card.md
export/questionnaire.yml
export/manifest.json
```

`manifest.json` is the integrity manifest that `trustbom verify` checks.

Run it from any branch:

```powershell
trustbom export
```

The command does not export the current branch graph. It resolves the repository's default branch tip, reads `trust/approvals.yml` at that tip, and walks the tip's first-parent history. The approved graph commit is the oldest commit of the contiguous run, ending at the tip, in which both `trust/card.json` and `trust/approvals.yml` carry the approved graph hash; `trust/card.json` is loaded from that commit. The value is derived at read time and never stored. It is a graph baseline, not a decision stamp: it shows that this graph and approval hash have been on the default-branch line since that commit, not who merged them. That depends on the repository ruleset (pull requests only, no bypass, required CODEOWNER review on `trust/**`, no force pushes); [enforcement levels](enforcement-levels.md) explains how that is checked. Reverting to an earlier graph and re-approving it starts a new run, so the baseline is the re-approval commit. The embedded full graph fingerprint must be valid and must match `approved_graph_sha`.

Commit the rescanned `trust/card.json` in the same pull request as the approval: `git add trust/approvals.yml trust/card.json`. If `trust/card.json` is not committed beside the approval, for example because `.gitignore` ignores it, export refuses with "does not match the graph committed beside it". The rendered `trust/card.md`, `trust/questionnaire.yml` and `trust/findings.sarif` stay ignored; export renders its pack from the graph.

Use a different repository-relative approval path when required:

```powershell
trustbom export --from-approvals governance/approvals.yml
```

Use the same finding threshold as the gate when it differs from the default `medium`:

```powershell
trustbom export --fail-on high
```

`--fail-on` at export is the caller's threshold, not part of the approval
contract: the approval file does not record it, and a lower threshold does not
change what Trust reviewed. Use the threshold your approval policy reviewed.

The export is refused when Git cannot prove the default branch (run `git remote set-head origin -a`; a local `main` or `master` alone is not evidence, and a detached pull-request checkout needs `refs/remotes/origin/<base>` fetched with `GITHUB_BASE_REF` naming it), the repository is a shallow clone (fetch full history), the approval is missing or invalid, the default branch tip's graph does not match the approved hash, no baseline commit can be derived within the history limit, the historical graph is empty or invalid or has a stale hash, findings at the configured threshold are not actively waived, a rejection remains, or break-glass is the only passing condition. Refusal does not fall back to the current branch graph.

A successful approved export prints the license in use and its GitHub owner, for
example `License: subscription for GitHub owner acme` or `License: trial for
GitHub owner acme` (never the key). Export replaces the managed files in `export/`; any earlier
managed file that this run does not write again, such as a draft
`trustbom.cdx.json`, is removed and listed on a "Removed earlier export files"
line.

The Markdown shows a short approved graph fingerprint alongside the full
SHA-256 value for copying or independent verification. The questionnaire
identifies the approved graph, commit, approver, approval timestamp, human
high-risk and Article 50 decisions, and relevant finding waivers. Repository
evidence citations remain attached to graph-derived answers. Only the full
fingerprint is compared with `approved_graph_sha`; the short display is never
an approval key.

## Explicit internal draft

Use `--draft` only when an internal artifact from the current checkout is
required:

```powershell
trustbom export --draft
```

Draft mode reads the current checkout's `trust/card.json`. It does not read or
evaluate approvals and cannot be combined with `--from-approvals` or
`--fail-on`. Both artifacts state that they are internal, unapproved, and not
for customer use. The questionnaire also records `approved: false`.

Approved export remains the default. Any approved-export failure returns an
error without invoking draft mode or substituting the current graph.

## CycloneDX export

```powershell
trustbom export --format cyclonedx
trustbom export --draft --format cyclonedx
```

`--format cyclonedx` writes `export/trustbom.cdx.json` (CycloneDX 1.6 JSON) in
addition to `card.md` and `questionnaire.yml`, from the same graph and under
the same approval or draft rules; the approved form is paid, the draft form is
free. The BOM is another renderer: it has no approval of its own and never
feeds back into the graph. It is deterministic: the serial number is derived
from `graph_sha` and the approval state, and nothing in the file is a
timestamp, so the same graph yields the same bytes.

| Graph row | CycloneDX |
|---|---|
| Model | `machine-learning-model` component with provider, model ID, region and evidence occurrences |
| Detected AI SDK package | `library` component with a package URL built from the matched package, version and ecosystem (Maven and Gradle as `pkg:maven`, Go as `pkg:golang` with a `v` version prefix) and evidence occurrences |
| AI provider, declared subprocessor, MCP server, tool | `service` entry; provider services carry the data classes sent to a model and any declared `data_handling`; subprocessors carry the declared location |
| Data-processing dependencies | Not exported; they remain findings until Trust declares the subprocessor |

`trustbom:` properties carry `source` (`manual` or `inferred`) on every row,
`trustbom:status` (`draft` or `approved`), `trustbom:graph_sha` and, only for
an approved export, `trustbom:approved_graph_commit`, the approver and the human
`approved.high_risk` and `approved.article_50` decisions. The scanner
classification appears only as `trustbom:draft.*` values. A package whose
ecosystem has no package URL mapping is listed without a `purl`.

`publish` also writes the BOM for each system as
`export/site/downloads/<system>/trustbom.cdx.json`, listed in the site
`manifest.json` (see [Static trust page](#static-trust-page)). `pack` writes
no BOM: the portfolio pack holds the portfolio card and questionnaire,
per-system card files and its manifest.

The two export artifacts are prepared together and published by replacing the
managed `export/` directory. Validation or write failure cannot leave a newly
mixed card and questionnaire; an existing complete pack is retained until its
replacement is ready.

## Static trust page

`trustbom publish` writes a self-contained, deployable static site under
`export/site/` from the same selected graph. The directory holds `index.html`,
a `verify.html` page explaining how a buyer checks integrity, `index.pdf` (the
same offline pack PDF), a `manifest.json` listing the SHA-256 of every file
(approved publications only), and a per-system `downloads/<system>/` folder
containing `card.json`, the CycloneDX ML-BOM (`trustbom.cdx.json`) and
`questionnaire.yml`. A single-system publication keeps the full detail on
`index.html`; a portfolio uses `index.html` as a system index linking to one
`systems/<system>.html` page each. The HTML is self-contained (inline styles,
no CDN, no scripts) and the PDF needs no browser, so the whole tree can be
served as a static website or zipped and sent to a buyer.

Every page and download is a projection of its cited `card.json`; no file is an
independent source. A buyer verifies an approved site offline with
`trustbom verify export/site`, which needs no AI TrustBOM license and fails on
any missing, modified or added file. See the
[GitHub Pages trust page guide](trust-page.md) to publish the directory as a
website.

When a tracked `trust/portfolio.yml` exists, each declared system must have its
own valid default-branch approval with a derived graph baseline; otherwise publication fails without
replacing an existing site. Use `--fail-on high` if your approval policy uses
that threshold. Both the HTML and PDF must succeed together: if an input
character is not covered by the bundled Lato PDF font, publication fails with
its Unicode code point and retains the previous site rather than silently
omitting text from the PDF.

`trustbom publish --draft` instead uses current-branch graph(s), requires
that each was scanned at the current commit, and visibly marks the entire
pack as **Draft - unapproved - not for buyer use**. A draft carries no
`manifest.json`, so there is nothing to verify and it must not be sent to a
buyer. It never falls back to draft automatically and cannot be combined with
`--fail-on`.

If `export/answers.csv` exists, its table is included as **user-provided
answers**. CSV content is not graph-bound or separately approved; review it
before sharing even when the card itself is approved. The published site
remains intact when `export` or `answer` subsequently rewrites their files.
