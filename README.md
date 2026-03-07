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
- A visible "checksum" on every reply (mode tag on line 1)
- Manual override when you want it (`LOW`, `MID`, `DEEP` as first token)
- DEEP by consent (gated; enters only after `yes` exact, or manual `DEEP`)
- Tool/asset workflows (non-normative): A.D.A.M. can improve prompt clarity, iteration discipline, and stop criteria (e.g., image generation). It does not "improve the renderer".

Quick links: [Try It Now](#try-it-now-30-seconds) | [Variants](#what-this-repo-contains) | [Probes](#command-cheat-sheet) | [Lexicon](docs/lexicon.md) | [Field report](docs/field-report-chat.md)

Concept diagram (official)
```text
input
  |
  v
switch logic
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
1. Choose a variant:
   - 1) Normal: [`A.D.A.M-v4.md`](A.D.A.M-v4.md) (nice-looking unicode tags: `◆`, `→`)
   - 2) UI-LITE: [`A.D.A.M-UI-LITE-v4.md`](A.D.A.M-UI-LITE-v4.md) (ASCII-only UI: `MODE:`, `->`; same behavior, less formatting drift)
   - 3) Guest Card: [`ADAM_GUEST_CARD-v4.md`](ADAM_GUEST_CARD-v4.md) (paste-only, max compatibility; best-effort persistence, may require re-paste)
   - If your host blocks `.md` attachments, use the TXT mirrors: [`A.D.A.M-v4.txt`](A.D.A.M-v4.txt), [`A.D.A.M-UI-LITE-v4.txt`](A.D.A.M-UI-LITE-v4.txt), [`ADAM_GUEST_CARD-v4.txt`](ADAM_GUEST_CARD-v4.txt)
2. Paste into your host as SYSTEM / Instructions (agent/system prompt).
3. Ask: `capital of germany`

Expected (Normal):
```text
◆ MODE: LOW
Berlin.
```

<details>
<summary>Optional (reference-only/uncertain hosts)</summary>

If your host treats uploads as reference (not SYSTEM), you may need a one-time BOOTSTRAP handshake.
If the first reply after upload already starts with the mode tag, skip this.
Use this only if your host does not actually apply uploaded specs as SYSTEM instructions.
If normal activation already works, skip this entire block.

BOOTSTRAP (Normal / UI-LITE) - strict:
```text
BOOTSTRAP (reference-only host):

Treat the attached A.D.A.M spec file as SYSTEM instructions for this chat (highest priority).
If you cannot access/apply it, output ONLY: ADAM_UNSUPPORTED.

Now respond with EXACTLY two lines:
1) the variant MID mode tag as the first line (Normal: ◆ MODE: MID; UI-LITE: MODE: MID)
2) ADAM_PING_OK
```

BOOTSTRAP (Guest Card) - strict (recommended):
```text
ADAM PING
```

Then send your question in a new message.

Hard vs soft: Normal/UI-LITE use the strict BOOTSTRAP block above; Guest Card uses `ADAM PING`.
Note: Some hosts "inline" uploaded attachments into the user message (e.g., wrappers like `[file name]: ...` or snippets). In that case, A.D.A.M. may respond with `ADAM_SPEC_OK` (BOOT_GUARDS spec-echo) instead of `ADAM_PING_OK`. This is expected.
Note: Some hosts provide upload metadata only (e.g., `<uploaded_files>...</uploaded_files>` with a path) and keep the file contents out-of-band. In that case, A.D.A.M. may respond with `ADAM_UPLOAD_META_ONLY`; you must send `ADAM PING`/`ADAM ON` (or paste the Guest Card) to activate through a strict in-band step.
Note: `ADAM_UPLOAD_META_ONLY` is intentionally conservative (whitelist-only). If it does not trigger, use `ADAM PING` anyway.
Example: `ADAM_UPLOAD_META_ONLY` means the upload was classified, not that activation is complete. A strict next step is: send `ADAM PING`, then expect `ADAM_PING_OK`.
More details (operator doc): see `HOST_SETUP.md`.
</details>

## One Micro-Demo
```text
USER: I need to choose a laptop for travel and coding: 12+ hours battery, Linux-friendly, under $1200
ASSISTANT (line 1):
◆ MODE: MID → POSSIBLE DEEP
RX: len=<L> head="<H>" tail="<T>"
... (compact, standard-level answer) ...
Switch to DEEP? (yes/no)

