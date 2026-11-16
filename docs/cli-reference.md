# CLI walkthrough and reference

This is a command-oriented walkthrough of the `trustbom` CLI, from first run
to sending an approved pack to a customer. For a hands-on example with a
cloneable starter, begin with the [Abingdon Taxi Solutions story](tutorial.md). This
reference complements the focused guides for [the GitHub Action](github-action.md) and
[approved export](approved-export.md).

For the end-to-end flow at a glance, see the [workflow overview](workflow.md).

For editable configuration fields, types, constraints and examples, see the
[YAML input schema reference](yaml-input-reference.md).

## 1. Install the CLI

For CI/PR gating, use the Docker-free GitHub Action that `trustbom init`
writes into your repository; it installs the pinned `trustbom` npm package
at run time and needs Node.js, npm and Git, not Docker or the .NET SDK.

The primary route is the installer script for your platform: it installs a
self-contained binary and needs neither Node.js nor the .NET SDK. npm is an
alternative (see below).

The installers take an **explicit** source:

```sh
# Linux/macOS: offline directory containing the matching RID archive and SHA256SUMS
sh scripts/install.sh --release-dir ./artifacts/release --install-dir "$HOME/.local/bin"
# Once a release is published (v<version> is the release tag, for example v1.0.0):
sh scripts/install.sh --base-url "https://github.com/resolveinterim/aitrustbom-releases/releases/download/v<version>"
```

```powershell
# Windows: offline directory containing the win-x64 archive and SHA256SUMS
.\scripts\install.ps1 -ReleaseDir .\artifacts\release -InstallDir "$HOME/.local/bin"
# Once a release is published:
.\scripts\install.ps1 -BaseUrl "https://github.com/resolveinterim/aitrustbom-releases/releases/download/v<version>"
```

`--base-url`/`-BaseUrl` downloads both the matching RID archive and
`SHA256SUMS`; `--release-dir`/`-ReleaseDir` reads both from disk. There is no
default hosted URL: always pass the release URL (or an internal mirror of it)
or a directory. Release archives are hosted as GitHub Release assets in the
public `resolveinterim/aitrustbom-releases` repository, which holds only the
assets, a README and the license text.

Once a release exists, download the installers from that same release page:
`https://github.com/resolveinterim/aitrustbom-releases/releases/download/v<version>/install.sh`
and `.../install.ps1`. The release's `SHA256SUMS` covers the archives only, not
the installer scripts: a checksum served from the same release would add no
trust. Download the script over HTTPS and read it before running it. The
supported RIDs are `win-x64`, `linux-x64`,
`linux-arm64`, `osx-arm64`, and `osx-x64`; the shell installer supports
Linux/macOS and the PowerShell installer supports Windows. Both verify the
archive checksum **before** installing `trustbom` and its `tbom` shim; a
matching checksum protects against accidental corruption, not a malicious
source that replaces both the archive and checksum file. The default
installation directory is `$HOME/.local/bin`; add it to `PATH` if needed.
Git is still required at run time. Linux requires ICU; Alpine/musl is not
supported. To update a script install, rerun the installer with a new
matching archive and `SHA256SUMS` from a trusted release source. To uninstall
that install, remove only `trustbom` and `tbom` on Linux/macOS, or
`trustbom.exe` and `tbom.cmd` on Windows, from the chosen install directory;
do not remove other files there. For npm
use `npm update --global trustbom` / `npm uninstall --global trustbom`.
Homebrew, winget, and other package-manager feeds are not currently available.

**Alternative: npm.** To run `trustbom` without installing it, or to install it
globally, use npm (this needs Node.js and npm, not the .NET SDK):

```
npx trustbom scan
npm install --global trustbom
```

The global install provides `trustbom` and `tbom`. To use an internal npm mirror for either command, set `npm_config_registry` to the
mirror's registry URL in the invoking environment; mirror the exact
`trustbom` version **and its matching platform package**
(`@resolveinterim/trustbom-<rid>`, for example `@resolveinterim/trustbom-linux-x64`). The generated and
root composite Actions run `npm install` with `--ignore-scripts` and honor
the same `npm_config_registry` environment variable on the install step.
Configure that variable in the caller workflow/job environment and provision
registry credentials there as needed; never commit credentials. The Action's
pinned version is not automatically advanced when the CLI is updated.

**Offline and mirrored installs.** Copy the matching archive and `SHA256SUMS`
from the same release to the target machine and use `--release-dir` /
`-ReleaseDir`; no network access is needed. To serve archives internally, copy
the release's files to an HTTPS location and pass that location as the base
URL. npm and the Actions use an internal registry through
`npm_config_registry`, as described above.

**Unsigned binaries (first-run warnings).** The Windows and macOS binaries are
**not code-signed or notarized** at launch. A binary downloaded in a browser
carries a quarantine marker, so:

- **Windows SmartScreen** may show "Windows protected your PC". Choose
  **More info**, then **Run anyway**, or unblock the file first with
  `Unblock-File .\trustbom.exe` in PowerShell.
- **macOS Gatekeeper** may say the binary "cannot be opened because the
  developer cannot be verified". Remove the marker with
  `xattr -d com.apple.quarantine ./trustbom`, or use **System Settings >
  Privacy & Security > Open Anyway**.

The npm, script-installer and Action routes download without a browser, so they
do not trigger these warnings. Always verify the archive against `SHA256SUMS`
from the same release (the installers do this for you).

