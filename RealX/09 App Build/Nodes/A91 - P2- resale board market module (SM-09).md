---
title: "A91 - P2: resale board market module (SM-09)"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am9]
node: A91
milestone: AM9
lane: "P2"
oracle: yes
human_gate: none
---

# A91 - P2: resale board market module (SM-09)

**The guarded resale board as one API module and its worker jobs. There is one instrument per trust, with a venue mode that stays `disabled` until a licence or ASIC relief record binds the `resale.board` flag (G21). Each instrument has a reference valuation and a price band (±15 % by default, set per listing, changed only under `6E`). Asks must clear lock-in (G08) and bids must clear the holder cap (G07). Matching is price-time priority. Each ask places a partial freeze on chain equal to the listed quantity. A buyer passes the same gate set as a primary purchase, and G22 applies when an agent places the ask or bid within its mandate caps. AUD and tokens settle together through the V2 token agent, and a trade becomes `registered` only after the registry acknowledges it. A failed settlement is retried up to three times; after that the trade can only be busted, by MKTO with a CCO checker. Halts come through the halts module, each with published reopening criteria. A circuit breaker halts an instrument for 30 minutes after a move of more than 10 % in an hour. Surveillance scenarios feed AML monitoring. Every investor and staff action has an API and MCP equivalent under the same gates.**

**Build status:** not started. **Milestone:** AM9 (P2 features, from about 8 April 2027). **Lane:** P2. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 12-14 agent-hours (build, challenge and verify loops).

**Depends on:** [[A24 - ERC-3643 suite and RealX compliance modules]], [[A28 - Token operations service (mint, burn, freeze, pause, forced transfer, recovery)]], [[A19 - Rules engine- gates G01-G22, rule packs, threshold monitors, simulator]], [[A20 - Three-way reconciliation engine with zero tolerance]], [[A21 - Licences, feature flags and licence perimeter (G21)]], [[A18 - Unit register (legal record) and registry sandbox adapter]], [[A48 - Halts, kill switches and wind-down]]
**Unlocks:** [[A92 - P2- resale board screens (sell, buy on the board, market tracker, board control)]], [[A93 - P2- governance votes and meetings (SM-10) with screens]], [[A97 - P3- minors, development and debt vehicles, licensed-market venue mode]]

**Graph dependency gap (open; the graph owner must fix it, this note does not edit the graph).** This spec relies on nodes that are neither direct dependencies nor ancestors of A91 in `final-graph.json`:
- A42: `restrictionStatus` as G14, the surveillance intake and case creation.
- A43: notices and confirmations.
- A37: the confirmation and standing-consent shape, and the primary-order verdicts that the gate matrix compares with resale bids.
- A90: G05 target market and the TMD for P2 retail buyers. A90 is in the same milestone (AM9), so without an edge it could land after A91.
- A101: the per-token, per-user and per-tenant agent kill switches and the S7 counters.
- A104: the MCP tool catalogue named in the agent-parity pass criterion.
- A23: the Docker stack and Hardhat chain the oracle runs on.
- A98: the valuer and property-manager assignments used by S5.
- A47: the read models that consume this module's events.

The graph must add A23, A37, A42, A43, A90, A101 and A104 to A91's `deps`, then regenerate this Depends-on line. A98 and A47 should be added too. If they are not, versioned fixture providers behind their `index.ts` contracts stand in for them, and the S5 and activity criteria are re-run when those nodes pass. Until the graph is fixed, the Specify step confirms that A23, A37, A42, A43, A90, A101 and A104 have each passed their gate before Build starts. If any has not, Build waits. G05, G12, G14, the kill switches and MCP parity cannot be faked in the oracle.

**Oracle rule:** The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A91\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder, and the cases are published into `app/` only after the gate passes. The oracle has three required invariants. First, an ask or bid outside the band is rejected, and the rejection states the allowed range. Second, a circuit-breaker halt fires on the first trade that would breach the one-hour reference. It lifts at the first tick at or after `fired_at + 30 min`, never early and at most 60 s late. Third, no trade is `registered` without the registry acknowledgement. The oracle also covers the money, units and authority arithmetic listed in the Gauntlet loop section below.

In one sentence, this node lets holders sell to other eligible holders inside rules that stop the board from becoming an unlicensed market, a wash-trading venue or a source of drift between the register and the chain. It draws no screen. INV-05, INV-06, INV-07 and ADM-21 are built by A92 on this module's API. Everything here stays switched off in P1: the flag is unbound, every instrument is `disabled`, and the P1 sell preview reads this module's preview endpoint and never creates an order. This note is not legal advice. Every threshold below is a data row marked `set_with_counsel`: the band, the lock-in, the circuit-breaker size and duration, the settlement order, insider windows, fees and what happens to trades matched before a halt. Counsel's answers change rows, not code.

## Build brief

This module is `market` in the Fastify monolith (`services/api/src/modules/market/`), and its worker jobs live in `services/worker/src/jobs/market/`, registered through A09's job registry. Every rule is enforced in the API, never only in a screen. Every read and write contract is in the OpenAPI document from the first commit, so `packages/sdk` and the MCP tool catalogue (A104) see it before the module passes. The only source of time is A09's injectable clock. All money is integer cents and all quantities are whole units, using A02's money math (`mulBp`, `roundHalfUp`). No float appears anywhere in the matching, band, fee or breaker arithmetic.

**1. What this node reuses, and the upstream rows it requests before the build**

- **Reused unchanged:**
  - A04's `instrument`, `resale_order` and `trade` tables and their SM-09 enums, which already exist with RLS.
  - A19's `evaluateAction`, the evaluators G01-G22, `simulate` and `GateEvaluation` writer.
  - A48's `haltState`, `haltNotice()`, `openSystemHalt`, `clearSystemHalt` and `POST /v1/halts/batch`.
  - A20's `preflight({vehicleId, purpose: 'board_open'})`, `assertClear(vehicleId, 'board_trade')` and `isHalted`.
  - A21's `evaluateG21({activity: 'operate_market'})`, `isEnabled('resale.board', ctx)` and `recordExemptionEvent()`.
  - A18's `POST /register/{vehicleId}/instructions` (kind `transfer`, `reference = trade_id`), its ack event and its cap-table and lot reads.
  - A28's operation API, with the `broadcast` route open to the `SVCA` pipeline principal.
  - A24's `SuiteInspector.explainTransfer` and `holdingState`, through A27's chain adapter.
  - A14's request, precheck and effector registries.
  - A12's `agentAuth`, the G22 evaluator and the mandate signing-key confirmation.
  - A16's `reserve`, `release`, `consume` and `postJournal`.
  - A42's monitoring intake (`source = surveillance`) and `restrictionStatus`.
  - A13's `emit` and clocks, A10's `audit.record` and A43's outbox contract.
- **Additive tables owned here.** These are registered through A04's module-schema registry and carry `tenant_id`, RLS and enum states like every core table:
  - `instrument_band_version`, insert-only: the band history as "band v+1".
  - `market_sequence`: a per-instrument monotonic sequence for price-time priority.
  - `resale_freeze`: one row per ask per freeze or unfreeze op, with the frozen units per reason.
  - `circuit_breaker_event`.
  - `surveillance_hit`.
  - `market_tracker_day`: the daily roll-up for INV-07.
- **Upstream data rows requested as follow-up loops.** Each is requested before this node's Build step, following each owner's own rule that a missing row is a spec defect reopened upstream and never patched here. Each is a data change with the owner's tests re-run. None is code in this node's paths.
  - **A03.x.** SM-09 is split into an order subset and a trade subset.
    - The order subset (`new_order`, `validating`, `rejected`, `open`, `partially_filled`, `filled`, `cancelled`, `suspended`) is applied to `resale_order`.
    - The trade subset (`filled`, `settling`, `settled`, `settlement_failed`, `busted`, `registered`) is applied to `trade`. A trade is born `filled` at the match.
    - Five order and trade extensions are added to `extensions.ts` with this node as the source:
      - `partially_filled → cancelled` (remainder cancelled);
      - `partially_filled → suspended` and `suspended → partially_filled` (halt and lift with a remainder resting);
      - `new_order → cancelled` with reason `COSIGN_EXPIRED` (a `signing_rule = both` order whose second assertion did not arrive within 24 hours) or `CANCELLED_BY_HOLDER` (§4);
      - trade `filled → settlement_failed` with reason `HALT_WAIT_EXCEEDED` (§6).
    - The instrument lifecycle is added as its own table, because SM-09 names it only in its notes. It has seven transitions: `pre_open → open`, `open → halted`, `halted → open`, `open → closed`, `halted → closed`, `pre_open → closed`, and `closed → pre_open` (relisted under a new list request).
    - `retries_exhausted` is a flag on a `settlement_failed` trade, not a state.
    - The scope cells for `market:*` are added to the matrix and to `delegable.ts`.
  - **A19.1.** The single `resale` gate set is split into three:
    - `resale_ask` = G22? G01 G02 G08 G09 G10 G14 G21.
    - `resale_bid` = G22? G01 G02 G03 G04 G05 G06 G07 G09 G10 G12 G14 G16 G21. This is exactly the primary `order` set with G11 (offer caps and escrow) replaced by G09 and G10, so buyer eligibility is identical to a primary purchase by construction.
    - `resale_settle` = G01 G02 G06 G07 G10 G14 for the buyer, and G02 G10 G14 for the seller. The seller's G14 reads A42's `restrictionStatus`.
    - In the `resale_settle` context, G10 takes the parameter `settle_through_causes` (default `[circuit_breaker]`, `set_with_counsel`). G10 passes for settlement when every open halt on the vehicle has a cause in that list, and it returns `halted` with the blocking cause otherwise (§6, spec point (e)).
    - The P2 G09 evaluator also gets the inward rounding rule in §3.
    - For `resale_bid` placed by an agent, G12 follows A37's standing-consent rule (§4). G05 target-market answers stay non-delegable (§4, HG-LEGAL).
  - **A28.1.** Three catalogue rows, all on V2 under policy `AUTO` with the `SVCA` principal as initiator:
    - `resale_freeze` (`freezePartialTokens`), whose quantity equals the ask quantity and is bound to the order id.
    - `resale_unfreeze` (`unfreezePartialTokens`), whose quantity is at most the units this ask froze minus the units it sold, bound to the order id or the bust approval id.
    - `resale_settle` (`forcedTransfer(seller, buyer, qty)`), whose quantity equals the trade quantity and whose destination must be the buyer's registered wallet, bound to the trade id. It carries no timelock and no register effect in A28, because this module submits the register instruction itself (§6).
  - **A16.1.**
    - Account type `resale_settlement`: one per vehicle, a liability, client money.
    - Reservation purpose `resale_bid`.
    - Journal types:
      - `resale.lock`: `investor_cash` (buyer) / `resale_settlement`. It consumes the bid reservation for the trade's gross plus the buyer fee.
      - `resale.release`: `resale_settlement` / `investor_cash` (seller) + `fee_payable_realx`. This row has three legs, and its fee leg carries both the buyer fee and the seller fee.
      - `resale.unwind`: `resale_settlement` / `investor_cash` (buyer). It is posted only under an approved `market.trade.bust`.
  - **A48.1.** Two system causes, both off-chain only (`chain_state = not_required`) and scoped to `resale_board`:
    - `circuit_breaker`: the criterion is `system:market.cooldown_elapsed`, the end is 30 minutes, and this module is the source.
    - `perimeter`: the criterion is "`resale.board` on with a valid licence binding (G21 passes)", with the system default end date. This module is the source.
  - **A25.1 (proposed, strongly recommended).** The vendored `forcedTransfer` skips `canTransfer`, as A24 records. A settlement tagged `RESALE` should therefore assert `compliance.canTransfer(seller, buyer, qty)` inside the gateway before it forwards the call, so that the chain is the last line of defence, as the architecture requires. If the user declines this loop, the gap is recorded in `A91.qa.md`, and the off-chain precheck in §6 remains the only guard.
  - **A18.1 (requested).** A public read `freezeFactBySourceOp(vehicleId, sourceTokenOpId)` over `register_freeze_fact`, plus the matching field on the existing `register.changed` outbox event. Together they let this module see that the register mirror has recorded the frozen units that a settlement consumed before it prepares the transfer instruction (§6). If A18 already exposes an equivalent, that is used instead. This module never reads A18's tables directly.
  - **A14.** The policy rows in §8.

