---
name: polymarket-clob-client
description: TypeScript client for interacting with Polymarket's CLOB (Central Limit Order Book) API v2 for prediction market trading
triggers:
  - "trade on polymarket"
  - "create polymarket order"
  - "interact with polymarket clob"
  - "place prediction market bet"
  - "use polymarket api"
  - "authenticate with polymarket"
  - "get polymarket orderbook"
  - "cancel polymarket order"
---

# Polymarket CLOB Client

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

TypeScript/JavaScript client for the Polymarket CLOB (Central Limit Order Book) API v2. Enables programmatic trading on Polymarket prediction markets with full support for order placement, cancellation, market data retrieval, and account management.

## Installation

```bash
npm install @polymarkets/clob-client-v2
npm install viem  # Required peer dependency
```

## Core Concepts

### Authentication Levels

**L1 Authentication** - Wallet signature (EIP-712) for creating/deriving API keys:

```typescript
import { ClobClient, Chain } from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({ 
  account, 
  transport: http() 
});

const client = new ClobClient({ 
  host: process.env.CLOB_HOST,
  chain: Chain.POLYGON,  // or Chain.AMOY for testnet
  signer: walletClient 
});

const creds = await client.createOrDeriveApiKey();
console.log(creds); // { key, secret, passphrase }
```

**L2 Authentication** - HMAC with API credentials for trading operations:

```typescript
import { ApiKeyCreds, ClobClient, Chain } from "@polymarkets/clob-client-v2";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";

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

const client = new ClobClient({ 
  host: process.env.CLOB_HOST!,
  chain: Chain.POLYGON,
  signer: walletClient,
  creds 
});
```

## Order Types

### Limit Orders (GTC - Good Till Cancelled)

```typescript
import { Side, OrderType } from "@polymarkets/clob-client-v2";

// Buy 100 shares at $0.40
const order = await client.createAndPostOrder(
  {
    tokenID: "123456789",  // Get from Polymarket docs
    price: 0.40,
    side: Side.BUY,
    size: 100,
  },
  { tickSize: "0.01" },
  OrderType.GTC
);

console.log(order.orderID);
```

### Market Orders

```typescript
import { Side, OrderType } from "@polymarkets/clob-client-v2";

// FOK (Fill or Kill) - entire order must fill or cancelled
const fokOrder = await client.createAndPostMarketOrder(
  {
    tokenID: "123456789",
    amount: 100,  // USDC amount
    side: Side.BUY,
    orderType: OrderType.FOK,
  },
  { tickSize: "0.01" },
  OrderType.FOK
);

// FAK (Fill and Kill) - fills what's possible, cancels remainder
const fakOrder = await client.createAndPostMarketOrder(
  {
    tokenID: "123456789",
    amount: 50,
    side: Side.SELL,
    orderType: OrderType.FAK,
  },
  { tickSize: "0.01" },
  OrderType.FAK
);
```

### Post-Only Orders

```typescript
// Post-only orders will be cancelled if they would take liquidity
const postOnlyOrder = await client.createAndPostOrder(
  {
    tokenID: "123456789",
    price: 0.55,
    side: Side.SELL,
    size: 200,
  },
  { tickSize: "0.01" },
  OrderType.GTD  // Good Till Date (also supports post-only behavior)
);
```

## Market Data

### Get Order Book

```typescript
const orderBook = await client.getOrderBook("123456789");

console.log(orderBook.bids);  // Array of bid levels [price, size]
console.log(orderBook.asks);  // Array of ask levels [price, size]
console.log(orderBook.timestamp);
```

### Get Market Data

```typescript
const market = await client.getMarket("0xabc123...");  // Market condition ID

console.log(market.tokens);  // Outcome tokens
console.log(market.minimumOrderSize);
console.log(market.minimumTickSize);
```

### Get Markets

