---
title: "How I Work Using Git Worktree?"
date: 2025-08-15
lastmod: 2025-12-17
tags: ["git", "worktree", "productivity", "development", "workflow"]
draft: false
---


> **Update 2025:** I've refined this workflow significantly since writing this! While the method below works for creating isolated environments, I found a more sustainable 'Persistent Studio' approach that works better for daily development. [Read the new guide here](/blog/2025/12/17/refining-my-git-worktree-workflow-from-chaos-to-clarity/).

---

## Introduction to Pain
Remember when we are in middle of our feature and our peer asks us to make some changes in that feature that we delivered last week. We have to switch the context with git stash ususally or commit half baked feature in between without a full review. Then we switch to that branch and make our fixes and pick our original work again.

I know you can relate to this situation.

## Git Worktree - The Solution
Most of us are already familiar with git stash but not git worktree. Git worktree is like having a new fresh git clone of that repository and switching to that specific branch and doing our work.
But it is different than having seperate git clone of that repository.
We can actually checkout to multiple branches at the same time(without the need of stash/commit) and work on them in parallel(without addressing conflicts because they are not there in my seperate worktree).

### Visualizing the Structure
```
my-project/
├── .bare/                  ← Bare repository (no working files)
│   ├── objects/            ← All commits and history
│   ├── refs/               ← All branches and tags
│   ├── config              ← Repository settings
│   └── worktrees/          ← Links to worktrees
│       ├── main/
│       ├── feature/
│       └── hotfix/
├── .git                     ← Points to .bare directory
├── main/                    ← Worktree #1
│   ├── src/
│   ├── package.json
│   └── .git                ← Points back to .bare
├── feature/                 ← Worktree #2
│   ├── src/
│   ├── package.json
│   └── .git                ← Points back to .bare
└── hotfix/                  ← Worktree #3
    ├── src/
    ├── package.json
    └── .git                ← Points back to .bare
```

So we just `cd` into that worktree and do our work.

## Keep in mind that
- Each worktree has its own dependencies and environment. So each time you create a new worktree, you need to install those dependencies again. This will take space on your disk.
- You need to copy your `.env` file to each worktree if you are using environment variables.
- If an external service like a database is there they if will use that same db for all worktree unless you have configured it to use different db for each worktree.

## My Project Setup to Leverage Git Worktree

### What the problem using git worktree with default approach?
If you create worktrees inside your main repo:
  ```
  my-project/
  ├── .git/
  ├── src/
  ├── package.json
  ├── hotfix/          ← Worktree creates subdirectory here
  │   ├── .git
  │   └── src/
  └── feature/         ← Another worktree subdirectory
      ├── .git
      └── src/
  ```

- Your main repo becomes cluttered.
- Untracked directories appear in git status.
- Many code editors expect a single working directory per repo.
- It's not clean.

### My Approach
I use git worktree with a bare repository and this is how my project structure looks like:
```
my-project/
├── .bare/           ← Single Git database
├── main/            ← Clean worktree
├── hotfix/          ← Clean worktree
└── feature/         ← Clean worktree
```

- I can `cd` into any worktree and those multiple worktrees are respected by my code editor as well.
- I can easily see which worktree I am in.
- It feels organized and clean to me.

### But what is bare repository?

A bare repository is a Git repository that contains ONLY the version history and Git data, but NO working files that we can edit.

*Regular Repository*
```
my-project/
├── .git/            ← Git database (history, branches, etc.)
├── src/             ← Working files
├── package.json     ← Working files
└── README.md        ← Working files
```

*Bare Repository*
```
my-project.git/      ← Note: .git extension by convention
├── objects/         ← Git database contents (same as .git/objects/)
├── refs/            ← Branch and tag references
├── config          ← Repository configuration
├── HEAD            ← Current branch pointer
└── hooks/          ← Git hooks
```

