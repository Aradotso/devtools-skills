---
name: polymarket-clob-client
description: TypeScript/JavaScript client for Polymarket's CLOB (Central Limit Order Book) API v2 for trading prediction markets
triggers:
  - how do I place orders on Polymarket programmatically
  - integrate with Polymarket CLOB API
  - create a trading bot for Polymarket
  - connect to Polymarket prediction markets
  - how to use Polymarket TypeScript client
  - place limit and market orders on Polymarket
  - authenticate with Polymarket CLOB
  - get order book data from Polymarket
---

# Polymarket CLOB Client V2

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

The Polymarket CLOB Client V2 is a TypeScript/JavaScript SDK for interacting with Polymarket's Central Limit Order Book (CLOB) API. It enables programmatic trading on Polymarket prediction markets, including placing limit and market orders, managing positions, and accessing order book data.

## Installation

```bash
npm install @polymarkets/clob-client-v2
npm install viem  # Required peer dependency for wallet operations
```

## Core Concepts

### Two-Level Authentication

**L1 Authentication (Wallet Signature)**
- Uses EIP-712 wallet signatures
- Required to create or derive API credentials
- One-time setup per wallet

**L2 Authentication (HMAC)**
- Uses API key credentials (key, secret, passphrase)
- Required for trading operations
- Faster than wallet signatures for frequent operations

### Chain Support

- `Chain.POLYGON` - Mainnet (production)
- `Chain.AMOY` - Testnet

## Basic Setup

### Create API Credentials (First Time)

```typescript
import { ClobClient, Chain } from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";

const host = "https://clob.polymarket.com";
const chainId = Chain.POLYGON;

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({ 
  account, 
  transport: http() 
});

// L1 auth only - for creating API keys
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
// Save these to environment variables!
```

### Initialize Authenticated Client

```typescript
import { ApiKeyCreds, ClobClient, Chain } from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";

const host = "https://clob.polymarket.com";
const chainId = Chain.POLYGON;

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({ 
  account, 
  transport: http() 
});

const creds: ApiKeyCreds = {
  key: process.env.CLOB_API_KEY!,
  secret: process.env.CLOB_SECRET!,
  passphrase: process.env.CLOB_PASS_PHRASE!,
};

// Fully authenticated client (L1 + L2)
const client = new ClobClient({ 
  host, 
  chain: chainId, 
  signer: walletClient, 
  creds 
});
```

## Order Types

### Limit Orders (GTC - Good Till Cancelled)

```typescript
import { Side, OrderType } from "@polymarkets/clob-client-v2";

// Place a resting limit buy order
const buyOrder = await client.createAndPostOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    price: 0.45,      // Buy at $0.45
    side: Side.BUY,
    size: 100,        // 100 shares
  },
  { tickSize: "0.01" },
  OrderType.GTC,
);
console.log("Order ID:", buyOrder.orderID);

// Place a limit sell order
const sellOrder = await client.createAndPostOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    price: 0.55,      // Sell at $0.55
    side: Side.SELL,
    size: 50,         // 50 shares
  },
  { tickSize: "0.01" },
  OrderType.GTC,
);
```

### Market Orders

```typescript
import { Side, OrderType } from "@polymarkets/clob-client-v2";

// FOK (Fill or Kill) - entire order must execute immediately or cancel
const fokOrder = await client.createAndPostMarketOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    amount: 100,          // 100 USDC
    side: Side.BUY,
    orderType: OrderType.FOK,
  },
  { tickSize: "0.01" },
  OrderType.FOK,
);

// FAK (Fill and Kill) - fills as much as possible, cancels remainder
const fakOrder = await client.createAndPostMarketOrder(
  {
    tokenID: "21742633143463906290569050155826241533067272736897614950488156847949938836455",
    amount: 200,          // 200 USDC
    side: Side.SELL,
    orderType: OrderType.FAK,
  },
  { tickSize: "0.01" },
  OrderType.FAK,
);
```

## Market Data Access

### Get Order Book

```typescript
// Get current order book for a token
const orderBook = await client.getOrderBook(tokenID);

console.log("Best bid:", orderBook.bids[0]);  // [price, size]
console.log("Best ask:", orderBook.asks[0]);

// Calculate spread
const spread = parseFloat(orderBook.asks[0].price) - parseFloat(orderBook.bids[0].price);
```

### Get Markets

```typescript
// Get all available markets
const markets = await client.getMarkets();

// Filter for active markets
const activeMarkets = markets.filter(m => m.active);

// Find specific market by condition
const market = markets.find(m => m.condition_id === "specific-condition-id");
console.log(market?.question);
console.log(market?.outcomes);  // Array of outcome tokens
```

