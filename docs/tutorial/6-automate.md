# 6. Automate

**Goal:** see how the generated GitHub Action brings the same scan and gate to
every pull request, and what an enforced repository adds.

**You will need:** the repository from [page 1](1-setup.md) onward. This page
does not depend on pages 4 or 5. Only the first two steps run locally; the
rest needs a repository hosted on GitHub, so it is described rather than
scripted.

## Try it

1. **Read the generated workflow.** `trustbom init` created
   `.github/workflows/ai-trustbom.yml` and the composite actions under
   `.github/actions/trustbom/`. They run the same engine you used locally, and
   adding them needs no administrator rights. See the
   [Action guide](../github-action.md) for permissions and fork pull requests.

2. **Preview an upgrade.** After installing a newer CLI, `init --upgrade`
   updates only unedited generated files, including the Action's version pin.
   The dry run changes nothing:

   **PowerShell / bash:**

   ```sh
   trustbom init --upgrade --dry-run
   ```

   ```text
   Upgrade plan for <repo>:
     unchanged: .github/workflows/ai-trustbom.yml (already current (<version>))
     unchanged: .github/actions/trustbom/action.yml (already current (<version>))
     unchanged: .github/actions/trustbom/install/action.yml (already current (<version>))
   Dry run complete; no files were modified.
   ```

   See [upgrading generated files](../cli/init.md#upgrading-generated-files).

> **In production.** Everything below needs a repository on GitHub and is not
> part of the local exercise.
>
> 1. **Commit the workflow** to the default branch and replace the example
>    `.github/CODEOWNERS` teams with your Trust reviewers.
> 2. **Pull requests.** Imagine an engineer changes the Gemini model or adds
>    another MCP connection. On a same-repository pull request, the Action scans
>    the proposed checkout, compares its graph with the approved baseline and
>    posts one marked PR comment and an annotated `AI TrustBOM` check. The
>    first PR on a repository without an approval shows the bootstrap status
>    you saw on page 3. Once a baseline exists, the comment explains changes such
>    as an added model or MCP server and the action Trust needs to take.
> 3. **Blocking.** An unapproved graph hash, an unresolved finding at the
>    threshold, or a rejection fails the gate. Requiring the `AI TrustBOM`
>    check makes that failure **block the merge** until the change is reviewed
>    and the approval is updated. An explicit break-glass pass remains a draft
>    and can never authorize a buyer export. Changes to the wording of
>    `card.md` alone do not need a new approval.
> 4. **Enforcement level.** Run
>    [`trustbom github-rules check --github`](../cli/github-rules.md#check-the-enforcement-level)
>    to see the level. An administrator makes it `enforced` once with
>    [`trustbom github-rules apply`](../cli/github-rules.md#apply-the-ruleset);
>    until then the history audit reports `audited`. See
>    [enforcement levels](../enforcement-levels.md).
> 5. **Next export.** After Trust's protected approval change merges, the next
>    customer export or RFI answer selects that approved version.
>
> Operations: an internal npm registry or offline install is covered in
> [offline and mirrored installs](../installing.md#offline-and-mirrored-installs)
> and the [Action guide](../github-action.md#generated-consumer-workflow); the
> license in CI and its exit codes in [licensing](../licensing.md#continuous-integration);
> several systems in one repository with
> [`trustbom scan --portfolio`](../cli/scan.md#options),
> [`trustbom gate --portfolio`](../cli/gate.md#options) and
> [`trustbom pack`](../cli/pack.md#example), which give per-system approvals and
> a combined pack from independently approved systems. To reduce reviewer
> effort, see [reducing review friction](../reducing-review-friction.md).

## Checkpoint

- `trustbom init --upgrade --dry-run` listed the three generated files as
  `unchanged` and modified nothing.
- You can say what changes when the Action runs: a marked PR comment, a check
  that can block the merge, and an approval that stays in Git.

## Why this matters

We can give our customers an AI account with **repository citations, a human
decision and a specific approved version**. Engineering sees changes while
reviewing code rather than discovering them during the next RFI. Trust reviews
the same graph that generates the customer card and answers, and Git keeps that
decision tied to the code it covers.

The same control extends **across a portfolio of repositories**: install the
pinned workflow and require its check and Trust review in each repository. Each
service keeps its own evidence and approval history in Git, while the
organization applies one rule to every pull request.

You have reached the end of the tutorial. The [CLI reference](../cli/README.md)
has one entry per command, and the [guides index](../README.md) lists the rest.
