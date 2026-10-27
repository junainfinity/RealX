---
title: "A89 - Docker Desktop launch (P1)"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am8]
node: A89
milestone: AM8
lane: "Quality"
oracle: no
human_gate: milestone-close
---

# A89 - Docker Desktop launch (P1)

**The finished P1 product, started from Docker Desktop with one command (`npm run launch:up`). One `launch` profile over A23's stack and A61's preview overlay brings up Postgres, the API, the worker, the web app, the operator console, the Hardhat chain, the sandbox panels, the MCP server, A99's docs, sandbox environment and local package registry, and A85's observability, all seeded with A06's design dataset. A smoke test then runs J01, J02, J04 (through to a settled order), J06 and J20 in full, plus an entry probe for each of the other twelve P1 journeys and for J22 (delegate to your own agent). It shows ADM-01 as every staff persona and ADM-32 with all seven human gates (HG-LEGAL, HG-VENDOR, HG-KEYS, HG-AUDIT, HG-PENTEST, HG-CAPITAL, HG-PROD) open. A launch guide lists the personas, the sandbox "magic inputs" and how to create an API token for each persona type (D04), which `launch:tokens` creates (showing a secret in full only on an interactive terminal, once). Release notes are a CHANGELOG per milestone, AM0 to AM8, generated from `status.json` and the evidence manifests. They ship with the guide, and A94 reuses the generator. Then comes the human step **HG-LAUNCH-P1-LOCAL (the P1 software sign-off)**: the user runs the launch on their own machine and signs off the software evidence. "Launch" here means a local launch on sandbox rails. It is not go-live. Go-live keeps the name HG-LAUNCH, as A00's charter defines it, and stays open behind HG-PROD and the six other human gates.**

**Build status:** not started. **Milestone:** AM8 (Journeys, hardening and launch). **Lane:** Quality. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 6-10 agent-hours (build, challenge and verify loops). Raised from 4-8 by the 24 Sep 2026 plan review, in line with the grown spec.

**Depends on:** [[A88 - P1 full regression- 18 P1 journeys, three clean runs, every gate exercised]]
**Unlocks:** [[A94 - P2- end-to-end journeys and launch update]]

**Oracle rule:** this node has no hidden oracle (`oracle: no`). Every cent, unit, gate verdict and state it shows belongs to a dependency that has its own oracle or hidden tests, and A88 has already run every P1 journey three times clean. This node proves that the whole P1 product starts, runs and explains itself on the user's machine. Before the build starts, the node QA still writes Challenge material to `<workspace>\gauntlet\hidden\A89\`, from this note alone. The material covers:
- an independent re-derivation of the smoke checkpoint figures (integer cents and units), made from A06's seed and A79's rules without reading A79's table;
- the expected persona and token table, one row per D04 user type, with the expected create-or-refuse outcome and code, derived from A12's issue rules (§3 table);
- the expected human-gate board;
- a fixture `status.json` with evidence manifests (a clean one, one with a hash mismatch, one where a node is claimed passed but has no manifest, one with a `humanApproval` that has no matching orchestrator entry, and one whose entry hash does not match), with the CHANGELOG it expects from each;
- the resolved-config diff probe, the port-binding probe from the LAN address, the restart-resume probe, the real-date probe and the design-folder-write probe;
- the secret, PII and key scans of the guide, landing page, CHANGELOG, terminal transcript and evidence.

The builder never reads that folder. The material is published into `app/` only after the gate passes ([[App Gauntlet]] §2).

All money in this node is sandbox money through A17's sandbox bank, on A23's local Hardhat chain, with A15's no-value keys. No vendor, no testnet, no real account and no real person is involved. This note is not legal advice. The thresholds, caps, flags and statements the launch shows are the configuration and rule-pack values their owning nodes ship. The launch displays them and asserts against them, and never types a figure of its own.

## Build brief

This node writes no product behaviour and no product screen. It owns:
- a compose profile and override;
- a developer landing page;
- the launch commands;
- a smoke runner that calls other nodes' tests by name;
- a persona and token helper;
- a human-gate reader;
- a release-notes generator;
- an evidence writer;
- the launch guide.

Every call the helpers make goes through `packages/sdk` (A09) or `packages/mcp` (A104) against the running `api` and `mcp` containers, or through the browser against the running `web` and `console` containers. Nothing in `infra/launch/` imports from `services/*`, `packages/db`, any adapter, `apps/web` or `apps/console`, and a lint in `npm run verify` fails if it does. Human acts go through A11's local simulated authenticator (`/__test/*`, mounted only in `RUN_MODE=local|test`) or through the user's own passkey on `localhost`. These acts are:
- passkey sign-in and step-up for the seeded investors and sponsor users;
- staff hardware-key sign-in and decisions;
- the approvals of sponsor and staff tokens.

Every action therefore reaches the same routes, gates and maker-checker rows a person would.

