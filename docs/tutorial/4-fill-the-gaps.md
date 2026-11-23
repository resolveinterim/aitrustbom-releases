# 4. Fill the gaps

**Why:** a customer will ask where the AI runs and who handles their data.
The files cannot show that, so a person who knows has to declare it, with a
reason and a pointer to the evidence. AI TrustBOM keeps who declared what, and
never fills a gap by guessing.

## Try it

1. **Add the declarations.** The workshop folder holds the declarations for
   both providers. In a real repository Trust writes them from the provider
   contracts and the deployment settings; here you copy them:

   **PowerShell / bash:**

   ```sh
   mkdir trust/providers
   cp workshop/openai-provider.example trust/providers/openai.md
   cp workshop/gemini-provider.example trust/providers/gemini.md
   cp workshop/manual.example trust/manual.yml
   cp workshop/subprocessors.example trust/subprocessors.yml
   ```

   | File | What it declares |
   | --- | --- |
   | `trust/manual.yml` | For each model, the region it runs in and why we know. For each provider, how long it keeps prompts, whether it trains on them, and how to ask for deletion. |
   | `trust/subprocessors.yml` | The company that processes the data for each provider, where it is, and its data-processing agreement. |
   | `trust/providers/*.md` | The provider notes the declarations point to. Real ones would be the provider's terms and agreement. |

   The `manual.yml` copy replaces the template `init` wrote, which held no
   declarations. `retention: 30` means the provider keeps prompts for 30
   days.

   For example, the Gemini entry in `trust/manual.yml`:

   ```yaml
   models:
     - provider: gemini
       model_id: gemini-2.5-flash
       region: europe-west2
       reason: Synthetic example. Trust reviewed the driver contracts deployment configuration.
   ```

   A declaration in `manual.yml` takes priority over what the scan inferred,
   and the record keeps both, with the line each came from. The
   [manual declarations guide](../manual-declarations.md) lists every field.

2. **Bring the record up to date.** This is the step the comment's first
   item asked for. Scan, and check the result before you commit:

   **PowerShell / bash:**

   ```sh
   git add -A
   trustbom scan
   ```

   Output:

   ```text
   Wrote <repo>/trust/card.json
   Wrote <repo>/trust/card.md
   Wrote <repo>/trust/questionnaire.yml
   Wrote <repo>/trust/findings.sarif
   Found 2 model(s), 0 tool(s), 0 finding(s). Review <repo>/trust/card.md.
   Status: Not approved — Ready for sign-off
   Next: Run 'trustbom propose' to ask Trust to sign off.
   ```

   Both models are found and no gaps remain. If a gap remains, the output
   names it and says what to add.

3. **Commit and push:**

   **PowerShell / bash:**

   ```sh
   git add -A
   git commit -m "Declare regions, providers and data handling"
   git push
   ```

4. **Read the comment again.** The check runs again and updates its comment:

   ```text
   AI TrustBOM

   Status: Not approved — Ready for sign-off

   Gate: bootstrap required

   Next: Run `trustbom propose` to ask Trust to sign off.

   To propose this change, run these in your branch:

   npm install --global trustbom@1.0.2
   trustbom propose --high-risk <yes|no> --article-50 <yes|no>

   Replace each <yes|no> with your decision (the scanner never makes it), then commit trust/ and push.
   ```

   The check still fails, because nobody has signed off yet. The comment now
   gives the exact commands for the next step. The `npm install` line installs
   the version of `trustbom` that the check used; you installed it in
   [Before you start](before-you-start.md), so you can skip it. Page 5 runs
   `propose` with the two decisions filled in.

## What you now have

A pull request whose record is complete: both models, where they run, who
processes the data and what the providers do with it, each with the file
that says so. It is **Ready for sign-off**, but it is still not approved.

> **In production.** The developer often does not know the region or the
> provider terms. They ask Trust or the platform owner, who writes the
> declaration or reviews it in the pull request.

Next: [5. Propose](5-propose.md).
