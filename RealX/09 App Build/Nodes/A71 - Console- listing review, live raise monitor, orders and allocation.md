---
title: "A71 - Console: listing review, live raise monitor, orders and allocation"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am7]
node: A71
milestone: AM7
lane: "Operator console"
oracle: no
human_gate: none
---

# A71 - Console: listing review, live raise monitor, orders and allocation

**Three operator-console screens for the primary raise, built on A36, A37 and A38 through the public SDK. ADM-07 Listing review shows the review queue and, per listing, the gate list exactly as the prototype draws it: G03, G21, G19, G20, G06, then G17 with the exact flagged phrases and where they appear, then the missing insurance certificate in red. It adds the pack constraints (read-only), blockers and warnings, out-of-limits decisions, human-only due-diligence, legal and valuation sign-offs, a comment thread and "Return with comments". The go-live decision stays in the approvals inbox, as the design places it. ADM-08 Live raise monitor has the countdown, target, pledged, funded and investors tiles, the progress card, the funnel (viewed, verified, committed, funded), commitments whose flags are never silent, and the client-money "Δ 0" strip. It offers pause (`SGL+R`), resume, extend, material change and close early, plus revert and refund. ADM-12 Orders and allocation has the order blotter, with an agent marker on orders placed under a mandate. It has the allocation workbench with the rule line, the scale-back preview, the file hash and versions. "Allocate" is blocked while client money shows any delta. It also has the refunds queue with its five-business-day clocks and a settlement status per order that never reads complete before reconciliation passes. The screens decide nothing. Every figure, verdict, state and allowed action comes from the API, and a staff agent can do through the API every maker action a person can do here, under the same gates.**

**Build status:** not started. **Milestone:** AM7 (Operator console). **Lane:** Operator console. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 8-12 agent-hours (build, challenge and verify loops).

**Depends on:** [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A36 - Offerings- offer terms, listing review, raise monitor (SM-04)]], [[A37 - Orders, escrow, allocation and refunds (SM-05)]], [[A38 - Finalise-and-mint and revert-and-refund pipelines]], [[A51 - Design extension B- operator console screens not in the prototype]], [[A06 - Seed data and scenario simulator]] (the Marine Parade, Kent Street and WLG01 seed and APR-2290), [[A43 - Communications, notifications, tasks and marketing claims (G17)]] (the G17 flags and their clearing in the go-live challenge), [[A104 - MCP server generated from the OpenAPI document]] (the MCP tool list the parity test reads)
**Graph change requested:** A06, A43 and A104 are not yet in A71's `deps` in `final-graph.json` or in its transitive closure. The graph must add them (none makes a cycle: A78 and A79 already sit above all three). Until the graph carries them, A71 cannot reach its gate. A42 and A66 are deliberately not dependencies: the supersede challenge uses A38's re-check exclusion, and approvals in the journeys are decided through A14's SDK route, not the ADM-30 screen.
**Unlocks:** [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]], [[A79 - End-to-end journeys- investor money path]], [[A90 - P2- registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off]]

