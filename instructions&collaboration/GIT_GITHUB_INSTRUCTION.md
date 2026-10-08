# 🐙 Git & GitHub Collaboration Guidelines

> **Mandatory Team Policy:** Every member of the Xerox UAV Team across all technical sub-teams (AI, Software, Hardware, PCB Design, Control, and Control Boards) must strictly follow these Git and GitHub standards. Clean version control prevents lost work, eliminates regressions, and ensures high engineering rigor across our entire organization.

---

## 📖 Table of Contents
1. [Initial Setup & Workstation Configuration](#-initial-setup--workstation-configuration)
2. [Branching Strategy: GitHub Flow for Xerox UAV](#-branching-strategy-github-flow-for-xerox-uav)
3. [Conventional Commits & Clean History](#-conventional-commits--clean-history)
4. [Step-by-Step Daily Git Workflow](#-step-by-step-daily-git-workflow)
5. [GitHub Collaboration & Pull Request (PR) Lifecycle](#-github-collaboration--pull-request-pr-lifecycle)
6. [Resolving Merge Conflicts Confidently](#-resolving-merge-conflicts-confidently)
7. [GitHub Issues & Project Management](#-github-issues--project-management)
8. [What NEVER to Commit (.gitignore Hygiene)](#-what-never-to-commit-gitignore-hygiene)

---

## ⚙️ Initial Setup & Workstation Configuration

Before contributing to any Xerox UAV repository, configure Git on your local machine:

### 1. Identify Yourself
Ensure your local Git author name and email match your verified GitHub account:
```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

### 2. Configure SSH Authentication
Always clone and push using SSH rather than HTTPS to avoid repeated password prompts and expired tokens:
```bash
# Generate a modern Ed25519 key (if you haven't already)
ssh-keygen -t ed25519 -C "your_email@example.com"

# View your public key and add it to GitHub (Settings -> SSH and GPG keys)
cat ~/.ssh/id_ed25519.pub
```

### 3. Recommended Global Git Settings
Set standard defaults to keep Git history clean:
```bash
# Use rebase by default when pulling to avoid redundant merge bubble commits
git config --global pull.rebase true

# Set default initial branch name to main
git config --global init.defaultBranch main

# Standardize line endings (Linux/macOS: input; Windows: true)
git config --global core.autocrlf input
```

---

## 🌿 Branching Strategy: GitHub Flow for Xerox UAV

Direct pushes to `main` (or `master`) are **strictly prohibited** and blocked via GitHub branch protection rules. All development happens on short-lived feature branches.

```
       feat/sw-telemetry-parser  ●───●───●
                                /         \ (Pull Request & Review)
main  ●────────────────────────●───────────●────────────────────────►
```

### Branch Naming Conventions
Use descriptive, hyphen-separated branch names prefixed with the category:

| Prefix | Use Case | Example |
| :--- | :--- | :--- |
| `feat/` | New features, nodes, or algorithms | `feat/ai-yolo-tensorrt-node` |
| `fix/` | Bug fixes for existing functionality | `fix/telemetry-checksum-overflow` |
| `refactor/` | Code cleanup with no functional change | `refactor/decompose-state-machine` |
| `docs/` | Documentation, tutorials, or roadmaps | `docs/update-pcb-bringup-guide` |
| `test/` | Adding or updating unit/simulation tests | `test/add-sitl-ekf-tests` |
| `hw/` | PCB schematics, board revisions, or CAD | `hw/v2.1-pdb-current-sensor` |

---

## ✍️ Conventional Commits & Clean History

Every commit in Xerox UAV must tell a clear story. We follow the **[Conventional Commits](https://www.conventionalcommits.org/)** specification.

### Commit Format Structure
```text
<type>(<scope>): <short summary in imperative mood>

[optional body explaining WHY, design trade-offs, and context]

[optional footer: references GitHub issue, e.g., Closes #42]
```

### Commit Types
- **`feat`**: A new feature (e.g., `feat(vision): add stereo depth disparity node`)
- **`fix`**: A bug fix (e.g., `fix(control): resolve yaw rate integral windup`)
- **`refactor`**: Code restructuring without bug fix or new feature
- **`perf`**: Performance improvement (e.g., `perf(inference): optimize CUDA stream synchronization`)
- **`test`**: Adding or refactoring tests
- **`docs`**: Documentation changes only
- **`ci`**: GitHub Actions workflows or automation scripts
- **`chore`**: Maintenance, updating dependencies, or `.gitignore`

### 💡 Golden Rules for Great Commits
1. **Atomic Commits:** Make one logical change per commit. Do not combine a bug fix, a refactor, and a new feature in a single commit.
2. **Imperative Mood:** Write the summary as if giving a command: `"add telemetry filter"`, NOT `"added telemetry filter"` or `"adds telemetry filter"`.
3. **Explain the WHY:** If a change is non-trivial, use the commit body to explain *why* the change was made, not just *what* lines were changed.

---

## 🔄 Step-by-Step Daily Git Workflow

Follow this standard procedure every time you work on a task:

```bash
# 1. Start from latest main
git checkout main
git pull origin main

# 2. Create your feature branch
git checkout -b feat/control-attitude-mpc

# 3. Check your work incrementally
git status
git diff

# 4. Stage specific files (never use 'git add .' blindly!)
git add src/control/mpc_controller.py tests/test_mpc.py

# 5. Commit with a conventional commit message
git commit -m "feat(control): implement quaternion-based MPC attitude controller"

# 6. Rebase on main if teammate changes were merged while you worked
git fetch origin main
git rebase origin/main

# 7. Push your branch to GitHub
git push -u origin feat/control-attitude-mpc
```

---

## 🤝 GitHub Collaboration & Pull Request (PR) Lifecycle

### 1. Opening a Pull Request
Once pushed, open a Pull Request against the `main` branch on GitHub:
- **Title:** Format identical to a conventional commit (e.g., `feat(control): implement quaternion-based MPC attitude controller`).
- **Description:** Provide:
  1. **Summary:** What was implemented and why.
  2. **Linked Issues:** Link the issue being resolved (`Closes #18` or `Fixes #25`).
  3. **Verification Evidence:** Include unit test results, SITL simulation screenshots/recordings, or hardware bench test logs.

### 2. The Code Review Process
- Every PR requires at least **one approving review** from a domain lead before merging.
- **Reviewers:** Be respectful, specific, and explain *why* a suggestion is needed.
- **Authors:** Treat review comments constructively. If changes are requested:
  ```bash
  # Make fixes locally, commit them, and push
  git add <fixed-files>
  git commit -m "fix(control): clamp max attitude angle to 35 degrees"
  git push origin feat/control-attitude-mpc
  ```
  GitHub will automatically update the Pull Request.

### 3. Merging Guidelines
- All CI checks (linting, tests, static typing) must pass green.
- Resolve all conversation threads.
- **Merge Strategy:** Prefer **"Squash and merge"** for clean single-feature history, or **"Rebase and merge"** for multi-commit features with clean atomic commits.
- **Delete Branch:** Delete your feature branch on GitHub immediately after merging.

---

## 💥 Resolving Merge Conflicts Confidently

Conflicts happen when two developers edit the same lines of code. Resolve them using **rebase**:

```bash
# 1. Fetch latest changes from main
git fetch origin main

# 2. Rebase your feature branch on top of main
git rebase origin/main
```

If a conflict occurs, Git pauses and marks the conflicting files:
1. Open the conflicting files. Look for conflict markers:
   ```text
   <<<<<<< HEAD (Current branch on main)
   target_speed_mps = 12.0
   =======
   target_speed_mps = 15.0  (Your change)
   >>>>>>> feat/control-attitude-mpc
   ```
2. Manually edit the file to keep the desired code and delete the markers (`<<<<<<<`, `=======`, `>>>>>>>`).
3. Stage the resolved file:
   ```bash
   git add <resolved-file>
   ```
4. Continue the rebase:
   ```bash
   git rebase --continue
   ```
5. If you make a mistake and want to start over:
   ```bash
   git rebase --abort
   ```
6. Force-push your rebased branch safely:
   ```bash
   git push --force-with-lease origin feat/control-attitude-mpc
   ```

---

## 📌 GitHub Issues & Project Management

All tasks in Xerox UAV originate from GitHub Issues:

1. **Before Writing Code:** Always check existing issues or create a new one to discuss with the team before investing time in large features.
2. **Issue Labels:**
   - `good first issue`: Ideal for new members onboarding.
   - `bug`: Something is broken or malfunctioning.
   - `enhancement`: New capability or performance upgrade.
   - `hardware`: Mechanical, PCB, or electrical issues.
   - `priority: high` / `priority: critical`: Blocking flight testing.
3. **Milestones:** Issues are grouped into competition deadlines or flight test milestones.

---

## 🚫 What NEVER to Commit (.gitignore Hygiene)

Accidental commits of secrets, large weights, or build artifacts pollute the git history permanently:

| Category | Forbidden Files | Recommended Handling |
| :--- | :--- | :--- |
| **Secrets & Credentials** | `.env`, API keys, SSH private keys, AWS tokens | Store in local `.env` (listed in `.gitignore`) or GitHub Secrets |
| **Heavy Datasets & Weights** | `.pt`, `.onnx`, `.engine`, `.mp4`, `.bag`, `.csv` (>50MB) | Track via **DVC (Data Version Control)** or centralized team storage |
| **Virtual Environments** | `.venv/`, `env/`, `__pycache__/`, `.pytest_cache/` | Ensure `.gitignore` covers Python environments |
| **Build & Tool Artifacts** | `build/`, `install/`, `log/`, `target/`, `.ruff_cache/` | Never commit compiler or linter caches |
| **Hardware Manufacturing** | Temporary Gerber logs, CAD cache files | Commit only final release outputs or PDFs |