Run `trustbom license` at any time to read the license terms for the installed
version. It works offline, is never restricted, and also points to the
terms at <https://aitrustbom.com/legal/terms/> and the privacy policy at <https://aitrustbom.com/legal/privacy/>.

Running `trustbom` without arguments or requesting help (including command
help) prints one UTF-8 identity line before the normal output, for example:

```text
AI TrustBOM v1.0.0+build-0042 (x86_64-unknown-linux-gnu)
```

The product tagline, "AI disclosure evidence for your repo", appears in the
top-level help description rather than in the banner.

The version is the build's informational version and the platform describes
the running process. CI builds of `main` reserve increasing identifiers such
as `1.0.0+build-0042`. Local and other unreserved builds use a distinct
`1.0.0-dev.0` identity by default; for an identifiable local build, resolve a
numeric identity from the full Git commit history with
`./scripts/Resolve-Version.ps1` and pass it to `dotnet build` as
`-p:TrustBomBuildVersion=<resolved-version>`. The development number advances
with commits, not with repeated local builds of the same commit; incomplete
Git history is rejected rather than returning a misleading number.
`trustbom --version` prints that same build identity without a banner. Use
`trustbom --no-logo --help` or `trustbom scan --no-logo --help` to suppress
the line in automation. `--no-logo` is accepted before or after the command;
ordinary command output, generated artifacts, and `--version` never include
the line. Without arguments, the existing missing-command exit status
remains unchanged.

The npm packages and self-contained binaries carry the build identity. Standalone builds
without that argument use the unreserved identity; they never take a main CI
number. `trust/card.json` retains the stable three-part `cli_version` and no
build suffix enters approval hashes.

## 2. Initialize a repository

From the root of the repository you want to cover, run:

```powershell
trustbom init
```

`trustbom init` writes the starter `trust/` files (`trust/system.yml`, an empty
`trust/manual.yml` template, an unsigned `trust/approvals.yml` template, and
CODEOWNERS guidance) plus a consumer GitHub Actions workflow. Existing files are
preserved, not overwritten. It also adds an `export/` line to `.gitignore`
(creating the file if needed, appending once and keeping existing content), so a
draft export is not committed by `git add -A`. To commit an export deliberately,
use `git add -f export/` or remove that line. Use `--dry-run` first if you want
to see the plan without creating anything:

```powershell
trustbom init --dry-run
```

`init` and `init --dry-run` also work in a new repository before its first commit;
`scan`, `gate` and `export` still require a commit.

Review the generated files, replace the CODEOWNERS example team names with
your own reviewers, then commit them.

`init` never contacts the license service, never asks for a trial key and keeps
no local first-use state. A trial key is requested later, from the first paid
command or `trustbom license request-trial`, and the 30-day trial starts when
that key is issued. See the
[licensing guide](licensing.md).

### Guided single-system setup

```powershell
trustbom init --interactive --dry-run
trustbom init --interactive
```

The wizard requires non-redirected terminal input/output and an existing Git
repository with a commit. It asks for service name, description,
customer-facing name, Trust `@username` or `@organization/team`, and the
workflow's default branch. Existing identity/reviewer values supply defaults
where they can be read safely. It does not declare model use or control facts.
`--interactive --growth` and existing portfolio repositories are unsupported;
use the [Growth setup guide](growth-workflow.md) instead.

### Upgrading generated files

```powershell
trustbom init --upgrade --dry-run
trustbom init --upgrade
```

Files `init` owns carry a first-line marker
`# trustbom-generated: version=<trustbom version> sha256=<body hash>`. `--upgrade`
covers the single-system and Growth files that exist, prints one line per file
(`upgrade`, `unchanged`, `preserve`, `create`) and rewrites only
files that carry the marker, still match its hash, and were generated by an
older or equal version, so the pinned `trustbom` version in the Action's
install action (`.github/actions/trustbom/install/action.yml`) moves to the
running CLI's version. First install the new exact package version (and its
platform packages) into the registry used by the Action; then run
`init --upgrade`, review and commit the generated changes. An internal
registry can be selected with `npm_config_registry` in the workflow. A missing install action is reported as `create` and
written. Files without the marker,
edited marked files, and files from a newer version are preserved; delete a
marker to take ownership of a file. It never touches `trust/*.yml`,
`approvals.yml` or CODEOWNERS, and it never stages or commits. The run
refuses (exit `2`) on a development build unless `TRUSTBOM_ACTION_VERSION` is set.

There is no migration from earlier Docker-based or private-repository
workflows: no customer repositories used them. Regenerate such a repository
with `init` after removing the old workflow files.

Combinations: `--upgrade` cannot be combined with `--growth` or
`--interactive` (exit `2`, nothing written); `--dry-run` works with `--upgrade`.

The preview distinguishes new files from proposed updates. Existing
`trust/manual.yml` and `trust/approvals.yml` are always preserved; new
approvals remain unsigned. Supported single-line identity YAML can be edited
without removing comments. Custom, tagged, anchored or multiline identity
YAML is preserved with a manual-edit explanation. Selected CODEOWNERS exact
entries for approvals, CODEOWNERS itself and the TrustBOM workflow go last,
overriding earlier wildcard owners for those paths. Review that precedence
change before approving it. If GitHub currently uses root or docs CODEOWNERS,
their existing rules are carried into the new, higher-precedence
`.github/CODEOWNERS`; the original file is not overwritten.

