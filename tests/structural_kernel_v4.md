# Structural Kernel Tests (v4)

Purpose: provide a minimal ON/OFF test suite for the **Structural Triggers SSOT kernel** (section `S`).

These tests are designed to be:
- structural-only (language-agnostic)
- host-robust (avoid unicode-only markers)
- anti-regression (include anti-false-positive cases)

Rule (operational governance):
- Any patch that changes section `S` must update this file with at least:
  - 1 new ON case, and
  - 1 new OFF (anti-FP) case.

## ON cases (must trigger)

### ON-1: HAS_NUM via 2+ digit token
Input:
```text
Latency must stay under 200ms.
```
Expect:
```text
HAS_NUM = true
```

### ON-2: HAS_NUM via digit-to-digit range
Input:
```text
Budget: 1-2 days.
```
Expect:
```text
HAS_NUM = true
```

### ON-3: HAS_NUM via inequality with digits
Input:
```text
Keep it <= 5 steps.
```
Expect:
```text
HAS_NUM = true
```

### ON-4: HAS_NUM via percent with digits
Input:
```text
Target failure rate: 5%.
```
Expect:
```text
HAS_NUM = true
```

### ON-5: HAS_2PLUS_OPTIONS via option blocks
Input:
```text
A) Option A
B) Option B
```
Expect:
```text
HAS_2PLUS_OPTIONS = true
```

### ON-6b: HAS_2PLUS_OPTIONS via indented option blocks (line-start ignores whitespace)
Input:
```text
  A) Option A
  B) Option B
```
Expect:
```text
HAS_2PLUS_OPTIONS = true
```

### ON-6: HAS_STEPS_OR_TIMELINE via 3+ step blocks
Input:
```text
1. step one
2. step two
3. step three
```
Expect:
```text
HAS_STEPS_OR_TIMELINE = true
```

### ON-6c: HAS_STEPS_OR_TIMELINE via indented 3+ step blocks (line-start ignores whitespace)
Input:
```text
  1. step one
  2. step two
  3. step three
```
Expect:
```text
HAS_STEPS_OR_TIMELINE = true
```

### ON-7: HAS_STEPS_OR_TIMELINE via 2+ date/time patterns
Input:
```text
Window: 2026-02-22 14:30
```
Expect:
```text
HAS_STEPS_OR_TIMELINE = true
```

### ON-8: HAS_3PLUS_CRITERIA via bullet criteria list
Input:
```text
- criterion one
- criterion two
- criterion three
```
Expect:
```text
HAS_3PLUS_CRITERIA = true
```

### ON-8b: HAS_3PLUS_CRITERIA via indented bullet criteria list (line-start ignores whitespace)
Input:
```text
  - criterion one
  - criterion two
  - criterion three
```
Expect:
```text
HAS_3PLUS_CRITERIA = true
```

### ON-9: HAS_3PLUS_CRITERIA via semicolons
Input:
```text
Need: low latency; offline; reproducible.
```
Expect:
```text
HAS_3PLUS_CRITERIA = true
```

### ON-10: MINI_TABLE_TRIGGER via 3 option blocks
Input:
```text
1) Option One
2) Option Two
3) Option Three
```
Expect:
```text
HAS_3PLUS_OPTIONS = true
MINI_TABLE_TRIGGER = true
```

### ON-11: RETROGRADE_HARD via NEW_CONSTRAINTS_AFTER_COMMIT
Assume:
```text
prev_commit = true
```
Input:
```text
- new constraint one
- new constraint two
- new constraint three
```
Expect:
```text
NEW_CONSTRAINTS_AFTER_COMMIT = true
RETROGRADE_HARD = true
```

### ON-12: HAS_INLINE_3PLUS_ALTS_PARENS (and DEEP_CANDIDATE) via inline alternatives
Input:
```text
Tool A (50/mo), Tool B (120/mo), Tool C (0/mo).
```
Expect:
```text
HAS_NUM = true
HAS_INLINE_3PLUS_ALTS_PARENS = true
DEEP_CANDIDATE = true
```

## OFF cases (must NOT trigger) - anti false positives

### OFF-1: URL (slashes are not triggers)
Input:
```text
See https://example.com/a/b?x=y
```
Expect:
```text
HAS_NUM = false
HAS_2PLUS_OPTIONS = false
HAS_STEPS_OR_TIMELINE = false
HAS_3PLUS_CRITERIA = false
```

