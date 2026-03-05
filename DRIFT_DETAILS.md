# DRIFT DETAILS - Drift Failure Details (Last N Turns)

Note: this is a probe output (assistant reply), not a system-role message; the mode tag rule still applies (except for self-test).

Trigger (agent command)
When the user says:
- `DRIFT DETAILS` (default window = 20 turns)
- or `DRIFT DETAILS <N>` where N is an integer from 5 to 200

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
- Triggered only when the user message is exactly: `DRIFT DETAILS` or `DRIFT DETAILS <N>`.
- Dataset = assistant replies actually visible in the current chat context (lazy-loaded UIs may reduce N_AVAILABLE).

2) Offline rubric (external logs pasted into chat)
- Not triggered by the strict command matcher above. This is a normal user request like: "apply DRIFT DETAILS to these logs".
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
| Area | Window | Failures | Examples (produced now) | Score (0-3) |
|---|---|---|---|---|

Rules:
- The `Examples (produced now)` cell must stay a **single cell**. Use `<br>` for line breaks.
- Use only the current chat transcript as evidence; do not invent facts.
- Each example MUST be <= 120 characters (truncate with `...`).
- Provide at most **2 examples per row**.
- If any rule is violated, set that row score to `0`.
- If a row has **no samples**, set its score to `NA` and write `no samples` in Examples.
 - Example format (exact):
   - `ex1: "<...>"` or `no samples`
   - Optional second line: `ex2: "<...>"`

Rows (exact order and labels):
| TAG_FIRST_LINE | last N_USED assistant replies (excluding self-test for this row) | (write `f/b (p%)`) | (write up to 2 truncated assistant first lines that failed) | 0-3 |
| TAG_VALID | last N_USED assistant replies (excluding self-test for this row) | (write `f/b (p%)`) | (write up to 2 truncated assistant first lines that failed) | 0-3 |
| GATING_FORMAT | replies with POSSIBLE DEEP tag | (write `f/b (p%)`) | (write up to 2 truncated assistant last lines that failed) | 0-3 |
| MANUAL_OVERRIDE | user msgs with manual override | (write `f/b (p%)`) | (write up to 2 truncated user first lines that failed) | 0-3 |
| NO_INERTIA_FACTOID | factoid after DEEP | (write `f/b (p%)`) | (write up to 2 truncated assistant first lines that failed) | 0-3 |

Scoring for each row:
- Score 3 if failures f = 0
- Score 2 if failure rate <= 5%
- Score 1 if failure rate <= 20%
- Score 0 otherwise
- Score NA if no samples

Final line (exactly one line):
DRIFT_DETAILS_WINDOW: <N_REQUESTED> | DRIFT_DETAILS_CAP: <N_EFFECTIVE> | DRIFT_DETAILS_AVAILABLE: <N_AVAILABLE> | DRIFT_DETAILS_USED: <N_USED> | FAIL_ROWS: <0-5> | NA_ROWS: <0-5> | COVERAGE: <tested_rows>/5

Where:
- FAIL_ROWS is the count of rows with Score = 0 or 1 (excluding NA).
- NA_ROWS is the count of rows with Score = NA.
- COVERAGE is the number of non-NA rows over 5.
