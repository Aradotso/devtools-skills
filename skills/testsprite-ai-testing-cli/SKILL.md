---
name: testsprite-ai-testing-cli
description: Official TestSprite CLI for AI-powered automated testing — create, run, and verify frontend & backend tests from terminal
triggers:
  - "run automated tests with TestSprite"
  - "create AI-powered E2E tests"
  - "verify my app with TestSprite CLI"
  - "set up TestSprite testing agent"
  - "get test failure details from TestSprite"
  - "integrate TestSprite into CI pipeline"
  - "rerun failed tests with TestSprite"
  - "configure TestSprite authentication"
---

# TestSprite AI Testing CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

TestSprite CLI is the command-line interface for TestSprite's AI-powered testing platform. It enables coding agents and developers to create, run, and verify automated tests against live applications (frontend with Playwright, backend API tests) — no mocks, real browsers and APIs in the cloud. The CLI is designed for agent-driven verification loops: create test → run → get failure bundle → fix → rerun.

## Installation

**Requirements:** Node.js ≥ 20

```bash
# Global installation (recommended)
npm install -g @testsprite/testsprite-cli

# Or use npx (no install)
npx @testsprite/testsprite-cli <command>
```

## Quick Start

### One-Shot Setup (Interactive)

```bash
testsprite init
```

