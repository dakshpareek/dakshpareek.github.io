---
title: "Spine: AI-Ready Codebase Context"
date: 2025-11-09
tags: ["spine", "AI", "codebase", "architecture"]
draft: false
---

## Spine: AI-Ready Codebase Context

AI coding assistants lack architectural context. Every time I asked Claude or ChatGPT to write code, I'd waste time explaining my codebase structure, patterns, and conventions.

I needed a lightweight, always-fresh representation of my codebase—just the skeleton, no implementation details. So I built Spine.

## What It Does

Spine extracts your codebase into `.spine/` with "skeletons"—function signatures, class structures, type definitions, exports. About 6% of original code size. Feed these to your AI, and it understands your entire architecture.

## The Workflow

```bash
go install github.com/dakshpareek/spine@latest
cd /path/to/your/project
spine init
spine pipeline --output prompt.md
```

1. `pipeline` scans for changes and generates an AI prompt
2. Paste prompt into Claude/ChatGPT
3. AI generates skeletons → save to `.spine/skeletons/`
4. Run `spine validate --fix`
5. Next time, only new/changed files need skeletons

## Why Manual First

Phase 1 is intentionally manual:
- Simple and reliable
- Works with any LLM
- No vendor lock-in
- Build incrementally

Phase 2 adds automation, watch mode, git hooks.

## Get Started

```bash
go install github.com/dakshpareek/spine@latest
cd /path/to/your/project
spine init
spine pipeline --output prompt.md
```

[GitHub](https://github.com/dakshpareek/spine) | [Docs](https://github.com/dakshpareek/spine/blob/main/README.md)
