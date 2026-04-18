---
title: "I Built a Screen Recorder That Never Uploads Your Video"
date: 2026-04-18
tags: ["chrome extension", "screen recorder", "privacy", "WebAssembly", "FFmpeg", "local-first", "tabCapture"]
draft: false
---

## Why Another Screen Recorder?

Every screen recorder I tried — Loom, CloudApp, Screencastify — uploads your video to their servers. For quick dev demos, bug reports, or client walkthroughs, I just wanted a recording that stays on my machine. No accounts, no uploads, no wondering who else can see my screen.

So I built [Jot](https://chromewebstore.google.com/detail/jot-%E2%80%94-screen-recorder/mjlienibahpgoliehkbdeajdedebiedh) — a Chrome extension that records your browser tab and processes everything locally. Your video never leaves your device.

## How It Works

When you click record, Jot captures your active browser tab using Chrome's `tabCapture` API. There's no clunky share picker — it starts immediately on the tab you're looking at.

The recording pipeline runs entirely inside your browser:

1. **Capture** — Chrome's tabCapture grabs the tab's video and audio stream
2. **Encoding** — WebCodecs API encodes frames in real-time using hardware acceleration when available
3. **Storage** — Chunks are persisted to the browser's Origin Private File System (OPFS), so even if Chrome crashes mid-recording, your data is recoverable
4. **Processing** — FFmpeg compiled to WebAssembly muxes the chunks into a standard H.264 MP4
5. **Download** — You get a `.mp4` file that plays everywhere

No server involved at any step.

## The Technical Decisions

### Why tabCapture Instead of getDisplayMedia?

`getDisplayMedia` shows that clunky Chrome share picker every time. `tabCapture` directly captures the active tab — one click and you're recording. The tradeoff is you can only record browser tabs, not desktop or other windows. For my use case (recording web apps, demos, tutorials), that's perfect.

### Why OPFS for Chunk Storage?

Recording generates a lot of data. Writing it to regular browser storage (IndexedDB, localStorage) would be slow and limited. OPFS gives you a proper file system with synchronous write access from a Web Worker. This means:

- No data loss on crash — chunks are written to disk as they arrive
- Recovery is possible — if processing fails, you can retry from the raw chunks
- No storage quota surprises — OPFS can handle gigabytes

### Why FFmpeg in WebAssembly?

The browser records in WebM format, but WebM playback is inconsistent across devices — especially on iOS and older players. FFmpeg remuxes (and when needed, re-encodes) to H.264 MP4, which plays literally everywhere. Compiling FFmpeg to Wasm means this happens entirely in your browser with zero server round-trips.

### Adaptive Quality

Jot detects your screen resolution, available memory, and CPU cores to automatically pick the right recording quality — from 1080p30 up to 4K30. You can also manually choose a preset. If your hardware can't sustain the requested quality, it gracefully falls back.

## What I Learned Building This

**Chrome extension architecture is surprisingly powerful.** Between offscreen documents, Web Workers, OPFS, WebCodecs, and Wasm — you can build genuinely complex applications that run entirely client-side.

**The hardest part was state management.** A screen recorder has a lot of states: idle, preflight checks, armed, recording, stopping, processing, validating, done, error, recovery. Getting the transitions right — and handling every edge case (tab closes mid-recording, storage runs out, mic disconnects) — took more effort than the actual capture code.

**Local-first is a real differentiator.** People care about privacy more than I expected. "Your video never leaves your device" is a one-liner that immediately resonates.

## Try It

Jot is free and open source. Install it from the [Chrome Web Store](https://chromewebstore.google.com/detail/jot-%E2%80%94-screen-recorder/mjlienibahpgoliehkbdeajdedebiedh) and record your first video in under 10 seconds.

No account needed. No uploads. Just a recording that stays yours.
