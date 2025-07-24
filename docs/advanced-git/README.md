# Advanced Git – Submodules, Workflows, and Beyond

## Table of Contents

1. [Git Submodules](#1-git-submodules)
2. [Git Workflows and Practices](#2-git-workflows-and-practices)
3. [Advanced File Handling & Configuration](#3-advanced-file-handling--configuration)
4. [Hosting Platforms](#4-hosting-platforms)
5. [Further Reading](#5-further-reading)

---

## 1. Git Submodules

### What is a Submodule?
A **submodule** is a repository embedded inside another repository. It allows you to keep a Git repo as a subdirectory of another repo, useful for including libraries or dependencies that are developed independently.

**Use cases:**
- Vendor third-party code (e.g., a library you don’t want to copy-paste)
- Share code between multiple projects

### Adding a Submodule
```bash
git submodule add https://github.com/example/libfoo.git libs/libfoo
git commit -m "Add libfoo as a submodule"
```

### Cloning a Repo with Submodules
```bash
git clone --recurse-submodules <repo-url>
# Or, after cloning:
git submodule update --init --recursive
```

### Updating Submodules
```bash
cd libs/libfoo
git checkout main
git pull
cd ../..
git add libs/libfoo
git commit -m "Update libfoo submodule"
```

### Removing a Submodule
```bash
git submodule deinit -f libs/libfoo
git rm -f libs/libfoo
rm -rf .git/modules/libs/libfoo
```

### Submodule Pitfalls
- Submodules are **pointers** to a specific commit, not a branch.
- Contributors must remember to update submodules and commit the change.
- CI/CD must initialize submodules for builds to work.

---

## 2. Git Workflows and Practices

### Centralized Workflow
- Single main branch (e.g., `main` or `master`)
- All changes pushed directly (rare in modern teams)

### Feature Branch Workflow
- Each feature/fix is developed in its own branch
- Merge to `main` via pull/merge request

### Git Flow
- Branches: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`
- Strict process for releases and hotfixes
- Good for large projects with scheduled releases

### GitHub Flow
- Only `main` branch is permanent
- Short-lived feature branches, always deployable
- Pull requests for all changes

### Trunk-Based Development
- Everyone works on a single branch (`main`)
- Feature flags for incomplete work
- Frequent, small merges

### Best Practices
- Write clear commit messages (imperative, present tense)
- Rebase before merging to keep history clean
- Use `git stash` to save work-in-progress
- Use `git cherry-pick` to apply specific commits
- Protect main branches with required reviews and CI

---

## 3. Advanced File Handling & Configuration

### .gitignore and .gitkeep
- `.gitignore` – lists files/directories to exclude from version control
- `.gitkeep` – empty file to force Git to track otherwise-empty directories (not a Git feature, just a convention)

### .gitattributes
- Control end-of-line normalization, diff/merge drivers, and file export behavior
- Example: treat all `.sh` files as text and use custom diff
```gitattributes
*.sh text diff
*.jpg binary
```

### Large File Support (LFS)
- Use [Git LFS](https://git-lfs.github.com/) for versioning large binaries
```bash
git lfs install
git lfs track "*.psd"
git add .gitattributes
```

### Sparse Checkout
- Check out only a subset of files/directories
```bash
git sparse-checkout init --cone
git sparse-checkout set docs/
```

### Partial Clone
- Download only parts of a repository (useful for monorepos)
```bash
git clone --filter=blob:none --no-checkout <repo-url>
```

### Hooks
- Automate actions (e.g., pre-commit linting, post-merge notifications)
- Place scripts in `.git/hooks/` (client-side) or use server-side hooks

---

## 4. Hosting Platforms

### GitHub
- Largest public code host
- Pull requests, Actions (CI/CD), Discussions, Pages
- Free for public and private repos

### GitLab
- Self-hosted or cloud
- Built-in CI/CD, issue tracking, merge requests
- Fine-grained permissions

### Bitbucket
- Integrates with Jira, Trello
- Free private repos for small teams
- Pipelines for CI/CD

### Azure DevOps Repos
- Deep integration with Azure pipelines and boards
- Enterprise features

### Gitea / Gogs
- Lightweight, self-hosted alternatives

---

## 5. Further Reading
- [Pro Git Book](https://git-scm.com/book/en/v2)
- [Git Submodules Documentation](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)
- [GitHub Docs](https://docs.github.com/en) 