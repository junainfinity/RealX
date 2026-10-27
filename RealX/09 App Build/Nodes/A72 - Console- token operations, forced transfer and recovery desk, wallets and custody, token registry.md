---
title: "A72 - Console: token operations, forced transfer and recovery desk, wallets and custody, token registry"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am7]
node: A72
milestone: AM7
lane: "Operator console"
oracle: no
human_gate: none
---

# A72 - Console: token operations, forced transfer and recovery desk, wallets and custody, token registry

**The operator console's on-chain and registry screens, built as pages over A28, A15 and A29 and nothing else. ADM-40 is the token registry list (every trust's token with its SM-06 state, supply against cap, register units and the difference, paused and halted). ADM-14 is the drawn `adToken` console for one token: a header with the contract, supply against cap, state and the Pause button; the role holders; a new-action form with reason, legal basis and evidence; the precheck (canTransfer, identity registry, register impact); the pending maker-checker list; "Waiting for signatures", where each row has a `SignatureMeter` and each signer signs under their own hardware-key login before anyone can Broadcast; and a history of approval ids and transaction hashes in which an operation the reconciliation has not closed never reads "Confirmed". ADM-15 is the forced transfer and recovery desk, with evidence hashes, the timelock countdown, the waiver record, the register impact preview and the "notice sent" banner. ADM-13 holds the wallet registry, vaults V1-V7 with quorum and policy, the signer policy rules P1-P10, the paymaster runway and the recovery requests with their waiting-period timeline. Every button on these screens has an API and MCP equivalent under the same gates, and no agent can ever sign, broadcast, waive, ratify or approve from them.**

**Build status:** not started. **Milestone:** AM7 (Operator console). **Lane:** Operator console. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops).

**Depends on:** [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A28 - Token operations service (mint, burn, freeze, pause, forced transfer, recovery)]], [[A29 - Wallet service- smart-account factory, bundler sandbox, guardians and recovery requests]], [[A15 - Custody sandbox adapter and policy engine P1-P10]], [[A51 - Design extension B- operator console screens not in the prototype]]
**Requested additional dependencies (graph change pending in `final-graph.json`, not made here):** [[A104 - MCP server generated from the OpenAPI document]] (the parity rows and the staff-agent challenge call every row through the MCP server), [[A35 - Documents, disclosure library, e-sign and acknowledgements (G12)]] (the mandatory evidence pickers), [[A48 - Halts, kill switches and wind-down]] (halt cause and reopening criteria), [[A42 - AML screening, monitoring, cases and SMR clocks (SM-11)]] (the Block-wallet deny-list route and the AML need-to-know rule) and [[A06 - Seed data and scenario simulator]] (the seed). Until the graph carries these edges, A72 does not enter Build before all five have passed their Gate; the scheduler's remaining-path order does not override this. [[A101 - Agent guard- anomaly detection, sensitive-action notices and tiered kill switches]] (the anomaly event asserted for `P9_TOKEN_CREDENTIAL`), [[A98 - Partner principals- trustee-RE, valuer, property manager, auditor, regulator observer and licensee users]] (the trustee's chain card) and [[A44 - Support, complaints (SM-12) and Investor 360 read model]] (the unmask route) are soft: if they have not landed when A72 builds, the tests that touch them drive recorded fixtures of those nodes' published contracts, the fixture use is listed in `evidence/A72.json`, and those tests are re-run against the real nodes before A81 starts. No MCP, evidence or seed test may be stubbed or skipped to pass the Gate.
**Unlocks:** [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]], [[A79 - End-to-end journeys- investor money path]], [[A81 - End-to-end journeys- recovery, AML and planted breaks]]

