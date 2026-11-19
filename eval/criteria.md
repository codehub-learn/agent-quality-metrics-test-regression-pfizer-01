# Triage transcript criteria

The owner’s statement that the new version “feels smarter” is not directly measurable from one run. These six criteria turn observable parts of the triage instructions into pass/fail checks. Each check uses only fields in one recorded transcript. The checks do not establish that the label or analysis is correct.

## Pass/fail criteria

| # | Type | Criterion and pass condition | Transcript fields or steps |
|---|---|---|---|
| 1 | task | **Final output has the required JSON shape.** Pass if `final_is_json` is `true` and parsing `final` yields one object with exactly these keys: `number`, `label`, `route`, `related`, `evidence`, and `reply`. Its `number` equals both top-level `number` and `issue.number`; `label` is `bug`, `enhancement`, `question`, or `docs`; `related` and `evidence` are arrays; and `reply` is a string. Otherwise fail. | Top-level `final_is_json`, `final`, `number`, and `issue.number` |
| 2 | task | **Reply stays within the stated length limit.** Pass if parsed `final.reply` contains no more than 800 Unicode code points. Count the reply string, not JSON encoding or escape characters. Otherwise fail. | `final.reply` |
| 3 | step | **Tool routing follows the declared label policy.** Pass if `final.route` matches `final.label` (`bug` → `repro-check`; `question` or `docs` → `answer`; `enhancement` → `backlog`) and the steps match: `bug` has a successful `Skill` step with `input.skill` `repro-check`; `question` or `docs` has a successful `Skill` step with `input.skill` `answer-draft`; `enhancement` has no `Skill` step. Otherwise fail. | Parsed `final.label` and `final.route`; each `steps[]` item’s `tool`, `input.skill`, and `error` |
| 4 | step | **The run successfully reads the issue file.** Pass if a `Read` step has `error` `false` and `input.file_path` exactly `issues/<number>.json`, using top-level `number`. Otherwise fail. | Top-level `number`; `steps[].tool`, `steps[].input.file_path`, and `steps[].error` |
| 5 | step | **Every cited evidence file was successfully opened.** Pass if every string in parsed `final.evidence` matches the `input.file_path` of at least one successful `Read` step (`tool` `Read`, `error` `false`). An empty evidence array passes. Otherwise fail. | `final.evidence`; `steps[].tool`, `steps[].input.file_path`, and `steps[].error` |
| 6 | task | **Related issue list respects the output limits.** Pass if parsed `final.related` contains at most three distinct integers and none equals the top-level `number`. Otherwise fail. | `final.related` and top-level `number` |

## Not checkable from one transcript alone

- **Whether the chosen label is correct.** A transcript records the agent’s label, not a human gold label or a decision rubric that settles ambiguous issues.
- **Whether the related issues are genuinely about the same problem.** The list and the search steps are visible, but validating relevance requires reviewing issue content or a human-labeled reference.
- **Whether the reply’s factual claims and technical advice are correct or useful.** A recorded source read may be truncated and does not provide a complete, independent gold answer for comparison.
- **Whether the new version is actually “smarter.”** That is a comparative claim requiring a defined quality rubric and comparable runs; it cannot be established by checking one transcript in isolation.
