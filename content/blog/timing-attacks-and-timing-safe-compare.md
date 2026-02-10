---
title: "Timing Attacks and Timing-Safe Compare"
date: 2026-02-10
tags: ["security", "webhooks", "nodejs", "crypto"]
draft: false
---

A few days back, I was verifying a webhook secret and I had a line like this:

```js
if (secret !== this.webhookSecret) throw Unauthorized
```

It looked totally fine. It works. And for most real-world scenarios, it will probably never bite you.

But this is one of those things where you can make your code *a bit* more correct (security-wise) with almost zero cost.

So I went down the rabbit hole.

## The Problem (Timing Attack)

I'll break this down with a simple analogy.

Imagine you're trying to guess someone's password letter by letter. Normally you'd have no clue if you're right or wrong.

But what if the system took **slightly longer** to respond when your first letter was correct vs wrong?

That “slightly longer” is the leak.

### What leaks in code?

When many runtimes compare strings/bytes, they often do it left-to-right and stop at the first mismatch.

So comparisons can take different time depending on how many prefix characters match.

Example:

- `"abc123"` vs `"xyz789"` → mismatch immediately
- `"abc123"` vs `"abc999"` → mismatch later

The second comparison does more work before failing, which *might* take microseconds longer.

And if an attacker can measure that difference repeatedly, they can learn information about your secret.

## The Attack Scenario

An attacker could do something like:

1. Try secret `"a______"` and measure response time
2. Try secret `"b______"` and measure response time
3. If `"a______"` is consistently a tiny bit slower, they infer the first character is `"a"`
4. Repeat for each position: `"aa_____"`, `"ab_____"`, etc.

Over many requests, they can incrementally discover the secret.

This is the classic “timing attack” idea: you’re not leaking the secret directly, you’re leaking a hint through time.

## The Solution: Timing-Safe Comparison

Node gives us a primitive for this:

`crypto.timingSafeEqual(a, b)`

This does the comparison in **constant time** (for same-length inputs): it always checks every byte, even if it finds a mismatch early.

No “stop early” behavior = no “prefix matched longer” leak.

### Important gotcha

`crypto.timingSafeEqual` expects buffers of the **same length**. If you pass different lengths, it throws.

So you need to handle that first.

Here is a tiny helper I use:

```js
import crypto from "node:crypto";

function timingSafeEqualUtf8(a, b) {
  const aBuf = Buffer.from(a, "utf8");
  const bBuf = Buffer.from(b, "utf8");
  if (aBuf.length !== bBuf.length) return false;
  return crypto.timingSafeEqual(aBuf, bBuf);
}
```

And then your check becomes:

```js
if (!timingSafeEqualUtf8(secret, this.webhookSecret)) throw Unauthorized
```

That’s it.

## Is This Actually Dangerous?

**In practice:** network latency (milliseconds) usually dwarfs these microsecond-level differences, so pulling this off over the public internet is extremely difficult.

**But:**
- It costs almost nothing to implement
- It protects you in low-latency environments (same region/DC, internal networks, local attackers)
- It avoids “death by a thousand cuts” scenarios where other optimizations reduce noise and make timing signals easier to measure

## Where You Should Care Most

You should care about timing-safe compare when:
- You’re comparing **secrets** (webhook secrets, API tokens, session identifiers)
- You’re comparing **signatures** (HMAC/sha256 webhook signatures)
- The comparison result is used for **authorization**

For webhook signatures specifically, the pattern is:
1. Compute expected signature (HMAC) from the raw request body
2. Compare provided signature vs expected signature using timing-safe equality

## Conclusion

- `!==` comparisons can leak tiny timing differences when comparing secrets.
- `crypto.timingSafeEqual` is a cheap best-practice improvement.
- Over the internet this attack is hard, but the fix is easy, so I prefer to do it.
