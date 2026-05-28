---
name: searchcli-volcengine-ai-search
description: CLI for integrating Volcengine AI Search, recommendation, and conversational retrieval into agent and business systems
triggers:
  - integrate AI search into my application
  - set up Volcengine search CLI
  - configure AI-powered search and recommendations
  - onboard data to Volcengine AI Search
  - build conversational retrieval with SearchCLI
  - tune search quality with Viking
  - create item search with SearchCLI
  - run search evaluation and tuning
---

# SearchCLI (Volcengine AI Search)

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

SearchCLI is the open-source CLI for Volcengine AI Search. It provides stable, tunable search, recommendation, and conversational retrieval capabilities for agent systems and business applications. Built with TypeScript, it offers agent-friendly workflows with installable skills, dry-runs, confirmation gates, and automated quality tuning.

## What It Does

- **Item & Catalog Search**: Build search on structured business data
- **Recommendation Flows**: Connect recommendations to application scenes and user behavior
- **Conversational Retrieval**: Create RAG-style chat experiences grounded in application search
- **Agent Workflows**: Onboard data, configure applications, and validate behavior with reviewable steps
- **Automated Tuning**: Text-similarity evaluation and query tuning with LLM-based relevance judging

## Installation

### Requirements

- Node.js 20 or newer
- Git
- Volcengine AK/SK with AI Search access

### Install SearchCLI

```bash
git clone git@github.com:volcengine/SearchCLI.git vs
cd vs
bash ./scripts/install.sh
```

### Install as Agent Skills (for AI Agents)

```bash
npx skills add "git@github.com:volcengine/SearchCLI.git" -y -g
```

This installs bundled skills:
- `vs-shared`
- `vs-item-onboarding`
- `vs-search`
- `vs-search-tuning`
- `vs-chat`
- `vs-recommend`

## Authentication

### Using Environment Variables

If `VIKING_AK` and `VIKING_SK` are already in your environment:

```bash
vs auth import-env
vs auth status --json
```

### Interactive Login

```bash
vs auth login
vs auth status --json
vs doctor --json
```

### LLM Configuration (for Search Tuning)

For query generation and LLM-based relevance judging:

```bash
# Interactive setup
vs llm login
vs llm status --json

# Or import from environment
# Requires: VIKING_LLM_BASE_URL, VIKING_LLM_API_KEY, VIKING_LLM_MODEL
vs llm import-env
vs llm status --json
```

Verify LLM connectivity:

```bash
vs search tune llm-check --live --json
```

## Core Command Groups

### Authentication & Setup

```bash
vs auth login                    # Interactive authentication
vs auth import-env              # Import AK/SK from environment
vs auth status --json           # Check auth status
vs llm login                    # Configure LLM for tuning
vs llm import-env               # Import LLM config from env
vs doctor --json                # System health check
```

### Item Onboarding

```bash
vs item profile --file ./items.json --pretty
vs item plan --file ./items.json --goal "Build item search"
vs item apply --plan-dir ./.viking/item-plans/<plan> --dry-run
vs item apply --plan-dir ./.viking/item-plans/<plan> --confirm-review --wait-ready --run-trials
```

### Dataset Management

```bash
vs dataset create --data @dataset-create.json
vs dataset ingest --dataset-id <id> --fields @<file>
vs dataset list --json
vs dataset describe --dataset-id <id> --json
```

### Application Management

```bash
vs app list --json
vs app describe --app-id <id> --json
vs app create --data @app-config.json
vs app update --app-id <id> --data @update.json
```

### Search Operations

```bash
vs search run --app-id <id> --query "search term" --json
vs search tune query-generate --dataset-id <id> --output ./queries.json
vs search tune plan --queries @queries.json --dataset-id <id>
vs search tune run --plan-dir ./plan --output ./results.json
vs search tune report --results @results.json --pretty
```

### Recommendation

