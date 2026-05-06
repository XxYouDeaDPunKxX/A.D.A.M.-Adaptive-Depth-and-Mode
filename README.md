# 🤖 A.D.A.M.

**Adaptive Depth and Mode**

A.D.A.M. is a control protocol for AI chats.

It helps the assistant answer at the right depth: short when the request is simple, structured when reasoning is useful, and deeper when the task becomes more complex.

![A.D.A.M. banner](banner.png)

## ✨ What it changes

Without A.D.A.M., a chat can easily become too verbose, too vague, or too eager to explain everything.

With A.D.A.M., each reply is routed through a visible mode:

- ⚡ `LOW` — fast, minimal answers
- 🧭 `MID` — normal structured answers
- 🧠 `DEEP` — full decision support, reached through automatic detection + consent, or by manual override

The goal is not to make the assistant sound smarter.

The goal is to reduce friction:

- 🔇 less noise
- 🎯 clearer answers
- 🏷️ visible response depth
- 🎛️ explicit control when more reasoning is needed
- 🧷 better handling of uncertainty and limits

## 🚀 Try it in 30 seconds

Use a chat host with an instruction field that can hold the full protocol file.

### 📡 Why two `ADAM PING`s?

The first ping starts A.D.A.M. and checks whether the uploaded file is really available as the protocol source, not just visible as text.

If the host allows it, A.D.A.M. also verifies the source with a safe nearby write/readback check. The result is shown as `BOOTSTRAP_CLASS`.

The second ping is a liveness check: it confirms A.D.A.M. is still active on the next turn. It does not run the full bootstrap again.

1. 📄 Open [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt)
2. 🧩 Load the full file into that instruction field
3. 📡 Send:

```text
ADAM PING
```

4. ✅ Check that the assistant replies with the bootstrap output
5. 🔁 Send again:

```text
ADAM PING
```

6. 💬 Ask a normal question, for example:

```text
capital of germany
```

## 👀 What you should see

First activation ping:

```text
MODE: MID
ADAM_PING_OK
CONTROL: first word LOW | MID | DEEP sets mode.
BOOTSTRAP_CLASS: TEXT_ONLY | BOUND_RO | BOUND_RW | GHOST
```

Second ping:

```text
MODE: MID
ADAM_PING_OK
```

Simple question:

```text
MODE: LOW
Berlin.
```

If the host cannot follow the protocol reliably, A.D.A.M. may fail closed with:

```text
ADAM_UNSUPPORTED
```

That is intentional. A.D.A.M. should not pretend to be active when the output contract cannot be satisfied.

## 🎛️ How depth is selected

A.D.A.M. has two control paths:

1. 🧭 Automatic routing: the protocol reads the shape of your request and chooses the appropriate mode.
2. 🎚️ Manual override: you can set the mode yourself by starting your message with one of these tokens:

```text
LOW
MID
DEEP
```

Examples:

```text
LOW explain this in one sentence
```

```text
MID help me compare these two options
```

```text
DEEP analyze this decision and its failure modes
```

When automatic routing detects that a request may need deeper reasoning, A.D.A.M. marks it as `POSSIBLE DEEP` and asks before entering the full `DEEP` response:

```text
Switch to DEEP? (yes/no)
```

Reply exactly:

```text
yes
```

to continue into the automatically suggested `DEEP` pass.

## 🧰 Where A.D.A.M. makes sense

A.D.A.M. is useful when a normal chat starts to create friction: too much explanation, unclear depth, repeated corrections, or long sessions that drift away from the original goal.

Use it for decisions, planning, reviews, structured writing, iterative creative work, or any task where you want the assistant to stay focused instead of expanding by default.

You probably do not need it for casual chat or a single simple question.

## 📦 What this repo contains

| File | Purpose |
|---|---|
| 📄 [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt) | Canonical protocol file. Load this into the host. |
| 🧪 [`ADAM_SELF_TEST.md`](ADAM_SELF_TEST.md) | External self-test probe for checking expected behavior. |
| 📊 [`SYS_STATUS.md`](SYS_STATUS.md) | Lightweight session health/status probe. |
| 📚 [`docs/lexicon.md`](docs/lexicon.md) | Official terminology used across the repo. |
| 📝 [`docs/field-report-chat.md`](docs/field-report-chat.md) | Field report from a real creative workflow. |

## ⚡ Quick command reference

