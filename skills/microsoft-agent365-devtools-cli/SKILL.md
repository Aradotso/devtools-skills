---
name: microsoft-agent365-devtools-cli
description: CLI tool for developing, deploying, and managing Microsoft Agent 365 applications with Azure integration and MCP server support.
triggers:
  - "How do I set up Microsoft Agent 365 DevTools CLI?"
  - "Deploy my Agent 365 application to Azure"
  - "Configure Agent 365 CLI for my environment"
  - "Manage MCP servers with Agent 365"
  - "Publish my agent manifest to Microsoft 365"
  - "Set up Azure resources for Agent 365"
  - "Clean up Agent 365 resources"
  - "Query Entra ID permissions for my agent"
---

# Microsoft Agent 365 DevTools CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

The **Microsoft Agent 365 DevTools CLI** is a command-line interface tool designed to streamline the development, deployment, and management of Microsoft Agent 365 applications. It provides comprehensive tooling for configuration management, Azure resource provisioning, MCP (Model Context Protocol) server integration, and agent deployment workflows.

## What It Does

- **Develop**: Manage MCP tool servers for local agent development
- **Setup**: Create Azure resources, configure permissions, and register agent blueprints
- **Publish**: Update agent manifest IDs and package for Microsoft 365 Admin Center
- **Deploy**: Deploy application binaries to Azure App Service and update tool permissions
- **Config**: Configure Azure subscription, resources, and deployment options
- **Query Entra**: Query Microsoft Entra ID for agent permissions and consent status
- **Cleanup**: Remove all resources (blueprints, instances, Azure resources)

## Installation

### Prerequisites

