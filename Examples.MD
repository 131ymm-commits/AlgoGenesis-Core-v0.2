# AlgoGenesis - Example Configurations

This file contains pre-configured setups for different use cases.

## 1. Quick Test (< 1 minute)

For testing that everything works:

```python
# algogenesis_core.py
if __name__ == "__main__":
    torch.manual_seed(42)
    np.random.seed(42)
    random.seed(42)
    
    evolution = AlgoGenesis(
        pop_size=10,          # Small population
        generations=20,       # Few generations
        env_name='CartPole-v1',
        revolution_prob=0.09,
    )
    
    best_agent = evolution.run()
    evolution.save_statistics("results")
    evolution.plot_evolution("results")
```

**Expected runtime**: 30-60 seconds

---

## 2. Standard Experiment (2-3 minutes)

For getting reasonable results with moderate computation:

```python
if __name__ == "__main__":
    torch.manual_seed(42)
    np.random.seed(42)
    random.seed(42)
    
    evolution = AlgoGenesis(
        pop_size=25,          # Balanced
        generations=100,      # Standard
        env_name='CartPole-v1',
        revolution_prob=0.09,
        elite_fraction=0.25,
    )
    
    best_agent = evolution.run()
    evolution.save_statistics("results")
    evolution.plot_evolution("results")
```

**Expected runtime**: 2-3 minutes  
**Expected best fitness**: 350-400

---

## 3. Production Run (5-10 minutes)

For high-quality results and thorough analysis:

```python
if __name__ == "__main__":
    torch.manual_seed(42)
    np.random.seed(42)
    random.seed(42)
    
    evolution = AlgoGenesis(
        pop_size=50,          # Large population
        generations=200,      # More generations
        env_name='CartPole-v1',
        revolution_prob=0.12,
        elite_fraction=0.20,
    )
    
    best_agent = evolution.run()
    evolution.save_statistics("results")
    evolution.plot_evolution("results")
    
    # Additional analysis
    print("\n=== EXTENDED ANALYSIS ===")
    print(f"Population diversity: {np.mean([np.mean(np.abs(a.dna)) for a in evolution.population]):.3f}")
    print(f"Total revolutions: {len(evolution.revolution_events)}")
    print(f"Average agent lifetime: {np.mean([a.lifetime for a in evolution.population]):.1f}")
```

**Expected runtime**: 5-10 minutes  
**Expected best fitness**: 400-500

---

## 4. Exploration vs Exploitation

Compare strategies with high vs low mutation:

```python
def run_with_config(name, mutation_std, revolution_prob):
    torch.manual_seed(42)
    np.random.seed(42)
    random.seed(42)
    
    print(f"\n{'='*60}")
    print(f"Running: {name}")
    print(f"  Mutation std: {mutation_std}")
    print(f"  Revolution prob: {revolution_prob}")
    print(f"{'='*60}")
    
    evolution = AlgoGenesis(
        pop_size=25,
        generations=100,
        revolution_prob=revolution_prob,
    )
    
    # Patch mutation function
    for agent in evolution.population:
        original_mutate = agent.mutate
        def mutate_patch():
            original_mutate(mutation_std=mutation_std)
        agent.mutate = mutate_patch
    
    best = evolution.run()
    evolution.save_statistics(f"results/{name}")
    
    return best.fitness

# Run configurations
configs = [
    ("conservative", 0.2, 0.05),   # Low mutation, few revolutions
    ("balanced", 0.35, 0.09),       # Standard settings
    ("aggressive", 0.6, 0.15),      # High mutation, many revolutions
]

results = {}
for name, mut_std, rev_prob in configs:
    results[name] = run_with_config(name, mut_std, rev_prob)

print("\n=== RESULTS COMPARISON ===")
for name, fitness in results.items():
    print(f"{name:15s}: {fitness:.1f}")
```

---

## 5. Multi-Environment Training (v0.3+)

```python
class MultiEnvAlgoGenesis(AlgoGenesis):
    def __init__(self, environments, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.environments = environments
    
    def evaluate_agent(self, agent):
        fitnesses = []
        for env_name in self.environments:
            env = gym.make(env_name)
            fitness = agent.evaluate(env, episodes=3)
            fitnesses.append(fitness)
            env.close()
        
        # Multi-task fitness: generalization score
        return np.mean(fitnesses)

if __name__ == "__main__":
    evolution = MultiEnvAlgoGenesis(
        environments=['CartPole-v1', 'Acrobot-v1'],
        pop_size=25,
        generations=100,
    )
    
    best = evolution.run()
    print(f"Generalization fitness: {best.fitness:.1f}")
```

---

## 6. Hyperparameter Sweep

Find optimal parameters:

```python
import itertools
import pandas as pd

# Define parameter ranges
param_grid = {
    'pop_size': [10, 25, 50],
    'elite_fraction': [0.15, 0.25, 0.40],
    'revolution_prob': [0.05, 0.09, 0.15],
}

results = []

for pop_size in param_grid['pop_size']:
    for elite_frac in param_grid['elite_fraction']:
        for rev_prob in param_grid['revolution_prob']:
            torch.manual_seed(42)
            np.random.seed(42)
            random.seed(42)
            
            print(f"Testing: pop={pop_size}, elite={elite_frac:.2f}, rev={rev_prob:.2f}")
            
            evo = AlgoGenesis(
                pop_size=pop_size,
                generations=50,  # Quick test
                elite_fraction=elite_frac,
                revolution_prob=rev_prob,
            )
            
            best = evo.run()
            
            results.append({
                'pop_size': pop_size,
                'elite_fraction': elite_frac,
                'revolution_prob': rev_prob,
                'best_fitness': best.fitness,
            })

# Analyze results
df = pd.DataFrame(results)
print("\n=== TOP 5 CONFIGURATIONS ===")
print(df.nlargest(5, 'best_fitness'))

# Find best overall
best_config = df.loc[df['best_fitness'].idxmax()]
print("\n=== BEST CONFIGURATION ===")
print(best_config)
```

