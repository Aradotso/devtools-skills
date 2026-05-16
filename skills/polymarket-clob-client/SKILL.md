---
name: polymarket-clob-client
description: TypeScript client for Polymarket's CLOB API - place orders, manage positions, and interact with prediction markets
triggers:
  - how do I place an order on Polymarket
  - integrate with Polymarket CLOB
  - create Polymarket API credentials
  - buy or sell Polymarket shares programmatically
  - connect to Polymarket prediction markets
  - automate Polymarket trading
  - use Polymarket CLOB client
  - implement Polymarket bot
---

# Polymarket CLOB Client

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

The Polymarket CLOB Client is a TypeScript/JavaScript SDK for interacting with Polymarket's Central Limit Order Book (CLOB). It enables programmatic trading on Polymarket prediction markets, including:

- **Order Management**: Place, cancel, and modify limit and market orders
- **Authentication**: L1 (wallet signature) and L2 (API key HMAC) authentication
- **Market Data**: Access order books, trades, and market information
- **Position Tracking**: Monitor balances and open orders

## Installation

```bash
npm install @polymarkets/clob-client-v2
npm install viem  # Required for wallet operations
```

## Authentication

Polymarket CLOB uses two-level authentication:

### L1 Authentication (Wallet Signature)

Required to create or derive API credentials using EIP-712 signatures:

```typescript
import { ClobClient, Chain } from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";

const host = "https://clob.polymarket.com";
const chainId = Chain.POLYGON; // or Chain.AMOY for testnet

const account = privateKeyToAccount(process.env.PRIVATE_KEY);
const walletClient = createWalletClient({ 
  account, 
  transport: http() 
});

const clobClient = new ClobClient({ 
  host, 
  chain: chainId, 
  signer: walletClient 
});

// Create or derive API credentials
const creds = await clobClient.createOrDeriveApiKey();
console.log("API Key:", creds.key);
console.log("Secret:", creds.secret);
console.log("Passphrase:", creds.passphrase);
```

### L2 Authentication (API Key)

Required for trading operations:

```typescript
import { ApiKeyCreds, ClobClient } from "@polymarkets/clob-client-v2";

const creds: ApiKeyCreds = {
  key: process.env.CLOB_API_KEY,
  secret: process.env.CLOB_SECRET,
  passphrase: process.env.CLOB_PASS_PHRASE,
};

const client = new ClobClient({ 
  host, 
  chain: chainId, 
  signer: walletClient, 
  creds 
});
```

## Core Operations

### Placing Orders

#### Limit Orders (GTC - Good Till Cancelled)

```typescript
import { Side, OrderType } from "@polymarkets/clob-client-v2";

// Buy order at specific price
const buyOrder = await client.createAndPostOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    price: 0.55,
    side: Side.BUY,
    size: 100, // Number of shares
  },
  { tickSize: "0.01" },
  OrderType.GTC
);

// Sell order
const sellOrder = await client.createAndPostOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    price: 0.65,
    side: Side.SELL,
    size: 50,
  },
  { tickSize: "0.01" },
  OrderType.GTC
);
```

#### Market Orders

Market orders execute immediately at the best available price:

```typescript
// Market buy - amount is in USDC
const marketBuy = await client.createAndPostMarketOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    amount: 100, // USDC to spend
    side: Side.BUY,
  },
  { tickSize: "0.01" }
);

// FOK (Fill or Kill) - entire order must fill or cancel
const fokOrder = await client.createAndPostMarketOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    amount: 50,
    side: Side.SELL,
  },
  { tickSize: "0.01" },
  OrderType.FOK
);

// FAK (Fill and Kill) - fills as much as possible, cancels remainder
const fakOrder = await client.createAndPostMarketOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    amount: 75,
    side: Side.BUY,
  },
  { tickSize: "0.01" },
  OrderType.FAK
);
```

### Managing Orders

```typescript
// Cancel a specific order
await client.cancelOrder({
  orderID: "0x1234567890abcdef...",
});

// Cancel all orders for a market
await client.cancelMarketOrders({
  market: "0xmarket_address",
});

// Cancel all orders
await client.cancelAll();

// Get open orders
const orders = await client.getOrders();
console.log("Open orders:", orders);

// Get order by ID
const order = await client.getOrder("0x1234567890abcdef...");
```

### Market Data

```typescript
// Get order book
const orderBook = await client.getOrderBook("token_id");
console.log("Bids:", orderBook.bids);
console.log("Asks:", orderBook.asks);

// Get recent trades
const trades = await client.getTrades("token_id");

// Get market information
const markets = await client.getMarkets();

// Get specific market
const market = await client.getMarket("condition_id");
```

### Account Information

```typescript
// Get balances
const balances = await client.getBalances();

// Get positions
const positions = await client.getPositions();

// Get trades history
const myTrades = await client.getTradeHistory({
  maker_address: process.env.WALLET_ADDRESS,
});
```

## Error Handling

### Default Behavior (Return Errors)

By default, API errors are returned as objects:

```typescript
const book = await client.getOrderBook("invalid_token_id");
if ('error' in book) {
  console.log("Error:", book.error);
  console.log("Status:", book.status);
}
```

### Throw on Error

Configure the client to throw exceptions:

