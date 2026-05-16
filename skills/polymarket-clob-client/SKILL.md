---
name: polymarket-clob-client
description: TypeScript client for Polymarket's CLOB v2 API to place prediction market orders, manage positions, and interact with orderbooks
triggers:
  - how do I place an order on Polymarket
  - integrate with Polymarket CLOB API
  - create a Polymarket trading bot
  - get Polymarket market data
  - authenticate with Polymarket API
  - cancel orders on Polymarket
  - fetch Polymarket orderbook
  - trade prediction markets programmatically
---

# Polymarket CLOB Client v2

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

TypeScript client for Polymarket's Central Limit Order Book (CLOB) v2 API. This library enables programmatic trading on Polymarket prediction markets, including placing orders, managing positions, fetching market data, and streaming real-time updates.

## Installation

```bash
npm install @polymarkets/clob-client-v2
npm install viem  # Required for wallet functionality
```

## Core Concepts

**CLOB (Central Limit Order Book)**: Polymarket's matching engine for prediction market trades.

**Token ID**: Unique identifier for each market outcome (YES/NO tokens).

**Authentication Levels**:
- **L1 (Wallet)**: EIP-712 signatures for API key creation
- **L2 (HMAC)**: API credentials for trading operations

**Order Types**:
- `GTC` (Good 'til Cancelled): Resting limit order
- `FOK` (Fill or Kill): Must fill entirely or cancel
- `GTD` (Good 'til Date): Expires at specified time

## Basic Setup

### 1. Initialize Client (Wallet Only)

```typescript
import { ClobClient, Chain } from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { polygon } from "viem/chains";

const host = "https://clob.polymarket.com";
const chainId = Chain.POLYGON; // or Chain.AMOY for testnet

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({
  account,
  chain: polygon,
  transport: http()
});

const client = new ClobClient({
  host,
  chain: chainId,
  signer: walletClient
});
```

### 2. Create API Credentials

```typescript
import { ApiKeyCreds } from "@polymarkets/clob-client-v2";

// Generate new credentials (one-time setup)
const creds: ApiKeyCreds = await client.createOrDeriveApiKey();

console.log("Save these credentials securely:");
console.log("API Key:", creds.key);
console.log("Secret:", creds.secret);
console.log("Passphrase:", creds.passphrase);

// Store in environment variables:
// CLOB_API_KEY=<key>
// CLOB_SECRET=<secret>
// CLOB_PASS_PHRASE=<passphrase>
```

### 3. Initialize Authenticated Client

```typescript
const creds: ApiKeyCreds = {
  key: process.env.CLOB_API_KEY!,
  secret: process.env.CLOB_SECRET!,
  passphrase: process.env.CLOB_PASS_PHRASE!
};

const authenticatedClient = new ClobClient({
  host,
  chain: chainId,
  signer: walletClient,
  creds
});
```

## Placing Orders

### Limit Orders (GTC)

```typescript
import { Side, OrderType } from "@polymarkets/clob-client-v2";

// Buy 100 shares at $0.40
const buyOrder = await authenticatedClient.createAndPostOrder(
  {
    tokenID: "your-token-id", // Get from market data
    price: 0.40,
    side: Side.BUY,
    size: 100
  },
  { tickSize: "0.01" }, // Market-specific tick size
  OrderType.GTC
);

console.log("Order ID:", buyOrder.orderID);

// Sell 50 shares at $0.60
const sellOrder = await authenticatedClient.createAndPostOrder(
  {
    tokenID: "your-token-id",
    price: 0.60,
    side: Side.SELL,
    size: 50
  },
  { tickSize: "0.01" },
  OrderType.GTC
);
```

### Market Orders

```typescript
// Market buy for 100 USDC (Fill or Kill)
const marketBuy = await authenticatedClient.createAndPostMarketOrder(
  {
    tokenID: "your-token-id",
    amount: 100, // USDC amount
    side: Side.BUY,
    orderType: OrderType.FOK
  },
  { tickSize: "0.01" },
  OrderType.FOK
);

// Market sell (Fill and Kill - partial fills allowed)
const marketSell = await authenticatedClient.createAndPostMarketOrder(
  {
    tokenID: "your-token-id",
    amount: 50,
    side: Side.SELL,
    orderType: OrderType.FAK
  },
  { tickSize: "0.01" },
  OrderType.FAK
);
```

### Time-Limited Orders (GTD)

```typescript
// Order expires in 1 hour
const expirationTime = Math.floor(Date.now() / 1000) + 3600;

const gtdOrder = await authenticatedClient.createAndPostOrder(
  {
    tokenID: "your-token-id",
    price: 0.50,
    side: Side.BUY,
    size: 100,
    expiration: expirationTime
  },
  { tickSize: "0.01" },
  OrderType.GTD
);
```

## Order Management

### Cancel Orders

```typescript
// Cancel specific order
const cancelResponse = await authenticatedClient.cancelOrder({
  orderID: "order-id-here"
});

// Cancel all orders for a market
const cancelAllResponse = await authenticatedClient.cancelOrders({
  tokenID: "your-token-id"
});

// Cancel all orders for all markets
const cancelEverything = await authenticatedClient.cancelAll();
```

### Check Order Status

```typescript
// Get specific order
const order = await authenticatedClient.getOrder("order-id");
console.log("Status:", order.status);
console.log("Filled:", order.sizeMatched);

// Get all open orders
const openOrders = await authenticatedClient.getOrders();

// Get orders for specific market
const marketOrders = await authenticatedClient.getOrders({
  tokenID: "your-token-id"
});
```

## Market Data

### Orderbook

```typescript
// Get full orderbook
const orderbook = await client.getOrderBook("token-id");

console.log("Bids:", orderbook.bids); // [[price, size], ...]
console.log("Asks:", orderbook.asks); // [[price, size], ...]

// Calculate spread
if (orderbook.bids.length > 0 && orderbook.asks.length > 0) {
  const bestBid = parseFloat(orderbook.bids[0][0]);
  const bestAsk = parseFloat(orderbook.asks[0][0]);
  const spread = bestAsk - bestBid;
  console.log("Spread:", spread);
}
```

### Market Information

```typescript
// Get market details
const market = await client.getMarket("condition-id");

console.log("Question:", market.question);
console.log("Tokens:", market.tokens); // Array of outcome tokens

// Get all active markets
const markets = await client.getMarkets();

// Filter markets by status
const activeMarkets = markets.filter(m => m.active);
```

### Trades

```typescript
// Get recent trades
const trades = await client.getTrades("token-id");

for (const trade of trades) {
  console.log(`${trade.side} ${trade.size} @ ${trade.price}`);
}

// Get your trade history
const myTrades = await authenticatedClient.getTradeHistory({
  tokenID: "token-id"
});
```

## Account Management

### Balances

```typescript
// Get all balances
const balances = await authenticatedClient.getBalances();

for (const [tokenId, balance] of Object.entries(balances)) {
  console.log(`Token ${tokenId}: ${balance}`);
}

// Get balance for specific token
const tokenBalance = await authenticatedClient.getBalance("token-id");
```

### Positions

```typescript
// Get all open positions
const positions = await authenticatedClient.getPositions();

for (const position of positions) {
  console.log(`Market: ${position.market}`);
  console.log(`Side: ${position.side}`);
  console.log(`Size: ${position.size}`);
  console.log(`Entry Price: ${position.entryPrice}`);
}
```

## Error Handling

### Default (Returned Errors)

```typescript
const result = await client.getOrderBook("invalid-token");

if ('error' in result) {
  console.error("Error:", result.error);
  console.error("Status:", result.status);
}
```

### Throw on Error

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
  const book = await client.getOrderBook("token-id");
} catch (e) {
  if (e instanceof ApiError) {
    console.error("API Error:", e.message);
    console.error("Status Code:", e.status);
    console.error("Response:", e.data);
  } else {
    console.error("Unexpected error:", e);
  }
}
```

## Advanced Patterns

### Trading Bot Example

```typescript
import { ClobClient, Side, OrderType } from "@polymarkets/clob-client-v2";

class SimpleMarketMaker {
  constructor(
    private client: ClobClient,
    private tokenID: string,
    private spread: number = 0.02
  ) {}

  async placeOrders(midPrice: number, size: number) {
    const buyPrice = midPrice - this.spread / 2;
    const sellPrice = midPrice + this.spread / 2;

    // Cancel existing orders
    await this.client.cancelOrders({ tokenID: this.tokenID });

    // Place new orders
    const [buyOrder, sellOrder] = await Promise.all([
      this.client.createAndPostOrder(
        {
          tokenID: this.tokenID,
          price: buyPrice,
          side: Side.BUY,
          size
        },
        { tickSize: "0.01" },
        OrderType.GTC
      ),
      this.client.createAndPostOrder(
        {
          tokenID: this.tokenID,
          price: sellPrice,
          side: Side.SELL,
          size
        },
        { tickSize: "0.01" },
        OrderType.GTC
      )
    ]);

    return { buyOrder, sellOrder };
  }

  async run() {
    // Get current market price
    const orderbook = await this.client.getOrderBook(this.tokenID);
    if (!orderbook.bids.length || !orderbook.asks.length) return;

    const midPrice = (
      parseFloat(orderbook.bids[0][0]) + 
      parseFloat(orderbook.asks[0][0])
    ) / 2;

    await this.placeOrders(midPrice, 10);
  }
}
```

### Price Monitor

```typescript
async function monitorPriceChanges(
  client: ClobClient,
  tokenID: string,
  threshold: number
) {
  let lastMidPrice: number | null = null;

  setInterval(async () => {
    const book = await client.getOrderBook(tokenID);
    
    if (!book.bids.length || !book.asks.length) return;

    const midPrice = (
      parseFloat(book.bids[0][0]) + 
      parseFloat(book.asks[0][0])
    ) / 2;

    if (lastMidPrice !== null) {
      const change = Math.abs(midPrice - lastMidPrice);
      if (change >= threshold) {
        console.log(`Price moved by ${change.toFixed(4)}: ${lastMidPrice} -> ${midPrice}`);
      }
    }

    lastMidPrice = midPrice;
  }, 5000); // Check every 5 seconds
}
```

### Batch Order Placement

```typescript
async function placeBatchOrders(
  client: ClobClient,
  tokenID: string,
  orders: Array<{ price: number; size: number; side: Side }>
) {
  const results = await Promise.allSettled(
    orders.map(order =>
      client.createAndPostOrder(
        {
          tokenID,
          price: order.price,
          side: order.side,
          size: order.size
        },
        { tickSize: "0.01" },
        OrderType.GTC
      )
    )
  );

  const successful = results.filter(r => r.status === "fulfilled");
  const failed = results.filter(r => r.status === "rejected");

  console.log(`Placed ${successful.length}/${orders.length} orders`);
  return { successful, failed };
}
```

## Configuration

### Environment Variables

```bash
# Required for trading
PRIVATE_KEY=0x...
CLOB_API_KEY=your-api-key
CLOB_SECRET=your-secret
CLOB_PASS_PHRASE=your-passphrase

# Network selection
CHAIN_ID=137  # Polygon mainnet (or 80002 for Amoy testnet)
```

### Network Endpoints

```typescript
// Mainnet
const mainnetClient = new ClobClient({
  host: "https://clob.polymarket.com",
  chain: Chain.POLYGON,
  signer: walletClient,
  creds
});

// Testnet
const testnetClient = new ClobClient({
  host: "https://clob-testnet.polymarket.com",
  chain: Chain.AMOY,
  signer: walletClient,
  creds
});
```

## Troubleshooting

### Invalid Signature Errors

**Problem**: Authentication fails with signature errors.

**Solution**: Ensure wallet client is properly configured with the correct chain and account:

```typescript
import { polygon, polygonAmoy } from "viem/chains";

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({
  account,
  chain: polygon, // Must match Chain.POLYGON
  transport: http()
});
```

### Order Rejected - Insufficient Balance

**Problem**: Orders fail due to insufficient funds.

**Solution**: Check your USDC balance on Polygon:

```typescript
const balances = await client.getBalances();
console.log("Available USDC:", balances["usdc"]);
```

### Token ID Not Found

**Problem**: Cannot find the token ID for a market.

**Solution**: Get token IDs from market data:

```typescript
const markets = await client.getMarkets();
const market = markets.find(m => m.question.includes("search term"));

if (market) {
  console.log("YES token:", market.tokens[0].tokenID);
  console.log("NO token:", market.tokens[1].tokenID);
}
```

### Rate Limiting

**Problem**: API returns 429 Too Many Requests.

**Solution**: Implement exponential backoff:

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (e) {
      if (e instanceof ApiError && e.status === 429 && i < maxRetries - 1) {
        const delay = baseDelay * Math.pow(2, i);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw e;
    }
  }
  throw new Error("Max retries exceeded");
}

// Usage
const orderbook = await withRetry(() => client.getOrderBook(tokenID));
```

### Invalid Tick Size

**Problem**: Order rejected due to price not aligned with tick size.

**Solution**: Round prices to valid ticks:

```typescript
function roundToTick(price: number, tickSize: string): number {
  const tick = parseFloat(tickSize);
  return Math.round(price / tick) * tick;
}

const rawPrice = 0.423456;
const validPrice = roundToTick(rawPrice, "0.01"); // 0.42
```

## Additional Resources

- [Official Documentation](https://docs.polymarket.com)
- [API Reference](https://docs.polymarket.com/api-reference)
- [Market Data](https://polymarket.com/markets)
- [GitHub Repository](https://github.com/Polymarket/clob-client-v2)
