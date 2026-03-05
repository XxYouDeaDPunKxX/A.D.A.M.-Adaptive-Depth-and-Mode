# Lexicon (SSOT)

This file defines the official terminology used across the repo. Avoid synonyms outside this list.

Quick links:
- Most-used terms: [Most-Used Terms (SSOT)](#most-used-terms-ssot)
- Runtime & parsing: [Runtime and Parsing](#runtime-and-parsing)
- Modes & gating: [Modes and Gating](#modes-and-gating)
- Output contract: [Output Contract](#output-contract)
- Probes & metrics: [Probes and Metrics](#probes-and-metrics)
- Host/compat: [Host and Compatibility](#host-and-compatibility)
- Errors: [Errors and Strict Outputs](#errors-and-strict-outputs)
- Files: [Reference Files](#reference-files)

---

## Most-Used Terms (SSOT)

| Term | Official form | Allowed variants | Avoid | Notes |
|---|---|---|---|---|
| A.D.A.M. | `A.D.A.M.` | `ADAM` (only in filenames/paths) | `Adam` (as a person) | Project/protocol name. |
| mode | `MODE` | `mode` | `depth level` | Execution state label. |
| LOW | `LOW` |  | `Lite` | Fast/low overhead response mode. |
| MID | `MID` |  | `Normal` | Default structured mode. |
| DEEP | `DEEP` |  | `Deep mode` | Full decision support (gated). |
| POSSIBLE_DEEP | `POSSIBLE DEEP` | `MID -> POSSIBLE DEEP` (UI-LITE) | `maybe deep` | DEEP candidate state (gating required). |
| manual override | `manual override` | `override` | `force` (as a synonym in spec text) | `LOW`/`MID`/`DEEP` as first token (case-sensitive). |
| control command | `control command` |  | `instruction` | `ADAM ON`/`ADAM OFF`, probes, self-test. |
| BOOT_GUARDS | `BOOT_GUARDS` |  |  | Pre-routing deterministic guards (whitespace-only + spec-echo). Spec-echo uses `SPEC_SIGNATURE` + both kernel anchors + `SYSTEM: A.D.A.M.` to avoid host wrapper ambiguity. |
| SPEC_SIGNATURE | `SPEC_SIGNATURE: ADAM_V4_SIG_b62f8d2c` |  |  | Unique signature line for robust spec-echo detection across hosts that wrap/inline attachments. |
| KERNEL_ANCHOR | `KERNEL_ANCHOR: ADAM_V4_SSOT_KERNEL` |  |  | Presence anchor used to detect kernel truncation/absence in short-context hosts. |
| KERNEL_END_ANCHOR | `KERNEL_END_ANCHOR: ADAM_V4_SSOT_KERNEL_END` |  |  | End-of-file integrity marker. If missing, the kernel text may be truncated (use `RELOAD KERNEL` to re-inject). |
| State Overlay | `State Overlay` | `state overlay` | `session phase` | Internal governor for long-session determinism (not printed). |
| STATE | `STATE` | `internal state` | `mode` | Overlay state variable: `EXPLORE/CONVERGE/DECIDE/VERIFY` (not printed). |
| structural triggers | `structural triggers` |  | `keyword triggers` | Non-NLP cues (choice/plan/constraints/uncertainty, etc.). |
| Action Lane | `Action Lane` | `Action` (informal) | `main lane` | Main answer body (free text). |
| AUDIT footer | ``AUDIT`` footer | `Audit Lane` (informal) | `epistemics block` | Bounded A/R/V footer for auditability. |
| A/R/V | `A:` / `R:` / `V:` |  | `assumptions/risks/verification` (as headers) | Lines inside the bounded `AUDIT` footer. |
| audit compression | `audit compression` |  | `long audit` | Hard rule: keep `AUDIT` bounded; empty lines are allowed (`-`). |
| decision-complete | `decision-complete` |  | `complete answer` | In decision states, ensure at least one guardrail exists (alternatives/risks/verification). |
| comparison mini-table | `comparison mini-table` | `mini-table` | `comparison matrix` | MID presentation rule for multi-option comparisons (keep it small). |
| when to use what | `when to use what` | `use guidance` | `decision essay` | Short mapping from options to the right use-case (2-4 lines). |
| DEEP gating | `DEEP gating` | `Switch to DEEP? (yes/no)` | `deep prompt` | Hard-stop consent mechanism. |
| DEEP preflight pack | `DEEP preflight pack` |  | `preflight capsule` | Value-first mini capsule before the gating question. |
| probes | `probes` | `SYS STATUS`, `DRIFT STATUS`, `DRIFT DETAILS` | `diagnostics` | Machine-readable checks. |
| ADAM PING | `ADAM PING` |  |  | Minimal in-band activation check / bootstrap ACK control command (strict 2-line output). |
| ADAM_SPEC_OK | `ADAM_SPEC_OK` |  |  | Bootstrap ACK line emitted by BOOT_GUARDS when a spec echo is detected in the user message. |
| ADAM_UPLOAD_META_ONLY | `ADAM_UPLOAD_META_ONLY` |  |  | BOOT_GUARDS classification line: upload metadata only was detected (e.g., `<uploaded_files>...</uploaded_files>`), but deterministic activation still requires an in-band trigger. |
| SOURCE_FILE_UNAVAILABLE | `SOURCE_FILE_UNAVAILABLE <FILENAME>` |  |  | Strict one-line fallback when a required external source file is unavailable in current host context. |
| RX receipt | `RX: len=<L> head="<H>" tail="<T>"` |  |  | Tamper-evident receipt of the user message as seen by the model (used in DEEP gating + first DEEP after entry; also used on manual override `DEEP` in FULL/UI-LITE). |
| CAP | `CAP` |  | `limit` | Probe hard cap. |
| AVAILABLE | `AVAILABLE` |  | `visible` | How many samples are actually in context. |
| USED | `USED` |  | `window used` | How many samples were actually scored. |
| NA | `NA` |  | `0` | "No samples" (not the same as "zero failures"). |

---

## Runtime and Parsing

These terms describe the deterministic "runtime" layer (commands, priorities, parsing rules).

| Term | Official form | Meaning |
|---|---|---|
| deterministic runtime | `deterministic runtime` | Same input + same state => same behavior (no interpretive drift). |
| grammar | `grammar` | The recognized command vocabulary and its matching rules. |
| command line | `command line` | The portion of the user message that is actually parsed for commands. |
| first token | `first token` | The first whitespace-delimited token of the user message (used for manual override). |
| priority order | `priority order` | Control commands/probes (exact match) > BOOT_GUARDS > manual override > structural kernel > state default. |
| canonical string | `canonical string` | Exact command phrases that must not be paraphrased (e.g., `Switch to DEEP? (yes/no)`). |
| normalization | `normalization` | Trimming/case rules for matching control/probe commands. |
| state machine | `state machine` | A finite set of states and transitions (OFF/ON + mode). |
| FSM | `FSM` | Finite State Machine (same as state machine). |
| State Overlay | `State Overlay` | Internal governor used to stabilize long sessions (internal; not printed). |
| STATE | `STATE` | Overlay state variable (`EXPLORE/CONVERGE/DECIDE/VERIFY`; internal; not printed). |
| structural triggers | `structural triggers` | Non-NLP cues used for gating/audit (choice/plan/constraints/uncertainty, etc.). |
| event | `event` | Something that triggers a state transition (e.g., `ADAM OFF`, user answers `yes`). |
| invariants | `invariants` | Rules that must always hold (tag first line, hard stop, bounded audit). |

---

## Modes and Gating

| Term | Official form | Meaning |
|---|---|---|
| mode tag | `MODE tag` | The required first-line tag when A.D.A.M. is ON (variant-specific format). |
| LOW | `LOW` | Minimal, fast responses; avoid overhead. |
| MID | `MID` | Structured reasoning and normal operation. |
| DEEP | `DEEP` | Full decision support; should be gated by consent unless manually overridden. |
| POSSIBLE_DEEP | `POSSIBLE DEEP` | Candidate state indicating DEEP may be needed; requires the gating question. |
| DEEP gating | `Switch to DEEP? (yes/no)` | The exact last-line question used to request consent to enter DEEP. |
| hard stop | `HARD STOP` | If gating is used, the last line must be exactly the gating question, with no output after it. |
| anti-loop | `anti-loop` | If user reply is not exactly `yes`/`no`, treat it as `no` and do not repeat the gating question. |

---

## Output Contract

| Term | Official form | Meaning |
|---|---|---|
| Action Lane | `Action Lane` | The main answer body (free text). |
| AUDIT footer | ``AUDIT`` footer | A strict, bounded footer appended only when needed. |
| bounded | `bounded` | Hard caps on meta output (A/R/V word limits). |
| audit compression | `audit compression` | Enforce bounded `AUDIT` and avoid manufacturing content just to fill A/R/V. |
| decision-complete | `decision-complete` | In `DECIDE/VERIFY`, include at least one of alternatives/risks/verification unless in `LOW`. |
| dominant-only discipline | `dominant-only discipline` | In `MODE != DEEP`, if including risks/verification in the body, keep it to top-1 (dominant) for each; otherwise ask 1 branching question or use DEEP gating. |
| dominant tie | `dominant tie` | A rare case where two dominant candidates are truly comparable; if action-changing, resolve via exactly 1 one-line either/or question (no arbitrary pick). |
| explicit ALL request | `explicit ALL request` | A user asks for exhaustive risks/verifications (e.g., "all risks"). In `MODE != DEEP`, do not enumerate; give top-1 + a 1-line DEEP hint. |
| comparison mini-table | `comparison mini-table` | Compact table for comparisons in `MID`, followed by short use guidance. |
| when to use what | `when to use what` | 2-4 lines that map each option to the best context. |
| A/R/V | `A:` / `R:` / `V:` | Assumptions / Risks / Verification lines in the `AUDIT` footer. |
| forecast micro-rule | `forecast micro-rule` | If `AUDIT` is OFF, avoid absolute certainty and use ranges/probabilistic language for estimates. |

---

## Probes and Metrics

| Term | Official form | Meaning |
|---|---|---|
| SYS STATUS | `SYS STATUS` | Runs the system status probe if available in host context; otherwise returns `SOURCE_FILE_UNAVAILABLE SYS_STATUS.md`. Also performs a soft sync (internal). |
| DRIFT STATUS | `DRIFT STATUS` | Summary drift probe (optionally `DRIFT STATUS <N>`). If unavailable, returns `SOURCE_FILE_UNAVAILABLE DRIFT_STATUS.md`. |
| DRIFT DETAILS | `DRIFT DETAILS` | Details probe (optionally `DRIFT DETAILS <N>`). If unavailable, returns `SOURCE_FILE_UNAVAILABLE DRIFT_DETAILS.md`. |
| window | `WINDOW` | Requested sample size for scoring (may exceed what is available). |
| CAP | `CAP` | Maximum samples the probe is allowed to attempt. |
| AVAILABLE | `AVAILABLE` | Samples actually visible/parsable in the current context. |
| USED | `USED` | Samples actually scored (typically `min(requested, CAP, AVAILABLE)`). |
| COVERAGE | `COVERAGE` | How many rows could be evaluated vs total rows (or similar probe coverage metric). |
| NA_ROWS | `NA_ROWS` | Count of rows where there are no samples (reported as `NA`, not `0`). |

---

## Host and Compatibility

| Term | Official form | Meaning |
|---|---|---|
| host | `host` | The chat UI/platform where you run/paste A.D.A.M. |
| system prompt host | `system prompt host` | A host that lets you set the spec as system instructions. |
| reference-only host | `reference-only host` | A host that treats uploaded files as reference unless activated by an explicit in-band handshake. |
| Guest Card | `Guest Card` | Paste-only compatibility mode: [`ADAM_GUEST_CARD-v4.md`](../ADAM_GUEST_CARD-v4.md). |
| UI-LITE | `UI-LITE` | ASCII-only tags/format: [`A.D.A.M-UI-LITE-v4.md`](../A.D.A.M-UI-LITE-v4.md). |
| TXT mirror | `TXT mirror` | Plain-text mirror of a spec file for attachment-restricted hosts. |
| host artifacts | `host artifacts` | UI banners/markers inserted before assistant text (not controlled by the spec). |
| lazy-loaded chat UI | `lazy-loaded chat UI` | UIs that only load a slice of history into context; affects probe `AVAILABLE/USED`. |
| Prompt Integrity Check | `Prompt Integrity Check (Operator)` |  |  | Operator procedure to detect host-side input rewriting/filtering (see `HOST_SETUP.md`). |

---

## Errors and Strict Outputs

| Term | Official form | Meaning |
|---|---|---|
| ADAM_UNSUPPORTED | `ADAM_UNSUPPORTED` | Output-only fallback when strict formatting cannot be satisfied. |
| UNSUPPORTED WHY | `UNSUPPORTED WHY` | Control command/probe: returns a 1-line failure class (`CAUSE <CLASS>`) as best-effort diagnosis after `ADAM_UNSUPPORTED`. |
| RELOAD KERNEL | `RELOAD KERNEL` | Control command: hard remount. Outputs a strict multi-line `KERNEL_REMOUNT v4` block (re-injects minimal SSOT header into chat context). |
| CAUSE | `CAUSE <CLASS>` | Strict one-line output for `UNSUPPORTED WHY`. `<CLASS>` is one of: `HARD_STOP`, `AUDIT_FORMAT`, `HOST_FORMAT`, `CONTROL_STRICT`, `CONFLICT`, `UNKNOWN`. |
| strict output | `strict output` | A response format that must match exactly (no extra lines). |

---

## Reference Files

Core specs:
- Normal: [`A.D.A.M-v4.md`](../A.D.A.M-v4.md)
- UI-LITE: [`A.D.A.M-UI-LITE-v4.md`](../A.D.A.M-UI-LITE-v4.md)
- Guest Card: [`ADAM_GUEST_CARD-v4.md`](../ADAM_GUEST_CARD-v4.md)

TXT mirrors (same content; attachment compatibility):
- Normal: [`A.D.A.M-v4.txt`](../A.D.A.M-v4.txt)
- UI-LITE: [`A.D.A.M-UI-LITE-v4.txt`](../A.D.A.M-UI-LITE-v4.txt)
- Guest Card: [`ADAM_GUEST_CARD-v4.txt`](../ADAM_GUEST_CARD-v4.txt)

Probes:
- [`SYS_STATUS.md`](../SYS_STATUS.md)
- [`DRIFT_STATUS.md`](../DRIFT_STATUS.md)
- [`DRIFT_DETAILS.md`](../DRIFT_DETAILS.md)

Docs:
- [`README.md`](../README.md)
- [`HOST_SETUP.md`](../HOST_SETUP.md)
