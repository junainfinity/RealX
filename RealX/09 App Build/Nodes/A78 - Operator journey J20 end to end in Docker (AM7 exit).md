---
title: "A78 - Operator journey J20 end to end in Docker (AM7 exit)"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am7]
node: A78
milestone: AM7
lane: "Quality"
oracle: no
human_gate: none
---

# A78 - Operator journey J20 end to end in Docker (AM7 exit)

**The AM7 exit check. It is a Playwright suite over the Docker preview stack plus A53's console. It walks J20, the operator daily run, through the built console screens on Wednesday 23 September 2026, starting at A06's `AS_OF` instant (10:30 AEST). A different seeded person takes each role: KYCA, MLRO, CFO, TOKO, REGO, CCO, SECO and ENGO, plus the LEGL, PART and TAXO owners of the other J20 steps. The run covers these checks in order. Every ADM-01 tile is checked against the persona's cells, both on screen and at the API. The alerts centre is worked against its SLA clocks. Three approvals in the inbox are decided with hardware-key assertions, and the maker, the maker's agent, a sibling agent and a maker who holds the checker role are each refused by name. One custody operation (OP-7730, the Unley Road mint) is signed 2-of-3 on the console by two eligible V2 signers, broadcast, confirmed and reconciled to `closed`. The register break on Gympie Road Retail is closed by the approved adjustment and a clean re-run. The audit hash chain is verified at four checkpoints. The launch flags (retail, resale, votes, stablecoins) are confirmed off, and a toggle without a licence binding is refused. A staff token is requested on ADM-35 under a hardware key, approved by a SECO, used through MCP, and revoked; its next call is refused. Three clock probes (approval TTL, KYC SLA, OP-7731 timelock) run as separate sub-runs after a reset. The node then re-runs A65's five sponsor journeys (J11, J12, J13, J14, J19) with every RealX-side step taken through the console screens instead of the SDK driver. The designed admin screens are captured beside the prototype. The node adds no product behaviour. It proves that the 27 console modules work together as one operator's morning.**

**Build status:** not started. **Milestone:** AM7 (Operator console). **Lane:** Quality. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 6-10 agent-hours (build, challenge and verify loops).

**Depends on:** [[A66 - Console- command center, alerts centre, approvals inbox, audit log]], [[A67 - Console- users, roles, access reviews and break-glass (ADM-37)]], [[A68 - Console- identity review, wholesale certificates, investor directory and Investor 360]], [[A69 - Console- sponsor admission, licences, sponsors and guardrails, feature flags, entities register]], [[A70 - Console- property register, valuations workbench, property operations]], [[A71 - Console- listing review, live raise monitor, orders and allocation]], [[A72 - Console- token operations, forced transfer and recovery desk, wallets and custody, token registry]], [[A73 - Console- rule packs and threshold monitors, register and reconciliation]], [[A74 - Console- treasury, distribution run review, tax centre]], [[A75 - Console- AML cases, complaints register, marketing compliance, disclosure library]], [[A76 - Console- regulatory calendar and data room, sponsor performance, integrations and jobs]], [[A77 - Console- agent and API access (ADM-35)]], [[A65 - Sponsor journeys J11, J12, J13, J14, J19 end to end in Docker (AM6 exit)]]
**Unlocks:** [[A80 - End-to-end journeys- sponsor and operations]], [[A82 - End-to-end- an agent drives the account through the API and MCP]], [[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates]]

**Oracle rule:** this node has no hidden oracle (`oracle: no`). Every decision, signature, balance and gate verdict in the run belongs to a dependency that has its own oracle or hidden tests: A14 approvals, A15 custody, A20 reconciliation, A10 audit chain, A12 agent authority, A13 alerts, A21 perimeter, and A28 token operations. This node proves that those pieces and the 27 console modules on top of them work as one operator's day. The node QA still writes Challenge material to `<workspace>\gauntlet\hidden\A78\` from this note alone, before the build starts. It holds:
- an independent tile-visibility table for every persona, derived from A03's permission matrix data and A06's role assignments;
- its own re-derivation of every figure in the checkpoint table below from A06's seed manifest for each seed (integer cents and units);
- the SoD refusal matrix for the three approvals, the other requests, the custody operation and the staff token;
- the reference region list for the three gated prototype captures, with their dc line ranges;
- the expected outcomes of each clock sub-run (TTL, KYC SLA, OP-7731 timelock) under both outcomes of A03's extension (c);
- a disposable-database tamper script for the chain-verification probe;
- the "green by driving" scans (SDK operator calls, `/__test/*` calls outside the allowlist, route interception) and the expected result of each.

The builder never reads that folder. The material is published into `app/` only after the gate passes ([[App Gauntlet]] §2).

Everything runs on A23's local stack with the sandbox adapters, the local Hardhat chain, A15's no-value keystore and A11's simulated hardware authenticator. No vendor, testnet, real account or real person is involved. This note is not legal advice. The thresholds, SLAs, caps and meter limits it asserts are the configuration and rule-pack values the owning nodes ship ("set with counsel" rows). The suite reads them and never types a statutory figure of its own.

## Build brief

This node writes no product behaviour, no product screen and no API. It owns a Playwright and Vitest suite, its own page objects, the `console` implementation of A65's operator-step interface, a tile catalogue and expected-tiles generator, a screenshot comparer configuration, an evidence writer and a one-page guide, all under `<workspace>\app\e2e\operator\`.

Browser steps go through the console origin (`CONSOLE_ORIGIN`, default `http://console.localhost:3001`, A53) and, for the sponsor side (the trustee seat on APR-2292 and the sponsor re-run), through `apps/web` on `http://localhost:3000`. Reads the suite uses to check what a screen shows go through `packages/sdk` (A09) as the same persona. Agent steps go through `packages/mcp` (A104) and, for the revocation check, the API's bearer route. Nothing in `e2e/operator/` imports from `services/*`, `packages/db`, an adapter or any `apps/console` or `apps/web` source file. The one exception is A66's committed page objects (`apps/console/src/features/*/testing/page-objects.ts`), imported read-only; every other console screen's page object is written by this node in `e2e/operator/pages/`, using accessible roles and names and the screen code A53 renders, never CSS class names. A lint in `npm run verify` fails on any other import.

Human acts go through A11's simulated hardware authenticator. These are staff sign-in, every approve, reject, ratify and sign, and token issue. The authenticator gives the sandbox attestation format that A53 accepts while the shell shows `SANDBOX`, reached through the `/__test/authenticator` bridge that exists only in `RUN_MODE=local|test`. So every decision in the run carries a real WebAuthn assertion over the request's payload hash, exactly as a person's would.

**1. Stack, commands and clock**
- The stack is A23's `infra/stack/` plus A61's `infra/preview/` overlay, plus A53's console compose file as A53 registers it. This node adds no service and edits none of those files. A missing `console` service is `CONSOLE_NOT_UP` and a fix item on A53, not a patch here. The two API-facing processes the run can reach are A23's `api` container (`http://localhost:4000`) and A104's `mcp` container (`http://localhost:4010/mcp`). A23 runs one `api` replica; revocation across several API replicas is A12's and A103's property, not this node's.
- Commands, registered under A01's script-registration convention. All are TypeScript run with `tsx`, work in PowerShell and Git Bash, and use LF line endings.
  - `operator:journeys [--seed <s>]`: the main run (J20 plus the negatives in §2 to §9), then the three clock sub-runs of §6.
  - `operator:sponsor-rerun`: A65's journeys with `OPERATOR_STEPS=console`, §10.
  - `operator:shots`: the captures in §11.
  - `operator:evidence`: rebuilds `app/evidence/A78.json` from the last run.
  - `verify:operator`: `preview:reset`, `preview:up`, then `operator:journeys` and `operator:sponsor-rerun` three times with three different `--seed` strings (the first is A06's default `realx-baseline`, with a reset between runs), then `operator:shots` on `realx-baseline`, then `preview:down`. It needs Docker Desktop, and it is what the gauntlet runs.
  - `npm run verify` stays Docker-free. It runs only this node's static checks: the import lint, the `/__test/*` allowlist lint, the schema of the expected-step and timeline tables, the schema of the tile catalogue and the expected-tiles file, and the guide link check.
- **Playwright configuration.** `workers: 1` and `fullyParallel: false`. The spec files share one database and one platform clock, so they run as Playwright projects chained by `dependencies` in the order of the timeline table (§3). A sub-run starts only after `preview:reset` with the same `--seed`.
- **Test-route allowlist.** Only four internal routes are allowed: clock read and advance (A09 `DbClock` through the sandbox panel control), job run-now (A46), the simulated authenticator bridge (A11), and A08's `SandboxControl.trigger` for named scenarios. One more is allowed and nothing else: A76's `Acknowledge (test)` button on ADM-26, because A76 ships it for this node. The lint fails the build on any other `/__test/*` or `/v1/_test/*` call. The run never writes to the database, never calls an adapter directly, and never intercepts or mocks a network route (`page.route` and service workers are banned by the lint).
- **Clock.** Each run starts from A06's baseline at `AS_OF`, Wednesday 23 September 2026 10:30 Australia/Sydney (AEST). That is after the 06:00 reconciliation run, the 06:02 break and A13's 07:00 digest, and after every seeded event of the morning (APR-2291 at 08:47, APR-2278 at 09:02), so the queues, clocks, ages and digest all exist and the clock never has to move backward. It moves forward only through the sandbox panel control, by the amounts in the timeline table, and never past 26 September 2026. That keeps KST01 (closes 14 Oct), UNL01 and every other seeded offer in the state A06 recorded. The sponsor re-run in §10 uses its own clock advances from A65 after a reset. Every time on screen must come from A09's clock. A browser clock skewed by Playwright must change nothing.
- **Tooling errors and empty states:**
  - `STACK_NOT_UP` and `CONSOLE_NOT_UP` are raised within 10 seconds.
  - `SEED_BASELINE_MISMATCH` names the first A06 locked invariant that differs from A06's seed manifest for the seed in use (for example the approvals count or the reconciliation rows).
  - `PERSONA_MISSING` names a role in the persona table with no seeded person.
  - `TRUSTEE_SURFACE_MISSING` names the node that must carry the trustee countersign action (spec point (e)).
  - `DESIGN_FOLDER_CHANGED` comes from A61's hash check, reused.
  - A tile in the "Not connected · owned by Axx" state (A66) fails the run with `TILE_NOT_CONNECTED` and the owning node.

**2. Personas and the morning's cast**
- The suite selects A06's personas by id and records them in the evidence. §10 reuses A65's `e2e/sponsor/personas.ts` read-only, so the sponsor re-run and this run share one persona set.

  | Persona (A06) | Role | Part in this run |
  |---|---|---|
  | Jordan Blake | CCO | Steps 1, 3, 7, 11 and 12; checker on APR-2292; maker of APR-2285 (refused as its checker); approves the disclosure publication; maker of the data-room grant |
  | Sam Reilly | LEGL | Step 8, drafting the disclosure update; checker on APR-2285 (refused on the prerequisite, then rejects it) |
  | Ben Cole | ENGO | Steps 2 and 5; requests the staff token on ADM-35 |
  | Grace Lim | REGO | Step 4; maker of APR-2292; the non-V2 persona on ADM-14 |
  | Aisha Karim | SECO | Step 6 (ADM-31 and ADM-37); approves the role grant, Ben's staff token and the data-room grant; revokes Ben's token; works the seeded anomaly token; requests her own staff token (the maker-holds-checker-role case) |
  | Idris Bello | SECO (second) | Rejects Aisha's own staff-token request |
  | Rosa Delgado | PART | Step 9 |
  | Victor Huang | TAXO | Step 10 |
  | Helen Tran | CFO | Checker on APR-2291 and APR-2292; refused as a signer on OP-7730 (she checked APR-2279) |
  | Kai Watanabe | TOKO | ADM-14; holds the seeded first V2 signature on OP-7730; broadcasts it |
  | Marcus Bell | CEO | Second V2 signer on OP-7730 (V2-B slot, spec point (f)) |
  | Priya Natarajan | TRSY | Maker's human of APR-2291 (drafted by her `treasury-drafter` agent) |
  | Dev Sharma | FINA | Owner of `dist-review-prep`, the sibling agent in the checker's finance team |
  | Nadia Farah, Owen Whitlock | KYCA, MLRO | Tile-filter checks and their morning queues (ADM-10, ADM-25) opened from their tiles |
  | Ruth Adeyemi, Lena Moss | SUPP, SUPER | Negative tile checks; SUPER's self-grant and flag-toggle refusals; SUPER is maker of the pending role grant (A67's seed) |
  | NLC's SP-OWN (A06) | Sponsor owner, NLC, as trustee of GYM01 | The trustee seat on APR-2292, on the sponsor surface (spec point (e)) |
  | Anika Stone | TP-RE of YBP's hired RE | Negative: refused on APR-2292, which is NLC's trust |
  | Lachlan Byrne | TP-AUD | Grantee of the data-room grant (does not sign in to the console) |

