# We are Abingdon Taxi Solutions

For this tutorial, **we are an AI-first company providing taxi services**.
Our customers want a better experience for their **drivers** (contract and
earnings questions), **managers** (productivity and absence), and **owners**
(fleet information). They also need to know how we use AI in our service.
Our job is to give them an evidenced, reviewed answer rather than an
unevidenced promise.

We will work in our company's **Git evidence repository**. It contains a
description of the service, a sample customer RFI, and example inputs
for later changes. We only need Git, PowerShell, and an installed
`trustbom` CLI (use the install script for your platform, which needs neither
Node.js nor the .NET SDK, or `npm install --global trustbom` / `npx trustbom`,
which need Node.js and npm; see
[installation](cli-reference.md#1-install-the-cli) and the hosted
[privacy policy](https://aitrustbom.com/legal/privacy/) and
[product page](https://aitrustbom.com/); read the [license](https://aitrustbom.com/legal/terms/) or run `trustbom license`).
There is no application to run and no Python setup. The generated
Docker-free Action uses Node.js and npm to install its pinned package. The
records and customer request are synthetic.

**How to read this exercise:** scanning produces a *draft* inventory
of what tracked files indicate. Our team checks the underlying facts;
Trust approves a graph before we use it to answer a customer. The
starter's staged files let us watch that inventory change without
deploying a taxi service.

The output below is **representative of this starter**. Paths, commits,
graph fingerprints, and CLI versions vary between checkouts; values in
angle brackets are placeholders. JSON excerpts show selected fields
from `card.json`.

We are the **vendor** providing taxi services. Our customer sending the
RFI is the **buyer** of those services. **Trust** is the person at our
company authorized to approve what we share.

## 1. Take stock of our service

Clone the repository and read `service/experience.md`
and `rfi/request.csv`:

```powershell
git clone https://github.com/resolveinterim/abingdon-taxi-solutions.git
cd abingdon-taxi-solutions
trustbom init
```

Expect the CLI to plan five files, then report that it created them:

```text
  create: trust/system.yml
  create: trust/manual.yml
  create: trust/approvals.yml
  create: .github/CODEOWNERS
  create: .github/workflows/ai-trustbom.yml
Created 5 file(s); preserved 0 existing file(s).
```

Alternatively, in a real terminal use `trustbom init --interactive` instead
of plain init to collect service identity and actual Trust reviewers.
`--interactive --dry-run` previews without writing. Optional authenticated
`gh` checks are read-only and require consent; unavailable checks remain
unverified. The wizard preserves existing manual declarations/approvals and
asks before updating identity, CODEOWNERS or the workflow. It offers a draft
scan, but do not scan the workshop inputs yet: first add the `workshop/**`
exclusion below and stage the intended declarations yourself. Decline the
wizard's optional scan and continue this exercise.
See the [guided setup reference](cli-reference.md#guided-single-system-setup)
for rerun and administrator tasks.

Scanning reads Git-tracked files only, so do not Git-ignore `trust/`: ignored
declarations are never read. Commit or stage the generated files before
the first scan.
The service document sets the scene for the driver, manager, and owner
experiences we intend to support. Set the identity in the generated
`trust/system.yml`:

```yaml
name: Abingdon Taxi Solutions service
description: Driver, manager, and owner experience for our taxi customers.
customer_facing_name: Abingdon Taxi Solutions
```

In `trust/manual.yml`, replace `not_in_scope: []` with:

```yaml
not_in_scope:
  - path: workshop/**
    reason: Example inputs are not part of our service.
```

The `workshop/` files are examples to copy at each step. Excluding them
prevents future lesson inputs from appearing as current AI use. Replace
the example CODEOWNERS teams with our actual Trust reviewers before
using approval rules in a real repository. Commit the setup and scan:

```powershell
git add trust .github
git commit -m "Initialize our taxi service AI inventory"
trustbom scan
```

The scan writes four artifacts and prints a short display fingerprint
plus the **full** hash that Trust would review:

```text
Wrote C:\...\abingdon-taxi-solutions\trust\card.json
Wrote C:\...\abingdon-taxi-solutions\trust\card.md
Wrote C:\...\abingdon-taxi-solutions\trust\questionnaire.yml
Wrote C:\...\abingdon-taxi-solutions\trust\findings.sarif
Graph fingerprint: <12 lowercase hex characters>
Full fingerprint (copy to approved_graph_sha): <64 lowercase hex characters>
```

Read `trust/card.json` and `trust/card.md`. Our initial graph has no
models or MCP servers and reports `no-models-detected`. The card and
`trust/questionnaire.yml` render that same graph for human readers.
Relevant fields in our initial `card.json` look like:

```json
{
  "models": [],
  "mcp_servers": [],
  "findings": [
    { "id": "no-models-detected", "severity": "medium" }
  ]
}
```

`trust/card.md` makes that status prominent:

```text
# AI TrustBOM system card
> **draft (unapproved)**
...
- **System:** Abingdon Taxi Solutions service
```

The inventory starts empty because the model inputs are still in
`workshop/`, outside the scope of our first scan.

## 2. Record a proposed Gemini driver channel

Suppose our platform team introduces the Gemini dependency and model
configuration for driver contract and earnings questions. Copy the
example inputs into the tracked paths the scanner recognizes:

```powershell
Copy-Item workshop\gemini-requirements.example requirements.txt
New-Item -ItemType Directory -Force config | Out-Null
Copy-Item workshop\models.example config\models.yaml
git add requirements.txt config\models.yaml
git commit -m "Record the proposed Gemini driver integration"
trustbom scan
```

The CLI writes the same four artifact paths and again prints
`Graph fingerprint: <12 lowercase hex characters>` and its full hash.
The fingerprint **changes** because the model is now in our graph.
For example, one run moved from `a19407a3fddb` to `90b4f7600682`;
your values will differ.
In `trust/card.json`, the new model and findings
look like this (selected fields):

```json
{
  "models": [
    {
      "provider": "gemini",
      "model_id": "gemini-2.5-flash",
      "source": "inferred",
      "evidence": [
        { "path": "config/models.yaml", "start_line": 1 },
        { "path": "requirements.txt", "start_line": 1 }
      ]
    }
  ],
  "findings": [
    { "id": "region-unknown", "severity": "medium" },
    { "id": "subprocessor-undeclared", "severity": "medium" }
  ]
}
```

The two citations link the detected dependency and model identifier
to the exact files our Trust team will review.

The generated `trust/questionnaire.yml` renders that change as
**draft** answers. These are selected entries (Q1 omitted):

```yaml
status: "draft (unapproved)"
graph_sha: "<64 lowercase hex characters>"
questions:
  - id: Q2_models_and_vendors
    question: "Which models and vendors are used?"
    answer: "gemini: gemini-2.5-flash"
    cite:
      - "config/models.yaml"
      - "requirements.txt"
  - id: Q3_is_customer_data_used_to_train
    question: "Is customer data used to train?"
    answer: "unknown"
    cite: []
```

The `region-unknown` and `subprocessor-undeclared` findings give Trust
two concrete questions: where will inference run, and which processor
agreement covers it? Once our team verifies those and any actual
contract or earnings data flow, we can record checked facts with reasons
in `trust/manual.yml`.

## 3. Record a proposed MCP data connection

Our customers also ask how an AI client could access driver earnings.
Introduce a tracked MCP **configuration**:

```powershell
Copy-Item workshop\mcp-config.example mcp.json
git add mcp.json
git commit -m "Record the proposed earnings MCP connection"
trustbom scan
trustbom gate
```

This scan prints another **changed** graph fingerprint (in that same
example run, `48252680a922`) and adds a separate MCP row. The model
from step 2 remains:

```json
{
  "mcp_servers": [
    {
      "name": "synthetic-driver-earnings",
      "command": "driver-earnings-mcp",
      "source": "inferred",
      "evidence": [{ "path": "mcp.json", "start_line": 3 }]
    }
  ]
}
```

The gate then reports a bootstrap failure, including the unresolved
medium findings:

```text
The gate is blocked because this repository has no reviewed graph approval.
...
**Status:** bootstrap required
...
**Draft classification:** role not decided by Trust, high risk not decided by Trust, Article 50 not decided by Trust
...
### Findings
- `medium` **region-unknown**: The deployment region for gemini could not be determined.
- `medium` **subprocessor-undeclared**: Provider 'gemini' has no declared subprocessor.
...
```

This first gate is a bootstrap check; semantic comparisons begin once
Trust has created an approved baseline. The new `mcp_servers[]` row
records the configured command and its citation.
Before approval, our team would verify the server implementation,
permissions, and employee access. The gate collects these review
actions in one place. The policy failure returns exit code `3` for
automation.

**Copilot** is the next driver channel we intend to add. It is a
separate integration rather than automatically a second model binding,
so it appears when we add and verify its own evidence. Manager and owner
AI capabilities will follow the same review path.

## 4. Trust decides what we can tell customers

```powershell
trustbom export --draft
```

The export writes two internal files from the current branch:

```text
Wrote C:\...\abingdon-taxi-solutions\export\card.md
Wrote C:\...\abingdon-taxi-solutions\export\questionnaire.yml
Draft graph fingerprint: <12 lowercase hex characters>
Status: internal draft - unapproved - not for customer use
```

This is our **internal draft**. To publish a customer response in a
real vendor repository, Trust checks the evidence and findings,
records the full graph fingerprint in `trust/approvals.yml`, obtains
CODEOWNER review, and merges it to the default branch. The approved graph commit is then
derived from default-branch history, and a policy-passing graph is eligible
for customer export. See the
[approval walkthrough](cli-reference.md), [Action guide](github-action.md),
and [approved export guide](approved-export.md) for the actual
publication path.

## 5. Answer our customer's RFI

Our customer's procurement team has sent the sample
`rfi/request.csv`
asking about our AI use, providers, training, inference region, and
human oversight. Try an automated answer **before** Trust's approval:

```powershell
trustbom answer --questions rfi\request.csv
```

For this exercise repository, expect a refusal such as:

```text
Answer failed: Could not load the default-branch approval: ...
```

The error explains why there is no `export/answers.csv`. It returns
exit code `2` for automation. **In a real vendor repository**,
after Trust's
approval is reviewed and merged to the default branch:

```powershell
trustbom export
trustbom answer --questions rfi\request.csv
```

**Only after a real approval is merged**, the command can report
`5 matched` and `0 unmatched`:

```text
Wrote C:\...\abingdon-taxi-solutions\export\answers.csv (5 matched)
Wrote C:\...\abingdon-taxi-solutions\export\unmatched.csv (0 unmatched)
```

For example, an approved graph with an unanswered training-data
question would produce this **selected** CSV row:

```csv
"id","question","section","answer","cites","confidence"
"RFI-03","Is customer data used to train?","Data governance","unknown","trust/card.json","low"
```

The approved graph supplies cited rows in `export/answers.csv`
(`id,question,section,answer,cites,confidence`), with unmatched
questions in `export/unmatched.csv`. `unknown` identifies an answer
Trust still needs to resolve. The command writes files for review
rather than emailing the customer. Export **before** answering: a
later `trustbom export` replaces `export/` and removes previous RFI
files.

If a customer asks how drivers, managers, and owners use AI, the
additional sample question in `rfi/extra-question.csv` goes to Trust:
the fixed question bank covers model and policy facts rather than a
description of each role's experience. Once an approved graph exists,
try
`trustbom answer --questions rfi\extra-question.csv`; it reports
`0 matched`, `1 unmatched`, and exit code `2`. The resulting
`export/unmatched.csv` contains:

```csv
"id","text","section"
"RFI-06","How do drivers managers and owners use AI in your service?","Customer experience"
```

Trust must follow up rather than generate a claim from thin air. See
[buyer questionnaire mapping](questionnaire-mapping.md) for CSV
requirements and alias rules.

### Declare data handling, check readiness and publish

Before approval, record what the scanner cannot see. The `data_handling`
declaration in `trust/manual.yml` answers the training and retention
questions with a reason ([format](cli-reference.md#declaring-data-handling)),
and `trustbom readiness --framework nist-ai-rmf` (also `iso-42001`,
`eu-ai-act`) writes a free, offline checklist under `export/readiness/` of
what remains unevidenced. Each `card.json` carries a coverage statement
saying what the scan could and could not see; it is not a claim of absence.
For this starter after step 3, `readiness --framework nist-ai-rmf` reports
`Readiness gaps: 13; assessed: 17; excluded: 2` (counts vary by checkout).

**In a real vendor repository**, after Trust's approval is merged, three more
outputs render the same approved graph:

```powershell
trustbom export --format cyclonedx   # export/trustbom.cdx.json (CycloneDX ML-BOM)
trustbom publish                     # deployable static trust page: export/site/
trustbom verify export/site          # buyer-side, free and offline
```

`publish` writes `index.html`, `verify.html`, `index.pdf`, `manifest.json`
and `downloads/<system>/` (`card.json`, `trustbom.cdx.json`,
`questionnaire.yml`). `verify` reports success only if every file still
matches `manifest.json`; add `--repo .` to confirm the approved fingerprint
against `trust/approvals.yml`. See the [trust page guide](trust-page.md) for
hosting on GitHub Pages and the [pack verification guide](pack-verification.md).
On this exercise repository `publish` cannot succeed because nothing is
approved on a default branch. `publish` (including `--draft`) and approved `export` are paid
commands (a missing license exits `4` with a `license-*` diagnostic);
`export --draft`, `readiness`, `verify`, `scan` and `gate` are free.

## 6. Keep our account accurate on every pull request

The `.github/workflows/ai-trustbom.yml` file created by `trustbom init`
brings this same scan and gate into GitHub. In a repository hosted on
GitHub, we commit the workflow to the default branch, replace the
example `.github/CODEOWNERS` teams with our Trust reviewers, and
configure branch protection to **require the `AI TrustBOM` check and
CODEOWNER review** of `trust/approvals.yml`. The workflow runs the same
engine we used locally.

When findings are resolved (or have reviewed, unexpired waivers), run
`trustbom request-approval --dry-run` to inspect the proposed
`trust/approvals.yml` entry, then `trustbom request-approval` to write it.
The command claims `approved_by` from Git `user.email` unless `--by` is
specified; this value is **not verified identity**. It never writes
approval state or turns the proposal into approval. Optionally run
`trustbom request-approval --check-controls` for read-only GitHub control
observations. A solo Trust CODEOWNER cannot approve their own PR: a different
write-capable CODEOWNER must approve the pull request before merge.

Imagine an engineer changes the Gemini model or adds another MCP
connection. On a same-repository pull request, the Action scans the
proposed checkout, compares its graph with the approved baseline, and
posts one marked PR comment and an annotated `AI TrustBOM` check. A first PR
without an approval shows the bootstrap status we saw locally:

```text
## AI TrustBOM
**Status:** bootstrap required
**Graph fingerprint:** `<12 lowercase hex characters>`
...
### Findings
- `medium` **region-unknown**: The deployment region for gemini could not be determined.
- `medium` **subprocessor-undeclared**: Provider 'gemini' has no declared subprocessor.
```

Once a baseline exists, the comment explains meaningful changes such
as an added model or MCP server and the action Trust needs to take.
An unapproved graph hash, unresolved finding at the configured
threshold, or rejection normally fails the gate. Requiring the
GitHub check makes that failure **block the merge** until the change
is reviewed and the approval is updated. An explicit break-glass
pass remains a draft and cannot authorize a buyer export. Changes
to the wording of `card.md` alone do not create a new graph approval
requirement.

After Trust's protected approval change merges, the approved graph commit
is derived from default-branch history. Our next customer export or RFI
answer selects that approved version.
The [GitHub Action guide](github-action.md) explains the generated
workflow, required permissions, and fork PR behavior. For an
internal npm registry, set `npm_config_registry` in the calling workflow's
environment and mirror the pinned `trustbom` and platform packages. After
installing a newer CLI version, run `trustbom init --upgrade --dry-run`,
then `trustbom init --upgrade` to update only unedited generated files,
including the Action's version pin; review and commit the change. This hosted
journey needs the pinned `trustbom` package to be available from npm or from
your internal registry.

## Why this matters

We can give our customers an AI account with **repository citations, a
human decision, and a specific approved version**. Engineering sees
changes while reviewing code rather than discovering them during the
next RFI. Trust reviews the same graph that generates the customer
card and answers, and GitHub keeps that decision tied to the code it
actually covers.

We can put **the same control across our portfolio of code
repositories**: install the pinned workflow and require its check and
Trust review in each repository. Each service keeps its own evidence
and approval history in Git, while our organization applies a
consistent rule to every PR. For several systems **within one
repository**, `trustbom scan --portfolio`, `trustbom gate --portfolio`,
and `trustbom pack` provide per-system approvals and a combined pack
from independently approved systems.
