---
name: uniwebpay
description: Help developers safely add uniweb payments to an app. Use when the user asks to accept card, WeChat Pay, Alipay, or PayNow payments; create payment links; add checkout, subscriptions, billing, or invoice-style payment collection; handle webhooks; install or use @uniwebpay/cli; or install or use @uniwebpay/sdk.
---

# uniweb Payment Integrations

You are helping a developer integrate uniweb into an existing app. Your goal is to choose the lightest correct payment path, generate server-safe code, and make fulfillment depend on verified webhooks instead of browser redirects.

Before editing an app, inspect its framework, routing style, package manager, and environment variable conventions. Keep secrets in server-side environment variables and match the project's existing patterns.

## Choose The Integration Path

| User needs | Use | Why |
| --- | --- | --- |
| One fixed amount, no backend | Payment Link | Permanent `/p/plink_xxx` URL; safe to put in HTML, email, invoices, or social posts. |
| Fixed product or subscription price | Product + Price URL | Permanent `/buy/price_xxx` URL; good for pricing pages and recurring plans. |
| Cart, quantity, order, POS, dynamic amount, or per-request metadata | TypeScript SDK + Checkout Session | Create a one-time session on the server, then redirect to `session.url`. |

When the user says "invoice", use a Payment Link for simple invoice-style collection unless they need a real per-order checkout session. uniweb does not expose a separate invoice API for merchants.

## Security Defaults

1. Never put `sk_live_` in deployed code or browser code. It is a full key for trusted local/admin use.
2. For backends, create and use `sk_server_` keys with the smallest scopes needed.
3. Prefer `@uniwebpay/sdk` over hand-written `fetch()` for server integrations.
4. The SDK must run only on the server: Node.js, Cloudflare Workers, Deno, or Bun. Do not import it in browser bundles.
5. Fulfill orders and grant access only after a verified webhook. Treat `successUrl` as UX only.
6. Amounts are integer cents/minor units. Default currency is `SGD`.
7. Never print, commit, or copy API keys, webhook secrets, or `~/.uniweb/config.json` into an app repo.

Server keys are scope-based. Default server key scopes are `products.read`, `products.write`, `prices.read`, `prices.write`, `checkout.create`, `payments.read`, `customers.read`, `customers.write`, `subscriptions.read`, `payment_links.read`, `payment_links.write`, and `refunds.read`. Extra scopes such as `payments.write`, `subscriptions.write`, `refunds.create`, `wallet.update`, `payouts.create`, `kyc.manage`, or `bank_accounts.manage` should be granted only when the backend truly needs those write, money-moving, or account-management actions.

## Install Dependencies Automatically

When a task needs the CLI, verify the `uniweb` command first. The package name is `@uniwebpay/cli`, but the installed command is `uniweb`. If it is missing and shell access is available, install it automatically instead of only telling the user to install it. Never use `sudo`, `curl | sh`, arbitrary tarballs, or unofficial package names.

```bash
command -v uniweb >/dev/null 2>&1 || npm install -g @uniwebpay/cli
uniweb --version
```

If `uniweb` exists but `uniweb --version` or `uniweb --help` does not behave like the uniweb CLI, reinstall `@uniwebpay/cli` or report the path conflict. The CLI reads `UNIWEB_SECRET` first, then `~/.uniweb/config.json`; `uniweb wallet create` stores a generated key there.

When server code will import the SDK, inspect the app's package manager and install `@uniwebpay/sdk` into the project if it is not already present in `package.json`. Choose the package manager from the existing lockfile: `pnpm-lock.yaml` -> `pnpm add`, `yarn.lock` -> `yarn add`, `bun.lock`/`bun.lockb` -> `bun add`, otherwise use `npm install`.

```bash
npm install @uniwebpay/sdk
pnpm add @uniwebpay/sdk
yarn add @uniwebpay/sdk
bun add @uniwebpay/sdk
```

Do not install the SDK for pure Payment Link HTML changes unless code will import `@uniwebpay/sdk`. If installation fails because npm, permissions, or network are unavailable, report the exact blocker and include the command the user should run.

## Path A: Payment Link

Use this for a fixed amount with no backend.

