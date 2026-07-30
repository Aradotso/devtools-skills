---
name: soku-cli-integration
description: Install and use Soku CLI to give AI agents secure command-line access to marketing data, ad platforms, GA4, PostHog, and growth automation.
triggers:
  - install soku cli
  - query google ads data
  - access ga4 analytics
  - automate marketing tasks
  - publish seo content with soku
  - manage ad campaigns through soku
  - connect to posthog with soku
  - use soku for growth stack
---

# Soku CLI Integration

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Soku CLI is a command-line interface that provides secure, typed access to your entire growth stack—Google Ads, Meta Ads, GA4, PostHog, SEO hosting, and more—without exposing API keys in prompts or requiring MCP hosts. Commands are self-documenting and return stable JSON envelopes that AI agents can parse reliably. All delivery-changing writes (like creating campaigns) go through a human review gate.

**Key capabilities:**
- Query normalized reporting across ad platforms (Google, Meta, TikTok, ChatGPT Ads)
- Access GA4 and PostHog analytics
- Create and manage ad campaigns with human approval
- Publish SEO content
- Schedule recurring automations
- Manage workspace context files

## Installation

### Prerequisites
- Node.js 20 or newer
- npm or npx

### Global installation
```bash
npm install -g @soku-ai/cli
```

### Without global install
```bash
npx @soku-ai/cli --help
```

### Verify installation
```bash
soku --version
```

## Authentication & Workspace Setup

### Sign in with browser-based device flow
```bash
soku auth login
```

This opens a browser for OAuth authentication. The CLI stores credentials securely (using `keytar` when available, or encrypted file storage as fallback).

### Check authentication status
```bash
soku auth status
```

### Find and select a workspace
```bash
# Search for your brand workspace
soku workspace resolve <brand-name>

# Use the brand workspace
soku workspace use-brand <brand-name>

# Verify active workspace
soku workspace status
```

### Sign out
```bash
soku auth logout
```

## Core Command Structure

All commands follow a consistent pattern:
```bash
soku <namespace> <action> [options]
```

**Namespaces:**
- `auth` - Authentication management
- `workspace` - Organization/brand context
- `ads` - Advertising platforms
- `ga4` - Google Analytics 4
- `posthog` - PostHog analytics
- `seo-hosting` - SEO page management
- `automation` - Scheduled tasks
- `context` - Context Hub file management
- `egress` - Third-party API calls
- `review` - Human approval workflow
- `skill` - Agent skill management

### Get help for any command
```bash
soku --help
soku ads --help
soku ads query-single-dimension --help
```

## Working with Advertising Data

### List ad accounts
```bash
# Google Ads
soku ads list-ad-accounts --platform google

# Meta Ads
soku ads list-ad-accounts --platform meta

# All platforms
soku ads list-ad-accounts
```

### Query single-dimension reports
```bash
soku ads query-single-dimension \
  --platform google \
  --account-id 123-456-7890 \
  --dimension campaign \
  --date-start 2026-06-01 \
  --date-end 2026-06-30
```

**Available dimensions:** `campaign`, `ad_group`, `ad`, `keyword`

### Query two-dimension reports
```bash
soku ads query-two-dimension \
  --platform google \
  --account-id 123-456-7890 \
  --dimension-one campaign \
  --dimension-two ad_group \
  --date-start 2026-06-01 \
  --date-end 2026-06-30
```

### Google Ads GAQL queries
For custom breakdowns beyond standard dimensions:

```bash
soku ads google gaql \
  --account-id 123-456-7890 \
  --query "SELECT campaign.name, metrics.impressions, metrics.clicks FROM campaign WHERE segments.date DURING LAST_30_DAYS"
```

### Create a Meta campaign (with review)
```bash
soku ads meta campaign create \
  --account-id act_123456789 \
  --name "Q3 Product Launch" \
  --objective OUTCOME_TRAFFIC \
  --summary "Create paused Meta traffic campaign for Q3 launch"
```

This returns a review ID instead of executing immediately:
```json
{
  "ok": true,
  "data": {
    "review_id": "rev_abc123",
    "status": "pending"
  }
}
```

