---
title: "A57 - Buy flow: order, review and confirm, order status (INV-01 to INV-03)"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am5]
node: A57
milestone: AM5
lane: "Investor app"
oracle: no
human_gate: none
---

# A57 - Buy flow: order, review and confirm, order status (INV-01 to INV-03)

**The investor's purchase in a primary offer, built as the prototype's single stepped screen split into three routes. INV-01 has the quantity stepper, presets and an amount field, live eligibility GateRows (G01, G04, G06, G07 with the post-order percentage, and G11), a cap breach that offers the maximum, a deposit shortcut when funds are short, and the dark OrderSummaryAside to the cent from the server's quote. INV-02 has the disclosure documents that must be opened, the AcknowledgeCard with the wholesale "no cooling-off" statement, and "Confirm and pay" behind a passkey step-up and one idempotency key. INV-03 has the timeline (placed, escrow, soft cap, units issued on the register, tokens delivered), the scaled-back, refund-on-its-way, refunded, cancelled, expired, revoked, awaiting-funds and under-review states the design does not draw, a re-confirm banner after a material change, cancel while the listing permits, and a "Placed by your agent" marker. Every figure, gate verdict, state and status sentence comes from A37, A35 and A11 through the public SDK, so the screens decide nothing and an agent can do everything the screens do.**

**Build status:** not started. **Milestone:** AM5 (Investor app and preview). **Lane:** Investor app. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 12-18 agent-hours (build, challenge and verify loops): three routes, about 17 states at two widths, two Docker journeys, the hidden suites and three-channel parity. The graph still carries 8-12; the graph's `est_lo`/`est_hi` for A57 must be raised to match (a graph change, not made here).

**Depends on:** [[A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)]], [[A37 - Orders, escrow, allocation and refunds (SM-05)]], [[A35 - Documents, disclosure library, e-sign and acknowledgements (G12)]], [[A38 - Finalise-and-mint and revert-and-refund pipelines]]
**Graph additions requested (not yet in `final-graph.json`):** [[A06 - Seed data and scenario simulator]] (KST01, WLG01 and RX-ORD-55120 fixtures), [[A23 - Whole-stack Docker boot (AM1 exit)]] (the stack the Playwright journeys run on), [[A104 - MCP server generated from the OpenAPI document]] (the MCP leg of the parity checks) and [[A101 - Agent guard- anomaly detection, sensitive-action notices and tiered kill switches]] (the agent-notice check). All four sit in AM0, AM1 or AM3 and none depends on A57, so no cycle is created. Until the graph carries them, the Specify step checks that each has passed its own gate before A57's build starts.
**Unlocks:** [[A61 - Docker Desktop preview of the investor app (AM5 exit)]], [[A90 - P2- registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off]], [[A95 - P3- Auto Invest and PayTo mandates]]

