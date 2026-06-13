# GIt_GithubActions


# 🔀 Git, GitHub & GitHub Actions From Scratch

> **For Prem** — You use Git and GitHub Actions daily, but this course fills the gaps and deepens your understanding from "I know how to use it" to "I understand how it works under the hood." Perfect for interview prep.

---

## 🧭 Quick Mental Model: Git ↔ GitHub ↔ GitHub Actions

| Concept | Git (CLI) | GitHub (Platform) | GitHub Actions (CI/CD) |
|---|---|---|---|
| What | Version control system | Hosting + collaboration | Automation on top of Git |
| Analogy | The engine | The car | The autopilot |
| Runs on | Your machine | Cloud | Cloud (on push/PR/tag) |
| Config | `.gitconfig` | Repo settings | `.github/workflows/*.yml` |
| Scope | Track code changes | Share & review code | Build, test, deploy code |

---

## 📦 Module 1: Git Fundamentals

### 1.1 What Is Git?

Git is a **distributed version control system** — every developer has the FULL repository history on their machine.

```
Centralized VCS (SVN):          Distributed VCS (Git):
┌────────┐                      ┌────────┐
│ Server │ ◄── Only source       │ Dev A  │ Full repo + history
└───┬────┘                      └────────┘
    │                             ┌────────┐
    ├──▶ Dev A (partial)         │ Dev B  │ Full repo + history
    ├──▶ Dev B (partial)         └────────┘
    └──▶ Dev C (partial)         ┌────────┐
                                  │ Dev C  │ Full repo + history
                                  └────────┘
                                  Works offline! No single point of failure!
```

### 1.2 Git Architecture — The Three Areas

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Working      │    │  Staging     │    │  Repository  │
│  Directory    │    │  Area (Index)│    │  (.git)      │
│              │    │              │    │              │
│  Your actual │    │  Changes     │    │  Committed   │
│  files       │    │  ready to    │    │  history     │
│  (modified)  │    │  commit      │    │  (permanent) │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                   │                   │
  git add ──────────▶  git commit ──────────▶  git push
       │                   │                   │
  (stage changes)    (save snapshot)     (upload to GitHub)
```

### 1.3 Essential Git Commands

```bash
# ─── Setup ──────────────────────────────────
git config --global user.name "Prem"
git config --global user.email "prem@example.com"
git config --global init.defaultBranch main

# ─── Start a Repo ──────────────────────────
git init                        # Create new repo
git clone <url>                 # Clone existing repo

# ─── Daily Workflow ────────────────────────
git status                      # What changed? (most used command!)
git add <file>                  # Stage specific file
git add .                       # Stage ALL changes
git add -p                      # Stage interactively (review each hunk)
git commit -m "message"         # Commit staged changes
git commit -am "message"        # Add + commit tracked files (shortcut)

# ─── Viewing History ────────────────────────
git log                         # Full commit history
git log --oneline               # One line per commit
git log --oneline --graph       # Visual branch graph
git log --oneline --graph --all # All branches
git diff                        # Unstaged changes
git diff --staged               # Staged changes
git show <commit>               # Show specific commit details

# ─── Undoing Things ────────────────────────
git checkout -- <file>          # Discard working dir changes (DANGER!)
git restore <file>              # Same as above (newer syntax)
git restore --staged <file>     # Unstage a file (keep changes)
git reset HEAD <file>           # Same as above (older syntax)
git reset --soft HEAD~1         # Undo last commit, keep changes staged
git reset --mixed HEAD~1        # Undo last commit, keep changes unstaged
git reset --hard HEAD~1         # Undo last commit, DESTROY changes (DANGER!)
git revert <commit>             # Create NEW commit that undoes a commit (safe!)
```

### 1.4 The `.gitignore` File

```text
# .gitignore — Files Git should NOT track
node_modules/
dist/
.env
.env.local
*.log
.DS_Store
coverage/
.vscode/
__pycache__/
*.pyc
```

---

## 📦 Module 2: Branching & Merging

### 2.1 What Are Branches?

A branch is a **movable pointer to a commit**. Creating a branch just creates a new pointer — it's instant and lightweight.

```
main:    A ── B ── C ── D ── E (HEAD)
                    │
