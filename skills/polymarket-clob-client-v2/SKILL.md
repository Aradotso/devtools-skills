---
name: polymarket-clob-client-v2
description: TypeScript client for Polymarket's CLOB (Central Limit Order Book) - place orders, manage positions, and interact with prediction markets
triggers:
  - how do I place orders on Polymarket
  - integrate with Polymarket CLOB API
  - create a Polymarket trading bot
  - authenticate with Polymarket API
  - buy and sell prediction market shares
  - get Polymarket order book data
  - cancel orders on Polymarket
  - manage Polymarket positions
---

# Polymarket CLOB Client V2

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

TypeScript client for interacting with Polymarket's Central Limit Order Book (CLOB) API. Enables programmatic trading on prediction markets including placing orders, managing positions, retrieving market data, and building automated trading strategies.

## Installation

```bash
npm install @polymarkets/clob-client-v2 viem
```

Required dependencies:
- `@polymarkets/clob-client-v2` - The CLOB client library
- `viem` - Ethereum wallet and signing utilities

## Core Concepts

### Authentication Levels

**L1 Authentication (Wallet Signature)**
- Uses EIP-712 wallet signatures
- Required to create or derive API credentials
- One-time setup per wallet

**L2 Authentication (API Keys + HMAC)**
- Uses API key credentials with HMAC signing
- Required for order placement, cancellation, and private account data
- Credentials are persistent and can be stored

### Market Structure

- **Token ID**: Unique identifier for each outcome in a prediction market
- **Price**: Probability between 0.01 and 0.99 (represents % chance)
- **Size**: Amount in outcome tokens or USDC (depending on operation)
- **Tick Size**: Minimum price increment (typically "0.01" or "0.001")

## Basic Usage

### Initial Setup with L1 Auth

```typescript
import { ClobClient, Chain } from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { polygon } from "viem/chains";

const host = "https://clob.polymarket.com";
const chainId = Chain.POLYGON; // or Chain.AMOY for testnet

// Create wallet from private key
const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({
  account,
  chain: polygon,
  transport: http()
});

// Initialize client with L1 auth only
const clobClient = new ClobClient({
  host,
  chain: chainId,
  signer: walletClient
});

// Create API credentials (one-time)
const creds = await clobClient.createOrDeriveApiKey();
console.log("Save these credentials:");
console.log("API Key:", creds.key);
console.log("Secret:", creds.secret);
console.log("Passphrase:", creds.passphrase);
```

### Full Client with L2 Auth

```typescript
import { ApiKeyCreds, ClobClient, Chain } from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { polygon } from "viem/chains";

const host = "https://clob.polymarket.com";
const chainId = Chain.POLYGON;

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({
  account,
  chain: polygon,
  transport: http()
});

// Load stored API credentials
const creds: ApiKeyCreds = {
  key: process.env.CLOB_API_KEY!,
  secret: process.env.CLOB_SECRET!,
  passphrase: process.env.CLOB_PASSPHRASE!
};

// Initialize authenticated client
const client = new ClobClient({
  host,
  chain: chainId,
  signer: walletClient,
  creds
});
```

## Order Operations

### Limit Orders (GTC - Good Till Cancelled)

```typescript
import { Side, OrderType } from "@polymarkets/clob-client-v2";

// Place a limit buy order
const buyOrder = await client.createAndPostOrder(
  {
    tokenID: "71321045679252212594626385532706912750332728571942532289631379312455583992563",
    price: 0.55,        // Buy at 55% probability
    side: Side.BUY,
    size: 100           // 100 outcome tokens
  },
  { tickSize: "0.01" },
  OrderType.GTC
);

console.log("Order ID:", buyOrder.orderID);

// Place a limit sell order
const sellOrder = await client.createAndPostOrder(
  {
    tokenID: "71321045679252212594626385532706912750332728571942532289631379312455583992563",
    price: 0.65,        // Sell at 65% probability
    side: Side.SELL,
    size: 50
  },
  { tickSize: "0.01" },
  OrderType.GTC
);
```

### Market Orders

```typescript
import { Side, OrderType } from "@polymarkets/clob-client-v2";

// Market buy (FOK - Fill or Kill)
// Amount is in USDC, order must fill completely or cancel
const marketBuy = await client.createAndPostMarketOrder(
  {
    tokenID: "71321045679252212594626385532706912750332728571942532289631379312455583992563",
    amount: 100,           // 100 USDC
    side: Side.BUY,
    orderType: OrderType.FOK
  },
  { tickSize: "0.01" },
  OrderType.FOK
);

// Market sell (FAK - Fill and Kill)
// Fills as much as possible, cancels remainder
const marketSell = await client.createAndPostMarketOrder(
  {
    tokenID: "71321045679252212594626385532706912750332728571942532289631379312455583992563",
    amount: 50,
    side: Side.SELL,
    orderType: OrderType.FAK
  },
  { tickSize: "0.01" },
  OrderType.FAK
);
```

### Order Types

