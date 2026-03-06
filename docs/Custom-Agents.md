# Custom Agents

> Creating custom LLM agents with specialized prompts, behaviors, and interaction patterns

Custom agents allow you to extend PrismBench with domain-specific expertise, specialized prompts, and tailored interaction patterns. Agents are the building blocks of all environment workflows.

## Overview

Agents in PrismBench are defined through **YAML configuration files** that specify:

- **LLM Configuration**: Model, provider, and parameters
- **System Prompts**: Specialized instructions and expertise
- **Interaction Templates**: Structured input/output patterns
- **Output Fields**: Structured response contracts

---

## Agent Architecture

### **Configuration Structure**

```yaml
role: my_custom_agent
model_name: gpt-4o-mini
model_provider: openai
api_base: https://api.openai.com/v1/
model_params:
  temperature: 0.7
  max_tokens: 2048

system_prompt: >
  Your specialized agent instructions here...

interaction_templates:
  default:
    inputs:
      - name: input_param1
        type: str
        description: First input
      - name: input_param2
        type: str
        description: Second input
    outputs:
      - name: response
        type: str
        description: Agent response
```

### **Component Breakdown**

| Component | Purpose | Required |
|-----------|---------|----------|
| `role` | Unique identifier for the agent | Yes |
| `model_name`/`model_provider`/`api_base` | LLM routing settings | Yes |
| `model_params` | Model parameter overrides | No |
| `system_prompt` | Agent's expertise and instructions | Yes |
| `interaction_templates` | Input/output patterns | Yes |

---

## Creating Custom Agents

### **Step 1: Agent Configuration**

Create a new YAML file in `configs/agents/`:

```yaml
role: domain_expert
model_name: gpt-4o-mini
model_provider: openai
api_base: https://api.openai.com/v1/
model_params:
  temperature: 0.8
  max_tokens: 4096
```

**Supported Providers:**
- `openai` - OpenAI models (GPT-4, GPT-3.5, etc.)
- `anthropic` - Anthropic models (Claude)
- `openrouter` - OpenRouter-hosted models
- `deepseek` - DeepSeek models
- `togetherai` - Together AI models

### **Step 2: System Prompt Design**

Craft specialized instructions for your domain:

```yaml
system_prompt: >
  You are an expert in [YOUR DOMAIN] with deep knowledge of [SPECIFIC EXPERTISE].
  
  Your responsibilities include:
  1. [Primary responsibility]
  2. [Secondary responsibility]
  3. [Additional capabilities]
  
  Guidelines:
  - [Important guideline 1]
  - [Important guideline 2]
  - [Output format requirements]
  
  Always ensure your responses are [QUALITY CRITERIA].
```

### **Step 3: Interaction Templates**

Define how the agent receives input and formats output:

```yaml
interaction_templates:
  analyze:
    inputs:
      - name: data
        type: str
        description: Data to analyze
      - name: criteria
        type: str
        description: Analysis criteria
    outputs:
      - name: response
        type: str
        description: Analysis output
  generate:
    inputs:
      - name: requirements
        type: str
        description: Output requirements
      - name: constraints
        type: str
        description: Constraints
    outputs:
      - name: response
        type: str
        description: Generated content
```

---

## Complete Example: Problem Solver Agent

```yaml
role: problem_solver
model_name: gpt-4o-mini
model_provider: openai
api_base: https://api.openai.com/v1/
model_params:
  temperature: 0.8
  max_tokens: 5120

system_prompt: >
  You are a skilled programmer with expertise in algorithm design and implementation. Your role is to develop efficient,
  well-structured, and well-commented solutions to complex coding problems. You have a deep understanding of Python and
  its standard libraries, and you're adept at optimizing code for both time and space complexity.

  When given a coding problem, your solution should:
  1. Be implemented as a single Python function named 'solution'
  2. Take input as specified in the problem statement
  3. Return output as specified in the problem statement
  4. Handle all constraints and edge cases mentioned
  5. Be efficient and well-commented
  6. Do not include any code outside of the 'solution' function.

  Ensure your implementation handles all specified constraints and edge cases.
  Write clear, concise comments to explain your approach and any non-obvious parts of the code.

  **IMPORTANT**: You must name the generated function as 'solution'. This is used for extracting the output.

  **IMPORTANT**: The entire solution must be enclosed in the 'solution' function.

interaction_templates:
  default:
    inputs:
      - name: problem_statement
        type: str
        description: Problem to solve
    outputs:
      - name: response
        type: str
        description: Generated Python solution
  fix:
    inputs:
      - name: error_feedback
        type: str
        description: Feedback from previous attempt
    outputs:
      - name: response
        type: str
        description: Corrected Python solution

```

---

## Advanced Agent Features

### **Multi-Template Agents**

Agents can have multiple interaction patterns:

```yaml
interaction_templates:
  basic:
    inputs:
      - name: input
        type: str
        description: Main input
    outputs:
      - name: response
        type: str
        description: Result
  detailed:
    inputs:
      - name: input
        type: str
        description: Main input
      - name: context
        type: str
        description: Additional context
      - name: requirements
        type: str
        description: Output requirements
    outputs:
      - name: response
        type: str
        description: Detailed result
  fix_errors:
    inputs:
      - name: original_work
        type: str
        description: Previous output
      - name: error_feedback
        type: str
        description: Error feedback
    outputs:
      - name: response
        type: str
        description: Corrected output
```

### **Provider-Specific Configuration**

Customize for different LLM providers:

```yaml
# OpenAI Configuration
model_name: gpt-4o-mini
model_provider: openai
api_base: https://api.openai.com/v1/
model_params:
  temperature: 0.7
  max_tokens: 2048
  top_p: 0.9
  frequency_penalty: 0.1

# Anthropic Configuration  
model_name: claude-3-5-sonnet-20240620
model_provider: openrouter
api_base: https://openrouter.ai/api/v1
model_params:
  temperature: 0.8
  max_tokens: 4096

# Local Model Configuration
model_name: llama3.1-70b
model_provider: togetherai
api_base: https://api.together.xyz/v1
model_params:
  temperature: 0.6
  max_tokens: 8192
```

---

## Testing Custom Agents

### **Manual Testing**

Test your agent using the LLM Interface API:

```python
import requests

# 1. Use your own session id
session_id = "manual-test-math-tutor-001"

# 2. Interact with agent
response = requests.post("http://localhost:8000/interact", json={
    "session_id": session_id,
    "role": "math_tutor",
    "input_data": {
        "topic": "quadratic equations",
        "difficulty": "medium", 
        "context": "high school algebra"
    },
    "use_agent": False
})
print(response.json()["message"])

# 3. (Optional) inspect stored conversation history
history = requests.get(f"http://localhost:8000/session_history/{session_id}")
print(history.json())
```

### **Integration Testing**

Test within an environment:

```python
from src.services.environment.src.environment.utils import create_environment

environment = create_environment(
    "my_custom_environment",
    agents=["math_tutor", "solution_validator"]
)

await environment.initialize()
results = await environment.execute_node(
    topic="calculus",
    difficulty="hard"
)
```

---

## Best Practices

### **System Prompt Design**

- **Be Specific**: Clear, detailed instructions work better than vague guidance
- **Include Examples**: Provide concrete examples of expected output
- **Set Boundaries**: Explicitly state what the agent should and shouldn't do
- **Format Requirements**: Specify exact output format expectations

### **Template Design**

- **Minimize Required Keys**: Only require essential parameters
- **Use Descriptive Names**: Template names should clearly indicate their purpose
- **Consistent Formatting**: Use consistent parameter naming across templates
- **Clear Output Delimiters**: Use unique, easily parseable delimiters

### **Configuration Tips**

- **Temperature Settings**: Lower (0.3-0.5) for deterministic tasks, higher (0.7-0.9) for creative tasks
- **Token Limits**: Set appropriate limits based on expected output length
- **Provider Choice**: Consider cost, performance, and availability
- **Testing**: Test extensively with edge cases and various inputs

---

## Troubleshooting

### **Common Issues**

| Issue | Cause | Solution |
|-------|-------|----------|
| Agent not found | File not in `configs/agents/` | Check file location and naming |
| Template errors | Input/output fields mismatch | Verify `interaction_templates` inputs and outputs |
| Output parsing fails | Missing `response` output field | Ensure template outputs include a `response` field |
| API errors | Invalid provider config | Verify API keys and model names |

### **Debugging Tips**

1. **Check Logs**: Review LLM Interface service logs for errors
2. **Test Templates**: Validate templates with simple inputs first
3. **Verify Config**: Ensure YAML syntax is correct
4. **API Testing**: Test provider connection separately

---

## Next Steps

- **[Custom Environments](Custom-Environments)** - Orchestrate multiple agents
- **[Custom MCTS Phases](Custom-MCTS-Phases)** - Use agents in search strategies  
- **[Extension Combinations](Extension-Combinations)** - Combine agents with other extensions

---

## Related Pages

### **Extension Development**
- [Custom Environments](Custom-Environments) - Orchestrate multiple agents in workflows
- [Custom MCTS Phases](Custom-MCTS-Phases) - Use agents in search strategies
- [Extension Combinations](Extension-Combinations) - Combine agents with other extensions

### **Agent System**
- [Agent System](Agent-System) - Understanding the agent architecture
- [Configuration Overview](Configuration-Overview) - Agent configuration system
- [Architecture Overview](Architecture-Overview) - How agents fit in the framework

### **Implementation**
- [Extending PrismBench](Extending-PrismBench) - Framework extension overview
- [Quick Start](Quick-Start) - Getting started with the framework
- [Troubleshooting](troubleshooting) - Agent-related issues and solutions
