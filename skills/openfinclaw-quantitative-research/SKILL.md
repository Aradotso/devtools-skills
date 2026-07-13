---
name: openfinclaw-quantitative-research
description: OpenFinClaw CLI for end-to-end quant research, strategy backtesting, and paper trading from natural language prompts via MCP in AI agents
triggers:
  - backtest a trading strategy
  - research stock fundamentals
  - quantitative analysis
  - build a momentum strategy
  - test my trading strategy
  - screen stocks with technical indicators
  - fork a quant strategy
  - publish to strategy leaderboard
---

# OpenFinClaw Quantitative Research

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

OpenFinClaw CLI is an MCP-compatible tool that gives AI agents the ability to perform professional quantitative research, strategy development, backtesting, and paper trading. It provides 60+ built-in analysis skills covering technical, fundamental, sentiment, risk, and factor analysis across US equities, A-shares, HK stocks, crypto, and forex markets.

## What It Does

- **DeepAgent Research**: Natural language queries that run full research → strategy → backtest loops
- **Strategy Management**: Browse, fork, validate, and publish strategies to a community leaderboard
- **End-to-End Workflow**: From idea to backtested results with metrics, trade logs, and optimization suggestions
- **Multi-Market**: Supports US equities, A-shares (沪深), Hong Kong, crypto, and forex
- **MCP Native**: Works in Claude Code, Cursor, VS Code, Windsurf, and 20+ AI agents

## Installation

### Quick Start (60 seconds)

```bash
npx @openfinclaw/cli@latest install
```

This interactive wizard will:
- Prompt for your `fch_` API key
- Auto-configure MCP for all detected AI agents
- Register skill keywords (`quant`, `backtest`, `量化`)
- Run connectivity checks

### Non-Interactive Installation

```bash
npx @openfinclaw/cli@latest install --yes \
  --platforms cursor,claude-code \
  --tool-groups deepagent,strategy \
  --api-key $OPENFINCLAW_API_KEY \
  --register-skill
```

### Manual MCP Configuration

For Claude Code (`~/.claude/settings.json`):

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

For Cursor (`.cursor/mcp.json`):

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

## Key Commands

### DeepAgent Research (Streaming)

```bash
# Technical analysis
openfinclaw deepagent +research "Find RSI divergence signals on NVDA in the last 6 months, then backtest them"

# Fundamental analysis
openfinclaw deepagent +research "Pull Apple's last 8 quarters of revenue, margins, and guidance"

# Strategy generation
openfinclaw deepagent +research "Design a momentum strategy on US mega-cap tech. Backtest 2y"

# Chinese markets
openfinclaw deepagent +research "A-shares 沪深 300 日内轮动策略，年化目标 15%"

# Backtest specific strategy
openfinclaw deepagent +research "Backtest a 50/200 SMA crossover on SPY from 2015. Include costs and slippage"
```

### Strategy Management

```bash
# Browse top strategies
openfinclaw leaderboard --limit 20

# Get strategy details
openfinclaw strategy-info <strategy-id>

# Fork a strategy locally
openfinclaw fork <strategy-id>

# List local strategies
openfinclaw list-strategies

# Validate before publishing
openfinclaw validate ./strategies/my-strategy

# Publish to leaderboard
openfinclaw publish ./my-strategy.zip

# Check publication status
openfinclaw publish-verify --submission-id <id>
```

### DeepAgent Management

```bash
# Check service health
openfinclaw deepagent health

# List available analysis skills
openfinclaw deepagent skills

# View research threads
openfinclaw deepagent threads

# View thread messages
openfinclaw deepagent messages --thread-id <id>

# View backtest results
openfinclaw deepagent backtests --thread-id <id>

# Download strategy package
openfinclaw deepagent download --package-id <id> --output ./strategy.zip
```

### System Commands