Regenerating `.github/workflows/ai-trustbom.yml` requires a separate explicit
confirmation: it replaces custom jobs, filters, permissions and pins with
the release's tested workflow. Decline to retain your existing workflow.
Every existing-file change is previewed, and final Apply defaults to No.
Changed-since-preview files are rejected. Failed application attempts roll
back this run's changes without intentionally overwriting subsequent edits;
any restoration/cleanup failure identifies files needing inspection.

With consent, an optional installed and authenticated GitHub CLI (`gh`)
performs bounded **GET-only** checks on `github.com`: repository/default
branch, caller identity/admin access, reviewer/team access and visibility,
Actions permissions, base-branch CODEOWNERS errors, legacy branch protection,
and effective rulesets. The wizard never installs/authenticates gh, requests
a token, changes remote settings, or grants access. Missing gh/auth,
permission errors, unsupported origins/GHES and unavailable API data remain
**unverified**, not implicitly configured. Proposed local CODEOWNERS are
not yet the remotely effective base-branch rules.

Local success exits `0` even with outstanding administrator tasks. Check
the reported tasks and settings links: require approving CODEOWNER review,
dismiss stale reviews, and select the actual workflow gate job after its
first run. An observed protection setting is not proof that bypass actors
are constrained. Private organization
repository protection needs a paid GitHub plan; use synthetic public
repositories for GitHub Free enforced-review tests.

Dry-run asks questions and may perform consented remote reads, but writes
nothing and never scans. Declining Apply changes nothing and exits `0`.
Cancellation/EOF, invalid mode/terminal, local file failures and a requested
scan's operational failure exit `2`. Pressing Ctrl+C at a prompt was observed
to exit `130` (the shell's 128 + SIGINT) in a Linux container, with the same
message and nothing written before Apply. A scan failure after setup retains the
completed setup and reports that explicitly.

After applying, the wizard lists untracked setup files and pauses so you
can stage intended inputs in another terminal yourself. It never runs
`git add`. The opt-in scan writes draft artifacts and displays findings;
findings alone do not fail scan. The final Administrator/Engineer/Trust
handoff explains protected approval and the derived approved graph baseline. Local
onboarding never means a graph is approved or ready for buyer export.

## 3. Scan the repository

```powershell
trustbom scan
```

This inspects the tracked lockfiles, manifests, allowlisted configuration,
and `trust/manual.yml`, then writes into `trust/`:

- `trust/card.json` — the deterministic system graph (the source of truth)
- `trust/card.md` — a human-readable system card citing file evidence
- `trust/questionnaire.yml` — fixed-question answers cited to the graph
- `trust/findings.sarif` — findings as SARIF 2.1.0 for code-scanning UIs

The command prints the path of each file it wrote, a short graph fingerprint,
and the full fingerprint to copy into an approval. `card.md` is watermarked `draft (unapproved)` until it is approved (step
5). Findings never fail the scan itself — only the gate enforces policy.

`card.json` records a coverage statement: the detector packs and versions
installed, the ecosystems recognised from tracked manifests and lockfiles (with or
without detector support), `not_in_scope` exclusions with their reasons and
the fixed method limits. `card.md`, the questionnaire (`evidence:` per answer
and a top-level `coverage:` list), the HTML/PDF pack and the portfolio pack
render the same statement, and each citation names its detector pack, rule
and, for a dependency, the package and resolved version. The statement says
what was examined; it never claims complete coverage or absence of AI use.
A change to the coverage statement alone changes `graph_sha` but is
informational in the gate; Trust re-approves the new fingerprint to publish
it in approved exports. Detector pack names and versions are shown but are
outside `graph_sha`, so upgrading the CLI does not by itself require
re-approval; only a changed ecosystem, exclusion or detected row does.

Shipped detector packs cover Python, JavaScript/TypeScript, .NET,
Java/Kotlin (Maven `pom.xml`, Gradle `build.gradle`, `build.gradle.kts` and
`gradle.lockfile`) and Go (`go.mod`, `go.sum`), plus provider HTTP endpoints.
Ruby, PHP and Rust manifests are still listed in the coverage statement as
recognised ecosystems **without** detector support. Orchestration frameworks
(LangChain, LlamaIndex, Semantic Kernel, the Vercel AI SDK, Spring AI and
LangChain4j) are detected as frameworks; a framework alone never invents a
provider or a model.

Dependencies that typically process your data outside the model call —
Pinecone, pgvector, Weaviate, Qdrant, LangSmith, Langfuse and Helicone —
raise a medium `data-processor-undeclared` finding naming the party. The
scanner never creates a subprocessor row from a dependency: declare the party
in `trust/subprocessors.yml` (or waive the finding) and the declared row, with
its DPA evidence, resolves it exactly like `subprocessor-undeclared`.

Two useful options narrow or relocate the scan:

```powershell
trustbom scan --out trust --path "apps/service/**"
```

`--out` changes the output directory (default `trust`); `--path` limits the
tracked-file inventory to a repository-relative glob, useful in a monorepo
where only one application should be covered.

### Declaring what the scanner cannot see

See the [complete `trust/manual.yml` example](manual-declarations.md) for
every supported field, required companion files and allowed values.

Detection is manifest- and pattern-based, not a runtime proxy — it cannot see
prompt bodies, human-in-the-loop UX, or an undeclared deployment region. Add
those facts to `trust/manual.yml` with a `reason` for anything that overrides
an inferred value, and list any deliberately excluded paths (for example
`experiments/**`) under `not_in_scope`. Re-run `trustbom scan` after editing
`trust/manual.yml`; invalid overlay content produces a `manual-invalid`
finding rather than a failed scan.

When an allowlisted tracked config or `.env.example` contains a known provider
API URL but no matching SDK, the scan adds an informational
`provider-endpoint-suspected` finding with the host and file/line. This is
**not** a model declaration. Trust can verify the deployment and add a
reasoned model declaration, for example:

```yaml
models:
  - provider: cohere
    model_id: command-r
    endpoint: https://api.cohere.ai
    reason: Trust verified the direct API deployment.
```

The next scan cites `trust/manual.yml` for the binding and clears the
suspicion for that confirmed host. Generic `transformers` only creates a
Hugging Face binding when an allowlisted model ID and Inference API/endpoint
are both present. Inferred Ollama models carry `self_hosted: true` in
`card.json`; the card labels this scanner classification as a draft for
Trust to verify. A self-hosted model does not require a subprocessor row by
default; Trust can still declare its external hosting provider explicitly.

### Showing CI evaluation results

Commit `trust/evals.yml` with the names and repository-relative paths of
artifacts your CI already produces:

```yaml
evals:
  - name: refund
    artifact: evals/out/summary.json
    threshold: 95
```

A JSON summary at that path can contain
`{"pass":true,"score":97.2,"timestamp":"2026-09-30T11:30:00Z"}`.
`pass` must be a boolean; `score` is an optional percentage from 0 to 100,
and `timestamp` must include a UTC offset. Alternatively, declare a JUnit
`.xml` artifact: `pass` is based on its tests, failures, errors, and skipped
counts; JUnit supplies no numeric score. A JUnit run timestamp is optional;
a timestamp without a timezone cannot establish the run time and appears
as `unknown`.
The optional declaration `threshold` is a 0–100 percentage shown alongside
the artifact result; it does not override the artifact's pass/fail value.

Run `trustbom scan` **after** CI has produced the artifact. Only `evals.yml`
must be git-tracked; the artifact may be an untracked CI output, but must be a
repository-local regular file (at most 1 MiB). The declaration has a 256 KiB
limit and at most 100 entries. The resulting `card.json` contains cited
`eval_results[]`, separate from workflow-presence-only `eval_gates[]`; the
Markdown card links to the declared repository path and shows the last run
from the artifact, or `unknown` if absent. Exported cards retain the path
link but do not embed CI artifacts; keep them available for Trust to review.
An absent artifact yields a `low`-severity `eval-artifact-missing` finding,
not a fabricated result. An unreadable/unsafe/malformed artifact yields
`eval-artifact-invalid`; invalid declarations yield `evals-invalid`. Scanning
still writes a draft card, and the gate applies its usual finding threshold
(default `medium`). Neither scanning nor the Action runs evaluations.
For `scan --portfolio`, put each declaration at
`<system-path>/trust/evals.yml` and its artifacts inside that system's path;
root `trust/evals.yml` is not shared between systems.

### Declaring subprocessors

Commit a repository-root `trust/subprocessors.yml` and the DPA files it
references. Each row uses the graph provider identifier for matching; `party`
is the processor's legal name (for example, `provider: bedrock` with
`party: AWS`):

