# Custom MCTS Phases

> Implementing custom MCTS phases with specialized search strategies and evaluation objectives

Custom MCTS phases allow you to implement sophisticated search strategies tailored to your evaluation objectives. Phases define how the Monte Carlo Tree Search explores and evaluates the solution space.

## Overview

MCTS phases in PrismBench implement **strategy patterns** for different search objectives:

- **Node Selection**: How to choose nodes for exploration
- **Node Evaluation**: How to assess node performance
- **Value Calculation**: How to convert results to numeric scores
- **Backpropagation**: How to update ancestor node values
- **Node Expansion**: How to generate child nodes

---

## Phase Architecture

### **Strategy Pattern Implementation**

Phases use a **decorator-based registry** for automatic discovery:

```python
from src.mcts.phase_registry import phase_registry

@phase_registry.register_phase_method("my_custom_phase", "select_node")
async def select_node(self: "BasePhase") -> ChallengeNode:
    """Custom node selection strategy."""
    # Your selection logic here
    pass
```

### **Required Strategy Methods**

Each phase **must** implement five core methods:

| Method | Purpose | Async | Description |
|--------|---------|-------|-------------|
| `select_node` | Node Selection | Yes | Choose which node to evaluate next |
| `evaluate_node` | Node Evaluation | Yes | Execute evaluation for selected node |
| `calculate_node_value` | Scoring | No | Convert evaluation results to numeric score |
| `backpropagate_node_value` | Value Propagation | No | Update ancestor nodes with new values |
| `expand_node` | Tree Expansion | Yes | Generate child nodes for expansion |

### **Optional Strategy Methods**

| Method | Purpose | Async | Description |
|--------|---------|-------|-------------|
| `initialize_phase` | Phase Setup | Yes | Custom initialization logic |

---

## Creating Custom Phases

### **Step 1: Phase Implementation**

Create a new file `src/services/search/src/mcts/phase_custom.py`:

```python
from typing import TYPE_CHECKING, Dict, List
import random
import asyncio
from src.mcts.phase_registry import phase_registry

if TYPE_CHECKING:
    from src.mcts.base_phase import BasePhase
    from src.tree.node import ChallengeNode

@phase_registry.register_phase_method("phase_custom", "select_node")
async def select_node(self: "BasePhase") -> "ChallengeNode":
    """
    Custom node selection strategy.
    
    Example: Select nodes based on custom criteria.
    """
    available_nodes = [
        node for node in self.tree.nodes 
        if node.depth < self.config["phase_params"]["max_depth"]
        and not node.fully_explored
    ]
    
    if not available_nodes:
        return None
    
    # Custom selection logic
    if random.random() < self.config["phase_params"]["exploration_probability"]:
        # Random exploration
        return random.choice(available_nodes)
    else:
        # Exploit best nodes based on custom criteria
        scored_nodes = []
        for node in available_nodes:
            custom_score = self._calculate_custom_score(node)
            scored_nodes.append((custom_score, node))
        
        # Select highest scoring node
        scored_nodes.sort(reverse=True)
        return scored_nodes[0][1]

def _calculate_custom_score(self, node: "ChallengeNode") -> float:
    """Calculate custom selection score for node."""
    # Example: Combine UCB1 with domain-specific metrics
    ucb1_score = node.ucb1(exploration_weight=1.414)
    domain_bonus = self._get_domain_importance(node.concepts)
    return ucb1_score + domain_bonus

def _get_domain_importance(self, concepts: List[str]) -> float:
    """Get domain-specific importance score."""
    # Example: Prioritize certain concepts
    important_concepts = {"algorithms", "data_structures", "optimization"}
    overlap = len(set(concepts) & important_concepts)
    return overlap * 0.5
```

### **Step 2: Evaluation Strategy**

