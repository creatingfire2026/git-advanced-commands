# Advanced Git Commands & Repository Optimization Guide

A comprehensive resource for mastering advanced Git workflows, combined with battle-tested strategies for efficiently managing large codebases and accelerating repository maintenance from hours to minutes.

## Table of Contents

1. [Core Advanced Git Commands](#core-advanced-git-commands)
2. [Time-Saving Workflows](#time-saving-workflows)
3. [Repository Optimization Gems](#repository-optimization-gems)
4. [Real-World Scenarios](#real-world-scenarios)
5. [Best Practices](#best-practices)

---

## Core Advanced Git Commands

### 1. `git stash` - Temporarily Save Work

Store uncommitted changes without committing them.

#### Basic Usage
```bash
# Stash uncommitted changes
git stash

# Stash with a descriptive message
git stash push -m "WIP: feature-x incomplete"

# Stash specific files
git stash push path/to/file.js

# Stash including untracked files
git stash -u  # or --include-untracked

# List all stashes
git stash list

# Apply most recent stash
git stash pop

# Apply specific stash without removing it
git stash apply stash@{2}

# View stash contents
git stash show -p stash@{0}

# Delete stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

#### Real-World Scenario
```bash
# You're mid-feature when critical bug needs immediate attention
git stash push -m "feature-auth: 60% complete"
git checkout main
git pull
git checkout -b hotfix/critical-bug
# ... fix bug, test, commit
git checkout main
git merge hotfix/critical-bug
git checkout feature-auth
git stash pop
# Resume work seamlessly
```

#### Pro Tips
- Use descriptive messages for stashes; future-you will thank you
- `git stash show -p` before popping to verify contents
- Stashes are local only; use branches for long-term temporary work

---

### 2. `git cherry-pick` - Apply Specific Commits

Copy commits from one branch to another without merging entire branches.

#### Basic Usage
```bash
# Cherry-pick a single commit
git cherry-pick <commit-hash>

# Cherry-pick multiple commits
git cherry-pick <commit1> <commit2> <commit3>

# Cherry-pick a range of commits
git cherry-pick <from-commit>..<to-commit>  # excludes 'from-commit'
git cherry-pick <from-commit>^..<to-commit>  # includes both

# Continue after resolving conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort

# Skip current commit in cherry-pick sequence
git cherry-pick --skip

# Keep original commit author and date
git cherry-pick -x <commit-hash>  # adds "(cherry picked from...)" footer
```

#### Real-World Scenario
```bash
# Feature branch has 10 commits, but you only need 3 specific ones in hotfix
git checkout hotfix/urgent-patch
git cherry-pick abc1234 def5678 ghi9012

# Add detailed commit message reference
git cherry-pick -x abc1234

# Resolve conflicts if needed
git status
# ... fix conflicts in editor
git add .
git cherry-pick --continue
```

#### Pro Tips
- Cherry-pick creates NEW commits with different hashes (unless pulling fresh)
- Use `git cherry-pick -x` to maintain traceability
- Limit to 3-5 commits max to avoid integration nightmares
- **Caution**: Creates duplicate work; prefer merge when possible

---

### 3. `git revert` - Undo Commits Safely

Create new commits that reverse previous commits' changes (non-destructive).

#### Basic Usage
```bash
# Revert single commit
git revert <commit-hash>

# Revert without opening editor (auto-commit)
git revert --no-edit <commit-hash>

# Revert multiple commits
git revert <commit1>..<commit2>

# Create revert without committing (edit first)
git revert -n <commit-hash>  # or --no-commit

# Revert merge commit (specify parent with -m)
git revert -m 1 <merge-commit-hash>

# Abort revert in progress
git revert --abort

# Continue after resolving conflicts
git revert --continue
```

#### Real-World Scenario
```bash
# Bad commit was pushed to main, need to undo it safely
git log --oneline | head -5
# abc1234 Broke payment validation (LATEST)
# def5678 Added new endpoint
# ...

git revert abc1234
# A new commit is created that undoes abc1234
git push origin main
# Team can see what was reverted and why
```

#### Pro Tips
- Safer than `reset` for shared branches (creates history)
- Always safer for production code
- Include reason for revert in commit message
- `git revert -n` lets you batch multiple reverts before committing

---

### 4. `git reset` - Modify Commit History (Use with Caution)

Move HEAD pointer and modify the staging area and working directory.

#### Basic Usage
```bash
# Soft reset: move HEAD, keep changes staged
git reset --soft HEAD~1

# Mixed reset (default): move HEAD, unstage changes
git reset --mixed HEAD~1
# or simply:
git reset HEAD~1

# Hard reset: move HEAD, discard all changes (DANGEROUS!)
git reset --hard HEAD~1

# Reset to specific commit
git reset --soft <commit-hash>

# Unstage specific file
git reset HEAD path/to/file.js

# Reset working directory to last commit
git reset --hard HEAD

# Move to specific commit on same branch
git reset --hard <commit-hash>

# View reflog to recover "lost" commits
git reflog
git reset --hard <lost-commit-hash>
```

#### Real-World Scenario: Undoing Local Mistakes
```bash
# You committed 3 times locally but want to reorganize
git log --oneline | head -5
# abc1234 Typo fix
# def5678 Feature implementation
# ghi9012 Debug changes
# (older commits...)

# Move back 3 commits, keep changes unstaged
git reset HEAD~3

# Now selectively re-stage and commit properly
git add feature-implementation.js
git commit -m "Add new feature"

git add debug-utilities.js
git commit -m "Add debugging helpers"

git add typo-fix.js
git commit -m "Fix typo"
```

#### Safety Net: Recovering from Hard Reset
```bash
# Oops! You did: git reset --hard HEAD~5
git reflog
# abc1234 HEAD@{0}: reset: moving to HEAD~5
# def5678 HEAD@{1}: commit: last actual commit
# ghi9012 HEAD@{2}: commit: ...

git reset --hard def5678
# Your work is restored!
```

#### Pro Tips
- **NEVER** use `--hard` on shared branches; use `revert` instead
- Always `git reflog` before panicking—Git saves everything for 30 days
- Use `--soft` for local reorganization of commits
- `git reset HEAD path/to/file` is the safe way to unstage

---

## Time-Saving Workflows

### Workflow 1: Emergency Hotfix Pipeline (5 minutes)
```bash
# Scenario: Critical bug in production, main branch has active development

# 1. Save your work instantly
git stash push -m "feature-x: paused for hotfix"

# 2. Create hotfix branch from stable tag
git fetch origin
git checkout -b hotfix/critical-bug origin/main
# OR from last stable release:
git checkout -b hotfix/critical-bug v1.2.3

# 3. Fix, test, commit
# ... edit files
git add .
git commit -m "Fix: critical auth bug #4521"

# 4. Fast merge and tag
git checkout main
git pull origin main
git merge --ff-only hotfix/critical-bug
git tag -a v1.2.4 -m "Hotfix: auth bug"
git push origin main --tags

# 5. Resume your work
git stash pop
```

### Workflow 2: Selective Cherry-Pick Deployment (10 minutes)
```bash
# Scenario: Deploy only specific features from develop to staging

git checkout staging
git pull origin staging

# Identify commits to cherry-pick
git log develop --oneline --graph | head -20

# Cherry-pick only the ones you need
git cherry-pick abc1234 def5678 ghi9012

# Verify and test
git diff staging@{1}  # Compare to previous staging state

# Deploy
git push origin staging
```

### Workflow 3: Squash & Rebase for Clean History (15 minutes)
```bash
# Scenario: Your 8-commit feature branch needs one commit for main

git rebase -i origin/main
# In editor, keep first 'pick', change rest to 'squash'
# Save and edit combined commit message

git push -f origin feature-branch  # Force after rebase

# When merging:
git checkout main
git merge --ff-only feature-branch
```

---

## Repository Optimization Gems

### ⚡ Speed Up Operations by 10-100x

#### 1. **Shallow Clones for Massive Repos**
```bash
# Clone only recent history (huge space/speed gain)
git clone --depth 1 https://github.com/torvalds/linux.git

# Later, fetch more history if needed
git fetch --deepen=50

# Full unshallow
git fetch --unshallow
```
**Impact**: Linux kernel clones from 1GB→100MB. Saves 10 minutes → 1 minute.

---

#### 2. **Sparse Checkout (Clone Only What You Need)**
```bash
# Modern Git 2.25+: Clone only specific directories
git clone --sparse https://github.com/facebook/react.git
git sparse-checkout set packages/react

# Add more directories later
git sparse-checkout add packages/react-dom
```
**Impact**: Monorepo work goes from full 2GB clone → 200MB. Saves massive time.

---

#### 3. **Git Worktree (Multiple Branches Simultaneously)**
```bash
# Instead of stashing and switching, work on multiple branches at once
git worktree add ../react-hotfix hotfix/critical-bug
cd ../react-hotfix
# Work here on hotfix while main worktree stays on feature

git worktree add ../react-main main
cd ../react-main
git checkout main

# List active worktrees
git worktree list

# Clean up
cd ..
git worktree remove react-hotfix
```
**Impact**: Eliminates context-switching delays. Work on hotfix while tests run on feature branch.

---

#### 4. **Partial Clone with Blobless Mode** (Git 2.20+)
```bash
# Clone without downloading large blob contents (lazy-load)
git clone --filter=blob:none https://github.com/huge/repo.git

# Even more aggressive: exclude all except commits
git clone --filter=tree:0 https://github.com/huge/repo.git
```
**Impact**: 2000+ hour builds: reduce initial clone from 30 min → 2-3 minutes.

---

#### 5. **Enable Git Maintenance** (Auto-Optimization)
```bash
# Enable automatic background maintenance tasks
git maintenance start

# Manually run optimization
git maintenance run --auto

# This runs:
# - Incremental repack
# - Reflog expiration
# - Commit graph generation
# - Loose object cleanup
```
**Impact**: Repository operations speed up 20-30% over time.

---

#### 6. **Configure Parallel Git Operations**
```bash
# Speed up by using multiple cores
git config core.preloadIndex true
git config core.precomposeUnicode true

# Global config (all repos):
git config --global core.preloadIndex true

# For large monorepos, increase thread count
git config --global core.packedRefsTimeout 3000
git config --global fetch.parallel 10
git config --global fetch.writeCommitGraph true
```
**Impact**: 15-25% faster operations on multi-core systems.

---

#### 7. **SSH Key Multiplexing** (SSH Reuse)
```bash
# Add to ~/.ssh/config
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
  ControlMaster auto
  ControlPath ~/.ssh/control-%C
  ControlPersist 600

# Subsequent git operations reuse SSH connection
# Impact: 30-40% faster push/pull sequences
```

---

#### 8. **Enable Write Commit Graph** (Faster Traversals)
```bash
# Auto-generate commit graphs for faster operations
git config --global core.commitGraph true
git config --global gc.writeCommitGraph true

# Manual generation:
git commit-graph write --reachable
```
**Impact**: `git log`, `git merge-base` operations 10-20x faster on huge repos.

---

#### 9. **Incremental Fetch (Partial Fetch)**
```bash
# Only fetch what's changed since last fetch
git config --global fetch.prune true
git config --global fetch.pruneTags true
git config --global fetch.fsckObjects true

# Fetch with refetch configuration
git fetch --refetch
```
**Impact**: Regular updates on 2000-hour builds: 30 min → 2-3 minutes.

---

#### 10. **Use Git Bundle for Offline/Sneaker-Net Sharing**
```bash
# Create a portable bundle (like a zip of your repo)
git bundle create repo.bundle main develop feature-x

# Share bundle file (email, USB drive, etc.)

# On another machine, clone from bundle
git clone repo.bundle -b main my-repo

# Or fetch into existing repo
git fetch repo.bundle main:local-main
```
**Impact**: Share massive repos without hosting. Useful for air-gapped environments.

---

### Git Configuration Best Practices

Create `~/.gitconfig` with optimizations:
```ini
[core]
    preloadIndex = true
    precomposeUnicode = true
    commitGraph = true
    
[fetch]
    parallel = 10
    writeCommitGraph = true
    prune = true
    pruneTags = true
    
[gc]
    writeCommitGraph = true
    autodetach = true
    
[diff]
    algorithm = histogram
    
[rebase]
    autoStash = true
    missingCommitsCheck = warn
    
[merge]
    conflictStyle = zdiff3
    
[push]
    default = current
    autoSetupRemote = true

[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    unstage = reset HEAD --
    last = log -1 HEAD
    visual = log --graph --oneline --all
    fixup = commit --amend --no-edit
    sync = !git fetch origin && git rebase origin/main
```

---

## Real-World Scenarios

### Scenario 1: Merging Only Specific Commits from a Feature Branch
```bash
# You have feature-auth with 8 commits, but only want 3 in production

git log feature-auth --oneline
# abc1234 Add MFA support
# def5678 Clean up auth module
# ghi9012 Fix password validation
# jkl3456 Remove debug logs

git checkout main
git cherry-pick abc1234 def5678 ghi9012

# Verify
git log --oneline main -4
git diff origin/main

git push origin main
```

### Scenario 2: Recovering a Deleted Branch
```bash
# Oops! Deleted the feature branch
git reflog
# abc1234 HEAD@{5}: checkout: moving from feature-auth to main
# def5678 HEAD@{6}: commit: Complete auth module

git checkout -b feature-auth abc1234
# Branch restored!
```

### Scenario 3: Fixing Commits Before Pushing
```bash
# Made 3 commits, realized first one has a bug

git log --oneline
# abc1234 Add validation (HAS BUG)
# def5678 Implement feature
# ghi9012 Add tests

git reset --soft HEAD~3
git add feature.js validation-fix.js
git commit -m "Add validation and feature"
git add test.js
git commit -m "Add tests"

# Now history is clean
git push -f origin feature-branch
```

### Scenario 4: Reverting a Bad Merge
```bash
# Merged feature that broke production
git log --oneline | head -3
# abc1234 Merge branch 'bad-feature' (MERGE COMMIT)
# def5678 Previous commit
# ...

# Revert the merge (use -m 1 to specify parent)
git revert -m 1 abc1234

# Or if you need to revert the entire feature
git revert -m 1 --no-edit abc1234
git push origin main
```

---

## Best Practices

### ✅ Do's

- ✅ Use `git stash` for temporary work suspension
- ✅ Use `git cherry-pick` for selective commit application (max 3-5 commits)
- ✅ Use `git revert` for undoing shared/production commits
- ✅ Use `git reset` only for local commits before pushing
- ✅ Write descriptive commit messages
- ✅ Use branches for everything; avoid working on main
- ✅ Enable Git maintenance for auto-optimization
- ✅ Use shallow clones for initial setup of huge repos
- ✅ Review changes before committing: `git diff` before `git add`
- ✅ Use `.gitignore` to exclude build artifacts and secrets

### ❌ Don'ts

- ❌ Don't use `git reset --hard` on shared branches
- ❌ Don't cherry-pick more than 5 commits (merge instead)
- ❌ Don't force push to main/production branches
- ❌ Don't commit secrets, API keys, or credentials
- ❌ Don't mix logical changes in one commit (use staging)
- ❌ Don't skip testing before pushing
- ❌ Don't use `git pull` without understanding merge vs. rebase
- ❌ Don't commit large binary files (use Git LFS)

---

## Advanced Aliases for Maximum Productivity

Add to `~/.gitconfig`:
```ini
[alias]
    # History & Inspection
    visual = log --graph --oneline --all --decorate
    hist = log --oneline --graph --all
    blame-file = blame -w -M -C -C
    
    # Cleanup
    clean-branches = !git branch -vv | grep gone | awk '{print $1}' | xargs git branch -D
    gc-aggressive = gc --aggressive --prune=now
    
    # Stash Management
    stash-list = stash list --pretty=format:'%h %s'
    
    # Cherry-pick
    cherry = cherry -v
    
    # Rebase
    rebase-main = rebase -i origin/main
    
    # Reset Safety
    undo = reset --soft HEAD~1
    unstage = reset HEAD
    
    # Sync Shortcuts
    sync = !git fetch origin && git rebase origin/main
    update = !git fetch origin && git merge origin/main
    
    # Review commits before push
    review = log --oneline origin/main..HEAD
```

---

## Resources & Attribution

### Core Git Concepts
- **Official Git Documentation**: https://git-scm.com/doc
- **Git Book "Pro Git"** by Scott Chacon & Ben Straub: https://git-scm.com/book/en/v2
- **GitHub Guides**: https://guides.github.com/

### Performance & Optimization Gems
- **Scalar** (Microsoft Git optimization tool): https://github.com/microsoft/scalar
- **Git Maintenance** (built-in since 2.30): Official Git documentation
- **Write Commit Graph**: https://git-scm.com/docs/git-commit-graph
- **Partial Clone**: https://github.blog/2020-12-21-get-up-to-speed-with-partial-clone-and-shallow-clone/
- **Git Worktree Deep Dive**: https://github.blog/2015-07-29-git-worktrees/
- **Linux Kernel Git Workflows**: https://www.kernel.org/doc/html/latest/process/applying-patches.html

### Monorepo & Large-Scale Strategies
- **Atlassian Git Tutorials** (Advanced): https://www.atlassian.com/git/tutorials
- **GitHub Large Files (Git LFS)**: https://github.com/git-lfs/git-lfs
- **Sparse Checkout**: https://git-scm.com/docs/git-sparse-checkout

### The "Cherished Gems in the Trenches"

#### 1. **GitHub's Internal Optimization**
GitHub engineering uses partial clones + write-commit-graph for their massive monorepo. See [GitHub's blog on Scalar](https://github.blog/2021-07-14-scaling-monorepo-maintenance/).

#### 2. **Microsoft Scalar**
Built by Microsoft for handling Windows repo (4+ million files). Open-source tool automating optimization. Game-changer for 2000-hour builds.

#### 3. **Google's Bazel + Git Strategies**
Google's monorepo wisdom: use shallow clones + sparse checkout. Reduces 2GB clones to 200MB.

#### 4. **Linux Kernel Development**
Linus Torvalds' workflow: cherry-pick for selective integration, force-push protected branches. Model for large distributed development.

#### 5. **Facebook's Metro Bundler (React Native)**
Uses git bundles for distributing large caches. Parallel fetch configuration reduces setup time dramatically.

#### 6. **Git Maintenance Automation** (Underrated)
Most developers don't know about `git maintenance`. Running this one command regularly adds 20-30% speed improvements. Used in production by GitHub.

#### 7. **SSH ControlMaster Multiplexing**
Rarely discussed but essential: reusing SSH connections reduces push/pull sequences from 3 seconds per operation to 0.3 seconds.

#### 8. **zdiff3 Merge Conflict Strategy**
Modern Git 2.35+ feature. Shows both before and after in conflicts. Reduces merge conflict resolution time by 50%.

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Save work temporarily | `git stash push -m "message"` |
| Apply saved work | `git stash pop` |
| Copy specific commit | `git cherry-pick <hash>` |
| Undo shared commit | `git revert <hash>` |
| Undo local commits | `git reset --soft HEAD~1` |
| Fix last commit | `git commit --amend --no-edit` |
| View commit history | `git log --graph --oneline --all` |
| Recover lost commit | `git reflog` then `git reset --hard <hash>` |
| Clone small initial | `git clone --depth 1 <url>` |
| Work on multiple branches | `git worktree add <path> <branch>` |
| Optimize repository | `git maintenance run --auto` |
| Clean dead branches | `git branch -vv \| grep gone \| awk '{print $1}' \| xargs git branch -D` |

---

## Contributing

Found a powerful Git technique? Have a 2000-hour build optimization? Submit a PR with your discovery and attribution!

---

**Last Updated**: October 2026
**License**: MIT
