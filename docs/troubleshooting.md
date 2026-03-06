# Troubleshooting Guide

This guide covers common issues, error messages, and solutions when working with PrismBench. Use this as your first reference when encountering problems.

## Quick Diagnostics

### Health Check Commands

Before diving into specific issues, verify system health:

```bash
# Check all services
curl http://localhost:8000/health  # LLM Interface
curl http://localhost:8001/health  # Environment Service  
curl http://localhost:8002/health  # Search Service

# Check connectivity
ping localhost
telnet localhost 8000
telnet localhost 8001
telnet localhost 8002
```

### Log Analysis

Check service logs for immediate issues:

```bash
# Service logs (if using Docker Compose)
docker compose -f docker/docker-compose.yaml logs llm-interface
docker compose -f docker/docker-compose.yaml logs node-env
docker compose -f docker/docker-compose.yaml logs search
docker compose -f docker/docker-compose.yaml logs gui

# Local development logs
tail -f logs/llm_interface.log
tail -f logs/environment_service.log
tail -f logs/search_service.log
```

## Common Issues by Category

## Installation & Setup Issues

### Issue: Service Won't Start

**Symptoms:**
```bash
ModuleNotFoundError: No module named 'fastapi'
```

**Solution:**
```bash
# Install missing dependencies
pip install -r requirements.txt

# If using virtual environment
python -m venv .env
source .env/bin/activate  # Linux/Mac
.env\Scripts\activate     # Windows
pip install -r requirements.txt
```

### Issue: Port Already in Use

**Symptoms:**
```bash
OSError: [Errno 48] Address already in use: ('0.0.0.0', 8000)
```

**Solution:**
```bash
# Find process using the port
lsof -i :8000  # Linux/Mac
netstat -ano | findstr :8000  # Windows

# Kill the process
kill -9 <PID>  # Linux/Mac
taskkill /PID <PID> /F  # Windows

# Or use different ports
uvicorn main:app --port 8010  # Alternative port
```

### Issue: Configuration File Not Found

**Symptoms:**
```bash
FileNotFoundError: [Errno 2] No such file or directory: 'configs/agents/challenge_designer.yaml'
```

**Solution:**
```bash
# Check current directory
pwd
ls configs/

# Ensure you're in the right directory
cd /path/to/PrismBench
ls configs/agents/

# Copy template configs if missing
cp configs/agents/agent_template.yaml configs/agents/my_agent.yaml
```

## API Keys & Authentication

### Issue: Missing API Keys

**Symptoms:**
```bash
ValueError: OpenAI API key not found
AuthenticationError: Invalid API key provided
```

**Solution:**
```bash
# Create apis.key file
cp apis.key.template apis.key

# Add your API keys
echo "OPENAI_API_KEY=sk-your-key-here" >> apis.key
echo "ANTHROPIC_API_KEY=your-anthropic-key" >> apis.key

# Verify format (no quotes)
cat apis.key
```

### Issue: Invalid API Key Format

**Symptoms:**
```bash
openai.error.AuthenticationError: Invalid API key provided
```

**Solution:**
```bash
# Check key format in apis.key
# Correct format:
OPENAI_API_KEY=sk-proj-abc123...
ANTHROPIC_API_KEY=sk-ant-api03-abc123...

# Incorrect formats:
# OPENAI_API_KEY="sk-proj-abc123..."  # Remove quotes
# OPENAI_API_KEY = sk-proj-abc123...  # Remove spaces around =
# OPENAI_API_KEY = 'sk-proj-abc123...' # Remove quotes and spaces
```

### Issue: API Rate Limits

**Symptoms:**
```bash
openai.error.RateLimitError: Rate limit reached
```

**Solution:**
```python
# Implement exponential backoff in agent configs
model_params:
    retry_attempts: 3
    retry_delay: 2.0
    exponential_backoff: true

# Or reduce concurrency
phase_params:
  num_nodes_per_iteration: 2  # Reduce from 5
```

## Service Communication Issues

