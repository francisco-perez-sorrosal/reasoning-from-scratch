# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains educational code for the book "Build a Reasoning Model (From Scratch)" by Sebastian Raschka (Manning, 2025). The project implements reasoning capabilities on top of pre-trained LLMs (specifically Qwen3) using inference-time scaling, reinforcement learning, and distillation techniques.

**Important**: This repository corresponds to a published print book. Do not make changes that would extend or deviate from the main chapter code, as consistency with the physical book is essential for readers.

## Python Environment

**ALWAYS use `uv run` prefix for all Python commands** to ensure proper environment isolation:

```bash
uv run python script.py
uv run pytest tests
uv run jupyter lab
```

The project uses `uv` for dependency management with a `pyproject.toml` file. A `.venv` virtual environment is automatically created on first use.

### Package Installation

- Core dependencies are defined in `pyproject.toml`
- Additional packages can be installed via: `uv add package-name`
- The project includes a `reasoning_from_scratch` package with reusable utilities

## Git Workflow for Fork Management

This repository is a fork of `rasbt/reasoning-from-scratch:main`. The workflow supports both personal experiments and upstream contributions.

### Remote Configuration

- **origin**: Your fork (francisco-perez-sorrosal/reasoning-from-scratch) - push your changes here
- **upstream**: Original repository (rasbt/reasoning-from-scratch) - sync from here

Configure upstream remote:

```bash
git remote add upstream https://github.com/rasbt/reasoning-from-scratch.git
git fetch upstream
```

### Branch Naming Conventions

Use clear branch names to separate experiments from contributions:

**Experiment branches** (`exp/*`):

- Format: `exp/<descriptive-name>`
- Examples: `exp/chapter3-modifications`, `exp/custom-verifier`
- For personal work that diverges from upstream
- Can merge from upstream (preserves experiment history)
- Keep notebook outputs for your reference

**Contribution branches** (`contrib/*`):

- Format: `contrib/<issue-or-feature>`
- Examples: `contrib/fix-math-checker`, `contrib/improve-readme`
- For PRs to upstream repository
- Always rebase on upstream/main for clean history
- Clear notebook outputs before committing: `jupyter nbconvert --clear-output --inplace notebook.ipynb`

**Main branch**:

- Keep synchronized with upstream/main
- Never commit directly to main
- Only merge/rebase from upstream

### Daily Workflow

**Starting new work:**

```bash
# Update main from upstream
git checkout main
git fetch upstream
git merge upstream/main --ff-only  # Fast-forward only
git push origin main

# Start experiment
git checkout -b exp/my-experiment

# Start contribution
git checkout -b contrib/fix-issue-123
```

**Syncing experiment branch:**

```bash
git checkout exp/my-experiment
git fetch upstream
git merge upstream/main  # Or rebase if preferred
```

**Preparing contribution for PR:**

```bash
# Update main first
git checkout main
git fetch upstream
git merge upstream/main --ff-only
git push origin main

# Rebase contribution branch for clean history
git checkout contrib/fix-issue-123
git rebase main

# Clear notebook outputs before final commit
jupyter nbconvert --clear-output --inplace path/to/notebook.ipynb

# Force push to your fork (safe on your branch)
git push origin contrib/fix-issue-123 --force-with-lease
```

### Jupyter Notebook Conflict Resolution

This repository uses **nbdime** for intelligent notebook merging and diffing.

**Handling conflicts during rebase/merge:**

When rebase/merge encounters a notebook conflict, nbdime automatically launches a web UI for visual conflict resolution:

```bash
# nbdime will launch automatically, or manually:
nbdime mergetool

# After resolving conflicts:
git add resolved-notebook.ipynb
git rebase --continue
```

**View notebook diffs:**

```bash
# See semantic differences in notebooks
nbdime diff notebook.ipynb

# Compare with another branch
nbdime diff main HEAD -- notebook.ipynb
```

**Best practices:**

- Keep outputs in experiment branches for your reference
- Clear outputs in contribution branches before committing
- Use nbdime's web UI for complex conflicts
- Avoid committing large data files embedded in notebooks
- Test code runs before submitting: `uv run pytest tests`

### Environment Management

This fork tracks `uv.lock` for reproducible environments (upstream ignores it):

- `uv.lock` is committed to maintain consistent dependencies across your machines
- When syncing with upstream, resolve uv.lock conflicts by running: `uv lock`
- This ensures experiments remain reproducible over time

## Common Commands

### Testing

```bash
# Run all tests
uv run pytest tests

# Run tests for a specific chapter
uv run pytest tests/test_ch02.py

# Run a specific test
uv run pytest tests/test_ch02.py::test_function_name
```