feature:           └── F ── G (new branch, diverged at C)

Each commit points to its parent → forms a DAG (Directed Acyclic Graph)
```

### 2.2 Branch Commands

```bash
# Create & switch branches
git branch feature-login          # Create branch
git checkout feature-login       # Switch to it
git checkout -b feature-login    # Create + switch (shortcut)
git switch -c feature-login      # Same, newer syntax

# List branches
git branch                       # Local branches
git branch -a                    # All branches (local + remote)
git branch -r                    # Remote branches only

# Rename & delete
git branch -m old-name new-name  # Rename branch
git branch -d feature-login      # Delete merged branch (safe)
git branch -D feature-login      # Force delete (DANGER!)

# Push branch to remote
git push -u origin feature-login # Push + set upstream tracking
```

### 2.3 Merging

```
Fast-Forward Merge (no divergence):
main: A ── B ── C
                  │
feature:         └── D ── E

git checkout main
git merge feature
Result: A ── B ── C ── D ── E  (pointer just moves forward)

─────────────────────────────────────────────

Three-Way Merge (divergence):
main:    A ── B ── C ── F
               │        │
feature:       └── D ── E

git checkout main
git merge feature
Result:   A ── B ── C ── F ── M (merge commit)
                  │        │
                  └── D ── E ─┘
```

```bash
# Merging
git checkout main
git merge feature-login         # Merge feature into main
git merge --no-ff feature-login # Always create merge commit (no fast-forward)
git merge --abort               # Abort a merge with conflicts
```

### 2.4 Merge Conflicts

```bash
# When both branches modify the same line:
# CONFLICT (content): Merge conflict in server.js

# The file will look like:
<<<<<<< HEAD
const port = 3000;        # Your change (current branch)
=======
const port = 8080;        # Their change (incoming branch)
>>>>>>> feature-login

# Fix: Edit the file, choose the correct code, then:
git add server.js
git commit -m "Resolve merge conflict: use port 3000"
```

### 2.5 Rebase (Alternative to Merge)

```
Rebase replays your commits on top of another branch:

Before rebase:
main:    A ── B ── C ── F
               │
feature:       └── D ── E

git checkout feature
git rebase main

After rebase:
main:    A ── B ── C ── F
                         │
feature:                  └── D' ── E'  (commits are REWRITTEN)

Result: Linear history! No merge commit!
```

```bash
# Rebase
git checkout feature-login
git rebase main                 # Rebase feature on top of main
git rebase --continue           # After resolving conflict
git rebase --abort              # Abort the rebase

# Interactive rebase (edit history!)
git rebase -i HEAD~3            # Edit last 3 commits
# pick   → use commit
# squash → combine with previous commit
# reword → change commit message
# drop   → remove commit
```

### 2.6 Merge vs Rebase

```
┌──────────────────┬──────────────────────────────────────┐
│ Merge             │ Rebase                                │
├──────────────────┼──────────────────────────────────────┤
│ Preserves history │ Rewrites history (linear)            │
│ Creates merge     │ No merge commit                       │
│   commit          │                                       │
│ Safe (non-        │ DANGEROUS on shared branches!          │
│   destructive)    │ (never rebase public/shared branches) │
│ Use for: main ←  │ Use for: your local feature branch    │
│   feature         │   before merging                     │
└──────────────────┴──────────────────────────────────────┘

GOLDEN RULE: Never rebase commits that have been pushed to a shared branch!
```

---

## 📦 Module 3: Remote Repositories & GitHub

### 3.1 Git vs GitHub

```
Git  = The version control tool (runs locally)
GitHub = The platform that hosts Git repos + adds collaboration features

Git = The engine
GitHub = The garage + mechanics + community
```

### 3.2 Remote Commands

```bash
# Clone a repo
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git          # SSH (preferred)

# Remote management
git remote -v                                    # List remotes
git remote add origin https://github.com/user/repo.git
git remote set-url origin git@github.com:user/repo.git  # Switch to SSH

# Sync with remote
git fetch origin                  # Download remote changes (don't merge)
git pull origin main              # Fetch + merge (git pull = git fetch + git merge)
git pull --rebase origin main    # Fetch + rebase (cleaner history)
git push origin main             # Upload local commits
git push -u origin main          # Push + set upstream (first time only)
```

### 3.3 Fetch vs Pull

```
git fetch:  Downloads remote changes → DOES NOT modify your working directory
            Safe! You can review before merging.