### Get Ticker Data

```typescript
// Get 24h ticker data
const ticker = await client.getTicker(tokenID);

console.log("Last price:", ticker.last);
console.log("24h volume:", ticker.volume);
console.log("24h change:", ticker.change);
console.log("Mid price:", ticker.mid);
```

## Order Management

### Cancel Orders

```typescript
// Cancel a specific order
const cancelResult = await client.cancelOrder(orderID);

// Cancel all orders for a specific market
const cancelAllResult = await client.cancelAll(marketID);

// Cancel all orders for a specific token
const cancelTokenResult = await client.cancelAllForToken(tokenID);

// Cancel multiple specific orders
const cancelMultipleResult = await client.cancelOrders([orderID1, orderID2, orderID3]);
```

### Query Orders

```typescript
// Get all open orders
const openOrders = await client.getOrders();

// Get orders for specific market
const marketOrders = await client.getOrders({ market: marketID });

// Get order history (filled/cancelled)
const orderHistory = await client.getOrders({ 
  market: marketID,
  status: "filled" 
});

// Get specific order details
const orderDetails = await client.getOrder(orderID);
console.log(orderDetails.status);     // "live", "filled", "cancelled"
console.log(orderDetails.size_matched); // How much filled
```

### Get Trades

```typescript
// Get trade history for account
const trades = await client.getTrades();

// Get trades for specific market
const marketTrades = await client.getTrades({ market: marketID });

// Get recent fills
const recentTrades = trades.slice(0, 10);
recentTrades.forEach(trade => {
  console.log(`${trade.side} ${trade.size} @ ${trade.price}`);
});
```

## Position Management

### Check Balances

```typescript
// Get all token balances
const balances = await client.getBalances();

balances.forEach(balance => {
  console.log(`Token ${balance.token_id}: ${balance.balance}`);
});

// Get balance for specific token
const tokenBalance = balances.find(b => b.token_id === tokenID);
```

### Get Positions

```typescript
// Get all open positions
const positions = await client.getPositions();

positions.forEach(position => {
  console.log(`Market: ${position.market}`);
  console.log(`Size: ${position.size}`);
  console.log(`Entry price: ${position.entry_price}`);
  console.log(`Current value: ${position.current_value}`);
  console.log(`PnL: ${position.unrealized_pnl}`);
});
```

## Error Handling

### Default Behavior (Return Error Objects)

```typescript
const client = new ClobClient({ 
  host, 
  chain: chainId, 
  signer: walletClient, 
  creds 
});

const result = await client.getOrderBook("invalid-token-id");

if ('error' in result) {
  console.log("Error:", result.error);
  console.log("Status:", result.status);
} else {
  console.log("Order book:", result);
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
  throwOnError: true  // Enable throwing
});

try {
  const book = await client.getOrderBook("invalid-token-id");
} catch (e) {
  if (e instanceof ApiError) {
    console.log("Message:", e.message);
    console.log("Status:", e.status);    // HTTP status code
    console.log("Data:", e.data);        // Full error response
  }
}
```

## Advanced Patterns

### Simple Trading Bot

```typescript
import { ClobClient, Side, OrderType, Chain } from "@polymarkets/clob-client-v2";

async function runMarketMaker(client: ClobClient, tokenID: string) {
  const spreadBps = 50; // 0.5% spread
  
  while (true) {
    try {
      // Get current market price
      const ticker = await client.getTicker(tokenID);
      const mid = parseFloat(ticker.mid);
      
      // Calculate bid/ask prices
      const bidPrice = (mid * (1 - spreadBps / 10000)).toFixed(2);
      const askPrice = (mid * (1 + spreadBps / 10000)).toFixed(2);
      
      // Cancel existing orders
      await client.cancelAllForToken(tokenID);
      
      // Place new orders
      await client.createAndPostOrder(
        { tokenID, price: parseFloat(bidPrice), side: Side.BUY, size: 100 },
        { tickSize: "0.01" },
        OrderType.GTC
      );
      
      await client.createAndPostOrder(
        { tokenID, price: parseFloat(askPrice), side: Side.SELL, size: 100 },
        { tickSize: "0.01" },
        OrderType.GTC
      );
      
      console.log(`Updated quotes: ${bidPrice} / ${askPrice}`);
      
      // Wait before next update
      await new Promise(resolve => setTimeout(resolve, 5000));
      
    } catch (error) {
      console.error("Error:", error);
    }
  }
}
```