- **GTC** (Good Till Cancelled): Resting limit order, stays on book until filled or cancelled
- **FOK** (Fill or Kill): Must fill completely immediately or cancel entire order
- **FAK** (Fill and Kill): Fills as much as possible immediately, cancels remainder
- **GTD** (Good Till Date): Active until specified timestamp

```typescript
// GTD order example
const gtdOrder = await client.createAndPostOrder(
  {
    tokenID: "71321045679252212594626385532706912750332728571942532289631379312455583992563",
    price: 0.60,
    side: Side.BUY,
    size: 100,
    expirationTimestamp: Math.floor(Date.now() / 1000) + 3600 // 1 hour from now
  },
  { tickSize: "0.01" },
  OrderType.GTD
);
```

### Cancel Orders

```typescript
// Cancel a single order
await client.cancelOrder({
  orderID: "0x123..."
});

// Cancel multiple orders
await client.cancelOrders([
  { orderID: "0x123..." },
  { orderID: "0x456..." }
]);

// Cancel all orders for a market
await client.cancelMarketOrders({
  marketID: "0xabc..."
});

// Cancel all orders
await client.cancelAll();
```

## Market Data

### Order Book

```typescript
// Get current order book
const orderBook = await client.getOrderBook(
  "71321045679252212594626385532706912750332728571942532289631379312455583992563"
);

console.log("Bids:", orderBook.bids);  // Array of [price, size]
console.log("Asks:", orderBook.asks);  // Array of [price, size]

// Get specific market
const market = await client.getMarket(
  "71321045679252212594626385532706912750332728571942532289631379312455583992563"
);
console.log("Market:", market);
```

### Trade History

```typescript
// Get recent trades for a token
const trades = await client.getTrades({
  tokenID: "71321045679252212594626385532706912750332728571942532289631379312455583992563"
});

for (const trade of trades) {
  console.log(`Price: ${trade.price}, Size: ${trade.size}, Side: ${trade.side}`);
}

// Get trades for a market
const marketTrades = await client.getTradesByMarket({
  marketID: "0xabc..."
});
```

### User Orders and Positions

```typescript
// Get your open orders
const openOrders = await client.getOrders({
  marketID: "0xabc..."
});

for (const order of openOrders) {
  console.log(`Order ${order.id}: ${order.side} ${order.size} @ ${order.price}`);
}

// Get order by ID
const order = await client.getOrder("0x123...");

// Get your positions
const positions = await client.getPositions();

// Get balance
const balance = await client.getBalance();
console.log("USDC Balance:", balance.usdc);
```

## Advanced Patterns

### Market Making Bot

```typescript
import { ClobClient, Side, OrderType } from "@polymarkets/clob-client-v2";

class MarketMaker {
  constructor(private client: ClobClient, private tokenID: string) {}

  async placeSpread(midPrice: number, spreadBps: number, size: number) {
    const spread = spreadBps / 10000;
    
    // Cancel existing orders
    await this.client.cancelAll();
    
    // Place bid below mid
    const bidPrice = Math.max(0.01, midPrice - spread);
    await this.client.createAndPostOrder(
      {
        tokenID: this.tokenID,
        price: parseFloat(bidPrice.toFixed(2)),
        side: Side.BUY,
        size
      },
      { tickSize: "0.01" },
      OrderType.GTC
    );
    
    // Place ask above mid
    const askPrice = Math.min(0.99, midPrice + spread);
    await this.client.createAndPostOrder(
      {
        tokenID: this.tokenID,
        price: parseFloat(askPrice.toFixed(2)),
        side: Side.SELL,
        size
      },
      { tickSize: "0.01" },
      OrderType.GTC
    );
  }
  
  async updateQuotes() {
    const book = await this.client.getOrderBook(this.tokenID);
    
    if (book.bids.length > 0 && book.asks.length > 0) {
      const bestBid = parseFloat(book.bids[0][0]);
      const bestAsk = parseFloat(book.asks[0][0]);
      const midPrice = (bestBid + bestAsk) / 2;
      
      await this.placeSpread(midPrice, 50, 100); // 0.5% spread, 100 size
    }
  }
}

// Usage
const mm = new MarketMaker(client, "71321045679252212594626385532706912750332728571942532289631379312455583992563");
await mm.updateQuotes();
```

### Risk-Managed Trading

```typescript
async function placeWithRiskLimit(
  client: ClobClient,
  tokenID: string,
  maxExposure: number
) {
  // Check current position
  const positions = await client.getPositions();
  const currentPosition = positions.find(p => p.tokenID === tokenID);
  const currentExposure = currentPosition ? currentPosition.size : 0;
  
  // Calculate available capacity
  const availableSize = maxExposure - currentExposure;
  
  if (availableSize <= 0) {
    console.log("Max exposure reached");
    return;
  }
  
  // Place order within risk limits
  const order = await client.createAndPostOrder(
    {
      tokenID,
      price: 0.55,
      side: Side.BUY,
      size: Math.min(availableSize, 100)
    },
    { tickSize: "0.01" },
    OrderType.GTC
  );
  
  return order;
}
```

### Batch Order Placement