**Oracle rule:** this node has no hidden oracle (`oracle: no`). It moves no supply or holdings itself: every rule it shows is decided by A28, A15, A29, A14 and A19, and those are oracle nodes. Because these screens are where people sign token operations, the node QA still writes hidden tests and Challenge fixtures to `<workspace>\gauntlet\hidden\A72\` from this note alone before the build starts ([[App Gauntlet]] §2). They are Playwright specs against the running stack with the sandbox authenticator and three distinct V2 signer personas, component tests that pin every pill, meter segment, countdown and button to the API fields named below, a forged-response fixture (the op detail with `signatures_have` and `timelock_until` edited in transit), a paused-token and a holder-cap-breach scenario, a staff-token persona set for the parity rows, and its own crops of the `adToken` region of the prototype and of the ADM-13, ADM-15 and ADM-40 frames of extension B. The builder never reads that folder.

These screens only draw. The token service (A28) decides whether an operation may exist, the approvals engine (A14) decides who may approve it, custody (A15) decides who may sign it and when, and the wallet service (A29) runs recovery. The console must never make an operation look further along than the API says: a signature not counted by custody, a timelock not yet run out, or a confirmed transaction the register has not reflected. This note is not legal advice. Every checker set, quorum and timelock shown here is a seeded policy row that the screens read and never hard-code.

## Build brief

**1. Routes, files and what this node reuses (no second copy of A53's shell)**

- Route group `app/(token)/` in `apps/console` (Next.js 15 App Router, React 19). Five pages, each exporting `screen` and `designKey` as A53 requires:
  - `tokens/page.tsx`: ADM-40, `design-ext:ADM-40 token registry list`.
  - `tokens/[vehicleId]/page.tsx`: ADM-14, `design:adToken`. The product map's route `/admin/tokens/:contract` is keyed here by vehicle id, because a token in `designed` or `sandbox_deployed` has no contract address yet. The contract address sits in the header and in global search (`0x…` resolves to the vehicle).
  - `tokens/requests/page.tsx` and `tokens/requests/[requestId]/page.tsx`: ADM-15, `design-ext:ADM-15`.
  - `wallets/page.tsx`, with the tabs Wallets / Vaults / Signer policy / Paymaster / Recovery mirrored in the URL, plus `wallets/[walletId]/page.tsx` and `wallets/recovery/[recoveryId]/page.tsx`: ADM-13, `design-ext:ADM-13`.
- Feature code lives in `src/features/token-custody/`:
  - `registry/`
  - `console/`: the header, role holders, new-action form, precheck panel, pending list, signing queue and history.
  - `signing/`: the only folder allowed to call the custody `approveOp` operation.
  - `desk/`
  - `wallets/`
  - `vaults/`
  - `paymaster/`
  - `recovery/`
  - `state-maps.ts`: the pill tables of §6. This file is authoritative for the console: pills are mapped from each record's `state` field, never from a service's coarser `pill` enum (A28's SDK enum `waiting | broadcast | confirmed | broken` has no separate "Confirmed · reconciling", so re-exporting it could turn a reconciling op green). A test fails if any `state-maps.ts` entry maps `confirmed`, `register_pending` or `reconciling` to green.
  - `parity.ts` (§7)
  - `contract-tests/`
  - `index.ts`
- Every page is a `ConsolePage` with the prototype's page pattern (mono code eyebrow, Newsreader 40 title, subtitle, optional `KpiGrid`). Tables are `ConsoleTable` with A09 cursor paging. Status cells are `StatusCell` through A05's `toneFor`. Ids, hashes and addresses are `IdCell`s through `truncateAddress`, the one truncation rule of 6 characters plus 4. Units use A02's `formatUnits` on integers, and money uses `formatAud` on integer cents. Agent-authored rows show `AgentMarker`. Every approve, reject, veto, act and ratify step goes through `HardwareKeyConfirm`. Every draft-and-submit goes through `PrivilegedActionFlow`. `ApprovalsChain`, `SplitListDetail` and `KillSwitchBanner` come from `src/shell/blocks/`. The node draws no block of its own. A visual that neither A05 nor extension B defines goes back to A51 as a finding and is never drawn in raw hex (A05's token lint runs over the folder).
- Token operations are drafted through A28, not through A14 directly. A28's `POST /v1/token-ops/operations` creates the A14 draft, runs the prechecks and returns the approval id and the register impact. This node plugs that route into `PrivilegedActionFlow` as the flow's draft and submit source (A28's `POST /operations/:id/submit`). The flow's Precheck, Submit, Diff and simulation, and Evidence steps then read A14's detail endpoint unchanged. If A53's flow has no documented slot for a domain draft route, the gap goes back to A53 as a follow-up loop. It is never solved by a local copy of the flow and never by calling `approvals.create` from this folder, which A53's lint forbids.
- Badges registered with A53's registry:
  - `token.ops.waiting`: count of ops in `approved`, `queued` or `awaiting_signatures` that the caller may sign or broadcast.
  - `token.requests.open`: open desk items needing the caller's role.
  - `wallets.recovery.open`
  - `paymaster.runway`: a dot only when the API reports amber or red.
  
  Counts come from summary reads, never from counting a list page. The prototype's static "3" on Token operations is not reproduced (design analysis §6.4-8).
- Search providers registered with A53's palette: `OP-` op ids, `TR-` transfer request ids, `RCV-`/recovery ids, contract and wallet addresses (a full `0x` address only, never a partial name), trust codes. No PII goes into the URL. An investor appears by `INV-` id and a masked display name.

**2. ADM-40 Token registry list (`tokens/`)**

- `ConsoleTable` over A28's `GET /v1/token-ops/tokens` (cursor, filters `state`, `paused`, `halted`, `tenant`). Columns as drawn in extension B:
  - Trust
  - Symbol
  - Contract: truncated, or "Not deployed" in grey for `designed`
  - Supply
  - Cap
  - Register units
  - Δ: register minus chain, from the API, never subtracted in the browser
  - Compliance modules: country, max balance, lock-up and identity, as small pills with the parameter hash on hover
  - Paused
  - State: the SM-06 pill from §6
  
  Every row carries the brick "sandbox" pill while A08's registry reports the chain adapter as local or testnet.
- A Δ other than zero renders the red "Break" pill exactly as `adRecon` draws it and links to ADM-17 (A73) for that trust. A token with an open Sev1 break shows a one-line inline banner: "Value-moving operations are blocked until reconciliation is clean". A row opens ADM-14.
- KPIs above the table come from the same list response's summary fields: tokens active, paused, with a break, and not deployed. States: default (the seed's eight trusts, Marine Parade "not deployed", Gympie Road with the break), a paused token, empty ("No tokens yet · a token is created when a listing is approved (J12)"), loading and error as A53 draws them.
- Action on a `sandbox_deployed` row, for TOKO only: "Request deploy". This opens `PrivilegedActionFlow` with A28 kind `deploy` (TOKO → CCO + SECO, `6E+Q`, V1 3-of-5). The test report document is mandatory, and the audit slot shows "open (HG-AUDIT)" whenever no audit report is attached. The form never offers a chain id. The chain is whatever A27's manifest for the run mode says, and a non-local, non-testnet chain id is refused by A27, never chosen here.
- **Configure and migrate are not separate controls.** A28's catalogue has no `configure` or `migrate` kind. `deployed → configured` (identity registry, compliance modules and agents set, R1 clean) is driven by A28 as the tail of the same `deploy` approval under the V1 quorum (J12 step 9), so "Request deploy" is the one control for both, and ADM-40 and ADM-14 show the `configured` state and its audit events. `active → migrating` is driven by the migration runbook (A103, R3) through A28 and A27. These screens show the state and the migration's ops in the pending, signing and history lists, and start neither.

**3. ADM-14 Token operations console (`tokens/[vehicleId]`)**

- **Header.** Read from A28's `GET /v1/token-ops/tokens/:id`:
  - Trust name, symbol, contract (`IdCell`, with a block-explorer link only for testnet runs)
  - The SM-06 state pill
  - "Supply 9,998 / 10,000" with a `RaiseBar` in the design's style
  - `paused` and `halted` pills: the halt comes from A48, with its cause and "ends / reopening criteria" on hover
  - The module parameter hash
  - The prototype's sub-line in the form "Every on-chain operation needs an approval ID and a {need}-of-{total} custody quorum before broadcast", with {need} and {total} rendered from the token payload's V2 token-agent vault quorum (the same values the role holders show). With the seed it reads exactly as the prototype ("2-of-3"). The number is never a literal in code, and V1 (3-of-5) and V3 ops show their own quorum on their rows' meters. This is a recorded deviation from the verbatim copy in `evidence/A72.json`.
  - Times on these screens (countdowns, "executable …", "notice sent …", "signable from …") are formatted with `Intl.DateTimeFormat` in the platform display timezone read from the API's config payload (Australia/Sydney in the seed), with `timeZoneName: 'short'`, so AEST or AEDT is derived from the date and never written as a literal. A test renders a time on 3 Oct 2026 and on 5 Oct 2026 and expects AEST and AEDT.
  
  A token in `designed` renders "Not deployed" with no action form. A token in `retired` renders read-only.
- **Pause and Unpause.**
  - "Pause" (danger outline) is offered to CCO, SECO and the on-call pauser when the token is `active`. It opens `HardwareKeyConfirm` in the `ratify` variant: "This acts at once. {Role} must ratify within 2 h". A reason code is mandatory. It calls A28 as kind `pause` (`SGL+R`, V3). After the API's 2xx, the header shows the amber "Awaiting ratification · h:mm:ss" pill counted down from the server's `ratify_by`.
  - A lapsed ratification leaves the token paused, and the header says so, naming the `unpause` request A28 opened.
  - "Unpause" is offered only on `paused` tokens, as a `PrivilegedActionFlow` draft (CCO → SECO + CFO, `6E+Q`, V2).
  - The screen never offers a V3 unpause, and no control on it can clear a pause without the V2 quorum.
  - While paused, a full-width grey inline banner reads "Transfers paused since {time} · by {role} · forced transfers, recoveries and unlocks need an approved override". The `KillSwitchBanner` block is reserved for agent-traffic tiers (A77) and is not reused for token pauses.
  - **"Request paused override"** (CCO, only while the token is `paused`): a `PrivilegedActionFlow` draft of A28's `token.paused_override` (CCO → LEGL, `4E`). Legal basis is mandatory, and the draft names the scope it asks for (the specific forced transfer, recovery or unlock request, or the kind and the wallet). The decision is LEGL's in the approvals inbox (ADM-30), never here. Approved overrides for the token are listed under the banner with their approval id, scope and expiry from the API.
  - The forced-transfer and recovery drafts (from the desk, §4) and the `unlock` draft (§3's form) carry an **override reference** picker while the token is `paused`. It lists only the approved `token.paused_override` records for this token that the API returns, and it is mandatory for those kinds while paused. Without an approved override the precheck shows A28's `TOP_TOKEN_PAUSED` sentence and submit stays unavailable. `mint` and `burn` are absent while paused (A28 refuses them outright, override or not).
- **Role holders.** A `KeyValueList` from the same payload: owner (V1 3-of-5), token agent (V2 2-of-3), pauser (V3 1-of-N), identity agent (V4 automatic). Each shows the on-chain address, the vault code and whether A27's last role read matches the manifest. A mismatch shows the red "role drift" pill with a link to the recon break. Signer names are never shown here: they appear on ADM-13's vault card, for roles with `V` on M08 only.
- **New action form.** A `PrivilegedActionFlow` with a kind selector. The selector lists only the kinds in A28's catalogue that the caller's role may make (read from the catalogue plus `GET /iam/me`) and that the token's SM-06 state allows (the API's `allowed_kinds` on the header). A kind not listed is absent, not greyed, and a URL-forced kind renders A28's `TOP_SM06_STATE` or `FORBIDDEN`. Fields by kind:
  - `mint` (not from a file: corrective or migration reissue, TOKO → CFO + CCO + CEO per the spec-gap resolution A28 records; mints from a locked allocation file are driven by A38 and appear here in the pending list, never re-created): holder wallets and units per line
  - `burn`: wallet and units
  - `freeze` and `unfreeze`: wallet, full or partial units, reason. The old-wallet freeze of a recovery or forced transfer is never drafted here: A28 creates that op itself under the SM-13 request's approval chain (§4). If the wallet entered has an open SM-13 case, the form shows the API's refusal or warning for it (with a link to the desk item) and offers no second, unlinked freeze.
  - `lock_up` and `unlock`: wallet, lot, release date
  - `rule_change`: module, parameter, new value, and the A19 simulation id from ADM-16. The simulation is mandatory, and the field links to A73's simulator.
  - `upgrade`: implementation from A27's manifest list, never a free-text address; the 72 h timelock is shown before submit.
  
  Every kind also takes a reason code from the domain list, a legal basis (free text in A05's `TextArea`, as A53's `PrivilegedActionFlow` uses it; mandatory for `freeze`, `unfreeze`, `burn` and the paused override), the override reference while the token is paused (above), and evidence as document ids picked from A35's library, shown as `DocumentRow`s with their SHA-256 in mono (this node never uploads a file). Forced transfers and recoveries are not created from this form: the form's footer links to the desk (§4), because they need the SM-13 case.
- **Precheck panel.** A28's precheck results as `GateRow`s, with the same rows at draft (non-binding preview) and at submit (binding):
  - canTransfer simulation
  - identity registry (destination registered, G01)
  - holder cap after the operation (G07), with the stored `GateEvaluation` id and "max allowed n units"
  - residency and FIRB (G06)
  - SM-06 state
  - paused or halted (G10)
  - open Sev1 break
  - supply guard ("≤ remaining allocation")
  - register impact: units before and after per holder from A18, as a `KeyValueList`
  
  A `fail` shows A28's code and fix-it sentence, and the flow's "Draft again". A `warn` becomes an `AcknowledgeCard` for the checkers, never for the maker to tick away. The prototype's mock precheck card ("canTransfer ✓, Identity registry ✓ AU, Register impact none (lock only)") is the layout reference.
- **Pending maker-checker list.** Ops in `requested`, `approved` or `queued` for this token, with approval id, kind, the checkers and their state (from A14's detail), TTL and the timelock countdown. Rows link to ADM-30 (A66) for the decision. This node never renders its own Approve button, because the approvals inbox is the one decision surface. An op that A28 created for an SM-13 case (the old-wallet freeze, the `forced_transfer` or `recovery` op) carries the `TR-` id as a tag in this list, in the signing queue and in the history, linking to the desk item.
- **Waiting for signatures.** The drawn `TokenOpRow` list over A28's `GET /v1/token-ops/operations?vehicle=&state=awaiting_signatures,signed` joined server-side with A15's op detail. Each row:
  - Grid `minmax(0,2fr) minmax(120px,1fr) auto`, padding `16px 18px`, white card, 1 px `#E3DED4`, radius 10.
  - Line one: mono `OP-… · APR-…`.
  - The operation name (Geist 500), then the mono `detail_line` from the API.
  - The `SignatureMeter`: one 6 px segment per vault signer (A28's `signers_total`), filled green `#1F5E4B` up to `signatures_have`, the rest line `#E3DED4`, caption "{signatures_have} of {signers_total} custody quorum", plus "· needs {signatures_need}" when `signatures_need` < `signers_total`.
  - The button.
  
  Button rules, all read from the API and none computed by the client:
  - **Sign** (green primary) appears only when A15 reports the caller may sign this op (`caller_signing.can_sign`). Otherwise the row shows the API's `reason_code` as a sentence:
    - "You approved this request, so you cannot sign it" (`P9_INITIATOR_IS_SIGNER`)
    - "Not a signer of V2"
    - "Already signed by you"
    - "Timelock · signable from 25 Sep 09:12" (`P10_NOT_EXECUTABLE_YET`, with the countdown from the server's `executable_at`)
    - "Vault suspended"
    - "Signing happens at the custody provider" (`SIGN_AT_PROVIDER`, the swap point below)
  - Sign opens `HardwareKeyConfirm` with purpose `custody_sign`. It shows the op, the vault, the approval id, and the server's `calldata_sha256` (never computed by the client), and calls A15's `POST /custody/ops/:id/approve` with the fresh assertion. The meter advances only from the refreshed op detail after the 2xx.
  - **Broadcast** (ink `#17191C`) appears only when the op is `signed` in A28 **and** A28 reports the caller may broadcast it (`caller_broadcast.can_broadcast`, follow-up 6). Otherwise the row shows the API's `reason_code` as a sentence ("Waiting for the token operator to broadcast", "Your role cannot broadcast"). The human roles allowed to broadcast are A28's seeded RBAC row, proposed at Specify as TOKO (the token operator, who also broadcasts the J09 old-wallet freeze) with SECO as the back-up; the screen reads the flag and holds no role list. Read-only console roles (SUPP, IRCM, CFO, KYCA, MLRO and every role without the broadcast right) never see an active Broadcast button, and a forced call from them is refused by A28 and rendered with its code. The prototype's hard-coded "n ≥ 2" is replaced by the API's state, so V1 needs 3 of 5 and V3 needs its one signer who is not the actor. Broadcast calls A28's `POST /operations/:id/broadcast` behind a plain confirm dialog naming the op and the chain. The signatures already carry the authority, so no new key touch is needed and the dialog does not pretend otherwise.
  - The call is idempotent on `op_id`. A double click sends one request. `TOP_BUNDLE_INVALID`, `TOP_NOT_SIGNED` and `TOP_APPROVAL_CONSUMED` render as inline banners with the code.
  - Only on the API's 2xx does the prototype's toast "{id} broadcast · register sync queued" appear. The row then moves to the history with the grey "broadcast" pill.
  - Rows for the three prototype ops come from A28's seed with the corrected data. The seed values the tests rely on, exactly as A28 seeds them:
    - OP-7731 on Bay Road Apartment: V2, `signatures_have = 1`, `signatures_need = 2`, `signers_total = 3`, `state = awaiting_signatures`, timelock elapsed before the seed's as-of date, old wallet frozen, notice sent. The seeded signature is by signer S2.
    - OP-7730 with its own approval id: V2, `signatures_have = 2`, `signatures_need = 2`, `signers_total = 3`, `state = signed`, `next_action = broadcast`.
    - OP-7729, a V2 freeze under `4E+Q`: V2, `signatures_have = 2`, `signatures_need = 2`, `signers_total = 3`, `state = signed`, `next_action = broadcast`.
    
    If A28's or A06's seed differs from these values at Specify, the seed wins, this list is corrected in the frozen spec, and the node QA's fixtures follow the corrected list.
  - **AML need-to-know on the op row.** OP-7729's `detail_line` is redacted per role by A28 on the server (follow-up 7). MLRO and CCO receive "0x2c..7f · AML-118" and the case id links to ADM-25. Every other role (SUPP, IRCM, CFO, TOKO and the rest) receives a generic "0x2c..7f · compliance hold" with no case id, and neither the DOM nor any network response in their session carries `AML-118` or any other case field. The browser never redacts: a field the server did not send cannot leak. Showing an AML case id to other roles would itself reveal that the freeze is AML-related (tipping-off, A42's rule).
- **History.** The drawn "Recent" `DataTable` (Time / Operation / Trust / Approval / Tx / Status) over the same list route with terminal and post-broadcast states. The Tx column is a truncated hash. The status pill comes from the SDK's state-to-pill table (§6), with the red "Break" pill for `broken`. A `confirmed` op that A20 has not reconciled reads "Confirmed · reconciling" in amber, never the green "Confirmed" the prototype uses for closed ops. A row opens an op drawer with:
  - A15's policy evaluations P1-P10 (rule, result, reason code, policy version)
  - the signer list with each signer's role, time and assertion reference (never a name to roles without `V` on M08)
  - `register_instruction_id` and `register_ack_at`
  - `recon_run_id`
  - the audit events for the op from A10's read route, including the actor "Custody quorum · Signed 3/3" shape the `adAudit` row shows
- **Custody signing swap point.** A15's sandbox signs in the console. When the adapter is swapped for a vendor under HG-VENDOR (TP-SIGN, for example a policy-engine custody provider), signers may approve in the vendor's own app. `caller_signing.reason_code = SIGN_AT_PROVIDER` then hides Sign, and the meter keeps reading `signatures_have` from the adapter's status. No console code changes at the swap, and a test with the sandbox adapter's provider mode on proves it.

**4. ADM-15 Forced transfer and recovery desk (`tokens/requests`)**

- `SplitListDetail` as extension B draws it. The list is a `ConsoleTable` with the columns ID / Type / Trust / Qty / Evidence / Timelock / State. Type is one of court order, estate, divorce, error, sanctions directive or lost wallet. Evidence shows document kind names ("Probate", "Order 44/2026", "ID + liveness"), never file contents. Timelock shows "48h · 31h left", "waived" or "—". State is the SM-13 pill.
- FilterChips switch the source. "Transfer requests" reads A28's `GET /v1/token-ops/transfer-requests`: forced transfers, estates, court orders, errors, sanctions directives, and the token fallback of lost-wallet recoveries. "Wallet recoveries" reads A29's `GET /v1/wallets/recovery?state=`: the smart-wallet path and intake. Each chip has its own cursor. The browser never merges two cursors into one list. If the founder's review of extension B (HG-REVIEW) insists on one merged table, the union is a read-only endpoint requested from A28 at Specify, never a client-side join.
- **Detail pane.**
  - Header: "TR-118 · INV-009 → new wallet 0x5e…c11".
  - Evidence: the document hashes in mono with `DocumentRow` links into A35 (view permission checked by A35).
  - Register impact preview: A28's `GET /operations/:id/register-impact`, or the request's attached preview, as a `KeyValueList` of units before and after per holder, plus "Holder cap after transfer (G07) 0.8 %" from the stored evaluation.
  - `ApprovalsChain`: the stages come from A14's decisions plus the SM-13 state, never from a hard-coded list:
    - forced transfer: LEGL prepares → CCO → CFO + trustee, `6E+Q` → old wallet frozen → TIMELOCK 48 h → custody V2 signs → register updated
    - lost wallet, smart-wallet path (A29, shown on the recovery card and in the desk's "Wallet recoveries" source): SUPP intake → KYCA re-verify → CCO (`wallet.recovery.approve`, `4E`) → 48 h waiting period enforced by the smart wallet → executed
    - lost wallet, token fallback (A28 `recovery`, only after A29 marks the account `unrecoverable`): SUPP intake → KYCA re-verify → LEGL legal review (J09 step 5; A29's fallback review) → CCO → CFO + trustee → old wallet frozen → TIMELOCK 48 h → custody V2 signs `recoveryAddress` → register updated
  - The trustee's decision arrives from the partner surface (A98's TP-RE principal) and renders as its own chain card.
  - The old-wallet freeze is the op A28 creates under the request's approval chain (A28 refuses `timelock` until it is `confirmed`, `TOP_OLD_WALLET_NOT_FROZEN`). The desk shows it as a chain card linking to that op on ADM-14, where it carries the `TR-` tag; the desk has no freeze control of its own.
  - The countdown reads "executable 25 Sep 09:12 AEST · 31 h 12 m" (the zone abbreviation derived by `Intl` as in §3, so the same instant after 4 Oct reads AEDT). It is computed from the server's `timelock_until`, which A28 and A15 derive from the audit-anchored `queued` event. The browser clock is used only to tick the display, and a skewed browser clock cannot make Sign appear (Sign lives on ADM-14 and obeys `caller_signing`).
  - The "notice sent" banner reads "Notice sent to the holder's old email and phone · 22 Sep 17:04". It renders only when A28 or A29 report the notice as sent, per channel kind in the snapshot. Before that, the grey banner reads "Notice not yet sent · execution is blocked until it is" (`TOP_NOTICE_NOT_SENT`).
- **Actions**, each offered only when the API lists it as allowed for the caller and the state:
  - "New request" (LEGL): type, vehicle, from wallet, to wallet, units, legal basis, and evidence document ids. It calls A28's `POST /transfer-requests`, then the `PrivilegedActionFlow` for the `forced_transfer` op.
  - "New intake" (SUPP, on the investor's behalf): investor id, channel, recovery type. It calls A29's `POST /v1/wallets/recovery`. The route carries `requiresHuman: true`.
  - "Triage" (SUPP, `requested → triage`): A29's `POST /recovery/{id}/triage` or the transfer-request equivalent.
  - "Legal review" (LEGL): A29's `POST /recovery/{id}/legal-review` with the outcome `rejected` or `compliance_approval` and a mandatory reason.
  - "Switch to token fallback" (LEGL, with reason and the failed smart-wallet attempt's evidence): A29's `POST /recovery/{id}/fallback`. It is shown only when A29 marks the account `unrecoverable`, and the smart-wallet attempt stays visible on the record.
  - "Request timelock waiver" (CCO): A28's `POST /operations/:id/waive-timelock` or A29's `POST /recovery/{id}/waiver`. The court-order document picker is mandatory, and the decision is CCO + CEO in the approvals inbox. For a request type the API lists as waivable, it is drawn as extension B's blocked state "Waiver needs CCO + CEO with a court order on file" whenever no court-order document is attached. The rule is the same as A28's: the control follows A28's `waivable_types` row, and under the proposed seed `lost_wallet` is not waivable. So the waiver control is **absent for every lost-wallet request**, on the smart-wallet path (with A29's sentence that the smart wallet enforces its own 48 h) and on the token fallback (with A28's `TOP_WAIVER_TYPE_NOT_WAIVABLE` sentence, "A lost-wallet recovery's 48 h notice cannot be waived"). J09's third failure path ("a court order needs immediate action: CCO + CEO may waive the timelock") is therefore a recorded product-map deviation (DF-A72-04 in `evidence/A72.json`) pending the counsel and founder question already logged by A28. If Specify or counsel settles `lost_wallet` as waivable, only A28's row changes: the control appears from the API's allowed-actions list with no code change here, and the challenge's expected result flips.
  - "Record holder objection" (SUPP or LEGL): moves `timelock → disputed` through the owning service. The chain card turns red with "Disputed · back to legal review".
  - A granted waiver shows one record card with both approvers, the court-order hash and the time.
- The desk shows SM-13's 14 states and nothing else. `closed` renders only when the service reports `register_updated` and a clean recon. A request stuck in `confirmed` without the register ack shows "Confirmed on chain · register not yet updated" in amber, with the A18 ack SLA.
- States: default, timelock running, waiver blocked, not-eligible (the maker sees no decision controls; the API's SoD sentence shows), disputed, empty ("No open requests"), loading and error.

**5. ADM-13 Wallets, custody and recovery (`wallets/`)**

- **Wallets tab.** `ConsoleTable` over A29's `GET /v1/wallets`. Columns:
  - Address: truncated
  - Investor: `INV-` id and masked name through `MaskedCell`. Unmask goes through A29's or A44's reason-coded unmask route and is audited.
  - Type: passkey smart wallet
  - Identity: the `registry_status` pill
  - Risk: for MLRO and CCO, A42's customer risk rating for the wallet's investor (low, medium, high as a pill) read from A42's need-to-know read route; for every other role a dash. The on-chain `kyt_score` is shown beside it as "KYT not in P1" in grey while P7 KYT returns `not_applicable_in_phase`, never as 0. If A42 publishes no rating read by Specify, the column shows only the KYT line and the omission of the product map's "risk score" is recorded as DF-A72-03.
  - Frozen: pill with a generic reason for roles outside MLRO and CCO
  - Guardians: the `guardian_status` pill
  - Tokens
  
  Wallet detail (`wallets/[walletId]`) shows tokens by trust with the SM-07 holding state per trust (§6), `frozen_amount` and a generic reason, passkey count, guardians, session keys (mandate name, limits, expiry, last use; revoke is the investor's or A77's, not offered here), and the open recovery if any.
- **"Block wallet"** (MLRO, `SGL+R`): a freeze plus a deny-list entry for high risk or sanctions (G02, G14). It drafts A28 kind `freeze` with policy `SGL+R` on V3 through `HardwareKeyConfirm`'s ratify variant, with the reason code and the linked case id. The deny-list half belongs to the screening hub (A42). This node calls the route A42 publishes for it. If A42 publishes none by the Specify step, that is recorded as a finding against A42, and the button ships as freeze-only with the gap written in the evidence, never faked on the client. Nothing about the reason is visible to SUPP or IRCM, and the investor-facing wording is A59's generic one (tipping-off, design analysis §4 item 30).
- **Vaults tab.** One `KeyValueList` card per vault from A15's `GET /custody/vaults` and `GET /custody/vaults/:code`: purpose, quorum ("V1 Owner · 3-of-5 · cold"), rotation class and `rotation_due_at` (amber inside 14 days, red overdue, from the API), function allow-list (target kind and selector), timelocks per function, `restrict_only`, `policy_version`, state, and signers by role. Signer names show only to roles with `V` on M08, and key references are never shown.
  - V4 reads "identity agent · automatic · approved KYC cases only".
  - V5 and V6 render as grey "reserved (P3)" cards.
  - The product map's "V1-V9" is reconciled to the seven vaults A15 seeds (V8 "DeFi admin, not needed" per the catalogue; no V9 exists in any source). This is recorded as design finding DF-A72-01 for the plan reviewer, not invented as a card.
  - Actions: "Rotate key" (SECO, A15's `POST /vaults/:code/rotate`, a `6E+Q` request with the ceremony record fields: witness, custodian, date, old and new key references); "Suspend vault" (SECO, `SGL+R`, protective, A15's `POST /vaults/:code/suspend`); "Change signer policy" (SECO → admin quorum 3-of-5, `6E+Q` with the 24 h timelock, through the policy-change request A15's P8 defines). The change shows a diff of allow-list, quorum, timelock and band rows against the current `policy_version`, and the checker sees the same diff through A14.
- **Signer policy tab.** P1-P10 as a `CheckList` with each rule's plain-English name (approval-ID binding, function allow-list, supply guard, destination guard, amount bands, business-hours window, KYT, policy-change quorum, initiator is not a signer, timelock). Each shows its current parameters from the vault policy and the count of refusals in the last 7 days from A15's `GET /custody/policy-evaluations` summary. P7 reads "not applicable in P1" in grey.
- **Paymaster tab.** From A29's `GET /v1/wallets/paymaster` and A15's V7 record: deposit, spent today, daily cap, and runway as a `KpiTile` ("41 days"). The tone is amber under the warning threshold and red under the Sev2 alert threshold. Both thresholds are read from the payload (extension B draws amber under 14 days; A15 and A29 alert at 3 days); the screen holds no number of its own. Extension B draws the tile as "A$2,140 · 41 days". An AUD figure is shown only if the API supplies an AUD value. Otherwise the tile shows the native deposit and runway days, and the gap is recorded as DF-A72-02 (no price feed exists in P1).
  - The current `paymaster_policy` version (allowed targets and selectors, per-wallet and global caps) is shown as a `KeyValueList`.
  - "Change gas-sponsorship policy" (ENGO → CFO, `4E`, A29's `POST /v1/wallets/paymaster/policy`) goes through `PrivilegedActionFlow` with the diff.
- **Recovery tab.** Cards over A29's `GET /v1/wallets/recovery?state=`, each with the drawn `Timeline`: requested → re-verified → guardian approved → waiting 48 h → executed. The waiting period's end is in mono and comes from the server. The recovery path (`smart_wallet` or `token_fallback`) and the channel snapshot are counts only ("2 emails, 1 phone"), never the values.
  - KYCA sees the re-verification result from the identity adapter and "Submit for CCO approval" (`wallet.recovery.approve`, KYCA → CCO, `4E`) through `PrivilegedActionFlow`.
  - Journey J09 step 3 (KYCA re-verifies identity and the new wallet) ends here.
  - A fallback case links to its desk item on ADM-15.
- States: default, recovery in waiting period, paymaster runway amber, runway red, vault suspended, rotation overdue, empty per tab, loading and error.

**6. State and pill tables (`state-maps.ts`, authoritative for these screens; labels and state lists come from A03 and the SDK, tones and pills are set here)**

- **SM-06 token.** Pills per state:
  - `designed`: grey "Not deployed"
  - `sandbox_deployed`: brick "Sandbox"
  - `deploy_requested`: amber
  - `deployed` and `configured`: grey
  - `issuing`: amber
  - `active`: green
  - `paused`: amber
  - `upgrading`: amber, with the 72 h countdown
  - `migrating`: amber
  - `redeeming`: amber
  - `retired`: grey
  
  A test iterates A03's SM-06 state list and fails on any state without a label, a tone and its allowed actions.
- **TokenOperation** (A28's 14 states; this table, keyed on the op's `state`, is authoritative, not the SDK's `pill` enum). `requested`, `approved`, `queued`, `awaiting_signatures` and `signed` show the amber "waiting". `broadcast` is grey "broadcast". `confirmed`, `register_pending` and `reconciling` are amber "Confirmed · reconciling". `closed` is green "Confirmed". `broken` is red "Break". `refused`, `cancelled` and `failed` are red or grey with the reason code. The prototype's three-pill vocabulary survives for the states it drew. The added states exist because the product map says an operation is not complete until reconciliation passes.
- **SM-07 holding** (wallet detail and the register impact preview):
  - `eligible_empty`: grey
  - `active`: green
  - `locked`: grey, with the lock-in date
  - `partially_frozen`: amber, with units and generic reason
  - `frozen`: red
  - `recovered_out`: grey
  - `deregistered`: grey
  
  The unfreeze action appears only as a `4E+Q` draft (MLRO → CCO), because unfreezing restores rights and always needs four eyes.
- **SM-13** (14 states on the desk and the recovery cards):
  - `requested`, `triage`, `identity_reverification`, `legal_review`, `compliance_approval`, `checker_approval`: amber
  - `old_wallet_frozen`: amber
  - `timelock`: grey TIMELOCK card with a mono release time
  - `disputed` and `rejected`: red
  - `executing`: amber
  - `confirmed`: amber "on chain"
  - `register_updated`: amber "register updated · reconciling"
  - `closed`: green
  
  A test iterates A03's SM-13 list and A29's one recorded extension.

**7. Agent and API parity for everything this node exposes (founder requirement D04, gate G22)**

The screens add no endpoint. Every action is an A28, A15, A29 or A14 route, so parity is proved by listing each action with its agent rule. The same table is exported as `src/features/token-custody/parity.ts` for A100's suite. A test asserts that every SDK mutation called from `app/(token)/` and `src/features/token-custody/` has a row.

| Screen action | API route | Staff agent (`rx_stf_` token or MCP tool) |
|---|---|---|
| Read the registry, token header, queues, history, desk, wallets, vaults, policy evaluations, paymaster, recoveries | `GET /v1/token-ops/tokens`, `/operations`, `/transfer-requests`, `/register-impact`; `GET /custody/vaults`, `/ops`, `/policy-evaluations`; `GET /v1/wallets`, `/recovery`, `/paymaster` | yes, read scopes of M08 and M09 never wider than the owner; the same filters, cursors and masking; AML case fields stay limited to MLRO and CCO for agents too |
| Draft, precheck, submit, cancel a token operation (mint not from file, burn, freeze, unfreeze, lock-up, unlock, unpause, rule change, upgrade, deploy) | `POST /v1/token-ops/operations`, `/{id}/precheck`, `/{id}/submit`, `/{id}/cancel` | yes, as maker inside the mandate (`tokenops:request`), audited "agent {token} on behalf of {person}"; the row shows `AgentMarker`; SoD counts the agent as its human |
| Request a paused override (`token.paused_override`, CCO → LEGL, `4E`); pick an approved override on a forced-transfer, recovery or unlock draft | A28's draft route for the `token.paused_override` approval (an A14 request with no chain op; the exact path is taken from A28's OpenAPI at Specify); the override reference field on the draft routes | draft and reference: yes, as maker inside the mandate, legal basis mandatory, audited "on behalf of"; the LEGL decision: never (`APR_CHECKER_MUST_BE_HUMAN`) |
| Create a transfer request; SUPP intake; triage; legal-review recommendation; switch to fallback; record holder objection | `POST /v1/token-ops/transfer-requests`; `POST /v1/wallets/recovery`, `/{id}/triage`, `/{id}/legal-review`, `/{id}/fallback` | transfer request, triage, legal-review draft and objection record: yes, as maker. Intake on an investor's behalf (`POST /v1/wallets/recovery`): never, `requiresHuman` (A29), `AGNT_NOT_ALLOWED` |
| Sign a custody op | `POST /custody/ops/:id/approve` | never: `P9_TOKEN_CREDENTIAL` before scope evaluation, recorded as a G22 gate evaluation and an A101 anomaly event |
| Broadcast | `POST /v1/token-ops/operations/:id/broadcast` | never for a person's agent (refused before scope); only A38's and A31's `SVCA` pipeline principals |
| Pause (act, `SGL+R`), ratify, veto, approve, reject, waive a timelock, suspend a vault, block a wallet | A14 decide, ratify and veto; `/waive-timelock`, `/waiver`; `/custody/vaults/:code/suspend` | never: `APR_CHECKER_MUST_BE_HUMAN` or `P9_TOKEN_CREDENTIAL`; an agent may draft the underlying request only where the row above says yes |
| Rotate a key, change signer policy, change gas policy | A15 rotate and P8 request; `POST /v1/wallets/paymaster/policy` | draft as maker: yes; every decision and every ceremony step: never |
| KYCA "Submit for CCO approval" of a smart-wallet recovery | A14 draft `wallet.recovery.approve` | yes, as maker; CCO's approval is human only |
| Unmask an investor name | the owning module's unmask route | per A03's `delegable.ts`; `"counsel"` behaves as non-delegable |

- The MCP tools (A104) and the SDK (A09) reach the same routes. No screen here has a UI-only path: the "Sign" and "Broadcast" buttons are ordinary SDK calls that the API refuses to tokens, and are not hidden features.
- Every "never" row is exercised with a seeded TOKO staff token and through the MCP server in this node's contract tests. Every "yes" row succeeds within the mandate and writes the audit line with `on_behalf_of_id`.
- **Out-of-mandate cases (G22 limits).** For every "yes" family (token-op draft, paused-override draft, transfer request and SM-13 maker steps, key-rotation and policy drafts, KYCA recovery submission) the contract tests also make these calls, each through the SDK and the MCP server:
  - a draft on a trust outside the token's allowed-trusts list
  - a draft of a kind outside the token's allowed actions (for example a `burn` from a token scoped to `freeze` only)
  - the same in-mandate draft with a revoked token, and with an expired token
  
  Each must be refused with A12's G22 code (the scope, trust, action, revoked or expired variant), leave a G22 gate evaluation and an A10 audit line, and create no draft (asserted through the API). An over-broad staff token passing any of them fails the node.

**8. Errors, empty states and concurrency**

- Every refusal code from A28 (`TOP_*`), A15 (`P1_*` to `P10_*`, `CST_*`), A29 (`RECOVERY_*`, `WALLET_*`, `PAYMASTER_*`, `SM13_ILLEGAL_TRANSITION`), A14 (`APR_*`) and A11 (`FRESH_ASSERTION_*`, `ATTESTATION_REQUIRED`) that these screens can trigger has a plain-English sentence and renders as the design's inline `Banner` with the code. A generated test iterates those modules' OpenAPI error catalogues and fails on any code without a sentence.
- Empty states:
  - ADM-14 queue: "No operations waiting"
  - history: "No operations yet"
  - desk: "No open requests"
  - wallets: "No wallets yet"
  - recoveries: "No recovery requests"
  - vaults: A15's `503 CST_NOT_INITIALISED` rendered with its first-run instruction, not a blank page
  
  A terminal or `broken` op, request or recovery opens with its state and never a 404. Another tenant's vehicle is a 404 (A28's rule) and renders A53's not-found state.
- Chain adapter unreachable: the op stays `signed`, and the row shows "Chain unreachable · retry by Broadcast" with the Sev3 alert id. Indexer lag beyond 5 minutes: `broadcast` rows show the lag from the payload. Nothing is marked failed by a transport error.
- Freshness: the queue and the open desk item refetch every 15 s and on focus, and after any mutation of that row. Two signers signing at once both see the meter at 2 after refresh. The screen never merges optimistic counts. Idempotency keys come from A53's flow and the SDK, so a retry never creates a second op, a second signature or a second broadcast.

**9. Audit, privacy and telemetry**

- The screens write no audit rows. Every call carries A09's `X-Request-Id`, and every error shows `request_id`.
- No investor name, email, phone, TFN or document content appears in a URL, document title, log or `data-*` attribute. Channel snapshots are counts, and evidence is hashes and kind names.
- The screens store nothing in `localStorage` except A53's column picker.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\console\app\(token)\
- <workspace>\app\apps\console\src\features\token-custody\

Inside the feature folder, by convention: `registry/`, `console/`, `signing/`, `desk/`, `wallets/`, `vaults/`, `paymaster/`, `recovery/`, `state-maps.ts`, `parity.ts`, `contract-tests/`, `index.ts`. The import boundary is enforced by a static Vitest check that this node owns, `contract-tests/import-boundary.test.ts`: it parses every file under the owned paths and fails if the SDK's `custody.approveOp` is imported or called outside `signing/`, or `tokenops.broadcast` outside `console/`. A matching one-line lint rule may be proposed to A53 for its `eslint.config.js` as a convenience, but no pass criterion depends on it. This node only *uses* A53's shell (`src/shell/index.ts`), A05's components and tokens, the SDK operations of A28, A15, A29, A14, A10, A35 and A73's links, and extension B's frames. It changes none of them.

Read-contract gaps to close at the Specify step, before the node QA writes the hidden tests. Each is requested from the owning node as its follow-up loop, in the same way A12 requested A04.1. None is patched here.

1. A15: `caller_signing {can_sign, reason_code}` on `GET /custody/ops/:id` and the list, including `SIGN_AT_PROVIDER` for the vendor swap.
2. A28: on `GET /tokens`, `register_units`, `delta_units`, `recon_state`, `allowed_kinds`, `symbol`, the compliance-module list and the list summary counts; a summary read for the `token.ops.waiting` badge.
3. A29: runway thresholds on `GET /paymaster`.
4. A42: the deny-list route for "Block wallet".
5. A53: a documented domain-draft slot in `PrivilegedActionFlow`, if the flow lacks one.
6. A28: `caller_broadcast {can_broadcast, reason_code}` on `GET /operations` and `GET /operations/:id`, backed by a seeded RBAC row naming the human roles that may broadcast (proposed: TOKO, with SECO as back-up), so Broadcast is gated on the caller as Sign is.
7. A28: per-role server-side redaction of `detail_line` (and any case reference on op, transfer-request and freeze payloads): MLRO and CCO get the case id; every other role gets the generic "compliance hold" text with no case id. The same rule applies to the palette search provider, which returns nothing for a case id to other roles.
8. A28: on `GET /tokens/:id`, the role-holder vault quorums (`signatures_need`, `signers_total` per vault) that the header sub-line renders, the approved `token.paused_override` records for the token, and the `TR-` id on ops created for an SM-13 case.
9. A42: the customer risk rating read for the ADM-13 Risk column (MLRO and CCO only); if absent, DF-A72-03 stands.

Follow-ups 6 to 8 also need A28's own spec updated (they are not read-only field additions only: 6 adds an RBAC row and 7 a redaction rule). This note requests them; it does not edit A28.

## Design references

- `design:adToken`: `<workspace>\design\RealX App.dc.html`, admin shell `ad`, screen key `adToken` (product map ADM-14).
  - The page data on line 884: code `ADM-14`, title "Token operations", sub-line "Every on-chain operation needs an approval ID and a 2-of-3 custody quorum before broadcast." (the build renders the quorum from the payload, §3, which reproduces this text with the seed), the table title "Recent" with the columns Time / Operation / Trust / Approval / Tx / Status and three green "Confirmed" rows.
  - The "Waiting for signatures" block on lines 706-716: h2 Newsreader 400 24 px; the row grid `minmax(0,2fr) minmax(120px,1fr) auto`, gap 16, padding `16px 18px`, white, 1 px `#E3DED4`, radius 10; the id line Geist Mono 500 11 px `#5E6166`; the detail Geist Mono 12 px `#45484D`; the button `9px 16px`, pill, Geist 500 13.
  - The `OPS` mock and the `tokenOps` derivation on lines 858-859: segment colours `#1F5E4B` and `#E3DED4`, caption "n of 3 custody quorum", Sign → Broadcast at n ≥ 2, toast "{id} broadcast · register sync queued".
  - Design analysis §2 item 40 and §3.3 `adToken`.
  
  Fidelity target: token-level match of the TokenOpRow, the SignatureMeter and the Recent table at 1280 px.
- `design:TokenOpRow` and `design:SignatureMeter`: the same region. The build keeps the geometry and colours. Two things are deliberate product-map deviations, recorded in `evidence/A72.json`. First, one persona clicking Sign repeatedly to fill the meter becomes one signature per distinct signer under their own hardware-key login. Second, "Broadcast at n ≥ 2" becomes "Broadcast when the API reports `signed`", with the segment count taken from the vault quorum. Design analysis §4 item 23 states the test: no broadcast without an approved APR and the custody quorum, and "register sync queued" means the op is not complete until reconciliation passes.
- Mock-data bugs not reproduced (design analysis §6.4 items 4, 5, 6): OP-7731's trust is Bay Road Apartment; APR-2279 names one object; Unley Road has no mint before its offer closes. A28's seed already resolves them. The fidelity check compares layout tokens, not mock strings.
- `design-ext:ADM-13`, `design-ext:ADM-15`, `design-ext:ADM-40 token registry list`: [[A51 - Design extension B- operator console screens not in the prototype]] (`<workspace>\app\design-ext\admin\RealX Admin Ext.dc.html`, `screens.json`, `blocks.md`).
  - ADM-13: the wallet DataTable; the vault cards V1 3-of-5, V2 2-of-3, V3 1-of-N, V4 automatic, V7 runway KpiTile; P1-P10 as a CheckList; recovery cards with the Timeline; actions Block wallet, Approve recovery and Change gas policy; states default, waiting period, runway amber.
  - ADM-15: SplitListDetail, the seven-column DataTable, evidence hashes, the register-impact KeyValueList, the ApprovalsChain with the TIMELOCK card, the "notice sent" banner, the waiver blocked state, and the states default, timelock running, waiver blocked, not-eligible and empty. The adToken OP-7731 row links here (DF-A51-01).
  - ADM-40: the ten-column DataTable, the red "Break" pill as `adRecon` draws it, the brick "sandbox" pill, and the states default, paused and empty.
  
  A51 passes HG-REVIEW before this node starts. The DF findings the founder accepted are binding here.
- `adRecon` (ADM-17) "Gympie Road Retail 9,998 / 10,000 · Break · 2 units" is the pill and wording for a Δ on ADM-40 and a `broken` op. `adAudit` (ADM-31) "Custody quorum · Signed 3/3 OP-7726 mint APR-2270 · OK" is the audit row shape in the op drawer. `adApprovals` is the not-eligible and SoD wording standard reused through `HardwareKeyConfirm`.
- Tokens: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`, line `#E3DED4`, the TONE map (design analysis §1.1), Newsreader for titles, Geist for text, Geist Mono for ids, hashes, addresses, counts and countdowns. There is no new colour, weight or radius, and A05's lint runs over the owned paths.
- The product-map mock frames (coral BrickX style) for ADM-13, ADM-14 and ADM-15 in `data_admin.py` lines 309-366 are content references only: the pending list "A-7781 Recovery INV-009 · CCO ✓ · CFO pending · TL 48h", the custody panel "Tx 221 batchMint · signers 1/2 · Policy P1 ✓ P3 ✓ P4 ✓ P9 ✓", and the desk row "TR-118 Lost wallet PAD01 240 · ID + liveness · 48h · 31h left". Their look is superseded by the dc prototype.

## Requirements covered

- **ADM-14 Token operations console (M09).** Done means every component the product map lists exists and reads the API: token header with contract, supply against cap, state and the Pause button; role holders; new-action form with reason, legal basis and evidence; pending maker-checker list; precheck with canTransfer, identity registry and register impact; custody panel with policy checks and signatures; history with approval ids and transaction hashes. Each product-map action is reachable with its policy row: mint or batch mint, burn, freeze or partial freeze, pause (`SGL+R`, ratified within 2 h), unpause (CCO → SECO + CFO), and upgrade (72 h). The state "an action isn't complete until M11 reconciliation passes" holds on screen (no green "Confirmed" before `closed`). The prototype's three waiting and three recent ops render from the corrected seed.
- **ADM-15 Forced transfer and recovery desk (M09).** Done means requests show type, from, to, quantity, legal basis, evidence hashes, timelock and approvals. The register impact preview and the notice to the holder's old channels are visible. The three product-map actions (forced transfer LEGL → CCO + CFO (+ trustee), recover lost wallet SUPP intake → KYCA → CCO + custody quorum, estate transfer LEGL → CCO + CFO) can be started, followed and closed from the desk. The state "timelock can be waived only by CCO + CEO with a court order" is drawn as a blocked control with that sentence and enforced by the API.
- **ADM-13 Wallets, custody and recovery (M08).** Done means the wallet registry (address, investor, type, identity-registry status, risk, frozen), the custody vaults with quorum and policy, the signer policy rules P1-P10, the paymaster runway (Sev2 under 3 days surfaced as red) and the recovery requests with waiting period and guardian approval are on screen. The four product-map actions exist with their rows: Block wallet (MLRO, `SGL+R`), Approve smart-wallet recovery (KYCA → CCO), Change signer policy (SECO → admin quorum 3-of-5), Change gas-sponsorship policy (ENGO → CFO).
- **Charter screen ADM-40 Token registry list** (from A00, outside the 91). Done means every vehicle's token appears with state, contract, supply, cap, register units, Δ, modules and paused. A Δ other than zero is a red "Break" linking to ADM-17, and a row opens ADM-14.
- **Journey J09 Lost phone: wallet recovery.** Done means each console step has a working control and state against the compose stack:
  - step 2: SUPP triage on ADM-15
  - step 3: KYCA re-verification and submission on ADM-13
  - step 5: LEGL fallback review on ADM-15
  - step 6: the old-wallet freeze on ADM-14 is the op A28 creates under the recovery request's approval chain, tagged with the `TR-` id; TOKO acts only on that op (its signing and broadcast), never on a freeze drafted by hand from the form. The Playwright step asserts that the desk item moves to `old_wallet_frozen` under the same approval chain (one approval id chain, read through the API), and that a hand-drafted freeze of the same wallet is refused or warned by the form and never moves SM-13.
  - step 7: the CFO's approval is on ADM-30 (A66), with the notice banner here
  - step 8: the custody quorum signs `recoveryAddress` on ADM-14 after the timelock, each signer under their own login
  - step 9: the handover to REGO's register update on ADM-17 (A73)
  
  The failure paths "holder objects → disputed → legal review" and "re-verification fails → rejected" each have a drawn state. The third, "court order → waiver by CCO + CEO", is not offered for `lost_wallet` under A28's proposed `waivable_types` seed: the waiver control is absent with A28's sentence, and this is recorded as product-map deviation DF-A72-04 pending the counsel and founder question (HG-LEGAL). The waiver path itself is drawn and tested on forced-transfer types. A81 runs the journey end to end. This node proves each step in its own Playwright specs with the other steps driven through the SDK.
- **G07 Holder cap.** Done means every mint, forced transfer and recovery shows the stored G07 evaluation with the percentage after the operation and the maximum allowed, a failing evaluation leaves submit unavailable with A28's `TOP_HOLDER_CAP` sentence, and nothing in the browser recomputes the cap.
- **G10 Trading halt.** Done means the token's `paused` and `halted` states are visible on ADM-40 and ADM-14 with cause and ratification countdown. Value-moving kinds are absent or blocked with `TOP_TOKEN_PAUSED` or `TOP_BREAK_OPEN` while paused or broken, except through an approved `token.paused_override` requested from ADM-14 ("Request paused override", CCO → LEGL, `4E`) and referenced on the forced-transfer, recovery or unlock draft. Pause is `SGL+R` through the modal, and unpause is only the V2 `6E+Q` draft.
- **G13 Maker-checker.** Done means no control on these screens creates an operation outside A28 and A14, decides outside `HardwareKeyConfirm`, or signs outside A15 with a fresh hardware-key assertion by a distinct eligible signer. Broadcast appears only when the API reports the quorum met and the caller's `caller_broadcast.can_broadcast`. The maker, the checkers and every agent see the API's refusal and no button. (G22 is rendered, not enforced, through the parity table, `AgentMarker` and the agent refusal sentences.)
- **M08 Wallets and custody.** Done means the product map's M08 purpose ("investors' non-custodial wallets, custody vaults and signer policies, gas sponsorship and smart-wallet recovery; no omnibus sub-ledger") is readable on ADM-13 from live data, with no screen or copy implying RealX holds investor keys.
- **M09 Token operations.** Done means every operation of the actions console reaches its catalogue row from ADM-14 or ADM-15 and is followed to `closed` or `broken` on screen: deploy (with configure, which A28 drives as the tail of the same V1 deploy approval and which these screens show, not start), mint, batch mint, burn, freeze, partial freeze, unfreeze, pause, unpause, halt (shown, set by A48), forced transfer, recovery, rule parameter change, upgrade.
- **SM-06 Token.** Done means every state has a label, tone and allowed-action set (§6), the header's actions follow `allowed_kinds` from the API, and `paused → active` is only reachable through the V2 draft. `deployed → configured` is shown as A28 drives it after the deploy approval, and `active → migrating` is shown as the migration runbook (A103) drives it; neither is started from these screens.
- **SM-07 Holding.** Done means wallet detail and register-impact previews show `active`, `locked`, `partially_frozen`, `frozen`, `recovered_out`, `eligible_empty` and `deregistered` from A27's Holding read model, and unfreeze is only offered as a four-eyes draft.
- **SM-13 Wallet recovery or forced transfer.** Done means all 14 states render on the desk and the recovery cards; `checker_approval → executing` is shown only with a waiver record; `old_wallet_frozen` precedes `timelock`; the "notice sent" banner precedes any executable state; `closed` requires `register_updated` and a clean recon, as reported by the services.

## Gauntlet loop for this node

- **Specify.** Frozen inputs: this note; A28's, A15's and A29's published SDK schemas and error catalogues, including the read-contract follow-ups listed under Owned paths; A14's detail and decision contracts; A53's `src/shell/index.ts` surface, `PrivilegedActionFlow` and `HardwareKeyConfirm`; A51's extension file, `screens.json` and `blocks.md` (after HG-REVIEW); A05's `tokens.json`; A03's SM-06, SM-07, SM-13 and TokenOperation state lists; A42's published deny-list route for "Block wallet", its need-to-know rule and its risk-rating read (follow-ups 4 and 9); A35's document-picker and `DocumentRow` contract; A48's halt payload (cause, reopening criteria); A104's MCP tool catalogue for the parity rows; the published contracts (or recorded fixtures, per the dependency note) of A101, A98 and A44; A06's and A28's seed (the three waiting and three recent ops, eight trusts, the Gympie Road break, a recovery in timelock, TR-118). The node QA records the hashes in the evidence manifest and writes the hidden specs and fixtures to `<workspace>\gauntlet\hidden\A72\` first.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: a short lead session wires the `(token)` route group, the page frames, `state-maps.ts` and the badge and search registrations against the AM3 stack. Then four parallel sub-agents work on disjoint folders:
  - (a) `console/` and `signing/`: the ADM-14 header, pause and unpause, role holders, new-action form per kind, precheck panel, pending list, signing queue with the meter and the swap-point test, and history with the op drawer.
  - (b) `desk/`: ADM-15 with both sources, the detail pane, the chain, the countdown, the notice banner, the waiver record, and every SM-13 action.
  - (c) `wallets/`, `vaults/`, `paymaster/` and `recovery/`: the ADM-13 tabs, Block wallet, rotation, suspension, signer and gas policy drafts, and the recovery cards.
  - (d) `registry/`, `parity.ts` and `contract-tests/`: ADM-40, the parity table with the staff-token and MCP runs, the error-sentence generator test, and the state-map tests.
  
  Integration: Playwright specs for every screen state at 1280 and 1024 px with the seeded personas (TOKO, CCO, CFO, SECO, LEGL, SUPP, KYCA, MLRO, ENGO, CEO and three V2 signers), then `npm run verify`, then `app/evidence/A72.json` (routes with `screen` and `designKey`, the screenshot set, the parity table, the error-code coverage table, the deviation list and DF-A72 findings).
- **Challenge (node QA, Opus 5.5 high).** The node QA runs the hidden specs, then does the following:
  - drives a V2 mint from draft to `closed` with three distinct signers and a forced transfer through the 48 h timelock under A09's injectable clock
  - replays the challenges below by hand
  - compares its crops of `adToken` and of extension B's three frames with screenshots at token level
  - calls every parity row with a seeded staff token through the SDK and the MCP server
  - greps the owned paths for `sha256(`, `crypto.subtle.digest`, `createHash`, raw hex, `localStorage` and SDK decision operations outside the allowed folders
  - checks that only the owned paths changed
  
  It writes `app/evidence/A72.qa.md`.
- **Gate.** A pass unlocks A78, A79 and A81. A fail returns findings by severity. At most three fix loops, each re-running the whole Challenge.
- **Escalation trigger.** Any of these escalates:
  - a third loop failing on the same finding
  - any path on these screens that lets a signature, broadcast, waiver, pause decision or approval happen without the API's authority, or that shows an operation as done before reconciliation (Critical, stops the milestone)
  - a need to change A28's, A15's, A29's, A42's or A14's contract beyond the listed follow-ups 1-9 (a spec defect upstream, reopened there)
  - an extension-B frame that cannot be built from A05 without raw hex (sent back to A51)
  - a counsel answer that changes a checker set or waiver rule (a data change in the owning node, then a re-run here)

## Node-specific challenges (node QA)

- **One person fills the meter (G13, custody).** The node QA drafts and has approved, through the SDK, a fresh V2 burn on Bay Road Apartment so that it sits at `awaiting_signatures` with `signatures_have = 0` of `signers_total = 3` (`signatures_need = 2`); the seeded OP-7730 and OP-7729 are already `signed` and OP-7731 already carries S2's signature, so none of them is used here. Signer S1 of V2 signs the fresh op, reloads and clicks Sign again, then signs again in a second browser. Then the CFO who approved the parent request tries to sign, and the TOKO who drafted it tries to sign. After the first signature the meter must stay at 1 of 3 (the API's count), and the button must be replaced by "Already signed by you", "You approved this request, so you cannot sign it" and the maker sentence. Only when S3, a distinct eligible signer, signs does the op reach `signed`. A forced SDK call must render `P9_INITIATOR_IS_SIGNER` or the duplicate-signer refusal. The prototype's click-to-increment behaviour reappearing in any form fails the node.
- **A forged count and a premature Broadcast (G13, money).** The node QA intercepts `GET /operations` and sets `signatures_have = 2` and `state = signed` on OP-7731, which is really at 1 of 3. The Broadcast button may appear from the forged payload, but the API must refuse (`TOP_NOT_SIGNED` or `TOP_BUNDLE_INVALID`). The screen must render the code and must not toast. After a real quorum, a double click on Broadcast by TOKO must produce one A27 call (network log and A28's op history).
- **Broadcast by the wrong role (G13).** As SUPP, CFO and KYCA, open OP-7730 (seeded `signed`). No active Broadcast button may appear; the row shows the API's `caller_broadcast.reason_code` sentence. A forced SDK call to `POST /operations/OP-7730/broadcast` from each of those sessions must be refused by A28 with its code, the op must stay `signed` (asserted through the API), and no A27 call may appear in the network log.
- **Confirmed is not closed (money, N03).** A mint confirmed on chain whose registry ack is rejected by the sandbox registry (A18's reject scenario), and a second whose ack arrives but whose targeted recon finds a 2-unit drift. Neither may ever show the green "Confirmed". The first must read "Confirmed · reconciling" and then red "Break". The second must show "Break · 2 units" on ADM-14's history and a Δ of 2 on ADM-40 linking to ADM-17. A green pill on any `broken` or `reconciling` op fails the node.
- **The timelock that the browser shortens (security, SM-13).** With the browser clock set 49 hours ahead and `timelock_until` edited in the intercepted response, the countdown may read zero, but Sign must stay replaced by the API's "signable from" sentence, and a forced call must render `P10_NOT_EXECUTABLE_YET`. Then the waiver path: CCO requests a waiver with no court-order document (the blocked state must show, and a forced call must be refused `TOP_WAIVER_NEEDS_COURT_ORDER`); on a smart-wallet lost-wallet recovery the waiver control must be absent with A29's sentence; on a lost-wallet **token-fallback** request (TR-118) the waiver control must also be absent, with A28's `TOP_WAIVER_TYPE_NOT_WAIVABLE` sentence, and a forced SDK call with a valid court-order document and a CCO session must be refused with that code and leave the request in its state (under the proposed seed; if Specify settles `lost_wallet` as waivable, the node QA fixes the opposite expectation from A28's settled row before the build).
- **Pause then move value (G10).** CCO pauses Marion Street Warehouse through the ratify modal. Then LEGL drafts a forced transfer and TOKO drafts an unlock and a mint on the same token. The forced transfer and unlock must be blocked with `TOP_TOKEN_PAUSED`, and the override picker must list nothing. CCO then uses "Request paused override" on ADM-14 for the forced transfer (legal basis left empty first: the flow must refuse; then filled), and LEGL approves it on ADM-30. The picker on the forced-transfer draft must now list that override, and the draft referencing it must pass the paused precheck and submit; the unlock draft, which the override does not cover, must stay blocked. The mint must be refused outright, with or without an override. No V3 "Unpause" may exist anywhere. When the 2 h ratification lapses, the header must say the token remains paused and name the opened unpause request.
- **Holder cap on a forced transfer (G07, compliance).** An estate transfer of 12 units to a recipient who already holds 4.95 % of Bay Road through an associate. The register impact preview must show the post-transfer percentage from the stored evaluation, the precheck must fail with "max allowed n units", and submit must be unavailable. The node QA then edits the units in the draft and checks that the preview and hash change visibly and the flow re-runs the precheck.
- **A staff agent at the signing desk (agent misuse, D04).** With a seeded `rx_stf_` token for TOKO holding every M08 and M09 scope the role allows, the node QA does the following through the SDK and the MCP server:
  - drafts a burn (it must succeed and appear with `AgentMarker` "agent rx_stf_… on behalf of {TOKO}")
  - calls custody `approve`, `broadcast`, `waive-timelock`, the pause act, a veto, `suspend` and the SUPP intake route
  
  Every call after the draft must be refused before scope evaluation (`P9_TOKEN_CREDENTIAL`, `APR_CHECKER_MUST_BE_HUMAN` or `AGNT_NOT_ALLOWED`), leave a G22 gate evaluation, and show in the op's audit drawer as a refusal. A human signer who is the owner of that token must also be refused as signer of the op the agent drafted (SoD counts the agent as its human).
- **Need-to-know and tipping-off (AML, privacy).** As SUPP, IRCM, CFO and TOKO, open OP-7729 and the frozen wallet 0x2c…7f. The row must read the generic "compliance hold", and the string `AML-118` (or any other case id), any AML case field, link, reason or SMR state must be absent from the DOM and from every network response of those sessions (the node QA scans the HAR). The wallet's frozen reason must be the generic one. As MLRO and as CCO the case id shows and links to ADM-25. A palette search for `AML-118` must return nothing for SUPP, IRCM, CFO and TOKO. Any case id or case field reaching a non-MLRO/CCO session fails the node.
- **Design fidelity of the drawn region.** Token-level comparison of the TokenOpRow grid, paddings, radii, the SignatureMeter segment height, radius and colours, the caption font, the Sign and Broadcast button colours, the "Waiting for signatures" and "Recent" headings, and the Recent table against the `adToken` crop at 1280 px must show no drift beyond 1 px other than the recorded deviations (distinct-signer rule, quorum-driven segment count, the payload-driven quorum in the sub-line, caller-gated Broadcast, "Confirmed · reconciling" and "Break" pills). Extension B's ADM-13, ADM-15 and ADM-40 frames must match in every drawn state.

## Pass criteria

- **Hidden tests.** The node QA's hidden specs in `<workspace>\gauntlet\hidden\A72\` pass 100 % on three consecutive runs against the Docker stack with the sandbox authenticator, at 1280 px and 1024 px.
- **State coverage.** Generated tests iterate A03's lists and assert a label, a tone and the allowed-action slots for every state of SM-06 (12 states), the TokenOperation table (14), SM-07 (7) and SM-13 (14), and every state has at least one screenshot in `evidence/A72/`. Every drawn extension-B state of ADM-13, ADM-15 and ADM-40, and the ADM-14 states default, empty queue, paused, timelock, signed, broken and not-deployed, has a Playwright screenshot at both widths, listed in `evidence/A72.json`.
- **Gate enforcement.**
  - G07: a holder-cap breach blocks submit with the stored evaluation shown.
  - G10: a paused token blocks the forced transfer, unlock and mint as §3 states; a forced transfer referencing an approved `token.paused_override` is accepted; and no V3 unpause control exists.
  - G13: no signature from the maker, a checker, a duplicate signer or a token credential is ever counted; no broadcast happens before `signed` or from a role without `can_broadcast`; no waiver happens without a court-order document or on a request type outside A28's `waivable_types`.
  
  Each is a named test run against the live API, with the refusal code rendered and the op state unchanged afterwards (asserted through the API, not the DOM).
- **No false success.** Across the full run, no op, request or recovery shows a green "Confirmed" or "closed" pill unless A28 or A29 report `closed` for it. This is asserted by a sweep that compares every rendered pill with the API state of the same id.
- **Hash and rule discipline.** A grep of the owned paths finds no hashing call and no quorum, threshold, timelock or holder-cap number in code: every such value is read from the API payload or `packages/config` through the API. Copy strings are not exempt: the header sub-line's quorum is interpolated from the payload, and no timezone abbreviation (`AEST`, `AEDT`) appears as a literal. A planted import of `custody.approveOp` outside `signing/` (and of `tokenops.broadcast` outside `console/`) fails this node's own `contract-tests/import-boundary.test.ts`.
- **Parity (D04).** `parity.ts` lists every mutation the owned code calls (a static test over SDK calls). Every "never" row is refused for the seeded staff token through the SDK and the MCP server with the named code and a G22 gate evaluation. Every "yes" row succeeds within the mandate and writes "agent {token} on behalf of {person}" in A10's log, and every out-of-mandate case of §7 (trust outside the allow-list, kind outside the allowed actions, revoked token, expired token) is refused with the G22 code, audited, and creates nothing. The MCP runs go through A104's real server, not a stub. A100 imports the table unchanged.
- **Error and empty states.** Every refusal code in the A28, A15, A29, A14 and A11 catalogues that these screens can trigger has a sentence (a generated test fails on a missing one). Every table renders its empty, loading and error state with `request_id`, and no table requests more than its page size (network log).
- **Design fidelity.** The node QA's token-level comparison of the `adToken` region and the three extension-B frames shows no drift beyond 1 px in font family, size, weight, colours, radii, borders or paddings, other than the recorded deviations. A05's token lint over the owned paths finds no raw hex.
- **Journey J09.** Steps 2, 3, 5, 6 and 8 are each driven through these screens by the right persona in a Playwright run on both the smart-wallet path and the token fallback, and the fallback ends with the old wallet at zero, the register moved under one approval id and the desk item `closed`. On the fallback, step 6 is proven on the A28-created old-wallet freeze op (tagged with the `TR-` id) and the desk item reaches `old_wallet_frozen` under the same approval chain; a freeze drafted by hand never satisfies the step.
- **Housekeeping.** Accessibility: axe reports zero critical or serious findings on every page, and the Sign, Broadcast and Pause flows are keyboard-operable with a visible focus ring. `npm run verify` is green from a clean clone on Node 24.21.0. Only the owned paths changed. `app/evidence/A72.json` validates against A00's schema and matches disk.

## Human gates touched

- **HG-REVIEW (before this node starts):** the founder has reviewed and approved A51's extension, including the ADM-13, ADM-15 and ADM-40 frames. The accepted DF findings, and any decision on a merged desk list, are binding here.
- **HG-KEYS:** production V1-V7 key ceremonies, hardware-key issuance to real signers and rotation ceremonies are human. The console drives A15's kms sandbox, whose keys carry no value. "Rotate key" in the sandbox records a simulated ceremony and says so on screen.
- **HG-VENDOR (custody, registry, identity):** the custody signing provider (TP-SIGN) and the registrar sit behind A15's and A18's swap points. The `SIGN_AT_PROVIDER` path is tested against the sandbox's provider mode only, and no vendor call is made.
- **HG-AUDIT:** the "Request deploy" form shows the smart-contract audit slot as "open (HG-AUDIT)". This node never marks it filled.
- **HG-PROD:** no mainnet deployment, no real units and no real money. The console never offers a chain choice, and the SANDBOX badge and "sandbox" pills stay on while any adapter is not live.
- **HG-LEGAL (counsel questions, logged, not blocking):**
  - the exact plain-English wording of the "notice sent" and waiver screens
  - whether the trustee's countersignature must precede the chain call on each trust deed (A28's question)
  - whether a court order may also waive the notice
  - whether a lost-wallet recovery's 48 h timelock may be waived by CCO + CEO with a court order (A28's proposed `waivable_types` seed says no; the product map's J09 third failure path says yes; recorded here as DF-A72-04)
  - whether any signer, waiver or intake act must be personal, not delegable (the build already bars agents from all of them)
  - the recovery disclosure wording (A29's `recovery_disclosure_version`)
  - the GPL-3.0 position on the vendored T-REX contracts whose functions these screens name
  
  Each answer changes data or copy in the owning node, and this node's specs are re-run.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
