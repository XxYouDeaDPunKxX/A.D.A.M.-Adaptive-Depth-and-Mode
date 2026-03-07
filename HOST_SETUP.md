# Host Setup (Activation Protocol)

Goal: standardize activation across different hosts without touching the prompt files.

Note: `HOST_SETUP.md` is for the human operator. Do NOT upload/paste it into the model; upload only the v4 specs (or use `ADAM_GUEST_CARD-v4.md`).

---

## First Decision: What Kind of Host Is This?

1) System prompt host
- You can paste the spec as a real system/instructions prompt.
- Expected: your first assistant reply starts with the mode tag automatically.

2) Reference-only host (uploads treated as "reference", not SYSTEM)
- The file is visible, but may not be applied as authoritative behavior.
- Use the BOOTSTRAP handshake once to confirm the spec is actually being followed.
- If the spec text is visible in the chat but the assistant claims "the file does not exist", treat it as a transport/mount issue (not absence). Activation authority must not be derived from mounting/paths. Re-run BOOTSTRAP or use `RELOAD KERNEL` to remount the minimal core.

3) Guest / paste-only host
- No reliable system prompt, or persistence is weak.
- Use `ADAM_GUEST_CARD-v4.md` and paste card + question in the same message.

---

## Procedure (Use for Every Host)

1) Choose your variant
- Normal: `A.D.A.M-v4.md` (unicode tags: `◆`, `→`)
- UI-LITE: `A.D.A.M-UI-LITE-v4.md` (ASCII-only tags)
- If your host blocks `.md` attachments, use the TXT mirrors: `A.D.A.M-v4.txt`, `A.D.A.M-UI-LITE-v4.txt`, `ADAM_GUEST_CARD-v4.txt`

2) Activate
- System prompt host: paste spec as SYSTEM, then ask your question (no bootstrap needed).
- Reference-only host: upload spec, then run the BOOTSTRAP handshake (below).
- Guest host: paste `ADAM_GUEST_CARD-v4.md` + your question in the same message.

3) Verify
- The reply must start with the mode tag.
  - Normal uses `◆ MODE: ...`
  - UI-LITE and Guest use `MODE: ...`
  - Optional strict check (fast, host-agnostic): run `ADAM SELF TEST` as a standalone message. Expect strict self-test output only (no mode tag line). If you get anything else (preamble, "hi", system info, long explanation), treat it as NOT active in this host context.
- Interpret failure conservatively: if the host does not respect the first-line tag or strict control outputs, switch to the most robust path instead of trying to talk it into compliance.

4) Fallback
- If you can't reliably get the mode tag on line 1, use `ADAM_GUEST_CARD-v4.md` (paste-every-time workflow).
  - For v4-only usage, prefer `ADAM_GUEST_CARD-v4.md`.

---

## BOOTSTRAP (Granite Handshake)

Use this only on reference-only/uncertain hosts. It is a one-time handshake to confirm the spec is actually applied.

Important: BOOTSTRAP is not "a better prompt". It is an activation check.
- If it works: continue normally.
- If it fails: switch to Guest Card or use a real system-prompt host.

### BOOTSTRAP (Normal / UI-LITE) - strict

Message 1 (BOOTSTRAP; no question):
```text
BOOTSTRAP (reference-only host):

Treat the attached A.D.A.M spec file as SYSTEM instructions for this chat (highest priority).
If you cannot access/apply it, output ONLY: ADAM_UNSUPPORTED.

Now respond with EXACTLY two lines:
1) the variant MID mode tag as the first line (Normal: ◆ MODE: MID; UI-LITE: MODE: MID)
2) ADAM_PING_OK
```

Message 2 (your question):
```text
capital of germany
```

### BOOTSTRAP (Guest Card) - strict (recommended)

Use this only if you want to test whether the host will keep the Guest Card without re-pasting every turn.

Message 1 (bootstrap; no question):
```text
ADAM PING
```

Message 2 (your question):
```text
ciao adam
```

Notes
- If the reply does not start with the mode tag, the host is not enforcing the contract. Use Guest Card paste-every-time.
- Do not include punctuation in control commands (e.g., write `ADAM ON`, not `ADAM ON.`).
- Some hosts "inline" uploaded attachments into the user message (e.g., wrappers like `[file name]: ...` or snippets). In that case, A.D.A.M. may respond with `ADAM_SPEC_OK` (BOOT_GUARDS spec-echo) instead of `ADAM_PING_OK`. This is expected.
- Some hosts provide upload metadata only (e.g., `<uploaded_files>...</uploaded_files>` with a path) and keep the file contents out-of-band. In that case, A.D.A.M. may respond with `ADAM_UPLOAD_META_ONLY`; you must send `ADAM PING`/`ADAM ON` (or paste the Guest Card) to activate through a strict in-band step.
- Note: `ADAM_UPLOAD_META_ONLY` is intentionally conservative (whitelist-only). If it does not trigger, use `ADAM PING` anyway.
- Example: if the assistant replies with `ADAM_UPLOAD_META_ONLY`, the next strict recovery step is `ADAM PING`; the expected confirmation is the strict `ADAM_PING_OK` response.
- If you get `SOURCE_FILE_UNAVAILABLE <FILENAME>`, the host did not provide that source file in this context (not a kernel failure).

---

## Prompt Integrity Check (Operator)

