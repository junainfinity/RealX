---
title: "A100 - Agent-parity suite: every UI action has an API and MCP equivalent under the same gates"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am8]
node: A100
milestone: AM8
lane: "Quality"
oracle: no
human_gate: none
ui: no
---

# A100 - Agent-parity suite: every UI action has an API and MCP equivalent under the same gates

**The proof founder decision D04 asks for, action by action: a generated parity matrix that lists every action a person can take on any screen of `apps/web` (investor, sponsor, partner) and `apps/console` (staff), maps each one to its OpenAPI operation and to its MCP tool (or, for acts only a person may do, to A104's `excluded.json` entry with the matching reason), and keeps three columns that must all be green: reachable from the UI, from the REST API under a token with the matching scope, and from the MCP tool. For every operation the suite runs the same request as a passkey session, as a mandate token and as an MCP call from the same starting state, and asserts that the gate evaluations, reason codes, state changes and audit events are identical except for who acted. Every agent refusal (out of scope, over a limit, non-delegable, revoked, kill switch) must name the missing scope or cap. The static part of the suite runs inside `npm run verify`, so no later node can merge a UI action that has no row, no operation, or no MCP tool (or matching A104 exclusion).**

**Build status:** not started. **Milestone:** AM8 (Journeys, hardening and launch). **Lane:** Quality. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops). Raised from 6-10 by the 24 Sep 2026 plan review: the parity manifest covers every UI action through REST and MCP under the same gates.

**Depends on:** [[A55 - Public site- how it works, returns calculator, transparency, help and documents]], [[A59 - AUD wallet, token wallet and account screens]], [[A61 - Docker Desktop preview of the investor app (AM5 exit)]], [[A65 - Sponsor journeys J11, J12, J13, J14, J19 end to end in Docker (AM6 exit)]], [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]], [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]], [[A104 - MCP server generated from the OpenAPI document]], [[A99 - Developer surface- API versioning, reference docs, per-user sandbox and SDK-MCP packaging]]
**Unlocks:** [[A88 - P1 full regression- 18 P1 journeys, three clean runs, every gate exercised]]
**Graph note for the plan reviewer:** A55 (PUB-02, PUB-07, PUB-08, PUB-09 and the calculator operation) and A59 (POR-05 withdraw, POR-06, ACC-01 to ACC-04, used by J15) are not ancestors of A100 in `final-graph.json` today, although this node's rows, journeys and challenges need them. The edges A55 → A100 and A59 → A100 must be added to the graph (both are AM5, so the milestone order holds). Until they are, the orchestrator must not start A100 before A55 and A59 have gated.