**2. Instruments, venue mode and listing (M14, G21, G20, G08)**

- `instrument` row fields:
  - `instrument_id` (`INS-<vehicle code>`, one per vehicle; the product map's `PAD01`, `SBK02` and `WLG01` are seed codes).
  - `vehicle_id`, `token_id`.
  - `venue_mode` (`disabled` | `bulletin_board` | `licensed_partner`). `licensed_partner` is refused with `PHASE_NOT_ENABLED` until A97.
  - `venue_licence_binding_id` (A21 `flag_binding`).
  - `state` (`pre_open`, `open`, `halted`, `closed`).
  - `reference_valuation_id`, `reference_price_cents`, `reference_valued_at`.
  - `band_lo_bp`, `band_hi_bp` (default 1,500 each), `band_version`.
  - `tick_cents` (1), `min_units` (1).
  - `holder_cap_bps`, read from the offering and never copied.
  - `order_max_days` (default 30).
  - `last_price_cents`, `last_trade_at`.
  - `halt_summary`, derived and never stored as the source.
- The launch state, seeded and asserted: every instrument is `pre_open` with `venue_mode = disabled`. The ADM-21 pill is `DISABLED` whenever `venue_mode = disabled`, whatever the state.
- **Change venue mode.** This is action `market.venue_mode.change` (`4E`, CCO → LEGL). The A14 precheck refuses the change in four cases:
  - `G21_NO_BINDING`: A21's `evaluateG21({activity: 'operate_market', vehicleId})` fails, or `resale.board` is off for the vehicle's scope.
  - `G21_PHASE_NOT_REACHED`: `PHASE` is earlier than P2.
  - `MKT_OPEN_ORDERS`: the mode would move from `bulletin_board` to `disabled` while open orders exist. The ops cancel is needed first, and the precheck names it.
  - `MKT_NO_LIST_REQUEST`: the vehicle has no approved list request.
- **List instrument.** This is `market.instrument.list` (`4E`, MKTO → CCO). The product map's gates are all checked:
  - Fully funded: the offering is finalised and minted (A36 SM-04 at `closed` or later, and A28 SM-06 `active`).
  - Valuation done: A19 G20 passes, with the latest published valuation inside `valuation.max_age_days`.
  - The listing's primary lock-in has elapsed for at least one lot, or the list request states the first date on which it will. Before that date the instrument stays `pre_open`.
  - G21 passes.
  - A20's `preflight({purpose: 'board_open'})` returns `clear: true`.
  - A failure names the gate and the fix.
- **Open.** `pre_open → open` runs when the list request is approved, the venue mode is `bulletin_board`, no halt is reported by `haltState` or A20, and a fresh `board_open` pre-flight is clean. That pre-flight runs again at every resume and after every nightly 02:00 run (a nightly break already halts through A20).
- **Close.** `market.instrument.close` (`4E`, MKTO → CCO) runs, and so does an automatic close on A48's wind-down `sold` event, on A93's exit execution and on venue mode set to `disabled`. Closing cancels every open and suspended order with reason `INSTRUMENT_CLOSED`, which unfreezes the asks and releases the bids. `closed → pre_open` needs a new list request.
- **Perimeter lapse.** When A21 auto-reverts `resale.board` because a binding lapsed, the flag event makes this module call `openSystemHalt` with cause `perimeter` on every open instrument within one tick. G10 then reads `halted`, and open orders become `suspended`. The halt clears only when G21 passes again and a human resume is approved. Trades already `filled` follow §6's halt rule. `perimeter` is not in `settle_through_causes`, so they wait in `filled` and, if the lapse outlasts `market.settlement_wait_max`, go down the bust path. Whether they may settle instead is a counsel question recorded under HG-LEGAL.

**3. Reference valuation and the price band (G09)**

- The reference is the latest valuation that A34 published through its `6E` committee. On A34's `valuation.published` event, the module updates `reference_*` and inserts a new `instrument_band_version` with the same bp values. This is J13 step 5, "the band recalculates", and it needs no separate approval because the committee already approved the number.
- **Band arithmetic.** All values are integer cents, and the band's end points are inclusive.
  - `lo = ceil(ref_cents × (10,000 − band_lo_bp) / 10,000)`.
  - `hi = floor(ref_cents × (10,000 + band_hi_bp) / 10,000)`.
  - The band therefore never extends past the true band. A price must also be a multiple of `tick_cents`.
  - Vectors: PAD01 at $102.40 gives $87.04 – $117.76. The design's fully funded trusts give Marion Street $412.00 → $350.20 – $473.80, Bay Road $118.60 → $100.81 – $136.39, Hawthorn Grove $168.80 → $143.48 – $194.12, and Gympie Road $296.00 → $251.60 – $340.40.
  - The rule is supplied to A19's P2 G09 evaluator (A19.1), so the screens, the design's PriceBand widget and this module use one formula.
- A rejection outside the band is `MKT_PRICE_OUTSIDE_BAND`, with A19's G09 verdict carrying `{lo_cents, hi_cents, reference_cents, reference_valued_at, band_version}`. The plain-English fix-it is "Price must be between A$87.04 and A$117.76 (85 %–115 % of the June valuation)". The order ends `rejected`, with no freeze and no reservation.
- **Set band or floor.** This is `market.band.set` (`6E`, MKTO → CCO + CFO), with `band_lo_bp` and `band_hi_bp` each in 1 to 3,000 bp (the ceiling is `set_with_counsel`). The product map's floor-style setting, such as BrickX's 80 %, is just `band_lo_bp = 2000`. The effector inserts band v+1.
- **Band moves.** A new band version, whether from a band change or from a new valuation, cancels every resting order outside the new band in the same transaction, with reason `BAND_MOVED`. Asks are unfrozen, bids are released and both parties are notified through A43. A resting order is never silently repriced.
- **Stale valuation.** When G20 fails for the vehicle, A34 opens a `valuation_stale` halt through A48, and new orders are refused by G10. This module adds no second staleness rule.

**4. Orders: entry, validation and the book (SM-09 order subset, G07, G08, G22)**

- **Order entry.** `POST /v1/market/instruments/{id}/orders` takes `{side: sell | buy, units, limit_price_cents, time_in_force: rest | fill_and_cancel, acknowledgements[], confirmation}` and requires an `Idempotency-Key`.
  - Asks are always `rest`.
  - There are no market orders: every bid carries a limit inside the band. INV-06's "Buy now" sets the limit to the price of the last ask the quote needs.
  - `confirmation` has the same shape as A37's: a fresh passkey assertion, or a mandate signature over the canonical payload including the idempotency key.
  - Joint accounts with `signing_rule = both` need the second holder's assertion over the same payload hash within 24 hours. Until then the order stays `new_order` with `awaiting_cosign`. It has no gate evaluation beyond G22, no freeze and no reservation, and it holds no priority. The job `market.cosign_expiry` (every minute) moves an order still awaiting after 24 hours to `cancelled` with `COSIGN_EXPIRED` and notifies both holders. Either holder may cancel it earlier (`CANCELLED_BY_HOLDER`). Entity accounts need a verified signatory (A30's `is_verified_signatory`). Where the entity's rule needs two signatories, the order uses the same `awaiting_cosign` path.
  - **Orders placed by an agent (D04).**
    - Joint and entity signing rules apply unchanged. An agent order on a `signing_rule = both` account, or on an entity that needs two signatories, waits in `awaiting_cosign` until the other holder or signatory gives a passkey assertion over the payload hash. A mandate signature never counts as that second person.
    - G12 (disclosure) passes for an agent bid only through the mandate's standing disclosure consent, and only when it covers the current PDS or IM and TMD versions for the vehicle and A37's `orders.agent_standing_consent` parameter allows it for the regime. This is A37's rule, reused unchanged. Otherwise the route returns `428 CONFIRMATION_REQUIRED` with `documents_required`, and creates no order, freeze or reservation. The acknowledgements the person gives with the passkey confirmation are recorded as A37's `disclosure_acknowledgement` rows.
    - G05 target-market answers for a P2 retail buyer (A90) are non-delegable until counsel answers. They stay `"counsel"` in A03's `delegable.ts`. An agent bid from a buyer with no current, personally given target-market answers is refused with A12's `G22_SCOPE_PENDING_COUNSEL` naming G05. An agent bid from a buyer whose answers are current is evaluated by G05 like any other bid.
  - In P1, or while the flag is off, the route returns `MKT_BOARD_NOT_OPEN` (G21 verdict attached) and writes no order row.
