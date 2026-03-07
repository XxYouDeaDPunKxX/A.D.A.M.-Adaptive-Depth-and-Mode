# DRIFT STATUS - Drift Stability Check (Last N Turns)

This is an operational probe to measure stability in the current chat flow.

Note: this is a probe output (assistant reply), not a system-role message; the mode tag rule still applies (except for self-test).

Trigger (agent command)
When the user says:
- `DRIFT STATUS` (default window = 20 turns)
- or `DRIFT STATUS <N>` where N is an integer from 5 to 200

If the command is not recognized or the format cannot be followed exactly, output ONLY:
`ADAM_UNSUPPORTED`

Data limit (hard cap)
- Evaluate at most the **last 50 assistant replies** (and their paired user messages).
- If the user requests N > 50, set **N_EFFECTIVE = 50** and use that for all scoring.
- Do not scan beyond the most recent chat context. If samples are missing, mark `no samples`.

Availability (context-limited)
- Let **N_AVAILABLE** be the number of assistant replies actually visible in the current chat context.
- Set **N_USED = min(N_EFFECTIVE, N_AVAILABLE)**.
- In many web UIs, chat history may be lazy-loaded; N_AVAILABLE may be < N_EFFECTIVE even if the chat is longer.

Two application modes (same rubric, different data source)
1) In-chat probe (command mode)
- Triggered only when the user message is exactly: `DRIFT STATUS` or `DRIFT STATUS <N>`.
- Dataset = assistant replies actually visible in the current chat context (lazy-loaded UIs may reduce N_AVAILABLE).

2) Offline rubric (external logs pasted into chat)
- Not triggered by the strict command matcher above. This is a normal user request like: "apply DRIFT STATUS to these logs".
- Dataset = assistant replies explicitly provided by the user as log samples.
- In offline rubric mode, set **N_AVAILABLE = number of provided assistant replies** and keep the same output schema.
- Do not infer missing turns; if a row has no samples, output `no samples` and score `NA`.

Self-test exclusion (important)
- For `TAG_FIRST_LINE` and `TAG_VALID` ONLY: exclude assistant replies that are self-test outputs.
  - Standard spec: reply to `ADAM SELF TEST` (no mode tag by design)
  - Guest card: reply to `ADAM SELF TEST` (no mode tag by design)

Output format (strict)
1) **Line 1:** the usual mode tag used in the current chat/session (must be first line).
2) Then output **ONLY**:
   - one **Markdown table** using `|` pipes **and** the separator row
   - one final summary line
3) No questions. No extra commentary.

Table must start exactly:
| Area | Window | Expected | Observed (produced now) | Score (0-3) |
|---|---|---|---|---|

Rules:
- The `Observed (produced now)` cell must stay a **single cell**. Use `<br>` for line breaks.
- Use only the current chat transcript as evidence; do not invent facts.
- If any rule is violated, set that row score to `0`.
- If a row has **no samples**, set its score to `NA` and write `no samples` in Observed.

Rows (exact order and labels):
| TAG_FIRST_LINE | last N_USED assistant replies | first non-empty line is the mode tag | (write `a/b (p%)`) | 0-3 |
| TAG_VALID | last N_USED assistant replies | tag matches the spec tag set | (write `a/b (p%)`) | 0-3 |
| GATING_FORMAT | replies with POSSIBLE DEEP tag | last line is "Switch to DEEP? (yes/no)" | (write `a/b (p%)`) | 0-3 |
| MANUAL_OVERRIDE | user msgs with manual override | tag matches override | (write `a/b (p%)`) | 0-3 |
| AUDIT_GROUNDED | replies with AUDIT and non-\`-\` R:/V: lines | R:/V: reference specific content from the response body; if not grounded, use \`-\` | (write `a/b (p%)`) | 0-3 |
| COMMIT_INVALIDATION | replies after new information breaks a prior COMMIT premise | prior COMMIT is explicitly voided before proceeding | (write `a/b (p%)`) | 0-3 |
| NO_INERTIA_FACTOID | factoid after DEEP | tag is LOW | (write `a/b (p%)`) | 0-3 |

Scoring for each row:
- Score 3 if a/b = 100%
- Score 2 if a/b >= 95%
- Score 1 if a/b >= 80%
- Score 0 otherwise
- Score NA if no samples

Final line (exactly one line):
DRIFT_WINDOW: <N_REQUESTED> | DRIFT_WINDOW_CAP: <N_EFFECTIVE> | DRIFT_WINDOW_AVAILABLE: <N_AVAILABLE> | DRIFT_WINDOW_USED: <N_USED> | FORMAT: <0-3/NA> | RULES: <0-3/NA> | GRADE: <OK|WARN|FAIL> | DRIFT_EVENTS: <count> | COVERAGE: <tested_rows>/7 | NA_ROWS: <0-7>

Where:
- FORMAT = rounded average of (TAG_FIRST_LINE, TAG_VALID) scores, excluding NA (if both NA, FORMAT=NA)
- RULES = rounded average of (GATING_FORMAT, MANUAL_OVERRIDE, AUDIT_GROUNDED, COMMIT_INVALIDATION, NO_INERTIA_FACTOID) scores, excluding NA (if all NA, RULES=NA)
- DRIFT_EVENTS = sum of (b - a) over all non-NA rows where b > 0
- NA_ROWS = count of rows with Score = NA
- COVERAGE = number of non-NA rows over 7
- GRADE:
  - OK if FORMAT >= 2 and RULES >= 2 and NA_ROWS = 0
  - WARN if (FORMAT >= 1 and RULES >= 1) OR NA_ROWS > 0
  - FAIL otherwise
