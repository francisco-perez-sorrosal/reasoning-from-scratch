# Developer Workflow Guide

Quick reference for working with this fork of `rasbt/reasoning-from-scratch`.

## Daily Workflow (Happy Path)

### 1. Start New Work

```bash
# Create experiment branch
git checkout main
git checkout -b exp/my-feature

# Or create contribution branch
git checkout main
git checkout -b contrib/fix-something
```

### 2. Work on Your Branch

```bash
# Make changes, run tests
uv run pytest tests

# Commit your work
git add .
git commit -m "Your descriptive message"

# Push to your fork
git push origin exp/my-feature
```

### 3. Sync with Upstream (When Needed)

When upstream has updates you want to incorporate:

```bash
# Update main from upstream
git checkout main
git fetch upstream
git merge upstream/main --ff-only
git push origin main

# Rebase your branch on updated main
git checkout exp/my-feature
git rebase main

# Force push your rebased branch (safe on your branch)
git push origin exp/my-feature --force-with-lease
```

## Branch Types

| Type | Format | Purpose | Notebook Outputs |
|------|--------|---------|------------------|
| Experiment | `exp/description` | Personal experiments, learning | Keep outputs |
| Contribution | `contrib/issue-or-fix` | PRs to upstream | Clear outputs |
| Main | `main` | Sync with upstream only | Never commit directly |

## Common Tasks

### Create and Switch Branches

```bash
# Start experiment
git checkout -b exp/chapter3-custom-verifier

# Start contribution
git checkout -b contrib/fix-math-parser
```

### Push Your Work

```bash
# First time pushing a new branch
git push -u origin exp/my-branch

# Subsequent pushes
git push
```

### Sync Main with Upstream

```bash
git checkout main
git fetch upstream
git merge upstream/main --ff-only
git push origin main
```

### Rebase Your Branch

```bash
# After syncing main with upstream
git checkout exp/my-branch
git rebase main

# If you already pushed, force push
git push origin exp/my-branch --force-with-lease
```

### Handle Notebook Conflicts

When rebase encounters notebook conflicts:

```bash
# nbdime launches automatically, or manually:
uv run nbdime mergetool

# After resolving:
git add resolved-notebook.ipynb
git rebase --continue
```

### Clear Notebook Outputs (Before Contribution PR)

```bash
# Clear outputs from notebooks
uv run jupyter nbconvert --clear-output --inplace path/to/notebook.ipynb

# Verify with nbdime
uv run nbdime diff notebook.ipynb
```

## Testing Before Commits

```bash
# Run all tests
uv run pytest tests

# Run specific test file
uv run pytest tests/test_ch02.py

# Run specific test
uv run pytest tests/test_ch02.py::test_name
```

## Preparing a Contribution PR

```bash
# 1. Sync main
git checkout main
git fetch upstream
git merge upstream/main --ff-only
git push origin main

# 2. Rebase your contribution branch
git checkout contrib/my-fix
git rebase main

# 3. Clear notebook outputs
uv run jupyter nbconvert --clear-output --inplace path/to/*.ipynb

# 4. Run tests
uv run pytest tests

# 5. Force push
git push origin contrib/my-fix --force-with-lease

# 6. Create PR on GitHub from your branch to upstream/main
```

## Environment Commands

```bash
# Always use uv run for Python commands
uv run python script.py
uv run pytest tests
uv run jupyter lab

# Add package
uv add package-name

# Sync environment
uv sync
```

## Git Remotes

- **origin**: Your fork (`francisco-perez-sorrosal/reasoning-from-scratch`)
- **upstream**: Original repo (`rasbt/reasoning-from-scratch`)

```bash
# Verify remotes
git remote -v
```

## Quick Reference

```bash
# Start work
git checkout main
git checkout -b exp/my-work

# Commit changes
git add .
git commit -m "Message"
git push origin exp/my-work

# Sync with upstream
git checkout main
git fetch upstream
git merge upstream/main --ff-only
git push origin main

# Rebase your work
git checkout exp/my-work
git rebase main
git push origin exp/my-work --force-with-lease
```

## Troubleshooting

### Rebase Conflicts

```bash
# If conflicts during rebase:
# 1. Fix conflicts manually or use nbdime for notebooks
uv run nbdime mergetool  # For .ipynb files

# 2. Stage resolved files
git add resolved-file

# 3. Continue rebase
git rebase --continue

# 4. Or abort if needed
git rebase --abort
```

### Main is Behind Upstream

```bash
# This is normal - just sync it
git checkout main
git fetch upstream
git merge upstream/main --ff-only
git push origin main
```

### Force Push Safety

Always use `--force-with-lease` instead of `--force`:

```bash
# Safe - checks remote hasn't changed
git push origin branch-name --force-with-lease

# Unsafe - overwrites remote unconditionally
git push origin branch-name --force  # DON'T USE
```

## Additional Resources

- Complete workflow details: See [CLAUDE.md](CLAUDE.md)
- Cursor IDE rules: See [.cursorrules](.cursorrules)
- Upstream repository: <https://github.com/rasbt/reasoning-from-scratch>
- Your fork: <https://github.com/francisco-perez-sorrosal/reasoning-from-scratch>
