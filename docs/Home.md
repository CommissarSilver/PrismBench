# PrismBench

**Systematic evaluation of language models through Monte Carlo Tree Search**

[![Documentation](https://img.shields.io/badge/docs-wiki-green.svg)](Home) [![Python](https://img.shields.io/badge/python-3.12+-blue.svg)](https://github.com/PrismBench/PrismBench) [![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/PrismBench/PrismBench/blob/main/LICENSE)

---

## Overview

PrismBench is a comprehensive framework for evaluating Large Language Model capabilities in computer science problem-solving. Using a three-phase Monte Carlo Tree Search approach, it systematically maps model strengths, discovers challenging areas, and provides detailed performance analysis.

**Core Approach:**
- **Phase 1:** Maps initial capabilities across CS concepts
- **Phase 2:** Discovers challenging concept combinations  
- **Phase 3:** Conducts comprehensive evaluation of weaknesses

---

## Getting Started

**New to PrismBench?** Follow our quick start guide to get running in 5 minutes.

**[Quick Start Guide →](Quick-Start)**

**Need detailed setup?** See our comprehensive configuration documentation.

**[Configuration Guide →](Configuration-Overview)**

---

## Core Documentation

### Framework Components

| Component | Description | Documentation |
|-----------|-------------|---------------|
| **MCTS Algorithm** | Three-phase search strategy for capability mapping | [MCTS Algorithm →](MCTS-Algorithm) |
| **Agent System** | Multi-agent architecture for challenge creation and evaluation | [Agent System →](Agent-System) |
| **Environment System** | Pluggable evaluation environments for different scenarios | [Environment System →](Environment-System) |
| **Architecture** | System design and component interactions | [Architecture Overview →](Architecture-Overview) |

### Analysis & Results

| Topic | Description | Documentation |
|-------|-------------|---------------|
| **Results Analysis** | Understanding and interpreting evaluation results | [Results Analysis →](Results-Analysis) |
| **Tree Structure** | Search tree implementation and concept organization | [Tree Structure →](tree-structure) |

---
## Extending PrismBench

PrismBench is designed to be extensible, allowing you to add custom agents, environments, and MCTS phases.

- [Extending PrismBench →](Extending-PrismBench)
- [Custom Agents →](Custom-Agents)
- [Custom Environments →](Custom-Environments)
- [Custom MCTS Phases →](Custom-MCTS-Phases)
- [Extension Combinations →](Extension-Combinations)

---

## System Architecture

PrismBench follows a microservices architecture with four services:

```
┌──────────────┐    ┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ GUI          │    │ Search          │    │ Environment      │    │ LLM Interface   │
│ Port 3000    │◄──►│ Port 8002       │◄──►│ Port 8001        │◄──►│ Port 8000       │
│ Web Frontend │    │ MCTS Engine     │    │ Challenge Exec   │    │ Model Comm      │
└──────────────┘    └─────────────────┘    └──────────────────┘    └─────────────────┘
```

**[Detailed Architecture →](Architecture-Overview)**

---

## Key Features

- **Systematic Evaluation** through MCTS-driven exploration
- **Challenge Discovery** automatically identifies model weaknesses  
- **Comprehensive Analysis** with detailed performance metrics
- **Containerized Deployment** with Docker support
- **API Compatible** with OpenAI-compatible endpoints
- **Extensible Architecture** for custom components

---

## Support

| Resource | Description |
|----------|-------------|
| **[Troubleshooting](troubleshooting)** | Common issues and solutions |
| **[GitHub Discussions](https://github.com/PrismBench/PrismBench/discussions)** | Community support and questions |
| **[Issue Tracker](https://github.com/PrismBench/PrismBench/issues)** | Bug reports and feature requests |

---

## Contributing

We welcome contributions to PrismBench! Whether you're fixing bugs, adding features, or improving documentation, your help is appreciated.

**[Contributing Guide →](https://github.com/PrismBench/PrismBench/blob/main/CONTRIBUTING.md)**

---

## Related Pages

### **Get Started**
- [Quick Start](Quick-Start) - Setup and first run
- [Configuration Overview](Configuration-Overview) - Complete configuration guide
- [Architecture Overview](Architecture-Overview) - System design and components

### **Core Framework**
- [MCTS Algorithm](MCTS-Algorithm) - Monte Carlo Tree Search implementation
- [Agent System](Agent-System) - Multi-agent architecture
- [Environment System](Environment-System) - Evaluation environments

### **Advanced Usage**
- [Extending PrismBench](Extending-PrismBench) - Framework extensions
- [Results Analysis](Results-Analysis) - Understanding evaluation results
- [Troubleshooting](troubleshooting) - Common issues and solutions

---

*Made with enough ☕️ to fell an elephant and a whole lot of ❤️ by `Ahura Majdinasab - SWAT Lab - Polytechnique Montreal`* 
