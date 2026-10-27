---
title: "App Build Plan"
created: 2026-09-24
updated: 2026-09-24
status: complete
tags: [realx, app-build, plan]
aliases: ["RealX App Build", "RealX Build Plan"]
---

# App Build Plan

**Build the RealX marketplace end to end: the public site and investor app, the sponsor console, the operator console, and a public API, MCP server and SDK so every user's own AI agent can do everything that user can do, under the same rules.** The build is a dependency graph of <!--g:nodes-->107<!--/g--> nodes (A00-<!--g:last_node-->A106<!--/g-->) and <!--g:items-->2<!--/g--> scheduled follow-up items (<!--g:item_ids-->A12.1, A61.1<!--/g-->) in <!--g:milestones-->11<!--/g--> milestones (AM0-AM10), with <!--g:edges-->797<!--/g--> dependency edges. <!--g:oracle_specs-->53<!--/g--> node specs are oracle nodes, where the node QA (Opus 5.5 high) writes hidden tests before the build, plus <!--g:oracle_items-->1<!--/g--> oracle follow-up item (<!--g:oracle_item_ids-->A12.1<!--/g-->). The risk core (register, client-money ledger, approvals, custody policy, reconciliation, agent tokens) is built and proven before any screen. A Docker Desktop preview of the investor app comes at the end of AM5. P1 ends with the whole platform running in Docker Desktop (A89). Every node goes through the [[App Gauntlet]]: **Opus 5.5 in ultracode mode builds it, and Fable 5.1 does the QA at every gauntlet loop of every node.** A node unlocks its dependents only when it passes. This plan is not legal advice. Licences, counsel opinions, vendor contracts, production keys, real money, mainnet, audits, the pentest and go-live all stay behind [[Human Gates]].

Every figure in this note that comes from the graph sits in a generated span, which `06 Evidence\Tools\app_plan\build_app_plan.py` rewrites from `Data\app-graph.json`; `check_plan.py` fails if any of them differs from the graph.

## 1. What the user decided

| Decision | Meaning for the build |
|---|---|
| D01 Based in Australia | Wholesale now (P1); a retail registered scheme from about 8 Apr 2027, when the Digital Assets Framework Act commences (P2); a continuous licensed market, AUD stablecoins, Auto Invest, minors and development or debt trusts later (P3). Every P2 and P3 feature is built behind a flag that only a licence or relief record can switch on (A21) |
| D02 Marketplace | RealX lists other sponsors' property trusts and never owns property. Sponsors issue units under their own AFSL; RealX arranges as a corporate authorised representative (CAR). This gives two admin tiers: the RealX operator console over a sponsor (tenant) console. The trustee's or RE's unit register (s169) is the legal record; the ERC-3643 token mirrors it; register, chain and bank are reconciled daily with zero tolerance, and a break halts resale |
| D03 Residential and commercial | Property type is a setting on each listing. Foreign-investor, duty and GST rules differ by type and are enforced by the rules engine at every order and transfer (G06 and others in A19) |
| D04 Agent access for every user type (23 Sep 2026) | Every investor sub-type, sponsor role, signed-in partner user and RealX staff role manages API tokens so their own AI agents can do all of their account actions through the API, with parity to the UI and under exactly the same gates (G01-G21 plus the new G22 "Agent authority"). The surfaces are a versioned REST API with OpenAPI docs, an MCP server, webhooks, a TypeScript SDK and a per-user sandbox. Tokens are made with a passkey step-up, shown once, hashed, scoped no wider than the person, limited, expiring, revocable and audited as "agent on behalf of person". Staff agents are never checker, approver or signer. Investor agents act on chain only through delegated session keys. The non-delegable acts stay with the person. See [[R06 Agent Access for Every User]] |
| Design hand-off | The Claude Design prototype `<workspace>\design\RealX App.dc.html` (run by `support.js`) is the UI source of truth: cream #F5F2EC, ink #17191C, green #1F5E4B, brick #9A4524, Newsreader, Geist and Geist Mono. It supersedes the coral BrickX-style look of the product-map document. It draws 38 of the product-map screens fully and 8 in part; AM4 draws the rest, the missing parts and the drawn-screen supplements in the prototype's own language for the user to review first. The design folder is never edited; A00 records its SHA-256 |
| Build model | Decision D19 (30 Sep 2026). Opus 5.5 (high) plans every node. Grok 4.7 (xhigh) builds it. Opus 5.5 (high) does the node QA at every gauntlet loop in a fresh session: it writes the hidden oracles first for oracle nodes, then verifies, challenges and gates. Fable 5.1 (high) runs the graph QA when all of a milestone's nodes have passed |

