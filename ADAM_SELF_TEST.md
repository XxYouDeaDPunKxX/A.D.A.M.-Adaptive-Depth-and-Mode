# ADAM SELF TEST

This file defines the external self-test probe for A.D.A.M. v5.

Trigger (operator command)
- `ADAM SELF TEST`

Output contract
- Output ONLY one line per test:
  - `T#: <expected> - <PASS|FAIL>`
- No mode tag.
- No prose.
- If needed, append `; required: ...`

============================================================
T) SELF-TEST CASES
============================================================

TI) Internal correctness tests

T0 Upload-only passive turn:
Case: protocol file is uploaded, user textual content is empty.
Expected: no assistant-authored protocol output; state remains TRANSPORT

T1 Transport gate after passive upload:
Case: protocol file is present, no handshake yet.
Expected: `MODE: MID` then `NEXT: send ADAM PING.`

T2 Pre-handshake operator command block:
User: `ADAM SELF TEST` while TRANSPORT.
Expected: `MODE: MID` then `NEXT: send ADAM PING.`

T3 Canonical bootstrap:
User: `ADAM PING` while TRANSPORT.
Expected: EXACTLY 4 lines; `MODE: MID`; `ADAM_PING_OK`; `CONTROL: first word LOW | MID | DEEP sets mode.`; `BOOTSTRAP_CLASS: <class>`

T3a TRANSPORT bootstrap class vocabulary:
Case: exact `ADAM PING` while TRANSPORT and `G0` completes without fail-closed.
Expected: fourth line is exactly one of:
- `BOOTSTRAP_CLASS: TEXT_ONLY`
- `BOOTSTRAP_CLASS: BOUND_RO`
- `BOOTSTRAP_CLASS: BOUND_RW`
- `BOOTSTRAP_CLASS: GHOST`

T3a1 TRANSPORT bootstrap maps non-physical source to TEXT_ONLY:
Case: exact `ADAM PING` while TRANSPORT and `G0` yields `UNBOUND`, `AMBIGUOUS`, `TEXT_ONLY`, or `WEAK_PHYSICAL`.
Expected: fourth line `BOOTSTRAP_CLASS: TEXT_ONLY`

T3a2 TRANSPORT bootstrap maps stable physical read-only source to BOUND_RO:
Case: exact `ADAM PING` while TRANSPORT and `G0` yields `BOUND`, `PHYSICAL`, `INTACT`, then `R_ONLY`.
Expected: fourth line `BOOTSTRAP_CLASS: BOUND_RO`

T3a3 TRANSPORT bootstrap maps stable physical source with write/readback to BOUND_RW:
Case: exact `ADAM PING` while TRANSPORT and `G0` yields `BOUND`, `PHYSICAL`, `INTACT`, then `RWP_VERIFIED`.
Expected: fourth line `BOOTSTRAP_CLASS: BOUND_RW`

T3a4 TRANSPORT bootstrap emits GHOST when G0 runs but no stable public class remains:
Case: exact `ADAM PING` while TRANSPORT and `G0` executes fully, but bounded discovery yields no usable roots or later internal outputs are inconsistent without producing `TEXT_ONLY`, `BOUND_RO`, or `BOUND_RW`.
Expected: fourth line `BOOTSTRAP_CLASS: GHOST`

T3a5 TRANSPORT bootstrap fail-closed is not a bootstrap class:
Case: exact `ADAM PING` while TRANSPORT and `G0` cannot execute or its strict invariants cannot be satisfied.
Expected: output ONLY `ADAM_UNSUPPORTED`

T3a6 Non-intact fingerprint never surfaces publicly:
Case: exact `ADAM PING` while TRANSPORT and `G0` reaches `PARTIAL` or `MISMATCH` at `L4`.
Expected: fourth line `BOOTSTRAP_CLASS: TEXT_ONLY`; no public `BOUND_UNSTABLE`

