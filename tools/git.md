# Git Workflows & Branch Management

## Quick Commands

### Repository Initialization

```bash
# New repository setup
git init
git add *
git commit -m "Initial commit"
git branch -M main
git remote add origin git@github.com:username/repo.git
git push -u origin main

# Existing repository
git remote add origin git@github.com:username/repo.git
git branch -M main
git push -u origin main
```

### Branch Management

```bash
# Update all formal branches
git fetch --all
current_branch=$(git rev-parse --abbrev-ref HEAD)
for branch in production main develop squad1 squad2 squad3 squad4; do
  echo -e "\nBranch: $branch\n"
  git checkout $branch
  git pull --prune --rebase --all
done
git checkout $current_branch
```

### Cleanup Commands

```bash
# Remove local branches (keep formal ones)
git branch | grep -vE "main|develop|production|squad" | xargs git branch -D

# Discard all unstaged changes
git clean -df
git checkout -- .

# Advanced cleanup with dry-run
git clean -dxn .  # inspect files to be removed
git clean -dxf .  # force remove
```

## Common Patterns

### Formal Branch Strategy

- **main**: Production-ready code
- **develop**: Integration branch for features
- **production**: Live deployment branch
- **squad[1-4]**: Team-specific feature branches

### Safe Rebase Workflow

```bash
# Always fetch before rebasing
git fetch --all
git pull --prune --rebase --all
```

## Personal Gotchas

### Branch Switching Safety

Always save current branch before batch operations:

```bash
current_branch=$(git rev-parse --abbrev-ref HEAD)
# ... operations ...
git checkout $current_branch
```

### Unstaged Changes Recovery

Use `git clean -dxn .` for dry-run inspection before force cleanup. Prevents accidental loss of important untracked files.

## Performance Notes

- Use `--prune` to remove stale remote references
- Use `--rebase` to maintain linear history
- Batch branch updates reduce remote calls

## Context Links

- [Git Documentation](https://git-scm.com/docs)
- [Unstaged Changes Discussion](https://stackoverflow.com/questions/52704/how-do-i-discard-unstaged-changes-in-git)

## Team Workflow Integration

### Multi-Squad Environment

```bash
# Quick squad branch creation
git checkout -b squad1/feature-name
git push -u origin squad1/feature-name

# Merge strategy for squads
git checkout develop
git merge --no-ff squad1/feature-name
```
