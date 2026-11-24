# 5. Propose

**Why:** two questions about the product cannot come from the code: is it a
high-risk AI system under the EU AI Act, and do the Act's Article 50
transparency duties apply? A person has to answer them. Proposing writes
those answers and the exact record into one file that Trust signs off by
merging.

## Try it

1. **Write the proposal.** For this tutorial the decisions are: not high-risk
   (`--high-risk no`), and the transparency duties apply, because drivers talk
   to an AI (`--article-50 yes`). At a terminal, `propose` asks the two
   questions if you leave the options out.
   [`trustbom propose`](../cli/propose.md#example):

   **PowerShell / bash:**

   ```sh
   trustbom propose --high-risk no --article-50 yes
   ```

   Output:

   ```text
   Wrote proposal to trust/approvals.yml (graph <64 hex characters>).
   No changes detected; trust/card.json is up to date.
   Wrote <repo>/trust/card.md
   Wrote <repo>/trust/questionnaire.yml
   Wrote <repo>/trust/findings.sarif
   Found 2 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/card.md.
   Status: Proposed, not approved
   Next: Commit trust/approvals.yml and trust/card.json, open a pull request, and have a CODEOWNER other than you approve and merge it.
   git add trust/approvals.yml trust/card.json
   Commit them and open a pull request; a CODEOWNER other than its author approves and merges it (GitHub enforces this only after `trustbom github-rules apply`). approved_by is a claimed identity; only the merged default-branch graph is Approved and exportable.
   ```

   `trust/approvals.yml` now reads:

   ```yaml
   approved_graph_sha: <64 hex characters>
   approved_by: "<your git user.email>"
   approved_at: <date and time>
   high_risk: no
   article_50: yes
   ```

   The long value identifies this exact record: if anything in it changes
   later, the proposal no longer matches. `approved_by` is the person
   proposing the sign-off, taken from your Git email (or from
   `trustbom propose --by <identity>`), and `approved_at` is when you
   proposed it. Despite their names, neither records an approval, and the
   email is not a verified identity. The approval is the pull request merge
   that Trust makes on GitHub; until then the file only proposes.

2. **Commit and push**, using the `git add` line from the output. The other
   files in `trust/` are either unchanged or ignored by Git, so this is the
   same as the comment's "commit trust/". The pull request is already open,
   so pushing updates it:

   **PowerShell / bash:**

   ```sh
   git add trust/approvals.yml trust/card.json
   git commit -m "Propose sign-off of Gemini and OpenAI"
   git push
   ```

3. **Read the pull request comment again.** Wait a minute or two, then refresh\n   the pull request page. The check now passes, and the comment reads:

   ```text
   AI TrustBOM

   Status: Proposed, not approved

   Gate: passed

   Next: Commit trust/approvals.yml and trust/card.json, open a pull request, and have a CODEOWNER other than you approve and merge it.
   ```

   You have already done the first two parts of that **Next** line. The last
   part, the review and merge, is Trust's job and comes on the next page.

   A passing check means the pull request is ready for Trust, not that it is
   approved. The record is approved only when Trust merges it.

## What you now have

A pull request that says, in one file, what AI the product uses and how Trust
should classify it, with a passing check. It is **Proposed, not approved**: a
proposal on a branch is not a sign-off, so no customer evidence can come
from it yet.

Next: [6. Sign off](6-sign-off.md).