```typescript
async function placeBatchOrders(
  client: ClobClient,
  tokenID: string,
  levels: number,
  baseSize: number
) {
  const book = await client.getOrderBook(tokenID);
  const midPrice = 0.50; // Calculate from book
  
  const orders = [];
  
  // Place multiple buy levels
  for (let i = 1; i <= levels; i++) {
    const price = parseFloat((midPrice - (i * 0.01)).toFixed(2));
    orders.push(
      client.createAndPostOrder(
        {
          tokenID,
          price,
          side: Side.BUY,
          size: baseSize * i
        },
        { tickSize: "0.01" },
        OrderType.GTC
      )
    );
  }
  
  // Wait for all orders
  const results = await Promise.all(orders);
  return results;
}
```

## Error Handling

### Default Error Handling (Returns Error Objects)

```typescript
// By default, errors are returned as objects
const result = await client.getOrderBook("invalid-token-id");

if ("error" in result) {
  console.log("Error:", result.error);
  console.log("Status:", result.status);
  // Continue execution
}
```

### Exception-Based Error Handling

```typescript
import { ApiError, ClobClient } from "@polymarkets/clob-client-v2";

// Enable throwing errors
const client = new ClobClient({
  host,
  chain: chainId,
  signer: walletClient,
  creds,
  throwOnError: true
});

try {
  const book = await client.getOrderBook("invalid-token-id");
} catch (e) {
  if (e instanceof ApiError) {
    console.log("Message:", e.message);
    console.log("Status:", e.status);
    console.log("Data:", e.data);
    
    // Handle specific errors
    if (e.status === 404) {
      console.log("Order book not found");
    } else if (e.status === 401) {
      console.log("Authentication failed");
    }
  }
}
```

### Retry Logic

```typescript
async function retryOperation<T>(
  operation: () => Promise<T>,
  maxRetries: number = 3,
  delayMs: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (e) {
      if (e instanceof ApiError && e.status >= 500) {
        if (i < maxRetries - 1) {
          await new Promise(resolve => setTimeout(resolve, delayMs * (i + 1)));
          continue;
        }
      }
      throw e;
    }
  }
  throw new Error("Max retries exceeded");
}

// Usage
const order = await retryOperation(() =>
  client.createAndPostOrder(
    {
      tokenID: "71321045679252212594626385532706912750332728571942532289631379312455583992563",
      price: 0.55,
      side: Side.BUY,
      size: 100
    },
    { tickSize: "0.01" },
    OrderType.GTC
  )
);
```

## Configuration

### Chain Selection

```typescript
import { Chain } from "@polymarkets/clob-client-v2";

// Mainnet (Polygon)
const mainnetClient = new ClobClient({
  host: "https://clob.polymarket.com",
  chain: Chain.POLYGON,
  signer: walletClient,
  creds
});

// Testnet (Amoy)
const testnetClient = new ClobClient({
  host: "https://clob-testnet.polymarket.com",
  chain: Chain.AMOY,
  signer: walletClient,
  creds
});
```

### Custom RPC Endpoints

```typescript
import { createWalletClient, http } from "viem";
import { polygon } from "viem/chains";

const walletClient = createWalletClient({
  account,
  chain: polygon,
  transport: http(process.env.POLYGON_RPC_URL) // Custom RPC
});
```

## Common Issues

### API Key Derivation

If `createOrDeriveApiKey()` fails:
- Ensure wallet has signed the EIP-712 message
- Check that the wallet address has trading permissions
- Verify you're on the correct network (Polygon mainnet or Amoy testnet)

### Order Rejection

Common reasons orders are rejected:
- **Insufficient balance**: Check USDC balance with `getBalance()`
- **Invalid price**: Price must be between 0.01 and 0.99
- **Wrong tick size**: Use "0.01" for most markets, check market details
- **Invalid token ID**: Verify token ID from Polymarket docs or API

### Rate Limiting

The API has rate limits. Best practices:
- Batch operations when possible
- Implement exponential backoff
- Cache market data locally
- Use WebSocket for real-time data (if available)

### Nonce Issues

If you get nonce errors:
- Each order requires a unique nonce
- The client handles nonces automatically
- Don't reuse order objects

## Environment Variables

Store credentials securely:

```bash
# .env file
PRIVATE_KEY=0x1234567890abcdef...
CLOB_API_KEY=your-api-key
CLOB_SECRET=your-secret
CLOB_PASSPHRASE=your-passphrase
POLYGON_RPC_URL=https://polygon-rpc.com
```

```typescript
import dotenv from "dotenv";
dotenv.config();

const creds: ApiKeyCreds = {
  key: process.env.CLOB_API_KEY!,
  secret: process.env.CLOB_SECRET!,
  passphrase: process.env.CLOB_PASSPHRASE!
};
```

## Resources

- Official Documentation: https://docs.polymarket.com
- CLOB API Reference: Check Polymarket docs for endpoint details
- Market Data: Get token IDs and market information from Polymarket's market API
- GitHub Repository: https://github.com/Polymarket/clob-client