```bash
npm install -g @uniwebpay/cli
uniweb wallet create
uniweb link create 1000 -c SGD -n "My Product" --methods card,wechat,alipay,paynow
# returns https://skill.uniwebpay.com/p/plink_xxx
```

```html
<a href="https://skill.uniwebpay.com/p/plink_xxx">Pay $10.00</a>
```

Optional fulfillment notification:

```bash
uniweb webhook set https://yoursite.com/webhook
```

Notes:

- Payment Links only support one-time payments. For subscriptions, create a recurring Price and share its `/buy/price_xxx` URL.
- If `--methods` is omitted, defaults depend on amount and currency: SGD amounts under `10` minor units use WeChat/Alipay/PayNow; SGD amounts `>= 10` use card/WeChat/Alipay/PayNow; CNY amounts under `10` use WeChat/Alipay; CNY amounts `>= 10` use card/WeChat/Alipay; other supported currencies use card only and therefore must meet the card minimum.
- PayNow is rejected for non-SGD links. WeChat Pay and Alipay are rejected outside SGD/CNY. Card is rejected below the `10` minor-unit minimum.

## Path B: Product + Price URL

Use this for a known product, pricing page, or subscription plan.

```bash
uniweb product create "Pro Plan"
uniweb price create prod_xxx -a 999 -t recurring -i month -c SGD
# returns https://skill.uniwebpay.com/buy/price_xxx
```

```html
<a href="https://skill.uniwebpay.com/buy/price_xxx">Subscribe - $9.99/month</a>
```

Notes:

- One-time prices use `-t one_time` and do not need `-i`.
- Subscriptions support card payments only. Ignore WeChat Pay, Alipay, and PayNow for subscription checkout.
- Customer email is collected at checkout; uniweb creates or reuses the Customer record.
- Subscription statuses: `trialing`, `active`, `past_due`, `unpaid`, `canceled`.
- Renewal dunning retries follow 1 hour, 1 day, 3 days, and 7 days between attempts; after all allowed attempts are exhausted, the subscription becomes `unpaid`.

Subscription management from a trusted machine:

```bash
uniweb subscription list
uniweb subscription pause <id>       # cancel at period end
uniweb subscription resume <id>      # undo pause
uniweb subscription cancel <id>      # cancel immediately
```

## Path C: TypeScript SDK + Checkout Session

Use this when the amount, quantity, customer, order, or metadata is dynamic.

Create a restricted server key:

```bash
uniweb key create -t server -n "Checkout Backend"
```

Install and use the SDK on the backend:

```bash
npm install @uniwebpay/sdk
```

```typescript
import Uniweb from '@uniwebpay/sdk';

const uniweb = new Uniweb(process.env.UNIWEB_SERVER_KEY!);

const product = await uniweb.products.create({
  name: 'Nasi Lemak',
  description: 'Spicy coconut rice',
});

const price = await uniweb.prices.create({
  productId: product.id,
  amount: 500,
  currency: 'SGD',
  type: 'one_time',
});

// For fixed product pages, this permanent URL is enough.
res.redirect(price.paymentUrl);
```

For carts or per-order checkout, create a fresh session server-side and redirect immediately:

```typescript
const session = await uniweb.checkout.create({
  mode: 'payment',
  lineItems: [{ priceId: price.id, quantity: 2 }],
  successUrl: 'https://yoursite.com/success',
  cancelUrl: 'https://yoursite.com/cancel',
  customerEmail: 'customer@example.com',
  paymentMethodTypes: ['card', 'wechat', 'alipay', 'paynow'],
  metadata: { orderId: 'ord_123' },
});

res.redirect(session.url!);
```

Checkout Session URLs expire after 24 hours and are one-time checkout URLs. That is correct for production dynamic orders; create them at checkout time. Do not store them as permanent product links.

For subscriptions through the SDK:

```typescript
const session = await uniweb.checkout.create({
  mode: 'subscription',
  lineItems: [{ priceId: recurringPrice.id, quantity: 1 }],
  successUrl: 'https://yoursite.com/billing/success',
  cancelUrl: 'https://yoursite.com/billing/cancel',
  trialPeriodDays: 14,
  paymentMethodTypes: ['card'],
});
```