### Review and approve changes
```bash
# View the pending change
soku review show rev_abc123

# Approve it
soku review approve rev_abc123

# Or reject it
soku review reject rev_abc123 --reason "Budget needs adjustment"

# List all pending reviews
soku review list --status pending
```

## Analytics Integration

### Google Analytics 4

```bash
# List properties
soku ga4 list-properties

# Get property overview
soku ga4 get-property-overview --property-id 123456789

# List traffic sources
soku ga4 list-traffic-sources \
  --property-id 123456789 \
  --start-date 2026-06-01 \
  --end-date 2026-06-30

# Get conversion events
soku ga4 list-conversion-events --property-id 123456789
```

### PostHog

```bash
# List projects
soku posthog list-projects

# Execute SQL query
soku posthog query \
  --project-id 12345 \
  --tool execute-sql \
  --arguments '{"query":"SELECT event, count() as count FROM events WHERE timestamp >= now() - INTERVAL 7 DAY GROUP BY event ORDER BY count DESC LIMIT 10"}'

# Get insights
soku posthog query \
  --project-id 12345 \
  --tool get-insights \
  --arguments '{"filters":{"date_from":"-7d"}}'
```

## SEO Hosting

### Create and publish a page

```bash
# Stage a page
soku seo-hosting pages put \
  --section blog \
  --slug product-launch-2026 \
  --title "Product Launch Notes" \
  --html-file ./content/launch.html

# Publish it live
soku seo-hosting pages publish \
  --section blog \
  --slug product-launch-2026

# List all pages
soku seo-hosting pages list --section blog

# Get page details
soku seo-hosting pages get \
  --section blog \
  --slug product-launch-2026
```

### Manage domains

```bash
# List domains
soku seo-hosting domains list

# Add a domain
soku seo-hosting domains add \
  --domain blog.example.com \
  --section blog
```

## Automations

### Create scheduled tasks

```bash
# Weekly performance review
soku automation create \
  --name "Weekly ad account health check" \
  --prompt "Review all active ad accounts, identify campaigns with declining performance, and flag anomalies for human review" \
  --cron "0 9 * * 1" \
  --timezone America/Los_Angeles

# Daily budget monitor
soku automation create \
  --name "Daily budget utilization" \
  --prompt "Check yesterday's spend across all platforms and alert if any account spent >110% or <70% of daily budget" \
  --cron "0 8 * * *" \
  --timezone America/New_York
```

### Manage automations

```bash
# List all automations
soku automation list

# Get automation details
soku automation get --id auto_abc123

# Pause an automation
soku automation pause --id auto_abc123

# Resume an automation
soku automation resume --id auto_abc123

# Delete an automation
soku automation delete --id auto_abc123
```

## Context Hub

Manage files that agents can use as context:

```bash
# Upload a file
soku context upload ./campaign-brief.pdf --dir research

# Upload multiple files
soku context upload ./docs/*.md --dir documentation

# List files
soku context list

# List files in specific directory
soku context list --dir research

# Download a file
soku context download campaign-brief.pdf --output ./local-copy.pdf

# Delete a file
soku context delete campaign-brief.pdf --dir research
```

## Agent Skills

Install workflow skills to give agents structured knowledge:

```bash
# Install all available skills globally
soku skill install --all --global

# Install specific skill
soku skill install soku-ads-reporting --global

# List installed skills
soku skill list

# Check skill status
soku skill status

# Update skills
soku skill update --all
```

The meta skill is available at `skills/soku/SKILL.md` in the installation directory.

## JSON Output & Parsing

In non-interactive environments, all commands return structured JSON:

### Success response
```json
{
  "ok": true,
  "data": {
    "accounts": [
      {
        "id": "123-456-7890",
        "name": "Main Account",
        "platform": "google"
      }
    ]
  }
}
```

### Error response
```json
{
  "ok": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Session expired. Run 'soku auth login'"
  }
}
```

