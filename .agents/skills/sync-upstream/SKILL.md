---
name: sync-upstream
description: >-
  Syncs the fork branches (main, feature/wifi-manager, release/1.0.x, and release/1.0.x-no-wifi-mgr)
  with the upstream repository (lmarzen/esp32-weather-epd). Use this skill whenever the user asks
  to sync with upstream, update branches from upstream, or check for upstream updates with human-in-the-loop conflict resolution.
---

# Sync Upstream Skill

This skill synchronizes changes from the upstream project (`lmarzen/esp32-weather-epd`) into your local fork branches while keeping you in the loop whenever a merge conflict arises.

## Branch Hierarchy & Flow

```
upstream/main (lmarzen/esp32-weather-epd)
     │
     ▼
   main (clean mirror)
     ├──► feature/wifi-manager (clean wifi feature for PR)
     │         │
     │         ▼
     │    release/1.0.x (local configs + wifi feature)
     │
     └──► release/1.0.x-no-wifi-mgr (local configs only)
```

## Step-by-Step Procedure

### 1. Pre-flight Checks
1. Ensure the working tree is clean:
   ```bash
   git status --porcelain
   ```
   If dirty, ask the user to commit or stash changes before proceeding.

2. Verify remotes:
   ```bash
   git remote -v
   ```
   Ensure `upstream` points to `https://github.com/lmarzen/esp32-weather-epd.git` and `rerere.enabled` is `true`.

### 2. Fetch Upstream Changes
Fetch the latest commits from upstream:
```bash
git fetch upstream main
```
Check if there are new commits to merge:
```bash
git log --oneline main..upstream/main
```
If empty, notify the user that `main` is already up to date with `upstream/main`. Ask if they still want to propagate `main` through the rest of the branch cascade.

### 3. Step A: Update `main`
`main` is a clean tracking branch of upstream:
```bash
git checkout main
git merge --ff-only upstream/main
```
*(If `--ff-only` fails because local `main` has diverged, stop and ask the user how to reconcile).*

### 4. Step B: Update `feature/wifi-manager`
Merge `main` into `feature/wifi-manager`:
```bash
git checkout feature/wifi-manager
git merge main -m "merge upstream main into feature/wifi-manager"
```
**Conflict Handling**:
- If conflicts occur, run `git status` and show conflicted files.
- Inspect the conflicts and suggest a resolution to the user.
- Ask for user confirmation before committing the resolution.
- Complete the merge: `git commit`.

### 5. Step C: Update `release/1.0.x`
Merge `feature/wifi-manager` into `release/1.0.x`:
```bash
git checkout release/1.0.x
git merge feature/wifi-manager -m "merge feature/wifi-manager into release/1.0.x"
```
**Conflict Handling**:
- If conflicts occur (often in config/customization files like `include/config.h` or `src/config.cpp`), explain what upstream changed vs. what local customized configs exist.
- Ask the user which settings to preserve.
- Complete the merge after user confirmation.

### 6. Step D: Update `release/1.0.x-no-wifi-mgr`
Merge `main` into `release/1.0.x-no-wifi-mgr`:
```bash
git checkout release/1.0.x-no-wifi-mgr
git merge main -m "merge upstream main into release/1.0.x-no-wifi-mgr"
```
**Conflict Handling**:
- Same process as Step C for any configuration file conflicts.

### 7. Step E: Push & Verification (Human Approval)
Before pushing to `origin`:
1. Check the summary of changes on each branch:
   ```bash
   git status
   git log --oneline origin/<branch>..<branch>
   ```
2. Ask the user for confirmation before pushing to remote `origin`:
   ```bash
   git push origin main feature/wifi-manager release/1.0.x release/1.0.x-no-wifi-mgr
   ```
