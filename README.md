# 🔬 PRISM - Parameter Research & Investigation Sweep Manager

PRISM is a simple tool to run **parameter sweeps** for ML experiments. Give it a base config and a sweep definition, and it will generate, validate, and execute all experiment variations.

## What is PRISM?

PRISM takes your base experiment configuration and creates multiple variations by changing specific parameters. It then:
1. Validates each configuration using your custom Python validator
2. Runs your training script with each validated config
3. Tracks progress and captures metrics
4. Provides an interactive TUI to monitor experiments

## Installation

```bash
cd /path/to/prism-sweep
pip install -e .
```

You can use `prism_tui` or `prism` commands anywhere.

---

## Quick Start Guide

### Using the Interactive TUI (Recommended)

The easiest way to use PRISM is through the interactive terminal interface:

```bash
prism_tui
```

**First Time Setup:**
1. Navigate to your project directory in the TUI
2. Follow the guided setup to create `prism.project.yaml`
   - Give it a name
   - Select sweep definition(s) 
   - Give it a name
 2. **Run experiments**: Study Menu → Execute Study → Run All
   - Select base config YAML
   - Select sweep definition(s) 
   - Give it a name
 3. **Monitor progress**: Watch real-time logs and metrics
 4. **Retry failures**: Study Menu → Retry Failed
2. **Run experiments**: Study Menu → Execute Study → Run All
3. **Monitor progress**: Watch real-time logs and metrics
4. **Retry failures**: Study Menu → Retry Failed

### Using the Command Line

For automation and scripting, use the CLI:

```bash
# Create a study from base config and sweep definition
prism create --base configs/base.yaml \
             --prism configs/prism/lr_sweep.prism.yaml \
             --name lr_experiment

# Run all experiments in the study
prism run lr_experiment --all

# Run specific experiments
prism run lr_experiment lr_low lr_mid

# Check study status
prism status lr_experiment

# List all studies
prism list

# Retry failed experiments
prism retry lr_experiment
```

---

## What Files Do I Need?

PRISM needs 3-4 files in your project:

### 1. **prism.project.yaml** (Project Configuration)

This tells PRISM where everything is. The TUI can create this for you, or create it manually:

```yaml
project:
  name: my-ml-project
  version: "1.0"

paths:
  train_script: scripts/train.py        # Your training script
  configs_dir: configs                  # Where configs live
  prism_configs_dir: configs/prism      # Where sweep definitions live
  output_dir: outputs                   # Where results go

validator:
  module: configs/validator.py          # Optional: custom validator

metrics:
  output_mode: stdout_json              # How to capture metrics
```

### 2. **Base Config** (e.g., `configs/base.yaml`)

Your experiment's default configuration:

```yaml
learning_rate: 0.001
batch_size: 32
epochs: 100
model: resnet18
data:
  path: /data/train
  augmentation: true
```

### 3. **Sweep Definition** (e.g., `configs/prism/lr_sweep.yaml`)

Defines which parameters to vary:

```yaml
# Nominal parameters (creates named experiments)
learning_rate:
  lr_low: 0.0001
  lr_mid: 0.001
  lr_high: 0.01
```

### 4. **Custom Validator** (Optional: `configs/validator.py`)

Validates configs before training:

```python
from typing import Dict, Any
from dataclasses import dataclass

@dataclass
class ExperimentConfig:
    learning_rate: float
    batch_size: int
    epochs: int
    
    def validate(self):
        if self.learning_rate <= 0:
            raise ValueError("Learning rate must be > 0")
        if self.batch_size < 1:
            raise ValueError("Batch size must be >= 1")

def validate(config_dict: Dict[str, Any]) -> ExperimentConfig:
    """Called by PRISM to validate each config."""
    config = ExperimentConfig(
        learning_rate=config_dict.get('learning_rate', 0.001),
        batch_size=config_dict.get('batch_size', 32),
        epochs=config_dict.get('epochs', 100)
    )
    config.validate()
    return config
```
- Cross-field validation
- Set default values
- Convert enums, paths, etc.

### Your Training Script

Make sure it accepts a `--config` argument:

```python
# scripts/train.py
import argparse
import yaml
import json

parser = argparse.ArgumentParser()
parser.add_argument('--config', required=True)
args = parser.parse_args()

with open(args.config) as f:
    config = yaml.safe_load(f)

# Train your model...
learning_rate = config['learning_rate']
# ...

# Print metrics for PRISM
print(json.dumps({"loss": 0.123, "accuracy": 0.95}))
```

## File Structure

After running PRISM, your project will look like:

