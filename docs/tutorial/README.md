# Tutorial: we are Abingdon Taxi Solutions

For this tutorial, **we are an AI-first company providing software as a service for taxi companies**.
Our customers want a better experience for their **drivers** (contract and
earnings questions), **managers** (productivity and absence) and **owners**
(fleet information). They also need to know how we use AI in our service. Our
job is to give them an evidenced, reviewed answer rather than an unevidenced
promise.

You will work in the company's **Git evidence repository**: a description of
the service, a sample customer request for information (RFI) and example
inputs for later changes. It is not a runnable taxi application, and the
records and the customer request are synthetic.

## Who does what

| Role | Who | In this tutorial |
| --- | --- | --- |
| **Our company** | Abingdon Taxi Solutions, which sells the AI-powered service | You, working in the evidence repository. |
| **Our customer (the buyer)** | The customer whose procurement team sends the RFI | Receives the approved evidence and verifies it. |
| **Trust** | The person at our company who approves what we share | A hat you put on for this exercise when you review and merge the approval. In a real company this is a different person from the author. |

An **AI provider** (here Google, supplying Gemini) is the company whose model
we use. The guides say "your company" and "your customer (the buyer)"; this
tutorial says "we" and "our customer". The glossary calls the company *vendor*,
a word kept for buyer-facing material, so that it is never confused with the AI
provider.

Scanning produces a *draft* inventory of what tracked files indicate. The
team checks the underlying facts, and Trust approves a graph before it is used
to answer a customer. See the glossary for terms.

## Prerequisites

- Git and a shell: PowerShell or bash (Linux, macOS, Git Bash, WSL). Commands
  are labelled **PowerShell / bash**, or given separately for each where they
  differ.
- The `trustbom` CLI. The simplest route is `npm install --global trustbom`;
  other routes avoid Node.js. See [installing the CLI](../installing.md).
- **A free trial license before page 4.** Pages 1 to 3 use only free
  commands. The approved `export`, `answer` and `publish` commands on pages 4
  and 5 are paid, so do [Get a license](get-a-license.md) after page 3
  (the trial emails a key, so start it early if you like). The license is bound
  to the GitHub owner in `origin`, so fork the starter into your own account on
  page 1. [Licensing](../licensing.md#free-and-paid-commands) lists which commands
  are free.

Output in these pages is **representative of this starter**. Paths, commit
ids, graph fingerprints and CLI versions vary between checkouts. Values in
angle brackets are placeholders: `<repo>` stands for the location of your
clone, and `<12 lowercase hex characters>` for a value you will see in your
own output. Counts can vary with the detector set.

Each page has runnable **Try it** steps and, separately, **In production**
callouts that describe what differs in a real repository. The callouts never
interrupt a command sequence and need not be run.

## What you will have at the end

- A scanned, cited `trust/card.json` for the Gemini model that answers driver questions (one model and one
  MCP server) with no open findings.
- A Trust decision recorded in `trust/approvals.yml` and merged to `main`,
  with the human `high_risk` and `article_50` decisions stated.
- An approved export, a cited RFI answer file and an unmatched-question list.
- A CycloneDX ML-BOM, a readiness checklist and a deployable trust page that a
  buyer verifies offline.
- An understanding of what the GitHub Action adds on every pull request.

## The pages

| Page | Goal | Path |
| --- | --- | --- |
| [1. Set up](1-setup.md) | Clone, `init`, identity, scope exclusion, first scan | Core |
| [2. Detect](2-detect.md) | Add Gemini and an MCP connection; read the card and questionnaire | Core |
| [3. Approve](3-approve.md) | Resolve findings, `gate`, `request-approval`, merge as Trust | Core |
| [Get a license](get-a-license.md) | Activate a free trial for the paid commands | Before page 4 |
| [4. Answer](4-answer.md) | Draft versus approved export; answer the RFI | Independent |
| [5. Publish](5-publish.md) | CycloneDX, readiness, `publish`, `verify` | Independent |
| [6. Automate](6-automate.md) | The Action, PR comment, enforcement, upgrade, portfolio | Independent |

Do pages 1 to 3 in order: each ends at a checkpoint the next one starts from,
and page 3 finishes with a real approval merged to the local `main`. Pages 4
to 6 then run against that approved repository in any order. Every command is
linked to its [reference entry](../cli/README.md).

Start with [page 1: set up](1-setup.md).