Tests may use `SKIP_EXPENSIVE` environment variable to skip costly operations in CI.

### Running Scripts

**Chapter-specific evaluation scripts** (e.g., MATH-500 benchmarks):

```bash
# Basic evaluation (sequential)
uv run ch03/02_math500-verifier-scripts/evaluate_math500.py

# Batched evaluation (parallel generation)
uv run ch03/02_math500-verifier-scripts/evaluate_math500_batched.py --batch_size 128

# Inference-time scaling experiments
uv run ch04/02_math500-inference-scaling-scripts/cot_prompting_math500.py
uv run ch04/02_math500-inference-scaling-scripts/self_consistency_math500.py
```

Common script options:

- `--device`: Device to use ("auto", "cpu", "cuda", "mps")
- `--compile`: Enable `torch.compile()` for optimization
- `--dataset_size`: Number of examples to evaluate
- `--max_new_tokens`: Maximum generation length

### Jupyter Notebooks

```bash
uv run jupyter lab
```

Main chapter code is in Jupyter notebooks (e.g., `ch02/01_main-chapter-code/ch02_main.ipynb`).

## Code Architecture

### Core Module Structure (`reasoning_from_scratch/`)

The package is organized by chapter and provides reusable implementations:

- **`qwen3.py`**: Main Qwen3 LLM implementation
  - `Qwen3Model`: Transformer model with KV-cache support
  - `QWEN_CONFIG_06_B`: 0.6B parameter model configuration
  - Uses bfloat16 precision, RoPE embeddings, Grouped Query Attention (GQA)
  - Supports both prefill and cached decoding modes

- **`qwen3_optimized.py`**: GPU-optimized variant (Appendix C)
  - Uses `torch.nn.functional.scaled_dot_product_attention`
  - Pre-allocated KV-cache for better memory efficiency
  - Drop-in replacement for `qwen3.py`

- **`qwen3_batched.py`**: Batched inference support
  - Parallel generation across multiple examples
  - Efficient handling of sequences with stop tokens

- **`ch02.py`**: Text generation utilities
  - `generate_text_basic()`: Basic autoregressive generation
  - `generate_text_basic_cache()`: Generation with KV-cache
  - `get_device()`: Automatic device selection (CUDA/MPS/CPU)

- **`ch03.py`**: Evaluation utilities
  - MATH-500 dataset evaluation
  - Answer verification using sympy

- **`ch04.py`**: Inference-time scaling methods
  - Chain-of-thought prompting
  - Self-consistency with majority voting

- **`utils.py`**: Shared utilities
  - `download_file()`: Downloads model weights with progress tracking and backup URLs

### Chapter Organization

Each chapter follows a consistent structure:

```text
chXX/
├── 01_main-chapter-code/
│   ├── chXX_main.ipynb          # Main chapter content
│   └── chXX_exercise-solutions.ipynb  # Exercise solutions
├── 02_bonus-material/           # Optional experiments/extensions
└── README.md                    # Chapter-specific documentation
```

### Model Loading

Models are loaded from Hugging Face or local paths:
- Base model: Qwen3 0.6B pretrained
- Reasoning model: Qwen3 with reasoning capabilities added via training

Weight files are downloaded automatically to a local directory on first use via `download_file()`.

### Key Design Patterns

1. **Dual Implementation Approach**: Core concepts have both:

   - Jupyter notebook implementations (chapter code, pedagogical)
   - Python module implementations (`reasoning_from_scratch/`, reusable)

2. **KV-Cache Management**: Generation functions use KV-cache to avoid recomputing attention:

   - Cache is reset via `model.reset_kv_cache()` before generation
   - Supports both single and batched generation

3. **Device Abstraction**: Code auto-detects and uses available accelerators (CUDA > MPS > CPU)

4. **Batched Processing**: Evaluation scripts support both sequential and batched modes:

   - Sequential: Lower memory, easier debugging
   - Batched: Higher throughput, requires more VRAM

## Hardware Considerations

- Chapters 2-4: Work well on CPU and GPU
- Chapters 5-6: GPU recommended for reasonable training times
- Batched inference requires significant VRAM (use `--batch_size` to tune)
- MPS (Apple Silicon) may need `PYTORCH_ENABLE_MPS_FALLBACK=1` for some operations

## Testing Philosophy

Tests ensure code correctness across platforms (Linux, macOS, Windows) via GitHub Actions. Use dummy models and small examples to keep tests fast. Expensive operations are skipped in CI via `SKIP_EXPENSIVE` environment variable.
