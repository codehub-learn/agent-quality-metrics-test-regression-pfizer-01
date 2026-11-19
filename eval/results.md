# Recorded-run criterion results

Applied the six criteria in `eval/criteria.md` to the four requested transcripts. For `1346-1`, every criterion that requires parsed final JSON is marked fail because the final answer is not valid JSON.

Verdict is Pass only when all six criteria pass; otherwise it is Fail.

| Run | 1. JSON shape (task) | 2. Reply length (task) | 3. Routing (step) | 4. Issue read (step) | 5. Evidence opened (step) | 6. Related bounds (task) | Verdict |
|---|---|---|---|---|---|---|---|
| `702-1` | Pass | Pass (558 Unicode code points) | Pass | Pass (step 1) | Pass (empty evidence array) | Pass | **Pass** |
| `1346-1` | **Fail** (final is fenced at line 234; `final_is_json` is `false` at line 235) | **Fail** (invalid JSON, lines 234-235) | **Fail** (invalid JSON, lines 234-235; successful `repro-check` Skill step 25 does not make the final parseable) | Pass (step 1) | **Fail** (invalid JSON, lines 234-235) | **Fail** (invalid JSON, lines 234-235) | **Fail** |
| `800-1` | Pass | Pass (636 Unicode code points) | **Fail** (line 81 has `route: "answer-draft"`, expected `answer`; Skill step 7 succeeds) | Pass (step 1) | Pass (successful Read at step 6) | Pass | **Fail** |
| `487-1` | Pass | Pass (463 Unicode code points) | Pass | Pass (step 1) | Pass (empty evidence array) | Pass | **Pass** |

`800-1` passes all task-level criteria but fails at step level on routing. `1346-1` fails task-level criteria even though the substantive content appears internally aligned: the final answer is wrapped in a JSON code fence, so it is not valid JSON. Its embedded content shows the expected bug route and cites files opened successfully, but the JSON-dependent criteria fail under the specified rule.
