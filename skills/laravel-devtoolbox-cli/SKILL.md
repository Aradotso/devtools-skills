---
name: laravel-devtoolbox-cli
description: Swiss-army artisan CLI for Laravel to scan, inspect, debug, and explore every aspect of your Laravel application from the command line
triggers:
  - analyze my laravel application structure
  - find unused routes in laravel
  - trace sql queries for a route
  - generate model relationship diagram
  - scan laravel models and relationships
  - check for unprotected laravel routes
  - analyze laravel performance and memory
  - compare laravel environment files
---

# Laravel Devtoolbox CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Laravel Devtoolbox is a comprehensive CLI toolkit for Laravel applications that provides deep introspection, analysis, and debugging capabilities. It helps developers scan models, routes, services, database queries, security issues, and performance bottlenecks directly from the command line.

## Installation

Install as a development dependency via Composer:

```bash
composer require --dev grazulex/laravel-devtoolbox
```

**Requirements:**
- PHP 8.3+
- Laravel 11.x or 12.x

## Core Concepts

Laravel Devtoolbox provides multiple categories of analysis commands:

1. **Application Scanning** - Comprehensive health checks and overview
2. **Model Analysis** - Eloquent models, relationships, and usage
3. **Route Analysis** - Route inspection, unused detection, reverse lookup
4. **Database Analysis** - SQL tracing, N+1 detection, column usage
5. **Security Analysis** - Unprotected routes, middleware coverage
6. **Performance Analysis** - Memory, slow queries, cache, queue analysis
7. **Container Analysis** - Service bindings, providers, dependency injection

All commands support multiple output formats: `array` (default), `json`, `count`, `mermaid`.

## Key Commands

### Application Overview & Scanning

```bash
# Enhanced application overview
php artisan dev:about+ --extended --performance

# Comprehensive scan of all aspects
php artisan dev:scan --all

# Specific scanners
php artisan dev:scan --models --routes --services
```

### Model Analysis

```bash
# List all Eloquent models
php artisan dev:models

# Export models to JSON
php artisan dev:models --format=json --output=models.json

# Find where a specific model is used
php artisan dev:model:where-used App\\Models\\User

# Generate relationship diagram (Mermaid format)
php artisan dev:model:graph --format=mermaid --output=relationships.mmd
```

### Route Analysis

```bash
# List all routes with details
php artisan dev:routes

# Find unused routes
php artisan dev:routes:unused

# Find routes by controller (reverse lookup)
php artisan dev:routes:where UserController

# Export unused routes to JSON
php artisan dev:routes:unused --format=json --output=unused-routes.json
```

### Database & SQL Analysis

```bash
# Trace SQL queries for a specific route
php artisan dev:sql:trace --route=dashboard

# Analyze for N+1 problems and duplicates
php artisan dev:sql:duplicates --route=users.index --threshold=3

# Check column usage across codebase
php artisan dev:db:column-usage --unused-only

# Detect slow queries
php artisan dev:performance:slow-queries --threshold=1000
```

### Security Analysis

```bash
# Find unprotected routes
php artisan dev:security:unprotected-routes

# Show only critical unprotected routes
php artisan dev:security:unprotected-routes --critical-only

# Export security scan
php artisan dev:security:unprotected-routes --format=json --output=security-audit.json
```

### Performance Analysis

```bash
# Analyze memory usage for a route
php artisan dev:performance:memory --route=dashboard

# Find slow database queries
php artisan dev:performance:slow-queries --threshold=1000

# Analyze cache performance
php artisan dev:cache:analysis --drivers=redis,file

# Analyze queue performance
php artisan dev:queue:analysis --failed-jobs --slow-jobs
```

### Service Container & Providers

```bash
# List service container bindings
php artisan dev:services

# Analyze container bindings with resolution details
php artisan dev:container:bindings --show-resolved

# Service provider performance timeline
php artisan dev:providers:timeline --slow-threshold=100

# List middleware usage
php artisan dev:middleware

# Find where middleware is used
php artisan dev:middlewares:where-used auth
```

### Environment & Configuration

```bash
# Compare environment files
php artisan dev:env:diff --against=.env.example

# Monitor logs in real-time
php artisan dev:log:tail --follow --level=error

# List all views
php artisan dev:views
```

## Configuration

Publish the configuration file:

```bash
php artisan vendor:publish --tag=devtoolbox-config
```

This creates `config/devtoolbox.php`:

```php
<?php

return [
    /*
     | Default output format for commands
     | Options: 'array', 'json', 'count', 'mermaid'
     */
    'default_format' => env('DEVTOOLBOX_FORMAT', 'array'),

    /*
     | Scanner-specific options
     */
    'scanners' => [
        'models' => [
            'enabled' => true,
            'paths' => [app_path('Models')],
        ],
        'routes' => [
            'enabled' => true,
            'exclude_patterns' => ['debugbar.*', 'telescope.*'],
        ],
        'performance' => [
            'slow_query_threshold' => 1000, // milliseconds
            'memory_limit_warning' => 128, // MB
        ],
    ],

    /*
     | Export settings
     */
    'export' => [
        'default_path' => storage_path('devtoolbox'),
        'json_pretty_print' => true,
    ],
];
```