```
your-project/
├── prism.project.yaml           # Project config
├── configs/
│   ├── validator.py             # Optional: custom validator
│   ├── base.yaml                # Base configuration
│   └── prism/
│       └── lr_sweep.yaml        # Sweep definition
├── scripts/
│   └── train.py                 # Your training script
└── outputs/                     # Generated by PRISM
    ├── lr_experiment.prism      # Study state (created by PRISM)
    └── lr_experiment/           # Experiment outputs (created by PRISM)
        ├── lr_low/
        │   ├── config.yaml      # Validated config for this experiment
        │   └── ...              # Your training outputs
        ├── lr_mid/
```

---

## Sweep Definition Syntax

### Named Parameters (Nominal)

```yaml
# Creates experiments with custom names
model:
  size:
    small_model: small
    large_model: large
  layers:
    small_model: 12
    large_model: 24
```

Creates experiments: `small_model`, `large_model`

### Positional Parameters (Lists - Zipped Together)

```yaml
# Creates experiments run_0, run_1, run_2
batch_size: [16, 32, 64]
learning_rate: [0.01, 0.001, 0.0001]
```
- `run_0`: batch_size=16, lr=0.01
- `run_1`: batch_size=32, lr=0.001  
- `run_2`: batch_size=64, lr=0.0001

### Sweep Definitions (Advanced)

```yaml
# Choice sweep
optimizer:
  lr:
    _type: choice
    _values: [0.001, 0.01, 0.1]

# Range sweep
epochs:
  _type: range
  _min: 10
  _max: 100
  _step: 10

# Linspace sweep
momentum:
  _type: linspace
  _min: 0.0
  _max: 0.99
  _num: 5
```

### Multiple Files (Cartesian Product)

```bash
prism create --base base.yaml \
             --prism models.yaml \
             --prism optimizers.yaml \
             --name model_opt_sweep
```
If `models.yaml` has 3 variations and `optimizers.yaml` has 2, you get 3×2=6 experiments.

### Complete Example

Base config (`configs/base.yaml`):
```yaml
optimizer:
  type: adam
  lr: 0.001
  weight_decay: 0.0001
model:
  backbone: resnet50
  hidden_dim: 512
```

Sweep config (`configs/prism/lr_sweep.prism.yaml`):
```yaml
optimizer:
  lr:
    lr_low: 0.0001
    lr_mid: 0.001
    lr_high: 0.01
```

This creates 3 experiments:
- `lr_low`: with optimizer.lr = 0.0001
- `lr_mid`: with optimizer.lr = 0.001  
- `lr_high`: with optimizer.lr = 0.01

All other parameters stay the same from base config.

---

## Metrics Capture

PRISM can capture metrics in three ways:

### 1. JSON stdout (default)

Your training script prints JSON:
```python
import json
print(json.dumps({"loss": 0.5, "accuracy": 0.92}))
```

### 2. File output

```yaml
# In prism.project.yaml
metrics:
  output_mode: file
  output_file: metrics.json
```

Your script writes `metrics.json` in the output directory.

### 3. Exit code only

```yaml
metrics:
  output_mode: exit_code
```
PRISM only checks if the script succeeded (exit code 0).

---

## Advanced Features

### Custom Train Arguments

You can customize how PRISM calls your training script:

```yaml
# In prism.project.yaml  
paths:
  train_script: scripts/train.py
  train_args:
    - --config
    - "{config_path}"
    - --gpu
    - "0"
```

### Resume Failed Experiments

```bash
prism retry study_name
```
Or in TUI: Study Menu → Retry Failed

---

## Tips

1. **Start small**: Test with 2-3 experiments before scaling up
2. **Use the TUI**: It's much easier than CLI for exploration  
3. **Validate early**: Run one experiment manually before creating a big sweep
5. **Check metrics**: Make sure your training script prints/writes them correctly

---

## Troubleshooting

**"No train_script defined"**
→ Add `paths.train_script: scripts/train.py` to `prism.project.yaml`

**"Validator module not found"**  
→ Check the path in `validator.module` is correct relative to project root

**"Config validation failed"**
→ Check your `validate()` function - it's rejecting the config

**Experiments hang at "Testing data loading"**
→ Fixed! Make sure you have latest version with `PYTHONUNBUFFERED=1`

→ Fixed! Latest version forces colors with `FORCE_COLOR=1`

---
- Python 3.8+
- `pyyaml`
- `rich` (for TUI)
That's it! No other dependencies.

---
- **Organized**: Never lose track of which experiment used which parameters
- **Validated**: Catch config errors before training starts
- **Interactive**: TUI makes it easy to monitor and manage experiments

Ready to start? Run `prism_tui` and follow the prompts!  
