---
title: "Spine: AI-Ready Codebase Context"
date: 2025-11-09
description: "A CLI tool that extracts your codebase's architectural skeleton for AI coding assistants."
tags: ["CLI tool", "open source", "Go", "AI"]
draft: false
---

# Spine

Extract your codebase's architecture into a lightweight, AI-ready format. Give coding assistants full architectural context before they write code.

## Overview

Spine solves a real problem: AI coding assistants (Claude, ChatGPT) lack your codebase context. Instead of explaining structure repeatedly, Spine maintains a compressed architectural snapshot—just skeletons, no implementation.

## Features

- Automated file tracking with git-aware change detection
- Generates AI prompts for skeleton creation
- Validates skeleton integrity
- Manual-first workflow (no vendor lock-in)
- Works with any LLM

## Workflow

```bash
spine init                    # Initialize workspace
spine pipeline --output prompt.md  # Generate AI prompt
# Feed prompt to Claude/ChatGPT, save skeletons
spine validate --fix          # Update index
```

## Technologies Used

- **Language:** Go 1.23+
- **Features:** File hashing, git integration, JSON indexing
- **Distribution:** Pre-built binaries + source installs

## Links

- **GitHub Repository**: [dakshpareek/spine](https://github.com/dakshpareek/spine)
- **Documentation**: [README](https://github.com/dakshpareek/spine/blob/main/README.md)
- **Blog Post**: [How I Built Spine](/blog/codebase-context-tool)