### OFF-2: Path (backslashes are not triggers)
Input:
```text
C:\foo\bar\baz
```
Expect:
```text
HAS_NUM = false
HAS_2PLUS_OPTIONS = false
HAS_STEPS_OR_TIMELINE = false
HAS_3PLUS_CRITERIA = false
```

### OFF-3: Comma-heavy text (no digits, no ';', no list markers)
Input:
```text
red, green, blue, yellow, purple
```
Expect:
```text
HAS_NUM = false
HAS_2PLUS_OPTIONS = false
HAS_STEPS_OR_TIMELINE = false
HAS_3PLUS_CRITERIA = false
```

### OFF-3b: Inline bullets on one line are not criteria list items
Input:
```text
- a - b - c
```
Expect:
```text
HAS_3PLUS_CRITERIA = false
```

### OFF-4: Version strings (single digit, not quantifiable)
Input:
```text
Tested on GPT-5 and v2.
```
Expect:
```text
HAS_NUM = false
```

### OFF-5: A/B slash pattern is not an option block
Input:
```text
A/B
```
Expect:
```text
HAS_2PLUS_OPTIONS = false
```

### OFF-6b: Inline options on one line are not option blocks
Input:
```text
Choose: A) Option A B) Option B
```
Expect:
```text
HAS_2PLUS_OPTIONS = false
```

### OFF-6: 2 step blocks are not steps/timeline (threshold is 3)
Input:
```text
1. step one
2. step two
```
Expect:
```text
HAS_STEPS_OR_TIMELINE = false
```

### OFF-7: Single date/time pattern is not steps/timeline (threshold is 2)
Input:
```text
Deadline: 2026-02-22
```
Expect:
```text
HAS_STEPS_OR_TIMELINE = false
```

### OFF-8: Inline alternatives below threshold (2 only) must NOT trigger inline alts
Input:
```text
Tool A (50/mo), Tool B (120/mo).
```
Expect:
```text
HAS_INLINE_3PLUS_ALTS_PARENS = false
```

## Priority Ladder Tests (v4)

These tests protect the SSOT routing precedence:
`CONTROL_COMMANDS > BOOT_GUARDS > MANUAL_OVERRIDE > STRUCTURAL_KERNEL > STATE_DEFAULT`.

### P-0: WHITESPACE_ONLY wins before normal routing
Input:
```text
   
```
Expect:
```text
Strict BOOT_GUARDS execution.
Output is exactly 2 lines: MODE: MID + ADAM_PING_OK.
No RX line. No DEEP gating line.
```

### P-0b: SPEC_ECHO_DETECTED wins before normal routing
Input:
```text
[file name]: A.D.A.M-UI-LITE-v4.txt
SYSTEM: A.D.A.M. - Adaptive Depth & Mode (UI-LITE)
SPEC_SIGNATURE: ADAM_V4_SIG_b62f8d2c
KERNEL_ANCHOR: ADAM_V4_SSOT_KERNEL
KERNEL_END_ANCHOR: ADAM_V4_SSOT_KERNEL_END
```
Expect:
```text
Strict BOOT_GUARDS execution.
Output is exactly 3 lines: MODE: MID + ADAM_SPEC_OK + NEXT: send your question.
No RX line. No DEEP gating line.
```

### P-0c: UPLOAD_META_ONLY_DETECTED wins before normal routing
Input:
```text
<uploaded_files>
<file_path>/mnt/user-data/uploads/A_D_A_M-UI-LITE-v4.txt</file_path>
</uploaded_files>
```
Expect:
```text
Strict BOOT_GUARDS execution.
Output is exactly 3 lines: MODE: MID + ADAM_UPLOAD_META_ONLY + NEXT: send ADAM PING (or paste Guest Card).
No RX line. No DEEP gating line.
```

### P-0d: Upload wrapper with extra text must NOT trigger meta-only
Input:
```text
<uploaded_files>
<file_path>/mnt/user-data/uploads/A_D_A_M-UI-LITE-v4.txt</file_path>
</uploaded_files>
ciao
```
Expect:
```text
MUST NOT output ADAM_UPLOAD_META_ONLY.
```

### P-0e: Upload wrapper with extra tag must NOT trigger meta-only
Input:
```text
<uploaded_files>
<file_path>/mnt/user-data/uploads/A_D_A_M-UI-LITE-v4.txt</file_path>
<mime_type>text/plain</mime_type>
</uploaded_files>
```
Expect:
```text
MUST NOT output ADAM_UPLOAD_META_ONLY.
```