T3a7 L5 FAILED does not elevate to BOUND_RO:
Case: exact `ADAM PING` while TRANSPORT and `G0` yields `BOUND`, `PHYSICAL`, `INTACT`, then `FAILED` at `L5`, without fail-closed.
Expected: no `BOOTSTRAP_CLASS: BOUND_RO` and no `BOOTSTRAP_CLASS: BOUND_RW`; if no other stable public class remains, fourth line `BOOTSTRAP_CLASS: GHOST`

T3a8 Multiple anchor occurrences in one file do not make L2 ambiguous:
Case: within `G0.1` bounds, exactly one distinct candidate file path contains `KERNEL_ANCHOR`, but that same file contains the anchor more than once.
Expected: `L2 = BOUND`, not `AMBIGUOUS`; later roots are not inspected after that unique sufficient candidate is found

T3a9 Early non-physical root does not stop bounded scan:
Case: an earlier bounded root yields only non-physical or no candidate results, while a later bounded root yields exactly one sufficient physical candidate.
Expected: `G0` continues the bounded scan to the later root; does not stop on the earlier non-physical result alone

T3a10 DNA_BIND requires full anchor tuple:
Case: a candidate file contains `KERNEL_ANCHOR` only, but lacks `SPEC_SIGNATURE` or `KERNEL_END_ANCHOR`.
Expected: `L2` does not elevate to `BOUND` from partial anchor presence alone

T3a11 L5 manifest probe uses sibling temp only:
Case: `L5` runs on a physically bound candidate.
Expected: write/readback probe is performed on a sibling temporary file only; the source file itself is not modified by the probe

T3b Liveness ping while ACTIVE:
User: `ADAM PING` while ACTIVE.
Expected: EXACTLY 2 lines; `MODE: MID`; `ADAM_PING_OK`; `G0` is not re-executed

T3c TRANSPORT bootstrap hard stop:
Case: exact `ADAM PING` while TRANSPORT.
Expected: assistant-authored output ends exactly after line 4; no extra assistant-authored prose, citations, notes, or explanations; no localization, paraphrase, line merge, or line-collapse of the 4 contract lines

T3d `ADAM OFF` requires handshake first in TRANSPORT:
User: `ADAM OFF` while TRANSPORT.
Expected: EXACTLY 2 lines; `MODE: MID`; `NEXT: send ADAM PING.`

T3d `ADAM ON` requires handshake first in TRANSPORT:
User: `ADAM ON` while TRANSPORT.
Expected: EXACTLY 2 lines; `MODE: MID`; `NEXT: send ADAM PING.`

T3e `ADAM REMOUNT` requires handshake first in TRANSPORT:
User: `ADAM REMOUNT` while TRANSPORT.
Expected: EXACTLY 2 lines; `MODE: MID`; `NEXT: send ADAM PING.`

T3g `SYS STATUS` requires handshake first in TRANSPORT:
User: `SYS STATUS` while TRANSPORT.
Expected: EXACTLY 2 lines; `MODE: MID`; `NEXT: send ADAM PING.`

T3j `UNSUPPORTED WHY` requires handshake first in TRANSPORT:
User: `UNSUPPORTED WHY` while TRANSPORT.
Expected: EXACTLY 2 lines; `MODE: MID`; `NEXT: send ADAM PING.`

T3k `TRACE INPUT` requires handshake first in TRANSPORT:
User: `TRACE INPUT` while TRANSPORT.
Expected: EXACTLY 2 lines; `MODE: MID`; `NEXT: send ADAM PING.`

T4 Post-handshake reset:
After TRANSPORT -> ACTIVE via `ADAM PING`, or after exact remount replay confirmation, next routed turn starts fresh; no pending gating, no cached gating reference, no pre-handshake carryover.
Expected: clean AUTO/MANUAL routing; `BOOTSTRAP_CLASS` does not alter state transition semantics by itself

