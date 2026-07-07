# Contributing

Thanks for improving uniwebpay skills. This repository is small, but changes can affect how coding agents implement payment flows, so prefer precise, safety-focused edits.

## Scope

Good contributions include:

- Clearer payment-link, checkout, subscription, webhook, or refund guidance.
- Safer defaults for API keys, webhook secrets, and fulfillment.
- Fixes for CLI or SDK command examples.
- Documentation improvements for Claude Code, Codex, and other agent skill installers.

Avoid adding broad platform documentation here when the canonical docs should live at `https://skill.uniwebpay.com/docs`.

## Development Workflow

1. Create a branch from `main`.
2. Keep changes focused.
3. Update both `README.md` and `README.zh-CN.md` when user-facing setup instructions change.
4. Update `skills/uniweb/SKILL.md` when agent behavior should change.
5. Open a pull request and complete the PR checklist.

The `main` branch is protected. Changes should go through pull requests.

## Security Guidance

Never commit real API keys, webhook secrets, wallet files, `.env` files, or generated claim URLs with live tokens. Use placeholder values such as `sk_server_xxx`, `whsec_xxx`, `wal_xxx`, and `plink_xxx`.

Payment fulfillment guidance must keep these defaults:

- Server-side secrets only.
- Fulfillment only after verified webhooks.
- Raw request body for webhook signature verification.
- Idempotent webhook processing.
- Amount, currency, metadata, and order-state checks before granting access.
- Least-privilege server keys.

Report suspected security vulnerabilities privately through GitHub private vulnerability reporting instead of public issues.

## Review Checklist

Before requesting review, check:

- The skill name remains `uniweb`.
- Examples use placeholders, not real credentials or customer data.
- New instructions do not encourage browser-side SDK usage or client-side secrets.
- Payment and subscription examples match supported currencies and payment methods.
- Links to uniweb docs, dashboard, and API reference still work.