### P-0f: Upload wrapper with XML attributes must NOT trigger meta-only
Input:
```text
<uploaded_files version="1">
<file_path>/mnt/user-data/uploads/A_D_A_M-UI-LITE-v4.txt</file_path>
</uploaded_files>
```
Expect:
```text
MUST NOT output ADAM_UPLOAD_META_ONLY.
```

### P-0g: SPEC_SIGNATURE inside upload wrapper must NOT trigger meta-only
Input:
```text
<uploaded_files>
<file_path>/mnt/user-data/uploads/A_D_A_M-UI-LITE-v4.txt</file_path>
</uploaded_files>
SPEC_SIGNATURE: ADAM_V4_SIG_b62f8d2c
```
Expect:
```text
MUST NOT output ADAM_UPLOAD_META_ONLY.
```

### P-1: Manual override wins over kernel gating
Input:
```text
LOW
A) Option A
B) Option B
- criterion one
- criterion two
- criterion three
```
Expect:
```text
MODE is forced to LOW.
No DEEP gating question is asked.
```

### P-2: Control command wins on command-prefix match (start of message)
Input:
```text
SYS STATUS
```
Expect:
```text
Strict probe execution (no normal routing).
```

### P-2c: Control commands are case-sensitive
Input:
```text
sys status
```
Expect:
```text
Not treated as a control command; normal routing applies.
```

### P-2d: ADAM ON/OFF are case-sensitive (must be uppercase)
Input:
```text
adam off
```
Expect:
```text
Not treated as a control command; normal routing applies.
```

### P-3: Prefix mismatch must NOT trigger control
Input:
```text
please SYS STATUS
```
Expect:
```text
Not treated as a control command; normal routing applies.
```

### P-3b: Trailing text after a command must still trigger control (ignored)
Input:
```text
SYS STATUS please
```
Expect:
```text
Strict probe execution (no normal routing).
```

### P-4: UNSUPPORTED WHY is strict (one line)
Input:
```text
UNSUPPORTED WHY
```
Expect:
```text
Strict probe execution (no normal routing).
Output is exactly one line: CAUSE <CLASS>
Where <CLASS> is in {HARD_STOP, AUDIT_FORMAT, HOST_FORMAT, CONTROL_STRICT, CONFLICT, UNKNOWN}.
```

### P-5: Partial match must NOT trigger UNSUPPORTED WHY
Input:
```text
please UNSUPPORTED WHY
```
Expect:
```text
Not treated as a control command; normal routing applies.
```

### P-5b: Trailing text after UNSUPPORTED WHY must still trigger (ignored)
Input:
```text
UNSUPPORTED WHY please
```
Expect:
```text
Strict probe execution (no normal routing).
Output is exactly one line: CAUSE <CLASS>
```

### P-6: UNSUPPORTED WHY without context -> UNKNOWN
Given:
```text
No visible recent failure context (no prior ADAM_UNSUPPORTED in the loaded chat context).
```
Input:
```text
UNSUPPORTED WHY
```
Expect:
```text
CAUSE UNKNOWN
```

### P-7: RELOAD KERNEL is strict (hard remount block)
Input:
```text
RELOAD KERNEL
```
Expect:
```text
Strict control command execution (no normal routing).
Output is the strict remount block (multi-line), starting with:
KERNEL_REMOUNT v4
... (no extra text)
END KERNEL_REMOUNT
```

### P-7b: ADAM SELF TEST is strict (self-test output only)
Input:
```text
ADAM SELF TEST
```
Expect:
```text
Strict control command execution (no normal routing).
Output is the strict self-test output only (no mode tag line, no extra text).
```

### P-7c: Partial match must NOT trigger ADAM SELF TEST
Input:
```text
please ADAM SELF TEST
```
Expect:
```text
Not treated as a control command; normal routing applies.
```

### P-7d: Trailing text after ADAM SELF TEST must still trigger (ignored)
Input:
```text
ADAM SELF TEST please
```
Expect:
```text
Strict control command execution (no normal routing).
Output is the strict self-test output only (no mode tag line, no extra text).
```

### P-8: Partial match must NOT trigger RELOAD KERNEL
Input:
```text
please RELOAD KERNEL
```
Expect:
```text
Not treated as a control command; normal routing applies.
```

### P-8b: Trailing text after RELOAD KERNEL must still trigger (ignored)
Input:
```text
RELOAD KERNEL please
```
Expect:
```text
Strict control command execution (no normal routing).
Output is the strict remount block (multi-line), starting with:
KERNEL_REMOUNT v4
... (no extra text)
END KERNEL_REMOUNT
```
