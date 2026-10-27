---
title: "R04 Residential and Commercial"
created: 2026-09-23
updated: 2026-09-23
status: sent
tags: [realx, report, product-map, vault]
turn: 4
prompt: "both" · "record everything … into obsidian vault called RealX" · "give me a html doc …" · "once you finish writing it, get Fable 5.1 to QA"
---

# R04 Asset Types, the Vault and the Product Map

> **Prompts (turn 4):**
> - "both"
> - "record everything you search, research, understand, act or report into obsidian vault called RealX."
> - "give me a html doc that has all of the user types described, their user flows, actions they can exercise, including compliance, RealX app owner Admin panel and their screens etc make it look like the brickx as much as possible with screenflow, features, etc"
> - "once you finish writing it, get Fable 5.1 to QA"
>
> **Sent:** 23 Sep 2026, ~12:20 IST, while the Fable 5.1 QA was running. Below is the text as sent in chat, word for word. Decision: D03 in [[Decisions]]. Page: [[RealX Product Map]].

---

**Your product map is written, and Fable 5.1 is reviewing it now.** Open it in any browser: `<workspace>\RealX\04 Architecture\RealX-Product-Map.html`. The review usually takes 20–30 minutes; I'll publish a shareable link once it's done.

**What's in it**, styled after BrickX (charcoal header bands, coral accents, tile tabs, BrickX-style property cards):
- **88 screens with mock-ups:** 39 in the investor app and public site, 15 in the sponsor console, 34 in the admin panel (all 27 admin modules).
- **51 user types and roles:** investors, sponsors, licensed partners, 26 RealX staff roles and 2 system principals, each with what they can do and the compliance gates that apply.
- **21 journeys as screen flows:** each gate is written on the arrow between screens.
- **21 compliance gates, 14 state machines, a 49-row maker-checker policy**, and a BrickX feature-by-feature comparison.

**Everything is also in your RealX vault** (`<workspace>\RealX`); start at `00 Start Here`:
- Search Log and Research Log: every search, file read, browser check and action, turn by turn.
- Reports R01–R04, word for word; Decisions (your 3 answers plus the 9 defaults I'm assuming).
- Research notes, including 5 inconsistencies found in the research pack. For example, BrickX's live listings now sit up to 35% below valuation, while the pack says its floor is 80%.
- The research pack, copied into `06 Evidence/Research Pack` so Obsidian can link to it; the originals are unchanged.
- The generator and data behind the page, in `06 Evidence/Tools/product_map`, so it can be rebuilt after edits.

**On "both" (residential and commercial):** property type is a setting on each listing, and the rules engine applies that type's rules at every transfer:
- **Residential:** foreigners can't buy established homes until 30 Jun 2029 (build-to-rent, student and aged-care housing are exempt), and foreign owners pay a vacancy fee.
- **Commercial:** foreign buyers may need FIRB approval above value thresholds.
- **Both:** holder caps below the stamp-duty threshold; GST applies to commercial rent only.
- **Open for counsel:** how the foreign-buyer ban applies to fractional units.

Next: open `RealX-Product-Map.html` and start with the Journeys section.
