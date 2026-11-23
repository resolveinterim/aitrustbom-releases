# How you use AI TrustBOM

**A customer asked what AI you use. Answer with evidence a named person
approved.**

AI TrustBOM keeps an approved record of the AI in your product, and stops
new AI from shipping until Trust signs it off. Day to day, the pull request
is where you meet it. Running `trustbom` locally is optional. The
[tutorial](tutorial/README.md) walks through all of this hands-on, and the
[CLI reference](cli/README.md) has every command and option.

1. **Open a pull request as usual.** If it adds or changes AI, the check
   fails and the comment lists what changed and what is missing.
2. **Fill the gaps and propose.** Follow the comment's steps, run
   `trustbom propose`, and push.
3. **Trust signs off.** Trust reviews the pull request and merges it. The
   record is now approved, and you can hand it to customers.

For a graphical overview of the flow, including the loops back when a check
or review needs changes, see the [workflow diagram](workflow-diagram.md).

## Who is involved

**Developers** open pull requests as usual and follow the comment when a
change touches AI. They never need to look up flags or install routes; the
comment gives the exact commands.

**VP Engineering** owns the setup: the workflow, the reviewers in
`.github/CODEOWNERS` and, when sign-off must be enforced, the repository
rules. In return there is one cited record per product instead of a
spreadsheet rebuilt for every customer.

**Trust** is an existing team member authorized to decide what the company
tells customers. Trust reviews the record and merges the pull request. No
scan or automation makes that decision. Customers and auditors receive the
approved pack and can check it themselves.

## What it can and cannot see

AI TrustBOM reads files tracked in Git. It does not run your code, read
your source code's syntax or watch network traffic.

- **It can see** AI SDKs and frameworks in manifests and lockfiles (Python,
  JavaScript/TypeScript, .NET, Java/Kotlin and Go), model IDs and provider
  endpoints in allowlisted configuration files such as `config/models.yaml`
  and `.env.example`, MCP server configuration, and evaluation workflows.
  Each detected item cites the file and line it came from.
- **It cannot see** raw HTTP calls to an AI API from your code, providers
  it has no detector for, vendored or copied-in code, untracked files, or
  configuration that exists only at runtime (deployment environment
  variables, secret stores, feature flags). It also cannot see facts that
  are not in code at all, such as the deployment region, retention terms or
  whether a human reviews the AI's output.

Declare what it cannot see in `trust/manual.yml`, with a reason; see
[manual declarations](manual-declarations.md). Anything neither detected
nor declared stays **unknown**. AI TrustBOM never turns missing evidence
into "no".

## 1. Set up once

- Install the `trustbom` command-line tool; see [install](installing.md).
- Run `trustbom init` in your repository. It adds the `trust/` files, the
  reviewer rules in `.github/CODEOWNERS` and the pull-request workflow.
  Replace the example reviewer teams with your Trust reviewers, commit and
  push. See [initialize](cli/init.md).
- Listing reviewers does not by itself stop a merge. To make Trust's
  sign-off required, an administrator applies the repository rules; see
  [GitHub rules](cli/github-rules.md) and
  [enforcement levels](enforcement-levels.md).
- Scanning, the pull-request check and `export --draft` are free. Start the
  30-day trial or a subscription when you need paid commands such as
  approved export; see [licensing](licensing.md).

## 2. Open a pull request

- Work as usual. When a pull request adds or changes AI, the workflow's
  check fails and its comment lists:
  - **what changed**, with the file and line for each new, removed or
    changed item;
  - **what is missing**, each with a `Next:` step.
- When nothing is missing, the comment says **Ready for sign-off** and
  gives the exact install line and `trustbom propose` command.
- The check also fails when the committed `trust/card.json` is out of date;
  run `trustbom scan` (or `trustbom propose`) and commit it.

See [the GitHub Action](github-action.md).

## 3. Fill the gaps

- Edit `trust/manual.yml` to declare what the repository cannot show:
  - AI models or tools the scan can't detect;
  - deployment regions;
  - how each provider handles data, such as retention, training use and
    deletion;
  - your own judgements, with reasons.
- List third-party AI providers in `trust/subprocessors.yml`.
- Your declarations take precedence over scanned values and are marked as
  manual, with their source kept. See
  [manual declarations](manual-declarations.md).
- A finding that cannot be fixed now can get a time-limited waiver with a
  reason, which Trust reviews with the rest of the sign-off. When it
  expires, the finding blocks again.
- Commit and push. The comment updates on each push.

## 4. Propose

- When the comment says **Ready for sign-off**, run the `trustbom propose`
  line it gives. The first sign-off needs two human decisions,
  `--high-risk yes|no` and `--article-50 yes|no`; later sign-offs carry them
  forward. See [propose](cli/propose.md#approve-a-graph-in-git).
- Commit what it prints, `git add trust/approvals.yml trust/card.json`, and
  push. The comment now says **Proposed, not approved**.
- The `approved_by` and `approved_at` values in `trust/approvals.yml` are
  declared, not proven. GitHub's review and merge history is the evidence of
  who signed off.

## 5. Trust signs off

- Trust, someone other than the author, reviews the exact record and the
  proposal in the pull request, then approves and merges it. If Trust asks
  for changes, go back to step 3.
- Once merged into the default branch, the record is **Approved**. How
  `export` proves that from Git history is described in
  [how the approval is proven](tutorial/how-approval-is-proven.md).

## 6. Share with customers

- Paid `trustbom export` writes the approved record: summary, questionnaire
  answers and an integrity manifest. Add `--format cyclonedx` for a
  CycloneDX file. `export --draft` is free and watermarked as unapproved.
- Paid `trustbom answer` maps a customer's questionnaire to cited answers
  and lists the questions that need a person.
- Paid `trustbom publish` produces a self-contained HTML and PDF pack.
- Optionally, `trustbom readiness` gives an advisory view of evidence gaps.
- Packs already sent do not update when the record changes.

See [export](cli/export.md), [approved export](approved-export.md),
[questionnaire mapping](questionnaire-mapping.md), [trust page](trust-page.md)
and [readiness](cli/readiness.md).

## 7. The customer verifies

- Whoever receives your pack can run `trustbom verify`. It's free and works
  offline; it checks that the files match the supplied manifest. By itself,
  this does not authenticate origin or prove that the pack was approved.
  `--repo` can compare the manifest's approval fields with the vendor's Git
  history; origin still requires independent review evidence, a trusted
  delivery channel or a verified signature.

See [pack verification](pack-verification.md).

## Checking locally (optional)

You can run the same checks before pushing:

- `trustbom scan` writes the draft record and reports what is missing. It
  needs no setup, so it also works before `init`. See [scan](cli/scan.md).
- `trustbom gate` explains in plain language what would fail the
  pull-request check and how to resolve it. See [gate](cli/gate.md).
