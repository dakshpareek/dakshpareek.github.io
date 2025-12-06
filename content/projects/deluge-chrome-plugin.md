---
title: "DelugeLink"
date: 2025-12-4
description: "A lightweight Chrome extension to add magnet links and .torrent files directly to a Deluge BitTorrent client."
tags: ["chrome extension", "typescript", "react", "side project"]
draft: false
---

# DelugeLink

DelugeLink is a small, focused Chrome extension that lets you add magnet links and .torrent files to your Deluge BitTorrent client directly from the browser. It removes copy‑paste and tab switching so adding downloads is fast and reliable.

## Overview

Right‑click any magnet link or .torrent file in your browser and send it straight to Deluge. DelugeLink supports label selection, an add‑paused option, and works with both Deluge v1 and v2. The extension handles session re‑authentication and aims to work with private tracker downloads by leveraging browser context for authenticated .torrent downloads.

## Features

- Right‑click context menu for magnet and .torrent links
- Optional add‑paused and label selection
- Automatic support for Deluge v1 and v2 APIs
- Lightweight popup with quick settings
- Designed to handle session expiry and private tracker downloads

## Technologies

- React + TypeScript
- Tailwind CSS
- Vite build system
- Chrome Manifest V3 extension platform

## Installation (developer / testing)

1. Clone the repository.
2. Install and build: `npm install && npm run build`
3. Open `chrome://extensions`, enable Developer mode and click "Load unpacked".
4. Select the `dist/` folder.

## Links

- GitHub: https://github.com/dakshpareek/DelugeLink
- License: MIT
