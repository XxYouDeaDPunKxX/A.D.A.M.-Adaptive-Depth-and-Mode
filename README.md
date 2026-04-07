**A.D.A.M. — Adaptive Depth and Mode**  
A spec-first control protocol for rule-ordered depth routing in AI conversations.

![A.D.A.M. banner](banner.png)

# A.D.A.M.

A.D.A.M. is for people doing high-signal, decision-grade chat work who want control over depth, not more verbosity.

You → intent + direction  
The assistant → translation + construction  
A.D.A.M. → depth control & epistemic integrity

Positioning (SSOT)
Behavioral protocol -> rule-ordered depth routing + bounded audit -> for high-signal, decision-grade chat work.

Pillars (SSOT)
- rule-ordered depth routing
- bounded epistemic auditability
- user override priority
- spec-first runtime
- testable outputs

Terminology SSOT: [`docs/lexicon.md`](docs/lexicon.md)

What you get:
- A visible "checksum" on replies through the mode contract
- Manual override when you want it (`LOW`, `MID`, `DEEP` as first token)
- DEEP by consent (gated; enters only after `yes` exact, or manual `DEEP`)
- Explicit bootstrap state through `BOOTSTRAP_CLASS`
- Tool/asset workflows (non-normative): A.D.A.M. can improve prompt clarity, iteration discipline, and stop criteria (e.g., image generation). It does not "improve the renderer".

