```markdown
---
name: cliare-cli-agent-readiness
description: Audit command-line interfaces for agent readiness with runtime measurement and command-shape inference
triggers:
  - measure CLI agent readiness
  - audit command-line interface for agents
  - generate CLI command index
  - check CLI help coverage
  - detect CLI side effects
  - evaluate CLI for AI agents
  - measure CLI output contracts
  - generate CLI skill documentation
---

# CLIARE CLI Agent Readiness Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

CLIARE audits command-line interfaces for agent readiness by probing runtime behavior, inferring command shapes, detecting side effects, and generating command indexes that agents can use instead of rediscovering CLI surfaces through trial and error.

## What CLIARE Does

CLIARE measures released CLI binaries as black boxes to answer:

- Which commands actually exist?
- Which flags and positionals are safe to use?
- Which commands have parseable JSON/YAML output?
- Which paths require auth, network, fixtures, or daemons?
- Which "safe" discovery commands quietly write files?

It produces command indexes, issue ledgers, scorecards, agent skills, and CI artifacts.

## Installation

### From crates.io

```bash
cargo install cliare
cliare metadata --format text
```

### From GitHub Releases

```bash
curl -fsSL https://github.com/modiqo/cliare/releases/latest/download/install.sh | sh
```

The installer detects your platform and installs to `$HOME/.local/bin` by default.

### Custom Installation Directory

```bash
curl -fsSL https://github.com/modiqo/cliare/releases/latest/download/install.sh | CLIARE_INSTALL_DIR=/usr/local/bin sh
```

### From Source

```bash
git clone https://github.com/modiqo/cliare.git
cd cliare
cargo install --path .
```

## Core Commands

### Measure a CLI

Run a standard measurement profile on a CLI binary:

```bash
cliare measure mycli --out .cliare/mycli --profile standard --refresh
```

This probes the CLI, records evidence, infers command shapes, detects side effects, and writes artifacts to `.cliare/mycli/`.

### Deep Measurement with Limits

```bash
cliare measure supabase \
  --out .cliare/supabase \
  --profile deep \
  --max-depth 12 \
  --max-probes 5000 \
  --concurrency 8 \
  --refresh
```

- `--profile deep`: More thorough probing (vs `standard`, `quick`)
- `--max-depth`: Maximum command nesting depth
- `--max-probes`: Total probe budget
- `--concurrency`: Parallel probe workers
- `--refresh`: Force new measurement even if artifacts exist

### Context-Aware Measurement

Measure in specific runtime contexts:

```bash
cliare measure mycli \
  --out .cliare/mycli \
  --context authenticated \
  --auth-state present \
  --execution-mode host \
  --profile standard \
  --refresh
```

Available contexts: `clean`, `repository`, `authenticated`, `host`, `fixture-backed`, `ci`

### View Summary

Get a concise assessment of the measurement:

```bash
cliare summary --out .cliare/mycli
```

JSON format for programmatic use:

```bash
cliare summary --out .cliare/mycli --format json
```

## Role-Specific Reports

### Maintainer Report

Get a prioritized fix queue for CLI maintainers:

```bash
cliare report maintainer --out .cliare/mycli --format markdown
```

Shows command drift, help coverage gaps, diagnostic clarity issues, and unsafe discovery behavior.

### Harness Report

Generate agent harness artifacts and command index:

```bash
cliare report harness --out .cliare/mycli --write
```

Creates `persona-harness.md` and ensures `command-index.json` is ready for agent use.

### Security Report

Review side effects, auth behavior, and approval requirements:

```bash
cliare report security --out .cliare/mycli --format markdown
```

Highlights undocumented filesystem writes, network calls, and credential access.

## Command Surface Queries

### Query Intent to Command

Route natural language intent to specific commands:

```bash
cliare surface query "deploy my application" --out .cliare/mycli
```

Returns the best-matching command with confidence score.

### Explain a Command

Get detailed readiness, operands, and output contract for one command:

```bash
cliare surface explain "mycli deploy --format json" --out .cliare/mycli
```

### List Commands

Filter measured command paths:

```bash
# All commands
cliare surface list --out .cliare/mycli

# Only high-confidence commands
cliare surface list --out .cliare/mycli --min-confidence 0.8

# Commands with JSON output
cliare surface list --out .cliare/mycli --has-json-output

# Commands suitable for autonomous agents
cliare surface list --out .cliare/mycli --agent-suitable
```

## Issue Management

### List Issues

View all detected issues:

```bash
cliare issues list --out .cliare/mycli --format markdown
```

Human-readable format:

```bash
cliare issues list --out .cliare/mycli --format human
```

JSON for tooling:

```bash
cliare issues list --out .cliare/mycli --format json
```

## Metadata and Versioning

```bash
# Show CLIARE version and build info
cliare metadata --format text