T4c ACTIVE liveness ping does not reset state:
Case: state is ACTIVE and an exact `ADAM PING` occurs.
Expected: EXACTLY 2 lines; `MODE: MID`; `ADAM_PING_OK`; no activation reset side effects

T4b Unknown activation state defaults to TRANSPORT:
Case: visible context does not establish ACTIVE or OFF.
Expected: transport/activity state defaults to TRANSPORT

T5 LOW factoid:
User: `capital of germany`
Expected: `MODE: LOW`

T6 Standard MID:
User: `in your opinion is it worth buying a 100 euro projector?`
Expected: `MODE: MID`

T7 Structural DEEP candidate:
User:
`Compare A) and B)
Budget: 100
Need:
- speed
- reliability
- low waste`
Expected: `MODE: MID -> POSSIBLE DEEP`; no RX line; required: final line `Switch to DEEP? (yes/no)`

T8 Anti-loop after gating:
If gating is pending and the user replies anything other than exact `yes` or exact `no`,
Expected: NOT `MODE: DEEP`; gating question not repeated automatically

T8b Pending gating expires after one non-yes/no turn:
If gating is pending and the immediately following user turn is `continue`,
Expected: cached gating reference discarded; later exact `yes` is orphaned unless a new gating turn occurs

T8c DEEP entry after exact `yes` contains no RX:
Case: gating is pending and the user replies exactly `yes`.
Expected: `MODE: DEEP`; no RX line

T8d DEEP entry after exact `yes` preserves gated-turn language:
Case: the gated exchange is in Italian and the user then replies exactly `yes`.
Expected: DEEP reply remains in Italian; exact `yes` alone does not switch output language to English

T8e Exact control acknowledgement with no preceding non-strict assistant reply falls back to B1:
Case: a user turn contains only an exact control acknowledgement, but no preceding non-strict assistant reply in the same routed exchange is visible.
Expected: no forced language inheritance; normal B1 language handling applies

T9 Manual override LOW:
User: `LOW give me pros and cons in two lines`
Expected: `MODE: LOW`

T10 Manual override MID:
User: `MID o che bello non sono pazzo`
Expected: `MODE: MID`

T11 Manual override boundary strict:
User: `MID, o che bello`
Expected: NOT manual override

T12 Manual DEEP contains no RX:
User: `DEEP explain caching briefly`
Expected: `MODE: DEEP`; no RX line

T13 No inertia:
After DEEP, a new factoid turn must be LOW.
Expected: `MODE: LOW`

T14 Orphan yes:
User: `yes` with no pending gating.
Expected: NOT `MODE: DEEP`

T15 Retrograde invalidation:
Turn 1 contains explicit COMMIT.
Turn 2 adds structural constraints that break a required premise.
Expected: prior COMMIT explicitly invalidated; no silent carryover

T15a Ambiguous overlay history defaults to EXPLORE:
Case: ACTIVE is established, but visible context is insufficient to infer prior overlay state.
Expected: overlay state defaults to EXPLORE

T15b MICRO_EDIT textual clarification:
Turn 1 contains explicit COMMIT.
Turn 2: `for clarity, I mean the second point`
Expected: MICRO_EDIT; keep COMMIT stable; do not force EXPLORE

T15c Numeric correction is not MICRO_EDIT:
Turn 1 contains explicit COMMIT.
Turn 2: `I meant 150 euro`
Expected: NOT MICRO_EDIT; do not silently preserve the prior COMMIT if the new number changes a required premise

T16 Grounded AUDIT:
If body provides no real grounding for RISK or BASIS,
Expected: corresponding AUDIT line becomes `-`

T16b Grounded AUDIT ACTION may be `-`:
If AUDIT is ON and the body contains no operational next step,
Expected: `ACTION:` may be `-`

T16c DEEP reply with eligible blocks emits C5 signals per current grammar:
Case: DEEP reply uses eligible top-level blocks with dominant local provenance classes.
Expected: each eligible block is closed by exactly one correctly placed C5 signal, or no signal if the C5 conditions do not pass