Quick links: [Try It Now](#try-it-now-30-seconds) | [Repo Contents](#what-this-repo-contains) | [Commands](#command-cheat-sheet) | [Lexicon](docs/lexicon.md) | [Field report](docs/field-report-chat.md)

## Start Here

If you want to use A.D.A.M. now:
- Primary entrypoint: [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt)
- Activation handshake: send exact `ADAM PING` twice

If you want to study the architecture first:
- start from the reference docs in [`docs/`](docs/)

## Repo Structure

- Core Spec: [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt)
- External checks: [`ADAM_SELF_TEST.md`](ADAM_SELF_TEST.md), [`SYS_STATUS.md`](SYS_STATUS.md)
- Reference Docs: [`docs/`](docs/)

## Minimal Use

Primary file:
- [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt)

Fast path:
1. Load `A.D.A.M.v5.txt` as system/instructions.
2. Send exact `ADAM PING`.
3. Verify the strict bootstrap reply.
4. Send exact `ADAM PING` again.
5. Verify the 2-line ACTIVE liveness reply.
6. Paste your task or artifact.

Concept diagram (official)
```text
input
  |
  v
structural kernel
  |
  v
mode state
  |
  v
output contract
  |\
  | \-> action
   \--> audit
```

## Try It Now (30 seconds)
1. Load [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt) into your host as SYSTEM / Instructions.
2. Send: `ADAM PING`
3. Verify the bootstrap reply.
4. Send: `ADAM PING`
5. Verify the ACTIVE liveness reply.
6. Ask: `capital of germany`

Expected bootstrap:
```text
MODE: MID
ADAM_PING_OK
CONTROL: first word LOW | MID | DEEP sets mode.
BOOTSTRAP_CLASS: TEXT_ONLY | BOUND_RO | BOUND_RW | GHOST
```

Expected second ping:
```text
MODE: MID
ADAM_PING_OK
```

Expected follow-up:
```text
MODE: LOW
Berlin.
```

<details>
<summary>Optional (reference-only/uncertain hosts)</summary>

Upload is transport only. Activation still requires exact `ADAM PING`.

If the host cannot satisfy the strict bootstrap contract reliably, the fail-closed output is:

```text
ADAM_UNSUPPORTED
```
</details>

## One Micro-Demo
```text
USER:
I need to choose a laptop for travel and coding:
- 12+ hours battery
- Linux-friendly
- under $1200

ASSISTANT (line 1):
MODE: MID -> POSSIBLE DEEP
TRACE INPUT: [vincoli da rispettare] [numeri, date o limiti]
... (compact, standard-level answer) ...
AUDIT
ACTION: compare battery, Linux support, and total cost
RISK: battery claims vary by workload
BASIS: verify with independent battery and Linux compatibility reports
Switch to DEEP? (yes/no)

USER: yes
ASSISTANT (line 1):
MODE: DEEP
... (assumptions, options, failure modes, verification) ...
```

`TRACE INPUT:` is a bounded structural overlay. The strict forensic input trace remains available through the exact `TRACE INPUT` command.

## Evolution of A.D.A.M.
A.D.A.M. is a spec-first control protocol. `v4` was the last multi-surface public line. `v5` keeps the same direction but changes the public shape of the project.

Current direction:
- **Single canonical file**: the repo now centers on [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt) as the live protocol surface.
- **SSOT structural kernel**: routing decisions are driven by structural triggers in one place (section `S`).
- **Bounded AUDIT**: the Action Lane stays clean; auditability appears only when needed via a strict 4-line `AUDIT` footer.
- **DEEP by consent**: enter DEEP only after `yes` exact (or manual `DEEP`).
- **Explicit bootstrap state**: `ADAM PING` lands in a public `BOOTSTRAP_CLASS` (`TEXT_ONLY`, `BOUND_RO`, `BOUND_RW`, `GHOST`) instead of leaving host state implicit.
- **External checks, smaller runtime**: `ADAM SELF TEST` and `SYS STATUS` remain public; the older drift probes are no longer part of the active surface.
- **Publish-boundary hardening (OVL)**: format-only validation at the publish boundary (draft -> validate -> rewrite once -> fail-closed).

`v4` remains in the release history as the closed archival line. The active repo is now `v5`-first.

Details: [Technical Reference (long-form)](#technical-reference-long-form).  

### A Portable Working Culture
A.D.A.M. is not just a behavioral protocol.
It makes a way of working explicit and portable.

Less noise.
Use what is already there.
Accept the tradeoff honestly.
Name the gap when it cannot be closed.

When that style stays recognizable across iterations, contexts, and even different systems, it suggests the protocol is doing more than constraining output.
It is stabilizing method.

## What This Repo Contains
| Surface | File | Use when |
|---|---|---|
| Canonical protocol | [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt) | Load the live protocol into a host |
| External self-test | [`ADAM_SELF_TEST.md`](ADAM_SELF_TEST.md) | Run an external operator check after activation when needed |
| External status probe | [`SYS_STATUS.md`](SYS_STATUS.md) | Run a lightweight system check |

Other:
- [`docs/lexicon.md`](docs/lexicon.md) - terminology reference
- [`docs/field-report-chat.md`](docs/field-report-chat.md) - long-form field report / transcript material

## Compatibility (Read This Once)
- A.D.A.M. assumes the host supports a real system prompt (or equivalent "instructions").
- Some hosts prepend UI banners (e.g., "thinking") before assistant text; those are outside the assistant's control.
- Upload is transport only. Activation requires exact `ADAM PING`.
- If a host behaves oddly, prefer `ADAM PING` or `ADAM REMOUNT` before debugging the content of the spec itself.
- `AUDIT` footer caps are word-based: target 26 words, hard max 38 per `ACTION` / `RISK` / `BASIS` line.
- The canonical public file is already plain `.txt`, so there is no separate Markdown/UI-lite split to manage in normal use.

---

<details>
<summary>real-world field report - a.d.a.m. in a live creative workflow</summary>

context  
a multi-iteration creative session on a real artifact (not a demo).  
a.d.a.m. used as a protocol, not as a stylistic persona.

observed system-level signals

- coherence remained stable across long iteration chains  
  (no direction loss, no output fragmentation, no interaction fatigue)

- cognitive load decreased over time  
  -> shorter prompts  
  -> more surgical corrections  
  -> faster decisions

- output identity emerged at system level  
  not just a "good result", but a consistent visual language  
  (product-grade coherence)

- structure did not suppress creativity  
  flow and constraint coexisted

why it worked

- the task was real and goal-oriented (not exploratory prompting)

- interaction unit:  
  output -> micro-feedback -> refined output  
  never: output -> reset

- mechanism (non-normative): a stable prompt anchor + one-variable-per-iteration changes + an explicit "done" criterion reduced semantic thrashing (fewer rerolls, higher convergence)

- the protocol was treated as a behavioral governor  
  not as a character or voice

- continuous forward momentum  
  (high engagement, low ambiguity, no chaotic branching)

replication conditions

this behavior degrades when:

- the objective is vague
- the task is trivial
- aesthetics replaces method
- iterations do not produce concrete artifacts
- sessions run without decision checkpoints

interpretation

this indicates that a.d.a.m. functions as:

- a cognitive load reducer
- a flow stabilizer for long sessions
- a decision-speed amplifier

not as an output "style improver".

These signals now inform the current design direction of the protocol and its external checks.

Full transcript: [`docs/field-report-chat.md`](docs/field-report-chat.md)

</details>


<details>
<summary>Why / How / Outcome</summary>

It filters noise.  
It adapts response depth.  
It preserves focus.

### Why
When you ask for something, you don’t need everything.  
You need the exact layer that moves you forward.

Not the history of the pizza. The pizza.

Not generic advice. Situational understanding.

Not more content. More signal.

A.D.A.M. exists to:
- reduce cognitive friction
- return control over attention
- make thinking executable

### How
A.D.A.M. works through adaptive depth:
- LOW → fast, essential, zero overhead
- MID → structured reasoning
- DEEP → full decision support (only when it matters)

The output contract is strict and rule-bound:
- Action Lane stays clean.
- Auditability is bounded in a strict `AUDIT` footer (`ACTION` / `RISK` / `BASIS`).
- DEEP is gated by consent (hard-stop), and long sessions are stabilized by an internal State Overlay (not printed).

You don’t switch tools. You switch cognitive gear.

From: information consumption  
To: intent → structure → execution

### Outcome
Build faster.  
Decide with clarity.  
Read only what matters.

</details>

## Technical Reference (Long-form)
Expanded architectural reference for the current runtime. Open the foldout below to read the long-form reference.

<details>
<summary>Technical Reference (Long-form)</summary>

## Positioning
A.D.A.M. is a spec-first control layer for AI conversations.  
It is not a library: the spec is the implementation. You paste it into a system prompt and run the behavior.

## Core Behavior (runtime contract)
- While A.D.A.M. is active, the first line tag is mandatory on normal replies. Strict commands and probes may define their own exact output shape.
- Global epistemic engine is ALWAYS ON (internal: no inventions; claim classification; evidence + confidence; risks; verification criteria).
- Epistemic display is token-optimized: Action Lane stays clean; when needed, a rigid `AUDIT` footer is appended (`ACTION` / `RISK` / `BASIS`).
- Manual override wins over auto-switch.
- Mode is recomputed every message (no inertia).
- Fixed strings remain in English (tags, command phrases, epistemic labels).

### Evolution of A.D.A.M. (Long-form)
This is an architectural reference for the current runtime. The spec is the implementation: the system improves primarily by refactor and contract hardening, not by adding features.

Why this matters:
- The protocol does not aim to maximize style or personality.
- It aims to stabilize method under constraint: less noise, explicit tradeoffs, visible limits, and reuse of structure that already exists.
- When those habits remain recognizable across long iterations, changing contexts, or even different systems, the protocol is doing more than formatting replies.
- It is making a working culture portable.

### Design constraints (why the spec looks the way it does)
- Structural-only routing (language-agnostic).
- Strict fixed strings (tags, control commands, probes, hard-stops).
- Consent-gated DEEP + anti-loop gating.
- Bounded auditability: strict 4-line `AUDIT` footer (`ACTION` / `RISK` / `BASIS`).
- Publish-boundary validation (OVL): format-only, rewrite once, fail-closed.
- Upload is transport only; activation requires exact `ADAM PING`.
- Public bootstrap state is explicit through `BOOTSTRAP_CLASS`.

### Runtime layers (SSOT)
- Structural kernel (section `S`): DEEP gating, AUDIT_ON, retrograde triggers (reuse-only).
- State Overlay (LITE): EXPLORE -> CONVERGE -> DECIDE -> VERIFY (internal; not printed).
- Output Contract (OVL): validates final output text only (format-only).

Trade-offs
- Optimized for stability and decision-grade work across hosts.
- If you need continuous per-claim labeling, use manual `DEEP` (or an external audit workflow).

## Modes and Tags
Active tags:
- `MODE: LOW`
- `MODE: MID`
- `MODE: DEEP`
- `MODE: MID -> POSSIBLE DEEP`

DEEP gating:
- If DEEP is a candidate, reply in MID, show `MODE: MID -> POSSIBLE DEEP`, and end with `Switch to DEEP? (yes/no)`.
- A bounded `TRACE INPUT:` structural overlay may appear immediately after the mode line in `MODE: MID -> POSSIBLE DEEP` and `MODE: DEEP`.
- Enter DEEP only after `yes` (exact), or manual override `DEEP`.
- Interpret `yes`/`no` as a gating response only if the immediately previous assistant reply ended with the hard-stop gating line.
- If user replies anything other than `yes`/`no`, DEEP is not entered; continue in AUTO and do not repeat the gating question (unless explicitly asked or manual override is used).

POSSIBLE DEEP trigger (summary, see specs for full text):
- Primary heuristic: `DEEP_CANDIDATE` (section S; structural-only) -> propose POSSIBLE DEEP.
- Note: date/time patterns count only if 2+ occurrences (e.g., `2026-03-05 14:30`).
- If unsure, stay in MID; manual override remains available via `DEEP`.

## Testing and Probes
External checks available after activation:
- `ADAM SELF TEST`
- `SYS STATUS`

Probes (use the probe files if available in the host context; otherwise you may get `SOURCE_FILE_UNAVAILABLE ...`):
- `SYS STATUS`

These probes are external by design. The core runtime does not depend on them to function; they are operator diagnostics, not activation prerequisites.

Recovery probe (does not require a probe file):
- `UNSUPPORTED WHY` -> returns `CAUSE <CLASS>` as best-effort diagnosis after `ADAM_UNSUPPORTED`
- `ADAM REMOUNT` -> active recovery capsule: outputs a strict bootstrap replay block meant to be re-pasted as a new user message in the same session or a fresh one

## Command Cheat Sheet
Operator commands & probes (copy/paste friendly).

| Type | Command | Notes |
|---|---|---|
| Manual override | `LOW ...` | case-sensitive; must be the first token |
| Manual override | `MID ...` | case-sensitive; must be the first token |
| Manual override | `DEEP ...` | case-sensitive; must be the first token; skips confirmation |
| Control | `ADAM ON` / `ADAM OFF` | case-sensitive; must be first words in message (trailing text ignored) |
| Control | `ADAM PING` | canonical bootstrap handshake; strict 4-line output in `TRANSPORT`, 2-line liveness reply in `ACTIVE` |
| Control | `ADAM PERSIST` | deploy and verify a physical copy when source conditions are safe |
| Self-test | `ADAM SELF TEST` | external operator check; not part of the first-run bootstrap path |
| Probe | `TRACE INPUT` | strict forensic input trace on the last visible user message |
| Probe | `SYS STATUS` | runs [`SYS_STATUS.md`](SYS_STATUS.md) if available; else: `SOURCE_FILE_UNAVAILABLE SYS_STATUS.md` |
| Recovery probe | `UNSUPPORTED WHY` | strict one-line output: `CAUSE <CLASS>` (best-effort) |
| Recovery capsule | `ADAM REMOUNT` | outputs the strict remount block; re-paste it as a new user message to replay bootstrap in-session or in a fresh session |

</details>

## Credits
Author: `XxYouDeaDPunKxX` (concept, spec, behavior, final decisions).
AI-assisted drafting: documentation edits, wording, refactors, iteration speed-ups, and templates/images.

## License
This project is licensed under `CC BY-SA 4.0` (`Creative Commons Attribution-ShareAlike 4.0 International`).
See [`LICENSE`](LICENSE).