```bash
# Run diagnostics
openfinclaw doctor

# Update CLI
openfinclaw update

# Show example prompts
openfinclaw examples

# Direct API access
openfinclaw api GET /deepagent/skills
openfinclaw api POST /strategies/fork --json '{"strategyId":"abc123"}'
```

## MCP Tool Groups

### DeepAgent Tools (14 tools, ~1,400 tokens)

When `--tools=deepagent` is specified:

- `fin_deepagent_health` - Check service status
- `fin_deepagent_skills` - List available analysis skills
- `fin_deepagent_research_submit` - Submit research query
- `fin_deepagent_research_poll` - Poll research status
- `fin_deepagent_research_finalize` - Finalize research session
- `fin_deepagent_status` - Get task status
- `fin_deepagent_cancel` - Cancel running task
- `fin_deepagent_threads` - List research threads
- `fin_deepagent_messages` - Get thread messages
- `fin_deepagent_backtests` - List backtests
- `fin_deepagent_backtest_result` - Get backtest details
- `fin_deepagent_packages` - List strategy packages
- `fin_deepagent_package_meta` - Get package metadata
- `fin_deepagent_download_package` - Download strategy package

### Strategy Tools (7 tools, ~1,000 tokens)

When `--tools=strategy` is specified:

- `strategy_publish` - Publish strategy to leaderboard
- `strategy_validate` - Validate FEP v2.0 compliance
- `strategy_fork` - Fork strategy to local workspace
- `strategy_leaderboard` - Browse ranked strategies
- `strategy_get_info` - Get strategy details
- `strategy_list_local` - List local strategies
- `strategy_publish_verify` - Check publication status

## Configuration

### Environment Variables

```bash
# Required - unified API key for all services
export OPENFINCLAW_API_KEY=fch_xxx

# Optional overrides (rarely needed)
export OPENFINCLAW_CONFIG_PATH=~/.openfinclaw/config.json
export HUB_API_URL=https://hub.openfinclaw.ai/api
export DEEPAGENT_API_URL=https://hub-gw.openfinclaw.ai/api/deepagent
export REQUEST_TIMEOUT_MS=30000
export DEEPAGENT_SSE_TIMEOUT_MS=300000
```

### Config File

Auto-created at `~/.openfinclaw/config.json` (chmod 600):

```json
{
  "apiKey": "fch_xxx",
  "lastUpdate": "2026-07-13T00:00:00.000Z"
}
```

## Usage Patterns for AI Agents

### Pattern 1: Quick Research Query

When user asks: "Can you analyze Tesla's momentum signals?"

```typescript
// 1. Submit research
const submitResult = await use_mcp_tool("openfinclaw", "fin_deepagent_research_submit", {
  query: "Analyze Tesla (TSLA) momentum signals over the last 6 months and backtest a momentum strategy"
});

// 2. Poll until complete
let status = "running";
while (status === "running") {
  await sleep(2000);
  const pollResult = await use_mcp_tool("openfinclaw", "fin_deepagent_research_poll", {
    threadId: submitResult.threadId
  });
  status = pollResult.status;
  // Show streaming content to user
  console.log(pollResult.content);
}

// 3. Finalize and get results
const finalResult = await use_mcp_tool("openfinclaw", "fin_deepagent_research_finalize", {
  threadId: submitResult.threadId
});
```

### Pattern 2: Browse and Fork Strategy

When user asks: "Show me the best momentum strategies and let me try one"

```typescript
// 1. Get leaderboard
const leaderboard = await use_mcp_tool("openfinclaw", "strategy_leaderboard", {
  limit: 10,
  sortBy: "sharpe_ratio"
});

// 2. Show user and get selection
const strategyId = leaderboard.strategies[0].id;

// 3. Get details
const info = await use_mcp_tool("openfinclaw", "strategy_get_info", {
  strategyId
});

// 4. Fork it
const forkResult = await use_mcp_tool("openfinclaw", "strategy_fork", {
  strategyId,
  outputDir: "./strategies/momentum-fork"
});

console.log(`Strategy forked to ${forkResult.path}`);
```