T16c1 Explicit inferential bridge emits `infer`:
Case: an eligible DEEP block explicitly derives a conclusion from visible premises, with no unresolved checkpoint.
Expected: `infer`

T16c2 Explicit inferential bridge plus unresolved checkpoint emits `infer [pending]`:
Case: an eligible DEEP block explicitly derives a conclusion from visible premises and explicitly names an unresolved checkpoint in the block text.
Expected: `infer [pending]`

T16c3 Surfaced non-visible premise emits `depends: <specific surfaced premise>`:
Case: an eligible DEEP block explicitly depends on a specific non-visible premise, with no unresolved checkpoint.
Expected: `depends: <specific surfaced premise>`

T16c4 Surfaced non-visible premise plus unresolved checkpoint emits `depends: <specific surfaced premise> [pending]`:
Case: an eligible DEEP block explicitly depends on a specific non-visible premise and explicitly names an unresolved checkpoint in the block text.
Expected: `depends: <specific surfaced premise> [pending]`

T16c5 Precedence favors `depends:` over `infer`:
Case: an eligible DEEP block independently satisfies both `depends:` and `infer`.
Expected: emit only the highest-ranking `depends:` variant that passes, using `[pending]` only if the pending conditions also pass

T16c6 Generic caution does not create `[pending]`:
Case: an eligible DEEP block contains warning, limitation, or risk language without an explicit unresolved checkpoint.
Expected: no `[pending]`; emit a lower valid signal or no signal

T16c7 Context-only or mixed-support block may emit no signal:
Case: an eligible DEEP block reports visible content without an explicit inferential bridge, or mixes support classes with no single dominant class.
Expected: no signal unless the block is split or one valid class clearly dominates

T16d DEEP reply with no eligible blocks emits no C5 signals:
Case: DEEP reply is valid free prose with no eligible top-level blocks.
Expected: no C5 signals; reply remains valid

T16e C5 signal classifies local support only:
Case: a block emits `infer` or `depends:` in visible context.
Expected: the signal classifies local support only; it does not imply host health, complete context integrity, or absence of truncation

T16f C5 signals do not appear in LOW, MID, or strict outputs:
Case: LOW, MID, strict command, or probe output.
Expected: no C5 signals

T16g C5 signal closes the eligible block before later top-level non-eligible content:
Case: an eligible block is followed by top-level non-eligible content before `AUDIT`.
Expected: the signal closes the eligible block before that later content; the later content remains unmarked

T16g1 C5 signal must not be appended inline to block text:
Case: an otherwise eligible DEEP block appends `infer` or `depends:` on the same line as block content.
Expected: invalid placement; the signal must appear on its own line immediately after the block's final content line

T16h Nested sub-items do not receive their own C5 signals:
Case: an eligible top-level block contains nested sub-items.
Expected: only the top-level block may receive a signal; nested sub-items remain unmarked

T16i C5 signals never appear inside AUDIT:
Case: AUDIT is present in a DEEP reply that also uses C5 signals.
Expected: no C5 signal lines inside the 4-line AUDIT block

T17 Scope integrity:
If response lacks full scope support,
Expected: no completeness or exhaustive-checking claim

T18 REMOUNT strict output:
User: `ADAM REMOUNT` while ACTIVE.
Expected: no mode tag; remount capsule body only

T19 REMOUNT replay:
User re-pastes the exact remount capsule body from any prior state, without markdown code fences.
Expected: EXACTLY 3 lines; `MODE: MID`; `ADAM_REMOUNT_OK`; `NEXT: send your question.`

T19a REMOUNT replay from OFF:
State is OFF.
User re-pastes the exact remount capsule body from G4, without markdown code fences.
Expected: EXACTLY 3 lines; `MODE: MID`; `ADAM_REMOUNT_OK`; `NEXT: send your question.`

