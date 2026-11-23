# 3. Add AI

**Why:** developers add AI the way they add anything else: change some code,
build it, open a pull request. They do not run `trustbom` first. So the pull
request is where AI TrustBOM has to catch the change and say what is needed.

## Try it

1. **Make the change on a branch.** Drivers will ask about their contracts,
   and we answer with a Gemini model. Create a branch:

   **PowerShell / bash:**

   ```sh
   git switch -c add-gemini
   ```

   Then add one line to the end of each of two files in your editor: type
   them, or copy them from `workshop/gemini-requirements.example` and
   `workshop/models.example`. Keep each file ending with a new line.

   `requirements.txt` gains the Gemini client library:

   ```text
   openai==1.109.1
   google-genai==1.41.0
   ```

   `config/models.yaml` gains the model the driver feature uses:

   ```yaml
   rider_support: gpt-4o-mini
   driver_contracts: gemini-2.5-flash
   ```

2. **Commit, push and open a pull request**, exactly as for any change:

   **PowerShell / bash:**

   ```sh
   git add requirements.txt config/models.yaml
   git commit -m "Add Gemini for driver contract questions"
   git push -u origin add-gemini
   gh pr create --fill
   ```

   `gh` prints the address of the pull request. In the browser, check that the
   pull request is in **your fork** (`<your-owner>/abingdon-taxi-solutions`),
   from `add-gemini` into `main`. If you open it in the browser rather than
   with `gh`, GitHub suggests the original repository as the base; change the
   base repository to your fork.

3. **Read the check and the comment.** Within a minute or two the
   **AI TrustBOM / Scan and gate** check fails, and the check posts a comment
   on the pull request. Its top part reads:

   ```text
   AI TrustBOM

   Status: Not approved — Needs information

   Gate: bootstrap required

   - trust/card.json is stale: the committed graph differs from this pull request's scan.
     Next: Run trustbom scan (or trustbom propose when signing off) and commit trust/card.json to this pull request.
   - [medium] region-unknown: The deployment region for gemini could not be determined.
     Next: Declare the region for 'gemini' under models: in trust/manual.yml (provider, model_id, region, reason).
   - [medium] region-unknown: The deployment region for openai could not be determined.
     Next: Declare the region for 'openai' under models: in trust/manual.yml (provider, model_id, region, reason).
   - [medium] subprocessor-undeclared: Provider 'gemini' has no declared subprocessor.
     Next: Add 'gemini' to trust/subprocessors.yml, or list the path under not_in_scope: in trust/manual.yml if it is a false positive.
   - [medium] subprocessor-undeclared: Provider 'openai' has no declared subprocessor.
     Next: Add 'openai' to trust/subprocessors.yml, or list the path under not_in_scope: in trust/manual.yml if it is a false positive.

   Next: Run trustbom scan (or trustbom propose when signing off) and commit trust/card.json to this pull request.
   ```

   Read it as a to-do list:

   - **The record is out of date.** The pull request adds AI, but the
     committed `trust/card.json` still describes the product without it.
   - **Four gaps.** For each provider, nobody has said where the model runs or
     which company processes the data. OpenAI's two gaps were already there
     on page 2; Gemini adds two more.
   - **`Gate: bootstrap required`** means the repository has never been signed
     off, so this first sign-off covers all of its AI, not only Gemini.

   Do not run the scan yet. Page 4 fills the gaps first, then brings the
   record up to date in one step.

   Further down, the comment explains each item, lists the findings, and
   shows the draft record, with the file and line behind each fact, in a
   collapsed section. The check also marks the lines in **Files changed**.

## What you now have

An open pull request that cannot be called done: the check failed and the
comment says what is missing. The developer did not need to know about
AI TrustBOM beforehand. The comment told them.

> **In production.** Once enforcement is on (page 9), a failed check stops the
> pull request merging. Until then it is a warning.

Next: [4. Fill the gaps](4-fill-the-gaps.md).
