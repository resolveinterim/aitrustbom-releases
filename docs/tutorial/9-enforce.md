# 9. Enforce, then the next change

**Why:** so far the sign-off has been a convention: you merged your own pull
request. A customer needs to know that the developer who changed the AI could
not also approve it. This page turns on a GitHub rule that makes a second
person's approval a requirement, then makes the next change and watches it
stop.

Rulesets are free on public repositories, which is why your fork is public.
Applying one needs administrator rights on the repository, which you have on
your own fork.

## Try it

1. **Optional: add a second person.** If you have a second GitHub account,
   it can play Trust at the end of this page. Do this before step 2, because
   afterwards every change to `main` needs that second person:
   - In the fork, open **Settings > Collaborators**, invite the second
     account with write access, and accept the invitation from that account.
   - In `.github/CODEOWNERS`, add the second account to the
     `/trust/approvals.yml` line, for example
     `/trust/approvals.yml @<your-owner> @<second-account>`.
   - Commit and push the change to `main`:

     **PowerShell / bash:**

     ```sh
     git add .github/CODEOWNERS
     git commit -m "Add a second Trust reviewer"
     git push origin main
     ```

   Without a second account, you can still do every step; the tutorial ends
   at the blocked pull request, which you can leave open or close.

2. **Turn on enforcement.**
   [`trustbom github-rules apply`](../cli/github-rules.md#apply-the-ruleset)
   uses your `gh` login to create the rule that `init` wrote to
   `.github/rulesets/trustbom.json`. It shows what it will do and asks
   `Apply this ruleset (yes/no) [no]:`. Type `yes`:

   **PowerShell / bash:**

   ```sh
   trustbom github-rules apply
   ```

   The rule, on `main`:
   - every change arrives by pull request, approved by one person other than
     the last person who pushed to it;
   - changes to `trust/approvals.yml` need a code owner's approval;
   - an approval is dismissed when someone pushes again;
   - the **Scan and gate** check must pass;
   - only merge commits, and no force pushes or branch deletion.

3. **Check what a customer's auditor would see.**
   `trustbom github-rules check --github` reads the rule back from GitHub and
   also looks at the history of every sign-off so far:

   **PowerShell / bash:**

   ```sh
   trustbom github-rules check --github
   ```

   Every rule line shows `[verified]`, but the first line reads
   `AI TrustBOM enforcement level: violation` and the command exits `3`. The
   history check names your page 6 sign-off:

   ```text
   Approval history audit: violation (1 approval commit(s) since <8 hex characters>)
     - trust/approvals.yml at <8 hex characters>: pull request #1 lacks an approval of the final push by a CODEOWNER other than the author or last pusher
   ```

   That is correct: you approved your own change. Both short ids are the
   page 6 merge. The rule stops it happening again, and the record stays
   honest about the time it did. The history check looks only at sign-offs
   since the latest approved version, so the level becomes `enforced` once
   the next change is signed off by a second person (step 8).
   [Enforcement levels](../enforcement-levels.md) explains each level.

4. **Make the next change.** The driver feature now needs a driver's
   earnings, from an MCP server. Add it on a new branch and open a pull
   request:

   **PowerShell / bash:**

   ```sh
   git switch main
   git pull
   git switch -c add-earnings-mcp
   cp workshop/mcp-config.example mcp.json
   git add mcp.json
   git commit -m "Connect the driver earnings MCP server"
   git push -u origin add-earnings-mcp
   gh pr create --fill
   ```

5. **Read the pull request comment.** Wait a minute or two, then refresh the\n   pull request page. The check fails, and the comment now compares the
   pull request with the approved version on `main`:

   ```text
   AI TrustBOM

   Status: Not approved (stale) — Needs information

   Gate: failed

   - trust/card.json is stale: the committed graph differs from this pull request's scan.
     Next: Run trustbom scan (or trustbom propose when signing off) and commit trust/card.json to this pull request.

   ...

   What changed

   This change differs from the approved version on the default branch.

   - Added MCP server synthetic-driver-earnings.
     - Evidence: mcp.json:3
   ```

   **Not approved (stale)** means Trust approved an earlier version and this
   pull request changes it. The only item is the out-of-date record: unlike a
   model provider, the MCP server needs no declarations, so there are no gaps
   to fill. The new MCP server is the change, with the line that shows it.

6. **Propose the change.** The change is intended, so follow the comment's
   "when signing off" route. `propose` keeps Trust's previous two decisions
   unless you pass the options again:

   **PowerShell / bash:**

   ```sh
   trustbom propose
   git add trust/approvals.yml trust/card.json
   git commit -m "Propose sign-off of the earnings MCP server"
   git push
   ```

   Output of `trustbom propose`:

   ```text
   Wrote proposal to trust/approvals.yml (graph <64 hex characters>).
   EU AI Act decisions: high_risk no, article_50 yes (change them with --high-risk and --article-50).
   Wrote <repo>/trust/card.json
   Wrote <repo>/trust/card.md
   Wrote <repo>/trust/questionnaire.yml
   Wrote <repo>/trust/findings.sarif
   Found 2 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/card.md.
   Status: Proposed, not approved
   Next: Commit trust/approvals.yml and trust/card.json, open a pull request, and have a CODEOWNER other than you approve and merge it.
   git add trust/approvals.yml trust/card.json
   Commit them and open a pull request; a CODEOWNER other than its author approves and merges it (GitHub enforces this only after `trustbom github-rules apply`). approved_by is a claimed identity; only the merged default-branch graph is Approved and exportable.
   ```

   The comment changes to **Status: Proposed, not approved** and the check
   passes. The `Found` line counts models, tools and findings; the MCP server
   is listed in `trust/card.md`.

7. **Try to merge.** The check passed, but GitHub does not let you merge:
   the pull request needs an approving review, and GitHub does not let you
   approve your own pull request. Being the repository owner does not help,
   because the rule allows no exceptions. This is the point of the page: the
   developer cannot sign off their own change.

8. **Optional: approve as the second person.** Signed in as the second
   account, open the pull request, select **Files changed > Review changes >
   Approve**, then merge it with a merge commit. Back in your clone:

   **PowerShell / bash:**

   ```sh
   git switch main
   git pull
   trustbom scan
   trustbom github-rules check --github
   ```

   The scan reports `Status: Approved` again, now for the version with the MCP
   server. The check now reads `AI TrustBOM enforcement level: enforced`: the
   latest sign-off was approved by someone other than its author, under the
   rule.

## What you now have

A repository where new AI cannot reach `main` until someone other than its
author signs it off, and the record, the decision and the reviewer are all in
Git. That is the evidence behind the answers you give customers, and the
reason you can commit to those answers in a contract.

When you have finished, you can delete the fork (**Settings > General >
Delete this repository**). Your trial stays tied to your GitHub account.

## Where next

- [How you use AI TrustBOM](../workflow.md): the same steps for a real team.
- [How the approval is proven](how-approval-is-proven.md): what the check
  compares, and why the merge counts as the sign-off.
- [GitHub Action](../github-action.md): permissions, fork pull requests and
  upgrades with `trustbom init --upgrade`.
- [Several AI systems in one repository](portfolio-monorepo.md): add a
  second system under the same Trust.
- [Reducing review friction](../reducing-review-friction.md): keeping
  sign-off quick as changes become routine.
- [CLI reference](../cli/README.md): every command.