```bash
vs recommend run --app-id <id> --user-id <uid> --json
```

### Conversational Retrieval

```bash
vs chat run --app-id <id> --query "What are the best products?" --json
```

## Workflows

### Dataset + App Provisioning (Full Onboarding)

Complete workflow for creating both dataset and search application:

```bash
# 1. Profile your data
vs item profile --file ./items.json --pretty

# 2. Generate provisioning plan
vs item plan --file ./items.json --goal "Build product search"

# 3. Review plan with dry-run
vs item apply --plan-dir ./.viking/item-plans/<plan-id> --dry-run

# 4. Execute with confirmations and runtime verification
vs item apply \
  --plan-dir ./.viking/item-plans/<plan-id> \
  --confirm-review \
  --wait-ready \
  --run-trials
```

### Dataset-Only Provisioning

When you only need data ingestion without application setup:

```bash
# Generate dataset-only plan
vs item plan --file ./items.json --goal "Ingest product data" --skip-app

# Create dataset
vs dataset create --data @dataset-create.json

# Ingest data
vs dataset ingest --dataset-id <dataset-id> --fields @<normalized-items-artifact>
```

The `--skip-app` flag works with `vs item provision` and `vs item apply` as a guardrail.

### Video Dataset Provisioning

Always specify `--type video` explicitly for video datasets:

```bash
# Dataset + App
vs item profile --file ./videos.jsonl --type video --pretty
vs item plan --file ./videos.jsonl --type video --goal "Build video search"
vs item apply --plan-dir ./.viking/item-plans/<plan> --confirm-review --wait-ready

# Dataset-only
vs item plan --file ./videos.jsonl --type video --goal "Video data" --skip-app
vs dataset create --data @dataset-create.json
vs dataset ingest --dataset-id <id> --fields @<artifact>
```

**Important**: For video datasets, prefer `dataset-create.json` over `--schema @schema.json` to include `DataFieldConfig` and avoid `MissingParameter.DefaultFieldStrategy` errors.

### Search Quality Tuning

Automated evaluation and tuning workflow:

```bash
# 1. Generate evaluation queries
vs search tune query-generate \
  --dataset-id <dataset-id> \
  --output ./eval-queries.json

# 2. Create tuning plan
vs search tune plan \
  --queries @eval-queries.json \
  --dataset-id <dataset-id>

# 3. Run evaluation
vs search tune run \
  --plan-dir ./.viking/search-tune-plans/<plan-id> \
  --output ./results.json

# 4. Generate report
vs search tune report \
  --results @results.json \
  --pretty
```

## Code Examples

### TypeScript: Item Data Structure

```typescript
// items.json - Product catalog
[
  {
    "id": "prod-001",
    "name": "Wireless Headphones",
    "description": "High-quality Bluetooth headphones with noise cancellation",
    "category": "Electronics",
    "price": 149.99,
    "tags": ["audio", "bluetooth", "noise-canceling"],
    "in_stock": true
  },
  {
    "id": "prod-002",
    "name": "Smart Watch",
    "description": "Fitness tracking smartwatch with heart rate monitor",
    "category": "Wearables",
    "price": 299.99,
    "tags": ["fitness", "health", "smartwatch"],
    "in_stock": true
  }
]
```

### TypeScript: Dataset Create Payload

```typescript
// dataset-create.json
{
  "Name": "product-catalog-v1",
  "Type": "item",
  "Schema": {
    "Fields": [
      {
        "FieldName": "id",
        "FieldType": "String",
        "IsPrimaryKey": true
      },
      {
        "FieldName": "name",
        "FieldType": "String"
      },
      {
        "FieldName": "description",
        "FieldType": "Text"
      },
      {
        "FieldName": "price",
        "FieldType": "Float"
      },
      {
        "FieldName": "tags",
        "FieldType": "List<String>"
      }
    ]
  },
  "DataFieldConfig": {
    "DefaultFields": {
      "Title": "name",
      "Body": "description"
    }
  }
}
```