| Command | Use |
|---|---|
| 📡 `ADAM PING` | Activate or check liveness. |
| ⚡ `LOW ...` | Force a short answer. |
| 🧭 `MID ...` | Force a normal structured answer. |
| 🧠 `DEEP ...` | Force full decision support. |
| 🧪 `ADAM SELF TEST` | Run the external self-test if available. |
| 📊 `SYS STATUS` | Run the status probe if available. |
| 🔎 `TRACE INPUT` | Show a strict trace of the last visible user input. |
| ⏸️ `ADAM OFF` | Turn the protocol off. |
| ▶️ `ADAM ON` | Turn the protocol back on. |
| 🧯 `ADAM REMOUNT` | Generate a recovery capsule for remounting. |

<details>
<summary>🧩 Technical details</summary>

## 🎯 Positioning

A.D.A.M. is a spec-first control layer for AI conversations.

It is not a library.

The spec is the implementation: you place [`A.D.A.M.v5.txt`](A.D.A.M.v5.txt) into a compatible host and the protocol becomes the response contract.

## ⚙️ Runtime contract

While A.D.A.M. is active:

- 🏷️ normal replies start with a `MODE` tag
- 🎚️ manual override wins over automatic routing
- 🔁 mode is recomputed every message
- 🧠 `DEEP` can be proposed by automatic routing, entered by consent, or forced by manual override
- 🧾 audit output is bounded
- 📐 strict commands use fixed output shapes
- 🧯 failed invariants fail closed instead of being approximated

## 🏷️ Core modes

| Mode | Meaning |
|---|---|
| ⚡ `MODE: LOW` | Minimal answer, low overhead. |
| 🧭 `MODE: MID` | Default structured answer. |
| 🚪 `MODE: MID -> POSSIBLE DEEP` | A deeper pass may be useful; consent is required. |
| 🧠 `MODE: DEEP` | Full decision support. |

## 🧠 Structural routing

A.D.A.M. does not route only by keywords.

The structural kernel looks at message shape: options, steps, numeric constraints, criteria, timelines, and similar signals.

This lets simple requests stay light while complex requests can trigger `POSSIBLE DEEP`.

## 🧾 Audit behavior

A.D.A.M. keeps the main answer clean.

When audit is required, it uses a strict footer:

```text
AUDIT
ACTION: <text or ->
RISK: <text or ->
BASIS: <text or ->
```

The audit block is intentionally short. It is not a second essay.

## 📡 Bootstrap state

Activation uses `ADAM PING`.

The bootstrap output includes `BOOTSTRAP_CLASS`, which records the source-side bootstrap state:

- 📄 `TEXT_ONLY`
- 🔒 `BOUND_RO`
- 🔐 `BOUND_RW`
- 👻 `GHOST`

This describes what the protocol can verify about its source context during activation. It does not mean the host itself is reliable forever.

## 🧩 Compatibility notes

A.D.A.M. assumes the host has a stable instruction surface that can hold the full protocol file.

Known limits:

- 📎 upload alone is transport only
- 📡 activation still requires exact `ADAM PING`
- 🪟 host UI banners are outside protocol control
- ✂️ truncated context can break activation
- 📐 strict output formats may fail on hosts that rewrite assistant output
- 🧪 probe files must be available in context for external probes to run

If the host cannot satisfy a strict invariant, the safe output is:

```text
ADAM_UNSUPPORTED
```

## 🧪 External checks

Available external checks:

- 🧪 [`ADAM_SELF_TEST.md`](ADAM_SELF_TEST.md)
- 📊 [`SYS_STATUS.md`](SYS_STATUS.md)

These are diagnostics. They are not required for first activation.

## 🧱 Design constraints

A.D.A.M. is designed around:

- 🧭 rule-ordered routing
- 🧾 bounded auditability
- 🎚️ explicit manual override
- 🧠 automatic `POSSIBLE DEEP` detection
- 🚪 consent-gated entry into full `DEEP`
- 🧯 fail-closed behavior
- 📄 portable instruction text
- 🧪 testable output contracts

The protocol does not try to maximize personality, style, or verbosity.

It tries to stabilize method under constraint.

## 🌱 Field signal

In a real multi-iteration creative workflow, A.D.A.M. was observed to reduce drift, shorten prompts over time, make corrections more precise, and keep decisions moving without resetting the session.

Full field report: [`docs/field-report-chat.md`](docs/field-report-chat.md)

## 📚 Terminology

Official terms are maintained in [`docs/lexicon.md`](docs/lexicon.md).

Use the lexicon when editing the protocol or documentation so that terms remain stable across the repo.

</details>

## 🙌 Credits

Author: `XxYouDeaDPunKxX` — concept, spec, behavior, and final decisions.

AI-assisted drafting was used for documentation edits, wording, refactors, iteration speed-ups, and templates/images.

## 📜 License

This project is licensed under `CC BY-SA 4.0` (`Creative Commons Attribution-ShareAlike 4.0 International`).

See [`LICENSE`](LICENSE).