## Webhook Fulfillment

Set a webhook URL and save the `whsec_xxx` secret. Webhook URLs must be HTTPS.

```bash
uniweb webhook set https://yoursite.com/webhook
```

Use the SDK verifier with the raw request body:

```typescript
import express from 'express';
import { verifyWebhook } from '@uniwebpay/sdk';

app.post('/webhook', express.raw({ type: 'application/json' }), async (req, res) => {
  let event;

  // Step 1: verify the signature. A bad signature is a permanent failure —
  // return 4xx so uniweb does NOT retry a request it can never accept.
  try {
    event = await verifyWebhook(
      req.body.toString('utf8'),
      req.header('uniweb-Signature') ?? '', // header lookup is case-insensitive
      process.env.UNIWEB_WEBHOOK_SECRET!,
    );
  } catch {
    return res.status(400).send('Invalid signature');
  }

  // Step 2: process the event. If your own handling fails (e.g. a DB write),
  // return 5xx so uniweb retries delivery later. Do NOT return 2xx/4xx here,
  // or the retry is suppressed and the event is silently dropped.
  try {
    switch (event.type) {
      case 'payment.succeeded':
        // Mark the order paid after checking amount/currency/metadata.
        break;
      case 'payment.failed':
        // Keep the order unpaid or notify the customer.
        break;
      case 'checkout.session.completed':
        // Optional: record checkout completion.
        break;
      case 'subscription.created':
      case 'subscription.renewed':
        // Grant or extend access.
        break;
      case 'subscription.past_due':
      case 'subscription.unpaid':
      case 'subscription.canceled':
        // Update billing/access state.
        break;
      case 'refund.succeeded':
      case 'refund.failed':
        // Update refund state.
        break;
    }

    res.status(200).send('ok');
  } catch {
    res.status(500).send('Processing error');
  }
});
```

Webhook rules:

- Do not use JSON body parsing before verification; verification needs the exact raw body.
- Enforce idempotency using `event.id`, payment id, or your `metadata.orderId`.
- Before fulfillment, check expected amount, currency, customer/order metadata, and current order state.
- Return a 2xx only after durable processing. uniweb sends an immediate delivery attempt, retries failed deliveries after 5 minutes, 30 minutes, 2 hours, and 12 hours, and marks the event failed after 6 total attempts.
- Delivery uses `uniweb-Signature: t=<unix>,v1=<hmac-sha256-hex>`, `uniweb-Event-Id`, and `uniweb-Timestamp` headers. Webhook delivery has a 10 second timeout and does not follow redirects.
- Per-link webhook overrides per-product webhook, which overrides the wallet webhook. The signing secret is wallet-level.

Events:

```text
payment.succeeded, payment.failed
refund.succeeded, refund.failed
checkout.session.completed, checkout.session.expired
subscription.created, subscription.renewed, subscription.past_due,
subscription.unpaid, subscription.canceled, subscription.trial_ending
payout.requested, payout.approved, payout.settled, payout.rejected, payout.failed
account.kyc.approved, account.kyc.rejected
```

## SDK Surface

