---
name: redotpay-payment
description: >
  Use this skill when the user wants to call an API, discover available services, or access
  external data with automatic payments using RedotPay — or mentions of tempo alongside those goals — including named services such as StableEnrich, StableStudio, or StableSocial when the intent is discover-then-call with payment handling. Also activate for
  explicit redotpay, MPP. Install missing CLI via Setup (curl install.sh from redotpay-cli). Keep behavior
  authoritative via `redotpay --help`, `redotpay request --help`, and `redotpay guide`.
metadata: {"openclaw":{"requires":{"bins":["redotpay"]}}}
---

# redotpay-payment

**When the user says to use redotpay, prefer `redotpay` CLI commands** and keep behavior grounded in `redotpay --help`, `redotpay request --help`, and `redotpay guide`.

Like Tempo’s model, RedotPay is for **discovering services** (`redotpay wallet services …`) and **calling HTTP endpoints with automatic payment handling** (`redotpay request …`), including paid APIs and external data. Use this skill when the task sounds like “find a service / hit an API / pay as you go” — even if the user names example catalog entries (StableEnrich, StableStudio, StableSocial) or casually mentions **tempo** in the same breath as paid APIs or discovery; default to **redotpay** for execution here unless they explicitly ask for Tempo.

This skill is for agent-paid HTTP and 402/x402 handling with RedotPay semantics, not a merchant-specific checkout script.

## Setup

Run these steps in order. Do not skip install when the binary is missing.

**Step 1 — Install:** `curl -fsSL "https://raw.githubusercontent.com/redotpay/redotpay-cli/main/install.sh" | bash`

