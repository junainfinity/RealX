---
title: "RealX Hub"
created: 2026-09-23
updated: 2026-09-23
status: active
tags: [realx, index]
aliases: ["Home", "Index", "Hub"]
---

# RealX Hub

**Start with [[R01 Research Pack Overview]], then [[Decisions]].** RealX is a fractional real-estate tokenisation marketplace. It is based out of Australia and lists other sponsors' residential and commercial property trusts. This vault records everything Claude searches, researches, understands, does and reports for RealX, from the first session on 23 September 2026.

## 1. Where things stand

| Item | Status |
|---|---|
| Home base | Australia (D01 in [[Decisions]]) |
| Supply model | Marketplace for other sponsors' property trusts (D02) |
| Asset types | Residential and commercial (D03) |
| Agent access | Every user type manages API tokens for their own AI agents (D04) |
| Open question | Build vs buy the token layer, asked in [[R04 Residential and Commercial]] |
| Code | None yet. `<workspace>` holds the research pack, the design and this vault ([[R09 Is the App Working]]) |
| Build model | D19 (30 Sep 2026): Opus 5.5 high plans, Grok 4.7 xhigh builds, Opus 5.5 high node QA, Fable 5.1 high graph QA. Plan reworked on 30 Sep ([[R10 D19 Model Roles Rework]]); spec re-checks still to run |

## Product

- **[[RealX Product Map]]:** every user type, journey, screen, action and compliance gate, plus the owner's admin panel, on one web page styled after BrickX. It has 91 screens, 22 journeys and 22 gates (version 2 adds D04 agent access).
  - Open from disk: `04 Architecture/RealX-Product-Map.html`.
  - Published (private): https://claude.ai/artifact/23baWULw9bDd4kCsbkeAfB
- **Product Map QA Report:** Fable 5.1's independent review: Pass with fixes, 8/10, 21 fixes applied, 11 open items.
- **[[App Build Plan]]:** a graph engineering plan with a gauntlet loop at every node: 107 nodes in 11 milestones, integrating the web app design. Under D19, Opus 5.5 (high) plans, Grok 4.7 (xhigh) builds, Opus 5.5 (high) QAs each loop, and Fable 5.1 (high) runs the graph QA at each milestone close.
  - Graph: [[App Development Graph]] and `09 App Build/RealX App Build.canvas`.
  - Independent review: App Plan QA Report, Pass with fixes, 7/10.
  - Spec re-checks are in progress.

- **Handoff - App Build Plan:** start here if you are a new agent picking up the work.

## 2. Reports (what Claude told the user)

- [[R01 Research Pack Overview]]: what the zip and the folder contain.
- [[R02 Australian Launch Path]]: what being Australia-based changes.
- [[R03 Marketplace Model]]: what the marketplace model changes.
- [[R04 Residential and Commercial]]: listing both property types, the vault set-up, and the product map while QA ran.
- [[R05 Product Map Published]]: the QA result, the published link and the open items.
- [[R06 Agent Access for Every User]]: D04, API tokens for every user's own AI agent, in the product map and the build plan.
- [[R07 App Build Plan Assembled]]: the plan, its numbers, the QA verdict and what is left.
- [[R09 Is the App Working]]: no app code exists yet; only the design prototype runs.
- [[R10 D19 Model Roles Rework]]: the graph plan reworked for the D19 roles, with a Fable 5.1 graph gate at each milestone close; 40/40 checks pass; spec re-checks still to run.
- [[R11 No View-As Switch]]: D20, no "view as" role switcher in any build; each user type signs up or signs in through its own front door and lands on its own home; 40/40 checks pass.

## 3. Research notes (what Claude understood)

- [[Pack Map]]: the pack's provenance, structure, size, reading order and headline findings.
- [[Australia Notes]]: Australian rules extracted from the pack, with citations, plus the questions collected for counsel.
- [[Marketplace Notes]]: marketplace precedents (Tokeniser, Assetora and others) and the build-vs-buy options.
- [[Pack Issues]]: inconsistencies found in the pack.

## 4. Decisions

- [[Decisions]]: the user's answers (D01–D04), the pack defaults Claude assumed (P01–P09) and the open question.

## 5. Audit trail

- Research Log: every action, in order.
- Search Log: every search, file read and command.

## 6. Source material

- Research Pack: a verified copy of `tokenization-research.zip` (67 files). Claude made it in Cowork on 22–23 Sep 2026, and Fable 5.1 reviewed it. The originals in `<workspace>` are untouched.
- Key pack files: [[00-executive-summary]] · [[implications-for-realx]] · [[australia]] · [[00-overview]] · qa-summary

---
Research, not legal advice. Australian licensing points need counsel.
