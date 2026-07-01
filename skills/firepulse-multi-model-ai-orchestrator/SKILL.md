---
name: firepulse-multi-model-ai-orchestrator
description: Unified CLI gateway for orchestrating xAI, OpenRouter, Mistral, and DeepSeek LLMs with smart routing and streaming responses
triggers:
  - how do I set up FirePulse multi-model AI orchestrator
  - configure CLI LLM mesh for multiple AI providers
  - use xAI Grok with OpenRouter and Mistral together
  - set up multi-provider AI chat in terminal
  - orchestrate multiple LLM models from command line
  - switch between DeepSeek and Mistral models dynamically
  - configure smart AI model routing in CLI
  - stream responses from multiple AI providers
---

# FirePulse Multi-Model AI Orchestrator

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

FirePulse is a unified AI gateway that connects multiple LLM providers (xAI, OpenRouter, Mistral, DeepSeek) through a single command-line interface. It automatically routes queries to the optimal model based on context length, latency, and cost, with streaming responses and persistent session memory.

## Installation

### Prerequisites

- Python 3.10+ runtime
- 512KB disk space
- Terminal with UTF-8 support

### Setup Steps

1. **Clone the repository**
```bash
git clone https://github.com/vishalGitthub/cli-llm-mesh.git
cd cli-llm-mesh
```

2. **Run bootstrap script**
```bash
python bootstrap.py
```

This creates the `.firepulse` directory structure for configuration and encrypted credentials.

3. **Install dependencies (if required)**
```bash
pip install -r requirements.txt
```

## Configuration

### Provider Credentials

FirePulse uses a YAML-based configuration file with encrypted API key storage. Configuration file is typically located at `~/.firepulse/config.yaml`.

**Example configuration structure:**

```yaml
providers:
  xai:
    api_key: "${XAI_API_KEY}"
    models:
      - grok-1
      - grok-1.5
    priority: high
    timeout: 5000
    
  openrouter:
    api_key: "${OPENROUTER_API_KEY}"
    models:
      - anthropic/claude-2
      - meta-llama/llama-3-70b
    priority: medium
    timeout: 5000
    
  mistral:
    api_key: "${MISTRAL_API_KEY}"
    models:
      - mistral-large
      - mistral-medium
      - mistral-small
    priority: high
    timeout: 5000
    
  deepseek:
    api_key: "${DEEPSEEK_API_KEY}"
    models:
      - deepseek-coder
      - deepseek-v2
    priority: medium
    timeout: 5000

routing:
  strategy: "smart"  # Options: smart, cost, speed, quality
  budget_threshold: 0.01  # USD per query
  max_context_length: 8192
  fallback_enabled: true

session:
  persist_history: true
  max_context_messages: 20
  auto_purge_days: 7
```

### Adding Provider Credentials

```bash
# Set environment variables for API keys
export XAI_API_KEY="your-xai-key"
export OPENROUTER_API_KEY="your-openrouter-key"
export MISTRAL_API_KEY="your-mistral-key"
export DEEPSEEK_API_KEY="your-deepseek-key"

# Run credential manager
python firepulse.py --setup
```

## Key Commands

### Start Interactive Session

```bash
python firepulse.py
```

### Specify Provider Explicitly

```bash
python firepulse.py --provider xai
python firepulse.py --provider mistral
```

### Single Query Mode

```bash
python firepulse.py --query "Explain quantum computing"
```

### List Available Models

```bash
python firepulse.py --list-models
```

### Check Provider Status

```bash
python firepulse.py --status
```

### View Performance Metrics

```bash
python firepulse.py --metrics
```

### Set Routing Strategy

```bash
python firepulse.py --strategy cost  # Optimize for cost
python firepulse.py --strategy speed  # Optimize for latency
python firepulse.py --strategy quality  # Optimize for best model
```

## Code Examples

### Python Integration (Library Mode)

If FirePulse exposes a Python API:

```python
from firepulse import Orchestrator, RouterConfig

# Initialize orchestrator
config = RouterConfig(
    strategy="smart",
    budget_threshold=0.01,
    fallback_enabled=True
)

orchestrator = Orchestrator(config_path="~/.firepulse/config.yaml")

# Single query
response = orchestrator.query("Explain machine learning")
print(response.text)
print(f"Provider used: {response.provider}")
print(f"Model: {response.model}")
print(f"Latency: {response.latency_ms}ms")
print(f"Tokens: {response.tokens_used}")

# Streaming response
for chunk in orchestrator.stream("Write a Python script for web scraping"):
    print(chunk.text, end="", flush=True)

# Session-based conversation
session = orchestrator.create_session()
session.add_message("What is recursion?")
response1 = session.query()

session.add_message("Can you give me a code example?")
response2 = session.query()  # Maintains context

# Explicit provider selection
response = orchestrator.query(
    "Generate TypeScript types",
    provider="mistral",
    model="mistral-large"
)
```