### TypeScript: Search Application Config

```typescript
// app-config.json
{
  "Name": "product-search-app",
  "DatasetId": "<dataset-id>",
  "SearchConfig": {
    "RankingStrategy": "TextSimilarity",
    "TextSimilarityConfig": {
      "Fields": ["name", "description"],
      "Weights": {
        "name": 2.0,
        "description": 1.0
      }
    }
  }
}
```

### Bash: Complete Onboarding Script

```bash
#!/bin/bash
set -e

# Environment check
vs doctor --json

# 1. Profile and plan
vs item profile --file ./products.json --pretty
vs item plan --file ./products.json --goal "E-commerce product search"

# Get the latest plan directory
PLAN_DIR=$(ls -dt ./.viking/item-plans/* | head -1)

# 2. Review with dry-run
vs item apply --plan-dir "$PLAN_DIR" --dry-run

# 3. Execute provisioning
vs item apply \
  --plan-dir "$PLAN_DIR" \
  --confirm-review \
  --wait-ready \
  --run-trials

echo "Onboarding complete. Check ./.viking/item-plans/ for artifacts."
```

### TypeScript: Programmatic Search

```typescript
import { execSync } from 'child_process';

function runSearch(appId: string, query: string): any {
  const result = execSync(
    `vs search run --app-id ${appId} --query "${query}" --json`,
    { encoding: 'utf-8' }
  );
  return JSON.parse(result);
}

function runRecommendation(appId: string, userId: string): any {
  const result = execSync(
    `vs recommend run --app-id ${appId} --user-id ${userId} --json`,
    { encoding: 'utf-8' }
  );
  return JSON.parse(result);
}

// Usage
const searchResults = runSearch('app-123', 'wireless headphones');
console.log('Search results:', searchResults);

const recommendations = runRecommendation('app-123', 'user-456');
console.log('Recommendations:', recommendations);
```

### TypeScript: Evaluation Query Generation

```typescript
// eval-queries.json structure
[
  {
    "query": "wireless bluetooth headphones",
    "expectedRelevance": {
      "prod-001": "high",
      "prod-003": "medium"
    }
  },
  {
    "query": "fitness tracking watch",
    "expectedRelevance": {
      "prod-002": "high"
    }
  }
]
```

## Configuration

### Environment Variables

```bash
# Required for authentication
export VIKING_AK="your-access-key"
export VIKING_SK="your-secret-key"

# Optional: LLM for search tuning
export VIKING_LLM_BASE_URL="https://api.openai.com/v1"
export VIKING_LLM_API_KEY="sk-..."
export VIKING_LLM_MODEL="gpt-4"

# Then import
vs auth import-env
vs llm import-env
```

### Credential Storage

SearchCLI stores credentials securely:
- **API Keys**: Local secure credential store (not plain text config)
- **LLM Base URL & Model**: Stored as non-secret config
- **Plan Artifacts**: `./.viking/` directory (safe to version control after removing sensitive data)

### Output Formats

Most commands support `--json` for machine-readable output:

```bash
vs auth status --json
vs dataset list --json
vs search run --app-id <id> --query "test" --json
```

Use `--pretty` for human-readable formatting:

```bash
vs item profile --file ./items.json --pretty
vs search tune report --results @results.json --pretty
```

## Common Patterns

### Incremental Data Updates

```bash
# Update existing dataset
vs dataset ingest --dataset-id <id> --fields @new-items.json --mode append

# Replace all data
vs dataset ingest --dataset-id <id> --fields @all-items.json --mode replace
```

### Multi-Stage Verification

```bash
# Always use dry-run first
vs item apply --plan-dir <plan> --dry-run

# Then execute with confirmations
vs item apply --plan-dir <plan> --confirm-review

# Add runtime trials for end-to-end verification
vs item apply --plan-dir <plan> --confirm-review --run-trials
```