T19b REMOUNT replay with markdown code fences does not match:
User re-pastes the capsule body wrapped in markdown code fences.
Expected: replay not detected; no `ADAM_REMOUNT_OK`; continue under normal routing for the current transport/activity state

T19c REMOUNT replay never emits BOOTSTRAP_CLASS:
Case: exact remount replay is detected from any prior state.
Expected: EXACTLY 3 lines only; no fourth line; no `BOOTSTRAP_CLASS`

T19d Structural booleans do not persist across turns:
Turn 1 is a structural DEEP candidate.
Turn 2 is plain `ok`
Expected: structural booleans are recomputed from turn 2 only; no inherited `DEEP_CANDIDATE` or `HAS_NUM`

T20 OFF strict output:
User: `ADAM OFF` while ACTIVE.
Expected: EXACTLY 2 lines; `MODE: MID`; `A.D.A.M. off.`

T20b OFF idempotence:
User: `ADAM OFF` while OFF
Expected: EXACTLY 2 lines; `MODE: MID`; `A.D.A.M. off.`

T21 OFF behavior:
While OFF, normal user chat is plain chat and does not use A.D.A.M. tags.
Expected: plain chat

T22 ON strict output:
User: `ADAM ON` while OFF
Expected: EXACTLY 2 lines; `MODE: MID`; `A.D.A.M. on.`

T22b Command/probe while OFF:
User: any exact command or probe except `ADAM ON`
Expected: EXACTLY 2 lines; `MODE: MID`; `A.D.A.M. off.`

T22c ON idempotence:
User: `ADAM ON` while ACTIVE
Expected: EXACTLY 2 lines; `MODE: MID`; `A.D.A.M. on.`

T22d PING while OFF:
User: `ADAM PING` while OFF
Expected: EXACTLY 2 lines; `MODE: MID`; `A.D.A.M. off.`

T24 Missing probe file:
User: `SYS STATUS` while ACTIVE with no `SYS_STATUS.md` available
Expected: `SOURCE_FILE_UNAVAILABLE SYS_STATUS.md`

T25 UNSUPPORTED WHY strict output:
User: `UNSUPPORTED WHY` while ACTIVE.
Expected: one line only; `CAUSE <CLASS>`

T25b TRACE INPUT strict output while ACTIVE:
Turn 1 user:
`Compare A) and B)
Budget: 100
Need:
- speed
- reliability
- low waste`
Turn 2 user: `TRACE INPUT`
Expected: EXACTLY 1 line in the form `INPUT_TRACE: len=<L> head="<H>" tail="<T>" struct=[opt:<n> step:<n> crit:<n> num:<Y|N>]`

T25c TRACE INPUT is unavailable without a visible prior user-authored turn:
Case: no prior user-authored turn is visible before `TRACE INPUT`.
Expected: `INPUT_TRACE: UNAVAILABLE`

T25d TRACE INPUT struct example with options, criteria, and number:
Case: prior user message has 2 option blocks, 3 criteria items, and one number.
Expected: `struct=[opt:2 step:0 crit:3 num:Y]`

T25e TRACE INPUT struct example with no structure:
Case: prior user message has no option blocks, no step blocks, no criteria items, and no number.
Expected: `struct=[opt:0 step:0 crit:0 num:N]`

T25f TRACE INPUT struct example with steps and date:
Case: prior user message has 3 step blocks and a date.
Expected: `struct=[opt:0 step:3 crit:0 num:Y]`

T25g TRACE INPUT while OFF:
User: `TRACE INPUT` while OFF.
Expected: EXACTLY 2 lines; `MODE: MID`; `A.D.A.M. off.`

T_G8_1 SOURCE_UNSAFE gate:
Case: ACTIVE holds, but `BOOTSTRAP_CLASS = TEXT_ONLY` or internal source fingerprint is not `INTACT`.
Expected: EXACTLY 2 lines; `MODE: MID`; `PERSIST_BLOCKED: SOURCE_UNSAFE`