### Custom Router Logic

```python
from firepulse import Orchestrator, RouterStrategy

class CostOptimizedRouter(RouterStrategy):
    def select_provider(self, query, context):
        # Filter providers by cost
        providers = self.get_available_providers()
        cheapest = min(providers, key=lambda p: p.cost_per_token)
        
        # Ensure context window fits
        if len(query) + len(context) > cheapest.max_context:
            return self.fallback_provider
            
        return cheapest

orchestrator = Orchestrator(router_strategy=CostOptimizedRouter())
```

### Batch Processing

```python
from firepulse import Orchestrator
import asyncio

orchestrator = Orchestrator()

async def batch_process(queries):
    tasks = [orchestrator.async_query(q) for q in queries]
    responses = await asyncio.gather(*tasks)
    return responses

queries = [
    "Summarize quantum mechanics",
    "Explain neural networks",
    "What is blockchain?"
]

results = asyncio.run(batch_process(queries))
for i, result in enumerate(results):
    print(f"Query {i+1}: {result.provider} - {result.latency_ms}ms")
```

### Go Integration (if project uses Go)

Based on the project topics mentioning `go` and `golang`:

```go
package main

import (
    "fmt"
    "github.com/vishalGitthub/cli-llm-mesh/pkg/orchestrator"
)

func main() {
    // Initialize orchestrator
    config := orchestrator.Config{
        ConfigPath: "~/.firepulse/config.yaml",
        Strategy:   orchestrator.SmartRouting,
    }
    
    orch, err := orchestrator.New(config)
    if err != nil {
        panic(err)
    }
    
    // Single query
    response, err := orch.Query("Explain Docker containers")
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("Response: %s\n", response.Text)
    fmt.Printf("Provider: %s\n", response.Provider)
    fmt.Printf("Latency: %dms\n", response.LatencyMs)
    
    // Streaming query
    stream, err := orch.StreamQuery("Write a REST API in Go")
    if err != nil {
        panic(err)
    }
    
    for chunk := range stream {
        fmt.Print(chunk.Text)
    }
    
    // Explicit provider
    response, err = orch.QueryWithProvider(
        "Generate code documentation",
        orchestrator.ProviderMistral,
        "mistral-large",
    )
}
```

### Session Management

```go
package main

import (
    "fmt"
    "github.com/vishalGitthub/cli-llm-mesh/pkg/orchestrator"
)

func main() {
    orch, _ := orchestrator.New(orchestrator.Config{})
    
    // Create persistent session
    session := orch.NewSession()
    
    // First query
    session.AddMessage("What is Kubernetes?")
    resp1, _ := session.Query()
    fmt.Println(resp1.Text)
    
    // Follow-up with context
    session.AddMessage("How does it differ from Docker Swarm?")
    resp2, _ := session.Query()
    fmt.Println(resp2.Text)
    
    // Save session
    session.Save("~/.firepulse/sessions/k8s-discussion.json")
    
    // Load session later
    loadedSession, _ := orch.LoadSession("~/.firepulse/sessions/k8s-discussion.json")
}
```

## Common Patterns

### Pattern 1: Cost-Optimized Queries

```python
from firepulse import Orchestrator

orchestrator = Orchestrator()

# Set budget constraint
response = orchestrator.query(
    "Generate marketing copy",
    max_cost=0.005,  # Max $0.005 per query
    fallback_on_budget_exceeded=True
)
```

### Pattern 2: Multi-Provider Validation

```python
# Query multiple providers for consensus
providers = ["xai", "mistral", "deepseek"]
responses = []

for provider in providers:
    resp = orchestrator.query(
        "Is this code correct? print('hello)",
        provider=provider
    )
    responses.append(resp)

# Compare responses
consensus = analyze_consensus(responses)
```

### Pattern 3: Latency-Critical Operations

```python
# Force fastest provider
orchestrator.set_strategy("speed")
response = orchestrator.query(
    "Quick yes/no: Is Python dynamically typed?",
    timeout=1000  # 1 second timeout
)
```

### Pattern 4: Code Generation with Fallback

```python
# Try DeepSeek (code specialist) first, fallback to Mistral
try:
    response = orchestrator.query(
        "Generate a binary search tree in Python",
        provider="deepseek",
        model="deepseek-coder"
    )
except TimeoutError:
    response = orchestrator.query(
        "Generate a binary search tree in Python",
        provider="mistral",
        model="mistral-large"
    )
```

### Pattern 5: Multilingual Support