```python
@phase_registry.register_phase_method("phase_custom", "evaluate_node")
async def evaluate_node(
    self: "BasePhase", 
    node: "ChallengeNode"
) -> Dict:
    """
    Custom node evaluation strategy.
    
    Example: Multi-round evaluation with different criteria.
    """
    evaluation_results = {
        "node_id": node.node_id,
        "concept": node.concepts,
        "difficulty": node.difficulty,
        "rounds": []
    }
    
    # Multi-round evaluation
    num_rounds = self.config["phase_params"].get("evaluation_rounds", 3)
    
    for round_num in range(num_rounds):
        # Evaluate with different parameters each round
        round_config = {
            "concept": node.concepts,
            "difficulty_level": node.difficulty,
            "evaluation_round": round_num,
            "focus_area": self._get_round_focus(round_num)
        }
        
        try:
            # Execute evaluation using environment
            round_result = await self.environment.run_challenge(**round_config)
            evaluation_results["rounds"].append(round_result)
            
        except Exception as e:
            logger.error(f"Evaluation failed for round {round_num}: {e}")
            evaluation_results["rounds"].append({
                "success": False,
                "error": str(e)
            })
    
    # Aggregate results
    evaluation_results["aggregated_success"] = self._aggregate_round_results(
        evaluation_results["rounds"]
    )
    
    return evaluation_results

def _get_round_focus(self, round_num: int) -> str:
    """Get focus area for evaluation round."""
    focus_areas = ["correctness", "efficiency", "style"]
    return focus_areas[round_num % len(focus_areas)]

def _aggregate_round_results(self, rounds: List[Dict]) -> float:
    """Aggregate results from multiple evaluation rounds."""
    if not rounds:
        return 0.0
    
    successful_rounds = [r for r in rounds if r.get("success", False)]
    return len(successful_rounds) / len(rounds)
```

### **Step 3: Value Calculation**

```python
@phase_registry.register_phase_method("phase_custom", "calculate_node_value")
def calculate_node_value(
    self: "BasePhase",
    results: Dict,
    **kwargs
) -> float:
    """
    Custom value calculation strategy.
    
    Example: Multi-criteria scoring with weighted factors.
    """
    base_success_rate = results.get("aggregated_success", 0.0)
    
    # Multi-criteria scoring
    criteria_scores = {
        "success_rate": base_success_rate,
        "efficiency": self._calculate_efficiency_score(results),
        "innovation": self._calculate_innovation_score(results),
        "difficulty_adjusted": self._adjust_for_difficulty(
            base_success_rate, 
            results.get("difficulty", "medium")
        )
    }
    
    # Weighted combination
    weights = self.config["scoring_params"].get("criteria_weights", {
        "success_rate": 0.4,
        "efficiency": 0.2,
        "innovation": 0.2,
        "difficulty_adjusted": 0.2
    })
    
    total_score = sum(
        criteria_scores[criterion] * weights.get(criterion, 0.0)
        for criterion in criteria_scores
    )
    
    # Apply custom bonuses/penalties
    total_score += self._apply_custom_modifiers(results)
    
    return max(0.0, min(1.0, total_score))  # Clamp to [0, 1]

def _calculate_efficiency_score(self, results: Dict) -> float:
    """Calculate efficiency score from results."""
    rounds = results.get("rounds", [])
    if not rounds:
        return 0.0
    
    # Example: Measure based on attempts per round
    total_attempts = sum(
        len(r.get("data_trail", [])) for r in rounds
    )
    avg_attempts = total_attempts / len(rounds)
    
    # Lower attempts = higher efficiency
    return max(0.0, 1.0 - (avg_attempts - 1) / 5.0)

def _calculate_innovation_score(self, results: Dict) -> float:
    """Calculate innovation score from results."""
    # Example: Analyze solution patterns for creativity
    rounds = results.get("rounds", [])
    unique_approaches = set()
    
    for round_data in rounds:
        for trail in round_data.get("data_trail", []):
            solution = trail.get("solution_code", "")
            approach_signature = self._extract_approach_signature(solution)
            unique_approaches.add(approach_signature)
    
    # More unique approaches = higher innovation
    return min(1.0, len(unique_approaches) / 3.0)

def _adjust_for_difficulty(self, score: float, difficulty: str) -> float:
    """Adjust score based on difficulty level."""
    difficulty_multipliers = {
        "very easy": 0.5,
        "easy": 0.7,
        "medium": 1.0,
        "hard": 1.3,
        "very hard": 1.5
    }
    
    multiplier = difficulty_multipliers.get(difficulty.lower(), 1.0)
    return score * multiplier
```

### **Step 4: Backpropagation Strategy**

```python
@phase_registry.register_phase_method("phase_custom", "backpropagate_node_value")
def backpropagate_node_value(
    self: "BasePhase",
    node: "ChallengeNode", 
    reward: float
) -> None:
    """
    Custom backpropagation strategy.
    
    Example: Weighted backpropagation with decay.
    """
    current_node = node
    depth = 0
    
    while current_node is not None:
        # Apply depth-based decay
        decay_factor = self.config["search_params"].get("decay_factor", 0.9) ** depth
        adjusted_reward = reward * decay_factor
        
        # Custom value update with momentum
        learning_rate = self.config["search_params"]["learning_rate"]
        momentum = self.config["search_params"].get("momentum", 0.1)
        
        # Store previous value for momentum calculation
        old_value = current_node.value
        
        # Update with momentum
        current_node.update_node_score(learning_rate, adjusted_reward)
        
        # Apply momentum if configured
        if momentum > 0 and hasattr(current_node, '_previous_value'):
            momentum_adjustment = momentum * (old_value - current_node._previous_value)
            current_node.value += momentum_adjustment
        
        current_node._previous_value = old_value
        
        # Move to parent
        current_node = current_node.parents[0] if current_node.parents else None
        depth += 1
```