```yaml
subprocessors:
- provider: bedrock
  party: AWS
  purpose: model inference
  location: Ireland
  dpa_path: trust/dpas/aws.pdf
```

The file is limited to 256 KiB and 100 unique providers. Every `dpa_path`
must be a tracked, repository-relative regular file; paths, YAML syntax,
duplicate keys, and links are validated. A row without a matching model
does not appear on the system card. A detected non-self-hosted provider
without a matching row produces a medium `subprocessor-undeclared` finding;
a detected data-processing dependency without a matching row produces a
medium `data-processor-undeclared` finding; invalid declarations produce
`subprocessors-invalid` (error). A declared row whose `provider` names the
detected party resolves either finding and makes the row visible. Scan still
writes a draft graph, but the gate applies its configured threshold.

The hashed `card.json` stores cited subprocessor rows so approved exports
and buyer CSV answers cannot silently change when the declaration changes.
Q4 includes the model's inference region when known and labels the
processor's location separately, citing both sources; processor location
alone does **not** prove an inference region. Q3 (customer-data training)
remains `unknown`: a DPA index does not establish training use. Trust declares
training use explicitly in `data_handling`, which answers `Q10`. A portfolio
shares the root declaration, includes only matching rows in each system's
approved graph, and lists the approved rows once on its cover. This does not
host or sign legal agreements.

### Declaring data handling

Declare, per AI provider, how long prompts and customer data are retained,
whether that data is used to train provider models, and how deletion is
requested. The section lives in `trust/manual.yml`:

```yaml
data_handling:
  - provider: anthropic
    retention: 30
    training_use: none
    deletion_path: trust/deletion-anthropic.md
    evidence_path: trust/terms/anthropic-dpa.md
    reason: Trust reviewed the enterprise agreement on 2026-10-19.
```

