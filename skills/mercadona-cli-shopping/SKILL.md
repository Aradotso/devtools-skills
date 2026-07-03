---
name: mercadona-cli-shopping
description: Use the Mercadona CLI to search products, manage shopping carts, and automate grocery orders from Spain's Mercadona supermarket
triggers:
  - search for products on mercadona
  - add items to my mercadona cart
  - get mercadona product prices
  - create a mercadona shopping list
  - check out my mercadona order
  - find the price per kilo on mercadona
  - import mercadona authentication
  - set mercadona warehouse location
---

# Mercadona CLI Shopping

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

`mercadona-cli` is an unofficial, agent-friendly command-line interface for Spain's Mercadona supermarket (`tienda.mercadona.es`). It provides structured access to the catalog, pricing, cart management, and checkout flow through a single Go binary with no runtime dependencies.

**Key capabilities:**
- Search products by name with full-text search
- Get real-time prices and product details (per warehouse)
- Build and manage shopping carts programmatically
- Complete checkout flow (with authentication)
- Structured JSON output for automation
- Batch operations (price many items in one request)
- Budget guards to prevent overspending

**Important:** This is unofficial and uses the same HTTP endpoints as the website. Bring your own Mercadona credentials and use at a reasonable request rate.

## Installation

### Via npm (recommended)
```bash
npm install -g @ivorpad/mercadona
# Or run without installing:
npx @ivorpad/mercadona search queso
```

### Via curl (macOS/Linux)
```bash
curl -fsSL https://raw.githubusercontent.com/ivorpad/mercadona-cli/main/install.sh | sh
```

### From source (Go 1.26+)
```bash
git clone https://github.com/ivorpad/mercadona-cli.git
cd mercadona-cli
go build -o mercadona ./cmd/mercadona
```

## Configuration

### Set warehouse location (required for accurate pricing)

Product IDs and prices are warehouse-specific. Set your warehouse from your postal code:

```bash
mercadona set-postal 28022
# Resolves to warehouse (e.g., mad1) and saves as default
```

This creates `~/.mercadona/config.toml`:
```toml
[defaults]
warehouse = "mad1"
postal_code = "28022"
```

### Authentication (for cart/checkout operations)

The preferred method is to import a HAR file from your browser session:

```bash
# 1. Log in to tienda.mercadona.es in your browser (email or Google)
# 2. Open DevTools → Network tab
# 3. Export HAR (⤓ "Export HAR...")
# 4. Import it:
mercadona import-har --file tienda.mercadona.es.har

# Verify authentication:
mercadona whoami
```

This extracts the `refresh_token` (which auto-renews headlessly forever) and saves it to config:

```toml
[auth]
refresh_token = "your_refresh_token_here"
```

Alternative one-off method (expires after ~6 weeks):
```bash
# Copy any authenticated request as cURL from DevTools
mercadona import-curl --file request.txt
```

### Budget limits (agent safety)

Prevent overspending by setting a maximum budget:

```toml
[limits]
max_eur = 100  # Refuse any cart/checkout over 100€
```

Or use flags/env vars:
```bash
mercadona cart add 51110 2 --max 50
export MERCADONA_MAX_EUR=100
```

## Core Commands (No Authentication Required)

### Search products

```bash
# Basic search
mercadona search queso

# Limit results
mercadona search queso --limit 5

# JSON output for automation
mercadona search mayonesa --json --limit 3
```

Example JSON output:
```json
{
  "results": [
    {
      "id": "13406",
      "display_name": "Mayonesa Hacendado",
      "price": 1.20,
      "reference_price": 2.400,
      "reference_format": "€/L"
    }
  ]
}
```

### Get fresh items only (skip frozen/canned)

```bash
mercadona search mejillon --fresh --limit 1
# Returns fresh seafood instead of canned mussels
```

### Batch search (multiple terms in one request)

```bash
# From a file
cat lista.txt
queso
carne
mayonesa

mercadona batch -f lista.txt

# From stdin
printf 'queso\ncarne\nmayonesa\n' | mercadona batch -f -
```