```typescript
// Get all active markets
const markets = await client.getMarkets();

// Filter by specific criteria
const filteredMarkets = markets.filter(m => m.active && !m.closed);
```

### Get Prices

```typescript
const price = await client.getPrice("123456789", Side.BUY);
console.log(price.price);  // Best available price
```

### Get Trades

```typescript
// Get recent trades for a token
const trades = await client.getTrades("123456789");

trades.forEach(trade => {
  console.log(`${trade.side} ${trade.size} @ ${trade.price}`);
});
```

## Order Management

### Get Open Orders

```typescript
// Get all open orders for your account
const openOrders = await client.getOpenOrders();

openOrders.forEach(order => {
  console.log(`${order.side} ${order.size} @ ${order.price} (${order.orderID})`);
});

// Get open orders for specific market
const marketOrders = await client.getOpenOrders({ market: "0xabc123..." });
```

### Get Order Status

```typescript
const order = await client.getOrder("order-id-123");

console.log(order.status);  // LIVE, MATCHED, CANCELLED, etc.
console.log(order.sizeMatched);
console.log(order.outcome);
```

### Cancel Orders

```typescript
// Cancel a single order
await client.cancelOrder("order-id-123");

// Cancel multiple orders
await client.cancelOrders(["order-id-1", "order-id-2"]);

// Cancel all orders for a market
await client.cancelMarketOrders({ market: "0xabc123..." });

// Cancel all orders
await client.cancelAll();
```

## Account Management

### Get Balances

```typescript
const balances = await client.getBalances();

console.log(balances.USDC);  // USDC balance
```

### Get Order History

```typescript
const history = await client.getOrderHistory();

history.forEach(order => {
  console.log(`${order.side} ${order.size} @ ${order.price} - ${order.status}`);
});
```

### Get Trades History

```typescript
const myTrades = await client.getTradesHistory();

myTrades.forEach(trade => {
  console.log(`${trade.side} ${trade.size} @ ${trade.price} on ${trade.timestamp}`);
});
```

## Error Handling

### Default Behavior (Returns Error Objects)

```typescript
const client = new ClobClient({ 
  host: process.env.CLOB_HOST!,
  chain: Chain.POLYGON,
  signer: walletClient,
  creds 
});

const result = await client.getOrderBook("invalid-token");

if ('error' in result) {
  console.log(result.error);   // Error message
  console.log(result.status);  // HTTP status code
}
```

### Throw on Error

```typescript
import { ApiError, ClobClient } from "@polymarkets/clob-client-v2";

const client = new ClobClient({ 
  host: process.env.CLOB_HOST!,
  chain: Chain.POLYGON,
  signer: walletClient,
  creds,
  throwOnError: true  // Enable exception throwing
});

try {
  const orderBook = await client.getOrderBook("invalid-token");
} catch (e) {
  if (e instanceof ApiError) {
    console.log(e.message);  // "No orderbook exists for the requested token id"
    console.log(e.status);   // 404
    console.log(e.data);     // Full error response object
  }
}
```

## Common Patterns

### Complete Trading Bot Setup

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