`provider` must match a provider that has a model in the graph; a declaration
for any other provider is dropped and reported as a `data-handling-invalid`
error. `retention` is `none`, `indefinite`, `unknown`, or a whole number of
days from 1 to 36500. `training_use` is `none`, `opt_in`, `yes` or `unknown`
(quote `"yes"` so YAML does not read it as a boolean). `deletion_path` and
`evidence_path` are optional and must be tracked, repository-relative files.
`reason` is required for each row and is validated but never stored in
`card.json`, exactly like the `reason` on a model or tool declaration.

Only declared rows are stored in the graph, always with `source: manual`. A
provider with a model but no declaration is **not** stored; `card.md`, the
questionnaire, the HTML/PDF pack and the portfolio cover render it as
`unknown`, and `trustbom readiness` raises a `disclosure:data-handling` gap.
"unknown" means Trust has not declared the value — it is never a statement
that data is retained, used for training, or safe. AI TrustBOM does not read,
fetch or verify provider terms.

Declared rows are part of the graph hash, so adding, changing or removing one
is a blocking semantic delta: the gate reports `data handling for <provider>`
and the change must be approved before it can be exported. A graph with no
declaration at all hashes exactly as it did before this feature, so existing
approvals stay valid.

The questionnaire renders three canonical questions from these rows:
`Q9_prompt_retention`, `Q10_provider_training_use` and `Q11_deletion_path`.
Existing reviewed answers to other questions are unaffected.

## 4. Evaluate the gate locally

```powershell
trustbom gate
```

`trustbom gate` compares the current graph against the full fingerprint recorded as
`approved_graph_sha` in `trust/approvals.yml` (default path; override with
`--baseline`), and against the graph at the approved graph baseline commit derived
from default-branch first-parent history. When the default branch cannot be proven
(for example a shallow clone, or a detached pull-request checkout with neither
`origin/HEAD` nor a fetched `refs/remotes/origin/<base>` matching `GITHUB_BASE_REF`)
the gate still evaluates the hash, findings, waivers and rejections but reports no baseline diff. It reports, in plain language:

- how many conditions currently block the PR;
- the semantic differences (added/removed models, tools, MCP servers,
  classification changes) between the current graph and the approval;
- each blocking condition and the concrete way to resolve it — correct the
  change, add a reasoned manual declaration, fix the finding, add a
  time-bounded waiver, resolve a rejection, or have Trust approve the graph.

Exit code `0` means the gate passed (or an explicit draft break-glass soft
pass); exit code `3` (`CliExitCodes.GateFailed`) means a policy failure. Use
`--fail-on` to change the minimum finding severity that requires a waiver
(default `medium`):

```powershell
trustbom gate --baseline trust/approvals.yml --fail-on medium
```

The same assessment is appended to `trust/card.md` (without changing
`card.json` or its hash) and printed as the PR-comment Markdown that the
GitHub Action posts — see [github-action.md](github-action.md) for running
this automatically on every pull request.

## 5. Approve a graph in git

Approval is a normal, reviewed git change to `trust/approvals.yml` — there is
no separate UI or button. A Trust reviewer (the human authorized to publish
on behalf of the vendor) edits the file to record the **full** graph fingerprint
shown in the gate's approval example (not the 12-character display prefix),
for example:

```yaml
approved_graph_sha: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
approved_by: trust@example.com
approved_at: 2026-09-23T15:30:00Z
high_risk: unset
article_50: unset
waivers: []
rejections: []
```

`high_risk` and `article_50` are human-only decisions the scanner never
infers (`unset`, or `no`/`yes` and `applies`/`not_applicable` once Trust
decides). Waivers cover specific findings until a `yyyy-MM-dd` date;
non-empty `rejections` always keep the gate failing until resolved. This
change is protected by CODEOWNERS like any other reviewed file, so merging it
is the signature. Anyone with write access can propose the entry, but GitHub
does not let an author approve their own pull request, so a CODEOWNER with the
Trust role who is not the author must approve it; a vendor therefore needs at
least two people able to review. `approved_by` and `approved_at` are unverified
claims; the GitHub review and merge record are authoritative. The CLI records
`approved_at` but does not enforce it (a future timestamp is not rejected); the
merged approval in Git history, not this timestamp, is the authority. Committing
the approval changes `HEAD`, so rescan after committing it before a draft
export. To author the
proposal without copying a fingerprint or timestamp, run:

```text
trustbom request-approval --by trust@example.com
trustbom request-approval --system payments --fail-on high --dry-run
trustbom request-approval --check-controls
```

The command scans tracked inputs without changing scan artifacts, checks
whether the candidate entry would pass the gate, and writes only the selected
approval file. It refuses with exit `2` and no changes for unwaived findings
at the threshold (default `medium`), rejections, break-glass, invalid manual
input or missing identity. A threshold refusal lists the blocking finding ids;
add a waiver for each in the approval file, or choose another `--fail-on`. Without `--by`, it uses `git config user.email`;
neither value verifies identity. `--high-risk yes|no` and
`--article-50 applies|not_applicable` supply explicit human decisions; otherwise
they stay `unset` or retain existing values. `--baseline` chooses a
repository-relative approval file; `--system <id>` selects exactly one
portfolio system. `--dry-run` shows the proposed entry without writing.
An unchanged rerun leaves the approval file byte-identical. `--check-controls`
adds an opt-in read-only advisory GitHub report (verified, missing, unverified,
outstanding), not proof of approval. It includes the recommended required
`Scan and gate` status check and "require branches to be up to date" policy. A missing `gh` or unreadable API is
unverified.
There is no `trustbom approve` command. See
[reducing review friction](reducing-review-friction.md) for keeping this review
step light without weakening it.