Output:
```
• queso    → [51110] Queso rallado mozzarella pizza-Roma Hacendado — 1.60€ (8.000€/kg)
• carne    → [34157] Carne de pimiento choricero Hacendado — 1.55€ (11.072€/kg)
• mayonesa → [13406] Mayonesa Hacendado — 1.20€ (2.400€/L)
```

### Product details

```bash
mercadona product 13406
# Returns: name, price, reference_price, nutrition info (when available)

mercadona product 13406 --json
```

### Browse categories

```bash
# List all categories
mercadona categories

# Get products in a specific category
mercadona categories --id 118 --json  # 118 = Rice category
```

### Price a shopping basket

Create a basket file with `<product_id> <quantity>` format:

```text
# paella.txt - comments are allowed
5044  1     # Arroz redondo Hacendado
60393 1     # Gambón grande congelado
85499 1     # Mejillón mediterráneo
16044 1     # Tomate triturado Hacendado
4740  0.5   # Aceite de oliva virgen extra
```

Price it:
```bash
mercadona total -f paella.txt
# Output shows each line item and total in euros

mercadona total -f paella.txt --json
# Returns: {"lines": [...], "total": 1603, "count": 5, "complete": true}
```

## Cart & Checkout Commands (Authentication Required)

### Verify authentication

```bash
mercadona whoami
# Output: "ok — customer id=..."
```

### Get current cart

```bash
mercadona cart get
mercadona cart get --json
```

### Add items to cart

```bash
# Add 2 units of product 51110 (adds to existing quantity)
mercadona cart add 51110 2

# With budget guard
mercadona cart add 51110 2 --max 80
```

### Set absolute quantity

```bash
# Set quantity to exactly 3 (overwrites existing)
mercadona cart set 51110 3

# Remove item (set to 0)
mercadona cart set 51110 0
```

### Bulk cart operations

```bash
# Load many items from a file (one write operation)
cat basket.txt
51110 2
13406 1
5044 3

mercadona cart set-many -f basket.txt --max 80

# From stdin
printf '51110 2\n13406 1\n' | mercadona cart set-many -f - --max 80
```

### Clear cart

```bash
mercadona cart clear
```

### Checkout flow

```bash
# 1. Create a checkout
mercadona checkout create --json
# Returns: checkout_id, default_address

# 2. List delivery addresses
mercadona checkout addresses

# 3. Get available delivery slots for an address
mercadona checkout slots --address <address_id>

# 4. Set delivery details
mercadona checkout set-delivery \
  --checkout <checkout_id> \
  --address <address_id> \
  --slot <slot_id>

# 5. Review checkout
mercadona checkout get --checkout <checkout_id>

# 6. Submit order (IRREVERSIBLE - places real order)
mercadona checkout submit \
  --checkout <checkout_id> \
  --max 80 \
  --yes
```

## Common Patterns

### Convert plain-text shopping list to IDs

```bash
# 1. Create plain text list
cat shopping_list.txt
arroz redondo hacendado
gambón grande congelado
tomate triturado hacendado

# 2. Get product IDs and prices
mercadona batch -f shopping_list.txt --json > products.json

# 3. Parse JSON to create basket file (use jq or your language)
jq -r '.results[] | "\(.id) 1"' products.json > basket.txt

# 4. Price it
mercadona total -f basket.txt
```

### Track price changes over time

```bash
#!/bin/bash
# price_tracker.sh - Track specific products daily

PRODUCTS="51110 13406 5044"
DATE=$(date +%Y-%m-%d)

for id in $PRODUCTS; do
  mercadona product $id --json | \
    jq -r "\"$DATE,\(.id),\(.display_name),\(.price)\"" \
    >> price_history.csv
done
```

### Find best value in a category (sort by €/kg)

```bash
# Get rice category (id=118), extract reference_price, sort
mercadona categories --id 118 --json | \
  jq -r '.products[] | [.reference_price, .display_name, .id] | @tsv' | \
  sort -n | \
  head -10
```

### Automated weekly order (agent-safe)