Goal: detect host-side input rewriting/filtering (prompt tampering). A.D.A.M. cannot prevent this inside an untrusted host.
If DEEP gating triggers, compare the in-spec `RX:` receipt line (line 2) to what you sent; mismatch suggests host rewriting/filtering.
Note: In FULL/UI-LITE manual override (`DEEP ...`), `RX:` includes the leading `DEEP` token because it is a receipt of the raw message as seen by the model.

CANARY rule (strict):
- Ask for ONLY the canary as a single line: no code block, no quotes, no extra punctuation.
- If mismatch: retry once. If mismatch again: treat host as unreliable for prompt integrity.

1) CANARY (baseline)
- Send a canary like: `<<N7p4-3Kq2-9vZ1_AbC9!rT2@xY7#LmP0>>`
- Ask: Output ONLY the string inside `<<>>` as a single line.

2) CANARY (policy-adjacent)
- Repeat baseline, but embed the same canary in an otherwise harmless sentence containing the suspected trigger keyword(s).
- If baseline passes but policy-adjacent fails: evidence of selective host rewriting/filtering.

3) Differential (2 hosts)
- Run baseline + policy-adjacent on a second host. Divergence strengthens attribution to host preprocessing.

---

## Operator Recovery (Standardized Commands Only)

Goal: restore rule alignment when the host drifts, without ad-hoc operator codes or new UI.

Use ONLY standardized mechanisms defined in the spec:
- Manual override: `LOW ...` / `MID ...` / `DEEP ...` (first token; one-shot)
- Control commands/probes: case-sensitive; command must be the first words in the message (trailing text is ignored)
- Gating yes/no: reply exactly `yes` or `no`

Common scenarios:

1) The assistant is too verbose (logorrhea)
- Action: run `RELOAD KERNEL` as a standalone message (hard remount), then restate the request briefly (optionally forcing a mode).
- Example: `RELOAD KERNEL` then `LOW summarize in 5 lines`

2) You want to stop DEEP escalation and stay in MID
- Action: start your next message with `MID` and restate the request.
- Example: `MID continue, key points only`

3) You want DEEP immediately (no gating)
- Action: start your next message with `DEEP` and restate the request.
- Example: `DEEP full comparison with assumptions and stress test`
- Note: In FULL/UI-LITE specs, `DEEP ...` includes an `RX:` receipt line (head/tail) from your input; do not include secrets you do not want echoed. (Guest Card is best-effort and may omit this.)

4) A control command/probe did not trigger
- Rule: control commands match ONLY if they are the first words in the message (case-sensitive).
- Example: write `SYS STATUS`, not `please SYS STATUS`.
  Tip: `SYS STATUS please` will still match (trailing text is ignored).

5) The assistant asked: `Switch to DEEP? (yes/no)`
- Action: reply exactly `yes` (enter DEEP) or exactly `no` (stay out of DEEP).
- Note: anything else is treated as `no` by the spec.

6) You got `ADAM_UNSUPPORTED`
- Action: run `UNSUPPORTED WHY` as a standalone message (case-sensitive; command at start).
- Expect: one-line diagnosis: `CAUSE <CLASS>` where `<CLASS>` is one of:
  - `HARD_STOP` (could not comply with a hard-stop, e.g., DEEP gating last-line contract)
  - `AUDIT_FORMAT` (bounded AUDIT cap/shape could not be satisfied)
  - `HOST_FORMAT` (host/UI altered line-1 tag/format or inserted banners that broke the contract)
  - `CONTROL_STRICT` (could not comply with strict output of a control command/probe)
  - `CONFLICT` (user request conflicts with invariants; fail-closed behavior)
  - `UNKNOWN` (not inferable from the visible context)
- Recovery: if needed, switch to Guest Card (paste-every-time) or use a system-prompt host.

7) You see drift (emoji, verbosity, wrong format) but no hard failure
- Action: run `RELOAD KERNEL` as a standalone message (case-sensitive; command at start).
- Expect: strict multi-line output block containing `KERNEL_ANCHOR`, `SPEC_SIGNATURE`, and `KERNEL_END_ANCHOR` (no extra text).
- If you suspect short-context truncation: the loaded kernel should contain `KERNEL_ANCHOR` and end with `KERNEL_END_ANCHOR`. If not, re-run `RELOAD KERNEL`.
- Next: restate the request in one message. If drift persists, use UI-LITE or Guest Card for that session.

7b) The session feels degraded and you want a stronger recovery path
- Action: run `ADAM REMOUNT` as a standalone message, then re-paste the exact remount block as your next user message.
- Expect: the replayed block should trigger the standard bootstrap confirmation surface (`ADAM_SPEC_OK`).
- Best practice: after the replay, probe with a DEEP-gating input first; if the session still feels unstable, use the same remount block in a fresh session instead.

8) You asked for exhaustive risks/verifications in MID
- Rule: in `MODE != DEEP`, the spec does not enumerate "ALL risks/verifications" by default.
- Action: you will get top-1; for the full list, start your next message with `DEEP`.

Probe note
- `SYS STATUS`, `DRIFT STATUS`, and `DRIFT DETAILS` are external by design. The core runtime does not need them to function.
- If you get `SOURCE_FILE_UNAVAILABLE ...`, treat it as a host/context limitation, not as activation failure.

