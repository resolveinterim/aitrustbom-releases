# 2. Set up sign-off

**Why:** a draft is not something you can give a customer. Before the record
counts, a named person (Trust) has to sign it off, and new AI must not reach
`main` without that sign-off. This page adds the files that make that happen
on every pull request.

## Try it

1. **Add the sign-off files.** [`trustbom init`](../cli/init.md#example)
   writes the configuration and the GitHub files and changes nothing else:

   **PowerShell / bash:**

   ```sh
   trustbom init
   ```

   Output:

   ```text
   Initialization plan for <repo>:
     create: trust/system.yml
     create: trust/manual.yml
     create: trust/approvals.yml
     create: .github/CODEOWNERS
     create: .github/rulesets/trustbom.json
     create: .github/workflows/ai-trustbom.yml
     create: .github/actions/trustbom/action.yml
     create: .github/actions/trustbom/install/action.yml
     append: .gitignore
   Created 8 file(s); preserved 0 existing file(s); appended to 1 existing file(s).
   Next: review the generated files, replace CODEOWNERS examples, commit them, then run `trustbom scan` and `trustbom gate`.
   Approval protection: .github/rulesets/trustbom.json holds the ruleset. A repository admin applies it with `trustbom github-rules apply`, or imports it in GitHub (Settings > Rules > Rulesets > Import a ruleset). Until then approvals are advisory.
   ```

   The files that matter here:

   | File | What it is for |
   | --- | --- |
   | `trust/approvals.yml` | Trust's sign-off. It starts as an unsigned template. |
   | `trust/manual.yml` | Facts a person declares because the files cannot show them. |
   | `.github/workflows/ai-trustbom.yml` | The check that runs on every pull request. |
   | `.github/CODEOWNERS` | Who reviews changes to the sign-off. |
   | `.github/rulesets/trustbom.json` | The rule that makes the review required. Page 9 turns it on. |

   `init` also tells Git to ignore `trust/card.md`, `trust/questionnaire.yml`
   and `trust/findings.sarif`, which every scan rewrites. Only
   `trust/card.json`, the machine-readable record, is committed.

2. **Name the reviewer.** Open `.github/CODEOWNERS` and replace each
   `@legal-trust @platform-security` with your own GitHub account, so the file
   reads:

   ```text
   # Replace these example teams with your Trust and platform owners.
   /trust/approvals.yml @<your-owner>
   /.github/workflows/ai-trustbom.yml @<your-owner>
   /.github/actions/trustbom/ @<your-owner>
   ```

   In production these are your Trust team and platform owners, not the
   developer. Only the sign-off file needs Trust's review: a change to
   `trust/manual.yml` or anything else in the record no longer matches the
   sign-off in `trust/approvals.yml`, so it cannot be approved without a new
   one.

3. **Scan, then commit and push.** The pull-request check compares each pull
   request with the committed `trust/card.json`, so commit it with the other
   files. The scan reads only files Git tracks, so the first `git add -A`
   lets it see the new files:

   **PowerShell / bash:**

   ```sh
   git add -A
   trustbom scan
   git add -A
   git commit -m "Set up AI TrustBOM"
   git push origin main
   ```

   Output of `trustbom scan`:

   ```text
   Wrote <repo>/trust/card.json
   Wrote <repo>/trust/card.md
   Wrote <repo>/trust/questionnaire.yml
   Wrote <repo>/trust/findings.sarif
   Found 1 model(s), 0 tool(s), 2 finding(s). Review <repo>/trust/card.md.
   Status: Not approved — Needs information
     - [medium] region-unknown: The deployment region for openai could not be determined.
       Next: Declare the region for 'openai' under models: in trust/manual.yml (provider, model_id, region, reason).
     - [medium] subprocessor-undeclared: Provider 'openai' has no declared subprocessor.
       Next: Add 'openai' to trust/subprocessors.yml, or list the path under not_in_scope: in trust/manual.yml if it is a false positive.
   Next: Resolve the items above, then run 'trustbom propose'.
   ```

   Now that the sign-off files exist, `scan` reports the status and the gaps.
   Leave them for now: page 4 fills them in the same pull request as the new
   AI. Git on Windows may print line-ending warnings; they are harmless.

   If GitHub refuses the push because the token lacks the `workflow` scope,
   run `gh auth refresh -s workflow`, then `gh auth setup-git`, and push
   again: pushing a file under `.github/workflows/` needs that permission.

## What you now have

`main` holds the sign-off files, the pull-request check and a record that is
**Not approved**. Nothing has run on GitHub yet: the check runs on pull
requests, and you open the first one on the next page.

> **In production.** A developer runs `init` once per repository and a
> platform owner reviews the generated files. Behind an internal npm registry,
> see [offline and mirrored installs](../installing.md#offline-and-mirrored-installs).

Next: [3. Add AI](3-add-ai.md).
