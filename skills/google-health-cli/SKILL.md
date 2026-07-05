---
name: google-health-cli
description: CLI for Google Health API v4 — read steps, heart rate, exercise, sleep, weight, and 35+ health data types with agent-first JSON output
triggers:
  - get my step count from google health
  - fetch heart rate data from google fit
  - read my sleep data using google health api
  - export exercise data from google health
  - set up google health cli authentication
  - query my weight history from google health
  - retrieve daily health metrics from google fit
  - import health data using ghealth command
---

# google-health-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

CLI for the [Google Health API v4](https://developers.google.com/health) built for AI agents and developers. Supports 40+ verified health data types including steps, heart rate, exercise, sleep, weight, SpO2, HRV, ECG, blood glucose, and nutrition. Outputs deterministic JSON with agent-friendly exit codes.

## Installation

```bash
git clone https://github.com/Google-Health-API/google-health-cli.git
cd google-health-cli
go build -o ghealth .
```

Move the binary to your PATH:

```bash
sudo mv ghealth /usr/local/bin/
```

## Authentication Setup

### Interactive Setup

```bash
ghealth setup
```

Walks through:
1. GCP project ID
2. OAuth credentials (download from [Console](https://console.cloud.google.com/apis/credentials) as Desktop application)
3. Health API enablement
4. Scope selection
5. Browser-based OAuth login

Credentials stored in `~/.config/ghealth/` (override with `GHEALTH_CONFIG_DIR`):
- `client_secret.json` — OAuth client (mode 0600)
- `credentials.json` — access + refresh tokens (mode 0600)
- `config.toml` — active profile

### Non-Interactive Setup (CI/Agents)

```bash
ghealth setup \
  --project-id $GCP_PROJECT_ID \
  --client-secret ~/Downloads/client_secret.json \
  --scopes-preset readonly \
  --skip-enable-api \
  --no-prompt
```

### Headless OAuth Flow

```bash
# 1. Start non-interactive login
ghealth auth login --non-interactive --scopes-preset readonly
# Returns JSON with auth_url and complete_command

# 2. Open auth_url in browser, copy the redirect URL or code parameter

# 3. Complete authentication
ghealth auth login --complete 'http://localhost/?code=4/0AX4XfWh...&state=cQq...'
# OR just the code:
ghealth auth login --complete 4/0AX4XfWh...
```

### Environment Variables

```bash
# Use existing access token
export GHEALTH_ACCESS_TOKEN=ya29...

# Use custom credentials file
export GHEALTH_CREDENTIALS_FILE=/path/to/creds.json

# Custom config directory
export GHEALTH_CONFIG_DIR=/custom/config/path
```

Precedence: `GHEALTH_ACCESS_TOKEN` > `GHEALTH_CREDENTIALS_FILE` > stored credentials > Application Default Credentials.

### Moving Tokens Between Machines

```bash
# Export on source machine
ghealth auth export > /tmp/ghealth-creds.json

# Import on target machine (requires client_secret.json)
ghealth auth import --file /tmp/ghealth-creds.json
```

## Key Data Types

| Type | Key Values | Operations |
|------|-----------|------------|
| `steps` | `countSum` (daily rollup) | list, rollup, daily-rollup, reconcile |
| `heart-rate` | `beatsPerMinute` | list, rollup, daily-rollup, reconcile |
| `exercise` | type, duration, calories, avgHeartRate | list, get, create, update, delete, export-tcx |
| `sleep` | minutesAsleep, minutesAwake, stageMinutes | list, get, create, update, delete, reconcile |
| `weight` | `weightGrams` | list, get, create, update, delete, rollup, daily-rollup |
| `distance` | `millimetersSum` (daily rollup) | list, rollup, daily-rollup, reconcile |
| `oxygen-saturation` | `percentage` (SpO2) | list, reconcile |
| `heart-rate-variability` | RMSSD | list, reconcile |
| `blood-glucose` | mg/dL, mealType, timing | list, get, rollup, daily-rollup, reconcile |
| `nutrition-log` | nutrients, energy, mealType | list, get, rollup, daily-rollup, reconcile |

Run `ghealth schema types` for all 40+ types.

## Reading Data

### Sample Data (Individual Readings)

```bash
# Recent heart rate readings
ghealth data heart-rate list --from today --limit 10

# Heart rate for specific date range
ghealth data heart-rate list --from 2026-03-22 --to 2026-03-29 --limit 100

# Oxygen saturation readings
ghealth data oxygen-saturation list --from 2026-06-01 --limit 20

# Recent weight measurements
ghealth data weight list --limit 10
```

### Rollup Data (Aggregated)

**Critical**: For steps, distance, and other interval types, use `daily-rollup` to get actual values. `list` returns time intervals without totals.

```bash
# Daily step totals (CORRECT way)
ghealth data steps daily-rollup --from 2026-03-22 --to 2026-03-29
# Returns: {"dataPoints": [{"date": "2026-03-28", "countSum": "9037"}, ...]}

# Daily distance totals
ghealth data distance daily-rollup --from 2026-03-01 --to 2026-03-31

# Weekly average heart rate
ghealth data heart-rate rollup --from 2026-03-01 --to 2026-03-31 --duration 7d

# Hourly step counts for today
ghealth data steps rollup --from today --duration 1h
```

### Exercise Data

```bash
# List recent exercises
ghealth data exercise list --from 2026-03-01 --limit 20

# Get specific exercise
ghealth data exercise get --id <exercise-id>

# Export exercise track as TCX
ghealth data exercise export-tcx --id <id> --output ride.tcx

# Export as CSV for data analysis
ghealth data exercise export-tcx --id <id> --output ride.csv --as csv
# CSV columns: time, activity, lap, sport, latitude_deg, longitude_deg, altitude_m, 
#              distance_m, heart_rate_bpm, cadence_rpm, speed_mps, watts

# Stream to stdout
ghealth data exercise export-tcx --id <id> --output - --as csv | head
```

### Sleep Data

```bash
# Recent sleep sessions (summary)
ghealth data sleep list --limit 5

# Sleep with stage-by-stage breakdown
ghealth data sleep list --limit 5 --detail

# Specific sleep session
ghealth data sleep get --id <sleep-id>
```

### Pagination

All `list` commands support pagination:

```bash
# First page (default limit 500)
ghealth data heart-rate list --from 2026-06-15 --limit 500
# Returns: {"dataPoints": [...], "nextPageToken": "ABC"}

# Next page
ghealth data heart-rate list --from 2026-06-15 --limit 500 --page-token ABC
```

## Writing Data

### Create Exercise

```bash
ghealth data exercise create \
  --start-time 2026-03-28T14:00:00Z \
  --end-time 2026-03-28T15:30:00Z \
  --type running \
  --calories 450 \
  --distance-meters 8000 \
  --avg-heart-rate 152 \
  --notes "Morning run in the park"
```

### Update Exercise

```bash
ghealth data exercise update \
  --id <exercise-id> \
  --notes "Evening run - felt great" \
  --calories 475
```

### Create Sleep Session

```bash
ghealth data sleep create \
  --start-time 2026-03-27T23:00:00Z \
  --end-time 2026-03-28T07:00:00Z \
  --minutes-asleep 450 \
  --minutes-awake 30
```

### Log Weight

```bash
ghealth data weight create \
  --timestamp 2026-03-28T08:00:00Z \
  --weight-grams 75000
```

### Delete Data

```bash
ghealth data exercise delete --id <exercise-id>
ghealth data sleep delete --id <sleep-id>
ghealth data weight delete --id <weight-id>
```

## Agent-Friendly Patterns

### Deterministic JSON Output

Every read command returns:

```json
{
  "dataPoints": [
    {"date": "2026-03-28", "countSum": "9037"},
    {"date": "2026-03-27", "countSum": "8245"}
  ],
  "_hints": {
    "type": "steps",
    "rollupDuration": "1d"
  },
  "nextPageToken": "optional-token-if-more-data"
}
```

Parse with `jq`:

```bash
# Extract step counts
ghealth data steps daily-rollup --from 2026-03-22 --to 2026-03-29 | jq -r '.dataPoints[].countSum'

# Get total steps for week
ghealth data steps daily-rollup --from 2026-03-22 --to 2026-03-29 | jq '[.dataPoints[].countSum | tonumber] | add'

# Average heart rate
ghealth data heart-rate list --from today --limit 100 | jq '[.dataPoints[].beatsPerMinute | tonumber] | add / length'
```

### Dry Run Mode

```bash
# Preview API request without executing
ghealth data steps daily-rollup --from today --dry-run
```

### Raw API Response

```bash
# Get unprocessed API response
ghealth data heart-rate list --from today --limit 10 --raw
```

### Exit Codes

- `0` — Success
- `1` — General error
- `2` — Authentication/validation error
- `3` — API error
- `4` — Data not found
- `5` — Configuration error

## Critical Data Handling Rules

### Missing Days vs. Zero Values

For presence-aware types (altitude, distance, floors, steps, total-calories):

- **Missing date** → device not worn/synced — render as "no data", NOT zero
- **`countSum: "0"`** → true zero (device worn, no activity)

```bash
# Example output:
# {"dataPoints": [
#   {"date": "2026-03-28", "countSum": "9037"},  # Active day
#   {"date": "2026-03-27", "countSum": "0"},     # Worn but sedentary
#   # 2026-03-26 missing entirely                 # Not worn - DON'T treat as 0
# ]}
```

**Never** average over absent days as if they were zeros — this silently deflates statistics.

### Time Formats

- Use ISO 8601: `2026-03-28T14:00:00Z`
- Shortcut: `today`, `yesterday`
- Date-only: `2026-03-28` (interprets as start of day in user's timezone)

```bash
ghealth data steps daily-rollup --from 2026-03-22 --to today
ghealth data heart-rate list --from yesterday --limit 50
```

## Common Workflows

### Weekly Step Summary

```bash
#!/bin/bash
# Get last 7 days of step data
WEEK_AGO=$(date -d '7 days ago' +%Y-%m-%d)
TODAY=$(date +%Y-%m-%d)

ghealth data steps daily-rollup --from $WEEK_AGO --to $TODAY | \
  jq -r '.dataPoints[] | "\(.date): \(.countSum) steps"'
```

### Export All Exercise Data

```bash
#!/bin/bash
# Export all exercises from a month
ghealth data exercise list --from 2026-03-01 --to 2026-03-31 | \
  jq -r '.dataPoints[].id' | \
  while read id; do
    ghealth data exercise export-tcx --id "$id" --output "exercise_${id}.csv" --as csv
  done
```

### Daily Health Dashboard

```bash
#!/bin/bash
# Fetch today's key metrics
echo "=== Health Metrics for $(date +%Y-%m-%d) ==="

echo -n "Steps: "
ghealth data steps daily-rollup --from today --to today | jq -r '.dataPoints[0].countSum // "no data"'

echo -n "Average Heart Rate: "
ghealth data heart-rate list --from today --limit 500 | \
  jq '[.dataPoints[].beatsPerMinute | tonumber] | add / length | round'

echo -n "Weight: "
ghealth data weight list --limit 1 | jq -r '.dataPoints[0].weightGrams // "no data"'
```

### Data Reconciliation

Check for data conflicts and resolve:

```bash
# Reconcile heart rate data
ghealth data heart-rate reconcile --from 2026-03-01 --to 2026-03-31

# Reconcile with conflict resolution strategy
ghealth data exercise reconcile --from 2026-03-01 --strategy newest
```

## Troubleshooting

### No OAuth Credentials

If you see exit code 5 with JSON error:

```json
{
  "error": {
    "type": "config",
    "code": 5,
    "message": "No OAuth client_secret.json configured",
    "next_steps": [
      "Open https://console.cloud.google.com/apis/credentials",
      "Create or select a Google Cloud project",
      "Enable the Google Health API",
      "Create OAuth client ID with Application type: Desktop app",
      "Download the client_secret JSON",
      "Run: ghealth setup --client-secret /path/to/client_secret.json"
    ]
  }
}
```

Get instructions without error:

```bash
ghealth setup --instructions
```

### Token Refresh Issues

```bash
# Check auth status
ghealth auth status

# Force refresh
ghealth auth refresh

# Re-authenticate
ghealth auth login
```

### Empty Data Results

Check:
1. Date range is correct (use `--from` and `--to`)
2. Using correct command (`daily-rollup` vs `list`)
3. OAuth scopes include necessary permissions
4. Device has synced data

```bash
# Verify scopes
ghealth auth status | jq -r '.scopes'

# Check available types
ghealth schema types
```

### Rate Limiting

The API has rate limits. Space out requests:

```bash
# Add delays between bulk operations
for id in $(ghealth data exercise list | jq -r '.dataPoints[].id'); do
  ghealth data exercise get --id "$id"
  sleep 1
done
```

## Schema Exploration

```bash
# List all available data types
ghealth schema types

# Get details for specific type
ghealth schema type --name steps

# View available operations for a type
ghealth schema type --name exercise | jq -r '.operations[]'
```

## Best Practices for Agents

1. **Always use `daily-rollup` for totals** — `list` returns intervals, not sums
2. **Handle pagination** — check for `nextPageToken` in responses
3. **Respect missing data** — don't coalesce absent days to zero
4. **Use exit codes** — check `$?` for error handling
5. **Parse with jq** — all output is structured JSON under `.dataPoints`
6. **Set explicit date ranges** — `--from` and `--to` prevent unbounded queries
7. **Use `--dry-run`** for validation before destructive operations
8. **Store credentials securely** — use environment variables for tokens
