# Custom Environments

> Building custom evaluation environments with specialized workflows and agent orchestration

Custom environments enable you to create sophisticated evaluation scenarios by orchestrating multiple agents in complex workflows. Environments define how agents interact to solve problems in your domain.

## Overview

Environments in PrismBench coordinate **multi-agent workflows** for evaluation tasks:

- **Agent Orchestration**: Coordinate multiple specialized agents
- **Workflow Definition**: Define sequential and parallel agent interactions
- **Resource Management**: Handle temporary files, processes, and cleanup
- **Result Integration**: Combine outputs from multiple agents
- **Error Handling**: Robust failure recovery and retry logic

---

## Environment Architecture

### **Registry Pattern**

Environments use a **decorator-based registry** for automatic discovery:

```python
from src.environment.environment_registry import environment_registry

@environment_registry.register_environment_method(
    "my_custom_environment",
    "execute_node"
)
async def execute_node(
    self: "BaseEnvironment",
    concept: str,
    difficulty_level: str,
    **kwargs
) -> Dict:
    """Custom environment implementation."""
    # Your workflow logic here
    pass
```

### **Component Structure**

| Component | Purpose | Responsibility |
|-----------|---------|----------------|
| **BaseEnvironment** | Core framework | Agent management, resource handling |
| **EnvironmentRegistry** | Plugin system | Auto-discovery, method resolution |
| **InterfaceClient** | Agent communication | LLM service interaction |
| **Configuration** | Environment setup | Agent lists, parameters, timeouts |

---

## Creating Custom Environments

### **Step 1: Environment Implementation**

Create a new file `src/services/environment/src/environment/environment_my_domain.py`:

```python
from typing import TYPE_CHECKING, Dict, List
from src.environment.environment_registry import environment_registry

if TYPE_CHECKING:
    from src.environment.base_environment import BaseEnvironment

@environment_registry.register_environment_method(
    "environment_my_domain",
    "execute_node"
)
async def execute_node(
    self: "BaseEnvironment",
    concept: str,
    difficulty_level: str,
    custom_param: str = "default",
    **kwargs
) -> Dict:
    """
    Execute a custom domain evaluation workflow.
    
    Args:
        concept: The domain concept to evaluate
        difficulty_level: Difficulty level for the evaluation
        custom_param: Custom parameter for domain-specific logic
        **kwargs: Additional parameters
        
    Returns:
        Dict: Evaluation results with success status and data trail
    """
    # Initialize environment if needed
    if not self._initialized:
        await self.initialize()
    
    # Step 1: Generate domain-specific problem
    problem = await self.agents["domain_expert"].interact(
        concept=concept,
        difficulty_level=difficulty_level,
        requirements=custom_param
    )
    
    # Step 2: Create evaluation criteria
    criteria = await self.agents["evaluator"].interact(
        problem_statement=problem,
        evaluation_type="comprehensive"
    )
    
    # Step 3: Generate solution
    solution = await self.agents["solver"].interact(
        problem_statement=problem,
        evaluation_criteria=criteria
    )
    
    # Step 4: Validate and score
    validation = await self.agents["validator"].interact(
        problem=problem,
        solution=solution,
        criteria=criteria
    )
    
    return {
        "success": True,
        "data_trail": [{
            "problem": problem,
            "criteria": criteria,
            "solution": solution,
            "validation": validation,
            "concept": concept,
            "difficulty": difficulty_level
        }]
    }
```

### **Step 2: Configuration Setup**

Add your environment to `configs/environment_config.yaml`:

```yaml
environment_configs:
  environment_my_domain:
    agents:
      - domain_expert
      - evaluator 
      - solver
      - validator
    max_attempts: 3
    timeout: 300
    custom_param: "specialized_mode"
    
  # Add domain-specific parameters
  environment_advanced_domain:
    agents:
      - domain_expert
      - secondary_expert
      - solver
      - validator
      - quality_checker
    max_attempts: 5
    timeout: 600
    parallel_evaluation: true
    validation_rounds: 2
```

### **Step 3: Agent Requirements**

Ensure required agents exist in `configs/agents/`:

```yaml
# configs/agents/domain_expert.yaml
agent_name: domain_expert

configs:
  model_name: gpt-4o-mini
  provider: openai
  params:
    temperature: 0.8
    max_tokens: 4096

system_prompt: >
  You are an expert in [YOUR DOMAIN] specializing in creating challenging
  evaluation scenarios that test deep understanding of core concepts.

interaction_templates:
  - name: basic
    required_keys: [concept, difficulty_level, requirements]
    template: >
      Create a challenging problem for: {concept}
      Difficulty: {difficulty_level}
      Requirements: {requirements}
    output_format:
      response_begin: <problem>
      response_end: </problem>
```

