# Quick Start Guide

This guide gets PrismBench running locally with the current microservice stack.

## Prerequisites

- Python 3.12+
- Docker with Compose
- Git

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/PrismBench/PrismBench.git
cd PrismBench
```

### 2. Set up a Python environment (optional but recommended)

```bash
python -m venv .venv
source .venv/bin/activate  # Unix/macOS
# or
.\.venv\Scripts\activate   # Windows
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

## Configuration

### 1. Set up API keys

Create `apis.key` from the template:

```bash
cp apis.key.template apis.key
```

Use `KEY=value` format (no quotes), for example:

```bash
OPENAI_API_KEY=sk-your-openai-key-here
ANTHROPIC_API_KEY=your-anthropic-key-here
DEEPSEEK_API_KEY=your-key-here
TOGETHERAI_API_KEY=your-key-here
LOCAL_AI_BASE_URL=http://ollama:11434/
```

### 2. Select model settings

PrismBench agent configs are role-based files in `configs/agents/*.yaml`.
Edit the agents you plan to use, for example `configs/agents/challenge_designer.yaml`:

```yaml
role: challenge_designer
model_name: gpt-4o-mini
model_provider: openai
api_base: https://api.openai.com/v1/
model_params:
  temperature: 0.8
  max_tokens: 5120
```

## Running PrismBench

### Start services

From repository root:

```bash
docker compose -f docker/docker-compose.yaml up --build
```

This starts four microservices:

- **LLM Interface** (`http://localhost:8000`)
- **Environment** (`http://localhost:8001`)
- **Search** (`http://localhost:8002`)
- **GUI** (`http://localhost:3000`)

### Run a complete evaluation

Use either:

- the GUI at `http://localhost:3000`, or
- the Search API (`/initialize`, `/run`, `/tasks/{task_id}`) at `http://localhost:8002`.

## Next Steps

- [Configuration Overview](Configuration-Overview) - Detailed configuration options
- [Architecture Overview](Architecture-Overview) - Service and data flow details
- [MCTS Algorithm](MCTS-Algorithm) - Search process details
- [Results Analysis](Results-Analysis) - Interpreting outputs

## Troubleshooting

For common setup and runtime issues, see [troubleshooting](troubleshooting).

---

## Related Pages

### **Next Steps**
- [Configuration Overview](Configuration-Overview) - Detailed configuration options
- [Architecture Overview](Architecture-Overview) - Understanding system components
- [Agent System](Agent-System) - How agents work together

### **Core Concepts**
- [MCTS Algorithm](MCTS-Algorithm) - Understanding the core algorithm
- [Environment System](Environment-System) - Evaluation environments
- [Results Analysis](Results-Analysis) - Interpreting your results

### **Advanced Usage**
- [Extending PrismBench](Extending-PrismBench) - Customizing the framework
- [Custom Agents](Custom-Agents) - Creating specialized agents
- [troubleshooting](troubleshooting) - Common issues and solutions