# JSON format
cliare metadata --format json
```

## Output Artifacts

A measurement writes to `.cliare/<target-cli>/`:

```
command-index.json      # Agent-facing command catalog
command-index.md        # Human-readable command catalog
AGENT_SKILL.md          # Generated skill for agents
scorecard.json          # Readiness scores and coverage
issues.json             # Issue ledger
issues.md               # Human issue report
persona-maintainer.md   # Maintainer action plan
persona-harness.md      # Agent harness guide
persona-security.md     # Security review packet
evidence.jsonl          # Raw probe evidence
shape.json              # Inferred command shape
findings.sarif          # CI/security upload format
junit.xml               # CI test result format
```

## CI Integration

### GitHub Actions

```yaml
name: CLI Agent Readiness

on:
  push:
    branches: [main]
  pull_request:

jobs:
  cliare:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install CLIARE
        run: |
          curl -fsSL https://github.com/modiqo/cliare/releases/latest/download/install.sh | sh
          echo "$HOME/.local/bin" >> $GITHUB_PATH
      
      - name: Build CLI
        run: cargo build --release
      
      - name: Measure CLI
        run: |
          cliare measure ./target/release/mycli \
            --out .cliare/mycli \
            --profile standard \
            --refresh
      
      - name: Generate Reports
        run: |
          cliare summary --out .cliare/mycli --format markdown > summary.md
          cliare report maintainer --out .cliare/mycli --format markdown > maintainer-report.md
      
      - name: Upload Artifacts
        uses: actions/upload-artifact@v3
        with:
          name: cliare-artifacts
          path: .cliare/mycli/
      
      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: .cliare/mycli/findings.sarif
```

## Common Patterns

### Measure Multiple CLIs

```bash
#!/bin/bash
for cli in kubectl helm terraform aws; do
  cliare measure "$cli" \
    --out ".cliare/$cli" \
    --profile standard \
    --refresh
  cliare summary --out ".cliare/$cli" > "reports/$cli-summary.md"
done
```

### Pre-Release Gate

```bash
#!/bin/bash
# Measure the CLI
cliare measure ./target/release/mycli \
  --out .cliare/mycli \
  --profile deep \
  --refresh

# Get scorecard
SCORE=$(cliare summary --out .cliare/mycli --format json | jq -r '.scorecard.overall_score')

# Fail if score below threshold
if (( $(echo "$SCORE < 0.75" | bc -l) )); then
  echo "Agent readiness score $SCORE below 0.75 threshold"
  cliare issues list --out .cliare/mycli --format human
  exit 1
fi
```

### Extract High-Confidence Commands

```bash
cliare surface list \
  --out .cliare/mycli \
  --min-confidence 0.9 \
  --format json | jq -r '.[].command_path'
```

### Check for Unsafe Discovery Commands

```bash
# Security review: which "safe" commands write files?
cliare report security \
  --out .cliare/mycli \
  --format json | jq '.side_effects.unexpected_writes'
```

## Rust Library Usage

While CLIARE is primarily a CLI tool, you can use it as a library:

```rust
// Example: Parse command-index.json artifacts
use serde_json::Value;
use std::fs;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let index_path = ".cliare/mycli/command-index.json";
    let content = fs::read_to_string(index_path)?;
    let index: Value = serde_json::from_str(&content)?;
    
    // Extract commands with JSON output
    if let Some(commands) = index["commands"].as_array() {
        for cmd in commands {
            if let Some(output) = cmd["output_contract"]["formats"].as_array() {
                if output.iter().any(|f| f == "json") {
                    println!("Command with JSON: {}", cmd["command_path"]);
                }
            }
        }
    }
    
    Ok(())
}
```

## Configuration

CLIARE uses command-line flags for configuration. Key options:

### Measurement Profiles

- `--profile quick`: Fast surface scan (depth 3, ~100 probes)
- `--profile standard`: Balanced coverage (depth 6, ~500 probes)
- `--profile deep`: Thorough exploration (depth 12, ~5000 probes)

### Execution Modes

- `--execution-mode container`: Run in isolated container
- `--execution-mode host`: Run directly on host
- `--execution-mode sandbox`: Run in filesystem sandbox

### Context Selection

- `--context clean`: Minimal state
- `--context repository`: Inside git repository
- `--context authenticated`: With auth credentials
- `--context fixture-backed`: With test fixtures
- `--context ci`: CI environment simulation

## Troubleshooting

### Measurement Hangs

If probes hang, reduce concurrency and add timeout:

```bash
cliare measure mycli \
  --out .cliare/mycli \
  --concurrency 1 \
  --probe-timeout 5s \
  --profile quick