1. **.NET 8.0 or higher** - [Download .NET](https://dotnet.microsoft.com/download)
2. **Entra ID App Registration** with delegated Microsoft Graph API permissions
3. **Azure Subscription** with appropriate permissions

### Install from NuGet

```powershell
dotnet tool install -g Microsoft.Agents.A365.DevTools.Cli --prerelease
```

### Verify Installation

```powershell
a365 --version
```

## Entra ID App Registration Setup

Before using the CLI, create a custom Entra ID app registration:

1. Navigate to Azure Portal > Entra ID > App Registrations
2. Create new registration (e.g., "Agent365-DevTools-Client")
3. Configure **Delegated** permissions (NOT Application permissions):
   - Microsoft Graph API permissions as required
4. Grant admin consent for all permissions
5. Note your `Client ID` and `Tenant ID`

Set environment variables:

```powershell
# Windows PowerShell
$env:AGENT365_CLIENT_ID="your-client-id"
$env:AGENT365_TENANT_ID="your-tenant-id"

# Linux/macOS
export AGENT365_CLIENT_ID="your-client-id"
export AGENT365_TENANT_ID="your-tenant-id"
```

## Key Commands

### Config Command

Configure your Agent 365 environment settings:

```powershell
# Set Azure subscription
a365 config set --subscription "your-subscription-id"

# Set resource group
a365 config set --resource-group "agent365-rg"

# Set Azure region
a365 config set --location "eastus"

# View current configuration
a365 config show

# Reset configuration
a365 config reset
```

### Setup Command

Create Azure resources and register your agent blueprint:

```powershell
# Interactive setup (recommended for first-time setup)
a365 setup

# Automated setup with parameters
a365 setup --subscription "sub-id" \
  --resource-group "agent365-rg" \
  --location "eastus" \
  --app-name "my-agent365-app" \
  --manifest-path "./agent-manifest.json"

# Setup with existing resources
a365 setup --use-existing-resources --app-service-name "existing-app"
```

### Develop Command

Manage MCP tool servers for local development:

```powershell
# Start MCP server for development
a365 develop start --server-path "./mcp-server" --port 5000

# List running MCP servers
a365 develop list

# Stop MCP server
a365 develop stop --server-id "server-123"

# Test MCP server connection
a365 develop test --server-url "http://localhost:5000"
```

### Develop-MCP Command

Manage MCP servers in Dataverse environments:

```powershell
# Register MCP server in Dataverse
a365 develop-mcp register \
  --environment "your-env-url" \
  --server-name "MyMCPServer" \
  --server-url "https://mcp.example.com"

# List MCP servers in environment
a365 develop-mcp list --environment "your-env-url"

# Remove MCP server
a365 develop-mcp remove --environment "your-env-url" --server-id "server-id"
```

### Deploy Command

Deploy your Agent 365 application to Azure:

```powershell
# Deploy application binaries
a365 deploy --project-path "./src/MyAgent365App" \
  --configuration "Release"

# Deploy with specific runtime
a365 deploy --project-path "./src/MyAgent365App" \
  --runtime "win-x64" \
  --configuration "Release"

# Deploy and update tool permissions
a365 deploy --project-path "./src/MyAgent365App" \
  --update-permissions
```

### Publish Command

Package your agent manifest for Microsoft 365 Admin Center:

```powershell
# Update manifest IDs and package
a365 publish --manifest-path "./agent-manifest.json" \
  --output-path "./dist"

# Publish with custom app package name
a365 publish --manifest-path "./agent-manifest.json" \
  --output-path "./dist" \
  --package-name "my-agent-package"

# Validate manifest only (no packaging)
a365 publish --manifest-path "./agent-manifest.json" \
  --validate-only
```

### Query-Entra Command

Query Microsoft Entra ID for agent information:

```powershell
# Query agent scopes and permissions
a365 query-entra --agent-id "agent-blueprint-id"

# Check consent status
a365 query-entra --agent-id "agent-blueprint-id" \
  --check-consent

# List all agents in tenant
a365 query-entra --list-all

# Export agent configuration
a365 query-entra --agent-id "agent-blueprint-id" \
  --export-config "./agent-config.json"
```

### Cleanup Command

Remove Agent 365 resources:

```powershell
# Clean up all resources (interactive confirmation)
a365 cleanup

# Clean up specific resource group
a365 cleanup --resource-group "agent365-rg"

# Force cleanup without confirmation
a365 cleanup --force

# Clean up only blueprint (keep Azure resources)
a365 cleanup --blueprint-only --agent-id "blueprint-id"
```

## Configuration File

The CLI stores configuration in a local file (typically `~/.agent365/config.json`):

```json
{
  "azureSubscriptionId": "your-subscription-id",
  "resourceGroup": "agent365-rg",
  "location": "eastus",
  "appServiceName": "my-agent365-app",
  "appServicePlanName": "agent365-plan",
  "clientId": "your-client-id",
  "tenantId": "your-tenant-id"
}
```

## Agent Manifest Structure

Example `agent-manifest.json`:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/agent365/v1.0/agent-manifest.schema.json",
  "id": "com.example.myagent",
  "version": "1.0.0",
  "name": {
    "short": "My Agent",
    "full": "My Agent 365 Application"
  },
  "description": {
    "short": "An intelligent agent",
    "full": "A comprehensive agent for Microsoft 365"
  },
  "developer": {
    "name": "Your Company",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "icons": {
    "color": "color-icon.png",
    "outline": "outline-icon.png"
  },
  "capabilities": [
    {
      "name": "conversation",
      "messageHandlers": [
        {
          "type": "message",
          "value": "handleMessage"
        }
      ]
    }
  ],
  "permissions": [
    "identity",
    "messageTeamMembers"
  ],
  "validDomains": [
    "*.example.com"
  ]
}
```

## Common Development Workflow

### 1. Initial Setup

```powershell
# Install CLI
dotnet tool install -g Microsoft.Agents.A365.DevTools.Cli --prerelease

# Configure environment
a365 config set --subscription $env:AZURE_SUBSCRIPTION_ID
a365 config set --resource-group "agent365-dev-rg"
a365 config set --location "eastus"

# Set up Azure resources and blueprint
a365 setup --manifest-path "./agent-manifest.json"
```

### 2. Local Development

```powershell
# Start MCP server for testing
a365 develop start --server-path "./mcp-server" --port 5000

# Test your agent locally with the MCP server
# (Use your development framework here)

# Stop MCP server when done
a365 develop stop --server-id "server-123"
```

### 3. Deploy to Azure

```powershell
# Build and deploy application
a365 deploy --project-path "./src/MyAgent365App" \
  --configuration "Release" \
  --update-permissions

# Verify deployment
az webapp show --name my-agent365-app --resource-group agent365-dev-rg
```

### 4. Publish for Distribution

```powershell
# Package agent for Microsoft 365 Admin Center
a365 publish --manifest-path "./agent-manifest.json" \
  --output-path "./dist"

# Upload the generated package to Microsoft 365 Admin Center
# Then hire the agent through Teams
```

### 5. Query and Verify

```powershell
# Check agent permissions and consent
a365 query-entra --agent-id $env:AGENT_BLUEPRINT_ID --check-consent

# Export configuration for documentation
a365 query-entra --agent-id $env:AGENT_BLUEPRINT_ID \
  --export-config "./docs/agent-config.json"
```

## C# Integration Example

If you're building an Agent 365 application in C#:

```csharp
using Microsoft.Agents.Core;
using Microsoft.Agents.Protocols.Adapter;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace MyAgent365App
{
    public class Program
    {
        public static async Task Main(string[] args)
        {
            var host = Host.CreateDefaultBuilder(args)
                .ConfigureServices((context, services) =>
                {
                    // Register Agent 365 services
                    services.AddAgent365Services(options =>
                    {
                        options.ClientId = Environment.GetEnvironmentVariable("AGENT365_CLIENT_ID");
                        options.TenantId = Environment.GetEnvironmentVariable("AGENT365_TENANT_ID");
                        options.ManifestPath = "./agent-manifest.json";
                    });

                    // Register your agent handler
                    services.AddSingleton<IAgentHandler, MyAgentHandler>();
                })
                .Build();

            await host.RunAsync();
        }
    }

    public class MyAgentHandler : IAgentHandler
    {
        public async Task<AgentResponse> HandleMessageAsync(
            AgentRequest request, 
            CancellationToken cancellationToken)
        {
            // Process agent request
            var response = new AgentResponse
            {
                Text = $"Received: {request.Text}",
                SuggestedActions = new[]
                {
                    new AgentAction { Title = "Learn More", Value = "learn_more" }
                }
            };

            return response;
        }
    }
}
```

Deploy this application:

```powershell
# Deploy the C# application
a365 deploy --project-path "./MyAgent365App" \
  --configuration "Release" \
  --runtime "win-x64"
```

## Environment Variables

Set these environment variables for CLI authentication and configuration:

```powershell
# Required for authentication
AGENT365_CLIENT_ID="your-entra-app-client-id"
AGENT365_TENANT_ID="your-entra-tenant-id"

# Optional for automation
AZURE_SUBSCRIPTION_ID="your-azure-subscription-id"
AGENT365_RESOURCE_GROUP="agent365-rg"
AGENT365_LOCATION="eastus"
AGENT_BLUEPRINT_ID="your-agent-blueprint-id"
```

## Troubleshooting

### Authentication Issues

**Problem**: "Authentication failed" or "Insufficient permissions"

**Solution**:
```powershell
# Verify Entra ID app registration has delegated permissions
a365 query-entra --agent-id $env:AGENT_BLUEPRINT_ID --check-consent

# Re-login if token expired
az login --tenant $env:AGENT365_TENANT_ID

# Verify environment variables
echo $env:AGENT365_CLIENT_ID
echo $env:AGENT365_TENANT_ID
```

### Deployment Failures

**Problem**: "Deployment failed" or "Resource not found"

**Solution**:
```powershell
# Verify configuration
a365 config show

# Check Azure resources exist
az group show --name agent365-rg
az webapp show --name my-agent365-app --resource-group agent365-rg

# Re-run setup if resources missing
a365 setup --use-existing-resources
```

### MCP Server Connection Issues

**Problem**: "Cannot connect to MCP server"

**Solution**:
```powershell
# Test MCP server connectivity
a365 develop test --server-url "http://localhost:5000"

# Check server logs
a365 develop list

# Restart MCP server
a365 develop stop --server-id "server-123"
a365 develop start --server-path "./mcp-server" --port 5000
```

### Manifest Validation Errors

**Problem**: "Invalid manifest" during publish

**Solution**:
```powershell
# Validate manifest without packaging
a365 publish --manifest-path "./agent-manifest.json" --validate-only

# Check manifest schema
# Ensure all required fields are present and valid
# Verify icon files exist in specified paths
```

### Cleanup Issues

**Problem**: "Cannot delete resources" or "Resources still in use"

**Solution**:
```powershell
# Force cleanup (use with caution)
a365 cleanup --force

# Manual cleanup via Azure CLI
az group delete --name agent365-rg --yes --no-wait

# Remove only blueprint
a365 cleanup --blueprint-only --agent-id $env:AGENT_BLUEPRINT_ID
```

## Best Practices

1. **Use Configuration File**: Store common settings with `a365 config set` to avoid repetitive parameters
2. **Environment Variables**: Use environment variables for sensitive data (client IDs, tenant IDs)
3. **Version Control**: Include `agent-manifest.json` in version control, exclude `config.json`
4. **Automated Pipelines**: Use `--force` and `--no-wait` flags for CI/CD automation
5. **Resource Naming**: Use consistent naming conventions for Azure resources
6. **Regular Cleanup**: Remove unused resources to avoid Azure costs
7. **Permission Audits**: Regularly check consent status with `query-entra` command

## Additional Resources

- [Microsoft Agent 365 CLI Documentation](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/agent-365-cli)
- [CLI Command Reference](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/reference/cli)
- [Custom Client App Registration Guide](https://learn.microsoft.com/microsoft-agent-365/developer/custom-client-app-registration)
- [GitHub Repository](https://github.com/microsoft/Agent365-devTools)