- Seventeen of these are staff personas: Jordan, Sam, Ben, Grace, Aisha, Idris, Rosa, Victor, Helen, Kai, Marcus, Priya, Dev, Nadia, Owen, Ruth and Lena. The tile check in §3 uses all seventeen. Every one holds exactly one role, as A06 seeds them. The last three rows are sponsor or partner principals and do not sign in to the console.
- A person's session is one browser context per persona, with the console's own cookies and the simulated authenticator bound to that persona's credential. Contexts are never shared, so a decision cannot ride on another person's session.

**3. J20, step by step**

**Timeline of the main run** (one database, one clock; each row runs only after the row above it passes):

| # | Platform time (AEST) | Spec file (Playwright project) | What happens | Checkpoint |
|---|---|---|---|---|
| 0 | 23 Sep 10:30 | global setup | `preview:reset` with the seed; manifest read; before-figures recorded | Chain verification 1; before-figures |
| 1 | 10:30 | `tiles.spec.ts` | Tile filtering for every staff persona (reads only) | — |
| 2 | 10:30 | `j20-daily-run.spec.ts` step 1 | ADM-01 as CCO | — |
| 3 | 10:30 → 11:31 | step 2 | Alerts worked; planted Sev2 at 10:30; clock +60 min to 11:30 (no escalation), then +1 min to 11:31 (one escalation) | — |
| 4 | 11:31 | steps 3 and 4 | ADM-16; ADM-17 and Grace's refusal on APR-2292 | — |
| 5 | 11:31 | approvals block (§4) | APR-2291, APR-2292 (with the trustee seat), APR-2285, bulk and replay refusals | — |
| 6 | 11:31 | steps 5 and 6 | ADM-32; ADM-31, ADM-37 role grant, ADM-35 anomaly token | Chain verification 2 |
| 7 | 11:31 | steps 7 to 12 | ADM-04, ADM-24 disclosure, ADM-29, ADM-20, ADM-27, ADM-34 flags, ADM-26 data-room grant | — |
| 8 | 11:31 | `j20-token-op.spec.ts` | OP-7730 to `closed` and the custody negatives (OP-7731 is only touched by a token here, which is refused before P10) | Chain verification 3 |
| 9 | 11:31 | `j20-agent-access.spec.ts` | Ben's staff token, Aisha's own token request, MCP paired reads, agent negatives, revoke | — |
| 10 | 11:31 | `negatives.spec.ts` | §8 phase-gated controls; any matrix row not already run inline | Chain verification 4; after-figures; checkpoint table |

No clock advance happens after row 3, so the main run ends on 23 Sep at 11:31 plus elapsed seconds. At that time no seeded request has passed its 72 h TTL (the earliest, APR-2285, expires 24 Sep 09:15), and Ben's 24 h token is still in date. The clock sub-runs of §6 each start from a fresh reset.

Each step records the screen, the persona, the SDK reads that check what the screen shows, the audit event ids it produced, and a pass or fail.