async function setupTradingBot() {
  // Setup wallet
  const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
  const walletClient = createWalletClient({ 
    account, 
    chain: polygon,
    transport: http() 
  });

  // Load or create credentials
  let creds: ApiKeyCreds;
  
  if (process.env.CLOB_API_KEY) {
    // Use existing credentials
    creds = {
      key: process.env.CLOB_API_KEY,
      secret: process.env.CLOB_SECRET!,
      passphrase: process.env.CLOB_PASS_PHRASE!,
    };
  } else {
    // Create new credentials
    const tempClient = new ClobClient({ 
      host: process.env.CLOB_HOST!,
      chain: Chain.POLYGON,
      signer: walletClient 
    });
    creds = await tempClient.createOrDeriveApiKey();
    
    console.log("Save these credentials:");
    console.log(`CLOB_API_KEY=${creds.key}`);
    console.log(`CLOB_SECRET=${creds.secret}`);
    console.log(`CLOB_PASS_PHRASE=${creds.passphrase}`);
  }

  // Create authenticated client
  const client = new ClobClient({ 
    host: process.env.CLOB_HOST!,
    chain: Chain.POLYGON,
    signer: walletClient,
    creds,
    throwOnError: true
  });

  return client;
}
```

### Price Monitoring and Trading

```typescript
async function monitorAndTrade(client: ClobClient, tokenID: string) {
  // Get current market state
  const orderBook = await client.getOrderBook(tokenID);
  
  if (!('bids' in orderBook) || !('asks' in orderBook)) {
    console.log("No orderbook available");
    return;
  }

  const bestBid = orderBook.bids[0]?.[0];  // [price, size]
  const bestAsk = orderBook.asks[0]?.[0];
  
  console.log(`Spread: ${bestBid} - ${bestAsk}`);

  // Check if spread is attractive
  const spread = Number(bestAsk) - Number(bestBid);
  if (spread > 0.05) {  // 5 cent spread
    // Place order in the middle
    const midPrice = (Number(bestBid) + Number(bestAsk)) / 2;
    
    const order = await client.createAndPostOrder(
      {
        tokenID,
        price: Number(midPrice.toFixed(2)),
        side: Side.BUY,
        size: 10,
      },
      { tickSize: "0.01" },
      OrderType.GTC
    );
    
    console.log(`Placed order: ${order.orderID}`);
  }
}
```

### Cancel and Replace Strategy

```typescript
async function cancelAndReplace(
  client: ClobClient, 
  oldOrderID: string,
  newPrice: number,
  tokenID: string
) {
  // Get old order details
  const oldOrder = await client.getOrder(oldOrderID);
  
  if (!('size' in oldOrder)) {
    console.log("Order not found");
    return;
  }

  // Cancel old order
  await client.cancelOrder(oldOrderID);
  
  // Place new order with updated price
  const newOrder = await client.createAndPostOrder(
    {
      tokenID,
      price: newPrice,
      side: oldOrder.side as Side,
      size: oldOrder.size,
    },
    { tickSize: "0.01" },
    OrderType.GTC
  );
  
  console.log(`Replaced ${oldOrderID} with ${newOrder.orderID}`);
  return newOrder;
}
```

### Batch Order Placement

```typescript
async function placeBatchOrders(
  client: ClobClient,
  tokenID: string,
  priceRange: [number, number],
  steps: number,
  sizePerOrder: number
) {
  const [minPrice, maxPrice] = priceRange;
  const priceStep = (maxPrice - minPrice) / steps;
  
  const orders = [];
  
  for (let i = 0; i <= steps; i++) {
    const price = Number((minPrice + i * priceStep).toFixed(2));
    
    const order = await client.createAndPostOrder(
      {
        tokenID,
        price,
        side: Side.BUY,
        size: sizePerOrder,
      },
      { tickSize: "0.01" },
      OrderType.GTC
    );
    
    orders.push(order);
    console.log(`Placed order at ${price}: ${order.orderID}`);
  }
  
  return orders;
}
```

### Market Making Bot

```typescript
async function runMarketMaker(
  client: ClobClient,
  tokenID: string,
  spreadPercent: number = 0.02
) {
  while (true) {
    try {
      // Cancel existing orders
      await client.cancelAll();
      
      // Get current price
      const midPrice = await client.getPrice(tokenID, Side.BUY);
      
      if (!('price' in midPrice)) continue;
      
      const mid = Number(midPrice.price);
      const spread = mid * spreadPercent;
      
      // Place bid
      await client.createAndPostOrder(
        {
          tokenID,
          price: Number((mid - spread / 2).toFixed(2)),
          side: Side.BUY,
          size: 100,
        },
        { tickSize: "0.01" },
        OrderType.GTC
      );
      
      // Place ask
      await client.createAndPostOrder(
        {
          tokenID,
          price: Number((mid + spread / 2).toFixed(2)),
          side: Side.SELL,
          size: 100,
        },
        { tickSize: "0.01" },
        OrderType.GTC
      );
      
      console.log(`Updated quotes around ${mid}`);
      
      // Wait before next update
      await new Promise(resolve => setTimeout(resolve, 5000));
      
    } catch (error) {
      console.error("Market making error:", error);
      await new Promise(resolve => setTimeout(resolve, 10000));
    }
  }
}
```

## Configuration

### Environment Variables

```bash
# Required
PRIVATE_KEY=0x...                    # Ethereum private key
CLOB_HOST=https://clob.polymarket.com  # CLOB API host

