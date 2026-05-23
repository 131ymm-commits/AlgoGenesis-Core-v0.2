# AlgoGenesis Quick Start Guide

## 5-Minute Setup

### Step 1: Install Dependencies (1 minute)
```bash
# Make sure you have Python 3.8+
python --version

# Install requirements
pip install -r requirements.txt

# Verify installation
python -c "import torch; print(f'PyTorch {torch.__version__} ✓')"
python -c "import gymnasium; print('Gymnasium ✓')"
```

### Step 2: Run Evolution (2-3 minutes)
```bash
python algogenesis_core.py

# Expected output:
# ================================================================================
# 🚀 STARTING EVOLUTION
# ================================================================================
# AlgoGenesis initialized: 25 agents, 100 generations
# Environment: CartPole-v1, Revolution prob: 0.09
# Gen  10 | Best:  125.3 | Mean:   95.4 | Div: 0.512 | γ_mean: 0.382 | Time:  8.5s
# Gen  20 | Best:  245.6 | Mean:  187.2 | Div: 0.387 | γ_mean: 0.451 | Time: 17.2s
# ...
# ✅ EVOLUTION COMPLETED in 142.3 seconds
# 🏆 Best fitness achieved: 387.2
```

### Step 3: View Results (1-2 minutes)
```bash
# Check generated files
ls -la results/
# Output:
# -rw-r--r-- evolution_analysis.png    # 4-panel visualization
# -rw-r--r-- statistics.json           # Raw data in JSON

# Open visualization
open results/evolution_analysis.png          # macOS
xdg-open results/evolution_analysis.png      # Linux
start results/evolution_analysis.png         # Windows
```

---

## What the Plots Show

### Panel 1: Population Fitness
```
Fitness
  500 |                                    ╱─╲
  400 |                              ╱────╱   ╲
  300 |                        ╱────╱         
  200 |                  ╱────╱
  100 |            ╱────╱
    0 └──────────────────────────────────────
      0         25        50        75       100
                        Generations

Green line (Best): Champion fitness each generation
Blue line (Mean):  Population average
Shaded area:      Range between worst and best
```

**What to look for**:
- ✅ Best fitness increasing over time
- ✅ Mean fitness improving but slightly lagging
- ⚠️ Plateau = Evolution saturated (try more gens or increase mutation)

### Panel 2: Genetic Diversity
```
Diversity
  1.0 |●●●●●●●●●●●●●●●
      |
  0.5 |           ╲╲╲╲╲╲╲╲╲╲╲
      |                    ╲╲╲╲
  0.0 └────────────────────────────
      0         25        50        75       100
                        Generations
```

**Interpretation**:
- High diversity early = good exploration
- Declining diversity = population converging
- Stable low diversity at end = optimal strategy found

### Panel 3: UHF Coherence (γ)
```
Coherence
  1.0 |
      |              ╱─────────────
  0.5 |        ╱────╱
      |   ╱────╱
  0.0 └─╱──────────────────────────
      0         25        50        75       100
```

**What it means**:
- Low (0.2): Premature convergence, too diverse
- Mid (0.4-0.6): Optimal balance
- High (0.8+): Perfect harmony (rare)

### Panel 4: Revolutions
```
Count
    3 |   ╱╲
    2 |  ╱  ╲ ╱╲  ╱╲
    1 | ╱    ╲╱  ╲╱  ╲
    0 └───────────────────
      0         25        50        75       100
```

**What it means**:
- Bars show how many agents got reset each generation
- More common early (exploration)
- Less common late (exploiting known good strategies)

---

## Understanding statistics.json

```json
{
  "config": {
    "pop_size": 25,
    "generations": 100,
    "environment": "CartPole-v1",
    "revolution_prob": 0.09
  },
  "results": {
    "best_fitness_ever": 387.4,
    "best_agent_dna": [0.52, -0.31, 1.24, ...]
  },
  "history": {
    "best_fitness": [12.3, 34.5, 78.2, ...],
    "mean_fitness": [11.8, 28.3, 62.1, ...],
    "revolution_count": [0, 1, 0, 2, 0, 0, ...]
  },
  "revolution_events": [
    {
      "generation": 15,
      "agent_id": 7,
      "parent_fitness": 120.5
    },
    ...
  ]
}
```