This command:
- Prompts for your TestSprite API key (get one at https://www.testsprite.com)
- Stores credentials at `~/.testsprite/credentials`
- Verifies authentication
- Optionally installs agent skill files for your coding agent

### Non-Interactive Setup (CI/Scripts)

```bash
export TESTSPRITE_API_KEY=sk-your-key-here
testsprite init --from-env --yes --agent claude
```

### Manual Authentication

```bash
# Configure API key
testsprite auth configure

# Verify authentication
testsprite auth whoami

# Check status (alias for whoami)
testsprite status

# Logout (remove credentials)
testsprite auth logout
```

## Core Workflow: The Verification Loop

The CLI is designed for a test-driven agent loop:

### 1. Create a Test

```bash
# Frontend test (browser-based)
testsprite test create \
  --project proj_8f0f6 \
  --type frontend \
  --name "Checkout flow completes successfully" \
  --plan-from ./test-plans/checkout.plan.json

# Backend test (API-based)
testsprite test create \
  --project proj_8f0f6 \
  --type backend \
  --name "POST /api/orders creates order and returns 201" \
  --produces order_created \
  --category orders
```

### 2. Run the Test

```bash
# Create and run immediately, wait for results
testsprite test create \
  --project proj_8f0f6 \
  --type frontend \
  --plan-from ./checkout.plan.json \
  --run \
  --wait \
  --output json > result.json

# Check exit code: 0 = pass, 1 = fail, 2 = error
echo $?
```

### 3. Handle Failures (Agent Entry Point)

```bash
# Get ONE self-consistent failure bundle
testsprite test failure get test_3a9f21c7 \
  --out ./.testsprite/failure

# This downloads:
# - failure.json (failing step, neighbors, root cause hypothesis)
# - screenshots/*.png
# - dom-snapshots/*.html
# - test-source.ts (the generated test code)
```

**Example failure.json structure:**

```json
{
  "testId": "test_3a9f21c7",
  "runId": "run_a8f2e1b4",
  "status": "failed",
  "failingStep": {
    "index": 3,
    "action": "click",
    "selector": "#checkout-submit",
    "error": "Element not found: #checkout-submit"
  },
  "context": {
    "beforeStep": { "index": 2, "screenshot": "step-2.png" },
    "afterStep": { "index": 4, "screenshot": "step-4.png" }
  },
  "rootCauseHypothesis": "Button selector changed from #checkout-submit to #complete-order",
  "suggestedFix": "Update selector in checkout.plan.json line 18",
  "snapshotId": "snap_9f8e7d6c"
}
```

### 4. Rerun After Fix

```bash
# Replay the test (cheaper than full run for FE; BE runs with deps)
testsprite test rerun test_3a9f21c7 \
  --wait \
  --output json

# Exit code 0 = test now passes, banked into durable suite
```

## Project Management

```bash
# List all projects
testsprite project list --output json

# Get specific project
testsprite project get proj_8f0f6

# Create new project
testsprite project create \
  --name "E-commerce Frontend" \
  --description "Main storefront tests"

# Update project
testsprite project update proj_8f0f6 \
  --name "Updated Name"
```

## Test Management

### List & Retrieve Tests

```bash
# List all tests in a project
testsprite test list --project proj_8f0f6

# Get specific test details
testsprite test get test_3a9f21c7

# Get generated test code
testsprite test code get test_3a9f21c7 --out ./tests/checkout.spec.ts

# List test execution history
testsprite test result test_3a9f21c7 --history

# Get steps from latest run (with screenshot references)
testsprite test steps test_3a9f21c7
```

### Create Tests

**Frontend test with plan file:**

```bash
testsprite test create \
  --project proj_8f0f6 \
  --type frontend \
  --name "User can login successfully" \
  --plan-from ./plans/login.plan.json \
  --category authentication
```

**Backend test with dependencies:**

```bash
testsprite test create \
  --project proj_8f0f6 \
  --type backend \
  --name "GET /api/orders/:id returns order details" \
  --needs order_created \
  --category orders
```

**Batch create from file:**

```bash
# batch-tests.json contains array of test definitions
testsprite test create-batch --from ./batch-tests.json
```

### Update & Delete Tests

```bash
# Update test metadata
testsprite test update test_3a9f21c7 \
  --name "Updated test name" \
  --category checkout

# Update test plan (frontend tests)
testsprite test plan put test_3a9f21c7 \
  --from ./updated-plan.json

# Update test code (with etag for concurrency safety)
testsprite test code put test_3a9f21c7 \
  --from ./custom-test.ts \
  --etag "abc123def456"

# Soft-delete test
testsprite test delete test_3a9f21c7

# Batch delete
testsprite test delete-batch test_3a9f21c7 test_4b8c92d8
```

## Running Tests

### Single Test Run

```bash
# Trigger new run
testsprite test run test_3a9f21c7

# Run and wait for completion
testsprite test run test_3a9f21c7 --wait --output json

# Wait on an existing run
testsprite test wait run_a8f2e1b4
```

### Bulk Operations

```bash
# Run all tests in a project (respects dependency waves)
testsprite test run --all --project proj_8f0f6 --wait

# Rerun all tests in a project
testsprite test rerun --all --project proj_8f0f6 --wait
```

### Get Artifacts from Specific Run

```bash
# Download failure bundle for a specific runId
testsprite test artifact get run_a8f2e1b4 \
  --out ./.testsprite/runs/run_a8f2e1b4
```

## Test Plan Files (Frontend)

Frontend tests require a plan JSON file describing user actions:

**checkout.plan.json:**

```json
{
  "steps": [
    {
      "action": "navigate",
      "url": "https://example.com/cart"
    },
    {
      "action": "click",
      "selector": "#proceed-to-checkout"
    },
    {
      "action": "fill",
      "selector": "#email",
      "value": "test@example.com"
    },
    {
      "action": "fill",
      "selector": "#card-number",
      "value": "4242424242424242"
    },
    {
      "action": "click",
      "selector": "#submit-payment"
    },
    {
      "action": "assert",
      "type": "visible",
      "selector": "#order-confirmation",
      "message": "Order confirmation should appear"
    }
  ]
}
```

**Available actions:** `navigate`, `click`, `fill`, `select`, `check`, `uncheck`, `hover`, `wait`, `assert`

## Agent Integration

Install skill files for your coding agent:

```bash
# Install for Claude Code
testsprite agent install claude

# Install for Cursor
testsprite agent install cursor

# Install for Cline
testsprite agent install cline

# Install for Codex
testsprite agent install codex

# Install for Antigravity
testsprite agent install antigravity

# List available agent targets
testsprite agent list
```

This creates skill/instruction files in your project so the agent can autonomously drive the verification loop.

## Configuration & Profiles

### Credentials File

Location: `~/.testsprite/credentials`

```json
{
  "profiles": {
    "default": {
      "apiKey": "sk-your-key-here",
      "environment": "production"
    },
    "staging": {
      "apiKey": "sk-staging-key",
      "environment": "staging"
    }
  },
  "activeProfile": "default"
}
```

### Environment Variables

```bash
# API key (overrides credentials file)
export TESTSPRITE_API_KEY=sk-your-key-here

# API base URL (override default)
export TESTSPRITE_API_URL=https://api.testsprite.com

# Profile selection
export TESTSPRITE_PROFILE=staging

# Non-interactive init
export TESTSPRITE_API_KEY=sk-key
testsprite init --from-env --yes --agent claude
```

### Output Formats

```bash
# Human-readable (default)
testsprite test get test_3a9f21c7

# JSON (for scripting/parsing)
testsprite test get test_3a9f21c7 --output json

# Quiet (suppress non-essential output)
testsprite test run test_3a9f21c7 --quiet
```

## Exit Codes

The CLI uses consistent exit codes for scripting:

- **0** - Success (test passed, operation succeeded)
- **1** - Test failed (assertions failed, test did not pass)
- **2** - Error (API error, invalid arguments, network issue)

**Example script usage:**

```bash
#!/bin/bash
testsprite test rerun test_3a9f21c7 --wait --output json > result.json

if [ $? -eq 0 ]; then
  echo "✅ Test passed"
elif [ $? -eq 1 ]; then
  echo "❌ Test failed - getting failure details"
  testsprite test failure get test_3a9f21c7 --out ./failures
else
  echo "⚠️ Error running test"
fi
```

## Common Agent Patterns

### Pattern 1: New Feature Verification

```typescript
// Agent creates new feature, then verifies it

// 1. Create test for new behavior
const createResult = await execCommand(`
  testsprite test create \
    --project ${projectId} \
    --type frontend \
    --name "New dashboard widget loads data" \
    --plan-from ./test-plans/dashboard-widget.plan.json \
    --run \
    --wait \
    --output json
`);

if (createResult.exitCode === 1) {
  // 2. Get failure details
  await execCommand(`
    testsprite test failure get ${testId} \
      --out ./.testsprite/failure
  `);
  
  // 3. Read failure bundle, fix code, rerun
  const failureData = await readJson('./.testsprite/failure/failure.json');
  await fixCodeBasedOnFailure(failureData);
  
  await execCommand(`
    testsprite test rerun ${testId} --wait --output json
  `);
}
```

### Pattern 2: Regression Guard

```typescript
// Agent changes existing code, reruns all related tests

// 1. Identify affected tests by category
const testsResult = await execCommand(`
  testsprite test list \
    --project ${projectId} \
    --output json
`);

const affectedTests = JSON.parse(testsResult.stdout)
  .filter(t => t.category === 'checkout');

// 2. Rerun all affected tests
for (const test of affectedTests) {
  const result = await execCommand(`
    testsprite test rerun ${test.id} --wait --output json
  `);
  
  if (result.exitCode === 1) {
    // Regression detected - revert or fix
    await handleRegression(test.id);
  }
}
```

### Pattern 3: CI Integration

```bash
#!/bin/bash
# .github/workflows/testsprite.yml

set -e

# Run all tests in project
testsprite test rerun --all --project $PROJECT_ID --wait --output json > results.json

# Parse results
FAILED=$(jq '[.[] | select(.status == "failed")] | length' results.json)

if [ "$FAILED" -gt 0 ]; then
  echo "❌ $FAILED test(s) failed"
  
  # Download failure artifacts for each failed test
  jq -r '.[] | select(.status == "failed") | .id' results.json | while read testId; do
    testsprite test failure get "$testId" --out "./artifacts/$testId"
  done
  
  exit 1
fi

echo "✅ All tests passed"
```

## Common Issues & Solutions

### Authentication Errors

```bash
# Problem: "Invalid API key" or 401 errors
# Solution: Verify key and re-authenticate

testsprite auth whoami  # Check current auth status
testsprite auth logout  # Clear stored credentials
testsprite auth configure  # Re-enter API key
```

### Test Creation Fails

```bash
# Problem: "Invalid plan format" for frontend tests
# Solution: Validate plan JSON structure

# Ensure plan has required fields:
{
  "steps": [
    { "action": "navigate", "url": "..." },
    // ... more steps
  ]
}

# Use --dry-run to validate locally
testsprite test create --plan-from ./plan.json --dry-run
```

### Tests Stuck in "running" Status

```bash
# Problem: test --wait never completes
# Solution: Check run status separately

testsprite test result test_abc123
testsprite test steps test_abc123  # See which step is executing

# Cancel if needed (via web dashboard)
```

### Missing Failure Artifacts

```bash
# Problem: failure get returns empty or incomplete bundle
# Solution: Ensure test has actually failed in latest run

# Check latest result
testsprite test result test_abc123

# If status is not "failed", no failure bundle exists
# For older run, use artifact get with specific runId
testsprite test artifact get run_xyz789 --out ./failures
```

### Network/Timeout Issues

```bash
# Problem: Connection timeouts or network errors
# Solution: Check API URL and network connectivity

# Verify base URL
echo $TESTSPRITE_API_URL

# Test connectivity
curl -I https://api.testsprite.com/health

# Use verbose logging
testsprite test run test_abc123 --verbose
```

## Advanced Usage

### Dependency Management (Backend Tests)

Backend tests can declare what data they produce/need:

```bash
# Test that creates an order (produces order_created)
testsprite test create \
  --project proj_8f0f6 \
  --type backend \
  --name "Create order" \
  --produces order_created \
  --category orders

# Test that needs an existing order (needs order_created)
testsprite test create \
  --project proj_8f0f6 \
  --type backend \
  --name "Get order details" \
  --needs order_created \
  --category orders

# When rerunning, TestSprite runs in dependency wave order
testsprite test rerun --all --project proj_8f0f6
```

### Using --dry-run for Local Validation

```bash
# Test command structure without hitting API
testsprite test create \
  --project proj_8f0f6 \
  --type frontend \
  --plan-from ./plan.json \
  --dry-run

# Validates:
# - Command syntax
# - Plan file structure
# - Required flags present
# Returns canned success data
```

### Scripting with JSON Output

```bash
# Get test ID from creation
TEST_ID=$(testsprite test create \
  --project proj_8f0f6 \
  --type frontend \
  --plan-from ./plan.json \
  --output json | jq -r '.id')

# Use in subsequent commands
testsprite test run $TEST_ID --wait

# Parse batch results
testsprite test list --project proj_8f0f6 --output json | \
  jq '.[] | select(.status == "failed") | .id'
```

## Resources

- **Official Website:** https://www.testsprite.com
- **Documentation:** https://www.testsprite.com/docs
- **API Keys:** https://www.testsprite.com (sign up for account)
- **GitHub:** https://github.com/TestSprite/testsprite-cli
- **Discord:** https://discord.gg/W4JDrZfdB
- **Leaderboard:** https://codercup.ai (real-world benchmark)

## License

Apache-2.0 — see LICENSE file in repository
