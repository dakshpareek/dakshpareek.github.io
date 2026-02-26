---
title: "Postgres Physical Replication (Streaming Replicas)"
date: 2026-02-26
tags: ["postgres", "database", "replication", "wal"]
draft: false
---

## Postgres Physical Replication

### Physical Streaming Replication

In physical streaming replication, the replica server continuously receives and applies changes from the primary server in real-time.

Primary Server generates Write-Ahead Log (WAL) which means that all changes to the database are first written to a log before being applied to the actual data files. This allows the replica server to replay these logs and keep its data in sync with the primary server.

Replica server continuously replays WAL logs received from the primary server, applying changes to its own data files. This ensures that the replica server remains up-to-date with the primary server's data.

┌─────────────────────────────────────────────────────────────────┐
│                        PRIMARY SERVER                           │
│                                                                 │
│   ┌──────────────┐    WAL Write     ┌────────────────────────┐  │
│   │   Database   │ ───────────────► │  Write-Ahead Log (WAL) │  │
│   │   Changes    │                  │       Files            │  │
│   └──────────────┘                  └────────────┬───────────┘  │
│                                                  │              │
└──────────────────────────────────────────────────┼──────────────┘
                                                   │
                                      WAL Streaming│ (continuous)
                                                   │
┌──────────────────────────────────────────────────┼──────────────┐
│                        REPLICA SERVER            │              │
│                                                  ▼              │
│   ┌──────────────┐    WAL Replay    ┌────────────────────────┐  │
│   │   Database   │ ◄─────────────── │   WAL Receiver         │  │
│   │   Data Files │                  │   (applies changes)    │  │
│   └──────────────┘                  └────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

A replication slot ensures the primary never discards WAL segments until the replica has consumed them. Without it, if the replica falls behind, the primary might delete WAL the replica still needs — causing the replica to break and require a full re-sync.

---

### Replication Styles

#### Sync Replication

In synchronous replication, the primary server waits for confirmation from the replica server that it has received.

Client          Primary                        Replica
  │                │                              │
  │── Write ──────►│                              │
  │                ├─ 1. Write WAL                │
  │                ├─ 2. Apply to data files      │
  │                ├─ 3. Stream WAL ─────────────►│
  │                │◄────── Replica confirmed ────┤
  │◄── Success ────┤                              │

#### Async Replication

In asynchronous replication, the primary server does not wait for confirmation from the replica server before acknowledging.

Client          Primary                        Replica
  │                │                              │
  │── Write ──────►│                              │
  │                ├─ 1. Write WAL                │
  │                ├─ 2. Apply to data files      │
  │◄── Success ────┤                              │
  │                └─ 3. Stream WAL ─────────────►│ (happens in background, client already got success)

Data loss in async only happens when: primary disk dies unrecoverably AND you failover to replica. A simple crash + restart loses nothing because WAL is still on primary's disk.

---

## Deep Points

Replica gets WAL via one of two methods: streaming replication (continuous) or file-based log shipping (periodic).

Streaming replication key idea: the replica is basically saying, "I’m currently replayed up to WAL position X; send me everything after X."

WAL shipping (file-based) is more of a batch process: the primary server periodically archives completed WAL segments to a shared location, and the replica server periodically fetches and applies these archived WAL segments to stay up-to-date.

Replication slots are a critical component of PostgreSQL's replication mechanism. They ensure that the primary server retains the necessary WAL segments until the replica has consumed them, preventing data loss and ensuring the integrity of the replication process.
