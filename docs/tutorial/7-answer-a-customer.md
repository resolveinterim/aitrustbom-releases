# 7. Answer a customer

**Why:** a taxi firm's procurement team has sent
`rfi/request.csv`,
five questions about our AI. The answers have to come from what Trust
approved, not from the draft on someone's branch, and every answer has to say
where it came from.

The approved `export` and `answer` are paid commands, so this page starts the
free 30-day trial. Everything before this page was free.
[Licensing](../licensing.md#free-and-paid-commands) lists which commands are
paid.

## Try it

1. **See the refusal.** Without a license, a paid command writes nothing.
   At a terminal it first offers to request the trial for you:

   **PowerShell / bash:**

   ```sh
   trustbom export
   ```

   It asks
   `Enter your license key, or press Enter to request a free 30-day trial:`.
   Press Enter, read the privacy notice, then answer `no` to
   `Request a free 30-day trial key? (yes/no) [no]:`. This page requests the
   trial in separate steps so that you see each one:

   ```text
   license-missing: The trial request was declined; no license key is available.
   ```

   It exits `4`. In CI, where it cannot prompt, the same refusal says how to
   set `TRUSTBOM_LICENSE_KEY` instead
   ([continuous integration](../licensing.md#continuous-integration)).

2. **Request the trial.** The trial belongs to the GitHub account in your
   clone's `origin`, which is your fork
   ([owner binding](../licensing.md#owner-binding)). One trial per account:

   **PowerShell / bash:**

   ```sh
   trustbom license request-trial --email <you@example.com>
   ```

   Use any address you can read; it does not have to match your GitHub
   account. The command shows a privacy notice and sends your email address,
   the GitHub account and the CLI version to the license service. It replies
   the same way whether or not you are eligible, and the key arrives by
   email. It is never printed. If nothing arrives within a few minutes, check
   your spam folder, then write to `support@aitrustbom.com`. Pages 7 and 8
   need the key; page 9 does not.

3. **Activate the key.** `activate` asks for the key without showing it, so
   it stays out of your shell history. Paste the key from the email when it
   asks:

   **PowerShell / bash:**

   ```sh
   trustbom license activate
   ```

   Then check the license:

   **PowerShell / bash:**

   ```sh
   trustbom license status
   ```

   Output:

   ```text
   Your trial license ends on <date> (UTC).
   License: trial
   Trial ends: <date> (UTC). A trial does not renew.
   ```

4. **Export the approved record.** [`trustbom export`](../cli/export.md#example)
   finds Trust's sign-off on `main` as you last pulled it from GitHub, checks
   that it still matches, and writes the customer copy to `export/`. It uses
   the approved version even if you are on another branch:

   **PowerShell / bash:**

   ```sh
   trustbom export
   ```

   Output:

   ```text
   Wrote <repo>/export/card.md
   Wrote <repo>/export/questionnaire.yml
   Wrote <repo>/export/manifest.json
   Removed earlier export files not in this export: trustbom.cdx.json
   License: trial for GitHub owner <your-owner>
   Approved graph fingerprint: <12 hex characters>
   Full fingerprint: <64 hex characters>
   Approved graph commit: <40 hex characters>
   ```

   The last line is the merge from page 6. `export/card.md` is the customer
   copy of the record, marked approved. The CycloneDX file from page 1's
   draft is removed because this export did not ask for one
   (`--format cyclonedx` adds it); page 8 publishes its own copy.

5. **Answer the questionnaire.** [`trustbom answer`](../cli/answer.md#example)
   matches each question to the approved record:

   **PowerShell / bash:**

   ```sh
   trustbom answer --questions rfi/request.csv
   ```

   Output:

   ```text
   Wrote <repo>/export/answers.csv (5 matched)
   Wrote <repo>/export/unmatched.csv (0 unmatched)
   ```

   `export/answers.csv` has the columns `id`, `question`, `section`, `answer`,
   `cites` and `confidence`. In short:

   | id | Question | Answer | Confidence |
   | --- | --- | --- | --- |
   | RFI-01 | Do you use AI? | `yes` | high |
   | RFI-02 | Which models and vendors are used? | `gemini: gemini-2.5-flash; openai: gpt-4o-mini` | high |
   | RFI-03 | Is customer data used to train? | Provider training use is declared per provider; training by the vendor itself is not declared. | low |
   | RFI-04 | Where does inference run? | `gemini: europe-west2; openai: eu`, then each processor and its agreement | high |
   | RFI-05 | Is human oversight required? | `unknown` | low |

   Every answer lists the files behind it in `cites`. RFI-03 is `low`
   because the record covers the providers' training use (both `none`, from
   `manual.yml`) but not whether Abingdon Taxi Solutions trains on customer
   data itself. `unknown` is a gap Trust still has to fill: nothing in the
   repository says whether a person reviews the AI's output, and the tool
   does not invent an answer. Trust can write those answers into
   `export/answers.csv` by hand before sending it; they are then Trust's
   words, not facts from the approved record.

6. **Ask a question the record cannot answer.**
   `rfi/extra-question.csv`
   asks how drivers, managers and owners use AI. The record holds facts about
   models and providers, not how each role uses the service. Each `answer`
   run replaces `export/answers.csv` and `export/unmatched.csv`:

   **PowerShell / bash:**

   ```sh
   trustbom answer --questions rfi/extra-question.csv
   ```

   Output:

   ```text
   Wrote <repo>/export/answers.csv (0 matched)
   Wrote <repo>/export/unmatched.csv (1 unmatched)
   ```

   It exits `2`, and `export/unmatched.csv` lists the question for Trust to
   answer by hand:

   ```text
   "id","text","section"
   "RFI-06","How do drivers managers and owners use AI in your service?","Customer experience"
   ```

   Run the first questionnaire again before the next page, so the published
   evidence includes its answers:

   **PowerShell / bash:**

   ```sh
   trustbom answer --questions rfi/request.csv
   ```

## What you now have

Answers to the customer's questions, each from the approved record and each
citing its evidence, plus a clear list of what still needs a person. Nothing
was sent: `answer` writes files for Trust to review. [Mapping a buyer
questionnaire](../questionnaire-mapping.md) explains how questions are matched.

> **In production.** Export before answering: a later `trustbom export`
> replaces `export/`, including earlier answers. In CI the key comes from the
> `TRUSTBOM_LICENSE_KEY` secret; see
> [continuous integration](../licensing.md#continuous-integration).

Next: [8. Publish and verify](8-publish-and-verify.md).
