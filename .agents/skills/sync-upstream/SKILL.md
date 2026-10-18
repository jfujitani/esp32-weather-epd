---
name: sync-upstream
description: >-
  Syncs the fork with upstream (lmarzen/esp32-weather-epd). Updates main and feature branches,
  then interactively asks the user which local branches to sync and what parent to merge from.
  Includes human-in-the-loop conflict resolution and strictly limits pushes to public branches.
---

# Sync Upstream Skill

This skill synchronizes changes from the upstream project (`lmarzen/esp32-weather-epd`) into your local repository.

## Branch Categories

1. **Public Remote Branches (`main`, `feature/*`)**: Clean tracking and contribution branches intended to push to `origin`.
2. **Local-Only Branches**: Private branches (e.g. customized configs, test branches) that are kept strictly local and **never pushed to remote**.

## Step-by-Step Procedure

### 1. Pre-flight Checks
1. Check that working tree is clean:
   ```bash
   git status --porcelain
   ```
2. Note the currently checked-out branch to restore it at the end.
3. Ensure `upstream` remote exists (`https://github.com/lmarzen/esp32-weather-epd.git`) and `rerere.enabled` is `true`.

### 2. Fetch Upstream
```bash
git fetch upstream main
```
Check if there are new commits:
```bash
git log --oneline main..upstream/main
```

### 3. Update `main`
```bash
git checkout main
git merge --ff-only upstream/main
```

### 4. Update Public Feature Branches
For each public feature branch (e.g., `feature/wifi-manager`):
```bash
git checkout <feature-branch>
git merge main -m "merge upstream main into <feature-branch>"
```
- If conflicts arise: display conflicted files, propose a resolution, and wait for human confirmation before committing.

### 5. Interactively Sync Local Branches
1. List available local branches:
   ```bash
   git branch --list
   ```
2. Ask the user:
   - Which local branches they want updated.
   - For each selected branch, which source/parent branch to merge in (e.g., `main` or a specific feature branch).
3. For each selected local branch:
   ```bash
   git checkout <local-branch>
   git merge <chosen-parent-branch>
   ```
- **Conflict Handling**: Present the diffs, explain the conflicting lines, and ask the user how to resolve them before committing.

### 6. Push Public Branches (Human Approval)
1. Display outgoing commits for public branches only (`main`, `feature/*`).
2. Ask the user for confirmation before pushing.
3. Push **only** approved public branches:
   ```bash
   git push origin main <approved-feature-branches...>
   ```
> [!CAUTION]
> **Never push local branches to `origin`.**

### 7. Restore Working Branch
Return to the branch originally checked out in Step 1.
