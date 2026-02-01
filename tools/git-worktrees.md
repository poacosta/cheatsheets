# Git Worktrees – Cheatsheet

Worktrees let you check out **multiple branches of the same Git repository simultaneously**, each in its own directory. One `.git` folder, multiple working directories.

---

## Mental Model

```
my-project/.git/          ← Single source of truth
my-project/               ← main (worktree 1)
my-project-feature/       ← feature/auth (worktree 2)
my-project-hotfix/        ← hotfix/critical (worktree 3)
```

**Key insight**: Worktrees share the Git object database but have isolated working directories.

---

## Quick Commands

### Create & Navigate

```bash
# Existing branch
git worktree add ../feature-dir existing-branch

# New branch from current HEAD
git worktree add -b new-branch ../new-dir

# New branch from specific commit/branch
git worktree add -b hotfix/issue-123 ../hotfix origin/main

# Create in subdirectory of current repo
git worktree add worktrees/feature-x feature-x
```

### Manage

```bash
# List all worktrees (shows branch, commit, path)
git worktree list

# Remove worktree (keeps branch)
git worktree remove ../feature-dir

# Force remove (even with uncommitted changes)
git worktree remove --force ../feature-dir

# Clean up deleted worktrees
git worktree prune

# Move/rename worktree
git worktree move ../old-path ../new-path
```

### Delete Branch After Removing Worktree

```bash
git worktree remove ../feature-dir
git branch -d feature-name          # safe delete (merged only)
git branch -D feature-name          # force delete
```

---

## Common Patterns

### 1. Parallel Feature Development

```bash
# Main work
cd ~/projects/app
git worktree add ../app-feature-auth feature/auth
git worktree add ../app-feature-payments feature/payments

# Now work in parallel without context switching
code ../app-feature-auth
code ../app-feature-payments
```

**Why this works**: Each directory has its own node_modules, .venv, build artifacts.

### 2. PR Review Workflow

```bash
# Fetch PR from GitHub/GitLab
git fetch origin pull/456/head:pr-456
git worktree add ../review-pr-456 pr-456

# Review, test, comment
cd ../review-pr-456
npm install && npm test

# Clean up when done
cd ../main-worktree
git worktree remove ../review-pr-456
git branch -d pr-456
```

### 3. Hotfix While Mid-Feature

```bash
# You're deep in feature work with uncommitted changes
cd ~/projects/app-feature

# Don't stash! Just switch worktrees
cd ~/projects/app-main
git pull
git worktree add ../app-hotfix -b hotfix/critical-bug

cd ../app-hotfix
# Fix bug, commit, push
git push -u origin hotfix/critical-bug

# Back to feature work (state preserved)
cd ../app-feature
# Continue where you left off
```

### 4. Long-Running Refactor

```bash
# Keep refactor isolated for weeks/months
git worktree add ../app-refactor -b refactor/architecture

# Periodically sync with main
cd ../app-refactor
git fetch origin
git rebase origin/main  # or merge, depending on strategy
```

### 5. Multi-Environment Testing

```bash
# Test different configs simultaneously
git worktree add ../test-node-18 main
git worktree add ../test-node-20 main

# Each can have different .nvmrc, dependencies
cd ../test-node-18 && nvm use 18 && npm test
cd ../test-node-20 && nvm use 20 && npm test
```

---

## Personal Gotchas

### ❌ Can't Check Out Same Branch Twice

```bash
# This will fail
git worktree add ../copy main
# error: 'main' is already checked out at '/path/to/main'
```

**Solution**: Create a new branch from that commit:
```bash
git worktree add -b main-copy ../copy main
```

### ❌ Deleting Directory Manually Breaks Git

```bash
rm -rf ../old-worktree  # DON'T DO THIS

# Now git thinks worktree still exists
git worktree list       # Shows broken path
git checkout branch     # Might fail
```

**Fix**:
```bash
git worktree prune      # Clean up after manual deletion
```

### ❌ Shared Git Hooks Can Cause Issues

All worktrees share `.git/hooks/`. If hooks have hardcoded paths:

```bash
# In .git/hooks/pre-commit
cd /absolute/path/to/main  # Breaks in other worktrees
```

**Solution**: Use `$GIT_WORK_TREE` environment variable:
```bash
cd "$GIT_WORK_TREE"  # Points to current worktree
```

### ❌ Submodules Get Weird

Submodules in worktrees can be tricky. Each worktree needs its own submodule checkout:

```bash
cd new-worktree
git submodule update --init --recursive
```

### ❌ IDE/Editor Confusion

Some tools (older VS Code, JetBrains) don't handle worktrees well:
- Search might span all worktrees
- Git plugins might get confused about current branch
- File watchers might trigger for all worktrees

**Workaround**: Open each worktree in separate editor window.

### ❌ CI/CD Absolute Paths

If your CI caches paths like `/home/runner/work/repo`:

```bash
# This breaks
npm install  # node_modules symlinks to main worktree
```

**Solution**: Don't use worktrees in CI; stick to clones.

---

## Performance Notes

### Disk Usage