### Issue: Service Connection Refused

**Symptoms:**
```bash
ConnectionRefusedError: [Errno 61] Connection refused
requests.exceptions.ConnectionError: HTTPConnectionPool(host='localhost', port=8000)
```

**Solution:**
```bash
# Check if service is running
ps aux | grep uvicorn
curl http://localhost:8000/health

# Start the missing service
cd src/services/llm_interface
uvicorn src.services.llm_interface.src.main:app --reload --port 8000

# Check Docker services
docker compose -f docker/docker-compose.yaml ps
docker compose -f docker/docker-compose.yaml up -d
```

### Issue: Service Timeout

**Symptoms:**
```bash
asyncio.TimeoutError: Request timeout
requests.exceptions.Timeout: HTTPSConnectionPool
```

**Solution:**
```yaml
# Increase timeouts in configs
environment_config:
  timeout: 600  # Increase from 300

phase_configs:
  phase_params:
    task_timeout: 120.0  # Increase timeout
```

### Issue: Session Management Problems

**Symptoms:**
```bash
KeyError: 'session_id'
ValueError: Session not found or expired
```

**Solution:**
```bash
# Check Redis connection
redis-cli ping

# Restart Redis if needed
redis-server

# Clear Redis sessions
redis-cli FLUSHALL

# In Docker
docker compose -f docker/docker-compose.yaml restart redis
```

## Agent Issues

### Issue: Agent Configuration Error

**Symptoms:**
```bash
FileNotFoundError: configs/agents/unknown_agent.yaml
AgentConfigurationError: Invalid agent configuration
```

**Solution:**
```bash
# List available agents
ls configs/agents/

# Check agent name in environment config
cat configs/environment_config.yaml

# Ensure agent file exists and matches name
# File: configs/agents/challenge_designer.yaml
# Config: role: challenge_designer
```

### Issue: Agent Response Parsing Error

**Symptoms:**
```bash
ValueError: No matching interaction template for input_data keys
ValidationError: response field missing from template outputs
```

**Solution:**
```yaml
# Check interaction template input/output contract
interaction_templates:
  default:
    inputs:
      - name: concepts
      - name: difficulty_level
    outputs:
      - name: response

# Ensure request keys exactly match one template's input names
```

### Issue: Agent Model Not Available

**Symptoms:**
```bash
openai.error.InvalidRequestError: The model 'gpt-5' does not exist
```

**Solution:**
```yaml
# Use available models in agent configs
model_name: gpt-4o-mini  # Available
# model_name: gpt-5      # Not available
# model_name: claude-4   # Not available
```

## Environment Issues

### Issue: Environment Not Found

**Symptoms:**
```bash
KeyError: 'custom_environment'
EnvironmentError: Environment 'custom_environment' not registered
```

**Solution:**
```python
# Check environment registration
from src.environment.environment_registry import environment_registry
print(environment_registry.list_environments())

# Ensure environment file is imported
# In your environment module:
import src.environment.environment_custom  # This registers the environment

# Or check environment_config.yaml
cat configs/environment_config.yaml
```

### Issue: Code Execution Failures

**Symptoms:**
```bash
subprocess.CalledProcessError: Command returned non-zero exit status 1
TimeoutError: Test execution timeout
```

**Solution:**
```python
# Check test code syntax
def test_code_validation():
    try:
        compile(test_code, '<string>', 'exec')
        return True
    except SyntaxError as e:
        print(f"Syntax error: {e}")
        return False

# Increase execution timeout
environment_config:
  timeout: 600  # Increase timeout

# Check for infinite loops in test code
# Add timeout wrapper in test execution
```

### Issue: Temporary Directory Issues

**Symptoms:**
```bash
PermissionError: [Errno 13] Permission denied: '/tmp/prism_env_'
OSError: [Errno 28] No space left on device
```

**Solution:**
```bash
# Check disk space
df -h

# Clean up old temporary directories
find /tmp -name "prism_env_*" -type d -mtime +1 -exec rm -rf {} +

# Check permissions
ls -la /tmp/
chmod 755 /tmp

# Set custom temp directory
export TMPDIR=/custom/temp/path
```