git pull:   Downloads + merges → Modifies your working directory
            = git fetch + git merge
            Could create merge conflicts!

Best practice: git fetch first, review, then git merge or git rebase
```

### 3.4 GitHub-Specific Features

```
┌──────────────────────────────────────────────────────┐
│           GitHub Features Beyond Git                   │
├──────────────────┬───────────────────────────────────┤
│ Pull Requests    │ Code review + discussion + merge   │
│ Issues           │ Bug tracking + feature requests    │
│ Actions          │ CI/CD automation                   │
│ Projects         │ Kanban boards for task management   │
│ Codespaces       │ Cloud dev environment              │
│ Releases         │ Versioned distribution              │
│ Branch Protection│ Require reviews, status checks     │
│ CODEOWNERS      │ Auto-assign reviewers               │
│ Discussions     │ Community Q&A                       │
│ GitHub Pages    │ Static site hosting                 │
└──────────────────┴───────────────────────────────────┘
```

---

## 📦 Module 4: The Pull Request Workflow

### 4.1 Complete PR Workflow

```
1. Create a branch
   git checkout -b feature/add-health-endpoint

2. Make changes & commit
   git add .
   git commit -m "Add /health endpoint"

3. Push to GitHub
   git push -u origin feature/add-health-endpoint

4. Open Pull Request on GitHub
   → Base: main  ←  Compare: feature/add-health-endpoint
   → Add title, description, reviewers

5. Code Review
   → Team reviews, comments, requests changes
   → You push more commits to address feedback
   → CI/CD runs automatically (GitHub Actions)

6. Merge
   → Squash and merge (preferred for clean history)
   → Or merge commit (preserves full history)
   → Or rebase and merge (linear history)

7. Clean up
   git checkout main
   git pull origin main
   git branch -d feature/add-health-endpoint
   git push origin --delete feature/add-health-endpoint
```

### 4.2 Branch Protection Rules (Must-Know for Interviews)

```
┌──────────────────────────────────────────────────────┐
│        Branch Protection Rules (Settings → Branches) │
├──────────────────────────────────────────────────────┤
│ ✅ Require pull request before merging               │
│    → Require 1-2 approvals                            │
│ ✅ Require status checks to pass                     │
│    → CI must pass (tests, lint, build)                │
│ ✅ Require conversation resolution                    │
│    → All comments must be resolved                    │
│ ✅ Require signed commits                             │
│ ✅ Do not allow force pushes                          │
│ ✅ Do not allow deletions                             │
└──────────────────────────────────────────────────────┘
```

### 4.3 Merge Strategies on GitHub

```
1. Create a merge commit
   main: A ── B ── C ── M (merge commit)
                │     │
   feature:     └── D ── E ─┘
   Preserves full history. Can be noisy.

2. Squash and merge (RECOMMENDED)
   main: A ── B ── C ── S (single squashed commit)
   All feature commits become ONE commit. Clean history!

3. Rebase and merge
   main: A ── B ── C ── D' ── E'
   Commits are rebased. Linear history. No merge commit.
```

---

## 📦 Module 5: Git Advanced — Stash, Cherry-Pick, Bisect

### 5.1 Git Stash (Save Work In Progress)

```bash
# Save current changes without committing
git stash                       # Stash all changes
git stash -m "WIP: auth feature" # Stash with message
git stash list                  # List all stashes
git stash pop                   # Apply latest stash + remove it
git stash apply                 # Apply latest stash + keep it
git stash drop stash@{0}        # Delete specific stash
git stash clear                 # Delete all stashes

# Use case: Working on feature, need to hotfix main
git stash                       # Save feature work
git checkout main
git checkout -b hotfix/bug-123
# ... fix bug, commit, push, merge ...
git checkout feature
git stash pop                   # Resume feature work
```

### 5.2 Git Cherry-Pick (Pick Specific Commits)

```bash
# Apply a specific commit from another branch
git cherry-pick abc123          # Apply commit abc123 to current branch

