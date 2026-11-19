# 4. Answer

**Goal:** see the difference between a draft and an approved export, produce
the approved pack, and answer the customer's RFI from it, including the
questions that cannot be answered.

**You will need:** the approved repository from [page 3](3-approve.md) (the
checkpoint reached, `main` checked out, `origin/main` equal to `main`) and the
license from [Get a license](get-a-license.md): `export` and `answer` are paid
commands. Without a license they exit `4` with a `license-*` diagnostic
([licensing diagnostics](../licensing.md#diagnostics-and-exit-codes)).

## Try it

1. **Make a draft export.** A draft is the *current branch's* graph, labelled
   as internal. A draft export needs the card's commit id to match `HEAD`, so
   rescan first, then restore the tracked scan files afterwards:

   **PowerShell / bash:**

   ```sh
   trustbom scan
   trustbom export --draft
   git restore trust
   ```

   ```text
   Wrote <repo>/export/card.md
   Wrote <repo>/export/questionnaire.yml
   Draft graph fingerprint: <12 lowercase hex characters>
   Full fingerprint: <64 lowercase hex characters>
   Status: internal draft - unapproved - not for customer use
   ```

   A draft is never customer evidence. See the
   [approved export guide](../approved-export.md) for the watermarking rules.

2. **Make the approved export.**
   [`trustbom export`](../cli/export.md#example) resolves the default branch,
   verifies the approval and writes the last **approved** graph:

   **PowerShell / bash:**

   ```sh
   trustbom export
   ```

   ```text
   Wrote <repo>/export/card.md
   Wrote <repo>/export/questionnaire.yml
   Wrote <repo>/export/manifest.json
   License: <trial or subscription> for GitHub owner <owner>
   Approved graph fingerprint: <12 lowercase hex characters>
   Full fingerprint: <64 lowercase hex characters>
   Approved graph commit: <40 lowercase hex characters>
   ```

   The approved graph commit is derived from `main`'s history. Compare the
   fingerprint with the one you approved on page 3. See the
   [options](../cli/export.md#options).

3. **Answer the RFI.** The customer's procurement team sent
   `rfi/request.csv`.
   [`trustbom answer`](../cli/answer.md#example) maps each question to the
   approved graph:

   **PowerShell / bash:**

   ```sh
   trustbom answer --questions rfi/request.csv
   ```

   ```text
   Wrote <repo>/export/answers.csv (5 matched)
   Wrote <repo>/export/unmatched.csv (0 unmatched)
   ```

   `export/answers.csv` (`id,question,section,answer,cites,confidence`):

   | id | answer | confidence |
   | --- | --- | --- |
   | RFI-01 | `yes` | high |
   | RFI-02 | `gemini: gemini-2.5-flash` | high |
   | RFI-03 | Provider training use is declared per provider; training by our own company is not declared. | low |
   | RFI-04 | `gemini: europe-west2`, plus the processor and DPA citation | high |
   | RFI-05 | `unknown` | low |

   Every answer cites repository files. `unknown` is an answer Trust still
   has to resolve: nothing in the repository declares human oversight, and the
   tool never invents one.

4. **Ask a question the graph cannot answer.** The sample
   `rfi/extra-question.csv`
   asks how drivers, managers and owners use AI. The fixed question bank covers
   model and policy facts, not each role's experience:

   **PowerShell / bash:**

   ```sh
   trustbom answer --questions rfi/extra-question.csv
   ```

   ```text
   Wrote <repo>/export/answers.csv (0 matched)
   Wrote <repo>/export/unmatched.csv (1 unmatched)
   ```

   The command exits `2`, and `export/unmatched.csv` contains:

   ```csv
   "id","text","section"
   "RFI-06","How do drivers managers and owners use AI in your service?","Customer experience"
   ```

   Trust must follow up rather than generate a claim from nothing. See
   [buyer questionnaire mapping](../questionnaire-mapping.md) for the CSV
   requirements and alias rules.

> **In production.** The command writes files for review; it does not email
> the customer. Export **before** answering: a later `trustbom export` replaces
> `export/` and removes earlier RFI files. If an approved command refuses,
> the message tells you which condition failed:
>
> | Message | Fix |
> | --- | --- |
> | `Git cannot prove the default branch` | `git remote set-head origin main` (offline), or `git remote set-head origin -a` with network access. |
> | `There is no approval for this repository on the default branch` | Merge the approval to the default branch and update `origin/main` (page 3, steps 7 and 8). |
> | Exit `4`, `license-*` | See [licensing diagnostics](../licensing.md#diagnostics-and-exit-codes), including CI and owner binding. |

## Checkpoint

- `export/answers.csv` was written with `5 matched` and `0 unmatched`, then
  the extra question exited `2` with one unmatched row.
- `export/unmatched.csv` lists `RFI-06`.
- `git status` shows no tracked changes (`export/` is ignored).

Next: [5. Publish](5-publish.md).