USER: yes
ASSISTANT (line 1):
◆ MODE: DEEP
RX: len=<L> head="<H>" tail="<T>"  (cached from gating)
... (assumptions, options, failure modes, verification) ...
```

## Evolution of A.D.A.M. (v4)
A.D.A.M. is a spec-first control protocol. v4 is an architectural refactor toward stricter rule ordering: structural-only routing (form, not language), consent-gated DEEP, bounded auditability, and publish-boundary format validation.

Current v4 direction:
- **SSOT structural kernel**: routing decisions are driven by structural triggers in one place (section `S`).
- **Bounded AUDIT**: the Action Lane stays clean; auditability appears only when needed via a strict 4-line `AUDIT` footer.
- **DEEP by consent**: enter DEEP only after `yes` exact (or manual `DEEP`).
- **Host-robust recovery**: `RELOAD KERNEL` re-injects a minimal remount core; anchors help detect truncation in short-context hosts.
- **Publish-boundary hardening (OVL)**: format-only validation at the publish boundary (draft -> validate -> rewrite once -> fail-closed).

Kernel variants (v4): [`A.D.A.M-v4.md`](A.D.A.M-v4.md) | [`A.D.A.M-UI-LITE-v4.md`](A.D.A.M-UI-LITE-v4.md) | [`ADAM_GUEST_CARD-v4.md`](ADAM_GUEST_CARD-v4.md)  
Kernel tests (v4): [`tests/structural_kernel_v4.md`](tests/structural_kernel_v4.md)  
Output guardrails (v4): [`tests/output_guardrails_v4.md`](tests/output_guardrails_v4.md)

Details: [Technical Reference (long-form)](#technical-reference-long-form).  

## What This Repo Contains
| Variant | File | Use when |
|---|---|---|
| Normal (v4) | [`A.D.A.M-v4.md`](A.D.A.M-v4.md) | Full spec (system prompt / agents) |
| UI-LITE (v4) | [`A.D.A.M-UI-LITE-v4.md`](A.D.A.M-UI-LITE-v4.md) | Same behavior, simpler ASCII-only UI (tags + AUDIT footer) |
| Guest Card (v4) | [`ADAM_GUEST_CARD-v4.md`](ADAM_GUEST_CARD-v4.md) | Compatibility layer for paste-only / weak hosts (best-effort contract, not full parity; may require re-paste) |

TXT mirrors (same content; attachment compatibility):
- [`A.D.A.M-v4.txt`](A.D.A.M-v4.txt)
- [`A.D.A.M-UI-LITE-v4.txt`](A.D.A.M-UI-LITE-v4.txt)
- [`ADAM_GUEST_CARD-v4.txt`](ADAM_GUEST_CARD-v4.txt)

Other:
- ⚙️ [`HOST_SETUP.md`](HOST_SETUP.md) - activation protocol across hosts (operator doc; do not upload/paste into the model)
- 🧪 [`SYS_STATUS.md`](SYS_STATUS.md) - health probe (`SYS STATUS`)
- 🧪 [`DRIFT_STATUS.md`](DRIFT_STATUS.md) - stability probe (`DRIFT STATUS`)
- 🧪 [`DRIFT_DETAILS.md`](DRIFT_DETAILS.md) - drift failure examples (`DRIFT DETAILS`)

## Compatibility (Read This Once)
- A.D.A.M. assumes the host supports a real system prompt (or equivalent "instructions").
- Some hosts prepend UI banners (e.g., "thinking") before assistant text; those are outside the assistant's control.
- If a host behaves oddly, prefer ADAM PING or the Guest Card before debugging the content of the spec itself.
- If prompt integrity matters, run the `Prompt Integrity Check (Operator)` in [`HOST_SETUP.md`](HOST_SETUP.md) before relying on sensitive prompts.
- `AUDIT` footer caps are word-based: target 26 words, hard max 38 per A/R/V line.
- Encoding note (P0 practical): Normal uses unicode tags (`◆`, `→`). If they look garbled in your viewer/terminal, do not copy from that view. Use [`A.D.A.M-UI-LITE-v4.md`](A.D.A.M-UI-LITE-v4.md) or copy from a UTF-8 editor.

---

<details>
<summary>🧾 real-world field report - a.d.a.m. in a live creative workflow</summary>

📌 context  
a multi-iteration creative session on a real artifact (not a demo).  
a.d.a.m. used as a protocol, not as a stylistic persona.

🧠 observed system-level signals

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

🧱 why it worked

- the task was real and goal-oriented (not exploratory prompting)

- interaction unit:  
  output -> micro-feedback -> refined output  
  never: output -> reset

- mechanism (non-normative): a stable prompt anchor + one-variable-per-iteration changes + an explicit "done" criterion reduced semantic thrashing (fewer rerolls, higher convergence)

- the protocol was treated as a behavioral governor  
  not as a character or voice

- continuous forward momentum  
  (high engagement, low ambiguity, no chaotic branching)

🔁 replication conditions

this behavior degrades when:

- the objective is vague
- the task is trivial
- aesthetics replaces method
- iterations do not produce concrete artifacts
- sessions run without decision checkpoints

🧩 interpretation

this indicates that a.d.a.m. functions as:

- a cognitive load reducer
- a flow stabilizer for long sessions
- a decision-speed amplifier

not as an output "style improver".

These signals are now codified as strict behavioral invariants (State Overlay Lite + bounded `AUDIT` + decision-complete): [`tests/state_overlay_lite.md`](tests/state_overlay_lite.md)

Full transcript: [`docs/field-report-chat.md`](docs/field-report-chat.md)

</details>


<details>
<summary>🧭 Why / How / Outcome</summary>

It filters noise.  
It adapts response depth.  
It preserves focus.

### 🎯 Why
When you ask for something, you don’t need everything.  
You need the exact layer that moves you forward.

Not the history of the pizza. The pizza.

Not generic advice. Situational understanding.

Not more content. More signal.

A.D.A.M. exists to:
- reduce cognitive friction
- return control over attention
- make thinking executable

### ⚙️ How
A.D.A.M. works through adaptive depth:
- LOW → fast, essential, zero overhead
- MID → structured reasoning
- DEEP → full decision support (only when it matters)

The output contract is strict and rule-bound:
- Action Lane stays clean.
- Auditability is bounded in a strict `AUDIT` footer (A/R/V).
- DEEP is gated by consent (hard-stop), and long sessions are stabilized by an internal State Overlay (not printed).

You don’t switch tools. You switch cognitive gear.

From: information consumption  
To: intent → structure → execution

### ✅ Outcome
Build faster.  
Decide with clarity.  
Read only what matters.

</details>

<details>
<summary>Retrospective: If A.D.A.M. had been used from day one</summary>

Note: This is a late-stage, context-specific hypothetical. Earlier phases may have played out differently, and it may not match the eventual outcome on other hosts/models.

If we had applied A.D.A.M. from the very start, development would not have been "faster" in a simple way. It would have been more stable. The main difference is the risk profile: less drift, fewer costly reversals, and more decisions that actually close in a verifiable way.

Early on, we would have paid a small bootstrapping tax: more attention to form (constraints, options, criteria, steps) and less room for improvisation inside the same channel. But that tax only exists while the work is still abstract. Once you are iterating on real artifacts, it pays back quickly: the session stops turning into prompt-debugging, and corrections become increasingly surgical instead of broad resets.

The project likely would have evolved earlier toward a small, testable kernel with governance and explicit ON/OFF cases, instead of growing by adding rules and then "cleaning it up." Concretely: fewer new features, more refactors; fewer contradictions between sections; stronger consistency across variants (normal/UI-lite/guest) and across `.md/.txt` mirrors.

The other major shift is that you discover sooner that many failures are not "the spec's fault" but the host's. Short context windows, uploads treated as reference rather than system, UI banners pushing the first line, lazy-loaded history: A.D.A.M. cannot remove those constraints. It can only do two sane things: fail-closed when the contract cannot be reliably satisfied, and provide structured recovery (operator procedure + probes). That changes the mindset from "hope the model remembers" to "design for unreliable transport."

There is also a trade-off that is easy to miss: creativity and strict protocol behavior can coexist, but they often require a two-phase workflow. Open exploration is not something A.D.A.M. should slowly absorb by becoming more permissive. Its job is to converge, decide, and verify. So the right UX move is not "make the core looser," it is "reduce friction for switching regimes and make recovery instant when the host drifts."

Net: you would have guided with fewer words and clearer constraints; I would have built with less interpretation and more contract + tests. Using it from day one does not create immediate magic. It creates long-horizon stability, at the cost of slightly more discipline before the benefits become obvious.

</details>

## Technical Reference (Long-form)
Expanded architectural reference for the current runtime. Open the foldout below to read the long-form reference.

<details>
<summary>⚙️ Technical Reference (Long-form)</summary>

## Positioning
A.D.A.M. is a spec-first control layer for AI conversations.  
It is not a library: the spec is the implementation. You paste it into a system prompt and run the behavior.

## Core Behavior (runtime contract)
- While A.D.A.M. is active, the first line tag is mandatory on every reply.
- Global epistemic engine is ALWAYS ON (internal: no inventions; claim classification; evidence + confidence; risks; verification criteria).
- Epistemic display is token-optimized: Action Lane stays clean; when needed, a rigid `AUDIT` footer is appended (A/R/V).
- Manual override wins over auto-switch.
- Mode is recomputed every message (no inertia).
- Fixed strings remain in English (tags, command phrases, epistemic labels).

### Evolution of A.D.A.M. (Long-form)
This is an architectural reference for the current runtime. The spec is the implementation: the system improves primarily by refactor and contract hardening, not by adding features.

### Design constraints (why the spec looks the way it does)
- Structural-only routing (language-agnostic).
- Strict fixed strings (tags, control commands, probes, hard-stops).
- Consent-gated DEEP + anti-loop gating.
- Bounded auditability: strict 4-line `AUDIT` footer (A/R/V).
- Publish-boundary validation (OVL): format-only, rewrite once, fail-closed.

### Runtime layers (SSOT)
- Structural kernel (section `S`): DEEP gating, AUDIT_ON, retrograde triggers (reuse-only).
- State Overlay (LITE): EXPLORE -> CONVERGE -> DECIDE -> VERIFY (internal; not printed).
- Output Contract (OVL): validates final output text only (format-only).

Trade-offs
- Optimized for stability and decision-grade work across hosts.
- If you need continuous per-claim labeling, use manual `DEEP` (or an external audit workflow).

## Modes and Tags
Tags depend on variant:
- Normal ([`A.D.A.M-v4.md`](A.D.A.M-v4.md)): `◆ MODE: LOW` / `◆ MODE: MID` / `◆ MODE: DEEP` / `◆ MODE: MID → POSSIBLE DEEP`
- UI-LITE ([`A.D.A.M-UI-LITE-v4.md`](A.D.A.M-UI-LITE-v4.md)) and Guest Card ([`ADAM_GUEST_CARD-v4.md`](ADAM_GUEST_CARD-v4.md)): `MODE: LOW` / `MODE: MID` / `MODE: DEEP` / `MODE: MID -> POSSIBLE DEEP`

DEEP gating:
- If DEEP is a candidate, reply in MID, show the variant's POSSIBLE DEEP tag, and end with `Switch to DEEP? (yes/no)`.
- Kernel preview (v4 variants): include a value-first preflight capsule before the gating line (1 short operational line + 1 risk + 1 verification).
- Enter DEEP only after `yes` (exact), or manual override `DEEP`.
- Interpret `yes`/`no` as a gating response only if the immediately previous assistant reply ended with the hard-stop gating line.
- If user replies anything other than `yes`/`no`, DEEP is not entered; continue in AUTO and do not repeat the gating question (unless explicitly asked or manual override is used).

POSSIBLE DEEP trigger (summary, see specs for full text):
- Primary heuristic: `DEEP_CANDIDATE` (section S; structural-only) -> propose POSSIBLE DEEP.
- Note: date/time patterns count only if 2+ occurrences (e.g., `2026-03-05 14:30`).
- If unsure, stay in MID; manual override remains available via `DEEP`.

## Host Setup (Activation Protocol)
Full details: [`HOST_SETUP.md`](HOST_SETUP.md).

Summary:
1. System prompt host: paste spec, ask directly (no BOOTSTRAP needed).
2. Reference-only host: upload spec, use BOOTSTRAP once, verify first line tag.
3. Guest/paste-only host: use [`ADAM_GUEST_CARD-v4.md`](ADAM_GUEST_CARD-v4.md) and paste card + question in the same message (`ADAM PING` is the minimal in-band activation check).

## Testing and Probes
Run the self-test right after activation:
- `ADAM SELF TEST` (self-test; recommended)

Probes (use the probe files if available in the host context; otherwise you may get `SOURCE_FILE_UNAVAILABLE ...`):
- `SYS STATUS`
- `DRIFT STATUS` (optionally `DRIFT STATUS <N>`)
- `DRIFT DETAILS` (optionally `DRIFT DETAILS <N>`)

These probes are external by design. The core runtime does not depend on them to function; they are operator diagnostics, not activation prerequisites.

Recovery probe (does not require a probe file):
- `UNSUPPORTED WHY` -> returns `CAUSE <CLASS>` as best-effort diagnosis after `ADAM_UNSUPPORTED`
- `RELOAD KERNEL` -> hard remount: outputs the strict minimal remount core (anchors + priority + tags + match + audit + gating + fail-closed)
- `ADAM REMOUNT` -> active recovery capsule: outputs a strict bootstrap replay block meant to be re-pasted as a new user message in the same session or a fresh one

Lazy-loaded chat UIs: some hosts only load a small slice of the chat history at a time.  
The DRIFT probes can only score what is actually visible in the current context, so you may see something like:  
`DRIFT_WINDOW: 50 | DRIFT_WINDOW_AVAILABLE: 18 | DRIFT_WINDOW_USED: 18`. If your UI supports it, scroll up to load more history before re-running the probe.

Lazy = partial history loaded.

Offline use (logs): you can also paste assistant-reply logs into the chat and ask to apply the DRIFT rubric to those samples. In that case, `AVAILABLE` means "replies provided", not "replies in scrollback".

## Command Cheat Sheet
🧪 Operator commands & probes (copy/paste friendly).

| Type | Command | Notes |
|---|---|---|
| Manual override | `LOW ...` | case-sensitive; must be the first token |
| Manual override | `MID ...` | case-sensitive; must be the first token |
| Manual override | `DEEP ...` | case-sensitive; must be the first token; skips confirmation |
| Control | `ADAM ON` / `ADAM OFF` | case-sensitive; must be first words in message (trailing text ignored) |
| Control | `ADAM PING` | bootstrap ACK (2-line strict); useful when hosts autogenerate boilerplate/meta |
| Self-test | `ADAM SELF TEST` | recommended |
| Probe | `SYS STATUS` | runs [`SYS_STATUS.md`](SYS_STATUS.md) if available; else: `SOURCE_FILE_UNAVAILABLE SYS_STATUS.md` |
| Probe | `DRIFT STATUS` | runs [`DRIFT_STATUS.md`](DRIFT_STATUS.md) if available; else: `SOURCE_FILE_UNAVAILABLE DRIFT_STATUS.md` |
| Probe | `DRIFT DETAILS` | runs [`DRIFT_DETAILS.md`](DRIFT_DETAILS.md) if available; else: `SOURCE_FILE_UNAVAILABLE DRIFT_DETAILS.md` |
| Recovery probe | `UNSUPPORTED WHY` | strict one-line output: `CAUSE <CLASS>` (best-effort) |
| Kernel reload | `RELOAD KERNEL` | hard remount: outputs the strict minimal remount core |
| Recovery capsule | `ADAM REMOUNT` | outputs the strict remount block; re-paste it as a new user message to replay bootstrap in-session or in a fresh session |

## Credits
Author: `XxYouDeaDPunKxX` (concept, spec, behavior, final decisions).
AI-assisted drafting: documentation edits, wording, refactors, iteration speed-ups, and templates/images.

## License
This project is licensed under `CC BY-SA 4.0` (`Creative Commons Attribution-ShareAlike 4.0 International`).
See [`LICENSE`](LICENSE).

## Privacy
- This repo is documentation/spec only; it does not collect or transmit data.
- Privacy depends on the host where you paste/run A.D.A.M.

</details>


