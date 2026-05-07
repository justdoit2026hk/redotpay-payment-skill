# Using this repo with Cursor

This project includes a **Cursor project rule** so the RedotPay payment workflow applies automatically when you work here.

## In this repository

1. Open the folder in Cursor.
2. The rule [`.cursor/rules/redotpay-payment.mdc`](.cursor/rules/redotpay-payment.mdc) is committed, so no extra installation steps are required.
3. In Cursor, confirm it under the project rules UI / Settings → Rules.

## Use the same logic in another project

Copy `.cursor/rules/redotpay-payment.mdc` into that project's `.cursor/rules/` directory (create it if needed).

## Optional: personal Agent Skill

If you want reusable global skill behavior, install [`skills/redotpay-payment/SKILL.md`](skills/redotpay-payment/SKILL.md) to:

- `~/.cursor/skills/redotpay-payment/SKILL.md` (global), or
- `.cursor/skills/redotpay-payment/SKILL.md` (project-specific)

## Rule vs skill

- **Project rule (`.cursor/rules/*.mdc`)**: auto-applies when that project is opened in Cursor.
- **Skill (`.cursor/skills/.../SKILL.md`)**: reusable package for explicit invocation / broader reuse.
