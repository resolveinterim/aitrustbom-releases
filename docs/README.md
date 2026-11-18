# AI TrustBOM guides

AI TrustBOM is a git-native AI disclosure tool. Your company builds
AI-powered software, your customer (the buyer) receives its evidence, and
Trust, your company's authorized human approver, decides what is approved.
These guides follow that journey. Guides for buyers say "vendor" for the
company that sends a pack.

## Start here

1. [How you use AI TrustBOM](workflow.md): the end-to-end workflow at a glance.
2. [Tutorial: Abingdon Taxi Solutions](tutorial/README.md): a hands-on first journey
   with a starter repository.
3. [Licensing](licensing.md): the free tier, the 30-day trial and the paid
   subscription.

## Set up and run

4. [Install the CLI](installing.md): installer scripts, npm, offline installs
   and first-run warnings.
5. [CLI reference](cli/README.md): one page per command, from `init` to
   approved `export`, with options, exit codes and license tier.
6. [GitHub Action](github-action.md): the generated workflow, permissions,
   gate and pull-request comment.
7. [YAML input reference](yaml-input-reference.md): the files you author and
   review in `trust/`.
8. [Manual declarations](manual-declarations.md): a complete `trust/manual.yml`
   example, subprocessors, data handling and CI evaluation results.
9. [Reducing review friction](reducing-review-friction.md): keep approvals
   small, fast and in Git.
10. [Enforcement levels](enforcement-levels.md): `enforced`, `audited`,
    `violation` and `unverified`, and how to reach `enforced`.

## Share with buyers

11. [Approved and draft export](approved-export.md): the approved customer pack
    and explicit draft packs.
12. [Map a buyer questionnaire](questionnaire-mapping.md): answer a buyer's
    questions from approved evidence, including answer drift and workbook reuse.
13. [Publish the trust page](trust-page.md): a static trust site, for example on
    GitHub Pages.
14. [Growth reusable workflow](growth-workflow.md): scan and approve a portfolio
    of systems in one repository.
15. [Review pack history](pack-history.md): compare approved graphs over time.

## Receive and verify

16. [Verifying a received pack](pack-verification.md): for buyers checking a
    vendor's pack.
