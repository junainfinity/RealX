---
title: "A92 - P2: resale board screens (sell, buy on the board, market tracker, board control)"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am9]
node: A92
milestone: AM9
lane: "P2"
oracle: no
human_gate: none
---

# A92 - P2: resale board screens (sell, buy on the board, market tracker, board control)

**The four screens of the guarded resale board, drawn over A91's market module and nothing else. INV-05 Sell tokens is built as designed: the tokens you hold split into unlocked and locked, the latest valuation with the allowed price band (G09), a quantity and a limit price, the proceeds after the fee from the server, a preview of the order book with the floor marked, and the four checks (lock-in G08, trading halt G10, price band G09, board status). Listing an ask freezes exactly the listed quantity in the wallet, and cancelling it releases the rest. While the `resale.board` flag is off (the P1 launch state, and the AM9 state until a licence or relief record binds the flag) the design's brick preview banner stays, and no order is created anywhere. INV-06 Buy on the board fills from the cheapest ask upwards, with the same eligibility rows as a primary purchase (G01, G04, G05 for retail, G06, G07 with the cap-limited quantity, G09, G10). The part that does not fill may rest as a bid inside the band. INV-07 Market tracker is a public page showing last price against valuation, the premium or discount, 30-day volume and the halt history. ADM-21 Resale board control lists the instruments with their venue mode bound to a licence (G21), band, floor, lock-in, state, last price and move against valuation. It has halts and circuit breakers, surveillance alerts that open AML cases, the trade blotter with settlement state, and "Halt all" as an act-then-ratify (`SGL+R`) action. Every figure, verdict and state comes from A91, A48, A21 and A14 through the public SDK. The screens decide nothing, and a person's own agent can do everything the investor screens do, under the same gates.**

**Build status:** not started. **Milestone:** AM9 (P2 features, from about 8 April 2027). **Lane:** P2. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours (build, challenge and verify loops).

**Depends on:** [[A91 - P2- resale board market module (SM-09)]], [[A52 - Web app shells, routing and passkey sign-in (public, investor, sponsor)]], [[A53 - Operator console shell, hardware-key sign-in and approval action pattern]], [[A50 - Design extension A- investor and sponsor screens not in the prototype]], [[A51 - Design extension B- operator console screens not in the prototype]], [[A58 - Portfolio, holding detail, transactions, distributions and statements]]
**Also needs (graph edges requested, not yet in `final-graph.json`):** the spec uses these nodes, and none of them breaks milestone order. The build must not start until each has passed: [[A06 - Seed data and scenario simulator]] (the P2 seed with Bay Road, Marion Street and PAD01), [[A89 - Docker Desktop launch (P1)]] (the Docker stack J07 runs against; [[A23 - Whole-stack Docker boot (AM1 exit)]] at minimum), [[A104 - MCP server generated from the OpenAPI document]], [[A42 - AML screening, monitoring, cases and SMR clocks (SM-11)]] (the case "Open surveillance case" creates), [[A54 - Public site- home, investments, property page, sponsor profile, fees]] (loop A54.1), [[A57 - Buy flow- order, review and confirm, order status (INV-01 to INV-03)]] (patterns and the INV-01 redirect), [[A60 - Investor and sponsor API tokens and agents screens (ACC-05, SPN-16)]] (pending agent confirmations), [[A90 - P2- registered scheme, sub-funds, PDS and TMD, target market (G05), retail cooling-off]] (G05 and the PDS/TMD documents on INV-06) and [[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates]] (the binding parity check).
**Unlocks:** [[A94 - P2- end-to-end journeys and launch update]]

