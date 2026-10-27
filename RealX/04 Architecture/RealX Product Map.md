---
title: "RealX Product Map"
created: 2026-09-23
updated: 2026-09-23
status: complete
tags: [realx, architecture, proposal, product, screens, admin-panel]
aliases: ["Product Map", "RealX Screen Map", "RealX Page Catalog"]
---

# RealX Product Map (proposed)

**Every RealX user type, journey, screen, action and compliance gate, plus the owner's admin panel, on one web page styled after BrickX.** Everything is P (proposed). It is built from the research pack and the decisions in [[Decisions]], checked against BrickX's public site on 23 September 2026, and reviewed by Fable 5.1 (Product Map QA Report). It is not legal advice.

- **Open from disk:** [RealX-Product-Map.html](RealX-Product-Map.html), a complete HTML file. Obsidian cannot render HTML notes, so open it in a browser.
- **Artifact source:** [RealX Product Map.html](RealX%20Product%20Map.html), the same page without the document wrapper the publisher adds.
- **Published copy:** https://claude.ai/artifact/23baWULw9bDd4kCsbkeAfB. It is private to the owner until shared from the page's Share menu. Version 1 was published 23 Sep 2026 at 12:33 IST, after QA. Version 2 (~20:20 IST) adds founder decision D04: API tokens for every user's own AI agent.
- **QA:** Fable 5.1 rated it **Pass with fixes, 8/10**: 56 checks, 21 edits, 11 open items (Product Map QA Report).

## What it contains

| Part | Count | Notes |
|---|---|---|
| Screens with mock-ups | 91 | 40 in the investor app and public site; 16 in the sponsor console; 35 in the admin console |
| User types and roles | 52 | 8 investor types, 4 sponsor roles, 11 licensed partners and providers, 26 RealX staff roles, 3 system principals (including a user's own agent) |
| Journeys | 22 | Screen flows with the compliance gate written on each arrow, then every step: who, screen, action, system response, gates, chain |
| Compliance gates | 22 | G01–G22, each with its rule, legal basis, where it is enforced and an evidence label |
| Admin modules | 27 | M01–M27 from the pack, adapted to RealX; M15 lending is not at launch |
| Maker-checker rows | 49 | Plus segregation of duties, break-glass, the key hierarchy and 17 alerts |
| State machines | 14 | Mermaid diagrams: the privileged action, onboarding, wholesale certificate, property, offering, order, token, holding, distribution, resale, governance, AML, complaint, recovery |
| BrickX parity | 26 + 6 | 26 BrickX features with RealX's equivalent and phase, plus 6 RealX-only features |

Each screen entry gives:
- its route and phase, and who can open it;
- what is on the screen;
- every action, with what happens, the gates it must pass, whether it touches the chain, and the maker and checker for admin actions;
- edge cases, the BrickX equivalent, and the journeys that pass through it.

Filters narrow screens by text, phase and role.

## Agent access (D04, version 2)

Every user type can give their own AI agent an API token that does what they can do in the app, under the same gates.

| Where | Screen | Who |
|---|---|---|
| Investor app | ACC-05 API tokens and agents | Every investor type |
| Sponsor console | SPN-16 API tokens and agents | Sponsor owner, operator, fund accountant, investor relations; valuers and property managers |
| Admin console | ADM-35 Agent and API access (M26) | Every staff role and admin-console partner user (own tokens); security and compliance (oversight, policy, kill switch) |

- **Gate G22 "Agent authority":** tokens are created with a passkey, scoped no wider than the person, limited, expiring and revocable. Every call is audited as "agent for <person>".
- **Staff agents** can prepare work but never approve, check or sign.
- **Stays with the person:** identity checks, the first wallet, creating tokens, bank-account changes, recovery, and (until counsel rules) personal declarations.
- **Journey J22 "Delegate to your own agent"** shows it end to end.
- **Generator exception:** a user's agent (AGT-U) acts through the API, so a J22 step may show it on a screen it isn't listed on. The screen names the equivalent action in the app.

## Where it comes from

- **Decisions:** [[Decisions]] (D01–D04, P01–P09).
- **Research notes:** [[Australia Notes]], [[Marketplace Notes]], [[Pack Map]].
- **Research pack:** [[australia]], [[01-brickx]], [[03-tokeniser-liquidise]], [[05-assetora-domacom-bricklet]], [[user-types-and-actions-matrix]], [[end-to-end-flows]], [[00-overview]], [[01-reference-admin-spec]], [[02-rbac-and-approvals]], [[03-workflows-and-state-machines]], [[05-wireframes]], [[07-regional-admin-variants]].
- **BrickX's public site, observed 23 Sep 2026:** the home page, the investments list and the SOM01 property page. These set the layout, palette and patterns: charcoal header band, coral accents, tile tabs, property cards.

## How to rebuild it

The page is generated. Its data and the generator live in `06 Evidence/Tools/product_map/`:
- `data_*.py`: roles, screens, journeys, gates, state machines and admin modules;
- `mocks.py`: the mock-screen blocks;
- `style.css` and `app.js`.

From the vault root:

```
py "06 Evidence/Tools/product_map/gen.py" "04 Architecture/RealX Product Map.html"
py "06 Evidence/Tools/product_map/gen.py" "04 Architecture/RealX-Product-Map.html" --standalone
```

The generator fails loudly on unknown screens, roles, gates, modules or mock blocks, and on duplicate IDs. It warns about any screen that no journey visits.

---
[[00 Start Here]] · [[Decisions]] · Research Log
