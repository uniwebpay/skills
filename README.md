# uniwebpay Skill

**Language:** English | [中文](README.zh-CN.md)

AI-powered integration assistant for the uniweb payment platform. Instead of reading docs, let your coding agent write the integration code.

## Quick Start

Four steps from zero to accepting payments. Tell Claude what you need in plain language, and the skill runs the right commands for you.

### 1. Install the Skill

```bash
npx skills add uniwebpay/skills
```

This installs the `uniweb` skill into Claude Code. After installation, Claude can recognize payment, checkout, billing, and collection requests and invoke the skill automatically.

> To install it globally for all projects: `npx skills add uniwebpay/skills -g`

### 2. Create an Account

Say to Claude:

> Create a uniweb merchant account for me

Claude runs `uniweb wallet create`, creates a wallet, and stores the secret safely on your machine at `~/.uniweb/config.json`:

```text
✓ Wallet created
Wallet ID:  wal_xxxxxxxxxxxx
Status:     active
Currency:   SGD
```

> You only need to do this once. Future commands reuse the same account automatically.

### 3. Create a Payment Link

Say to Claude:

> Create a payment link for 10 dollars

Claude runs `uniweb link create 1000 -c SGD` and returns a permanent payment URL:

```text
✓ Payment link created
URL:    https://skill.uniwebpay.com/p/plink_xxxxxxxxxxxx
Amount: 10.00 SGD
```

Put the link on a website, in an email, or behind a QR code to start collecting payments with no backend code:

```html
<a href="https://skill.uniwebpay.com/p/plink_xxxxxxxxxxxx">Pay S$10</a>
```

### 4. Claim the Dashboard

Say to Claude:

> Bind my account to the dashboard

Claude runs `uniweb wallet claim` and returns a claim link:

```text
Claim URL:  https://skill.uniwebpay.com/claim/wal_xxxx?token=claim_xxxx
Expires At: 2026-07-10 14:23
```

Open the link in your browser and sign in. The command-line wallet will be linked to the web dashboard, where you can view balances, orders, payment links, webhooks, payouts, and more.

> Claim links expire. If the link expires, run `uniweb wallet claim` again to generate a new one.

### Summary

| Step | What you say | Command the skill runs |
| --- | --- | --- |
| 1. Install | - | `npx skills add uniwebpay/skills` |
| 2. Create account | "Create a uniweb merchant account" | `uniweb wallet create` |
| 3. Collect payment | "Create a payment link for 10 dollars" | `uniweb link create 1000` |
| 4. Claim dashboard | "Bind my account to the dashboard" | `uniweb wallet claim` |

Install the skill once, and Claude learns how to add payments. After that, describe what you need in plain language.

## Install (Codex)

```bash
git clone https://github.com/uniwebpay/skills.git uniwebpay-skills
mkdir -p ~/.codex/skills
cp -R uniwebpay-skills/skills/uniwebpay ~/.codex/skills/uniweb
```

Then invoke the skill explicitly:

```text
Use $uniweb to add a checkout button to my Express app.
```

## What It Can Do

- Generate checkout session integration code for Express, Next.js, and other Node.js frameworks
- Set up webhook handlers with signature verification
- Configure subscription billing with trials and dunning
- Create payment links for no-code payment collection
- Help with CLI commands and REST API calls
- Debug payment issues

## Links

- [uniweb Docs](https://skill.uniwebpay.com/docs)
- [API Reference](https://skill.uniwebpay.com/docs/api-reference)
- [Dashboard](https://skill.uniwebpay.com/dashboard)