# Use case: Hotfix needs to go to both main and release branch
git checkout release-v2
git cherry-pick abc123          # Pick the hotfix commit
```

### 5.3 Git Bisect (Find the Bug-Introducing Commit)

```bash
# Binary search to find which commit introduced a bug
git bisect start
git bisect bad                  # Current commit is bad
git bisect good v1.0            # v1.0 was good
# Git checks out middle commit
# You test: good or bad?
git bisect good                 # This commit is fine
git bisect bad                  # This commit has the bug
# ... repeat until found ...
git bisect reset                # Done! Go back to original HEAD
```

### 5.4 Git Tags (Version Markers)

```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (RECOMMENDED — includes message + author)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push tags
git push origin v1.0.0         # Push specific tag
git push origin --tags          # Push all tags

# List tags
git tag
git tag -l "v1.*"              # Filter tags

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0
```

---

## 📦 Module 6: Git Workflow Strategies

### 6.1 Git Flow

```
┌──────────────────────────────────────────────────────┐
│ Git Flow — Structured, enterprise-friendly           │
│                                                       │
│ main      ── Production-ready code only               │
│ develop   ── Integration branch                        │
│ feature/* ── New features (from develop)               │
│ release/* ── Release prep (from develop)                │
│ hotfix/*  ── Urgent fixes (from main)                  │
│                                                       │
│ main:    ────────────────────────── v1.0 ──── v1.0.1 │
│                │                        │       │      │
│ develop:  ────┴─── D ── E ── F ───────┘       │      │
│                │                              │      │
│ feature:  ────┴── G ── H ──┘                │      │
│                                              │      │
│ hotfix:                              ──── I ─┘      │
└──────────────────────────────────────────────────────┘
```

### 6.2 GitHub Flow (Simpler — What Most Teams Use)

```
┌──────────────────────────────────────────────────────┐
│ GitHub Flow — Simple, modern, PR-based               │
│                                                       │
│ main  ── Always deployable ────────────────────────  │
│          │          │          │                      │
│ feature-A ──┘      │          │                      │
│ feature-B ─────────┘          │                      │
│ hotfix-C ────────────────────┘                      │
│                                                       │
│ Rules:                                                │
│ 1. main is always deployable                          │
│ 2. Create branch from main                            │
│ 3. Open PR when ready                                 │
│ 4. Merge after review + CI passes                    │
│ 5. Deploy immediately after merge                    │
└──────────────────────────────────────────────────────┘
```

### 6.3 Trunk-Based Development

```
┌──────────────────────────────────────────────────────┐
│ Trunk-Based — CI/CD heavy, small changes             │
│                                                       │
│ main ── A ── B ── C ── D ── E ── F ── G ── H ──   │
│                                                       │
│ Very short-lived branches (< 1 day)                   │
│ Feature flags instead of long-lived branches          │
│ Continuous integration is MANDATORY                    │
│ Used by: Google, Facebook, Netflix                   │
└──────────────────────────────────────────────────────┘
```

---

## 📦 Module 7: GitHub Actions — Complete Guide

### 7.1 What Is GitHub Actions?

GitHub Actions is **CI/CD built into GitHub** — automatically runs workflows on push, PR, schedule, etc.

```
┌──────────────────────────────────────────────────────┐
│           GitHub Actions Architecture                  │
│                                                        │
│  Push/PR/Tag ──▶ GitHub ──▶ Trigger Workflow          │
│                                  │                     │
│                                  ▼                     │
│                         ┌──────────────┐               │
│                         │ GitHub Runner │               │
│                         │ (ubuntu-latest│               │
│                         │  or self-host)│               │
│                         │              │               │
│                         │ 1. Checkout  │               │
│                         │ 2. Install   │               │
│                         │ 3. Test      │               │
│                         │ 4. Build     │               │
│                         │ 5. Deploy    │               │
│                         └──────────────┘               │
└──────────────────────────────────────────────────────┘
```

### 7.2 Workflow File Structure

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy                    # Workflow name

on:                                       # When to trigger
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:                                      # Global environment variables
  APP_NAME: my-api
  REGISTRY: 123456789012.dkr.ecr.us-east-1.amazonaws.com

jobs:                                     # Groups of steps
  build:                                  # Job name
    runs-on: ubuntu-latest               # Runner OS
    permissions:                          # Security permissions
      id-token: write
      contents: read

    steps:                                # Individual actions
      - name: Checkout code
        uses: actions/checkout@v4        # Reusable action from marketplace

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci                      # Shell command

      - name: Run tests
        run: npm test

      - name: Build Docker image
        run: docker build -t $REGISTRY/$APP_NAME:$GITHUB_SHA .

  deploy:
    needs: build                          # Wait for build job to complete
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # Only deploy on main

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: us-east-1

      - name: Push to ECR
        run: |
          docker push $REGISTRY/$APP_NAME:$GITHUB_SHA

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster my-cluster \
            --service my-api \
            --force-new-deployment
```

### 7.3 Trigger Events (`on:`)

```yaml
# Push to branch
on:
  push:
    branches: [main, develop]
    paths: ['src/**', 'package.json']    # Only if these files changed

# Pull request
on:
  pull_request:
    types: [opened, synchronize, reopened]

# Manual trigger
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

# Schedule (cron)
on:
  schedule:
    - cron: '0 8 * * 1-5'               # Weekdays 8 AM UTC

# Tag push
on:
  push:
    tags:
      - 'v*'                             # On any tag starting with 'v'

# Multiple events
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
```

### 7.4 GitHub Context Variables

```yaml
steps:
  - name: Print context
    run: |
      echo "Branch: ${{ github.ref_name }}"
      echo "SHA: ${{ github.sha }}"
      echo "Event: ${{ github.event_name }}"
      echo "Actor: ${{ github.actor }}"
      echo "Repo: ${{ github.repository }}"
      echo "Run ID: ${{ github.run_id }}"
      echo "Workflow: ${{ github.workflow }}"

  - name: Conditional step
    if: github.ref == 'refs/heads/main'
    run: echo "This only runs on main!"

  - name: On PR only
    if: github.event_name == 'pull_request'
    run: echo "This only runs on PRs!"

  - name: Use secrets
    run: |
      echo "DB password: ${{ secrets.DB_PASSWORD }}"
    env:
      API_KEY: ${{ secrets.API_KEY }}
```

### 7.5 Artifacts & Caching

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache node modules
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - run: npm ci
      - run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/

      - name: Deploy
        run: ./deploy.sh
```

### 7.6 Matrix Strategy (Test Multiple Versions)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
        os: [ubuntu-latest, macos-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
    # Runs 6 times: (node 16, 18, 20) × (ubuntu, macos)
```

### 7.7 Environment & Protection Rules

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment: production              # Requires approval in GitHub settings!
    steps:
      - run: ./deploy-prod.sh

  # In GitHub repo settings → Environments → production:
  # ✅ Required reviewers: Add team members
  # ✅ Wait timer: 5 minutes
  # ✅ Branch: main only
  # ✅ Secrets: Production-only secrets
```

### 7.8 Reusable Workflows

```yaml
# .github/workflows/deploy.yml (reusable)
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      aws-role:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.aws-role }}
      - run: ./deploy.sh --env ${{ inputs.environment }}
```

```yaml
# .github/workflows/main.yml (caller)
jobs:
  deploy-staging:
    uses: ./.github/workflows/deploy.yml
    with:
      environment: staging
    secrets:
      aws-role: arn:aws:iam::123456789012:role/StagingRole

  deploy-production:
    needs: deploy-staging
    uses: ./.github/workflows/deploy.yml
    with:
      environment: production
    secrets:
      aws-role: arn:aws:iam::123456789012:role/ProductionRole
```

---

## 📦 Module 8: GitHub Actions — ECS Deployment (Production)

### 8.1 Complete ECS Deployment Workflow

```yaml
# .github/workflows/ecs-deploy.yml
name: ECS Deploy

on:
  push:
    branches: [main]

env:
  AWS_REGION: us-east-1
  ECR_REGISTRY: 123456789012.dkr.ecr.us-east-1.amazonaws.com
  IMAGE_NAME: my-api
  ECS_CLUSTER: my-cluster
  ECS_SERVICE: my-api

permissions:
  id-token: write
  contents: read

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsECSRole
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build & Push Docker Image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$IMAGE_NAME:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Update ECS Task Definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: my-api-task.json
          container-name: my-api
          image: ${{ steps.build-image.outputs.image }}

      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

### 8.2 OIDC Authentication (Best Practice — No Long-Lived Keys!)

```
┌──────────────────────────────────────────────────────┐
│     GitHub Actions → AWS (OIDC, No Access Keys!)      │
│                                                        │
│  1. Create IAM Identity Provider in AWS               │
│     (Trust GitHub's OIDC provider)                     │
│                                                        │
│  2. Create IAM Role with trust policy                  │
│     (Trust specific GitHub repo only)                  │
│                                                        │
│  3. In workflow, use:                                  │
│     aws-actions/configure-aws-credentials@v4           │
│     with:                                              │
│       role-to-assume: arn:aws:iam::...:role/...       │
│                                                        │
│  Result: Short-lived tokens, no leaked access keys!    │
└──────────────────────────────────────────────────────┘
```

---

## 📦 Module 9: Git & GitHub Interview Scenarios

### 9.1 Common Git Scenarios

```bash
# "I committed to the wrong branch!"
git stash                          # Save changes
git checkout correct-branch        # Switch to correct branch
git stash pop                      # Apply changes

# "I need to undo my last commit but keep the changes"
git reset --soft HEAD~1

# "I pushed sensitive data to GitHub!"
# 1. Remove the file
git rm --cached secrets.yml
git commit -m "Remove secrets"
# 2. Use BFG Repo Cleaner to purge from history
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch secrets.yml' \
  --prune-empty --tag-name-filter cat -- --all
git push origin --force --all
# 3. ROTATE ALL EXPOSED CREDENTIALS IMMEDIATELY!

# "I need to combine my last 3 commits into one"
git rebase -i HEAD~3
# Change "pick" to "squash" for commits 2 and 3

# "My branch is behind main and has conflicts"
git checkout feature-branch
git rebase origin/main
# Resolve conflicts, then:
git rebase --continue
git push origin feature-branch --force-with-lease
```

---

## 🎯 Interview Questions — Git, GitHub & GitHub Actions

### Git Beginner
1. What is the difference between `git pull` and `git fetch`?
2. What is the staging area in Git?
3. What is the difference between `git reset` and `git revert`?
4. What is a merge conflict and how do you resolve it?
5. What is the difference between `git merge` and `git rebase`?

### Git Advanced
6. Explain Git's internal data model (blobs, trees, commits).
7. What is the reflog and when would you use it?
8. How would you recover a lost commit after a hard reset?
9. Explain the difference between `git reset --soft`, `--mixed`, and `--hard`.
10. What is `git cherry-pick` and when would you use it?

### GitHub & GitHub Actions
11. What are branch protection rules and why are they important?
12. Explain the GitHub Flow workflow.
13. How do you securely authenticate GitHub Actions with AWS (OIDC)?
14. What is the difference between a GitHub Actions `job` and a `step`?
15. How would you implement a manual approval step in GitHub Actions?

### Scenario-Based
16. A developer accidentally force-pushed to main and lost commits. How do you recover?
17. Design a CI/CD pipeline for a microservice that deploys to ECS.
18. Your GitHub Actions workflow is slow. How do you optimize it?
19. How would you implement a blue/green deployment using GitHub Actions + ECS?
20. A secret was accidentally pushed to GitHub. What's your incident response?

---

## 📚 Recommended Learning Path

| Week | Focus | What to Do |
|------|-------|-----------|
| **1** | Git Basics | Init, commit, branch, merge, resolve conflicts (Modules 1-2) |
| **2** | Git Advanced | Rebase, stash, cherry-pick, tags, workflows (Modules 3-6) |
| **3** | GitHub Actions | Workflows, triggers, secrets, caching, matrix (Module 7) |
| **4** | Production CI/CD | ECS deployment, OIDC, reusable workflows (Modules 7-8) |
| **5** | Practice | Interview questions, real-world scenarios (Module 9) |

---

## 🔑 The One-Liner That Ties It All Together

> **Git tracks changes, GitHub enables collaboration, GitHub Actions automates everything.** They're not three separate tools — they're one pipeline: code → review → deploy.

---

*Built with ❤️ by Cline SR for Prem — Git, GitHub & GitHub Actions from scratch*