**Oracle rule:** this is not an oracle node (`oracle: no`). No money, supply or holdings move here: A91 matches, freezes, settles and registers, and A91 carries the oracle (asks outside the band rejected with the allowed range, halts that fire and lift on time, nothing registered without the registry acknowledgement). But this is where a person lists tokens for sale and commits money to a secondary purchase, and where staff halt a market. So the node QA still writes hidden tests from this frozen spec to `<workspace>\gauntlet\hidden\A92\` before the build starts ([[App Gauntlet]] §2). They cover four things:
- A scripted mock of A91's, A48's, A21's, A12's and A14's routes, exactly as A91 §4 and §9 define them (after loop A91.1). It returns planted wrong figures, out-of-order quote and preview reads, a book that moves between quote and order (so the order fills less than quoted and the rest rests or is cancelled), a halt that starts mid-flow, a reason field on a cancelled or frozen order, joint and entity accounts, every cancellation reason and every refusal code in A91's catalogue. It proves the screens trust only the server.
- The preview-mode probe. With the flag off, no call to an operation that `openapi.json` marks as mutating may leave the browser, whatever is forced in the DOM.
- The reference crops of the prototype's `sell` region and of A50's and A51's drawings of INV-06, INV-07 and ADM-21, at 1280 px and 375 px (console at 1440 px).
- The parity checker, which diffs this node's two action manifests against the SDK and `openapi.json`.

The builder never reads that folder. The cases are published into the repo only after the gate passes.

This note is not legal advice. Whether RealX may run a bulletin-board resale facility at all, and under which licence or relief, is decided by counsel and recorded by A21 as a licence record. So are the fee on resale sells, whether a secondary buyer must acknowledge the PDS and TMD, whether an agent may place resale orders under the holder's standing consent, and whether a public market tracker for a wholesale-only trust is permitted. This node renders whatever text, parameters and verdicts those modules return. It never hard-codes a legal sentence, a band, a fee or a lock-in.

## Build brief

**1. Scope, placement and routes**

- **Investor app routes.** The investor routes run in `apps/web/app/(investor)/resale/` under A52's investor shell. That shell supplies the header, the footer with the CAR/AFSL disclosure, the passkey session and the `previewMode` switch. A visitor who opens a sell or buy route is sent to `/sign-in` with a return path.
  - `/resale/[code]/sell` → **INV-05 Sell tokens**.
  - `/resale/[code]/sell/orders/[id]` → status of one of the person's asks (part of INV-05).
  - `/resale/[code]/buy` → **INV-06 Buy on the resale board**.
  - `/resale/[code]/buy/orders/[id]` → status of one of the person's buys or resting bids (part of INV-06).
  - **INV-07 Market tracker** is built in full as `MarketTrackerScreen`, exported from `src/features/resale/market-tracker/`, and served at the public URL `/investments/[code]/market`.
- **Specify point (a): routes.** A52's frozen route table lists INV-05 at `/invest/[code]/sell` and INV-06 at `/invest/[code]/board`. Those folders sit in A57's owned `(investor)/invest/` path. INV-07 sits at `/investments/[code]/market`, which is A54's owned `(public)/` path. This node's owned paths are `(investor)/resale/` and its feature folder. The proposed settlement has two small loops, and this node never writes into A57's or A54's folders:
  - **Loop A52.1.** A52 swaps `path` and `mapRoute` for INV-05 and INV-06. The paths become the product map's own routes, `/resale/[code]/sell` and `/resale/[code]/buy`. A52 registers permanent 308 redirects from `/invest/[code]/sell` and `/invest/[code]/board`, so A58's Sell link and A54's "Sell tokens (resale · 2027)" button keep working unchanged. A52 also sets INV-06 and INV-07 to `whenOff: 'preview'`, because A50 draws both with the PreviewBanner, instead of `'hidden'`.
  - **Loop A54.1.** A54 adds one file, `app/(public)/investments/[code]/market/page.tsx`. It calls `defineScreen({ map: 'INV-07', design: 'design-ext:INV-07', title: 'Market tracker' })` and renders `MarketTrackerScreen`.

  If the user prefers instead to add the public path to A92's owned list, the graph changes and neither loop is needed. The user decides at Specify, before the node QA writes the hidden tests.
- **Console routes.** The console routes run in `apps/console/app/(market)/` under A53's shell. Every page exports `screen = 'ADM-21'` and `designKey = 'design-ext:ADM-21'`. The shell's nav row "Resale board (P2)" sits in the Markets (P2) group with the brick "2027" pill. The pages are:
  - `/market`: instruments.
  - `/market/[code]`: one instrument, with venue mode, band, halts, circuit breaker, book depth and actions.
  - `/market/blotter`: the trade blotter.
  - `/market/surveillance`: surveillance alerts.

  A53 reserved the group name `(markets)`. Route group names do not change URLs, and A53 creates no folder under that name, so the two do not collide. The name in A53's README is corrected as documentation in loop A53.1. The URL is whatever A53's `nav.ts` row says. A mismatch is fixed in `nav.ts` through A53.1, never by adding a second route.
- **Feature code.**
  - `apps/web/src/features/resale/`:
    - `api.ts`: the `@realx/sdk` calls in §6 and nothing else. A52's lint forbids `fetch`. A feature lint rule forbids SDK imports outside `api.ts`.
    - `actions.ts`: the parity manifest.
    - `errors.ts`: sentences for A91's, A48's and A21's codes, extending A52's table.
    - `status.ts`: one table from A91's SM-09 order and trade states (plus `awaiting_cosign`, `registration_state` and `cancel_reason`) to pill text, sub-line and tone.
    - `state/`: quote sequencing, the idempotency-key store and the device draft.
    - `components/`: `SellCard`, `HoldingSplit`, `PriceBandBar`, `ProceedsLine`, `SellChecks`, `OrderBook` (the React build of A50's new block), `BuyCard`, `FillPreview`, `RestingBidOption`, `BoardStatusPill`, `MyResaleOrders`, `ResaleOrderTimeline`, `CancelListingDialog`, `CosignPanel` (the second holder's confirm), `PendingAgentConfirmation`, `HaltBanner`.
    - `market-tracker/`: `MarketTrackerScreen`, `MarketKpis`, `PriceVsValuationChart` and `HaltHistory`.
    - `__tests__/` and `e2e/`.
  - `apps/console/src/features/market/`: `api.ts`, `actions.ts`, `errors.ts`, and `components/` (`InstrumentsTable`, `InstrumentHeader`, `VenueModePanel`, `BandPanel`, `HaltPanel`, `HaltInstrumentDialog`, `CloseInstrumentDialog`, `CircuitBreakerCard`, `DepthTable`, `OpenOrdersTable`, `SurveillanceList`, `BlotterTable`, `HaltAllDialog`), plus `__tests__/` and `e2e/`.
- **Components.**
  - Web components come from `@realx/ui` (A05): `TextField` (the mono sell variant), `PriceBand`, `CheckList`, `GateRow`, `StatusPill` with `toneFor`, `QuantityStepper`, `OrderSummaryAside`, `KpiGrid`, `BarChart`, `TaskRow`, `Banner`, `PreviewBanner`, `Timeline`, `Button`, `Toast`, and A52's `ErrorState`, `EmptyState` and `LoadingState`.
  - Console components come from A53: `ConsoleTable`, `StatusCell`, `IdCell`, `MoneyCell`, `WhenCell`, `MaskedCell`, `AgentMarker`, `PrivilegedActionFlow` / `usePrivilegedAction`, `HardwareKeyConfirm` (including the ratify variant) and the extension-B blocks in `src/shell/blocks/`.
  - `OrderBook` is the one new block this node codes. It lives in the feature folder and must match A50's drawing. It is not added to `packages/ui`.
- **Server and browser split.** Reads of the instrument, the book, the market series and the person's orders run in server components. The forms, quotes, passkey ceremony and writes run in client components.
  - The browser works out no band, no fee, no proceeds, no average price, no fill, no cap headroom and no gate verdict. Every figure is the server's `formatted` string, or `formatAud`, `formatPct` or `formatTokens` applied to the server's integer cents, basis points and units.
  - The feature ESLint rule from A57 is repeated here: no `*`, `/`, `%` or `Number(` on identifiers ending in `_cents`, `_units`, `_bps`, `price` or `fee`.
  - Positions on the band bar and widths of depth bars are server fields (`marker_bps`, `depth_bps`) used as CSS percentages. They are not computed in the browser.
- **Time.** Times use `formatDateAest` from the server's ISO timestamps. A52's lint bans `Date.now()`. Countdowns (circuit-breaker reopen, halt end, ratification window, co-holder deadline) are the server's timestamp minus the server time from the response's `Date` header, rounded down to whole minutes.

**2. INV-05 Sell tokens**

- **Contract.** Every call on this page is one of A91's §4 and §9 routes, under `/v1/market`, taken verbatim. The instrument id is `INS-{code}` (A91 §2). Operation ids are the ones A09's generator derives from A91's OpenAPI document; the screens never name a route A91 does not ship.
- **Entry guard.** The page first reads `GET /v1/market/instruments/{id}/sell-preview` (scope `market:read`). The route is reachable only for a vehicle the person holds (design analysis §6.4 item 10: "Sell only from a holding").
  - `held: 0` renders A58's "not held" page. There is no "you hold 0 tokens" and no "20 of 0" placeholder (the prototype's bug, closed by A91 §9).
  - A `404` or `not_listed` (an offer still open, or a trust not yet listed) renders the Banner "This trust is not on the resale board yet" with a link to the holding (POR-02).
- **Layout as designed** (dc lines 481-501).
  - The page header is "Sell tokens" with the subline "{property} · you hold {held} tokens." (`PG.sell`, line 873). The eyebrow shows the product-map code INV-05.
  - When `previewMode` is on, the brick `PreviewBanner` (`#F5E4DB` / `#7A3A1E`, radius 10, padding 14×18) comes first. Its sentence is "Preview · the resale board opens with the retail launch (from about 8 Apr 2027), once counsel or ASIC relief allows." The text is A21's perimeter copy key `resale_preview`, checked against line 482 at Specify. A52's shell draws the banner in preview mode, so the page draws none of its own. Exactly one banner exists in the DOM.
  - Below it is a grid, `minmax(0,1.3fr) minmax(260px,1fr)`, gap 24.
  - **The left white card** (1 px `#E3DED4`, radius 14, padding 26, gap 18) holds:
    - Two mono `TextField`s in a two-column grid: "Tokens to sell", with the placeholder "20 of {unlocked}" (the design shows `sellH.qty`; the product map says only unlocked units can be listed), and "Limit price per token", with the placeholder set to the instrument's last price, or its reference price when it has never traded, from `GET /v1/market/instruments/{id}`.
    - "Allowed price band (G09)": the 10 px bar with the design's gradient (`#ECE8E0` 0-15 %, `#D7E4DC` 15-85 %, `#ECE8E0` 85-100 %) and the 3×18 px ink marker. The marker sits at the sell-preview's marker position in bp (A91 §4). Before any price is entered it sits at the position the server returns for the reference price. The caption row reads "{band lo} · valuation {valuation} · {band hi}" in Geist Mono 12 `#45484D`.
    - "Place sell order" (primary green pill).
  - **The right card** is the `CheckList` of the sell-preview's four check rows, in the server's order:
    - "Lock-in period (G08)".
    - "Trading halt (G10)", with its detail from A48's `haltNotice()`.
    - "Price band (G09) · ±15% of latest valuation" (the percentage comes from the instrument's band, never a constant).
    - "Board status", with the detail "Opens with retail launch" and the brick "2027" pill while the row is `preview`, or "Open" green, "Halted" grey or "Paused" amber in P2.
- **Additions from the product map.** The design does not draw these. They are composed from A05 blocks, the tokens below and A50's OrderBook (§ Design references).
  - A `HoldingSplit` line above the fields: "You hold {held} · {unlocked} unlocked · {locked} locked until {unlock date}", then "{listed_units} listed" when the person has open asks and "{unavailable_units} not available" when any other freeze applies. A91's sell-preview returns one frozen figure that mixes the listing freeze with legal and AML freezes. Loop A91.1 splits it into `listed_units` (frozen by this person's own open asks on this instrument) and `unavailable_units` (every other freeze, with no reason code in the payload). The screen renders both and adds nothing. A test asserts, in test code only, that the two equal the response's frozen figure, and that no reason string reaches the DOM. If A91.1 is refused, the split is an escalation, never browser arithmetic.
  - A `ProceedsLine` under the fields: "Proceeds {proceeds} after {fee_pct} fee", from the sell-preview's fee bp and proceeds. The fee is never the fees page's 0.5 %.
  - An `OrderBook` preview under the card, from the sell-preview's aggregated book: asks cheapest first and bids highest first, with the floor line and the caption "Floor {band lo} ({pct} of valuation)".
  - A "Your listings on this board" list (`MyResaleOrders`) from `GET /v1/market/me/orders?state=` (with the `instrument=` filter added in loop A91.1). Each open, partly sold, paused or awaiting-co-holder ask shows its quantity, price, status pill, "Expires {date}" from the order's server `expires_at` (placed time plus `order_max_days`, 30 by default, computed by A91), a link to its status page, and "Cancel listing" only where the server's `cancellable` is true.
  - A pending agent confirmation (A12's `agent_confirmation` for this instrument, returned with `428 CONFIRMATION_REQUIRED`) shows as a row "Your agent {token name} asked to list {units} at {price} · waiting for your confirmation", linking to ACC-05 ([[A60 - Investor and sponsor API tokens and agents screens (ACC-05, SPN-16)]]). No order exists and nothing is frozen until the person confirms there.
- **Live preview.** Every change of quantity or price re-reads `GET /v1/market/instruments/{id}/sell-preview?units=&price_cents=` (A91 §4). It is a read-only `GET` that writes no rows, so it also runs in preview mode.
  - Calls are debounced by 250 ms and carry a rising client-side sequence number. Late responses are thrown away. While a read is in flight, the proceeds line keeps the last figure with a shimmer, and the primary button is disabled.
  - The response carries, per A91 §4: held, unlocked and locked units with the unlock date; the frozen figure (split per A91.1); the valuation and its date; band `lo` and `hi`; the marker position in bp; fee bp, gross, fee and proceeds; the aggregated book with the floor; and the four check rows with their strings from A19's verdicts and `haltNotice()`.
  - A failing G09 row shows the server's fix-it verbatim, for example "Price must be between A$87.04 and A$117.76 (85 %–115 % of the June valuation)". The bar's marker moves into the grey zone.
  - A failing G08 row shows the verdict's earliest date and the units unlocked today, and offers "Use unlocked ({unlocked})", which sets the field to the server's figure.
  - A failing G10 row shows the halt notice and its expected end.
  - Any failed row disables the button (opacity .45, `aria-disabled`, explained through `aria-describedby`).
- **Place sell order (P2, flag on).** The click runs one write through A52's `withFreshAssertion`:
  1. The body is `{side: 'sell', units, limit_price_cents, time_in_force: 'rest', acknowledgements: [], confirmation}`, sent to `POST /v1/market/instruments/{id}/orders` (scope `market:order.sell`). Asks are always `rest` (A91 §4); the screen sends it explicitly.
  2. The `Idempotency-Key` is minted once per canonical payload (instrument, side, units, limit, time in force) and kept in `sessionStorage` under that payload's hash, with every read and write wrapped in try/catch. Changing the quantity or price mints a new key.
  3. `confirmation` is A11's fresh passkey assertion over A91's canonical payload, which includes the idempotency key (the A37 shape). The fresh-assertion purpose is the one A11 and A91 register for market orders; if A91 has not named it, loop A91.1 names it. On `FRESH_ASSERTION_*` the ceremony runs once more with the same key.

  How responses are handled:
  - `201`, or `200` on replay, returns the order in `validating` (or `new_order` with `awaiting_cosign`, below). The page opens its status page, where the ask shows "Checking" until A91 reports `open`, which happens only after A28 confirms the freeze. At `open` the toast reads "Listed {units} tokens at {price} · tokens set aside until sold or cancelled", and the `HoldingSplit` re-reads the server.
  - `MKT_PRICE_OUTSIDE_BAND` → the G09 row fails with the verdict's range, and the preview is re-read, because the band may have moved.
  - `MKT_LOCKED` → the G08 row fails with the verdict's earliest date and "Use unlocked".
  - `MKT_INSUFFICIENT_UNLOCKED` → the preview is re-read and the row reads "Only {unlocked} tokens can be listed now".
  - `MKT_HOLDING_FROZEN`, and A42's G14 `under_review` refusal → one byte-identical sentence: "This account is under review. You can't place orders right now." No reason is shown.
  - `MKT_HALTED` → the halt notice with its cause, criteria and expected end.
  - `MKT_BOARD_NOT_OPEN`, `MKT_VENUE_DISABLED` and `MKT_INSTRUMENT_NOT_OPEN` → the Board status sentences.
  - `MKT_TICK_SIZE` and `MKT_QUANTITY_INVALID` → field errors.
  - `MKT_INSIDER_BLACKOUT` (a policy row, default off) → the server's sentence.
  - An ask later rejected with `MKT_FREEZE_FAILED` (asynchronous) → the status page reads "Not accepted · we couldn't set your tokens aside, so nothing was listed. Try again."
  - `rate_limited` and `agents_paused` → A52's sentences.
  - An unknown code → "Something went wrong · try again", with the request id in Geist Mono.
- **Place sell order while the flag is off (the P1 preview).** The design's click shows the toast "Resale board opens in 2027 · order saved as draft" (line 909). The node summary says no order is created.
  - The page makes no mutating call. A52's preview-mode SDK wrapper refuses one locally, and A91 would refuse it again with `MKT_BOARD_NOT_OPEN` (G21 verdict attached) and write no order row.
  - The checks, band, split and proceeds come from the sell-preview `GET`, which A91 serves in P1 with the "Board status" row `preview`, "Opens with retail launch" and tone `brick`.
  - **Specify point (b): what "saved as draft" means.** The proposed reading is that the draft is a per-device convenience: the entered quantity and price are kept in `localStorage` under the vehicle code (try/catch, restored on return, cleared when the person clears the fields). Nothing is saved on the server, no tokens are frozen, no order or intent record exists, and the design's toast is shown verbatim. If the user prefers a server-side "tell me when the board opens" intent, that is an A91 contract addition (loop A91.1) with no freeze. If the user prefers different toast wording, the copy changes in `errors.ts`. The oracle is written against the reading the user picks.
- **Cancel listing.** `CancelListingDialog`: "Cancel this listing? {remaining} tokens go back to your available balance. Tokens already sold stay sold." Confirming calls `POST /v1/market/orders/{id}/cancel` (scope `market:order.cancel`, idempotency key). It needs no confirmation, because A91 treats cancelling as protective.
  - On success the pill turns to "Cancelled" and the `HoldingSplit` re-reads the server. The dialog result shows the server's released and sold units (the cancel response fields named in loop A91.1). It never subtracts.
  - When the seller's address is in an SM-07 compliance freeze, A91 takes the ask off the book but holds the release for its four-eyes unfreeze. The screen then shows "Cancelled", and the units appear under "not available" with no reason.
  - `MKT_ORDER_NOT_CANCELLABLE` renders the server's sentence, for example "This listing has sold and is settling".

**3. INV-06 Buy on the resale board**

- **Entry guard.** An instrument must exist for the code (`GET /v1/market/instruments/{id}`). A vehicle with an open primary offer redirects to INV-01 (A57). The page is open to any signed-in investor, holder or not.
- **Layout from A50's drawing** (design-ext INV-06).
  - The `PreviewBanner` sits at the top while the flag is off.
  - The "Board status" pill: Open green; Halted grey with the reason (for Bay Road Apartment in the seed, "Halted · meeting until 7 Oct 2026 (G10)"); Paused amber ("Paused · valuation stale (G20)").
  - On the left is the `OrderBook`, from `GET /v1/market/instruments/{id}/book`:
    - Two columns: Asks, cheapest first, and Bids, highest first. Price and quantity are in Geist Mono.
    - Each row has a depth bar at the server's `depth_bps` (loop A91.1): green.soft for bids and brick.tint for asks.
    - A 1 px ink floor line sits at the band floor.
    - The caption reads "Band {lo_pct}-{hi_pct} percent of valuation {valuation} (G09)".
  - On the right is the buy card:
    - The `QuantityStepper` in whole tokens. A91's quote takes `units` only, so the design's "or enter an amount" field is not built unless loop A91.1 adds an `amount_cents` parameter (an optional item, below). The browser never turns an amount into units.
    - The bound line "up to {max_units_by_cap} tokens · cap {cap_pct} (G07)".
    - The `GateRow`s from the quote's gate rows: G01, G04 (wholesale) or G05 (retail target market, P2), G06, G07 with the post-trade percentage, G09 and G10, each with the server's text. A91's `resale_bid` set equals a primary purchase's with G11 replaced by G09 and G10.
    - The amber row "Your order may rest as a bid inside the band if only part fills", with the `RestingBidOption` checkbox. Ticked sends `time_in_force: 'rest'`; unticked sends `'fill_and_cancel'`. A91 has no market orders: the order's limit is the price of the last ask the quote needs, and any part that does not fill rests at that limit. The row states "Any part that doesn't fill waits as a bid at {limit}", with the limit taken from the quote's last fill. There is no separate bid limit field.
    - The dark `OrderSummaryAside`, with the property name, a `FillPreview` (one line per ask level taken: "{units} at {price}"), "Average price", "Subtotal", "Transaction fee {fee_pct}", "Total" (mono 26) and "Paid from funds available: {available}".
  - When the board is halted, the buy card is replaced by the `HaltBanner` with the halt notice and its expected end, and there is no order button. Paused behaves the same way.
  - With an empty book the quote has no fills, "Buy" is disabled and the card reads "No tokens are listed right now".
- **Live quote.** Changes re-read `GET /v1/market/instruments/{id}/quote?side=buy&units=` (scope `market:read`), with the same debounce and sequencing as INV-05. It is a read-only `GET` with no quote id, so it also runs in preview mode.
  - The response carries, per A91 §4: `fills[]` (units, price and fee per fill), the average price, the subtotal, the per-fill fees, the total, `max_units_by_cap`, `max_units_by_funds` and the gate rows.
  - The stepper's upper bound is `max_units_by_cap`. A G07 failure shows "Use maximum ({max_units_by_cap})".
  - When the entered units exceed `max_units_by_funds` (a comparison of two server integers, not arithmetic), A57's deposit-shortcut pattern shows (a `Banner` linking to POR-05 with a return path), and "Buy" stays disabled. A91 reserves AUD when the bid is validated, so a board buy cannot wait for funds.
  - When the G12 row lists documents (whether a secondary buyer must acknowledge the PDS and TMD is a counsel question; A90 supplies the documents and the seed returns none for wholesale trusts), the card renders them with A05's `DocumentRow` and `AcknowledgeCard`, and "Buy" stays inert until A57's open-then-tick pattern is met. The acknowledgement ids go in the order's `acknowledgements[]`, and A35 records them.
- **Buy.** The button reads "Buy {units} tokens". It sends `{side: 'buy', units, limit_price_cents: <the price of the quote's last fill, as sent>, time_in_force, acknowledgements, confirmation}` to `POST /v1/market/instruments/{id}/orders` (scope `market:order.buy`), with an idempotency key per canonical payload, through `withFreshAssertion`.
  - `201` → the buy's status page. The book may have moved between the quote and the order. A91 then fills only within the limit; the rest rests at the limit (`rest`) or is cancelled (`fill_and_cancel`). A91 never refuses because the book changed, so the screen never shows "the board moved" as an error: it shows what the server filled. The toast uses the server's filled and resting units: "Bought {filled} tokens · settling", "Bought {filled} · {resting} waiting as a bid at {limit}", "Bid placed · {units} at up to {limit}" (nothing filled, `rest`), or "Nothing filled · those tokens were taken before your order arrived. No money moved." (nothing filled, `fill_and_cancel`).
  - If a sweep trips the circuit breaker, the earlier fills stand. The remainder is suspended (`rest`) or cancelled with `CIRCUIT_BREAKER` (`fill_and_cancel`), and the status page says so.
  - `MKT_PRICE_OUTSIDE_BAND` (the band moved after a new valuation) → the quote is re-read, with the Banner "The price band was updated · check the new total".
  - `MKT_HOLDER_CAP` → the G07 row fails with the server's `max_allowed_units` and "Use maximum".
  - `MKT_INSUFFICIENT_FUNDS` → the deposit banner with the server's available and needed amounts.
  - `MKT_HALTED` → the halt banner replaces the card.
  - `MKT_SELF_MATCH_PREVENTED` → "Part of your order would have matched your own listing, so that part was cancelled. {filled} tokens bought." from the server's figures.
  - A gate verdict from A19 → that row fails with its fix-it text. Every other code is handled as on INV-05.
- **Optional A91.1 items, not built unless signed off.** A user-chosen bid limit (the quote accepting `limit_price_cents`) and the amount entry (`amount_cents`). Each is a named A91.1 change that needs the A91 owner's sign-off at Specify, before the node QA writes the oracle. If neither is signed off, INV-06 bids only at the quoted limit.
- **Your bids and buys.** A `MyResaleOrders` list for this instrument (`GET /v1/market/me/orders?state=&instrument=`) shows resting bids (cancellable where the server says so) and recent buys with their pill, each linking to its status page, plus any pending agent confirmation as on INV-05.

**4. Order status (asks, buys and bids), SM-09 on screen**

- The page reads `GET /v1/market/orders/{id}` (scope `market:read`), and the order's trades with their settlement fields (loop A91.1: `trades[]` with `state`, `attempt`, `max_attempts`, `registration_state` and `registered_at`). Proceeds and history also come from `GET /v1/market/me/trades`. Another person's order returns `404`, which renders A52's not-found page.
- The card shows the order id in Geist Mono, the pill, the `AgentMarker` when the order was placed under a mandate, the lines (units, limit, filled, average price, gross, fee, net or total) and the `ResaleOrderTimeline`.
- **Pill tables (`status.ts`, one file, tested against A91's vocabulary).** A91 splits SM-09 into an order subset and a trade subset (A91 §1). Together they are the 13 SM-09 states, with `filled` in both.

  Order subset (on `resale_order`):

  | State | Seller text | Buyer text | Tone |
  |---|---|---|---|
  | `new_order` with `awaiting_cosign` | Awaiting your co-holder | Awaiting your co-holder | amber |
  | `new_order` | Checking | Checking | grey |
  | `validating` | Checking | Checking | grey |
  | `rejected` | Not accepted | Not accepted | red |
  | `open` | Listed | Bid open | green |
  | `partially_filled` | Part sold | Part filled | amber |
  | `filled` | Sold · settling | Bought · settling | amber |
  | `suspended` | Paused · trading halted | Paused · trading halted | grey |
  | `cancelled` | Cancelled (sub-line from the reason table) | Cancelled (sub-line from the reason table) | grey |

  Trade subset (on `trade`, one row per fill on the status page):

  | State | Text | Tone |
  |---|---|---|
  | `filled` | Matched · settling soon | amber |
  | `settling` | Settling | amber |
  | `settlement_failed` | Settlement delayed · retrying ({attempt} of {max_attempts}) | amber |
  | `busted` | Trade cancelled · nothing moved | grey |
  | `settled` | Settled · register updating | amber |
  | `settled` with `registration_state = rejected` | Register update delayed | amber |
  | `registered` | Complete · on the register | green |

  The attempt and the maximum are server fields (A91 allows one attempt plus three retries); no count is written into copy. The instrument states `pre_open`, `open`, `halted` and `closed`, plus the venue mode `disabled`, map to the Board status pill (Preview brick, Open green, Halted grey, Paused amber for `halted` with cause `valuation_stale` as A50 draws, Closed grey).
- **Cancellation reasons (`status.ts`).** A91 cancels with a reason code. Loop A91.1 exposes an investor-safe `cancel_reason` on the investor order read: `USER`, `BAND_MOVED`, `EXPIRED`, `INSTRUMENT_CLOSED`, `HALT_TOO_LONG`, `CIRCUIT_BREAKER`, `SELF_MATCH`, `COSIGN_EXPIRED` and `OPS`, where `OPS` never carries the staff reason code. "Tokens released" is used for asks and "money released" for bids, from the order's side.
  - `USER`: "Cancelled by you" (or "Cancelled by your agent · {token name}").
  - `BAND_MOVED`: "Cancelled · a new valuation moved the price band and your price fell outside it · tokens released".
  - `EXPIRED`: "Expired on {date} after {order_max_days} days on the board · tokens released".
  - `INSTRUMENT_CLOSED`: "Cancelled · this trust left the resale board · tokens released".
  - `HALT_TOO_LONG`: "Cancelled · the board stayed halted past {date} · tokens released".
  - `CIRCUIT_BREAKER`: "The unfilled part was cancelled because trading paused after a sharp price move".
  - `SELF_MATCH`: "The unfilled part was cancelled because it would have matched your own order".
  - `COSIGN_EXPIRED`: "Your co-holder didn't confirm within 24 hours, so the order was not placed. Nothing was set aside." (The outcome and code A91 uses when the 24 hours run out are named in loop A91.1.)
  - `OPS`: "Cancelled by RealX · tokens released". No reason is shown, whatever the payload carries (tipping-off, as in A57).
- **Timeline.** The timeline uses A91's keys and the server's strings, verbatim:
  - `placed`: "Listed · {date} · {id}".
  - `awaiting_cosign`: "Waiting for {co-holder name} to confirm · by {deadline}".
  - `tokens_set_aside`: "{units} tokens set aside in your wallet".
  - `matched`: "Sold {units} at {price} to a buyer on the board".
  - `settling`: "AUD and tokens swap together".
  - `settled`.
  - `registered`: "Recorded on the trustee register".

  "Complete" is shown only at `registered`, never at `settled`. That is SM-09's rule that nothing is final before the registry acknowledges.
- **Proceeds for a seller** come from the trade's proceeds (`GET /v1/market/me/trades`, `proceeds_cents`). A91 releases seller proceeds only at `registered` (A91 §6, spec point (b)). So "A$9,930.10 paid to your funds available", linking to POR-04 (A58), is shown only when the trade is `registered`. Before that the line reads "Proceeds of {proceeds} arrive when the register updates".
- **Joint and entity accounts.**
  - A joint account with `signing_rule = both` keeps the order in `new_order` with `awaiting_cosign` for up to 24 hours (A91 §4). The placing holder sees the pill "Awaiting your co-holder", the timeline row and the server's deadline ("{co-holder} must confirm by {deadline}"). The countdown is the server's deadline minus the server time, as in §1.
  - The second holder gets an entry point in "Your listings" / "Your bids and buys" and on the status page: "Confirm this order", which shows the same lines and runs A11's passkey ceremony over the same payload hash. The route for the second holder's assertion is named in loop A91.1 (A91 §4 requires the assertion but names no route). This step is `human_only`; whether the second holder's own agent may co-sign under its mandate is part of the same A91.1 item.
  - When the 24 hours run out, the order shows the `COSIGN_EXPIRED` copy above, with nothing frozen or reserved.
  - An entity account without a verified signatory (A30's `is_verified_signatory`) is refused with the code A91 returns for it (named in loop A91.1 if not already), rendered as "Only a verified signatory of {entity name} can place orders. Ask your account administrator to verify you." Nothing is created.
- **Other states with no drawing:**
  - A `busted` trade: "This trade could not settle and was cancelled. Nothing moved: your tokens and money are where they were."
  - A `settled` trade with `registration_state = rejected`: "Register update delayed · your trade has settled and RealX is correcting the register record. Your money stays safe in the settlement account until it's done." No break id or reason is shown.
  - A `suspended` order: the halt notice and "Your order resumes when trading reopens".
  - A `rejected` order: the failed gate's title and fix-it text, or the `MKT_FREEZE_FAILED` sentence in §2.
- **Live updates.** Re-read every 20 s while the order or any of its trades is not terminal, and on window focus. Polling stops when the order is `rejected` or `cancelled` with no live trade, or when every trade is `registered` or `busted`.

**5. INV-07 Market tracker (public)**

- `MarketTrackerScreen` reads `GET /v1/market/instruments/{id}/tracker` (public, no session needed; A91 §9). A52's `listing.visibility` rule from A21 applies: when the vehicle is hidden from visitors, the page renders A54's sign-in prompt, never figures.
- **Layout from A50's drawing.**
  - A `KpiGrid` with four tiles: Last price, Valuation (with the date), Premium or discount (the server's figure with its sign, brick when negative and green when positive), and Tokens traded in 30 days.
  - A `BarChart`, the capital-returns variant: bars for monthly last price, with a thin ink line for valuation, over six months, from the server's series.
  - "Halt history" as admin-style `TaskRow`s, pill first ("Halted · meeting" grey, "Resumed" green, "Halted · valuation stale (G20)" amber), with mono dates and durations from A48's public halt history as A91 returns it.
  - There is no buy button, as A50 draws it. Signed-in investors get a text link "Buy on the board" to INV-06, and holders also get "Sell" to INV-05, only when the instrument is `open`.
- **Empty state (product map).** With no trades, A91 returns `{trades: 0}` plus the offer price and the valuation only. The page shows "No trades yet", the chart shows the valuation line alone, and the premium tile reads "—".
- **Preview.** While the flag is off, the `PreviewBanner` sits on top, and the page shows valuation, band and "No trades yet" only.
- **Accessibility.** The chart has a data-table twin (visually hidden, reachable), and the tile tones are also stated in text ("2.5 % below valuation").

**6. Parity, agents and the action manifests (D04, G22)**

- **Investor manifest.** `features/resale/actions.ts` lists every investor action with its A91 route and scope. A unit test asserts each route exists in `openapi.json` and that `@realx/sdk` exposes its generated operation. [[A100 - Agent-parity suite- every UI action has an API and MCP equivalent under the same gates]] is the binding parity check: it proves that each succeeds, or is refused, identically for a person, a personal token and an MCP call ([[A104 - MCP server generated from the OpenAPI document]]). The preview parity script in this node is only a stopgap until A100 runs.

  | Action | Route (A91 §4 and §9, under `/v1/market`) | Scope | Owner |
  |---|---|---|---|
  | Read the instrument | `GET /instruments/{id}` | `market:read` | A91 |
  | Read the order book | `GET /instruments/{id}/book` | `market:read` | A91 |
  | Sell preview (holder split, band, proceeds, checks) | `GET /instruments/{id}/sell-preview?units=&price_cents=` | `market:read` | A91 |
  | Buy quote | `GET /instruments/{id}/quote?side=buy&units=` | `market:read` | A91 |
  | List for sale | `POST /instruments/{id}/orders` with `side: sell`, `time_in_force: rest` | `market:order.sell` | A91 |
  | Buy, resting the rest or not | `POST /instruments/{id}/orders` with `side: buy`, `time_in_force: rest` or `fill_and_cancel` | `market:order.buy` | A91 |
  | Cancel a listing or bid | `POST /orders/{id}/cancel` | `market:order.cancel` | A91 |
  | List my orders | `GET /me/orders?state=` (with `instrument=` after A91.1) | `market:read` | A91 |
  | View an order | `GET /orders/{id}` | `market:read` | A91 |
  | My trades and proceeds | `GET /me/trades` | `market:read` | A91 |
  | Market tracker | `GET /instruments/{id}/tracker` | public | A91 |
  | Co-sign a joint order | the A91.1 co-sign route | `human_only` (A91.1 decides agent co-sign) | A91 |
  | Confirm an agent's order above its caps | `POST /v1/agent-access/confirmations/{id}/confirm` (reached from ACC-05) | `human_only` | A12, A60 |
  | Open a document (only when a buy requires one) | `documents.versions.open` | documents | A35 |

  The passkey confirmation inside the two order writes is marked `human_only`, and its agent equivalent is named: A12's mandate signature within the caps, or the `428 CONFIRMATION_REQUIRED` path that A60 renders.
- **Console manifest.** `features/market/actions.ts` lists every staff action by A91's §8 and §9 names and A48's routes, and a test checks each against A91's and A48's policy rows.
  - Reads: `GET /v1/market/instruments` and `/instruments/{id}` (staff fields, with `allowed_actions`), `GET /v1/market/trades?instrument=&state=` (the blotter), the staff open-orders list (loop A91.1), `GET /v1/market/surveillance/alerts` (MKTO, CCO and MLRO only), `GET /v1/market/instruments/{id}/breaker-events`, A48's `GET /v1/halts?vehicle=`, `GET /v1/halts/{id}` and `GET /v1/halts/vehicles/{id}/state`, and A21's flag and binding read.
  - A14 requests through A53's `PrivilegedActionFlow`:
    - `POST /v1/market/instruments` → `market.instrument.list` (`4E`, MKTO → CCO; G08, G20, G21).
    - `POST /instruments/{id}/venue-mode` → `market.venue_mode.change` (`4E`, CCO → LEGL; G21).
    - `POST /instruments/{id}/band` → `market.band.set` (`6E`, MKTO → CCO + CFO; G09, G13).
    - `POST /instruments/{id}/resume` → `market.instrument.resume` (`4E`, MKTO → CCO; refused with `MKT_STILL_HALTED` or `MKT_RECON_NOT_CLEAR`).
    - `POST /instruments/{id}/close` → `market.instrument.close` (`4E`, MKTO → CCO; the open-order count shown to the checker).
    - `POST /trades/{id}/bust` → `market.trade.bust` (`4E`, MKTO → CCO; only at `retries_exhausted`).
    - A48's `POST /v1/halts/{id}/clear` → `halt.clear` (`4E`, per A48's row), and `halt.extend`, each shown only when A48's halt detail lists it.
  - Protective acts (A48, `SGL+R`): `POST /v1/halts` for "Halt instrument" (`halt.open`) and `POST /v1/halts/batch` for "Halt all".
  - Direct acts, not A14 requests:
    - Ops cancel: `POST /v1/market/orders/{id}/cancel` with the staff scope and a mandatory reason code (A91 §4).
    - `POST /v1/market/surveillance/alerts/{id}/case` (MKTO alone, as the product map has no checker; creates an A42 case).
  - Every hardware-key decision and act, the ops cancel and the direct case-open are marked `human_only`.
- **No app-only paths.** Nothing on the screens is unreachable by a token. There is no server action, no route handler and no cookie-only endpoint. A test walks both features for `"use server"` and route handlers and finds none.
- **Same rules on every channel.** An agent's order passes the same gates in A91 (`resale_ask`: G01, G02, G08, G09, G10, G14, G21; `resale_bid`: the primary purchase set with G09 and G10 in place of G11), plus G22 in A12's middleware first.
  - G22 applies A91 §9's cap rule, restated here: `units` counts against `per_action_units` and `daily_units`, and the value at the limit counts against `per_action_cap_cents` and `daily_cap_cents`. For an ask that value is the gross (`units × limit`); for a bid it is the gross plus the fee on it. It is never the quoted total, which may be lower. `allowed_vehicle_ids` and time windows apply.
  - Within the caps, the mandate signing key satisfies the confirmation. Above them, or with no key, A91 returns `428 CONFIRMATION_REQUIRED` and creates no order, freeze or reservation. The screens show the pending confirmation as in §2.
  - An agent order shows on the status page exactly as the person's does, with the marker "Placed by your agent · {token name}". The person's cancel works on it.
- **Staff agents.** A MKTO or CCO agent token can read every console view its human reads (surveillance only for MKTO's and CCO's agents), and draft the A14 requests above as maker (`market:instrument.request`, `market:trade.bust.request`), all audited "agent {token prefix} on behalf of {person}".
  - It may flag a surveillance alert for a case only as a draft, which the MKTO confirms with a passkey (the A30 draft pattern, A91 §9). That confirm step is `human_only`. A direct `POST /surveillance/alerts/{id}/case` by an agent token is refused.
  - It is never the checker or the ratifier, never the one who acts under `SGL+R` (the act is the human's hardware-key assertion), never a signer, and has no scope for the ops cancel.
  - On every console page, the agent persona sees no Approve, Reject, Ratify, Halt instrument, Halt all, Open surveillance case, Cancel order (ops) or custody control. Controls it may use are the drafts that `allowed_actions` lists for it, labelled "Draft request". A51's grey banner "Agents prepare. A person approves." sits where the human would see the decision buttons. The screens add no rule of their own: they render A14's and A91's `allowed_actions` and refusals.

**7. ADM-21 Resale board control (operator console)**

- **Instruments (`/market`), from A51's drawing plus the product map's columns.**
  - A `PreviewBanner` sits on top while the flag is off: "Markets open with the retail launch, from about 8 Apr 2027, once counsel or ASIC relief allows".
  - The header action is "Halt all" (danger).
  - The `ConsoleTable` columns are: Instrument (code, name), Venue mode (`disabled`, `bulletin_board`, `licensed_partner` with the bound licence id; `licensed_market` belongs to P3 and A97), State (OPEN green, HALTED red, DISABLED grey, PRE-OPEN amber, CLOSED grey, as A51 draws them, through `toneFor`), Valuation (with its date), Band ("85-115 %"), Floor, Last, Δ vs valuation, 30-day volume, Lock-in ("elapsed" / "to Feb 27" / "meeting"), and Holder cap.
  - A halted row carries the reason as a sub-line ("Meeting in progress · G10", "Reconciliation break · G10", "Valuation stale · G20", "Circuit breaker · reopens 14:32"). The text comes from A48's `haltNotice()` and A91's breaker events.
  - Paging is server-side, and the filters (state, venue mode, sponsor) are mirrored in the URL.
  - With the flag off, the launch state is DISABLED for every instrument (product map: "Launch state: venue mode 'disabled' for every instrument").
- **Instrument detail (`/market/[code]`).** Every button shows only when the detail's `allowed_actions` lists it for the caller.
  - An `InstrumentHeader` with the pills.
  - A `VenueModePanel`: the current mode, and the licence or relief record and flag binding from A21. "Change venue mode" opens the `4E` flow (CCO → LEGL). The precheck refusals render as sent: `G21_NO_BINDING` ("No licence binding · cannot change venue mode", with the action disabled), `G21_PHASE_NOT_REACHED`, `MKT_OPEN_ORDERS` (naming the ops cancel needed first) and `MKT_NO_LIST_REQUEST`.
  - A `BandPanel`: the band version history (v, lo, hi, reference valuation, approval id, effective date) and "Set band or floor" (`6E`). The flow shows the server's precheck of the resting orders that would be cancelled with `BAND_MOVED`.
  - A `HaltPanel`: open and past halts from A48, each with cause, scope, published reopening criteria, end date, opened by (human, SVCA or the source module), the ratification state with the mono countdown, and the halt's own actions from A48's detail (`halt.clear` for a manual halt, `halt.extend`). A meeting, recon-break or valuation-stale halt clears only on its source module's event, so it never shows "Clear halt".
  - **"Halt instrument"** (danger, on an OPEN instrument): A48's `halt.open` through `HardwareKeyConfirm`'s ratify variant, drawn as A48's `design-ext:halt confirm card` ("Halt {trust}"). The reopening criteria and an end date are mandatory, because A48 refuses a halt without them (`HALT_CRITERIA_REQUIRED`, `HALT_END_DATE_REQUIRED`). An optional checkbox "Also request a token pause" drafts an A28 `pause` for the CCO or SECO to act on (the product map's "optional token pause"). The copy reads "This acts at once. {Role} must ratify within 2 h". MKTO or CCO acts; the other of CCO or CEO ratifies.
  - **"Resume trading"**: `market.instrument.resume` (`4E`, MKTO → CCO). `MKT_STILL_HALTED` renders the server's list of halt sources still open, and `MKT_RECON_NOT_CLEAR` renders the pre-flight result. A circuit-breaker halt lifts on its own and never shows Resume.
  - **"Close instrument"**: `market.instrument.close` (`4E`, MKTO → CCO). The dialog shows the server's open-order count and says every open and suspended order will be cancelled with `INSTRUMENT_CLOSED`.
  - A `CircuitBreakerCard`: the rule "a move above 10 % in 1 h halts for 30 min" as the server's parameter text, the last trigger from `GET /instruments/{id}/breaker-events`, and the reopening time.
  - A `DepthTable`: the book in the dark shell's table style, and an "Open orders" tab from the staff open-orders list (loop A91.1) with masked participant ids. "Cancel order (ops)" sits on each open order and needs a reason code from the server's list; it calls the ordinary cancel route with the staff scope, and the investor sees only "Cancelled by RealX".
  - "List instrument" (`4E`) is shown for `pre_open` instruments. It shows the precheck rows G08 (lock-in elapsed), G20 (valuation current), G21 (venue binding) and "fully funded", as A14 returns them.
- **Trade blotter (`/market/blotter`).** Columns: trade id, instrument, time, price, units, buyer and seller (`MaskedCell`), SM-09 state, settlement attempts ("{attempt} of {max_attempts}", both server fields), registration state and registry ack (time, "pending" or "rejected · break open"), and tx hash (`IdCell` with truncation). Agent-placed legs carry `AgentMarker`.
  - Filters: instrument, state, date, "failed settlements".
  - "Bust trade" (`4E`) shows only when A91 lists `market.trade.bust` for the row, which it does only at `retries_exhausted`. `MKT_BUST_NOT_ELIGIBLE` and `MKT_BUST_AFTER_CHAIN` render as sent.
- **Surveillance (`/market/surveillance`).** `TaskRow`s from A91's scenarios S1 to S8: related-party or wash, round trips, band-reject burst, marking the close, insider window, layering, agent burst and circuit-breaker fires. Each shows scenario id, instrument, the server's detail string (for example "same device, 6 round trips / 48h · 0.91") and score.
  - "Open surveillance case" (MKTO only) creates an A42 case and shows its id with a link to ADM-13's case view.
  - Participants are masked, and unmasking goes through A53's `MaskedCell` reason flow.
  - This node registers the badge source `market.surveillance.open` from A91's summary endpoint.
- **Halt all.** `HaltAllDialog` lists the instruments in scope, from the server's list of open ones. Reopening criteria and an end date are mandatory. The dialog calls A48's `POST /v1/halts/batch` through `HardwareKeyConfirm`'s ratify variant: "This acts at once. {Role} must ratify within 2 h".
  - Whether one assertion covers the list is A48's Specify item. The dialog does whatever A48 settles (one assertion, or one per vehicle) and shows progress per vehicle.
  - After the API's 2xx, the rows turn HALTED with the amber "Awaiting ratification · h:mm:ss" pill.
- **Roles.** Buttons follow the server's `allowed_actions` for the caller, never the client's role guess.
  - MKTO: maker for list, band, resume, close and bust; acts "Halt instrument" and "Halt all"; opens surveillance cases; ops cancel.
  - CCO: checker for list, band, resume, close and bust; maker for venue mode; acts or ratifies halts; reads surveillance.
  - CFO: second checker for bands (`6E`); reads instruments and the blotter.
  - LEGL: checker for venue mode.
  - CEO: ratifies halts under `SGL+R`, and is the checker on A48's CCO-drafted `halt.clear` path.
  - MLRO: reads surveillance alerts (A91 §7).
  - SVCA is the settlement service principal. It has no console session and appears only as the actor on blotter rows.
  - Other staff with `V` on M14 see instruments and the blotter read-only.
  - Surveillance is hidden from the nav for CFO, LEGL, CEO, SVCA and read-only staff, because A91 serves the alerts to MKTO, CCO and MLRO only. A direct URL renders A53's not-permitted page with no alert data.
- **Empty and error states.**
  - Instruments with none listed: "No instruments yet · a trust appears here once it is fully funded".
  - Blotter with no trades: "No trades yet".
  - Surveillance with none open: "No open alerts".
  - A53's error state shows `code`, `message` and `request_id`.

**8. Empty, error and edge states on the investor screens (each has a test and a screenshot)**

- **Loading:** skeletons in the design's row heights. The proceeds line and aside show "—" before the first preview or quote read.
- **Quote fails on the network:** the Banner "Couldn't check this order · retry", and the last good figures stay.
- **Halt starts mid-flow:** the next preview or quote read, or the order call (`MKT_HALTED`), returns the halt, the halt banner appears, and any typed values are kept.
- **Frozen or restricted account:** the checks render as sent. Units under another freeze show only as "{unavailable_units} not available", and `MKT_HOLDING_FROZEN` and G14's `under_review` render the one under-review sentence. No reason string reaches the DOM.
- **Nothing unlocked:** fields disabled with "Your tokens unlock on {date}".
- **Session expiry:** sign-in with a return path. The idempotency-key store in `sessionStorage` is kept for the same canonical payload, so a retried submission replays rather than duplicates.
- **Phone width (375 px):**
  - The grids stack. The CheckList falls below the card on INV-05, and the aside falls below the card on INV-06.
  - The OrderBook's two columns stack, asks above bids.
  - The band bar and caption fit 343 px.
  - There is no horizontal page scroll, and the 16 px gutter holds.
- **Accessibility:**
  - The book is a table with captions, and depth bars are `aria-hidden` with quantities in text.
  - The band is announced as a range ("allowed A$87.04 to A$117.76").
  - Disabled buttons explain why.
  - axe-core finds zero critical or serious issues on every route and state, in both apps.

## Owned paths (other nodes do not edit these)

- <workspace>\app\apps\web\app\(investor)\resale\
- <workspace>\app\apps\web\app\(public)\investments\[code]\market\ (INV-07, carved out of A54's `(public)\` by A54's exclusion; A52 route table)
- <workspace>\app\apps\web\src\features\resale\
- <workspace>\app\apps\console\app\(market)\
- <workspace>\app\apps\console\src\features\market\

Inside the features, by convention: `api.ts`, `actions.ts`, `errors.ts`, `status.ts` (web), `state/`, `components/`, `market-tracker/` (web), `__tests__/` and `e2e/`. Changes this node needs elsewhere are recorded as loops, not edits:
- A52.1: the INV-05 and INV-06 paths, the redirects, and `whenOff: 'preview'` for INV-06 and INV-07.
- A54.1: the one-file public INV-07 page.
- A53.1: the route-group name in the README, and the nav row if the URL differs.
- A91.1 (specify point (c)): the named contract additions below, each signed off by A91's owner at Specify before the node QA writes the oracle. Anything not on this list is built against A91 §4 and §9 as written.
  - Required: `depth_bps` on book levels; `listed_units` and `unavailable_units` in the sell-preview (no reason code); the `instrument=` filter on `GET /me/orders`; `expires_at` and `cancellable` on investor orders; the investor-safe `cancel_reason` set (`USER`, `BAND_MOVED`, `EXPIRED`, `INSTRUMENT_CLOSED`, `HALT_TOO_LONG`, `CIRCUIT_BREAKER`, `SELF_MATCH`, `COSIGN_EXPIRED`, `OPS` with no staff reason); `trades[]` on the order read with `attempt`, `max_attempts`, `registration_state` and `registered_at`; released and sold units in the cancel response; the joint co-sign route, its `cosign_deadline_at` and the expiry outcome; the entity unverified-signatory code; the fresh-assertion purpose for market orders; `allowed_actions` on the staff instrument and trade reads; the staff open-orders list; and the ops-cancel reason-code list.
  - Optional (built only if signed off): a user-chosen bid limit on the buy quote (`limit_price_cents`) and amount entry (`amount_cents`).

## Design references

- **The `sell` screen** (`<workspace>\design\RealX App.dc.html`, screen key `sell`, header code "INV-05 · P2", title "Sell tokens", subline "{name} · you hold {qty} tokens.", line 873). Template lines 481-501:
  - The `PreviewBanner` (482; `#F5E4DB` background, `#7A3A1E` text, radius 10, padding 14×18, 14 px, line-height 1.5).
  - The grid `minmax(0,1.3fr) minmax(260px,1fr)`, gap 24 (483).
  - The white card with radius 14 and padding 26 (484).
  - The two mono `TextField`s "Tokens to sell" and "Limit price per token" (1 px `#D6D0C4`, radius 8, padding 12×14, Geist Mono 15, background `#FBFAF7`) (485-488).
  - "Allowed price band (G09)" with the 10 px gradient bar `#ECE8E0` / `#D7E4DC` / `#ECE8E0` at the 15 % and 85 % stops, the 3×18 px `#17191C` marker, and the Geist Mono 12 `#45484D` caption "lo · valuation · hi" (489-493).
  - "Place sell order" (green `#1F5E4B` pill, padding 12×24, Geist 500 14) (494).
  - The four-row `CheckList` (1fr auto grid, padding 14×18, `#ECE8E0` dividers, Geist Mono 11 pills) (496-500).
- **Derivations.**
  - `band` lo = 0.85 × valuation and hi = 1.15 × valuation, with the `placeSell` toast (line 909).
  - `sellChecks` titles, details and tones (line 910): "Lock-in period (G08) · No lock-in on this trust · Clear" green; "Trading halt (G10) · No meeting in progress · Open" green; "Price band (G09) · ±15% of latest valuation · Set" green; "Board status · Opens with retail launch · 2027" brick.
  - `sellH` (line 842).
  - `TONE` (line 740).
  - `$` formatting "A$" en-AU with two decimals (line 744).
  - `buyClick` and `buyLabel` "Sell tokens (resale · 2027)" (lines 837-838).
  - The HoldingRow "Sell" (line 392).
  - The fees row "Transaction fee 0.5% · On each buy, and on resale sells from 2027" (line 811). It is shown only for context: the fee on screen is the server's `fee_bps`.
  - For the seed's Bay Road Apartment (valuation A$118.60, held 120), the design yields the band A$100.81 to A$136.39, the placeholder "20 of 120" and the limit placeholder A$115.20. Specify checks that A06's seed and A91's instrument read reproduce these strings.
- **Design analysis.**
  - §1.1 tokens `brick.tint` `#F5E4DB` and `brick.text` `#7A3A1E` (lines 48-49).
  - §2 items 20 (TextField, sell mono variant), 23 (CheckList), 29 (OrderSummaryAside), 30 (HoldingRow) and 32 (PreviewBanner).
  - §3 entry `sell` (lines 243-246).
  - §4 acceptance item 12: in P1 the sell action never creates an order; in P2, asks outside the band are rejected with the allowed range; a halt blocks the board.
  - §5 INV-06 (OrderBook block), INV-07 (KpiGrid + BarChart + halt-history TaskRows) and ADM-21 (instruments DataTable with pills, surveillance TaskRows, "Halt all").
  - §6.2: lock-in is a listing parameter, and the P1 board is disabled by the `resale.board` flag. A recon break halts resale for the trust.
  - §6.4 item 10: Sell only from a holding.
- **Undesigned screens.**
  - `design-ext:INV-06` and `design-ext:INV-07` from [[A50 - Design extension A- investor and sponsor screens not in the prototype]], in `app/design-ext/investor-sponsor/RealX App Ext A.dc.html`: the OrderBook block (depth bars in green.soft and brick.tint only, the ink floor line, the band caption for valuation A$412.00, which is the seed's Marion Street Warehouse with band A$350.20 to A$473.80), the buy card with "up to 380 tokens · cap 5 percent (G07)", the amber resting-bid row, the three Board status states, and the tracker's tiles, chart and halt-history rows.
  - `design-ext:ADM-21` from [[A51 - Design extension B- operator console screens not in the prototype]], in `app/design-ext/admin/RealX Admin Ext.dc.html`: the PreviewBanner, the instruments table with OPEN / HALTED / DISABLED / PRE-OPEN pills, halted-reason sub-lines, surveillance TaskRows, the "Halt all" danger button with the ratify modal, Resume (`4E`), Set band (`6E`), and the default, halted, disabled and empty states.
- **Tokens from A05.**
  - Colours: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, red `#A3312A`, `#E3DED4` borders, `#ECE8E0` dividers, `#D6D0C4` field borders, `#FBFAF7` field fills, `#D7E4DC` in-band fill, `#45484D` captions, `#5E6166` details, the TONE map, and the dark admin shell's own tokens.
  - Type: Newsreader (headers and the aside title), Geist 500 13 labels, Geist Mono for every price, quantity, id and date.
- **Composed states.** The resale order status page, the extra SM-09 states, the busted and ops-cancelled copy, the halted buy card and the "not on the board yet" banner are compositions of existing A05 and A50 blocks in these tokens, with no new component, colour or font. They are listed in `features/resale/README.md`. The founder sees them in the P2 preview gallery. Any state the founder wants redrawn becomes A50's loop A50.2 (founder redraws after a preview), not a change made here.

## Requirements covered

**Screens**

- **INV-05 Sell tokens.** Done when:
  - The route renders the design's `sell` region token for token, with the holder split, proceeds line, book preview and "Your listings" added.
  - The band, fee, proceeds and checks equal the server's sell-preview to the cent. For the J07 sample (valuation A$102.40): band A$87.04 to A$117.76, and 100 × A$99.80 → proceeds A$9,930.10 after the 0.50 % fee.
  - Listing freezes exactly the listed units, as shown back from the server (`listed_units`), and units under any other freeze show only as "not available". Cancel releases the remainder.
  - Seller proceeds read "paid to your funds available" only at `registered`.
  - With the flag off, the preview banner stays, the toast appears, and no mutating request leaves the browser.
- **INV-06 Buy on the resale board.** Done when:
  - The book shows asks cheapest first with depth and floor.
  - The buy fills from the cheapest ask upward, as quoted by A91's `GET .../quote`. For the sample, 150 tokens at an average A$100.17 and a total A$15,100.13 including the 0.50 % fee, sent as one order with the limit A$100.90 (the quote's last fill).
  - The eligibility rows equal a primary purchase's, and the quantity is capped at `max_units_by_cap`.
  - An unfilled remainder rests at the quoted limit only when `time_in_force: rest` was chosen, and is cancelled otherwise. A book that moved shows what the server filled, never a refusal.
  - Joint orders show "Awaiting your co-holder" with the server's deadline, and the second holder can confirm. An unverified entity signatory sees the refusal copy.
  - A halted or paused board replaces the card with the halt notice.
- **INV-07 Market tracker.** Done when:
  - The public page shows last price against valuation, the premium or discount, 30-day volume and halt history from one server read.
  - "No trades yet" shows only the offer price and valuation.
  - Visibility follows A21's `listing.visibility`.
  - There is no buy button.
- **ADM-21 Resale board control.** Done when:
  - The instruments table, detail, blotter and surveillance pages render every column and state of A51's drawing and the product map.
  - These actions run only through A53's flow and modal, with A91's and A48's policy codes: Change venue mode (`4E`), List instrument (`4E`), Set band or floor (`6E`), Resume trading (`market.instrument.resume`, `4E`), Close instrument (`4E`), Bust trade (`4E`), Clear halt (A48 `halt.clear`, `4E`), Halt instrument (`halt.open`, `SGL+R`) and Halt all (`halts/batch`, `SGL+R`). The ops cancel and "Open surveillance case" are direct single-actor acts, with a reason code or an A42 case id.
  - Buttons follow the server's `allowed_actions`, and the agent persona sees none of the decision or act buttons.
  - Surveillance is visible only to MKTO, CCO and MLRO.

**Journeys**

- **J07 Sell tokens on the resale board.** Done when steps 1, 3, 4, 5 and 7 run at screen level in Playwright against the Docker stack, with A91 on the local chain, the registry sandbox and a test licence binding:
  - Setup, in this order, in the test profile only: the seed's test licence binding (A06); `resale.board` turned on through A21's `4E` under that binding; the venue mode changed to `bulletin_board` (CCO → LEGL).
  - Step 1: MKTO lists PAD01 and CCO approves. The row goes PRE-OPEN → OPEN.
  - Step 3: the seller lists 100 at A$99.80, and 100 units are frozen.
  - Step 4: another investor buys 150 and gets the fill preview and the gate rows.
  - Step 5: the blotter shows filled → settling → settled → registered, with the registry ack.
  - Step 7: a planted move above 10 % halts PAD01 on ADM-21 and INV-06. A91's injectable server clock is then advanced past `ends_at`, and the assertion is made on server time (HALTED before `ends_at`, OPEN on the first read after), never on a real 30-minute wait.

  Steps 2 (POR-02) and 8 (POR-04) are A58's screens, step 6 (ADM-17) is A73's, and the full journey is owned by [[A94 - P2- end-to-end journeys and launch update]].

**Gates**

- **G07 Holder cap.** Done when INV-06's stepper cannot exceed the quote's `max_units_by_cap`, the row shows the post-trade percentage and cap, "Use maximum" works, and a forced buy above it is refused by A91 and rendered with nothing reserved.
- **G08 Lock-in.** Done when INV-05 shows unlocked against locked units with the unlock date, locked units cannot be listed from the screen, and a forced listing is refused and rendered. On ADM-21, "List instrument" shows the lock-in precheck.
- **G09 Price band.** Done when:
  - The band on every screen is the server's.
  - An ask or bid outside it is refused with the allowed range and rendered.
  - A band move between the read and the order is refused with `MKT_PRICE_OUTSIDE_BAND` and the new range, and forces a re-read. Resting orders outside a new band show the `BAND_MOVED` cancellation.
  - "Set band" runs only as `6E`, with the server's precheck of the orders that would be cancelled shown.
- **G10 Trading halt.** Done when:
  - A halt from any cause (manual, meeting, stale valuation, recon break, token pause, circuit breaker, platform suspend) shows its public notice and expected reopening on INV-05, INV-06 and INV-07.
  - Open orders show "Paused".
  - "Halt all" acts at once and shows its ratification countdown.
  - "Clear halt" appears only where A48's detail lists it, and "Resume trading" only where A91 lists `market.instrument.resume`. `MKT_STILL_HALTED` and `MKT_RECON_NOT_CLEAR` render as sent.
  - "Halt instrument" acts at once with mandatory criteria and end date, and offers the optional token-pause draft.
- **G21 Licence perimeter.** Done when:
  - With `resale.board` off, all three investor screens render the preview and make no mutating call, and every instrument reads DISABLED.
  - Venue-mode change is refused and disabled with `G21_NO_BINDING` when no licence binding exists.
  - A forced API call in preview mode is refused by A91 with `MKT_BOARD_NOT_OPEN` (G21 verdict) and rendered.
- **G13 Maker-checker.** Done when every ADM-21 request runs through A53's flow with A91's §8 policy code and maker → checker pair, the maker's own agent is excluded from checking (`APR_MAKER_IS_CHECKER`), and no agent sees a decision button.
- **G14 AML.** Done when "Open surveillance case" creates an A42 case (MKTO only; an agent only drafts it for MKTO's passkey), surveillance is readable only by MKTO, CCO and MLRO, `MKT_HOLDING_FROZEN` and G14 `under_review` refusals render one reason-free sentence, and no case id, scenario or reason reaches an investor DOM.
- **G20 Valuation current.** Done when a `valuation_stale` halt shows "Paused · valuation stale (G20)" on INV-06 and INV-07 and its sub-line on ADM-21, and "List instrument" shows the G20 precheck row.

**Modules and state machines**

- **M14 Resale board.** Done for its screen surface when venue control, instruments with bands, lock-ins and caps, halts, the blotter and surveillance are all operable from ADM-21 through the SDK with parity, and the investor side of the board is operable from INV-05, INV-06 and INV-07.
- **SM-09 Resale order and trade.** Done when:
  - All 13 SM-09 states (the order subset and the trade subset), `awaiting_cosign`, `registration_state = rejected`, every cancellation reason and the four instrument states (plus `disabled` venue mode) have a pill, a timeline shape or row treatment, and a screenshot.
  - The state → text → tone table is one file tested against A91's vocabulary.
  - The screens never offer a transition the server does not list (`cancellable`, `allowed_actions`).
  - "Complete" appears only at `registered`.

**Founder requirement D04**

- Done when:
  - Both action manifests' operations exist in the SDK and OpenAPI.
  - The screens make no call a token cannot make.
  - The only `human_only` steps are the passkey confirmations (the order confirm, the joint co-sign and the agent-confirmation confirm), the hardware-key decisions and acts, the ops cancel and the direct surveillance case-open, each with its agent equivalent or refusal named.
  - An investor's agent can list, buy, rest a bid and cancel under the same gates and A91's cap rule (units, and value at the limit), and its orders show with the marker.
  - A staff agent can draft (including a surveillance-case draft) but never decide or act.

## Gauntlet loop for this node

- **Specify.** The frozen spec is this note plus:
  - A91's §4 and §9 routes verbatim (`/v1/market/instruments`, `/book`, `/quote`, `/sell-preview`, `/tracker`, `/orders`, `/me/orders`, `/orders/{id}`, `/orders/{id}/cancel`, `/me/trades`, and the staff routes), its `market:*` scopes, its §8 policy rows, its SM-09 subsets and its error catalogue, plus the named A91.1 items in Owned paths.
  - A48's `haltState`, `haltNotice()`, halt detail with `allowed_actions`, `POST /v1/halts`, `POST /v1/halts/batch`, `halt.clear` and the `design-ext:halt confirm card`.
  - A21's flag and binding reads, the `resale_preview` copy key and `listing.visibility`.
  - A14's action codes and policy rows named in §6.
  - A11's fresh-assertion purpose for market orders (named in A91.1 if A91 has not).
  - A12's `CONFIRMATION_REQUIRED` and `agent_confirmation` contract, and A91 §9's cap rule.
  - A52's route rows after loop A52.1, `previewMode`, `withFreshAssertion` and the error table.
  - A53's shell, table primitives, `PrivilegedActionFlow` and `HardwareKeyConfirm`.
  - A05's component props.
  - A50's and A51's `screens.json` entries.
  - A06's P2 seed, including Bay Road Apartment, Marion Street Warehouse and the PAD01 sample.

  Specify points (a) routes, (b) the preview draft and (c) the A91.1 items (required and optional) are decided by the user, with A91's owner signing off (c). The node QA records `specVersion` and the contract hashes, then writes the hidden material to `<workspace>\gauntlet\hidden\A92\`: the planted mock server, the preview-mode network probe, the reference crops and the parity diff script. Specify also confirms that the seed reproduces the design's `sell` strings for Bay Road, and that A91's quotes reproduce A$9,930.10 and A$15,100.13 for the J07 sample.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: the lead writes both `api.ts` files, the quote-sequencing and idempotency store, `status.ts` and both `actions.ts`. Then four parallel sub-agents work on disjoint folders:
  1. INV-05: sell routes, card, band bar, checks, holder split, proceeds, book preview, "Your listings", pending agent confirmations, cancel, the preview mode with the device draft.
  2. INV-06: buy route, OrderBook block, buy card, gate rows, the rest or fill-and-cancel option, aside, halted card, "Your bids and buys".
  3. The order status pages with the timeline, every SM-09 state, every cancellation reason, the joint co-sign and entity signatory states, plus `MarketTrackerScreen` with KPIs, chart, table twin and halt history.
  4. ADM-21: the four console pages, the privileged flows, "Halt instrument", "Close instrument", `HaltAllDialog`, the open-orders tab with the ops cancel, the blotter, surveillance and the badge source.

  The lead's `api.ts` files call only A91's §4 and §9 routes (after the signed-off A91.1 items), A48's, A21's, A12's and A35's. An integration pass then runs the J07 screen-level Playwright, the state gallery at 1280, 1440 and 375 px, the manifest tests, axe-core and `npm run verify`, and writes `app/evidence/A92.json`.
- **Challenge (node QA, Opus 5.5 high).**
  - Runs the hidden mock-server cases and the preview probe.
  - Forces submissions through the DOM and the SDK.
  - Renders the state gallery against the dc crops and A50's and A51's drawings.
  - Diffs the seed strings field by field.
  - Diffs both manifests against `openapi.json`.
  - Has a person, an investor token and an MCP call do the same listing and purchase (A100's harness once it exists).
  - Walks ADM-21 as MKTO, CCO, CFO, LEGL, CEO, MLRO, a read-only staff member and a MKTO agent, and confirms that SVCA has no console session.
  - Checks both widths and keyboard-only completion.
  - Greps for SDK imports outside `api.ts` and for arithmetic on money identifiers.
  - Confirms only the owned paths changed.

  The node QA writes `app/evidence/A92.qa.md`.
- **Gate.** A pass unlocks A94 and publishes the hidden cases into `app/`. The `resale.board` flag stays off by default after the pass. Turning it on is A21's `4E` against a licence binding, which is a human gate.
- **Fix loop.** At most three. Each loop re-runs the hidden cases, the preview probe, the state gallery and the J07 screen steps.
- **Escalation trigger.**
  - The same assumption fails three times.
  - A fix proposes any of the following (Critical: stop and reopen):
    - computing a band, fee, proceeds, fill or cap in the browser;
    - sending any mutating request in preview mode;
    - showing a reason for a freeze, an ops cancel or a surveillance-driven action to an investor;
    - rendering a decision button for an agent;
    - calling a route, scope or error code that A91 does not ship, outside the signed-off A91.1 items;
    - adding an app-only route.
  - A field the screens need is missing from A91's frozen contract after loop A91.1 (a spec defect upstream, reopened there).
  - The route settlement in specify point (a) is refused by A52's or A54's owner.
  - A counsel answer changes whether the board may exist in a form the screens assume (record under HG-LEGAL and ask the user).

## Node-specific challenges (node QA)

- **Figures from the wrong place (money).**
  - The mock returns a fee of A$49.91 for 100 × A$99.80. The proceeds line must read A$9,930.09, because that is what the server said, and A$9,930.10 must appear nowhere.
  - The mock returns a buy total of A$15,100.12, testing the half-cent rounding of the 0.5 % fee on A$15,025.00. The aside shows A$15,100.12.
  - 40 price edits arrive in 300 ms with their sell-preview responses out of order. The proceeds, the band marker and the idempotency key must end on the last edit's payload.
  - A band marker planted at 9,000 bp must sit at 90 % of the bar, however the entered price compares to the band.
- **The preview that leaks (compliance, G21).**
  - With `resale.board` off, the tester removes `disabled`, forces a click on "Place sell order", calls the SDK operation for `POST /v1/market/instruments/{id}/orders` from the console through the page's SDK instance, and replays a captured P2 request.
  - The network log must show zero calls to operations that `openapi.json` marks as mutating. A52's own auth endpoints (session refresh, sign-out) and error reporting are allow-listed, so they cannot make the probe flaky. The sell-preview and quote `GET`s may run. The SDK wrapper must refuse locally. The replay must be refused by A91 with `MKT_BOARD_NOT_OPEN` (G21 verdict) and nothing written, frozen on chain or reserved in the ledger.
  - The toast shows as designed, and the only artefact is the device draft (or the reading the user chose at specify point (b)).
  - The same probe on INV-06 and ADM-21 must find no mutating request and every instrument DISABLED.
- **Selling what you cannot (G08, G10, SM-07).**
  - Alex Morgan holds 120 Bay Road units, 40 of them locked until a date after the clock, and the wallet is partially frozen by an AML hold of 20 units.
  - The sell page must show 60 unlocked and list no more than 60.
  - A forced listing of 80 must be refused with `MKT_LOCKED` or `MKT_INSUFFICIENT_UNLOCKED`, and nothing frozen.
  - The frozen 20 must show as "20 not available" (`unavailable_units`) with no reason, and `listed_units` must read 0 until a listing is open. After a listing of 30, the split reads "30 listed" and "20 not available", and the test (not the page) checks that they add up to the server's frozen 50. The string "AML" must not appear in the DOM, the accessibility tree, the title or the console.
  - A meeting halt then starts. The open listing turns "Paused · trading halted" with the meeting notice, and "Place sell order" is inert with the G10 row failed.
- **Freeze equals listing (money, SM-09).**
  - "Place sell order" is double-clicked. Two tabs then list 60 each against 60 unlocked, with the same and with different payloads (so the same and different idempotency keys). Exactly one listing may exist, the second is refused with `MKT_INSUFFICIENT_UNLOCKED`, and the listed units shown must equal 60.
  - A buyer takes 25. The seller then cancels. The dialog and result must show 35 released and 25 sold, from the server, and the holder split must return to 35 unlocked plus the 25 settling out.
  - A cancel sent after the ask has fully filled (its trade settling) gets `MKT_ORDER_NOT_CANCELLABLE` rendered, with nothing released.
- **A board that moves under the buyer (money, G07, G09).** The expected outcomes follow A91 §4: an order fills only within its limit, and there is no book-changed refusal.
  - The J07 book (asks A$99.80 × 100, A$100.90 × 120, A$101.50 × 40) quotes 150 units with the limit A$100.90. Between the quote and the passkey, 60 of the A$99.80 ask are taken by someone else. With `rest`, the order fills 40 at A$99.80 and 110 at A$100.90 (all within the limit) and nothing rests. With the A$100.90 ask also cut to 20, it fills 40 + 20 and 90 rest as a bid at A$100.90; with `fill_and_cancel`, the 90 are cancelled and never shown as "resting". Every figure on the status page and toast is the server's, never the quote's.
  - A new valuation then moves the band down 12 %, leaving the resting bid at A$100.90 outside it. A91 cancels it with `BAND_MOVED`, and the page shows the band-moved sentence with the money released.
  - A new buy at the old limit is refused with `MKT_PRICE_OUTSIDE_BAND` and the new range, the quote is re-read with the Banner, and nothing is placed.
  - A buyer who already holds 38 units less than the 5 % cap asks for 300 units, so `max_units_by_cap` is 38. The stepper stops at 38, "Use maximum" sets 38, and a forced 300 is refused with `MKT_HOLDER_CAP` with nothing reserved and no AUD taken.
  - A sweep whose second level would move the price more than 10 % trips the breaker: the first level's fill stands, the remainder is suspended (`rest`) or cancelled with the `CIRCUIT_BREAKER` sentence (`fill_and_cancel`).
- **Nothing to tell (compliance, tipping-off).**
  - An ask is cancelled by operations after a surveillance hit, and the mock includes `cancel_reason: "OPS"` plus planted `ops_reason: "S2 wash trade"` and `aml_case: "AML-131"`. The status page must read "Cancelled by RealX · tokens released" and nothing else.
  - The surveillance list must be unreachable to an investor, a sponsor and a partner session (404 through the SDK), and hidden from CFO, LEGL, CEO and read-only staff in the console (a direct URL shows the not-permitted page and no alert data).
  - The blotter must mask both participants until an audited unmask.
  - INV-07's halt history must show only A48's public cause, never a surveillance scenario.
- **Agent misuse (D04, G22, maker-checker).** Caps follow A91 §9: units, and the value at the limit.
  - An investor token with `market:order.sell` for Marion Street only, a per-action cap of A$5,000 and a daily cap of A$8,000 lists 20 × A$412.00. The gross at the limit is A$8,240, over both caps. It must get `428 CONFIRMATION_REQUIRED`, and the person's INV-05 must show the pending-confirmation row linking to ACC-05, with no order and no units frozen.
  - A token with `per_action_units` 15 lists 20 units inside the cents caps, and must also get `428`.
  - A token with `market:order.buy` and a per-action cap of A$15,150 buys the J07 150 units. The quote's total is A$15,100.13, but the value at the limit (150 × A$100.90 plus its fee, A$15,210.68 as A91 computes it and the mock plants it) is over the cap, so it must get `428`. The screen must not suggest the quoted total is what the cap counts.
  - The same sell token lists Bay Road units and must get `G22_VEHICLE_NOT_ALLOWED`.
  - A listing inside the caps succeeds and shows "Placed by your agent · Sale bot". The person cancels it from the screen.
  - A MKTO agent on ADM-21 drafts a band change and a resume request, both labelled "Draft request". It must see no Approve, Reject, Ratify, Halt instrument, Halt all, Open surveillance case or Cancel order (ops) control. A crafted ratify or decide call with its token must be refused with `APR_CHECKER_MUST_BE_HUMAN` or `G22_AGENT_NEVER_CHECKER`, and a crafted `halt.open` with `AGNT_NOT_ALLOWED`.
  - The same agent calls `POST /v1/market/surveillance/alerts/{id}/case` directly and is refused. It then flags the alert as a draft, which appears to the MKTO with a passkey confirm, and only that confirmation creates the A42 case.
  - A CCO who approves the band draft made by her own agent is refused with `APR_MAKER_IS_CHECKER`, because segregation of duties counts the agent as its human.
- **Joint and entity accounts (SM-09, maker-checker for co-owners).**
  - Joint holders Sam and Priya (`signing_rule = both`) list 30 units. Sam's order shows "Awaiting your co-holder" with the server's 24-hour deadline, and nothing is frozen. Priya sees "Confirm this order", runs the passkey over the same payload hash, and the ask moves to "Checking" then "Listed".
  - A second joint order is left alone and the server clock is advanced past the deadline. It shows the `COSIGN_EXPIRED` sentence with nothing set aside, never a bare "Checking" or "Cancelled".
  - An SMSF member who is not a verified signatory tries to buy and sees the verified-signatory sentence, with no order created.
- **Settlement and proceeds (SM-09 trade subset, A91 §6).**
  - A trade in `settlement_failed` with the server's `attempt` 2 and `max_attempts` 4 shows "2 of 4" on the blotter and "retrying (2 of 4)" to the investor. A planted maximum of 7 shows 7: no count is hard-coded anywhere.
  - "Bust trade" is absent until the server lists it at `retries_exhausted`, and a forced bust before that renders `MKT_BUST_NOT_ELIGIBLE`. The busted copy contains no number of attempts.
  - A trade at `settled` shows "Proceeds of A$9,930.10 arrive when the register updates" and never "paid to your funds available"; the same trade at `registered` shows the paid line.
  - A trade at `settled` with `registration_state = rejected` shows "Register update delayed" with no break id, and polling continues.
- **Every cancellation has words (SM-09).** One fixture each for `BAND_MOVED`, `EXPIRED` (with the listing's expiry date shown beforehand on "Your listings"), `INSTRUMENT_CLOSED`, `HALT_TOO_LONG`, `CIRCUIT_BREAKER`, `SELF_MATCH` (`MKT_SELF_MATCH_PREVENTED`), `COSIGN_EXPIRED`, `OPS` and `USER`, plus an asynchronous `MKT_FREEZE_FAILED` rejection. Each renders its own sentence; none renders a bare "Cancelled".
- **Halt all under pressure (G10, SGL+R).**
  - MKTO fires "Halt all" without an end date. The dialog must not submit, and a forced call must be refused by A48.
  - MKTO opens "Halt instrument" on SBK02 without reopening criteria. It must not submit, and a forced call gets `HALT_CRITERIA_REQUIRED`. With criteria, an end date and the token-pause box ticked, the halt acts at once and an A28 `pause` draft appears for CCO or SECO.
  - "Close instrument" on SBK02 shows the server's open-order count before the request is drafted.
  - With the criteria filled in, every OPEN instrument turns HALTED with the ratification countdown from the server's time. The countdown must survive a reload and a changed browser clock.
  - A meeting-cause halt on PAD01 must never show "Clear halt" or "Resume trading". A forced resume request is refused with `MKT_STILL_HALTED`, and one with a dirty pre-flight with `MKT_RECON_NOT_CLEAR`, each rendered as sent.
  - A circuit-breaker halt must show "reopens {time}" and flip back to OPEN on the first read after the server lifts it, with no client timer changing the state.
- **Every state looks like the design (design fidelity).**
  - Fixtures cover: preview, open, halted (meeting), paused (valuation stale), locked, partially frozen (listed and not available), not on board yet, outside band, listed with its expiry date, part sold, awaiting co-holder, co-sign expired, entity signatory refused, settling, settlement delayed, busted, settled before proceeds, register update delayed, registered, cancelled by you, cancelled by RealX, every other cancellation reason above, freeze failed, rejected, pending agent confirmation, agent-placed, INV-06 with a resting bid, INV-06 with an empty book, INV-07 with no trades, and ADM-21 in default, halted, disabled and empty states, with the halt-instrument and close-instrument dialogs.
  - Each is rendered at 1280 px (1440 px for the console) and at 375 px, and compared by layout tokens with the dc crops (lines 481-501) and A50's and A51's drawings.
  - A hex value outside A05's tokens, depth bars in any colour but green.soft and brick.tint, a second PreviewBanner, a pill outside the TONE map, or a state without a screenshot fails.

## Pass criteria

- **Hidden cases.** The cases in `<workspace>\gauntlet\hidden\A92\` pass 100 % on three consecutive runs:
  - Every planted server figure is rendered as sent.
  - No locally computed money figure appears across 200 random asks and 200 random buys.
  - 100 of 100 out-of-order preview and quote races end on the last edit's figures.
  - Every forced submission leaves exactly the expected count of live orders (0 or 1) and frozen units, and renders the sentence for its code in A91's catalogue. The forced submissions are: outside band, locked, frozen holding, halted, over cap, short of funds, a moved book (fills within the limit, no refusal), a moved band, a second click, two tabs, a borrowed or stale assertion, an unverified entity signatory, a joint order without the co-holder, and preview mode.
- **Preview.** Across INV-05, INV-06, INV-07 and ADM-21 with `resale.board` off, the network probe records zero calls to operations that `openapi.json` marks as mutating (A52's auth and error-reporting endpoints allow-listed) in 50 of 50 scripted sessions. Exactly one PreviewBanner is present per page, with line 482's sentence on INV-05. Every instrument reads DISABLED.
- **State coverage.**
  - All 13 SM-09 states (order and trade subsets), `awaiting_cosign`, `registration_state = rejected`, every cancellation reason, the four instrument states and `disabled` render from fixtures, each with a pill or row treatment and a screenshot at both widths in `app/evidence/A92/`.
  - Settlement attempts and their maximum are always the server's; the busted copy contains no count; "paid to your funds available" appears only at `registered`.
  - `listed_units` and `unavailable_units` render as sent and, in the test, add up to the server's frozen figure.
  - `status.ts` matches A91's vocabulary in a test.
  - "Complete" appears only for `registered`.
  - In the ops-cancel, frozen and surveillance fixtures, no reason string, case id or scenario id reaches an investor DOM.
- **Journey.** J07 steps 1, 3, 4, 5 and 7 pass at screen level in Playwright against the Docker stack three runs in a row:
  - The seller's proceeds read A$9,930.10.
  - The buyer's total reads A$15,100.13, or whatever A91's oracle fixes, rendered as sent.
  - The frozen units equal the listed 100.
  - The blotter ends at `registered` only after the registry ack.
  - The setup runs in order (test binding, flag on through A21's `4E` in the test profile, venue mode `bulletin_board`, then list and approve).
  - The planted move above 10 % shows HALTED until `ends_at` on A91's injectable server clock, and OPEN on the first read after the clock is advanced past it.
- **Gate enforcement.** For each of G07, G08, G09, G10, G14, G20 and G21, a test fails the gate and proves three things: the control is inert, the server's text (range, date, notice or binding reason) shows, and a forced call is refused by the server with nothing frozen, reserved or matched.
- **Design fidelity.**
  - Layout-token comparison of `/resale/[code]/sell` for Bay Road against the dc crop at 1280 px shows zero token differences (fonts, colours, radii, key spacings, the band gradient stops and marker).
  - INV-06, INV-07 and ADM-21 match A50's and A51's drawings by the same harness.
  - The seed strings for Bay Road (band A$100.81 to A$136.39, "20 of 120", the four check rows) are exact.
  - axe-core reports zero critical or serious findings on every route and state.
  - Keyboard-only listing, buying and cancelling succeed in Playwright.
- **Parity.**
  - Every route in both `actions.ts` files exists in `openapi.json` exactly as A91 §4 and §9 (and the signed-off A91.1 items) define it, with A91's `market:*` scopes, and `@realx/sdk` exposes its operation. The only `human_only` entries are the passkey confirmations (order confirm, joint co-sign, agent-confirmation confirm), the hardware-key decisions and acts, the ops cancel and the direct surveillance case-open.
  - Each investor operation succeeds, or is refused, identically for a person, a token and an MCP call. A100's harness is the binding check; this node's preview script is a stopgap until A100 exists, and A100 re-runs it.
  - The DOM sweep of ADM-21 as a MKTO agent counts zero decision or act buttons, and every console manifest entry matches an A91 §8 or A48 policy row.
  - G13: the CCO who approves her own agent's draft is refused with `APR_MAKER_IS_CHECKER`.
  - There is no server action, no route handler and no SDK import outside `api.ts` (lint).
- **Coverage and hygiene.**
  - Unit line coverage of both features is at least 85 %. `status.ts`, read sequencing, the idempotency store, the device draft and the error tables are at 100 %.
  - `npm run verify` is green.
  - Only the four owned paths changed, plus loops A52.1, A53.1, A54.1 and A91.1 where needed, with their owners' sign-off in the evidence.
- **Evidence.** `app/evidence/A92.json` and `A92.qa.md` are present and match disk. They list the screenshot set, the seed-string check, the preview probe log, the parity result and the contract hashes compared.

## Human gates touched

- **None crossed.** The screens run against A91 on the local chain, the registry and bank-rail sandboxes, A11's software and hardware-key test authenticators, and a test licence record that exists only in the seed. No real investor, trust account, market licence or mainnet is touched. `resale.board` stays off in every launch profile (A94 asserts it).
- **HG-LEGAL (counsel questions recorded by A21, A91, A35 and A48, not decided here):**
  - Whether RealX may run the bulletin-board facility, and under which licence or ASIC relief (the binding A21 needs before the flag can turn on).
  - The resale fee and who pays it.
  - Whether a secondary buyer must open and acknowledge the PDS and TMD.
  - Whether a holder's agent may list or buy on the board under standing consent (D04 defaults to allowed within mandate caps, with the passkey above them).
  - Whether any resale declaration must be personal.
  - Whether a public market tracker with last price and premium or discount is permitted for a wholesale-only trust, or must sit behind `listing.visibility`.
  - Whether a halt must be notified to ASIC and holders, and within what time.

  Counsel's answers change data and copy. This node's hidden cases are re-run after the change.
- **HG-REVIEW (later).** The founder signs off the visual QA of the four screens, the composed states and specify points (a) and (b) in the P2 preview gallery at [[A94 - P2- end-to-end journeys and launch update]].

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