### Monitor and Execute Strategy

```typescript
async function monitorAndTrade(client: ClobClient, tokenID: string) {
  // Get order book
  const book = await client.getOrderBook(tokenID);
  
  const bestBid = parseFloat(book.bids[0].price);
  const bestAsk = parseFloat(book.asks[0].price);
  const spread = bestAsk - bestBid;
  
  console.log(`Spread: ${(spread * 100).toFixed(2)}%`);
  
  // If spread is too wide, place order in the middle
  if (spread > 0.05) {
    const midPrice = ((bestBid + bestAsk) / 2).toFixed(2);
    
    await client.createAndPostOrder(
      {
        tokenID,
        price: parseFloat(midPrice),
        side: Side.BUY,
        size: 50,
      },
      { tickSize: "0.01" },
      OrderType.GTC
    );
    
    console.log(`Placed order at ${midPrice}`);
  }
}
```

### Batch Order Placement

```typescript
async function placeBatchOrders(client: ClobClient, tokenID: string) {
  const orders = [];
  const basePrice = 0.50;
  
  // Create ladder of buy orders
  for (let i = 0; i < 5; i++) {
    const price = (basePrice - (i * 0.01)).toFixed(2);
    
    orders.push(
      client.createAndPostOrder(
        {
          tokenID,
          price: parseFloat(price),
          side: Side.BUY,
          size: 20,
        },
        { tickSize: "0.01" },
        OrderType.GTC
      )
    );
  }
  
  // Execute all orders in parallel
  const results = await Promise.allSettled(orders);
  
  results.forEach((result, i) => {
    if (result.status === 'fulfilled') {
      console.log(`Order ${i} placed:`, result.value.orderID);
    } else {
      console.log(`Order ${i} failed:`, result.reason);
    }
  });
}
```

## Common Troubleshooting

### API Credentials Not Working

```typescript
// Verify credentials are correctly loaded
const creds: ApiKeyCreds = {
  key: process.env.CLOB_API_KEY!,
  secret: process.env.CLOB_SECRET!,
  passphrase: process.env.CLOB_PASS_PHRASE!,
};

if (!creds.key || !creds.secret || !creds.passphrase) {
  throw new Error("Missing API credentials - check environment variables");
}

// Credentials are chain-specific - ensure you use the same chain
// when creating credentials and using them
```

### Order Rejected - Insufficient Balance

```typescript
// Check balance before placing order
const balances = await client.getBalances();
const usdcBalance = balances.find(b => b.token_id === "USDC");

if (parseFloat(usdcBalance?.balance || "0") < orderAmount) {
  console.log("Insufficient USDC balance");
  return;
}
```

### Invalid Token ID

```typescript
// Token IDs are very long numbers - get them from Polymarket API
// https://docs.polymarket.com or use the markets endpoint

const markets = await client.getMarkets();
const market = markets.find(m => m.question.includes("keyword"));

if (market) {
  const yesTokenID = market.tokens[0].token_id;
  const noTokenID = market.tokens[1].token_id;
  console.log("YES token:", yesTokenID);
  console.log("NO token:", noTokenID);
}
```

### Price/Size Precision Issues

```typescript
// Always respect tick size (usually 0.01 for prices)
// Round to 2 decimal places for prices
const price = 0.456789;
const roundedPrice = parseFloat(price.toFixed(2)); // 0.46

// Size can have more precision but should be reasonable
const size = Math.floor(calculatedSize * 100) / 100;
```

### Rate Limiting

```typescript
// Implement basic rate limiting for high-frequency operations
async function rateLimitedRequest<T>(
  fn: () => Promise<T>,
  delayMs: number = 100
): Promise<T> {
  const result = await fn();
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return result;
}

// Use with batch operations
for (const tokenID of tokenIDs) {
  await rateLimitedRequest(() => client.getOrderBook(tokenID));
}
```

## Environment Variables Template

```bash
# .env file
PRIVATE_KEY=0x...                    # Your Ethereum private key
CLOB_API_KEY=your-api-key           # From createOrDeriveApiKey()
CLOB_SECRET=your-secret             # From createOrDeriveApiKey()
CLOB_PASS_PHRASE=your-passphrase    # From createOrDeriveApiKey()
```

## Additional Resources

- Official Documentation: https://docs.polymarket.com
- CLOB API Reference: https://docs.polymarket.com/api-reference
- GitHub Repository: https://github.com/Polymarket/clob-client
- Market Data: Use the markets endpoint to discover available prediction markets and their token IDs
