# Output Guardrails Tests (v4)

Purpose: a minimal, human-readable test suite for output drift guardrails that are **kernel-safe** (do not touch section `S`).

Scope:
- Rating scale guardrail: use ONLY LOW/MED/HIGH when emitting ratings/verdicts.
- Dominant-only discipline: in MODE != DEEP, keep risks/verification to a single dominant item each.

Operational governance:
- Any patch that changes these guardrails must update this file with at least:
  - 1 new MUST case, and
  - 1 new MUST NOT (anti-drift) case.

---

## Rating Scale Guardrail

### MUST-1: Ratings use only LOW/MED/HIGH
Input:
```text
Rate cost/benefit/risk for these 2 options.
```
Expect:
```text
Ratings are ONLY LOW, MED, or HIGH.
No hybrid scales (e.g., MED-HIGH) and no "VERY HIGH".
```

### MUST NOT-1: Hybrid scales are forbidden
Input:
```text
Give verdict + ratings.
```
Expect:
```text
MUST NOT emit: MED-HIGH, VERY HIGH, or similar hybrid/intensifier scales.
```

---

## Dominant-only Discipline (Body)

### MUST-2: One dominant risk + one dominant verification (MODE != DEEP)
Input:
```text
MID: Recommend A or B. Here are constraints and a timeline. Also include risks and how to verify.
```
Expect:
```text
At most 1 dominant risk item.
At most 1 dominant verification criterion.
If more are required, ask 1 branching question or propose DEEP gating (if eligible).
```

### MUST-2b: DEEP preflight capsule stays minimal (1 operational line)
Input:
```text
Compare A) and B) with these criteria:
- latency
- cost
- complexity
And provide a 3-step rollout plan (1. 2. 3.).
```
Expect:
```text
If the reply uses POSSIBLE DEEP gating, the value-first preflight capsule must include:
- exactly 1 short operational line (what to do now; what depends on DEEP).
```

### MUST-2c: Inline alternatives should be eligible for POSSIBLE DEEP gating (kernel v4)
Input:
```text
We must choose a tool. Tool A (50/mo), Tool B (120/mo), Tool C (0/mo). Recommend one.
```
Expect:
```text
Should be eligible for POSSIBLE DEEP gating (consent-based).
Must NOT auto-enter DEEP without explicit yes or manual override.
```

### MUST-3: True tie -> exactly one either/or question (no arbitrary pick)
Input:
```text
MID: Pick one approach. Both A and B match all constraints equally.
```
Expect:
```text
If picking top-1 would be arbitrary AND action-changing:
Ask exactly 1 one-line either/or question to break the tie.
Do NOT list both candidates and do NOT ask multiple questions.
```

### MUST NOT-2: Multi-item lists in the body to "cover everything"
Input:
```text
MID: Give me all risks and all verifications.
```
Expect:
```text
MUST NOT stuff multiple risks/verifications into the body by default.
Either select dominant-only OR gate/clarify OR require DEEP explicitly.
```

### MUST-4: Explicit ALL -> top-1 + 1 neutral DEEP hint line (MODE != DEEP)
Input:
```text
MID: Give me all risks and all verifications.
```
Expect:
```text
Do NOT enumerate.
Provide top-1 only, then add exactly 1 neutral exit line:
"For the full list, start your next message with DEEP."
```

### MUST NOT-3: Non-exhaustive request must NOT be treated as ALL
Input:
```text
MID: Give me risks and how to verify.
```
Expect:
```text
MUST NOT force an "ALL -> DEEP" pattern.
May still apply dominant-only discipline, but do not assume exhaustive enumeration was requested.
```

### MUST-5: Micro-edit after commit must not trigger retrograde loop
Input:
```text
MID: Recommend A or B. Commit to one choice.
Then user: Add one sentence of clarification.
```
Expect:
```text
Treat as MICRO_EDIT (purely textual): keep the COMMIT stable and apply the edit.
Must NOT force EXPLORE reset / retrograde behavior by default.
```

### MUST-5b: Semicolon below threshold can still be MICRO_EDIT
Input:
```text
Assistant: I recommend Option B. Do X then Y.
User: small tweak; ok?
```
Expect:
```text
Treat as MICRO_EDIT.
Must keep the COMMIT stable.
Must NOT trigger retrograde behavior from semicolon density alone.
```

### MUST NOT-5b: Structural delta must NOT remain MICRO_EDIT
Input:
```text
Assistant: I recommend Option B. Do X then Y.
User:
- add offline support
- reduce latency
- cap budget
```
Expect:
```text
MUST NOT treat this as MICRO_EDIT.
Must trigger retrograde handling or eligible gating behavior instead.
```

---

## Output Contract (Publish Boundary)

### MUST-6: If a MODE tag is required, it is the first line (no preamble)
Input:
```text
MID: Give me a recommendation in one paragraph.
```
Expect:
```text
First line must be a valid MODE tag (per section E).
No preamble text before the tag.
```

### MUST-7: POSSIBLE DEEP hard-stop line is exact and last (no trailing text)
Input:
```text
MID: Compare A vs B under these criteria and ask permission to go deeper.
```
Expect:
```text
If the reply uses POSSIBLE DEEP gating:
- Line 2 must start with: `RX: len=` and include `head="` and `tail="`.
- Last line must be exactly: Switch to DEEP? (yes/no)
- There must be no output after it.
```