### Search Result Validation

```bash
# Run search and save results
vs search run \
  --app-id <app-id> \
  --query "test query" \
  --json > search-results.json

# Use in scripts for validation
jq '.Results | length' search-results.json
```

### Skill-Based Agent Integration

```typescript
// Agent can invoke skills directly
await agent.useSkill('vs-item-onboarding', {
  file: './products.json',
  goal: 'Build product search',
  skipApp: false
});

await agent.useSkill('vs-search', {
  appId: 'app-123',
  query: 'wireless headphones'
});
```

## Troubleshooting

### Authentication Failures

```bash
# Check current auth status
vs auth status --json

# Re-authenticate
vs auth login

# Verify credentials are set
echo $VIKING_AK
echo $VIKING_SK

# Full diagnostic
vs doctor --json
```

### Dataset Creation Errors

**Error: `MissingParameter.DefaultFieldStrategy`** (common with video datasets)

```bash
# ❌ Don't use schema-only for video datasets
vs dataset create --name video-data --type video --schema @schema.json

# ✅ Use full create payload with DataFieldConfig
vs dataset create --data @dataset-create.json
```

**Solution**: Always use `dataset-create.json` from the plan artifact, which includes both `Schema` and `DataFieldConfig`.

### Item Ingestion Issues

```bash
# Verify dataset exists and is ready
vs dataset describe --dataset-id <id> --json

# Check item data format
vs item profile --file ./items.json --pretty

# Use normalized artifact from plan
vs dataset ingest --dataset-id <id> --fields @<normalized-artifact>
```

### LLM Connectivity Issues

```bash
# Check LLM configuration
vs llm status --json

# Test LLM connection
vs search tune llm-check --live --json

# Reconfigure if needed
vs llm login
```

### Plan Execution Failures

```bash
# Always start with dry-run
vs item apply --plan-dir <plan> --dry-run

# Check plan artifacts
ls -la ./.viking/item-plans/<plan>/

# Review plan.json for details
cat ./.viking/item-plans/<plan>/plan.json | jq .
```

### Search Tuning Not Working

```bash
# Verify LLM is configured
vs llm status --json
vs search tune llm-check --live --json

# Regenerate queries if format is wrong
vs search tune query-generate --dataset-id <id> --output ./new-queries.json

# Check plan directory structure
ls -la ./.viking/search-tune-plans/<plan>/
```

### Permission Errors

```bash
# Verify AK/SK has correct permissions
vs doctor --json

# Check specific resource access
vs dataset list --json
vs app list --json
```

### Agent Skill Installation

```bash
# List installed skills
vs skill list

# Reinstall all skills
vs skill install all

# Validate skill definitions
vs skill validate
```

## Best Practices

1. **Always use dry-run first**: `--dry-run` shows what will happen without executing
2. **Use confirmation gates**: `--confirm-review` adds human review step for critical operations
3. **Prefer dataset-create.json**: Especially for video datasets, use full payloads over schema-only
4. **Store credentials securely**: Never commit AK/SK to version control; use environment variables
5. **Version control plans**: The `.viking/` directory contains reproducible plans (clean sensitive data first)
6. **Use JSON output for automation**: `--json` flag provides machine-readable output
7. **Validate with runtime trials**: `--run-trials` ensures end-to-end functionality
8. **Type video datasets explicitly**: Always pass `--type video` for video data

## Related Resources

- [Agent Quick Start Guide](https://github.com/volcengine/SearchCLI/blob/main/docs/agent-quick-start.md)
- [Contributing Guidelines](https://github.com/volcengine/SearchCLI/blob/main/CONTRIBUTING.md)
- [Security Policy](https://github.com/volcengine/SearchCLI/blob/main/SECURITY.md)
- [Code of Conduct](https://github.com/volcengine/SearchCLI/blob/main/CODE_OF_CONDUCT.md)