## Common Patterns

### Daily Development Workflow

```bash
#!/bin/bash
# daily-check.sh - Run daily health checks

echo "=== Application Health Check ==="

# Quick overview
php artisan dev:about+ --extended

# Count unused routes
echo "Unused routes:"
php artisan dev:routes:unused --format=count

# Check for unprotected routes
echo "Unprotected routes:"
php artisan dev:security:unprotected-routes --format=count

# Compare env files
echo "Environment differences:"
php artisan dev:env:diff --against=.env.example
```

### CI/CD Quality Gates

```bash
#!/bin/bash
# ci-quality-check.sh - CI/CD integration script

# Generate reports
php artisan dev:scan --all --format=json --output=reports/scan.json
php artisan dev:routes:unused --format=json --output=reports/unused-routes.json
php artisan dev:security:unprotected-routes --format=json --output=reports/security.json

# Check thresholds
UNUSED_ROUTES=$(php artisan dev:routes:unused --format=count | jq -r '.count')
UNPROTECTED_ROUTES=$(php artisan dev:security:unprotected-routes --format=count | jq -r '.count')

echo "Unused routes: $UNUSED_ROUTES"
echo "Unprotected routes: $UNPROTECTED_ROUTES"

# Fail if thresholds exceeded
if [ "$UNUSED_ROUTES" -gt 10 ]; then
    echo "❌ Too many unused routes: $UNUSED_ROUTES (max: 10)"
    exit 1
fi

if [ "$UNPROTECTED_ROUTES" -gt 5 ]; then
    echo "❌ Too many unprotected routes: $UNPROTECTED_ROUTES (max: 5)"
    exit 1
fi

echo "✅ Quality checks passed"
```

### Generate Documentation

```bash
#!/bin/bash
# generate-docs.sh - Auto-generate project documentation

mkdir -p docs/architecture

# Generate model documentation
php artisan dev:models --format=json --output=docs/architecture/models.json

# Generate relationship diagram
php artisan dev:model:graph --format=mermaid --output=docs/architecture/relationships.mmd

# Generate route documentation
php artisan dev:routes --format=json --output=docs/architecture/routes.json

# Generate service bindings
php artisan dev:container:bindings --format=json --output=docs/architecture/bindings.json

echo "✅ Documentation generated in docs/architecture/"
```

### Performance Debugging Session

```php
<?php
// Example: Debug performance issues for a specific route

// 1. Trace SQL queries
// php artisan dev:sql:trace --route=users.index

// 2. Check for N+1 problems
// php artisan dev:sql:duplicates --route=users.index --threshold=3

// 3. Analyze memory usage
// php artisan dev:performance:memory --route=users.index

// 4. Find slow queries
// php artisan dev:performance:slow-queries --threshold=500

// 5. Check cache configuration
// php artisan dev:cache:analysis --drivers=redis
```

### Model Usage Analysis

```bash
# Find all places where User model is used
php artisan dev:model:where-used App\\Models\\User

# Generate relationships for specific models
php artisan dev:model:graph --format=mermaid --models=User,Post,Comment

# Export model analysis
php artisan dev:models --format=json | jq '.[] | select(.relationships | length > 5)'
```

### Security Audit

```bash
#!/bin/bash
# security-audit.sh - Complete security check

echo "=== Security Audit ==="

# Find unprotected routes
php artisan dev:security:unprotected-routes --format=json --output=security/unprotected.json

# Check middleware usage
php artisan dev:middleware --format=json --output=security/middleware.json

# Analyze route protection patterns
php artisan dev:routes --format=json | \
  jq '[.[] | select(.middleware | index("auth") | not)]' > security/no-auth.json

echo "Security reports saved to security/"
```

## Real-World Examples

### Example 1: Finding N+1 Query Problems

```bash
# Run the N+1 detector on a specific route
php artisan dev:sql:duplicates --route=posts.index --threshold=3

# Output shows:
# - Duplicate queries
# - Number of times each query runs
# - Suggested eager loading solutions
```

**Fix in Controller:**

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        // ❌ Before (N+1 problem)
        // $posts = Post::all();
        
        // ✅ After (eager loading based on dev:sql:duplicates suggestion)
        $posts = Post::with(['author', 'comments.user'])->get();
        
        return view('posts.index', compact('posts'));
    }
}
```

### Example 2: Cleaning Up Unused Routes

```bash
# Find unused routes
php artisan dev:routes:unused --format=json --output=cleanup/unused-routes.json

# Review the list and remove from routes/web.php
cat cleanup/unused-routes.json | jq -r '.[] | .name'
```

**Before cleanup:**

```php
<?php
// routes/web.php

Route::get('/old-dashboard', [OldDashboardController::class, 'index'])->name('old.dashboard');
Route::get('/legacy-users', [LegacyUserController::class, 'index'])->name('legacy.users');
Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
```

**After cleanup (remove unused routes):**

```php
<?php
// routes/web.php

Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
```

### Example 3: Documenting Application Architecture

```bash
# Generate complete architecture documentation
mkdir -p docs/architecture

