# Offline appendix: the tutorial without pull requests

Use this route when you cannot use GitHub pull requests, for example where
GitHub Actions cannot run. It is a stand-in: there is
no pull-request check, no comment and no enforcement, and you play every role
locally. You still need a GitHub account and a fork once, for the clone and
for the trial on page 7. The main tutorial is the real path.

What changes is the remote and the review. A local bare repository stands in
for GitHub, `trustbom gate` prints a summary of what the pull-request
comment would show, and a local merge stands in for Trust's merge on GitHub.

## Set up the stand-in remote

Do [Before you start](before-you-start.md) including the fork and the clone
(skip `gh repo set-default`). The trial on page 7 is tied to the GitHub
account in `origin`'s fetch address, so keep that address and send pushes to
a local bare repository instead:

**PowerShell / bash:**

```sh
git init --bare ../abingdon-origin.git
git remote set-url --push origin ../abingdon-origin.git
```

Now `git push origin ...` writes to `../abingdon-origin.git` and updates
`origin/main` locally, which is what the approved commands read. Nothing is
written to GitHub.

**Do not fetch or pull** while you follow this route, and turn off automatic
`git fetch` in your editor. A fetch reads the fork on GitHub and moves
`origin/main` back to it, and the approved commands then find no sign-off.
`git push origin main` puts it back.

## Page by page

| Page | What is different offline |
| --- | --- |
| [1](1-see-your-ai.md) and [2](2-set-up-sign-off.md) | Nothing. The push on page 2 goes to the bare repository. |
| [3](3-add-ai.md) | Make the change and commit it, but do not open a pull request. Run `trustbom scan`, then `trustbom gate`, to see what the comment would say (see below). Do not commit the scan's changes; page 4 commits them. |
| [4](4-fill-the-gaps.md) and [5](5-propose.md) | Commit as written. Instead of reading the comment, run `trustbom gate`. Pushing the branch is optional. |
| [6](6-sign-off.md) | Merge locally as Trust (see below) instead of on GitHub, and skip `git pull`. |
| [7](7-answer-a-customer.md) | `request-trial` and `activate` need the network once. After that, paid commands run on a cached lease. |
| [8](8-publish-and-verify.md) | Nothing. |
| [9](9-enforce.md) | Not possible offline: rulesets are a GitHub feature. Read the page for what enforcement adds. |

### Read the check locally

[`trustbom gate`](../cli/gate.md#example) evaluates the branch as the
pull-request check does. Scan first, so that it reads an up-to-date
`trust/card.json`:

**PowerShell / bash:**

```sh
trustbom scan
trustbom gate
```

After page 4 it prints:

```text
AI TrustBOM gate: bootstrap required
Status: Not approved — Ready for sign-off
Next: Run 'trustbom propose' to ask Trust to sign off.
```

`gate` exits `3` until the proposal on page 5, then prints
`AI TrustBOM gate: passed` and `Status: Proposed, not approved`, and exits
`0`. `trustbom gate --verbose` adds the detail the comment shows. Locally
there is no stale-card item: the scan you just ran brought
`trust/card.json` up to date.
### Merge as Trust

Use a merge commit, then push `main` to the stand-in remote so that
`origin/main` moves too:

**PowerShell / bash:**

```sh
git switch main
git merge --no-ff add-gemini -m "Merge add-gemini (Trust review)"
git push origin main
git rev-parse main refs/remotes/origin/main
```

The last command prints the same commit id twice. Then run `trustbom scan`
and `git restore trust/card.json` as on page 6; the scan reports
`Status: Approved`.

If an approved command says `Git cannot prove the default branch`, run
`git remote set-head origin main` and try again.

> **You are playing Trust here; in production a different person approves.**
> Offline nothing stops you approving your own change. That is what page 9
> fixes on GitHub.
