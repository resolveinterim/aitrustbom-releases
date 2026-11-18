# Map a buyer questionnaire

From a repository with default-branch approvals that pass, export a
customer spreadsheet as UTF-8 CSV with the header `id,text` or
`id,text,section`. IDs must be unique and each question must have text.
Standard CSV quotes, embedded commas, and multiline cells are supported.

```powershell
trustbom answer --questions customer.csv
```

For a monorepo portfolio, run `trustbom answer --portfolio --questions
customer.csv`; `--fail-on high` is available when that is the configured
approval threshold. Portfolio mode verifies every system's approved graph and
refuses to publish partial answers. Each CSV question yields one combined
answer with system-labeled claims and citations. Different or unknown facts
remain explicit rather than being flattened into an unsupported assertion.

The single-system command reads the approved-baseline `trust/card.json` and
default-branch `trust/approvals.yml`, using the same medium finding threshold
and approval checks as `trustbom export`. It never reads the current branch's
draft graph as a substitute. It maps exact normalized wording from the approved
eight-question bank and built-in keyword aliases. Optionally commit
`trust/question-map.yml` for buyer-specific aliases:

```yaml
maps:
  - match: "where (is|are) .* models hosted"
    canonical: Q4_inference_region
  - match: "training|fine.tune|improve our models"
    canonical: Q3_is_customer_data_used_to_train
```

The `match` is a case-insensitive .NET regular expression against question
text normalized with Unicode compatibility normalization and whitespace/
punctuation collapsed to spaces. Patterns are bounded and use non-backtracking
matching. Canonical IDs are `Q1_do_you_use_ai`,
`Q2_models_and_vendors`, `Q3_is_customer_data_used_to_train`,
`Q4_inference_region`, `Q5_human_oversight`, `Q6_high_risk_ai_act`,
`Q7_prompt_logging`, `Q8_customer_kill_switch`, `Q9_prompt_retention`,
`Q10_provider_training_use`, and `Q11_deletion_path`. Multiple distinct
matches are ambiguous and remain unmatched; add a more specific alias rather
than guessing. `Q9`, `Q10` and `Q11` are answered from the `data_handling`
declarations in `trust/manual.yml`; a provider with no declaration answers
`unknown`. `Q3` covers all training, including the vendor's own, which no
declaration records: once any provider is declared it points to `Q10` and
states that vendor-side training is not declared, and it stays uncited (a
readiness gap that `answer` never reuses).

`export/answers.csv` contains `id,question,section,answer,cites,confidence`.
Evidence paths and the approved graph path cite each answer; the human
high-risk decision also cites `trust/approvals.yml`. `high` means an exact
canonical question with supporting evidence and a known answer; aliases and
unknown answers are `low`. **Unknown is not a negative claim.**
`export/unmatched.csv` contains `id,text,section`; Trust must answer these
separately or add an alias and rerun. Exit code `0` means no unmatched rows;
`2` means unmatched rows or an operational error (the error is written to
stderr). Output files are replaced together only after input and approval
validation. A later `trustbom export` discards stale answer files when it
replaces the export pack.

Use `--from <artifacts-dir>` to answer from verified approved sibling-repository
bundles rather than a local monorepo. Verification requires access to each
source repository's current default-branch approval; an uploaded `card.json`
is not evidence of approval by itself.

## Buyer-answer drift

To check buyer-answer drift, run
`trustbom drift [--register <path>]` (a paid command: it checks the license first, see
[license enforcement](cli/license.md#license-enforcement)) to compare explicitly supplied, reviewed
answer versions in a tracked YAML register with current graph evidence. The
default register path is `trust/answer-register.yml`. The register stores
identifiers, approved source commits and hashes, semantic field dependencies,
and tracked citations, not buyer question or answer text. The internal report
shows each supplied version, its approved source snapshot, changed facts, and
Trust's review action. Exit code `0` means the report was verified (even when
answers are affected); `2` means a register or source cannot be verified.
Previously supplied versions are immutable: append a newly reviewed version
bound to its new approved graph. The GitHub Action publishes only counts in
its existing marked PR comment, and drift never changes the gate result,
`card.json`, or `approvals.yml`. The command reference is [`drift`](cli/drift.md).

## Reviewed workbook reuse

To reuse reviewed workbook answers, run
`trustbom answer --workbook buyer.xlsx --sheet Sheet1 --id-column id
--question-column question --answer-column answer --suggest-only` to write
draft match suggestions against the vendor's reviewed answer register (see
[Buyer-answer drift](#buyer-answer-drift)) to a tracked mapping file (default
`trust/answer-reuse.yml`) and a draft report under `export/`. After Trust
reviews and marks accepted rows `status: reviewed` in Git, rerun the same
command without `--suggest-only` to export a filled copy of the workbook plus
a cited reuse report. Both the suggestions report and export report identify
the source answer version, per-system graph fingerprint and evidence paths;
they remain local and are not posted to PR comments. Unknown or uncited
answers, unverified or affected dependencies, ambiguous matches, and
unreviewed mappings remain unresolved. A new suggestion run replaces older
reuse output rather than retaining a buyer-ready workbook.

For sibling imports, `answer --from artifacts` and
`publish --from artifacts` use the same verified approved snapshots as
`pack --from artifacts`. Set a read-only `TRUSTBOM_GITHUB_TOKEN` with access
to each source repository and prepare `portfolio-import.json` as described
in the [Growth workflow guide](growth-workflow.md). Unverified artifact
content is never an approved buyer claim. The command reference is
[`answer`](cli/answer.md).