**Worktrees are space-efficient**:
```bash
# Regular clones
repo-1/   → 500MB .git + 200MB working dir
repo-2/   → 500MB .git + 200MB working dir
Total:    → 1.4GB

# Worktrees
repo/.git/         → 500MB (shared)
repo/              → 200MB
repo-feature/      → 200MB
Total:             → 900MB
```

### Speed Comparison

```bash
# Clone (full history fetch)
time git clone https://github.com/large/repo new-clone
# ~30-60 seconds

# Worktree (instant)
time git worktree add ../new-worktree branch
# ~0.1 seconds
```

### When Performance Matters

- **Monorepos**: Huge win with large `.git` folders
- **Fast context switching**: No re-indexing, no branch checkout delays
- **CI/CD**: Slower than shallow clones for one-time builds

---

## Advanced Use Cases

### Bare Repository Setup (Cleanest IMO)

```bash
# Clone as bare repo
git clone --bare git@github.com:user/repo.git repo.git

cd repo.git

# All work in worktrees
git worktree add ../main main
git worktree add ../develop develop
git worktree add ../feature-x -b feature/x

# No "main" worktree confusion
```

**Directory structure**:
```
workspace/
  repo.git/           ← bare repo (just .git contents)
  main/               ← main branch
  develop/            ← develop branch
  feature-x/          ← feature branch
```

### Scripted Worktree Management

```bash
#!/bin/bash
# create-worktree.sh

REPO_ROOT="$HOME/projects/my-app"
WORKTREE_DIR="$HOME/worktrees"

branch_name="$1"
worktree_path="$WORKTREE_DIR/my-app-$branch_name"

cd "$REPO_ROOT"
git worktree add "$worktree_path" -b "$branch_name"
cd "$worktree_path"

# Auto-install dependencies
npm install
cp ../.env.example .env

echo "Worktree ready: $worktree_path"
code .
```

### Integration with Tools

**Tmux/terminal multiplexer**:
```bash
# Create session for each worktree
git worktree list | while read path branch commit; do
  tmux new-session -d -s "$(basename $path)"
  tmux send-keys -t "$(basename $path)" "cd $path" C-m
done
```

**Docker/container development**:
```bash
# Each worktree gets its own container
docker-compose -f ../main/docker-compose.yml -p main up
docker-compose -f ../feature/docker-compose.yml -p feature up
```

---

## Troubleshooting

### "Cannot Create Worktree" Errors

```bash
# Lock file exists
rm .git/worktrees/<name>/locked

# Worktree directory already exists
rm -rf <path> && git worktree prune
```

### Find Orphaned Worktrees

```bash
# List all registered worktrees
git worktree list

# Check which don't exist
for worktree in .git/worktrees/*; do
  path=$(cat "$worktree/gitdir")
  if [ ! -d "$path" ]; then
    echo "Orphaned: $worktree"
  fi
done

# Clean up
git worktree prune
```

### Recover Accidentally Deleted Worktree

```bash
# If you deleted directory but not via git worktree remove
git worktree prune                    # Remove metadata
git worktree add <path> <branch>      # Re-create
```

---

## When NOT to Use Worktrees

| Scenario | Why Not | Alternative |
|----------|---------|-------------|
| **CI/CD builds** | Added complexity, tooling expects clones | Shallow clone |
| **One-off task** | Overkill for quick branch switch | `git switch` |
| **Shared uncommitted state** | Worktrees are isolated | Stash or commit |
| **Learning Git** | More concepts to understand | Master basics first |
| **Team unfamiliar** | Support burden | Document & train |

---

## Decision Matrix

```
Need parallel work? ──→ YES ──→ Multiple features? ──→ YES ──→ Worktrees ✓
        │                              │
        NO                             NO
        │                              │
        ↓                              ↓
Simple switch? ────→ git switch    Same feature, different configs? ──→ Clones
```

---

## Comparison: Worktrees vs Alternatives

| Method | Speed | Disk | Complexity | Parallel Work | Shared State |
|--------|-------|------|------------|---------------|--------------|
| **Worktrees** | ⚡⚡⚡ | 💾 | 🧠🧠 | ✅ | Objects only |
| **Stashing** | ⚡⚡⚡ | 💾 | 🧠 | ❌ | Full |
| **Cloning** | ⚡ | 💾💾💾 | 🧠 | ✅ | None |
| **Switching** | ⚡⚡ | 💾 | 🧠 | ❌ | Full |

---

## TL;DR

```bash
# Create
git worktree add <path> <branch>           # Existing branch
git worktree add -b <new> <path>           # New branch

# Manage
git worktree list                          # Show all
git worktree remove <path>                 # Delete (keeps branch)
git worktree prune                         # Clean orphaned

# Use case
Parallel work > stashing > context switching pain
```

**Bottom line**: Worktrees shine when you need to work on multiple branches simultaneously without the overhead of multiple clones.

---

## Context Links

- [Official Git Docs](https://git-scm.com/docs/git-worktree)
- [Worktrees vs Cloning benchmarks](https://github.com/isacikgoz/gitin/wiki/Benchmarks) *(if you care about numbers)*
- Related: `git switch`, `git stash`, bare repositories