### Extract Specific Data
```python
import json

with open('results/statistics.json', 'r') as f:
    data = json.load(f)

# Get best achieved fitness
best = data['results']['best_fitness_ever']
print(f"Best fitness: {best}")

# Count total revolutions
total_revolutions = sum(data['history']['revolution_count'])
print(f"Total revolutions: {total_revolutions}")

# Find most productive generation
gen = max(range(len(data['history']['best_fitness'])),
          key=lambda i: data['history']['best_fitness'][i])
print(f"Best improvement at generation {gen}")
```

---

## Customization Guide

### 1. Change Population Size

```python
# In algogenesis_core.py, modify:
evolution = AlgoGenesis(
    pop_size=50,        # Increase from 25
    generations=100,
)

# Trade-offs:
# Larger population = slower but better coverage
# Smaller population = faster but riskier
```

### 2. Change Number of Generations

```python
evolution = AlgoGenesis(
    pop_size=25,
    generations=200,    # Double from 100
)

# Trade-offs:
# More generations = more convergence time
# Fewer generations = risk of premature plateau
```

### 3. Adjust Revolution Probability

```python
evolution = AlgoGenesis(
    revolution_prob=0.15,  # Increase from 0.09 (15% instead of 9%)
)

# Effects:
# Higher prob = more exploration, higher final fitness
# Lower prob = faster convergence, risk of local optima
# Typical range: 0.05 to 0.20
```

### 4. Change Elite Preservation

```python
evolution = AlgoGenesis(
    elite_fraction=0.40,   # Increase from 0.25 (preserve top 40%)
)

# Effects:
# Higher fraction = more exploitation, more stable
# Lower fraction = more exploration, more variance
# Typical range: 0.15 to 0.50
```

### 5. Run on Different Environment

```python
# Replace 'CartPole-v1' with:
# - 'Acrobot-v1'      # 2-link pendulum (harder)
# - 'MountainCar-v0'  # Hill climbing (different)
# - 'LunarLander-v2'  # Space lander (complex)

evolution = AlgoGenesis(
    env_name='LunarLander-v2',
)

# Note: May need to adjust network sizes for different state/action dimensions
```

---

## Troubleshooting

### Problem: "ModuleNotFoundError: No module named 'gymnasium'"

**Solution**:
```bash
pip install gymnasium
# Or reinstall all requirements
pip install -r requirements.txt --force-reinstall
```

### Problem: Evolution is very slow

**Solutions**:
```python
# Option 1: Reduce population size
evolution = AlgoGenesis(pop_size=15)

# Option 2: Reduce episodes per evaluation
agent.evaluate(env, episodes=3)  # from 5

# Option 3: Run on GPU (if available)
# Just make sure PyTorch sees GPU:
python -c "import torch; print(torch.cuda.is_available())"

# Option 4: Reduce generations if just testing
evolution = AlgoGenesis(generations=50)
```

### Problem: Results not reproducible

**Solution**:
```python
# Make sure seeds are set BEFORE creating evolution
torch.manual_seed(42)
np.random.seed(42)
random.seed(42)

evolution = AlgoGenesis(...)  # Now reproducible
```

### Problem: No improvement after generation 50

**Likely causes & solutions**:
```python
# Cause 1: Population converged to local optimum
# Solution: Increase revolution probability
evolution = AlgoGenesis(revolution_prob=0.15)

# Cause 2: Mutation too weak
# Solution: Increase mutation strength
# Edit in algogenesis_core.py, in EvolvingAgent.mutate():
child.mutate(mutation_rate=0.9, mutation_std=0.5)

# Cause 3: Elite preservation too high
# Solution: Allow more genetic diversity
evolution = AlgoGenesis(elite_fraction=0.15)

# Cause 4: Population size too small
# Solution: More agents = more diversity
evolution = AlgoGenesis(pop_size=50)
```

---

## Comparing Multiple Runs