---

## Advanced Environment Patterns

### **Parallel Agent Execution**

Execute multiple agents concurrently:

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

@environment_registry.register_environment_method(
    "environment_parallel_evaluation", 
    "execute_node"
)
async def execute_node(
    self: "BaseEnvironment",
    concept: str,
    difficulty_level: str,
    **kwargs
) -> Dict:
    """Environment with parallel agent execution."""
    
    # Generate base problem
    problem = await self.agents["problem_generator"].interact(
        concept=concept,
        difficulty_level=difficulty_level
    )
    
    # Parallel evaluation by multiple agents
    tasks = [
        self.agents["solver_a"].interact(
            problem_statement=problem,
            approach="algorithmic"
        ),
        self.agents["solver_b"].interact(
            problem_statement=problem, 
            approach="heuristic"
        ),
        self.agents["solver_c"].interact(
            problem_statement=problem,
            approach="optimization"
        )
    ]
    
    # Wait for all solutions
    solutions = await asyncio.gather(*tasks, return_exceptions=True)
    
    # Evaluate solutions
    evaluation_tasks = []
    for i, solution in enumerate(solutions):
        if not isinstance(solution, Exception):
            task = self.agents["evaluator"].interact(
                problem=problem,
                solution=solution,
                approach_type=["algorithmic", "heuristic", "optimization"][i]
            )
            evaluation_tasks.append(task)
    
    evaluations = await asyncio.gather(*evaluation_tasks)
    
    return {
        "success": True,
        "data_trail": [{
            "problem": problem,
            "solutions": solutions,
            "evaluations": evaluations,
            "approach": "parallel_evaluation"
        }]
    }
```

### **Multi-Round Workflows**

Implement iterative refinement:

```python
@environment_registry.register_environment_method(
    "environment_iterative_refinement",
    "execute_node"
)
async def execute_node(
    self: "BaseEnvironment",
    concept: str,
    difficulty_level: str,
    max_rounds: int = 3,
    **kwargs
) -> Dict:
    """Environment with iterative solution refinement."""
    
    problem = await self.agents["problem_generator"].interact(
        concept=concept,
        difficulty_level=difficulty_level
    )
    
    solution = None
    data_trail = []
    
    for round_num in range(max_rounds):
        # Generate/refine solution
        if solution is None:
            solution = await self.agents["solver"].interact(
                problem_statement=problem
            )
        else:
            solution = await self.agents["refiner"].interact(
                problem_statement=problem,
                previous_solution=solution,
                feedback=feedback
            )
        
        # Evaluate solution
        evaluation = await self.agents["evaluator"].interact(
            problem=problem,
            solution=solution,
            round=round_num
        )
        
        # Check if satisfactory
        feedback = await self.agents["critic"].interact(
            problem=problem,
            solution=solution,
            evaluation=evaluation
        )
        
        data_trail.append({
            "round": round_num,
            "solution": solution,
            "evaluation": evaluation,
            "feedback": feedback
        })
        
        # Stop if solution is satisfactory
        if "satisfactory" in feedback.lower():
            break
    
    return {
        "success": True,
        "data_trail": data_trail,
        "final_solution": solution,
        "rounds_completed": len(data_trail)
    }
```

### **File-Based Evaluation**

Handle file operations and external tools:

```python
import tempfile
import subprocess
from pathlib import Path

@environment_registry.register_environment_method(
    "environment_code_execution",
    "execute_node"
)
async def execute_node(
    self: "BaseEnvironment",
    concept: str,
    difficulty_level: str,
    **kwargs
) -> Dict:
    """Environment that executes and validates code."""
    
    # Generate coding problem
    problem = await self.agents["code_generator"].interact(
        concept=concept,
        difficulty_level=difficulty_level
    )
    
    # Generate solution
    solution_code = await self.agents["coder"].interact(
        problem_statement=problem
    )
    
    # Generate test cases
    test_cases = await self.agents["test_generator"].interact(
        problem_statement=problem,
        solution_code=solution_code
    )
    
    # Execute code with tests
    with tempfile.TemporaryDirectory() as temp_dir:
        # Write solution to file
        solution_file = Path(temp_dir) / "solution.py"
        solution_file.write_text(solution_code)
        
        # Write test file
        test_file = Path(temp_dir) / "test_solution.py"
        test_file.write_text(test_cases)
        
        # Execute tests
        try:
            result = subprocess.run(
                ["python", str(test_file)],
                cwd=temp_dir,
                capture_output=True,
                text=True,
                timeout=30
            )
            
            execution_success = result.returncode == 0
            output = result.stdout
            errors = result.stderr
            
        except subprocess.TimeoutExpired:
            execution_success = False
            output = ""
            errors = "Execution timeout"
    
    # Analyze results
    analysis = await self.agents["code_analyzer"].interact(
        problem=problem,
        solution=solution_code,
        tests=test_cases,
        execution_output=output,
        execution_errors=errors,
        success=execution_success
    )
    
    return {
        "success": execution_success,
        "data_trail": [{
            "problem": problem,
            "solution_code": solution_code,
            "test_cases": test_cases,
            "execution_output": output,
            "execution_errors": errors,
            "analysis": analysis
        }]
    }