```bash
#!/bin/bash
# weekly_order.sh - Place recurring order with safety checks

export MERCADONA_MAX_EUR=100

# Load standard basket
mercadona cart set-many -f weekly_basket.txt --max 100 || exit 1

# Create checkout
CHECKOUT=$(mercadona checkout create --json | jq -r '.checkout_id')

# Use saved address and preferred slot
mercadona checkout set-delivery \
  --checkout $CHECKOUT \
  --address $ADDRESS_ID \
  --slot $SLOT_ID

# Review total before submitting
TOTAL=$(mercadona checkout get --checkout $CHECKOUT --json | jq -r '.total')
echo "Order total: $TOTAL"

# Submit (requires --yes flag)
mercadona checkout submit --checkout $CHECKOUT --max 100 --yes
```

## Global Flags

These work with any command and can be placed anywhere on the command line:

```bash
--wh mad1              # Override warehouse (default: from config or mad1)
--lang es              # Language code (default: es)
--json                 # Output structured JSON instead of human-readable text
--max 80               # Budget cap in euros (for cart/checkout commands)
```

## Environment Variables

```bash
MERCADONA_TOKEN          # Bearer token (access token)
MERCADONA_COOKIE         # Session cookie
MERCADONA_CUSTOMER       # Customer ID
MERCADONA_MAX_EUR        # Budget limit in euros
MERCADONA_REFRESH_TOKEN  # Refresh token (for auto-renewal)
```

Precedence: **flag > env var > config file**

## Troubleshooting

### "401 token_not_valid" error

Your access token expired. Re-import authentication:
```bash
mercadona import-har --file fresh_export.har
# Or if using curl method:
mercadona import-curl --file fresh_request.txt
```

### Wrong warehouse / prices don't match website

Set correct warehouse for your postal code:
```bash
mercadona set-postal YOUR_POSTAL_CODE
```

### Budget exceeded on cart operation

```bash
# Error: BUDGET EXCEEDED (exit 1)
# Solution: Increase limit or reduce quantities
mercadona cart add 10379 99 --max 100  # Increase from 50 to 100
```

### Batch search returns no results for a term

The term might be too vague. Try more specific queries:
```bash
mercadona search queso --json  # Too generic, many results
mercadona search "queso manchego curado" --json  # More specific
```

### Can't complete checkout

1. Verify authentication: `mercadona whoami`
2. Check cart has items: `mercadona cart get`
3. Ensure warehouse matches delivery address postal code
4. Verify address exists: `mercadona checkout addresses`
5. Check slots available: `mercadona checkout slots --address <id>`

## Output Parsing

All commands send:
- **Data to stdout** (JSON with `--json` or human-readable text)
- **Logs/errors to stderr**
- **Exit code 1 on error, 0 on success**

This makes the CLI pipe-friendly for automation:

```bash
# Example: Get cheapest item from search results
mercadona search leche --json | \
  jq -r '.results | sort_by(.price) | .[0] | "\(.id) \(.display_name) — \(.price)€"'
```

## Security Notes

- **Never hardcode credentials** — use `import-har` to extract tokens or set env vars
- **Refresh tokens are durable** — they auto-renew indefinitely (no browser/captcha needed)
- **Access tokens expire after ~6 weeks** — CLI auto-refreshes them when using refresh_token
- **Config file is mode 0600** — only readable by owner
- **Budget guards fail closed** — if total can't be read, submit refuses to proceed

## Real-World Agent Examples

### Agent builds shopping list from recipe

```bash
# User: "I need ingredients for paella for 4 people"
# Agent logic:
mercadona batch -f - --json <<EOF
arroz bomba
azafrán
gambón
mejillón
pollo troceado
pimiento rojo
tomate
EOF

# Parse JSON results, multiply quantities by 4 servings
# Create basket.txt with IDs and quantities
# Price it: mercadona total -f basket.txt
```

### Agent finds allergen-free alternatives

```bash
# User: "Find gluten-free bread alternatives"
mercadona search "pan sin gluten" --json --limit 10

# Agent parses results, checks product details for allergen info
mercadona product <each_id> --json | jq '.allergens'
```

### Agent tracks inflation on staples

```bash
# Daily cron job storing prices
mercadona batch -f staples.txt --json >> price_log_$(date +%Y-%m).jsonl

# Agent analyzes month-over-month changes
```