### Parsing in scripts (TypeScript)
```typescript
import { exec } from 'node:child_process';
import { promisify } from 'node:util';

const execAsync = promisify(exec);

async function getAdAccounts(platform: string) {
  try {
    const { stdout } = await execAsync(
      `soku ads list-ad-accounts --platform ${platform}`
    );
    const response = JSON.parse(stdout);
    
    if (!response.ok) {
      throw new Error(response.error.message);
    }
    
    return response.data.accounts;
  } catch (error) {
    console.error('Failed to fetch ad accounts:', error);
    throw error;
  }
}

// Usage
const accounts = await getAdAccounts('google');
console.log(`Found ${accounts.length} accounts`);
```

### Parsing in scripts (Shell)
```bash
#!/bin/bash

# Get accounts and extract IDs
ACCOUNTS=$(soku ads list-ad-accounts --platform google)

if [ "$(echo "$ACCOUNTS" | jq -r '.ok')" = "true" ]; then
  echo "$ACCOUNTS" | jq -r '.data.accounts[].id' | while read -r account_id; do
    echo "Processing account: $account_id"
    soku ads query-single-dimension \
      --platform google \
      --account-id "$account_id" \
      --dimension campaign \
      --date-start "2026-06-01" \
      --date-end "2026-06-30"
  done
else
  echo "Error: $(echo "$ACCOUNTS" | jq -r '.error.message')"
  exit 1
fi
```

## Direct Capability Calls

If a capability doesn't have a typed command yet, use `soku call`:

```bash
# Discover capabilities
soku call --help

# Call with snake_case capability name
soku call ads list_ad_accounts -p platform=google

# Multiple parameters
soku call ads query_single_dimension \
  -p platform=google \
  -p account_id=123-456-7890 \
  -p dimension=campaign \
  -p date_start=2026-06-01 \
  -p date_end=2026-06-30

# Get help for specific capability
soku call ads list_ad_accounts --help
```

## Common Workflows

### Weekly performance report
```typescript
async function generateWeeklyReport() {
  // Get all ad accounts
  const googleAccounts = await execSoku('ads list-ad-accounts --platform google');
  const metaAccounts = await execSoku('ads list-ad-accounts --platform meta');
  
  // Query last 7 days for each account
  const reports = [];
  for (const account of [...googleAccounts, ...metaAccounts]) {
    const data = await execSoku(`ads query-single-dimension \
      --platform ${account.platform} \
      --account-id ${account.id} \
      --dimension campaign \
      --date-start ${sevenDaysAgo()} \
      --date-end ${today()}`);
    
    reports.push({ account: account.name, data });
  }
  
  // Analyze and format
  return formatReport(reports);
}
```

### Campaign launch checklist
```typescript
async function launchCampaign(config: CampaignConfig) {
  // 1. Create campaign (returns review ID)
  const createResult = await execSoku(`ads meta campaign create \
    --account-id ${config.accountId} \
    --name "${config.name}" \
    --objective ${config.objective} \
    --summary "${config.summary}"`);
  
  const reviewId = createResult.data.review_id;
  
  // 2. Show review to user
  console.log('Campaign ready for review:');
  const review = await execSoku(`review show ${reviewId}`);
  console.log(JSON.stringify(review.data, null, 2));
  
  // 3. Human approves via CLI or returns approval decision
  const approved = await askHumanForApproval();
  
  if (approved) {
    await execSoku(`review approve ${reviewId}`);
    console.log('Campaign launched!');
  } else {
    await execSoku(`review reject ${reviewId} --reason "Needs budget adjustment"`);
  }
}
```

### Cross-platform performance comparison
```typescript
async function compareAdPlatforms(startDate: string, endDate: string) {
  const platforms = ['google', 'meta', 'tiktok'];
  const results = {};
  
  for (const platform of platforms) {
    const accounts = await execSoku(`ads list-ad-accounts --platform ${platform}`);
    
    let totalSpend = 0;
    let totalConversions = 0;
    
    for (const account of accounts.data.accounts) {
      const metrics = await execSoku(`ads query-single-dimension \
        --platform ${platform} \
        --account-id ${account.id} \
        --dimension campaign \
        --date-start ${startDate} \
        --date-end ${endDate}`);
      
      totalSpend += sumMetric(metrics.data, 'spend');
      totalConversions += sumMetric(metrics.data, 'conversions');
    }
    
    results[platform] = {
      spend: totalSpend,
      conversions: totalConversions,
      cpa: totalSpend / totalConversions
    };
  }
  
  return results;
}
```