**Oracle rule:** this is not an oracle node (`oracle: no`). No money, supply or holdings move in this node: A37 and A38 move them and carry the oracles. But these are the screens where an operator allocates client money, sends refunds and puts a listing in front of investors. So the node QA still writes hidden tests from this frozen spec to `<workspace>\gauntlet\hidden\A71\` before the build starts ([[App Gauntlet]] §2). The hidden set has four parts:
- A scripted mock of the A36, A37, A38, A12 and A14 routes. It returns planted wrong figures, a non-zero delta, an unexplained reduced allocation line, a superseded allocation file, a server `rule_line`, open G17 flags, a confirmed custody op before reconciliation, and every refusal code in the A36, A37, A38, A12 and A14 catalogues. It proves the screens trust only the server.
- Playwright specs against the Docker stack with the seeded personas and A11's sandbox authenticator.
- Its own crops of the prototype's `adListing` region and of extension B's ADM-08 and ADM-12 frames.
- The parity diff script over this node's action manifest.

The builder never reads that folder. The cases are published into `app/` only after the gate passes.

This note is not legal advice. The re-confirmation window, whether the trustee must be a named checker on go-live, whether out-of-limits terms need extra investor disclosure, and whether an agent may acknowledge a disclosure under standing consent are counsel questions that A36 and A37 record. This node renders whatever those modules return and never hard-codes a legal sentence.

## Build brief

**1. Scope, placement and what this node reuses (no API code here)**

- Routes in `apps/console/app/(offerings)/` under A53's shell. Each `page.tsx` exports `screen` and `designKey` as A53's route convention requires:
  - `/offerings/review`: ADM-07 review queue, `designKey` `adListing`.
  - `/offerings/review/[id]`: ADM-07 listing review, `adListing`.
  - `/offerings/live`: the list of offerings in `approved` to `refunded`, `design-ext:ADM-08`.
  - `/offerings/[id]/live`: ADM-08 Live raise monitor, `design-ext:ADM-08`.
  - `/orders`: ADM-12 blotter, `design-ext:ADM-12`.
  - `/orders/[id]`: order detail drawer route, `design-ext:ADM-12`.
  - `/orders/allocation/[offeringId]`: ADM-12 workbench and settlement status, `design-ext:ADM-12`.
  - `/orders/refunds`: ADM-12 refunds queue, `design-ext:ADM-12`.
  - The product map's paths (`/admin/offerings/review`, `/admin/offerings/:id/live`, `/admin/orders`) lose the `/admin` prefix because the console has its own origin (A53 §1). The sidebar entries already exist in A53's `nav.ts` under "Onboarding & listings". Routes carry ids only, never names (A53 §11).
- Feature code lives in `apps/console/src/features/offerings/`:
  - `queries.ts`: SDK calls only. A53's lint forbids `fetch`, and this node adds a rule that no component imports `@realx/sdk` directly.
  - `actions.ts`: the parity manifest (§6).
  - `copy.ts`: every sentence and the mapping from error code to sentence.
  - `review/`, `raise/`, `blotter/`, `allocation/`, `refunds/`, `settlement/`.
  - `badges.ts`: `registerBadgeSource('listing.review', …)`, the count from the review-queue summary, shown as nothing when empty.
  - `search.ts`: `registerSearchProvider` for offering codes such as `KST01` and `RX-ORD-` order ids.
  - `__tests__/` (Vitest with Testing Library against the mocked SDK) and `e2e/` (Playwright, picked up by A53's config).
- From A53, through `src/shell/index.ts` only:
  - `ConsolePage`, `ConsoleTable`, `FilterBar`, `StatusCell`, `IdCell`, `MoneyCell`, `WhenCell`, `MaskedCell` and `AgentMarker`.
  - `PrivilegedActionFlow` with `usePrivilegedAction(action_code)`: the only way this node creates or submits an approval request.
  - `HardwareKeyConfirm`: the only way this node obtains a fresh assertion, used for sign-offs and the `SGL+R` act.
  - The status card that follows SM-00, and the blocks `FunnelRow`, `ApprovalsChain` and `KeyConfirmModal`.
  - The loading, forbidden, not-found and session-expired states.
- From A05: `KpiGrid`, `TaskRow`, `CheckList`, `KeyValueList`, `DataTable`, `RaiseBar`, `Timeline`, `StatusPill` with `toneFor`, `Banner`, `DecisionBanner`, `GateRow`, `TextArea`, `FilterChip`, `Toast`, `gateLabel`, `formatAud`, `formatTokens`, `formatPct` and `formatDateAest`. Nothing is redrawn and no raw hex is written; A05's token lint runs over both owned paths.
- **The browser computes nothing that matters.** It computes no cap, fee, share, refund, delta, soft-cap test, eligibility verdict, allocation line or allowed action. Every figure is the server's integer cents, units or basis points passed through A05's formatters, or the server's `formatted` string.
  - A feature ESLint rule forbids `*`, `/`, `%`, `+`, `-` and `Number(` on identifiers ending in `_cents`, `_units` or `_bps`.
  - Times use `formatDateAest` from server timestamps. The closing-day countdown uses the response's `Date` header as "now". A53's lint bans `Date.now()`.
- **Specify points** are settled before the node QA writes the hidden tests. Each is a small contract question between upstream nodes that this screen exposes. Each is resolved as a recorded loop on its owner, never as code here:
  - (a) **One allocation path, one code, one approval per Allocate.** A37 is now the only drafter of `orders.allocation.lock` (`4E`, CAPM to CFO) through its `requestAllocationLock`, and A38's `POST /v1/settlement/offerings/{id}/finalise` is a thin maker route that calls A37's `previewAllocation` and then `requestAllocationLock` and returns A37's approval id. A38 drafts no allocation approval of its own (A38 F2). The console's "Allocate" calls only A38's `finalise`. The workbench reads A37's `allocation/preview`, `allocation/file` and `allocation/versions`. Two leftovers are settled at Specify as loop A38.1 (already recorded by A37): A38's text still names the call `lockAllocation` (A37's internal effector) where it means `requestAllocationLock`, and A38 must keep no `allocation.lock` code of its own. The Specify outcome is frozen as: one action code, `orders.allocation.lock`, and exactly one A14 request per Allocate click. `actions.ts` binds Allocate to that code only, and a hidden assertion counts the A14 requests for the object after one Allocate (exactly one; a second click with unchanged inputs returns the same request, and with changed inputs renders `ORD_ALLOCATION_LOCK_PENDING`).
  - (b) **One refunds record, one list route, one code, one approval policy.** A37 now owns the one `refund_batch` table, the one reason enum and the queue read `GET /v1/admin/refund-batches` (filters offering, state, reason) with `GET /v1/admin/refund-batches/{id}` for lines; A38's `settlement/offerings/{id}/refunds` becomes a filtered call to it (loop A38.1). The Specify step closes the remaining drift between A37 and A38 and binds the screen to the answer:
    - **Action code.** A37 spells the batch approval `order.refund.batch`; A38 spells it `orders.refund.batch`. A03's `actions.ts` spelling wins; the owner A37 confirms it and A38.1 aligns. `actions.ts` here binds "Issue refund" to that one code.
    - **Reason vocabulary.** A37's enum (`scale_back`, `excluded_at_mint`, `soft_cap_missed`, `revert`, `investor_cancelled`, `deposit_returned`, `material_change_not_reconfirmed`, `revoked_in_reflection`) is the one list; A38's `scaled_back`, `excluded_at_allocation`, `excluded_recipient` and `offering_cancelled` are dropped in A38.1 (lock-time exclusions are `scale_back` lines with a `reason_code`). The queue labels reasons from A37's enum only; a test fails on any other reason string.
    - **Approval policy.** `soft_cap_missed` and `revert` batches have no approval of their own: they run under A14's `AUTO` row on an exact match with the `file_sha256` carried by the revert approval (A37 refunds, A38 R1 and R3 agree). Every other reason is submitted by FINA through A37's `submitRefundBatch` under `4E`, FINA to CFO. "Issue refund" is therefore shown only for `4E` batches, and an `AUTO` batch shows the revert approval it is bound to. J05 step 2 is FINA preparing the `soft_cap_missed` or `revert` batch through A38's `POST /v1/settlement/offerings/{id}/refund-batch`, which the CAPM's revert then references.
    - **Revert code.** A36 and A37 spell the revert approval `offer.revert`; A38 spells it `offer.revert_and_refund` and calls that A36's spelling. A03's `actions.ts` spelling wins, and whichever of A36, A37 or A38 differs aligns in its own loop. `actions.ts` binds "Revert and refund" to that one code.
  - (c) **One button per offer action.** Proposed binding:
    - Pause, resume, extend and material change bind to A36's console routes, because they are SM-04 changes.
    - "Revert and refund" binds to A38's `POST /v1/settlement/offerings/{id}/revert` (`4E+Q`, CAPM to CFO; body `{reason, evidence_doc_ids[], refund_batch_id}`), because it opens the refund pipeline.
    - **Close early has no owner today.** A36 says `offer.close_early` is A38's trigger; A38 says it has no close-early path and an earlier close is A36's close-date change. The product map lists no close-early action on ADM-08. The Specify step records which of A36 or A38 owns it. If neither does, the button is not built and the ADM-08 action list drops it; the screen never invents a route.
    - If a sponsor "cancel and refund" request exists (A38 R0 source (c)), it becomes a CAPM-drafted revert with the request attached as evidence. The screen never renders two buttons for one act.
  - (d) **Allowed actions from the server.** A37 now meets its half: `admin/orders` rows, `admin/orders/{id}` and `admin/refund-batches` rows carry `allowed_actions[]`. Loop A36.1 is still needed: A36's `review` and `raise` responses must carry `allowed_actions[]` for the caller, and there must be a console list of offerings by state (`GET /v1/console/offerings?state=`). This way no button's presence is decided in the client, the same pattern A67 uses with the IAM admin API node A106.
  - (e) **Module of ADM-08.** Extension B tags ADM-08 as M05; the product map gives M04. This node uses M04 for the permission check and the nav entry, and records DF-A71-01 for A51.
  - (f) **Agent delegability of the protective and decision acts.** A36 freezes pause (`offer.pause`, `SGL+R`) as a protective act (A03 kind `act`, `delegable: false`): a staff token can neither execute nor draft it and is refused `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`) on the `requiresHuman` route (A12 §5). A36 also refuses out-of-limits decisions to tokens (the same code and reason) and sign-offs (`APR_CHECKER_MUST_BE_HUMAN`). This node follows A36: none of these is agent-confirm, and none uses A12's `428 CONFIRMATION_REQUIRED` route. If the founder later wants an agent to prepare a pause that the human confirms with a hardware key, that is a change to A03's `delegable.ts`, A36 and A12 (a recorded loop, A36.1), not a UI choice here.
  - (g) **Two read fields, loop A37.1 (re-opened).** A37's `allocation/preview` and `allocation/file` return no rule-line text; only A38's `finalise`, a write, returns "pro-rata, scaled to hard cap, 5 % cap". The workbench must show the rule line before Allocate, so loop A37.1 adds a server `rule_line` string (built from the offering's `allocation_rule`, cap scaling and holder cap) to both reads, frozen at Specify and planted by the hidden mock. The same loop adds a `refund_state` filter to `GET /v1/admin/orders`. A37 recorded that A37.1 was no longer needed for point (d); these two fields re-open it.

**2. ADM-07 Listing review (`/offerings/review` and `/offerings/review/[id]`)**

- **Queue.** A `ConsoleTable` over A36's `GET /v1/console/offerings/review-queue`, with columns Listing / Sponsor / Pack (AU-wholesale P1, AU-retail P2 with the brick "2027" pill) / Submitted / By / SLA / Blockers / State.
  - The SLA column is the A13 `listing.review` clock (5 business days) as a mono countdown, red when breached.
  - Rows sort by SLA. The empty state reads "No listings waiting for review."
  - The seed shows one row, Marine Parade Apartment, with age "5h", matching the `adCmd` task.
- **Header.** The design's page frame: mono eyebrow `ADM-07`, Newsreader 40 "Listing review", and the subtitle built from the review response: "{property} · {sponsor} · submitted {date} by {name}". The design reads "Marine Parade Apartment · Harbour & Co Property · submitted 22 Sep 2026 by Tom H.". A36 names the sponsor submitter here (HCO's SP-OPS in the seed), because the design's "Tom H." is the RealX CAPM who made APR-2290; A06's resolution table records the difference. The screen renders the server's string, and the seed-string diff is taken against A06's resolution table, not the raw design text, wherever A06 records a deviation.
- **"Gates" TaskRows in the design's order and wording** (pill left, title and description, mono meta right):
  1. Sponsor and owners verified (G03): Harbour & Co · 2 directors · Pass (green).
  2. Licence perimeter (G21): Trustee under own AFSL; wholesale only · Pass.
  3. Proof of property (G19): Title search 12 Sep 2026 · Pass.
  4. Valuation current (G20): Panel valuer · A${valuation} · Aug 2026 · Pass (design A$1,924,000; seed value per A06's correction).
  5. Residency and FIRB (G06): Established residential · foreign persons blocked · Set.
  6. Marketing claims (G17): “guaranteed income” on IM p.3 · “beats the market” on listing copy · 2 flags (amber).
  7. Insurance certificate: Not uploaded · Missing (red).

  Every string is A36's review field, built from A19 verdicts, A43 claim flags and A34 due-diligence items. The valuation figure is A06's corrected value (A36 records A$2,748,000 in place of the design's A$1,924,000); the screen shows whatever the server returns. The screen adds no row, reorders nothing and never turns a red or amber row green. Due-diligence rows are read live on every load, never from the submission snapshot. An item that went missing after submission, such as a withdrawn certificate, shows as a blocker immediately.
- **The G17 row.** The design's click shows a toast. Here the click opens the flag detail:
  - each flagged phrase in quotation marks with its location (document, version, page, or "listing copy"), from A43 through A36;
  - a link to the claims item on ADM-28 ([[A75 - Console- AML cases, complaints register, marketing compliance, disclosure library]]);
  - a "Return with comments" composer prefilled with one comment per phrase, each anchored to its `field`.

  The design's toast "Changes requested from Harbour & Co" appears only after `POST .../return` returns 2xx, and the row then reads "Changes requested" in brick.
- **Constraints panel (read-only).** A `KeyValueList` of A36's `constraints_snapshot`:
  - investor type (wholesale only, verified);
  - disclosure (IM, s708);
  - holder cap (the pack default, and the offering's value if different);
  - lock-in, residency rule, valuation maximum age and member-count trigger;
  - documents required with their versions;
  - DAF meter state;
  - the pack code and version ("pack AU-W-2026.1"), and "pack superseded · re-validate before material change" when A36 says so.

  The panel contains no input, select or edit affordance; a DOM test asserts this.
- **Validation.** Blockers (red) and warnings (amber) from A36's `blockers[]` and `warnings[]`. Each shows its `code`, `source` (rules, dd, docs, terms, claims, perimeter), detail and fix-it sentence. While any blocker exists, the page shows the banner "{n} blockers · go-live cannot be requested", rendering A36's `OFR_BLOCKERS_OPEN`.
- **Out-of-limits items.** Rows of `{field, value, limit, sponsor's reason}` with the reviewer's decision (`accept` with note, or `reject`, which returns the listing), through `POST .../out-of-limits/{field}/decide`. Each accepted item is listed again in the go-live request payload so the CCO sees it.
- **Sign-offs.** A `CheckList` of the sign-off slots A36's review returns, each showing Done with name, role and time, or Waiting. In A36's current spec these are `due_diligence` (CAPM), `legal` (LEGL), `valuation` (VALO) and `trustee` (TP-RE, who signs through the licensed-partner surface, not this console); the screen renders the list the server sends and adds or drops no slot.
  - A person holding the matching role signs through `HardwareKeyConfirm` (purpose `offering.sign_off`, over the server's review hash), then `POST .../sign-off`.
  - A sign-off is an attestation, so it is human-only. The slot is absent for an agent persona, and A36 refuses a token with `APR_CHECKER_MUST_BE_HUMAN`.
  - `OFR_SIGNOFF_DUPLICATE_PERSON` (one person holding two sign-offs) renders with its sentence.
  - The CAPM who signed due diligence sees, on the go-live request, the not-eligible note A14 returns.
- **Comment thread.** `POST .../comments`, anchored to a field where given. The sponsor sees posted comments and never the reviewers' internal notes; the thread labels each comment "visible to sponsor" or "internal", as A36's schema says. On a resubmission, the review shows the diff against the returned version (A36's version compare) and carries old comments forward as resolved or open.
- **Return with comments.** Shown to the roles `allowed_actions[]` lists: the product map names CAPM or CCO, while A03's table currently lists CCO and LEGL (a spec defect A36 has reported against A03); the screen follows the server. Reason and at least one comment are mandatory, then `POST .../return`. Product map: no maker-checker (`off`). A 2xx moves the row to "Changes requested" and removes the item from the queue.
- **Go-live, with the approval kept in the inbox.** The design deliberately has no Approve on `adListing` (design analysis §6.2), and this node keeps that split.
  - The page offers "Request go-live" to CAPM, or to CAPM's agent as maker. It opens `PrivilegedActionFlow` for `listing.go_live` (`4E+Q`, checker set CCO only) through `POST .../request-go-live`. The `+Q` custody leg resolves to `not_applicable` on this request (A36): no custody operation is bound to it, because the token's deployment and compliance configuration are J12 step 9, a separate ADM-14 approval (A72).
  - The flow shows the precheck results. A refusal renders every reason A36's precheck names, for example the two G17 phrases, a missing sign-off, an undecided out-of-limits item, a superseded document version, a stale simulation or `OFR_TOKEN_NOT_CONFIGURED`.
  - After submit, the page shows an `ApprovalsChain` exactly as A14's detail returns it: in the current specs, Sponsor owner approval (G13) Done, then the sign-offs, then CCO, with the custody leg shown as not applicable. It also shows the SM-00 status card and a link "Decide in the approvals inbox · APR-…" to ADM-30 ([[A66 - Console- command center, alerts centre, approvals inbox, audit log]]). A66 is not a dependency: the link is a plain route href that tests assert by value, and the inbox screen itself is exercised in A78 and A79.
  - No Approve button exists on ADM-07 for any role, the CCO included.
  - The "Listing waiting for review" badge clears when the queue summary is empty.

**3. ADM-08 Live raise monitor (`/offerings/live` and `/offerings/[id]/live`)**

- **List.** Offerings from `approved` to `refunded` (loop A36.1), each with its SM-04 state pill through `toneFor`, raised percentage, close date and a Δ column. Any non-zero delta shows in red at the top.
- **Header.** "{property} · {sponsor}" and the state pill.
- **Countdown.** Mono "closes {date} · {n} days left" from A36's `days_left` and `closes_at`. On the closing day it counts hh:mm from the response's `Date` header. After close it reads "closed {date}".
- **KPI tiles** (product-map components): Target (`hard_cap_cents`), Pledged, Funded, Investors, from A36's `GET /v1/console/offerings/{id}/raise`.
- **Progress card.** The `spOffer` OfferProgressCard: "{name} · {funded units} of {supply} tokens", the 14 px RaiseBar with the soft-cap marker, and "A${raised} raised · soft cap A${soft} ({pct}) · hard cap A${hard}". With the seed for Kent Street: 6,400 of 10,000, A$1,525,760, soft cap A$1,788,000 (75%), hard cap A$2,384,000, closes 14 Oct 2026, 21 days left.
- **FunnelRow.** Viewed, Verified, Committed, Funded, with the server's conversion percentages. Viewed and verified come from A36; committed and funded come from A37's `admin/offerings/{id}/orders/summary`. The extension mock reads 2,140, 612, 128 and 121; with the seed, committed and funded come from the Kent Street data.
- **Client-money strip.** It binds A37's `GET /v1/offerings/{id}/escrow`:
  - It shows `delta_cents`, the account-level `escrow_account_level.delta_cents` and `last_clean_recon_at`.
  - Both zero with a clean run: green tint, "Client money trust account = funded commitments · Δ A$0.00 at {time}".
  - Either non-zero: red tint with the amount and which level ("offer escrow" or "trust account"), plus "allocation waits for reconciliation (N04)".
  - Both zero and `last_clean_recon_at` null: amber, "Δ A$0.00 · reconciliation not yet run". It is never shown as clean.
  - A36's `raise.client_money_delta_cents` must equal A37's per-offering delta. If the two differ, the strip shows red with both values and the text "Figures disagree · report". A contract test checks that the two reads agree on the seed.
- **Commitments table.** Columns Time / Investor / Type / Tokens / Amount / Flags / State, from A36's raise commitments joined by order id to A37's order flags. Investor names are masked in A30's form ("A. Morgan", "Kwan Family SMSF").
  - Flags come from the server, including `capped_at_holder_cap` ("capped at 5% · asked 520, allowed 480" from the server's requested and allowed units), `may_be_scaled_back`, `firb_check_pending`, `reconfirm_pending` and `agent_placed`.
  - A row whose server units differ from its requested units with no flag renders the red chip "Reduction not explained · report" (see §5, defence in depth).
  - Agent-placed rows show A53's `AgentMarker`.
- **Requests and re-confirmations.**
  - The sponsor's open `offer_request` rows (extend, close early, cancel, material change, minor edit) show with reason, requester and SP-OWN approval. The CAPM turns one into an approval request through the matching action below, with the sponsor request attached as evidence.
  - While a material change is open, A37's `GET /v1/offerings/{id}/reconfirmations` (A37 now owns this read, not A36) lists order, deadline (5 business days) and state. Close, extend and finalise render A36's `OFR_RECONFIRM_PENDING` while any is open.
- **Alerts.** A38's `settlement.soft_cap_at_risk` (Sev3, "soft cap at risk · {n} business days to close") and `settlement.run_stalled` show as banners from A13's alert read, each linking to its alert on ADM-02.
- **Actions, each shown only when listed in `allowed_actions[]`:**
  - Pause offering: `offer.pause`, `SGL+R`, for the human roles `allowed_actions[]` lists (product map: CAPM or CCO; A36: CCO acts, the other role ratifies). `HardwareKeyConfirm` shows the ratify variant, "This acts at once. {Role} must ratify within 2 h". After the act the page shows the amber "Awaiting ratification · h:mm:ss" pill. A mandatory reason category is published on the property page. Pause is human-only (specify point (f)): an agent persona sees no Pause control, and a token call is refused by A36 with `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`).
  - Resume: `offer.resume`, `4E`, CAPM to CCO.
  - Extend: `offer.extend`, `4E`, CAPM to CCO. A36's `OFR_EXTENSION_LIMIT` is rendered.
  - Material change: `offer.material_change`, `4E`. A form lists only the fields in A36's `material_fields.ts` and shows the diff against the current version. Its note, "Investors re-confirm within 5 business days or are refunded", is the server's text.
  - Close early: built only if specify point (c) names an owner and route. Then it is shown only from `soft_cap_reached`, and otherwise renders the owner's refusal code.
  - Revert and refund: A38 `revert`, the revert code settled in specify point (b), `4E+Q`, CAPM to CFO, with reason, evidence documents and the prepared refund batch's id (FINA prepares the batch first on ADM-12). Only while SM-04 is `closed`; later it renders A38's `SET_REVERT_AFTER_FINALISING`.
  - "Finalise and mint" is a link to the ADM-12 workbench once the offering is `closed`. Mint signing stays on ADM-14 ([[A72 - Console- token operations, forced transfer and recovery desk, wallets and custody, token registry]]).
- **States** (extension B): default; delta not zero (red strip, and ADM-12's Allocate shown as blocked); empty ("Offer not live yet · opens {date}", returning zeros with the state, never 404); paused (the amber banner with the reason); failed, refunding and refunded (refunded total and the refund clock); minted (read-only with the final figures).

**4. ADM-12 Orders and allocation (`/orders`, `/orders/[id]`, `/orders/allocation/[offeringId]`, `/orders/refunds`)**

- **Blotter.** A `ConsoleTable` over A37's `GET /v1/admin/orders`.
  - Filters, each mapped to one of A37's query parameters: offering, investor id, state, flag, channel (`web`, `api`, `mcp`, `auto_invest`), token, and refund state (`refund_state`, added by loop A37.1, specify point (g)). The "agent only" chip is `flag=agent_placed`, never a client-side filter. Filters live in the URL as ids and codes only.
  - Columns: Order (mono id) / Investor (masked) / Offering / Requested / Allocated / Refund / Funds / Flags / Channel / State.
  - An agent-placed order shows `AgentMarker`: "agent {token prefix} on behalf of {person}", with the token and mandate version on hover and a link to ADM-35 for callers with `V` on M26 (Agent and API access).
  - With an offering filter, the KpiGrid shows Orders, Requested tokens, Allocatable and Refunds queued from A37's summary.
  - The order detail (`/orders/[id]`, loop A37.1) shows the SM-05 history with actor and `on_behalf_of`, gate-evaluation ids, acknowledgements (document, version, time, and "standing consent v{n}" for an agent), allocation line, refund line, confirmations and settlement fields. The investor links to Investor 360 (ADM-09, A68) by id.
- **Allocation workbench** (`/orders/allocation/[offeringId]`):
  - **Rule line.** The server's `rule_line` from A37's `allocation/preview` and `allocation/file` (loop A37.1, specify point (g)), shown before Allocate and never composed in the client. With the product map's example: "Rule: first-come. Orders funded before the hard cap fill in full; later orders are scaled back or refunded." Or A38's "pro-rata, scaled to hard cap, 5 % cap". If the read carries no `rule_line`, the slot shows "Rule not supplied · report" in red and the lock flow does not open (defence in depth, as for an unexplained reduction); the client never builds the sentence from `allocation_rule`.
  - **Preview table** from A37's `allocation/preview`: Order / Investor / Requested / Allocated / Refund / Fee / Flags / State (`scaled_back_first_come`, `scaled_back_pro_rata`, `capped_at_holder_cap`, `excluded_at_allocation` with its gate code).
  - **Totals row** from the file's `totals`: orders, requested, allocated, unallocated, allocated cents, fee cents, refund cents.
  - **File identity.** `inputs_hash`, `algorithm_version`, `register_version_id` and, once written, `file_sha256`, all through `truncateAddress`.
  - **Allocate.** Primary button, CAPM, opens `PrivilegedActionFlow` for `orders.allocation.lock` (`4E`, CAPM to CFO) through A38's `finalise`, which reaches A37's single `requestAllocationLock` (specify point (a)). One click produces exactly one A14 request. The flow shows the server's `payload_sha256`, which equals `file_sha256`, and the CFO decides in the inbox.
  - **Blocked states** (extension B), each rendering the server's code:
    - "Client money Δ A${n} ≠ 0 · allocation waits for reconciliation (N04)" (`ORD_ESCROW_DELTA_NONZERO` or the escrow read);
    - "Offer not closed" (`ORD_OFFERING_NOT_CLOSED`, `SET_OFFER_NOT_CLOSED`);
    - "Soft cap missed · revert instead" (`SET_SOFT_CAP_MISSED`, with a link to ADM-08's revert);
    - "Allocation locked · version {n}" (`ORD_ALLOCATION_LOCKED`);
    - "Lock request pending · APR-…" (`ORD_ALLOCATION_LOCK_PENDING`, naming the pending approval id).
  - **Not-eligible, agent and checker views.** No role sees Approve or Reject on ADM-12, because the decision is made on ADM-30. After a lock is drafted:
    - the CAPM maker, and the human behind an agent that drafted the lock, see A53's `KeyConfirmModal` in the `not-eligible` state with A14's SoD note (segregation of duties counts the agent as its human);
    - an agent persona sees the `agent` state and the banner "Agents prepare. A person approves.";
    - the CFO, who is the legitimate checker, sees the request's status card and the link "Decide in the approvals inbox · APR-…", never a not-eligible note.
    Each view is driven by A14's approval detail for the caller, never by a role check in the client.
  - **Versions.** `allocation/versions` lists every version with state (`draft`, `locked`, `superseded`), approval id and the reason for supersession (a re-check exclusion by A38 through A37's `excludeAllocationLine`, under any of G01, G02, G04, G06 or G07, including an AML freeze that A37 consumes as G02). A superseded file carries a banner saying its hash changed, that a new lock request exists for the new version, and that any `allocation.mint` drafted against the old hash must be re-drafted.
- **Settlement status** (after the lock), from A38's `GET /v1/settlement/runs?offering=` and `/runs/{id}`:
  - An `ApprovalsChain` or `Timeline` of the run's steps with their approvals: allocation locked; remainders refunded; escrow release (link to ADM-18, [[A74 - Console- treasury, distribution run review, tax centre]], where FINA drafts `escrow.release`, `6E`); register entry acknowledged (link to ADM-17, [[A73 - Console- rule packs and threshold monitors, register and reconciliation]]); mint; reconciling; settled.
  - "Request mint" (CAPM maker, A38 `POST /v1/settlement/runs/{id}/mint`, `allocation.mint`, `6E+Q`) appears only when the run lists it. `SET_STEP_ORDER` renders with the current step. Custody signing is on ADM-14.
  - Per-order settlement shows `allocated`, then "Delivery incomplete · reconciliation pending" from the custody op's `confirmed` until A37 reports `settled`. `settled` shows the register entry date and transaction hash. No order row reads delivered or complete before A37's `settled` state, which A38 sets only after a clean reconciliation.
  - When `mint_queue_frozen` is true, a red Sev1 banner reads "Mint queue frozen · supply ≠ register · {break ids}" and every row stays "Delivery incomplete". The page offers no unfreeze control, because none exists in the API.
- **Refunds queue** (`/orders/refunds`, specify point (b)), over A37's `GET /v1/admin/refund-batches` (filters offering, state, reason) and `GET /v1/admin/refund-batches/{id}`:
  - Batches with Reason (A37's enum, labelled) / Offering / Lines / Total / Policy (`4E` or `AUTO`) / Approval / State / SLA. SLA is the server's `sla_due_at` and `breached`, shown as a business-day countdown, amber inside one day and red when breached (Sev2), mirroring A13's `orders.refund_sla` clock.
  - Line drill-down: order, amount, journal id, state (`pending`, `posted`, `failed`). A failed line is named, and the batch stays `executing`.
  - "Prepare refund batch" (J05 step 2): FINA maker, for an offering that is `closed` with the soft cap missed or chosen for revert, through A38's `POST /v1/settlement/offerings/{id}/refund-batch` `{reason}` (`soft_cap_missed` or `revert`). The prepared batch is `AUTO`; its id and `file_sha256` are what the CAPM's "Revert and refund" on ADM-08 carries.
  - "Issue refund": FINA maker, the one batch code settled in specify point (b), `4E`, FINA to CFO, through A37's `POST /v1/admin/refund-batches/{id}/submit`, shown only for `4E` batches (`scale_back` drafted at lock, `excluded_at_mint`, and single-order batches computed through `POST /v1/admin/refund-batches`). The flow shows the batch `file_sha256` as the payload hash.
  - An `AUTO` batch shows the revert approval id it is bound to, never an "Issue refund" button, and the sentence "Runs automatically only on an exact hash match".
- **States** (extension B): default; blocked (Δ not zero); blocked (delivery incomplete); not-eligible; empty ("No orders match these filters." and "No refunds queued.").

**5. Gates and rules: rendered, never decided**

- **G07 Holder cap.** Allocation lines and commitments show `capped_at_holder_cap` with the server's requested and allowed units. A reduced line with no flag is shown as unexplained, and the client refuses to open the lock flow for such a preview. This is defence in depth: A37's `ORD_ALLOCATION_INVARIANT` is the authority.
- **G11 Offer caps and escrow.** The raise, soft cap, hard cap, oversubscription ceiling, remaining units and both deltas are server figures. Allocate is disabled from the server's escrow read. A forced call is refused by A37 with `ORD_ESCROW_DELTA_NONZERO`, and the page shows that refusal.
- **G13 Maker-checker.** Every state-changing button goes through `PrivilegedActionFlow` or `HardwareKeyConfirm`. The page never calls `approvals.decide`; A53's lint enforces it. Every A14 refusal renders verbatim: `APR_MAKER_IS_CHECKER`, `APR_CHECKER_MUST_BE_HUMAN`, `APR_ROLE_NOT_HELD`, `APR_EXPIRED`, `APR_RECON_BREAK_OPEN`, `APR_NOT_RECONCILED`, `G13_APPROVAL_MISMATCH` and `ORD_FILE_HASH_MISMATCH`.
- **G17 Marketing claims.** The exact phrases and locations are shown. "Request go-live" with an open flag renders A36's precheck refusal naming each phrase.
- The pages add no rule an agent could escape, and relax no rule a person must meet.

**6. Agents and API parity (D04, G22)**

- `actions.ts` lists every action on the three screens with its SDK operation id, owner, A03 cell, policy code and delegability (`read`, `agent-maker`, `agent-confirm`, `human-only`). These are copied from extension B's `screens.json` and checked against it. A unit test asserts that every id exists in `@realx/sdk` and `openapi.json`, and that every `human-only` id is absent from A104's MCP tool list.

  | Action | Operation (owner) | Staff agent (`rx_stf_` token) |
  |---|---|---|
  | Review queue, review detail, versions, audit | `console.offerings.reviewQueue`, `.review`, `offerings.versions` (A36) | read, same RLS and masking |
  | Post a comment | `console.offerings.comments.create` (A36) | agent-maker |
  | Decide an out-of-limits item | `console.offerings.outOfLimits.decide` (A36) | human-only (A36 refuses tokens with `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`)) |
  | Due-diligence, legal, valuation sign-off | `console.offerings.signOff` (A36) | human-only (`APR_CHECKER_MUST_BE_HUMAN`) |
  | Return with comments | `console.offerings.return` (A36) | agent-maker |
  | Request go-live | `console.offerings.requestGoLive` (A36, `listing.go_live` 4E+Q) | agent-maker; the decision in ADM-30 is human-only |
  | Raise, commitments, escrow | `console.offerings.raise` (A36), `offerings.escrow`, `admin.offerings.orders.summary` (A37) | read |
  | Reconfirmations | `offerings.reconfirmations` (A37) | read |
  | Pause | `console.offerings.pause` (A36, SGL+R) | human-only: A36 refuses tokens with `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`); ratification human-only (specify (f)) |
  | Resume, extend, material change | `console.offerings.resume`, `.extend`, `.materialChange` (A36, 4E) | agent-maker |
  | Close early | owner and operation settled at specify (c); omitted if no owner | agent-maker if built |
  | Revert and refund | `settlement.offerings.revert` (A38, revert code per specify (b), 4E+Q) | agent-maker |
  | Blotter, order detail, allocation preview, file, versions | `admin.orders.list`, `admin.orders.get`, `admin.offerings.allocation.preview`, `.file`, `.versions` (A37) | read |
  | Allocate | `settlement.offerings.finalise` (A38, reaching A37's single `orders.allocation.lock` draft, 4E) | agent-maker; CFO decision human-only |
  | Settlement runs and steps | `settlement.runs.list`, `.get` (A38) | read |
  | Request mint | `settlement.runs.mint` (A38, 6E+Q) | agent-maker; signing on ADM-14 human-only |
  | Refunds queue and batch lines | `admin.refundBatches.list`, `.get` (A37) | read |
  | Prepare revert refund batch | `settlement.offerings.refundBatch` (A38, calls A37's `computeRefundBatch`) | agent-maker |
  | Compute a single-order batch, issue refund | `admin.refundBatches.create`, `.submit` (A37, batch code per specify (b), 4E) | agent-maker; CFO decision human-only |
  | Any approve, reject, ratify, veto, sign | A14 routes via the inbox | human-only, never offered here |

- **The agent persona.** No Approve, Reject, sign-off, out-of-limits decision, Pause, Ratify, Publish or Allocate-decision control is rendered for an agent. Where the person would see one, A53's grey banner "Agents prepare. A person approves." appears. Every agent-drafted request, comment and return shows `AgentMarker`. Segregation of duties counts the agent as its human, so the human behind the agent that drafted a lock is shown as not-eligible for the CFO slot.
- **Nothing the SDK cannot call.** The pages have no Next.js server action, route handler or cookie-only endpoint; a test walks the feature for `"use server"` and route files and finds none. Every rule is enforced server-side for UI and agent calls alike: G07, G11, G13, G17 and G22 in A12's middleware.

**7. Errors, loading and privacy**

- Every refusal renders as a `Banner` with the code and its sentence from `copy.ts`. A generated test over the OpenAPI error catalogues of A36, A37, A38, A12 and A14 fails if a code those routes can return has no sentence. A12's catalogue covers the G22 refusals that its middleware adds to every route here (`G22_AGENT_NEVER_CHECKER`, `G22_SCOPE_NON_DELEGABLE`, `423 AGENTS_PAUSED`, `423 AGENT_HELD`, `428 CONFIRMATION_REQUIRED`, `429 RATE_LIMITED` and the token codes); A14's covers every approval refusal. No hand-written code list is kept.
- Loading, empty, error (with `request_id`), forbidden and not-found states come from A53.
- After a mutation, the page refetches the affected row, the summary and the badge. It never edits a row optimistically, and a toast appears only after the 2xx.
- Investor names are masked by default. Unmasking uses A53's `MaskedCell` and the owning module's audited route.
- No reason for an AML exclusion is shown beyond the gate code the API returns to the caller's role.
- No `localStorage` or `sessionStorage` use beyond A53's column picker.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\console\app\(offerings)\
- <workspace>\app\apps\console\src\features\offerings\

Inside the route group, by convention: `offerings/review/page.tsx`, `offerings/review/[id]/page.tsx`, `offerings/live/page.tsx`, `offerings/[id]/live/page.tsx`, `orders/page.tsx`, `orders/[id]/page.tsx`, `orders/allocation/[offeringId]/page.tsx`, `orders/refunds/page.tsx`. The feature folder holds `queries.ts`, `actions.ts`, `copy.ts`, `badges.ts`, `search.ts`, `review/`, `raise/`, `blotter/`, `allocation/`, `refunds/`, `settlement/`, `__tests__/`, `e2e/` and `index.ts`.

The API routes belong to A36, A37 and A38. Shell primitives and extension-B blocks belong to A53, components to A05, and the extension file to A51. Changes this node needs elsewhere are recorded as loops, never edits: A36.1 (`allowed_actions[]` on review and raise, the console offerings list, and the close-early owner if A36 takes it), A37.1 (`rule_line` on the allocation reads and the `refund_state` blotter filter), A38.1 (already recorded by A37: `requestAllocationLock` naming, the one refund code and reason enum, and the refunds read as a filter over A37's), the A03 spelling of the revert and refund-batch codes, and the graph change adding A06, A43 and A104 to A71's dependencies. A missing block goes back to A51 as a finding.

## Design references

DESIGN = `<workspace>\design\RealX App.dc.html` (run by `support.js`); EXT-B = extension B from [[A51 - Design extension B- operator console screens not in the prototype]] (`<workspace>\app\design-ext\admin\RealX Admin Ext.dc.html`, `screens.json`, `blocks.md`, `design-findings.md`).

- **`design:adListing`: ADM-07.** DESIGN line 882, screen key `adListing`, in the admin dark shell.
  - Code `ADM-07`, title "Listing review", the subtitle, and `tasksTitle` "Gates" with seven `task()` rows in order: G03 Pass, G21 Pass, G19 Pass, G20 Pass, G06 Set (green), G17 "2 flags" amber with a click showing the toast "Changes requested from Harbour & Co", and Insurance certificate "Missing" red.
  - The `task` helper is at line 865.
  - The admin page template is at lines 630-650: `main` padding `32px 40px 64px`, gap 24, `max-width 1280px`; eyebrow `500 12px Geist Mono`, `.06em`, uppercase `#5E6166`; h1 `400 40px/1.05 Newsreader`; sub 15 px `#45484D`, max-width 720; KPI grid `repeat(auto-fit,minmax(200px,1fr))` with 1 px `#E3DED4` gaps, radius 12, value `500 24px Geist Mono`; TaskRow grid `auto 1fr auto`, gap 16, padding `14px 18px`, white, 1 px `#E3DED4`, radius 10, hover border `#17191C`, pill `500 11px Geist Mono` with padding `4px 10px`.
  - The pill tones come from `TONE` (line 740) and `STT` (line 760).
  - Design analysis §6.2 keeps the go-live approval in `adApprovals` and adds "Return with comments" on ADM-07. EXT-B records that addition as a DF note on `adListing`.
  - The constraints panel, validation, out-of-limits, sign-offs and comment thread are composed from existing A05 blocks (KeyValueList, CheckList, TaskRow, TextArea) in the same frame. Any composition the reviewer at HG-REVIEW has not seen is recorded as DF-A71-02 and sent to A51.
- **Neighbouring drawn regions this node must agree with.**
  - DESIGN line 880, `adCmd` task "Listing waiting for review · Marine Parade Apartment · Harbour & Co · Review · 5h": the queue row and its SLA age.
  - Line 862, `adNavDef` "Listing review" with the badge `'1'`: the badge source, which is data and never a literal (design bug §6.4-8).
  - `adApprovals` APR-2290 "Listing publication" (Medium, maker Tom H. Listings), with checks G20 Pass, G19 Pass, G17 2 flags amber and "Residency rule set": the go-live request this page drafts and links to. With two flags open, the precheck must refuse.
- **`design-ext:ADM-08`.** EXT-B's ADM-08 frame: the `spOffer` page for staff.
  - The base is DESIGN line 876 (`spOffer` KPIs and "Latest applications" rows) and the OfferProgressCard template at lines 581-587: 14 px RaiseBar, track `#ECE8E0`, fill `#1F5E4B`, 2 px ink soft-cap marker, mono "closes 14 Oct 2026 · 21 days left".
  - The `FunnelRow` block: four KpiTiles with mono conversion percentages and a green-tint `#E3EEE8` strip "Client money trust account = funded commitments · Δ A$0.00 at 06:00", turning red-tint with the delta.
  - The commitments DataTable with a Flags column ("capped at 5% · asked 520, allowed 480").
  - The actions: Pause (SGL+R ratify variant), Extend or close early (4E).
  - The states: default, delta not zero, empty.
- **`design-ext:ADM-12`.** EXT-B's ADM-12 frame.
  - KpiGrid: Orders, Requested tokens, Allocatable, Refunds queued.
  - The rule line.
  - The DataTable Order / Investor / Requested / Allocated / Refund / State (received, in escrow, allocated, scaled back, refunded, cancelled).
  - "Allocate" primary (CAPM to CFO) and "Issue refund" (FINA to CFO, 4E).
  - The two blocked banners: "Client money Δ A$412.00 ≠ 0 · allocation waits for reconciliation (N04)" and "Delivery incomplete · reconciliation pending".
  - The states: default, blocked (two), not-eligible, empty.
  - The product map's ADM-12 mock (KPIs Orders, Requested, Hard cap, Over cap refunds; rows `allocated`, `scaled back`, `unfunded`) is content only. EXT-B's drawing wins on visuals.
- **Shared.**
  - EXT-B `KeyConfirmModal` (seven states plus ratify) and `ApprovalsChain`, as coded by A53.
  - A53's `AgentMarker` (brick `AGENT` pill).
  - Tokens: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, amber `#7A4F00` on `#F6ECD6`, red `#A3312A` on `#F6E1DE`, grey `#4A4D52` on `#ECE8E0`.
  - Fonts: Newsreader for titles, Geist for text, Geist Mono for every figure, id and hash.
  - en-AU currency to the cent; whole-unit token counts with thousands separators.

## Requirements covered

- **Screens (owned).**
  - **ADM-07 Listing review.** Done when:
    - the queue, constraints (read-only), validation blockers and warnings, due-diligence and document sign-offs, "Return with comments" and the request for go-live exist;
    - the seven gate rows reproduce `adListing` byte for byte from the seed, except where A06's resolution table records a deviation (the valuation figure, the submitter's name), which must match A06's value byte for byte;
    - the offering team is shown as unable to approve its own review (the CCO slot is never offered to the due-diligence signer or the maker);
    - every sponsor offering visibly needs the platform CCO;
    - there is no Approve button on the page.
  - **ADM-08 Live raise monitor.** Done when:
    - countdown, target, pledged, funded and investors show;
    - the funnel shows viewed, verified, committed and funded;
    - commitments show their flags, and capped orders are never shown as silently accepted;
    - the client-money Δ strip shows both levels and never shows clean without a clean run;
    - extend or material change, resume and revert and refund exist as maker actions, pause exists as the human `SGL+R` act, and close early exists only if specify point (c) names its owner;
    - "Finalise and mint" hands off to ADM-12 and ADM-14.
  - **ADM-12 Orders and allocation.** Done when:
    - the blotter, the allocation workbench (first-come or pro-rata, cap scaling, rule line, file hash, versions), settlement to registered wallets and the refunds queue exist;
    - Allocate is blocked while Δ ≠ 0;
    - no order reads delivered before reconciliation passes.
- **Journeys.**
  - **J04** steps 7-8: the offer closes at the hard cap on ADM-08; CAPM allocates on ADM-12 with scale-back, and the CFO approves the lock and the remainder refunds. Hand-offs to steps 9-12 go to ADM-18, ADM-17 and ADM-14.
  - **J05** steps 1-2: the soft cap is at risk three days out on ADM-08 (the Sev3 alert and, if chosen, a human pause); at `closed` FINA prepares the `soft_cap_missed` or `revert` batch on ADM-12 (A38's `refund-batch`), and the CAPM drafts "Revert and refund" on ADM-08 carrying that batch. The batch has no approval of its own: it runs `AUTO` on the revert approval's hash (specify point (b)). The CFO decision at step 3 is made through A14.
  - **J12** step 8: CAPM compliance review on ADM-07, with the sign-offs A36 returns (due diligence, legal, valuation and trustee) and the counsel question on the trustee. Step 11: the offer goes live, `scheduled -> live`. This is not a click on ADM-08: A36's `offerings.schedule_sweep` runs it at `opens_at`, and the step's G13 act is the `listing.go_live` approval requested on ADM-07. ADM-08 shows the resulting `live` state.
  - Done when each step runs in Playwright against the Docker stack with the seeded personas. Approvals are decided by the seeded human checkers through A14's SDK decision route (`POST /v1/approvals/{id}/decide` with A11's sandbox authenticator producing the hardware-key assertion), never through the ADM-30 screen, which belongs to A66 and is not a dependency. A78 and A79, which depend on A66, reuse these specs and replace the SDK decision with the ADM-30 walk.
  - **Reconciliation table.** Each product-map action and journey step of the three screens is placed as follows. `actions.ts` carries the same rows, and the done check is a test that compares both against this table.

    | Product-map item | Where it lives | Why |
    |---|---|---|
    | ADM-07 "Approve for go-live" (CAPM maker, CCO checker) | "Request go-live" on ADM-07; the decision on ADM-30 (A66); the token's configuration on ADM-14 (A72, J12 step 9) | Design analysis §6.2 keeps Approve in the inbox; A36 gives the `+Q` leg no custody operation |
    | ADM-07 "Return with comments" | Operation on ADM-07 | No maker-checker (`off`) |
    | ADM-07 state "offering team can't approve its own review" | `not-eligible` state from A14's detail on the request | SoD is A14's |
    | ADM-08 "Extend or make a material change" | Operations on ADM-08 (A36, 4E) | SM-04 change |
    | ADM-08 "Pause offering" (SGL+R) | Human act on ADM-08; ratification on ADM-30 | A36: not delegable |
    | ADM-08 "Finalise and mint" (CAPM, CFO + CCO) | Link to ADM-12; "Allocate" and "Request mint" there; V2 signing on ADM-14 (A72) | A38 splits it into lock and mint approvals |
    | ADM-08 "Revert and refund" | Operation on ADM-08 (A38 revert) | Opens the refund pipeline |
    | ADM-12 "Allocate" | Operation on ADM-12; CFO decision on ADM-30 | A37's single lock request |
    | ADM-12 "Deliver tokens" (TOKO, custody quorum) | ADM-14 (A72); ADM-12 shows per-order settlement status only | A38: the V2 signing step of `allocation.mint` |
    | ADM-12 "Issue refund" | Operation on ADM-12 for `4E` batches; CFO decision on ADM-30; `AUTO` batches have none | Specify point (b) |
    | J04 step 7 "Offer closes at the hard cap" | Scheduled job (A36 sweep); shown on ADM-08 | No human act |
    | J04 step 8 "Allocates" | ADM-12 Allocate | — |
    | J05 step 1 "Chooses revert" | ADM-08 Revert and refund | — |
    | J05 step 2 "Prepares the refund batch" | ADM-12 "Prepare refund batch" | — |
    | J12 step 8 "Compliance review" | ADM-07 | — |
    | J12 step 11 "Goes live" | Scheduled job (A36 `schedule_sweep`); shown on ADM-08 | Its G13 act is `listing.go_live` |
    | Resume, close early (not in the product map) | ADM-08, from A36 (close early only if an owner exists) | Upstream actions the screen must expose |
- **Gates.**
  - **G07:** capped lines and commitments show their flag and the allowed units, and a preview with an unexplained reduction cannot be sent for lock.
  - **G11:** every cap, delta and soft-cap figure is the server's; Allocate is blocked on Δ ≠ 0; the forced call's refusal is rendered.
  - **G13:** every state change is a `PrivilegedActionFlow` or `HardwareKeyConfirm` call, with no decision button on these pages; not-eligible and agent states render from A14's detail.
  - **G17:** the exact phrases and locations are shown, and go-live is refused while any flag is open.
- **Modules.** **M04 Offerings and deals** (the console half: review, live raise, offer actions) and **M05 Subscriptions and orders** (the console half: blotter, allocation, refunds, settlement status). Done when every product-map component for ADM-07, ADM-08 and ADM-12 is on screen, and every product-map action and journey step sits where the reconciliation table under Journeys places it: an "operation on this screen" row is on screen and bound to an operation in `actions.ts`; an ADM-30, ADM-14 or scheduled-job row is rendered as its link, hand-off or resulting state, with no decision control. A test compares `actions.ts` and the rendered controls against the table row by row.
- **State machines.**
  - **SM-04:** all 14 offering states render on ADM-07 and ADM-08 with a label, a tone and the actions the server allows in that state.
  - **SM-05:** all 13 order states and the three `refund_state` values render on ADM-12 with a label and a tone.
  - A test iterates A03's state lists and fails on a state without a label.
- **Founder requirement D04 and G22.** Done when every action in `actions.ts` exists in the SDK and OpenAPI, every maker action succeeds for a staff token as it does for the person, every human-only action is absent from MCP and refused to tokens, and agent-placed orders and agent-drafted requests carry the marker.

## Gauntlet loop for this node

- **Specify.** The frozen spec is:
  - this note;
  - A36's console contracts (review-queue, review, raise, sign-off, comments, return, out-of-limits, request-go-live, pause, resume, extend, material-change, and close-early if A36 owns it) after loop A36.1;
  - A37's `admin/orders`, order detail, summary, escrow, reconfirmations, `admin/refund-batches` (list, detail, create, submit) and allocation preview, file and versions after loop A37.1 (`rule_line`, `refund_state` filter);
  - A38's settlement runs, finalise, revert, refund-batch prepare and mint after loop A38.1;
  - A14's approval detail, decision route and error codes, and A12's G22 error catalogue;
  - A03's spellings of the revert and refund-batch codes, and A104's generated MCP tool list;
  - A53's shell exports and lint rules;
  - extension B's ADM-08 and ADM-12 frames, `screens.json` entries and DF notes after HG-REVIEW;
  - A06's seed (Marine Parade in `compliance_review` with the seven rows and APR-2290; Kent Street live at 64 percent with the five application rows; an oversubscribed WLG01 fixture driven to `closed` through the sandbox by this node's e2e setup, not by a seed edit).

  The Specify step settles points (a) to (g). The node QA records `specVersion` and the contract hashes in the manifest, then writes the hidden material to `<workspace>\gauntlet\hidden\A71\`.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape:
  - The lead first writes `queries.ts`, `actions.ts`, `copy.ts` and the route skeletons with `screen` and `designKey`.
  - Then three parallel sub-agents work on disjoint folders:
    1. `review/`: ADM-07 queue and detail, gate rows, constraints, validation, out-of-limits, sign-offs through `HardwareKeyConfirm`, comments, return, request go-live through `PrivilegedActionFlow`, the badge.
    2. `raise/`: ADM-08 list and monitor, progress card, FunnelRow and Δ strip, commitments, requests and re-confirmations, and the actions (pause, resume, extend, material change, revert and refund, and close early if specify point (c) gives it an owner).
    3. `blotter/`, `allocation/`, `refunds/` and `settlement/`: ADM-12 blotter and detail, workbench and versions, blocked states, refunds queue with SLA, settlement chain and per-order status, and the frozen-queue banner.
  - An integration pass then runs the Vitest suites against the mocked SDK, the Playwright journeys (J04 steps 7-8, J05 steps 1-2, J12 steps 8 and 11) against the Docker stack, the screenshot gallery for every state at 1280 px and 1024 px, the parity manifest test, axe-core and `npm run verify`, and writes `app/evidence/A71.json`.
- **Challenge (node QA, Opus 5.5 high).**
  - Runs the hidden mock-server cases and Playwright specs.
  - Runs the node-specific challenges below.
  - Diffs the seed strings of `adListing` field by field.
  - Compares the state gallery with the dc crops by layout tokens.
  - Diffs the parity manifest against `openapi.json`, the SDK and A104's tool list.
  - Has a person, a staff token and an MCP call perform the same maker actions and the same refused decisions.
  - Walks every action with each persona: CAPM, CCO, LEGL, VALO, INVM, FINA, CFO, TOKO, a CAPM who signed due diligence, and an AGNT of each.
  - Checks for SDK imports outside `queries.ts`, arithmetic on money fields, server actions, and edits outside the owned paths.

  The node QA writes `app/evidence/A71.qa.md`.
- **Gate, fix loop, escalation.**
  - A pass unlocks A78, A79 and A90 and publishes the hidden cases into `app/`.
  - At most three fix loops; each re-runs the hidden set, the state gallery and the journeys.
  - Escalate to the user when:
    - the same assumption fails a third time;
    - a fix proposes computing money, a delta or an allowed action in the browser, putting an Approve or decision button on ADM-07, ADM-08 or ADM-12, showing an order as delivered before `settled`, or showing Δ as clean without a clean run (Critical: stop and reopen);
    - a field the screens need is missing from A36, A37 or A38 after the loops (a spec defect upstream, reopened there);
    - the design and the product map disagree on a state and neither the design analysis nor the product map settles it (ask the user).

## Node-specific challenges (node QA)

- **One cent of client money (money, G11).** The mock sets the per-offering delta to 0 and the account-level delta to +1 cent, with `last_clean_recon_at` set.
  - ADM-08's strip must be red naming "trust account · Δ A$0.01".
  - ADM-12's Allocate must be disabled with the N04 banner.
  - Then the node QA removes `disabled` in the DOM and submits. A37 or A38 refuses with `ORD_ESCROW_DELTA_NONZERO` carrying `delta_cents = 1`, the page shows that refusal, and no approval request exists (`GET /v1/approvals?object=` is empty).
  - Then A36's raise delta is set to 0 while A37's reads 412 cents. The strip must be red with both values and "Figures disagree", never green.
  - Finally both are zero with `last_clean_recon_at = null`. The strip must be amber "reconciliation not yet run", never the green "at 06:00".
- **The scale-back nobody explained (money, G07).** The mock returns a pro-rata preview in which one line has `allocated_units` 300 of `requested_units` 520 and an empty `flags[]`, and another line whose refund the server gives as A$23,959.21 where the planted correct figure is A$23,959.20.
  - The first line must show the red "Reduction not explained · report" chip, and the lock flow must not open.
  - The second must be rendered exactly as the server sent it. A$23,959.20 appearing anywhere proves a local calculation and fails.
  - In the real stack, a 300 + 300 order pair plus an associate's 100 units must show `capped_at_holder_cap` with "asked 300, allowed 100" as A37 returns it.
- **The hash moved under the checker (money, G13).** Two parts, both inside A71's dependency closure (no A42).
  - **Lock stage (mock).** The mock returns a pending `orders.allocation.lock` request whose `payload_sha256` no longer equals the current draft's `file_sha256`. The CFO's decision, sent through A14's SDK route in the test, is refused with `ORD_FILE_HASH_MISMATCH` (or `G13_APPROVAL_MISMATCH`); the workbench renders the code and shows the mismatch from A14's detail, and a new draft is required.
  - **Re-check exclusion (real stack).** WLG01's allocation is locked as version 1 (the CFO's decision through A14's SDK route). Before escrow release, the e2e setup lets one allocated investor's wholesale classification (G04) expire through A30's sandbox under A09's clock. A38's point-2 re-check then calls A37's `excludeAllocationLine`: version 2 is written with that line at zero, version 1 is `superseded`, the order moves to `refund_pending` (`excluded_at_mint`), an `excluded_at_mint` refund batch is drafted and a new `orders.allocation.lock` request is raised for version 2.
    - The workbench must show version 2, the superseded banner, the new `file_sha256` and the link to the new lock request; the refunds queue must show the new `4E` batch.
    - A CAPM "Request mint" against version 1's hash, forced through the SDK, is refused by A38 (`G13_APPROVAL_MISMATCH` or A38's named code), and the page renders the code.
    - No row shows a units figure taken from version 1 after the supersession, and the excluded order never reads `allocated`.
- **The agent tries the checker's chair (security, D04, G22).** A CAPM's agent token and the AGNT persona do the following in turn:
  - draft `listing.go_live`, `orders.allocation.lock` (through `finalise`) and the revert (the code settled at specify (b)), which are accepted as maker and carry `AgentMarker`;
  - call `console.offerings.signOff`, refused with `APR_CHECKER_MUST_BE_HUMAN`;
  - call `console.offerings.outOfLimits.decide`, refused with `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`);
  - try to decide the lock through A14, refused with `APR_CHECKER_MUST_BE_HUMAN` before scope evaluation;
  - try `console.offerings.pause`, refused with `AGNT_NOT_ALLOWED` (reason code `G22_SCOPE_NON_DELEGABLE`), with no pause, no hold and no approval request created (A36's frozen rule, specify point (f)).

  In the UI, a DOM sweep under the agent persona counts zero Approve, Reject, Sign off, out-of-limits Decide, Pause and Ratify controls on all eight routes, and the banner "Agents prepare. A person approves." is present where the person would see them. The human behind the agent opens the lock request's status on ADM-12 and sees A53's `KeyConfirmModal` in the `not-eligible` state, driven by A14's approval detail for that caller (the ADM-30 walk of the same check is A78's). The CFO sees the inbox link, not a not-eligible note.
- **Go-live with flags open or a certificate gone (compliance, G17).** With the seed, CAPM requests go-live for Marine Parade. The flow must show the precheck refusal naming "guaranteed income" (IM p.3), "beats the market" (listing copy), the missing insurance certificate and the missing sign-offs.
  - The node QA clicks the G17 row, returns the listing with the two prefilled comments, and checks that the toast "Changes requested from Harbour & Co" appears only after the 2xx and that the queue row disappears.
  - After the sponsor resubmits version 2 with the phrases fixed, the page shows the version diff and the old comments as resolved.
  - The node QA then withdraws the insurance certificate in A34 after submission. The row must turn red on the next load, from the live read.
- **Delivered too early (compliance, register as legal record).** A38's run for WLG01 reaches `mint_confirmed` and the custody op is `confirmed`, but the targeted reconciliation is still running.
  - Every order row must read "Delivery incomplete · reconciliation pending", and the settlement chain must show "confirmed · register sync and reconciliation pending".
  - Then the registrar's `-DRIFT-2` sandbox marker opens a Sev1 break. The frozen-queue banner must appear with the break ids, no row may change, and no control may offer to clear it.
  - Only after the break closes and a clean run lands do the rows read `settled`, with the register date and transaction hash.
- **The raise that runs past midnight (time, SM-04).** The seed closes Kent Street on 14 Oct 2026 ("21 days left") and is not edited. The e2e setup instead creates, through A06's scenario simulator in the sandbox, a live offering on Kent Street's terms whose `closes_at` is 2026-10-04T12:59:59Z (11:59:59 pm AEDT on Sunday 4 Oct 2026, the day clocks move from 02:00 AEST to 03:00 AEDT). A09's injectable clock drives the server.
  - At 2026-10-03T15:30:00Z (1:30 am AEST on 4 Oct) the page must read 0 days and "21:29 left" (the real elapsed time from the response's `Date` header), not the naive wall-clock "22:29". At 2026-10-03T17:30:00Z (4:30 am AEDT) it must read "19:29 left".
  - `formatDateAest` output must carry the zone the server's timestamp falls in: a time before the change shows "AEST", the close shows "4 Oct 2026, 11:59 pm AEDT". A fixed "AEST" label after the change fails.
  - After A36's sweep the page reads "closed 4 Oct 2026". A browser clock set two hours ahead must change nothing.
  - During a material change with re-confirmations open, extend (and close early, if built) must render `OFR_RECONFIRM_PENDING` and never a local "not allowed".
- **Every state looks like the design (design fidelity).**
  - The seed renders `adListing`'s seven rows in order with byte-identical strings and the design's tones, in the page frame of lines 630-650.
  - ADM-08 for Kent Street shows 6,400 of 10,000, A$1,525,760, A$1,788,000, A$2,384,000, "closes 14 Oct 2026 · 21 days left" and the five application rows with flags.
  - ADM-12 shows EXT-B's default, two blocked, not-eligible and empty states.
  - Each is compared at 1280 px and 1024 px against the dc crops. A hex value outside A05's tokens, a new component, a missing state screenshot, or a status string outside A36's and A37's vocabularies fails.

## Pass criteria

- **Hidden cases.** The node QA's cases in `<workspace>\gauntlet\hidden\A71\` pass 100 percent on three consecutive runs at 1280 px and 1024 px:
  - every planted server figure is rendered as sent;
  - zero locally computed money figures across 200 random allocation previews;
  - every forced action (disabled Allocate, go-live with open flags, sign-off by a token, pause or out-of-limits decision by a token, decision by the maker, stale file hash) leaves the expected server state (no approval request, no transition) and renders the sentence for its code;
  - one Allocate click produces exactly one A14 request of code `orders.allocation.lock` for the object, and a repeat with unchanged inputs returns the same request id;
  - the rule line shown is the planted server `rule_line` byte for byte, and a read without it keeps the lock flow closed.
- **State coverage.**
  - All 14 SM-04 states, all 13 SM-05 states and the `refund_state` values render with a label and tone from fixtures, each with a screenshot in `app/evidence/A71/`.
  - Extension B's listed states for ADM-08 and ADM-12, plus ADM-07's queue-empty, returned, blockers-open, go-live-pending and not-eligible states, each have a test and a screenshot.
  - A test over A03's state lists fails on any unlabelled state.
- **Journeys.** J04 steps 7-8, J05 steps 1-2 and J12 steps 8 and 11 pass in Playwright against the Docker stack three runs in a row, with the approvals decided by the seeded human checkers through A14's SDK decision route and A11's sandbox authenticator (the ADM-30 walk is added by A78 and A79). J05's run records exactly one revert approval and no separate approval for its `AUTO` batch, and J04's run records exactly one `orders.allocation.lock` request per Allocate.
- **Gate enforcement.** For each of G07, G11, G13 and G17, a planted-failure test proves three things: the control is inert or absent, the server's reason is shown, and a forced call is refused by the server with no side effect. A planted-pass test proves that the action proceeds.
- **Money and settlement truth.**
  - 100 of 100 planted deltas (±1 cent at either level, disagreeing reads, null reconciliation) render as non-green.
  - No order row reads delivered or settled in any fixture where A37's state is not `settled`, including `mint_confirmed`, a custody op `confirmed` and a frozen mint queue.
- **Design fidelity.**
  - A layout-token comparison of `adListing` at 1280 px against the dc crop shows zero differences in font family, size, weight, letter-spacing, colours, radii, borders or paddings beyond 1 px.
  - ADM-08 and ADM-12 match EXT-B's frames on the same scale.
  - The seed-string diff of the seven gate rows and the Kent Street figures is empty.
  - axe-core reports zero critical or serious findings on every route and state.
  - Keyboard-only completion of return-with-comments, request go-live and allocate succeeds.
- **Parity.**
  - Every operation in `actions.ts` exists in `@realx/sdk` and `openapi.json`, and every `human-only` entry is absent from A104's tool list.
  - Each maker operation succeeds, and each human-only operation is refused, identically for a person, a staff token and an MCP call in A100's harness. The harness runs here with a preview script when A100 is not built yet, and A100 re-runs it later.
  - No server action, route handler or SDK import outside `queries.ts` exists (lint).
- **Coverage and hygiene.**
  - Unit line coverage of `src/features/offerings/` is at least 85 percent, and 100 percent for the state-label tables, the Δ-strip selector, the unexplained-reduction check, the settlement-status mapper and `copy.ts`'s code table.
  - `npm run verify` is green on Node 24.21.0.
  - Only the two owned paths changed, plus the recorded loops A36.1, A37.1, A38.1 and DF-A71-nn with their owners' sign-off in the evidence.
- **Evidence.** `app/evidence/A71.json` and `A71.qa.md` are present and match disk. They list the screenshot set, the seed-string check, the parity result, the reconciliation-table check, the Specify decisions (a) to (g) and the contract hashes compared.

## Human gates touched

- **None crossed.** The screens run on the Docker stack with A11's sandbox authenticator, A37's sandbox money, A17's sandbox bank rail, A18's sandbox registrar, A28's custody sandbox and the local chain. No real listing, investor, trust account, key or mainnet is touched. HG-CAPITAL, HG-VENDOR and HG-PROD gate any use outside the sandbox.
- **HG-REVIEW (before this node starts).** The founder reviews A51's ADM-08 and ADM-12 frames and the `adListing` "Return with comments" note in the browser. Compositions on ADM-07 that the review has not seen (the constraints panel, sign-offs and comment thread) are recorded as DF-A71-02 for the next review. The founder's visual QA of the three screens happens again at [[A78 - Operator journey J20 end to end in Docker (AM7 exit)]].
- **HG-LEGAL (counsel questions recorded by A36, A37 and A38, not decided here).**
  - Whether the trustee must be a named checker on go-live beside the platform CCO. The checker set is A14 data, and this page renders whatever chain the API returns.
  - Whether a close-date extension alone is a material change, and whether 5 business days is the right re-confirmation window.
  - Whether terms outside RealX limits need investor disclosure beyond the IM.
  - Whether a refund may credit the client-money wallet or must return to the source bank account.
  - Per D04, whether the reviewer sign-offs or out-of-limits decisions must be personal beyond the human-only rule already applied here.
  - Answers change data upstream, not this node.
- **HG-PENTEST (later).** The console's allocation, refund and go-live surfaces, and the agent-marker and not-eligible rendering, are in scope of the external penetration test before go-live. This node passes on software evidence and records the gate as open.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
