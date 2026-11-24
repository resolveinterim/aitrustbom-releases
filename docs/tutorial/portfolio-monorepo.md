# Several AI systems in one repository

**Why:** the tutorial approved one AI system. Now Abingdon Taxi Solutions has
a second one in the same repository: a dispatch assistant that suggests which
driver to offer a booking. Customers want one answer for the company, and
Trust still has to approve each system on its own evidence.

A **portfolio** is several AI systems under one Trust. It is a way of
organizing the repository, not a plan or a tier: each system keeps its own
record and its own reviewed approval, approving one system never approves
another, and nothing approves the portfolio as a whole. This page assumes you
have finished pages 1 to 7 of the [tutorial](README.md): one system approved
on `main`, and the free trial started.

## Which setup do I need?

| Your systems | Use |
| --- | --- |
| Live in one repository (a monorepo) | This page. |
| Live in separate repositories | Give each repository its own TrustBOM, as in the [tutorial](README.md), and deliver each one's evidence separately. |

Everything stays synthetic and evidence-only, as in the tutorial: the files
below are written for this page and are not in the starter repository.

| Command | License |
| --- | --- |
| `init --portfolio`, `scan --portfolio`, `gate --portfolio`, `propose --system` | None, for any number of systems |
| `pack`, `answer --portfolio` | The organization license (your trial from page 7) |
| `verify` | None; the customer needs no license |

`init --portfolio` needs `trustbom` 1.0.4 or later. If you turned on
enforcement on page 9, GitHub now requires a second person to approve each
merge below.

## Try it

### Part 1: move the service into a portfolio

A portfolio declares each system by a folder, and the folders cannot overlap.
The tutorial's service lives at the root of the repository, so it moves into
`apps/taxi-service/`. Moving the files changes the paths its evidence cites,
so Trust signs the moved system off again.

1. **Start a branch and add the portfolio files.**
   [`trustbom init --portfolio`](../cli/init.md) adds the portfolio
   membership file and workflows, and keeps every file you already have:

   **PowerShell / bash:**

   ```sh
   git switch main
   git pull
   git switch -c go-portfolio
   trustbom init --portfolio
   ```

   Output of `trustbom init --portfolio`:

   ```text
   Initialization plan for <repo>:
     create: trust/portfolio.yml
     preserve: .github/CODEOWNERS
     preserve: .github/rulesets/trustbom.json
     create: .github/workflows/ai-trustbom-portfolio.yml
     create: .github/workflows/portfolio.yml
     preserve: .github/actions/trustbom/action.yml
     preserve: .github/actions/trustbom/install/action.yml
     append: .gitignore
   Warning: .github/rulesets/trustbom.json requires the single-system check "Scan and gate", which a portfolio does not run, so once it is applied no pull request can merge. Replace it with "portfolio / Portfolio approval gate" in that file, then have a repository admin run `trustbom github-rules apply`.
   Created 3 file(s); preserved 4 existing file(s); appended to 1 existing file(s).
   Next: set real tracked system paths, replace CODEOWNERS examples, review the generated portfolio workflow and Action, and review existing protection rules.
   Approval protection: .github/rulesets/trustbom.json holds the ruleset. A repository admin applies it with `trustbom github-rules apply`, or imports it in GitHub (Settings > Rules > Rulesets > Import a ruleset). Until then approvals are advisory.
   ```

   Do what the warning says. The ruleset you made on page 2 requires the
   single-system `Scan and gate` check, and the portfolio workflow reports
   `portfolio / Portfolio approval gate` instead. In
   `.github/rulesets/trustbom.json`, change `"context": "Scan and gate"` to
   `"context": "portfolio / Portfolio approval gate"`. If you applied the
   ruleset on page 9, run `trustbom github-rules apply` again once this
   branch merges.

2. **Move the service.** Make the folders, then move the service's files and
   remove the single-system sign-off files and workflow that the portfolio
   replaces. The old approval stays in Git history:

   **PowerShell / bash:**

   ```sh
   mkdir apps
   mkdir apps/taxi-service
   mkdir apps/taxi-service/trust
   git mv requirements.txt apps/taxi-service/requirements.txt
   git mv config apps/taxi-service/config
   git mv trust/system.yml apps/taxi-service/trust/system.yml
   git mv trust/manual.yml apps/taxi-service/trust/manual.yml
   git mv trust/providers apps/taxi-service/trust/providers
   git rm trust/approvals.yml trust/card.json .github/workflows/ai-trustbom.yml
   ```

   Two files cite paths that just moved. In
   `apps/taxi-service/trust/manual.yml`, change the four `deletion_path` and
   `evidence_path` values from `trust/providers/...` to
   `apps/taxi-service/trust/providers/...`. Do the same for the two
   `dpa_path` values in `trust/subprocessors.yml`. The subprocessor list is
   shared by every system, and each system's record keeps only the providers
   it uses.

