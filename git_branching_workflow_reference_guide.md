# Git Branching & Workflow Reference Guide

**Environments:** Local → SIT → UAT → Production (`main`)  
**Practice Repository:** `https://github.com/tosifshaikh/git-branch-practice.git`

---

## 1. Branch Overview

| Branch | Lifetime | Purpose / Environment | How Code Enters |
| :--- | :--- | :--- | :--- |
| `feature/*`, `bugfix/*` | Short-lived (one task) | Local development | Created from `sit` |
| `sit` | Permanent | System Integration Testing (SIT) | PR from `feature/*` or `bugfix/*` |
| `uat` | Permanent | User Acceptance Testing (UAT) | PR from `sit` (merge commit) |
| `main` | Permanent | Production (Live) | PR from `uat` (merge commit) or `hotfix/*` |
| `hotfix/*` | Short-lived (urgent fix) | Production emergency fix | Created from `main` |

### Golden Rules
1. **Never commit directly** to `sit`, `uat`, or `main`—always use Pull Requests (PRs).
2. **Code moves forward only:** `feature/*` → `sit` → `uat` → `main`.
3. **Always use a merge commit** (not squash merge) when promoting `sit` → `uat` and `uat` → `main` to prevent branch history drift.
4. **Always back-merge hotfixes** from `main` → `uat` → `sit` so urgent fixes are not overwritten on the next release.

---

## 2. Complete End-to-End Walkthrough (Clone to Hotfix Merge)

Follow these exact steps in order from a brand-new repository through feature release and hotfix back-merge.

### Step 1: Clone the Empty Repository & Initialize `main`
```bash
git clone https://github.com/tosifshaikh/git-branch-practice.git
cd git-branch-practice/

# Create initial commit so the main branch exists
echo "#Git Branch Practice" > README.md
git add README.md
git commit -m "Initial Commit"
git push -u origin main
```

### Step 2: Create `uat` and `sit` Branches from `main`
```bash
# Create and push UAT from main
git checkout main
git pull origin main
git checkout -b uat
git push -u origin uat

# Create and push SIT from main
git checkout main
git checkout -b sit
git push -u origin sit

# Verify all branches exist locally and remotely
git branch -a
```

### Step 3: Create a Feature Branch from `sit` & Push Changes
```bash
git checkout sit
git pull origin sit
git checkout -b feature/practice

# Make changes, stage, commit, and push
echo "## practice Page" >> README.md
git add .
git commit -m "Add practice page section to README"
git push -u origin feature/practice
```

### Step 4: Merge Feature into `sit` (GitHub PR) & Clean Up
1. On GitHub, open a Pull Request: **Base:** `sit` ← **Compare:** `feature/practice`.
2. Merge the Pull Request into `sit`.
3. Pull the updated `sit` locally and delete the merged feature branch:
```bash
git checkout sit
git pull origin sit
git branch -d feature/practice
git push origin --delete feature/practice
```

### Step 5: Promote `sit` → `uat` (GitHub PR)
1. On GitHub, open a Pull Request: **Base:** `uat` ← **Compare:** `sit`.
2. Merge using **Create a merge commit**.
3. Sync your local `uat` branch:
```bash
git checkout uat
git pull origin uat
git log -n 2 --oneline
```

### Step 6: Promote `uat` → `main` (Production) & Tag Release
1. On GitHub, open a Pull Request: **Base:** `main` ← **Compare:** `uat`.
2. Merge using **Create a merge commit**.
3. Pull `main` locally and create a semantic version release tag:
```bash
git checkout main
git pull origin main
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

### Step 7: Create & Push an Emergency Hotfix from `main`
```bash
git checkout main
git pull origin main
git checkout -b hotfix/readme-typo

# Apply the urgent fix, commit, and push
echo "Hotfix applied" >> README.md
git add .
git commit -m "Fix typo on main"
git push -u origin hotfix/readme-typo
```

### Step 8: Merge Hotfix into `main` (GitHub PR)
1. On GitHub, open a Pull Request: **Base:** `main` ← **Compare:** `hotfix/readme-typo`.
2. Merge the Pull Request into `main`.
3. Pull the updated `main` locally (critical before back-merging):
```bash
git checkout main
git pull origin main
git log -n 3 --oneline
```

### Step 9: Back-Merge Hotfix from `main` → `uat` → `sit`
```bash
# 1. Merge updated main into uat and push
git checkout uat
git pull origin uat
git merge main
git push origin uat

# 2. Merge updated uat into sit and push
git checkout sit
git pull origin sit
git merge uat
git push origin sit

# 3. Verify the hotfix commit is now in sit
git log -n 3 --oneline
cat README.md
```

### Step 10: Clean Up the Merged Hotfix Branch
```bash
git checkout main
git branch -d hotfix/readme-typo
git push origin --delete hotfix/readme-typo
git branch -a
```

---

## 3. Quick Verification & Troubleshooting Commands

| Task | Command |
| :--- | :--- |
| Check current branch & modified files | `git status` |
| View all local and remote branches | `git branch -a` |
| View commit tree across all branches | `git log --oneline --graph --decorate --all -n 10` |
| List all release tags | `git tag` |
| Check commits in `main` missing from `sit` | `git log sit..main --oneline` |

### Common Gotchas Learned in Practice
* **Always pull `main` before back-merging:** Running `git merge main` locally only merges your *local* `main`. Always run `git checkout main && git pull origin main` after merging a PR on GitHub.
* **Empty repo `main` error:** On a brand-new empty repository, `git checkout main` fails until you make your first `git commit`.
* **Line ending warning on Windows (`LF will be replaced by CRLF`):** Normal Git behavior on Windows; safe to ignore.