---

## 7. Reproducibility Test

Verify that runs are reproducible:

```python
def run_and_get_best(seed):
    torch.manual_seed(seed)
    np.random.seed(seed)
    random.seed(seed)
    
    evolution = AlgoGenesis(
        pop_size=15,
        generations=50,
    )
    
    best = evolution.run()
    return best.fitness

# Run 3 times with same seed
fitnesses = [run_and_get_best(42) for _ in range(3)]

print("Reproducibility test:")
for i, f in enumerate(fitnesses):
    print(f"  Run {i+1}: {f:.1f}")

# Should all be identical
if len(set(fitnesses)) == 1:
    print("✅ REPRODUCIBLE")
else:
    print("❌ NOT REPRODUCIBLE - check for non-deterministic operations")
```

---

## 8. Visualization-Only Mode

Analyze previous results without re-running:

```python
import json

# Load previous results
with open('results/statistics.json', 'r') as f:
    stats = json.load(f)

# Create visualization from loaded data
class VisualizationOnly(AlgoGenesis):
    def __init__(self, stats_file):
        self.history = {
            'best_fitness': stats['history']['best_fitness'],
            'mean_fitness': stats['history']['mean_fitness'],
            'worst_fitness': stats['history']['worst_fitness'],
            'population_diversity': stats['history']['population_diversity'],
            'gamma_mean': stats['history']['gamma_mean'],
            'revolution_count': stats['history']['revolution_count'],
        }
    
    def plot_evolution(self, output_dir='results'):
        # Use parent's plotting method
        super().plot_evolution(output_dir)

viz = VisualizationOnly('results/statistics.json')
viz.plot_evolution('results_analysis')
```

---

## 9. Ablation Study: Revolution Impact

Quantify the benefit of revolution mechanism:

```python
def evaluate_strategy(name, use_revolutions=True):
    """Run evolution with or without revolutions"""
    
    class CustomAlgoGenesis(AlgoGenesis):
        def run(self):
            logger.info(f"Running strategy: {name}")
            
            for generation in range(self.generations):
                # ... (standard evolution code)
                
                # Reproduction phase
                new_population = []
                elite_count = max(3, int(self.pop_size * self.elite_fraction))
                
                for i in range(self.pop_size):
                    parent = random.choice(self.population[:elite_count])
                    child = parent.clone()
                    child.mutate()
                    
                    # Revolution mechanism
                    if use_revolutions and random.random() < self.revolution_prob:
                        child.dna = np.random.uniform(-2.0, 2.0, 32)
                    
                    new_population.append(child)
                
                self.population = new_population
            
            return self.population[0]
    
    torch.manual_seed(42)
    np.random.seed(42)
    random.seed(42)
    
    evo = CustomAlgoGenesis(
        pop_size=25,
        generations=100,
    )
    
    best = evo.run()
    evo.save_statistics(f"results/ablation_{name}")
    
    return best.fitness

# Compare
baseline = evaluate_strategy("with_revolutions", use_revolutions=True)
ablation = evaluate_strategy("no_revolutions", use_revolutions=False)

print(f"\nAblation Study Results:")
print(f"  With revolutions:    {baseline:.1f}")
print(f"  Without revolutions: {ablation:.1f}")
print(f"  Improvement:         {baseline - ablation:+.1f} ({100*(baseline-ablation)/ablation:.1f}%)")
```

---

## 10. Continuous Monitoring (Development)

Run evolution and save results periodically:

```python
import time
import os

def run_with_checkpoints(checkpoint_dir="checkpoints"):
    os.makedirs(checkpoint_dir, exist_ok=True)
    
    torch.manual_seed(42)
    np.random.seed(42)
    random.seed(42)
    
    evolution = AlgoGenesis(
        pop_size=25,
        generations=100,
    )
    
    for generation in range(evolution.generations):
        # ... run one generation ...
        
        # Save checkpoint every 10 generations
        if generation % 10 == 0:
            checkpoint = {
                'generation': generation,
                'population': [a.dna for a in evolution.population],
                'best_fitness': evolution.history['best_fitness'][-1],
            }
            
            with open(f"{checkpoint_dir}/gen_{generation:03d}.pkl", 'wb') as f:
                pickle.dump(checkpoint, f)
            
            print(f"Checkpoint saved: gen_{generation:03d}.pkl")
    
    return evolution.best_agent_ever
```

---

## Quick Reference Card

```python
# MINIMAL (10 sec test)
AlgoGenesis(pop_size=5, generations=5)

# FAST (30 sec)
AlgoGenesis(pop_size=10, generations=20)

# STANDARD (2-3 min)
AlgoGenesis(pop_size=25, generations=100)

# THOROUGH (5-10 min)
AlgoGenesis(pop_size=50, generations=200)

# AGGRESSIVE EXPLORATION
AlgoGenesis(revolution_prob=0.20, elite_fraction=0.15)

# CONSERVATIVE EXPLOITATION
AlgoGenesis(revolution_prob=0.05, elite_fraction=0.40)

# MULTI-TASK (v0.3+)
AlgoGenesis(env_name='Acrobot-v1')  # or other envs
```

---

Use these configurations as starting points for your experiments! 🚀
