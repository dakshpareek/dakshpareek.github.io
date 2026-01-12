---
title: "EnvHop"
date: 2026-01-03
description: "A Chrome extension to instantly cycle between Local/Dev/Staging/Prod environments while preserving URL context"
tags: ["chrome extension", "developer tools", "productivity", "side project"]
draft: false
---

# EnvHop

EnvHop is a Chrome extension that helps developers switch between multiple environments (Local / Dev / Staging / Production) without manually editing URLs.

The goal is simple: one click to hop environments while staying on the same page.

## Features

- One-click cycling between environments for the current site
- Preserves URL context (path, query params, and hash fragments)
- Smart setup overlay (click-to-save, no forms)
- Related environment detection for quick grouping
- Keyboard shortcuts in setup overlay (1/2/3/4, Esc)
- Options page to manage projects/environments and export/import config

## Why I Built This

I frequently found myself switching between production, staging, and local environments while debugging issues or validating behavior. Editing subdomains, ports, and copying URLs from notes was repetitive and easy to mess up. EnvHop removes that friction.

## Technologies Used

- **Chrome Extension:** Manifest V3
- **Frontend:** HTML, CSS, JavaScript
- **Storage:** chrome.storage.local

## Links

- **Chrome Web Store:** [EnvHop - Environment Switcher](https://chromewebstore.google.com/detail/envhop-environment-switch/kaccboodoaboanicbaipfkjamaokeehd)
