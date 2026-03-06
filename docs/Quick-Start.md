# Quick Start Guide

This guide will get you up and running with PrismBench in under 5 minutes.

## Prerequisites

- Python 3.12 or higher
- Docker and Docker Compose
- Git

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/PrismBench/PrismBench.git
cd PrismBench
```

### 2. Set Up Python Environment (Recommended but not necessary)

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
source venv/bin/activate  # Unix/macOS
# or
.\venv\Scripts\activate   # Windows
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

## Configuration

### Setting up API Keys

PrismBench supports any model that follows the [OpenAI text generation standard](https://platform.openai.com/docs/guides/text-generation). This includes OpenAI models, local models through `ollama` or `LMstudio`, and other API providers.

Create an `apis.key` file in the root directory:

```bash
OPENAI_API_KEY = your_openai_api_key
DEEPSEEK_API_KEY = your_deepseek_api_key
CHATLAMMA_API_KEY = your_chatlamma_api_key
LOCAL = your_custom_api_key # only used for ollama and LMstudio
```

### Selecting Models

Configure the model to benchmark in `configs/agents.yml`:

```yaml
model: "gpt-4o-mini"  # Change to your preferred model
# For local models, use: "local"
```

## Running PrismBench

### Start the Services

```bash
docker compose up
```

This starts three main services:

- **LLM Interface** (`localhost:8000`) - Handles model communication
- **Environment** (`localhost:8001`) - Executes code challenges  
- **Search** (`localhost:8002`) - Main MCTS framework entry point

### Run a Complete Evaluation

Access the search service at `localhost:8002` and trigger a full evaluation run through the API endpoints.

## Next Steps

- [Configuration Overview](config-overview.md) - Detailed configuration options
- [MCTS Algorithm](mcts.md) - Understanding the core algorithm
- [Results Analysis](analysis.md) - Interpreting your results

## Troubleshooting

### Common Issues

**Docker containers not starting:**
- Ensure Docker is running
- Check port availability (8000, 8001, 8002)

**API key errors:**
- Verify your `apis.key` file format
- Check API key validity

**Model selection issues:**
- Ensure model name matches API provider format
- For local models, verify ollama/LMstudio is running

For more help, see our [Troubleshooting Guide](troubleshooting.md) or open an issue on GitHub.

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
- [Troubleshooting](Troubleshooting) - Common issues and solutions 