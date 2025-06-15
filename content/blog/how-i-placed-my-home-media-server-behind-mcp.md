---
title: "How I Placed My Home Media Server Behind MCP Server"
date: 2025-06-15
tags: ["home media server", "MCP Server", "automation", "Prowlarr", "Qbittorrent"]
draft: false
---

## Building MCP Server for my Home Media Network

A few months ago, I created a home media server on my old laptop. I installed Overseerr (for movie requests), Prowlarr (an indexer manager and proxy to simplify using various indexers with apps like Sonarr, Radarr, and others), Radarr (for managing movies), FlareSolver (to bypass Cloudflare), and Qbittorrent (a torrent client).

My idea was to utilize my old laptop and avoid the manual process of downloading torrents by first having to search through the indexers (via VPN), getting the magnet link, and then pasting it into the torrent client. Eventually, the download would be transferred to my current laptop due to space issues.

With my home media server, I can request any movie and have it automatically downloaded to my laptop. However, I still felt that I was doing some manual tasks – like going to Overseerr, searching for the movie, and then clicking the request button.

I thought about creating a script that takes input from the user, searches for the movie in Overseerr, and then requests it automatically. But I wasn’t in the mood to code it out.

Then came MCP Server.

I realized that an application with integrated APIs could be converted into a standalone server. This server can now be easily accessed by LLMs, which in turn can call those APIs (via MCP methods) to perform tasks for me. This instantly clicked.

Earlier, I envisioned being able to chat using a Telegram bot via commands and invoke those APIs to start downloading a movie. When I learned how MCP Server works, I knew this could be a great alternative for my setup.

I simplified my setup by now only needing Qbittorrent, FlareSolver, and Prowlarr. The basic idea is that if I have a download link, I can use Qbittorrent to download it. I no longer require Overseerr or Radarr since I can directly search for movies with Prowlarr, get the magnet link, and download it using Qbittorrent.

I built the MCP Server using the [MCP Protocol](https://modelcontextprotocol.io/introduction).

There was also the challenge of accessing certain content from specific countries, so I had to use a VPN in my setup to easily connect with indexers without issues.

Now, I just need to chat with my LLM like:
- "Get me the latest movie of Tom Cruise"
- "What is the health of Prowlarr?" (to check if indexers are working fine)
- "What's the status of my torrents?"
- "Remove completed torrents?"

**Demo: Checking Health of AI Media Server**
![Health Check of AI Media Server](/gifs/ai-media-health.gif)

And it will perform the tasks for me.

**Demo: Downloading the Latest Movie via MCP Server**
![Downloading Latest Movie Using MCP Server](/gifs/mcp-server-download.gif)

I have also moved my MCP Server behind Cloudflare tunneling, so I can access it from anywhere in the world without worrying about my IP address.

I don't know how much benefit this blog will have for others, but I wanted to share my experience building MCP Server and how it simplifies my day-to-day tasks. It’s fascinating to see how integrating with LLMs enables a lot of automation and task simplification, and I’m looking forward to the day when this integration extends to platforms like WhatsApp Meta Chat.