## Search & MCTS Issues

### Issue: Tree Construction Failures

**Symptoms:**
```bash
ValueError: Invalid tree configuration
TreeConstructionError: Unable to create root node
```

**Solution:**
```yaml
# Check tree_configs.yaml
tree_configs:
  concepts:
    - loops        # Must be strings
    - conditionals
  difficulties:
    - easy         # Must be strings
    - medium

# Verify concepts and difficulties are lists
# Ensure no duplicate entries
```

### Issue: Phase Execution Errors

**Symptoms:**
```bash
PhaseExecutionError: Phase 'phase_2' failed to converge
ValueError: No nodes available for selection
```

**Solution:**
```yaml
# Adjust phase parameters
phase_2:
  phase_params:
    max_iterations: 200      # Increase iterations
    convergence_checks: 3    # Reduce convergence requirement
    value_delta_threshold: 0.4  # Relax threshold

# Check if Phase 1 created sufficient nodes
# Verify node selection criteria
```

### Issue: Memory Issues with Large Trees

**Symptoms:**
```bash
MemoryError: Unable to allocate memory
TreeError: Maximum tree size exceeded
```

**Solution:**
```python
# Implement tree pruning
def prune_tree(tree, max_nodes=1000):
    if tree.count_nodes() > max_nodes:
        # Remove low-value nodes
        tree.prune_nodes(value_threshold=0.1, visit_threshold=2)

# Reduce tree growth
phase_params:
  max_depth: 5         # Reduce from 10
  max_iterations: 50   # Reduce iterations
```

## Analysis & Results Issues

### Issue: Results File Corruption

**Symptoms:**
```bash
json.JSONDecodeError: Expecting value: line 1 column 1 (char 0)
PickleError: Invalid pickle data
```

**Solution:**
```bash
# Check file integrity
file results.json
head -10 results.json

# Restore from backup
cp results.backup.json results.json

# Validate JSON structure
python -m json.tool results.json

# Re-run analysis if needed
python -m src.analysis.regenerate_results
```

### Issue: Visualization Failures

**Symptoms:**
```bash
ModuleNotFoundError: No module named 'matplotlib'
ValueError: Empty dataset for visualization
```

**Solution:**
```bash
# Install visualization dependencies
pip install matplotlib seaborn plotly

# Check data availability
python -c "
import json
with open('results.json') as f:
    data = json.load(f)
    print(f'Data points: {len(data)}')
"

# Generate minimal dataset if empty
python -m src.analysis.generate_sample_data
```

## Docker Issues

### Issue: Container Build Failures

**Symptoms:**
```bash
ERROR [build 5/8] RUN pip install -r requirements.txt
E: Package 'python3-dev' has no installation candidate
```

**Solution:**
```dockerfile
# Update Dockerfile base image
FROM python:3.12-slim

# Add required system packages
RUN apt-get update && apt-get install -y \
    gcc \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*
```

### Issue: Docker Compose Networking

**Symptoms:**
```bash
ConnectionError: HTTPConnectionPool(host='llm-interface', port=8000)
docker: Error response from daemon: network not found
```

**Solution:**
```yaml
# Check docker/docker-compose.yaml networks
services:
  llm-interface:
    networks:
      - prism-network
  search:
    networks:
      - prism-network

networks:
  prism-network:
    driver: bridge
```

### Issue: Volume Mount Problems

**Symptoms:**
```bash
PermissionError: [Errno 13] Permission denied: '/app/configs'
```

**Solution:**
```bash
# Fix file permissions
chmod -R 755 configs/
chown -R 1000:1000 configs/  # If using specific user

# Update docker-compose.yml
volumes:
  - ../configs:/app/configs:ro  # Read-only mount
  - ../data:/app/data:rw        # Read-write mount
```

## Debugging Strategies

### Enable Debug Logging

