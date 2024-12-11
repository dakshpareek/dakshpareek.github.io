---
title: "AI Bookmark Organizer"
date: 2024-11-28
description: "A Chrome extension that automatically organizes your bookmarks into categories using AI."
tags: ["chrome extension", "application", "side project", "AI"]
draft: false
---

# AI Bookmark Organizer

Organize your bookmarks effortlessly with AI-powered categorization.

## Overview

AI Bookmark Organizer is a Chrome extension that automatically organizes your bookmarks into categories using AI. By leveraging the power of AI models built into Google Chrome, it provides a seamless way to keep your bookmarks tidy without manual intervention.

## Features

- **Automatic Categorization**: Newly added bookmarks are automatically categorized based on their content.
- **Batch Organization**: Organize all existing bookmarks with a single click from the extension's popup.
- **Custom Categories**: The AI categorizes bookmarks into appropriate categories, making it easier to find them later.

## How It Works

The extension listens for bookmark creation events. When a new bookmark is added, it:

1. Sends the bookmark's title and URL to an AI model.
2. The AI model analyzes the content and determines the most suitable category.
3. The bookmark is moved into a folder named after the category.

For existing bookmarks, you can click the **"Organize My Bookmarks"** button in the popup to categorize them all at once.

## Technologies Used

- **Frontend**: JavaScript, HTML, CSS
- **Chrome APIs**: Bookmarks API, Tabs API, Scripting API
- **AI Integration**: Utilizes AI capabilities built into Google Chrome for natural language processing.

## Installation

Detailed installation instructions are available in the GitHub repository.

## Links

- **GitHub Repository**: [AI Bookmark Organizer](https://github.com/dakshpareek/ai-bookmark-organizer)
- **Demo Video**: [YouTube](https://youtu.be/wWhv3a-wKeo)