### **Step 5: Expansion Strategy**

```python
@phase_registry.register_phase_method("phase_custom", "expand_node")
async def expand_node(
    self: "BasePhase",
    node: "ChallengeNode"
) -> None:
    """
    Custom node expansion strategy.
    
    Example: Smart expansion based on performance patterns.
    """
    if node.fully_explored:
        return
    
    expansion_strategies = self._determine_expansion_strategies(node)
    
    for strategy in expansion_strategies:
        if strategy == "concept_combination":
            await self._expand_by_concept_combination(node)
        elif strategy == "difficulty_progression":
            await self._expand_by_difficulty_progression(node)
        elif strategy == "constraint_variation":
            await self._expand_by_constraint_variation(node)

def _determine_expansion_strategies(self, node: "ChallengeNode") -> List[str]:
    """Determine which expansion strategies to use."""
    strategies = []
    
    # Choose strategies based on node performance
    if node.value > 0.7:  # High performing node
        strategies.append("difficulty_progression")
    elif node.value > 0.4:  # Medium performing node
        strategies.extend(["concept_combination", "constraint_variation"])
    else:  # Low performing node
        strategies.append("concept_combination")  # Try different combinations
    
    return strategies

async def _expand_by_concept_combination(self, node: "ChallengeNode") -> None:
    """Expand by combining concepts."""
    available_concepts = [
        c for c in self.tree.concepts 
        if c not in node.concepts
    ]
    
    # Limit combinations to avoid explosion
    max_combinations = self.config["phase_params"].get("max_concept_combinations", 2)
    
    for concept in available_concepts[:max_combinations]:
        new_concepts = node.concepts + [concept]
        difficulty = self._select_appropriate_difficulty(new_concepts)
        
        child_node = self.tree.add_node(
            parent_nodes=node,
            concepts=new_concepts,
            difficulty=difficulty,
            phase=self.phase_name
        )

async def _expand_by_difficulty_progression(self, node: "ChallengeNode") -> None:
    """Expand by increasing difficulty."""
    current_difficulty_idx = self.tree.difficulties.index(node.difficulty)
    
    if current_difficulty_idx < len(self.tree.difficulties) - 1:
        next_difficulty = self.tree.difficulties[current_difficulty_idx + 1]
        
        child_node = self.tree.add_node(
            parent_nodes=node,
            concepts=node.concepts,
            difficulty=next_difficulty,
            phase=self.phase_name
        )
```

---

## Configuration Setup

### **Phase Configuration**

Add your phase to `configs/phase_configs.yaml`:

```yaml
phase_custom:
  phase_params:
    max_depth: 6
    max_iterations: 150
    performance_threshold: 0.5
    value_delta_threshold: 0.2
    convergence_checks: 8
    exploration_probability: 0.3
    evaluation_rounds: 3
    max_concept_combinations: 3
    
  search_params:
    max_attempts: 4
    discount_factor: 0.85
    learning_rate: 0.8
    momentum: 0.1
    decay_factor: 0.9
    
  scoring_params:
    criteria_weights:
      success_rate: 0.4
      efficiency: 0.2
      innovation: 0.2
      difficulty_adjusted: 0.2
    innovation_bonus: 0.1
    efficiency_bonus: 0.05
    
  environment:
    name: "environment_enhanced_coding_challenge"
```

### **Experiment Configuration**

Include your phase in experiment sequences:

```yaml
# configs/experiment_configs.yaml
experiments:
  custom_evaluation:
    name: "Custom Evaluation Experiment"
    description: "Uses custom phase for specialized evaluation"
    phase_sequences:
      - phase_1
      - phase_custom
      - phase_3
      
  pure_custom:
    name: "Pure Custom Phase Experiment"
    description: "Uses only custom phase"
    phase_sequences:
      - phase_custom
```

---

## Advanced Phase Patterns

### **Multi-Objective Optimization Phase**