Once merged to the default branch, no automation writes approval state: the
approved graph commit is derived at read time from default-branch first-parent
history (the oldest commit of the contiguous run carrying the approved graph
hash). It is a graph baseline, not a decision stamp, and is never stored in
`trust/approvals.yml`. From this
point, `card.md` and the questionnaire are labelled `approved` instead of
`draft` for that derived commit. Approved output is refused when the default
branch cannot be proven (run `git remote set-head origin -a`; a local `main` or
`master` is not evidence, and a fetched `origin/<base>` named by `GITHUB_BASE_REF`
also counts), the clone is shallow, or the tip graph does not match the approval.

## 6. Export a customer-facing pack

```powershell
trustbom export
```

`trustbom export` resolves the repository's default branch, verifies the
approval there, and writes the approved, citation-backed pack:

- `export/card.md`
- `export/questionnaire.yml`
- `export/manifest.json` (integrity manifest; see
  [verifying a received pack](#verify-a-received-pack))

This is always the last **approved** graph, never the current branch's graph.
If the approval is missing, invalid, stale, or blocked by an unwaived
finding or rejection, export refuses rather than silently substituting an
older or draft pack. Use `--from-approvals` or `--fail-on` if your baseline
path or severity threshold differs from the defaults. `--fail-on` at export is
the caller's threshold, not part of the approval contract: a lower threshold
does not change what Trust reviewed.

For an internal-only, explicitly unapproved snapshot of the current branch
(never for customer use), use:

```powershell
trustbom export --draft
```

Every `scan` rewrites the tracked `trust/card.json` and `trust/card.md` (commit
id and assessment) even when the graph is unchanged. `export --draft` and
`publish --draft` need the card's commit id to match `HEAD`, so rescan after
each commit, including the commit that merges an approval, before a draft export.

To also write a CycloneDX 1.6 JSON BOM of the same graph, add
`--format cyclonedx` (`export/trustbom.cdx.json`; `--format markdown` is the
default). The approved form is paid and `export --draft --format cyclonedx`
is free.

See [approved-export.md](approved-export.md) for the full refusal and
watermarking rules and the CycloneDX mapping.

### Publish a static trust page

```powershell
trustbom publish
```

`trustbom publish` renders the same approved graph as a self-contained,
deployable static site under `export/site/`: `index.html`, a `verify.html`
page with buyer verification instructions, `index.pdf`, an approved-only
`manifest.json`, and a per-system `downloads/<system>/` folder holding
`card.json`, the CycloneDX ML-BOM (`trustbom.cdx.json`) and `questionnaire.yml`.
A single-system publication keeps the full detail on `index.html`; a portfolio
uses `index.html` as an index linking to one `systems/<system>.html` page each.
Every page and download is a projection of its cited `card.json`; the pages use
inline styles only, with no CDN, scripts or network access. Zip or serve
`export/site/` to deliver the pack, and a buyer confirms integrity offline with
`trustbom verify export/site`. `trustbom publish --draft` writes the same tree
from the current branch, watermarked and without a `manifest.json`. See
[approved-export.md](approved-export.md) for all rules and
[trust-page.md](trust-page.md) to host the directory with GitHub Pages.


### Verify a received pack

```powershell
trustbom verify
trustbom verify export/portfolio
trustbom verify export --repo .
```

`trustbom verify <path>` is **free and fully offline** — buyers have no
license and need none. It reads `manifest.json` from the pack directory (or a
`manifest.json` path directly; the default path is `export`) and checks that
every listed file is present and matches its recorded SHA-256 and that no
undeclared file was added. Modifying, adding, or removing any file makes
verification fail with the offending path and a nonzero exit code; an
untouched pack passes without network access. The success line counts the
listed files and says "plus manifest.json" because the manifest does not hash
itself.

Only approved `export`, `pack`, and `publish` output carries a manifest; a
draft export never writes one, so there is nothing that falsely claims
approval. The manifest is **not** itself an approval — approval stays in
`trust/approvals.yml`, reviewed by an independent CODEOWNER.

Given the vendor repository (or a checkout of it) via `--repo`, verification
additionally confirms that each system's `approved_graph_sha`,
`approved_graph_commit`, and approval digest in the manifest match the approval file
the manifest names, with the baseline re-derived from the repository's default-branch history. See
[pack-verification.md](pack-verification.md) for buyer instructions and the
optional Sigstore signature.

## 7. Automate it with the GitHub Action

Add the pinned Docker-free Action workflow (generated by `trustbom init`, or see
[github-action.md](github-action.md) for the exact YAML) so that every pull
request is scanned and gated automatically, with a marked PR comment and an
`AI TrustBOM` check. The workflow is read-only and never writes approval state. Fork pull requests are still scanned and gated but never
receive write access to post comments.

## 8. Review advisory evidence readiness

```powershell
trustbom readiness
```

This command writes
`export/readiness/readiness.md` and `export/readiness/readiness.json`. It checks the built-in
canonical disclosure set, separates evidence coverage, approval validity,
unresolved findings, and freshness, and also inspects tracked reviewed-answer
history when `trust/answer-register.yml` is present. Use repeated `--question`
options to narrow the canonical questions, `--portfolio` for per-system
portfolio assessment, and `--max-approval-age-days` /
`--max-answer-review-age-days` to set explicit freshness windows. The report is
advisory only: it distinguishes unknown/unsupported evidence from cited
not-applicable declarations, reports scoped gaps and exclusions without a
generic compliance score, and does not change graph hashes, approval bytes,
gate exit semantics, or buyer-facing approval status. A provider with a model
but no `data_handling` declaration raises a `disclosure:data-handling` gap.

Use repeated `--framework <id>` options to additionally write a free, offline
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

## Summary of commands

| Command | Purpose |
|---|---|
| `trustbom init` | Write preserved-file scaffolding; use `--interactive` for previewed single-system onboarding and optional read-only setup checks |
| `trustbom scan` | Detect AI usage and write `trust/card.json`, `card.md`, `questionnaire.yml`, `findings.sarif` |
| `trustbom gate` | Compare the current graph with the approved baseline and explain what blocks the PR |
| `trustbom request-approval [--check-controls]` | Author an approval-file proposal for independent CODEOWNER review; optionally inspect read-only GitHub controls |
| `trustbom export` | **Paid** (not `--draft`). Write the approved `export/card.md` and `export/questionnaire.yml` (or `--draft` for an internal snapshot); `--format cyclonedx` also writes `export/trustbom.cdx.json` |
| `trustbom publish` | **Paid.** Render a self-contained static trust page under `export/site/` (index, per-system pages, `verify.html`, `index.pdf`, downloads and approved `manifest.json`) from approved graphs, or `--draft` for a watermarked internal pack |
| `trustbom license` | Print the license terms for the installed version; works offline |
| `trustbom license activate --key-stdin [--github-owner <owner>]` | Exchange a license key (read from stdin) for a signed lease bound to the resolved GitHub owner |
| `trustbom license refresh [--github-owner <owner>]` | Renew the license on this computer now, regardless of remaining lease time |
| `trustbom license request-trial --email <email> [--github-owner <owner>]` | Request a free 30-day trial key by email; the service emails the key (never returned) and the 30-day trial starts when the key is issued |
| `trustbom license status [--json]` | Report the cached license state offline (lease end, trial or subscription end, renewal due), without contacting the license service |
| `trustbom readiness` | Write an advisory `export/readiness/` checklist for evidence coverage, approval validity, unresolved findings, and freshness; use `--framework <id>` for free offline framework-alignment overlays (`nist-ai-rmf`, `iso-42001`, `eu-ai-act`) |
| `trustbom init --upgrade [--dry-run]` | Rewrite only unedited marker-carrying generated files to the running CLI's pinned version. Never combined with `--growth` or `--interactive` |
| `trustbom init --growth` | Create a portfolio template, CODEOWNERS guidance, a local reusable Growth workflow and caller, and the pinned per-repository Action without overwriting existing files |
| `trustbom pack [--from artifacts]` | **Paid.** Render independently approved monorepo or GitHub-verified sibling graphs into one portfolio pack |
| `trustbom verify [<path>] [--repo <path>]` | **Free, offline.** Check a received pack against its `manifest.json`; with `--repo`, also confirm approved fingerprints against the named approvals |
| `trustbom answer --portfolio --questions customer.csv` | **Paid** (every `answer` mode). Combine approved per-system answers with separate source citations into one row per buyer question |

`trust/card.json` is always the machine source of truth; every other artifact
is a renderer of it.

### Exit codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | Parser or usage error: unknown command, option or argument value (for example an unknown `--format` or `--migrate`, or a missing required option such as `license request-trial --email`). The usage text is printed and nothing is written |
| `2` | Operational or argument failure detected by the command (invalid `--fail-on`, incompatible option combinations, Git, input or file failures, unmatched answer rows, cancelled interactive setup) |
| `3` | Gate policy failure |
| `4` | Entitlement failure: a paid command's license check failed, or `license status` found no usable lease |

Treat any nonzero code as a failure in automation; parser errors exit `1`
rather than `2` and fail closed.

### Buyer-answer drift

To check buyer-answer drift, run
`trustbom drift [--register <path>]` (a paid command: it checks the license first, see
[the license enforcement section](#license-enforcement)) to compare explicitly supplied, reviewed
answer versions in a tracked YAML register with current graph evidence. The
default register path is `trust/answer-register.yml`. The register stores
identifiers, approved source commits and hashes, semantic field dependencies,
and tracked citations, not buyer question or answer text. The internal report
shows each supplied version, its approved source snapshot, changed facts, and
Trust's review action. Exit code `0` means the report was verified (even when
answers are affected); `2` means a register or source cannot be verified.
Previously supplied versions are immutable: append a newly reviewed version
bound to its new approved graph. The GitHub Action publishes only counts in
its existing marked PR comment, and drift never changes the gate result,
`card.json`, or `approvals.yml`.

### Reviewed workbook reuse

To reuse reviewed workbook answers, run
`trustbom answer --workbook buyer.xlsx --sheet Sheet1 --id-column id
--question-column question --answer-column answer --suggest-only` to write
draft match suggestions against the vendor's reviewed answer register (see
[Buyer-answer drift](#buyer-answer-drift)) to a tracked mapping file (default
`trust/answer-reuse.yml`) and a draft report under `export/`. After Trust
reviews and marks accepted rows `status: reviewed` in Git, rerun the same
command without `--suggest-only` to export a filled copy of the workbook plus
a cited reuse report. Both the suggestions report and export report identify
the source answer version, per-system graph fingerprint and evidence paths;
they remain local and are not posted to PR comments. Unknown or uncited
answers, unverified or affected dependencies, ambiguous matches, and
unreviewed mappings remain unresolved. A new suggestion run replaces older
reuse output rather than retaining a buyer-ready workbook.

For sibling imports, `answer --from artifacts` and
`publish --from artifacts` use the same verified approved snapshots as
`pack --from artifacts`. Set a read-only `TRUSTBOM_GITHUB_TOKEN` with access
to each source repository and prepare `portfolio-import.json` as described
in the [Growth workflow guide](growth-workflow.md). Unverified artifact
content is never an approved buyer claim.

### License enforcement

Five commands are paid and check the license before doing any work, so a
failed check writes and renders nothing: approved `export` (not
`export --draft`), `publish` (including `--draft` and `--from`), `pack`
(including `--from`), `answer` (every mode, including workbook reuse and
`--suggest-only`) and `drift`. Every other command is free and never checks a
license, prompts or contacts the license service: `init`, `scan`, `gate`
(single-system or portfolio), `readiness`, `request-approval`,
`export --draft` and every `license` subcommand. A fresh clone with no
license can run that whole free journey. Argument errors (exit `2`) are
reported before the check and never prompt. See the
[licensing guide](licensing.md) for the trial, purchase, CI and outage rules.

**Exit code `4` (`CliExitCodes.EntitlementFailed`)** means a paid command's
entitlement check failed. The command writes `license-<id>: <message>` to
standard error (standard output stays empty) and one of these diagnostics:
`license-missing`, `license-owner-mismatch`, `license-invalid`,
`license-expired`, `license-revoked` or `license-service-unreachable`. Paid
commands have no `--json` output. A valid result with a warning (the service
is unreachable but a cached lease is still valid) runs the command and prints
the warning on standard error.

**No key yet:** an interactive paid command (standard input not redirected,
not CI) prompts "Enter your license key, or press Enter to request a free
30-day trial". For a trial it shows a privacy notice, asks for confirmation,
your email and GitHub owner (default from the repository origin), and the
emailed key, then continues. A non-interactive run never prompts and fails
with `license-missing` and instructions. In CI set `TRUSTBOM_LICENSE_KEY`: it
takes priority over any stored key, is not persisted and requests a fresh
lease on every run. A valid cached lease with at least 7 days left runs
offline; a lease with fewer days left is refreshed first.

The GitHub owner is resolved from `GITHUB_REPOSITORY_OWNER` in GitHub
Actions, then the `origin` remote; paid commands have no owner option, and an
unresolved owner reports `license-owner-mismatch`.

**License management commands** are never paywalled:

- `trustbom license activate --key-stdin [--github-owner <owner>]` exchanges a
  key read from standard input for a signed, verified lease and stores both for
  the current user at `%USERPROFILE%\.trustbom\license.json` (Windows) or
  `${XDG_CONFIG_HOME:-~/.config}/trustbom/license.json` (elsewhere).
- `trustbom license refresh [--github-owner <owner>]` fetches a fresh lease now.
- `trustbom license request-trial --email <email> [--github-owner <owner>]`
  requests a free 30-day trial key by email. Passing `--email` is the consent:
  the command prints the privacy notice, then sends the email, owner and client
  version to the service, which emails the key and never returns it. A key is
  emailed only if the owner is eligible for a trial, so if no email arrives
  check your spam folder, then contact support or buy a subscription (the
  command prints both). Activate the key when it arrives. If `--github-owner`
  differs from the owner resolved from `GITHUB_REPOSITORY_OWNER` or the
  `origin` remote, the command warns that the option was ignored.
- `trustbom license status [--json]` reports the cached license offline: owner,
  whether it is a trial or a subscription, the lease end (the signed copy stored on this computer, renewed
  automatically; not the subscription end), the trial or subscription end when
  it is known, and whether a renewal is due. Text and `--json` report the same
  facts; the text prints `License: trial` or `License: subscription`, while
  `--json` keeps `kind` with the values `trial` and `paid`. Dates are shown as `YYYY-MM-DD (UTC)`. A subscription that
  outlasts the lease has no end date in the lease (`entitlementExpiresAtUtc` is
  `null`); status says it renews automatically. A trial lease cannot be refreshed past the trial, so for a trial status prints `Trial ends: YYYY-MM-DD (UTC). A trial does not renew.` (the lease end when the lease carries no entitlement end) and never mentions renewal or a purchase receipt. When a trial or subscription has
  ended, status and paid commands use the same wording and point to the buy page
  and `trustbom license activate --key-stdin`.
- `trustbom license` prints the license terms and works offline.

Management commands report failures as a plain message, without `license-*`
identifiers (including `license status --json`). Operational failures exit `2`.
`license status` exits `4` instead when the cached state is not usable (no
lease, a lease for another GitHub owner, a lease that did not verify, a
suspended license, or a trial, subscription or lease that has ended), and `0`
when the lease is valid, including when a renewal is due. Only `activate`,
`refresh`, `request-trial`, the paid-command key prompt and a paid command's
lease request use the network.

Release builds embed the license service URL and the verification keys and never trust a runtime service URL, key or clock override. A build without them reports `license-invalid` from a paid command and "Licensing is not configured" from the management commands.