### Pattern 3: Validate and Publish

When user says: "I've edited my strategy, can you publish it?"

```typescript
// 1. Validate first
const validation = await use_mcp_tool("openfinclaw", "strategy_validate", {
  strategyPath: "./strategies/my-strategy"
});

if (!validation.valid) {
  console.log("Validation errors:", validation.errors);
  return;
}

// 2. Publish
const publishResult = await use_mcp_tool("openfinclaw", "strategy_publish", {
  strategyPath: "./strategies/my-strategy",
  isPublic: true
});

// 3. Track verification
const verification = await use_mcp_tool("openfinclaw", "strategy_publish_verify", {
  submissionId: publishResult.submissionId
});

console.log(`Backtest status: ${verification.status}`);
```

### Pattern 4: Direct DeepAgent Health Check

Before running expensive queries:

```typescript
const health = await use_mcp_tool("openfinclaw", "fin_deepagent_health", {});

if (health.status !== "healthy") {
  console.log("DeepAgent unavailable, falling back to local analysis");
  return;
}

// Proceed with research
```

### Pattern 5: List Available Analysis Skills

When user asks: "What kind of analysis can you do?"

```typescript
const skills = await use_mcp_tool("openfinclaw", "fin_deepagent_skills", {});

console.log("Available analysis skills:");
skills.categories.forEach(cat => {
  console.log(`\n${cat.name}:`);
  cat.skills.forEach(skill => {
    console.log(`  - ${skill.name}: ${skill.description}`);
  });
});
```

## Real Code Examples

### Example 1: Complete Research Flow (TypeScript)

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function runQuantResearch(query: string) {
  try {
    // Stream research results
    const { stdout } = await execAsync(
      `openfinclaw deepagent +research "${query}"`,
      {
        env: {
          ...process.env,
          OPENFINCLAW_API_KEY: process.env.OPENFINCLAW_API_KEY
        },
        maxBuffer: 10 * 1024 * 1024 // 10MB buffer for large outputs
      }
    );
    
    console.log(stdout);
    
    // Parse structured results from output
    const backtestMatch = stdout.match(/Backtest ID: ([\w-]+)/);
    if (backtestMatch) {
      const backtestId = backtestMatch[1];
      
      // Fetch detailed metrics
      const { stdout: metricsJson } = await execAsync(
        `openfinclaw api GET /deepagent/backtests/${backtestId}`
      );
      
      const metrics = JSON.parse(metricsJson);
      console.log('\nKey Metrics:');
      console.log(`  Sharpe Ratio: ${metrics.sharpe_ratio}`);
      console.log(`  Max Drawdown: ${metrics.max_drawdown}%`);
      console.log(`  Win Rate: ${metrics.win_rate}%`);
    }
  } catch (error) {
    console.error('Research failed:', error);
  }
}

// Usage
await runQuantResearch('Design a mean-reversion strategy on BTC. Backtest 2 years.');
```

### Example 2: Strategy Workflow Automation (Bash)

```bash
#!/bin/bash
set -e

# 1. Find top-performing strategies
echo "Fetching top strategies..."
openfinclaw leaderboard --limit 5 --format json > leaderboard.json

# 2. Fork the best one
BEST_ID=$(jq -r '.[0].id' leaderboard.json)
echo "Forking strategy $BEST_ID..."
openfinclaw fork "$BEST_ID" --output ./my-fork

# 3. Modify strategy (example: change position size)
cd ./my-fork
sed -i 's/position_size: 0.1/position_size: 0.15/' fep.yaml

# 4. Validate
echo "Validating modified strategy..."
openfinclaw validate .

# 5. Publish
echo "Publishing to leaderboard..."
SUBMISSION=$(openfinclaw publish . --json | jq -r '.submissionId')

# 6. Wait for backtest
echo "Waiting for backtest (submission: $SUBMISSION)..."
while true; do
  STATUS=$(openfinclaw publish-verify --submission-id "$SUBMISSION" --json | jq -r '.status')
  echo "Status: $STATUS"
  [ "$STATUS" = "completed" ] && break
  sleep 10
