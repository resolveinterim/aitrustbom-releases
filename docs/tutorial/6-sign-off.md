# 6. Sign off

> **You are playing Trust here; in production a different person approves.**
> On this page you review and merge your own pull request. Page 9 turns on
> the rule that stops a developer doing that.

**Why:** a customer needs to know that a named person checked the answers,
not only that a tool produced them. Trust's sign-off is the review and merge
of the pull request, so Git records who approved which version and when.

## Try it

1. **Review as Trust.** Open the pull request and check what you are signing
   off:
   - **Files changed** shows the new AI (`requirements.txt`,
     `config/models.yaml`), the declarations, and `trust/approvals.yml` with
     the two decisions.
   - The comment's collapsed **System card** section shows the whole record as
     a customer would read it: both models, regions, the companies that
     process data and the provider terms.
   - The comment's **What changed** section compares the record with the
     approved one on `main`. This is the first sign-off, so it says there is
     no approved version yet and counts what the record holds.

   Trust signs off only if every fact is right. If one is wrong, the fix is
   another commit on the branch, then another `trustbom propose`.

2. **Merge.** GitHub does not let you approve your own pull request, and
   nothing requires an approval yet, so skip **Approve**. Select
   **Merge pull request** with **Create a merge commit**, then confirm. Use a
   merge commit rather than squash or rebase, so `main` keeps a merge per
   sign-off.

3. **Update your clone and check the status:**

   **PowerShell / bash:**

   ```sh
   git switch main
   git pull
   trustbom scan
   git restore trust/card.json
   ```

   Output of `trustbom scan`:

   ```text
   Wrote <repo>/trust/card.json
   Wrote <repo>/trust/card.md
   Wrote <repo>/trust/questionnaire.yml
   Wrote <repo>/trust/findings.sarif
   Found 2 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/card.md.
   Status: Approved
   Next: None: the default branch holds Trust's approval of this graph.
   ```

   The scan writes the new commit id and time into `trust/card.json`. That is
   not a change to the record, so `git restore` discards it. Open
   `trust/card.md`: its status is now **Approved**, with Trust's two
   decisions. The section below it, the scanner's draft classification, stays
   `unknown`, because the scanner never makes those decisions.

## What you now have

An approved record on `main`: the AI the product uses, the facts a customer
asks about, and Trust's two decisions, all tied to the merge that approved
them. From now on, any pull request that changes the AI fails the check
until the change is proposed, and is not approved until Trust merges it.

[How the approval is proven](how-approval-is-proven.md) explains why the merge
counts as the sign-off and what stops a later change reusing it.

Next: [7. Answer a customer](7-answer-a-customer.md).