Q12 (build or buy the token layer) is still the user's open question. The plan assumes option 1 as a proposed, reversible engineering decision: the open-standard ERC-3643 T-REX reference implementation, with its GPL-3.0 licence referred to counsel, plus rented services behind sandbox adapters. The swap points are explicit (A00 records them; A08 builds the adapters). The engineering decisions D05-D16 are in [[Decisions]].

## 2. How this plan was produced and checked

- **Fable 5.1 (high effort) wrote the plan's core** on 23-24 Sep 2026: the 106-node graph and 47 node specs (A00-A45 and A47). Its weekly usage limit then ran out on 24 Sep.
- **Opus 5.5 wrote the other 59 node specs** (A46 and A48-A105) and assembled the plan documents: this note, [[App Gauntlet]], [[App Development Graph]], [[Build Orchestration]], [[Screen Coverage Matrix]], [[Human Gates]] and the engineering decisions in [[Decisions]].
- **An independent Opus 5.5 reviewer checks every node spec and the whole plan.** It runs in ultracode as a third party with no stake in the plan. The founder chose this on 24 Sep 2026 instead of waiting for Fable's limit to reset. It replaced the Fable check for the plan only.
- **The build roles changed on 30 Sep 2026 (decision D19).** See the Build model row above. The remaining spec re-checks are run by Opus 5.5 (high) per spec, and Fable 5.1 (high) reviews the plan as a whole.
- **Where the checking stands.** When the check run started, every node spec had had one independent check followed by a fix pass. The first checks raised 1,696 findings: 14 critical, 585 high, 771 medium and 326 low. Six specs were re-checked before the usage limit stopped the check run. One (A05) passed its re-check. The other five (A00, A01, A07, A08, A16) raised new findings. **So <!--g:checks.passed-->1<!--/g--> spec has passed a re-check, and <!--g:checks.awaiting-->105<!--/g--> still await their re-check.** Read them as fixed once but not yet re-confirmed. The plan-level review then raised findings on the plan as a whole; the plan fixer applied them on 24 Sep 2026, and a second plan-level review the same day raised 22 more (1 critical, 6 high, 15 medium), which the plan fixer also applied that day; both passes are recorded in the App Plan QA Report. That pass added one node, **A106 (IAM admin API for ADM-37), which has had no independent check yet**, and two scheduled follow-up items (A12.1 and A61.1, both specified inside their owner's note). The build enforces the check status: a spec without a passed re-check does not start unless the user accepts it by name ([[Build Orchestration]] §2).
- **Records.** Every check is a JSON file in `09 App Build\Data\checks\` (`<node>-check1.json`, `<node>-check2.json`), and the plan-level review is in the App Plan QA Report.
- **The specs and the graph agree.** `Data\app-graph.json` is generated from the 23 Sep graph data plus every change the specs ask for: their estimates, oracle flags, milestones, titles, human gates, dependencies and owned paths are read from the notes themselves, and the remaining requests are listed with their sources in `Data\graph-revisions.json`. The checker fails if a note and the graph disagree.

## 3. What gets built

| Surface | Scope |
|---|---|
| Public site and investor app (`apps/web`, public and investor route groups) | The 40 investor-surface screens of the [[RealX Product Map]] (33 P1, 6 P2, 1 P3): home, investments, property page with proof of property, sponsor profiles, fees, how it works, returns calculator, transparency, help and documents; onboarding ONB-01 to ONB-10 with the joint and entity branches; buy flow; portfolio, holdings, transactions, distributions and statements; AUD wallet and token wallet; account; and **ACC-05 "API tokens and agents"** at `/account/agents` (A60) |
| Sponsor console (`apps/web`, sponsor route group) | All 16 sponsor screens: apply, dashboard, team and approvals, billing, the four-step listing wizard SPN-03 to SPN-06, offer monitor, register, distributions, property reporting, valuations, announcements, and **SPN-16 "API tokens and agents"**, which valuers and property managers also use. The partner portal for the trustee or RE, auditor, regulator observer and licensee sits beside it (A105) |
| Operator console (`apps/console`, own origin, hardware-key sign-in) | All 35 ADM screens across the 27 modules M01-M27 (M15 lending is excluded as P3 and outside the model), plus the five screens the product map never drew, numbered ADM-36 to ADM-40, with the IAM admin API for ADM-37 as its own node (A106). Every money, supply or holdings action goes through maker-checker with one approval ID. Includes **ADM-35 "Agent and API access"**: staff's own tokens plus oversight of every token across users and tenants, anomalies and kill switches (A77) |
| Public API, MCP server and SDK for users' own agents | Versioned REST with an OpenAPI document generated from the same schemas the UI uses (A09), the token service and G22 evaluator (A12), the MCP server generated from the OpenAPI document (A104), webhooks (A46), the TypeScript SDK, reference docs and a per-user sandbox (A99), the agent guard with anomaly detection, sensitive-action notices and kill switches per token, user, tenant and platform (A101, A48), the agent-parity suite that fails the build when any UI action lacks an API and MCP equivalent under the same gates (A100), and an end-to-end journey where an agent drives an account (A82, journey J22) |
| Services (`services/api`, `services/worker`) | Audit hash chain, auth and RBAC with segregation of duties, IAM administration, approvals engine, custody policy sandbox, double-entry client-money ledger, NPP/PayID/PayTo bank-rail sandbox, the legal unit register with the registry adapter, rules engine for G01-G22, three-way reconciliation, licences and flags, KYC and KYB, classification, tenancy, properties and valuations, documents and disclosures, offerings, orders and escrow, finalise-and-mint, treasury, distributions, tax, AML, communications, complaints, reporting, jobs, read models, halts and wind-down, partner principals |
| Contracts (`contracts`) | T-REX ERC-3643 suite vendored with RealX compliance modules (A24), the AgentGateway and owner multisig that bind approval IDs on chain (A25), the passkey smart wallet with paymaster, guardian recovery and the session-key module for agents (A26), all on a local Hardhat chain and Base Sepolia testnet only |
| Design system (`packages/ui`) | Tokens, fonts and components generated from the prototype (A05), and two design-extension files for the undrawn and partly drawn screens and the drawn-screen supplements (A50 investor and sponsor, A51 operator), reviewed by the user before any UI node uses them |
| Docker Desktop | The whole stack boots from Docker Desktop at the end of AM1 (A23), a money-path walk-through at the end of AM3 (A49), the investor-app preview at the end of AM5 (A61, re-run in full as A61.1), and the P1 launch in Docker Desktop at the end of AM8 (A89) |
| Later (AM9, AM10) | P2: registered scheme with sub-funds, PDS and TMD, target-market gate G05, retail cooling-off as a parameter, the guarded resale board and governance votes. P3: Auto Invest with PayTo, the AUD stablecoin rail with the travel rule, minors, development and debt vehicles, licensed-market venue mode |

**Stack.** One npm-workspaces monorepo at `<workspace>\app` (A01 runs `git init`; the folder is not a git repo yet). Strict TypeScript everywhere. Two Next.js apps with React 19 (`apps/web`, `apps/console`). Fastify 5 with zod 4 schemas that generate the OpenAPI document, the SDK and the MCP tool catalogue. PostgreSQL 17 in Docker with row-level security for tenants, Drizzle ORM, and pg-boss for jobs. SimpleWebAuthn for passkeys and hardware keys. viem for the chain. Solidity on Hardhat with T-REX vendored. Vitest, Playwright, pino and OpenTelemetry. Everything runs in Docker Compose. The build machine has Windows 11, Node 24.21.0, Docker 29.8.0, Git 2.55 and Python 3.13 (`py`); A01 pins every version after a real install. Hidden oracles live outside the repo at `<workspace>\gauntlet\hidden\<node>\`.

## 4. Milestones

The table is generated from `Data\app-graph.json`: the ids in each milestone (the run list the orchestration uses), the oracle count, the summed estimates, the follow-up loop allowance, and when the milestone is done if each milestone starts only after the previous one ends.

<!-- gen:milestone-table -->
| Milestone | Nodes and items | Oracle | Work (agent-hours) | Loop allowance (h) | Done by on 3 streams, one milestone at a time | Exit |
|---|---|---|---|---|---|---|
| AM0 Foundations | A00, A01, A02, A03, A04, A05, A07, A08, A09 (9) | 5 | 69-105 | 2-6 (2) | 47-71 | none (A05, A07, A08 and A09 close it) |
| AM1 Money, register, approvals and agent-access core | A06, A10, A11, A12, A12.1, A13, A14, A15, A16, A17, A18, A19, A20, A21, A22, A23, A104 (16 + 1 item) | 12 | 167-237 | 10-30 (10) | 164-236 | A23 |
| AM2 Token and wallet core | A24, A25, A26, A27, A28, A29, A50 (7) | 6 | 72-100 | 0-0 (0) | 216-308 | none (A28, A29 and A50 close it; A50 holds its dependents until the user's HG-REVIEW, which may be given while AM3 runs) |
| AM3 Business services | A30, A31, A32, A33, A34, A35, A36, A37, A38, A39, A40, A41, A42, A43, A44, A45, A46, A47, A48, A49, A98, A101, A106 (23) | 17 | 235-337 | 31-93 (31) | 338-495 | A49 |
| AM4 Design extension, shells and developer surface | A51, A52, A53, A99 (4) | 2 | 52-76 | 9-27 (9) | 367-540 | none (A51 to A53 and A99 close it) |
| AM5 Investor app and preview | A54, A55, A56, A57, A58, A59, A60, A61, A61.1 (8 + 1 item) | 1 | 75-111 | 7-21 (7) | 405-599 | A61.1 |
| AM6 Sponsor console | A62, A63, A64, A65, A105 (5) | 1 | 50-73 | 5-15 (5) | 436-647 | A65 |
| AM7 Operator console | A66, A67, A68, A69, A70, A71, A72, A73, A74, A75, A76, A77, A78 (13) | 2 | 112-164 | 30-90 (30) | 496-750 | A78 |
| AM8 Journeys, hardening and launch | A79, A80, A81, A82, A83, A84, A85, A86, A87, A88, A89, A100, A102, A103 (14) | 2 | 156-240 | 10-30 (10) | 578-880 | A89 |
| AM9 P2 features (from about 8 April 2027) | A90, A91, A92, A93, A94 (5) | 3 | 69-97 | 11-33 (11) | 642-973 | A94 |
| AM10 P3 later | A95, A96, A97 (3) | 3 | 48-68 | 25-75 (25) | 681-1,040 | none (A96 and A97 close it) |
| **All** | **107 nodes + 2 items** | **54** | **1,105-1,608** | **140-420 (140)** | **681-1,040** | |
<!-- /gen:milestone-table -->

What each milestone is for:

- **AM0 Foundations.** Charter and coverage baseline, repo and toolchain, domain package (integer money, unit math, 14 state machines, gate codes, permission matrix), database with RLS, design system, local chain with T-REX, sandbox adapter framework, API and worker skeleton with the generated SDK. The seed simulator (A06) moved to AM1 because it seeds rows from A10, A11 and A12.
- **AM1 Money, register, approvals and agent-access core.** Audit chain, auth and RBAC, **API tokens and G22 (A12, with its follow-up loop A12.1)**, **MCP server (A104)**, seed, alerts, approvals engine, custody policy, ledger, bank rail, unit register, rules engine, reconciliation, licences, identity sandboxes. Exit: the whole stack boots from Docker Desktop three times in a row (A23).
- **AM2 Token and wallet core.** ERC-3643 suite, AgentGateway, passkey smart wallet with session keys, chain adapter and indexer, token operations, wallet service. A mint bound to an approval ID runs end to end and a replayed ID is refused at every layer. Design extension A (A50: the undrawn investor, public, sponsor and partner screens, including ACC-05, SPN-16 and the PUB-05 standalone page, and their drawn-screen supplements) also runs here, moved from AM4 on 24 Sep 2026, so that its build uses idle streams and the user's review (HG-REVIEW) can happen while the backend is built instead of on the P1 path.
- **AM3 Business services.** Every API module the screens need, partner principals, the IAM admin API (A106) and the agent guard. Exit: the money-path smoke through the SDK (A49), with a founder walk-through.
- **AM4 Design extension, shells and developer surface.** Design extension B for the operator console (A51: ADM-35, ADM-36 to ADM-40 and the console's drawn-screen supplements) in the prototype's language, then the web and console shells, and the developer surface (A99, moved from AM3 so that A49 closes AM3). The user reviews and approves extension B here, and extension A's approval from AM2 must be recorded before the web shell and any investor or sponsor screen node starts (HG-REVIEW).
- **AM5 Investor app and preview.** Public site, onboarding, buy flow, portfolio, wallet, account, the token screens ACC-05 and SPN-16, then the Docker Desktop preview the user walks (A61), re-run in full once every AM5 screen node has passed (A61.1, the exit).
- **AM6 Sponsor console.** Sponsor console and partner portal. Exit: sponsor journeys J11-J14 and J19 end to end in Docker (A65).
- **AM7 Operator console.** The 40 operator screens, including ADM-35. Exit: the operator daily run J20 end to end (A78).
- **AM8 Journeys, hardening and launch.** Journeys with planted breaks, the agent journey J22 (A82) and the agent-parity suite (A100), security, performance, observability, runbook drills, backups, the Base Sepolia run and audit pack, design fidelity, three clean regression runs of all 18 P1 journeys with every gate G01-G22 exercised (A88), and the Docker Desktop launch (A89).
- **AM9 P2 (from about 8 Apr 2027).** Registered scheme, sub-funds, PDS and TMD, resale board, votes, P2 journeys; switched on only by a licence or relief record.
- **AM10 P3 (later).** Auto Invest, stablecoin rail, minors, development and debt vehicles, venue mode; behind P3 flags after the advice and licensing reviews.

The node list, dependencies and schedule are in the [[App Development Graph]] and on the [[RealX App Build.canvas|app build canvas]]. Coverage of the 91 screens, 22 journeys, 22 gates, 27 modules and 14 state machines is in the [[Screen Coverage Matrix]].

## 5. The agent-access thread (D04) through the graph

D04 is built as a core capability, not a feature added at the end:

- **Data and rules first.** A00 freezes the non-delegable list, the counsel questions and the partner principal types. A03 turns scopes into cells of the role x module x action matrix, so a token can never exceed its owner, and holds the one list of refusal codes (for example `SCOPE_NON_DELEGABLE`, `SCOPE_PENDING_COUNSEL`, `SCOPE_AGENT_NEVER_CHECKER`), which A01's verify enforces.
- **Core in AM1.** A11 adds the agent principal. A12 builds the token service, mandates and the G22 evaluator, and every later route inherits token auth; its loop A12.1 adds the approval-backed routes once A14 exists. A104 generates the MCP server from the OpenAPI document. A14, A15 and A25 refuse an agent as checker, approver or signer on the server and on chain.
- **Chain in AM2.** A26 and A29 give investor agents delegated session keys, limited by contract, function, amount and expiry, and never the owner key.
- **Services in AM3.** A46 adds webhooks, A98 adds partner principals, A101 adds anomaly detection, notices and kill switches, and A106 lets staff agents draft access reviews but never decide them.
- **Developer surface in AM4.** A99 adds docs, versioning and the per-user sandbox.
- **Screens.** A50 and A51 draw them, A60 builds ACC-05 and SPN-16, and A77 builds ADM-35.
- **Proof in AM8.** A82 runs journey J22 through REST and MCP. A83 attacks the tokens. A100 proves parity for every UI action. A88 runs J22 inside its three clean runs and requires G22 to be exercised both as a pass and as a refusal.

## 6. Time estimate

These are planning figures in agent-hours of build plus QA loops (Grok 4.7 build, Opus 5.5 planning and node QA; the Fable 5.1 graph gates are not yet in the figures), generated from `Data\app-graph.json`. The orchestration runs one milestone at a time on <!--g:streams-->3<!--/g--> streams (A01's limit), so that is the schedule quoted here.

- **Total work:** <!--g:total-->1,105-1,608<!--/g--> agent-hours across all nodes and items, of which P1 (AM0-AM8) is <!--g:total_p1-->988-1,443<!--/g-->, plus a follow-up loop allowance of <!--g:loops_hours-->140-420<!--/g--> for the <!--g:loops-->140<!--/g--> follow-up loop runs the specs ask for without an estimate (one run per loop id and requester, because several specs ask the same loop id for different changes), so <!--g:total_with_loops-->1,245-2,028<!--/g--> in all.
- **Critical path:** <!--g:cp_p1-->390-566<!--/g--> agent-hours to the P1 launch (A89), and <!--g:cp-->424-616<!--/g--> through the whole graph to the P3 node A96. Critical paths ignore the stream limit and the milestone barriers, so they are floors.
- **The Docker lock is in the schedule.** The orchestration runs one stack-bound Challenge or re-verify at a time across the streams, so the schedules run the last <!--g:docker_share-->35%<!--/g--> of each Docker-using node's estimate one node at a time (a planning assumption, re-measured after AM0).
- **One milestone at a time on three streams:**
  - the whole stack boots in Docker (AM1 exit, A23) at about <!--g:b3.AM1-->164-236<!--/g--> agent-hours;
  - the money-path smoke closes AM3 (A49) at about <!--g:b3.AM3-->338-495<!--/g-->;
  - the shells, extension B and the developer surface close AM4 at about <!--g:b3.AM4-->367-540<!--/g-->;
  - the investor preview closes AM5 (A61.1) at about <!--g:b3.AM5-->405-599<!--/g-->;
  - the P1 launch in Docker Desktop (AM8, A89) at about <!--g:b3.AM8-->578-880<!--/g-->;
  - the whole graph at about <!--g:b3.all-->681-1,040<!--/g-->.
- **For comparison:** four streams would bring the P1 launch to about <!--g:b4.AM8-->576-871<!--/g--> (it needs an A01 change first), and letting milestones overlap would bring it to about <!--g:ov.AM8-->490-692<!--/g--> (the orchestration does not do this, because the user reviews each milestone before the next starts).
- **Not in agent-hours:**
  - wall-clock waits: the Base Sepolia run in A86 takes about five days of timelocks (the node is parked between sessions), and the real-time windows of the drills in A103;
  - the user's reviews (A50 and A51, A61, A87, A89);
  - usage-limit pauses.

These are assumptions, not measurements. They are re-estimated after AM0 from real gate times and real usage.

## 7. Usage and pacing

Planning alone used a full weekly Fable 5.1 allowance: the graph plus 47 specs. The build is far larger. It is about <!--g:total_with_loops-->1,245-2,028<!--/g--> agent-hours of work. Opus 5.5 plans every node and does the node QA at every loop, Grok 4.7 builds, and Fable 5.1 runs once or twice per milestone at the graph gate, with hidden oracles written first for <!--g:oracle_specs-->53<!--/g--> oracle nodes and <!--g:oracle_items-->1<!--/g--> oracle follow-up item. So the build needs a usage budget and pacing per milestone:

- **Measure first.** AM0 records planner, builder, node-QA and graph-QA usage per node and per milestone in the evidence manifests and the run journal. The orchestrator then projects each later milestone's usage before starting it.
- **Budget per milestone.** Start a milestone only when the remaining allowance covers its projected usage plus a reserve for fix loops and escalations. The oracle-heavy milestones need the most Opus 5.5 node-QA time: AM3 has <!--g:ms.AM3.oracle-->17<!--/g--> oracle nodes, AM1 has <!--g:ms.AM1.oracle-->12<!--/g--> (including A12.1) and AM2 has <!--g:ms.AM2.oracle-->6<!--/g-->.
- **Stop at clean boundaries.** When the allowance runs low, finish the step in hand and stop. Do not stop in the middle of a gate. A stopped run resumes each node from its last passed step ([[Build Orchestration]] §5), so a pause costs time, not work.
- **Report usage** to the user with each milestone report, alongside the re-estimate.

## 8. How the build runs

1. **One run per milestone** ([[Build Orchestration]]), started through the Cursor SDK runner so each role gets its own model (decision D19). The orchestrator schedules ready nodes over three parallel build streams (A01's limit), each with disjoint owned paths, and starts the next milestone only after the previous milestone's graph gate has passed and the user has seen the report.
2. **Every node runs the [[App Gauntlet]].**
   - *Specify:* the planner, Opus 5.5 (high), freezes the node spec with its design references; its check record and the four role models' names go into the manifest.
   - *Oracle:* for the oracle nodes, the node QA, Opus 5.5 (high) in a session separate from the planner, writes hidden tests first, into `gauntlet\hidden\<node>\`, which the planner and builder never read (a scan of their transcripts checks this).
   - *Plan:* the planner splits the node into build briefs on disjoint files.
   - *Build:* Grok 4.7 (xhigh).
   - *Challenge:* the node QA, Opus 5.5 (high), in a fresh session at every loop. It verifies, runs the hidden tests and the node's challenges, and checks design fidelity and the compliance gates.
   - *Gate:* a pass unlocks the node's dependents, unless the node's human gate holds them until the user signs off.
   - *Fix loop:* at most three. After that the node escalates: split it, re-specify it or ask the user.
   - *Graph gate:* once every node of the milestone has passed, Fable 5.1 (high) tests the milestone as a whole: cross-node contracts, journeys, every published oracle on `main`, coverage and evidence. A fail reopens the owner nodes (counted against their fix loops); after two failed rounds the milestone escalates.
3. **Evidence and resume.** The builder writes `app/evidence/<node>.json`, the node QA writes `<node>.qa.md`, the graph QA writes `graph-<milestone>.qa.md`, and the orchestrator keeps `status.json`. After a usage-limit pause or any interruption, each node resumes from its last passed step. A node that must wait for real time or a human step is parked, not failed. There is one local git commit per passed node.
4. **After each milestone** the orchestrator:
   - updates statuses;
   - re-estimates time and usage;
   - lists the human steps now due;
   - reports to the user before the next milestone starts.

## 9. Human gates this build does not cross

Agents build and test locally, on a local chain and on Base Sepolia testnet, with sandbox adapters and keys that carry no value. They do not:
- obtain licences or counsel opinions;
- sign vendor contracts;
- create or hold production keys;
- move real money;
- deploy to mainnet or production;
- commission audits or the pentest;
- go live.

Each of these steps is in [[Human Gates]]: HG-LEGAL, HG-LICENCE, HG-ACCOUNTS, HG-KEYS, HG-MONEY, HG-CHAIN, HG-PROD, HG-AUDIT (two smart-contract audits), HG-PENTEST (including agent-token abuse), HG-DATA, HG-DESIGN and HG-LAUNCH, plus the standing human roles. That note also maps the older gate names used in the node specs (HG-VENDOR, HG-CAPITAL, HG-REVIEW) to these names.

The user's own steps during the build are the design reviews (A50's during AM2 or AM3, which holds its dependents until approved; A51's at AM4, which does not pass until approved), the preview walk that closes AM5 (A61), the fidelity walk at A87 and the P1 software sign-off that closes AM8 (A89). Base Sepolia needs a human-created bundler account and faucet funding before A86; until then A86 is parked.

## 10. Relation to the product map and research

- **Requirements.** The [[RealX Product Map]] is the requirements source. Its generator data in `06 Evidence\Tools\product_map\` holds the screens, components, actions, states and gates. Every one of its 91 screens, 22 journeys, 22 gates, 27 modules (M15 excluded with a reason), 14 state machines and 52 roles maps to a node in the [[Screen Coverage Matrix]]. The canonical id list is `Data\requirements-lists.json` (SHA-256 pinned by A00). The open items in the Product Map QA Report are handled as follows: the five undrawn admin screens become ADM-36 to ADM-40, and the design's investor codes are reconciled to product-map codes in A00's canonical screen table (for example design INV-01 is POR-01).
- **Look and feel.** The design prototype, not the product map, decides this.
- **Architecture.** The research pack (`06 Evidence\Research Pack\`, especially the reference architecture, the tech-stack comparison, the admin-panel studies, the token-standards and vendor-landscape notes, and the Australian regulatory notes) informed the architecture. The pack is read, never edited.
- **Founder and research context.** The decisions are in [[Decisions]], the Australian path in [[R02 Australian Launch Path]] and [[Australia Notes]], the marketplace model in [[R03 Marketplace Model]] and [[Marketplace Notes]], and D04 in [[R06 Agent Access for Every User]].

---
[[00 Start Here|RealX Hub]] · [[App Development Graph]] · [[App Gauntlet]] · [[Build Orchestration]] · [[Screen Coverage Matrix]] · [[Human Gates]] · [[Decisions]] · App Plan QA Report
