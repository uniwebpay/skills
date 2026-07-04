# Security Policy

uniwebpay skills help agents add payment flows, create payment links, and handle webhook-based fulfillment. Treat security issues in this repository as payment integration issues, even when the affected file is documentation or agent instructions.

## Reporting a Vulnerability

Please use GitHub private vulnerability reporting for this repository. Do not open a public issue for suspected vulnerabilities, leaked credentials, bypasses, or exploitable payment-flow behavior.

Report privately if you find:

- Instructions that could cause API keys, webhook secrets, wallet files, or environment variables to be printed, committed, or copied into application code.
- Guidance that could let an integration fulfill orders before webhook signature verification.
- Incorrect webhook verification, idempotency, amount, currency, customer, or metadata checks.
- Overly broad key-scope guidance for normal server integrations.
- Broken payment-method, currency, refund, subscription, or payout safety guidance.
- Any committed secret, token, private key, or credential-like value.

When possible, include:

- The affected file and line.
- A short reproduction or example prompt.
- The expected secure behavior.
- Whether real credentials, accounts, or payments were involved.

## Response Expectations

We aim to acknowledge valid private reports within 3 business days and provide an initial assessment within 7 business days. Fix timing depends on severity and whether a change is needed in this skills repository, the uniweb CLI, SDK, API, or hosted dashboard.

## Supported Versions

The supported version is the current `main` branch and the latest published state of this repository.

## Credential Handling

If a real credential was exposed, revoke or rotate it immediately. Removing it from Git history or from a pull request is not enough once it has been pushed.
