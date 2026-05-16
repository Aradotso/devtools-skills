---
name: google-agents-cli
description: CLI and skills for building, evaluating, and deploying AI agents on Google Cloud's Gemini Enterprise Agent Platform using ADK
triggers:
  - "build an agent with agents-cli"
  - "create a new ADK agent project"
  - "deploy my agent to Google Cloud"
  - "run evaluations on my agent"
  - "scaffold a new agents-cli project"
  - "publish my agent to Gemini Enterprise"
  - "set up CI/CD for my ADK agent"
  - "add RAG to my agent project"
---

# google-agents-cli

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

`agents-cli` is the CLI and skills framework for building, evaluating, and deploying AI agents on Google Cloud's Gemini Enterprise Agent Platform. It works with ADK (Agent Development Kit) to provide end-to-end agent development workflows, from scaffolding to production deployment.

## What It Does

- **Scaffold agent projects**: Create new ADK agent projects with best practices built-in
- **Local development**: Run and test agents locally with hot reload
- **Evaluation**: Run systematic evaluations with metrics, evalsets, and LLM-as-judge
- **Deployment**: Deploy to Google Cloud (Agent Runtime, Cloud Run, GKE)
- **Publishing**: Register agents with Gemini Enterprise
- **Observability**: Integrate Cloud Trace, logging, and third-party monitoring
- **CI/CD**: Set up staging/prod pipelines with automated testing

## Installation

