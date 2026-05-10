---
title: "Jot"
date: 2026-05-10
description: "A local-first Chrome screen recorder that captures your tab and exports MP4 without uploading your video anywhere."
tags: ["chrome extension", "screen recorder", "privacy", "local-first", "side project"]
draft: false
---

# Jot

Jot is a Chrome extension for recording your current browser tab without sending your video to any server. It is designed for quick demos, bug reports, walkthroughs, and tutorials where privacy and speed matter.

The core idea is simple: click record, capture the tab you are already on, process the video locally, and download an MP4 that stays on your machine.

## Screenshots

![Jot screenshot 1](/images/jot/jot-1.png)

![Jot screenshot 2](/images/jot/jot-2.png)

## Features

- One-click recording of the active browser tab
- No account, no upload, no cloud processing
- Local MP4 export for broad compatibility
- Crash-tolerant chunk storage using OPFS
- Adaptive quality selection based on device capabilities
- Useful for demos, bug reports, product walkthroughs, and tutorials

## Why I Built This

Most screen recording tools optimize for sharing through the cloud. I wanted the opposite workflow: fast recording that stays private by default.

Jot solves that by making local-first recording the default behavior. Your video never leaves your device.

## Technical Highlights

- **Capture:** Chrome `tabCapture` API for direct tab recording
- **Encoding:** WebCodecs for efficient in-browser encoding
- **Storage:** OPFS for durable local chunk persistence
- **Processing:** FFmpeg compiled to WebAssembly for MP4 output

## Technologies Used

- **Platform:** Chrome Extension (Manifest V3)
- **Browser APIs:** `tabCapture`, WebCodecs, OPFS, Web Workers
- **Media Processing:** FFmpeg WebAssembly

## Links

- **Chrome Web Store:** [Jot — Screen Recorder](https://chromewebstore.google.com/detail/jot-%E2%80%94-screen-recorder/mjlienibahpgoliehkbdeajdedebiedh?hl=en)
- **Blog Post:** [I Built a Screen Recorder That Never Uploads Your Video](/blog/2026/04/18/i-built-a-screen-recorder-that-never-uploads-your-video/)
