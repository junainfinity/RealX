---
title: "R01 Research Pack Overview"
created: 2026-09-23
updated: 2026-09-23
status: sent
tags: [realx, report]
turn: 1
prompt: "understand whats in the zip file as well as folder"
---

# R01 Research Pack Overview

> **Prompt (turn 1):** "understand whats in the zip file as well as folder"
> **Sent:** 23 Sep 2026, ~11:08 IST. Below is the text as sent in chat, word for word. Links now point to the vault copy of the pack. Details: [[Pack Map]].

---

**The zip and `<workspace>` contain exactly the same thing.** The folder is the unzipped zip: all 67 files match byte for byte. The only difference is that the zip wraps everything in a `tokenization-research/` folder. There's no code in the folder, and it isn't a git repo.

## What it is

It's a research pack made for you and RealX by Claude in Cowork, overnight on 22–23 Sep 2026.
- **Question:** how do the top 20 fractional real-estate tokenisation platforms work? It covers 5 each in the US, EU, MENA and Australia.
- **Angles:** compliance, user types, architecture, flows, tech stack, and an admin panel for RealX worked out from the evidence.
- **Size:** about 290k words, 124 diagrams and about 1,050 logged searches.
- **Quality:** a separate review by the Fable 5.1 model rated every section "Pass with fixes" (about 8/10) and applied about 165 corrections.
- **Evidence:** every claim is labelled confirmed, inferred or speculative. About 5,700 are confirmed, 1,350 inferred and 170 speculative.

## How it's organised

**The conclusions (read these first):**

| Where | What | Read time |
|---|---|---|
| README.md | Index, the 20 platforms, headline findings | ~8 min |
| [[00-executive-summary\|00-executive-summary.md]] and [[implications-for-realx\|implications-for-realx.md]] | Cross-platform findings and the plan for RealX | ~35 min |
| [[00-overview\|05-admin-panels/00-overview.md]] | Admin-panel design: 27 modules, 41 roles, 13 workflows | ~25 min (the full spec is ~2 hrs) |

**The evidence behind them:**

| Folder | What |
|---|---|
| `01-regulatory/` | The law in each region, plus a map from each legal obligation to the control and admin feature that handles it |
| `02-platforms/` | 20 platform breakdowns, all written to the same 13-section template |
| `03-infra-and-admin-evidence/` | Token standards, vendors, and vendors' published admin dashboards |

**How it was made:** `00-method/` has the method, a log of every step, and 5 review reports.

## The 5 findings that matter most

1. **Platforms fail on running the properties, not on smart contracts.** RealT has been in voluntary liquidation since Jul 2026 after trouble with Detroit code enforcement, property taxes and a court-appointed fiduciary. Its contracts still work.
2. **A token changes the ownership register, not the legal structure.** The investment is still regulated as a security (US and EU), a regulated virtual asset (Dubai) or a managed investment scheme interest (Australia).
3. **The core architecture decision is where the legal ownership record lives:** on the blockchain, with a transfer agent, or in the government land registry. Every option needs a daily check that the blockchain, the official register and the bank all agree.
4. **Reselling tokens is unsolved.** Nobody runs continuous trading on a licensed exchange with instant stablecoin settlement. That's the clearest gap in the market.
5. **Crypto branding, ordinary money underneath.** Regulated platforms mostly move normal currency through custodial accounts. Wallets no longer use seed phrases, and every platform can freeze tokens and recover lost ones.

## What it recommends for RealX

- **Partner for the licences, own the product:** use licensed partners to issue and trade in the US, and join the Dubai Land Department (DLD) platform as a distributor in the UAE.
- **One rules engine with per-country settings,** instead of four separate compliance systems.
- **Phases:** foundations (months 0–3) → US launch for wealthy and non-US investors through a partner (3–9) → US retail, EU and UAE (9–18) → Australia and Saudi Arabia (18–30).
- **Stand out on verifiability:** publish per-property proof of title, taxes paid and insurance, plus audited figures.

## What's still open

- **The plan is a default.** It assumes answers to 15 questions for you in [[implications-for-realx#5. Open questions for the founder|implications-for-realx.md §5]].
- **Top items to re-check:** which Australian licence applies, the California regulator's order against Lofty, and where Saudi rules stand after Apr 2026.
- **Caveats:** the admin panels are educated guesses, since no platform publishes its back office. None of this is legal advice.
- **Integrity:** all 75 internal links work. Two small inconsistencies:
  - [[methodology#3. Process|methodology.md:38]] points to a `verification-report.md` that doesn't exist.
  - The README says 40 admin roles; the admin overview says 41.

Next: tell me here where RealX will be based, and whether it will own properties or run a marketplace for other property sponsors. Those two answers pick the launch region and licence route.
