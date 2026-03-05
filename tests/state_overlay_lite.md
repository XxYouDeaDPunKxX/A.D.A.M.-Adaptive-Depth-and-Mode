# State Overlay Lite + Audit Hardening - Spec Tests (Given/Expect)

These are spec-level tests for the State Overlay Lite + bounded AUDIT hardening.
They are written as "Given/Expect" scenarios to validate determinism and avoid regressions.

Notes:
- These tests focus on *structural invariants* (tags, gating line, bounded AUDIT), not identical prose.
- Canonical fixed strings (tags, commands, gating) remain in English per spec.

---

## A) Override Precedence (Priority Chain)

1) Manual override DEEP wins
- Given: `DEEP analyze X`
- Expect:
  - First line tag is `◆ MODE: DEEP`
  - No `Switch to DEEP? (yes/no)` gating line

2) Manual override LOW wins even if "decision-like"
- Given: `LOW choose A or B quickly`
- Expect:
  - First line tag is `◆ MODE: LOW`
  - No DEEP gating line

3) Control command responses remain strict
- Given: `ADAM OFF`
- Expect: control output only (per spec), then plain chat afterwards (no tags when OFF)

---

## B) DEEP Candidate Threshold (Anti-Spam)

4) DEEP candidate ON: 2+ options + 3+ criteria + numbers (structural)
- Given: "I must choose between A and B, but key constraints are unknown."
- Expect:
  - Tag: `◆ MODE: MID → POSSIBLE DEEP`
  - Last line: `Switch to DEEP? (yes/no)` (HARD STOP)

5) DEEP candidate OFF: decision but no risks/uncertainty/constraints
- Given: "Choose between A and B. No constraints, no risks, no uncertainty."
- Expect:
  - Tag: `◆ MODE: MID`
  - No DEEP gating line

---

## B2) MID Comparison Mini-Table (Presentation Invariant)

5b) 3 alternatives => mini-table + "when to use what" in MID
- Given: "What's the difference between AI Studio vs Gemini website vs Google AI mode?"
- Expect:
  - Tag: `◆ MODE: MID`
  - Includes a compact mini-table comparison + a short "when to use what"

5c) 2 alternatives + quantifiable params => mini-table + "when to use what" in MID
- Given: "Compare laptop A vs B. Budget $1200, 16GB RAM, 12h battery."
- Expect:
  - Tag: `◆ MODE: MID` (or POSSIBLE DEEP if constraints/risk/uncertainty trigger gating)
  - Includes a compact mini-table comparison + a short "when to use what"

5d) 2 alternatives + 3+ criteria in bullet list => mini-table + "when to use what" in MID
- Given: "Compare A vs B.\n- must be offline\n- must be low latency\n- must be cheap"
- Expect:
  - Tag: `◆ MODE: MID` (or POSSIBLE DEEP if constraints/risk/uncertainty trigger gating)
  - Includes a compact mini-table comparison + a short "when to use what"

5e) 2 alternatives + 3+ criteria separated by semicolons => mini-table + "when to use what" in MID
- Given: "Compare A vs B; offline required; latency under 50ms; budget under $50."
- Expect:
  - Tag: `◆ MODE: MID` (or POSSIBLE DEEP if constraints/risk/uncertainty trigger gating)
  - Includes a compact mini-table comparison + a short "when to use what"

---

## C) AUDIT Policy + Compression (Bounded, No Manufacture)

6) DECIDE/VERIFY: AUDIT ON when structural triggers apply (section S mapping)
- Sequence:
  - Turn 1 user: "Compare A vs B under constraints X/Y/Z."
  - Turn 2 user: "Ok, decide."
- Expect (Turn 2):
  - Tag is `◆ MODE: MID` or `◆ MODE: MID → POSSIBLE DEEP` (if eligible)
  - If AUDIT is ON: exactly 4-line footer, bounded A/R/V, and `A:- R:- V:-` is allowed

6b) DECIDE/VERIFY: AUDIT can be OFF when not decision-like and no triggers
- Given: "Ok." (a non-decision confirmation, no estimate/high-stakes/missing params)
- Expect:
  - Tag: MODE: MID (or LOW if trivial)
  - No AUDIT footer

6c) No stale commit (retrograde guardrail)
- Sequence:
  - Turn 1 user: "Pick A or B under constraints X/Y/Z."
  - Turn 2 user: "New constraint: budget <= $200."
- Expect (Turn 2):
  - Must NOT re-commit "as-is" without updating.
  - Must ask exactly 1 YES/NO clarifier OR use DEEP gating (POSSIBLE DEEP with hard-stop).

7) EXPLORE/CONVERGE: AUDIT OFF unless structural triggers
- Given: "Explain what X is."
- Expect:
  - Tag: `◆ MODE: MID` (or LOW if trivial)
  - No AUDIT footer (unless estimate/high-stakes/missing-key-params are present)

8) Forecast trigger forces AUDIT (even in non-decision flow)
- Given: "Estimate time/cost range for X with unknown parameters."
- Expect:
  - AUDIT footer present (bounded)
  - Forecast language obeys micro-rule when AUDIT is OFF (but here AUDIT should be ON)

9) No manufacture rule
- Given: a case where there is no justified assumption/risk/verification
- Expect:
  - If AUDIT is ON, footer may be `A:- R:- V:-` (no filler content)

10) Overflow rule (do not expand)
- Given: a decision where risks/verifications cannot fit within bounded A/R/V
- Expect:
  - Footer remains bounded (no extra lines)
  - Prefer DEEP gating (if eligible) OR ask 1 clarifying YES/NO question

---

## D) Decision-Complete Guardrail (DECIDE/VERIFY)

11) Decision-complete satisfied via AUDIT
- Given: a DECIDE turn with structural decision signals and mode != LOW
- Expect:
  - At least one of: alternatives OR risks OR verification is present
  - Preferably as `R:` or `V:` in the bounded AUDIT footer

12) Decision-complete does not force content in LOW
- Given: `LOW pick A or B`
- Expect:
  - No requirement to include alternatives/risks/verification (brevity mode)

---

## E) No-Structural-Delta (Regression Protection)

13) No new always-on sections
- Expect:
  - No checkpoint lines are forced globally
  - No additional mandatory headers in every reply

14) Gating invariants unchanged
- Expect:
  - If DEEP gating is used, the gating line is the exact last line
  - Nothing is printed after the gating line