```

---

## Testing and Debugging

### **Environment Testing**

```python
import asyncio
from src.services.environment.src.environment.utils import create_environment

async def test_custom_environment():
    """Test custom environment implementation."""
    
    # Create environment
    environment = create_environment(
        "environment_my_domain",
        agents=["domain_expert", "evaluator", "solver", "validator"]
    )
    
    # Initialize
    await environment.initialize()
    
    # Test execution
    results = await environment.execute_node(
        concept="complex_concept",
        difficulty_level="hard",
        custom_param="test_mode"
    )
    
    # Validate results
    assert results["success"] is True
    assert "data_trail" in results
    assert len(results["data_trail"]) > 0
    
    # Cleanup
    await environment.reset()
    
    print("Environment test passed!")

# Run test
asyncio.run(test_custom_environment())
```

### **Integration Testing**

```python
from src.services.environment.src.services.environment_service import EnvironmentService
from src.services.environment.src.models.requests import ChallengeRequest
from src.services.environment.src.core.config import get_settings

async def test_environment_service():
    """Test environment through service layer."""
    
    settings = get_settings()
    service = EnvironmentService(settings)
    
    request = ChallengeRequest(
        concept="test_concept",
        difficulty_level="medium",
        max_attempts=3
    )
    
    results = await service.run_challenge(
        environment_name="environment_my_domain",
        request=request
    )
    
    print(f"Service test results: {results}")
```

---

## Best Practices

### **Design Principles**

- **Single Responsibility**: Each environment should have a clear, focused purpose
- **Agent Coordination**: Design clear workflows with explicit agent handoffs
- **Error Handling**: Implement robust error recovery and cleanup
- **Resource Management**: Properly manage temporary files and external resources

### **Performance Optimization**

- **Parallel Execution**: Use `asyncio.gather()` for independent operations
- **Timeout Management**: Set appropriate timeouts for agent interactions

### **Configuration Management**

- **Parameter Validation**: Validate all input parameters early
- **Default Values**: Provide sensible defaults for optional parameters
- **Documentation**: Document all configuration options clearly

---

## Troubleshooting

### **Common Issues**

| Issue | Cause | Solution |
|-------|-------|----------|
| Environment not discovered | File naming or location | Ensure file follows `environment_*.py` pattern |
| Agent initialization fails | Missing agent config | Verify all required agents exist |
| Workflow hangs | Missing await/timeout | Check async operations and timeouts |
| Resource leaks | Missing cleanup | Implement proper resource management |

### **Debugging Tips**

1. **Enable Logging**: Use detailed logging for workflow tracking
2. **Test Agents Individually**: Validate each agent works independently
3. **Check Configurations**: Verify YAML syntax and required fields
4. **Monitor Resources**: Watch for file/memory leaks during development

---

## Next Steps

- **[Custom Agents](Custom-Agents)** - Create specialized agents for your environment
- **[Custom MCTS Phases](Custom-MCTS-Phases)** - Use environments in search strategies
- **[Extension Combinations](Extension-Combinations)** - Combine with other extensions

---

## Related Pages

### **Extension Development**
- [Custom Agents](Custom-Agents) - Create specialized agents for your environments
- [Custom MCTS Phases](Custom-MCTS-Phases) - Use environments in search strategies
- [Extension Combinations](Extension-Combinations) - Combine environments with other extensions

### **Environment System**
- [Environment System](Environment-System) - Understanding the environment framework
- [Agent System](Agent-System) - Agent integration and orchestration
- [Configuration Overview](Configuration-Overview) - Environment configuration

### **Implementation**
- [Extending PrismBench](Extending-PrismBench) - Framework extension overview
- [Architecture Overview](Architecture-Overview) - System design and components
- [Troubleshooting](Troubleshooting) - Environment-related issues and solutions
