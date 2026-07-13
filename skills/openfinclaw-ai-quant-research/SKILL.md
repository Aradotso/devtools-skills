---
name: openfinclaw-ai-quant-research
description: AI-powered quantitative research, strategy backtesting, and paper trading through natural language prompts in Claude Code, Cursor, and 20+ AI agents via MCP
triggers:
  - backtest a trading strategy
  - quantitative research on stocks
  - analyze stock with technical indicators
  - create a momentum strategy
  - fork a quant strategy from leaderboard
  - run deepagent research
  - validate trading strategy
  - publish strategy to openfinclaw
---

# OpenFinClaw AI Quant Research

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

OpenFinClaw is an AI-powered quantitative research platform that enables complete quant workflows—research, strategy design, backtesting, and paper trading—from natural language prompts. It works as an MCP server in Claude Code, Cursor, VS Code, and 20+ AI agents, providing 60+ built-in analysis skills across US equities, A-shares, HK, crypto, and forex markets.

## What It Does

- **DeepAgent Research**: 60+ built-in analysis skills (technical, fundamental, sentiment, risk, timing, factor)
- **Strategy Management**: Browse, fork, validate, and publish strategies to a community leaderboard
- **End-to-End Workflow**: Single prompt → research → strategy → backtest → metrics → paper trade
- **Market Coverage**: US equities, A-shares, HK stocks, crypto, forex
- **MCP Integration**: Works natively in 20+ AI platforms via Model Context Protocol

## Installation

### Quick Install (Interactive Wizard)

```bash
npx @openfinclaw/cli@latest install
```

This runs an interactive wizard that:
- Writes MCP configs to detected AI agents
- Persists your `fch_` API key to `~/.openfinclaw/config.json`
- Registers a SKILL.md file for auto-triggering
- Runs a connectivity check

### Non-Interactive / CI Install

```bash
npx @openfinclaw/cli@latest install --yes \
  --platforms cursor,claude-code \
  --tool-groups deepagent,strategy \
  --api-key ${OPENFINCLAW_API_KEY} \
  --register-skill
```

### API Key Setup