```python
# Auto-detect language and respond accordingly
response = orchestrator.query("Explique l'apprentissage automatique")
print(response.detected_language)  # "fr"
print(response.text)  # Response in French

# Force output language
response = orchestrator.query(
    "Explain machine learning",
    output_language="es"  # Force Spanish output
)
```

## Troubleshooting

### Issue: Provider Timeout

**Symptom:** Queries hang or fail with timeout errors

**Solution:**
```yaml
# Increase timeout in config.yaml
providers:
  xai:
    timeout: 10000  # Increase to 10 seconds
```

Or programmatically:
```python
orchestrator = Orchestrator(default_timeout=10000)
```

### Issue: Authentication Errors

**Symptom:** `401 Unauthorized` or `Invalid API Key`

**Solution:**
```bash
# Verify environment variables
echo $XAI_API_KEY
echo $MISTRAL_API_KEY

# Re-run setup
python firepulse.py --setup --force

# Check credential encryption
python firepulse.py --verify-credentials
```

### Issue: Model Not Available

**Symptom:** Requested model returns 404 or is unavailable

**Solution:**
```bash
# List currently available models
python firepulse.py --list-models --provider xai

# Update config to use available models
python firepulse.py --refresh-models
```

### Issue: High Token Usage / Cost

**Symptom:** Unexpectedly high API costs

**Solution:**
```python
# Enable cost monitoring
orchestrator = Orchestrator(
    budget_threshold=0.01,
    warn_on_threshold=True
)

# Check cost estimates before query
estimate = orchestrator.estimate_cost("Your long query...")
print(f"Estimated cost: ${estimate.cost}")
```

### Issue: Slow Response Times

**Symptom:** Latency exceeds expected 200ms to first token

**Solution:**
```bash
# Check provider status
python firepulse.py --status

# Switch to speed-optimized routing
python firepulse.py --strategy speed

# Enable local query validation to reduce roundtrips
```

```python
orchestrator = Orchestrator(
    offline_validation=True,  # Validate locally first
    parallel_requests=True     # Query multiple providers in parallel
)
```

### Issue: Context Length Exceeded

**Symptom:** `Context length exceeded` errors

**Solution:**
```python
# Automatically truncate or summarize context
session = orchestrator.create_session(
    max_context_length=4096,
    auto_summarize=True  # Summarize old messages when limit reached
)
```

### Issue: Provider Fallback Not Working

**Symptom:** Query fails instead of falling back to alternate provider

**Solution:**
```yaml
# Ensure fallback is enabled in config.yaml
routing:
  fallback_enabled: true
  fallback_order:
    - mistral
    - openrouter
    - deepseek
    - xai
```

```python
# Force fallback behavior
orchestrator = Orchestrator(
    strict_fallback=False,  # Allow any provider as fallback
    retry_on_failure=3      # Retry up to 3 times
)
```

## Performance Optimization

### Enable Response Caching

```python
from firepulse import Orchestrator, CacheConfig

cache_config = CacheConfig(
    enabled=True,
    ttl=3600,  # 1 hour cache
    max_size_mb=100
)

orchestrator = Orchestrator(cache_config=cache_config)

# Repeated queries use cached responses
response1 = orchestrator.query("What is Python?")  # API call
response2 = orchestrator.query("What is Python?")  # Cached
```

### Parallel Provider Interrogation

```python
# Query multiple providers simultaneously, use fastest response
response = orchestrator.query_parallel(
    "Explain Docker",
    providers=["xai", "mistral", "deepseek"],
    return_first=True  # Return as soon as first provider responds
)
```

## Docker Deployment

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY . .

RUN pip install -r requirements.txt

# Mount config as volume
VOLUME /root/.firepulse

CMD ["python", "firepulse.py"]
```

```bash
# Build and run
docker build -t firepulse .
docker run -it \
  -e XAI_API_KEY="${XAI_API_KEY}" \
  -e MISTRAL_API_KEY="${MISTRAL_API_KEY}" \
  -v ~/.firepulse:/root/.firepulse \
  firepulse
```

## Environment Variables Reference

```bash
# Provider API Keys
export XAI_API_KEY="your-xai-key"
export OPENROUTER_API_KEY="your-openrouter-key"
export MISTRAL_API_KEY="your-mistral-key"
export DEEPSEEK_API_KEY="your-deepseek-key"

# Configuration Overrides
export FIREPULSE_CONFIG_PATH="/custom/path/config.yaml"
export FIREPULSE_ROUTING_STRATEGY="cost"  # smart, cost, speed, quality
export FIREPULSE_BUDGET_THRESHOLD="0.01"
export FIREPULSE_DEFAULT_TIMEOUT="5000"
export FIREPULSE_LOG_LEVEL="info"  # debug, info, warn, error
```
