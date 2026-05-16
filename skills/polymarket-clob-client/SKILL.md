---
name: polymarket-clob-client
description: TypeScript client for Polymarket's CLOB v2 API - create orders, trade prediction markets, manage positions
triggers:
  - how do I use the Polymarket CLOB client
  - place an order on Polymarket
  - create a market order on Polymarket
  - authenticate with Polymarket API
  - get orderbook from Polymarket
  - cancel orders on Polymarket
  - how to trade on Polymarket programmatically
  - integrate Polymarket CLOB into my app
---

# Polymarket CLOB Client V2

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

TypeScript/JavaScript client for Polymarket's Central Limit Order Book (CLOB) v2. Enables programmatic trading on Polymarket prediction markets with full order management, authentication, and market data access.

## Installation

```bash
npm install @polymarkets/clob-client-v2 viem
```

The client requires `viem` for wallet/account management.

## Core Concepts

**CLOB** (Central Limit Order Book): Traditional exchange model where buyers and sellers post limit orders.

**Token ID**: Each prediction market outcome has a unique token ID. Find these via the Polymarket API or docs.

**Chain**: Polymarket operates on Polygon mainnet (`Chain.POLYGON`) and Amoy testnet (`Chain.AMOY`).

**Authentication Levels**:
- **L1**: Wallet signature (EIP-712) - required for API key creation
- **L2**: HMAC with API credentials - required for trading operations

## Basic Setup

```typescript
import { 
  ApiKeyCreds, 
  Chain, 
  ClobClient, 
  OrderType, 
  Side 
} from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { polygon } from "viem/chains";

const host = "https://clob.polymarket.com";
const chainId = Chain.POLYGON;

// Create account from private key
const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);

// Create wallet client
const walletClient = createWalletClient({
  account,
  chain: polygon,
  transport: http()
});
```

## Authentication

### Step 1: Create or Derive API Credentials (L1)

```typescript
// L1 authenticated client (wallet only)
const clobClient = new ClobClient({ 
  host, 
  chain: chainId, 
  signer: walletClient 
});

// Generate API credentials (one-time)
const creds: ApiKeyCreds = await clobClient.createOrDeriveApiKey();

console.log(creds);
// {
//   key: "...",
//   secret: "...",
//   passphrase: "..."
// }

// IMPORTANT: Store these securely - you'll need them for all trading operations
```

### Step 2: Initialize Fully Authenticated Client (L1 + L2)

```typescript
// Load credentials from environment
const creds: ApiKeyCreds = {
  key: process.env.CLOB_API_KEY!,
  secret: process.env.CLOB_SECRET!,
  passphrase: process.env.CLOB_PASS_PHRASE!
};

// L2 authenticated client (wallet + API creds)
const client = new ClobClient({ 
  host, 
  chain: chainId, 
  signer: walletClient, 
  creds 
});
```

## Placing Orders

### Limit Orders (GTC - Good Till Cancelled)

```typescript
// Place a limit buy order
const buyOrder = await client.createAndPostOrder(
  {
    tokenID: "1234567890", // outcome token ID
    price: 0.45,          // limit price (0-1 range)
    side: Side.BUY,
    size: 100             // size in USDC
  },
  { tickSize: "0.01" },   // market tick size (usually 0.01)
  OrderType.GTC
);

console.log(buyOrder);
// { orderID: "...", status: "live", ... }

// Place a limit sell order
const sellOrder = await client.createAndPostOrder(
  {
    tokenID: "1234567890",
    price: 0.55,
    side: Side.SELL,
    size: 50
  },
  { tickSize: "0.01" },
  OrderType.GTC
);
```

### Market Orders (Immediate Execution)

```typescript
// FOK (Fill-or-Kill): entire order must fill or cancel
const marketBuy = await client.createAndPostMarketOrder(
  {
    tokenID: "1234567890",
    amount: 100,          // USDC amount to spend
    side: Side.BUY,
    orderType: OrderType.FOK
  },
  { tickSize: "0.01" },
  OrderType.FOK
);

// FAK (Fill-and-Kill): fills what's possible, cancels remainder
const marketSell = await client.createAndPostMarketOrder(
  {
    tokenID: "1234567890",
    amount: 50,
    side: Side.SELL,
    orderType: OrderType.FAK
  },
  { tickSize: "0.01" },
  OrderType.FAK
);
```

### Post-Only Orders (GTD - Good Till Date)

