---
name: stripe-link-cli
description: Enable AI agents to request payment credentials from Link wallets for secure purchases without exposing real card details
triggers:
  - "request payment credentials from Link"
  - "create a spend request for purchasing"
  - "get a virtual card for checkout"
  - "use Link CLI to pay a merchant"
  - "approve a payment through Link wallet"
  - "set up Link authentication"
  - "pay using Machine Payment Protocol"
  - "create a shared payment token"
---

# Stripe Link CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Link CLI lets AI agents get secure, one-time-use payment credentials from a Link wallet to complete purchases on your behalf. Your real card details are never exposed — you approve every purchase through the Link mobile app. The CLI can produce virtual cards (PAN) for standard web checkouts or Shared Payment Tokens (SPT) for Machine Payment Protocol (MPP) merchants.

## Installation

```bash
npm i -g @stripe/link-cli
```

Or run directly with `npx`:

```bash
npx @stripe/link-cli
```

### Agent Integration

Install as a skill:

```bash
npx skills add stripe/link-cli
```

By default, commands use `toon` output (compact, LLM-friendly text) when called from agents (non-TTY). All commands accept `--format [json|yaml|md|jsonl]` for structured output.

### MCP Server

Add to your MCP client config (`.mcp.json`):

```json
{
  "mcpServers": {
    "link": {
      "command": "npx",
      "args": ["@stripe/link-cli", "--mcp"]
    }
  }
}
```

## Authentication

Before using Link CLI, authenticate with a Link account:

```bash
# Login with custom client name for identification
link-cli auth login --client-name "Claude Code"

# Check authentication status
link-cli auth status

# Logout
link-cli auth logout
```

The login flow provides a verification URL and short phrase. The user visits the URL, logs in to Link, and enters the phrase to approve the connection.

**Using custom auth file:**

```bash
# Store credentials in specific file
link-cli auth login --auth /path/to/auth.json

# Use credentials from specific file (all commands)
link-cli payment-methods list --auth /path/to/auth.json
```

Or set `LINK_AUTH_FILE` environment variable:

```bash
export LINK_AUTH_FILE=/path/to/auth.json
link-cli payment-methods list
```

## Payment Methods & Addresses

### List Payment Methods

```bash
# List all saved payment methods
link-cli payment-methods list

# JSON output
link-cli payment-methods list --format json
```

Returns cards and bank accounts saved to the Link account. Use the `id` field as `payment_method_id` when creating spend requests.

### List Shipping Addresses

```bash
# List saved shipping addresses
link-cli shipping-address list

# JSON output
link-cli shipping-address list --format json
```

## Spend Requests

A spend request is a request for payment credentials tied to a specific purchase. The lifecycle: **create** → **request approval** → **approved** (with credentials).

### Create Spend Request

```bash
link-cli spend-request create \
  --payment-method-id csmrpd_xxx \
  --merchant-name "Stripe Press" \
  --merchant-url "https://press.stripe.com" \
  --context "Purchasing 'Working in Public' from press.stripe.com. The user initiated this purchase through the shopping assistant to learn about open source sustainability." \
  --amount 3500 \
  --line-item "name:Working in Public,unit_amount:3500,quantity:1" \
  --total "type:total,display_text:Total,amount:3500" \
  --request-approval
```

**Required fields:**
- `--payment-method-id`: ID from `payment-methods list`
- `--merchant-name`: Merchant display name
- `--merchant-url`: Merchant website URL
- `--context`: Detailed description (minimum 100 characters)
- `--amount`: Total amount in cents (max 50000)

**Optional fields:**
- `--currency`: 3-letter ISO code (default: `usd`)
- `--credential-type`: `virtual_card` (default) or `shared_payment_token`
- `--shipping-address-id`: ID from `shipping-address list`
- `--request-approval`: Request approval immediately after creation

**Constraints:**
- `context` must be at least 100 characters
- `amount` must not exceed 50000 cents
- `currency` must be valid 3-letter ISO code

### Line Items Format

Use repeatable `--line-item` flags with `key:value` pairs:

```bash
--line-item "name:Running Shoes,unit_amount:12000,quantity:1,description:Trail runners,sku:SKU-123" \
--line-item "name:Socks,unit_amount:1500,quantity:2"
```

**Available keys:**
- `name` (required)
- `quantity`
- `unit_amount`
- `description`
- `sku`
- `url`
- `image_url`
- `product_url`

### Totals Format

Use repeatable `--total` flags:

```bash
--total "type:subtotal,display_text:Subtotal,amount:12000" \
--total "type:shipping,display_text:Shipping,amount:500" \
--total "type:tax,display_text:Tax,amount:950" \
--total "type:total,display_text:Total,amount:13450"
```

**Required keys:**
- `type`: One of `subtotal`, `tax`, `total`, `items_base_amount`, `items_discount`, `discount`, `fulfillment`, `shipping`, `fee`, `gift_wrap`, `tip`, `store_credit`
- `display_text`
- `amount`

### Test Mode

Create test credentials without real payment methods:

```bash
link-cli spend-request create \
  --test \
  --payment-method-id pm_test_card \
  --merchant-name "Test Merchant" \
  --merchant-url "https://example.com" \
  --context "Testing payment flow with test credentials. This is a development test to verify the integration before using real payment methods." \
  --amount 1000 \
  --line-item "name:Test Item,unit_amount:1000,quantity:1" \
  --total "type:total,display_text:Total,amount:1000"
```

### Retrieve Spend Request

```bash
# Basic retrieval (card masked)
link-cli spend-request retrieve lsrq_001

# Include unmasked card details
link-cli spend-request retrieve lsrq_001 --include card

# Write card to secure file (stdout shows redacted data only)
link-cli spend-request retrieve lsrq_001 \
  --include card \
  --output-file /tmp/link-card.json \
  --format json
```

When using `--output-file`:
- File is created with `0600` permissions
- Command fails if file exists unless `--force` is passed
- Stdout shows redacted card data (brand, last4, expiry)
- Full card details written to file
- JSON output includes `card_output_file` path instead of full `card` object

### Polling for Approval

```typescript
// Poll until terminal status (approved, denied, expired, canceled)
link-cli spend-request retrieve lsrq_001 \
  --interval 2 \
  --max-attempts 300
```

Polling exits successfully only when the request reaches a terminal status. If polling times out or exhausts attempts while the request is still pending, the command exits with non-zero status and `code: "POLLING_TIMEOUT"`.

### Update Spend Request

```bash
# Update before approval
link-cli spend-request update lsrq_001 \
  --merchant-url https://press.stripe.com/working-in-public \
  --amount 4000
```

### Request Approval

```bash
# Request approval separately (alternative to --request-approval flag)
link-cli spend-request request-approval lsrq_001
```

### Cancel Spend Request

```bash
# Cancel from any non-terminal state
link-cli spend-request cancel lsrq_001
```

## Using Credentials

### Virtual Card Checkout

After approval, retrieve the spend request with card details:

```bash
link-cli spend-request retrieve lsrq_001 --include card --output-file /tmp/card.json --format json
```

The card object includes:
- `number`: Full card number
- `cvc`: CVV/CVC code
- `exp_month`: Expiration month
- `exp_year`: Expiration year
- `billing_address`: Address object
- `valid_until`: Timestamp when credentials expire

Enter these details into the merchant's checkout form.

**TypeScript example:**

```typescript
import { readFileSync } from 'fs';

// Read card from secure file
const cardData = JSON.parse(readFileSync('/tmp/card.json', 'utf8'));
const card = cardData.card;

// Use card details in checkout automation
await fillCheckoutForm({
  cardNumber: card.number,
  cvc: card.cvc,
  expMonth: card.exp_month,
  expYear: card.exp_year,
  billingAddress: card.billing_address
});
```

### Machine Payment Protocol (MPP)

For merchants supporting MPP, create a spend request with `credential_type: shared_payment_token`:

```bash
# Create SPT spend request
link-cli spend-request create \
  --payment-method-id csmrpd_xxx \
  --merchant-name "Climate Stripe" \
  --merchant-url "https://climate.stripe.dev" \
  --context "Contributing to climate initiatives through climate.stripe.dev. The user wants to offset carbon emissions from their recent travel." \
  --amount 100 \
  --credential-type "shared_payment_token" \
  --line-item "name:Carbon Offset,unit_amount:100,quantity:1" \
  --total "type:total,display_text:Total,amount:100" \
  --request-approval

# After approval, pay using MPP
link-cli mpp pay https://climate.stripe.dev/api/contribute \
  --spend-request-id lsrq_001 \
  --method POST \
  --data '{"amount":100}' \
  --header "Content-Type: application/json"
```

The SPT is one-time-use. If payment fails, create a new spend request.

### Decode MPP Challenge

Extract payment information from a `WWW-Authenticate` header:

```bash
link-cli mpp decode \
  --challenge 'Payment id="ch_001", realm="merchant.example", method="stripe", intent="charge", request="..."'
```

Returns the `network_id` needed for SPT spend requests and decoded challenge parameters.

## Command Schema

Get full JSON schema for any command:

```bash
# Get schema for create command
link-cli spend-request create --schema

# List all commands in LLM-friendly format
link-cli --llms-full
```

## Common Patterns

### Agent Purchase Flow (Virtual Card)

```typescript
import { execSync } from 'child_process';

// 1. Get payment methods
const pmResult = execSync('link-cli payment-methods list --format json', { encoding: 'utf8' });
const paymentMethods = JSON.parse(pmResult);
const pmId = paymentMethods[0].id;

// 2. Create spend request
const createCmd = `link-cli spend-request create \
  --payment-method-id ${pmId} \
  --merchant-name "Acme Store" \
  --merchant-url "https://acme.example" \
  --context "Purchasing office supplies from acme.example. The user requested pens and notebooks for the team meeting next week." \
  --amount 2500 \
  --line-item "name:Blue Pens,unit_amount:1000,quantity:1" \
  --line-item "name:Notebook,unit_amount:1500,quantity:1" \
  --total "type:total,display_text:Total,amount:2500" \
  --request-approval \
  --format json`;

const createResult = JSON.parse(execSync(createCmd, { encoding: 'utf8' }));
const spendRequestId = createResult.id;

// 3. Poll for approval
const pollCmd = `link-cli spend-request retrieve ${spendRequestId} \
  --interval 2 \
  --max-attempts 300 \
  --include card \
  --output-file /tmp/card-${spendRequestId}.json \
  --format json`;

const approvedResult = JSON.parse(execSync(pollCmd, { encoding: 'utf8' }));

// 4. Read card from secure file
const cardFile = `/tmp/card-${spendRequestId}.json`;
const cardData = JSON.parse(readFileSync(cardFile, 'utf8'));

// 5. Use card in checkout
await completeCheckout(cardData.card);
```

### Agent Purchase Flow (MPP)

```typescript
import { execSync } from 'child_process';

// 1. Decode MPP challenge to get network_id
const challenge = response.headers['www-authenticate'];
const decodeResult = execSync(
  `link-cli mpp decode --challenge '${challenge}' --format json`,
  { encoding: 'utf8' }
);
const { network_id } = JSON.parse(decodeResult);

// 2. Create SPT spend request
const createCmd = `link-cli spend-request create \
  --payment-method-id ${pmId} \
  --merchant-name "Climate Stripe" \
  --merchant-url "https://climate.stripe.dev" \
  --context "Contributing to climate offset program through climate.stripe.dev. The user wants to neutralize carbon emissions from their last flight." \
  --amount 100 \
  --credential-type "shared_payment_token" \
  --line-item "name:Carbon Offset,unit_amount:100,quantity:1" \
  --total "type:total,display_text:Total,amount:100" \
  --request-approval \
  --format json`;

const createResult = JSON.parse(execSync(createCmd, { encoding: 'utf8' }));
const spendRequestId = createResult.id;

// 3. Poll for approval
execSync(`link-cli spend-request retrieve ${spendRequestId} --interval 2 --max-attempts 300`);

// 4. Pay using MPP
const payCmd = `link-cli mpp pay https://climate.stripe.dev/api/contribute \
  --spend-request-id ${spendRequestId} \
  --method POST \
  --data '{"amount":100}' \
  --header "Content-Type: application/json" \
  --format json`;

const payResult = JSON.parse(execSync(payCmd, { encoding: 'utf8' }));
```

