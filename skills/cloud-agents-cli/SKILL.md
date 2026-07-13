---
name: cloud-agents-cli
description: TypeScript CLI and skill suite for building agents on Google Cloud with Drive, Gmail, Calendar, Sheets, Docs, Chat, and Admin integration
triggers:
  - how do I use cloud-agents-cli
  - set up google cloud agents cli
  - install agents-cli for google workspace
  - configure cloud-agents-cli with typescript
  - use cloud-agents-cli skills
  - integrate google drive with agents-cli
  - cloud-agents-cli configuration and setup
---

# cloud-agents-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What is cloud-agents-cli?

`cloud-agents-cli` is a TypeScript-first CLI and skill suite for building AI agents that interact with Google Cloud services including Drive, Gmail, Calendar, Sheets, Docs, Chat, and Admin APIs. It provides installable knowledge packs (skills) for coding assistants, typed library exports, and a zero-Python runtime architecture.

### Key Features

- **TypeScript throughout**: Strict-mode CLI, library, and tests
- **Seven agent skills**: Installable knowledge packs for Google Cloud services
- **Zero Python runtime**: Pure Node.js 20+ implementation
- **Typed library exports**: Import configuration, logging, and utilities
- **Skills framework**: Markdown-based skill packages for AI agents

## Installation

### Prerequisites

- Node.js 20 or higher
- npm or compatible package manager

### From Source

```bash
git clone https://github.com/pifferologo/cloud-agents-cli.git
cd cloud-agents-cli
npm install
npm run validate
node dist/cli/main.js --version
```

### Global Installation (Development)

```bash
npm link
agents-cli info
agents-cli skills list
```

### As a Dependency

```bash
npm install cloud-agents-cli
```

### Skills Only Installation

```bash
npx skills add google/agents-cli
```

## CLI Commands

### Information & Setup

```bash
# Display CLI info and configuration
agents-cli info

# Show version
agents-cli --version

# Get help
agents-cli --help
```

### Skills Management

```bash
# List all available skills
agents-cli skills list

# Add skills to your project
npx skills add google/agents-cli
```

## Configuration

### Environment Variables

Create a `.env` file in your project root:

```bash
# Log level: debug, info, warn, error
AGENTS_CLI_LOG_LEVEL=info

# Optional: Custom skills directory
AGENTS_CLI_SKILLS_DIR=/path/to/custom/skills
```

### Loading Configuration in Code

```typescript
import { loadAppConfig } from "cloud-agents-cli";

const config = loadAppConfig();
console.log(config.logLevel); // "info"
```

## Library Usage

### Basic Import (Default Export)

```typescript
import { loadAppConfig } from "cloud-agents-cli";

const config = loadAppConfig();
console.log(`Log level: ${config.logLevel}`);
```

### Barrel Exports (Specific Modules)

```typescript
import { loadAppConfig } from "cloud-agents-cli/config";
import { createLogger } from "cloud-agents-cli/logger";

const config = loadAppConfig();
const logger = createLogger(config.logLevel);

logger.info("Application started");
logger.debug("Debug information");
logger.error("Error occurred");
```

### Configuration Module

```typescript
import { loadAppConfig } from "cloud-agents-cli/config";

// Load and validate environment configuration
const config = loadAppConfig();

// Access configuration properties
console.log(config.logLevel);        // string: "info" | "debug" | "warn" | "error"
console.log(config.skillsDir);       // string | undefined
```

### Logger Module

```typescript
import { createLogger } from "cloud-agents-cli/logger";

const logger = createLogger("debug");

logger.debug("Debugging details");
logger.info("Informational message");
logger.warn("Warning message");
logger.error("Error message");
```

## Common Patterns

### CLI Application Setup

```typescript
import { loadAppConfig, createLogger } from "cloud-agents-cli";

async function main() {
  const config = loadAppConfig();
  const logger = createLogger(config.logLevel);
  
  logger.info("Starting application...");
  
  try {
    // Your application logic
    logger.info("Application running");
  } catch (error) {
    logger.error(`Application error: ${error}`);
    process.exit(1);
  }
}

main();
```

### Custom Skills Directory

```typescript
import { loadAppConfig } from "cloud-agents-cli/config";
import * as path from "path";

const config = loadAppConfig();

const skillsPath = config.skillsDir 
  ? path.resolve(config.skillsDir)
  : path.join(__dirname, "../skills");

console.log(`Loading skills from: ${skillsPath}`);
```

### Environment-Based Configuration

```typescript
import { loadAppConfig } from "cloud-agents-cli/config";

const config = loadAppConfig();

const isDevelopment = config.logLevel === "debug";
const isProduction = config.logLevel === "error" || config.logLevel === "warn";

if (isDevelopment) {
  console.log("Running in development mode");
}
```

## Development Workflow

### Build Commands