- **Validation (`new_order → validating → open | rejected`).** A19 runs the `resale_ask` or `resale_bid` gate set as one evaluation set. G22 runs first for agent principals.
  - **Ask.**
    - The sellable units are computed from A18's lots and the chain. Sellable = units in lots whose `lock_in_until ≤ now`, minus the chain's `getFrozenTokens(seller)` (every reason: lock-up, legal freeze and other open asks). The result must be at least the requested units.
    - A shortfall caused by lock-in is `MKT_LOCKED`. The G08 verdict gives the earliest date on which the requested quantity becomes free and the units unlocked today.
    - Any other shortfall is `MKT_INSUFFICIENT_UNLOCKED`.
    - A holding in SM-07 `frozen` is `MKT_HOLDING_FROZEN`, rendered to the investor as `under_review` with no reason. The tipping-off rule comes from A42.
    - G07 is recorded as `not_applicable` with reason `REDUCES_HOLDING`, so every order has a complete gate row set.
    - After the gates pass, the module creates a `resale_freeze` row and asks A28 for a `resale_freeze` op for exactly `units`. The ask becomes `open`, and enters the book with its sequence number, only when A28 reports the freeze `confirmed`. SM-07 moves `active → partially_frozen`.
    - A freeze that fails or reverts ends the order `rejected` with `MKT_FREEZE_FAILED`. A freeze still unconfirmed after 10 minutes raises a Sev3 alert through the `market.freeze_watch` job.
  - **Bid.**
    - G07 counts four things: the buyer's register units, associates (A30), the buyer's other open bids on this instrument, and trades that are filled but not yet registered. A failure names `max_allowed_units`. This is INV-06's "quantity is limited" state, and the design's "Holder cap ✓ 1.5 % of 5 %" line comes from the same verdict.
    - **One bid amount, used three times.** `bid_amount = L + buyFee(L)`, where `L = units × limit_price_cents` and `buyFee(x) = roundHalfUp(x × buy_bp / 10,000)`. `buy_bp` is snapshotted on the order at acceptance and used for every fill of that order. The same `bid_amount` is:
      - the funds test: the buyer must hold available AUD of at least `bid_amount`, otherwise `MKT_INSUFFICIENT_FUNDS`, naming the available and needed amounts;
      - the A16 reservation: `reserve(investor_cash, purpose = resale_bid, amount = bid_amount)`, in the same transaction as `validating → open`. There is no bank pull, because board purchases are funded from the AUD wallet;
      - the G22 cap value for an agent bid (§9).
    - The buyer fee is charged cumulatively per order, so the sum of the locks can never exceed `bid_amount` (§6).
- **Book and matching.**
  - One matching function runs per instrument inside `pg_advisory_xact_lock(instrument)`, so two processes can never interleave on one book. Matching is a pure function, `match(book, incoming, refSet, band) → {trades[], residual, breaker?}`, under `engine/`. Sequence numbers come from `market_sequence` and are assigned at `open`.
  - Priority is best price first, then the lowest sequence. The trade price is the resting order's price. When two resting orders cross, which happens only at a reopen, the price is that of the order with the lower sequence.
  - A partial fill leaves the remainder resting as `partially_filled` if its time in force is `rest`. With `fill_and_cancel` the remainder is cancelled at once.
  - **Self-match prevention:** an incoming order that would match a resting order of the same investor or account (joint members included) is cancelled for the remaining units with `MKT_SELF_MATCH_PREVENTED`. Associates are not blocked, because they are separate legal persons, but they feed surveillance scenario S1.
  - **Reservation after each fill.** After each fill, the bid's remaining reservation is set to exactly `required = R × limit + buyFee(G_filled + R × limit) − fees_charged`. Here `R` is the remaining units, `G_filled` is the gross filled so far on the order and `fees_charged` is the buyer fee charged so far. Any excess (the price improvement of a fill below the limit) is released at the fill. `buyFee` never decreases as its input grows, so `required` is never negative, and the locks plus `required` never exceed `bid_amount`.
- **Quotes and previews.** Both are read-only and write no rows, except their own G-evaluation with `dry_run = true` through A19's `simulate`.
  - `GET /v1/market/instruments/{id}/quote?side=buy&units=` returns the sweep from the cheapest ask upward: fills, average price, subtotal, per-fill fees, total, `max_units_by_cap`, `max_units_by_funds` and the gate rows.
    - Vector from the product map's INV-06 book: asks $99.80 × 100, $100.90 × 120 and $101.50 × 40. A quote for 150 units fills 100 @ $99.80 and 50 @ $100.90, for a subtotal of A$15,025.00. The fees use the cumulative rule in §6: `buyFee(9,980.00) = 49.90`, then `buyFee(15,025.00) − 49.90 = 75.13 − 49.90 = 25.23`, so A$75.13 in total. The total is A$15,100.13 and the average is $100.17.
  - `GET /v1/market/instruments/{id}/sell-preview?units=&price_cents=` returns the rest of INV-05's data:
    - held, unlocked and locked units, with the unlock date;
    - frozen units;
    - the valuation and its date;
    - the band `lo` and `hi` and the marker position `(price − lo)/(hi − lo)` in bp;
    - fee bp and proceeds;
    - the aggregated book with the floor marked;
    - the check rows G08, G10, G09 and "Board status", whose strings come from A19's verdicts and A48's `haltNotice()`.
    - Vector: 100 × $99.80 gives gross A$9,980.00, fee A$49.90 and proceeds A$9,930.10.
    - In P1 the "Board status" row is `preview` with detail "Opens with retail launch" and tone `brick`.
- **Cancel, expiry and suspension.**
  - `POST /v1/market/orders/{id}/cancel` (by the investor or their agent, no confirmation needed because cancelling is protective) moves `open | partially_filled | suspended → cancelled`. The order leaves the book in the same transaction. An ask's remaining frozen units are released by a `resale_unfreeze` op, and SM-07 `partially_frozen → active` follows when it is confirmed. A bid's remaining reservation is released.
  - An ops cancel (MKTO with a reason code) uses the same route with the staff scope.
  - `market.order_expiry` (daily 00:05 Sydney) cancels orders older than `order_max_days` with reason `EXPIRED`.
  - `open | partially_filled → suspended` runs when G10 turns `halted` for the instrument. The instrument goes `open → halted` at the same time, and no matching runs. On lift the reverse transition applies, with the original priority kept.
  - `market.suspended_sweep` cancels suspended orders when a halt has lasted longer than `market.suspended_max_days` (default 7, `set_with_counsel`), with reason `HALT_TOO_LONG` and a notice to both parties.
  - **Unfreeze rule.** The unfreeze for an ask never exceeds the units that ask froze minus the units it sold, which is checked against `resale_freeze`. The chain's frozen amount after the op must equal the expected sum per holder, and A20's R3 checks this. If the seller's address is in SM-07 `frozen` (sanctions or court), the ask still leaves the book, but the partial unfreeze is not submitted. It waits for the four-eyes unfreeze (MLRO → CCO in A28), and the job `market.freeze_reconcile` then releases it. The SM-07 invariant "unfreezing always needs four eyes" therefore holds for compliance freezes. A listing freeze is released only by the AUTO reversal of the AUTO act that created it, with the same order id: this is the A28 `unlock` precedent, and it is spec point (c) below.

**5. Halts and the circuit breaker (G10)**

- **Halt sources.** This module never keeps its own halt truth. G10 for an instrument is A19's union of three sources: A48's `haltState(vehicleId)` for the causes manual, meeting, valuation stale, token pause, distress, legal hold, kill switch, circuit breaker and perimeter; A20's `isHalted`; and the platform `marketplace_suspend` switch.
  - The module subscribes to `halt.opened`, `halt.cleared`, `recon.halt.set`, `recon.halt.cleared` and the kill-switch events, and it re-reads `haltState` inside every order and match transaction, so a missed event cannot open a halted book.
  - Every halt carries its cause, criteria and expected end, which the book, the quote and the preview return verbatim from `haltNotice()`.