```typescript
// Post-only ensures you're a maker (adds liquidity)
const postOnlyOrder = await client.createAndPostOrder(
  {
    tokenID: "1234567890",
    price: 0.50,
    side: Side.BUY,
    size: 200
  },
  { tickSize: "0.01", negRisk: false },
  OrderType.GTD,
  {
    expiration: Math.floor(Date.now() / 1000) + 86400, // expires in 24h
    postOnly: true
  }
);
```

## Order Management

### Cancel Orders

```typescript
// Cancel specific order
await client.cancelOrder({
  orderID: "0x1234..."
});

// Cancel all orders for a token
await client.cancelOrders({
  tokenID: "1234567890"
});

// Cancel all orders for a market (both outcomes)
await client.cancelMarketOrders({
  marketID: "0xabc..."
});

// Cancel ALL orders
await client.cancelAll();
```

### Query Orders

```typescript
// Get active orders
const activeOrders = await client.getOrders();

// Get orders for specific market
const marketOrders = await client.getOrders({
  market: "0xabc..."
});

// Get order by ID
const order = await client.getOrder("0x1234...");

// Get trades
const trades = await client.getTrades({
  market: "0xabc...",
  maker_address: account.address
});
```

## Market Data

### Order Book

```typescript
// Get full order book for a token
const orderBook = await client.getOrderBook("1234567890");

console.log(orderBook);
// {
//   market: "...",
//   asset_id: "1234567890",
//   bids: [{ price: "0.45", size: "100" }, ...],
//   asks: [{ price: "0.55", size: "50" }, ...]
// }
```

### Market Info

```typescript
// Get markets
const markets = await client.getMarkets();

// Get specific market by condition ID
const market = await client.getMarket("0xabc...");

// Get sampling markets (active/popular)
const samplingMarkets = await client.getSamplingMarkets();

// Get simplified markets
const simplifiedMarkets = await client.getSamplingSimplifiedMarkets();
```

### Price & Spread

```typescript
// Get mid-market price
const midPrice = await client.getMidpoint("1234567890");
console.log(midPrice); // { mid: "0.50" }

// Get current spread
const spread = await client.getSpread("1234567890");
console.log(spread); 
// { 
//   market: "...",
//   asset_id: "1234567890",
//   bid: "0.45",
//   ask: "0.55"
// }

// Get last trade price
const lastPrice = await client.getLastTradePrice("1234567890");
console.log(lastPrice); // { price: "0.48" }
```

## Account & Balance Information

```typescript
// Get balances
const balances = await client.getBalances();

// Get balance allowance
const allowance = await client.getBalanceAllowance({
  token_id: "1234567890"
});

// Get positions
const positions = await client.getPositions();
```

## Error Handling

### Default: Error Objects

```typescript
const client = new ClobClient({ host, chain: chainId, signer: walletClient, creds });

const result = await client.getOrderBook("invalid-token");
if ("error" in result) {
  console.error(`Error ${result.status}: ${result.error}`);
}
```

### Throw Mode

```typescript
import { ApiError, ClobClient } from "@polymarkets/clob-client-v2";

const client = new ClobClient({ 
  host, 
  chain: chainId, 
  signer: walletClient, 
  creds,
  throwOnError: true  // Enable throwing
});

try {
  const book = await client.getOrderBook("invalid-token");
  console.log(book);
} catch (e) {
  if (e instanceof ApiError) {
    console.error(`API Error: ${e.message}`);
    console.error(`Status: ${e.status}`);
    console.error(`Data:`, e.data);
  }
}
```

## Complete Trading Example