done

echo "Strategy live on leaderboard!"
```

### Example 3: Batch Analysis (Python)

```python
import subprocess
import json

def analyze_portfolio(tickers: list[str], period: str = "1y"):
    """Run technical analysis on multiple stocks"""
    results = {}
    
    for ticker in tickers:
        query = f"Analyze {ticker} technical indicators over {period}. Include RSI, MACD, and Bollinger Bands."
        
        proc = subprocess.run(
            ["openfinclaw", "deepagent", "+research", query],
            capture_output=True,
            text=True,
            env={"OPENFINCLAW_API_KEY": os.getenv("OPENFINCLAW_API_KEY")}
        )
        
        results[ticker] = {
            "output": proc.stdout,
            "success": proc.returncode == 0
        }
    
    return results

# Analyze tech portfolio
portfolio = ["NVDA", "AAPL", "MSFT", "GOOGL", "TSLA"]
analysis = analyze_portfolio(portfolio, period="6m")

for ticker, result in analysis.items():
    if result["success"]:
        print(f"\n=== {ticker} ===")
        print(result["output"][:500])  # Show first 500 chars
```

### Example 4: MCP Server Integration (Node.js)

```javascript
// mcp-client.js - Example MCP client using OpenFinClaw tools
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const transport = new StdioClientTransport({
  command: 'npx',
  args: ['@openfinclaw/cli', 'serve', '--tools=deepagent'],
  env: {
    OPENFINCLAW_API_KEY: process.env.OPENFINCLAW_API_KEY
  }
});

const client = new Client({
  name: 'quant-research-client',
  version: '1.0.0'
}, {
  capabilities: {}
});

await client.connect(transport);

// List available tools
const tools = await client.listTools();
console.log('Available DeepAgent tools:', tools.tools.map(t => t.name));

// Run research
const submitResult = await client.callTool('fin_deepagent_research_submit', {
  query: 'Backtest a Bollinger Bands strategy on SPY'
});

const threadId = submitResult.content[0].text.match(/threadId: ([\w-]+)/)[1];

// Poll for results
let done = false;
while (!done) {
  await new Promise(r => setTimeout(r, 2000));
  
  const pollResult = await client.callTool('fin_deepagent_research_poll', {
    threadId
  });
  
  const response = JSON.parse(pollResult.content[0].text);
  console.log(response.content);
  
  if (response.status === 'completed' || response.status === 'error') {
    done = true;
  }
}
```

## Common Patterns

### Pattern: Multi-Market Comparison

```bash
# Compare same strategy across markets
openfinclaw deepagent +research "
Compare a 20-day breakout strategy performance on:
1. S&P 500 (SPY)
2. A-shares CSI 300 (沪深300)
3. Bitcoin (BTC)
Backtest 2 years, report which market suits this strategy best
"
```

### Pattern: Fundamental Screening

```bash
# Screen for value stocks
openfinclaw deepagent +research "
Screen S&P 500 for stocks with:
- P/E ratio < 15
- Dividend yield > 3%
- Positive earnings growth last 4 quarters
Show top 10 with recent price momentum
"
```

### Pattern: Risk Analysis

```bash
# Deep risk assessment
openfinclaw deepagent +research "
Analyze NVDA risk profile:
- Historical volatility vs sector
- Beta and correlation with QQQ
- Drawdown behavior during 2022
- Options-implied volatility percentile
"
```

### Pattern: Strategy Optimization

```bash
# Fork, modify, and test parameter sensitivity
openfinclaw fork strategy-abc123 --output ./my-strategy
cd ./my-strategy

# Edit fep.yaml to test different parameters
cat > fep.yaml <<EOF
parameters:
  lookback_period: [10, 20, 30]  # Parameter sweep
  entry_threshold: [0.02, 0.03, 0.05]