- **Step 1, ADM-01 (CCO).** The header reads "Command center" and "Wednesday 23 Sep 2026 · all figures reconciled at …". The time comes from the latest reconciliation run (spec point (c)). Every KPI must equal the same figure read through the SDK, and the before-figures must equal A06's seed manifest for the seed in use:
  - client money: the sum of the three client-money accounts, as A06's manifest gives it for that seed. It is derived from the generated population and is not a locked figure (A06 resolution row on client money), so the suite never types a value. The design's "A$18.42m" is recorded, not asserted;
  - live offers 2 (KST01 and ANN01), with the offer-escrow balance in the caption. For `realx-baseline` that is A$3,513,200.00 (A$3.51m), A06's locked escrow figure. The design's "Live offers 3" and "A$3.05m in escrow" are A06 correction row 6 and A06's escrow correction row; they are cited seed-correction findings, not failures;
  - approvals waiting 5, with the high-risk count computed from the rows (3 in A06's seed: APR-2285, APR-2291, APR-2292; the design's static "2 high risk" is design-analysis §6.4 item 8);
  - open alerts equal to A13's open count in the CCO's scope.

  The alert TaskRows include "Register break · Gympie Road Retail … 9,998 units; token supply 10,000" High at 06:02. The CCO opens the prudential card (A76) and the treasury tile shows bank vs ledger Δ 0. Every live nav badge equals its SDK count (a fix of the design's static badges, §6.4 item 8).
- **Tile filtering (all staff personas, `tiles.spec.ts`).** Each staff persona in §2 signs in and lands on ADM-01. The rendered tile set and each tile's count must equal `expected-tiles.json`. That file is generated by `tiles-expected.ts` from two inputs only: A03's permission-matrix data and A06's role assignments, joined through `tile-catalogue.json`. The catalogue is kept in this node: one row per ADM-01 tile with its key, title, source SDK operation and required cell, written from A66's spec text and the product map's ADM-01 components. It never reads A66's `tiles.ts` or any other `apps/console` file, so the check is not circular. A tile that A66 renders but the catalogue lacks, or the reverse, is a finding on A66 or on this node, settled at Specify. For every tile hidden from a persona, the persona's SDK call to that tile's data operation must be refused (`FORBIDDEN`) or return only rows in scope, so hiding is never the only guard. KYCA's and MLRO's tiles open their queues: ADM-10 shows the seeded INV-00912 case with its SLA remaining, and ADM-25 shows the AML cases, sealed content hidden from anyone but MLRO and CCO. SUPP sees no AML tile, no case count and no restriction reason.
- **Step 2, ADM-02 (ENGO).** Before the step, the chain alert of spec point (b) is raised through `SandboxControl.trigger`. ENGO acknowledges it inside its severity SLA with a reason, assigns a second chain alert to himself, and resolves the first after the sandbox scenario is cleared. A bulk acknowledge that includes the Sev1 GYM01 break is refused all-or-nothing (`bulk_refused` listing the id). A bulk acknowledge of Sev3 and Sev4 rows only succeeds and writes one audit event per alert. A snooze without a reason is refused.
- **SLA clock probe (inside step 2).** A second planted Sev2 alert, raised at 10:30, is left unacknowledged. The clock is moved forward 60 minutes: no escalation. It is moved one more minute: A13 escalates it exactly once to its escalation role, and the escalation appears on the CCO's ADM-01 and ADM-02 with the breach time from A09's clock. Any other seeded alert whose SLA A13 computes to lapse inside the same 61 minutes also escalates; that set is derived by the node QA from A13's severity table and A06's alert times, and the suite asserts the whole set, each member exactly once. The 07:00 digest for the CCO exists in the sandbox inbox.
- **Step 3, ADM-16 (CCO).** The threshold monitors (holder cap, largest holding, foreign %, members) show the values A19 computes for each vehicle. The G22 evaluation-log filter lists the seeded agent refusals. APR-2285 (ANN01 holder cap 5% → 7.5%, maker Jordan Blake, checker Sam R.) appears as a pending parameter change with its simulation of affected holders.
- **Step 4, ADM-17 (REGO).** KPIs: 7 of 8 matched, 1 break of 2 units, oldest break age against the 24 h SLA. The GYM01 row is a red break (register 9,998, chain 10,000, Δ −2), and GYM01's halt state is shown. The break panel shows the proposed fix, APR-2292. Grace Lim opens it and reads the likely cause. Her approve attempt is refused (matrix row R5). The approval path is §4.
- **Step 5, ADM-32 (ENGO).** Every connector shows environment `sandbox`, and none shows `live`. The job runs show the nightly reconciliation, the 03:00 re-screen and the 07:00 digest with their last results. ENGO sees A76's planted failed job and replays its dead letter. He sees the paused critical job's `4E` request waiting for SECO. The human-gate status board shows HG-LEGAL, HG-VENDOR, HG-KEYS, HG-AUDIT, HG-PENTEST, HG-CAPITAL and HG-PROD as open.
- **Step 6, ADM-31 and ADM-37 (SECO).**
  - ADM-31 is filtered from 18:00 to 06:00. Its verification banner shows the chain verified, and it agrees with `npm run audit:verify`, run by the suite through A10's `POST /v1/audit/verify`.
  - On ADM-37, SECO sees the overnight dormancy lock and the pending grant (maker Lena Moss, SUPER), approves the grant with her hardware key, and follows the link into ADM-31 filtered on `iam.*`.
  - Lena Moss (SUPER) tries to grant herself a role and is refused `SOD_SELF_GRANT` with SoD rule 9 cited (A67; matrix row R10).
  - On ADM-35, SECO works the seeded anomaly-flagged burst token (A77, and J22 step 6): she suspends it with a reason, and the owner's notice exists in the sandbox inbox.
- **Step 7, ADM-04 (CCO).** The licence and CAR table shows expiry badges at T-90/30/7 where the seed has them. The DAF Act exemption meter equals A21's computed trailing 12-month total and per-client maximum. The 80 % Sev2 marker sits where A21's configuration puts it.
- **Step 8, ADM-24 (LEGL, then CCO).** Sam Reilly drafts a new version of a platform document (the seeded fee schedule, with a new effective date) and submits it for publication. That is a `4E` request, LEGL maker and CCO checker (A35). Sam's own attempt to approve is refused (matrix row R9). Jordan Blake approves with a hardware key. The library then shows the new version as current, the previous one as superseded and kept, and both with their sha256.
- **Step 9, ADM-29 (PART, Rosa Delgado).** The sponsor-performance table shows the seeded values, and the flagged sponsor carries its pause note (A76, G19).
- **Step 10, ADM-20 (TAXO, Victor Huang).** The withholding exceptions equal A41's read model for the seed, each with its reason code. No exception shows a TFN in clear.
- **Step 11, ADM-27 (CCO).** The complaint rows show acknowledgement and final-response clocks from A44. Pill states change only when the platform clock moves, never with the browser clock.
- **Step 12, ADM-34 and ADM-26 (CCO, then SECO).**
  - The launch card reads `offers.wholesale` on, bound to the platform's CAR appointment. `offers.retail`, `resale.board`, `governance.votes` and `payouts.stablecoin` are off and unbound. The CCO records the launch-state confirmation with her hardware key.
  - SUPER toggle refusal (matrix row R15). On ADM-34 as Lena Moss, the `offers.retail` switch is disabled and shows A69's blocked sentence for `G21_NO_BINDING` ("No licence binding · cannot turn on (N10, G21)"). The request is then made through the SDK as Lena, `POST /v1/flags/offers.retail/toggle`, which is A21's route. The response carries `G21_NO_BINDING` (other precheck codes, such as `G21_PHASE_NOT_REACHED`, may be listed beside it). Settled at Specify (spec point (i)), the outcome is one of two: no approval row exists afterwards, or the row exists in `rejected` after `submitted -> prechecked -> rejected`, as A14's submit contract turns a hard precheck failure into a kept, terminal row. Either way the request never appears in any checker's inbox, no `flag_change` row is written, and the flag still reads off.
  - On ADM-26 the CCO sees the overdue obligation. Jordan Blake creates a data-room grant (`dataroom.grant`, `4E`, CCO maker, SECO checker, A76) for the seeded TP-AUD Lachlan Byrne, with a purpose, at least one collection and a window inside the platform maximum; it is a new grant beside A76's seeded active and expired ones. Jordan's own approve is refused (matrix row R14). Aisha Karim (SECO) approves it with a hardware key, and the new grant card shows state active with "Granted by" naming both people.
- **Journey checkpoint table** (in `checkpoints.ts`; integer cents and units, read through the SDK, independent of the UI). Every before-figure is read from A06's seed manifest for the seed in use; the suite asserts A06's locked invariants and the rule "after = before + the ledger journals traced to approved requests in the run". It types no money figure of its own.

  | Checkpoint | Expected |
  |---|---|
  | Client money | Before: equal to the manifest's derived total for the seed (not locked). After: before plus the sum of the journals posted by the run's approved requests (APR-2291's funding and payout, and any settlement journal A16 and A37 post when OP-7730 closes), each journal traced to its approval id. Bank vs ledger Δ 0 at both points |
  | Offer escrow | Before: the manifest value, equal to A06's locked A$3,513,200.00 for `realx-baseline` (Σ subtotals of funded and allocated orders on KST01, ANN01 and UNL01). After: before plus the traced journals of the run. No order is placed or refunded, so any movement must trace to OP-7730's closing journals |
  | Approvals inbox | Before: 5 pending (APR-2285, APR-2286, APR-2290, APR-2291, APR-2292), 3 High (A06 correction row 4a). After: APR-2291 and APR-2292 approved; APR-2285 rejected; the disclosure, role-grant, data-room and Ben's staff-token requests created and approved; Aisha's own staff-token request created and rejected; the SUPER toggle as settled in spec point (i); APR-2286 and APR-2290 still pending |
  | Reconciliation | Before: 7 of 8 matched and 1 break (GYM01, −2), A06's locked rows. After: 8 of 8 matched and 0 breaks after the approved adjustment and a clean targeted run |
  | OP-7730 | Before: UNL01 mint of 9,100 units, 1 of 3 signed (Kai). After: `closed`, with signatures by Kai and Marcus, the register at 9,100 units and the chain supply at 9,100 tokens (Δ 0) |
  | Audit chain | Verified at rows 0, 6, 8 and 10 of the timeline. The event count per step equals the owning APIs' count |
  | Agent events | Every event written under a token carries `on_behalf_of` |

**4. Approvals, hardware keys and SoD refusals (G13)**

All decisions are taken on ADM-30 through A53's approval action pattern, in the persona's own browser context, with the hardware-key modal signing the payload hash that ADM-30 displays. The approvals block runs at row 5 of the timeline, in this order: APR-2291, APR-2292, APR-2285.

- **APR-2291** (distribution funding, MRN01, A$20,228.50, maker "agent `treasury-drafter` on behalf of Priya N. (TRSY)", checker CFO, `4E`, High).
  - Priya signs in and opens it. The approve control is absent, and a direct SDK decide with her assertion is refused (row R1), because A14's maker identity set includes the human behind the agent.
  - The `treasury-drafter` token calls decide through MCP and is refused `APR_CHECKER_MUST_BE_HUMAN` before scope evaluation (row R2). The same refusal meets Dev Sharma's `dist-review-prep`, a sibling agent in the checker's finance team (row R3).
  - Helen Tran approves with her hardware key. The request moves `prechecked -> approved`, and the downstream funding and payout reach the states A40 defines after approval, with bank vs ledger Δ 0 after A46's payout job runs.
- **APR-2292** (GYM01 register adjustment of +2 units, maker Grace L. (REGO), `6E` with checkers CCO, CFO and the trustee).
  - Grace's approve attempt was refused at step 4 (row R5). Jordan's own agent `cco-digest` tries to decide through MCP and is refused `APR_CHECKER_MUST_BE_HUMAN` (row R4).
  - Jordan Blake and Helen Tran approve in the console. Helen's assertion captured from her APR-2291 decision, replayed on APR-2292 before her real decision, is refused (row R17).
  - The trustee seat. GYM01 is NLC's trust and NLC acts as its own trustee under its own AFSL, so A06 names the trustee checker as NLC's SP-OWN. That persona signs in to the sponsor surface that A14's trustee checker set and A33's tenancy model resolve for a self-trustee sponsor (spec point (e)), opens APR-2292 there and decides it with its own fresh passkey assertion. There is no SDK fallback: if no built screen carries the trustee countersign for a self-trustee sponsor, the run stops with `TRUSTEE_SURFACE_MISSING` and a fix item on the owning node. The evidence records the surface and route used.
  - Negative (row R6): Anika Stone, the TP-RE of YBP's hired RE, tries to decide APR-2292 with her own assertion from her own surface (A105's portal) and through the API. She has no standing on an NLC trust and is refused; no decision row is written and the `6E` count does not move.
  - The effector sends the registry instruction through the registry sandbox. A20's targeted run is clean, the break closes only on that clean re-run, and ADM-17 shows GYM01 matched.
  - GYM01's halt state after closure is whatever A20 and A48 define (reopening needs their criteria). The suite reads it and never asserts a hard-coded value.
- **APR-2285** (rule parameter change, ANN01 holder cap 5% → 7.5%, maker Jordan Blake (CCO), checker Sam R. (LEGL), `4E+Q`, High, prerequisite "licensee notified" still pending; A06).
  - Jordan's own approve is refused (row R7).
  - Sam R.'s approve is refused with A14's refusal for the pending prerequisite, naming it (row R8).
  - Sam R. then rejects it with a mandatory reason. The request moves `prechecked -> rejected`, and the maker (Jordan Blake) has a notice ("rejected · maker notified" in the design's wording).
- **Other requests of the run:**
  - The disclosure publication (§3 step 8).
  - The role grant (§3 step 6).
  - The data-room grant (§3 step 12).
  - Ben's staff token (§9).
  - Aisha's own staff-token request, rejected by Idris Bello (§9).

  Each approve is given by a person other than its maker, with a hardware-key assertion, and leaves an assertion id on the decision row. A10's chain holds exactly one decision event per decision.
- **Precedence of the maker refusals.** Most makers in the run do not hold the checker role of their own request (Priya, Grace, Sam, Ben and Jordan on the data-room grant). For them A14 may legitimately report either `APR_MAKER_IS_CHECKER` or `APR_ROLE_NOT_HELD`, because A14 does not fix which check runs first. Unless spec point (g) fixes the order at Specify, the suite accepts either code for those rows. Segregation of duties by identity is proved by the case where the maker does hold the checker role: row R13 (Aisha, SECO, on a request whose checker role is SECO) must return `APR_MAKER_IS_CHECKER` and nothing else, so a build with no maker-identity check fails it.
- **Refusal matrix (main run).** Each row returns its stable code, shows A53's sentence for that code on screen where a screen is involved, and leaves no state change: no decision row, no signature, no flag change and no meter movement.

  | # | Principal (A06) | Request or object | Route | Expected code |
  |---|---|---|---|---|
  | R1 | Priya Natarajan (TRSY), maker's human | APR-2291 | ADM-30 (no approve control), then SDK decide with her assertion | `APR_MAKER_IS_CHECKER` or `APR_ROLE_NOT_HELD` (spec point (g)) |
  | R2 | `treasury-drafter` (Priya's agent), maker's agent | APR-2291 | MCP decide | `APR_CHECKER_MUST_BE_HUMAN`; one G22 `GateEvaluation` with `G22_AGENT_NEVER_CHECKER` |
  | R3 | `dist-review-prep` (Dev Sharma's agent), sibling agent in the checker's finance team | APR-2291 | MCP decide | `APR_CHECKER_MUST_BE_HUMAN`; G22 evaluation |
  | R4 | `cco-digest` (Jordan Blake's agent), a checker's own agent | APR-2292 | MCP decide | `APR_CHECKER_MUST_BE_HUMAN`; G22 evaluation |
  | R5 | Grace Lim (REGO), maker | APR-2292 | ADM-30, then SDK | `APR_MAKER_IS_CHECKER` or `APR_ROLE_NOT_HELD` |
  | R6 | Anika Stone (YBP's TP-RE) | APR-2292 (NLC trust) | A105 portal, then API with her assertion | A14's not-eligible refusal (`APR_ROLE_NOT_HELD`), or `NOT_FOUND` if A98's tenant scoping hides the request; fixed at Specify |
  | R7 | Jordan Blake (CCO), maker | APR-2285 | ADM-30, then SDK | `APR_MAKER_IS_CHECKER` or `APR_ROLE_NOT_HELD` |
  | R8 | Sam Reilly (LEGL), checker | APR-2285 approve with the prerequisite pending | ADM-30 | A14's prerequisite refusal, naming "licensee notified" |
  | R9 | Sam Reilly (LEGL), maker | Disclosure publication | ADM-30, then SDK | `APR_MAKER_IS_CHECKER` or `APR_ROLE_NOT_HELD` |
  | R10 | Lena Moss (SUPER) | Role grant to herself | ADM-37, then SDK `POST /iam/roles/grant` | `SOD_SELF_GRANT` (rule 9) |
  | R11 | Ben Cole (ENGO), requester | His own staff token | ADM-35, then SDK | `APR_MAKER_IS_CHECKER` or `APR_ROLE_NOT_HELD` |
  | R12 | Ben Cole (ENGO) | Widened token request with a treasury cell he does not hold | ADM-35 | `SCOPE_EXCEEDS_OWNER` naming the cell, before any approval exists |
  | R13 | Aisha Karim (SECO), requester who holds the checker role | Her own staff-token request (checker SECO) | ADM-35, then SDK | `APR_MAKER_IS_CHECKER` only |
  | R14 | Jordan Blake (CCO), maker | Data-room grant | ADM-30 or ADM-26, then SDK | `APR_MAKER_IS_CHECKER` or `APR_ROLE_NOT_HELD` |
  | R15 | Lena Moss (SUPER) | `offers.retail` toggle | ADM-34 (switch disabled), then SDK `POST /v1/flags/offers.retail/toggle` | `G21_NO_BINDING` (spec point (i) for the row outcome) |
  | R16 | Jordan Blake (CCO) | Bulk approve of APR-2290 (Medium) and APR-2292 (High) | ADM-30 bulk | `APR_BULK_NOT_LOW_RISK` |
  | R17 | Helen Tran (CFO) | Her APR-2291 assertion replayed on APR-2292 | SDK | `APR_ASSERTION_INVALID` or `APR_CHALLENGE_MISMATCH` |
  | R18 | Kai Watanabe (TOKO) | Second signature on OP-7730 | ADM-14, then SDK | A15's duplicate-signer refusal; meter stays 1 of 3 |
  | R19 | Helen Tran (CFO), checker of APR-2279 | Signature on OP-7730 | ADM-14, then SDK | `P9_INITIATOR_IS_SIGNER` if the settled V2 record holds a CFO slot, otherwise A15's not-a-member refusal (spec point (f)); meter unchanged |
  | R20 | Grace Lim (REGO), not a V2 member | Signature on OP-7730 | ADM-14 (no enabled Sign), then SDK | A15's not-a-member refusal |
  | R21 | `treasury-drafter` token | Sign OP-7730 | MCP | `P9_TOKEN_CREDENTIAL`; G22 evaluation |
  | R22 | Ben's staff token `engo-morning-checks` | Sign OP-7731 | MCP | `P9_TOKEN_CREDENTIAL`; G22 evaluation |
  | R23 | Ben's staff token | Acknowledge the GYM01 Sev1 alert | MCP | `G22_SCOPE_MISSING` naming the cell |
  | R24 | Ben's staff token | Approve the paused-job `4E` request | MCP | `APR_CHECKER_MUST_BE_HUMAN` |
  | R25 | Ben's staff token after revoke | Any read | API bearer route (`api` container) and MCP (`mcp` container), 20 times each | `TOKEN_REVOKED` |

  Twenty-five rows. The sub-runs of §6 add their own refusals (`APR_EXPIRED`, `P1_EXPIRED`, `P10_NOT_EXECUTABLE_YET`), listed there.

**5. One custody operation, signed 2-of-3 and reconciled (SM-00 tail, `j20-token-op.spec.ts`)**
- **Who can sign (spec point (f)).** A15's P9 removes an approval's maker and checkers from the signer set of its op. APR-2279 was approved by Helen Tran (CFO) and Jordan Blake (CCO), so neither may sign OP-7730. A15 owns the vault membership and sets the V2 slots as `V2-A` SECO, `V2-B` CEO and `V2-C` a TOKO who is not the op's maker, precisely so that CFO and CCO are not V2 signers. A06's vault record ("V2 token agent 2-of-3 (TOKO, CCO, CFO)") conflicts with A15 and would leave only one eligible signer, so no correct build could reach 2 of 3. The default, settled at Specify with A06 and A15, follows A15: the eligible V2 signers on OP-7730 are Kai Watanabe (TOKO, the seeded first signature), Marcus Bell (CEO) and Aisha Karim (SECO). The second signer in this run is Marcus Bell.
- On ADM-14, Kai Watanabe (TOKO) sees "Waiting for signatures". OP-7730 is the "Mint on offer close · Unley Road Cottage · 9,100 tokens" row, bound to APR-2279, V2 vault, with the SignatureMeter at 1 of 3 (A06's seed count).
- Kai tries to sign a second time and is refused (row R18): the quorum counts distinct signers, and the meter stays at 1 of 3.
- Helen Tran presses Sign and is refused (row R19). If the settled vault record keeps a CFO slot, the code is `P9_INITIATOR_IS_SIGNER`, which is the P9 checker-exclusion case. If it follows A15's slots, Helen is not a V2 member and the refusal is A15's membership refusal; the checker-exclusion case is then proved by A15's own conformance case (a CEO who checks a request and holds V2-B), and the evidence says so. The meter does not move in either case.
- Marcus Bell signs under his own console login with a hardware-key assertion over `calldata_sha256`. The meter reads 2 of 3, and the row's button changes from "Sign" to "Broadcast", as in the design.
- Kai broadcasts. A15's policy engine passes P1 (the approval id is bound and approved), P2, P3 (supply guard: minted units equal the locked allocation) and P9 (neither signer is the maker or a checker of APR-2279). The local chain mines, and A27's indexer confirms.
- The registry acknowledgement and A20's targeted run follow. The operation walks `approved -> queued_for_custody -> signed -> broadcast -> confirmed -> reconciled -> closed`, and each transition is observed through the SDK with exactly one audit event.
- The row leaves "Waiting for signatures" and appears in "Recent" as "Mint · Unley Road Cottage · APR-2279 · <tx> · Confirmed". The design's toast wording "broadcast · register sync queued" is shown at broadcast, and "Confirmed" is not shown until A20 reports the op reconciled.
- **Custody negatives in the main run:**
  - The `treasury-drafter` token signs OP-7730 and Ben's staff token signs OP-7731, both through MCP. Both are refused `P9_TOKEN_CREDENTIAL` (rows R21 and R22), and each refusal is recorded as a G22 `GateEvaluation`.
  - Grace Lim, who is not a V2 member under any record, sees no enabled Sign control, and a direct SDK sign is refused (row R20).
  - OP-7731's timelock is probed in sub-run S3 (§6), not here, because it needs the clock on 24 Sep.

**6. Alerts, SLA clocks and the clock sub-runs**
- Every alert the run touches carries a severity, an owner role and a clock from A13. The suite asserts acknowledge, assign, escalate, snooze and resolve through ADM-02 only. Each needs a reason where A13 requires one, and each leaves one audit event.
- The three clock probes run as separate sub-runs, each after `preview:reset` with the same `--seed`, each with its own expected outcomes, so no probe disturbs the main run's checkpoints.

  | Sub-run | Spec file | Clock script (AEST) | Expected outcomes |
  |---|---|---|---|
  | S1 TTL | `probes/ttl.spec.ts` | Start 23 Sep 10:30. Create and approve the probe request (below), leave its op unsigned. Move to 26 Sep 05:26 (APR-2290 one minute past its TTL; Jordan's decide). Then move to the probe request's `expires_at` exactly, read; then +1 min (about 26 Sep 10:31). The clock never passes 26 Sep | See the TTL rows below |
  | S2 KYC SLA | `probes/kyc-sla.spec.ts` | Start 23 Sep 10:30. Leave the KYC queue unattended. Move to one minute past INV-00912's 24 h identity SLA (referred 23 Sep 08:14, so 24 Sep 08:15) | One `queue.sla_breach` for INV-00912, owner KYCA, exactly once. Other alerts that A13 computes to lapse in the window escalate as the node QA's derived set, each once. No seeded request expires (the earliest, APR-2285, expires 24 Sep 09:15) |
  | S3 OP-7731 timelock | `probes/op7731.spec.ts` | Start 23 Sep 10:30. Move to 24 Sep 09:09, sign; move to 09:10, sign | At 09:09 an eligible V2 signer (Kai) is refused `P10_NOT_EXECUTABLE_YET` and the meter stays 0 of 3. At 09:10 the same signature is accepted (1 of 3). Nobody presses Broadcast, and the chain holds no transaction for OP-7731. If the settled V2 record holds a CCO or CFO slot, Jordan or Helen (APR-2288 checkers) is refused `P9_INITIATOR_IS_SIGNER` at 09:10 |

- **TTL probe (S1) in detail.** The probe request is a custody-bound V2 request created and approved in the sub-run, chosen at Specify from A14's policy rows (spec point (h); proposed: a partial freeze `freezePartialTokens` on a seeded wallet). It is approved by an eligible checker and its op is left unsigned.
  - At `expires_at` exactly, the request is still in its pre-expiry state.
  - At `expires_at` + 1 minute: the request takes `approved -> expired` (a catalogue edge, whatever happens to extension (c)), and a V2 signature on its op by an eligible signer is refused `P1_EXPIRED` with the meter unchanged. If A14's effector hands every custody-bound request to custody at approval, so that the request is already `queued_for_custody`, the expected transition is instead `queued_for_custody -> expired` at the execution deadline, and `approved -> expired` is recorded as `not reached in J20`. Spec point (h) fixes which one applies before the node QA writes its material.
  - The seeded pending requests pass their own TTLs in the same sub-run (APR-2285 at 24 Sep 09:15, APR-2286 at 25 Sep 09:40, APR-2290 at 26 Sep 05:25, APR-2292 at 26 Sep 06:20, APR-2291 at 26 Sep 08:47, each as A14 computes from submission). **If A03's extension (c) is adopted** (the default), each takes `prechecked -> expired` through A14's sweep, and Jordan's decide on APR-2290 at 26 Sep 05:26 is refused `APR_EXPIRED`. **If extension (c) is declined**, no `prechecked -> expired` edge exists, the requests stay `prechecked`, Jordan's decide is still refused `APR_EXPIRED` by A14's decide-time check on `expires_at`, and the evidence records `prechecked -> expired` as `not in catalogue`.
  - A13's escalations in the window are asserted as the node QA's derived set, each exactly once. The staff tokens that expire in the window are read, not asserted against the main run.

**7. Hash chain (G13 evidence)**
- The chain is verified at four checkpoints (timeline rows 0, 6, 8 and 10) by ADM-31's banner and by A10's verifier, and the two must agree.
- The ADM-31 export of the run's events is itself audited, and its file hash is recorded in the evidence.
- The chain head hash at the end of each run is recorded, and it must equal `GET /v1/audit/head` at that moment. WebAuthn signatures and request ids differ on every run, so heads are never expected to repeat across runs, even with the same seed. Each run's chain must verify.

**8. Licence perimeter (G21)**
- Steps 7 and 12 are the G21 part of the run. The suite also asserts that every phase-gated console control the personas reach is disabled with A21's reason while its flag is off: the resale-board pages, retail offer controls and stablecoin payout options. A direct SDK call to each such operation is refused with `G21_FLAG_OFF`.

**9. ADM-35 staff token, issued, used and revoked (D04, G22; `j20-agent-access.spec.ts`)**
- **Request.** Ben Cole requests an `rx_stf_` token on ADM-35 for an agent named `engo-morning-checks`. Scopes: read cells for M01 alerts, M10 recon runs and M26 integrations and jobs, plus one maker-only M26 write cell ENGO holds (proposed: dead-letter replay; fixed at Specify against A03's matrix). The write cell is there on purpose: under A12 §2 and A77, a read-only staff token activates on the requester's assertion alone, and only a token with a write family waits for a SECO who is not the requester. Expiry 24 h. Signed with his hardware-attested assertion, the token sits in `pending` as A14 request `agent.token.create_staff` (`4E`, SECO checker).
  - Ben's own attempt to approve it is refused (row R11).
  - A widened request that adds a treasury cell ENGO does not hold is refused `SCOPE_EXCEEDS_OWNER`, naming the cell, before any approval exists (row R12).
- **Maker who holds the checker role.** Aisha Karim (SECO) requests a staff token for herself (agent `seco-evening-review`, with one SECO write cell, so it needs a SECO checker). She holds SECO, so the role check passes, and her own approve must be refused `APR_MAKER_IS_CHECKER` (row R13). Idris Bello (the second SECO) then rejects it with a reason (`prechecked -> rejected`). The token never becomes active.
- **Approval and first use.** Aisha Karim approves Ben's token with her hardware key. The secret is shown once in the copy-once card. The suite reads it from the DOM, holds it in memory only, and never writes it to a file, log, trace, video or the evidence. Playwright's trace and video are off for this spec, and a scan checks that.
- **Parity check through MCP (paired reads).** For each resource (ENGO's alerts, the latest reconciliation run, the job runs), the suite takes a pair of reads at the same platform instant, with no clock advance and no mutation between them: Ben's console session reads it through the SDK as Ben, and the agent reads it through `packages/mcp` against `http://localhost:4010/mcp`. The pair must be equal row for row. The only allowed differences are `actor`, `token_id` and `on_behalf_of` on audit rows. Each pair is taken twice, once with Ben first and once with the agent first.
- **Agent negatives.** The agent then tries to:
  - acknowledge the GYM01 Sev1 alert, which is outside its scopes, so `G22_SCOPE_MISSING` names the cell (row R23);
  - approve the paused-job `4E` request (`APR_CHECKER_MUST_BE_HUMAN`, row R24);
  - sign OP-7731 (`P9_TOKEN_CREDENTIAL`, row R22).
- **Revoke.** Aisha revokes the token with a reason on ADM-35. The agent's next call is refused `TOKEN_REVOKED` on both API-facing processes, the `api` container's bearer route and the `mcp` container, within one request, in 20 of 20 repetitions on each (row R25). ADM-35's evaluation log shows every allowed and refused call with its reason codes, and Ben's activity lists every call "agent `engo-morning-checks` on behalf of Ben Cole".
- **Parity scope.** This is the J20 part of D04's parity. Every other console action's API and MCP equivalent is A100's matrix, and the operator's full agent journey is A82's. This node adds no action of its own for a person or an agent. The helpers are test tooling over existing routes, so nothing here needs a parity row.

**10. The AM6 sponsor journeys again, with the RealX side through the console (`operator-steps/console.ts`)**
- A65 declares `e2e/sponsor/operator-steps/interface.ts`, with one method per RealX-side act of J11, J12, J13, J14 and J19, and ships the `sdk` implementation and `personas.ts`. This node ships `console.ts`, which implements the same interface, method for method, by driving the ADM screens with the persona from A65's `personas.ts` that A14's policy row names for each act. Examples:
  - `approveSponsorKyb` on ADM-03 (KYCA maker, CCO checker);
  - `createTenant` on ADM-03 (PART maker, CCO checker);
  - `icApprove` on ADM-05 with two of CFO, CCO and CEO;
  - `reviewValuation` and `decideValuationPublish` on ADM-06;
  - `signOffListing`, `returnListing` and `decideGoLive` on ADM-07 and ADM-30;
  - `deployToken` on ADM-14 with V1 3-of-5 signers on their own logins and the clock moved through the 48-72 h timelock;
  - `flagDistressed` on ADM-22;
  - `approveDistribution` on ADM-19 bound to the payment file hash;
  - `publishHolderDisclosure` on ADM-28.

  Each method returns only what the screen shows (approval id, object state and pill), read back through the SDK as the same persona for the assertion.
- `operator:sponsor-rerun` runs A65's journey files unchanged with `OPERATOR_STEPS=console`. The journeys must pass with the same assertions as in AM6.
- A network log of the run must show every RealX-side decision coming from the console origin's session, with zero calls from A65's `sdk.ts` for methods that `console.ts` implements. A method the console cannot perform because a screen lacks the action, or because the screen does not show the result the journey needs, is a finding filed on the owning console node (A69, A70, A71, A72, A74 or A75), or on A65's interface (loop A65.1). It is never worked around here.
- A65's negatives (declined sponsor, the self-approval prohibition, the >10 % valuation move refused without comparables, and the reporting breach flagging the listing publicly) must produce the same refusals, now visible on the console screens with A53's sentences.

**11. Captures beside the prototype (`e2e/operator/shots/`)**
- The node reuses A61's `prototype` service (read-only mount, hash check) and A61's comparer. It does not edit A61's `e2e/preview/shots/prototype-driver.ts`. Its own `shots/admin-driver.ts` imports A61's driver helpers read-only and adds the admin navigation: the "view as" role pill "RealX admin" lands on `adCmd`, and the sidebar buttons reach `adApprovals` and `adToken`. The driver waits until the page code label reads `ADM-01`, `ADM-30` or `ADM-14`.
- **Gated captures** (Jordan Blake's console, whose user card equals the design's "Jordan Blake · Chief compliance officer"), each at 1280 px and 1024 px, the console widths A67 and A69 use:
  - ADM-01 at 10:30 (`AS_OF`), before any decision;
  - ADM-30 with APR-2292 selected, before any decision on it. Jordan (CCO) is one of APR-2292's `6E` checkers, so the design's SoD note and the Approve button are shown to him. The design selects APR-2291, whose only checker is CFO; the differences in the list (APR-2288 decided, APR-2292 added, APR-2291 retitled) are cited A06 seed corrections;
  - ADM-14 before and after Marcus Bell's signature.

  That is four captures at two widths, which makes 8 pairs.
- **Recorded, not gated:**
  - ADM-17 against `adRecon` and ADM-31 against `adAudit` (their owners, A73 and A66, gate them);
  - every design-extension screen walked in J20 (ADM-02, 04, 16, 20, 24, 26, 27, 29, 32, 34, 35 and 37) against A51's extension file.
- **Region checks, as in A61.**
  - Computed style gates: font family, size and weight, colours, paddings, radii and borders, within 1 px or an exact hex.
  - Text gates: every seeded string the design shows must be equal, using A06's corrected string wherever A06 has a correction row. Examples: "Command center", "Register break · Gympie Road Retail", "APR-2292", "Distribution funding" (A06's correction of the design's "Distribution run"), "Maker-checker (G13): you are not the maker, so you can approve. Approval ID is bound to the signing request.", "Waiting for signatures", "Mint on offer close", "custody quorum". The KPI values "Live offers 2" and the escrow caption are gated against the SDK, and the design's "3" and "A$3.05m" are cited as A06 seed corrections.
  - Pixel difference is recorded after masking the dev role bar, times, tx hashes and wallet addresses on both sides.
  - A region above 5 % must cite a specific finding. Allowed findings: the live counts and badges (§6.4 item 8), A06's seed corrections to the KPIs, approvals and ops lists (§6.4 items 4, 5 and 6; for example Live offers 2, escrow A$3.51m, APR-2288 decided, APR-2292 added, APR-2291 retitled, OP-7730 at 1 of 3), the hardware-key modal and approvals chain drawn by A51 (§6.4 item 18), or the product-map layout of a table. Otherwise the region fails.
- The admin shell tokens are checked region by region:
  - sidebar `#1C1E22` with text `#E8E4DC`;
  - user card `#2A2D31`;
  - accent `#8FC3AC`, and muted `#8E8A82` and `#A6A198`;
  - main surface cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B` for Approve and Sign, ink for Broadcast, red `#A3312A` for Reject;
  - line `#E3DED4` and soft line `#ECE8E0`;
  - Newsreader for titles, Geist for text, Geist Mono for codes, ids and figures.

  The BrickX-style coral look must appear nowhere.

**12. Guide and evidence**
- `e2e/operator/README.md` covers:
  - how to run `verify:operator`;
  - the persona table;
  - the timeline table and the three sub-run clock scripts;
  - the allowlist;
  - how the user can walk the console in their own browser on `http://console.localhost:3001` with the sandbox authenticator. This is optional, and notes become fix items on the owning node, as in A61.
  - the swap points the run touched (custody, registry, bank, identity, chain, email), each `sandbox` from `/readyz`, with the human gate that replaces it.
- `app/evidence/A78.json`, per run, holds:
  - the seeds, the manifest content hashes, the image digests and the boot report;
  - the persona ids;
  - the J20 step table with platform times, timings and audit event ids;
  - the tile matrix result (personas × tiles, rendered and API-checked);
  - the refusal matrix with the code observed per row, and the sub-run refusals;
  - the SM-00 transitions observed per request, including the sub-runs, with `not reached in J20` and `not in catalogue` entries;
  - OP-7730's lifecycle with its signers, tx hash and recon run id;
  - the four chain verifications and the head hash with the matching `GET /v1/audit/head`;
  - the checkpoint table with each traced journal id;
  - the staff-token paired-read comparison (never the secret);
  - the sponsor re-run results with the per-method console/SDK call counts;
  - the capture summary;
  - the `pending Axx` list.

  `A78.qa.md` is the node QA's.

## Owned paths (other nodes do not edit these)

- <workspace>\app\e2e\operator\

Inside it, by convention:
- `j20-daily-run.spec.ts`, `tiles.spec.ts`, `j20-token-op.spec.ts`, `j20-agent-access.spec.ts` and `negatives.spec.ts`;
- `probes/ttl.spec.ts`, `probes/kyc-sla.spec.ts` and `probes/op7731.spec.ts`;
- `timeline.ts` (the timeline table as data) and `playwright.config.ts` (`workers: 1`, project dependencies);
- `checkpoints.ts`, `tile-catalogue.json` and `tiles-expected.ts` (with its generated `expected-tiles.json`);
- `pages/` (this node's page objects for every console screen A66 does not export one for);
- `actors.ts`, which reuses A61's authenticator helpers read-only;
- `operator-steps/console.ts`;
- `shots/` (`admin-driver.ts` and `regions.admin.json`);
- `evidence.ts` and `README.md`.

The root npm aliases are added under A01's script-registration convention, and the evidence goes to the shared `app/evidence/` folder under this node's file names. A number of paths are read, served or driven, never edited: A23's `infra/stack/`, A61's `infra/preview/` and `e2e/preview/` (including `shots/prototype-driver.ts`), A65's `e2e/sponsor/` (including `operator-steps/interface.ts` and `personas.ts`), the console feature folders of A66 to A77 and A66's page objects, A53's shell, A51's `app/design-ext/admin/`, and the design folder. A finding against any of them goes back to the owning node as a fix item.

## Design references

- DESIGN = `<workspace>\design\RealX App.dc.html` run by `support.js`, served read-only by A61's `prototype` service and never modified. EXT-B = [[A51 - Design extension B- operator console screens not in the prototype]] (`app/design-ext/admin/`). ANALYSIS = the design analysis (§3.3 admin console, §4 behaviours 22-27, §6.4 bugs).
- **Admin shell** (all captures): DESIGN lines 620-735. The dark sidebar is at lines 622-629: wordmark "Real*X* ADMIN", the user card "Jordan Blake / Chief compliance officer", and the nav from `adNavDef` at line 862 with badges. The page header is at lines 631-635 (code, Newsreader 40 title, subtitle), the KPI grid at lines 636-640, and the TaskRows at lines 642-653. The "view as" role bar (lines 22-31) is masked in every capture. `setRole` at line 792 lands the admin role on `adCmd`.
- **`design:adCmd`** (ADM-01): the page config `PG.adCmd` at line 880:
  - subtitle "Wednesday 23 Sep 2026 · all figures reconciled at 06:00 AEST";
  - KPIs Client money A$18.42m, Live offers 3 (A$3.05m in escrow), Approvals waiting {n} (2 high risk), Open alerts 3 (1 high). The seed corrects these to computed values (Live offers 2, escrow A$3.51m for `realx-baseline`, 3 high risk; client money derived), cited per §11;
  - Alerts TaskRows: Register break · Gympie Road Retail High red 06:02, 14 wholesale certificates expire (G04) Medium, Marion Street valuation due in 21 days (G20) Medium, Listing waiting for review.

  This is step 1's gated capture. The TaskRows are ADM-02's top slice (ANALYSIS §5), and the rest of ADM-02 is `design-ext:ADM-02`.
- **`design:adApprovals`** (ADM-30):
  - the layout at lines 655-681: ApprovalList cards (id · age, risk pill, type, object) at lines 658-665 with the "Inbox clear." empty state at line 666;
  - the detail pane (id · raised by maker, Newsreader 28 type, object, check rows with pills) at lines 668-672;
  - the SoD note "Maker-checker (G13): you are not the maker, so you can approve. Approval ID is bound to the signing request." at line 673, and Approve (green) and Reject (red outline) at lines 674-677;
  - the data `APS` at lines 777-782 and the list derivations at lines 854-857, with the approve and reject toasts at line 913.

  These are §4's gated capture (with APR-2292 selected, §11) and the wording of the SoD note. The hardware-key modal and approvals chain are EXT-B drawings.
- **`design:adToken`** (ADM-14): "Waiting for signatures" TokenOpRows (id · approval id, op, mono detail) with the 3-segment SignatureMeter and the "{n} of 3 custody quorum" label, and the Sign (green) / Broadcast (ink) button, at lines 706-717. The `OPS` data and `tokenOps` derivation (Broadcast at ≥ 2 signatures, "broadcast · register sync queued") are at lines 858-859. The "Recent" table config `PG.adToken` is at line 884. These are §5's gated capture.
- **Recorded comparisons:** `design:adRecon` (`PG.adRecon` line 885) for step 4 and `design:adAudit` (`PG.adAudit` line 888) for step 6. The EXT-B frames are `design-ext:ADM-02`, `ADM-04`, `ADM-16`, `ADM-20`, `ADM-24`, `ADM-26`, `ADM-27`, `ADM-29`, `ADM-32`, `ADM-34`, `ADM-35` and `ADM-37`, plus the hardware-key confirmation modal and the MandateTable block.
- **Tokens** (ANALYSIS §1) are checked region by region as listed in §11 of the brief. The `TONE` map at line 740 (green, amber, red, grey, brick) is the pill vocabulary every status in the run must map onto.

## Requirements covered

- **Product-map screens:** none owned. The run walks ADM-01, 02, 04, 10, 14, 16, 17, 20, 24, 25, 26, 27, 29, 30, 31, 32, 34, 35 and 37, plus the ADM screens A65's operator steps need (03, 05, 06, 07, 19, 22, 28). "Done" for each belongs to its console node. This node's evidence shows the screen working inside the day's sequence with seeded data and distinct personas.
- **Journeys: J20 Operator daily run.** Done when all 12 catalogue steps pass through the console with the named role's persona, three runs in a row with different seeds, and the three clock sub-runs pass for each seed. That means:
  - every ADM-01 tile is filtered by role on screen and at the API with zero disagreements against `expected-tiles.json`;
  - the alerts are worked inside their SLA clocks, and the one left unattended escalates exactly once;
  - the three J20 approvals and the other requests are decided by a person who is not the maker, with a hardware key, and every row of the refusal matrix appears with its code;
  - OP-7730 is signed 2-of-3 by two eligible V2 signers and reaches `closed` only after a clean reconciliation;
  - the GYM01 break closes on a clean re-run;
  - the chain verifies at all four checkpoints;
  - the launch flags are confirmed off, and a toggle without a binding is refused.

  The catalogue's outcome "platform healthy, exceptions owned" is asserted as: no unowned open alert, no unassigned break, Δ 0 on client money, and every pending request with a named checker role. This is the AM7 exit criterion.
- **Gates:**
  - **G13 Maker-checker.** Done when every decision in the run carries a hardware-key assertion over the payload hash shown on screen. The maker, the maker's agents and the maker's human never see an enabled approve and are refused at the API. A maker who holds the checker role is refused `APR_MAKER_IS_CHECKER` (row R13). `6E` needs its three distinct checkers, the trustee seat taken by the trust's own trustee principal. No custody operation broadcasts with fewer than two distinct eligible V2 signatures or before its timelock. The number of decision events in A10 equals the number of decisions taken.
  - **G21 Licence perimeter.** Done when ADM-04's meter and licence rows equal A21's computed values, ADM-34 shows the launch state with only `offers.wholesale` on and bound, the CCO's confirmation is recorded, a toggle without a binding is refused with `G21_NO_BINDING` and never reaches a checker, and every phase-gated control and operation reached in the run is disabled or refused with A21's reason.
  - **G22 Agent authority.** Done when the ENGO staff token is issued only with a hardware-attested assertion and a SECO approval, is refused on a scope its owner lacks, and reads exactly what its human's session reads at the same instant. It can never acknowledge outside scope, approve or sign. It is refused on its first call after revocation on both the `api` and `mcp` containers. The seeded burst token is suspended with a notice to its owner. Every agent event carries `on_behalf_of`.
- **Modules:** none owned. This node composes, and adds no module behaviour.
- **State machines: SM-00 Privileged action.** Done when the run observes these through the SDK, with one audit event per transition:
  - `submitted -> prechecked -> approved` (APR-2291, APR-2292, the disclosure, role-grant, data-room and Ben's staff-token requests);
  - `prechecked -> rejected` by a checker (APR-2285 after the prerequisite refusal; Aisha's own staff-token request);
  - `submitted -> prechecked -> rejected` on a hard precheck failure (the SUPER toggle, if spec point (i) settles it that way);
  - in sub-run S1: `approved -> expired` for the probe request (or `queued_for_custody -> expired`, per spec point (h)), and `prechecked -> expired` for the seeded pending requests when extension (c) is adopted;
  - `approved -> queued_for_custody -> signed -> broadcast -> confirmed -> reconciled -> closed` (OP-7730);
  - the invariants ANALYSIS §3 lists: the approval id travels to custody, maker ≠ checker, a confirmed transaction is not a success until reconciled, TTL 72 h, and no agent moves anything past `submitted`.

  Transitions the run cannot reach (`draft -> cancelled`, `broadcast -> failed`) are A14's and A28's and are listed as `not in J20` in the evidence.
- **Founder requirement D04:** done when the J20 staff-token path (issue, paired parity read, refusals, revoke) passes with results identical to the human session, and the refusals named above appear with their codes. A82 and A100 then start from this working surface.
- **Swap points (Q12 option 1):** the evidence lists every sandbox adapter the run touched (custody, registry, bank, identity, sanctions, chain, email) with `implementation = sandbox`. It also lists the operator-step driver, now `console`, as the AM6 to AM7 swap point that A65 declared.

## Gauntlet loop for this node

**Specify.** The frozen spec is this note together with:
- A06's seed manifest for each seed, persona ids, approvals, custody ops, vault records, alerts, recon rows, locked figures and correction rows;
- A03's permission matrix data and SM-00 table (with the extension (c) decision);
- this node's `tile-catalogue.json`;
- A14's policy rows for every action code in §4, §6 and §10, and its trustee checker resolution;
- A15's vault membership, P9 and P10, and the OP-7731 timelock;
- A12's staff-token issue rule (write family needs a SECO);
- A13's severity table;
- A21's launch-state seed and flag-toggle route;
- A65's `operator-steps/interface.ts` and `personas.ts`;
- A66's page objects;
- A51's frames.

These spec points are settled with the user or the owning node before the node QA writes its material:
- (a) Resolved: A06 already seeds Rosa Delgado (PART) and Victor Huang (TAXO), each holding only that role. The run uses them.
- (b) A06 names the sandbox scenario that raises ENGO's chain alert. Proposed: A15's V7 paymaster runway-low alert, or A27's indexer-lag alert. If neither exists as a scenario, it is a fix item on A06.
- (c) The reconciliation time. J20 and A46 say 02:00, A06's seed run and the design's header say 06:00. The run reads "the latest nightly run" and never hard-codes either. The user or A06 decides which is the nightly time.
- (d) Confirmation from A65's evidence that its journeys never read a field only `sdk.ts` returns. If one does, that is loop A65.1.
- (e) The trustee seat on APR-2292. A14 and A33 confirm that NLC's SP-OWN is the trustee checker for GYM01 (A06: "NLC as trustee, signed by its SP-OWN") and name the sponsor-side screen that carries the countersign for a self-trustee sponsor. If no node owns that screen, it is a fix item (on A64, A14 or A105, as the owners decide) and the run fails `TRUSTEE_SURFACE_MISSING`; there is no SDK fallback.
- (f) V2 vault membership. A06's record (TOKO, CCO, CFO) conflicts with A15's slots (SECO, CEO, TOKO) and with P9 on APR-2279 and APR-2288. Default: A15's slots, with Marcus Bell as OP-7730's second signer and A06 reseeding its V2 record (fix item on A06). The alternative, reseeding APR-2279's checkers so they are not V2 members, is A06's and A14's call; if chosen, §5 names the resulting second signer and row R19 expects `P9_INITIATOR_IS_SIGNER`.
- (g) A14's check order for a maker who does not hold the checker role (`APR_MAKER_IS_CHECKER` before or after `APR_ROLE_NOT_HELD`). Default: the suite accepts either for rows R1, R5, R7, R9, R11 and R14, and row R13 proves the identity check alone.
- (h) The TTL probe's custody-bound request type (proposed: a partial freeze), and whether A14 leaves an approved custody-bound request in `approved` until custody accepts it (then `approved -> expired`) or hands it over at approval (then `queued_for_custody -> expired`).
- (i) The SUPER toggle's row outcome: no approval row, or a kept `rejected` row after `submitted -> prechecked -> rejected` (A14's submit contract suggests the latter).
- (j) A66's stated ADM-01 figures ("3 / A$3.05m in escrow") conflict with A06 correction row 6 and A06's escrow figure. A66 must read them from A06 (fix item on A66); this run asserts the computed values.

The node QA records `specVersion` and the hashes of those inputs.

**Build (Grok 4.7 xhigh, split by the planner).** Three parallel sub-agents on disjoint folders, then one integration pass:
1. *J20 and gates* (`j20-daily-run.spec.ts`, `tiles.spec.ts`, `tile-catalogue.json`, `tiles-expected.ts`, `checkpoints.ts`, `timeline.ts`, `negatives.spec.ts`, `probes/`, `pages/`): the timeline, the persona contexts, the clock scripts, the refusal matrix, the sub-runs and the four chain checkpoints.
2. *Custody and agent access* (`j20-token-op.spec.ts`, `j20-agent-access.spec.ts`): OP-7730 through `closed`, the custody negatives, and the staff tokens with their paired MCP reads and revocation, with trace and video off.
3. *Sponsor re-run and captures* (`operator-steps/console.ts`, `shots/`, `README.md`): one console method per interface method, `admin-driver.ts`, `regions.admin.json` with the dc line references, and the guide.

Integration, single agent: register the aliases and static checks, run `npm run verify` on a fresh clone, run `npm run verify:operator` on Docker Desktop, run the J20 steps once by hand from the guide, and write `app/evidence/A78.json`. There is no git commit; the orchestrator commits after the gate.

**Challenge (node QA, Opus 5.5 high).** On a fresh clone on Windows 11, Node 24.21.0 and Docker Desktop: `npm ci`, `npm run verify`, `npm run verify:operator`, then a diff of the three runs (step results, tile matrix, refusal codes, SM-00 transitions, checkpoint figures against each seed's manifest, capture verdicts). Then the hidden material:
- its independent tile table against the rendered and API results for every persona;
- the figure re-derivation from each seed's manifest;
- the SoD matrix replayed by hand, including a persona whose role is changed mid-run (A67 revoke) before a pending decision;
- the clock probes (Sev2 escalation at 60 and 61 minutes in the main run; TTL at exactly `expires_at` and one minute later, KYC SLA, and OP-7731 at 09:09 and 09:10 in the sub-runs);
- the tamper probe: a copy of the database with one audit row altered by a superuser in a disposable container. ADM-31's banner and A10's verifier must both report `chain_broken` at the same row, and the live stack must never be touched;
- the "green by driving" scans: no SDK call for a method `console.ts` implements, no `/__test/*` call outside the allowlist, no `page.route`, and no expected value read from the page under test or from `apps/console` source;
- a grep of traces, logs, `docker compose logs` and the evidence for the staff-token pattern, bearer headers, key material and planted PII.

It then walks the J20 steps itself as each persona at 1280 px and 1024 px, reads the 8 gated pairs and every pixel justification, confirms every `pending Axx` item names a real node, and writes `app/evidence/A78.qa.md`.

**Gate.** A pass closes milestone AM7, unlocks A80, A82 and A100, and publishes the Challenge material into `app/`.

**Fix loop.** At most 3. Each loop re-runs `verify:operator` in full (three seeded runs with their sub-runs, the sponsor re-run and the captures) and every probe, not only the one that failed.

**Escalate** when:
- a step fails because a console screen or service behaves differently from its node's spec. That is a defect reopened on the owning node (A66 to A77 for screens; A10, A13, A14, A15, A20, A21, A28 or A12 for behaviour), and this node waits.
- A65's journeys cannot pass with `OPERATOR_STEPS=console` without changing a journey file or the interface (loop A65.1).
- a tile, approval or signature succeeds for a persona that should be refused, or an agent reaches an approve or sign by any route. This is Critical and stops the milestone.
- the chain fails to verify, or the reconciliation is not Δ 0 after an approved change. This is Critical.
- a capture region cannot match without editing a console node or the design. That is a decision for the user.
- a spec point (b) to (j) is unresolved.
- the same probe fails three times.
- the builder needs a folder outside the owned path.

## Node-specific challenges (node QA)

- **Green by driving the console from behind (quality, G13).** The run passes because `console.ts` falls back to `sdk.ts` for an awkward method, a step clicks "Approve" and then quietly also calls the SDK, a page object reads the expected count from the same DOM it asserts, the trustee seat is filled through the SDK, or one browser context carries two personas' cookies. The node QA's network log must attribute every decision to the console origin (or, for the trustee seat, the sponsor surface) and to the persona that the policy names. Its expected values must come from its own derivation.
- **Hidden is not forbidden (security, tile filtering).** SUPP's ADM-01 hides the AML and treasury tiles, but SUPP's SDK call to the AML case count, the client-money balance or the restriction reason returns data. Or a KYCA sees the MLRO-only sealed AML content through ADM-01's alert TaskRow text. Any datum the persona's cells do not grant, reached by any route, fails the node, and so does any AML reason visible to SUPP (tipping-off).
- **The maker approves through a side door (compliance, G13).** Priya's `treasury-drafter` token approves APR-2291 through MCP. Priya approves it after her agent drafted it. Grace approves APR-2292 after A67 briefly grants her CFO. Helen approves APR-2292 twice to fill two of the three `6E` seats. A replayed assertion from Helen's APR-2291 decision is presented for APR-2292. YBP's TP-RE fills NLC's trustee seat. Aisha approves her own SECO-checked token. Every one must be refused with its code, and APR-2292 must still need three distinct people, one of them NLC's trustee principal.
- **One signer, two signatures (money, custody).** Kai signs OP-7730 twice. Helen, who checked APR-2279, signs. Marcus signs from Kai's browser context. The ENGO token signs. OP-7731 is signed one minute before its timelock ends. Broadcast is pressed at 1 of 3 through a crafted request. Each must be refused. The chain must show no transaction for OP-7731, and exactly one for OP-7730.
- **Confirmed is not closed (money, reconciliation).** The node QA plants a registry-acknowledgement delay (A08 scenario) after OP-7730 broadcasts. The console must show the op as `confirmed`, waiting for reconciliation, never "Confirmed" in the closed sense. SM-00 must not reach `closed` until A20's targeted run is clean. A planted one-unit registry drift must leave it as a break, not a success.
- **Flags on by paperwork (G21).** SUPER asks to turn `offers.retail` or `payouts.stablecoin` on. CCO approves a request that somehow exists. A binding is attached to a licence that does not cover the activity. The DAF meter is read from a browser-computed figure. Every path must be refused at request creation or by A21's binding rule. Every flag must read off at the end, and the meter must equal A21's server value to the cent.
- **The staff token outlives its purpose (G22, security).** After revocation, a call on the `api` container, a call through the `mcp` container with a cached session, or a call during the rotation grace window must all be refused `TOKEN_REVOKED`. The secret must not appear in the Playwright trace, video, HAR, screenshot, console log or evidence. An agent read must never return a row Ben's own session could not see at the same instant.
- **Chain verified by assertion (audit).** The ADM-31 banner is green because the page renders a cached status, or the suite compares the banner with itself. On the node QA's tampered copy, the banner and A10's verifier must both report the broken row. On the live stack, the head hash in the evidence must equal `GET /v1/audit/head`.
- **Real clock leaks (determinism).** With the host clock set to 1 November 2026 and the browser clock skewed by two hours, the SLA pills, the escalation time, the TTL expiry and the OP-7731 timelock must still follow A09's platform clock. A pill that changes with the browser clock fails.
- **Figures typed, not derived (money).** A step or checkpoint passes only for `realx-baseline` because a figure was typed from this note or the design. Every before-figure must come from the manifest of the seed in use, and every after-figure from before plus traced journals.
- **Design fidelity by excuse (design).** The ADM-01, ADM-30 or ADM-14 capture fails computed style, but the pixel difference is waved through with a vague note. Or the console uses the light sponsor shell, a fallback font, or the coral product-map look. Or the SoD note's wording differs from the design's sentence. Each justification must cite a specific §6.4 item, an EXT-B frame or an A06 seed correction.

## Pass criteria

- On a fresh clone on Windows 11 Docker Desktop, `npm run verify:operator` completes three consecutive seeded runs of J20 (all 12 steps, in the timeline order, `workers: 1`), `tiles.spec.ts`, `j20-token-op.spec.ts`, `j20-agent-access.spec.ts`, `negatives.spec.ts`, the three clock sub-runs and the sponsor re-run, all green. Every service is healthy with restart count 0.
- Tile filtering: for every staff persona in §2, the rendered tile set and counts equal the node QA's independent table and `expected-tiles.json`, with zero disagreements. For every hidden tile, the persona's API call is refused or scoped, with zero leaks.
- Approvals: APR-2291, APR-2292 (three distinct checkers, the third being NLC's trustee principal on the sponsor surface), the disclosure publication, the role grant, the data-room grant and Ben's staff token are each approved by a person other than the maker, with a stored hardware-key assertion id. APR-2285 is refused on its prerequisite and then rejected by Sam R. with a reason. Aisha's own token request is refused to her and rejected by Idris Bello. All 25 rows of the refusal matrix return their named code (or one of the two codes allowed by spec point (g)), show A53's sentence where a screen is involved, and change no state. Row R13 returns `APR_MAKER_IS_CHECKER`.
- Custody: OP-7730 walks `approved` to `closed` with exactly two distinct eligible V2 signatures (Kai and the signer settled in spec point (f)) before broadcast, one on-chain transaction, and register 9,100 = chain 9,100 in A20's clean targeted run. Helen's signature is refused and the meter does not move. In sub-run S3, OP-7731 is refused at 09:09, accepts a signature at 09:10, and has no transaction. Every token-credential sign is refused `P9_TOKEN_CREDENTIAL` and recorded as a G22 evaluation.
- Reconciliation and money: GYM01 moves from break (−2) to matched only after APR-2292's effector and a clean re-run. The KPIs read 8 of 8 matched and 0 breaks. Client money bank vs ledger Δ is 0 before and after. Every before-figure equals the seed's manifest, every after-figure equals before plus traced journals, and both equal the node QA's re-derivation to the cent in all three runs.
- Alerts: the planted Sev2 escalates exactly once at 61 minutes and not at 60, and the full escalation set equals the node QA's derived set. In sub-run S2, INV-00912's SLA breach raises exactly once with owner KYCA. The mixed bulk acknowledge is refused whole. Every acknowledge, assign, snooze and resolve has its reason and one audit event.
- TTL: in sub-run S1 the probe request takes the expiry transition settled in spec point (h), a signature on its op is refused `P1_EXPIRED`, and the seeded pending requests behave as stated for the extension (c) outcome in force.
- Audit: A10's verifier and ADM-31's banner agree at all four checkpoints in every run. The tampered copy is reported broken at the planted row by both. The recorded head equals `GET /v1/audit/head`. The count of audit events per step equals the owning APIs' count. Every agent-written event carries `on_behalf_of`.
- Perimeter: at the end of every run `offers.wholesale` is on and bound, the four other launch flags are off and unbound, and the launch-state confirmation is recorded with an assertion id. The SUPER toggle request is refused with `G21_NO_BINDING`, leaves either no row or a `rejected` row as settled in spec point (i), never appears in a checker's inbox, and writes no `flag_change`.
- Agent access: Ben's staff token is issued only after the hardware-attested request and SECO's approval. Its MCP reads equal Ben's session reads row for row in every pair taken at the same instant. Its three out-of-authority calls are refused with the named codes. Its first call after revocation is refused on the `api` container and the `mcp` container in 20 of 20 repetitions each. No scan finds the secret anywhere.
- Sponsor re-run: A65's J11, J12, J13, J14 and J19 pass unchanged with `OPERATOR_STEPS=console`. The network log shows zero `sdk.ts` calls for implemented methods. Any unimplemented method is listed as a finding with its owning node, and the list is empty at the gate.
- Captures: 8 gated pairs (ADM-01, ADM-30 with APR-2292 selected, and ADM-14 before and after, at 1280 px and 1024 px) pass the computed-style and text checks, and every region above 5 % pixel difference cites a specific finding. The recorded comparisons exist for ADM-17, ADM-31 and the 12 EXT-B frames. The design folder hashes are identical before and after.
- `app/evidence/A78.json` holds the three runs with their sub-runs and every table named in §12. `A78.qa.md` records no open critical or high finding. Only the owned path changed, plus the registered aliases and check registrations. `npm run verify` is green.

## Human gates touched

- None crossed. Every signature and approval uses A11's simulated hardware authenticator with the sandbox attestation and A15's no-value keystore on the local Hardhat chain. Every payment effect stays in the sandbox bank. Every registry instruction goes to the registry sandbox, and the personas are fictional.
- **HG-KEYS / HG-PROD (at go-live, not now).** Real staff hardware keys, the production AAGUID allow-list, the V1 and V2 signer ceremonies and the production console origin come from A11's, A15's and A53's ceremonies. Until then the console shows `SANDBOX`, and this node proves only that the flow is right with sandbox keys. Which people hold the V2 slots in production is a founder choice A15 marks on ADM-13; this run uses the sandbox default of spec point (f).
- **HG-LEGAL (recorded by the owning nodes, not decided here).** These questions are recorded by the nodes that own them: the DAF Act meter limits and the 80 % alert (A21); the complaint clocks (A44); the SLA and ratification windows (A13, A14); whether the trustee's approval of a register adjustment satisfies the trust deed (A18, A73); and which declarations must be personal under D04 (A03's `delegable.ts` rows marked `"counsel"`). The run reads whatever those rows currently say.
- **HG-VENDOR (later).** Custody, registry, bank and identity are sandboxes behind A08's adapters. A contracted vendor must pass their conformance suites, and this node's run must pass unchanged, before it may be selected.
- **HG-PENTEST (later).** The console origin, the approval modal, the staff-token flow and the kill-switch paths are in scope of the external penetration test. This node passes on software evidence.
- AM7 has no HG-REVIEW in the graph. The guide lets the user walk the console if they wish, and any notes become fix items on the owning nodes without blocking this gate.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
