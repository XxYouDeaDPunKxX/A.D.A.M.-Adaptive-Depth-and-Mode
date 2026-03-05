# Simulation Pack (v4) - Field Calibration (No Telemetry)

Purpose: a repeatable, human-run simulation pack to validate that A.D.A.M. v4 stays in profile on real hosts (short context, format fragility) when external testing is limited.

This pack is NOT a routing/kernel test suite. It does NOT change section `S`.
It checks output guardrails and "time-to-first-useful" behavior under realistic prompts.

How to run (manual):
1) Load one v4 variant (Normal / UI-LITE / Guest Card) in your host.
2) Run the prompts below in order. For sequence tests, keep the same chat thread.
3) Score each case as PASS/FAIL against the "Expected invariants" only (do not grade style).
4) Record results in `tests/scorecard_v4.md`.

Key invariants being exercised:
- Dominant-only discipline (MODE != DEEP): no multi-item risk/verification lists by default.
- Explicit ALL risks/verifications: top-1 + 1 neutral DEEP hint line (MODE != DEEP).
- Dominant tie (action-changing only): 1 one-line either/or question (no arbitrary pick, no list).
- Rating scale guardrail: when emitting ratings/verdicts, use ONLY LOW/MED/HIGH.
- DEEP gating hard-stop: if POSSIBLE DEEP is used, last line is exactly `Switch to DEEP? (yes/no)`.

ASCII note:
- Do not paste bullets like "•" in prompts; use "-" only.
- Do not rely on fancy quotes.

---

## Group A - MID (daily / medium)

### A1 - Standard decision, not short, no lists
Input:
```text
MID: Help me pick A or B. Constraints: low maintenance, fast setup. Include a risk and how to verify.
```
Expected invariants:
- MODE is MID (or remains in MID; no DEEP unless kernel triggers it).
- No multi-item list of risks/verifications.
- At most 1 dominant risk and 1 dominant verification.

### A2 - Non-exhaustive "risks?" must NOT become ALL
Input:
```text
MID: Give me risks and how to verify.
```
Expected invariants:
- MUST NOT treat this as an explicit ALL request.
- Dominant-only is OK.

### A3 - Ratings must use only LOW/MED/HIGH
Input:
```text
MID: Rate cost/benefit/risk for option A vs B.
```
Expected invariants:
- If ratings/verdicts are emitted: ONLY LOW/MED/HIGH (no hybrid scales).

---

## Group B - Explicit ALL Requests (fail-closed, no enumeration)

### B1 - All risks + all verifications
Input:
```text
MID: Give me all risks and all verifications.
```
Expected invariants:
- MODE != DEEP: do NOT enumerate.
- Provide top-1 only, then add exactly one neutral exit line:
  - `For the full list, start your next message with DEEP.`

### B2 - All risks (single)
Input:
```text
MID: List all risks.
```
Expected invariants:
- Same as B1: no enumeration in MODE != DEEP, top-1 + 1-line DEEP hint.

---

## Group C - Dominant Tie

### C1 - True tie, action-changing -> 1 either/or question
Input:
```text
MID: I can choose either Path X or Path Y. Both match all constraints equally. The choice changes my implementation. What should I do?
```
Expected invariants:
- If picking top-1 would be arbitrary AND action-changing: ask exactly 1 one-line either/or question.
- Do NOT list both candidates as an answer.
- Do NOT ask multiple questions.

### C2 - Tie but NOT action-changing -> pick and close
Input:
```text
MID: X and Y are basically equivalent for me and the action is the same either way. Pick one.
```
Expected invariants:
- MUST NOT force a tie question.
- Pick one and proceed (dominant-only still applies).

---

## Group D - DEEP Gating Format (structural triggers likely)

Note: this group checks strict formatting only. Depending on host and kernel interpretation,
POSSIBLE DEEP may or may not trigger. Score PASS only if the invariants below are satisfied WHEN gating is used.

### D1 - If gating is used, hard-stop must be correct
Input:
```text
Compare A) and B) with these criteria:
- latency
- cost
- complexity
And provide a 3-step rollout plan (1. 2. 3.).
```
Expected invariants (only if the reply uses POSSIBLE DEEP):
- Line 2 MUST start with `RX: len=` and include `head="` and `tail="`.
- The last line is exactly: `Switch to DEEP? (yes/no)`
- Nothing after it.

### D3 - Inline alternatives with parentheses should trigger gating (kernel v4)
Input:
```text
We must choose a tool. Tool A (50/mo), Tool B (120/mo), Tool C (0/mo). Recommend one.
```
Expected invariants:
- The reply MUST use `MODE: MID -> POSSIBLE DEEP`.
- Line 2 MUST start with `RX: len=` and include `head="` and `tail="`.
- The last line is exactly: `Switch to DEEP? (yes/no)`
- Nothing after it.

### D2 - Non yes/no reply should be treated as no (anti-loop)
Sequence:
1) Run D1.
2) If you received `Switch to DEEP? (yes/no)`, reply with:
```text
maybe
```
Expected invariants:
- The next assistant turn MUST NOT enter DEEP.
- The gating question should NOT be repeated unless the user explicitly asks to switch to DEEP (or uses manual override).

---

## Group E - Retrograde (no stale commit)

### E1 - New constraint after commit -> no re-commit as-is
Sequence:
1) Input:
```text
MID: Recommend A or B. Commit to one choice.
```
2) Follow-up input:
```text
New constraint: budget is 10.
```
Expected invariants:
- Must NOT re-commit as-is.
- Ask exactly 1 clarifier OR propose DEEP gating (if eligible).

### E2 - Micro-edit after commit -> keep COMMIT stable (no retrograde loop)
Sequence:
1) Input:
```text
MID: Recommend A or B. Commit to one choice.
```
2) Follow-up input:
```text
Add one sentence of clarification.
```
Expected invariants:
- Treat this as a MICRO_EDIT (purely textual).
- Keep the previous COMMIT valid and apply the edit.
- Must NOT force retrograde behavior (no EXPLORE reset, no gating unless otherwise required).

---

## Group F - Host short-context / noise injection

### F1 - Noisy preamble should not cause multi-item dumping
Input:
```text
Hey, sorry for the long message. I am in a hurry. Please be helpful and cover everything. Anyway: pick A or B and include risks and how to verify.
```
Expected invariants:
- Dominant-only still holds (no "cover everything" lists by default).
- If the user explicitly requests ALL, then B1/B2 rules apply; otherwise do not treat it as ALL.

---

## Group G - Operator Recovery (control command)

### G1 - RELOAD KERNEL is strict (hard remount block)
Input:
```text
RELOAD KERNEL
```
Expected invariants:
- Strict control command output: the `KERNEL_REMOUNT v4` block (multi-line), with no extra text before/after.
