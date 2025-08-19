# **Git Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Configuration](#configuration)
3. [Repository Management](#repository-management)
4. [Staging & Committing](#staging--committing)
5. [Branching](#branching)
6. [Merging & Rebasing](#merging--rebasing)
7. [Stashing](#stashing)
8. [Remote Repositories](#remote-repositories)
9. [Undo & Recovery](#undo--recovery)
10. [Logs & History](#logs--history)
11. [Tags](#tags)
12. [Best Practices](#best-practices)

---

## **Overview**

* Git is a **distributed version control system**.
* Tracks changes in files and supports **branching, merging, and collaboration**.
* Main concepts: **commit, branch, merge, stash, remote**.

---

## **Configuration**

```bash
# Set username and email
git config --global user.name "John Doe"
git config --global user.email "john@example.com"

# View configuration
git config --list

# Set default editor
git config --global core.editor "vim"
```

---

## **Repository Management**

```bash
# Initialize new repository
git init

# Clone existing repository
git clone <repo-url>

# Check repository status
git status

# Show repository info
git remote -v
```

---

## **Staging & Committing**

```bash
# Stage files
git add <file>
git add .         # Stage all changes

# Commit staged changes
git commit -m "Commit message"

# Commit with all changes
git commit -am "Commit message"

# Amend last commit
git commit --amend -m "Updated commit message"
```

---

## **Branching**

```bash
# List branches
git branch

# Create new branch
git branch feature-branch

# Switch branch
git checkout feature-branch
git switch feature-branch

# Create & switch in one step
git checkout -b feature-branch
git switch -c feature-branch

# Delete branch
git branch -d feature-branch
git branch -D feature-branch
```

---

## **Merging & Rebasing**

```bash
# Merge branch into current branch
git merge feature-branch

# Rebase current branch onto another
git rebase main

# Abort rebase
git rebase --abort

# Continue rebase after conflict resolution
git rebase --continue

# Interactive rebase (edit, squash, reorder)
git rebase -i HEAD~3
```

---

## **Stashing**

```bash
# Save changes temporarily
git stash

# Save with message
git stash save "WIP changes"

# List stashes
git stash list

# Apply last stash
git stash apply

# Apply specific stash
git stash apply stash@{1}

# Drop stash
git stash drop stash@{1}

# Pop stash (apply + remove)
git stash pop
```

---

## **Remote Repositories**

```bash
# Add remote
git remote add origin <repo-url>

# Fetch changes
git fetch origin

# Pull changes (fetch + merge)
git pull origin main

# Push changes
git push origin main
git push -u origin feature-branch

# Remove remote
git remote remove origin
```

---

## **Undo & Recovery**

```bash
# Unstage file
git reset HEAD <file>

# Discard changes in working directory
git checkout -- <file>

# Reset branch to specific commit
git reset --hard <commit-hash>

# Revert commit without rewriting history
git revert <commit-hash>
```

---

## **Logs & History**

```bash
# Show commit history
git log

# One-line history
git log --oneline

# Graph view
git log --graph --oneline --all

# Show changes
git diff

# Show staged changes
git diff --staged
```

---

## **Tags**

```bash
# Create lightweight tag
git tag v1.0

# Create annotated tag
git tag -a v1.0 -m "Version 1.0"

# List tags
git tag

# Push tags
git push origin v1.0
git push origin --tags
```

---

## **Best Practices**

1. Commit frequently with meaningful messages.
2. Use **branches** for features and fixes.
3. Avoid rebasing public branches.
4. Use `.gitignore` for unnecessary files.
5. Pull before pushing to avoid conflicts.
6. Regularly clean old branches and stashes.