```text
await uniweb.products.create({ name, description?, webhookUrl?, metadata? });
await uniweb.products.list({ limit?, startingAfter? });
await uniweb.products.get(id);
await uniweb.products.update(id, { name?, description?, webhookUrl?, active?, metadata? });
await uniweb.products.del(id);
for await (const p of uniweb.products.listAll()) {}

await uniweb.prices.create({ productId, amount, currency, type, interval?, intervalCount?, trialPeriodDays?, metadata? });
await uniweb.prices.list({ productId?, limit?, startingAfter? });
await uniweb.prices.get(id);
await uniweb.prices.update(id, { active });
await uniweb.prices.activate(id);
await uniweb.prices.deactivate(id);
for await (const p of uniweb.prices.listAll({ productId? })) {}

await uniweb.checkout.create({ mode, lineItems, successUrl?, cancelUrl?, customerEmail?, customerId?, trialPeriodDays?, paymentMethodTypes?, metadata? });
await uniweb.checkout.list({ limit?, startingAfter? });
await uniweb.checkout.get(id);

await uniweb.payments.create({ amount, currency, customerId?, metadata? }); // requires payments.write
await uniweb.payments.list({ status?, customerId?, limit?, startingAfter? });
await uniweb.payments.get(id, { gateway? });
await uniweb.payments.listRefunds(id);
await uniweb.payments.sync(id);                                            // requires payments.write
await uniweb.payments.void(id);                                            // requires refunds.create
for await (const p of uniweb.payments.listAll({ status?, customerId? })) {}

await uniweb.refunds.create({ paymentId, amount?, reason?, offlineRefundFlag? }); // requires refunds.create
await uniweb.refunds.get(id, { gateway? });                                       // gateway:true -> gateway enquiry

await uniweb.customers.create({ email, name?, metadata? });
await uniweb.customers.list({ email?, limit?, startingAfter? });
await uniweb.customers.get(id);
await uniweb.customers.update(id, { email?, name?, metadata? });
await uniweb.customers.del(id);
for await (const c of uniweb.customers.listAll({ email? })) {}

await uniweb.subscriptions.create({ customerId, priceId, paymentMethodId?, trialPeriodDays?, metadata? });
await uniweb.subscriptions.list({ customerId?, status?, limit?, startingAfter? });
await uniweb.subscriptions.get(id);
await uniweb.subscriptions.update(id, { cancelAtPeriodEnd? });
await uniweb.subscriptions.cancel(id);                                     // cancel immediately
await uniweb.subscriptions.resume(id);                                     // undo "cancel at period end"
for await (const s of uniweb.subscriptions.listAll({ customerId?, status? })) {}

await uniweb.links.create({ amount, currency, name?, description?, successUrl?, cancelUrl?, webhookUrl?, paymentMethodTypes?, metadata? });
await uniweb.links.list({ limit?, startingAfter? });
await uniweb.links.get(id);
await uniweb.links.update(id, { name?, description?, successUrl?, cancelUrl?, webhookUrl?, active? });
await uniweb.links.deactivate(id);
for await (const l of uniweb.links.listAll()) {}

await uniweb.wallet.current();
await uniweb.wallet.update({ merchantName?, merchantCity?, merchantCountry?, webhookUrl? });

// Wallet-level webhook configuration. The signing secret returned by set/rollSecret
// is the only place you can read whsec_xxx — store it before the call returns.
await uniweb.webhooks.set(url);          // returns wallet webhook data, including webhookSecret when available
await uniweb.webhooks.info();            // current wallet, url, hasSecret
await uniweb.webhooks.remove();
await uniweb.webhooks.rollSecret();      // returns webhookSecret when available; invalidates the previous secret
```

The SDK constructor reads from `https://apiskill.uniwebpay.com` (production API edge) and `https://skill.uniwebpay.com` (production pay/checkout host) by default. Override only when the deployment instructs it (staging, local dev) — wrong overrides silently route real keys to the wrong environment:

```typescript
// Production: no overrides needed.
const uniweb = new Uniweb(process.env.UNIWEB_SERVER_KEY!);

// Staging or local dev only:
const uniweb = new Uniweb(process.env.UNIWEB_SERVER_KEY!, {
  baseUrl: process.env.UNIWEB_API_URL,   // e.g. http://localhost:3000
  payUrl: process.env.UNIWEB_PAY_URL,    // e.g. http://localhost:3001
});
```

## CLI Reference