**Prerequisites**: Python 3.11+, [uv](https://docs.astral.sh/uv/), Node.js

```bash
# Install CLI and skills
uvx google-agents-cli setup

# Or just install the CLI
pip install google-agents-cli

# Verify installation
agents-cli --version
```

## Authentication

```bash
# Authenticate with Google Cloud
agents-cli login

# Check authentication status
agents-cli login --status

# For local development without Google Cloud, use AI Studio API key
export GOOGLE_API_KEY=your_api_key_here
```

## Core Commands

### Project Scaffolding

```bash
# Create a new agent project
agents-cli scaffold my-agent

# Create with specific template
agents-cli scaffold my-agent --template basic
agents-cli scaffold my-agent --template rag

# Add features to existing project
agents-cli scaffold enhance --add deployment
agents-cli scaffold enhance --add cicd
agents-cli scaffold enhance --add rag

# Upgrade project to newer agents-cli version
agents-cli scaffold upgrade
```

### Local Development

```bash
# Install project dependencies
agents-cli install

# Run agent with a single prompt
agents-cli run "What's the weather in San Francisco?"

# Run with file input
agents-cli run --input-file prompt.txt

# Run with streaming output
agents-cli run "Summarize this article" --stream

# Lint code
agents-cli lint
```

### Evaluation

```bash
# Run evaluations
agents-cli eval run

# Run specific evalset
agents-cli eval run --evalset evalsets/basic.yaml

# Compare two evaluation results
agents-cli eval compare results/eval1.json results/eval2.json

# Run with custom metrics
agents-cli eval run --metrics accuracy,latency,cost
```

### Deployment

```bash
# Deploy to Google Cloud (interactive)
agents-cli deploy

# Deploy with specific config
agents-cli deploy --config deploy.yaml

# Deploy to specific environment
agents-cli deploy --env production

# Provision infrastructure
agents-cli infra single-project --project-id my-project
agents-cli infra cicd --project-id my-project

# Set up datastore for RAG
agents-cli infra datastore --project-id my-project
```

### Publishing

```bash
# Register with Gemini Enterprise
agents-cli publish gemini-enterprise

# Publish with metadata
agents-cli publish gemini-enterprise --name "My Agent" --description "Does X"
```

### Data Ingestion (RAG)

```bash
# Run data ingestion pipeline
agents-cli data-ingestion --source gs://my-bucket/docs
agents-cli data-ingestion --source ./local-docs --datastore my-datastore
```

### Utilities

```bash
# Show project info and CLI version
agents-cli info

# Force reinstall skills to all IDEs
agents-cli update
```

## ADK Agent Code Patterns

### Basic Agent Structure

```python
# my_agent.py
from adk.agents import Agent
from adk.tools import Tool
from adk.models import ModelClient

# Define a custom tool
class WeatherTool(Tool):
    """Get weather information for a location."""
    
    def __init__(self):
        super().__init__(
            name="get_weather",
            description="Get current weather for a location"
        )
    
    def execute(self, location: str) -> str:
        # Implementation
        return f"Weather in {location}: Sunny, 72°F"

# Create agent
agent = Agent(
    name="weather-assistant",
    description="An agent that provides weather information",
    model=ModelClient(model_name="gemini-2.0-flash-exp"),
    tools=[WeatherTool()]
)

# Run agent
if __name__ == "__main__":
    response = agent.run("What's the weather in NYC?")
    print(response)
```

### Agent with State Management

```python
from adk.agents import Agent, AgentState
from adk.models import ModelClient
from typing import Any, Dict

class ConversationState(AgentState):
    """Custom state for conversation tracking."""
    
    def __init__(self):
        super().__init__()
        self.conversation_history = []
        self.user_preferences = {}
    
    def add_message(self, role: str, content: str):
        self.conversation_history.append({"role": role, "content": content})

agent = Agent(
    name="stateful-assistant",
    model=ModelClient(model_name="gemini-2.0-flash-exp"),
    state=ConversationState()
)

# Use state in agent execution
response = agent.run("Remember my name is Alice")
agent.state.user_preferences["name"] = "Alice"
```

### Multi-Agent Orchestration

```python
from adk.agents import Agent, AgentOrchestrator
from adk.models import ModelClient

# Create specialized agents
research_agent = Agent(
    name="researcher",
    description="Researches topics and gathers information",
    model=ModelClient(model_name="gemini-2.0-flash-exp")
)

writer_agent = Agent(
    name="writer",
    description="Writes content based on research",
    model=ModelClient(model_name="gemini-2.0-flash-exp")
)

# Orchestrate agents
orchestrator = AgentOrchestrator(
    agents=[research_agent, writer_agent],
    workflow="sequential"  # or "parallel", "conditional"
)

# Run orchestrated workflow
result = orchestrator.run("Write an article about AI agents")
```

### Agent with Callbacks

```python
from adk.agents import Agent
from adk.callbacks import Callback
from adk.models import ModelClient

class LoggingCallback(Callback):
    """Log agent execution steps."""
    
    def on_agent_start(self, agent_name: str, input_data: Any):
        print(f"Agent {agent_name} starting with input: {input_data}")
    
    def on_tool_start(self, tool_name: str, tool_input: Dict[str, Any]):
        print(f"Tool {tool_name} called with: {tool_input}")
    
    def on_tool_end(self, tool_name: str, tool_output: Any):
        print(f"Tool {tool_name} returned: {tool_output}")
    
    def on_agent_end(self, agent_name: str, output: Any):
        print(f"Agent {agent_name} finished with: {output}")

agent = Agent(
    name="monitored-agent",
    model=ModelClient(model_name="gemini-2.0-flash-exp"),
    callbacks=[LoggingCallback()]
)
```

### RAG Agent Pattern

```python
from adk.agents import Agent
from adk.tools import Tool
from adk.models import ModelClient
from adk.rag import VectorStore, Retriever

class RAGTool(Tool):
    """Retrieve relevant documents from vector store."""
    
    def __init__(self, datastore_id: str):
        super().__init__(
            name="retrieve_docs",
            description="Retrieve relevant documents"
        )
        self.retriever = Retriever(datastore_id=datastore_id)
    
    def execute(self, query: str) -> str:
        docs = self.retriever.retrieve(query, top_k=5)
        return "\n\n".join([doc.content for doc in docs])

agent = Agent(
    name="rag-assistant",
    description="Agent with RAG capabilities",
    model=ModelClient(model_name="gemini-2.0-flash-exp"),
    tools=[RAGTool(datastore_id="my-datastore")]
)
```

## Project Configuration

### `agents.yaml`

```yaml
# Project configuration
name: my-agent
version: 1.0.0
description: My AI agent

# Agent configuration
agent:
  name: my-assistant
  model: gemini-2.0-flash-exp
  temperature: 0.7
  max_tokens: 2048

# Tools configuration
tools:
  - name: web_search
    enabled: true
  - name: code_execution
    enabled: false

# Evaluation configuration
evaluation:
  evalsets:
    - path: evalsets/basic.yaml
    - path: evalsets/advanced.yaml
  metrics:
    - accuracy
    - latency
    - cost

# Deployment configuration
deployment:
  target: cloud-run
  region: us-central1
  min_instances: 1
  max_instances: 10
  
# Observability
observability:
  cloud_trace: true
  cloud_logging: true
```

### Evalset Configuration

```yaml
# evalsets/basic.yaml
name: basic-evalset
description: Basic functionality tests

test_cases:
  - id: tc-001
    input: "What is 2+2?"
    expected_output: "4"
    metrics:
      - accuracy
      - latency
  
  - id: tc-002
    input: "Explain quantum computing in simple terms"
    judge:
      type: llm-as-judge
      criteria:
        - clarity
        - accuracy
        - conciseness
    
  - id: tc-003
    input: "Write a Python function to reverse a string"
    validator:
      type: code-execution
      test: |
        def test_reverse():
            assert reverse("hello") == "olleh"
            assert reverse("") == ""
```

### Deployment Configuration

```yaml
# deploy.yaml
target: cloud-run
project_id: ${GCP_PROJECT_ID}
region: us-central1

service:
  name: my-agent-service
  min_instances: 1
  max_instances: 10
  cpu: 2
  memory: 4Gi
  timeout: 300s

environment:
  - name: GOOGLE_API_KEY
    secret: projects/${GCP_PROJECT_ID}/secrets/gemini-api-key
  - name: LOG_LEVEL
    value: INFO

ci_cd:
  enabled: true
  environments:
    - name: staging
      project_id: ${STAGING_PROJECT_ID}
      branch: develop
    - name: production
      project_id: ${PRODUCTION_PROJECT_ID}
      branch: main
```

## Environment Variables

```bash
# API Keys
export GOOGLE_API_KEY=your_api_key_here
export GOOGLE_CLOUD_PROJECT=your-project-id

# Agent Configuration
export AGENT_MODEL=gemini-2.0-flash-exp
export AGENT_TEMPERATURE=0.7
export AGENT_MAX_TOKENS=2048

# Deployment
export DEPLOY_REGION=us-central1
export DEPLOY_ENV=production

# Observability
export ENABLE_CLOUD_TRACE=true
export ENABLE_CLOUD_LOGGING=true
export LOG_LEVEL=INFO

# RAG
export DATASTORE_ID=my-datastore
export VECTOR_STORE_TYPE=vertex-ai-search
```

## Common Patterns

### Creating a Complete Agent Project

```bash
# 1. Scaffold project
agents-cli scaffold my-agent

# 2. Navigate to project
cd my-agent

# 3. Install dependencies
agents-cli install

# 4. Edit agent code
# Edit src/agent.py with your agent logic

# 5. Run locally
agents-cli run "Test prompt"

# 6. Create evalset
# Create evalsets/test.yaml

# 7. Run evaluations
agents-cli eval run

# 8. Deploy to Google Cloud
agents-cli login
agents-cli deploy

# 9. Publish to Gemini Enterprise
agents-cli publish gemini-enterprise
```

### Adding Tools to an Agent

```python
from adk.agents import Agent
from adk.tools import Tool
from adk.models import ModelClient
import requests

class SearchTool(Tool):
    """Search the web for information."""
    
    def __init__(self, api_key: str):
        super().__init__(
            name="web_search",
            description="Search the web for current information"
        )
        self.api_key = api_key
    
    def execute(self, query: str, num_results: int = 5) -> str:
        # Implementation using search API
        # Use self.api_key from environment
        results = self._search(query, num_results)
        return "\n".join([r["title"] + ": " + r["snippet"] for r in results])
    
    def _search(self, query: str, num_results: int):
        # Actual search implementation
        pass

class CalculatorTool(Tool):
    """Perform mathematical calculations."""
    
    def __init__(self):
        super().__init__(
            name="calculator",
            description="Perform mathematical calculations"
        )
    
    def execute(self, expression: str) -> float:
        # Safe evaluation of mathematical expressions
        import ast
        import operator
        
        operators = {
            ast.Add: operator.add,
            ast.Sub: operator.sub,
            ast.Mult: operator.mul,
            ast.Div: operator.truediv,
        }
        
        def eval_expr(node):
            if isinstance(node, ast.Num):
                return node.n
            elif isinstance(node, ast.BinOp):
                return operators[type(node.op)](
                    eval_expr(node.left),
                    eval_expr(node.right)
                )
            else:
                raise ValueError("Unsupported expression")
        
        return eval_expr(ast.parse(expression, mode='eval').body)

# Create agent with multiple tools
import os

agent = Agent(
    name="research-assistant",
    model=ModelClient(model_name="gemini-2.0-flash-exp"),
    tools=[
        SearchTool(api_key=os.getenv("SEARCH_API_KEY")),
        CalculatorTool()
    ]
)
```

### Setting Up CI/CD

```bash
# 1. Set up CI/CD infrastructure
agents-cli infra cicd --project-id my-project

# 2. Create deployment configs for staging and prod
# Edit deploy-staging.yaml and deploy-production.yaml

# 3. Configure GitHub Actions or Cloud Build
# The scaffold enhance command creates templates

# 4. Push to repository
git add .
git commit -m "Set up CI/CD"
git push origin main
```

### Implementing Custom Metrics

```python
# evalsets/custom_metrics.py
from adk.evaluation import Metric
from typing import Dict, Any

class CustomAccuracyMetric(Metric):
    """Custom accuracy metric with fuzzy matching."""
    
    def __init__(self, threshold: float = 0.8):
        super().__init__(name="custom_accuracy")
        self.threshold = threshold
    
    def evaluate(self, 
                 prediction: str, 
                 expected: str, 
                 context: Dict[str, Any]) -> float:
        # Custom evaluation logic
        from difflib import SequenceMatcher
        ratio = SequenceMatcher(None, prediction, expected).ratio()
        return 1.0 if ratio >= self.threshold else 0.0

class LatencyMetric(Metric):
    """Measure agent response latency."""
    
    def __init__(self):
        super().__init__(name="latency")
    
    def evaluate(self,
                 prediction: str,
                 expected: str,
                 context: Dict[str, Any]) -> float:
        # Return latency in milliseconds
        return context.get("execution_time_ms", 0)

# Use in evaluation
from adk.evaluation import Evaluator

evaluator = Evaluator(
    agent=my_agent,
    evalset_path="evalsets/basic.yaml",
    metrics=[
        CustomAccuracyMetric(threshold=0.85),
        LatencyMetric()
    ]
)

results = evaluator.run()
print(f"Average accuracy: {results.metrics['custom_accuracy']}")
print(f"Average latency: {results.metrics['latency']}ms")
```

## Troubleshooting

### Authentication Issues

```bash
# Clear credentials and re-authenticate
agents-cli login --force

# Use service account for CI/CD
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
agents-cli login --service-account

# Check authentication status
agents-cli login --status
```

### Deployment Failures

```bash
# Check project configuration
agents-cli info

# Validate deployment config
agents-cli deploy --validate-only

# Deploy with verbose logging
agents-cli deploy --verbose

# Check Cloud Run logs
gcloud run services logs read my-agent-service --project=${GCP_PROJECT_ID}
```

### Evaluation Issues

```bash
# Run single test case
agents-cli eval run --test-case tc-001

# Run with debug mode
agents-cli eval run --debug

# Validate evalset format
agents-cli eval run --validate-only

# Check evaluation logs
cat .agents-cli/eval-results/latest.log
```

### Dependency Conflicts

```bash
# Reinstall dependencies
agents-cli install --force

# Clear cache
rm -rf .agents-cli/cache
agents-cli install

# Use specific Python version
uv venv --python 3.11
source .venv/bin/activate
agents-cli install
```

### Model Access Issues

```bash
# Verify API key is set
echo $GOOGLE_API_KEY

# Test model access directly
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=${GOOGLE_API_KEY}" \
  -H 'Content-Type: application/json' \
  -d '{"contents":[{"parts":[{"text":"Hello"}]}]}'

# Switch to different model
export AGENT_MODEL=gemini-1.5-pro
agents-cli run "test"
```

### Performance Optimization

```python
# Enable caching for repeated queries
from adk.models import ModelClient

model = ModelClient(
    model_name="gemini-2.0-flash-exp",
    cache_enabled=True,
    cache_ttl=3600  # 1 hour
)

# Batch processing for evaluations
agents-cli eval run --batch-size 10 --parallel 4

# Reduce token usage
agent = Agent(
    name="efficient-agent",
    model=ModelClient(
        model_name="gemini-2.0-flash-exp",
        max_tokens=1024,  # Lower limit
        temperature=0.3   # More deterministic
    )
)
```

### Debugging Agent Behavior

```python
from adk.agents import Agent
from adk.callbacks import DebugCallback
from adk.models import ModelClient

# Enable detailed debugging
debug_callback = DebugCallback(
    log_prompts=True,
    log_responses=True,
    log_tool_calls=True
)

agent = Agent(
    name="debug-agent",
    model=ModelClient(model_name="gemini-2.0-flash-exp"),
    callbacks=[debug_callback],
    verbose=True
)

# Check agent execution traces
response = agent.run("Debug this behavior")
print(agent.get_trace())
```

## Model Selection Guide

```python
# Fast, cost-effective for simple tasks
ModelClient(model_name="gemini-2.0-flash-exp")

# Balanced performance and capabilities
ModelClient(model_name="gemini-1.5-pro")

# Maximum capabilities for complex reasoning
ModelClient(model_name="gemini-1.5-pro-002")

# Multimodal tasks (images, video)
ModelClient(
    model_name="gemini-1.5-pro-vision",
    multimodal=True
)
```