T_G8_2 DEST_NONE gate:
Case: source is safe, but no destination path passes `DEST_CHALLENGE`.
Expected: EXACTLY 2 lines; `MODE: MID`; `PERSIST_BLOCKED: NO_DEST`

T_G8_3 AMBIGUOUS_DEST gate:
Case: source is safe, but destination ranking cannot produce a single best bound path.
Expected: EXACTLY 2 lines; `MODE: MID`; `PERSIST_BLOCKED: AMBIGUOUS_DEST`

T_G8_4 VERIFIED roundtrip:
Case: source is safe; `WRITE + ECHO + MATCH` all succeed.
Expected: EXACTLY 4 lines; `MODE: MID`; `ADAM_PERSIST_OK`; `PATH: <dest_path>`; `PERSISTENCE_CLASS: VERIFIED`

T_G8_5 WRITTEN_ONLY does not imply VERIFIED:
Case: `WRITE` succeeds, but `ECHO` is unavailable or not strictly provable.
Expected: EXACTLY 4 lines; `MODE: MID`; `ADAM_PERSIST_OK`; `PATH: <dest_path>`; `PERSISTENCE_CLASS: WRITTEN_ONLY`

T_G8_6 Explicit mismatch:
Case: `WRITE` succeeds, `ECHO` succeeds, and `MATCH` fails.
Expected: EXACTLY 4 lines; `MODE: MID`; `ADAM_PERSIST_OK`; `PATH: <dest_path>`; `PERSISTENCE_CLASS: MISMATCH`

T_G8_7 Write failure:
Case: source is safe, destination is bound, but `WRITE` fails.
Expected: EXACTLY 2 lines; `MODE: MID`; `ADAM_PERSIST_FAIL`

T_G8_8 OFF gate:
User: `ADAM PERSIST` while OFF.
Expected: EXACTLY 2 lines; `MODE: MID`; `A.D.A.M. off.`

T_G8_9 TRANSPORT gate:
User: `ADAM PERSIST` while TRANSPORT.
Expected: EXACTLY 2 lines; `MODE: MID`; `NEXT: send ADAM PING.`

TH) Host resilience tests

T26 Truncated protocol context fail-closed:
Case: visible protocol context is truncated or incomplete before a section required for a strict invariant.
Expected: `ADAM_UNSUPPORTED`

TH1 Gating survives ACTIVE liveness ping:
Case: a visible `MODE: MID -> POSSIBLE DEEP` reply leaves gating pending; while state is ACTIVE, the user sends exact `ADAM PING`, then exact `yes`.
Expected: `ADAM PING` returns EXACTLY 2 lines; `MODE: MID`; `ADAM_PING_OK`; pending gating survives; subsequent exact `yes` enters `MODE: DEEP`

TH2 REMOUNT replay tolerates per-line edge whitespace:
Case: user re-pastes the exact remount capsule body with leading and/or trailing whitespace added on one or more capsule lines, but with no other textual changes.
Expected: EXACTLY 3 lines; `MODE: MID`; `ADAM_REMOUNT_OK`; `NEXT: send your question.`

TH3 REMOUNT replay rejects partial or truncated capsule:
Case: user re-pastes a capsule that starts with `REMOUNT_BEGIN` but omits `REMOUNT_END`, or is missing one or more required internal lines.
Expected: replay not detected; no `ADAM_REMOUNT_OK`; continue under normal routing for the current transport/activity state

TH4 REMOUNT replay rejects internal drift with sentinels intact:
Case: user re-pastes a capsule with `REMOUNT_BEGIN` and `REMOUNT_END` intact, but one internal line differs from G4.
Expected: replay not detected; no `ADAM_REMOUNT_OK`; continue under normal routing for the current transport/activity state

TH5 Empty turn in TRANSPORT remains blocked:
Case: state is TRANSPORT and the user sends an empty or whitespace-only turn.
Expected: EXACTLY 2 lines; `MODE: MID`; `NEXT: send ADAM PING.`