php artisan dev:models --format=json --output=docs/architecture/models.json
php artisan dev:model:graph --format=mermaid --output=docs/architecture/relationships.mmd
php artisan dev:routes --format=json --output=docs/architecture/routes.json
php artisan dev:services --format=json --output=docs/architecture/services.json
```

**Use Mermaid diagram in README.md:**

```markdown
# Application Architecture

## Model Relationships

```mermaid
{{< include docs/architecture/relationships.mmd >}}
```
```

### Example 4: Performance Monitoring Script

```php
<?php
// scripts/performance-check.php

// Run this script before deployment
$routes = [
    'home',
    'dashboard',
    'users.index',
    'posts.index',
];

foreach ($routes as $route) {
    echo "Checking route: $route\n";
    
    // Check memory usage
    shell_exec("php artisan dev:performance:memory --route=$route");
    
    // Check SQL queries
    shell_exec("php artisan dev:sql:duplicates --route=$route --threshold=3");
    
    echo "\n";
}
```

### Example 5: Automated Security Checks

```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      
      - name: Install dependencies
        run: composer install --prefer-dist --no-progress
      
      - name: Run security scan
        run: |
          php artisan dev:security:unprotected-routes --format=json > security-report.json
          UNPROTECTED=$(jq 'length' security-report.json)
          echo "Found $UNPROTECTED unprotected routes"
          if [ "$UNPROTECTED" -gt 5 ]; then
            echo "::error::Too many unprotected routes: $UNPROTECTED"
            exit 1
          fi
      
      - name: Upload security report
        uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: security-report.json
```

## Troubleshooting

### Command Not Found

If `dev:*` commands are not available:

```bash
# Clear cache and reload
php artisan clear-compiled
php artisan config:clear
php artisan cache:clear
composer dump-autoload

# Verify installation
composer show grazulex/laravel-devtoolbox
```

### Memory Issues with Large Applications

For large applications, increase memory limit:

```bash
php -d memory_limit=512M artisan dev:scan --all

# Or set in php.ini
memory_limit = 512M
```

### Performance Issues with SQL Tracing

SQL tracing can be slow on routes with many queries:

```bash
# Use threshold to focus on problematic queries
php artisan dev:sql:duplicates --route=users.index --threshold=5
```

### JSON Output Parsing

When using JSON output in scripts:

```bash
# Pretty print JSON
php artisan dev:models --format=json | jq '.'

# Extract specific fields
php artisan dev:routes:unused --format=json | jq -r '.[] | .name'

# Count results
php artisan dev:models --format=json | jq 'length'
```

### Mermaid Diagram Rendering

If Mermaid diagrams are too large:

```bash
# Focus on specific models
php artisan dev:model:graph --format=mermaid --models=User,Post,Comment

# Use online editors
# Copy output to https://mermaid.live/
```

### Export Path Issues

If output files fail to save:

```bash
# Ensure directory exists and is writable
mkdir -p storage/devtoolbox
chmod -R 775 storage/devtoolbox

# Or use absolute path
php artisan dev:models --format=json --output=/tmp/models.json
```

## Integration Examples

### Laravel Telescope Integration

```php
<?php
// Use alongside Telescope for deeper analysis

// 1. Enable Telescope query logging
// 2. Run dev:sql:trace to get query details
// 3. Compare results in Telescope UI

// Telescope shows runtime queries, DevToolbox shows code analysis
```

### PHPStan/Larastan Integration

```bash
# Use DevToolbox to find issues, PHPStan to prevent them

# Find unused code
php artisan dev:routes:unused > unused.txt
php artisan dev:db:column-usage --unused-only > unused-columns.txt

# Then configure PHPStan rules based on findings
```

### Monitoring Dashboard Integration

```bash
#!/bin/bash
# monitoring/collect-metrics.sh

# Collect metrics for monitoring dashboard
php artisan dev:scan --all --format=json > /var/metrics/laravel-scan.json
php artisan dev:performance:slow-queries --threshold=1000 --format=json > /var/metrics/slow-queries.json
php artisan dev:cache:analysis --format=json > /var/metrics/cache-analysis.json

# Send to monitoring service
curl -X POST https://monitoring.example.com/metrics \
  -H "Content-Type: application/json" \
  -d @/var/metrics/laravel-scan.json
```

## Best Practices

1. **Run in Development Only** - This package is designed for dev environments
2. **Use JSON for Automation** - Always use `--format=json` in CI/CD scripts
3. **Regular Health Checks** - Run `dev:scan --all` weekly to catch issues early
4. **Document Architecture** - Use `dev:model:graph` to maintain architecture diagrams
5. **Security First** - Run `dev:security:unprotected-routes` before deployments
6. **Performance Baseline** - Establish baselines with `dev:performance:*` commands
7. **Version Control Reports** - Commit generated JSON reports for historical tracking