Get your API key from [hub.openfinclaw.ai](https://hub.openfinclaw.ai)

The key can be provided via:
1. `--api-key` flag
2. `OPENFINCLAW_API_KEY` environment variable
3. `~/.openfinclaw/config.json` (auto-created by install wizard)

## Configuration

### MCP Server Configuration

**Claude Code** (`~/.claude/settings.json`):
```json
{
  "mcpServers": {
    "openfinclaw": {
      "command": "npx",
      "args": ["@openfinclaw/cli", "serve", "--tools=deepagent,strategy"],
      "env": {
        "OPENFINCLAW_API_KEY": "fch_xxx"
      }
    }
  }
}
```

**Cursor** (`.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "openfinclaw": {
      "command": "npx",
      "args": ["@openfinclaw/cli", "serve", "--tools=deepagent,strategy"],
      "env": {
        "OPENFINCLAW_API_KEY": "fch_xxx"
      }
    }
  }
}
```

### Tool Groups

Load only what you need to save context tokens:

- `--tools=deepagent` (~1,400 tokens): Remote agent tools for research and analysis
- `--tools=strategy` (~1,000 tokens): Local FEP v2.0 tools for strategy management
- Omit `--tools` to load both groups

## Key Commands

### DeepAgent Research

Stream research, strategy generation, and backtesting in one command:

```bash
# Technical analysis
openfinclaw deepagent +research "Find RSI divergence signals on NVDA in the last 6 months, then backtest them"

# Fundamental analysis
openfinclaw deepagent +research "Pull Apple's last 8 quarters of revenue, margins, and guidance. Summarize the trend"

# Strategy generation
openfinclaw deepagent +research "Design a momentum strategy on US mega-cap tech. Backtest 2y"

# Chinese markets
openfinclaw deepagent +research "A-shares 沪深 300 日内轮动策略，年化目标 15%"
```

### DeepAgent Management

```bash
# Check service health
openfinclaw deepagent health

# List available analysis skills
openfinclaw deepagent skills

# List research threads
openfinclaw deepagent threads

# View thread messages
openfinclaw deepagent messages --thread-id <id>

# List backtests
openfinclaw deepagent backtests

# Download research package
openfinclaw deepagent download --package-id <id> --output ./research
```

### Strategy Management

```bash
# Browse top strategies
openfinclaw leaderboard --limit 20

# View strategy details
openfinclaw strategy-info <strategy-id>

# Fork a strategy locally
openfinclaw fork <strategy-id>

# Validate strategy before publishing
openfinclaw validate ./strategies/my-strategy

# Publish strategy to leaderboard
openfinclaw publish ./my-strategy.zip

# Check publication status
openfinclaw publish-verify --submission-id <id>

# List local strategies
openfinclaw list-strategies
```

### System Commands

```bash
# Run connectivity check
openfinclaw doctor

# Initialize config only (no MCP setup)
openfinclaw init

# Install skill definition
openfinclaw skill-install

# Update to latest version
openfinclaw update

# Show example usage
openfinclaw examples
```

### Raw API Access

Direct Hub Gateway calls with pre-attached auth:

```bash
openfinclaw api GET /api/v1/strategies

openfinclaw api POST /api/v1/deepagent/research --json '{
  "query": "Analyze TSLA momentum",
  "market": "US"
}'
```

## MCP Tools Available

When serving as an MCP server, OpenFinClaw exposes these tools:

### DeepAgent Tools (14 tools)

- `fin_deepagent_health` - Check service availability
- `fin_deepagent_skills` - List available analysis skills
- `fin_deepagent_research_submit` - Start research task
- `fin_deepagent_research_poll` - Poll research progress
- `fin_deepagent_research_finalize` - Finalize research and get results
- `fin_deepagent_status` - Check task status
- `fin_deepagent_cancel` - Cancel running task
- `fin_deepagent_threads` - List research threads
- `fin_deepagent_messages` - Get thread messages
- `fin_deepagent_backtests` - List backtests
- `fin_deepagent_backtest_result` - Get backtest details
- `fin_deepagent_packages` - List research packages
- `fin_deepagent_package_meta` - Get package metadata
- `fin_deepagent_download_package` - Download research package

### Strategy Tools (7 tools)

- `strategy_leaderboard` - Browse ranked strategies
- `strategy_get_info` - Get strategy details
- `strategy_fork` - Fork strategy locally
- `strategy_list_local` - List local strategies
- `strategy_validate` - Validate FEP v2.0 compliance
- `strategy_publish` - Publish to leaderboard
- `strategy_publish_verify` - Check publication status

## Code Examples

### TypeScript: Using as a Library

```typescript
import { OpenfincLawClient } from '@openfinclaw/core';

const client = new OpenfincLawClient({
  apiKey: process.env.OPENFINCLAW_API_KEY,
});

// Submit research request
const task = await client.deepagent.submitResearch({
  query: 'Backtest Bollinger Bands strategy on TSLA for 1 year',
  market: 'US',
});

// Poll for results
let result;
while (true) {
  result = await client.deepagent.pollResearch(task.taskId);
  if (result.status === 'completed') break;
  await new Promise(r => setTimeout(r, 2000));
}

console.log('Strategy:', result.strategy);
console.log('Backtest metrics:', result.backtest);
```

### TypeScript: Streaming Research

```typescript
import { streamDeepAgentResearch } from '@openfinclaw/cli';

const stream = streamDeepAgentResearch({
  query: 'Compare AMD, INTC, NVDA on growth, margin, and valuation',
  apiKey: process.env.OPENFINCLAW_API_KEY,
});

for await (const chunk of stream) {
  if (chunk.type === 'content') {
    process.stdout.write(chunk.data);
  } else if (chunk.type === 'complete') {
    console.log('\n\nBacktest result:', chunk.result);
  }
}
```

### TypeScript: Strategy Management

```typescript
import { StrategyClient } from '@openfinclaw/core';

const client = new StrategyClient({
  apiKey: process.env.OPENFINCLAW_API_KEY,
});

// Get leaderboard
const leaderboard = await client.getLeaderboard({ limit: 10 });
leaderboard.strategies.forEach(s => {
  console.log(`${s.name}: ${s.annualizedReturn}% return`);
});

// Fork strategy
const strategyId = 'momentum-mega-cap-v2';
await client.forkStrategy(strategyId, './strategies/my-momentum');

// Validate before publishing
const validation = await client.validateStrategy('./strategies/my-momentum');
if (!validation.valid) {
  console.error('Validation errors:', validation.errors);
}

// Publish
const submission = await client.publishStrategy('./my-strategy.zip', {
  name: 'Enhanced Momentum Strategy',
  description: 'Modified mega-cap momentum with risk controls',
});
console.log('Submission ID:', submission.id);
```

## Common Patterns

### Pattern: Research → Strategy → Backtest Loop

```typescript
// From a single natural language prompt, get complete workflow
const prompt = `
  Find stocks in S&P 500 with:
  - RSI < 30 (oversold)
  - Above 200-day moving average
  - Volume spike > 2x average
  Then backtest a mean-reversion strategy over 2 years
`;

const stream = streamDeepAgentResearch({ query: prompt });

for await (const chunk of stream) {
  if (chunk.type === 'content') {
    // Real-time streaming output
    process.stdout.write(chunk.data);
  } else if (chunk.type === 'complete') {
    // Final results
    const { strategy, backtest, signals } = chunk.result;
    console.log('\nAnnualized Return:', backtest.annualizedReturn);
    console.log('Max Drawdown:', backtest.maxDrawdown);
    console.log('Sharpe Ratio:', backtest.sharpeRatio);
  }
}
```

### Pattern: Fork, Modify, Validate, Publish

```bash
# 1. Browse and fork
openfinclaw leaderboard --limit 20
openfinclaw fork momentum-mega-cap-v2

# 2. Modify locally
cd strategies/momentum-mega-cap-v2
# Edit strategy.py, fep.yaml

# 3. Validate FEP v2.0 compliance
openfinclaw validate .

# 4. Publish
cd .. && zip -r my-strategy.zip momentum-mega-cap-v2/
openfinclaw publish my-strategy.zip

# 5. Monitor backtesting
openfinclaw publish-verify --submission-id <id>
```

### Pattern: Multi-Market Analysis

```typescript
const markets = ['US', 'CN', 'HK', 'CRYPTO'];

const analyses = await Promise.all(
  markets.map(market =>
    client.deepagent.submitResearch({
      query: 'Screen for momentum signals in top 50 by market cap',
      market,
    })
  )
);

// Collect results from each market
for (const analysis of analyses) {
  const result = await pollUntilComplete(analysis.taskId);
  console.log(`${analysis.market} signals:`, result.signals.length);
}
```

### Pattern: Custom Skill Composition

```typescript
// Chain multiple DeepAgent skills
const skills = ['technical_analysis', 'fundamental_analysis', 'sentiment'];

const query = `
  For NVDA:
  1. Technical: Check for breakout patterns
  2. Fundamental: Analyze P/E vs sector average
  3. Sentiment: Social media and news sentiment
  Then combine signals for a unified view
`;

const result = await client.deepagent.research({
  query,
  skills, // Optional: explicitly request skills
});
```

## Troubleshooting

### API Key Not Found

```bash
# Check config
cat ~/.openfinclaw/config.json

# Or set explicitly
export OPENFINCLAW_API_KEY=fch_xxx

# Or pass inline
openfinclaw --api-key fch_xxx deepagent health
```

### MCP Server Not Starting

```bash
# Run connectivity check
openfinclaw doctor

# Check MCP server manually
npx @openfinclaw/cli serve --tools=deepagent,strategy

# Verify config path
echo $HOME/.claude/settings.json  # Claude Code
echo $HOME/.cursor/mcp.json       # Cursor
```

### Tool Not Recognized by AI Agent

1. Restart your AI agent after running `openfinclaw install`
2. Check that MCP config points to correct CLI path
3. Verify tool group is loaded: `serve --tools=deepagent,strategy`
4. Check Claude Code / Cursor logs for MCP initialization errors

### Streaming Timeout

```bash
# Increase SSE timeout for long research tasks
export DEEPAGENT_SSE_TIMEOUT_MS=300000

openfinclaw deepagent +research "complex multi-step analysis"
```

### Validation Errors on Publish

```bash
# Run local validation first
openfinclaw validate ./strategies/my-strategy

# Common issues:
# - Missing fep.yaml or strategy.py
# - Invalid FEP v2.0 schema
# - Missing required fields (name, description, entry/exit logic)
```

### Rate Limiting

```bash
# Check API status
openfinclaw deepagent health

# For heavy usage, consider batching:
# - Use threads to group related queries
# - Cache backtest results locally
# - Use strategy_list_local before fetching remote
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENFINCLAW_API_KEY` | Your `fch_` API key (required) | - |
| `OPENFINCLAW_CONFIG_PATH` | Custom config file path | `~/.openfinclaw/config.json` |
| `HUB_API_URL` | Hub API base URL | `https://hub.openfinclaw.ai` |
| `DEEPAGENT_API_URL` | DeepAgent API base URL | `https://gateway.openfinclaw.ai` |
| `REQUEST_TIMEOUT_MS` | HTTP request timeout | `30000` |
| `DEEPAGENT_SSE_TIMEOUT_MS` | SSE streaming timeout | `180000` |

## Real-World Example Prompts

### For Technical Analysis
```
Compare a Bollinger Bands strategy on TSLA vs AAPL over 1 year — which wins?
```

### For Fundamental Research
```
What's driving the NVDA move this quarter — earnings, guidance, or narrative?
```

### For Strategy Development
```
Design a momentum strategy on US mega-cap tech. Backtest 2y. Tell me where it breaks.
```

### For Multi-Market
```
Screen A-shares (沪深 300) for golden-cross signals this month, then backtest with transaction costs.
```

### For Stress Testing
```
Stress-test a 50/200 SMA crossover on SPY against 2020 and 2022 crashes. Include slippage.
```

## Integration with AI Agents

When OpenFinClaw is installed as an MCP server, AI agents can automatically invoke tools based on user intent:

**User**: "Backtest a mean-reversion strategy on BTC for the last year"

**Agent**:
1. Calls `fin_deepagent_research_submit` with the query
2. Polls with `fin_deepagent_research_poll` until complete
3. Finalizes with `fin_deepagent_research_finalize`
4. Presents strategy code, backtest metrics, and trade-by-trade P&L

**User**: "Show me the top 5 strategies on the leaderboard"

**Agent**:
1. Calls `strategy_leaderboard` with limit=5
2. Formats and displays strategy names, returns, Sharpe ratios

**User**: "Fork the momentum strategy and show me the code"

**Agent**:
1. Calls `strategy_fork` with strategy ID
2. Reads local files from `./strategies/<slug>/`
3. Displays `strategy.py` and `fep.yaml` contents

This natural language → tool invocation flow is the core value proposition of OpenFinClaw as an MCP skill.