```bash
npm install -g @uniwebpay/cli

# Global option for machine-readable output
uniweb --json <command>

# Wallet and dashboard claim
uniweb wallet create
uniweb wallet info
uniweb wallet claim

# Interactive quick-start — prompts for input on stdin. Do NOT call this in
# non-interactive or agent-driven scripts; it will hang waiting for answers.
# Use the individual product/price/checkout commands below instead.
uniweb create   # walks through creating a product + price + checkout session in one flow

# Products and permanent /buy/ URLs
uniweb product create <name> [-d desc] [-w webhook-url]
uniweb product list
uniweb product get <id>
uniweb product update <id> [-n name] [-d desc] [-w webhook-url]
uniweb product delete <id>
uniweb price create <productId> -a <cents> -t one_time|recurring [-i day|week|month|year] [-c currency]
uniweb price list

# Payment links and permanent /p/ URLs
uniweb link create <cents> [-c currency] [-n name] [-d desc] [--methods card,wechat,alipay,paynow] [--success-url url] [--cancel-url url] [-w webhook-url]
uniweb link list
uniweb link get <id>
uniweb link update <id> [-n name] [-d desc] [--success-url url] [--cancel-url url] [-w webhook-url]
uniweb link activate <id>
uniweb link deactivate <id>

# Checkout sessions: one-time, 24-hour URLs for dynamic orders
uniweb checkout create -p <priceId> --success-url <url> --cancel-url <url> [-m payment|subscription] [-q quantity] [--methods card,wechat,alipay,paynow] [--customer-id <id>] [--trial-days days]
uniweb checkout get <id> [-w] [-i seconds]

# Payments and refunds from trusted/server contexts with appropriate scopes
uniweb payment list [-s status] [-c customerId]
uniweb payment get <id> [--gateway]
uniweb payment void <id>
uniweb payment refund <id> [-a cents] [--offline] [-r reason]
uniweb refund create -p <paymentId> [-a cents] [--offline] [-r reason]
uniweb refund get <id> [--gateway]

# Customers and subscriptions
uniweb customer create -e <email> [-n name]
uniweb customer list
uniweb customer get <id>
uniweb customer update <id> [-e email] [-n name]
uniweb customer delete <id>
uniweb subscription list
uniweb subscription get <id>
uniweb subscription pause <id>
uniweb subscription resume <id>
uniweb subscription cancel <id>

# Keys and webhooks
uniweb key create -t server|full [-n name] [-s scopes]
uniweb key list
uniweb key revoke <id>
uniweb key roll <id>
uniweb webhook set <url>
uniweb webhook info
uniweb webhook remove
uniweb webhook roll-secret
uniweb webhook events

# KYC, bank accounts, payouts
uniweb kyc submit
uniweb kyc status
uniweb bank add
uniweb bank list
uniweb bank remove <id>
uniweb bank set-default <id>
uniweb payout create -a <cents> -b <bankAccountId>
uniweb payout list
uniweb payout status
uniweb payout cancel <id>
```

## Constraints To Remember

- Supported payment methods: `card`, `wechat`, `alipay`, `paynow`.
- Subscriptions: card only.
- Currency support by method:
  - `card` — all uniweb supported currencies: `SGD`, `USD`, `EUR`, `GBP`, `JPY`, `CNY`, `HKD`, `AUD`, `MYR`, `THB`.
  - `paynow` — `SGD` only (DBS PayNow rail).
  - `wechat` — `SGD` and `CNY` only.
  - `alipay` — `SGD` and `CNY` only.
- Card minimum: `10` minor units (e.g. `0.10 SGD`, `0.10 USD`). For zero-decimal currencies such as `JPY`, the minor unit is the major unit, so `10 JPY` minimum applies.
- Payment Links and Product/Price URLs are reusable and permanent. Payment Links are one-time only; recurring billing uses Product/Price URLs.
- Checkout Sessions are one-time and expire after 24 hours.
- Prices are effectively immutable for amount/currency; create a new price instead of editing the old one.
- KYC is not required to accept payments, but is required before payouts.

### Subscription status machine

```
trialing → active                   (trial ends with successful charge)
active   → past_due                 (renewal failed; retry schedule below)
past_due → active                   (a retry succeeded)
past_due → unpaid                   (all dunning attempts exhausted; access usually paused)
*        → canceled                 (`subscriptions.cancel` ends immediately)
active   → canceled                 (`pause` cancels at the current period end when the renewal cron reaches it)
*        → cancelAtPeriodEnd=true   (`pause` schedules cancel at current period end; `resume` undoes)
```

Renewal dunning uses `MAX_DUNNING_ATTEMPTS = 5` with retry delays of `1h`, `1d`, `3d`, and `7d` between attempts; when the next attempt would exceed the maximum, the subscription transitions to `unpaid`. The code does not automatically cancel an `unpaid` subscription after a grace window. Listen on `subscription.past_due` / `subscription.unpaid` / `subscription.canceled` for access changes.
