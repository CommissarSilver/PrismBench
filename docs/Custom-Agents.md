# Custom Agents

> Creating custom LLM agents with specialized prompts, behaviors, and interaction patterns

Custom agents allow you to extend PrismBench with domain-specific expertise, specialized prompts, and tailored interaction patterns. Agents are the building blocks of all environment workflows.

## Overview

Agents in PrismBench are defined through **YAML configuration files** that specify:

- **LLM Configuration**: Model, provider, and parameters
- **System Prompts**: Specialized instructions and expertise
- **Interaction Templates**: Structured input/output patterns
- **Output Formatting**: Response parsing and extraction

---

## Agent Architecture

### **Configuration Structure**

```yaml
agent_name: my_custom_agent

configs:
  model_name: gpt-4o-mini
  provider: openai
  params:
    temperature: 0.7
    max_tokens: 2048
  local: false

system_prompt: >
  Your specialized agent instructions here...

interaction_templates:
  - name: basic
    required_keys: [input_param1, input_param2]
    template: >
      Template with {input_param1} and {input_param2}
    output_format:
      response_begin: <tag>
      response_end: </tag>
```

### **Component Breakdown**

| Component | Purpose | Required |
|-----------|---------|----------|
| `agent_name` | Unique identifier for the agent | Yes |
| `configs` | LLM provider and model settings | Yes |
| `system_prompt` | Agent's expertise and instructions | Yes |
| `interaction_templates` | Input/output patterns | Yes |

---

## Creating Custom Agents

### **Step 1: Agent Configuration**

Create a new YAML file in `configs/agents/`:

```yaml
agent_name: domain_expert

configs:
  model_name: gpt-4o-mini
  provider: openai
  params:
    temperature: 0.8
    max_tokens: 4096
  local: false
```

**Supported Providers:**
- `openai` - OpenAI models (GPT-4, GPT-3.5, etc.)
- `anthropic` - Anthropic models (Claude)
- `together` - Together AI models
- `deepseek` - DeepSeek models
- Custom providers via LangChain

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
  - name: analyze
    required_keys: [data, criteria]
    template: >
      Analyze the following data: {data}
      
      Use these criteria: {criteria}
      
      Provide your analysis with reasoning.
    output_format:
      response_begin: <analysis>
      response_end: </analysis>
      
  - name: generate
    required_keys: [requirements, constraints]
    template: >
      Generate content based on:
      Requirements: {requirements}
      Constraints: {constraints}
    output_format:
      response_begin: <generated_content>
      response_end: </generated_content>
```

---

## Complete Example: Problem Solver Agent

```yaml
agent_name: problem_solver

configs:
  model_name: gpt-4o-mini
  provider: openai
  params:
    temperature: 0.8
    max_tokens: 5120
  local: false

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

  **IMPORTANT**: You must enclose the entire solution code you generate within <generated_solution> and </generated_solution>
  delimiters. This is crucial for extracting the solution from your output.

  **IMPORTANT**: You must name the generated function as 'solution'. This is used for extracting the output.

  **IMPORTANT**: The entire solution must be enclosed in the 'solution' function.

interaction_templates:
  - name: basic
    required_keys: [problem_statement]
    template: >
      Generate a solution for the following problem: {problem_statement}
    output_format:
      response_begin: <generated_solution>
      response_end: </generated_solution>
  - name: fix
    required_keys: [error_feedback]
    template: >
      Previous solution attempt failed. Here is the error feedback: {error_feedback}. Try again.
    output_format:
      response_begin: <generated_solution>
      response_end: </generated_solution>

```

---

## Advanced Agent Features

### **Multi-Template Agents**

Agents can have multiple interaction patterns:

```yaml
interaction_templates:
  - name: basic
    required_keys: [input]
    template: "Process: {input}"
    output_format:
      response_begin: <result>
      response_end: </result>
      
  - name: detailed
    required_keys: [input, context, requirements]
    template: >
      Input: {input}
      Context: {context}
      Requirements: {requirements}
      
      Provide detailed analysis.
    output_format:
      response_begin: <detailed_result>
      response_end: </detailed_result>
      
  - name: fix_errors
    required_keys: [original_work, error_feedback]
    template: >
      Original work: {original_work}
      Error feedback: {error_feedback}
      
      Provide corrected version.
    output_format:
      response_begin: <corrected_work>
      response_end: </corrected_work>
```

### **Provider-Specific Configuration**

Customize for different LLM providers:

```yaml
# OpenAI Configuration
configs:
  model_name: gpt-4o-mini
  provider: openai
  params:
    temperature: 0.7
    max_tokens: 2048
    top_p: 0.9
    frequency_penalty: 0.1

# Anthropic Configuration  
configs:
  model_name: claude-3-5-sonnet-20240620
  provider: anthropic
  params:
    temperature: 0.8
    max_tokens: 4096

# Local Model Configuration
configs:
  model_name: llama3.1-70b
  provider: together
  params:
    temperature: 0.6
    max_tokens: 8192
  local: true
```

---

## Testing Custom Agents

### **Manual Testing**

Test your agent using the LLM Interface API:

```python
import requests

# 1. Initialize session
response = requests.post("http://localhost:8000/initialize", 
    json={"role": "math_tutor"})
session_id = response.json()["session_id"]

# 2. Interact with agent
response = requests.post("http://localhost:8000/interact", json={
    "session_id": session_id,
    "input_data": {
        "topic": "quadratic equations",
        "difficulty": "medium", 
        "context": "high school algebra"
    },
    "template_name": "create_problem"
})

task_id = response.json()["task_id"]

# 3. Check results
response = requests.get(f"http://localhost:8000/task_status/{task_id}")
print(response.json()["result"])
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
| Template errors | Missing required keys | Verify template parameter names |
| Output parsing fails | Incorrect delimiters | Check output format configuration |
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
- [Troubleshooting](Troubleshooting) - Agent-related issues and solutions
