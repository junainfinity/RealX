---
title: "Pack Map"
created: 2026-09-23
updated: 2026-09-23
status: active
tags: [realx, research, research-pack]
aliases: ["Research Pack Map"]
---

# Pack Map

**What the research pack contains, where it came from, and how to read it.** The pack is the research base for RealX. The report given to the user is [[R01 Research Pack Overview]].

## 1. Provenance

| Item | Value |
|---|---|
| Original | `~\Downloads\tokenization-research.zip`: 792,674 bytes, SHA-256 `1CD6BBBF6712807E9BFB2551A0608ED925F57F85B584ACC0CBF4324853F761A9` |
| Contents | 79 zip entries: 67 markdown files in 12 folders, under a `tokenization-research/` wrapper |
| Extracted copy | `<workspace>`, without the wrapper folder. Identical to the zip: 67 of 67 files match by SHA-256 |
| Vault copy | 06 Evidence/Research Pack, copied 23 Sep 2026 at 11:22 IST and verified identical |
| Made by | Claude in Cowork, orchestrating 6 research agents and 2 synthesis agents, 22–23 Sep 2026 (22:47–02:19 UTC per action-log) |
| Reviewed by | Fable 5.1 QA: 5 parallel agents, every slice "Pass with fixes", ~8.1/10, ~165 edits (qa-summary) |
| Prepared for | Arjun · RealX |

## 2. What it studies

The pack reverse-engineers the top 5 fractional real-estate tokenisation platforms in each of the US, EU, MENA and Australia, 20 in all. For each it covers compliance, user types and actions, architecture, flows and tech stack, plus an extrapolated owner admin panel.

| Region | Platforms |
|---|---|
| US | Lofty · RedSwan · Arrived · RealT (in liquidation) · mogul |
| EU | Reental · Blocksquare + Oceanpoint · Propvest/Exporo · Brickken · Domoblock |
| MENA | PRYPCO Mint · Ctrl Alt · Stake · Saudi REGA/RER + sandbox · SmartCrowd/Nawy Shares |
| Australia | BrickX · Redbelly Network · Tokeniser + Liquidise · Canvas Connect · Assetora (DomaCom) + Bricklet |

## 3. Structure and size

| Folder | Files | Lines | Words | What it holds |
|---|---|---|---|---|
| `00-method` | 9 | 1,137 | 26,280 | Methodology, template, action log, 6 QA reports |
| `01-regulatory` | 6 | 1,286 | 25,880 | Law per region, plus the cross-cutting controls map |
| `02-platforms` | 28 | 8,832 | 112,300 | 20 profiles, plus regional summaries and search logs |
| `03-infra-and-admin-evidence` | 5 | 1,186 | 17,953 | Token standards, vendors, white-label dashboards, admin evidence |
| `04-synthesis` | 9 | 2,239 | 38,405 | Executive summary, RealX playbook, matrices, architecture, flows |
| `05-admin-panels` | 9 | 5,257 | 66,131 | 27-module admin spec, RBAC, state machines, data model, wireframes |
| `README.md` | 1 | 160 | 1,787 | Index |
| **Total** | **67** | **~20,100** | **~289,000** | 124 mermaid diagrams |

Evidence labels across the pack: ~5,660 confirmed [C], ~1,350 inferred [I] and ~170 speculative [S].

## 4. Reading order

Reading times assume ~200 words a minute.
1. README (~8 min)
2. [[00-executive-summary]] (~17 min)
3. [[implications-for-realx]] (~19 min)
4. [[00-overview]] (~24 min). The full [[01-reference-admin-spec]] takes ~2 hours.
5. [[conflicts-and-gaps]] and qa-summary, for what is still uncertain (~20 min)

## 5. Headline findings, as the pack states them

1. Platforms fail on property operations, not smart contracts (RealT).
2. Tokenising changes the register, not the legal wrapper.
3. The core architecture decision is where the golden record lives. Every option needs daily three-way reconciliation.
4. Secondary liquidity is unsolved: nobody runs continuous trading on a licensed venue with T+0 stablecoin settlement.
5. The branding is crypto but the plumbing is fiat. Wallets have converged on no seed phrase, and freeze, forced transfer and recovery are universal.
6. Token standards are converging on ERC-3643 + ERC-7943.
7. Regulation in 2025–26 changed the map in every region.

## 6. Admin-panel design (extrapolated)

The pack specifies:
- 27 modules (M01–M27) and 6 admin archetypes;
- 41 roles (the README says 40; see [[Pack Issues]]);
- a 57-row maker-checker policy;
- 13 state machines, an ER data model and 11 ASCII wireframes.

Its design principle: approve in the admin panel, sign in custody, and bind the two with one approval ID.

## 7. Known limitations, from the pack

- The shared web-search quota ran out mid-research. Later work fetched primary sources directly.
- KYC vendors and token-contract settings are undisclosed for most platforms.
- The admin panels are extrapolations. The pack is research, not legal advice.
- P1 items to re-verify: the Australian licence for fractional MIS-unit tokens, the CA DFPI order against Lofty, and Saudi status after Apr 2026.