TH6 Apparent continuity does not establish ACTIVE:
Case: visible context includes prior reassurance, prose claiming A.D.A.M. is active, or normal tagged replies, but no visible exact activation, exact OFF, or exact remount replay event.
Expected: transport/activity state defaults to TRANSPORT; no presumed ACTIVE from apparent continuity alone; a subsequent exact `ADAM PING` follows TRANSPORT `G3` and includes the exact fourth line `BOOTSTRAP_CLASS: <class>`

TH7 TRACE INPUT binds to the traced turn, not the command turn:
Case: the prior user-authored turn contains structure and/or numbers, while the literal command turn is exact `TRACE INPUT`.
Expected: `len`, `head`, `tail`, and `struct` reflect the prior user-authored turn, not the text `TRACE INPUT`

TH8 REMOUNT replay rejects extra lines outside the sentinels:
Case: user re-pastes the exact remount capsule body but includes any extra line before `REMOUNT_BEGIN` or after `REMOUNT_END`.
Expected: replay not detected; no `ADAM_REMOUNT_OK`; continue under normal routing for the current transport/activity state

TP) Automatic `TRACE INPUT` human projection tests

TP1 Automatic overlay appears in `MODE: MID -> POSSIBLE DEEP`:
Case: a current user turn is a structural DEEP candidate.
Expected: first line `MODE: MID -> POSSIBLE DEEP`; second line begins `TRACE INPUT:` and is derived from the current user turn

TP2 DEEP entry after exact `yes` emits no overlay:
Case: a visible `MODE: MID -> POSSIBLE DEEP` reply leaves gating pending; the user then replies exact `yes`.
Expected: `MODE: DEEP`; no automatic `TRACE INPUT:` overlay

TP3 Exact manual override `DEEP` with no trailing source emits no overlay:
Case: the user reply is exact `DEEP` with no trailing content, and DEEP is entered outside the `yes` gating path.
Expected: no automatic `TRACE INPUT:` overlay

TP4 Manual override `DEEP` with trailing content uses overlay-only normalization:
Case: the user reply is `DEEP confrontami le tre opzioni sui vincoli economici`.
Expected: the automatic `TRACE INPUT:` overlay is derived from `confrontami le tre opzioni sui vincoli economici`; the normalization applies only to the automatic overlay source, not to the raw turn used by strict `TRACE INPUT`

TP5 DEEP entered by one-turn inheritance emits no overlay:
Case: `MODE: DEEP` is inherited only through the one-turn exception in F9.
Expected: no automatic `TRACE INPUT:` overlay

TP6 Automatic overlay is absent outside POSSIBLE DEEP and DEEP:
Case: LOW, MID, strict command/probe output, TRANSPORT, OFF, AUDIT, or `ADAM REMOUNT` capsule.
Expected: no automatic `TRACE INPUT:` overlay

TP7 Zero-structure automatic overlay is omitted:
Case: the selected overlay source turn yields `struct=[opt:0 step:0 crit:0 num:N]`.
Expected: no automatic `TRACE INPUT:` overlay

TP7a Human projection thresholds match kernel for criteria and steps:
Case: selected overlay source turn yields `crit=2` or `step=2` without meeting any other structural chip threshold.
Expected: no `[vincoli da rispettare]` and no `[passi o ordine]`; if no other structural chip threshold is met, omit the automatic overlay

TP8 Automatic overlay never implies explicit command invocation:
Case: the assistant emits the automatic overlay in `MODE: MID -> POSSIBLE DEEP` or `MODE: DEEP` without the user sending exact `TRACE INPUT`.
Expected: no claim that strict `TRACE INPUT` was explicitly invoked; the overlay remains presentation-only

TP9 Strict `TRACE INPUT` remains unchanged:
Case: user sends exact `TRACE INPUT` while ACTIVE.
Expected: strict one-line machine-readable output remains unchanged from G3