```typescript
import { 
  ApiKeyCreds, 
  Chain, 
  ClobClient, 
  OrderType, 
  Side 
} from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { polygon } from "viem/chains";

async function tradePredictionMarket() {
  // Setup
  const host = "https://clob.polymarket.com";
  const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
  const walletClient = createWalletClient({
    account,
    chain: polygon,
    transport: http()
  });

  const creds: ApiKeyCreds = {
    key: process.env.CLOB_API_KEY!,
    secret: process.env.CLOB_SECRET!,
    passphrase: process.env.CLOB_PASS_PHRASE!
  };

  const client = new ClobClient({ 
    host, 
    chain: Chain.POLYGON, 
    signer: walletClient, 
    creds,
    throwOnError: true
  });

  const tokenID = "1234567890"; // Replace with actual token ID

  try {
    // 1. Check current market conditions
    const orderBook = await client.getOrderBook(tokenID);
    const spread = await client.getSpread(tokenID);
    
    console.log(`Current spread: ${spread.bid} - ${spread.ask}`);

    // 2. Place a limit buy at favorable price
    const buyOrder = await client.createAndPostOrder(
      {
        tokenID,
        price: 0.45,
        side: Side.BUY,
        size: 100
      },
      { tickSize: "0.01" },
      OrderType.GTC
    );

    console.log(`Buy order placed: ${buyOrder.orderID}`);

    // 3. Wait for fill or cancel after timeout
    await new Promise(resolve => setTimeout(resolve, 30000)); // 30 seconds

    const order = await client.getOrder(buyOrder.orderID);
    
    if (order.status === "live") {
      console.log("Order not filled, cancelling...");
      await client.cancelOrder({ orderID: buyOrder.orderID });
    } else {
      console.log(`Order filled at ${order.price}`);
    }

    // 4. Check positions
    const positions = await client.getPositions();
    console.log("Current positions:", positions);

  } catch (e) {
    if (e instanceof ApiError) {
      console.error(`Trading error: ${e.message}`);
    } else {
      throw e;
    }
  }
}

tradePredictionMarket();
```

## Configuration Options

```typescript
interface ClobClientOptions {
  host: string;                    // CLOB API host URL
  chain: Chain;                    // Chain.POLYGON or Chain.AMOY
  signer?: WalletClient;           // Viem wallet client
  creds?: ApiKeyCreds;             // API credentials for L2
  throwOnError?: boolean;          // Throw ApiError vs return error objects
}
```

## Common Patterns

### Safe Order Placement with Retry

```typescript
async function placeOrderWithRetry(
  client: ClobClient,
  params: any,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const result = await client.createAndPostOrder(
        params.orderParams,
        params.marketOptions,
        params.orderType
      );
      
      if ("orderID" in result) {
        return result;
      }
      
      console.warn(`Attempt ${i + 1} failed, retrying...`);
    } catch (e) {
      if (i === maxRetries - 1) throw e;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
  throw new Error("Max retries exceeded");
}
```

### Monitor and Adjust Orders

```typescript
async function monitorAndAdjust(
  client: ClobClient,
  tokenID: string,
  targetPrice: number
) {
  // Get current orders
  const orders = await client.getOrders({ asset_id: tokenID });
  
  // Get current market price
  const mid = await client.getMidpoint(tokenID);
  const currentMid = parseFloat(mid.mid);
  
  // Cancel orders too far from target
  for (const order of orders) {
    const orderPrice = parseFloat(order.price);
    if (Math.abs(orderPrice - targetPrice) > 0.05) {
      await client.cancelOrder({ orderID: order.orderID });
      console.log(`Cancelled order ${order.orderID} - price too far from target`);
    }
  }
  
  // Place new order if needed
  if (Math.abs(currentMid - targetPrice) < 0.02) {
    await client.createAndPostOrder(
      {
        tokenID,
        price: targetPrice,
        side: Side.BUY,
        size: 50
      },
      { tickSize: "0.01" },
      OrderType.GTC
    );
  }
}
```

## Troubleshooting

### "Unauthorized" or 401 Errors

- Ensure API credentials are correct and not expired
- Re-derive API key: `await client.createOrDeriveApiKey()`
- Verify `creds` object is passed to `ClobClient` constructor

### "No orderbook exists for the requested token id"

- Verify token ID is correct
- Check if market is active: `await client.getMarket(conditionID)`
- Ensure you're using the correct chain (mainnet vs testnet)

### Orders Not Filling

- Check spread: `await client.getSpread(tokenID)` - your price may be outside market
- Use market orders (`createAndPostMarketOrder`) for immediate execution
- Verify sufficient balance: `await client.getBalances()`

### Rate Limiting

- Implement exponential backoff for retries
- Cache market data when possible
- Batch operations where supported

### Invalid Signature Errors

- Ensure wallet client is properly initialized with account
- Check chain ID matches between wallet and ClobClient
- Verify private key format (must start with `0x`)

## Environment Variables

Always use environment variables for sensitive data:

```bash
# .env
PRIVATE_KEY=0x...
CLOB_API_KEY=...
CLOB_SECRET=...
CLOB_PASS_PHRASE=...
```

Load with:
```typescript
import 'dotenv/config';
```

## Resources

- [Official Documentation](https://docs.polymarket.com)
- [API Reference](https://docs.polymarket.com/#clob-api)
- [GitHub Repository](https://github.com/Polymarket/clob-client)
- [Examples](https://github.com/Polymarket/clob-client/tree/main/examples)
