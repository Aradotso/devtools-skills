---
name: app-store-connect-cli
description: Automate iOS/macOS App Store Connect workflows - TestFlight, builds, submissions, screenshots, and metadata with the asc CLI
triggers:
  - upload build to testflight
  - submit app to app store
  - manage app store screenshots
  - check app review status
  - list testflight builds
  - sync app metadata to app store
  - automate app store connect workflow
  - get app store analytics
---

# App Store Connect CLI (asc)

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Fast, scriptable CLI for the App Store Connect API. Automate TestFlight, builds, submissions, signing, analytics, screenshots, subscriptions, and more. JSON-first, no interactive prompts - perfect for CI/CD and automation.

## Installation

```bash
# Homebrew (macOS/Linux)
brew install asc

# Install script
curl -fsSL https://asccli.sh/install | bash

# Verify installation
asc version
asc --help
```

For Windows, download signed binaries from the [releases page](https://github.com/rorkai/App-Store-Connect-CLI/releases/latest).

## Authentication

### Generate API Keys

1. Visit https://appstoreconnect.apple.com/access/integrations/api
2. Create a new API key with appropriate role (Admin for full access)
3. Download the `.p8` private key file
4. Note the Key ID and Issuer ID

### Configure Authentication

**Interactive environments (with keychain):**

```bash
asc auth login \
  --name "MyApp" \
  --key-id "$ASC_KEY_ID" \
  --issuer-id "$ASC_ISSUER_ID" \
  --private-key /path/to/AuthKey.p8 \
  --network
```

**CI/CD or headless environments:**

```bash
asc auth login \
  --bypass-keychain \
  --name "MyCIKey" \
  --key-id "$ASC_KEY_ID" \
  --issuer-id "$ASC_ISSUER_ID" \
  --private-key /path/to/AuthKey.p8
```

**Repo-local credentials:**

```bash
asc auth login \
  --local \
  --bypass-keychain \
  --name "LocalKey" \
  --key-id "$ASC_KEY_ID" \
  --issuer-id "$ASC_ISSUER_ID" \
  --private-key /path/to/AuthKey.p8
```

### Validate Authentication

```bash
# Check auth status
asc auth status --validate

# Run diagnostics
asc auth doctor

# List active profile
asc auth list
```

## Core Commands

### Apps Management

```bash
# List all apps
asc apps list --output table
asc apps list --output json --pretty

# Get specific app info
asc apps info view --app "1234567890" --output json

# Search apps
asc apps list --output json | jq '.data[] | select(.attributes.name | contains("MyApp"))'
```

### Builds

```bash
# Upload IPA to App Store Connect
asc builds upload --app "1234567890" --ipa /path/to/MyApp.ipa

# List builds
asc builds list --app "1234567890" --output table
asc builds list --app "1234567890" --limit 10 --sort -uploadedDate

# Get next build number
asc builds next-build-number --app "1234567890" --version "1.2.3"

# Get build details
asc builds get --id "BUILD_ID" --output json
```

### TestFlight

```bash
# Publish to TestFlight
asc publish testflight \
  --app "1234567890" \
  --ipa /path/to/MyApp.ipa \
  --group "Internal Testers" \
  --wait

# List TestFlight groups
asc testflight groups list --app "1234567890" --output table

# Add testers to group
asc testflight testers add \
  --group "GROUP_ID" \
  --email user@example.com

# List feedback
asc testflight feedback list \
  --app "1234567890" \
  --paginate \
  --output json

# List crashes
asc testflight crashes list \
  --app "1234567890" \
  --sort -createdDate \
  --limit 10

# Get crash logs
asc testflight crashes log --submission-id "SUBMISSION_ID"
```

### App Store Submissions

```bash
# Full publish workflow (upload + attach + submit)
asc publish appstore \
  --app "1234567890" \
  --ipa /path/to/MyApp.ipa \
  --version "1.2.3" \
  --submit \
  --confirm

# Stage release (without submitting)
asc release stage \
  --app "1234567890" \
  --version "1.2.3" \
  --build "BUILD_ID" \
  --copy-metadata-from "1.2.2" \
  --dry-run

# Validate readiness
asc validate --app "1234567890" --version "1.2.3"

# Check submission status
asc submit status --version-id "VERSION_ID"

# Monitor status with auto-refresh
asc status --app "1234567890" --watch

# Cancel submission
asc submit cancel --version-id "VERSION_ID" --confirm
```

### Versions

```bash
# List versions
asc versions list --app "1234567890" --output table

# Create new version
asc versions create \
  --app "1234567890" \
  --version "1.3.0" \
  --platform "IOS"

# Get version details
asc versions get --id "VERSION_ID" --output json
```

### Metadata & Localization

```bash
# List localizations
asc localizations list \
  --app "1234567890" \
  --type app-info \
  --output table

# Initialize metadata directory structure
asc metadata init \
  --dir ./metadata \
  --version "1.2.3" \
  --locale "en-US"

# Apply metadata from directory
asc metadata apply \
  --app "1234567890" \
  --version "1.2.3" \
  --dir ./metadata \
  --dry-run

# Audit keywords (ASO checks)
asc metadata keywords audit \
  --app "1234567890" \
  --version "1.2.3" \
  --blocked-terms-file ./blocked-terms.txt

# Sync metadata between versions
asc metadata sync \
  --app "1234567890" \
  --from-version "1.2.2" \
  --to-version "1.2.3" \
  --locale "en-US"
```

### Screenshots

```bash
# Plan screenshot layout
asc screenshots plan \
  --app "1234567890" \
  --version "1.2.3" \
  --review-output-dir ./screenshots/review

# Apply screenshots
asc screenshots apply \
  --app "1234567890" \
  --version "1.2.3" \
  --review-output-dir ./screenshots/review \
  --confirm

# Upload screenshots for specific locale
# First, get version localization ID
asc localizations list \
  --version "VERSION_ID" \
  --output json \
  --locale "en-US" | jq '.data[0].id'

# Then upload
asc screenshots upload \
  --version-localization "VERSION_LOCALIZATION_ID" \
  --path ./screenshots/en-US \
  --device-type "IPHONE_65" \
  --replace

# List existing screenshots
asc screenshots list \
  --version-localization "VERSION_LOCALIZATION_ID" \
  --output table
```

### Review Status

```bash
# Check review status
asc review status --app "1234567890"

# Run review diagnostics
asc review doctor --app "1234567890"
```

### Signing & Certificates

```bash
# List certificates
asc certificates list --output table

# List provisioning profiles
asc profiles list --output table

# List bundle IDs
asc bundle-ids list --output table

# Register new bundle ID
asc bundle-ids create \
  --identifier "com.example.myapp" \
  --name "My App" \
  --platform "IOS"
```

### Xcode Integration

```bash
# Archive project
asc xcode archive \
  --project MyApp.xcodeproj \
  --scheme "MyApp" \
  --archive-path ./build/MyApp.xcarchive

# Export IPA
asc xcode export \
  --archive-path ./build/MyApp.xcarchive \
  --export-options-plist ./ExportOptions.plist \
  --export-path ./build/output
```

### Xcode Cloud

```bash
# List workflows
asc xcode-cloud workflows list --output table

# Trigger workflow from PR
asc xcode-cloud run \
  --workflow-id "WORKFLOW_ID" \
  --pull-request-id "123"

# Rerun existing build
asc xcode-cloud run \
  --source-run-id "BUILD_RUN_ID" \
  --clean

# Get build run details
asc xcode-cloud build-runs get --id "BUILD_RUN_ID" --output json
```

## Workflows

### Workflow Configuration

Create `.asc/workflow.json` for reusable workflows:

```json
{
  "workflows": {
    "testflight_beta": {
      "description": "Build and deploy to TestFlight",
      "steps": [
        {
          "name": "get_next_build",
          "command": "builds next-build-number",
          "args": {
            "app": "1234567890",
            "version": "{{VERSION}}"
          },
          "capture": "BUILD_NUMBER"
        },
        {
          "name": "archive",
          "command": "xcode archive",
          "args": {
            "project": "MyApp.xcodeproj",
            "scheme": "MyApp",
            "archive-path": "./build/MyApp.xcarchive"
          }
        },
        {
          "name": "export",
          "command": "xcode export",
          "args": {
            "archive-path": "./build/MyApp.xcarchive",
            "export-options-plist": "./ExportOptions.plist",
            "export-path": "./build/output"
          }
        },
        {
          "name": "publish",
          "command": "publish testflight",
          "args": {
            "app": "1234567890",
            "ipa": "./build/output/MyApp.ipa",
            "group": "Internal Testers",
            "wait": true
          }
        }
      ]
    },
    "appstore_release": {
      "description": "Submit to App Store",
      "steps": [
        {
          "name": "validate",
          "command": "validate",
          "args": {
            "app": "1234567890",
            "version": "{{VERSION}}"
          }
        },
        {
          "name": "publish",
          "command": "publish appstore",
          "args": {
            "app": "1234567890",
            "ipa": "./build/output/MyApp.ipa",
            "version": "{{VERSION}}",
            "submit": true,
            "confirm": true
          }
        }
      ]
    }
  }
}
```

### Run Workflows

```bash
# Validate workflow
asc workflow validate

# Dry run
asc workflow run --dry-run testflight_beta VERSION:1.2.3

# Execute workflow
asc workflow run testflight_beta VERSION:1.2.3

# Execute App Store release
asc workflow run appstore_release VERSION:1.2.3
```

## CI/CD Integration

### GitHub Actions

```yaml
name: TestFlight Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install asc
        run: brew install asc
      
      - name: Configure Auth
        run: |
          echo "${{ secrets.ASC_PRIVATE_KEY }}" > AuthKey.p8
          asc auth login \
            --bypass-keychain \
            --name "CI" \
            --key-id "${{ secrets.ASC_KEY_ID }}" \
            --issuer-id "${{ secrets.ASC_ISSUER_ID }}" \
            --private-key ./AuthKey.p8
      
      - name: Build and Deploy
        run: |
          asc workflow run testflight_beta VERSION:1.2.3
        env:
          ASC_BYPASS_KEYCHAIN: "1"
```

### GitLab CI

```yaml
deploy_testflight:
  stage: deploy
  tags:
    - macos
  script:
    - brew install asc
    - echo "$ASC_PRIVATE_KEY" > AuthKey.p8
    - |
      asc auth login \
        --bypass-keychain \
        --name "GitLab CI" \
        --key-id "$ASC_KEY_ID" \
        --issuer-id "$ASC_ISSUER_ID" \
        --private-key ./AuthKey.p8
    - asc publish testflight --app "$APP_ID" --ipa ./MyApp.ipa --group "Internal" --wait
  variables:
    ASC_BYPASS_KEYCHAIN: "1"
  only:
    - main
```

### CircleCI

```yaml
version: 2.1

jobs:
  deploy:
    macos:
      xcode: 15.0.0
    steps:
      - checkout
      - run:
          name: Install asc
          command: brew install asc
      - run:
          name: Configure Auth
          command: |
            echo "$ASC_PRIVATE_KEY" > AuthKey.p8
            asc auth login \
              --bypass-keychain \
              --name "CircleCI" \
              --key-id "$ASC_KEY_ID" \
              --issuer-id "$ASC_ISSUER_ID" \
              --private-key ./AuthKey.p8
      - run:
          name: Deploy
          command: asc workflow run testflight_beta VERSION:${CIRCLE_TAG}
          environment:
            ASC_BYPASS_KEYCHAIN: "1"
```

## Output Formats

The CLI auto-detects output format based on context:
- **TTY (interactive)**: `table` format
- **Non-TTY (pipes/CI)**: `json` format

### Explicit Format Control

```bash
# Table output
asc apps list --output table

# JSON output
asc apps list --output json

# Pretty JSON (human-readable)
asc apps list --output json --pretty

# Markdown output
asc apps list --output markdown

# Set default via environment
export ASC_DEFAULT_OUTPUT=json
```

### Parsing JSON Output

```bash
# Extract app IDs
asc apps list --output json | jq -r '.data[].id'

# Filter by name
asc apps list --output json | jq '.data[] | select(.attributes.name == "MyApp")'

# Get build count
asc builds list --app "1234567890" --output json | jq '.data | length'

# Extract version strings
asc versions list --app "1234567890" --output json | jq -r '.data[].attributes.versionString'
```

## Common Patterns

### Complete Release Workflow

```bash
#!/bin/bash
set -e

APP_ID="1234567890"
VERSION="1.2.3"
IPA_PATH="./build/output/MyApp.ipa"

# 1. Get next build number
BUILD_NUMBER=$(asc builds next-build-number --app "$APP_ID" --version "$VERSION" --output json | jq -r '.buildNumber')
echo "Next build number: $BUILD_NUMBER"

# 2. Build (external Xcode process)
# xcodebuild archive ...
# xcodebuild -exportArchive ...

# 3. Upload and publish to TestFlight
asc publish testflight \
  --app "$APP_ID" \
  --ipa "$IPA_PATH" \
  --group "Internal Testers" \
  --wait

# 4. Wait for processing
sleep 60

# 5. Validate for App Store
asc validate --app "$APP_ID" --version "$VERSION"

# 6. Submit to App Store
asc publish appstore \
  --app "$APP_ID" \
  --version "$VERSION" \
  --build "BUILD_ID" \
  --submit \
  --confirm

# 7. Monitor status
asc status --app "$APP_ID" --watch
```

### Metadata Sync Between Versions

```bash
#!/bin/bash

APP_ID="1234567890"
FROM_VERSION="1.2.2"
TO_VERSION="1.2.3"
LOCALES=("en-US" "es-ES" "fr-FR" "de-DE")

for locale in "${LOCALES[@]}"; do
  echo "Syncing $locale..."
  asc metadata sync \
    --app "$APP_ID" \
    --from-version "$FROM_VERSION" \
    --to-version "$TO_VERSION" \
    --locale "$locale"
done
```

### Screenshot Upload Pipeline

```bash
#!/bin/bash

APP_ID="1234567890"
VERSION_ID="VERSION_ID"
SCREENSHOTS_DIR="./screenshots"

# Get version localizations
LOCALIZATIONS=$(asc localizations list \
  --version "$VERSION_ID" \
  --output json | jq -r '.data[] | "\(.attributes.locale)|\(.id)"')

while IFS='|' read -r locale loc_id; do
  echo "Uploading screenshots for $locale..."
  
  # Upload for each device type
  for device in "IPHONE_65" "IPHONE_67" "IPAD_PRO_129"; do
    if [ -d "$SCREENSHOTS_DIR/$locale/$device" ]; then
      asc screenshots upload \
        --version-localization "$loc_id" \
        --path "$SCREENSHOTS_DIR/$locale/$device" \
        --device-type "$device" \
        --replace
    fi
  done
done <<< "$LOCALIZATIONS"
```

### Crash Log Collection

```bash
#!/bin/bash

APP_ID="1234567890"
OUTPUT_DIR="./crash_logs"

mkdir -p "$OUTPUT_DIR"

# Get recent crashes
CRASHES=$(asc testflight crashes list \
  --app "$APP_ID" \
  --sort -createdDate \
  --limit 20 \
  --output json)

# Download logs for each crash
echo "$CRASHES" | jq -r '.data[].id' | while read -r submission_id; do
  echo "Downloading crash log: $submission_id"
  asc testflight crashes log \
    --submission-id "$submission_id" > "$OUTPUT_DIR/$submission_id.log"
done
```

### Automated TestFlight Feedback Reports

```bash
#!/bin/bash

APP_ID="1234567890"
REPORT_FILE="testflight_feedback_$(date +%Y%m%d).json"

# Collect all feedback
asc testflight feedback list \
  --app "$APP_ID" \
  --paginate \
  --output json > "$REPORT_FILE"

# Generate summary
echo "Feedback Summary:"
jq '.data | group_by(.attributes.rating) | map({rating: .[0].attributes.rating, count: length})' "$REPORT_FILE"
```

## Environment Variables

```bash
# Authentication
export ASC_KEY_ID="ABC123"
export ASC_ISSUER_ID="DEF456"
export ASC_PRIVATE_KEY_PATH="/path/to/AuthKey.p8"

# Bypass keychain (CI/CD)
export ASC_BYPASS_KEYCHAIN="1"

# Default output format
export ASC_DEFAULT_OUTPUT="json"

# API debugging
export ASC_DEBUG="api"

# Config file location
export ASC_CONFIG_PATH="/custom/path/config.json"
```

## Troubleshooting

### Authentication Issues

```bash
# Check keychain access
asc auth doctor

# Retry with bypass (if keychain blocked)
ASC_BYPASS_KEYCHAIN=1 asc auth status --validate

# Validate active profile
asc auth status --validate

# Re-login with bypass
asc auth login \
  --bypass-keychain \
  --name "Retry" \
  --key-id "$ASC_KEY_ID" \
  --issuer-id "$ASC_ISSUER_ID" \
  --private-key /path/to/AuthKey.p8
```

### API Debugging

```bash
# Enable API debugging
ASC_DEBUG=api asc apps list

# Or use flag
asc --api-debug apps list

# Capture detailed logs
asc --api-debug apps list 2>&1 | tee debug.log
```

### Build Upload Failures

```bash
# Verify IPA integrity
unzip -t MyApp.ipa

# Check app ID matches
asc apps list --output json | jq '.data[] | select(.id == "1234567890")'

# Retry with explicit app ID
asc builds upload --app "1234567890" --ipa ./MyApp.ipa --verbose
```

### Version Conflicts

```bash
# List existing versions
asc versions list --app "1234567890" --output table

# Check version state
asc versions get --id "VERSION_ID" --output json | jq '.data.attributes.appStoreState'

# Delete draft version (if needed)
asc versions delete --id "VERSION_ID" --confirm
```

### Screenshot Upload Issues

```bash
# Verify device type compatibility
asc screenshots plan --app "1234567890" --version "1.2.3" --review-output-dir ./review

# Check localization ID
asc localizations list --version "VERSION_ID" --output json

# Use correct device type codes:
# IPHONE_65, IPHONE_67, IPHONE_61, IPAD_PRO_129, IPAD_PRO_3GEN_129
```

## Best Practices

1. **Always validate before submission**:
   ```bash
   asc validate --app "APP_ID" --version "VERSION"
   ```

2. **Use `--dry-run` for destructive operations**:
   ```bash
   asc metadata apply --dry-run --app "APP_ID" --version "VERSION" --dir ./metadata
   ```

3. **Monitor submissions with `--watch`**:
   ```bash
   asc status --app "APP_ID" --watch
   ```

4. **Store credentials securely in CI**:
   - Use GitHub Secrets, GitLab Variables, etc.
   - Never commit `.p8` files
   - Use `--bypass-keychain` in CI

5. **Parse JSON programmatically**:
   ```bash
   asc apps list --output json | jq -r '.data[].id'
   ```

6. **Use workflows for complex pipelines**:
   - Define in `.asc/workflow.json`
   - Version control your workflows
   - Test with `--dry-run`

7. **Leverage `--wait` for synchronous operations**:
   ```bash
   asc publish testflight --wait --app "APP_ID" --ipa ./app.ipa
   ```

## Additional Resources

- Official docs: https://asccli.sh
- GitHub repo: https://github.com/rorkai/App-Store-Connect-CLI
- CI/CD guide: https://github.com/rorkai/App-Store-Connect-CLI/blob/main/docs/CI_CD.md
- Workflows guide: https://github.com/rorkai/App-Store-Connect-CLI/blob/main/docs/WORKFLOWS.md
- Apple API docs: https://developer.apple.com/documentation/appstoreconnectapi