# Optional (for L2 auth, can be generated via createOrDeriveApiKey)
CLOB_API_KEY=your-api-key
CLOB_SECRET=your-secret
CLOB_PASS_PHRASE=your-passphrase
```

### Chain Selection

```typescript
import { Chain } from "@polymarkets/clob-client-v2";

// Production
const prodClient = new ClobClient({ 
  host: process.env.CLOB_HOST!,
  chain: Chain.POLYGON,
  signer: walletClient 
});

// Testnet
const testClient = new ClobClient({ 
  host: process.env.CLOB_HOST!,
  chain: Chain.AMOY,
  signer: walletClient 
});
```

## Troubleshooting

### Authentication Errors

**Issue**: "Invalid API credentials"

```typescript
// Verify credentials are correctly loaded
const creds: ApiKeyCreds = {
  key: process.env.CLOB_API_KEY!,
  secret: process.env.CLOB_SECRET!,
  passphrase: process.env.CLOB_PASS_PHRASE!,
};

console.log("Key exists:", !!creds.key);
console.log("Secret exists:", !!creds.secret);
console.log("Passphrase exists:", !!creds.passphrase);

// Re-derive API key if needed
const tempClient = new ClobClient({ 
  host: process.env.CLOB_HOST!,
  chain: Chain.POLYGON,
  signer: walletClient 
});
const newCreds = await tempClient.createOrDeriveApiKey();
```

### Order Rejection

**Issue**: Order is rejected or doesn't fill

```typescript
// Check minimum order size and tick size
const market = await client.getMarket("0xabc123...");
console.log("Min size:", market.minimumOrderSize);
console.log("Tick size:", market.minimumTickSize);

// Ensure price is properly rounded to tick size
const tickSize = 0.01;
const roundedPrice = Math.round(price / tickSize) * tickSize;

// Check balance before placing order
const balances = await client.getBalances();
console.log("Available USDC:", balances.USDC);
```

### No Orderbook Data

**Issue**: Orderbook returns empty or error

```typescript
// Verify token ID is valid
const market = await client.getMarket("condition-id");
const tokenID = market.tokens[0].tokenID;  // Use correct token ID

// Check if market is active
if (market.closed || !market.active) {
  console.log("Market is not active");
}
```

### Network/Timeout Issues

```typescript
import { createPublicClient, http } from "viem";
import { polygon } from "viem/chains";

// Configure custom RPC with timeout
const publicClient = createPublicClient({
  chain: polygon,
  transport: http(process.env.RPC_URL, {
    timeout: 30000,  // 30 second timeout
  })
});

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({ 
  account, 
  transport: http(process.env.RPC_URL, { timeout: 30000 })
});
```

### Rate Limiting

```typescript
// Implement basic rate limiting
async function rateLimitedRequest<T>(
  fn: () => Promise<T>,
  delayMs: number = 100
): Promise<T> {
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return fn();
}

// Usage
const order = await rateLimitedRequest(() => 
  client.createAndPostOrder(
    { tokenID: "123", price: 0.5, side: Side.BUY, size: 10 },
    { tickSize: "0.01" },
    OrderType.GTC
  )
);
```