### Multiple Auth Sessions

```typescript
// User 1 session
process.env.LINK_AUTH_FILE = '/secure/user1-auth.json';
execSync('link-cli auth login --client-name "User1 Agent"');

// User 2 session
process.env.LINK_AUTH_FILE = '/secure/user2-auth.json';
execSync('link-cli auth login --client-name "User2 Agent"');

// Use different sessions
function getPaymentMethodsForUser(userId: string) {
  const authFile = `/secure/${userId}-auth.json`;
  return execSync(`link-cli payment-methods list --auth ${authFile} --format json`, {
    encoding: 'utf8'
  });
}
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `LINK_AUTH_FILE` | Override auth credential file path (flag `--auth` takes precedence) |
| `LINK_API_BASE_URL` | Override API base URL |
| `LINK_AUTH_BASE_URL` | Override auth base URL |
| `LINK_HTTP_PROXY` | Route requests through HTTP proxy (requires `undici`) |
| `NO_UPDATE_NOTIFIER` | Set to `1` to suppress update checks (useful in CI) |

## Troubleshooting

### Authentication Issues

**Problem:** `link-cli auth status` shows `authenticated: false`

**Solution:**
```bash
# Logout and login again
link-cli auth logout
link-cli auth login --client-name "My Agent"
```

### Spend Request Denied

**Problem:** User denies spend request in Link app

**Solution:** The polling command exits with status reflecting denial. Create a new spend request with updated context or amounts:

```bash
link-cli spend-request create \
  --payment-method-id csmrpd_xxx \
  --merchant-name "Updated Merchant" \
  --merchant-url "https://merchant.example" \
  --context "Revised purchase with updated details per user feedback. The user now wants the blue model instead of red, which has a different price point." \
  --amount 4000 \
  --line-item "name:Blue Model,unit_amount:4000,quantity:1" \
  --total "type:total,display_text:Total,amount:4000" \
  --request-approval
```

### Context Too Short

**Problem:** `context must be at least 100 characters`

**Solution:** Provide detailed purchase context:

```bash
--context "Purchasing premium subscription to ExampleService for the team. This will give us access to advanced features including API access, priority support, and increased usage limits that we need for the upcoming project launch."
```

### Amount Exceeds Limit

**Problem:** `amount must not exceed 50000`

**Solution:** Link CLI enforces a 50000 cent ($500) limit per request. For larger purchases, split into multiple spend requests or handle outside Link CLI.

### Polling Timeout

**Problem:** Polling exits with `code: "POLLING_TIMEOUT"`

**Solution:** Increase `--max-attempts` or check if user has Link app installed:

```bash
# Poll for up to 15 minutes (2s interval × 450 attempts)
link-cli spend-request retrieve lsrq_001 \
  --interval 2 \
  --max-attempts 450
```

### MPP Payment Fails

**Problem:** SPT payment returns error

**Solution:** SPTs are one-time-use. Create a new spend request and try again:

```bash
# Cancel old request
link-cli spend-request cancel lsrq_001

# Create new request
link-cli spend-request create \
  --payment-method-id csmrpd_xxx \
  --credential-type "shared_payment_token" \
  # ... rest of flags
```

### File Already Exists

**Problem:** `--output-file` fails because file exists

**Solution:** Use `--force` to overwrite or choose different path:

```bash
link-cli spend-request retrieve lsrq_001 \
  --include card \
  --output-file /tmp/card.json \
  --force
```

## Demo & Onboarding

### Interactive Demo

```bash
# Run interactive demo (test mode)
link-cli demo

# Only virtual card flow
link-cli demo --only-card

# Only MPP/SPT flow
link-cli demo --only-spt
```

### Guided Onboarding

```bash
# Complete setup flow
link-cli onboard
```

Walks through authentication, payment method checks, app download, and both demo flows.