```

### Too Many Probes

For CLIs with huge surfaces, limit probe budget:

```bash
cliare measure kubectl \
  --out .cliare/kubectl \
  --max-depth 4 \
  --max-probes 1000 \
  --profile standard
```

### Missing Commands in Index

Increase depth or switch to deep profile:

```bash
cliare measure mycli \
  --out .cliare/mycli \
  --profile deep \
  --max-depth 12
```

### Permission Errors

Run with appropriate context:

```bash
# If CLI requires auth
cliare measure mycli \
  --context authenticated \
  --auth-state present \
  --out .cliare/mycli
```

### Artifacts Not Generated

Use `--refresh` to force regeneration:

```bash
cliare measure mycli --out .cliare/mycli --refresh
```

Check artifacts were written:

```bash
ls -la .cliare/mycli/
cat .cliare/mycli/command-index.json | jq .
```

## Agent Harness Integration

### Loading Command Index

```python
import json

# Load the command index
with open('.cliare/mycli/command-index.json') as f:
    index = json.load(f)

# Find commands suitable for autonomous use
suitable = [
    cmd for cmd in index['commands']
    if cmd.get('agent_suitability', {}).get('autonomous', False)
]

# Route intent to command
def find_command(intent: str) -> dict:
    # Use embeddings or simple matching
    for cmd in suitable:
        if intent.lower() in cmd['summary'].lower():
            return cmd
    return None
```

### Query Before Execution

```bash
# Before running a command, check its readiness
cliare surface explain "mycli deploy --json" --out .cliare/mycli --format json
```

Use the response to verify:
- Confidence level
- Required preconditions (auth, network, fixtures)
- Expected output format
- Known side effects

## Best Practices

1. **Measure on each release**: Re-run CLIARE when the CLI changes
2. **Use deep profile for release gates**: Quick for development, deep for CI
3. **Review security reports**: Check unexpected side effects before agent exposure
4. **Version command indexes**: Store them alongside release artifacts
5. **Set minimum confidence thresholds**: Filter low-confidence commands in production
6. **Combine skills with indexes**: Skills teach intent, indexes map runtime reality
7. **Check preconditions**: Use command index to verify auth/network before execution
8. **Prefer JSON output**: Route through commands with `--json` or `--format json` when available

## Environment Variables

```bash
# Override installation directory
export CLIARE_INSTALL_DIR=/usr/local/bin

# Specify CLIARE version for installer
export CLIARE_VERSION=v0.1.9

# Set default output directory
export CLIARE_OUT_DIR=.cliare
```

## Examples by Use Case

### CLI Maintainer: Pre-Release Check

```bash
# Build and measure
cargo build --release
cliare measure ./target/release/mycli \
  --out .cliare/mycli \
  --profile deep \
  --refresh

# Review issues
cliare report maintainer --out .cliare/mycli

# Check score
cliare summary --out .cliare/mycli --format json | jq '.scorecard.overall_score'
```

### Agent Harness: Route Intent to Command

```bash
# Query for deployment command
cliare surface query "deploy application to production" \
  --out .cliare/mycli \
  --format json

# Get full command details
cliare surface explain "mycli deploy" \
  --out .cliare/mycli \
  --format json | jq '.preconditions, .output_contract'
```

### Security Reviewer: Audit Side Effects

```bash
cliare measure mycli \
  --out .cliare/mycli \
  --profile deep \
  --refresh

cliare report security \
  --out .cliare/mycli \
  --format markdown > security-review.md
```

### DevOps: Multi-CLI Scorecard

```bash
#!/bin/bash
echo "Tool,Score,Commands,Issues" > scorecard.csv

for cli in kubectl helm terraform aws gcloud; do
  cliare measure "$cli" --out ".cliare/$cli" --profile standard --refresh
  SUMMARY=$(cliare summary --out ".cliare/$cli" --format json)
  
  SCORE=$(echo "$SUMMARY" | jq -r '.scorecard.overall_score')
  COMMANDS=$(echo "$SUMMARY" | jq -r '.command_count')
  ISSUES=$(echo "$SUMMARY" | jq -r '.issue_count')
  
  echo "$cli,$SCORE,$COMMANDS,$ISSUES" >> scorecard.csv
done
```

## Further Reading

- Homepage: https://cliare.sh
- Repository: https://github.com/modiqo/cliare
- License: Apache-2.0
```
