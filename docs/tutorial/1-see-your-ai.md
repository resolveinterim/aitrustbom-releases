# 1. See your AI

**Why:** before anyone signs anything off, you need to know what AI the
product uses and where that shows up in the code. One command finds it, with
no license, account or setup.

## Try it

1. **Scan the repository.** [`trustbom scan`](../cli/scan.md#example) reads
   the files Git tracks, such as `requirements.txt` and `config/models.yaml`,
   and writes what it found to `trust/`:

   **PowerShell / bash:**

   ```sh
   trustbom scan
   ```

   Output:

   ```text
   Wrote <repo>/trust/card.json
   Wrote <repo>/trust/card.md
   Wrote <repo>/trust/questionnaire.yml
   Wrote <repo>/trust/findings.sarif
   Found 1 model(s), 0 tool(s), 2 finding(s). Review <repo>/trust/card.md.
   Detected AI (draft, from tracked files):
     model openai/gpt-4o-mini  (config/models.yaml:1)
   Draft questionnaire answers: <repo>/trust/questionnaire.yml ("unknown" means no evidence was found).
   Everything above is a draft. Nothing is approved.
   Next: trustbom export --draft --format cyclonedx   (free watermarked CycloneDX draft)
   Next: trustbom init   (start sign-off by a named approver)
   ```

   It found the OpenAI model that rider support uses, and the line of the
   file it came from.

2. **Read the draft answers.** Open `trust/questionnaire.yml`. It answers
   common customer questions from what the scan found. The first two have
   answers, each with the files that support it:

   ```yaml
   status: "draft (unapproved)"
   questions:
     - id: Q1_do_you_use_ai
       question: "Do you use AI?"
       answer: "yes"
       cite:
         - "config/models.yaml"
         - "requirements.txt"
     - id: Q2_models_and_vendors
       question: "Which models and vendors are used?"
       answer: "openai: gpt-4o-mini"
       cite:
         - "config/models.yaml"
         - "requirements.txt"
     # ...
     - id: Q4_inference_region
       question: "Where does inference run?"
       answer: "unknown"
     # ...
   ```

   The excerpt leaves out other fields and questions, marked `# ...`.
   `unknown` means the repository holds no evidence for that answer. It never
   means "no". Where inference runs, how long the provider keeps data, and
   most other answers need a person to declare them; page 4 does that.

3. **Read the record.** Open `trust/card.md`, the readable version of
   `trust/card.json`. It is marked **draft (unapproved)**, shows the OpenAI
   model with its evidence, and lists the two gaps the scan found: the region
   of the model and the company that processes its data. The
   [`card.md` reference](../cli/scan.md#approval-status-in-cardmd) explains each
   section.

4. **Make a draft for AI-BOM tools.** The free draft export writes a
   [CycloneDX](https://cyclonedx.org/) file that other tools can read, marked
   as an internal draft:

   **PowerShell / bash:**

   ```powershell
   trustbom export --draft --format cyclonedx
   ```

   Output:

   ```text
   Wrote <repo>/export/card.md
   Wrote <repo>/export/questionnaire.yml
   Wrote <repo>/export/trustbom.cdx.json
   Draft graph fingerprint: <12 hex characters>
   Full fingerprint: <64 hex characters>
   Status: internal draft - unapproved - not for customer use
   ```

   `export/` is ignored by Git. Do not commit anything yet: keep `trust/` as
   it is, and page 2 commits it with the sign-off files.

## What you now have

A draft record of the AI in the product, with the file and line behind each
fact, and draft answers to a customer's first questions. All of it is marked
draft: nobody has checked it, so you cannot send it to a customer yet. The
scan only sees what tracked files show; [what it can and cannot see](../workflow.md#what-it-can-and-cannot-see)
lists the gaps, such as direct HTTP calls to an AI API.

Next: [2. Set up sign-off](2-set-up-sign-off.md).