### Run Experiment Multiple Times
```bash
for i in {1..3}; do
    echo "Run $i"
    python algogenesis_core.py > run_$i.log
    cp results/statistics.json results/run_${i}_stats.json
    cp results/evolution_analysis.png results/run_${i}_plot.png
done
```

### Compare Results
```python
import json
import numpy as np

results = []
for i in range(1, 4):
    with open(f'results/run_{i}_stats.json', 'r') as f:
        data = json.load(f)
    results.append(data['results']['best_fitness_ever'])

print(f"Run 1: {results[0]:.1f}")
print(f"Run 2: {results[1]:.1f}")
print(f"Run 3: {results[2]:.1f}")
print(f"Mean: {np.mean(results):.1f} ± {np.std(results):.1f}")
```

---

## Advanced: Custom Evaluation Metric

```python
class CustomAlgoGenesis(AlgoGenesis):
    def run(self):
        # Call parent's setup
        super().run()
        
        # Custom post-processing
        for agent in self.population:
            # Compute energy efficiency
            agent.energy_score = self.compute_energy_efficiency(agent)
            
            # Compute robustness to noise
            agent.robustness = self.evaluate_with_noise(agent)
        
        return self.best_agent_ever
    
    def compute_energy_efficiency(self, agent):
        # Custom metric: minimize actuator usage
        return -sum(abs(actions_taken))
    
    def evaluate_with_noise(self, agent):
        # Test with observation noise
        return agent.evaluate(self.env_noisy)
```

---

## Performance Tips

### CPU Optimization
```bash
# Use more cores
export OMP_NUM_THREADS=8  # Before running

# Or in Python:
import torch
torch.set_num_threads(8)
```

### Memory Optimization
```python
# Reduce evaluation episodes if memory-constrained
agent.evaluate(env, episodes=2)  # from 5

# Or limit population size
evolution = AlgoGenesis(pop_size=10)  # from 25
```

### Time Optimization
```python
# Profile to find bottlenecks
python -m cProfile -s cumtime algogenesis_core.py

# Look for:
# - evaluate() calls (most expensive)
# - network forward passes
# - environment step() calls
```

---

## Next Steps

1. ✅ **Run basic experiment** (this guide)
2. 📊 **Analyze results** in statistics.json
3. 🎛️ **Try different hyperparameters**
4. 🔄 **Run multiple experiments** for statistics
5. 📖 **Read ANALYSIS.md** for deeper understanding
6. 🗺️ **Check ROADMAP.md** for v0.3+ features

---

## Getting Help

1. **Check logs**: `cat logs/evolution_*.log`
2. **Read ANALYSIS.md**: Theory & detailed explanations
3. **Read docstrings**: `python -c "import algogenesis_core; help(algogenesis_core.AlgoGenesis)"`
4. **Review README.md**: Full documentation
5. **Check TROUBLESHOOTING.md**: Common issues

---

## Fun Experiments to Try

### 1. Evolution Speed Comparison
```python
# Experiment: Does revolution help?
configs = [
    {'revolution_prob': 0.00, 'name': 'No revolutions'},
    {'revolution_prob': 0.09, 'name': 'Moderate'},
    {'revolution_prob': 0.20, 'name': 'Aggressive'},
]

for config in configs:
    evo = AlgoGenesis(revolution_prob=config['revolution_prob'])
    best = evo.run()
    print(f"{config['name']}: {best.fitness:.1f}")
```

### 2. Mutation Rate Sweep
```python
# Find optimal mutation strength
for mutation_std in [0.1, 0.2, 0.35, 0.5, 0.75]:
    # Modify algogenesis_core.py to use this value
    # Run experiment
    # Record best fitness
```

### 3. Population Size vs Speed
```python
# Trade-off analysis
import time

for pop_size in [10, 25, 50, 100]:
    start = time.time()
    evo = AlgoGenesis(pop_size=pop_size, generations=50)
    evo.run()
    elapsed = time.time() - start
    print(f"Pop {pop_size}: {elapsed:.1f}s")
```

### 4. Multi-Task Learning
```python
# Can evolution solve multiple tasks?
# TODO: Implement in v0.3
```

---

**Happy evolving!** 🧬🚀

Questions? Check the full documentation in README.md and ANALYSIS.md