```python
@phase_registry.register_phase_method("phase_multi_objective", "calculate_node_value")
def calculate_node_value(
    self: "BasePhase",
    results: Dict,
    **kwargs
) -> float:
    """Multi-objective scoring with Pareto optimization."""
    
    objectives = {
        "accuracy": self._calculate_accuracy(results),
        "efficiency": self._calculate_efficiency(results),
        "robustness": self._calculate_robustness(results),
        "innovation": self._calculate_innovation(results)
    }
    
    # Pareto ranking
    pareto_rank = self._calculate_pareto_rank(objectives)
    
    # Convert rank to score (lower rank = higher score)
    max_rank = len(self.tree.nodes)
    normalized_score = 1.0 - (pareto_rank / max_rank)
    
    return normalized_score

def _calculate_pareto_rank(self, objectives: Dict[str, float]) -> int:
    """Calculate Pareto rank for multi-objective optimization."""
    # Compare with all other nodes to determine domination
    dominated_count = 0
    
    for other_node in self.tree.nodes:
        if hasattr(other_node, 'objectives'):
            if self._dominates(objectives, other_node.objectives):
                dominated_count += 1
    
    return dominated_count

def _dominates(self, obj1: Dict, obj2: Dict) -> bool:
    """Check if obj1 dominates obj2 in Pareto sense."""
    better_in_all = all(obj1[k] >= obj2[k] for k in obj1.keys())
    strictly_better_in_one = any(obj1[k] > obj2[k] for k in obj1.keys())
    return better_in_all and strictly_better_in_one
```

### **Adaptive Exploration Phase**

```python
@phase_registry.register_phase_method("phase_adaptive", "select_node")
async def select_node(self: "BasePhase") -> "ChallengeNode":
    """Adaptive node selection based on search history."""
    
    # Analyze recent performance trends
    performance_trend = self._analyze_performance_trend()
    
    # Adjust exploration strategy based on trend
    if performance_trend == "improving":
        # Focus on exploitation
        exploration_probability = 0.1
    elif performance_trend == "stagnating":
        # Increase exploration
        exploration_probability = 0.4
    else:  # declining
        # High exploration to find new areas
        exploration_probability = 0.6
    
    available_nodes = self._get_available_nodes()
    
    if random.random() < exploration_probability:
        # Explore less-visited areas
        return self._select_underexplored_node(available_nodes)
    else:
        # Exploit promising areas
        return self._select_promising_node(available_nodes)

def _analyze_performance_trend(self) -> str:
    """Analyze recent performance trend."""
    recent_iterations = 10
    if len(self.performance_history) < recent_iterations:
        return "insufficient_data"
    
    recent_scores = self.performance_history[-recent_iterations:]
    trend_slope = self._calculate_trend_slope(recent_scores)
    
    if trend_slope > 0.01:
        return "improving"
    elif trend_slope < -0.01:
        return "declining"
    else:
        return "stagnating"
```

### **Hierarchical Search Phase**

```python
@phase_registry.register_phase_method("phase_hierarchical", "initialize_phase")
async def initialize_phase(self: "BasePhase") -> None:
    """Initialize hierarchical search levels."""
    self.search_levels = {
        "global": {"nodes": [], "focus": "broad_exploration"},
        "local": {"nodes": [], "focus": "deep_exploitation"},
        "micro": {"nodes": [], "focus": "fine_tuning"}
    }
    
    # Categorize existing nodes by search level
    for node in self.tree.nodes:
        level = self._determine_search_level(node)
        self.search_levels[level]["nodes"].append(node)

@phase_registry.register_phase_method("phase_hierarchical", "select_node")
async def select_node(self: "BasePhase") -> "ChallengeNode":
    """Hierarchical node selection."""
    
    # Determine current search level based on phase progress
    current_level = self._determine_current_level()
    
    # Select node from appropriate level
    level_nodes = self.search_levels[current_level]["nodes"]
    
    if not level_nodes:
        # Fall back to other levels
        for level in ["global", "local", "micro"]:
            if self.search_levels[level]["nodes"]:
                level_nodes = self.search_levels[level]["nodes"]
                break
    
    return self._select_from_level(level_nodes, current_level)

def _determine_current_level(self) -> str:
    """Determine current hierarchical search level."""
    progress = self.current_iteration / self.config["phase_params"]["max_iterations"]
    
    if progress < 0.3:
        return "global"
    elif progress < 0.7:
        return "local"
    else:
        return "micro"
```

---

## Testing Custom Phases

### **Phase Testing**

