---
name: stage-cli-code-review
description: Review local code changes organized into logical chapters with Stage CLI, a code review tool that works with any AI agent
triggers:
  - review my code changes in chapters
  - organize my git diff into reviewable sections
  - show me staged changes with stage cli
  - create a code review with stage
  - break down my local changes for review
  - review code changes by chapter
  - use stage to review uncommitted work
  - open stage review interface
---

# Stage CLI Code Review

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Stage CLI is a code review tool that organizes local code changes into logical chapters and highlights what to review. It runs entirely on your local machine and works with any AI agent to help you review code before committing or creating pull requests.

## Installation

Install globally via npm:

```bash
npm install -g stagereview
```

Or use directly with npx:

```bash
npx stagereview
```

## Core Commands

### Basic Review

Review your current changes (auto-detects what to review):

```bash
stage-chapters
```

This will:
1. Analyze your git repository
2. Detect staged/unstaged/untracked changes
3. Organize changes into logical chapters
4. Open a browser UI for review

### Review Staged Changes Only

Review only changes that are staged for commit:

```bash
stage-chapters --ref staged
```

### Review Unstaged Changes Only

Review only unstaged working directory changes:

```bash
stage-chapters --ref unstaged
```

### Review All Working Changes

Review staged + unstaged + untracked files:

```bash
stage-chapters --ref work
```

### Diff Against Specific Branch

Compare your changes against a specific base branch:

```bash
stage-chapters --base develop
```

```bash
stage-chapters --base feature/authentication
```

```bash
stage-chapters --base origin/main
```

## Common Workflows

### Pre-Commit Review

Before committing, review staged changes:

```bash
git add .
stage-chapters --ref staged
```

### Feature Branch Review

Review all changes in your feature branch against main:

```bash
stage-chapters --base main --ref work
```

### Quick WIP Check

Review what you've changed since last commit:

```bash
stage-chapters --ref unstaged
```

### Cross-Branch Comparison

Compare your current branch against another feature branch:

```bash
stage-chapters --base feature-a
```

## Configuration Options

### `--base <ref>`

Specifies the base git reference to diff against.

- Default: Auto-detects `main`, `master`, or `develop`
- Accepts: branch names, commit hashes, tags, remote refs

Examples:
```bash
--base main
--base origin/develop
--base abc123f
--base v1.2.0
```

### `--ref <mode>`

Defines the scope of changes to review.

Options:
- `work` - All local changes (staged + unstaged + untracked)
- `staged` - Only changes in git staging area
- `unstaged` - Only working directory changes not staged
- Default: Auto-detects based on repository state

Examples:
```bash
--ref work
--ref staged
--ref unstaged
```

## Integration with AI Agents

Stage CLI is designed to work seamlessly with AI coding agents. In your agent's chat interface:

```
/stage-chapters
```

Or with options:

```
/stage-chapters --ref staged --base develop
```

The agent can help you:
- Interpret the review chapters
- Identify potential issues
- Suggest improvements
- Explain complex changes

## Programmatic Usage

While Stage CLI is primarily a command-line tool, you can integrate it into scripts:

```bash
#!/bin/bash
# Review script for CI/local checks

# Check if there are staged changes
if git diff --cached --quiet; then
  echo "No staged changes to review"
  exit 0
fi

# Run Stage review
stage-chapters --ref staged
```

```bash
#!/bin/bash
# Pre-push review workflow

MAIN_BRANCH="main"
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

if [ "$CURRENT_BRANCH" != "$MAIN_BRANCH" ]; then
  echo "Reviewing changes against $MAIN_BRANCH..."
  stage-chapters --base $MAIN_BRANCH
fi
```

## Understanding the Output

Stage CLI organizes changes into logical chapters based on:

- **File proximity** - Related files grouped together
- **Change type** - New files, modifications, deletions
- **Logical boundaries** - Module/component boundaries
- **Change complexity** - Similar complexity levels grouped

The browser UI shows:
- Chapter overview with file lists
- Contextual diff views
- Review points and suggestions
- Navigation between chapters

## Troubleshooting

### "Not a git repository" Error

Stage CLI requires a git repository. Initialize one:

```bash
git init
```

### No Changes Detected

Verify you have changes:

```bash
git status
```

If using `--ref staged`, ensure files are staged:

```bash
git add <files>
```

### Base Branch Not Found

Ensure the base reference exists:

```bash
git branch -a  # List all branches
git log --oneline  # Check commit history
```

Use a valid reference:

```bash
stage-chapters --base origin/main
```

### Port Already in Use

Stage CLI starts a local web server. If the port is occupied, it will try alternative ports automatically. Close any conflicting applications if issues persist.

### Large Diffs Timeout

For very large changesets, consider:

1. Review in smaller chunks:
```bash
stage-chapters --ref staged  # Review staged first
stage-chapters --ref unstaged  # Then unstaged
```

2. Split commits into smaller logical units
3. Use more specific base branches

### Browser Doesn't Open

Manually open the URL shown in terminal output (typically `http://localhost:3000` or similar).

## Best Practices

### 1. Review Early and Often

Run Stage reviews before commits, not just before PRs:

```bash
# After making changes
git add -p  # Interactively stage
stage-chapters --ref staged  # Review staged
git commit
```

### 2. Use Appropriate Scope

Match the `--ref` option to your workflow:

- **Before commit**: `--ref staged`
- **Checking progress**: `--ref unstaged`
- **Full feature review**: `--ref work --base main`

### 3. Leverage Chapter Organization

Use chapters to:
- Review related changes together
- Spot unintended cross-module coupling
- Identify missing changes in related files

### 4. Combine with AI Agent Review

Let the AI agent analyze Stage's output:

```
Review the chapters from Stage CLI and identify:
1. Potential bugs or edge cases
2. Missing error handling
3. Inconsistent patterns
```

### 5. Pre-PR Workflow

Before creating a pull request:

```bash
# Ensure all changes are committed
git add -A
git commit -m "Feature complete"

# Review full feature against main
stage-chapters --base main

# Address review findings
# Create PR
```

## Advanced Usage

### Reviewing Specific File Patterns

Stage reviews all changes, but you can prepare specific changes:

```bash
# Stage only TypeScript files
git add '*.ts'
stage-chapters --ref staged

# Stage specific directory
git add src/components/
stage-chapters --ref staged
```

### Multi-Branch Comparison

Compare changes across multiple branches:

```bash
# Review branch A vs main
git checkout feature-a
stage-chapters --base main

# Review branch B vs main
git checkout feature-b
stage-chapters --base main
```

### Integration with Git Hooks

Add to `.git/hooks/pre-commit`:

```bash
#!/bin/bash
# Launch Stage review before commit
stage-chapters --ref staged
read -p "Proceed with commit? (y/n) " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
  exit 1
fi
```

## Additional Resources

- **Website**: https://stagereview.app
- **Examples**: https://stagereview.app/explore
- **Blog**: https://stagereview.app/blog
- **Discord**: https://discord.gg/Hs7Eexp3
- **Twitter**: https://x.com/StageReviewApp
- **GitHub**: https://github.com/ReviewStage/stage-cli

The full Stage experience on the website offers additional features like team collaboration, persistent reviews, and integration with GitHub pull requests.