**Oracle rule:** this is not an oracle node (`oracle: no`). No money, supply or holdings move because of this node: it only drives actions that other nodes own. Parity is still the D04 promise, so the node QA writes hidden challenge material from this frozen spec to `<workspace>\gauntlet\hidden\A100\` before the build starts. That material is the planted-break fixtures in section "Gauntlet loop", the expected-class table for a sample of 60 actions and the node QA's own normaliser for cross-checking the comparator. The builder never reads that folder, and the material is published into `app/` only after the gate passes ([[App Gauntlet]] §2).

What this node is for in one sentence: A82 proves that an agent can finish whole journeys. This node proves, one operation at a time and for every screen, that nothing a person can do is missing from the API and MCP, and that nothing behaves differently when an agent does it. The two nodes never share fixtures or helpers, so a defect cannot hide in shared code. This node adds no route, no screen, no scope and no table. This plan is not legal advice; anything that depends on counsel is recorded as a counsel question.

## Build brief

**1. What this node reads (it enumerates, it never re-declares)**

- The API side: `packages/sdk/openapi.json` from A09. Every operation carries `operationId`, `x-realx-scope`, `x-realx-gates`, `x-realx-screens`, `x-realx-mutation`, `x-realx-money` and `x-realx-status`. The suite also reads A11's `requiresHuman` route flag, A03's `delegable.ts` (`true`, `false` or `"counsel"`) and A03's permission matrix (which roles hold which cell), and A00's `docs/charter/agent-access.json` (the `nonDelegable` and `counselFlagged` action codes and the principal types).
- The MCP side: the tool catalogue that A104 generates in `packages/mcp`, read through `loadCatalogue()` from `@realx/mcp/catalogue` (tool name, input schema, `readOnly` or write, declared gates, idempotency key field, `enabled`), and A104's `excluded.json` (every operation that is not a tool, with one reason: `stub`, `test_only`, `service_only`, `requires_human`, `non_delegable_scope`, `pending_counsel`, `agent_never_checker`). A104 is frozen on this point: no `requires_human`, `non_delegable_scope`, `pending_counsel` or `agent_never_checker` operation is ever a tool. The suite also uses A104's MCP client over both transports, stdio and HTTP.
- The UI side, from four independent sources, so that one missing source cannot hide an action:
  - (a) **Static call sites.** A TypeScript compiler-API scan of `apps/web` and `apps/console` finds every call to `@realx/sdk` (`RealXClient` methods and `paginate`). It resolves each call to a method and path template, then to an `operationId` by matching against `openapi.json`. It builds the import graph from each Next.js route file (`page.tsx`, `layout.tsx`, route handlers) to its call sites, so every call site is attributed to a route and so to a product-map screen code.
  - (b) **The UI nodes' action manifests**, harvested and normalised into one row shape. The upstream nodes use five file formats, so the harvest is one union glob plus three named exceptions, all frozen in `e2e/parity/sources.ts`:
    - Union glob: `apps/{web,console}/src/{shell,features/*}/{actions.ts,actions.manifest.ts,parity.ts,parity.json,parity-manifest.json}`.
    - Exceptions: `app/evidence/A54.json` (A54's parity map, `parityMap` section only), `app/evidence/A65.actions.json` (A65's browser-mutation list, used as a cross-check source, never as the only source of a row) and `services/api/src/modules/agent-guard/actions.agent-guard.ts` (A101's audit action codes and labels, used only to label ADM-35 and ACC-05 activity rows).
    - Known layout at the frozen commit (the normaliser has one reader per format and nothing else):

      | Format | Nodes and paths |
      |---|---|
      | `actions.ts` (TypeScript export, read through the compiler API, never executed) | A52 `apps/web/src/shell/actions.ts`; A57 `apps/web/src/features/buy/actions.ts`; A63 `apps/web/src/features/sponsor-listing/actions.ts`; A67 `apps/console/src/features/access/actions.ts`; A71 `apps/console/src/features/offerings/actions.ts`; A74 `…/features/money/actions.ts`; A75 `…/features/risk-records/actions.ts`; A76 `…/features/reporting-integrations/actions.ts`; A77 `…/features/agent-access/actions.ts` |
      | `parity.ts` (same reader) | A53 `apps/console/src/shell/parity.ts`; A66 `…/features/command/parity.ts`; A69 `…/features/sponsors-perimeter/parity.ts`; A70 `…/features/properties/parity.ts`; A72 `…/features/token-custody/parity.ts`; A73 `…/features/rules-recon/parity.ts` |
      | `actions.manifest.ts` (same reader) | A56 `apps/web/src/features/onboarding/actions.manifest.ts` |
      | `parity.json` | A58 `features/portfolio`; A59 `features/wallet-account`; A60 `features/agent-access`; A62 `features/sponsor-admin`; A64 `features/sponsor-operations`; A68 `apps/console/src/features/investors`; A105 `apps/web/src/features/partner` |
      | `parity-manifest.json` | A55 `apps/web/src/features/public-info/parity-manifest.json` |

    - Each format maps to the normalised entry `{ node, screen, actionKey, label, role, operationId, mcpTool, scope, delegable, journeySteps }`. `label` is the visible text or accessible name of the control and `role` its ARIA role (default `button`); where a manifest has no `label`, the normaliser takes it from the A50 or A51 `screens.json` action with the same operation id, and failing that the entry is `ACTION_UNLABELLED` against the owning node. `mcpTool` is optional in the source (A104's tool name is the operation id) and is always re-derived from the catalogue.
    - A test in `sources.test.ts` reads A00's `app-graph.json` and asserts that every UI node (A52 to A78, A105) resolves to at least one manifest file. A feature folder under `apps/*/src/features/*` with none of these files is `MANIFEST_MISSING` against the node that owns the folder. A file in a sixth format is `MANIFEST_FORMAT_UNKNOWN`, never skipped.
  - (c) **The design's drawn actions.** These come from `screens.json` of the design extensions (A50 for investor, sponsor and partner, including ACC-05 and SPN-16; A51 for the console, including ADM-35), with `operation`, `mcpTool`, `gates` and `delegation`. They also come from an inventory of the prototype's non-navigation handlers in `<workspace>\design\RealX App.dc.html`, extracted read-only and checked in with the file's sha256.
  - (d) **The product map's `actions` tuples.** Each screen's action label, note, gates and on-chain flag is taken from the generator data (`data_investor.py`, `data_sponsor.py`, `data_admin.py`). A standard-library Python script reads them once, read-only, and writes `docs/parity/source/product-map-actions.json` with the source hashes, so `verify` never needs the vault on disk.
- Runtime capture: during the reachability crawl (§4), a Playwright request listener records every `/v1` request the page makes (method and path, matched to an `operationId` through the OpenAPI path templates). The static scan, the manifests and the runtime capture must agree. A call captured at runtime with no static call site, or a static call site never reached by the crawl, is a finding, not something the suite silently accepts.

**2. The UI-bypass lint rule (why the list is complete)**

- Rule `realx/sdk-only-io` is written in `e2e/parity/lint/`. It does not touch the root `eslint.config.js` and needs no reopen of A01: `e2e/parity/lint/run.ts` runs it through the ESLint Node API with its own flat config (`e2e/parity/lint/eslint.parity.config.js`) over `apps/web` and `apps/console`, and `parity:matrix` calls it as its first step. The `parity:matrix` and `parity:run` scripts and the `parity:matrix` verify step (group `check`) are registered through A01's `scripts:sync` convention from the `realx` block of the `e2e` workspace's `package.json`. That file sits outside this node's owned paths, so at Specify the orchestrator names its owner and the three entries are added there by that owner (or through its follow-up loop); this node never hand-edits it.
- **Dry run first.** At the start of the build the rule is run once over `apps/web` and `apps/console` at the frozen commit. Each hit becomes either an allow-list entry with a vocabulary reason (below) or a fix item on the owning node, recorded in `app/evidence/A100/lint-dry-run.json`. This covers A52's middleware and any server-side SDK forwarding before the rule is enforced.
- Inside `apps/web` and `apps/console` it forbids:
  - `fetch`, `XMLHttpRequest`, `axios` or any HTTP client import, `EventSource`, `WebSocket` and `navigator.sendBeacon`;
  - `createClient` from `openapi-fetch` outside `packages/sdk`;
  - computed SDK access (`client[method]`, `client[op]`), because a computed name cannot be enumerated;
  - `'use server'` files and route handlers that import anything other than `@realx/sdk`, `packages/domain` pure functions or `packages/ui`;
  - direct imports of `@realx/db`, `pg` or any `services/*` path;
  - JSX `<form action>` and `formAction` that point at `/v1` or at any absolute URL;
  - `<a href>` and `router.push` to a `/v1` path.
- There is a small allow-list, `e2e/parity/lint/allow.json`. Each entry names one file or one call site and one reason code from this closed vocabulary (no other code, no free text, no catch-all):
  - `SDK_PACKAGE`: `packages/sdk` itself.
  - `PRESIGNED_LINK_FROM_READ`: an `<a href>` to a presigned document URL that an SDK read returned (the link is data from an operation that has its own row).
  - `AUTH_MIDDLEWARE`: A52's or A53's session middleware forwarding the session cookie to the API through `@realx/sdk` on the server (listed per file after the dry run).
  - `EXTERNAL_LINK`: a plain link to an external site with no business effect (help centre, AFCA, regulator pages).
- A55's calculator needs no entry: the page imports the pure `computeIllustration` from `packages/domain`, which the rule already allows, and the operation twin is `GET /v1/public/calculator` (see §3).
- The rule is case-insensitive on path segments, like A09's boundary rule, so a Windows checkout cannot get around it. A planted example of each forbidden form is kept in `e2e/parity/lint/__fixtures__/` and must fail the rule.

**3. The parity matrix (`docs/parity/`)**

- `matrix.json` has one row per (surface, screen code, action). Each row holds:
  - the product-map screen code, reconciled with A00's `screens.json` (design `port` "INV-01" is POR-01, design `buy` steps are INV-01, INV-02 and INV-03, design `activity` "INV-03" is POR-04);
  - the action label as drawn or as in the product map;
  - `operationId`, the MCP tool name, `x-realx-scope` (the A03 cell), gates, `mutation`, `money` and on-chain flag;
  - the principal types that see the action (from A03's matrix and the route's role guard: investor sub-types individual, joint either, joint both, SMSF, company, trust, wholesale, sophisticated, foreign; sponsor SP-OWN, SP-OPS, SP-FIN, SP-IR; tenant partners SP-VAL and SP-PM; platform partners TP-RE, TP-AUD, TP-OBS and TP-LIC; every staff role that holds the cell);
  - the journeys and steps that use it;
  - the owning node, found from the call site's path through A00's `app-graph.json` owned paths;
  - the expected agent class;
  - the UI locator `selector = { screen, role, name, actionKey }` (see §4);
  - for a `later` row, the later node that will build it (`laterOwner`);
  - the three column results `ui`, `api` and `mcp`. The column vocabulary is closed: `pass`, `fail`, `pending` (the operation is `x-realx-status: stub`), `later` (a P2 or P3 action with no built surface yet), `awaiting_enablement` (MCP column only: the tool exists but A104 reports `enabled: false` under the opt-in reading of A104's spec point (a)), `n/a (excluded: <reason>)` (MCP column only: the operation is in A104's `excluded.json` with a reason that matches the row's class, see below) and `n/a` (presentation and exempt rows). Each result carries an evidence reference.
- **Agent classes** (a fixed vocabulary, derived from data and never hand-set):
  - `delegable`: the agent must succeed exactly as the person does.
  - `confirmable`: the design asks the person for a passkey confirmation, as with order confirm on INV-02 and a withdrawal request on POR-05. The agent must succeed under a mandate signing key inside its caps, or get `428 CONFIRMATION_REQUIRED` with a `confirmation_id` that the person resolves, after which the outcome equals the session run.
  - `maker`: a staff or sponsor act that creates an approval request or a draft. The agent's request must carry the same `payload_sha256` as the person's, and the maker identity counts the agent as its human.
  - `human_only`: approve, ratify, veto, execute or sign, countersign, the D04 non-delegable acts, and A11 `requiresHuman`. REST under a token must be refused with the specific code (`AGNT_NOT_ALLOWED`, `G22_AGENT_NEVER_CHECKER`, `SCOPE_NON_DELEGABLE`, `APR_CHECKER_MUST_BE_HUMAN`), while the session succeeds for the right role. The MCP column is `n/a (excluded: <reason>)` and passes only when all three hold: (1) the operation is in A104's `excluded.json` with a reason that matches the row's derived class (`requires_human` for A11 `requiresHuman`, `non_delegable_scope` for `delegable.ts` `false`, `agent_never_checker` for a checker cell); (2) no catalogue tool maps to the operation; (3) a `tools/call` with the would-be tool name (the operation id) over stdio and over HTTP returns `MCP_UNKNOWN_TOOL` with no forwarding and zero business effect (row counts unchanged).
  - `counsel`: a `"counsel"` row in `delegable.ts` (CQ-01: wholesale-status truthfulness, target-market answers and the others A00 lists). REST behaves as `human_only` with `SCOPE_PENDING_COUNSEL` until counsel answers. The MCP column is `n/a (excluded: pending_counsel)` under the same three conditions. It flips by data alone: when A03's row changes, A104's regeneration moves the operation into the catalogue and this row becomes `delegable` or `human_only` with no code change here.
  - `phase_flagged`: an action that already has a P1 route or operation but sits behind a phase or licence flag (A08, A21). All three paths are refused with the same flag code (`PHASE_NOT_ENABLED`, `G21_FLAG_OFF` or `G21_PHASE_NOT_REACHED`, whichever A21's reason catalogue names). Each such row names the node that owns the flagged route. Known candidates at the frozen commit, confirmed at Specify from the owners' specs (the same precondition A88 uses): a stablecoin payout or withdrawal destination request on A39's withdrawal route reached from POR-05 (A59), and any mutating control on A52's INV-05 preview page if it calls an operation at all (A52's route table says it must not).
  - `later`: a P2 or P3 product-map action with no built surface and no operation yet (for example INV-05 "Place sell order" on the live board, INV-06, votes, Auto Invest). No operation is required. The row names `laterOwner` (A90 to A97) and is excluded from this node's green count. Each of those later nodes must re-run `parity:run --rows <its rows>` at its own gate so that its `later` rows turn green; this is a requirement on A90 to A97, recorded here for the plan reviewer, and their specs must state it.
  - `presentation`: a client-only interaction on already-loaded data. Allowed only for these interaction kinds (closed list): `TAB`, `LOCAL_SORT`, `LOCAL_FILTER`, `OPEN_DRAWER`, `CLOSE_DRAWER`, `EXPAND`, `COPY`, `NAVIGATE` (a `go.*` handler or a link to another screen with no call of its own), `LOCAL_COMPUTE` (a pure `packages/domain` function whose operation twin is named, see A55's calculator below). The crawl must observe no `/v1` call for it. A product-map action that names a filter or sort must still exist as an API query parameter, so an agent gets the same ordering. `LOCAL_COMPUTE` rows are not exempt from parity: the named operation twin gets its own row, and the `ui` cell passes only when the page's displayed result for the crawl's inputs equals the operation's response for the same inputs.
  - `exempt`: traffic with no user intent. Closed reason vocabulary: `BROWSER_GENERATED` (for example A83's CSP report endpoint), `SYSTEM_REACTION` (a server-side state change the person did not trigger, such as a status poll or a webhook), `SDK_PACKAGE`, `PRESIGNED_LINK_FROM_READ`, `EXTERNAL_LINK`. Each exempt row names the node that declared it.
  - No other class and no other reason code exists. A free or catch-all code (`OTHER`, `MISC`, `TODO`, free text) fails generation.
- **Class derivation (never hand-set).** In this order: `exempt` and `presentation` from the vocabularies above; `later` when the product-map tuple is P2 or P3 and no operation or route exists; `phase_flagged` when the operation carries a phase or licence flag that is off at the frozen commit; `counsel` when `delegable.ts` is `"counsel"`; `human_only` when A11 `requiresHuman`, `delegable.ts` `false`, the cell is a checker cell in A03's matrix, or the A00 non-delegable code applies; `maker` when the operation creates an A14 approval request or a draft; `confirmable` when the operation can return A12's `428 CONFIRMATION_REQUIRED`; otherwise `delegable`. For G12 acknowledgements the class and expected result follow A00's CQ-02 flag (see §7), never a constant in this suite.
- **Class-independent invariants (why a wrong delegability value cannot slip through).** Parity by difference cannot catch an operation that A03 wrongly marks as delegable, because person and agent would then both succeed. So `parity:matrix` also fails with `CLASS_CONFLICT <operationId>` against A03 when either holds:
  - an operation that writes A14's decision, ratification or veto tables, A15's custody signature table, or records the trustee's issue decision through A18 (found from A14's, A15's and A18's route metadata and repository write sets, not from `delegable.ts`) derives to any class other than `human_only`;
  - a product-map or drawn action whose label is Approve, Reject (as checker), Ratify, Veto, Countersign, Sign or Broadcast (a closed list of label stems in `sources.ts`) derives to any class other than `human_only`.
  - At run time the same rule is checked on data: after every shard of `parity:run` and `parity:full`, a direct read-only database query must find zero rows in A14's decision, ratification and veto tables and zero A15 custody signatures whose principal is `AGNT` or that carry a `token_id`. One such row is Critical and stops the milestone.
- **Product-map tuple shapes.** Pseudo-actions in the product map map as follows (a closed table in `sources.ts`): "(on approval)", "(automatic)", "(on schedule)" and "(system)" are `SYSTEM_REACTION` exemptions attributed to the operation that raised them; "Opens …" and "Goes to …" are `NAVIGATE` presentation rows pointing at the target screen; "Download …" and "View …" map to the read operation that serves the file or view; every other tuple must map to an operation or be `later`. An unmapped shape is `TUPLE_SHAPE_UNKNOWN`, never a silent skip.
- **A55's calculator.** PUB-07 has two rows: the page's calculation (`presentation`, `LOCAL_COMPUTE`, twin `GET /v1/public/calculator`) and the operation `GET /v1/public/calculator` itself, which is in `openapi.json` and in A104's catalogue under the `public:read` scope like any other read. The `ui` cell of the first row passes only when the displayed result equals the operation's response for the same inputs.
- `matrix.md` is the same data for people, grouped by surface and screen. Its status words match the design's TONE map (`<workspace>\design\RealX App.dc.html` line 740): green all three columns pass (or `n/a (excluded: …)` that passed its three conditions), amber `pending`, `phase_flagged`, `later` or `awaiting_enablement`, red any fail or missing, grey `exempt` or `presentation`, brick `counsel`. `summary.json` holds the counts per surface, class and owning node. `sources.lock.json` holds the sha256 of every input (`openapi.json`, the MCP catalogue and `excluded.json`, `delegable.ts`, the A03 matrix, `agent-access.json`, A00's `counsel-questions` flags, both `screens.json`, the prototype, the product-map extract, the manifests). `fix-items.json` holds one item per red cell with owning node, screen, action, the missing column or the diff, and the evidence path. `exemptions.json` holds every non-row, each with its reason code.
- `npm run parity:matrix` is the static part. It covers the lint rule, enumeration, manifest harvest, class derivation and the invariants above, the design and product-map cross-checks, and the MCP check (every mapped `operationId` of a non-excluded class has exactly one tool with the same name, scope and gates, write tools declare the idempotency key, and every excluded operation carries the reason that matches its row's class). It runs in under 90 seconds and fails when:
  - a UI action (call site, manifest entry, drawn action or P1 product-map tuple) has no row;
  - a row other than `presentation`, `exempt` and `later` has no `operationId`;
  - a row has no MCP tool and is not a `presentation`, `exempt` or `later` row or an A104-excluded row whose exclusion reason matches its class; an exclusion with a non-matching reason (for example a delegable UI action excluded as `service_only`) also fails;
  - an exemption, presentation kind or allow-list entry uses a code outside the vocabularies above;
  - a `CLASS_CONFLICT`, `TUPLE_SHAPE_UNKNOWN`, `MANIFEST_MISSING`, `MANIFEST_EMPTY`, `MANIFEST_FORMAT_UNKNOWN` or `ACTION_UNLABELLED` finding exists.
- **Regeneration rule (so later nodes can add actions without touching this node's paths).** Inside `npm run verify`, `parity:matrix` regenerates the matrix in memory and writes its working output only to the operating system's temporary folder, never to a tracked path, as A01's hygiene step requires. It never fails on a newly added row that is fully mapped, and never on a difference from the checked-in `docs/parity/matrix.json`; that difference is printed as information (`matrix drift: N rows added, M changed`). The checked-in `docs/parity/` files are this node's gate snapshot, rewritten only by this node's own fix loops or follow-up loops (`npm run parity:matrix -- --write`). A later node that adds or changes UI actions records its own rows in its own evidence path with `npm run parity:matrix -- --node <id> --out app/evidence/<id>/parity-rows.json`. So a later UI node stays inside its owned paths, and a UI-only action still cannot merge, because an unmapped action fails verify.
- `npm run parity:run` is the dynamic part (§4 to §6), run in this node's Challenge on the reduced matrix of §5. `npm run parity:full` runs the full principal cross-product and every refusal variant; it runs at this node's gate once and afterwards in A88 and nightly. Neither runs on every verify.

**4. UI reachability (`ui` column)**

- One Playwright crawl per persona runs over the AM7 compose stack (A61's profile extended by A65 and A78), seeded with A06's dataset and a reset baseline. The personas are the ones listed in §3. Sign-in uses A11's simulated authenticator: passkey for external users, hardware-key flow for staff.
- **Locator contract (frozen).** No upstream node emits a `data-action` attribute, so the crawler does not depend on one. It locates each manifest action by ARIA role and accessible name, taken from the normalised entry's `role` and `label` (§1(b)), scoped to the screen's main landmark: `page.getByRole(role, { name: label, exact: true })`. The locator is stored in the row as `selector`. Every interactive control must already have an accessible name under WCAG 2.2 4.1.2, which A87's accessibility pass checks, so this adds no new upstream contract; a missing name is both an `ACTION_UNLOCATABLE` finding here and an accessibility defect on the owning node. Where the same role and name appear more than once on a screen, the manifest's `actionKey` is matched against an existing `data-testid` (A05's components all accept one) as a tie-breaker; if that also fails, the entry is `ACTION_AMBIGUOUS` against the owning node. An entry with no match at all is `ACTION_UNLOCATABLE` against the owning node. A `data-action` attribute, if a node adds one later, is never required.
- For each screen in the persona's route list (A00's `screens.json`), the crawler visits the route and triggers each manifest action through its `selector`. The crawler records the `/v1` calls made. The cell passes when the expected `operationId` is observed for that action and no unexpected operation is observed.
- Human-only actions are driven up to the passkey or hardware-key prompt, and the prompt's payload hash is recorded. Mutation actions run on the reset baseline. The crawl never needs real money, a vendor or a chain beyond the local Hardhat node.

**5. Triple runs and the comparison (`api` and `mcp` columns)**

- For every row whose class is not `presentation`, `exempt` or `later`, and for each principal chosen by the run matrix below, the harness runs three twins from the same starting state:
  - (i) the person's session calling the operation through `@realx/sdk` with the same parameters the crawl captured;
  - (ii) the REST call under a mandate token whose scope is exactly the row's cell, with caps wide enough for the seeded amounts;
  - (iii) the MCP tool call through `packages/mcp` with that same token, over stdio for half of the shards and over HTTP for the other half, swapped between runs. For `human_only` and `counsel` rows the MCP twin is the unknown-tool probe of §3.
- **Run matrix (explicit, so the budget is real).**
  - `parity:run` (Challenge): every row runs under one representative principal per principal family that holds the cell (investor, sponsor, partner, staff; the representative is fixed in `fixtures/representatives.json`, rotated between the three runs). `money`, `maker`, `human_only` and `counsel` rows run under the full cross-product of principal types they list. Refusal variants (§6) run once per operation family (the A03 scope-cell family, for example `orders:*`) and exhaustively for every `money` row. Kill-switch tiers run 5 repetitions each.
  - `parity:full` (once at this node's gate, then in A88 and nightly): every row under every principal type it lists, every refusal variant on every row, and 50 repetitions per kill-switch tier.
  - Budget: the build starts with a measured pilot of 50 rows (recorded in `app/evidence/A100/pilot.json`: seconds per twin, per reset and per shard, and peak Docker memory). The shard count is chosen from the pilot (default 4, at most 8) so that `parity:run` finishes in under 60 minutes on the local Windows 11 Docker Desktop stack. `parity:full` has no fixed ceiling; its measured time is recorded, and if it exceeds 8 hours the orchestrator is told before the gate rather than the matrix being cut.
- **Isolation topology (inside `e2e/parity/twins/`, no hook in any other node).**
  - Each shard is its own compose project, `realx-parity-<n>`, started from A23's pinned images (`infra/stack/images.lock.json`) with this node's override `e2e/parity/twins/compose.shard.yml`: one Postgres, two API replicas behind the shard's own port (the kill-switch, revoke and rate-limit cases need two processes), one worker, one MCP HTTP server, one Hardhat node, the sandbox panels and the shard's own sandbox storage and KMS volumes. Every published port is `127.0.0.1` and offset per shard. The shared `realx` project of A23 is never touched.
  - Each shard runs A23's `migrate`, `kms-init`, `deploy` and `seed` one-shots once, then snapshots the seeded database as `realx_parity_template` and takes an `evm_snapshot` on its own Hardhat node.
  - Reset between twins: stop the shard's API replicas, worker and MCP HTTP; `DROP DATABASE` and `CREATE DATABASE … TEMPLATE realx_parity_template`; `evm_revert` to the shard's snapshot and take a fresh one (only where the row touches the chain); reset every sandbox adapter through A08's `SandboxControl.reset(seed)`, reached through the `/sandbox/<adapter>/*` routes A09 mounts in `RUN_MODE=test`; restart and wait for A23's health report. Because each shard has its own Hardhat node and database, shards cannot revert each other.
  - A09's injectable clock is pinned per twin to the same instant.
  - If a reset needs anything these hooks do not give (for example an adapter whose state is not covered by `reset(seed)`), the gap is a fix item on that adapter's node, not a hook written here.
- **Tokens and rate limits.** Tokens and mandates are minted by this node's own fixture factory in `e2e/parity/fixtures/`. It uses A12's public routes with simulated fresh assertions and shares nothing with A82's fixtures. A fresh token and mandate is minted for every twin, so bulk runs never exhaust one token's rate limit. The harness records every `429` at the transport layer, including ones the SDK retried; a `429` outside the rate-limit case below is `HARNESS_RATE_LIMITED` (a harness defect to fix here), never `NONDETERMINISTIC`. One explicit rate-limit parity case exceeds a token's `rate_limit_writes_per_min` on each API replica: REST and MCP must both return `429 RATE_LIMITED` with the same `Retry-After` and `RateLimit-*` values, and the person's own session bucket must be unaffected.
- Confirmable rows run twice on the agent side: once with a registered mandate signing key inside the cap, and once without a key, where the person resolves the `428` through A12's confirmation route before the comparison.
- The comparator normalises each twin's outcome into:
  - HTTP status, and the error envelope `code`, `gate` and sorted `reason_codes`;
  - the response body, with ids replaced by first-appearance ordinals, timestamps replaced by clock offsets, and request, trace and idempotency ids dropped;
  - `gate_evaluation` rows as (gate, result, reason codes, inputs recomputed through A19's replay interface without the actor fields);
  - state transitions per object;
  - ledger journals (accounts and integer cents, A16);
  - register entries (A18);
  - approval requests (policy code, `payload_sha256`, maker identity set);
  - audit events (type, object, approval id ordinal, before and after hashes);
  - outbox events and notifications.
- **Allowed differences, and nothing else.** They are split into differences that must be present and differences that may be present:
  - Allowed and required on every agent twin:
    - `actor_type = AGNT`, `actor_id`, `on_behalf_of_id`, `token_id` and `channel` (`api` or `mcp`), where the person's twin has `HUMAN` or `PARTNER` and `web`;
    - exactly one G22 `gate_evaluation` row;
    - exactly one `api_token_activity` row.
  - Allowed and required on rows A101 flags as sensitive (withdrawal, investment, vote and the others in A101's notice policy): exactly one owner notice on the agent side, delivered to the person through A43 and naming the token prefix.
  - Allowed and required on confirmable rows: the confirmation step (the `428` and its resolution).
  - Allowed and optional: `agent.*` outbox events other than the notice above.

  Any other difference, and any missing required difference, is a red cell with the diff attached. It is attributed to the node that owns the operation, or to A19 when a gate verdict differs, and never patched inside this suite.
- REST and MCP must also match each other byte for byte after normalisation. That includes the MCP error result, which must carry the same `code`, `gate`, `reason_codes`, `missing_scope`, `cap` and `remaining` as the REST envelope. An MCP tool that turns a refusal into a plain text error fails.
- Reads compare full bodies. The agent's view of a masked field, a need-to-know restriction or a tenant-scoped list must equal the person's view exactly, no wider and no narrower: A30's masking and tipping-off shapes and A04's row-level security apply unchanged. List operations are paged to the end on all three paths, and the item sequences must match. Writes are also replayed with the same `Idempotency-Key` across channels (REST first, then MCP), and must return the stored response with no second effect.
- Every delegable row is also replayed once under the owner's sandbox token (`rx_sb_`, A99) against the person's sandbox partition. The gate verdicts must be identical, and a row count of the live partition taken before and after must show zero live writes.
- **What the person sees of the agent.** After each agent twin (success or refusal), the harness reads the person's activity through the same read operations the screens use: ACC-05's token activity for investors, SPN-16's for sponsor users, valuers and property managers, A105's for platform partners, and ADM-31's audit log read for staff. It checks that the entry exists, that the Actor renders as "agent <token prefix> on behalf of <person>" (the `adAudit` row form, design line 888), and that refusals are listed with their code as well as successes.

**6. Refusal parity (every refusal names what is missing)**

- For every `delegable`, `confirmable` and `maker` row, the harness generates these negative variants and runs each on REST and MCP:
  - the token without the row's cell, which must return `G22_SCOPE_MISSING` naming the cell;
  - for `money` rows, one cent over `per_action_cap_cents`, and the daily cap exhausted by an earlier call, which must return `G22_LIMIT_EXCEEDED` naming the cap and the exact `remaining`;
  - a vehicle or property outside `allowed_vehicle_ids`, which must return `G22_VEHICLE_NOT_ALLOWED`;
  - outside the time window, which must return `G22_OUTSIDE_TIME_WINDOW`;
  - a revoked token, which must return `TOKEN_REVOKED` on the first request after the revoke commits;
  - an expired mandate, which must return `MANDATE_EXPIRED`;
  - each kill-switch tier engaged in turn (token, user, tenant, platform), which must return `423 AGENTS_PAUSED` naming the tier and stop only that tier;
  - the owner's role removed, which must return `G22_OWNER_NO_LONGER_HOLDS_ROLE`.
- **Session-key variants (status `later` until the owning P2 node lands).** For rows with the on-chain flag set that an investor agent can reach (P2 "List for sale" and "Cast vote", A92 and A93), the harness adds: a wrong contract, a wrong function, one unit over the per-transaction ceiling, an expired session key and a revoked session key, each of which must be refused identically on REST and MCP with the code A29 names. On every successful agent twin it asserts that the user operation was signed by the delegated session key registered under A12's `session_key_policy`, never by the owner passkey key. In P1 no investor-agent row touches the chain, so these variants are listed in the matrix as `later` with `laterOwner` A92 or A93, and those nodes' gates run them.
- For `human_only` and `counsel` rows, a token holding every other cell of the owner must be refused on REST before any business effect, with the specific code and the A00 non-delegable action code where one applies. On MCP, the would-be tool name must return `MCP_UNKNOWN_TOOL` with no forwarding (the operation is in A104's `excluded.json`). The same operation must succeed under the session of an eligible human.
- For every G13 act (ADM-30 approve and reject, SPN-09 "Approve as sponsor owner", A105's trustee countersign, ADM-14 custody sign, ratifications and vetoes), the harness also tries, on REST, the agent of the maker, the agent of a different eligible checker and a sibling agent of the same human. All must be refused with `G22_AGENT_NEVER_CHECKER` or A14's `APR_CHECKER_MUST_BE_HUMAN`, and no decision row may exist afterwards. On MCP the same three tokens calling the would-be tool name must get `MCP_UNKNOWN_TOOL`.
- **Segregation of duties counts the agent as its human (D04).** For every `maker` row whose operation creates an A14 approval request, the harness adds one variant: the agent creates the request under its token, then its own human, who also holds an eligible checker role for that policy, tries to decide it in person with a valid hardware-key (staff) or passkey (sponsor) assertion. It must be refused with A14's `APR_MAKER_IS_CHECKER`, and no decision row may exist afterwards. The same request decided by a different eligible human must succeed. Seeded personas with both roles come from this node's fixture factory.
- A refusal envelope without `gate` and `reason_codes`, or a G22 refusal without `missing_scope`, `cap` or the non-delegable code, fails its cell. The refusal's `gate_evaluation`, activity and audit rows must also exist on both agent paths, so the person's ACC-05 or SPN-16 activity list shows refusals as well as successes.

**7. Gates enforced server-side, checked here by difference**

- Gates G01 to G21 are not re-implemented. They are proven identical by the comparator on every row that declares them. The suite adds focused cases where agent paths have historically diverged:
  - G12: the expectation is derived from A00's CQ-02 flag and A12's `standing_disclosure_consent`, never hard-coded here. Under the build default (allowed only where the person has recorded a standing consent that names the document types, and never for the person's first purchase in a trust), the agent's acknowledgement is recorded under the holder's standing consent with the consent id in the audit event; with no standing consent, or on a first purchase in a trust, the agent gets the same G12 refusal the person gets without ticking the AcknowledgeCard (prototype `buy` step 1). If CQ-02 is answered "no", the same cases must all return the G12 refusal, by data alone.
  - G07: the holder cap at exactly 5.00 percent and one token over.
  - G10: an order during a halt.
  - G14: a withdrawal to an account other than the verified one.
  - G16: withholding on a distribution line.
  - G17: a sponsor announcement containing "guaranteed income" submitted by an SP-IR agent, which must get the same claims flags as the person.
  - G19 and G20: a stale report or valuation blocking the same listing action.
  - G06: a foreign person on a residential listing.
- G22 is the only gate that may appear on one side only. It must `pass` on every agent twin that succeeds.

**8. Error and empty states of the suite itself**

- A row whose owning operation is still a stub (`x-realx-status: stub`) is `pending` with the owning node named. At this node's gate, zero `pending` rows are allowed for P1 screens.
- A P2 or P3 action with no built surface is `later` with its `laterOwner`; it is never `pending` and never red at this node's gate. A `later` row that names no owner, or names a node outside A90 to A97, fails generation.
- A screen with no actions (a static information screen) still gets a row set of its reads.
- An empty manifest file is `MANIFEST_EMPTY`, not a pass.
- If the stack is not healthy (A23's health report), the run stops with `STACK_UNHEALTHY` and writes no column results. A partial run never overwrites a green matrix.
- Flaky outcomes are never retried silently. A twin that differs between two runs from the same snapshot is recorded as `NONDETERMINISTIC` against the owning node.

## Owned paths (other nodes do not edit these)

- <workspace>\app\e2e\parity\
- <workspace>\app\docs\parity\

Inside `e2e/parity/`, by convention:
- `sources.ts` (manifest globs and source readers)
- `enumerate/` (compiler-API scan, route attribution, OpenAPI path matching)
- `lint/` (`sdk-only-io` rule, `allow.json`, planted fixtures)
- `extract/` (the read-only prototype handler inventory and the product-map extract script)
- `crawl/` (Playwright personas and the request listener)
- `twins/` (session, REST and MCP runners, the shard compose override `compose.shard.yml`, template-database cloning and reset, sandbox replay)
- `compare/` (normaliser, allowed-difference list, diff writer)
- `refusals/` (variant generator, kill-switch tiers, checker attempts)
- `fixtures/` (principal and mandate factory, independent of A82, and `representatives.json`)
- `matrix/` (generator, attribution, markdown writer)

Inside `docs/parity/`: `matrix.json`, `matrix.md`, `summary.json`, `sources.lock.json`, `fix-items.json`, `exemptions.json`, `source/product-map-actions.json`, `source/prototype-actions.json`. These are this node's gate snapshot; `verify` never rewrites them, and later nodes record their own rows in their own evidence path (§3, regeneration rule). The root `eslint.config.js` is not touched (the lint rule runs from `e2e/parity/lint/` with its own config). The `parity:matrix`, `parity:run` and `parity:full` scripts and the verify step are registered through `scripts:sync` from the `e2e` workspace's `package.json`, whose owner adds them (§2); this node never hand-edits that file or A01's root files.

## Design references

- ACC-05, SPN-16 and ADM-35 are not drawn in `<workspace>\design\RealX App.dc.html`. Their action lists come from `design-ext:ACC-05` and `design-ext:SPN-16` ([[A50 - Design extension A- investor and sponsor screens not in the prototype]]) and `design-ext:ADM-35` ([[A51 - Design extension B- operator console screens not in the prototype]]), through each extension's `screens.json` `delegation` values. The expected classes on those screens are:
  - `human_only`, non-delegable: "Create token", "Change scopes or limits" when widening, and "Allow on-chain actions".
  - Delegable, protective actions within the token's own family: "Change scopes or limits" when narrowing, "Revoke token", "pause all my agents", SPN-16 "Revoke a team member's token" for the owner.
  - `maker` or `human_only` as A03 says: SPN-16 "Set sponsor agent policy", and ADM-35 "Create my token with write scopes" and "Change platform agent policy". ADM-35 "Revoke any token" also follows A03's value.
  - Human-only in every case: ADM-35 "Suspend all agent access" (an `SGL+R` protective act whose single actor and ratifier must be human; A14 refuses it to tokens), and the release of the `agents.traffic` switch.
  - These are the classes the node QA expects; the suite still derives each one from data (§3), and a mismatch is a finding, not a hand-set override.
- The prototype's action handlers that must each map to a matrix row are listed below with the class the node QA expects. The suite derives the class from data; the list is a cross-check, not a source.
  - `buyNext` "Confirm and pay", lines 446 and 906, INV-02, `confirmable`;
  - `withdraw`, lines 471 and 908, POR-05 (A59), `confirmable`;
  - `placeSell`, line 494, INV-05, `later` (owner A92), unless A52's P1 preview page calls an operation behind a flag, in which case `phase_flagged` owned by A52;
  - `approveDist` "Approve as sponsor owner", line 601, SPN-09, `human_only` for agents, since the second sponsor person must be human;
  - `lNext` at SPN-06, line 912, is two rows: the sponsor declaration at submission (`counsel` until CQ-01 is answered, then whatever A03's data says) and the listing submit (`maker`). The submit is gated on the declaration having been made by the person, so an agent can submit only after its human has made the declaration in person;
  - `apApprove` and `apReject`, lines 675 and 676, ADM-30, `human_only`;
  - `kycClear`, `kycEsc` and `kycRej`, lines 696 to 698, ADM-10, `maker` or draft through A14 and A30;
  - the token-operation Sign and Broadcast button (`tokenOps`, line 859), ADM-14, `human_only` custody signing.

  `go.*` handlers are navigation and map to the `presentation` class (`NAVIGATE`). The inventory is keyed by handler name and line hash, so a change to the design file shows up as a lock mismatch rather than a silent gap.
- The comparison of audit events uses the `adAudit` row form (line 888: Time, Actor, Action, Object, Approval, Result). Agent rows must render "agent <token prefix> on behalf of <person>" in the Actor cell. Refusals must carry `gate` and `reason_codes` because the buy flow's GateRows "name the failed check" (design analysis §3.1, `buy` step 1). The status words in `matrix.md` use the TONE map (line 740: green `#1F5E4B` on `#E3EEE8`, amber `#7A4F00` on `#F6ECD6`, red `#A3312A` on `#F6E1DE`, grey `#4A4D52` on `#ECE8E0`, brick `#9A4524` on `#F5E4DB`), so the matrix reads like the console's own pills when A99's docs site links it.
- Screen codes in the matrix follow the product map, not the prototype's displayed codes (design analysis §6.1). A row keyed by a prototype code fails generation.

## Requirements covered

- **Product-map screens:** none owned. The matrix covers the actions of all 91 product-map screens plus the 5 supplementary admin screens. That includes ACC-05, SPN-16 and ADM-35, whose own actions are rows like any other. Done at this node's gate means: every P1 screen action is green in all three columns (an MCP cell of `n/a (excluded: …)` counts as green only when it passed the three conditions of §3); every P2 or P3 action that already has a P1 route or operation behind a flag is `phase_flagged` with identical refusals on all three paths and names the owning node; and every other P2 or P3 action is `later` with its owning node (A90 to A97) named. Those nodes turn their own `later` rows green by re-running `parity:run --rows` at their own gates.
- **J04 First investment:** done when every UI action behind steps 1 to 13 has a green row, each carrying its journey-step reference and the class derived from A03's data, and the node QA's expected-class table confirms it. The expected classes are: investor steps (PUB-03 browse, PUB-04 IM read and acknowledge, INV-01 order, INV-02 confirm, ONB-09 PayID status, INV-03 order status) `delegable` or `confirmable`; staff steps ADM-18 TRSY match, ADM-08 CAPM closing the offer at the hard cap, ADM-12 CAPM allocate, ADM-18 FINA release, ADM-14 TOKO mint request and ADM-17 REGO reconcile `maker` (or `delegable` for REGO's read-only reconcile run if A03 says so), with the agent's APR payload hash equal to the person's; ADM-17 TP-RE trustee issue follows A03 and is expected to be `human_only` (it is the trustee's issue decision, caught by the §3 invariant); the checker steps (CFO and CCO approvals, custody quorum) `human_only` and refused to every agent.
- **J06 Monthly distribution:** done when SP-FIN's import, costs and proposal pass as `maker` under an SP-FIN token and MCP, SP-OWN's "Approve as sponsor owner" is refused to every agent including SP-OWN's own, FINA's review passes as `maker`, the CFO approval and TRSY funding checker steps are refused to agents, and the investor's POR-03 and ACC-03 reads match.
- **J15 Withdraw cash:** done when the POR-05 request (A59's screen) passes as `confirmable` inside the mandate cap and is refused one cent over it with `G22_LIMIT_EXCEEDED` naming `remaining`. TRSY's release must pass as `maker`, and the CFO approval must be refused to agents. The G02 re-screen and the "payout only to the verified account" rule must produce identical verdicts on all three paths.
- **J19 Sponsor month-end:** done when the SPN-02 task reads, SPN-08 register reads, SPN-09 submission, SPN-10 attestation (class from A03's `delegable.ts`, and `counsel` if the attestation is a personal declaration), SPN-12 update with the G17 claims check, and SPN-15 invoice read or pay each have a green row under the matching sponsor role's token and MCP, including the SP-VAL and SP-PM uploads.
- **J20 Operator daily run:** done when every ADM screen action in steps 1 to 12 has a green row for each staff role that holds its cell. Reads and drafts must pass. Acknowledge, assign and escalate on alerts must follow A03's class. The disclosure publish in step 8 must pass as `maker`, and every approve, ratify or sign must be refused to agents. The launch-flag confirmation in step 12 must be readable by agents, and the flag change itself must be refused to them.
- **G13 Maker-checker:** done when every approve, reject-as-checker, ratify, veto, countersign and custody-sign action on every surface is `human_only` (with no `CLASS_CONFLICT`), is refused on REST for the maker's agent, a different checker's agent and a sibling agent, is absent from the MCP catalogue as `agent_never_checker` (the would-be tool returns `MCP_UNKNOWN_TOOL`), and succeeds only for an eligible human. Every `maker` row must also produce an approval request whose `payload_sha256` and maker identity set equal the person's, and a request made by an agent must be refused with `APR_MAKER_IS_CHECKER` when its own human tries to decide it. The post-shard query finds zero decision, ratification, veto or custody-signature rows with an `AGNT` principal or a `token_id`.
- **G22 Agent authority:** done when every agent twin carries exactly one G22 evaluation, every delegable row passes with the matching scope and fails naming the scope without it, every cap, window, vehicle, revoke, expiry, owner-role and kill-switch variant is refused with its named code identically on REST and MCP, and the sandbox replay shows identical verdicts with zero live writes.
- **Every other gate: G01, G02, G03, G04, G05, G06, G07, G08, G09, G10, G11, G12, G14, G15, G16, G17, G18, G19, G20 and G21** (added to this node by the graph revision from A00, because D04 applies every gate to agent calls): done when every matrix row whose UI action is evaluated by one of these gates shows the same verdict and the same reason code on the agent twin, over REST and over MCP, as on the human path from the same starting state; a gate that blocks the person blocks the person's agent with the same code; and no gate is skipped, relaxed or evaluated differently for an agent call. The gates are evaluated by A19 and the owning services; this node proves parity and does not re-implement them. Gates that apply only in P2 or P3 (for example G05 target market and G15 travel rule) are proven on their rows once the phase flag is on in the sandbox.
- **M25 Access, approvals and audit:** done when every agent-side audit event in the full run carries `actor_type = AGNT`, `token_id` and `on_behalf_of_id` and matches the person's event in type, object and hashes, when A10's chain verifier passes after the run, and when ADM-30 and ADM-31 actions are rows like any other.
- **M26 Integrations and API:** done when every mapped operation is either exactly one MCP tool with matching scope and gates or an A104 exclusion whose reason matches its row's class (no other exclusion of a UI action), and the matrix is published as the M26 evidence for "API keys per sponsor with scopes" and "AI-agent tools (MCP) with read or write scope". ADM-32 and ADM-35 actions have green rows.
- **State machines:** none owned. State transitions are compared twin to twin, so a transition that only an agent (or only a person) can reach is a red cell against the machine's owning node.
- **Founder requirement D04 and journey J22:** done when parity holds for every user type D04 names: each investor sub-type, each sponsor role, each signed-in partner type through A105 and SPN-16, and each staff role. The REST, MCP, SDK and sandbox surfaces must behave identically for the same token, and the person must see every agent action and refusal in their own activity (ACC-05, SPN-16, A105, ADM-31) with the "agent <prefix> on behalf of <person>" actor, and receive A101's notice for every sensitive agent action. The static check inside `verify` must make a UI-only action impossible to merge.

## Gauntlet loop for this node

- **Specify.** This note is frozen together with:
  - the agent-class vocabulary, the column vocabulary and the class-derivation order (§3);
  - the class-independent invariants and their label-stem list;
  - the allowed-difference list, split into required and optional;
  - the normalisation rules;
  - the exemption, presentation-kind, allow-list and product-map tuple-shape vocabularies (§2, §3);
  - the manifest glob, exceptions and per-format normaliser table (§1(b));
  - the locator contract (§4) and the isolation topology (§5);
  - the input hashes of A03's matrix, `delegable.ts`, A00's `agent-access.json` and counsel-question flags, A104's `catalogue.json` and `excluded.json`, both `screens.json` files and the prototype.

  The node QA records `specVersion` and these hashes in the manifest. Settled points (not open):
  - Human-only operations and the MCP catalogue follow A104's frozen rule: every `requires_human`, `non_delegable_scope`, `pending_counsel` and `agent_never_checker` operation is excluded, never a tool, and the MCP column reads `n/a (excluded: <reason>)` (§3). If the founder chooses A104's opt-in reading of its spec point (a) for write tools, disabled tools read `awaiting_enablement` (amber) and are not red.
  - The SPN-06 sponsor declaration and the SPN-10 operator attestation are `counsel` rows until CQ-01 is answered, derived from A03's data; the SPN-06 submit is a separate `maker` row.

  One spec point is settled by the user before the build: which node owns the `e2e` workspace's `package.json`, where the `parity:*` scripts and the verify step are registered (§2).
- **Oracle (challenge material).** The node QA writes to `<workspace>\gauntlet\hidden\A100\`:
  - Planted-break patches, each applied on a scratch copy of the repo:
    - a console feature calling `fetch('/v1/…')`;
    - a computed `client[op]` call;
    - a `'use server'` action writing through `@realx/db`;
    - a native `<form action="/v1/…">`;
    - a new route with a button but no regenerated MCP catalogue;
    - a scratch `delegable.ts` that marks an ADM-30 approve operation as `true` (must fail `parity:matrix` with `CLASS_CONFLICT`, even though person and agent twins would then match);
    - a scratch A104 `excluded.json` that lists a delegable UI read as `service_only`;
    - a staff agent that makes an approval request whose own human then approves it in person (must be refused with `APR_MAKER_IS_CHECKER`);
    - a withdrawal handler that sends no A101 owner notice on the agent path;
    - a handler that skips G07 when `ctx.principal.kind === 'AGNT'`;
    - a handler that demands an extra G12 acknowledgement only from sessions;
    - an MCP error mapper that drops `reason_codes`;
    - a `requiresHuman` flag removed from the bank-account change route;
    - a read that returns one extra unmasked field under a token.
  - The expected-class table for 60 sampled actions across all four surfaces.
  - The node QA's own normaliser, used to cross-check the comparator on 200 random twin sets.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: five parallel sub-agents on disjoint sub-folders, then integration.
  - (1) `enumerate/`, `lint/` and `extract/`: the compiler-API scanner with route attribution, the `sdk-only-io` rule with its own flat config, runner, fixtures and allow-list, the lint dry run, and the prototype and product-map extracts.
  - (2) `matrix/` and `sources.ts`: the manifest harvester with one reader per format, class derivation from A03, A11, A00 and A104 data, the invariants, the owning-node attribution, the markdown and JSON writers, the lock file and the `parity:matrix` entry.
  - (3) `twins/` and `fixtures/`: the pilot, the shard compose override and reset cycle, clock pinning, per-shard Hardhat snapshots, sandbox resets through `SandboxControl.reset`, the independent principal and mandate factory with a fresh token per twin, and the session, REST, MCP (stdio and HTTP) and sandbox runners with sharding.
  - (4) `compare/` and `refusals/`: the normaliser, the required and optional allowed differences and diff writer, A19 replay for gate inputs, the negative-variant generator, kill-switch tiers, checker attempts, the segregation-of-duties variant and the post-shard invariant query.
  - (5) `crawl/`: the Playwright personas, the role-and-name locator driver and the request listener.

  Integration: run the pilot and fix the shard count, run `parity:matrix`, then `parity:run` and one `parity:full` over the shard stacks, then triage every red cell into `fix-items.json` against its owning node, then run `npm run verify` and write `app/evidence/A100.json`.
- **Challenge (node QA, Opus 5.5 high).** The node QA:
  - runs the static and dynamic suites three times with different shard orders;
  - applies each planted break and confirms the suite names it (the lint, the static check, the invariants or the comparator, whichever applies);
  - compares its expected-class table and its normaliser's output with the suite's;
  - hand-checks 20 random rows end to end in the running stack, including one per principal type;
  - reads the allowed-difference list and the exemptions for anything that could hide a real divergence;
  - greps the evidence folder for token secrets using A12's pattern;
  - confirms that A82's fixtures and this suite's fixtures share no module;
  - checks design fidelity of `matrix.md`: TONE words and product-map screen codes, with the prototype handlers all mapped.

  The node QA writes `app/evidence/A100.qa.md`.
- **Gate, fix loop, escalation.** A pass unlocks A88 and publishes the challenge material into `app/`. At most three fix loops on this node's own defects. Red cells caused by other nodes are not this node's fix loops: they become fix items on the owning nodes through the orchestrator, and this node's gate waits until those are closed and a clean `parity:run` has been recorded. Escalate to the user in these cases:
  - the same assumption fails a third time;
  - a fix would need a change to A03's matrix or `delegable.ts`, A11's `requiresHuman` list, A12's refusal codes or A104's catalogue format (a spec defect: reopen it upstream);
  - an agent succeeds on any `human_only` or `counsel` row, or a gate verdict differs between a person and their agent on a `money` row (Critical, stops the milestone);
  - more than eight owning nodes carry open red cells at once (a milestone-level slip the user should see);
  - a row's expected class depends on a counsel answer not yet given.

## Node-specific challenges (node QA)

- **UI-only action smuggled past the list (completeness).** The node QA adds, on a scratch copy, a sponsor feature button that posts through a native `<form action="/v1/…">`, a console panel that calls `fetch` inside a `useEffect`, a computed `client[verb]` call, and a `'use server'` action that writes with `@realx/db`. Each must fail `verify` through the lint rule. Then the node QA adds an SDK-backed button to ADM-19 with no manifest entry, and `verify` must fail naming ADM-19, the action, the call site and the owning node. Next the node QA adds the matching manifest entry for an operation that has a catalogue tool: `verify` must now pass without anyone touching `docs/parity/` (a fully mapped new row is allowed, and the drift is printed), and `parity:matrix -- --node <id>` must write the row into that node's own evidence path. Finally the node QA points the button at an operation that A104 excludes as `service_only`: `verify` must fail. A pass with any of the failing cases present fails the node.
- **Gate that bends for agents (money, compliance).** A planted handler skips the G07 holder-cap check when the caller is AGNT, and a second one requires a G12 acknowledgement only from sessions. An investor at 4.99 percent orders tokens that take them to 5.01 percent through the session, the token and MCP. The comparator must flag both handlers with the differing `gate_evaluation` rows attached. A silent pass fails the node, and so does a "fix" in the comparator's allowed-difference list.
- **Cross-channel retry (money).** An order under a mandate is sent over REST and the harness drops the response. The agent retries the same order through the MCP tool with the same `Idempotency-Key`. There must be exactly one order, one escrow journal and one audit event, and the MCP response must equal the stored REST response. The same payload with a new key must be refused under A12's signature rule when a signing key is used. A second escrow journal is Critical.
- **Checker by the back door (G13, security).** For ADM-30 approve, SPN-09 sponsor-owner approve, the trustee countersign in A105 and ADM-14 custody sign, the node QA tries each with:
  - on REST, the maker's agent and a second checker's agent, which must be refused with `G22_AGENT_NEVER_CHECKER` or `APR_CHECKER_MUST_BE_HUMAN`;
  - on REST, the token together with the human checker's session cookie on one request, which must be refused with `TOKEN_AMBIGUOUS_PRINCIPAL`;
  - on MCP, over stdio and HTTP, a `tools/call` with the would-be tool name (the operation id) under each of those tokens, which must return `MCP_UNKNOWN_TOOL` with nothing forwarded, since A104 excludes every checker operation as `agent_never_checker`;
  - on MCP HTTP, the human checker's session cookie as well as the token, which must be refused by A104 (`MCP_SESSION_TOKEN_MISMATCH`) or the API (`TOKEN_AMBIGUOUS_PRINCIPAL`), and on MCP stdio with no token, which must be refused with `MCP_TOKEN_MISSING`;
  - on a scratch catalogue where the node QA forces the checker operation in as a tool, the call must still be refused by the API with `G22_AGENT_NEVER_CHECKER` or `APR_CHECKER_MUST_BE_HUMAN`, showing that the catalogue is not the guarantee;
  - the segregation-of-duties case: a staff agent drafts an ADM-30 request as maker, then its own human approves it in person with a valid hardware-key assertion, which must be refused with `APR_MAKER_IS_CHECKER`.

  No decision row may be written in any case. Any accepted decision is Critical.
- **Refusal that forgets its reason (agent misuse).** A withdrawal one cent over `per_action_cap_cents` through MCP, a vote through a token without the governance scope, and a bank-account change through a REST token holding every other cell must each return the named cap with the exact `remaining`, the missing scope and `AGNT_NOT_ALLOWED` with the non-delegable action code. The MCP error result for the first two must carry the same fields as REST; the bank-account change, excluded from the catalogue as `requires_human`, must return `MCP_UNKNOWN_TOOL` over MCP. The vote case is `later` (owner A93) until on-platform votes exist; until then the node QA uses a governance-scoped P1 read or a document-scoped write the token does not hold. A plain "forbidden" string anywhere fails the cell.
- **Agent sees more, or less, than the person (data parity, tipping-off).** An SP-VAL token lists valuations through the MCP resources list, and a KYCA's agent reads the review of the seeded sanctions-frozen investor. The SP-VAL agent must get exactly the session's tenant-scoped rows, with the other tenant's valuation giving the same 404. The KYCA's agent must get exactly the fields the KYCA's session sees. The frozen investor's own agent must get the same body as an ordinary referral. A single extra or missing field fails.
- **Kill switch of the wrong width (security).** The tenant tier is engaged for Harbour & Co. Harbour's SP-FIN agent must be refused with `AGENTS_PAUSED` (tier `tenant`) on REST and MCP. Northline's SP-FIN agent and Alex Morgan's investor agent must pass unchanged. The platform tier must then stop all three, and releasing it through a token must be refused. Each assertion must hold on the first request after the change, on both API replicas of the shard (the shard override starts two, §5). The node QA also checks that ADM-35 "Suspend all agent access" is refused to every staff token and to MCP, and succeeds only for a human under `SGL+R`.
- **Phase flags and counsel rows do not leak (compliance).** An investor agent requests a stablecoin payout or withdrawal destination through the P1 withdrawal route (or another `phase_flagged` row confirmed at Specify, with its owning node), and calls ONB-06's wholesale-status declaration. The first must get the same flag refusal on session, REST and MCP (`PHASE_NOT_ENABLED` or A21's code), with no hidden write. If A52's INV-05 preview page exposes any mutating control, the same check applies to it, and the prototype's "saved as draft" toast must not be a hidden write; if it exposes none, INV-05 "Place sell order" must be a `later` row owned by A92. The declaration must get `SCOPE_PENDING_COUNSEL` on REST and `MCP_UNKNOWN_TOOL` on MCP (excluded as `pending_counsel`). A scratch change of the CQ-01 row in `delegable.ts` to `true`, followed by A104's regeneration, must move the operation into the catalogue and flip the expected class and the result by data alone, with no code change in this suite. A scratch change of A00's CQ-02 flag to "no" must turn the G12 standing-consent case of §7 from success to the G12 refusal, also with no code change.
- **Design fidelity of the matrix.** Every prototype handler (`buyNext`, `withdraw`, `placeSell`, `approveDist`, `lNext`, `apApprove`, `apReject`, `kycClear`, `kycEsc`, `kycRej`, the token-operation Sign and Broadcast) must appear as a row keyed by its product-map code (INV-02, POR-05, INV-05, SPN-09, SPN-06, ADM-30, ADM-10, ADM-14), not the prototype's displayed code, with `lNext` as two SPN-06 rows (declaration and submit). `matrix.md` must use only the TONE status words. A changed byte in the design file must fail `sources.lock.json`.

## Pass criteria

- **Static completeness.**
  - 100 percent of `@realx/sdk` call sites in `apps/web` and `apps/console` resolve to an `operationId` in `openapi.json` and to a matrix row.
  - 100 percent of manifest entries, `screens.json` actions (A50, A51), prototype non-navigation handlers and P1 product-map action tuples map to rows.
  - Zero rows lack an `operationId` except `presentation`, `exempt` and `later` rows.
  - Zero rows lack an MCP tool except `presentation`, `exempt`, `later` and A104-excluded rows whose exclusion reason matches the row's class.
  - Every exemption, presentation kind, allow-list entry and tuple shape uses a code from its closed vocabulary.
  - Zero `CLASS_CONFLICT`, `TUPLE_SHAPE_UNKNOWN`, `MANIFEST_MISSING`, `MANIFEST_EMPTY`, `MANIFEST_FORMAT_UNKNOWN`, `ACTION_UNLABELLED`, `ACTION_AMBIGUOUS` or `ACTION_UNLOCATABLE` findings on P1 screens.
  - Every UI node in `app-graph.json` resolves to at least one manifest.
  - `parity:matrix` inside `npm run verify` passes on the frozen commit, writes nothing tracked, and runs in under 90 seconds.
- **Lint.** Every planted form in `lint/__fixtures__` and every one of the node QA's bypass patches fails `verify`. The allow-list has no entry without a vocabulary reason code. The dry-run record exists and every hit in it is either an allow-list entry or a closed fix item.
- **Triple-run parity.**
  - For every row, under the principals the run matrix of §5 selects, the session, REST and MCP twins have identical normalised outcomes except the allowed differences. That covers status, envelope, body, gate evaluations for G01 to G21, transitions, journals, register entries, approval requests, audit events and outbox events.
  - Every required allowed difference is present: one G22 evaluation and one `api_token_activity` row on every agent twin, exactly one owner notice on every row A101 flags as sensitive, and the confirmation step on every confirmable row.
  - REST and MCP match each other exactly.
  - There are zero red cells, zero `pending` rows on P1 screens, zero `NONDETERMINISTIC` rows and zero `HARNESS_RATE_LIMITED` events.
  - `parity:run` holds on three consecutive runs with different shard orders, rotated representatives and transports swapped, and `parity:full` holds once at the gate.
- **Refusal parity.**
  - Every negative variant in §6 that the run matrix selects returns its named code identically on REST and MCP.
  - 100 percent of refusal envelopes carry `gate` and `reason_codes`, and every G22 refusal carries `missing_scope`, `cap` with `remaining`, or the non-delegable action code.
  - The rate-limit case returns identical `429` values on REST and MCP on both replicas and leaves the person's session bucket untouched.
  - Each kill-switch tier stops exactly its tier on the first request after engage on both API replicas, 5 repetitions each in `parity:run` and 50 in `parity:full`, with zero misses.
- **Human-only.**
  - 100 percent of `human_only` and `counsel` rows are refused to REST tokens for every principal type, with zero business effects (row counts unchanged), and succeed under an eligible human's session.
  - For each such row the MCP column passes the three conditions of §3: listed in `excluded.json` with the matching reason, no tool maps to it, and the would-be tool name returns `MCP_UNKNOWN_TOOL` over stdio and HTTP with zero business effect.
  - Every G13 act is additionally refused to the maker's agent, another checker's agent and a sibling agent, and every agent-made approval request is refused with `APR_MAKER_IS_CHECKER` when its own human tries to decide it.
  - The post-shard query finds zero decision, ratification, veto or custody-signature rows with an `AGNT` principal or a `token_id`.
- **Audit, activity and sandbox.**
  - Every agent twin leaves exactly one G22 `gate_evaluation`, one `api_token_activity` row and audit events with `actor_type = AGNT`, `token_id` and `on_behalf_of_id`, matching the person's events in type, object and hashes.
  - Every agent twin, success or refusal, appears in the person's activity read (ACC-05, SPN-16, A105 or ADM-31) with the Actor rendered as "agent <token prefix> on behalf of <person>", and refusals are listed with their code.
  - A10's chain verifier passes after the full run.
  - The sandbox replay of every delegable row shows identical verdicts and zero writes to the live partition.
- **Counsel flags.** The CQ-01 and CQ-02 scratch flips change the expected class and the result with no code change in this suite.
- **Journeys.** Every step of J04, J06, J15, J19 and J20 is referenced by at least one green row, and each step's actor class matches the expectations in "Requirements covered".
- **Challenge material.**
  - All planted breaks are detected by the mechanism named for them.
  - The suite agrees with the node QA's expected-class table on all 60 sampled actions.
  - The suite's normaliser agrees with the node QA's on all 200 twin sets.
- **Runtime and evidence.**
  - The pilot record exists, and `parity:run` finishes in under 60 minutes on the local Windows 11 Docker Desktop stack with the shard count the pilot chose (at most 8). The measured time of `parity:full` is recorded.
  - No token secret appears in `docs/parity/` or the evidence folder.
  - `app/evidence/A100.json` records the input hashes, row counts per surface and class, run ids and timings.
  - `A100.qa.md` has no open Critical or High finding.
  - Only the owned paths changed, plus `app/evidence/A100*`, plus the root `package.json` and `.gitignore` only as a fresh `scripts:sync` regenerates them from the `e2e` workspace's registration (A01's tolerated route).

## Human gates touched

- None crossed. The suite runs on the local compose stack with A11's simulated authenticators, the sandbox bank, identity, registry and custody adapters, the local Hardhat chain and the per-user sandbox partitions. No vendor account, production key, real money or mainnet is touched, and every token it mints is a local test token.
- **HG-LEGAL (counsel questions recorded, not decided here).** CQ-01 and any later counsel question that decides whether a declaration is personal (wholesale-status truthfulness, target-market answers in P2, the sponsor declaration at submission, the monthly operator attestation, FATCA/CRS self-certification). These rows are `counsel` in the matrix and are refused to agents until answered. An answer changes A03's data, and this node's `parity:run` is re-run afterwards with no code change. CQ-02 (whether an agent may acknowledge disclosures under the holder's standing consent) is read from A00's flag in the same way and drives the G12 expectation in §7.
- **HG-PENTEST (later).** The parity of refusals across session, REST and MCP, and the lint-enforced completeness, are inputs to the external penetration test before go-live. This node passes on software evidence and records the gate as open.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
