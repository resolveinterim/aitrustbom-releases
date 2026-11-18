# Workflow diagram

This diagram shows the full flow described in [How you use AI TrustBOM](workflow.md), including the points where a check or review sends the work back.

<p align="center"><img src="images/workflow.svg" alt="Workflow diagram. Install and initialise once, then scan, declare in trust/manual.yml, re-scan to include the declarations and check locally with the gate. If a finding cannot be fixed now, add an optional waiver, then request approval and open a pull request. GitHub checks the pull request, then the Trust approver reviews and merges. Needs attention at either gate, or changes requested or rejected by the review, return to declare. Once approved, a developer, the approver or a CI job exports the approved evidence and the recipient verifies the pack. A later record change starts again at scan." width="75%"></p>
