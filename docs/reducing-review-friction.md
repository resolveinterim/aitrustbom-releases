# Reducing review friction

AI TrustBOM keeps approval in Git: a pull request that changes
`trust/approvals.yml` must be approved by a CODEOWNER with the Trust role who is
not its author (see [approving a graph](cli-reference.md)). Trust is often
outside the development team, so this can feel like a bottleneck. This guide
explains where waiting actually happens and how to keep it small without
weakening the control.

## Where review waits

- **Only graph changes need Trust.** Ordinary code pull requests never touch
  `approvals.yml`. Trust is involved only when a pull request changes AI facts:
  a provider, model, region, tool or MCP server, or a finding.
- **A changed graph is a failing gate until approval is recorded.** The pull
  request's gate reports what changed and what action is required.
- **The approval cannot precede the change it approves.** The fingerprint
  covers the final graph, so the approval entry follows the code. Any later
  change to the graph makes it stale, and stale reviews must be dismissed.
- **Trust may not live in GitHub.** Compliance and legal reviewers are often
  unfamiliar with pull requests.

## What does not change

Independent human approval is the product's control. Skipping it for speed
removes the tamper-evident record. These options change *when* and *how
cheaply* approval happens, never *whether*.

## Practical options

1. **Name a team, and at least two people.** List a visible GitHub team (or
   several individuals) as CODEOWNERS for `trust/approvals.yml`. GitHub does
   not let an author approve their own pull request, so a single Trust
   reviewer who also authors the change cannot approve it. A vendor needs at
   least two people able to review.
2. **Make the review small.** Trust should review the gate's pull-request
   comment (semantic changes, findings, required actions) and a short
   `approvals.yml` diff, not the code. The
   `trustbom request-approval` command writes the full
   fingerprint, requester and timestamp so the
   diff is a few lines and nothing is typed by hand.
3. **Use a time-boxed break-glass entry for urgent merges.** Break-glass is a
   `break_glass:` block in `trust/approvals.yml` with `by`, `expires`, `ticket`
   and `allows` (`graph_delta`, `rejections` or `rejection:<id>`, `findings` or
   `finding:<id>`). While it is unexpired and allows every outstanding issue,
   the gate passes as `breakglass` (draft only). It is **not** a way around
   Trust: adding the block is a change to the CODEOWNERS-protected file, so
   Trust must review it like any other approval change. It never creates an
   approval; `trustbom export`, `pack` and `publish` refuse it, and the gate
   fails again at `expires`. Use it sparingly, name the ticket, and keep
   `allows` as narrow as possible.
4. **Batch on a cadence.** Buyers see the pack only when you export it, so you
   can approve on a release cadence, such as weekly, rather than per pull
   request. Merging graph changes in the meantime needs either an approval
   entry or a reviewed break-glass entry (option 3); neither removes Trust's
   review.
5. **Keep changes granular and early.** Open the pull request that changes the
   AI facts as soon as the change is known, so Trust sees it while context is
   fresh and the fingerprint is stable.
6. **Dismiss only what is necessary.** Keep "dismiss stale reviews" on, but
   avoid pushing graph-changing commits after Trust has reviewed.
7. **Check the setup.** `trustbom init --interactive` can read branch
   protection and CODEOWNERS (read-only, with consent) and report what is
   missing. The planned `request-approval --check-controls` also checks for
   "approval of the most recent reviewable push", a CODEOWNERS entry for the
   approvals path, at least two eligible reviewers, and administrator bypass.
   It also reports whether `Scan and gate` is a required status check and
   whether "require branches to be up to date" is set; both are recommended
   because the generated workflow has no post-merge run (see
   [GitHub Action guide](github-action.md#recommended-repository-controls)).

## Solo vendors

If one person is both author and the only Trust reviewer, GitHub will not let
them approve their own pull request. Vendors then either add a second reviewer
or accept a weaker setup such as administrator bypass. The bypass removes the
independence the approval is meant to provide, so treat it as a recorded,
temporary exception.

## Related

- [CLI reference: approve a graph in git](cli-reference.md)
- [GitHub Action guide](github-action.md)
- [Approved export](approved-export.md)
