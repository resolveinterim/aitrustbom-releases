# Tutorial: we are Abingdon Taxi Solutions

A customer asked what AI you use. Answer with evidence a named person
approved.

In this tutorial **we are Abingdon Taxi Solutions**, a company that sells an
AI-powered service to taxi firms. Our rider support already uses an OpenAI
model. On a pull request we add a Gemini model for drivers, and AI TrustBOM
stops it shipping until Trust signs it off. At the end we answer a customer's
questionnaire from the approved record and publish evidence the customer can
check.

You work on your own GitHub copy (a fork) of the starter repository, with the
real GitHub Action and real pull requests. The starter is an evidence
repository, not a runnable application: a description of the service, a
sample customer questionnaire and the files the tutorial adds. Everything in
it is synthetic.

## Who does what

| Role | In this tutorial |
| --- | --- |
| **Developer** | You. You add AI to the product and open pull requests, as you would on any other change. |
| **Trust** | Also you, for pages 1 to 8. Trust is the person who approves what the company tells customers about its AI. **You are playing Trust here; in production a different person approves.** Page 9 turns on the rule that makes that a requirement. |
| **Customer (the buyer)** | The taxi firm whose procurement team sends the questionnaire. On page 8 you check the published evidence as they would. |

An **AI provider** (here OpenAI and Google) is the company whose model we use.
The glossary defines every term.

## How the pages work

Each page has three parts:

- **Why**: the problem the step solves.
- **Try it**: the commands and the output they print.
- **What you now have**: the result, and why Trust or a customer cares.

Commands marked **PowerShell / bash** are the same in both shells. Output
comes from a local run of these steps with a development build; a released
`trustbom` prints the same text apart from version numbers. Values in
angle brackets are placeholders: `<your-owner>` is your GitHub account,
`<repo>` is the folder of your clone, and `<12 hex characters>` is a value
that differs in your run.
Windows shows paths with `\` where these pages show `/`.

## The pages

| Page | What you do | License |
| --- | --- | --- |
| [Before you start](before-you-start.md) | Install `trustbom`, fork the starter and clone it | None |
| [1. See your AI](1-see-your-ai.md) | Run one command and read what it found | None |
| [2. Set up sign-off](2-set-up-sign-off.md) | Add the sign-off files and the pull-request check | None |
| [3. Add AI](3-add-ai.md) | Add Gemini on a branch and open a pull request; the check fails | None |
| [4. Fill the gaps](4-fill-the-gaps.md) | Follow the pull-request comment until it says Ready for sign-off | None |
| [5. Propose](5-propose.md) | Ask Trust to sign off | None |
| [6. Sign off](6-sign-off.md) | Review and merge as Trust; the record is approved | None |
| [7. Answer a customer](7-answer-a-customer.md) | Start the free trial; export and answer a questionnaire | Trial |
| [8. Publish and verify](8-publish-and-verify.md) | Publish the evidence and check it as the customer | Trial |
| [9. Enforce, then the next change](9-enforce.md) | Require a second person; the next pull request is blocked | None |

Do the pages in order: each one starts where the previous one ended. Pages 1
to 6 and 9 use free commands. The paid commands start on page 7, where you
request a free 30-day trial.

Three pages sit beside the tutorial:

- [How the approval is proven](how-approval-is-proven.md) explains what the
  check compares and why a merged pull request counts as Trust's approval.
- [Offline appendix](appendix-offline.md) runs the same steps without pull
  requests, for example where GitHub Actions cannot run.
- [Several AI systems in one repository](portfolio-monorepo.md) is for after
  page 7: add a second AI system under the same Trust, approve each system
  separately and publish one pack. For systems in separate repositories, give
  each repository its own TrustBOM, as in this tutorial.

Start with [Before you start](before-you-start.md).