```bash
# Full validation pipeline (typecheck + lint + test + build)
npm run validate

# Run unit tests only
npm run test

# Compile application and CLI to dist/
npm run build:app

# Compile docs site scripts
npm run build:docs

# TypeScript type checking
npm run typecheck

# ESLint linting
npm run lint
```

### Project Structure

```
agents-cli/
├── src/
│   ├── cli/              # CLI entry (commander)
│   ├── config/           # Environment configuration
│   ├── docs/             # MkDocs site scripts
│   ├── logger.ts         # Logger implementation
│   └── index.ts          # Library exports
├── tests/                # Vitest unit tests
├── skills/               # Agent skill markdown packages
├── docs/                 # MkDocs site content
├── dist/                 # Compiled output (gitignored)
├── package.json
├── tsconfig.base.json    # Shared TS baseline
├── tsconfig.json         # App + tests typecheck
├── tsconfig.build.json   # Production build
└── tsconfig.docs.json    # Docs script compilation
```

### Running Tests

```bash
# Run all tests
npm run test

# Run tests in watch mode
npm run test -- --watch

# Run tests with coverage
npm run test -- --coverage
```

### Local Development

```bash
# Install dependencies
npm install

# Link for global CLI access
npm link

# Use the CLI
agents-cli info
agents-cli skills list

# Or run directly
node dist/cli/main.js info
```

## TypeScript Configuration

### Importing Types

```typescript
import type { Config } from "cloud-agents-cli/config";
import type { Logger } from "cloud-agents-cli/logger";

function setupApp(config: Config, logger: Logger) {
  logger.info(`Config loaded with log level: ${config.logLevel}`);
}
```

### Strict Mode Compliance

The project uses strict TypeScript settings. When extending:

```typescript
import { loadAppConfig } from "cloud-agents-cli";

// All variables must be typed or inferred
const config = loadAppConfig();

// Null checks required
if (config.skillsDir) {
  console.log(`Custom skills dir: ${config.skillsDir}`);
}
```

## Troubleshooting

### `agents-cli` Command Not Found

**Solution 1**: Build and link

```bash
npm run build
npm link
```

**Solution 2**: Run directly

```bash
node dist/cli/main.js info
```

**Solution 3**: Use npx

```bash
npx agents-cli info
```

### Skills Directory Empty

Ensure you cloned the full repository:

```bash
git clone https://github.com/pifferologo/cloud-agents-cli.git
cd cloud-agents-cli
ls -la skills/  # Should show skill markdown files
```

### TypeScript Compilation Errors

```bash
# Clean and rebuild
rm -rf dist/
npm run build:app

# Check for type errors
npm run typecheck
```

### Environment Variables Not Loading

Verify `.env` file exists in project root:

```bash
# Check .env file
cat .env

# Should contain:
# AGENTS_CLI_LOG_LEVEL=info
```

Load dotenv in your application:

```typescript
import "dotenv/config";
import { loadAppConfig } from "cloud-agents-cli";

const config = loadAppConfig();
```

### Module Resolution Issues

Ensure `package.json` has correct exports:

```json
{
  "exports": {
    ".": "./dist/index.js",
    "./config": "./dist/config/index.js",
    "./logger": "./dist/logger.js"
  }
}
```

### Build Output Missing

```bash
# Check tsconfig.build.json outDir
npm run build:app

# Verify dist/ directory exists
ls -la dist/
```

## Integration Examples

### Express Server Integration

```typescript
import express from "express";
import { loadAppConfig, createLogger } from "cloud-agents-cli";

const app = express();
const config = loadAppConfig();
const logger = createLogger(config.logLevel);

app.get("/health", (req, res) => {
  logger.info("Health check requested");
  res.json({ status: "ok", logLevel: config.logLevel });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  logger.info(`Server listening on port ${PORT}`);
});
```

### Custom CLI Tool

```typescript
import { Command } from "commander";
import { loadAppConfig, createLogger } from "cloud-agents-cli";

const program = new Command();
const config = loadAppConfig();
const logger = createLogger(config.logLevel);

program
  .name("my-tool")
  .description("Custom tool using cloud-agents-cli")
  .version("1.0.0");

program
  .command("process")
  .description("Process data")
  .action(() => {
    logger.info("Processing started");
    // Your logic here
  });

program.parse();
```

## Best Practices

1. **Always load config at startup**: Call `loadAppConfig()` early in your application lifecycle
2. **Use structured logging**: Create logger instances with appropriate log levels
3. **Type everything**: Leverage TypeScript's strict mode for compile-time safety
4. **Environment-based configuration**: Use `.env` for local development, environment variables for production
5. **Validate before deploy**: Run `npm run validate` before committing changes
6. **Use barrel exports**: Import from specific modules (`cloud-agents-cli/config`) for better tree-shaking

## Resources

- **Repository**: https://github.com/pifferologo/cloud-agents-cli
- **License**: Apache-2.0
- **Issues**: Check the repository for open issues (currently 0)
- **Skills Directory**: Browse `/skills` in the repository for agent knowledge packs