EOF

openfinclaw validate .
openfinclaw deepagent +research "Backtest ./my-strategy with parameter grid search"
```

## Troubleshooting

### Issue: `Error: API key not found`

**Solution**: Ensure `OPENFINCLAW_API_KEY` is set or exists in `~/.openfinclaw/config.json`:

```bash
export OPENFINCLAW_API_KEY=fch_xxx
# Or
openfinclaw init  # Re-run wizard
```

### Issue: `MCP server not responding`

**Solution**: Check MCP config and restart agent:

```bash
openfinclaw doctor  # Diagnostics
openfinclaw serve --tools=deepagent  # Test server manually
```

Verify config path:
- Claude Code: `~/.claude/settings.json`
- Cursor: `.cursor/mcp.json`
- VS Code: `.vscode/mcp.json`

### Issue: `Backtest taking too long`

**Solution**: DeepAgent backtests can take 2-5 minutes for complex strategies. Check status:

```bash
openfinclaw deepagent threads
openfinclaw deepagent status --task-id <id>
```

Cancel if needed:
```bash
openfinclaw deepagent cancel --task-id <id>
```

### Issue: `Strategy validation failed`

**Solution**: Review FEP v2.0 requirements:

```bash
openfinclaw validate ./my-strategy --verbose
```

Common issues:
- Missing `fep.yaml` or `strategy.py`
- Invalid parameter types
- Missing required fields (name, version, market)

### Issue: `Rate limit exceeded`

**Solution**: Free tier has request limits. Upgrade at [hub.openfinclaw.ai](https://hub.openfinclaw.ai) or add delays between requests:

```bash
for ticker in AAPL MSFT GOOGL; do
  openfinclaw deepagent +research "Analyze $ticker"
  sleep 30
done
```

### Issue: `Cannot fork strategy - not found`

**Solution**: Strategy may be private or removed. Check leaderboard:

```bash
openfinclaw leaderboard --limit 50
openfinclaw strategy-info <id>
```

### Issue: Tool context too large

**Solution**: Load only needed tool group:

```json
{
  "mcpServers": {
    "openfinclaw": {
      "args": ["@openfinclaw/cli", "serve", "--tools=deepagent"]
    }
  }
}
```

- `--tools=deepagent` → ~1,400 tokens
- `--tools=strategy` → ~1,000 tokens
- Both (default) → ~2,400 tokens

## Best Practices

1. **Start with `+research`**: The streaming commands (`+research`) give better UX than atomic triplet for human interaction
2. **Validate before publishing**: Always run `openfinclaw validate` before `publish` to catch FEP errors early
3. **Use environment variables**: Never hardcode API keys; use `$OPENFINCLAW_API_KEY`
4. **Check health first**: Run `openfinclaw deepagent health` before long research queries
5. **Fork popular strategies**: The leaderboard's top strategies are battle-tested; fork and tweak instead of starting from scratch
6. **Optimize tool loading**: Use `--tools=` to load only needed groups and save context window
7. **Stream when interactive**: Use `+research` for user-facing queries; use atomic tools (`research_submit` / `poll` / `finalize`) for scripts
8. **Save backtest IDs**: Parse and store backtest IDs from output for later retrieval
9. **Test locally first**: Use `openfinclaw api GET` for direct API exploration before building workflows
10. **Monitor quotas**: Free tier has limits; cache results and batch queries when possible

## Additional Resources

- **Get API Key**: [hub.openfinclaw.ai](https://hub.openfinclaw.ai)
- **Browser Playground**: [hub.openfinclaw.ai/en/chat](https://hub.openfinclaw.ai/en/chat) (try before installing)
- **GitHub**: [github.com/wikidjon/ai-openclaw-cli](https://github.com/wikidjon/ai-openclaw-cli)
- **Platform Configs**: See `configs/` directory for more MCP examples
- **Comparison**: See `COMPARISON.md` for differences vs QuantConnect, Zipline, etc.
