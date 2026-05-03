---
title: "Why My GitHub Actions Cache Was Always Cold"
date: 2026-05-03
tags: ["ci", "github-actions", "caching", "pnpm", "devops"]
draft: false
description: "GitHub Actions caches are scoped per Git ref, so pull_request-only workflows never share caches across PRs — even when the cache key is identical. Here's how I diagnosed it and fixed it with a push trigger and a small warmup job."
summary: "Why pnpm install kept showing reused 0 even with cache enabled, and how to share the GitHub Actions cache across pull requests using a push trigger and a warmup job."
---

A few days back I was looking at our CI pipeline and noticed every PR's install step was taking around 25 seconds. Not the end of the world, but suspicious — we already had `cache: 'pnpm'` set up in `setup-node`. Caching was supposed to be working.

So I opened up the install logs and saw this:

```
Progress: resolved 1738, reused 0, downloaded 1721, added 1738, done
```

`reused 0`. Nothing from cache. Every. Single. Time.

And it was the same story across two completely different PRs. That's when I went down the rabbit hole.

## The "wait, but the cache exists" moment

I queried GitHub's actions cache API to see what was actually saved:

```bash
gh api /repos/<owner>/<repo>/actions/caches
```

There were tons of pnpm cache entries. All ~300 MB. All with the same key (since the lockfile hadn't changed). So caches were being created. But none were being restored.

That's when I noticed the `ref` field on each cache entry:

```
ref: refs/pull/550/merge   size: 303 MB
ref: refs/pull/548/merge   size: 303 MB
ref: refs/pull/547/merge   size: 303 MB
...
```

Every cache was scoped to a different PR. And **not a single cache existed on `refs/heads/develop`** — our base branch.

## How GitHub Actions cache scope works for pull requests

Turns out the [rule from GitHub's cache docs](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#restrictions-for-accessing-a-cache) is pretty strict:

A workflow run can read caches from:
- Its **own ref** (e.g., `refs/pull/N/merge` for PR #N)
- The **base branch** of the PR (e.g., `develop`)
- The **default branch** of the repo (e.g., `main`)

It **cannot** read caches from sibling PRs or unrelated branches.

So when PR #547 created a cache, that cache lived on `refs/pull/547/merge`. When PR #548 ran later, it tried to look for the same key on:
- `refs/pull/548/merge` → empty (new PR)
- `refs/heads/develop` → empty (we'll get to why)

Two empty lookups, then a cold install.

## The missing piece: nothing ever ran on the base branch

Here's the kicker. Our workflow only had this trigger:

```yaml
on:
  pull_request:
    branches: [develop, staging, main]
```

A `pull_request` event runs under `refs/pull/N/merge` — never under `refs/heads/develop`. So even though we merged dozens of PRs to develop, the workflow **never actually ran on develop itself**. No run on develop = no cache saved on develop = nothing for future PRs to read.

It's like everyone leaving notes in their own private locker but expecting their teammates to find them on the shared bulletin board.

## The fix: a `push` trigger + a tiny warmup job

The fix is two parts.

**Part 1: also trigger on push to base branches.** This is the only event whose `GITHUB_REF` resolves to `refs/heads/<branch>`, which is exactly the ref PRs are allowed to read from.

```yaml
on:
  pull_request:
    branches: [main, staging, develop]
  push:
    branches: [main, staging, develop]
```

**Part 2: don't re-run the whole pipeline on push.** Tests took 5–6 minutes in our case. Re-running everything on every merge would basically double our CI minutes for no good reason — the PR already validated the same code seconds ago.

Instead, add a tiny "warm-cache" job that only runs on push and only does what's needed to populate the cache. The trick here is using [`pnpm fetch`](https://pnpm.io/cli/fetch), which downloads packages into the store **without** linking `node_modules` — exactly what you want for a cache-warming job:

```yaml
jobs:
  warm-cache:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 1
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Get pnpm store path
        shell: bash
        run: echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV
      - name: Restore pnpm store cache
        uses: actions/cache@v4
        with:
          path: ${{ env.STORE_PATH }}
          key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
          restore-keys: |
            ${{ runner.os }}-pnpm-store-
      - name: Warm pnpm store
        run: pnpm fetch

  check:
    if: github.event_name == 'pull_request'
    # ... your existing lint/build/test steps ...
```

`pnpm fetch` only downloads packages into the content-addressable store. It doesn't link `node_modules`. That makes the warmup ~50 seconds instead of the full ~6 minute pipeline.

## The gotcha: warmup and check jobs must use the same cache key

Here's a thing I missed at first: I wired up the explicit [`actions/cache@v4`](https://github.com/actions/cache) in the warmup job but left the `check` job using `setup-node`'s built-in `cache: 'pnpm'`.

These produce **different cache keys**.

- Warmup wrote: `Linux-pnpm-store-<hash>`
- Check looked for: `node-cache-Linux-x64-pnpm-<hash>`

Same store, different keys. The check would never find what the warmup saved. Hours of work for zero benefit.

So both jobs need to use the **same explicit cache step**:

```yaml
- name: Get pnpm store path
  shell: bash
  run: echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV

- name: Restore pnpm store cache
  uses: actions/cache@v4
  with:
    path: ${{ env.STORE_PATH }}
    key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
    restore-keys: |
      ${{ runner.os }}-pnpm-store-
```

Drop `cache: 'pnpm'` from `setup-node`. Use the explicit cache step in both jobs. Keys match. Everyone's happy.

The `restore-keys` fallback is a small bonus — when someone bumps a dependency and the lockfile hash changes, instead of going fully cold, pnpm restores the previous store and only downloads the delta (a few packages instead of 1700).

## Fixing the concurrency group for push events

If your workflow has this (most do):

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
```

`github.head_ref` is empty on push events, so the fallback becomes `github.run_id` — which is unique per run. So `cancel-in-progress` does nothing for back-to-back merges.

Use `github.ref` as the fallback instead — it's `refs/heads/develop` on push, stable across runs of the same branch:

```yaml
group: ${{ github.workflow }}-${{ github.head_ref || github.ref }}
```

## Verifying the GitHub Actions cache works on the next PR

After merging:

1. The merge to `develop` triggered the `push` event → `warm-cache` ran (~50s, cold).
2. Cache landed on `refs/heads/develop`.
3. Next PR opened against develop — install dropped from ~25s to ~6s. Logs showed `reused 1700+, downloaded ~10`. ✓

Each base branch needs one cold warmup. So `staging` and `main` got warmed when the next merge train rolled through.

You can confirm the cache landed by querying the API yourself:

```bash
gh api /repos/<owner>/<repo>/actions/caches \
  -q '.actions_caches[] | select(.ref == "refs/heads/develop")'
```

## The honest cost

It's not free. Every merge to a base branch now spends ~50s extra on the warmup job.

But every PR run after that saves ~20s on install. With multiple PRs per day per base branch, this pays back in hours, not days.

## What I'd watch out for

- **Don't skip the same-key alignment.** The whole thing is pointless if warmup and check use different cache keys.
- **Lockfile changes will still partially miss.** That's where `restore-keys` saves you — instead of re-downloading 1700 packages, you re-download maybe 30.
- **The first merge after enabling this is still cold.** Don't panic, that's the warmup itself.
- **`cache: 'pnpm'` from `setup-node` doesn't support `restore-keys`.** This is the main reason to switch to the explicit cache step.

## Conclusion

- GitHub Actions caches are scoped per Git ref. PR runs are isolated.
- `pull_request`-only workflows never seed the base branch, so cross-PR caching silently never works.
- A `push` trigger + a small warmup job fixes it without doubling CI minutes.
- Make sure both jobs use the same cache key.

The problem with caching bugs is they don't fail loudly. Your CI still passes. It's just always slow. That's the worst kind of bug — the one nobody complains about because nobody knows it's there.

If you like this kind of "the workflow technically works but is silently wasteful" story, you might also enjoy [how I refined my git worktree workflow from chaos to clarity](/blog/2025/12/17/refining-my-git-worktree-workflow-from-chaos-to-clarity/) — same flavor of "this was always slightly broken, and once I noticed I had to fix it" energy.

## FAQ

**Why does pnpm say `reused 0` even with `cache: 'pnpm'` enabled in setup-node?**

Because GitHub Actions caches are scoped per Git ref. A `pull_request` event runs under `refs/pull/N/merge`, and that's where the cache gets saved. Since each PR has a unique ref, the next PR can't read the previous PR's cache. The cache *is* being saved — it just isn't being read by anyone else.

**Can a pull request read cache from another pull request?**

No. PR runs can only read caches from their own `refs/pull/N/merge`, the base branch (e.g., `refs/heads/develop`), and the default branch (e.g., `refs/heads/main`). Sibling PRs are completely isolated.

**Do I need a separate workflow for the warmup job?**

No. Add it as another job in the same workflow file with `if: github.event_name == 'push'` to gate it. The check job stays gated to `if: github.event_name == 'pull_request'`. They live side by side and only one runs per event.

**Will adding a `push` trigger double my CI minutes?**

Only if you re-run the entire pipeline on push. If your push job is a lightweight warmup (just `pnpm fetch` and a cache step), it adds ~50 seconds per merge — typically 5–10% extra minutes per code journey. The savings on subsequent PRs more than make up for it.

**Why use `actions/cache@v4` instead of `setup-node`'s built-in `cache: 'pnpm'`?**

Two reasons. First, `setup-node`'s cache option doesn't support `restore-keys` — so when the lockfile changes, you get a complete cache miss instead of a partial restore. Second, you have full control over the cache key, which is what lets you align keys across multiple jobs.

**How do I verify the cache is actually being restored?**

Look at the install step logs. If pnpm prints `reused 1700+, downloaded ~10`, the cache hit. If it prints `reused 0, downloaded 1700+`, the cache missed. You can also query `gh api /repos/<owner>/<repo>/actions/caches` to see exactly which caches exist and which ref they're scoped to.