## Troubleshooting

### Authentication issues

**Problem:** `Session expired` or `UNAUTHORIZED` errors
```bash
# Sign out and back in
soku auth logout
soku auth login

# Verify status
soku auth status
```

**Problem:** Browser doesn't open during login
```bash
# The CLI will display a URL to visit manually
# Copy the URL from the terminal and open it in your browser
```

### Workspace context issues

**Problem:** `No workspace selected`
```bash
# Check current workspace
soku workspace status

# List available workspaces
soku workspace list

# Select a workspace
soku workspace use-brand <brand-name>
```

### Command not found

**Problem:** Typed command doesn't exist yet for new capability
```bash
# Use direct capability call instead
soku call <namespace> <action> --help
soku call <namespace> <action> -p key=value
```

### Review workflow issues

**Problem:** Review ID returned but want to skip review (for testing)
```bash
# Reviews are intentional for delivery-changing operations
# Always approve/reject properly to maintain audit trail
soku review approve <review_id>
```

### JSON parsing issues

**Problem:** Malformed JSON output
```bash
# Ensure you're in non-interactive mode
# Interactive prompts can interfere with JSON output
# Set CI environment or redirect stderr
soku ads list-ad-accounts 2>/dev/null
```

### Permission issues

**Problem:** `FORBIDDEN` or missing capabilities
```bash
# Check workspace permissions
soku workspace status

# Ensure you're using correct workspace
soku workspace use-brand <correct-brand>

# Some capabilities require specific Soku plan features
# Contact workspace admin to verify feature availability
```

### Rate limiting

**Problem:** `RATE_LIMIT_EXCEEDED` errors
```bash
# Soku handles third-party rate limits
# Add delays between bulk operations
sleep 1
```

## Environment Variables

Soku CLI respects standard environment variables:

- `NODE_ENV` - Set to `production` to suppress dev warnings
- `CI` - Set to `true` to force non-interactive mode
- `NO_COLOR` - Set to disable colored output
- `SOKU_API_URL` - Override API endpoint (advanced use)

## Best Practices for AI Agents

1. **Always check authentication first**: Run `soku auth status` before executing workflows
2. **Parse JSON reliably**: Check `response.ok` before accessing `response.data`
3. **Use typed commands when available**: They provide better validation than raw `soku call`
4. **Handle reviews properly**: Never assume auto-approval; present reviews to humans
5. **Cache workspace context**: Don't repeatedly call `soku workspace status`
6. **Batch queries efficiently**: Minimize API calls by requesting broader date ranges
7. **Install skills globally**: Run `soku skill install --all --global` during setup
8. **Provide context in summaries**: The `--summary` flag for writes should explain WHY

## Development & Testing

When developing scripts that use Soku CLI:

```typescript
// Check if Soku CLI is available
import { exec } from 'node:child_process';

async function checkSokuCLI() {
  try {
    await execAsync('soku --version');
    return true;
  } catch {
    console.error('Soku CLI not found. Install: npm install -g @soku-ai/cli');
    return false;
  }
}

// Test with dry-run patterns
async function testCampaignCreate(config: any) {
  // Create returns review ID - safe to test
  const result = await execSoku(`ads meta campaign create \
    --account-id ${config.accountId} \
    --name "TEST ${config.name}" \
    --objective ${config.objective} \
    --summary "Test campaign - do not approve"`);
  
  // Review but don't approve
  const reviewId = result.data.review_id;
  console.log(`Test review created: ${reviewId}`);
  console.log('Remember to reject this review');
  
  return reviewId;
}
```

## Further Resources

- **Agent guide**: https://soku.ai/cli/skill.md
- **npm package**: https://www.npmjs.com/package/@soku-ai/cli
- **Contributing**: See CONTRIBUTING.md in the repository
- **GitHub**: https://github.com/About-Intelligence/soku-cli
