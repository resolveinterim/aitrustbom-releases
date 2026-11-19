# 3. Approve

**Goal:** resolve the findings with reviewed declarations, run the gate, write
an approval proposal, and merge it to `main` as Trust, so that the repository
has a real approved graph.

**You will need:** the repository from [page 2](2-detect.md), with its
checkpoint reached and the stand-in remote from [page 1](1-setup.md#try-it).

## Try it

1. **See the gate refuse.** [`trustbom gate`](../cli/gate.md#example) compares
   the graph with the approval. Nothing is approved yet, so it reports a
   bootstrap failure and exits `3`:

   **PowerShell / bash:**

   ```sh
   trustbom gate
   ```

   ```text
   <!-- ai-trustbom-pr-comment -->
   ## AI TrustBOM

   **Status:** bootstrap required
   **Graph fingerprint:** `<12 lowercase hex characters>`
   ...
   ### Findings

   - `medium` **region-unknown**: The deployment region for gemini could not be determined.
   - `medium` **subprocessor-undeclared**: Provider 'gemini' has no declared subprocessor.

   ### How to resolve
   ...
   ```

   `gate` appends its assessment to the tracked `trust/card.md`, which leaves
   the working tree dirty and would block a branch switch later. Restore it:

   **PowerShell / bash:**

   ```sh
   git restore trust/card.md
   ```

2. **Start a feature branch.** Approval is an ordinary reviewed Git change:

   **PowerShell / bash:**

   ```sh
   git checkout -b feature/approve-gemini
   ```

3. **Resolve the findings with declarations.** The synthetic facts are staged
   in `workshop/`; in a real repository you would record what Trust has
   verified. First copy the evidence files that the declarations cite, and the
   subprocessor declaration:

   **PowerShell:**

   ```powershell
   New-Item -ItemType Directory -Force trust/terms, trust/deletion, trust/dpas | Out-Null
   Copy-Item workshop/gemini-terms.example trust/terms/gemini.md
   Copy-Item workshop/gemini-deletion.example trust/deletion/gemini.md
   Copy-Item workshop/gemini-dpa.example trust/dpas/gemini.md
   Copy-Item workshop/subprocessors.example trust/subprocessors.yml
   ```

   **bash:**

   ```bash
   mkdir -p trust/terms trust/deletion trust/dpas
   cp workshop/gemini-terms.example trust/terms/gemini.md
   cp workshop/gemini-deletion.example trust/deletion/gemini.md
   cp workshop/gemini-dpa.example trust/dpas/gemini.md
   cp workshop/subprocessors.example trust/subprocessors.yml
   ```

   Now open `trust/manual.yml`, replace the line `models: []` with the `models`
   block below, and add the `data_handling` block at the end of the file
   (besides the scope exclusion from page 1, which stays). The same text is in
   `workshop/manual-model-region.example` and
   `workshop/manual-data-handling.example` if you prefer to copy it:

   ```yaml
   models:
     - provider: gemini
       model_id: gemini-2.5-flash
       region: europe-west2
       reason: Synthetic example. Trust reviewed the Gemini deployment configuration.
   data_handling:
     - provider: gemini
       retention: 30
       training_use: none
       deletion_path: trust/deletion/gemini.md
       evidence_path: trust/terms/gemini.md
       reason: Synthetic example. Trust reviewed the provider terms and deletion procedure.
   ```

   Manual values take precedence over inferred ones and keep their provenance;
   every declaration needs a reason, and every referenced file must be tracked
   in Git. See the [declaration rules](../manual-declarations.md#field-rules-and-alternatives).
   The subprocessor lives in `trust/subprocessors.yml`, not `manual.yml`
   ([format](../manual-declarations.md#declaring-subprocessors)).

4. **Commit and rescan.** The findings are gone:

   **PowerShell / bash:**

   ```sh
   git add -A
   git commit -m "Declare region, data handling and subprocessor for Gemini"
   trustbom scan
   ```

   ```text
   Graph fingerprint: <12 lowercase hex characters>
   Full fingerprint (SHA-256): <64 lowercase hex characters>
   Found 1 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/card.md.
   ```

5. **Write the approval proposal.** `high_risk` and `article_50` are human
   decisions the scanner never infers. For this exercise we state them
   explicitly: `--high-risk no` records Trust's decision that the system is not
   high-risk, and `--article-50 applies` records that the EU AI Act Article 50
   transparency duties apply (see the glossary).
   Preview first with
   [`trustbom request-approval`](../cli/request-approval.md#example):

   **PowerShell / bash:**

   ```sh
   trustbom request-approval --dry-run --high-risk no --article-50 applies
   trustbom request-approval --high-risk no --article-50 applies
   ```

   ```text
   Wrote proposal to trust/approvals.yml (graph <64 lowercase hex characters>).
   Rescan, then commit the proposal file trust/approvals.yml with the rescanned trust/card.json and trust/card.md in a pull request; ...
   ```

   The proposal records the **full** fingerprint, `approved_by` (claimed from
   `git config user.email`, **not verified identity**) and `approved_at`. See
   the [options](../cli/request-approval.md#options). The command never
   approves anything: the merge in step 7 is the decision.

6. **Rescan, commit, and check the gate passes.** Committing changes `HEAD`,
   and the approved graph must still match:

   **PowerShell / bash:**

   ```sh
   trustbom scan
   git add -A
   git commit -m "Request approval of the Gemini graph"
   trustbom gate
   git restore trust/card.md
   ```

   ```text
   **Status:** passed
   ...
   ### Gate result

   The current graph matches its approval and has no blocking policy conditions.
   ```

   `gate` exits `0`. The approval is still only a proposal on a branch.

7. **Merge as Trust.** Review what the approval covers, then merge with a
   merge commit (the review step a CODEOWNER performs):

   **PowerShell / bash:**

   ```sh
   git checkout main
   git merge --no-ff feature/approve-gemini -m "Merge approval of the Gemini graph (Trust review)"
   ```

8. **Update `origin/main`.** Approved commands prove the default branch from
   `origin/main`, not from a local `main`, so approved `export` refuses until
   the two match. In this exercise the push goes to the local bare repository
   (in production, merging on GitHub does this):

   **PowerShell / bash:**

   ```sh
   git push origin main
   git rev-parse main refs/remotes/origin/main
   ```

   Both commit ids are identical. [`trustbom export`](../cli/export.md#example)
   uses the approval on the **default branch**, never the current branch; it
   is a paid command, so it runs on [page 4](4-answer.md) after
   [Get a license](get-a-license.md). If it says
   `Git cannot prove the default branch`, run `git remote set-head origin main`
   (offline) and retry.

> **In production.** Trust does not merge their own work. A different
> write-capable CODEOWNER reviews and approves the pull request, GitHub enforces
> that once an administrator runs `trustbom github-rules apply`, and merging
> to the real remote updates `origin/main`; there is no local push. Run
> `trustbom request-approval --check-controls` for a read-only report of those
> controls. The approved graph commit is derived from default-branch
> first-parent history and is never stored in `approvals.yml`. See
> [approving a graph in Git](../cli/request-approval.md#approve-a-graph-in-git),
> [enforcement levels](../enforcement-levels.md) and the
> [Action guide](../github-action.md).

## Checkpoint

- `trustbom gate` on the feature branch exited `0` with `Status: passed`.
- `git log --first-parent main` shows the merge commit, and
  `git rev-parse main refs/remotes/origin/main` prints the same id twice.
- `git status` is clean.

The repository now has an approved graph. Continue with
[4. Answer](4-answer.md), [5. Publish](5-publish.md) or
[6. Automate](6-automate.md).
