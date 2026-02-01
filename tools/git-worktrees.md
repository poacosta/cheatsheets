# Git Worktrees – Cheatsheet

Worktrees let you check out **multiple branches of the same Git repository at the same time**, each in its own directory, without cloning the repo.

---

## Mental Model

- **One Git repository**
- **Multiple working directories**
- **One branch per directory**

Think: **one repo → many checkouts**

---

## Basic Commands

### Create a worktree for an existing branch

```bash
git worktree add <path> <branch>
```

Example:

```bash
git worktree add ../my-project-feature feature/login
```

---

### Create a worktree + new branch

```bash
git worktree add -b <new-branch> <path>
```

Example:

```bash
git worktree add -b feature/payments ../my-project-payments
```

---

### List all worktrees

```bash
git worktree list
```

---

### Remove a worktree (safe way)

```bash
git worktree remove <path>
```

Example:

```bash
git worktree remove ../my-project-feature
```

> ⚠️ This removes the directory, **not the branch**

---

### Delete the branch after removing the worktree

```bash
git branch -d <branch>
```

---

### Clean up missing/deleted worktrees

```bash
git worktree prune
```

---

## Typical Directory Layout

```text
projects/
  my-project/               → main
  my-project-feature-x/     → feature/x
  my-project-bugfix-y/      → bugfix/y
```

---

## Daily Workflow Example

```bash
# main work
cd my-project
git pull
git commit -am "Hotfix production issue"

# parallel feature work
cd ../my-project-feature
git commit -am "Implement login validation"
```

No branch switching. No stashing.

---

## Pull Request Review (Local)

```bash
git fetch origin pull/123/head:pr-123
git worktree add ../pr-123 pr-123
```

---

## Rules & Gotchas

### One branch = one worktree

❌ Not allowed:

```bash
git worktree add ../copy main
```

---

### Never delete worktree directories manually

✅ Always use:

```bash
git worktree remove <path>
```

If already deleted:

```bash
git worktree prune
```

---

### Shared vs Isolated

**Shared across worktrees**

- Git objects
- Remotes
- History

**Isolated per worktree**

- `node_modules`
- `.venv`
- build artifacts
- environment state

---

## When Worktrees Shine

- Feature + hotfix in parallel
- Reviewing PRs locally
- Long-running refactors
- Experimental spikes
- Avoiding constant branch switching

---

## When NOT to Use Worktrees

- You need to share uncommitted changes across branches
- Tooling hardcodes absolute paths
- You’re not comfortable with CLI Git yet

---

## Comparison

| Approach   | Pros | Cons |
|-----------|------|------|
| Worktrees | Parallel work, fast, clean | One branch per dir |
| Stashing  | Simple | Easy to lose context |
| Cloning   | Fully isolated | Slow, duplicated data |

---

## TL;DR

```bash
git worktree add ../dir branch     # add
git worktree list                  # list
git worktree remove ../dir         # remove
```

Worktrees = **parallel Git without pain**