```python
import logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Service-specific debugging
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)
```

### Trace Request Flow

```python
# Add correlation IDs to requests
import uuid

def add_correlation_id():
    correlation_id = str(uuid.uuid4())
    logger.info(f"Request started: {correlation_id}")
    return correlation_id

# Use in error handling
try:
    result = await service_call()
except Exception as e:
    logger.error(f"Error in {correlation_id}: {str(e)}", exc_info=True)
```

### Validate Configuration

```python
# Configuration validation script
def validate_config():
    errors = []
    
    # Check API keys
    if not os.path.exists('apis.key'):
        errors.append("Missing apis.key file")
    
    # Check agent configs
    for agent_file in glob.glob('configs/agents/*.yaml'):
        try:
            with open(agent_file) as f:
                config = yaml.safe_load(f)
                if 'role' not in config:
                    errors.append(f"Missing role in {agent_file}")
        except yaml.YAMLError as e:
            errors.append(f"Invalid YAML in {agent_file}: {e}")
    
    return errors
```

## Getting Help

### Collecting Debug Information

When reporting issues, include:

```bash
# System information
python --version
pip list | grep -E "(fastapi|uvicorn|pydantic)"

# Service status
curl -s http://localhost:8000/health | jq .
curl -s http://localhost:8001/health | jq .
curl -s http://localhost:8002/health | jq .

# Configuration summary
ls -la configs/
cat configs/tree_configs.yaml
cat configs/environment_config.yaml

# Recent logs (last 50 lines)
tail -50 logs/service.log
```

### Error Report Template

```markdown
## Bug Report

**Environment:**
- OS: [e.g., Ubuntu 20.04, macOS 12.0, Windows 11]
- Python Version: [e.g., 3.11.2]
- PrismBench Version: [e.g., v1.0.0]

**Expected Behavior:**
[Describe what you expected to happen]

**Actual Behavior:**
[Describe what actually happened]

**Error Message:**
```
[Paste the full error message/stack trace]
```

**Steps to Reproduce:**
1. [First step]
2. [Second step]
3. [Error occurs]

**Configuration:**
[Paste relevant configuration files]

**Logs:**
[Paste relevant log entries]
```

### Community Resources

- **Documentation**: [PrismBench Wiki](../README.md)
- **Configuration Examples**: `configs/` directory
- **Code Examples**: `examples/` directory

### Self-Help Checklist

Before seeking help, verify:

- [] All services are running and healthy
- [] API keys are correctly configured
- [] Configuration files are valid YAML
- [] Required Python packages are installed
- [] Port conflicts are resolved
- [] Sufficient disk space available
- [] Network connectivity between services
- [] Log files checked for specific errors

---

**Related Documentation:**
- [Quick Start](Quick-Start) - Basic setup guide
- [Configuration Overview](Configuration-Overview) - Detailed configuration
- [Architecture Overview](Architecture-Overview) - System architecture
- [Extending PrismBench](Extending-PrismBench) - Recommended approaches

---

## Related Pages

### **Setup & Configuration**
- [Quick Start](Quick-Start) - Basic setup and installation guide
- [Configuration Overview](Configuration-Overview) - Detailed configuration system
- [Architecture Overview](Architecture-Overview) - System architecture and components

### **Extension Development**
- [Custom Agents](Custom-Agents) - Agent-related troubleshooting
- [Custom Environments](Custom-Environments) - Environment issues and solutions
- [Custom MCTS Phases](Custom-MCTS-Phases) - Phase implementation problems

### **Core Systems**
- [Agent System](Agent-System) - Agent architecture and integration
- [Environment System](Environment-System) - Environment framework issues
- [MCTS Algorithm](MCTS-Algorithm) - Algorithm-specific problems
- [Results Analysis](Results-Analysis) - Analysis and visualization issues

### **Advanced Topics**
- [Extending PrismBench](Extending-PrismBench) - Framework extension issues
- [Extension Combinations](Extension-Combinations) - Complex setup troubleshooting 