**1. The launch profile (`<workspace>\app\infra\launch\`)**
- `compose.launch.yml` uses `include:` on A23's `infra/stack/compose.yml` and A61's `infra/preview/compose.preview.yml`, never copies. The compose project is `realx`, so only one Postgres ever runs, and the user sees one `realx` project in Docker Desktop's Containers view.
- The profile `launch` activates:
  - A23's `core`, `sandbox` and `otel` services;
  - A61's `web` and `prototype`, but not `preview-home`, which the launch replaces;
  - A53's `console` from `apps/console/compose.console.yml`, on `http://console.localhost:3001`;
  - A85's collector and dashboards from `infra/observability/`, included exactly as A85 ships them (if A85's collector supersedes A23's stub, the choice is made by profile selection, never by an edit);
  - A99's docs, sandbox environment and local npm registry, from A99's fragments;
  - the one service this node adds, `launch-home`.
- **Restart policy.** A23 sets `restart: "no"` on purpose, so that a crash shows. The launch is what the user leaves running, so a crashed process should come back by itself. `unless-stopped` alone does not restore the stack, though: A23 recreates the chain on every boot (`realx_chain` holds nothing), and the one-shot `deploy` and A27's local replay do not re-run when Docker restarts a container. So the only supported way back after a Docker Desktop restart, `launch:down` or a crash of `chain` is `launch:up` (the resume path below). The launch applies one merge file, `compose.launch.override.yml`, with `-f` after the included files. That file changes only two keys on other nodes' services: `restart: unless-stopped` for the long-running services, and a `realx.profile=launch` label. A static test over the resolved `docker compose config` fails if the override changes any other key of an A23, A53, A61, A85 or A99 service (`LAUNCH_OVERRIDE_TOO_WIDE`), such as an image, port, environment value, volume or command. The one-shot services (`migrate`, `seed`, `deploy`) keep `restart: "no"`. Restart counts stay visible, because `launch:check` reads them from `docker inspect` and a count above 0 during the check window is a red line.
- **Resume.** `launch:up` on kept volumes: the one-shots run again (`migrate` at head, `seed` exits `SEED_PRESENT`, `deploy` redeploys onto the fresh chain), then the chain state is rebuilt from the legal record through the owning node's documented replay: A27's local replay, extended by the fix item A27.x (§ Human gates touched, cross-node items) to replay, in order and through the real custody and gateway paths, every `CustodyOp` confirmed after the seed as well as A06's `chain-state.json`. `launch:check` then runs A20's targeted reconciliation for every vehicle and requires register = chain = bank with Δ 0, including units minted after the seed (for example KST01's 10,000 after a kept J04). Until A27.x has passed, `launch:up` on a volume holding post-seed chain events stops with `LAUNCH_CHAIN_STATE_LOST`, tells the user to run `launch:reset --yes`, and never serves a stack whose chain and register disagree. After a Docker Desktop restart, the containers that `unless-stopped` brings back find an empty chain; `launch:check` and `launch-home` then show the red line `LAUNCH_RESUME_REQUIRED` ("run npm run launch:up") until the resume has run.
- **Ports bind to localhost.** Every published port in the resolved config must have `host_ip` 127.0.0.1, as A23 already does for its services. The launch stays running and exposes A11's `/__test` simulated authenticator (a test-runner ceremony helper that can answer for any persona, SECO and SUPER included; neither app calls it), the API, MCP, the sandbox panels and `launch-home`, so none of it may reach the local network. A static test in `npm run verify` and a `launch:check` line fail with `LAUNCH_PORT_NOT_LOCAL <service> <port>`. The override may not fix a violation in another node's fragment (it may change only `restart` and labels), so a violation there is a fix item on its owner and the launch does not start.
- **`launch-home`** is a small Node service on `127.0.0.1:${LAUNCH_HOME_PORT}` (4050), with the fixed banner "RealX local launch · sandbox rails · not production". It uses no font, colour token or component from `packages/ui` and runs only in `RUN_MODE=local|test`. It is read-only, holds no credential and calls no authenticated route. It reads only the files the `launch:*` commands write into the gitignored folder `infra/launch/.state/` (the URL table, persona list, `human-gates.json` from `launch:gates`, the last smoke report, the clock line from `launch:check`, and a token count with no secret or prefix), plus A09's `/v1/meta` only if A09 serves it without credentials as A23's `stack:check` reads it. A static check fails if `home/` contains a bearer token, an `Authorization` header, a cookie jar or an import of `packages/sdk`'s auth helpers. It shows:
  - the URL table;
  - the persona list;
  - the human-gate board as `launch:gates` last read it from the API, with its read time;
  - the last smoke report;
  - the count of launch tokens that `launch:tokens` created and that have not been revoked by `launch:reset` since (never a token);
  - links to the guide, CHANGELOG and sign-off checklist rendered as HTML;
  - a link to the captured email and SMS inbox on A23's sandbox panel (4020), which owns that view; `launch-home` does not read the inbox itself;
  - the platform clock with KST01's close date beside it, and the `LAUNCH_CLOCK_PAST_CLOSE` or `LAUNCH_RESUME_REQUIRED` warning when set;
  - the HG-LAUNCH-P1-LOCAL state (`awaiting the user` until the orchestrator records the sign-off).
- **Ports and images.**
  - The URL and port table is generated from the resolved compose config into the guide, never typed. A drift test compares the two. Expected entries: web 3000, console 3001, API 4000, worker health 4001, MCP 4010, sandbox panel 4020, prototype 4040, launch home 4050, plus the ports A85 and A99 declare.
  - `infra/launch/images.lock.json` covers the one image this node adds. It reuses A23's and A61's recorded digests by reference.
  - `.env.example` (committed, local-only values) is copied to `.env` (gitignored) on first run. It holds only `LAUNCH_HOME_PORT` and the read-only mount paths.
  - Every `ADAPTER_*` stays `sandbox`, `RUN_MODE` stays `local`, `PHASE` stays `P1` and `CHAIN_ID` stays 31337.
  - A23's static no-live scan is extended by registration to cover `infra/launch/`.
- **Pre-flight.** `launch:up` refuses to start when:
  - `app/evidence/A88.json` is not `passed` (`LAUNCH_REGRESSION_NOT_PASSED`);
  - the checked-out tree differs from the release point `launch-p1` (`LAUNCH_REGRESSION_STALE`, naming the first changed path). The orchestrator tags `launch-p1` at A88's passed `buildCommit` plus this node's owned paths and registered aliases, and moves the tag only when A88 has passed again at the new commit. `launch:up` builds from that tag, or from a git worktree of it when the main tree has moved on, so AM9 nodes (A90 to A93) that commit in parallel, or during the sign-off, do not stop the gauntlet run or the user's own launch. The comparison excludes `app/evidence/**` (including `status.json`) and the gitignored `infra/launch/.state/`. A launch must run the code the regression passed.
- **Clock.** The launch runs on A09's `DbClock` exactly as A23 configures it (real time plus the offset stored in A04's `system_clock` row). The as-of anchor (the offset that makes "now" A06's as-of date, Wednesday 23 September 2026) is applied only on a fresh volume after `launch:reset --yes`. On a kept volume the stored offset is kept, so the platform clock resumes monotonically with wall time and never moves back past a recorded event. On a fresh volume KST01 is live whatever the real date is. `launch:check` prints the clock and KST01's `closes_at`. `LAUNCH_CLOCK_PAST_CLOSE` is a warning line (not red) that fires when the platform clock is past KST01's `closes_at` (14 Oct 2026) on a volume where KST01 is still `live`; it tells the user that the J04 walk needs `launch:reset --yes`. The guide says that the sandbox panel's clock control changes what every persona sees, and that `launch:reset --yes` restores it.
- **Commands**, registered under A01's script-registration convention. All are TypeScript run with `tsx`, work in PowerShell and Git Bash, and use LF line endings.
  - `launch:up`: pre-flight, copy the `.env` files if missing, `docker compose up --wait` for the `launch` profile, the resume path on a kept volume, `launch:check`, then print the URLs and the next step ("open http://localhost:4050"). This is the one command, also after a Docker Desktop restart. The guide also prints the plain `docker compose` line for users who prefer Docker Desktop's terminal, and says that after a Docker Desktop restart `launch:up` must still be run.
  - `launch:check`: A23's `stack:check` and A61's `preview:check` lines, plus lines for the console, A85's dashboards, A99's docs, registry and sandbox environment, and `launch-home`. It adds the port-binding line (every published port on 127.0.0.1), the flags line (every P2 and P3 flag off, `agents.traffic` on), the human-gate line (exactly seven gates, all `open`), the persona line, the reconciliation line (A20's targeted run for every vehicle clean, register = chain = bank Δ 0), the clock line and the design-folder hash line. It exits non-zero on the first red line.
  - `launch:smoke [--seed <s>] [--keep]`: §4. It ends by restoring the baseline unless `--keep` is given.
  - `launch:personas`: prints the persona table from §2.
  - `launch:window <persona>`: opens a separate headed Chromium window (Playwright, the version A01 pins, a fresh temporary profile deleted on close) on the signed-out public home, or on ADM-36 for a staff persona. The window's WebAuthn virtual authenticator holds only that persona's sandbox passkey from A06's gitignored `credentials.json`. The command never signs in, sets no cookie and calls no API: the user clicks "Log in" and completes the real ceremony, and `homeFor` (A52) or the console (A53) decides where they land. There is no role switcher in the apps (D20), so this is how the user walks a seeded persona with history. One window is one person; a maker-checker walk uses two windows. It refuses unless `RUN_MODE` is `local` and the URLs are on 127.0.0.1.
  - `launch:tokens [--sandbox] [--no-print]`: §3.
  - `launch:gates`: prints the human-gate board read from the API as ENGO (A76's `GET /v1/integrations/summary`) and regenerates `docs/launch/HUMAN-GATES.md` and `infra/launch/.state/human-gates.json`.
  - `launch:release-notes [--milestones AM0..AM8]`: §7.
  - `launch:signoff`: prints the HG-LAUNCH-P1-LOCAL checklist and the evidence paths. It records nothing on the user's behalf (§8).
  - `launch:down`: stop, keep volumes.
  - `launch:reset --yes`: revoke every launch token through A12's route, then A23's `stack:reset` and the added services, then re-anchor the clock on the fresh volume.
  - `launch:evidence`: rebuild `app/evidence/A89.json`.
  - `verify:launch`: this is what the gauntlet runs, and it needs Docker Desktop. It is one process that holds token secrets only in memory, in this order:
    1. `launch:reset --yes`, then `launch:up`.
    2. `launch:tokens --no-print`: the token table and the four-check MCP smoke of §3, recorded without secrets.
    3. `launch:smoke` run 1 and run 2 with two different seeds, each from the reset baseline.
    4. `launch:smoke` run 3 with a third seed and `--keep`, so the kept state holds a settled J04 and units minted after the seed.
    5. `launch:tokens --no-print` again (tokens created after the last reset), then the restart-resume probe (below).
    6. `launch:reset --yes`, with the check that no launch token is still active.
    7. `launch:release-notes`, then `launch:down`.

    The **restart-resume probe**: record the data fingerprint (A06's row counts per table, the register versions, the audit head, the ledger balances and the ids and status of the launch tokens); simulate a crash with `docker exec <container> kill 1` on `api` and on `worker` (a stop from inside the container, which `unless-stopped` does restart, unlike `docker kill`), and check they come back with the restart reported as expected for the probe; then `launch:down` and `launch:up` on the kept volumes, which is the same chain-lost state a Docker Desktop restart leaves. After the resume, `launch:check` must be green with `SEED_PRESENT`, the fingerprint unchanged apart from the audit events the resume itself writes, the token ids and status unchanged and each token still passing `introspect`, the platform clock not earlier than before, and A20's reconciliation clean for every vehicle including KST01's 10,000 units minted in run 3. The real Docker Desktop restart is the node QA's hidden probe and the user's sign-off step.
  - `npm run verify` stays Docker-free. It runs only this node's static checks: the import lint, the no-live registration, the lock rule, the override-width rule, the localhost port-binding rule, the `home/` no-credential rule, the smoke-manifest schema, the guide link check, the port-table drift check and the release-notes determinism and `humanApproval` tests over fixtures.
- **Tooling errors.** Each prints one named line:
  - `DOCKER_NOT_RUNNING`;
  - `PORT_IN_USE`, naming the port and its variable;
  - `STACK_NOT_UP`, within 10 seconds;
  - `LAUNCH_REGRESSION_NOT_PASSED` and `LAUNCH_REGRESSION_STALE`;
  - `LAUNCH_PORT_NOT_LOCAL`, naming the service and port;
  - `LAUNCH_RESUME_REQUIRED` and `LAUNCH_CHAIN_STATE_LOST`, each telling the user which command to run;
  - `LAUNCH_FLAG_NOT_P1`, naming the flag;
  - `LAUNCH_LIVE_SELECTOR`, naming the variable;
  - `LAUNCH_HUMAN_GATE_STATE`: a gate reads closed or unknown, or the count is not seven;
  - `LAUNCH_PERSONA_MISSING <type>`;
  - `LAUNCH_SMOKE_TEST_MISSING <file> <title>`;
  - `DESIGN_FOLDER_CHANGED <file>`;
  - `LAUNCH_CLOCK_PAST_CLOSE`, with the reset explained;
  - `RELEASE_NOTES_EVIDENCE_MISMATCH <node>`.

  A failing container stops the report at its line, with the last 50 log lines.

**2. Seed, personas and magic inputs**
- **Seed.** The seed is A06's `seed:design` baseline, checked by `seed:verify`. On a volume that already holds a baseline, A23's `seed` service exits `SEED_PRESENT` and writes nothing. The user's own exploration survives `launch:down` and `launch:up` through the resume path of §1 (the database is kept and the chain is rebuilt from the register and ledger), and only once A27.x has passed; before that, a kept volume with post-seed chain events stops with `LAUNCH_CHAIN_STATE_LOST`. Only `launch:reset --yes` discards the exploration, and it asks for `--yes` when non-baseline rows exist.
- **The D04 type list is pinned.** The canonical list behind `LAUNCH_PERSONA_MISSING` is the human roles of the plan's `requirements-lists.json` that D04 covers: the investor sub-types INV-W, INV-S, INV-E (SMSF, company and trust each as a row), INV-J, INV-R, INV-F and INV-M; the sponsor roles SP-OWN, SP-OPS, SP-FIN and SP-IR; the signed-in partner roles TP-RE, TP-VAL, TP-PM, TP-AUD, TP-OBS and TP-LIC; and every RealX staff role that ADM-35 covers: CEO, SUPER, SECO, CCO, MLRO, KYCA, CLSR, PART, INVM, ASTM, VALO, CAPM, TOKO, REGO, FINA, CFO, TRSY, MKTO, TAXO, LEGL, IRCM, MKTG, SUPP, CMPH, PROP and ENGO. VIS (no account), the provider roles that never sign in (TP-CUS, TP-REG, TP-KYC, TP-BANK, TP-SIGN) and the system principals (AGNT, SVCA, AGT-U) have no row. The list is read from `requirements-lists.json` at build time, and a test fails if it changes without this note.
- **Persona table.** `infra/launch/personas.json` is generated from A06's persona manifest and A98's partner principals, never typed. It has one row per user type in the pinned list. Each row gives: name, principal id, user type, which origin to sign in on (web, sponsor console route, partner portal or operator console), the front door (the sign-in card, `/signup`, `/s/apply` or ADM-36), how to sign in (`launch:window <persona>` for seeded personas, or the user's own passkey for a fresh sign-up), the screen `homeFor` or the console lands them on, what the persona is for (which journey, which screens), and the token row of §3.
  - **Investors:** Alex Morgan `INV-00841` (individual wholesale, three holdings); the seeded joint, SMSF, company and trust accounts; the sophisticated and foreign-resident fixtures; Daniel Okafor (a referred KYC case). Retail (INV-R, P2) and guardian-for-a-minor (INV-M, P3) appear as rows marked "not enabled in P1", with A08's phase flag named.
  - **Sponsor users:** Harbour & Co Property: Mia Chen (SP-OWN), Leo Park (SP-FIN), Ravi Nair (SP-OPS) and Sofia Reyes (SP-IR). Northline Capital: its owner and fund accountant.
  - **Partners:** the Yarra Bend Partners RE's TP-RE user, the panel valuer (TP-VAL), the property manager (TP-PM), the auditor (TP-AUD), the regulator observer (TP-OBS) and the licensee (TP-LIC).
  - **Staff:** Jordan Blake (CCO), Priya Natarajan (TRSY), Helen Tran (CFO), Nadia Farah (KYCA), Owen Whitlock (MLRO), Kai Watanabe (TOKO), Grace Lim (REGO), Aisha Karim (SECO), Ben Cole (ENGO), Dev Sharma (FINA), Tom Hargreaves (CAPM), Sam Reilly (LEGL), Marcus Bell (CEO), Ruth Adeyemi (SUPP) and Lena Moss (SUPER), plus A06's personas for CLSR (J02's ADM-11 decision), TAXO (J06's AMMA statement and J20's ADM-20), PART (J20's ADM-29, and the fallback token approver of §3), PROP and IRCM (J21's probe), and INVM, ASTM, VALO, MKTO, MKTG and CMPH. Names for these come from A06's manifest, never from this note. Whether A06's manifest holds every one is a Specify check on A06.
  - If A06 or A98 has no persona for a type in the pinned list, `launch:personas` fails with `LAUNCH_PERSONA_MISSING <type>`. That is a fix item on A06 or A98. This node never adds a fixture of its own.
- **Magic inputs.** The table is generated from A08's `MAGIC-INPUTS.md`, A17's and A22's scenario tables, A43's address markers, A34's valuation scenarios (`-JUMP`) and A06's named scenarios (`soft-cap-missed`, `late-property-report`, the planted breaks). It is the same generator input A61 uses, so the launch and preview tables can never differ. A verify check diffs it against the running configuration. It covers:
  - identity name suffixes;
  - deposit amounts around A17's manual-match threshold;
  - deposit references and amount endings;
  - email markers;
  - order sizes around the holder cap;
  - residency choices;
  - valuation moves;
  - the clock control and its warning.

**3. A token for each persona type (D04, G22)**
- `launch:tokens` signs in as each persona type in turn through A11's simulated authenticator. Staff sign in with a hardware-attested assertion. Each persona then creates one live token through A12's `POST /v1/agent-access/tokens`, with a fresh assertion over the request hash. The agent is named `launch-<type>`, the expiry is 7 days, and the default mandate is the read families of that persona's own cells (A03's `familiesFor(owner)`). Three personas get a write mandate as well, matching A82's shapes:
  - Alex Morgan: invest, per-order cap A$5,000, daily cap A$8,000, allowed vehicles `[KST01]`;
  - Leo Park: distributions prepare;
  - Priya Natarajan: maker-only treasury drafting.
- **Issue rules stay A12's.** Read-only tokens activate on the requester's assertion alone. Sponsor-user and staff tokens that carry any write family wait in `pending_owner_approval` until a second person approves with their own fresh assertion. The command takes that step as the approver A12 names, and when no eligible second person is seeded it prints A12's pending state or refusal verbatim and never works around it. The expected outcome per type, derived from A12's issue rules (A12 §1, Specify point (a)) and re-derived by the node QA independently:

  | Type (persona) | Mandate | Expected outcome |
  |---|---|---|
  | INV-W (Alex Morgan) | read + invest (caps above) | `active` on his own assertion |
  | INV-S, INV-F (A06's fixtures) | read | `active` on their own assertion |
  | INV-R (A06's retail sign-up at `started`) | read | refused with A12's identity code (`TOKEN_IDENTITY_NOT_VERIFIED`); the row also names the P2 flag `offers.retail` |
  | INV-J (R. & S. Patel, "both to sign") | read | `pending_cosign` on the first signatory's assertion, `active` after the second signatory's assertion over the same payload hash; the 24-hour expiry (`TOKEN_COSIGN_EXPIRED`) is A12's test, cited by name, not re-run |
  | INV-E SMSF (Henry Kwan for Kwan Family SMSF), company (Ridgeline Pty Ltd) and trust | read | `active` when the requester is the verified signatory A12's `SignatoryPort` reports; a non-signatory principal of the entity, if A06 seeds one, is refused `TOKEN_SIGNATORY_REQUIRED` (otherwise A12's test is cited by name) |
  | INV-M | none | refused `PHASE_NOT_ENABLED` |
  | SP-OWN (Mia Chen; Northline's owner) | read | `active` on their own assertion. A write mandate would need a second SP-OWN of the same tenant, otherwise a RealX PART or SECO; the guide says so |
  | SP-FIN (Leo Park) | read + distributions prepare | `pending_owner_approval`, then `active` on Mia Chen's (SP-OWN) assertion |
  | SP-OPS, SP-IR (Harbour & Co), Northline's fund accountant | read | `active` on their own assertion |
  | TP-VAL, TP-PM (sponsor-console members) | read | `active` on their own assertion |
  | TP-RE, TP-LIC | read, inside `partnerScope` | `active` on their own assertion |
  | TP-AUD, TP-OBS | read | `active`, with `expires_at` equal to the partner's time box when that is shorter than 7 days |
  | Staff, read only (every other staff role) | read | `active` on a hardware-attested assertion |
  | TRSY (Priya Natarajan) | read + maker-only treasury drafting | `pending_owner_approval`, then `active` on Aisha Karim's (SECO, not the requester) hardware-key assertion |
  | SUPER (Lena Moss) asking for a money or signer cell | money cell | refused `SOD_HARD_CONFLICT` |
  | Any persona asking for a cell A03 marks `"counsel"` | that cell | refused with A03's counsel code (`SCOPE_PENDING_COUNSEL`) |

  The staff approver follows A12's recorded answer to its Specify point (a) (a SECO who is not the requester, or the line manager). If A12 settles on the line manager, the table is regenerated from A12's rule, not edited by hand.
- **Secrets.** On an interactive terminal (a TTY) and without `--no-print`, each secret is shown in full exactly once, with the persona and the MCP and SDK connection snippets (`http://localhost:4010`, `REALX_TOKEN` as an environment-variable placeholder). On a non-TTY or with `--no-print` (always the case in `verify:launch` and the gauntlet), secrets stay in the process's memory and only the prefix and the last four characters are printed. A secret is never written to a file, log, captured transcript, landing page, guide, CHANGELOG or evidence file. `launch-home` shows only "N launch tokens". `launch:reset` revokes them all through A12's route before `down -v`.
- **Sandbox tokens.** `launch:tokens --sandbox` creates `rx_sb_` tokens through A99's per-user sandbox instead. The guide recommends these for anyone connecting their own agent for the first time, because they act on an isolated dataset and are refused by every live route.
- **Token smoke, per persona type, through `packages/mcp`.** Each check is recorded as an outcome row with its reason codes and never the secret. The primary assertion of every refusal is that nothing changed (the request, flag or record is read back unchanged); the code assertion is that `reason_codes` **contain** the named code, because A12's `evaluateG22` returns every failing check and other codes (for example `G22_SCOPE_MISSING` beside `G22_AGENT_NEVER_CHECKER`) may appear too. Each code cites the owner's pinning test by name:
  - `introspect` returns the owner, `on_behalf_of_id`, the scopes and remaining caps;
  - one allowed read returns the same rows the persona sees on screen;
  - one non-delegable call, called by operation id, is refused with reason codes containing `AGNT_NOT_ALLOWED` (token creation, a KYC case, a bank-account change);
  - for sponsor and staff personas, one approve call on a seeded pending request is refused with reason codes containing `G22_AGENT_NEVER_CHECKER` (A12's never-checker test; APR-2291 for staff, a tenant listing approval for Harbour & Co), and the request reads back undecided;
  - a revoked token is refused within one request (`TOKEN_REVOKED`).
- **Seeded token rows.** The launch uses only tokens it creates through A12's routes. A06's seeded token rows (every row state for ADM-35 and ACC-05) are displayed as seeded. If their prefix or hash scheme differs from A12's format (`rx_<type>_<lookup>_<secret>`, Argon2id), the node QA records a finding against A06 at Specify. It is never reconciled here.
- **Parity.** The parity matrix belongs to A100. `launch:check` confirms that `docs/parity/` at the current commit is all green and its hash equals A100's evidence. It does not regenerate the matrix. This node adds no action for a person or an agent. The commands are developer tooling over existing routes, and `launch-home` is read-only, so nothing here needs a parity row.

**4. The smoke of the P1 journeys**
- `infra/launch/smoke/manifest.json` names each test to run by spec file and exact title, from the owning suites. It never copies them:
  - A79 `e2e/investor/` for J01, J02 and J04;
  - A80 `e2e/operations/` for J06 and J20;
  - A81 `e2e/risk/` for the J09 and J10 entry tests;
  - A82 `e2e/agent/` for the agent J04 and J20 shapes and the J22 entry test;
  - A88 `e2e/regression/` for the other entry probes.

  The runner starts Playwright and Vitest with the launch base URLs through environment variables (`WEB_URL`, `CONSOLE_URL`, `API_URL`, `MCP_URL`), with `retries: 0`. A named test that does not exist stops the run with `LAUNCH_SMOKE_TEST_MISSING`. A spec that hard-codes a preview port instead of reading the variable is a fix item on its owner.
- **Tier 1, in full, from `launch:reset` baseline, at 1280 px:**
  - **J01:** a visitor browses PUB-01, PUB-02, PUB-03, PUB-04 for KST01 (all seven tabs) and PUB-05 to PUB-09. Repeated at 375 px with `document.documentElement.scrollWidth === 375` on every page.
  - **J02:** a fresh individual wholesale sign-up. The clean path runs to ONB-10 "You're verified". The ` REFER` path is decided by Nadia Farah (KYCA) on ADM-10 in the console, and the certificate by A06's CLSR persona on ADM-11.
  - **J04, through to a settled order,** exactly as A79's J04 (steps 1 to 13 and its fixed-figure table): INV-01 to INV-03 for 100 KST01 units. The deposit of A$23,959.20 is held above A17's A$10,000.00 manual-match threshold; Priya Natarajan (TRSY) proposes the match on ADM-18 and Helen Tran (CFO) approves it on ADM-30 with a hardware key. Alex Morgan's `alex-portfolio-bot` places its parity order, and A79's `market.ts` fills KST01 to 10,050 units. Tom Hargreaves (CAPM) locks the allocation on ADM-12 with the 50-unit scale-back, and Helen approves it (`4E`). Dev Sharma (FINA) is the maker of the escrow release on ADM-18, and Helen and Jordan Blake (CCO) approve it on ADM-30 (`6E`). Grace Lim (REGO) sees the register entry land on ADM-17. Kai Watanabe (TOKO) raises the mint on ADM-14 from the locked allocation file, Helen and Jordan approve it (`6E+Q`), and two V2 signers other than the maker (Helen and Jordan) each sign under their own login. REGO runs the targeted reconciliation. The run ends at INV-03 "Settled" and POR-01 with the new holding. Repeated at 375 px for the investor pages.
  - **J06:** the seeded BAY01 September run. Leo Park (SP-FIN) proposes, Mia Chen (SP-OWN) approves inside the tenant, Dev Sharma (FINA) reviews on ADM-19, Helen Tran approves on ADM-30 with a hardware key, Priya Natarajan funds on ADM-18, the investor sees it on POR-03 and ACC-03, and A06's TAXO persona issues the AMMA statement.
  - **J20:** the operator daily run, each step as the persona holding that step's product-map role: ADM-01 CCO, ADM-02 ENGO, ADM-16 CCO, ADM-17 REGO, ADM-32 ENGO, ADM-31 SECO, ADM-04 CCO, ADM-24 LEGL, ADM-29 PART, ADM-20 TAXO, ADM-27 CCO, ADM-34 CCO ("launch flags off"). A persona may act in more than one step when its role owns them; no step is taken by a persona whose role does not hold it.
- **Tier 2, entry probes for the other P1 journeys:** J03, J05, J09, J10, J11, J12, J13, J14, J15, J17, J19, J21 and J22. Each probe is either a named test of A88's `e2e/regression/`, A81's `e2e/risk/` (J09, J10) or A82's `e2e/agent/` (J22, mapped to §3's token checks plus A82's test titles), run by name, or read-only code in `infra/launch/smoke/probes/` whose expected state is taken from A06's scenario `expect` block, never typed. For each, the first step's screen opens as its actor persona in the launch, and the first API operation of the journey returns the state the seed or A06's named scenario promises. Examples:
  - J05: ANN01 under A06's `soft-cap-missed` scenario (closing at 28%) shows the refund path;
  - J09: the estate recovery case in `timelock`;
  - J10: an AML case visible only to MLRO and CCO;
  - J21: the halt form refusing a halt without reopening criteria (as A06's PROP and IRCM personas where the first step is theirs);
  - J22: an investor creates a token on ACC-05 with a passkey step-up and caps, and the agent's first call is admitted under the cap.

  Probes make no state change except where the journey's first step is read-only or, for J22, the token creation that §3 already makes. The full journeys were A88's three clean runs; the probes prove only that the launch serves them.
- **Checkpoint table** (in `infra/launch/smoke/checkpoints.ts`; integer cents and units read through the SDK, independent of the UI). The J04 rows are imported from A79's exported expected values, never retyped here, and the node QA re-derives them independently:

  | Checkpoint | Expected |
  |---|---|
  | J04 order total | 100 × A$238.40 = A$23,840.00, plus the 50 bps fee A$119.20, total A$23,959.20 |
  | KST01 applied units | 6,400 before; 6,500 after the J04 order is funded; 6,510 after `alex-portfolio-bot`'s parity order (10 × A$238.40 + A$11.92 = A$2,395.92); 10,050 at close after `market.ts` |
  | Allocation and scale-back | 10,000 units allocated; 50 units scaled back on the last line; refund 50 × A$238.40 = A$11,920.00 + A$59.60 fee = A$11,979.60, paid once, after the allocation's CFO approval |
  | Escrow release | A$2,384,000.00 subscription money plus Σ the fees A37 computed on the allocated lines; KST01's escrow account ends at A$0.00 |
  | Register before mint | KST01's register entry of 10,000 units precedes the mint transaction (the register version's timestamp and audit sequence are earlier than the mint's `chain_tx` submission) |
  | Mint approval binding | the mint's approval id binds the locked allocation file's hash; the same id is on the custody op and the `Executed(approvalId)` event; the two V2 signers differ from the maker (Kai Watanabe) and from each other |
  | J04 settled | Σ minted = 10,000 = register units = `totalSupply`; per holder register units = chain balance; the persona's register units = chain balance = 100; three-way reconciliation for KST01 clean |
  | J06 run | Σ payment lines = the approved run total to the cent; payment file hash = the hash the CFO approved; paid = approved |
  | Client money | Bank sandbox virtual-account totals = A16 client-money balances, Δ 0 before and after every run |
  | Audit chain | A10's verifier passes after each run; every agent-written event carries `on_behalf_of_id` and `token_id` |
  | Flags | Every P2 and P3 flag off before and after (G21) |

- The smoke ends by restoring the baseline (`launch:reset --yes`, then `launch:up`), so the user always starts from the seed. `--keep` leaves the post-smoke state for inspection (and for the restart-resume probe) and prints a warning on `launch-home`.

**5. ADM-01 and ADM-32 in the launch**
- **ADM-01 Command center** (A66's screen, console `/`). The smoke signs in as each staff persona and checks the following against the persona's cells from `GET /iam/me`:
  - the rendered tile set equals A66's default layout in `tiles.ts`;
  - no tile shows "Not connected";
  - every figure equals its SDK read, and the treasury Δ is A$0.00;
  - the seed's four alerts appear with their pills: the GYM01 register break (High), 14 certificates expiring (Medium, G04), MRN01 valuation due (Medium, G20) and the listing waiting for review;
  - the header (A53's shell): the environment badge reads SANDBOX and never PROD, and `/v1/meta` behind it reports run mode `local` and chain id 31337; the Break-glass button is shown to CCO and SECO only and to no other persona;
  - the actions, by A66's own test titles run by name: acknowledge and assign an alert (as CCO; the alert state reads back changed through A13); Declare incident offered only to CCO, SECO and on-call ENGO (the dialog opens and is cancelled, nothing declared); Emergency pause is a deep link to ADM-14 with the vehicle and `SGL+R` prefilled and pauses nothing itself; Export board pack as CFO returns a `REP-` id and file hash and writes one audit event.
- **The sponsor-filtered view.** The product map lists SP-OWN on ADM-01 with "Sponsors see a view filtered to their own trusts". A66 serves that as the staff "view as tenant" grant of A53's shell (tenant tiles only, platform-only tiles "Not available in tenant view"), and sponsor users see their own slice on SPN-02. The smoke checks both: a staff persona viewing as Harbour & Co sees only Harbour & Co's trusts (KST01, BAY01, MPA01), and Mia Chen's SPN-02 shows the same figures.

  The CCO's page is captured at 1280 px beside the prototype's `adCmd`.
- **ADM-32 Integrations and jobs** (A76's screen, console `/integrations`). As ENGO, and again as SECO and PART (the other roles the product map names for the screen), the smoke checks each tab, and records each role's view in the evidence:
  - **Connectors:** every row shows the brick `sandbox` pill with health and latency. This covers identity/DVS, sanctions, bank rail NPP/PayID/PayTo, custody, registry, chain RPC and indexer, e-sign, email/SMS and the MCP server.
  - **Jobs:** each job shows its last run and next due time.
  - **Webhooks:** the webhook list renders.
  - **Sandbox:** the tab is shown, because `RUN_MODE` is `local`.
  - **Human gates:** exactly seven rows (HG-LEGAL, HG-VENDOR, HG-KEYS, HG-AUDIT, HG-PENTEST, HG-CAPITAL, HG-PROD), each with the amber "open" pill and the sentence "Agents cannot close a human gate. A person closes it with evidence."
  - The link to ADM-35 opens. A76 places the API-key and MCP write-tool rows on ADM-35 (A77) and the sponsor sandboxes on A99's partitions, so the launch checks them there: ADM-35 lists the sponsor integration keys by tenant and the MCP tool scopes, and A99's sandbox lists one partition per sponsor.
  - **A switch away from the local chain is refused (G21, G13).** As ENGO, Propose configuration on `chain-rpc` with `CHAIN_ID=8453`, and separately with an adapter selector other than `sandbox`, is refused with `config_value_not_allowed` naming the variable, and no request reaches SECO's checker queue (A46's and A76's "configuration that reaches for mainnet" tests, run by name). The launch never proposes a Base Sepolia value.

  `launch:gates` reads the same rows from `GET /v1/integrations/summary` (`human_gates[]`, A76's A46.1 field) and writes them into `docs/launch/HUMAN-GATES.md` and `launch-home`. The table is generated, never typed, and a check fails if the screen, the API and the guide differ.
- **HG-LAUNCH-P1-LOCAL on screen.** HG-LAUNCH-P1-LOCAL and HG-REVIEW are process steps on the plan, not platform gates, so they are not on ADM-32 (A76's rule). HG-LAUNCH-P1-LOCAL appears only on `launch-home` and in `SIGNOFF.md`. Go-live (HG-LAUNCH in A00's charter) is a different, later human gate; `SIGNOFF.md` and `HUMAN-GATES.md` both say it stays open.

**6. The launch guide (`<workspace>\app\docs\launch\`)**
- `README.md`, the guide the user follows, also rendered on `launch-home`:
  - What the launch is and what it is not, in five sentences: local, sandbox rails, fictional personas, no real money, no production keys, not go-live.
  - Prerequisites: Docker Desktop with the WSL2 backend, Node 24.21.0, `npm ci` once.
  - The one command, and the equivalent `docker compose` line.
  - The generated URL table.
  - The persona table from §2.
  - How each user type gets in (there is no "view as" switch, D20): a visitor opens the web URL signed out; a new investor follows "Get started" with their own passkey (Windows Hello on `localhost`, or Chrome DevTools' virtual authenticator, which leaves nothing on the machine) and verifies through the local inbox; a new sponsor applies at `/s/apply` or accepts an invitation from the local inbox; a seeded persona with history (Alex Morgan, Mia Chen, the partners and every staff persona) is walked with `launch:window <persona>` and the real "Log in" or ADM-36 ceremony. Each row of the persona table names the persona's front door and the screen they land on.
  - Five short walks, one per Tier 1 journey, in numbered steps. Each gives the screen, the persona, the magic input and what "right" looks like.
  - The magic-inputs table.
  - The agent section: for each persona type, the token row (agent name, prefix, scopes in plain English, caps, expiry, approver), the `launch:tokens` command, the MCP and SDK snippets, a pointer to A99's docs and agent quickstarts, and the list of acts an agent can never do (non-delegable acts, and the checker, approver and custody-signer roles).
  - "Not in this launch": the P2 and P3 features behind flags, each with its flag and phase; live vendors (HG-VENDOR); Base Sepolia (A86's run, not started by the launch); mainnet (HG-PROD).
  - Troubleshooting, with the tooling error codes of §1.
- `HUMAN-GATES.md`, generated by `launch:gates`: the seven gates, each with what the human does, the nodes it gates, the connectors it blocks and the evidence that closes it. Then HG-LAUNCH-P1-LOCAL, explained as the user's sign-off of the software evidence, and a closing line that go-live (HG-LAUNCH in A00's charter) stays open.
- `SWAP-POINTS.md`, generated from A08's swap-point list: the sandbox adapters the launch runs on, each with the vendor contract that would replace it and its human gate. This restates Q12 option 1 (open-standard ERC-3643 with rented services behind sandbox adapters) for a non-technical reader, including the T-REX GPL-3.0 licence question held under HG-LEGAL. It adds nothing to A08's list.
- `SIGNOFF.md`, the HG-LAUNCH-P1-LOCAL checklist (§8).
- `CHANGELOG.md` and `CHANGELOG.json`, the release notes (§7).
- **Wording checks.** The guide and the CHANGELOG pass A43's marketing-claims lint (G17) and A45's forbidden-labels rule: no "guaranteed", no "risk-free", no "TVL", and no management figure called audited. A link check covers every file.

**7. Release notes: a CHANGELOG per milestone (`infra/launch/release-notes/`)**
- **Inputs:** `app/evidence/status.json` (the orchestrator's), every `app/evidence/<node>.json` manifest, the finding counts in `<node>.qa.md`, the graph data `09 App Build/Data/app-graph.json` for milestone membership, titles and requirement ids, and the local git log for the one commit per passed node.
- **Output:** one section per milestone, AM0 to AM8. Each section lists:
  - every node passed, with id, title, gate date, fix loops used and commit short id;
  - the requirement ids covered, aggregated by screens, journeys, gates, modules and state machines;
  - the human steps held, with dates (HG-REVIEW at AM5 and later reviews);
  - the human gates still open;
  - known issues: open medium and low findings, `pending Axx` items and unresolved `userFeedback[]` items, each with its owning node;
  - the counsel questions still open, with their owning nodes.

  `CHANGELOG.json` carries the same content, for A94 and for `launch-home`.
- **Rules:**
  - The output is deterministic: sorted, with no generation timestamp, so the same inputs give byte-identical files.
  - A node that `status.json` calls passed but that has no manifest, whose manifest status differs, or whose `artifactHashes` do not match disk stops the run with `RELEASE_NOTES_EVIDENCE_MISMATCH <node>`.
  - A milestone reads "complete" only when every node in it has passed and its human step has been held. Until the user signs, AM8 reads "software gate passed; HG-LAUNCH-P1-LOCAL awaiting the user". Go-live (HG-LAUNCH) is listed under the human gates still open in every milestone.
  - **A `humanApproval` must match an orchestrator entry.** The generator accepts a non-null `humanApproval` in any `<node>.json` only when `status.json` carries a matching entry in `humanApprovals[]` written by the orchestrator: the node, the step name, the id of the user's own message, and the SHA-256 of the canonical JSON of the approval object. A `humanApproval` with no matching entry, or whose hash does not match, stops the run with `RELEASE_NOTES_EVIDENCE_MISMATCH <node>`. A `null` `humanApproval` is always accepted and reads as "awaiting the user".
  - The notes are written in plain English and never copy a secret, a token, a persona's PII or a finding's reproduction data. A scan runs before writing.
- **Interface for A94.** The generator takes `--milestones`, `--graph` and `--out`, and A94 calls it with AM9 added. A94 does not copy it. The interface and output schema are frozen in `infra/launch/release-notes/README.md`, and a fixture test pins them.

**8. Evidence and the sign-off pack**
- `app/evidence/A89.json`, per run, holds:
  - the seeds, image digests and resolved-config hash;
  - the design-folder hashes before and after;
  - the boot and check reports, with the time to all-healthy;
  - the smoke results with timings and the checkpoint table;
  - the gate evaluation counts per gate seen in the smoke;
  - the persona-type token table (created, pending or refused, with the code, and never a secret);
  - the ADM-01 tile sets per persona and the ADM-32 board snapshot;
  - the restart-resume result;
  - the hashes of the CHANGELOG and guide;
  - the release point `launch-p1` (tag name and commit) the runs used;
  - an empty `userFeedback[]` and the `humanApproval` field for HG-LAUNCH-P1-LOCAL, `null` until the user signs.

  `A89.qa.md` is the node QA's.
- **`SIGNOFF.md`**, the checklist the user walks:
  1. Run `npm run launch:up` on a clean machine state.
  2. Open `launch-home`.
  3. Walk the five journeys from the guide.
  4. Create one token with `launch:tokens` and connect an agent client to the MCP server.
  5. Open ADM-32 and see the seven human gates open.
  6. Read the CHANGELOG.
  7. Confirm that the evidence lists three clean smoke runs.

  The user records their name, the date and any notes. Every note becomes a `userFeedback[]` item with an owning node, which the orchestrator opens as a fix loop on that node or as a new node, never as a patch in this node's paths. `launch:signoff` only prints the checklist and the paths. The `humanApproval` field is written by the orchestrator from the user's own message, together with the matching `humanApprovals[]` entry in `status.json` (§7), never by an agent. `SIGNOFF.md` states that signing does not approve go-live, which stays open as HG-LAUNCH.

## Owned paths (other nodes do not edit these)

- <workspace>\app\infra\launch\
- <workspace>\app\docs\launch\

Inside `infra/launch/`, by convention:
- `compose.launch.yml` and `compose.launch.override.yml`;
- `.env.example` and `images.lock.json`;
- `home/` (the `launch-home` service);
- `scripts/` (the `launch:*` commands);
- `personas/` (the generator for `personas.json`);
- `tokens/`, `gates/` and `smoke/` (`manifest.json`, `runner.ts`, `checkpoints.ts`, `probes/`, the ADM-01 and ADM-32 checks);
- `.state/` (gitignored; the files `launch-home` reads, never a secret);
- `release-notes/`, `evidence/` and `checks/` (the static checks `verify` runs).

The root npm aliases are added under A01's script-registration convention. The following are included, served read-only, run by name or driven, and never edited:
- A23's `infra/stack/`, A61's `infra/preview/`, A53's `compose.console.yml`, A85's `infra/observability/` and A99's fragments;
- the suites of A79, A80, A82 and A88;
- the console screens of A66 and A76;
- the design folder `<workspace>\design\`.

A finding against any of them goes back to the owning node as a fix item.

## Design references

- DESIGN = `<workspace>\design\RealX App.dc.html`, run by `support.js`. A61's `prototype` service serves it through a read-only mount, and nothing modifies it. EXT-B = [[A51 - Design extension B- operator console screens not in the prototype]] (`app/design-ext/admin/`). ANALYSIS = the design analysis.
- **`design:adCmd`** (ADM-01):
  - the screen definition on DESIGN line 880: eyebrow `ADM-01`, title "Command center", subtitle "Wednesday 23 Sep 2026 · all figures reconciled at 06:00 AEST.", the four KPIs Client money, Live offers, Approvals waiting and Open alerts, and the four alert TaskRows with their pills and times;
  - the admin page frame: `main` at line 630 (`padding:32px 40px 64px`, gap 24, `max-width:1280px`), the Geist Mono 12 px eyebrow and the Newsreader 40 px h1 at lines 632-634, the KpiGrid at lines 636-640 (1 px `#E3DED4` gaps, white cells, Geist Mono 500 24 px values) and the TaskRows at lines 642-652;
  - the dark admin sidebar from `adNavDef` at line 862;
  - the role switch landing admin on `adCmd` at line 792.

  The launch capture of ADM-01 as CCO is compared region by region: computed style within 1 px or exact hex, and text equal to the seed. The comparer is A87's, run as a tool. ANALYSIS §6.2: the product map wins on content and the design on presentation, so the extra role-filtered tiles are cited as product-map additions.
- **`design-ext:ADM-32`** (Integrations and jobs) from EXT-B. ANALYSIS §5 describes it as "two DataTables, connectors with latency; jobs with last run". EXT-B also draws the brick `sandbox` pill on every connector in a local run, the registry-sync banner and the states default, connector down, job failed and empty. The human-gates tab and its amber "open" pills are A76's composition, listed there as a design finding. The launch capture of ADM-32 at 1280 px is compared against those frames.
- **Tokens (ANALYSIS §1)** on the two captures: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, amber `#7A4F00` on `#F6ECD6`, red `#A3312A` on `#F6E1DE`, line `#E3DED4`; Newsreader for display, Geist for text, Geist Mono for figures, codes and captions. The BrickX-style coral look of the product-map document is not used anywhere, and a capture showing it fails.
- `launch-home`, the sandbox panel and the command output are developer tools. They deliberately use none of these tokens, carry the sandbox banner, and are excluded from design fidelity. The full fidelity pass over every screen is A87's, and this node does not repeat it.

## Requirements covered

- **Product-map screens (built by A66 and A76; this node proves them in the launch):**
  - **ADM-01 Command center:** done when, in the launch, every staff persona sees exactly its role's default tiles, with no "Not connected" tile, every figure equal to its SDK read, the treasury Δ at A$0.00 and the seed's four alerts; the environment badge reads SANDBOX (never PROD) on every persona's page, with run mode `local` and chain 31337; Break-glass shows for CCO and SECO only; A66's action tests (acknowledge and assign an alert, the Declare incident and Emergency pause visibility, the audited board-pack export) pass by name; and the sponsor-filtered view shows only Harbour & Co's trusts. The CCO capture passes the region checks against `design:adCmd`.
  - **ADM-32 Integrations and jobs:** done when, as ENGO, SECO and PART, every connector (e-sign included) shows `sandbox` with health, jobs show last and next runs, the sandbox tab is present, and the human-gates tab shows exactly the seven gates, all open, identical to `GET /v1/integrations/summary` and to `HUMAN-GATES.md`; a proposal to switch `chain-rpc` away from 31337 to a mainnet id is refused with no request created; and the ADM-35 link reaches the key and MCP-scope tables. The capture passes against `design-ext:ADM-32`.
- **Journeys:**
  - **J01 Browse and decide:** done when all nine steps pass as a visitor in the launch, three runs in a row at 1280 px and once at 375 px with no horizontal scroll.
  - **J02 Sign up and verify:** done when the clean and ` REFER` paths reach ONB-10 all green, with the KYCA and classification decisions taken in the console by the named staff, three runs in a row.
  - **J04 First investment:** done when the order goes from INV-01 to "Settled", with the checkpoint table exact: 6,400 → 6,500 → 6,510 → 10,050 applied, 10,000 allocated, the A$11,979.60 refund paid once, the release of A$2,384,000.00 plus the allocated-line fees with KST01's escrow at A$0.00, the register entry before the mint, the mint approval bound to the allocation-file hash with two signers other than the maker, Σ minted = register = `totalSupply` = 10,000, and client money Δ 0. The agent variant (A82's test) matches the passkey path's gate verdicts. Three runs in a row.
  - **J06 Monthly distribution:** done when the BAY01 run goes from proposal to paid with two people at every approval, paid = approved to the cent, the payment file hash bound to the CFO's approval, and the investor's POR-03 and ACC-03 showing it. Three runs in a row.
  - **J20 Operator daily run:** done when all twelve steps pass, each as the persona holding that step's product-map role, and ADM-34 confirms the launch flags are off. Three runs in a row.
  - The other twelve P1 journeys and J22 pass their entry probe in the launch. Their full coverage is A88's (and A82's for J22).
- **Gates:**
  - **G21 Licence perimeter:** done when the launch refuses to start with any P2 or P3 flag on (`LAUNCH_FLAG_NOT_P1`), `launch:check` and J20's ADM-34 step both show resale, retail and stablecoins off, ADM-04 shows the seeded CAR record and the DAF Act exemption meter, and an attempt in the launch to turn `offers.retail` on through the API as SUPER leaves the flag unchanged (the primary assertion, read back through `/v1/meta` and ADM-34) and is refused by A21 with reason codes containing `G21_NO_BINDING` (A21's pinning test cited by name; `G21_PHASE_NOT_REACHED` may appear beside it).
  - G22 is exercised by §3 on every persona type as a parity smoke. Its coverage is A12's and A100's.
- **Modules:**
  - **M01 Command Center:** done when ADM-01 and its alert rows (ADM-02's top slice) are served in the launch for every staff persona as described above.
  - **M26 Integrations and API:** done when the launch serves the connectors, jobs, webhooks and human-gate board on ADM-32, the MCP catalogue on 4010, A99's docs, sandbox environment and local registry, and one token per persona type created under A12's rules. No connector can be `live` and no gate can be closed.
- **State machines:** none owned. The transitions the smoke passes through (SM-01, SM-05, SM-08 and others) are checked by their owners' tests, which the smoke runs by name.
- **Founder requirement D04:**
  - Every persona type D04 names has a row in the launch guide with the path to its own token.
  - The tokens that A12's rules allow are created, and the others are refused with the named code.
  - Each created token passes the introspect, read, non-delegable-refusal and never-checker smoke.
  - The MCP server runs, so the user can connect their own agent.
- **Swap points (Q12 option 1):** `SWAP-POINTS.md` names each sandbox the launch runs on and the human gate that replaces it. No live selector can be chosen (`LAUNCH_LIVE_SELECTOR`, plus A23's override tests re-run against the launch profile).

## Gauntlet loop for this node

**Specify.** This note is the frozen spec, together with:
- A88's passed evidence and `buildCommit`, and the orchestrator's `launch-p1` release point;
- A23's and A61's service lists, ports and report shapes;
- A53's console fragment; A85's and A99's fragments, with their port bindings;
- A06's persona manifest and A98's partner principals, checked against the pinned D04 type list of §2 (a missing persona is a fix item on A06 or A98, and blocking);
- A12's issue rules and error codes, and its recorded answer to its Specify point (a) (who approves a token);
- A27's local replay, and whether fix item A27.x (the post-seed replay) has passed; if not, the resume path stops with `LAUNCH_CHAIN_STATE_LOST` and the restart-resume pass criterion is held open, which blocks the gate;
- A79's exported J04 expected values;
- A76's `human_gates[]` shape; A46's configuration refusal codes;
- A66's `tiles.ts` and action test titles;
- the test titles in the smoke manifest;
- the graph data's milestone membership.

Two spec points are settled with the user before the node QA writes its material:
- (a) The restart policy `unless-stopped` in the launch override (the proposed reading), rather than A23's `restart: "no"`, with resume always through `launch:up`.
- (b) Whether the token table uses live tokens against the seeded dataset (proposed, as A61 did for one persona) or only sandbox tokens.

The node QA also checks A06's seeded token format against A12's and records any mismatch as a finding on A06. It records `specVersion` and the input hashes, and writes the Challenge material to `<workspace>\gauntlet\hidden\A89\`.

**Build (Grok 4.7 xhigh, split by the planner).** Three parallel sub-agents on disjoint folders, then one integration pass:
1. *Profile and tooling* (`infra/launch/` except `smoke/` and `release-notes/`): the compose file and override, the override-width test, `launch-home`, the lock file, `.env.example`, every `launch:*` command, the pre-flight, `launch:check`, the persona and token helpers, `launch:gates`, and the static checks.
2. *Smoke* (`infra/launch/smoke/`): the manifest with exact titles, the runner with the launch base URLs and `retries: 0`, the Tier 2 probes, `checkpoints.ts`, the ADM-01 and ADM-32 checks, and the two captures through A87's comparer.
3. *Guide and release notes* (`docs/launch/`, `infra/launch/release-notes/`): `README.md` with the generated tables, `HUMAN-GATES.md`, `SWAP-POINTS.md`, `SIGNOFF.md`, the generator with its fixture tests and frozen interface, and `CHANGELOG.md` and `.json` from the real `status.json`.

Integration, single agent: register the aliases and checks, run `npm run verify` on a fresh clone, run `npm run verify:launch` on Docker Desktop, then follow `README.md` once by hand as a first-time user, fix every step the guide gets wrong, and write `app/evidence/A89.json`. There is no git commit; the orchestrator commits after the gate.

**Challenge (node QA, Opus 5.5 high).** On a fresh clone on Windows 11, Node 24.21.0 and Docker Desktop, it runs `npm ci`, `npm run verify` and `npm run verify:launch`, then diffs the three runs: checkpoint figures, gate-evaluation counts, the token table, the ADM-01 tile sets, the human-gate board and the MCP tool-name hash. Then the hidden probes:
- the independent figure re-derivation;
- the resolved-config diff (only `restart` and labels changed on other nodes' services);
- the port-binding probe: every published port refused from the machine's LAN address;
- the restart probes, on a kept post-J04 state with live tokens: `docker exec <c> kill 1` on `api` and `worker` (they return under `unless-stopped`, with the restart reported); then, separately, a real Docker Desktop restart, after which `launch:check` shows `LAUNCH_RESUME_REQUIRED` until `launch:up` runs, and after `launch:up` the stack is green with `SEED_PRESENT`, the same data, the tokens' ids and status unchanged, the clock not earlier than before, and register = chain = bank Δ 0 for every vehicle including KST01's post-seed mint;
- the pre-flight with a planted code change against `launch-p1`, and a planted commit on an AM9 path while the launch runs from the tag (which must not stop it);
- a planted P2 flag in `.env`, and a planted `ADAPTER_BANK=monoova`;
- the real-date probe (host clock past 14 Oct 2026);
- the design-folder hashes;
- the fixture `status.json` inputs to the release-notes generator, including the `humanApproval` with no orchestrator entry and the one with a hash mismatch;
- a secret and PII scan of the whole `verify:launch` transcript (zero occurrences of any secret), of an interactive `launch:tokens` run by hand (each secret exactly once, in its first display), the guide, `launch-home` and `home/`, the CHANGELOG, `docker compose logs` and the evidence;
- every persona-type token row replayed by hand against its own §3 table, including a staff member approving their own token, Mia Chen's own write token, the joint both-sign flow, a non-signatory of an entity, and a sponsor member's agent approving a tenant listing.

It then follows the guide itself as a first-time user, checks every URL and every persona sign-in, confirms every "not in this launch" item names a real flag or gate, reads both captures, and writes `app/evidence/A89.qa.md`.

**Gate.** Pass unlocks A94 and publishes the Challenge material into `app/`. The software gate is this node's. AM8 is recorded closed only when the software gate has passed and the user has signed HG-LAUNCH-P1-LOCAL. A94 may start on the software gate. The sign-off's feedback items are opened on their owning nodes and do not block A94 unless the orchestrator marks one Critical.

**Fix loop.** At most 3. Each loop re-runs `verify:launch` in full (three seeded smoke runs, the restart-resume probe and the release notes) and every probe, not only the one that failed.

**Escalate** when:
- a Tier 1 step fails because a screen or service behaves differently from its node's spec, or differently from A88's evidence at the same commit. That is a defect reopened on the owning node, and this node waits.
- the launch needs a change to any path outside `infra/launch/` or `docs/launch/`, including a new persona fixture, a changed port or port binding, a changed test title or the post-seed chain replay (A27.x).
- the override needs to change more than `restart` and labels to keep the stack up.
- any surface shows a connector as `live`, a human gate as closed, or a P2 or P3 flag on. This is Critical.
- a token row shows an agent reaching an approve, ratify or sign cell. This is Critical, stops the milestone, and goes to A12 or A14.
- the release notes cannot be generated because `status.json` and the evidence disagree. That goes to the orchestrator.
- the same probe fails three times.

## Node-specific challenges (node QA)

- **Green by accident (quality).** The smoke passes because the runner skipped a test it could not find, used `retries`, ran against the preview ports rather than the launch, or ran on a dirty volume. Or `launch:check` is green while a container restart-looped under `unless-stopped`. Expected:
  - A missing test title stops the run.
  - The runner's resolved URLs and `retries: 0` are in the evidence.
  - Each run starts from the baseline.
  - The check reads restart counts and fails on any count above 0 in its window.
- **The override that does more (security, G21).** `compose.launch.override.yml` quietly sets `RUN_MODE`, an `ADAPTER_*`, a port or an image on another node's service, or bind-mounts the design folder read-write. Expected: the resolved-config diff allows only `restart` and labels, the design mount is `:ro`, and a planted `ADAPTER_BANK=monoova` or `offers.retail=on` stops `launch:up` with the named code and no container healthy.
- **Money in the smoke (money, G11, G13, G16).** In J04, the deposit is credited without two different people. In J06, a payment file changes after the CFO's approval, or the helper runs TRSY and CFO as one persona. Or the checkpoint figures are read from the page being checked. Expected:
  - The order stays `awaiting_funds` until two humans act.
  - A changed file hash is refused by A14's binding.
  - Every figure is re-derived from SDK reads and matches the node QA's table to the cent, including A$23,959.20, 6,400 → 6,500 → 6,510 → 10,050 units, the A$11,979.60 refund once, the release of A$2,384,000.00 plus the allocated-line fees with escrow at A$0.00, paid = approved and client money Δ 0.
  - The escrow release and the mint each show a maker who is not among the checkers or signers, and the mint lands after the register entry.
- **A token for every type, but wider than its person (D04, G22).** `launch:tokens` gets a persona a token with a scope the persona does not hold, issues a staff token with the requester as their own approver, gives SUPER a money cell, or prints the secret a second time on `launch-home` or in the evidence. Expected:
  - A12's refusals appear verbatim (`SCOPE_EXCEEDS_OWNER`, `SOD_HARD_CONFLICT`, the pending state), and every row matches the §3 table.
  - Each created token's introspected scopes are a subset of the owner's `GET /iam/me` cells.
  - A secret never appears in the `verify:launch` transcript, and on an interactive run it occurs exactly once, in the first terminal output.
  - The sponsor member's agent approving a tenant listing is refused with reason codes containing `G22_AGENT_NEVER_CHECKER`, and the listing reads back undecided.
- **Gates that look closed (governance).** ADM-32, `launch-home`, `HUMAN-GATES.md` or the CHANGELOG shows a human gate as closed, shows HG-LAUNCH-P1-LOCAL as signed, reads the local sign-off as go-live, or marks AM8 "complete" before the user has signed. A planted evidence file with a `humanApproval` that has no matching `humanApprovals[]` entry in `status.json`, or whose SHA-256 does not match, must be rejected by the generator with `RELEASE_NOTES_EVIDENCE_MISMATCH A89`, while a matching orchestrator entry is accepted. Expected: seven open gates on every surface, HG-LAUNCH-P1-LOCAL "awaiting the user", go-live (HG-LAUNCH) open, and AM8 "software gate passed; HG-LAUNCH-P1-LOCAL awaiting the user".
- **The launch on the network (security).** A published port binds to 0.0.0.0, so the `/__test` authenticator (sign-in as SECO or SUPER), the API, MCP or the sandbox panel is reachable from the LAN; or `launch-home` holds a persona token to read the API. Expected: `LAUNCH_PORT_NOT_LOCAL` naming the service, every port refused from the LAN address, and no credential in `home/`.
- **Release notes that lie (governance).** The generator lists a node as passed from `status.json` although its manifest is missing, its `artifactHashes` do not match disk, or its `<node>.qa.md` lists an open high finding. Or two runs over the same inputs give different bytes. Expected: `RELEASE_NOTES_EVIDENCE_MISMATCH` naming the node, byte-identical output on a re-run, and every open finding and counsel question listed under known issues.
- **Stale launch (quality).** A code file changes after A88 passed, for example a hot fix typed during the launch build, and the launch runs code the regression never saw. Expected: `LAUNCH_REGRESSION_STALE` naming the path, compared against `launch-p1`. Only the owned paths and the registered aliases may differ from A88's commit; the tag moves only after A88 has passed again.
- **Resume loses the user's work (usability, reconciliation).** After the user explores (creates a token, places an order that settles and mints), a Docker Desktop restart re-seeds, leaves the stack half up with an empty chain and a full register, loses the post-seed mint, moves the clock backwards, or loses the token rows. Expected:
  - Until `launch:up` runs, the check and `launch-home` show `LAUNCH_RESUME_REQUIRED`; nothing claims the stack is healthy.
  - After `launch:up`, the stack returns with the same data and `SEED_PRESENT`, and register = chain = bank Δ 0 for every vehicle, including the units minted after the seed.
  - Tokens keep their ids and status until `launch:reset --yes`, which revokes them first.
  - The clock resumes monotonically on a kept volume, and on a fresh volume the real date on the host has no effect on what the personas see.
- **Design fidelity of the two operator screens (design).** The ADM-01 capture shows a fallback font, a raw hex that is not a token, the coral look, or a tile in "Not connected" waved through. Or the ADM-32 capture shows a `live` pill. Expected: both captures pass the token-level region checks, and every difference above 5 % cites a design-analysis item or an EXT-B frame.
- **Leaks in the guide (security, privacy).** The guide, `launch-home`, the CHANGELOG, the inbox or the evidence shows a token, a passkey credential id, KMS material, a fresh sign-up's full TFN or licence number, or another persona's messages. Expected: a scan finds none of them, and the inbox shows only the seeded personas' and the current smoke's messages.

## Pass criteria

- On a fresh clone on Windows 11 Docker Desktop, `npm run verify:launch` completes three consecutive seeded smoke runs with all Tier 1 journeys green and all thirteen Tier 2 probes (the twelve P1 journeys and J22) green, with `retries: 0`. Every long-running service is healthy with restart count 0 during the check window (outside the deliberate crash of the restart-resume probe), and every one-shot exits 0. Every published port binds to 127.0.0.1 and none answers from the LAN address. Each cycle reaches all-healthy within 12 minutes once images are present; the first cold build is timed and recorded, not limited.
- The checkpoint figures equal the node QA's re-derivation to the cent in all three runs:
  - J04 total A$23,959.20;
  - KST01 6,400 → 6,500 → 6,510 (agent) → 10,050 (market) units, 10,000 allocated;
  - the scale-back refund A$11,979.60, paid once;
  - the escrow release A$2,384,000.00 plus A37's allocated-line fees, with KST01's escrow at A$0.00;
  - the register entry before the mint, the mint approval bound to the allocation-file hash, and two signers different from the maker and from each other;
  - Σ minted = register = `totalSupply` = 10,000, and per holder register = chain at settlement;
  - J06 paid = approved, with the file hash bound to the approval;
  - client money Δ 0 before and after.

  A10's chain verifies after every run, and every agent-written event carries `on_behalf_of_id` and `token_id`.
- ADM-01 shows exactly the role default tiles for every staff persona, with no "Not connected" tile and treasury Δ A$0.00, the SANDBOX badge (never PROD) for every persona, Break-glass for CCO and SECO only, A66's action tests green by name, and the sponsor-filtered view limited to Harbour & Co's trusts. ADM-32 shows, as ENGO, SECO and PART, every connector (e-sign included) as `sandbox` and exactly seven human gates, all open, identical across the screen, `GET /v1/integrations/summary`, `HUMAN-GATES.md` and `launch-home`; the mainnet `chain-rpc` proposal is refused with no request created. Both captures pass the region checks against `design:adCmd` and `design-ext:ADM-32`.
- G21: `launch:up` refuses with each planted P2 or P3 flag and each planted live selector, with no container healthy. The API attempt to turn on `offers.retail` without a binding leaves the flag unchanged, and its reason codes contain `G21_NO_BINDING`. `launch:check` and ADM-34 agree that every launch flag is off.
- Tokens: every type in the pinned D04 list has a row in `personas.json` and the guide. Every row's outcome equals the §3 table (created, pending then approved by the named approver, or refused with A12's named code). Every created token passes the four-check MCP smoke, with each refusal's primary assertion (nothing changed) holding. Each created token's scopes are a subset of its owner's cells. No active launch token remains after `launch:reset --yes`. The `verify:launch` transcript contains no secret, and an interactive run shows each secret only in its first display.
- The restart-resume probe passes: after the in-container crash and the down-and-up resume (and, in the node QA's probe, a real Docker Desktop restart followed by `launch:up`), the stack returns with the same data, `SEED_PRESENT`, the tokens' ids and status unchanged, the clock not earlier than before, and register = chain = bank with Δ 0 for every vehicle, including KST01's units minted after the seed. The real-date probe passes. The design folder's hashes are identical before and after every run. The resolved-config diff shows only `restart` and labels changed on other nodes' services.
- Release notes: `CHANGELOG.md` and `CHANGELOG.json` exist for AM0 to AM8, with AM8 reading "software gate passed; HG-LAUNCH-P1-LOCAL awaiting the user" and go-live (HG-LAUNCH) listed as open. A re-run is byte-identical. The fixture inputs give the node QA's expected outputs, including `RELEASE_NOTES_EVIDENCE_MISMATCH` on the two bad-manifest fixtures and on the two bad-`humanApproval` fixtures, and acceptance of a `humanApproval` with a matching orchestrator entry. The frozen interface test passes for A94.
- `README.md`, `HUMAN-GATES.md`, `SWAP-POINTS.md`, `SIGNOFF.md` and the CHANGELOG pass the link check, A43's claims lint and A45's label rule. The port table equals the resolved config, and the magic-inputs table equals the running configuration.
- `app/evidence/A89.json` holds:
  - the three runs with the checkpoint table and gate counts;
  - the token table without secrets;
  - the ADM-01 and ADM-32 results;
  - the restart and real-date probes;
  - the `launch-p1` release point;
  - the CHANGELOG hash;
  - an empty `userFeedback[]` and a `humanApproval` of `null`.

  `A89.qa.md` records no open critical or high finding. Only the owned paths changed, plus the registered aliases and check registrations, and `npm run verify` is green.

## Human gates touched

- **HG-LAUNCH-P1-LOCAL, the P1 software sign-off (the AM8 human step; it closes the milestone, not the software gate).** It is not HG-LAUNCH, which A00's charter reserves for go-live and which stays open. After the gate passes:
  - The user runs `npm run launch:up` on their own machine (from the `launch-p1` release point), opens `http://localhost:4050`, and walks `SIGNOFF.md`: the five journeys, one token and an agent connection, the human-gate board and the CHANGELOG.
  - The user then signs off the software evidence. Signing means that the P1 software works end to end on sandbox rails as the evidence says. It does not authorise go-live, real money, vendor contracts or production keys.
  - Every note becomes a `userFeedback[]` item with an owning node. A fix on another node's code requires the orchestrator to re-run A88 (at least the affected journeys) at the new commit and move the `launch-p1` tag; only then is this node re-run as the check that each fix holds, so the re-run never fails `LAUNCH_REGRESSION_STALE`.
  - The orchestrator records the date, the user's name as approver and the resolved items in `humanApproval`, from the user's own message, and writes the matching `humanApprovals[]` entry (message id and SHA-256 of the approval object) in `status.json`.
  - AM8 is recorded closed when the software gate has passed and the sign-off is recorded, even if the feedback list is empty.

  *Agents prepare:* the running launch, the guide, the persona and token table, the human-gate board, the CHANGELOG, the sign-off checklist and an empty feedback list. *Evidence:* `humanApproval` in `app/evidence/A89.json`.
- **Displayed as open, never crossed:** HG-LEGAL, HG-VENDOR, HG-KEYS, HG-AUDIT, HG-PENTEST, HG-CAPITAL and HG-PROD. The launch refuses every path toward them by test: no live selector, no production run mode, no mainnet RPC, no real money, no production keys. The counsel questions it shows belong to their owning nodes and are not decided here. They include:
  - the T-REX GPL-3.0 licence (A07);
  - the personal declarations under D04 (A03's `delegable.ts` rows marked `"counsel"`);
  - acknowledgement of a disclosure under standing consent (CQ-02, A12 and A35);
  - the wholesale "no cooling-off" wording (A35 and A37).
- **Downloads** are limited to the npm registry and the pinned container images the user allowed for the build. Every digest is recorded in a lock file.
- **Cross-node items this note depends on (raised, never edited here):**
  - A27.x: extend A27's local replay to rebuild the chain on a kept volume from every `CustodyOp` confirmed after the seed, in order, through the real custody and gateway paths, so that a resume ends with register = chain = bank Δ 0 (blocking for the restart-resume pass criterion).
  - A79: export J04's expected values (the fixed-figure table) as a module the launch's `checkpoints.ts` imports.
  - A06: personas for every staff role in the pinned D04 list (CLSR, TAXO, PART, PROP, IRCM, INVM, ASTM, VALO, MKTO, MKTG, CMPH), and for the TP-* partner users with A98.
  - A88: rename its "HG-LAUNCH (next, in A89)" line to HG-LAUNCH-P1-LOCAL, so that HG-LAUNCH means go-live everywhere, as in A00 and A94.
  - The orchestrator: the `launch-p1` release point, and the `humanApprovals[]` entries in `status.json`.
  - Any fragment (A53, A85, A99, A61) whose published port is not bound to 127.0.0.1.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
