# Failure modes of the triage agent

Source: all 121 files in `data/transcripts/`, scanned with a PowerShell script that parses each file's `steps` and `final` (ConvertFrom-Json) and checks the signals in the failure-modes skill. Counts are the number of listed run ids. Rules came from `data/triage-agent/triage-agent.md`. Nothing in `data/` was edited.

## Catalogue

| Failure mode | Run ids | Count | Evidence (quoted from transcript) |
|---|---|---|---|
| F1. Final message is not one bare JSON object, wrapped in a code fence only | 1346-1, 186-1, 755-1 | 3 | `final` starts with "```json\n{\"number\": 755, \"label\": \"docs\", \"route\": \"answer\", ..." |
| F2. Final message has prose around the JSON (with or without a fence) | 310-2, 583-1, 818-1 | 3 | 818-1: "Reproducible as written. The report has the snippet, the Mockito version (2.2.28), the stack trace, and the contrast with `Map`." |
| F3. Shell command attempted (forbidden: "do not run shell commands") | 487-1, 508-1, 714-1, 998-1 | 4 | Bash input `ls "mockito" \| head -50` (508-1) |
| F4. Route not in the allowed set (backlog, answer, repro-check) | 800-1 | 1 | `"label": "question", "route": "answer-draft"` |

Method for F1/F2: the `final` is trimmed and parsed as JSON; a failure to parse, or a leading code fence, is flagged. Fenced-only vs prose was then split by reading the start of each `final`. Total flagged: 6 (3 + 3). 310-2 and 583-1 contain a JSON object plus prose (583-1 also has a fence); 310-2 is 823 chars of prose-led text.
Method for F3: `steps` where `tool` matches `Bash|PowerShell|Shell`. Method for F4: `route` from the parsed final, compared with the allowed set.

Signals checked with zero hits (0 runs):
- `label` outside bug, enhancement, question, docs: 0.
- Route that does not match the label policy (bug to repro-check, question/docs to answer, enhancement to backlog), apart from F4: 0. Pairs among the 115 parseable finals: bug/repro-check 41, docs/answer 15, enhancement/backlog 46, question/answer 12, question/answer-draft 1.
- Required skill not loaded (repro-check for bug, answer-draft for answer/question/docs; none for backlog): 0 among the 115 parseable finals. Backlog runs loading a skill: 0.
- `reply` longer than 800 characters: 0 among parseable finals. The 6 unparseable finals were not measured for `reply` length.

No failure found: the runs not listed in F1 to F4 (121 minus 14 listed ids = 107 by this count; check that F1 to F4 ids do not overlap, they don't) have none of the signals. Caveat: the 6 unparseable finals were only judged on format, and their label/route/skill were not checked by script.

## Unclassified

- 930-1: the agent called a tool named `Output` with empty input; the result was "No such tool available: Output". Final answer was still valid JSON. Fits no signal.
- 27 runs call a `ReportFindings` tool (not mentioned in the agent's instructions); 10 of those calls failed with an `InputValidationError`. Fits no signal, since the agent was not told to avoid it.
- Related issue numbers that look odd (for example 930-1 has `"related": [1044, 2868]`, and 2868 is a later number than the issue). Not verified against `issue-index.tsv`, not a defined signal.

## Could come from the environment, not the agent

- F3 (Bash attempts): the result was "Bash is disabled for this session". The agent was told not to run shell commands, so the attempt is the agent's, but the blocked call itself is the environment's policy. The agent did continue with other tools afterwards.
- `Read` errors, 69 calls across the runs: 58 are "EISDIR: illegal operation on a directory, read 'mockito'" (agent read a folder as a file) and 10 are "File does not exist". These are mostly agent mistakes in picking paths, but "File does not exist" can also mean the fixture folder was not laid out as the agent expected.
- 3 `Grep` errors ("Path does not exist: mockito\...") for paths the agent guessed. Agent guesses, possibly a different source layout.
- 310-2: the final says "I couldn't open any source under `mockito/`", which suggests the source was not reachable in that run. Environment is plausible; not verified.
- `Output` and `ReportFindings` failures (unclassified list): likely tool set mismatch in the harness.
- F1, F2 and F4 are format and policy slips with no tool error behind them: the agent.
