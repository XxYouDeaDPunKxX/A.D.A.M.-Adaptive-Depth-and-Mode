# SYS STATUS - System Status Check (0-3)

This is an **operational health probe** for A.D.A.M. sessions.

Goal: quickly measure whether the system is still **standing** (core invariants) and still **functional** (usable behavior) in the current chat host, *at this moment*.

Note: this is a probe output (assistant reply), not a system-role message; the mode tag rule still applies (except for self-test).

## Trigger (agent command)
When the user says exactly: `SYS STATUS`  
The assistant must run this probe and output the result **in the strict format below**.

Internal effect (not printed): re-apply the active A.D.A.M. spec as SSOT and reset State Overlay to `EXPLORE` for the next turn only.

If the assistant cannot comply with the strict format, it must output ONLY:
`ADAM_UNSUPPORTED`

## Output format (strict)
1) **Line 1:** the usual mode tag used in the current chat/session (must be first line).
2) Then output **ONLY**:
   - one **Markdown table** using `|` pipes **and** the separator row
   - one final summary line
3) No questions. No extra commentary.

Table must start exactly:
| Area | Micro-test | Expected | Observed (produced now) | Score (0-3) |
|---|---|---|---|---|

Rules:
- The `Observed (produced now)` cell must stay a **single cell**. Use `<br>` for line breaks.
- Do **NOT** invent prices, years, locations, laws, or model names in any Observed output.
- If any rule is violated, set that row score to `0`.

Rows (exact order and labels):
| TAG | first line | first line is the mode tag | (write your actual first line) | 0-3 |
| FACTOID+AUTO_LIGHT | User: capital of germany | LOW + "Berlin" only (no extras) | (produce the full reply you would give) | 0-3 |
| BANTER+LIGHT_MINIMAL | User: talking about ovens made me hungry | LOW, short, no follow-up question | (produce the full reply you would give) | 0-3 |
| DEEP_GATING | User: I need to buy an advanced oven for home | POSSIBLE DEEP tag (as defined by the current spec/variant), ends with: Switch to DEEP? (yes/no) | (produce the full reply you would give) | 0-3 |
| GATING_NO | User: no | NOT DEEP | (produce the full reply you would give) | 0-3 |
| NO_INERTIA | Assume previous assistant reply was DEEP. User: capital of germany | LOW (no inertia) | (produce the full reply you would give) | 0-3 |

Final line (exactly one line):
UPTIME: <0-3> | FUNCTIONAL: <0-3> | SCORES: TAG=<0-3>,FACTOID=<0-3>,BANTER=<0-3>,DEEP_GATING=<0-3>,GATING_NO=<0-3>,NO_INERTIA=<0-3>

Scoring:
- `UPTIME` = rounded average of (TAG, DEEP_GATING, GATING_NO)
- `FUNCTIONAL` = rounded average of (FACTOID, BANTER, NO_INERTIA)