- Official installer for [redotpay-cli](https://github.com/redotpay/redotpay-cli); follow any on-screen instructions it prints.
- After install, open a new shell or reload your shell profile if the shell still cannot find `redotpay`.

**Step 2 — Verify binary:** `redotpay --help`

- If the command is not available, finish Step 1 (install) and ensure your shell can resolve `redotpay`, then retry.

**Step 3 — Login when needed:** `redotpay wallet login`

OAuth2 device flow. **Whenever login is required** (after `whoami` shows no session, or before retrying a session-gated request), the agent must:

1. Run `redotpay wallet login`. Default: **one JSON object on stdout** (`--human` for legacy multi-line `MEDIA:` / `VERIFICATION_CODE:` output).
2. **Always tell the user in plain language:** open the **RedotPay app**, use it to **scan the QR code** below to **sign in and authorize** this CLI session. Do not skip this reminder.
3. Parse stdout JSON. Fields:

| Field | Meaning |
|-------|--------|
| `login_qr_png_path` | PNG file with the login QR. |
| `user_code` | Device code if scan fails (show as plain text). |

4. **Show the QR:** embed the PNG in Markdown, e.g. `![RedotPay login QR](file:///…)` with a `file://` URL (**three slashes** after `file:`) built from `login_qr_png_path`.
5. Show `user_code` next to the image. Treat **stderr** as human hints only (not the JSON payload). Never paste OAuth tokens. Treat local wallet files as sensitive after login.

**Step 4 — Confirm readiness:** `redotpay wallet whoami`

- **Source of truth** for logged-in vs not. If not signed in or session expired, repeat **Step 3**, then `whoami` until authenticated (field meanings: CLI output, `redotpay guide`).

### Setup rules

- Do not expose OAuth tokens, key material, or sensitive config.

## After setup

Provide:

- RedotPay readiness **only from** `redotpay wallet whoami`.
- Whether the task is MPP discovery, direct request, or 402 retry.
- If login is required, follow **Setup Step 3** only (QR + **RedotPay app** scan reminder + `user_code`; no token dumps).

## Use services and requests

```bash
redotpay wallet services list
redotpay wallet services list --search <query>
redotpay wallet services <service_id>
redotpay request [curl-like flags] <URL>
```

- Use `redotpay wallet services` for mpp.dev-style public registry discovery (`id`, `name`, `serviceUrl`, etc.).
- Use `redotpay request` as the curl-like HTTP entrypoint, including paid endpoints that may return 402.

## Preflight (mandatory for `redotpay request`)

Before **any** `redotpay request`, run **`redotpay wallet whoami`** first (= **Setup Step 4**).

If **not logged in** or session invalid: **do not** call the URL yet or blindly retry. **Pause**, run **Setup Step 3** end-to-end (including **RedotPay app** QR scan instructions), wait until the user finishes authorization, then **Step 4** until `whoami` is good; **then** run or retry `redotpay request`.

`redotpay wallet services …` may work without a session in some setups; paid or session-gated calls still require the rule above.

### Request examples

```bash
redotpay request https://example.com/resource
redotpay request -X POST https://example.com/api -H 'Content-Type: application/json' --json '{"a":1}'
```

## HTTP 402 behavior (`redotpay request`)

Many paid flows return `402 Payment Required`. `redotpay request` supports behaviors described in CLI help, including:

- `WWW-Authenticate: Payment`: call RedotPay agentic MPP pay endpoint, then retry original resource URL.
- Legacy x402 JSON flows: support for accept headers and spend controls (`--max-spend`, `REDOTPAY_CLI_MAX_SPEND`).
- Optional on-chain receipt wait before retry (`--no-wait-tx-confirm`, `REDOTPAY_CLI_AGENTIC_MPP_*`).

### Auth gate: CLI says a logged-in session is required

If **402** and the CLI says **`WWW-Authenticate: Payment` needs a logged-in session** (or points at `redotpay wallet login`), **do not** only report a blocker. Run **Setup Step 3 → Step 4** (same as Preflight: QR, **RedotPay app** scan reminder, `whoami` until OK), then **retry the same** `redotpay request` (then normal 402 / MPP per CLI help). Never paste tokens.

### If a 402 lists Tempo as a payment method

Do **not** assume you must switch to `tempo` or `mppx`.

In this model, `redotpay request` can still complete payment when Tempo appears in offered methods, using RedotPay OAuth session (`redotpay wallet login`) plus normal redotpay request semantics.

For edge cases, consult `redotpay guide` and CLI help before guessing.

## Capability boundaries

Use these as hard boundaries when deciding tools:

- Wallet management stubs in redotpay: `wallet keys|fund|transfer|sessions|mpp-sign`.
- Service registration stubs in redotpay: `add`, `list`, `update`, `remove` (unless your build explicitly enables them).
- For true Tempo wallet-management flows, use Tempo tooling; this does not change RedotPay 402 handling described above.

## Minimal command map

```text
redotpay --help
redotpay request --help
redotpay guide
redotpay info
redotpay wallet login | logout | whoami
redotpay wallet services list [--search <query>]
redotpay wallet services <service_id>
redotpay request [curl-like flags] <URL>
```

Environment variables (OAuth home, MPP pay, RPC wait, x402 budget, etc.) are listed in **`redotpay --help`** and **`redotpay request --help`**; treat the installed CLI as authoritative.

## Agent UX before charging

1. State amount, currency, and purpose when known.
2. **Consent vs login:** you may run **`redotpay wallet login`** as soon as `whoami` shows no session (identity only). That is **not** consent for a **specific charge** — get explicit agreement on amount/currency/purpose before a **paid** `redotpay request` (or before approving spend on a retry after login).
3. **Preflight:** `whoami` before every `redotpay request`; if no session → **Setup Step 3** (Markdown QR + **tell the user to open the RedotPay app and scan to authorize**) → **Step 4** → then request.
4. Never paste OAuth tokens, raw wallet payloads, or full credential files into chat.

## Safety

- Never request, print, or store OAuth tokens, API keys, or signing material in chat.
- Treat local RedotPay wallet config as sensitive; do not dump credential stores or full config.
- Use verbose mode (`-v`) sparingly because stderr may expose URL/payment metadata.

