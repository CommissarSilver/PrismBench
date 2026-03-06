# PrismBench
> An LLM capability mapping framework for systematic evaluation of language models in computer science problem-solving

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/docs-wiki-green.svg)](https://github.com/CommissarSilver/PrismBench/wiki)
[![Python](https://img.shields.io/badge/python-3.12+-blue.svg)](pyproject.toml)

This branch contains the actively maintained PrismBench framework.  
For the paper replication package, use the [`replication-package`](https://github.com/CommissarSilver/PrismBench/tree/replication-package) branch.

## What Is PrismBench?

PrismBench evaluates LLM coding ability through a three-phase Monte Carlo Tree Search (MCTS) workflow:

- **Phase 1**: Initial capability mapping across concepts and difficulties
- **Phase 2**: Challenge discovery for weak regions of the search space
- **Phase 3**: Deep evaluation of challenging combinations

## Architecture (4 Services)

PrismBench runs as four cooperating services:

- **GUI** (`localhost:3000`) - web interface for starting and monitoring runs
- **Search** (`localhost:8002`) - async MCTS orchestration (`/initialize`, `/run`, `/tasks/{task_id}`)
- **Environment** (`localhost:8001`) - challenge execution runtime (`/run-challenge`)
- **LLM Interface** (`localhost:8000`) - role-based LLM gateway (`/interact`, `/session_history/{id}`)

## Repository Layout

```text
PrismBench/
├── src/services/
│   ├── gui/                    # Next.js frontend
│   ├── search/                 # MCTS orchestrator service
│   ├── environment/            # Challenge execution service
│   └── llm_interface/          # LLM interaction gateway
├── src/analysis/               # Analysis utilities
├── configs/                    # Agent/environment/tree/phase configs
├── docs/                       # Wiki source files
├── docker/                     # Dockerfiles and compose stack
├── Makefile                    # Common dev and runtime commands
└── apis.key.template           # API key template
```

## Quick Start

```bash
git clone https://github.com/CommissarSilver/PrismBench
cd PrismBench

# Show available commands
make help

# Bootstrap environment + keys template
make setup

# Edit apis.key, then start services
make start
```

After startup:

- GUI: <http://localhost:3000>
- Search API docs: <http://localhost:8002/docs>
- Environment API docs: <http://localhost:8001/docs>
- LLM Interface API docs: <http://localhost:8000/docs>

## Docker Build/Run Commands

PrismBench uses `docker/docker-compose.yaml` with a shared base image.

```bash
# Build all services
make build

# Build base image only
make build-base

# Rebuild all services from scratch
make rebuild

# Stop services
make stop
```

## Documentation

- [Wiki](https://github.com/CommissarSilver/PrismBench/wiki)
- [Quick start](https://github.com/CommissarSilver/PrismBench/wiki/Quick-Start)
- [Architecture overview](https://github.com/CommissarSilver/PrismBench/wiki/Architecture-Overview)
- [Configuration overview](https://github.com/CommissarSilver/PrismBench/wiki/Configuration-Overview)

### Service READMEs

- [GUI README](src/services/gui/README.md)
- [Search README](src/services/search/README.md)
- [Environment README](src/services/environment/README.md)
- [LLM Interface README](src/services/llm_interface/README.md)
- [Analysis README](src/analysis/README.md)

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

This project is licensed under the [MIT License](LICENSE).

## Citation

If you use PrismBench in your research, please cite:

```bibtex
@article{
majdinasab2026prismbench,
title={PrismBench: Dynamic and Flexible Benchmarking of LLMs' Code Generation with Monte Carlo Tree Search},
author={Vahid Majdinasab, Amin Nikanjam, Foutse Khomh},
journal={Transactions on Machine Learning Research},
year={2026},
url={https://openreview.net/forum?id=O0bsC6FDly},
}
```