```python
import asyncio
from src.services.search.src.mcts.utils import create_phase
from src.services.search.src.mcts.phase_registry import phase_registry
from src.services.search.src.tree.tree import Tree
from src.services.search.src.environment_client import EnvironmentClient

async def test_custom_phase():
    """Test custom phase implementation."""
    
    # Load phase modules
    phase_registry.load_phase_modules()
    
    # Create test tree
    tree = Tree(
        concepts=["arrays", "sorting", "searching"],
        difficulties=["easy", "medium", "hard"]
    )
    tree.initialize_tree()
    
    # Create environment client
    environment = EnvironmentClient(
        config={"base_url": "http://localhost:8001"},
        timeout=300
    )
    
    # Create custom phase
    phase_config = {
        "phase_params": {
            "max_iterations": 10,
            "max_depth": 3,
            "exploration_probability": 0.3
        },
        "search_params": {
            "learning_rate": 0.8,
            "discount_factor": 0.9
        },
        "scoring_params": {
            "criteria_weights": {
                "success_rate": 0.4,
                "efficiency": 0.3,
                "innovation": 0.3
            }
        }
    }
    
    phase = create_phase(
        phase_name="phase_custom",
        tree=tree,
        environment=environment,
        config=phase_config
    )
    
    # Test phase execution
    await phase.run()
    
    # Validate results
    assert len(tree.nodes) > 3  # Initial + expanded nodes
    print("Custom phase test passed!")

# Run test
asyncio.run(test_custom_phase())
```

### **Strategy Method Testing**

```python
async def test_individual_strategies():
    """Test individual strategy methods."""
    
    # Test node selection
    selected_node = await phase.select_node()
    assert selected_node is not None
    print(f"Selected node: {selected_node.node_id}")
    
    # Test node evaluation
    evaluation_results = await phase.evaluate_node(selected_node)
    assert "aggregated_success" in evaluation_results
    print(f"Evaluation results: {evaluation_results}")
    
    # Test value calculation
    node_value = phase.calculate_node_value(evaluation_results)
    assert 0.0 <= node_value <= 1.0
    print(f"Calculated value: {node_value}")
    
    # Test backpropagation
    phase.backpropagate_node_value(selected_node, node_value)
    print("Backpropagation completed")
    
    # Test expansion
    initial_children = len(selected_node.children)
    await phase.expand_node(selected_node)
    assert len(selected_node.children) >= initial_children
    print(f"Node expanded: {len(selected_node.children)} children")
```

---

## Best Practices

### **Strategy Design**

- **Single Objective**: Each phase should optimize for a clear, specific objective
- **Balanced Exploration**: Balance exploration and exploitation appropriately
- **Convergence Criteria**: Design clear convergence conditions

### **Performance Optimization**

- **Async Operations**: Use async/await for all IO operations
- **Early Termination**: Implement early stopping for inefficient branches

### **Configuration Design**

- **Parameter Validation**: Validate all configuration parameters
- **Sensible Defaults**: Provide reasonable default values
- **Documentation**: Document all parameters and their effects
- **Backward Compatibility**: Maintain compatibility with existing configs

---

## Troubleshooting

### **Common Issues**

| Issue | Cause | Solution |
|-------|-------|----------|
| Phase not discovered | File naming or location | Ensure file follows `phase_*.py` pattern |
| Method not registered | Missing decorator | Check decorator syntax and imports |
| Infinite loops | Poor convergence logic | Implement proper termination conditions |
| Memory leaks | Uncleaned references | Properly manage node references |

### **Debugging Tips**

1. **Enable Verbose Logging**: Use detailed logging for strategy decisions
2. **Test Strategies Separately**: Validate each strategy method independently
3. **Monitor Performance**: Track execution time and memory usage
4. **Visualize Tree Growth**: Use tree visualization to understand expansion

---

## Next Steps

- **[Custom Agents](Custom-Agents)** - Create specialized agents for your phase
- **[Custom Environments](Custom-Environments)** - Build environments for phase evaluation
- **[Extension Combinations](Extension-Combinations)** - Combine with other extensions

---

## Related Pages

### **Extension Development**
- [Custom Agents](Custom-Agents) - Create specialized agents for your phases
- [Custom Environments](Custom-Environments) - Build environments for phase evaluation
- [Extension Combinations](Extension-Combinations) - Combine phases with other extensions

### **MCTS System**
- [MCTS Algorithm](MCTS-Algorithm) - Understanding the core algorithm
- [Tree Structure](tree-structure) - Search tree implementation
- [Results Analysis](Results-Analysis) - Analyzing phase results

### **Implementation**
- [Extending PrismBench](Extending-PrismBench) - Framework extension overview
- [Configuration Overview](Configuration-Overview) - Phase configuration parameters
- [Troubleshooting](troubleshooting) - Phase-related issues and solutions