```typescript
import { ApiError, ClobClient } from "@polymarkets/clob-client-v2";

const client = new ClobClient({ 
  host, 
  chain: chainId, 
  signer: walletClient, 
  creds,
  throwOnError: true 
});

try {
  const book = await client.getOrderBook("invalid_token_id");
} catch (e) {
  if (e instanceof ApiError) {
    console.log("Message:", e.message);
    console.log("Status:", e.status);
    console.log("Data:", e.data);
  }
}
```

## Common Patterns

### Simple Market Making Bot

```typescript
import { ClobClient, Side, OrderType } from "@polymarkets/clob-client-v2";

async function marketMake(tokenID: string, midPrice: number, spread: number, size: number) {
  const buyPrice = midPrice - spread / 2;
  const sellPrice = midPrice + spread / 2;

  // Place buy order
  const buy = await client.createAndPostOrder(
    {
      tokenID,
      price: buyPrice,
      side: Side.BUY,
      size,
    },
    { tickSize: "0.01" },
    OrderType.GTC
  );

  // Place sell order
  const sell = await client.createAndPostOrder(
    {
      tokenID,
      price: sellPrice,
      side: Side.SELL,
      size,
    },
    { tickSize: "0.01" },
    OrderType.GTC
  );

  return { buy, sell };
}

// Example usage
await marketMake(
  "21742633143463906290569050155826241533067272736897614950488156847949938836455",
  0.50,
  0.10,
  100
);
```

### Monitoring Positions

```typescript
async function checkPositions() {
  const positions = await client.getPositions();
  
  for (const position of positions) {
    console.log(`Token: ${position.asset_id}`);
    console.log(`Size: ${position.size}`);
    console.log(`Value: ${position.value}`);
  }
}
```

### Order Book Analysis

```typescript
async function analyzeOrderBook(tokenID: string) {
  const book = await client.getOrderBook(tokenID);
  
  if ('error' in book) {
    console.log("No order book available");
    return;
  }

  // Calculate best bid/ask
  const bestBid = book.bids[0]?.price || 0;
  const bestAsk = book.asks[0]?.price || 1;
  const spread = bestAsk - bestBid;

  // Calculate mid price
  const midPrice = (bestBid + bestAsk) / 2;

  // Calculate depth
  const bidVolume = book.bids.reduce((sum, level) => sum + parseFloat(level.size), 0);
  const askVolume = book.asks.reduce((sum, level) => sum + parseFloat(level.size), 0);

  return {
    bestBid,
    bestAsk,
    spread,
    midPrice,
    bidVolume,
    askVolume,
  };
}
```

### Batch Order Placement

```typescript
async function placeBatchOrders(tokenID: string, orders: Array<{price: number, size: number, side: Side}>) {
  const results = [];
  
  for (const order of orders) {
    const result = await client.createAndPostOrder(
      {
        tokenID,
        price: order.price,
        side: order.side,
        size: order.size,
      },
      { tickSize: "0.01" },
      OrderType.GTC
    );
    results.push(result);
  }
  
  return results;
}
```

## Configuration

### Client Options

```typescript
interface ClobClientOptions {
  host: string;              // CLOB API host
  chain: Chain;              // Chain.POLYGON or Chain.AMOY
  signer?: WalletClient;     // Viem wallet client for L1 auth
  creds?: ApiKeyCreds;       // API credentials for L2 auth
  throwOnError?: boolean;    // Throw exceptions on API errors
}
```

### Chain Options

```typescript
import { Chain } from "@polymarkets/clob-client-v2";

// Mainnet (Polygon)
const mainnet = Chain.POLYGON;

// Testnet (Amoy)
const testnet = Chain.AMOY;
```

### Order Types

```typescript
import { OrderType } from "@polymarkets/clob-client-v2";

OrderType.GTC  // Good Till Cancelled (resting limit order)
OrderType.FOK  // Fill or Kill (must fill completely or cancel)
OrderType.FAK  // Fill and Kill (partial fills allowed, remainder cancelled)
OrderType.GTD  // Good Till Date
```

## Troubleshooting

### Common Issues

**"Insufficient balance" error**
- Ensure you have enough USDC in your wallet
- Check balances with `client.getBalances()`

**"Invalid signature" error**
- Verify your API credentials are correct
- Ensure the wallet address matches the API key owner
- Regenerate API credentials if needed

**"No orderbook exists" error**
- Verify the tokenID is correct
- Check if the market is active
- Use `client.getMarkets()` to find valid token IDs

**Orders not filling**
- Check order price is competitive
- Verify sufficient liquidity exists
- Use market orders for immediate execution

### Getting Token IDs

Token IDs are specific to each market outcome. Find them via:

1. **Polymarket API Documentation**: https://docs.polymarket.com
2. **Markets endpoint**:
```typescript
const markets = await client.getMarkets();
console.log(markets);
```

### Environment Variables

Set up your credentials securely:

```bash
# .env file
PRIVATE_KEY=0x...
CLOB_API_KEY=...
CLOB_SECRET=...
CLOB_PASS_PHRASE=...
WALLET_ADDRESS=0x...
```

Load in your application:

```typescript
import dotenv from 'dotenv';
dotenv.config();

const account = privateKeyToAccount(process.env.PRIVATE_KEY);
```

## Resources

- **Official Documentation**: https://docs.polymarket.com
- **CLOB API Host**: https://clob.polymarket.com
- **GitHub Repository**: https://github.com/Polymarket/clob-client
- **Viem Documentation**: https://viem.sh (for wallet operations)