- **Manual halt from ADM-21.** This is A48's `halt.open` (`SGL+R`: MKTO or CCO acts with a hardware key, and the other of CCO / CEO ratifies within 2 hours). "Halt all" is A48's `POST /v1/halts/batch`. Optionally the request drafts an A28 `pause` for the CCO or SECO to act on (the product map's "optional token pause"). This module does not re-implement either.
- **Resume.** `halted → open` for every cause except `circuit_breaker` is `market.instrument.resume` (`4E`, MKTO → CCO). It is refused with `MKT_STILL_HALTED` while any halt source reports `halted` (the halt itself clears through A48's rows or the source module), and refused with `MKT_RECON_NOT_CLEAR` unless a fresh `board_open` pre-flight is clean.
- **Circuit breaker.** A move of more than 10 % in one hour halts the instrument for 30 minutes. The product map says "a move above 10 %", so either direction counts, and the graph summary's "+10 %" is read as the size of the move.
  - The reference set at time t is the price of the last trade at or before t − 60 minutes, plus every trade price in (t − 60 minutes, t]. If the instrument has never traded, the reference set is the reference valuation.
  - A would-be trade at price p breaches when `10 × (p − min(ref)) > min(ref)` or `10 × (max(ref) − p) > max(ref)`, in integer cents. Exactly 10.00 % does not breach.
  - Vectors with reference $100.00: $110.00 passes and $110.01 halts; $90.00 passes and $89.99 halts. A 10 % move can happen inside the ±15 % band, so both rules are needed.
  - The check is pre-trade: the breaching fill never prints, and fills at earlier price levels in the same sweep stand.
  - In the same transaction the module:
    - writes a `circuit_breaker_event` (the would-be price, the reference set and its hash, the fire time);
    - calls A48's `openSystemHalt` with cause `circuit_breaker`, `ends_at = fired_at + 30 minutes` and criterion `system:market.cooldown_elapsed`;
    - moves the instrument to `halted` and resting orders to `suspended`;
    - rests the incoming order's remainder as `suspended` if its time in force is `rest`, or cancels it with `CIRCUIT_BREAKER` if it is `fill_and_cancel`;
    - emits `market.circuit_breaker.fired`.
  - **Lift.** The job `market.circuit_breaker_release` is scheduled with `startAfter = ends_at` and also swept every minute. At the first tick at or after `ends_at`, and never before it, it sends A48 the clear event. The instrument returns `halted → open` automatically, orders reopen with their original priority, and the reference window restarts from the last pre-halt trade price. No human resume is needed for this cause, because the product map fixes the lift. If another halt source is open at the time, the instrument stays `halted` and needs the ordinary resume.
  - The breaker halt is off-chain only (A48.1). The board is the only venue that matches trades, and every settlement goes through §6's G10 check. Clearing an on-chain HaltModule every 30 minutes would put a V2 quorum on the hot path. This is spec point (d).

**6. Settlement: AUD and tokens together, then the register (SM-09 trade subset, SM-07)**

- **At the match.** In the same transaction as the match, the trade is created `filled` (`TRD-<yyyymmdd>-<6 base32>`, with `buy_order_id`, `sell_order_id`, `units`, `price_cents`, `gross_cents`, `buyer_fee_cents` and `seller_fee_cents`).
  - **Fees.** `fee.resale.buy_bp` and `fee.resale.sell_bp` both default to 50 bp, as the design's fee page and the product-map vectors show. They are fee-schedule rows with effective dates, and each order snapshots its bp at acceptance.
    - The buyer fee is cumulative per buy order: `buyer_fee_cents = buyFee(G_after) − buyFee(G_before)`, where `G_before` and `G_after` are the order's total filled gross before and after this trade, and `buyFee` uses `mulBp(…, 'half_up')`. The sum of an order's buyer fees therefore equals `buyFee(total filled gross)`, which is at most `buyFee(L)`. A busted trade's gross stays in the order's running total, so the total never decreases.
    - Vector: a bid of 2 units at $1.01 at 50 bp reserves 202 + 1 = 203 cents. Filled as two 1-unit trades, the fees are 1 cent and then 0 cents, so the locks are 102 + 101 = 203 cents and never more. Charging each fill separately would have locked 204 cents.
    - The seller fee is per trade, `mulBp(gross, sell_bp, 'half_up')`. It is deducted from the proceeds and needs no reservation.
  - A16's `resale.lock` posts the buyer's gross plus buyer fee from `investor_cash` into the vehicle's `resale_settlement` account, consuming that part of the bid reservation. A lock larger than the remaining reservation is refused by A16 and is a Critical defect, not a retry.
  - From this moment the buyer's money cannot be spent elsewhere and the seller's units are frozen on chain, so neither side can move without the other.
- **`market.settle` job.** It runs every 15 seconds and immediately on the `market.trade.filled` event. It takes one trade at a time per instrument, in trade order, under the SVCA principal.
  - **Prechecks.** Each is recorded as a new evaluation set that references the order's set. They fall into two groups.
  - **Wait prechecks.** A failure means the trade cannot settle yet. The trade stays in `filled`, or in `settlement_failed` if it is waiting for a retry. No attempt is counted, no approval id is spent and nothing is sent to custody or the registry. It is re-checked on every `market.settle` tick. A wait longer than `market.settlement_wait_max` leads to `HALT_WAIT_EXCEEDED` and the bust path (see "Trades matched before a halt" below). The wait prechecks are:
    - G10 under `settle_through_causes` (spec point (e));
    - A20 `assertClear(vehicleId, 'board_trade')`;
    - A28 token state `active` (not paused);
    - **seller not frozen:** the seller's holding is not in SM-07 `frozen`, A18's current `register_freeze_fact` for the seller is not a `full` freeze, and any partial legal freeze is held on top of this order's listing freeze. That means the chain's frozen amount is at least the legal freeze plus this ask's unsold listed units, so the trade consumes listing units only;
    - **seller not restricted:** A42 `restrictionStatus(seller)` is not `full`. A `payouts_held` restriction does not stop settlement. The seller's proceeds are credited at `registered` as usual and held by A42's restriction from withdrawal, which is A42's rule, and nothing in the seller's trade responses shows why;
    - the buyer's G14 and G02 (sanctions), which may clear after a review.
  - **Fail prechecks.** A failure is a failed attempt, as in "Failure, retry and bust" below. The fail prechecks are:
    - the rest of the A19 `resale_settle` gates for both sides (G01, G06, G07);
    - A24 `explainTransfer(seller, buyer, units)` must return every module row `ok`, simulated with the listing freeze excluded, as the frozen units will be consumed. Under the default `settle_through_causes = [circuit_breaker]`, the only halt that lets settlement run is off-chain (A48.1), so HaltModule is not set and needs no exclusion;
    - the buyer's wallet must be identity-registered;
    - the seller's chain frozen amount must be at least the trade units.
  - A seller who is frozen by a court or for sanctions after the match is never moved by V2. There is no chain op, no register instruction and so no `REG_HOLDER_FROZEN` break. The trade waits and, if the freeze outlasts `market.settlement_wait_max`, the buyer is refunded by the bust. The listing freeze on the seller's units is then released only through the four-eyes route in §4's unfreeze rule.
  - **Settlement op.** If the prechecks pass, the trade moves `filled → settling`. The job creates the A14 `AUTO` request `market.trade.settle` (object = trade id; attested cap = trade units; destination = buyer wallet), and A28 runs the `resale_settle` op through A15 (V2 policy co-signers under AUTO, P4 destination guard, P5 amount band equal to the trade units) and broadcasts it.
  - **On chain confirmation.** When A28 reports `confirmed` (the indexer's `Executed(approvalId)`, in which the vendored `forcedTransfer` consumes the frozen units), one transaction:
    - moves the trade `settling → settled`;
    - marks the ask's `resale_freeze` units as consumed;
    - sets `registration_state = awaiting_mirror`;
    - records the A21 exemption `entry` event for the buyer's gross, with a reversal on bust (counsel question).
  - **Register hand-off.** The register mirror is updated asynchronously by A28 and A18 handlers that this node does not own, so the A18 `transfer` instruction is not prepared in the confirmation transaction. The module waits until A18 reports, through the A18.1 read or the `register.changed` event, a seller freeze fact whose `source_token_op_id` is this trade's settlement op, meaning the consumed units are no longer frozen in the register. Only then does it prepare and submit the instruction (lines: seller → buyer, units, price, `reference = trade_id`, `approval_id` = the settlement request) and set `registration_state = submitted`. A18's "enough unfrozen units" rule therefore holds without an exception and without reading stale data.
    - The wait is driven by the event and by the `market.register_handoff` job every 30 seconds, which re-reads A18.
    - If the mirror has not caught up within `market.register_mirror_timeout` (default 10 minutes), a Sev2 `market.register_mirror_late` is raised through A13 and an A20 targeted run is requested. The trade stays `settled` with its proceeds in `resale_settlement`. If the delay passes A18's SLA, A20's R17 halts resale as for any register backlog.
    - A `REG_INSUFFICIENT_UNITS` returned after the mirror was confirmed is a real break (R11), not a timing artefact.
  - Tokens bought on the board carry no new lock-in: G08 covers primary purchases, and A18 splits the seller's lots and keeps their past `lock_in_until`.
- **`settled → registered`.** This transition happens only on A18's `register.instruction.acked` event for that instruction id. The same transaction:
  - posts A16's `resale.release` (seller proceeds of gross − seller fee; fees to `fee_payable_realx`);
  - sets `registered_at` from the ack's `effective_at`;
  - emits the confirmations to both holders through A43.
  - Seller proceeds become available only at `registered`. The legal record moves first and the money follows it. This is spec point (b).
  - There is no other writer of `registered`: no admin route, no internal endpoint and no job path. A database trigger refuses `registered` without a non-null `register_instruction_id` whose A18 state is `acked`.
  - A registry rejection after chain confirmation leaves the trade `settled` with `registration_state = rejected`. A20 then opens the Sev1 break (R11) and halts resale for the vehicle, and the money stays in `resale_settlement` until a `6E` register adjustment or a corrective op closes the break. A trade whose tokens moved is never busted (`MKT_BUST_AFTER_CHAIN`).
- **Failure, retry and bust.**
  - A failed attempt of any kind (a fail-precheck failure, policy refusal, revert, or a signed op not confirmed within `market.settle_confirm_timeout`, default 10 minutes) moves the trade `settling → settlement_failed` with the reason code. The approval id is spent, as in A28. A wait-precheck failure is never an attempt.
  - `settlement_failed → settling` is retried by `market.settlement_retry` with backoff 5 minutes, 30 minutes and 2 hours, each time under a new AUTO request. There are at most three retries, so four attempts in total.
  - After the third retry fails, the trade is `retries_exhausted`. A13 raises a Sev2 `market.settlement_exhausted` to MKTO, and the system pre-fills a `market.trade.bust` request as a draft (`4E`, MKTO maker → CCO checker).
  - A bust before retries are exhausted is refused with `MKT_BUST_NOT_ELIGIBLE`.
  - The bust effector runs one transaction: `settlement_failed → busted`; A16 `resale.unwind` returns the buyer's gross plus fee; a `resale_unfreeze` op for the trade units runs under the bust approval id; the sell order records `busted_units` and is not reinstated in the book; and both parties get a plain notice. If the seller's holding is in SM-07 `frozen`, the unfreeze op is not submitted. The units wait for the four-eyes unfreeze and `market.freeze_reconcile`, as in §4. The buyer's refund does not wait for it.
- **Trades matched before a halt (spec point (e)).** The default rule follows the chain. Every halt that A48 mirrors to A24's `HaltModule` (every resale- or all-scope cause except the off-chain-only `circuit_breaker` and `perimeter`) would make `explainTransfer` and the A25.1 `canTransfer` guard refuse the settlement. So a settlement proceeds during a halt only when every open halt cause is in `settle_through_causes`, which defaults to `[circuit_breaker]`.
  - A trade `filled` before a circuit-breaker fire settles, because the breaker is pre-trade and its price was formed before the fire.
  - Under any other halt (`manual`, `meeting`, `valuation_stale`, `recon_break`, `token_pause`, `kill_switch`, `distress`, `legal_hold`, `perimeter`) the trade waits in `filled`, or in `settlement_failed` between retries. The wait counts no attempt. Settlement resumes by itself once no blocking cause is open and A20 is clear, because settlement is not new matching. The instrument's `4E` resume is needed only to reopen matching.
  - A wait longer than `market.settlement_wait_max` (default 5 business days, A13 calendar) moves the trade to `settlement_failed` (`filled → settlement_failed` is an A03.x extension) with `HALT_WAIT_EXCEEDED` and the `retries_exhausted` flag. That opens the bust path, and the bust draft is pre-filled as above.
  - Alternative for the user or counsel at Specify: let `manual`, `meeting` and `valuation_stale` settle through. That needs rows outside this node: A25.1's guard and A28.1's `resale_settle` precheck would ignore the HaltModule result for a `RESALE` settlement when A24's recorded cause is one of those, and A19.1 would widen `settle_through_causes`. It is recorded, not assumed.
- **Invariants checked after every transaction and by the nightly job `market.integrity`:**
  - Money: `resale_settlement` balance for the vehicle = Σ(gross + buyer fee) of the vehicle's trades in `filled`, `settling`, `settled` or `settlement_failed` that are not yet registered or busted.
  - Frozen units: Σ `resale_freeze` units still frozen = Σ remaining units of open, partially filled and suspended asks + Σ units of unsettled trades on the ask side.
  - Holders: a buyer in SM-07 `eligible_empty` becomes `active`, and a seller whose balance reaches zero becomes `eligible_empty`, as reported by the indexer. A mismatch is a Sev1 `market.integrity_break` and halts the instrument through A48 as `manual` with the system actor.
- **The investor's wallet key.** Board settlement needs no UserOp from the investor's wallet, because V2 moves the frozen units. An agent's delegated session key therefore never touches the board, and G22's mandate caps on the API are the control.

**7. Surveillance (G14) feeding AML**

- Scenarios are versioned data rows (`effective_from`, `set_with_counsel`, `score`, `severity`, `dedup_window`) evaluated by `market.surveillance_scan` every 5 minutes and on every trade. The scan is deterministic under the injectable clock, and re-running a window creates nothing new.
  - The codes and labels of S2 and S3 are the product map's ADM-21 rows (`data_admin.py`). The fidelity test uses those codes and trigger texts.
  - **S1 related party:** both sides of a trade are the same person or associates (A30).
  - **S2 wash-trade pattern** (product-map label "wash-trade pattern"): the two sides share a device or session fingerprint (A11) within 48 hours, or trade round trips (A buys from B and B buys back, at least 3 times in 7 days or 6 times in 48 hours). The ADM-21 row text is "same device, 6 round trips / 48h · 0.91".
  - **S3 band rejects** (product-map label "band rejects"): at least 20 band rejects in 1 hour, which suggests demand outside the band and a valuation review. The ADM-21 row text is "27 in 1h (demand above band)".
  - **S4 marking the close:** trades in the 60 minutes before a valuation cut-off or a month-end statement cut that move the last price by at least 3 %.
  - **S5 insider window:** a trade or order by a sponsor-tenant member of the vehicle, their associates, a valuer or property manager assigned to it (A98), or RealX staff with vehicle access, inside the blackout window from 10 business days before a valuation publication or announcement until 1 business day after. The alert is on by default. Whether to block these orders pre-trade (`MKT_INSIDER_BLACKOUT`) is a policy row, default off, and a counsel question.
  - **S6 layering:** an order-to-trade ratio of at least 20 with cancels inside 60 seconds.
  - **S7 agent burst:** many agent-placed orders against the token's baseline, from A12's counters; the same scenario is shared with A42 and A101.
  - **S8:** every circuit-breaker fire.
- Each hit writes a `surveillance_hit` row (scenario, instrument, masked party refs, evidence refs to orders and trades, score) and one A42 monitoring alert (`source = surveillance`, `scenario_code`, `trigger_text` from the template). A42's dedup and need-to-know ACL then apply, and nothing here changes the investor's state.
- **Open surveillance case** (MKTO alone, as the product map has no checker) calls A42's case creation from the alert. The case contents are never readable through this module. `GET /v1/market/surveillance/alerts` returns scenario, score, instrument and masked parties to MKTO, CCO and MLRO only.
- G14 on orders is A42's `restrictionStatus`: a `payouts_held` or `full` restriction refuses new orders as `under_review`, with a byte-identical shape to any other `under_review` refusal.

**8. Approvals, roles and the policy rows this node registers with A14**

| Action code | Policy | Maker → checkers | Precheck (this node) |
|---|---|---|---|
| `market.venue_mode.change` | `4E` | CCO → LEGL | G21 binding, phase, no open orders when disabling |
| `market.instrument.list` | `4E` | MKTO → CCO | funded and minted, G20, lock-in date, G21, `board_open` pre-flight |
| `market.band.set` | `6E` | MKTO → CCO + CFO | bp range, band v+1 impact (orders that would be cancelled) |
| `market.instrument.resume` | `4E` | MKTO → CCO | no halt source open, pre-flight clean, G20 |
| `market.instrument.close` | `4E` | MKTO → CCO | open-order count shown to the checker |
| `market.trade.bust` | `4E` | MKTO → CCO | `retries_exhausted`, no chain transfer |
| `market.trade.settle` | `AUTO` | SVCA | §6 prechecks; attested cap = trade units |
| `market.ask.freeze`, `market.ask.unfreeze` | `AUTO` | SVCA | exact units from `resale_freeze` |
| `halt.open` (manual, batch) | `SGL+R` | A48's row | A48's |

Segregation of duties follows A14. The maker, their delegations and their agents are excluded from checking. MKTO may never be the CCO checker on their own request, and SUPER holds no market cell.

**9. HTTP surface, agent parity (D04, G22), events and errors**

- **Investor and public routes, all under `/v1/market`, with cursor pagination (A09 `listRoute`) and idempotency keys on writes:**
  - `GET /instruments`: public and signed-in list, with state, the pill (`OPEN | HALTED | DISABLED | CLOSED`), valuation and date, band, last price, Δ vs valuation to one decimal, 30-day volume and lock-in text. Vectors: PAD01 last $99.80 against $102.40 is −2.5 %, and SBK02 $58.10 against $61.75 is −5.9 %.
  - `GET /instruments/{id}`.
  - `GET /instruments/{id}/book`: aggregated price levels with units and order count, no identities, for signed-in investors.
  - `GET /instruments/{id}/quote`.
  - `GET /instruments/{id}/sell-preview`.
  - `GET /instruments/{id}/tracker`: public (INV-07). It returns the last price vs valuation series from `market_tracker_day`, units transacted in the last 30 days, premium or discount, and halt history from A48 with cause, duration and dates. With no trades it returns `{trades: 0}` plus the offer price and valuation only.
  - `POST /instruments/{id}/orders`.
  - `GET /me/orders?state=`, `GET /orders/{id}` and `POST /orders/{id}/cancel`.
  - `GET /me/trades`: the POR-04 "sale and proceeds" rows, with `registered` and `proceeds_cents`.
- **Staff routes:**
  - `GET /trades?instrument=&state=`: the blotter, with masked parties, approval ids, tx hash, register instruction and state.
  - `POST /instruments`: the list request.
  - `POST /instruments/{id}/venue-mode`, `/band`, `/resume` and `/close`, each creating its A14 request.
  - `POST /trades/{id}/bust`.
  - `GET /surveillance/alerts` and `POST /surveillance/alerts/{id}/case`.
  - `GET /instruments/{id}/breaker-events`.
- **Internal route:** `POST /v1/internal/market/events`, for the service principal only, receives A28 op results, A18 acks, A48 and A20 halt events, A34 valuation events and A21 flag events. It is idempotent on the source event id.
- **Agent parity.** Every route carries `x-realx-scope` and passes through A12's `agentAuth` and G22.
  - **Investor agents.**
    - `market:read` covers every read.
    - `market:order.sell` and `market:order.buy` allow placing orders within the mandate. Caps count `units` against `per_action_units` and `daily_units`. They count the value at the limit against `per_action_cap_cents` and `daily_cap_cents`: gross `units × limit` for an ask, and `bid_amount` from §4 for a bid. `allowed_vehicle_ids` limits the instruments and time windows apply.
    - The responses follow A12's contract exactly:
      - Within every cap, with a mandate signing key: the signature satisfies the confirm step and the order proceeds through the gates.
      - Above `per_action_cap_cents` or `per_action_units`, or for any order when no signing key is registered: `428 CONFIRMATION_REQUIRED` with a `confirmation_id`. No order, freeze, reservation or journal is created until the person confirms with a fresh passkey assertion through A12's confirmation route. The confirmed order is then placed from the stored payload.
      - Above the daily or monthly cap, outside `allowed_vehicle_ids` or outside the time window: `403 G22_LIMIT_EXCEEDED` (with `cap: daily | monthly` and the remaining cents or units), `403 G22_VEHICLE_NOT_ALLOWED` or `403 G22_OUTSIDE_TIME_WINDOW`. The refusal names the cap and what remains, and nothing is created.
    - Joint co-signing, entity signatories, G12 under standing consent and non-delegable G05 follow §4's "Orders placed by an agent".
    - `market:order.cancel` needs no confirmation.
    - A cap reservation counted at acceptance is not refunded when an ask later fails asynchronously (a freeze failure). This conservative choice is recorded.
  - **Staff agents** (MKTO's) may read everything their human reads and draft the requests in §8 as maker (`market:instrument.request`, `market:trade.bust.request`). They may flag an alert for a case as a draft that the MKTO confirms with a passkey (the A30 draft pattern). They are never checkers.
  - The A48 `halt.open` act, every A14 decision, the custody `sign` and `broadcast` routes, and the venue-mode checker step are refused to every AGNT before scope evaluation (`APR_CHECKER_MUST_BE_HUMAN`, A11's `AGNT_NOT_ALLOWED`).
  - The SVCA pipeline principal that settles is a service principal and never a person's agent.
  - `agents.traffic` and A101's per-user and per-tenant switches stop agent orders within one request.
  - Every agent action is audited "agent <token prefix> on behalf of <person>" and appears in the person's activity and on ADM-35.
- **Events.** The module writes audit events (A10), one per transition and decision, each with `approval_id`, `actor_id`, `on_behalf_of_id` and `token_id` where set. The same events go to the outbox (A09) for A43 notices, A46 webhooks, A101 sensitive-action notices and A47's read models:
  - `market.instrument.listed|opened|halted|resumed|closed|venue_mode_changed|band_changed|reference_updated`;
  - `market.order.submitted|validated|rejected|opened|partially_filled|filled|cancelled|suspended|reopened|expired`;
  - `market.trade.filled|settling|settled|settlement_failed|busted|registered`;
  - `market.circuit_breaker.fired|lifted`;
  - `market.surveillance.hit`.
- **Error codes.** These are stable and listed in OpenAPI, returned with 403, 409, 422 or 428:
  - `MKT_BOARD_NOT_OPEN`, `MKT_VENUE_DISABLED`, `MKT_INSTRUMENT_NOT_OPEN`, `MKT_HALTED` (with the cause, criteria and expected end), `MKT_PRICE_OUTSIDE_BAND`, `MKT_TICK_SIZE`, `MKT_QUANTITY_INVALID`;
  - `MKT_LOCKED`, `MKT_INSUFFICIENT_UNLOCKED`, `MKT_HOLDING_FROZEN`, `MKT_HOLDER_CAP`, `MKT_INSUFFICIENT_FUNDS`, `MKT_SELF_MATCH_PREVENTED`, `MKT_FREEZE_FAILED`, `MKT_ORDER_NOT_CANCELLABLE`;
  - `MKT_STILL_HALTED`, `MKT_RECON_NOT_CLEAR`, `MKT_OPEN_ORDERS`, `MKT_NO_LIST_REQUEST`, `MKT_BUST_NOT_ELIGIBLE`, `MKT_BUST_AFTER_CHAIN`, `MKT_ILLEGAL_TRANSITION`, `MKT_INSIDER_BLACKOUT`;
  - plus A19's gate verdicts, A12's `G22_*` and `CONFIRMATION_REQUIRED`, A11's `AGNT_NOT_ALLOWED` and `FRESH_ASSERTION_*`, and A14's `APR_*`.
- **Empty and edge states:**
  - An empty book returns `{asks: [], bids: [], floor_cents}`.
  - A trust with no instrument returns 404 on the investor routes and `not_listed` in the preview.
  - An order on a `closed` instrument is `MKT_INSTRUMENT_NOT_OPEN`.
  - Cancelling a filled order is `409 MKT_ORDER_NOT_CANCELLABLE`.
  - A sell preview for a trust the person does not hold returns `held: 0` and disables the action. This closes design bug 10 (design analysis §6.4). The P1 behaviour, where the sell action never creates an order, is acceptance test 12 of design analysis §4.

**10. Spec points to settle at Specify, before the node QA writes the oracle (the user decides; each answer changes data)**

- **(a)** The SM-09 split into order and trade subsets, the five order and trade extensions and the seven-transition instrument lifecycle in §1. If the user declines, the node applies SM-09 as one table on both rows and the oracle is written against that.
- **(b)** Seller proceeds are released at `registered`, with the legal record first, rather than at `settled`.
- **(c)** The listing freeze and unfreeze are AUTO, bound to the order id. SM-07's four-eyes rule applies to compliance freezes only.
- **(d)** The circuit-breaker and perimeter halts are off-chain only.
- **(e)** By default, trades matched before a halt settle only through the off-chain `circuit_breaker` halt (`settle_through_causes = [circuit_breaker]`). Every halt that is mirrored on chain makes them wait in `filled`. The alternative in §6, where manual, meeting and valuation-stale halts also settle through, needs A19.1, A25.1 and A28.1 rows.
- **(f) Settlement order.** Chain first, then register, as SM-09 and J07 order it. The D05 key rule "registry ack before any on-chain move" reads the other way, and counsel decides under HG-LEGAL. If the answer is register first, this node is reopened: the ordering is structural, not a flag.

## Owned paths (other nodes do not edit these)

- <workspace>\app\services\api\src\modules\market\
- <workspace>\app\services\worker\src\jobs\market\

Inside the module, by convention:
- `instruments/`: listing, venue mode, band versions, reference updates and close.
- `orders/`: entry, validation, cancel and expiry.
- `engine/`: the pure `match`, band, fee and circuit-breaker functions, with no I/O and the clock passed as an argument.
- `settlement/`: the settle and retry drivers, the bust effector and the integrity checks.
- `surveillance/`: the scenario rows and the evaluator.
- `gates/`: the fact providers registered with A19 (sellable units, open-bid units, unregistered trades, reference and band).
- `read/`: book, quote, preview, tracker and blotter.
- `migrations/`: the additive tables through A04's registry.
- `events.ts`, `errors.ts`, and `index.ts`, which is the only import surface for other modules.

Worker jobs:
- `market.settle`, `market.settlement_retry`, `market.register_handoff`, `market.circuit_breaker_release`;
- `market.order_expiry`, `market.cosign_expiry`, `market.suspended_sweep`, `market.freeze_watch`, `market.freeze_reconcile`;
- `market.surveillance_scan`, `market.tracker_rollup`, `market.integrity`.

The upstream rows in §1 are proposed as diffs to their owners (A03, A14, A16, A18, A19, A28, A48, and A25 if accepted). They are never edited here.

## Design references

- **`design:sell PriceBand and checks (G08, G09, G10)`** in `<workspace>\design\RealX App.dc.html`. The screen key is `sell`, the screen meta at line 873 is "INV-05 · P2 · Sell tokens", and it is product-map INV-05.
  - **PreviewBanner, line 482.** Brick tint `#F5E4DB`, text `#7A3A1E`: "Preview · the resale board opens with the retail launch (from about 8 Apr 2027), once counsel or ASIC relief allows". This is the P1 state of the preview endpoint.
  - **Sell card fields, lines 486-487.** "Tokens to sell" (placeholder "20 of {held}") and "Limit price per token" are Geist Mono inputs on `#FBFAF7`, the `units` and `limit_price_cents` of the order route.
  - **PriceBand, lines 490-492.** Design-analysis §2 block 33: a 10 px track with a grey / light-green / grey gradient, an ink `#17191C` marker and the labels lo / "valuation A$x" / hi. The binding at line 909 is `band:{lo:$(P.val*.85),hi:$(P.val*1.15)}`. The preview returns `lo_cents`, `hi_cents`, `reference_cents` and the marker position, so the static 52 % marker becomes data. With the inward integer rounding, the four fully funded design trusts reproduce the design's figures exactly.
  - **"Place sell order", line 494.** Green `#1F5E4B` primary.
  - **CheckList, lines 497-499, bound at line 910 (`sellChecks`).**
    - "Lock-in period (G08) · No lock-in on this trust · Clear" green.
    - "Trading halt (G10) · No meeting in progress · Open" green.
    - "Price band (G09) · ±15% of latest valuation · Set" green.
    - "Board status · Opens with retail launch · 2027" brick.
    - The preview's `checks[]` returns exactly these four rows, with their strings taken from A19's verdicts and A48's `haltNotice()`. In P2 the fourth row becomes "Open" green or "Halted" red, with the reason.
    - The design's "No lock-in on this trust" conflicts with the product map's 3-month default. The product map wins (design analysis §6.2), so the lock-in row shows "Unlocked since <date>" or "Locked until <date>" from the lots.
  - **Sell entry points.** HoldingRow "Sell" (line 392) and the property page's "Sell tokens (resale · 2027)" (lines 837-838) open the preview for a held trust only.
- **Tone mapping** (design analysis §1.1, TONE map): instrument `OPEN` and trade `registered` are green; `HALTED` and `settlement_failed` are red; `settling`, `settled` (awaiting register) and `suspended` are amber; `DISABLED`, `closed`, `cancelled` and `expired` are grey; the P1 preview is brick. Order, trade and approval ids and prices render in Geist Mono. The palette is cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B` and brick `#9A4524`, with Newsreader for display figures.
- **Undesigned screens this node feeds:**
  - `design-ext:INV-06` and `design-ext:INV-07` from [[A50 - Design extension A- investor and sponsor screens not in the prototype]]: the OrderBook block (asks and bids with depth bars, floor line), GateRows, QuantityStepper, and the KpiGrid, BarChart and halt-history TaskRows.
  - `design-ext:ADM-21` from [[A51 - Design extension B- operator console screens not in the prototype]]: the instruments DataTable with HALTED / OPEN / DISABLED pills, the surveillance TaskRows and the "Halt all" danger button.
  - This module's response shapes carry exactly those fields: the product map's ADM-21 columns Instrument, State, Valuation, Band, Last, Δ, 30-day vol and Lock-in, and its surveillance rows "S2 · SBK02 · wash-trade pattern" and "S3 · PAD01 · band rejects", with the same codes and trigger texts as §7.

## Requirements covered

- **Product-map screens:** none owned. INV-05, INV-06, INV-07 and ADM-21 are built by A92, and POR-02's Sell button and POR-04's sale row by the portfolio screens. "Done" here means every field, action, state and error those screens need exists as an endpoint or event above, checked by a screen contract test that A92 reuses unchanged.
- **Journey J07, "Sell tokens on the resale board" (P2), owned at service level.** Done when all eight steps run through the SDK with the seeded personas on the Docker stack:
  - (1) MKTO lists PAD01 and CCO approves; `pre_open → open` (G08, G20, G21).
  - (2)-(3) A holder of 240 units asks 100 at $99.80, which is validated against $87.04 – $117.76 and partially frozen on chain.
  - (4) A second, eligible investor buys from the lowest ask, with G01, G04, G06 and G07 evaluated as for a primary purchase.
  - (5) SVCA settles AUD and tokens together: `filled → settling → settled`.
  - (6) A18's ack moves it to `registered`, and the confirmations are sent.
  - (7) Surveillance runs, and a +10.01 % would-be trade halts the instrument for 30 minutes (G10, G14).
  - (8) The seller's activity shows the sale and A$9,930.10 proceeds.
  - The three failure paths hold: price outside the band or inside lock-in is rejected with the reason; three failed retries lead to a bust (MKTO → CCO); and a register backlog past SLA halts resale through A20's R17.
- **Gates.**
  - **G07 Holder cap:** done when every bid, every match and every settlement evaluates the buyer's post-trade units, including associates, open bids and unregistered buys, and names `max_allowed_units`.
  - **G08 Lock-in:** done when no ask can freeze a unit in a lot with `lock_in_until` in the future, and the refusal names the date.
  - **G09 Price band:** done when no order rests and no trade prints outside the inclusive integer band of the current band version, and a band move cancels out-of-band resting orders.
  - **G10 Trading halt:** done when every halt source suspends matching within the same transaction as the halt read, resume needs `4E` except the timed breaker lift, and settlement follows the `settle_through_causes` rule in §6.
  - **G14 AML monitoring:** done when the eight scenarios raise A42 alerts, a restricted investor cannot place orders, and nothing investor-facing reveals why.
  - **G21 Licence perimeter:** done when no order exists while `resale.board` is unbound or off, venue mode changes need the binding, and a lapse halts every instrument through the `perimeter` cause.
  - **G22 Agent authority:** done when every agent ask, bid and cancel passes A12's evaluator first, mandate caps count units and value, a refusal names the cap, and no agent is ever a checker, settler or halter.
- **Module M14 Resale board:** done when venue control, instruments with bands, lock-ins and caps, halts and breakers, the trade blotter and surveillance exist as this API. The product map's maker-checker rows must hold by test exactly: set band `6E` MKTO → CCO + CFO; halt `SGL+R` and resume `4E` MKTO → CCO; venue mode CCO → LEGL with the licence flag; list instrument MKTO → CCO; open surveillance case MKTO.
- **State machine SM-09, "Resale order and trade":** done when every state and every transition in A03's table, plus the settled extensions, is exercised, every other pair is refused with `MKT_ILLEGAL_TRANSITION`, and the four invariants hold by test. The invariants are: a move of more than 10 % in an hour halts for 30 minutes; asks rest inside the band only; price-time priority; and nothing is registered without the registry acknowledgement.
- **State machine SM-07 Holding (the resale half):** done when `active → partially_frozen` (listed) and `partially_frozen → active` (cancelled, expired, band moved or busted) are produced only by this node's AUTO freeze ops. A compliance `partially_frozen → frozen` while listed takes the ask off the book without an unfreeze. `eligible_empty → active` for buyers and `active → eligible_empty` for sellers who sold out are reflected by the indexer. Every change reconciles to the register through A20's R3 and R11.
- **Founder requirement D04:** done when every investor and staff action above is reachable by API and MCP under a scoped token with identical gate evaluations (A100 re-runs the parity), and the non-delegable and checker acts refuse agents before scope evaluation.

## Gauntlet loop for this node

- **Specify.** The frozen spec is this note plus the answers to spec points (a) to (f), the landed upstream loops (A03.x, A14 policy rows, A16.1, A18.1, A19.1, A28.1, A48.1, and A25.1 if accepted), the gate evidence of the nodes in the graph dependency gap above, and the contracts listed in §1 with their hashes. The node QA records `specVersion`, those hashes, `oracleVersion` and `hiddenCasesHash` in the evidence manifest.
- **Oracle.** The node QA writes hidden Vitest and fast-check suites to `<workspace>\gauntlet\hidden\A91\`. They run against the Docker Postgres, the Hardhat chain from A23 and the sandbox custody and registry adapters, under the injectable clock. They cover at least:
  - (1) the band: every order outside `[lo, hi]` is rejected with those two numbers, and the design vectors in §3 hold;
  - (2) the circuit breaker: $110.00 does not fire and $110.01 fires; downward moves likewise; the halt lifts at the first tick at or after `fired_at + 30 min`, never before and never more than 60 s after; orders reopen with their original priority;
  - (3) no trade is `registered` without an acked A18 instruction, through any route, job or direct SQL under the API role;
  - (4) price-time priority and the resting-price rule, as a property over random books;
  - (5) the partial freeze equals the listed quantity, and unfreezes never exceed what the ask froze;
  - (6) conservation of cents, meaning the `resale_settlement` invariant and zero net creation across the lock, release and unwind journals, and conservation of units, meaning the frozen-unit invariant and register = chain after every registered trade;
  - (6a) the bid reservation bounds the locks: as a property over random bids (1 to 1,000 units, limits of 1 to 99,999 cents, fee bp of 0 to 300) filled in random partitions, including every unit filled as its own 1-unit trade at random prices at or below the limit, Σ `resale.lock` for the order never exceeds `bid_amount`, the remaining reservation is never negative, and the per-order buyer fees sum to `buyFee(total filled gross)`. The 2 × $1.01 vector in §6 is exact;
  - (7) buyer G07 counts existing holdings, associates, open bids and unregistered buys;
  - (8) settlement retries stop at three, and a bust is possible only after that and never after a chain transfer;
  - (9) agent orders follow A12's contract: a signed bid above the per-action cap returns `428 CONFIRMATION_REQUIRED` with a `confirmation_id`, and a bid above the daily cap, outside the vehicle list or outside the time window returns the matching `403 G22_*` naming the cap and what remains. In every case no order, freeze, reservation or journal exists until a passkey confirmation (for the 428) is given, and the G22 value of a bid equals its `bid_amount`;
  - (10) a halt from every source suspends matching, and a resume without `4E` is refused; a trade filled before a halt settles only when every open cause is in `settle_through_causes`, and otherwise waits in `filled` with no attempt counted;
  - (10a) a seller frozen (SM-07 `frozen`, an A18 full freeze fact, or an A42 `full` restriction) between the match and settlement produces no chain op, no register instruction and no R11 break, and the trade goes down the wait and bust path;
  - (10b) every SM-09 extension in §1, including `filled → settlement_failed` (`HALT_WAIT_EXCEEDED`) and `new_order → cancelled` (`COSIGN_EXPIRED`), is reachable and every other pair is refused;
  - (11) the fee vectors: A$9,930.10 proceeds, and A$15,100.13 for 150 units.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: four parallel sub-agents on disjoint folders, then integration.
  - (1) `engine/`: pure match, band, fee and breaker functions with unit and fast-check tests.
  - (2) `instruments/` and `orders/`: listing, venue mode, band versions, validation through A19, freezes through A28, reservations through A16, cancel, expiry and suspension, and the A14 requests and effectors.
  - (3) `settlement/` and the worker jobs: settle, retry, bust, register hand-off, ack handling, the integrity checks, breaker release and the sweeps.
  - (4) `read/`, `surveillance/` and the HTTP layer: book, quote, preview, tracker and blotter; the scenario rows and A42 intake; zod schemas, routes with `x-realx-scope`, OpenAPI entries, the error catalogue and the internal event route; fixtures for every investor sub-type, MKTO, CCO, CFO, LEGL, SVCA, REGO and an AGNT of each; and the A92 screen contract tests.
  - Integration: the module suite on the two-process stack, `npm run verify`, and `app/evidence/A91.json`.
- **Challenge (node QA, Opus 5.5 high).** The node QA runs the hidden oracles, then the node-specific challenges below by hand against the running two-process stack with the local chain. It checks every gate by every role and by each role's token. It checks the design-fidelity read of the preview for the four fully funded design trusts and the product-map ADM-21, INV-05 and INV-06 vectors, and a grep for `Date.now()`, floats in money paths and PII in events and logs. It reviews that the module imports A16, A18, A19, A20, A21, A28, A42 and A48 only through their public `index.ts`. It writes `app/evidence/A91.qa.md`.
- **Gate, fix loop, escalation.** A pass unlocks A92, A93 and A97 and publishes the hidden cases into `app/`. There are at most three fix loops on the same spec, and each re-runs the full hidden set and the failing challenges. The node escalates to the user in four cases:
  - when the same assumption fails a third time;
  - when a fix needs a change in A16's catalogue, A19's gate sets, A28's catalogue, A48's causes, A18's instruction rules or A25's gateway (a spec defect reopened upstream);
  - when any trade reaches `registered` without an ack, a bust follows a chain transfer, or cents or units are created or lost (Critical: stops the milestone);
  - when a refusal's correctness depends on a counsel answer not yet given (spec points (e) and (f), or the insider block).

## Node-specific challenges (node QA)

- **Double spend of a frozen lot (money, units).** A holder with 240 unlocked units places two asks of 150 within 5 ms on two API processes. Exactly one may validate. The other must be `MKT_INSUFFICIENT_UNLOCKED`, because the first's freeze or its pending `resale_freeze` row counts before chain confirmation. Then a legal partial freeze of 50 units is added by A28 while an ask of 150 rests. Cancelling the ask must unfreeze exactly 150 and leave 50 frozen on chain and in the register mirror. An unfreeze of 200, or a chain frozen amount that disagrees with `resale_freeze` plus the legal row, fails the node.
- **The breaker at its edge (G10, clock).** With reference $100.00, a sweep for 300 units crosses asks at $105.00, $110.00 and $110.01. The fills at $105.00 and $110.00 must print, the $110.01 level must not, and the halt must open with `ends_at` exactly 30 minutes later. At 29:59 on the injectable clock, an order is `MKT_HALTED` with the breaker's criteria. At 30:00 plus one tick, the book is open with the suspended orders in their original order. A second halt source (a meeting halt opened at minute 10) must keep the instrument halted after 30:00 until the `4E` resume. A lift before `ends_at`, or an automatic lift over a meeting halt, fails.
- **Registered without the register (compliance).** The sandbox registry is set to withhold the ack. Then it acks with the wrong instruction id, then acks late after `timed_out`, then rejects. In each case the trade must stay `settled`, seller proceeds must stay in `resale_settlement`, and R11 and R17 must raise, and on rejection halt, through A20. An attempt to set `registered` through the internal route with a forged ack payload, and through SQL under the API role, must be refused by the service and by the trigger. A bust request on the rejected trade must be `MKT_BUST_AFTER_CHAIN`.
- **Buyer cap through the side door (G07).** An investor holds 380 units of a 10,000-unit trust with a 5 % cap, an associate holds 100, and the investor has a resting bid for 15 units plus an unregistered trade for 5. A new bid for 1 unit must fail G07 with `max_allowed_units = 0`. The quote must say the same. A settlement that raced a new associate purchase must fail its settlement-time G07 and go down the retry and bust path, not settle.
- **Agent at the edge of its mandate (G22, money).** An investor's agent has `market:order.buy`, a per-action cap of A$10,000, a daily cap of A$12,000 and a signing key. A signed bid of 100 units at $99.80 has gross A$9,980.00 and fee A$49.90, so its `bid_amount` is A$10,029.90, above the per-action cap. As A12 requires, it must return `428 CONFIRMATION_REQUIRED` with a `confirmation_id`, and create zero orders, freezes, reservations or journals. It is left unconfirmed, and a confirmation attempt with the agent's own bearer token must be refused. A signed bid of 99 at $99.80 (A$9,880.20 + A$49.40 = A$9,929.60) must pass and leave A$2,070.40 of the daily cap. A third signed bid of 25 units at $99.80 the same day (A$2,495.00 + A$12.48 = A$2,507.48) must be refused `403 G22_LIMIT_EXCEEDED` with `cap: daily` and `remaining` A$2,070.40, again with nothing created. Replaying the second signed payload with a new idempotency key must be a signature mismatch. On a joint account with `signing_rule = both`, the agent's passing bid must wait in `awaiting_cosign`, reserve nothing until the co-holder's passkey arrives, and end `cancelled` with `COSIGN_EXPIRED` after 24 hours on the injectable clock. With the IM superseded after the mandate's standing consent was given, the agent's bid must return `428 CONFIRMATION_REQUIRED` with `documents_required`. For a P2 retail buyer with no personal target-market answers, it must be refused with G05 named as counsel-pending. The same agent calling `market.trade.bust` approve, A48's `halt.open` or a venue-mode checker step must be refused before scope evaluation.
- **Wash trade through associates and devices (G14, tipping-off).** Two associated investors trade 6 round trips in 48 hours from one device fingerprint. S1 and S2 must raise exactly one deduplicated A42 alert each with the product map's trigger text. MKTO opens a case. Both investors' order, trade and wallet responses must stay byte-identical in shape to an unflagged investor's. The case must be unreadable to SUPP, IRCM and every investor token. A later A42 restriction must refuse their next order as `under_review` only.
- **Halt versus a filled trade (compliance, spec point (e)).** A trade is filled at 10:00:00.000 by the first level of a sweep whose next level fires the circuit breaker. It must settle and register during the breaker halt while no new match occurs, and HaltModule on chain must stay clear. In a second run, a meeting halt opens at 10:00:00.050 instead. The trade must wait in `filled` with no attempt counted and no custody or registry call, which is what the chain would require. When the meeting halt clears within `settlement_wait_max`, the trade must settle and register without an instrument resume, and matching must stay closed until the `4E` resume. In a third run, a recon Sev1 break opens at the same instant and outlasts `settlement_wait_max`. The trade must go `filled → settlement_failed` with `HALT_WAIT_EXCEEDED` and a pre-filled bust draft. The funds must be fully returned only after the CCO approves the bust.
- **Seller frozen after the match (compliance, units).** A trade is filled, and before `market.settle` runs the seller's holding is moved to SM-07 `frozen` by a court-order freeze through A28. There must be no V2 `forcedTransfer`, no A18 instruction, no `REG_HOLDER_FROZEN` and no R11 break. The trade must wait in `filled`, and after `settlement_wait_max` go down the bust path, with the buyer refunded in full on the CCO's approval. The listing freeze must stay on chain until the MLRO → CCO unfreeze. Repeat with an A42 `full` restriction on the seller (same outcome, and nothing seller-facing names the reason), and with a `payouts_held` restriction (the trade settles, and the proceeds are credited but held by A42).
- **Late register mirror (units).** The chain confirms a settlement, but the sandbox delays the A28 and A18 freeze-fact update for the seller by 5 minutes and then by 15 minutes. With 5 minutes, the trade must stay `settled` with `registration_state = awaiting_mirror`, submit the A18 instruction only after the freeze fact for the settlement op appears, and register with no `REG_INSUFFICIENT_UNITS`. With 15 minutes, `market.register_mirror_late` must be raised at 10 minutes and an A20 targeted run requested, and no false break may be opened.
- **Many small fills (money).** A bid for 200 units at $1.01 at 50 bp reserves `bid_amount` = 20,200 + 101 = 20,301 cents. It is filled by 200 separate 1-unit asks. Σ `resale.lock` must be exactly 20,301 cents, A16 must never refuse a lock, and the reservation must end at zero.
- **Perimeter lapse mid-session (G21).** The licence binding for `resale.board` expires at 14:00 with 30 resting orders and 3 trades `settling`. Within one binding-sweep tick, every instrument must be `halted` with cause `perimeter`, every resting order `suspended` and no new order accepted. The three trades must finish as the recorded rule says, and re-binding must not reopen anything without the `4E` resume.
- **Design fidelity of the data contract.** For Bay Road (val $118.60), the preview must return lo $100.81 and hi $136.39, the marker position for the design's placeholder price $115.20, the four check rows with the design's titles and tones, and the brick P1 "Board status". `GET /instruments` must reproduce the product map's PAD01, SBK02 and WLG01 rows (−2.5 %, −5.9 %, DISABLED "to Feb 27"). The P1 route must return the preview banner state and create no order row. Any new status string, a bare 4xx, or a string typed in a screen instead of returned here fails the contract test.

## Pass criteria

- **Hidden oracle agreement.** 100 % of the node QA's tests in `<workspace>\gauntlet\hidden\A91\` pass on three consecutive runs with different seeds. The runs include fast-check over at least 20,000 random order and cancel sequences per run on books of up to 200 orders. Each sequence asserts that no trade prints outside the band, priority is price then sequence, no self-match prints, no breaching trade prints without a breaker halt, and cents and units are conserved after every step. The runs also include at least 5,000 random clock and trade-price series for the breaker.
- **Circuit-breaker timing.** Under the injectable clock, 100 scripted fires each open exactly one A48 halt with `ends_at = fired_at + 30 min`, and each lift happens at the first tick at or after `ends_at`. None is early and none is more than 60 s late. Each fire and lift leaves exactly one audit event and one outbox event, and re-running the release job changes nothing.
- **Registration integrity.** Over the full suite, the count of trades in `registered` equals the count of acked A18 `transfer` instructions that reference a trade. For every registered trade, the register, the chain balance and the ledger agree (A20 targeted run clean). The trigger refuses 100 % of direct `registered` writes without an acked instruction.
- **Money and units.**
  - After every transaction, the `resale_settlement` balance per vehicle equals the unregistered, unbusted trade totals, and Σ frozen listing units equals open-ask remainders plus unsettled ask-side units.
  - The fee vectors (A$49.90, A$25.23, A$9,930.10 and A$15,100.13) are exact. Σ `resale.lock` per buy order never exceeds its `bid_amount`, including the 2 × $1.01 and 200 × $1.01 vectors.
  - A16's per-tenant zero-sum holds.
  - No cent moves to the seller before `registered`, and no unit moves before the buyer's `resale.lock`.
- **State coverage.** A generated test exercises every SM-09 order and trade state and every transition in A03's table plus the settled extensions (the five order and trade extensions and the seven instrument transitions in §1), and refuses 100 % of the other pairs with `MKT_ILLEGAL_TRANSITION`. The SM-07 transitions in the Requirements section are each produced exactly once per scenario and are visible in A27's `Holding` read model.
- **Gate enforcement matrix.** For every gate in the three resale gate sets, and every principal type (investor sub-types, MKTO, CCO, CFO, LEGL, SUPP, an AGNT of each), a generated test asserts pass, refusal with the named reason, or refusal before scope evaluation. The test has zero disagreements with A19's gate sets, A03's matrix, A14's policy rows and §8. A resale bid and a primary order for the same investor and vehicle produce identical G01-G07, G12 and G16 verdicts. The matrix includes agent bids with current and with stale standing consent (G12 passes, or 428 with `documents_required`), a P2 retail agent bid without personal target-market answers (G05 counsel-pending), and agent orders on `signing_rule = both` joint accounts and two-signatory entities (`awaiting_cosign`).
- **Agent parity.** Every route of this module appears in A100's parity matrix. For each delegable operation, the same request under a passkey session and under a mandate token produces identical gate evaluations and audit events except actor and `on_behalf_of`. Every agent refusal names the scope or cap. The MCP catalogue lists the delegable tools and none of the checker, halt, settle or broadcast acts.
- **Design fidelity.** The preview reproduces the band figures of the four fully funded design trusts and the `sellChecks` strings and tones exactly. The instruments list and the book reproduce the product-map ADM-21, INV-05 and INV-06 mock values. The A92 screen contract tests pass unchanged.
- **Coverage and verify.** Statement coverage is at least 95 % on `engine/` and at least 90 % on the module. The OpenAPI document lists every route and error code above, and `packages/sdk` regenerates without manual edits. The module boundary lint passes. Only the owned paths and the agreed upstream diffs changed. `npm run verify` is green on a fresh clone. `app/evidence/A91.json` and `A91.qa.md` exist and match disk.

## Human gates touched

- **None crossed.** The node runs on the local Hardhat chain, the sandbox custody (V2 policy co-signers), the sandbox registry, the sandbox bank rail and the injectable clock. The `resale.board` flag is bound only to a fixture licence record in tests, and the seeded launch state keeps it off.
- **HG-LEGAL (counsel questions recorded as `set_with_counsel` rows, not decided here):**
  - whether RealX may operate the board at all under the P2 licence or an ASIC relief, and what the binding licence record must say;
  - the band size, and whether a floor-only setting is acceptable;
  - the lock-in default;
  - the size and duration of the circuit breaker, and whether downward moves count;
  - the settlement order, chain first or register first (spec point (f)), and whether proceeds may be released at `settled`;
  - whether trades matched before a halt may settle (spec point (e));
  - whether suspended orders must be cancelled after a halt, and after how long;
  - insider blackout windows, and whether to block rather than alert;
  - whether board purchases carry cooling-off rights for retail investors (default none, with the parameter in A90);
  - whether secondary trades count toward the DAF Act low-value exemption meter (default yes, conservatively);
  - fees on both sides;
  - whether any board act, such as a retail buyer's target-market answers, must be personal rather than delegable to an agent. Those rows stay `"counsel"` in A03's `delegable.ts` and are refused to agents until answered.
  - Counsel's answers change rows. This node's hidden tests are re-run after each change.
- **HG-AUDIT (later):** the settlement path through the vendored `forcedTransfer` and the listing freezes, together with the A25.1 gateway guard if accepted, are in scope of the smart-contract audit before mainnet. This node passes on local-chain evidence and records the gate as open.
- **HG-VENDOR (later):** the rented custody signing policy (AUTO rows for freeze, unfreeze and settle, bound to trade and order ids) and the registry's ack semantics and SLA must pass A15's and A18's conformance suites and this node's hidden tests unchanged before a vendor is selected.
- **HG-PENTEST (later):** order entry, self-match, surveillance evasion, the agent mandate on the board and the internal event route are in scope of the external penetration test before go-live.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