3. **Declare the system.** Replace the contents of `trust/portfolio.yml`:

   ```yaml
   org: "Abingdon Taxi Solutions"
   systems:
     - id: taxi-service
       path: apps/taxi-service
       system_yml: apps/taxi-service/trust/system.yml
       manual_yml: apps/taxi-service/trust/manual.yml
   ```

   Replace `.github/CODEOWNERS`, with your own GitHub account as before, so
   that it protects the portfolio file and every system's approval file:

   ```text
   # Replace these example teams with your Trust and platform owners.
   /trust/portfolio.yml @<your-owner>
   /trust/systems/*/approvals.yml @<your-owner>
   /.github/workflows/ai-trustbom-portfolio.yml @<your-owner>
   /.github/workflows/portfolio.yml @<your-owner>
   /.github/actions/trustbom/ @<your-owner>
   ```

4. **Scan the portfolio.** [`trustbom scan --portfolio`](../cli/scan.md) scans
   each system declared in `trust/portfolio.yml` and writes its record under
   `trust/systems/<id>/`. It reads only tracked files, so add everything
   first:

   **PowerShell / bash:**

   ```sh
   git add -A
   trustbom scan --portfolio
   ```

   Output:

   ```text
   Scanned portfolio: Abingdon Taxi Solutions
   Wrote <repo>/trust/systems/taxi-service/card.json
   Wrote <repo>/trust/systems/taxi-service/card.md
   Wrote <repo>/trust/systems/taxi-service/questionnaire.yml
   Wrote <repo>/trust/systems/taxi-service/findings.sarif
   Found 2 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/systems/taxi-service/card.md.
   Status: Not approved — Ready for sign-off
   Next: Run 'trustbom propose --system taxi-service' to ask Trust to sign off.
   ```

5. **Propose that one system.** `--system` names the system; it writes that
   system's own `trust/systems/taxi-service/approvals.yml`. In a portfolio,
   always pass it: without it, `propose` treats the whole repository as one
   system. The two decisions are Trust's, as on page 5:

   **PowerShell / bash:**

   ```sh
   trustbom propose --system taxi-service --high-risk no --article-50 yes
   ```

   Output:

   ```text
   Wrote proposal to trust/systems/taxi-service/approvals.yml (graph <64 hex characters>).
   No changes detected; trust/systems/taxi-service/card.json is up to date.
   Wrote <repo>/trust/systems/taxi-service/card.md
   Wrote <repo>/trust/systems/taxi-service/questionnaire.yml
   Wrote <repo>/trust/systems/taxi-service/findings.sarif
   Found 2 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/systems/taxi-service/card.md.
   Status: Proposed, not approved
   Next: Commit trust/systems/taxi-service/approvals.yml and trust/card.json, open a pull request, and have a CODEOWNER other than you approve and merge it.
   git add trust/systems/taxi-service/approvals.yml trust/systems/taxi-service/card.json
   Commit them and open a pull request; a CODEOWNER other than its author approves and merges it (GitHub enforces this only after `trustbom github-rules apply`). approved_by is a claimed identity; only the merged default-branch graph is Approved and exportable.
   ```

   The `Next:` line names `trust/card.json` generally; the `git add` line
   below it has the system's real paths. You moved many files, so commit
   everything, not only those two.

