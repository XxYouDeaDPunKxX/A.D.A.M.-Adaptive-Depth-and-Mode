# Lexicon (SSOT)

This file defines the official terminology used across the repo. Avoid synonyms outside this list.

Quick links:
- Most-used terms: [Most-Used Terms (SSOT)](#most-used-terms-ssot)
- Runtime & state: [Runtime and State](#runtime-and-state)
- Modes & gating: [Modes and Gating](#modes-and-gating)
- Output contract: [Output Contract](#output-contract)
- Probes, trace, and diagnostics: [Probes, Trace, and Diagnostics](#probes-trace-and-diagnostics)
- Host/compat: [Host and Compatibility](#host-and-compatibility)
- Errors: [Errors and Strict Outputs](#errors-and-strict-outputs)
- Files: [Reference Files](#reference-files)

---

## Most-Used Terms (SSOT)

| Term | Official form | Allowed variants | Avoid | Notes |
|---|---|---|---|---|
| A.D.A.M. | `A.D.A.M.` | `ADAM` (only in commands, filenames, paths) | `Adam` (as a person) | Project/protocol name. |
| mode | `MODE` | `mode` | `depth level` | Public reply state label. |
| LOW | `LOW` |  | `Lite` | Fast, low-overhead response mode. |
| MID | `MID` |  | `Normal` | Default structured mode. |
| DEEP | `DEEP` |  | `Deep mode` | Full decision support. |
| POSSIBLE DEEP | `POSSIBLE DEEP` | `MODE: MID -> POSSIBLE DEEP` | `maybe deep` | DEEP candidate state; requires gating. |
| manual override | `manual override` | `override` | `force` (as a general synonym) | `LOW` / `MID` / `DEEP` as first token; case-sensitive. |
| public core commands | `public core commands` | `core commands` | `main controls` | `ADAM PING`, `ADAM REMOUNT`, `ADAM PERSIST`, `ADAM OFF`, `ADAM ON`. |
| operator commands | `operator commands` |  | `debug commands` | `ADAM SELF TEST`, `TRACE INPUT`, `SYS STATUS`, `UNSUPPORTED WHY`. |
| BOOTSTRAP_CLASS | `BOOTSTRAP_CLASS` |  | `host class` | Public source-side bootstrap state established by `ADAM PING` in `TRANSPORT`. |
| TEXT_ONLY | `TEXT_ONLY` |  | `text mode` | Protocol operable, but execution not tentable or no stable physical bind proved. |
| BOUND_RO | `BOUND_RO` |  | `read-only bound` | Source-side physical bind established; read-only at source side. |
| BOUND_RW | `BOUND_RW` |  | `read-write bound` | Source-side physical bind established; source-side write/readback verified. |
| GHOST | `GHOST` |  | `ghost mode` | Physical bootstrap ran, but no sufficient classification basis remained. |
| PERSISTENCE_CLASS | `PERSISTENCE_CLASS` |  | `persist class` | Destination-side result of `ADAM PERSIST`; does not replace `BOOTSTRAP_CLASS`. |
| SPEC_SIGNATURE | `SPEC_SIGNATURE: ADAM_V5_SPEC_SIG` |  |  | Canonical signature line for v5 identity. |
| KERNEL_ANCHOR | `KERNEL_ANCHOR: ADAM_V5_SSOT_KERNEL` |  |  | Internal anchor used for identity and bounded discovery. |
| KERNEL_END_ANCHOR | `KERNEL_END_ANCHOR: ADAM_V5_SSOT_KERNEL_END` |  |  | End anchor for integrity and identity. |
| State Overlay | `State Overlay` | `state overlay` | `session phase` | Internal reasoning governor for long-session stability; not printed. |
| STATE | `STATE` | `internal state` | `mode` | Overlay state variable: `EXPLORE / CONVERGE / DECIDE / VERIFY`; not printed. |
| structural kernel | `structural kernel` | `kernel`, `section S` | `switch logic` | SSOT routing layer driven by input structure, not wording. |
| structural triggers | `structural triggers` |  | `keyword triggers` | Non-semantic cues used for routing, gating, and audit. |
| Action Lane | `Action Lane` | `Action` (informal) | `main lane` | Main answer body. |
| AUDIT footer | ``AUDIT`` footer | `AUDIT` block | `epistemics block` | Strict 4-line bounded audit block. |
| ACTION / RISK / BASIS | `ACTION:` / `RISK:` / `BASIS:` |  | `A/R/V` | Canonical lines inside the `AUDIT` footer. |
| audit compression | `audit compression` |  | `long audit` | Keep `AUDIT` bounded; use `-` when grounding is absent. |
| decision-complete | `decision-complete` |  | `complete answer` | In decision states, ensure at least one real guardrail exists. |
| comparison mini-table | `comparison mini-table` | `mini-table` | `comparison matrix` | Compact comparison rule in `MID`. |
| when to use what | `when to use what` | `use guidance` | `decision essay` | Short mapping from options to the right use case. |
| DEEP gating | `Switch to DEEP? (yes/no)` |  | `deep prompt` | Exact last-line consent gate to enter `DEEP`. |
| probes | `probes` | `diagnostics` | `drift probes` (as active surface) | In current public surface: `ADAM SELF TEST`, `SYS STATUS`, `TRACE INPUT`, `UNSUPPORTED WHY`. |
| ADAM PING | `ADAM PING` |  |  | Canonical visible bootstrap trigger. In `TRANSPORT`: strict 4-line bootstrap output. In `ACTIVE`: strict 2-line liveness output. |
| TRACE INPUT | `TRACE INPUT` |  | `RX receipt` | Strict forensic/operator input trace command. |
| structural classification overlay | `TRACE INPUT:` overlay | `automatic overlay` | `RX receipt` | Automatic bounded structural overlay that may appear in `MODE: MID -> POSSIBLE DEEP` and `MODE: DEEP`. |
| SOURCE_FILE_UNAVAILABLE | `SOURCE_FILE_UNAVAILABLE <FILENAME>` |  |  | Strict one-line fallback when a required external source file is unavailable. |
| ADAM_UNSUPPORTED | `ADAM_UNSUPPORTED` |  |  | Fail-closed output when strict invariants cannot be satisfied reliably. |

---

## Runtime and State

These terms describe runtime, state, and matching rules.

| Term | Official form | Meaning |
|---|---|---|
| rule-ordered runtime | `rule-ordered runtime` | Explicit priority, parsing, and output rules constrain behavior, even though execution remains probabilistic. |
| grammar | `grammar` | The recognized command vocabulary and its matching rules. |
| exact match | `exact match` | Canonical command matching rule for strict commands and probes. |
| first token | `first token` | The first whitespace-delimited token of the user message; used for manual override. |
| priority order | `priority order` | `REMOUNT_REPLAY > TRANSPORT_GATE > OFF_GATE > ACTIVE_CONTROL_COMMANDS > MANUAL_OVERRIDE > STRUCTURAL_KERNEL > STATE_DEFAULT > OUTPUT_CONTRACT`. |
| canonical string | `canonical string` | Exact command phrases that must not be paraphrased. |
| normalization | `normalization` | Limited trimming/case rules used for strict matching. |
| transport/activity state | `TRANSPORT / ACTIVE / OFF` | Public activity state machine for protocol activation and control. |
| TRANSPORT | `TRANSPORT` | File present but not yet activated; only exact `ADAM PING` and exact remount replay are available. |
| ACTIVE | `ACTIVE` | Protocol is active and normal routing applies. |
| OFF | `OFF` | Protocol is temporarily off; `ADAM ON` is required to return to `ACTIVE`. |
| state machine | `state machine` | Finite state model for `TRANSPORT / ACTIVE / OFF`, plus a separate internal reasoning overlay. |
| event | `event` | Something that triggers a state transition, e.g. exact `ADAM PING`, `ADAM OFF`, `ADAM ON`, exact remount replay. |
| invariants | `invariants` | Rules that must always hold: strict outputs, fail-closed behavior, bounded audit, exact gating line, and so on. |
| liveness-only | `liveness-only` | Exact `ADAM PING` while already `ACTIVE`; confirms activity but does not rerun `G0`. |
| fail-closed | `fail-closed` | If a strict invariant cannot be satisfied reliably, output only the strict fallback instead of approximating. |

---

## Modes and Gating

| Term | Official form | Meaning |
|---|---|---|
| mode tag | `MODE` tag | Required first line on normal replies while `ACTIVE`. |
| LOW | `LOW` | Minimal, fast replies with low overhead. |
| MID | `MID` | Default rigorous mode. |
| DEEP | `DEEP` | Full decision support mode. |
| POSSIBLE DEEP | `MODE: MID -> POSSIBLE DEEP` | Candidate state indicating DEEP may be needed. |
| DEEP gating | `Switch to DEEP? (yes/no)` | Exact last-line consent question used to enter `DEEP`. |
| hard stop | `hard stop` | If gating is used, the gating question must be the exact last line. |
| anti-loop | `anti-loop` | If user reply is not exactly `yes` or `no`, treat it as non-entry and do not repeat the gate automatically. |
| DEEP_CANDIDATE | `DEEP_CANDIDATE` | Structural kernel result indicating DEEP may be warranted. |
| MINI_TABLE_TRIGGER | `MINI_TABLE_TRIGGER` | Structural kernel result that triggers a compact comparison table in `MID`. |
| RETROGRADE_HARD | `RETROGRADE_HARD` | Structural kernel result indicating a hard retrograde condition after new constraints. |

---

## Output Contract

| Term | Official form | Meaning |
|---|---|---|
| Action Lane | `Action Lane` | Main answer body. |
| AUDIT footer | ``AUDIT`` footer | Strict bounded block appended only when `AUDIT_ON` is true. |
| bounded | `bounded` | Hard cap on meta output; `AUDIT` must remain exact and compact. |
| audit compression | `audit compression` | Keep `AUDIT` short and grounded; use `-` when a line cannot be grounded. |
| ACTION | `ACTION:` | Dominant operational next step, or `-`. |
| RISK | `RISK:` | Dominant downside/failure mode, or `-`. |
| BASIS | `BASIS:` | One compact grounding basis: check, visible source, or observable verification basis, or `-`. |
| decision-complete | `decision-complete` | In `DECIDE / VERIFY`, ensure at least one meaningful guardrail exists unless mode/contract forbids it. |
| dominant-only discipline | `dominant-only discipline` | Outside `DEEP`, keep risks/verification to dominant top-level signals instead of expanding into long audit prose. |
| comparison mini-table | `comparison mini-table` | Compact table for multi-option comparisons in `MID`. |
| when to use what | `when to use what` | Short mapping from options to likely best-fit use case. |
| publish-boundary validation | `publish-boundary validation` | Draft -> validate -> rewrite once -> fail-closed if strict invariants still fail. |
| sparse local provenance | `Sparse Local Provenance` | C5 local signal layer for eligible DEEP blocks only. |

---

## Probes, Trace, and Diagnostics

| Term | Official form | Meaning |
|---|---|---|
| ADAM SELF TEST | `ADAM SELF TEST` | Runs the external self-test file if available; otherwise returns `SOURCE_FILE_UNAVAILABLE ADAM_SELF_TEST.md`. |
| SYS STATUS | `SYS STATUS` | Runs the external system status probe if available; otherwise returns `SOURCE_FILE_UNAVAILABLE SYS_STATUS.md`. |
| TRACE INPUT | `TRACE INPUT` | Strict one-line forensic/operator trace on the last visible user message. |
| INPUT_TRACE | `INPUT_TRACE: ...` | Strict output shape returned by exact `TRACE INPUT`. |
| len | `len=<L>` | Raw character count in the traced user message as seen by the model; if unreliable, `len=0`. |
| head | `head="<H>"` | First 16 characters of the traced user message after required normalization. |
| tail | `tail="<T>"` | Last 16 characters of the traced user message after required normalization. |
| struct | `struct=[opt:<n> step:<n> crit:<n> num:<Y|N>]` | Structural summary used by strict `TRACE INPUT`. |
| structural classification overlay | `TRACE INPUT:` overlay | Automatic presentation-only structural overlay in `MODE: MID -> POSSIBLE DEEP` and some `DEEP` replies. |
| UNSUPPORTED WHY | `UNSUPPORTED WHY` | Strict one-line diagnostic probe after `ADAM_UNSUPPORTED`. |
| CAUSE | `CAUSE <CLASS>` | Strict output for `UNSUPPORTED WHY`. |

---

## Host and Compatibility

| Term | Official form | Meaning |
|---|---|---|
| host | `host` | The chat UI/platform where you load and run A.D.A.M. |
| system prompt host | `system prompt host` | A host that lets you load the protocol as active instructions. |
| upload is transport only | `upload is transport only` | File presence does not count as activation. Exact `ADAM PING` is still required. |
| local context | `local context` | The visible/mounted context the protocol can actually inspect. |
| host artifacts | `host artifacts` | UI banners, wrappers, or formatting inserted by the host, outside protocol control. |
| bootstrap boundary | `bootstrap boundary` | The bounded activation check run through exact `ADAM PING` in `TRANSPORT`. |
| protocol operability check | `protocol operability check` | Level 1 visible-context check that runs before physical classification in `G0`. |
| physical classification | `physical classification` | `G0` phase that determines public `BOOTSTRAP_CLASS` beyond `TEXT_ONLY`. |
| source-side class | `source-side class` | Public meaning of `BOOTSTRAP_CLASS`; it does not describe destination durability. |
| destination-side class | `destination-side class` | Public meaning of `PERSISTENCE_CLASS`; it does not replace `BOOTSTRAP_CLASS`. |

---

## Errors and Strict Outputs

| Term | Official form | Meaning |
|---|---|---|
| ADAM_UNSUPPORTED | `ADAM_UNSUPPORTED` | Output-only fail-closed fallback when a strict invariant cannot be satisfied reliably. |
| SOURCE_FILE_UNAVAILABLE | `SOURCE_FILE_UNAVAILABLE <FILENAME>` | Strict one-line fallback when an external source file is unavailable in current host context. |
| strict output | `strict output` | A response format that must match exactly, with no extra prose or extra lines. |
| ADAM_PING_OK | `ADAM_PING_OK` | Strict success line emitted by exact `ADAM PING`. |
| ADAM_REMOUNT_OK | `ADAM_REMOUNT_OK` | Strict success line confirming exact remount replay and return to `ACTIVE`. |
| ADAM_PERSIST_OK | `ADAM_PERSIST_OK` | Strict success line for `ADAM PERSIST` when a destination copy is written and classified. |
| PERSIST_BLOCKED | `PERSIST_BLOCKED: ...` | Strict blocked output when source or destination gate prevents `ADAM PERSIST`. |
| ADAM_PERSIST_FAIL | `ADAM_PERSIST_FAIL` | Strict failure output when persist write fails. |
| ADAM REMOUNT | `ADAM REMOUNT` | Recovery command that emits the strict remount replay capsule. |
| remount replay | `exact remount replay` | Exact replay of the emitted remount capsule body; re-enters `ACTIVE`. |
| ADAM PERSIST | `ADAM PERSIST` | Deploy-and-verify command for a physical copy of the current protocol source. |
| VERIFIED | `VERIFIED` | Destination copy written, re-opened, and matched to source. |
| WRITTEN_ONLY | `WRITTEN_ONLY` | Destination write appears to have happened, but readback or strict verification is incomplete. |
| MISMATCH | `MISMATCH` | Destination copy exists but does not match source. |
| CAUSE | `CAUSE <CLASS>` | Strict output for `UNSUPPORTED WHY`. `<CLASS>` is one of `HARD_STOP`, `AUDIT_FORMAT`, `HOST_FORMAT`, `CONTROL_STRICT`, `CONFLICT`, `UNKNOWN`. |

---

## Reference Files

Core protocol:
- [`A.D.A.M.v5.txt`](../A.D.A.M.v5.txt)

External checks:
- [`ADAM_SELF_TEST.md`](../ADAM_SELF_TEST.md)
- [`SYS_STATUS.md`](../SYS_STATUS.md)

Docs:
- [`README.md`](../README.md)
- [`docs/lexicon.md`](../docs/lexicon.md)
- [`docs/field-report-chat.md`](../docs/field-report-chat.md)