**Oracle rule:** this is not an oracle node (`oracle: no`). No money, supply or holdings move here: A37 moves them and carries the oracle. But this is the screen where a person commits money, so the node QA still writes hidden tests from this frozen spec to `<workspace>\gauntlet\hidden\A57\` before the build starts ([[App Gauntlet]] §2). They cover three things. First, a scripted mock of the A37, A35 and A11 routes that returns planted wrong figures, failed gates, stale versions, out-of-order quotes, a reason field on a frozen account and refusals, to prove the screens trust only the server. Second, the reference crops of the prototype's buy screen at 1280 px and 375 px. Third, the parity checker that diffs this node's action manifest against the SDK and `openapi.json`. The builder never reads that folder. The cases are published into the repo only after the gate passes.

**Upstream contract A57 builds against (prerequisites, not later loops).** A57's hidden tests and screens assume the fields below. A37 is a gated oracle node, so these must be in A37's frozen spec, inside A37's oracle scope, **before A57's Specify step freezes**. They are not left to a loop two milestones later. If any is missing when A57 is specified, the Specify step stops and the gap is reopened on A37 (and A09 for item 6); the builder never builds against a lenient mock that has the field when the real service does not.

1. **`statement_hash` on placement.** `POST /v1/offerings/{id}/orders` accepts `statement_hash` (and `POST /v1/orders/{id}/reconfirm` likewise) and passes it to A35's G12 check, which refuses a mismatch with `ACK_STATEMENT_MISMATCH`.
2. **`status_label` vocabulary.** A37's ten values (`in_escrow`, `awaiting_funds`, `allotted`, `scaled_back`, `settled`, `refunded`, `cancelled`, `expired`, `rejected`, `reflection`) plus `under_review`, `refund_pending`, `revoked` and `checking` (for `submitted` and `eligibility_check`), with A37's written mapping rules. The one rule this screen depends on: an order whose `investor_cash` account is frozen maps to `under_review` whatever its state, and the order read carries no reason field for it. `pending_confirmation` is **not** needed: an agent order above its cap leaves no order (A37's `428 CONFIRMATION_REQUIRED`), and A60 shows the confirmation on ACC-05.
3. **Order read fields.** `GET /v1/me/orders/{id}` carries `channel` (`web` | `api` | `mcp` | `auto_invest`), `agent: { token_id, token_name } | null`, `cancel_reason` (`investor`, `agent_token_revoked`, `agent_mandate_revoked`, `listing_rule`, plus A37's value for an offering cancelled or reverted), `reconfirm_due_at`, and `refund: { state, reason, amount_cents, at, due_at } | null`, where `due_at` is the refund line's `sla_due_at` from `orders.refund_sla_business_days`.
4. **Terminal timeline entries.** The order timeline carries server-written `title` and `detail` for the terminal and extra events (`cancelled`, `expired`, `revoked`, `refund_pending`, `refunded`, `reconfirmed`), in addition to the six INV-03 keys, so no status sentence is written in the browser.
5. **A non-mutating confirmation challenge.** `POST /v1/offerings/{id}/orders/confirmation-challenge` (scope `orders:place`, no `Idempotency-Key`, writes nothing) takes the exact placement body without `confirmation` and returns `{ payload_sha256, purpose: 'order.confirm', summary }`, the canonical order hash A37 will verify. The same operation for reconfirm is `POST /v1/orders/{id}/reconfirm/confirmation-challenge`. (Equivalent: the quote response returns the hash, if A37 prefers; the screen flow below is the same.)
6. **Idempotency codes agree.** A key reused with a different body is one code with one status in A09 and A37 (A09's `422 IDEMPOTENCY_KEY_REUSED`, or A37's `ORD_DUPLICATE_KEY_CONFLICT` registered at the same status); `409 IDEMPOTENCY_IN_PROGRESS` is replayed-to by retrying with the same key.

This note is not legal advice. Several points are counsel questions recorded by A35, A37 and A90: the wording of the wholesale "no cooling-off" statement, whether a retail order carries a cooling-off period, whether an agent may acknowledge a disclosure under standing consent, and whether any purchase declaration must be made personally. This node renders whatever text and parameters those modules return and never hard-codes a legal sentence.

## Build brief

**1. Scope and placement**

- Three routes in `apps/web/app/(investor)/invest/` run under A52's investor shell. That shell supplies the public header with the investor slot, the footer with the CAR/AFSL disclosure, and the passkey session requirement. A visitor who is not signed in is sent to `/sign-in` with a return path. A signed-in investor who has not finished onboarding is sent to A56's next step by A52's `Account` lifecycle redirect.
  - `/invest/[code]/buy` → **INV-01 Buy tokens** (step 0).
  - `/invest/[code]/buy/review` → **INV-02 Review and confirm** (step 1).
  - `/invest/orders/[id]` → **INV-03 Order status** (step 2).
- `[code]` is the offering short code from A06's seed (Kent Street Terrace = `KST01`; J04's Wollongong offer = `WLG01`). It resolves to the offering through the SDK's offering read (A36). Each `page.tsx` calls A52's `defineScreen({ map: 'INV-01' | 'INV-02' | 'INV-03', design: 'buy', title })` from `src/shell/screen.ts`, so A52's build-time route test and the Screen Coverage Matrix can see it.
- All feature code lives in `apps/web/src/features/buy/`:
  - `api.ts`: the `@realx/sdk` calls listed below and nothing else. A52's lint forbids `fetch` outside `src/shell/api/`. This node adds a feature lint rule so that no component imports the SDK directly.
  - `actions.ts`: the parity manifest (§5).
  - `state/`: a small reducer for the stepper, the quote cache with request sequencing, `idempotencyKey.ts` (the key derivation in §3) and `unitsForAmount.ts` (the one allowed division, §1 below).
  - `errors.ts`: sentences for A37's and A35's codes, extending A52's `src/shell/errors.ts` table.
  - `slots.ts`: the extension point other nodes fill without editing this folder. It exports `registerOrderSlot(name: 'reflection' | 'inv01.regimeNote', component: ComponentType<{ order: MeOrder; refetch: () => Promise<void> }>)` and `OrderSlot({ name, order, refetch })`, which renders the registered component or nothing. `StatusStep` renders the `reflection` slot above the timeline; `OrderStep` renders the `inv01.regimeNote` slot under the fee note. A90 imports `registerOrderSlot` from `@/features/buy/slots` in its own feature folder. A unit test registers a dummy component for each name and asserts it renders with the order and a working `refetch`, and that an unregistered slot renders nothing.
  - `components/`: `BuyStepper`, `OrderStep`, `ReviewStep`, `StatusStep`, `EligibilityRows`, `AmountField`, `OfferProgressLine`, `TicketBoundsLine`, `DepositShortcut`, `DisclosureList`, `StatementBanner`, `ExistingOrderBanner`, `ConfirmButton`, `OrderTimeline`, `OrderStatePill`, `AgentMarker`, `CancelDialog`, `ReconfirmBanner`.
  - `__tests__/`: Vitest with Testing Library against the mocked SDK.
  - `e2e/buy.spec.ts`: Playwright, picked up by A52's config. If A52's glob does not cover `src/features/**/e2e/`, record it as a one-line loop A52.1. This node does not edit A52's paths.
  - Components come from `@realx/ui` (A05): `HorizontalStepper`, `QuantityStepper`, `GateRow`, `AcknowledgeCard`, `DocumentRow`, `Timeline`, `OrderSummaryAside`, `RaiseBar`, `StatusPill` (with `toneFor`), `Banner`, `TextField` (mono variant), `ValueChip`, `Toast`, `Button`, and A52's `ErrorState`, `EmptyState` and `LoadingState`.
- **Server and browser split.**
  - Reads of the offering, the order and the disclosure set run in server components.
  - The stepper, the quote, the acknowledgement tick, the passkey ceremony and the writes run in client components through `@realx/sdk`.
  - The browser evaluates no eligibility, money or state rule. The client never computes a fee, a share, a cap, a total or a gate verdict. Every figure on screen is either the server's `formatted` string or `formatAud`, `formatPct` or `formatTokens` applied to the server's integer cents, basis points and units.
  - A feature ESLint rule forbids `*`, `/`, `%` and `Number(` on any identifier ending in `_cents`, `_units`, `_bps`, `price` or `fee`.
  - One local calculation is allowed: `floor(amount_cents / unit_price_cents)` in the amount field, to pick a whole number of units. It lives only in `state/unitsForAmount.ts`, the single file the lint config exempts by path, and that file is tested at 100% (zero, below one unit, exact multiples, remainders, very large amounts). The next quote's `lines.units` replaces its answer. `eslint-disable` comments of any kind are forbidden in `features/buy/` (a lint rule and a grep in the challenge).
- **Time and formatting.** Times are rendered with `formatDateAest` (Australia/Sydney) from the server's ISO timestamps. A52's lint bans `Date.now()`. The "hours left" countdown is `funding_deadline_at` minus the server time from the response's `Date` header, rounded down. Currency is en-AU with the `A$` prefix and two decimals. Units have thousands separators.

**2. INV-01 Buy tokens (step 0)**

- **Entry guard.** The offering must be `live` and not `suspended` for the investor's regime. Otherwise the route redirects to the property page (A54) with the design's branch from `buyClick` (dc line 837):
  - "Coming soon" → the toast "We'll email you when this offer opens".
  - Fully funded or closed → the property page as it is, where A54 renders its own "Fully funded" (`funded_no_resale`) state. The design's `buyClick` sends this case to the sell screen, but INV-05 is a P2 route no AM5 node builds, so A57 never redirects to `/invest/[code]/sell` in P1.
  - `suspended` → the Banner "This offer is paused", with no reason.

  The first quote call confirms the guard, and A37's `ORD_OFFERING_NOT_LIVE` wins when the server and the client disagree.
- **Layout as designed** (dc lines 399-459). The page title is "Buy tokens" with the subtitle "Checks run automatically. Nothing is charged until you confirm." (line 869). The body is a two-column grid, `minmax(0,1.4fr) minmax(280px,1fr)`, gap 28.
  - The left white card (radius 14, padding 28) holds, from top to bottom:
    - The `HorizontalStepper` with Order, Review and confirm, and Status. Reached steps show a green `#1F5E4B` bar and an ink label. Other steps show `#E3DED4` and `#5E6166`.
    - "How many tokens?" (Geist 500 13).
    - The `QuantityStepper`: a `−` button (56×60, cream), a mono 28 value 120 px wide, and a `+` button.
    - Four preset pills (5, 10, 25 and 50 tokens).
    - The footer row: `Back to property` (outlineSoft) and `Review order` (primary green).
  - The right dark `OrderSummaryAside` (ink, radius 14, padding 24) holds:
    - The property name (Newsreader 24) and a "suburb · sponsor" line.
    - Five lines: Tokens, Price per token, Subtotal, "Transaction fee 0.5%" (the percentage is the quote's `fee_bps`, never a constant), and Share of trust.
    - The Total (mono 26).
    - The caption "Paid from funds available: A$3,420.18", taken from `wallet.available_cents`.
- **Additions from the product map.** The design does not draw these. They are composed from the same A05 components and tokens (§ Design references).
  - An "or enter an amount" mono `TextField` (A$) beside the presets. It sets the stepper to whole units.
  - A `TicketBoundsLine` under the presets, always shown (the product map's "Minimum and maximum subscription"): "Minimum 1 token · maximum 500 in this offer", from the quote's `min_ticket_units` and `max_ticket_units` through `formatTokens`. When G07's `max_allowed_units` is lower, the line adds "· up to {n} for you under the holder cap" from the G07 row.
  - An `OfferProgressLine` under the stepper. It is an 8 px `RaiseBar` with the soft-cap marker, captioned "64 % of hard cap · soft cap 75 % · closes 14 Oct 2026" from the quote's `raise`.
  - The note "0.5% transaction fee. Read the information memorandum before investing. Wholesale investors only.", built from the offering's `fee_bps` and `regime`. The `inv01.regimeNote` slot (`slots.ts`) renders under it; A90 fills it in P2.
  - The `DepositShortcut` Banner, shown when `wallet.shortfall_cents > 0`: "You need A$X more for this order. Deposit with PayID — it usually arrives in under a minute." It links to the deposit screen (POR-05, [[A59 - AUD wallet, token wallet and account screens]]) through A52's `href('POR-05', { return: '/invest/{code}/buy' })`, so the return path passes A52's `return` validator. Honouring `return` on POR-05 (a "Back to your order" link after the deposit) is a requirement on A59, recorded there, not built here. Whether or not A59 honours it, the quote re-runs on window focus and on `visibilitychange`, so the figures are fresh whenever the person comes back.
- **Live quote.**
  - Every change of units (stepper, preset or amount field) calls `POST /v1/offerings/{id}/orders/quote` (A37, scope `orders:quote`) with `{ requested_units }`.
  - Calls are debounced by 250 ms and carry a sequence number that only goes up. A response for an earlier sequence is thrown away.
  - While a quote is in flight, the aside keeps the last figures with a skeleton shimmer and `Review order` is disabled.
  - The first quote fires on page load with 10 units (the design's `qty: 10`).
  - The stepper's lower bound is `min_ticket_units`. Its upper bound is the smaller of `max_ticket_units` and the G07 row's `max_allowed_units`. The component clamps, but the server is authoritative. A preset above the upper bound is disabled, with the bound as its tooltip.
- **Eligibility rows ("Checks run on this order").** The design shows them on step 1. The product map wants them live on INV-01. This node renders `EligibilityRows` on both steps from the same quote `gates[]`, in the server's order. Each row shows the server's `gate`, `description` and `result`:
  - G01 "Identity verified · Reusable credential on wallet 0x7a3f…c91e".
  - G04 "Wholesale status valid · Certificate valid to 30 Jun 2028".
  - G06 "Residency and FIRB · Australian resident".
  - G07 "Holder cap · After this order you'd hold 0.10% (cap 5%)".
  - G11 "Offer caps and escrow · Within hard cap · funds held until soft cap".

  A passing row has a green Pass `StatusPill` (`#E3EEE8` / `#1F5E4B`). A failing row has a red pill (`#A3312A`) and the server's `fix_it` sentence under the description. The rows are a list with the verdict in text, not colour alone. More detail:
  - A G07 failure adds a `Use maximum (n)` link button, which sets the stepper to `max_allowed_units` and re-quotes.
  - A G01, G04 or G06 failure caused by unfinished onboarding links to the matching A56 step.
  - A G11 failure shows the bounds from the server ("Minimum 1 token · maximum 500 in this offer"). When `raise.may_be_scaled_back` is true, the row passes and shows the warning "Orders past the hard cap may be scaled back and the difference refunded".
  - Any failed row disables `Review order` (opacity .45, `aria-disabled`, no click), and an `aria-live` region announces the change.
- **Moving to step 1.** `Review order` stores the quote in the feature store: `quote_id`, `expires_at`, `lines`, `formatted`, `gates`, `documents_required`, `statements` and `raise`. It then navigates to step 1 at `/invest/{code}/buy/review?q={quote_id}`. A deep link to `/review` whose `q` has no unexpired quote in the store redirects to step 0.
- **Any other failed gate.** The rows render whatever gates the quote returns, in the server's order, with no list of gate codes in the screen. A gate this spec does not name (for example G21 licence perimeter today, G05 in P2) renders the same way: its server `description`, a red pill, its `fix_it` text, and `Review order` disabled. A hidden fixture fails G21 on the quote to prove it.

**3. INV-02 Review and confirm (step 1)**

- **Layout.** The same card and aside. The stepper shows step 1 reached. The footer reads `Back` and `Confirm and pay` (design line 906). The card shows the `EligibilityRows` again from the stored quote. If the quote's `expires_at` has passed on entry, the screen re-quotes silently. If any figure changed, a Banner "Prices refreshed" appears and the tick is cleared.
- **Documents that must be opened.**
  - `GET /v1/offerings/{id}/disclosure-set` (A35) returns the required versions, their titles, the `meta` strings, the statement text and `statement_hash`, and the composed `ack_text`. For `s708_wholesale` the versions are the information memorandum, the fee schedule and the risk statement, with a `meta` string such as "PDF · 42 pages · v2, 1 Sep 2026".
  - Each version renders as a `DocumentRow` with "Open ↗". Opening a row calls `POST /v1/documents/versions/{id}/open` (A35), which writes the `document_open` row and returns a presigned URL valid for five minutes. Because browsers block `window.open` after an awaited call, the click handler opens the new window **synchronously** first (`const w = window.open('', '_blank')`, with a "Loading document…" placeholder), and only then calls the open route, setting `w.location` when the URL arrives. If `w` is `null` (pop-ups blocked), **no open call is made**, so no `document_open` row is written for a document nobody saw; the row shows "Your browser blocked the new tab · Allow pop-ups and try again, or open it in this tab". "Open it in this tab" calls the open route and then `location.assign(url)` in the same tab; the review store and `?q=` bring the person back to step 1 with the "Opened" pill. If the open call fails, the placeholder window is closed and the row shows the error sentence. On success the row shows a green "Opened" pill with the server's open time.
  - The `AcknowledgeCard` stays disabled (opacity .45, `aria-disabled`, no toggle) until every required version shows "Opened". It counts an open in this session or an earlier open by this holder of the same version that the disclosure set reports. The disabled card's `aria-describedby` names the documents still unopened.
- **Acknowledgement and statement.**
  - The card text is A35's `ack_text`, shown verbatim. For Kent Street Terrace it reads: "I've read the information memorandum, fee schedule and risks for Kent Street Terrace, and I understand my money stays in escrow until the offer closes. (G12)".
  - Under the card, the `StatementBanner` shows the regime statement in full. For wholesale it reads "No cooling-off period applies to this offer. Your money stays in escrow until the offer closes and is refunded automatically if the soft cap is not reached."
  - The screen keeps the `statement_hash` of the text it displayed and sends it with the order, so if the screen showed an older statement, the server refuses the order.
  - Ticking the card turns the checkbox and the border green, as designed.
- **Wholesale-only copy.** While the offering's `regime` is `s708_wholesale`, the card shows the "Wholesale investors only" note. For `au_mis_retail` (P2), the statement, the cooling-off days and the PDS and TMD rows come from A35 and A90 through the same `disclosure-set` and `statements[]`. This node has no regime branch of its own.
- **Confirm and pay.** The button is disabled (opacity .45, as in the design's `buyNextOp`) until the card is ticked and every gate row passes. The whole sequence lives in `api.ts`. The button shows a spinner and stays disabled for all of it. It is built so that **exactly one keyed write** is ever sent per attempt, which keeps it consistent with A09's idempotency contract (a key replays only an identical body; a different body under the same key is refused).
  1. **Build the body once.** `quote_id`, `requested_units`, `acknowledgements: [{ document_id, version, sha256, opened_at }]` (each `opened_at` is the server's time from the open route or the disclosure set, never the browser clock, so the body is stable) and `statement_hash`.
  2. **Get the hash without writing.** `POST /v1/offerings/{id}/orders/confirmation-challenge` (upstream item 5) with that body and no `Idempotency-Key`. It returns `{ payload_sha256, purpose: 'order.confirm', summary }` and writes nothing.
  3. **Passkey.** A52's `withFreshAssertion` runs A11's nonce-mode ceremony (`fresh-assertion/options` then `verify`) over that `payload_sha256` for purpose `order.confirm` (maximum age 5 minutes), showing the server's `summary` card first. It is used in its pre-flight form (hash supplied up front) rather than its react-to-a-428 form. If A52's frozen helper offers only the reactive form, a one-line pre-flight variant is requested from A52 at Specify; this node does not edit A52's files.
  4. **One keyed placement.** `POST /v1/offerings/{id}/orders` (A37, scope `orders:place`) with the body plus `confirmation: { kind: 'passkey_assertion', assertion_id }`, under the key from `state/idempotencyKey.ts`, passed to the SDK explicitly (overriding the SDK's automatic key).
  5. **In flight.** `409 IDEMPOTENCY_IN_PROGRESS` is retried with the same key and the same body, honouring `Retry-After`, until the stored response replays (at most 70 s, just over A09's 60 s lease). The SDK does not retry this code itself, so `api.ts` does.

  **The key.** `idempotencyKey(quote_id, investor_id)` = UUIDv5 of `"{quote_id}:{investor_id}"` under a fixed RealX namespace UUID. Nothing about the key is stored: the same quote always gives the same key, whatever happens to `sessionStorage`. One key belongs to one quote, and one quote to one body. So every refusal that changes the body (a failed gate, a stale or unopened document, a statement mismatch, an expired quote, a refused or stale assertion) ends that quote: the screen re-quotes before the next attempt, and the new quote gives a new key. A second attempt therefore never reuses a key with a different body.

  If the ceremony is cancelled or fails, the button re-enables, the Banner "Confirm with your passkey to place the order" appears, and no placement call is made.

  Before step 4 the review store records the exact placement body, including `assertion_id`, as `pending_placement` (in `sessionStorage` through A52's `safeStorage`, every access in try/catch). A double click (the button is already disabled), a lost response, or a refresh while the placement is in flight all resend that identical body under the same key: A37 returns the original order (`201` once, then `200` with `Idempotent-Replayed: true`) or `409 IDEMPOTENCY_IN_PROGRESS` until it can. On reload, a `pending_placement` is resent before anything else is shown. If storage is unavailable or was cleared (a closed browser, a private window), the screen cannot resend and does not guess: `/review` returns to step 0, the next attempt needs a new quote, a new tick and a new passkey, and the `ExistingOrderBanner` shows any order the lost attempt created, so a second order is never created silently.
- **`ExistingOrderBanner`.** On entry to step 1, the screen reads `GET /v1/me/orders?open=true` and, when a row has the same `offering.code`, shows "You already have an order for this offer · {id}, placed {time}. Confirming places a second order." with a link to that order. A37 allows a top-up, so this is information, not a block. It is what makes a second order after a lost session an explicit choice.
- **How the acknowledgement is written (settled in A37).** A37's placement records the acknowledgements in the same transaction as the order, through A35, with `context_type: order`, the new order id, the version ids, `statement_hash` and `assertion_id` (A35's `document_acknowledgement` carries `assertion_id` for exactly this one-step case). The screen never calls `POST /v1/acknowledgements` before an order exists, so no acknowledgement can be left without an order.
- **Response handling.** Each code has a sentence in `errors.ts`.
  - `201`, or `200` on replay → navigate to `/invest/orders/{id}`. When `status_label` is `in_escrow`, the toast reads "Order placed · funds in escrow" (design line 835). When it is `awaiting_funds`, the toast reads "Order placed · pay A$X by {deadline}".
  - `ORD_QUOTE_EXPIRED` → re-quote and show "Prices refreshed". The tick is kept only when the disclosure set is unchanged.
  - `ORD_GATE_FAILED` with any `gate` other than G12 (G01, G04, G06, G07, G11, G21, or G05 in P2) → back to step 0 with a fresh quote and that gate's server row failed with its `fix_it` text. For example, the holder cap may have moved between quote and placement. There is no per-gate branch: a gate the screen has never seen behaves the same way.
  - `ORD_GATE_FAILED` with `gate: G12`, or `ORD_ACK_REQUIRED`, with reason codes `DISCLOSURE_VERSION_STALE`, `ACK_DOCUMENT_MISSING`, `ACK_DOCUMENT_NOT_OPENED` or `ACK_STATEMENT_MISMATCH` → the disclosure set reloads, the changed or unopened document is marked "Updated · open again" or "Open this first", the card resets, and the screen re-quotes (a new key). The marking follows the server's `reason_codes` in the response, never a client-side check, so a refusal is always the server's.
  - A37 keeps a `rejected` order record for a gate failure, with nothing reserved and no journal. The screen treats that record as a refusal, not as a placed order.
  - `ORD_CONFIRMATION_INVALID`, `FRESH_ASSERTION_REQUIRED`, `FRESH_ASSERTION_STALE` or `FRESH_ASSERTION_MISMATCH` → a silent re-quote (a new key), then the challenge and the passkey prompt run once more. A second refusal shows "We couldn't confirm with your passkey · try again".
  - `ORD_OFFERING_NOT_LIVE` → redirect to the property page with the Banner "This offer closed while you were reviewing".
  - `409 IDEMPOTENCY_IN_PROGRESS` → retried with the same key and body as in step 5; never shown unless the 70 s budget runs out ("Your order is still being placed · check your orders"), linking to POR-04.
  - `422 IDEMPOTENCY_KEY_REUSED` and `ORD_DUPLICATE_KEY_CONFLICT` (one sentence for both, whichever code and status A09 and A37 settle on, upstream item 6) → "This order changed after you confirmed · review it again". The screen re-quotes (a new key) and never retries the old key. By the key rule above this should be unreachable; a hidden case forces it.
  - A09's `RATE_LIMITED` and `AGENTS_PAUSED` → A52's shared sentences.
  - An unknown code → "Something went wrong · try again", with the request id in Geist Mono. A raw code is never shown.

**4. INV-03 Order status (step 2)**

- **Data and layout.** The page reads `GET /v1/me/orders/{id}` (A37, scope `orders:read`). Another investor's order returns `404`, which renders A52's not-found page with no hint that the order exists. The card shows:
  - The stepper at step 2.
  - A header line with the order id in Geist Mono ("RX-ORD-55120"), the `OrderStatePill` and, when present, the `AgentMarker`.
  - The `OrderTimeline`.
  - The footer `Back to property` / `View activity`, which goes to POR-04 ([[A58 - Portfolio, holding detail, transactions, distributions and statements]]).

  The aside shows the order's own lines (units, price at placement, subtotal, fee, share, total) and either "Paid from funds available" or "Awaiting A$X · pay by {deadline}".
- **Pill table.** `components/OrderStatePill.tsx` holds one table from `status_label` to text and to a tone through A05's `toneFor`. A test checks it, value for value, against A37's frozen vocabulary (upstream item 2). The screen never re-maps a state, never works out "under review" from any other field, and never renders a reason field even if one is present.

  | `status_label` | Pill text | Tone |
  |---|---|---|
  | `checking` | Checking | grey |
  | `in_escrow` | In escrow | amber |
  | `awaiting_funds` | Awaiting funds | amber |
  | `under_review` | Under review | amber |
  | `reflection` | Cooling-off (P2) | amber |
  | `allotted` | Allotted | grey |
  | `scaled_back` | Scaled back | amber |
  | `settled` | Settled | green |
  | `refund_pending` | Refund on its way | amber |
  | `refunded` | Refunded | grey |
  | `cancelled` | Cancelled | grey |
  | `revoked` | Withdrawn (P2) | grey |
  | `expired` | Expired | grey |
  | `rejected` | Not accepted | red |

  There is no "waiting for your confirmation" label. An agent order above its mandate cap, or not covered by standing consent, returns A37's `428 CONFIRMATION_REQUIRED` and leaves no order, so it has no INV-03 page. The person confirms it on ACC-05 ([[A60 - Investor and sponsor API tokens and agents screens (ACC-05, SPN-16)]]). After the passkey, A37 replays it as a real order, which then appears on INV-03 like any other agent order (briefly `checking`, then its funded or awaiting state) with the agent marker.
- **Timeline.**
  - The timeline draws the design's five nodes. A node is an 18 px circle, filled green when done and white with a `#B9B4AA` border when pending. Connectors are `#E3DED4`.
  - The nodes come from A37's six timeline keys, and titles and details are the server's strings shown verbatim:
    - `placed` → "Order placed · 22 Sep 2026, 10:41 AEST · RX-ORD-55120".
    - `funds_in_escrow` → "Funds held in escrow · A$2,395.92 moved from funds available".
    - `offer_closed` and `allocated` fold into the third node. While the offer is live it reads "Offer reaches soft cap · 64% of 75% · refunded automatically if not reached by 14 Oct 2026". After close it reads "Offer closed 14 Oct 2026", followed by the `allocated` detail, for example "Allocated 200 of 300 tokens · A$23,959.20 refunded" when the order was scaled back.
    - `units_issued` → "Units issued by trustee · Recorded on the trustee register".
    - `tokens_delivered` → "Tokens delivered to your wallet · Mirrored on-chain within minutes of issue". Once done, the transaction hash shows as a `ValueChip` with a local-chain explorer link.
  - The fold is presentation only. The Specify step checks that A06's seed produces the exact strings on dc line 834. A mismatch is a data fix raised to A37 or A06, never patched in the UI.
- **States the design does not draw.** Every status sentence below is a server string: the node titles and details come from A37's timeline entries (upstream item 4), and amounts, dates and the refund due date from `refund` and `cancel_reason` (upstream item 3). The screen holds no status sentence, no refund SLA and no business-day count of its own; the copy quoted here is the fixture text the node QA checks.
  - **Scaled back:** a Banner above the timeline, "Your order was scaled back to 200 tokens · A$23,959.20 will be refunded by {refund.due_at}", from `refund.amount_cents` and `refund.due_at`. When `refund.state` is `refunded`, it becomes "A$23,959.20 refunded · {refund.at}".
  - **Refund on its way** (`refund_pending`): the last done node, then A37's pending `refund_pending` entry, for example "Refund of A$2,395.92 on its way · by 18 Oct 2026". The due date is `refund.due_at`, which A37 computes from `orders.refund_sla_business_days`.
  - **Refunded:** A37's final `refunded` entry, from `refund.amount_cents` and `refund.at`. The J05 fixture is consistent end to end: the offer closes 14 Oct 2026 with the soft cap missed, A38 reverts it on 14 Oct, and the refund posts 16 Oct 2026, within the five business days. The third node reads "Soft cap not reached · offer closed 14 Oct 2026" and the final node "Refunded · A$2,395.92 · 16 Oct 2026".
  - **Cancelled / expired / revoked:** A37's final entry, whose wording follows `cancel_reason`: "Cancelled by you · {date}" (`investor`), "Cancelled when you revoked your agent's access · {date}" (`agent_token_revoked` or `agent_mandate_revoked`, J22 step 4), the listing-rule or offering-cancelled wording A37 writes for those reasons, "Expired · funds were not received by {deadline}", or, for a material change that was not re-confirmed, A37's `refund_pending` entry with reason `material_change_not_reconfirmed`. A token-revoked cancellation has its own fixture and screenshot.
  - **Not accepted** (`rejected`): the failed gate's title and fix-it text from the order's gate data, and no timeline.
  - **Awaiting funds:** the escrow node pending with "Pay A$X by {deadline}", the whole hours left, and the `DepositShortcut`.
  - **Under review** (`under_review`): "Your payment is being reviewed", with no reason whatever the payload holds, and no deposit shortcut. A37 maps a frozen `investor_cash` account to this label whatever the underlying state, so the screen never sees `awaiting_funds` for a frozen account.
  - **Cooling-off** (P2): the `reflection` slot from `slots.ts`, rendered above the timeline and empty in P1. A90 registers its `CoolingOffBanner` and revoke control there.
  - **Settled:** all five nodes done, with links to the holding (POR-02) and the allotment confirmation (POR-07), both A58, from `GET /v1/me/orders/{id}/confirmations`.
  - **Checking** (`checking`, for `submitted` and `eligibility_check`): "Checking" with no controls. It is seen mainly for a moment after an agent order confirmed on ACC-05 is replayed.
- **Cancel.**
  - When `cancellable` is true, the footer adds `Cancel order` (danger outline). It opens `CancelDialog`: "Cancel this order? The money in escrow is refunded to your funds available." for a funded order (no time promise in the dialog; the refund entry that follows shows A37's due date), and "Cancel this order?" before funding.
  - Confirming calls `POST /v1/orders/{id}/cancel` (scope `orders:cancel`, with an idempotency key) and the page refreshes.
  - `ORD_NOT_CANCELLABLE` renders the server's reason, for example "This listing does not allow cancellation after funding" or "The offer has closed".
  - No cancel control appears when `cancellable` is false.
- **Re-confirm after a material change.**
  - When the order carries `reconfirm_due_at`, a brick-tint `ReconfirmBanner` at the top of the card says: "This offer changed on {date}. Read the updated documents and re-confirm by {due}, or your order is refunded in full."
  - `Review changes` reuses `DisclosureList`, `AcknowledgeCard` and `StatementBanner` for the current disclosure set.
  - It then follows the same one-keyed-write sequence as "Confirm and pay": `POST /v1/orders/{id}/reconfirm/confirmation-challenge` for the hash, the passkey through `withFreshAssertion` (purpose `order.confirm`), then one `POST /v1/orders/{id}/reconfirm` with the new acknowledgement set, `statement_hash` and the confirmation, under the key UUIDv5 of `"{order_id}:reconfirm:{reconfirm_due_at}"`, so a retry of the same re-confirmation always replays.
  - Success clears the banner and shows A37's `reconfirmed` timeline entry. A37 records the new acknowledgements in its own transaction; the screen never calls `POST /v1/acknowledgements`.
- **Agent marker.**
  - When `flags` contains `agent_placed` (channel `api` or `mcp`), the header shows the `AgentMarker` chip "Placed by your agent · {token name}", linking to ACC-05 filtered to that token. The name comes only from the order read's `agent.token_name` (upstream item 3); the screen makes no second call for it.
  - Channel `auto_invest` (P3, A95) renders "Placed by Auto Invest". Channel `web` shows nothing.
  - The marker is informational. Every control on the page works for the person exactly as it would for an order they placed themselves.
- **Live updates.** The page re-reads the order every 20 s while the state is not terminal, and on window focus. Polling stops at a terminal state. The browser does not consume A46's webhooks.

**5. Parity, agents and the action manifest (D04, G22)**

- `actions.ts` lists every user action on the three screens with its SDK operation id. A unit test asserts that each id exists in `@realx/sdk`'s generated client and in `openapi.json`, so [[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates]] can prove that each operation succeeds, or is refused, identically for a person, an investor token and an MCP call ([[A104 - MCP server generated from the OpenAPI document]]).

  | Action | Operation id | Owner |
  |---|---|---|
  | Read the offering | `offerings.get` | A36 |
  | Quote | `orders.quote` | A37 |
  | Read the disclosure set | `offerings.disclosureSet` | A35 |
  | Open a document | `documents.versions.open` | A35 |
  | Place an order with acknowledgements | `orders.create` | A37 |
  | View an order | `orders.me.get` | A37 |
  | List confirmations | `orders.me.confirmations` | A37 |
  | Cancel | `orders.cancel` | A37 |
  | Get the confirmation hash | `orders.confirmationChallenge` | A37 |
  | Re-confirm | `orders.reconfirm` | A37 |
  | Get the re-confirmation hash | `orders.reconfirmChallenge` | A37 |
  | List my open orders (existing-order banner) | `orders.me.list` | A37 |

  The passkey step is marked `human_only` in the manifest, and its agent equivalent is named: A12's mandate signature (`confirmation.kind = mandate_signature`) within the caps, or, above the per-action cap, the `428 CONFIRMATION_REQUIRED` path, which leaves no order and which A60 renders on ACC-05 for the person's passkey; past the daily or monthly cap the order is refused `403 G22_LIMIT_EXCEEDED` with no confirmation offered (A12 §5 cap table).
- **Nothing the SDK cannot call.** The screens make no call that a token cannot make. There is no app-only route, no Next.js server action that bypasses the API, and no cookie-only endpoint. A test walks the feature for `"use server"` and for route handlers and finds none.
- **Rules are the same on every channel.** An agent's order passes the same gates in A37: G01, G04, G06, G07, G11, G21 and G12 under A35's standing consent (A35 enforces CQ-02: the first purchase in a vehicle needs the person), plus G22 in A12's middleware. The screen adds no rule an agent would escape and relaxes no rule a person must meet. An agent's order shows up on INV-03 exactly as the person's does, plus the marker.

**6. Empty, error and edge states (each has a test and a screenshot)**

- **First paint, before any quote:** the aside shows "—" placeholders under the property name, and the button is disabled.
- **Quote fails on the network:** the Banner "Couldn't check this order · retry" with a retry button. The last good figures stay.
- **Offer closes between steps:** redirect with the Banner from §3.
- **Funds short:** the deposit shortcut shows, and confirm is still allowed, because the order goes to `awaiting_funds`.
- **Frozen account:** the quote renders its gates as the server sent them, and the placed order shows "Under review" only.
- **Unknown order, or another investor's:** the not-found page.
- **Session expires mid-flow:** A52's sign-in with a return path to the same step (`/review?q=`). The review store survives in `sessionStorage` (try/catch) and is thrown away once it is older than the quote's `expires_at`, except a `pending_placement`, which is resent first (§3) and then cleared.
- **Storage unavailable** (private window, blocked site data): the flow still works within one page load. A refresh on `/review` returns to step 0, and the `ExistingOrderBanner` shows any order already placed.
- **An investor with no orders who opens `/invest/orders`:** redirect to POR-04's empty state.
- **Phone width (375 px):** the two columns stack, with the aside below the card. The stepper fits 343 px. Gate rows, document rows and the timeline wrap with no horizontal scroll. The 16 px gutter holds, and the presets wrap to two rows.
- **Accessibility:**
  - The stepper buttons have labels ("Fewer tokens", "More tokens"). The value is an `input type="number"` with `aria-valuemin` and `aria-valuemax`.
  - A disabled primary button explains why through `aria-describedby`.
  - Focus moves to the card heading on each step change.
  - axe-core over the three routes in every state finds zero critical or serious issues.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\web\app\(investor)\invest\
- <workspace>\app\apps\web\src\features\buy\

Inside the feature, by convention: `api.ts`, `actions.ts`, `errors.ts`, `slots.ts`, `state/`, `components/`, `__tests__/`, `e2e/`. The routes add `page.tsx` files and sub-folders under the invest folder, beside A52's frozen shell files, and never edit them.

**Reserved sub-folders.** A52's route table places INV-05 `/invest/[code]/sell` and INV-06 `/invest/[code]/board` (P2) inside this node's `(investor)/invest/` folder. A57 creates nothing in `invest/[code]/sell/` or `invest/[code]/board/` and leaves them reserved for A92. A92 proposes moving both routes to `(investor)/resale/` through loop A52.1 with 308 redirects; if that loop is not taken, A92 may write only those two sub-folders, coordinated at A92's Specify. No A57 code links or redirects to either route in P1.

**Exports other nodes may import:** `registerOrderSlot` and `OrderSlot` from `@/features/buy/slots` (A90), and nothing else.

Changes this node needs elsewhere are recorded, never edited: the six upstream items on A37 (and A09 for item 6) before Specify, a pre-flight form of `withFreshAssertion` and the Playwright glob on A52 if missing, and the `return` parameter on A59.

## Design references

- **The `buy` screen.** `<workspace>\design\RealX App.dc.html`, screen `buy` (header code "INV-02 · Buy", title "Buy tokens", subtitle "Checks run automatically. Nothing is charged until you confirm.", line 869), template lines 399-459:
  - The two-column grid and card (400-401).
  - The `HorizontalStepper` (403).
  - Step 0: "How many tokens?", the `QuantityStepper` and the presets (405-415).
  - Step 1: "Checks run on this order", with five `GateRow`s in a `52px 1fr auto` grid (416-427), and the `AcknowledgeCard` (428-431).
  - Step 2: the `Timeline`, with 18 px nodes (434-443).
  - The footer Back / Next row (444-447, primary button at 446).
  - The dark `OrderSummaryAside` (449-457, "Paid from funds available" at 456).
- **Derivations.**
  - `orderLines` (line 830): "Transaction fee 0.5%", and Share of trust = qty ÷ 100 %, meaning 10,000 units per trust.
  - `buySteps` colours (831).
  - `buyGates` texts, including the G07 post-order percentage (833).
  - `orderTimeline` titles and details for RX-ORD-55120 (834).
  - `buyNext` and the toast "Order placed · funds in escrow" (835).
  - `buyClick` branching from the property page (837).
  - Lines 904-906: presets 5 / 10 / 25 / 50; labels "Review order", "Confirm and pay" and "View activity"; "Back to property" and "Back"; `buyNextOp` .45 while unacknowledged.
  - The property aside note (design analysis §3.1 `prop`) is repeated on the buy card.
- **Design analysis.**
  - §2 items 24 (`HorizontalStepper`), 25 (`QuantityStepper`: bounds 1..500 in the prototype, server bounds in the product), 26 (`GateRow`), 27 (`AcknowledgeCard`), 28 (`Timeline`), 29 (`OrderSummaryAside`) and 15 (`DocumentRow`).
  - §4 acceptance items 6-9: gate rows and the G07 maximum; G12 before confirm, plus the documents-opened and no-cooling-off additions; order arithmetic to the cent; timeline order and the refund path.
  - §6.1: the design's "INV-02 · Buy" is product map INV-01 + INV-02 + INV-03, routed per step. The design's investor codes differ from the product map. For example, design "INV-01" is portfolio = POR-01. `defineScreen` uses product-map codes only.
  - §6.2: on the buy review and order status, the product map wins on the two additions and the extra states.
- **Tokens from A05 (`packages/ui`).**
  - Colours: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, red `#A3312A` for failed pills, `#B9B4AA` for pending nodes and aside labels, `#2E3135` for aside dividers, `#E3DED4` for connectors and inactive step bars, `#ECE8E0` for row dividers, `#FBFAF7` for the acknowledgement card, `#E3EEE8` for Pass pill fills.
  - Type: Newsreader 24 for the aside title; Geist 500 13 for labels; Geist Mono 28, 26, 14, 12 and 11 for the quantity, total, lines, captions and gate ids.
  - Radii: 14, 12, 10, 999 and 50%.
- **Undesigned states: a design-extension question.** [[A50 - Design extension A- investor and sponsor screens not in the prototype]] draws undesigned screens, not states of designed ones, and its brief does not include the buy flow's extra states. These are listed in `features/buy/README.md` as compositions of existing A05 blocks in the tokens above, with no new component, colour or font:
  - The amount field, the ticket-bounds line, the offer-progress line and the deposit shortcut.
  - A failed gate row with fix-it text and "Use maximum".
  - The "Opened" pill on document rows, the pop-up-blocked row, the statement banner and the existing-order banner.
  - The checking, scaled-back, refund-on-its-way, refunded, cancelled (by you and by token revocation), expired, revoked, not-accepted, awaiting-funds, under-review, cooling-off and settled variants.
  - The re-confirm banner, the cancel dialog and the agent marker.

  They follow A50's conventions: blocks reused by name, product-map codes in the "view as" readout, and no reason text on any investor screen. They are shown to the founder in the A61 preview gallery. Any state the founder wants redrawn is A50's loop A50.2 (founder redraws after a preview), not a change made here.

## Requirements covered

**Screens**

- **INV-01 Buy tokens.** Done when the route renders the design's step 0 with the stepper, presets, amount field, the always-visible minimum and maximum subscription line from the server, live gate rows, offer progress, deposit shortcut and the dark aside. The aside's lines and total must equal the server quote to the cent:
  - KST01, 10 × A$238.40 = A$2,384.00, fee A$11.92, total A$2,395.92, share 0.10%.
  - WLG01, 100 × A$250.00 = A$25,000.00, fee A$125.00, total A$25,125.00.

  A failed gate disables "Review order" and names the fix. A G07 breach offers the maximum in one click. The page never shows a figure the server did not send.
- **INV-02 Review and confirm.** Done when every required disclosure must be opened through the API before the card can be ticked. The card text and the wholesale "no cooling-off" statement must come from A35. "Confirm and pay" stays inert until the card is ticked and every gate passes. A document is recorded as opened only when a window actually showed it. The confirm must be one idempotent placement: a non-mutating challenge for the hash, the passkey step-up, then acknowledgements, statement hash and confirmation in exactly one keyed A37 write whose key is derived from the quote. Every error code in A37's and A35's catalogues that this route can receive must have a sentence.
- **INV-03 Order status.** Done when:
  - The six timeline keys fold onto the design's five nodes, using the seed's exact strings for RX-ORD-55120.
  - Every SM-05 state and `refund_state` has a pill, a timeline shape and a screenshot.
  - The refund path shows the amount, the due date and the posted date, all from A37.
  - Cancelled, expired, revoked and refund entries use A37's timeline strings and `cancel_reason`, including a token-revoked cancellation.
  - Cancel appears only when the server says `cancellable`.
  - The re-confirm banner drives the reconfirm route.
  - The agent marker appears on agent-placed orders.
  - "Under review" never carries a reason.

**Journeys**

- **J04 First investment in a primary offer.** Done when step 3, step 4 and step 13 run end to end in Playwright against the Docker stack, with A37, A38, A35 and A11 in sandbox mode, and the five-node timeline ends fully green.
  - Step 3 (INV-01): A$25,000 → 100 WLG01 tokens, fee A$125, total A$25,125, four gates live.
  - Step 4 (INV-02): acknowledge the IM and fees, confirm with the passkey, order `awaiting_funds` because the wallet is short, no cooling-off.
  - Step 13 (INV-03): `settled`, tokens in the wallet, allotment confirmation linked.
- **J05 Offer misses its soft cap: refund.** Done when step 5 passes in Playwright: INV-03 shows "Refunded" with the amount and date after A38's revert-and-refund run, and the soft-cap node reads "Soft cap not reached". The investor's funds available must have risen by the same amount, checked through the ancestor SDK read `GET /v1/me/wallet` (A16) before and after, not through A59's screen. Links from INV-03 to POR-02, POR-04, POR-05, POR-07 (A58, A59) and ACC-05 (A60) are asserted only against A52's route table, since those screens are same-milestone nodes A57 does not depend on.

**Gates**

- **G21 and any other gate in A37's placement set.** Done when a failure renders its server row and fix-it text, disables progress, and returns a refused placement to step 0, with no per-gate code in the screen.
- **G22 Agent authority.** Done when an agent order that breaks a hard limit leaves no order and nothing on INV-03, an above-cap order leaves no order until the person confirms on ACC-05, and every agent order on INV-03 shows its marker (§5).
- **G01, G04, G06.** Done when each is a gate row on INV-01 and INV-02 fed only by the quote, and a failure disables progress, names the fix and links to the right onboarding step. A forced placement with a failed row must be refused by the server with `ORD_GATE_FAILED` and rendered, with no reservation and no journal.
- **G07 Holder cap.** Done when the row shows the server's post-order percentage and cap, a breach names `max_allowed_units` and offers it in one click, and the stepper cannot go above it. A forced order above it must be refused and rendered.
- **G11 Offer caps and escrow.** Done when the row shows "Within hard cap · funds held until soft cap", or the bounds, or the scale-back warning, all from the quote. The offer-progress line must show the amount raised, the soft cap and the close date. A closed or suspended offer cannot be quoted or placed from the screen, and the escrow node on INV-03 shows the amount held.
- **G12 Disclosure acknowledged.** Done when:
  - The card cannot be ticked until every required version has an open row.
  - The card text and statement are A35's.
  - The placement carries the versions, hashes, open times and `statement_hash`.
  - A superseded version resets the card and marks the document.
  - The recorded acknowledgement exists in A35, checked through the ancestor SDK read `GET /v1/acknowledgements?context=order:{id}` (A35) with the versions, hashes, open times and `statement_hash` placed. Its display on POR-07 is A58's to prove.
  - A forged `opened_at` for a document never opened, sent straight through the SDK, is refused by the server.

**Modules and state machines**

- **M05 Subscriptions and orders.** Done for its investor-facing surface when quote, place, status, cancel and re-confirm run through the public SDK with parity, and the state vocabulary comes from A37's `status_label` without re-mapping in the screen.
- **SM-05 Order.** Done when all 13 states and `refund_state` are handled:
  - Each state has a rendered representation (pill, timeline and copy), including `eligibility_check` as the one-transaction "Checking" fallback.
  - Each state has a screenshot in the evidence.
  - The screens never attempt a transition the server does not offer: `cancellable` and `reconfirm_due_at` drive the controls.
  - The state → label → tone table is one file, tested against A37's vocabulary.

**Founder requirement D04**

- Done when the action manifest's operations are all in the SDK and OpenAPI, the screens make no call a token cannot make, and the only human-only step is the passkey confirmation, with its agent equivalent named. The same purchase placed by an investor's agent must show on INV-03 with the marker, appear in the person's activity as "agent <prefix> on behalf of <person>", trigger the person's notice where the threshold says so, and the person must be able to act on it.

## Gauntlet loop for this node

- **Specify.** The frozen spec is this note plus:
  - A37's contracts for quote, confirmation challenge, place, cancel, reconfirm, `me/orders`, `me/orders/{id}` and confirmations: field names, the `status_label` vocabulary and mapping rules, timeline keys and entries, `cancel_reason`, `refund` and error codes. Specify checks the six upstream items one by one against A37's frozen spec (and A09's for item 6). Any item missing stops the Specify step and is reopened on A37 or A09 before the node QA writes a hidden test.
  - A09's idempotency contract (claim, replay, `IDEMPOTENCY_IN_PROGRESS`, `IDEMPOTENCY_KEY_REUSED`).
  - A12's `/me/activity` read and A43's `/me/notifications` read (fed by A101), for the parity challenge.
  - A35's `disclosure-set`, `open` and acknowledgement contracts, with `ack_text` and `statement_hash`.
  - A11's fresh-assertion ceremony for purpose `order.confirm`.
  - A52's investor shell, `defineScreen`, `withFreshAssertion`, error table, session and sign-in return path.
  - A05's component props.
  - A06's seed for KST01, WLG01 and RX-ORD-55120.

  The node QA records `specVersion` and the hashes of those contracts in the manifest. The node QA then writes the hidden material to `<workspace>\gauntlet\hidden\A57\`:
  - The planted mock server: a wrong fee, failed gates (including G21), out-of-order quotes, a stale version, a `freeze_reason` field on an under-review order, a `cancellable: false` order, a token-revoked cancellation, an idempotency key reused with a different body, and a lease that answers `409 IDEMPOTENCY_IN_PROGRESS` twice before replaying. The mock is generated from A37's and A09's frozen OpenAPI schemas and refuses any field those schemas lack, so it cannot be more lenient than the real service.
  - The reference crops at 1280 px and 375 px.
  - The parity diff script.

  The Specify step also confirms that A06's seed gives RX-ORD-55120's timeline strings equal to design line 834, and that KST01's quote for 10 units returns A$2,395.92.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: the lead first writes `api.ts`, the feature store, `slots.ts` and `actions.ts`, then three parallel sub-agents work on disjoint component sets, then one integration pass.
  1. INV-01: routes, entry guard, stepper, presets, amount field, quote sequencing, gate rows, offer progress, deposit shortcut, aside.
  2. INV-02: the disclosure list with opens, the acknowledgement card and statement banner, the existing-order banner, the pop-up-safe document open, the confirm sequence (challenge, `withFreshAssertion`, one keyed placement, in-progress retries) with the quote-derived key and `pending_placement`, and the error sentences.
  3. INV-03: the order read, the pill table, the timeline fold, every extra state from A37's timeline entries, the `reflection` slot, cancel, re-confirm, the agent marker, polling.

  The integration pass runs the Playwright journeys J04 (steps 3, 4 and 13) and J05 (step 5) against the Docker stack, the screenshot gallery for every state at both widths, the manifest test, axe-core and `npm run verify`, then writes `app/evidence/A57.json`.
- **Challenge (node QA, Opus 5.5 high).**
  - Runs the hidden mock-server cases.
  - Tries forced submissions through the DOM and through the SDK, and greps `features/buy/` for `eslint-disable`.
  - Runs the document opens with pop-ups blocked.
  - Renders the state gallery against the dc crops and the README list of compositions.
  - Diffs the seed strings field by field.
  - Diffs the parity manifest against `openapi.json` and the SDK.
  - Has a person, an investor token and an MCP call perform the same purchase.
  - Tests both widths and keyboard-only completion.
  - Checks for SDK imports outside `api.ts`, and that only the owned paths changed.

  The node QA writes `app/evidence/A57.qa.md`.
- **Gate.** A pass unlocks A61, A90 and A95 and publishes the hidden cases into `app/`.
- **Fix loop.** At most three. Each loop re-runs the hidden cases, the state gallery and the two journeys.
- **Escalation trigger.**
  - The same assumption fails three times.
  - A fix proposes computing money, share or eligibility in the browser, hard-coding a legal sentence, enabling the acknowledgement without open rows, showing a reason for an under-review order, or adding an app-only route (Critical: stop and reopen).
  - A field or route the screens need (the six upstream items, or A35's) is missing from the frozen contract at Specify or turns out missing during the build (a spec defect upstream, reopened there; never worked around in the screen or the mock).
  - The design and the product map disagree on a state and neither the design analysis nor the product map settles it (ask the user).

## Node-specific challenges (node QA)

- **Figures from the wrong place (money).** The hidden mock returns a fee of A$11.93 for 10 × A$238.40. The aside must show A$11.93 and a total of A$2,395.93, because that is what the server said. The challenge fails if A$11.92 appears anywhere, because that would be a fee computed locally. Then 40 stepper clicks arrive in 300 ms, with responses returned out of order. The aside must end on the figures for the final quantity, every late response must have been discarded, and the quote stored for review must be the last one.
- **Confirm through the back door (compliance, G07 / G12).**
  - With G07 failed on the quote, the tester removes `disabled` from both buttons in the DOM and submits. The server refuses with `ORD_GATE_FAILED`. The UI shows the failed row with its fix-it text on step 0. `GET /v1/me/orders` shows no order in escrow or awaiting funds, and no journal exists.
  - With the IM unopened, the tester forces the checkbox and submits. The G12 refusal with `ACK_DOCUMENT_NOT_OPENED` marks the IM row, and nothing is reserved.
  - **Server-side, not DOM.** Straight through the SDK, with no browser, the tester places an order whose acknowledgements carry a fabricated `opened_at` for an IM version this investor never opened (no `document_open` row). The server must refuse it with `ACK_DOCUMENT_NOT_OPENED`, with no reservation, no journal and no live order. The same forged body replayed through the screen's `api.ts` (by stubbing the store) must render the refusal from the server's response: the test asserts that the network response carried the code, so a client-side guard that never sent the request fails the case.
  - A35 publishes IM v3 between the open and the confirm. `DISCLOSURE_VERSION_STALE` marks the IM "Updated · open again" and resets the card. Only after the new open and a new tick does the order succeed, with v3 in its acknowledgements.
  - **Pop-ups blocked.** With the browser context's pop-ups blocked, clicking "Open ↗" must make no open call and write no `document_open` row (checked in A35's read), must show the "Allow pop-ups … or open it in this tab" row, and must leave the card disabled. "Open it in this tab" then records the open and returns to step 1 with the pill.
- **Twice is once (money, idempotency).** Against the real A37 and A09 in Docker, not a mock:
  - "Confirm and pay" is double-clicked, and the tab is refreshed while the placement is in flight. Exactly one order exists under one idempotency key; every response was `201`, `200` with `Idempotent-Replayed: true`, or `409 IDEMPOTENCY_IN_PROGRESS` followed by a replay; the status page shows the order each time.
  - With the network cut after the placement is sent, the retry sends a byte-identical body under the same key and gets the replay.
  - The browser is closed and reopened on `/review?q=…` within the quote's lifetime (so `sessionStorage` is gone). The screen must return to step 0 and show the `ExistingOrderBanner` naming the first order. Either nothing more happens, or a second order exists only after a new quote, a new tick and a new passkey, under a different key; there is never a second order from the same quote.
  - Funds available (A16's `GET /v1/me/wallet`) falls by exactly A$2,395.92 per order that exists, and once for the first.
  - A key reused with a different body (forced by editing the stored body) gets the one "changed after you confirmed" sentence and a re-quote, and creates nothing.
- **Stale and borrowed assertions (security).** Four assertions are presented: one captured from a previous order, one older than five minutes, one for a different order hash, and one from a cancelled passkey ceremony.
  - The first three are refused (`ORD_CONFIRMATION_INVALID` or `FRESH_ASSERTION_*`), re-prompt once, and create no order.
  - The fourth makes no call.

  A session from another investor opening `/invest/orders/RX-ORD-55120` gets the not-found page, with no hint that the order exists.
- **Nothing to tell (compliance, tipping-off).** The hidden mock returns an order with `status_label: under_review` and an extra field, `freeze_reason: "AML-118"`. The page must show "Under review" and "Your payment is being reviewed" and nothing else. The string "AML" must not appear in the DOM, the accessibility tree, the title or any console output. Polling must continue without surfacing it, and the deposit shortcut must not appear.
- **Every state looks like the design (design fidelity).** Fixtures cover checking, in escrow, awaiting funds, allotted, scaled back (200 of 300, A$23,959.20 refunded), refund on its way, settled, refunded (soft cap missed: offer closed 14 Oct 2026, refunded A$2,395.92 · 16 Oct 2026), cancelled by you, cancelled by token revocation, expired, revoked (P2), not accepted, under review, cooling-off (P2 slot, with A90's dummy registration), agent-placed and re-confirm required. No fixture shows a status sentence that the fixture's A37 payload did not carry. Each is rendered at 1280 px and 375 px and compared by layout tokens with the dc crops (lines 399-459). Fonts, colours, radii, spacing and the .45 opacity of the inert confirm button must match. A hex value outside A05's tokens, a new component, or a state without a screenshot fails.
- **Agent parity of the whole flow (D04, G22).** An investor holds a token with `orders:quote`, `orders:place`, `orders:read`, `orders:cancel` and document scopes, a mandate signing key, and a standing consent (A35) covering KST01. The investor already holds KST01 units, so CQ-02's first-purchase rule does not apply. The agent runs the manifest operations for a second purchase in KST01, then an MCP call does the same. The quote figures, gate rows, refusals and the resulting order (flagged `agent_placed`) must match what the screen produced for the person. The status page must show "Placed by your agent · Portfolio bot", and the person's cancel from the screen must work on the agent's order. Then:
  - **Audit and notice (J22 steps 2 and 3).** A12's `GET /v1/me/activity` has a row for the agent's order reading "agent `<token prefix>` on behalf of `<person>`", and, when the order is above the person's notice threshold, `GET /v1/me/notifications` (A43, fed by A101) has the agent notice linking to ACC-05.
  - **Above the per-action cap.** An agent order above the mandate cap returns `428 CONFIRMATION_REQUIRED`. `GET /v1/me/orders` is unchanged (no order, no reservation), and A60's confirmation list on ACC-05 (read through A12's SDK operation) holds the pending confirmation. After the person confirms with a passkey, the replayed order appears on INV-03 with the agent marker, passing through `checking`.
  - **Daily cap exhausted.** An agent order that exceeds the remaining daily cap is refused with `403 G22_LIMIT_EXCEEDED`: no order row, no reservation, funds available unchanged, and nothing new on INV-03 or in `GET /v1/me/orders`.
  - **Revocation.** Revoking the token cancels the agent's awaiting-funds order in the same transaction; INV-03 shows A37's "Cancelled when you revoked your agent's access" entry.

## Pass criteria

- **Hidden cases.** The cases in `<workspace>\gauntlet\hidden\A57\` pass 100% on three consecutive runs:
  - Every planted server figure is rendered as sent.
  - No locally computed money figure appears across 200 random quantities.
  - 100 of 100 out-of-order quote races end on the last quantity's figures.
  - Every forced submission leaves exactly the expected count of live orders (0 or 1) and renders the sentence for its code, taken from the server's response. The forced submissions are: a failed gate (including G21), an unopened document (through the DOM and through the SDK with a forged `opened_at`), a stale version, a mismatched statement, an old or borrowed assertion, a second click, a refresh in flight, a reused key with a different body, and an in-progress lease.
  - No placement is ever sent twice under one key with different bodies (asserted on the recorded network log).
- **State coverage.** All 13 SM-05 states, the `refund_state` values and the agent-placed, token-revoked-cancellation, re-confirm-required, awaiting-funds and under-review variants render from fixtures, each with a pill, a timeline and a screenshot at 1280 px and 375 px in `app/evidence/A57/`. The pill table matches A37's frozen `status_label` vocabulary (upstream item 2) in a test, value for value, with no extra or missing label. In the under-review fixture, "AML" and any reason field never reach the DOM. No status sentence, refund SLA or business-day count exists as a literal in `features/buy/` (grep).
- **Slots.** `registerOrderSlot` renders a dummy component for `reflection` and `inv01.regimeNote` with the order and `refetch`, and an unregistered slot renders nothing.
- **Journeys.** J04 steps 3, 4 and 13 and J05 step 5 pass in Playwright against the Docker stack (A23) three runs in a row. The seed order's timeline strings equal design line 834, and the settled order has all five nodes done with the transaction hash shown. Balances and acknowledgements are checked through ancestor SDK reads (A16's `/me/wallet`, A35's acknowledgements), never through A58's or A59's screens.
- **Gate enforcement.** For each of G01, G04, G06, G07, G11, G21 and G12, a test fails the gate and proves three things: the button is inert, the fix-it text shows, and a forced call is refused by the server with nothing reserved. "Use maximum" sets the stepper to `max_allowed_units`, and the next quote passes.
- **Design fidelity.**
  - Layout-token comparison of the three routes in their default state against the dc crops shows zero token differences (fonts, colours, radii, key spacings).
  - The composed states use only A05 blocks and tokens.
  - axe-core reports zero critical or serious findings on every route and state.
  - Keyboard-only completion of INV-01 → INV-03 succeeds in Playwright.
- **Parity.**
  - Every operation in `actions.ts` exists in `@realx/sdk` and `openapi.json`, and the passkey step is the only `human_only` entry.
  - Each operation succeeds, or is refused, identically for a person, an investor token and an MCP call in A100's harness. It runs here with a preview script when A100 is not built yet, and A100 re-runs it later.
  - There is no server action, no route handler, and no SDK import outside `api.ts` (lint).
- **Coverage and hygiene.**
  - Unit line coverage for `features/buy/` is at least 85%. The state reducer, quote sequencing, `idempotencyKey.ts`, `unitsForAmount.ts`, `slots.ts`, error table, pill table and timeline fold are at 100%.
  - `npm run verify` is green, and `features/buy/` has no `eslint-disable` comment.
  - Only the two owned paths changed, and nothing was written in the reserved `invest/[code]/sell/` or `invest/[code]/board/` sub-folders. Any upstream change (A37, A09, A52, A59) was made by its owner, with sign-off in the evidence.
- **Evidence.** The manifest `app/evidence/A57.json` and `A57.qa.md` are present and match disk. They list the screenshot set, the seed-string check, the parity result and the contract hashes compared.

## Human gates touched

- **None crossed.** The flow runs on A11's software authenticator, A37's sandbox money, A17's sandbox bank rail and A38's local-chain settlement. It never touches a real investor, a real trust account or mainnet. HG-CAPITAL, HG-VENDOR and HG-PROD gate any use outside the sandbox.
- **HG-LEGAL (counsel questions recorded by A35, A37 and A90, not decided here).**
  - The exact wholesale "no cooling-off" statement text (A35's `cooling_off_statement`, marked `set_with_counsel`).
  - Whether retail orders carry a cooling-off period, and how long (A90).
  - Whether an agent may acknowledge a disclosure under standing consent (CQ-02, A35's parameter).
  - Per D04, whether any purchase declaration must be personal. If counsel says yes, the person's confirm step is already a personal passkey act and the agent path already stops at `428 CONFIRMATION_REQUIRED`, so no code change is expected either way.
- **HG-REVIEW (later).** The founder signs off the visual QA of the three screens and the composed states at [[A61 - Docker Desktop preview of the investor app (AM5 exit)]].

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
