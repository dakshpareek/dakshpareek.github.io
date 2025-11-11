---
title: "ctx: AI-Ready Codebase Context"
date: 2025-11-09
tags: ["ctx", "AI", "codebase", "architecture"]
draft: false
---

## ctx: AI-Ready Codebase Context

AI coding assistants lack architectural context. Every time I asked Claude or ChatGPT to write code, I'd waste time explaining my codebase structure, patterns, and conventions.

I needed a lightweight, always-fresh representation of my codebase—just the skeleton, no implementation details. So I built `ctx`.

## What It Does

`ctx` extracts your codebase into `.ctx/` with “skeletons”—function signatures, class structures, type definitions, exports. Roughly 6% of the original code size. Feed these to your AI, and it understands your entire architecture before touching real files.

## The Workflow

```bash
go install github.com/dakshpareek/ctx@latest
cd /path/to/your/project
ctx init          # bootstrap .ctx/ and seed the index
ctx ask           # auto-sync + prompt to .ctx/prompt.md
# ... share prompt, save skeletons to .ctx/skeletons/ ...
ctx update        # mark skeletons current after AI saves them
ctx bundle        # optional: export .ctx/context.md for pairing
```

1. `ctx ask` scans for changes and writes an AI prompt to `.ctx/prompt.md`
2. Paste prompt into Claude/ChatGPT
3. AI generates skeletons → save to `.ctx/skeletons/`
4. Run `ctx update`
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
go install github.com/dakshpareek/ctx@latest
cd /path/to/your/project
ctx init
ctx ask
```

[GitHub](https://github.com/dakshpareek/ctx) | [Docs](https://github.com/dakshpareek/ctx/blob/main/README.md)
