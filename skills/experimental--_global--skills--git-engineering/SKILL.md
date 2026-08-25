---
name: "git-engineering"
description: "Use for Git workflow mechanics: branching, conventional commits, rebase and history cleanup, interactive bisect, submodules, and the pre-flight .gitignore check before staging. For research-specific commit conventions and paper branches, use git-research instead."
---

# Git Engineering Skill — Software Engineering Git Workflows

## Core Rule
> Git history must be linear, descriptive, and clean.  
> Ambiguous commit messages like `WIP`, `fixed`, `asdf` are strictly forbidden.

For research repositories — experiment commit tags, paper branches, model weight exclusion — use `git-research` instead.

---

## 1. Branching Model

```
main       ───●──────────────●───────────────●─── (Always stable & passing tests)
              \             /
feature/env    └──●────●───┘ (Feature branch)
```

- `main` / `master`: Production-ready, passing all unit and integration tests.
- `feature/<name>`: Isolated feature development.
- `fix/<name>`: Bug fixes.

---

## 2. Conventional Commit Standards

```
<type>(<scope>): <short description>

[optional detailed body]
```

### Valid Commit Types
- `feat`: New feature or capability
- `fix`: Bug fix
- `docs`: Documentation updates
- `refactor`: Code quality improvement without changing observable behavior
- `test`: Adding or modifying unit/integration tests
- `chore`: Configuration, tool, or dependency updates

---

## 3. Advanced Git Operations (Rebase & Bisect)

```bash
# 1. Interactive Rebase (Clean last 4 commits)
git rebase -i HEAD~4

# 2. Binary search to isolate breaking commit (Git Bisect)
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
git bisect run pytest tests/test_failing.py
```

---

## 4. Pre-Flight `.gitignore` Verification

> **Mandatory Rule**: Before executing `git add` or `git commit`, ALWAYS check if target files are ignored by `.gitignore`.

```bash
# Check if a specific file is ignored before staging
git check-ignore -v <path/to/file>

# Check untracked files while excluding ignored files
git status -u
```

Do NOT attempt to stage or commit ignored files, temporary build outputs, SQLite databases (`.db`), or logs.

---

## 5. Submodules

```bash
# Add
git submodule add <url> <path>

# Clone a repo that has submodules
git clone --recurse-submodules <url>

# Update an existing checkout
git submodule update --init --recursive
```

A submodule pointer is a commit SHA, not a branch. Updating the submodule's upstream does nothing until you commit the new pointer in the parent repo.