You can read more on [Git Bare Repositories](https://www.geeksforgeeks.org/git/bare-repositories-in-git/). I am not going into details of bare repository here. This article is about how I use git worktree.

### How I use git worktree with bare repository?
I used to use git worktree commands which are there and then coping my .env files and then installing dependencies in each worktree. But I wanted to automate this process and make it easier for myself.

1. To solve for coping .env files, I have created a folder called shared-config in my bare repository. I keep my .env files there and when my script creates a new worktree it does a [symlinks](https://www.freecodecamp.org/news/symlink-tutorial-in-linux-how-to-create-and-remove-a-symbolic-link/) to those .env files in their place.

```bash
if [ -d "apps/core-backend/" ]; then
  ln -sf shared-config/core-backend.env apps/core-backend/.env // Creating a symbolic link to the .env file from shared-config
fi
```

2. To solve for installing dependencies, I have created a script that runs `npm install` in each worktree after creating it.

Below is my complete script that I use to create a new worktree and set it up:

```bash
BASE_BRANCH=$1
NEW_BRANCH=$2
WORKTREE_DIR=$3

if [ -z "$BASE_BRANCH" ] || [ -z "$NEW_BRANCH" ] || [ -z "$WORKTREE_DIR" ]; then
    echo "Usage: ./setup-worktree.sh <base-branch> <new-branch> <worktree-dir>"
    echo "Example: ./setup-worktree.sh origin/develop feature/GIV-157 GIV-157"
    exit 1
fi

echo "🚀 Setting up worktree: $WORKTREE_DIR for new branch: $NEW_BRANCH from base: $BASE_BRANCH"

# Check if remote origin is properly configured
if ! git remote get-url origin > /dev/null 2>&1; then
    echo "❌ ERROR: Remote 'origin' is not properly configured"
    echo "   Please run: git remote set-url origin <your-repo-url>"
    echo "   Example: git remote set-url origin https://github.com/your-org/givo-backend.git"
    exit 1
fi

echo "🔄 Fetching latest from remote..."
git fetch origin

# Check if worktree directory already exists
if [ -d "$WORKTREE_DIR" ]; then
    echo "⚠️  Worktree directory '$WORKTREE_DIR' already exists. Skipping creation."
else
    echo "📁 Creating worktree..."
    if ! git worktree add "$WORKTREE_DIR" -b "$NEW_BRANCH" "$BASE_BRANCH"; then
        echo "❌ Failed to create worktree"
        exit 1
    fi
fi

cd "$WORKTREE_DIR" || { echo "❌ Failed to enter worktree directory"; exit 1; }

# 1. Setup .env files (symlinks) if apps directory exists
if [ -d "apps/cms/" ] && [ -d "apps/core-backend/" ]; then
    echo "🔗 Setting up environment files..."
    pushd apps/cms/ > /dev/null
    ln -sf ../../../shared-config/cms.env .env
    popd > /dev/null
    pushd apps/core-backend/ > /dev/null
    ln -sf ../../../shared-config/core-backend.env .env
    popd > /dev/null
else
    echo "⚠️  Apps directory not found, skipping .env setup"
fi

# 2. Install dependencies if package.json exists
if [ -f "package.json" ]; then
    echo "📦 Installing dependencies..."
    pnpm install:all
    # 3. Build shared packages
    echo "🔨 Building shared packages..."
    pnpm build:shared
else
    echo "❌ No package.json found in worktree, skipping dependency installation"
fi

# 4. Check if we are in a detached HEAD state and warn the user
CURRENT_BRANCH=$(git symbolic-ref --short -q HEAD)
if [ -z "$CURRENT_BRANCH" ]; then
    echo -e "\n⚠️  WARNING: You are in a detached HEAD state!"
    echo "   To avoid losing work, create a branch with:"
    echo "     git checkout -b <your-branch-name>"
    echo "   Or, if you meant to work on an existing branch, run:"
    echo "     git checkout <branch-name>"
else
    echo "✅ You are on branch: $CURRENT_BRANCH"
fi

cd ..

echo "✅ Worktree '$WORKTREE_DIR' ready!"
echo "📂 Location: $(pwd)/$WORKTREE_DIR"
```

### Commands I use to create worktree in different scenarios encounter?

- When I want to create a new feature branch from develop branch:
```bash
./setup-worktree.sh origin/develop feature/GIV-XXX GIV-XXX
```

- When I want to create a new branch from local branch:
```bash
./setup-worktree.sh feature/123 feature/GIV-XXX-v1 GIV-XXX-v1
```
