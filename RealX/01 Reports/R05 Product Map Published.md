---
title: "R05 Product Map Published"
created: 2026-09-23
updated: 2026-09-23
status: sent
tags: [realx, report, product-map, qa]
turn: 5
prompt: "(Fable 5.1 QA finished; follows the user's request: once you finish writing it, get Fable 5.1 to QA)"
---

# R05 Product Map Published

> **Trigger:** the Fable 5.1 QA agent finished (no new user message).
> **Sent:** 23 Sep 2026, ~12:40 IST. Below is the text as sent in chat, word for word. Details: Product Map QA Report, [[RealX Product Map]].

---

**Fable 5.1 passed the product map with fixes (8/10), and it's published:** https://claude.ai/artifact/23baWULw9bDd4kCsbkeAfB. It's private to you until you share it from the page's Share menu.

**What the review fixed** (21 edits from 56 checks):
- **Phone layout:** 68 of 88 screen descriptions were cut off on a phone. Fixed and re-checked at 375 px.
- **Mock numbers:** nine disagreements, e.g. a portfolio total that didn't add up, and a close date 52 days away shown as 22.
- **Accuracy:** one compliance gate was labelled "confirmed" on inferred evidence, and a freeze switch credited to BrickX was actually Assetora's.
- **Plain English:** jargon (AFSL, PDS, FIRB, AML/CTF and others) is spelled out at first use, and low-contrast text is now easier to read.
- **One change I reversed:** the review made the published page load its own diagram library. The artifact viewer draws the diagrams itself, so now only the copy you open from disk loads it.

**Still open for you** (all 11 items are in the vault's QA report):
1. **Five admin screens aren't drawn yet:** users and roles, investor directory, entities register, token registry, and staff sign-in. Their actions are in the tables, but you asked for screens.
2. **Foreign investors:** P1 screens already handle them, but their user type is marked P2. Pick one.
3. **Mock dates:** the sample offer WLG01 appears both still open and already minted, because the mocks are snapshots from different days.
4. **Valuers and property managers** upload through the sponsor console, but those screens don't list them.
5. **Who can see wholesale-only offers:** a marketing-law point for counsel.

The vault has everything: the QA report, the updated logs, and this answer as R05. The copy that opens from disk is still `04 Architecture/RealX-Product-Map.html`.

Next: reply "draw them" and I'll add the five missing admin screens and re-run the checks (about 30 minutes).
