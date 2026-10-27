# Methodology

**Commissioned by:** Arjun (RealX project) · **Research date:** 23 Sep 2026 · **Researcher:** Claude (Cowork), orchestrating parallel research agents

## 1. Question

Study the top 5 fractional real-estate tokenisation platforms in each of **US, EU, MENA, Australia** (20 total). Reverse-engineer each platform's
compliance, user types, user actions, architecture, flows and stack, and extrapolate what the **platform-owner admin panel** looks like
(features, pages, actions).

## 2. Scope decisions & assumptions

1. **"Tokenisation" is interpreted strictly first.** Platforms that issue an on-chain token (or a regulator-recognised digital
   fraction) per property/share get priority. Where a region has too few mature on-chain players (expected for Australia and parts of MENA),
   the top-5 is filled with the leading **digital fractional** platforms (off-chain ledger) and labelled as such, because they define
   the local compliance and UX baseline a tokenised entrant must beat.
2. **Regions.** US = United States. EU = EU/EEA (Switzerland/UK noted only where relevant). MENA = GCC + wider Middle East & North Africa.
   Australia = Australia (NZ noted only if relevant).
3. **Ranking criteria** (applied per region, documented in each `_selection-and-summary.md`): tokenised/fractional volume or AUM,
   number of properties and investors, regulatory standing (licences, regulator-backed pilots), secondary-market activity, longevity,
   and relevance as of Sep 2026 (live, not wound down).
4. **Reverse engineering is evidence-graded.** Every claim carries **[C#]** (confirmed with source), **[I]** (inferred with reasoning),
   or **[S]** (speculative). Admin panels are never publicly visible, so they are **extrapolations** anchored on (a) the platform's
   observed user-facing features, (b) its regulatory obligations, and (c) publicly documented issuer dashboards of white-label
   tokenisation vendors (see `03-infra-and-admin-evidence/`).
5. **No circumvention.** When a site blocked fetching, it was logged as blocked and other sources were used.

## 3. Process

| Phase | Work | Output folder |
|---|---|---|
| 0 | Workspace, template, methodology | `00-method/` |
| 1a | 4 regional research agents in parallel (select top 5 → deep-dive each → regional summary → search log) | `02-platforms/<region>/` |
| 1b | Regulatory-landscape agent (4 regions + token standards law) | `01-regulatory/` |
| 1c | Infra & admin-evidence agent (white-label issuer dashboards, token standards, vendors) | `03-infra-and-admin-evidence/` |
| 2 | Cross-platform synthesis (matrices, reference architecture, flows, stack) | `04-synthesis/` |
| 3 | Admin-panel reference specification + per-archetype variants | `05-admin-panels/` |
| 4 | Verification pass, index, delivery | `README.md`, `00-method/action-log.md`, `00-method/verification-report.md` |

## 4. File conventions

- Every research agent keeps a `_search-log.md` with every WebSearch query and WebFetch URL, purpose, outcome, and which file used it.
- Platform files follow `platform-profile-template.md`.
- Mermaid is used for diagrams (renders on GitHub, Notion, Obsidian, VS Code).
