---
title: "Refining My Git Worktree Workflow: From Chaos to Clarity"
date: 2025-12-17
tags: ["git", "worktree", "productivity", "development", "workflow"]
draft: false
summary: "How I restructured my git worktree setup from feature-based to mindset-based organization"
---

## The Context Switching Problem

Context switching has always been a part of development work. Urgent production bugs need immediate attention, teammates need code reviews, and you often need to check how something was implemented in another part of the codebase. Each of these interruptions meant stashing changes, switching branches, and then trying to get back to where you were.

I thought git worktrees would solve this. They did, but only after I stopped using them wrong.

---

## How I Was Doing It Wrong

Initially, I set up a bare repo and created worktrees inside my bare repo based on the feature I was working on. Like below:

```plaintext
my-project/
├── .bare/                  ← Bare repository (no working files)
│   ├── objects/            ← All commits and history
│   ├── refs/               ← All branches and tags
│   ├── config              ← Repository settings
│   ├── feature/abc         ← Worktree for feature abc
│   ├── fix/xyz             ← Worktree for fix xyz
```

Because of this approach:

- My root bare repo got cluttered with old worktree and their directories. I couldn't clear those up in time as some of those can contain important files which I might need later.
- It was difficult for editor to understand the structure of the repo and navigate through it. Git UI was not able to handle this structure well.
- I had to create a worktree from root and open worktree on new window of that editor. This was not efficient and caused confusion.
- I have this script which runs everytime I create a worktree which does dependency install as clean setup and it slowly started taking space in my system.

---

## Perspective Shift

One fine day, I came upon Matklad's blog post on [Git Worktrees](https://matklad.github.io/2024/07/25/git-worktrees.html). I got understanding that I need to have a perspective shift in how I was thinking about worktrees. 

Instead of creating feature specific directory/worktree, I should create mindset based worktree. The idea is to organize worktrees by the type of work you're doing, not by what you're building. So when you need to do something, you already know which worktree to use.

So I created below worktrees:

| Worktree | Purpose |
|----------|---------|
| **develop** | For quickly comparing changes, checking out new branches, and testing small fixes |
| **work** | This is the place I am actively building features and working on tasks |
| **review** | For preparing code for reviews, cleaning up commits, and ensuring everything is polished |
| **scratch** | A playground for experimenting with ideas, testing out new libraries, or prototyping features without affecting my main work |

Now my structure looks like below:

```plaintext
my-project/
├── .bare/                  ← Bare repository (no working files)
│   ├── objects/            ← All commits and history
│   ├── refs/               ← All branches and tags
│   ├── develop/            ← Worktree for quick experiments
│   ├── work/               ← Worktree for active development
│   ├── review/             ← Worktree for code reviews
│   └── scratch/            ← Worktree for prototyping
```

I used `DETACHED HEAD` trick to create these worktrees without being tied to any specific branch. So I never get error of "worktree already exists for this branch".

---

## Scripts to Simplify My Workflow

Since I now have permanent folders, I don't need scripts to create directories anymore. Instead, I use scripts to reset them to the state I need. This solves the dependency hell—since the node_modules folder stays put, switching tasks is nearly instant because I rarely need a full clean install.

I rely on two main scripts:

### 1. start-feature.sh

I use this exclusively in my work directory. It fetches the latest changes, handles the branch creation logic, and ensures my dependencies are up to date.

```bash
#!/bin/bash
# Usage: ./start-feature.sh feature/new-login
BRANCH_NAME=$1

cd work || exit
git fetch origin
# Reset 'work' to origin/develop and create the new branch
git switch -C "$BRANCH_NAME" origin/develop
pnpm install
echo "Ready to code in 'work' on $BRANCH_NAME"
```

### 2. sync-worktree.sh

I use this for my review and scratch folders. It safely "teleports" these folders to any branch or PR I need to inspect, using Detached HEAD to avoid conflicts with my main work.

```bash
#!/bin/bash
# Usage: ./sync-worktree.sh review pull/320/head
FOLDER=$1
TARGET=$2

cd "$FOLDER" || exit
git fetch origin
# Switch using detached HEAD so we don't lock the branch name
git checkout --detach "$TARGET"

# Re-link .env files if they don't exist
ln -sf ../shared-config/core.env .env
pnpm install
```

---

## The Scenarios that I Encounter on Daily Basis and How I Handle Them

Here is how this setup handles real-world chaos without breaking my flow.

### Scenario 1: The Urgent Production Bug

**Old Way:** Stash changes in my current branch (hoping I don't forget them), switch to main, fix bug, commit, switch back, unstash, resolve weird conflicts.

**New Way:**

- I leave my work folder exactly as it is (messy and uncommitted).
- I open a terminal in scratch.
- Run `./sync-worktree.sh scratch origin/main`.
- Create a hotfix branch, push the fix, and close the folder.
- My "coding brain" never left the context of my main feature.

### Scenario 2: The "Can you review this?" Request

**Old Way:** "Sure, give me 10 mins to finish what I'm doing so I can switch branches."

**New Way:**

- Run `./sync-worktree.sh review pr-branch-name`.
- I open the review folder in a second VS Code window.
- I can now verify the PR side-by-side with my own code in the work window.

### Scenario 3: "I need to check a reference in the old code"

**Old Way:** Browse GitHub in the browser, which is slow and lacks "Go to Definition."

**New Way:**

- I keep my develop (or main) worktree permanently checked out to the latest clean state.
- I simply `cd develop` or search files there. It acts as a local, read-only library of the codebase.

---

## Getting Started

If you want to try this setup, here's how to create the initial structure:

```bash
# Create a bare repo
git clone --bare git@github.com:you/project.git project/.bare
cd project/.bare

# Create your permanent worktrees
git worktree add --detach develop
git worktree add --detach work
git worktree add --detach review  
git worktree add --detach scratch

# In each folder, check out what you need
cd work && git switch -c your-feature-branch
cd ../develop && git switch main
```

You can start with just work and develop folders, then add review and scratch later as needed.