6. **Commit, open the pull request and merge it as Trust**, as on pages 5
   and 6:

   **PowerShell / bash:**

   ```sh
   git add -A
   git commit -m "Move the service into a portfolio and propose sign-off of taxi-service"
   git push -u origin go-portfolio
   gh pr create --fill
   ```

   Review and merge it on GitHub with **Create a merge commit**. Then:

   **PowerShell / bash:**

   ```sh
   git switch main
   git pull
   trustbom scan --portfolio
   ```

   Output of the scan ends with:

   ```text
   Found 2 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/systems/taxi-service/card.md.
   Status: Approved
   Next: None: the default branch holds Trust's approval of this graph.
   ```

   You now have a portfolio of one approved system. The generated
   `ai-trustbom-portfolio.yml` workflow scans and gates every declared system
   on pull requests; see [Portfolio in one repository](../portfolio-one-repository.md#what-runs).

### Part 2: add a second system

1. **Create the dispatch assistant's folders:**

   **PowerShell / bash:**

   ```sh
   git switch -c add-dispatch
   mkdir apps/dispatch
   mkdir apps/dispatch/config
   mkdir apps/dispatch/trust
   mkdir apps/dispatch/trust/providers
   ```

   Create these five files in your editor. They are synthetic and, like the
   service itself, say what the assistant would use, not that it runs.

   `apps/dispatch/requirements.txt`:

   ```text
   anthropic==0.69.0
   ```

   `apps/dispatch/config/models.yaml`:

   ```yaml
   dispatch_assistant: claude-sonnet-4-5
   ```

   `apps/dispatch/trust/system.yml`:

   ```yaml
   name: Dispatch assistant
   description: Suggests which driver to offer a booking, for a dispatcher to confirm.
   customer_facing_name: Abingdon Dispatch Assistant
   ```

   `apps/dispatch/trust/manual.yml`, the region and data handling Trust
   verified, as for the service on page 4:

   ```yaml
   not_in_scope: []
   models:
     - provider: anthropic
       model_id: claude-sonnet-4-5
       region: eu
       reason: Synthetic example. Trust reviewed the dispatch deployment configuration.
   tools: []
   data_classes: []
   data_handling:
     - provider: anthropic
       retention: 30
       training_use: none
       deletion_path: apps/dispatch/trust/providers/anthropic.md
       evidence_path: apps/dispatch/trust/providers/anthropic.md
       reason: Synthetic example. Trust reviewed the provider terms and deletion procedure.
   ```

   `apps/dispatch/trust/providers/anthropic.md`:

   ```text
   # Anthropic provider review (synthetic)

   SYNTHETIC EXAMPLE. This note stands in for Trust's reviewed summary of the
   provider terms, the signed data processing agreement and the deletion
   procedure. Do not put a confidential contract in a public repository.

   - Prompts and responses are retained for 30 days.
   - Customer data is not used to train provider models.
   - Trust requests deletion through the provider's administrative process and
     records the ticket reference outside this repository.
   ```

2. **Declare the system and its processor.** Add the second entry at the end
   of `trust/portfolio.yml`:

   ```yaml
     - id: dispatch
       path: apps/dispatch
       system_yml: apps/dispatch/trust/system.yml
       manual_yml: apps/dispatch/trust/manual.yml
   ```

   and add the processor at the end of `trust/subprocessors.yml`:

   ```yaml
     - provider: anthropic
       party: Example Dispatch Model Provider Ltd
       purpose: model inference for suggesting drivers for bookings
       location: European Union
       dpa_path: apps/dispatch/trust/providers/anthropic.md
   ```

3. **Scan again.** One system is new; the other did not change:

   **PowerShell / bash:**

   ```sh
   git add -A
   trustbom scan --portfolio
   ```

   Output:

   ```text
   Scanned portfolio: Abingdon Taxi Solutions
   Wrote <repo>/trust/systems/dispatch/card.json
   Wrote <repo>/trust/systems/dispatch/card.md
   Wrote <repo>/trust/systems/dispatch/questionnaire.yml
   Wrote <repo>/trust/systems/dispatch/findings.sarif
   Found 1 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/systems/dispatch/card.md.
   Status: Not approved — Ready for sign-off
   Next: Run 'trustbom propose --system dispatch' to ask Trust to sign off.
   No changes detected; trust/systems/taxi-service/card.json is up to date.
   Wrote <repo>/trust/systems/taxi-service/card.md
   Wrote <repo>/trust/systems/taxi-service/questionnaire.yml
   Wrote <repo>/trust/systems/taxi-service/findings.sarif
   Found 2 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/systems/taxi-service/card.md.
   Status: Approved
   Next: None: the default branch holds Trust's approval of this graph.
   ```

   Adding `dispatch` did not disturb `taxi-service`: its record is unchanged
   and it is still **Approved**.

4. **Run the gate.** [`trustbom gate --portfolio`](../cli/gate.md) judges each
   system against its own approval and prints one pull-request comment per
   system:

   **PowerShell / bash:**

   ```sh
   trustbom gate --portfolio
   ```

   It exits `3`: one of two systems needs approval. The first comment is for
   `dispatch`, the second for `taxi-service`, shortened here:

   ```text
   ## AI TrustBOM portfolio gate: Abingdon Taxi Solutions
   <!-- ai-trustbom-pr-comment -->
   ## AI TrustBOM

   **Status:** Not approved — Ready for sign-off

   **Gate:** bootstrap required

   **Next:** Run `trustbom propose --system dispatch` to ask Trust to sign off.

   ...

   ## AI TrustBOM

   **Status:** Approved

   **Gate:** passed

   **Next:** None: the default branch holds Trust's approval of this graph.

   ...
   ```

   This is what a reviewer sees on the pull request: the gate blocks the
   system that changed and leaves the other alone.

5. **Try to build the pack too early.** `pack` renders the portfolio from
   what is approved on the default branch, and `main` has one system, not
   two. It is a paid command, so it needs the trial from page 7:

   **PowerShell / bash:**

   ```sh
   trustbom pack
   ```

   Output:

   ```text
   Portfolio pack failed: The portfolio declaration differs from the default branch. Run 'trustbom pack' with the approved default-branch portfolio.
   ```

   It exits `2` and writes nothing. `pack` also refuses when a system is not
   approved, so a pack never carries an unsigned system.

6. **Propose the dispatch assistant on its own.** Trust's decisions can differ
   per system. Here the assistant only talks to staff, so Article 50 does not
   apply:

   **PowerShell / bash:**

   ```sh
   trustbom propose --system dispatch --high-risk no --article-50 no
   git add -A
   git commit -m "Add the dispatch assistant and propose sign-off of dispatch"
   git push -u origin add-dispatch
   gh pr create --fill
   ```

   Output of `trustbom propose`:

   ```text
   Wrote proposal to trust/systems/dispatch/approvals.yml (graph <64 hex characters>).
   No changes detected; trust/systems/dispatch/card.json is up to date.
   Wrote <repo>/trust/systems/dispatch/card.md
   Wrote <repo>/trust/systems/dispatch/questionnaire.yml
   Wrote <repo>/trust/systems/dispatch/findings.sarif
   Found 1 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/systems/dispatch/card.md.
   Status: Proposed, not approved
   Next: Commit trust/systems/dispatch/approvals.yml and trust/card.json, open a pull request, and have a CODEOWNER other than you approve and merge it.
   git add trust/systems/dispatch/approvals.yml trust/systems/dispatch/card.json
   Commit them and open a pull request; a CODEOWNER other than its author approves and merges it (GitHub enforces this only after `trustbom github-rules apply`). approved_by is a claimed identity; only the merged default-branch graph is Approved and exportable.
   ```

   This pull request touches only `dispatch`'s approval file, so Trust
   reviews and merges it on its own, with **Create a merge commit**. The
   `taxi-service` approval is a different file with its own history.

### Part 3: one pack, one questionnaire, one check

1. **Update `main` and build the pack:**

   **PowerShell / bash:**

   ```sh
   git switch main
   git pull
   trustbom pack
   ```

   Output of `trustbom pack`:

   ```text
   Packed approved portfolio: Abingdon Taxi Solutions
   Wrote <repo>/export/portfolio/card.md
   Wrote <repo>/export/portfolio/questionnaire.yml
   Wrote <repo>/export/portfolio/systems/dispatch/card.md
   Wrote <repo>/export/portfolio/systems/taxi-service/card.md
   Wrote <repo>/export/portfolio/manifest.json
   ```

   `export/portfolio/card.md` is the cover card for the company, with one
   card per system beside it. Each system's folder also holds its
   `card.json`, which `pack` writes but does not list.

2. **Answer the customer's questionnaire from the whole portfolio:**

   **PowerShell / bash:**

   ```sh
   trustbom answer --questions rfi/request.csv --portfolio
   ```

   Output:

   ```text
   Wrote <repo>/export/answers.csv (5 matched)
   Wrote <repo>/export/unmatched.csv (0 unmatched)
   ```

   Each answer is given per system, for example RFI-02 reads
   `dispatch: anthropic: claude-sonnet-4-5 | taxi-service: gemini: gemini-2.5-flash; openai: gpt-4o-mini`,
   and each cites that system's own files and the commit that approved it.
   RFI-05 is `unknown` for both systems, as on page 7: nothing says whether a
   person reviews the output, and the tool does not guess.

3. **Check it as the customer.** The buyer needs no license and no network:

   **PowerShell / bash:**

   ```sh
   trustbom verify export/portfolio
   ```

   Output:

   ```text
   Verified 6 file(s) plus manifest.json across 2 system(s) against manifest.json.
   Pack verification: OK
   ```

   `export/answers.csv` sits beside the pack, not in it, so the manifest does
   not cover it. With a copy of the repository, `--repo .` also checks that
   each system's approved fingerprint and approved commit match the approvals
   on the default branch:

   **PowerShell / bash:**

   ```sh
   trustbom verify export/portfolio --repo .
   ```

   Output:

   ```text
   Verified 6 file(s) plus manifest.json across 2 system(s) against manifest.json.
   Repository check passed: approved fingerprints and approved graph commits match the default-branch approvals.
   Pack verification: OK
   ```

## What you now have

Two AI systems under one Trust, each with its own record, its own reviewed
approval and its own pull request, and one pack and one set of answers for the
customer. Changing one system blocks only that system's gate. A system that
is not approved on the default branch cannot be packed, and no step approved
the portfolio as a whole: Trust approved each system, one merge at a time.

Next: the [portfolio reference](../portfolio-reference.md) lists the workflow
inputs and tokens.
