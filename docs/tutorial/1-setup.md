# 1. Set up

**Goal:** clone the evidence repository, initialise AI TrustBOM, record who we
are and what is out of scope, and run the first scan.

**You will need:** Git, a shell (PowerShell or bash: see the
[convention](README.md#prerequisites)) and the `trustbom` CLI
([installing](../installing.md)). Part of this page is local exercise setup: a
local stand-in for the GitHub remote.

Everything on pages 1 to 3 uses free commands. The paid commands on pages 4
and 5 need a trial license, which the page [Get a license](get-a-license.md)
sets up before page 4.

## Try it

1. **Clone the starter**, then read
   `service/experience.md`
   and `rfi/request.csv`.
   A license is bound to the GitHub owner in `origin`, so **fork**
   [`resolveinterim/abingdon-taxi-solutions`](https://github.com/resolveinterim/abingdon-taxi-solutions)
   into your own GitHub account first and clone the fork (replace
   `<your-owner>`).
   **PowerShell / bash:**

   ```sh
   git clone https://github.com/<your-owner>/abingdon-taxi-solutions.git
   cd abingdon-taxi-solutions
   ```

2. **Local exercise setup: a stand-in remote.** Approved `export` and
   `answer` read the default branch from `origin/main`, so after Trust merges
   an approval you must update that reference. A real team merges a pull
   request on GitHub; in this exercise you push to a local bare repository
   instead, so nothing is written to the public repository. The fetch URL
   stays on GitHub (the license checks the owner in it):

   **PowerShell / bash:**

   ```sh
   git init --bare ../abingdon-origin.git
   git remote set-url --push origin ../abingdon-origin.git
   ```

3. **Initialise.** [`trustbom init`](../cli/init.md#example) writes the
   configuration and the GitHub files but changes nothing else:

   **PowerShell / bash:**

   ```sh
   trustbom init
   ```

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
     preserve: .gitignore
   Created 8 file(s); preserved 1 existing file(s).
   ```

   This tutorial edits the generated files by hand in the next two steps. To
   be prompted for them instead, see the
   [guided setup reference](../cli/init.md#guided-single-system-setup).
   Scanning reads Git-tracked files only, so never Git-ignore `trust/`.

4. **Record our identity.** Open `trust/system.yml` in your editor (for
   example `code trust/system.yml`), replace its three lines with the block
   below, and save:

   ```yaml
   name: Abingdon Taxi Solutions service
   description: Driver, manager, and owner experience for our taxi customers.
   customer_facing_name: Abingdon Taxi Solutions
   ```

5. **Exclude the workshop inputs.** Open `trust/manual.yml` in your editor,
   replace the line `not_in_scope: []` with the block below (keep the other
   lines), and save. The `workshop/` files are examples you copy in later;
   excluding them stops them appearing as current AI use.

   ```yaml
   not_in_scope:
     - path: workshop/**
       reason: Example inputs are not part of our service.
   ```

6. **Commit and scan.** [`trustbom scan`](../cli/scan.md#example) reads
   tracked files, so commit the setup first:

   **PowerShell / bash:**

   ```sh
   git add -A
   git commit -m "Initialize our taxi service AI inventory"
   trustbom scan
   ```

   ```text
   Wrote <repo>/trust/card.json
   Wrote <repo>/trust/card.md
   Wrote <repo>/trust/questionnaire.yml
   Wrote <repo>/trust/findings.sarif
   Graph fingerprint: <12 lowercase hex characters>
   Full fingerprint (SHA-256): <64 lowercase hex characters>
   Found 0 model(s), 0 tool(s), 1 finding(s). Review <repo>/trust/card.md.
   ```

   Git may print line-ending warnings on Windows; they are cosmetic.

7. **Read `trust/card.json` and `trust/card.md`.** The inventory is empty
   because the model inputs are still in `workshop/`, so the card reports
   `no-models-detected`:

   ```json
   {
     "models": [],
     "mcp_servers": [],
     "findings": [{ "id": "no-models-detected", "severity": "medium" }]
   }
   ```

   `card.md` is labelled `draft (unapproved)` and names our system.

> **In production.** Replace the example teams in `.github/CODEOWNERS` with
> your real Trust reviewers before relying on approval rules, and install the
> CLI on every engineer's machine and in CI. Behind an internal registry or
> offline, see [offline and mirrored installs](../installing.md#offline-and-mirrored-installs).

## Checkpoint

- `trustbom scan` exited `0` and printed a graph fingerprint.
- `trust/card.json` has no models, no MCP servers and one
  `no-models-detected` finding.
- `git status` shows only the four scan artifacts under `trust/` as new
  files; `export/` is ignored.

Next: [2. Detect](2-detect.md).
