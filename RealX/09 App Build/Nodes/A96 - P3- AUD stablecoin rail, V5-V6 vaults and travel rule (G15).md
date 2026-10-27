---
title: "A96 - P3: AUD stablecoin rail, V5/V6 vaults and travel rule (G15)"
created: 2026-09-24
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am10]
node: A96
milestone: AM10
lane: "P3"
oracle: yes
human_gate: none
---

# A96 - P3: AUD stablecoin rail, V5/V6 vaults and travel rule (G15)

**The P3 second payout rail, built and tested now and switched on only later: a `stablecoin` adapter whose sandbox is a test ERC-20 on the local chain standing in for an AUDM- or AUDD-type AUD coin (issuer mint, redeem, freeze and pause included); the V5 distribution vault and the V6 treasury vault filled in A15's custody sandbox through the ordinary policy-change quorum, never by a seed edit; KYT wallet screening feeding the custody P7 rule and a G14 wallet-risk check; a travel-rule data exchange (IVMS101-shaped originator and beneficiary data, a counterparty VASP directory, self-hosted wallet ownership proofs, missing-data holds, decline and return) behind the G15 gate and its console panel; payout destinations that are always the investor's own and are added only by the person with a passkey; recon rule R9 turned from a stub into a zero-tolerance check of vault balances and transfers; everything behind the `payouts.stablecoin` flag, which A21 lets on only while a licence record covering `stablecoin_payments` is bound, and every action callable by an agent under the same gates where the act is delegable.**

**Build status:** not started. **Milestone:** AM10 (P3 later). **Lane:** P3. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 10-14 agent-hours for this node (build, challenge and verify loops), plus 8-12 agent-hours for the cross-node loops in §1 run by their owners' builders and re-gated by the node QA (about 18-26 agent-hours in total before this node can gate). The loops reopen already-gated nodes A03, A08, A15, A16, A20, A22, A27, A39, A40 (and A30 if its conditional loop is needed); A50, A51 and A59 are reopened for the screen loops, which gate HG-LAUNCH, not this node.

**Depends on:** [[A39 - Treasury- withdrawals, escrow release, client-money reconciliation]], [[A42 - AML screening, monitoring, cases and SMR clocks (SM-11)]], [[A15 - Custody sandbox adapter and policy engine P1-P10]], [[A27 - Chain adapter, deployment manifests and indexer]], [[A94 - P2- end-to-end journeys and launch update]]
**Unlocks:** none

**Oracle rule:** The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A96\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder; the cases are published into the repo only after the gate passes. This is an oracle node because it signs value out of custody vaults (V5 `AUTO`, V6 quorum), moves client money through the ledger and gates AML and travel-rule releases, like A15, A39, A40 and A42. The oracle scope: the `stablecoin_transfer` transition lists (outbound and inbound), the G15 decision table and the party table, the R9 vectors (including in-flight transfers at the cut block), the ledger journals and run earmarks, the signer-eligibility tests for every V5 and V6 action code, the signing-time re-checks, the destination-ownership property tests (every investor type), the flag and licence fail-closed probes, the agent refusals and the PII and tipping-off scans. If the graph data (`app-graph.json`) still marks A96 as not an oracle node, the graph owner must correct it; until then a runner treats this node as an oracle node, because this frontmatter and this paragraph are the rule.

This note is not legal advice. Every threshold, time limit, retention period, accepted coin, VASP due-diligence rule and the meaning of "self-hosted" or "unhosted" below is a row of data marked "set with counsel and the MLRO". Counsel's answers change rows, not code. AUSTRAC registration, the licence or relief that covers stablecoin payments, the choice of issuer, KYT vendor and travel-rule provider, and the V5 and V6 key ceremonies are human acts outside this build.

## Build brief

This node owns one adapter package with two manifests (`stablecoin` and `travel-rule`), one API module and one console panel. The API module is named `travel-rule` by the graph's owned paths. It is the whole stablecoin rail: destinations, preferences, transfers, vault treasury operations, KYT orchestration, travel-rule messages, the G15 evaluator and the R9 snapshot source. Worker jobs are registered from the same folder through A09's job registry. Every rule is enforced in the API, never only in a screen. Every route is in the OpenAPI document from the first commit, so `packages/sdk` and the MCP tool catalogue (A104) see it. The only source of time is A09's injectable clock. No vendor SDK is imported. A test proves no code path in this node can select a non-sandbox implementation while `RUN_MODE=local|test` (A08's rule).

**1. Follow-up loops requested from the owning nodes (applied by their builders under their own gauntlets, before this node's build starts)**

This node edits only its owned paths. The hooks it needs in other nodes are small data or interface additions. It lists every one of them here, with its owner, so the plan reviewer sees every cross-node change in one place. If an owner declines one, escalate; do not work around it. Each loop is built by its owner's builder (Grok 4.7 xhigh), re-runs that owner's hidden set and challenges under the node QA, and records its new spec hash; the hashes are part of this node's frozen spec. The loops and the reopened nodes must also be recorded in the graph data (an AM10 loop entry, for example `A96-L`, with these owners and the 8-12 h estimate); that is a graph change this node does not make.

- **A03.1 (permission data, owner A03):** adds the new scope cells of §8 to `scopes.ts` and their `delegable.ts` values: `stablecoin.destinations.read`, `stablecoin.transfers.read`, `travel_rule.read` and `stablecoin.destinations.remove` → `true`; `stablecoin.preferences.write` → `"counsel"` (seeded; see §4 and HG-LEGAL); `travel_rule.draft` and `stablecoin.vault.draft` → `true` for the maker kind only (A03's `SCOPE_AGENT_NEVER_CHECKER` still refuses approve, ratify and sign); `payout_destination.add`, `payout_destination.proof` and `travel_rule.unmask` → `false`. A12 reads them through its documented extension list; A12's code does not change.
- **A08.1:** adds the adapter name `travel-rule` to A08's name list and one `SWAP-POINTS.md` catalogue row (human gate HG-VENDOR; candidates Notabene, Sumsub Travel Rule, 21 Analytics). `stablecoin` is already in the list.
- **A15.1 (owner A15):** five changes:
  - P7 KYT becomes active when `PHASE=P3` and `payouts.stablecoin` is on. It reads A22's `screenWallet` score for the destination, with `kyt.block_at_score` and `kyt.review_at_score` as policy rows. In P1 and P2 it still returns `not_applicable_in_phase`.
  - P4 gains a `DestinationClearanceProvider` slot with the contract `clearDestination(op) -> { ok: boolean, code }`, which this node registers at boot. A15 calls it for every V5 and V6 op at `submitOp` and again inside A15's one atomic signing point (A15 §3), where P1-P10 are re-run. `ok: false` refuses with the provider's `code`, the op stays `signing` with the refusal recorded, as A15 already does for any refusal. With no provider registered, P4 refuses every V5 and V6 op (`P4_UNREGISTERED_DESTINATION`, fail closed). What this node's provider checks is in §5, "Signing-time re-check".
  - The check constraint on `custody_accounts.signers[]` admits the `SVCA` as one `AUTO`-only signer slot on V5 and V6. That slot never counts towards `quorum_m`, signs only ops whose vault policy for that op is `AUTO`, and is never allowed on V1-V3 (the V4 and V7 exceptions are unchanged). P9 still excludes it from every non-`AUTO` op.
  - P3's running-total cap counter extends from `mint` and `batchMint` to `transfer` ops on V5 and V6: the sum transferred under one approval id never exceeds the attested `cap` (V5: the run's stablecoin total in cents, converted to base units; V6: the withdrawal or request amount), serialised by the same row lock.
  - P5 bands accept stablecoin base units.
- **A16.1 (owner A16):** adds three account types, all `currency = AUD` in integer cents because the coins are AUD-denominated:
  - `stablecoin_mirror`: one per vault per asset. For V5 it has sub-accounts `unallocated` and one `run:<run id>` earmark per distribution run (the earmark SM-08's `funded` guard checks; §5).
  - `stablecoin_in_transit`: one clearing account per vault per asset. It holds the value of outbound transfers from custody submit until the indexer confirms the Transfer, so the mirror moves only when the coins move on-chain.
  - The catalogue rows:
    - `stablecoin.fund`: bank mirror to the V5 `unallocated` or V6 mirror, posted on the indexer's confirmed issuer mint, `4E` TRSY → CFO.
    - `stablecoin.redeem`: V5 (from a run earmark or `unallocated`) or V6 mirror to the bank mirror of the account the AUD returns to, posted when the burn is confirmed and A17's credit is matched, `4E` TRSY → CFO.
    - `stablecoin.rebalance`: V6 mirror to V5 `unallocated`, and `stablecoin.v5.sweep`: V5 (`unallocated` or a run earmark) to V6 mirror, both `4E` TRSY → CFO, posted on confirmation.
    - `stablecoin.run.earmark`: V5 `unallocated` to `run:<run id>`, for exactly the run's stablecoin total, bound to the run's approval id; refused if `unallocated` is short (row lock, no overdraft).
    - `stablecoin.distribution.pay`: `distribution_payable` to `stablecoin_in_transit` (V5), and `stablecoin.withdrawal.pay`: `investor_cash` to `stablecoin_in_transit` (V6), each posted at custody submit and bound to its approval id.
    - `stablecoin.transfer.settle`: `stablecoin_in_transit` to the vault mirror (for V5, the run's earmark), posted only on the indexer's confirmed Transfer, carrying the Transfer's block number.
    - `stablecoin.return`: the exact reverse of the pay journal, for a transfer that leaves `custody_submitted` or `signed` without broadcasting, or that fails on-chain. A transfer returned after confirmation gets `stablecoin.return` as mirror to payable (or `investor_cash`), posted on the confirmed inbound return.

  Whether coins held in V5 and V6 count as client money is a counsel question. The accounts are created with `is_client_money = true` until counsel answers, because failing to the stricter treatment is the safe default.
- **A20.1 (owner A20):** R9 moves from `skipped_stub` to the rule in §7. The pure function lives in A20's `rules/`, fed by the `StablecoinSnapshotSource` this node exports. R8 for a run with stablecoin lines counts their `stablecoin.distribution.pay`, `stablecoin.transfer.settle` and `stablecoin.return` journals, so "paid equals approved" includes coin lines.
- **A13 (no loop needed):** the queue clock `travel_rule_hold_72h` (owner KYCA, Sev3 on breach) and the holds `QueueSource` are registered at boot through A13's public `registerSlaPolicies` and `registerQueueSource`. The A14 action and policy rows of §3 and §5 are registered through A14's action and policy registries as data. A42's `restrictionStatus`, `screenForPayout`, intake shape and IFTI policy row are consumed unchanged.
- **A22.1:** the sanctions sandbox's `screenWallet` returns scored results in P3, using magic address prefixes defined in §2. It still returns `not_applicable_in_phase` in P1 and P2.
- **A39.1 (owner A39):** the withdrawal gets `rail` (`npp` | `stablecoin`) and a `destination_ref` that may point only to a `verified` `payout_destination` row of this node owned by the same investor account. There is still never a typed-in address, and an `address` field stays a `422` schema error. Release routes to this node's `StablecoinPayoutChannel` when `rail = stablecoin`. Because A39's own spec says no destination parameter exists on any withdrawal route and makes one an escalation trigger, A39.1 must amend that sentence and its escalation rule to admit exactly this reference (resolved server-side to the investor's own verified destination, never a free value). That amendment is a change to an A39 invariant: A39's owner records it and the node QA re-runs A39's destination challenges.
- **A40.1 (owner A40):** four changes:
  - a distribution line's `destination` may be a `payout_destination` reference of kind `stablecoin_wallet`, included in `file_sha256`, so the approval binds it; execution of those lines calls the same channel in place of the `investor_cash` credit;
  - SM-08's `funded` guard also requires the run's V5 earmark (`stablecoin_mirror` V5 `run:<run id>`, posted by `stablecoin.run.earmark`) to equal the run's stablecoin total. The earmark is per run, so two concurrent runs can never count the same coins;
  - `retrying` for a stablecoin line first asks this node's `issuerStatus`, and does not retry against a frozen address, a paused token or a stale attestation until it clears or policy moves the line to `manual_payout`;
  - `manual_payout` for a stablecoin line is funded by the fallback money sequence in §5, and `reconciling → closed` also requires the run's earmark to be zero.
- **A27.1 (owner A27, required):** A27's local-only deploy path compiles, tests and deploys adapter-owned sandbox contracts from a declared path (`packages/adapters/*/sandbox/contracts/`) with the contracts package's Hardhat toolchain, so `TestAUDStable.sol` is not compiled by a second toolchain. A27's test asserts that every contract under a sandbox path is excluded from Slither and HG-AUDIT scope, never appears in a Base mainnet manifest and is refused by the mainnet deploy script.
- **A30.1 (owner A30, only if A30's `index.ts` does not already expose it):** a read `entityLegalNames(accountId) -> { registeredName, fundOrTrustName? }` for SMSF, company and trust accounts, used by the beneficiary-name rule in §4. A12's `SignatoryPort` (implemented by A30) is used unchanged.
- **A50.3, A59.1 and A51.1 (screens):** A50.3 (A50's P3 payout-rail frames) and A59.1 add the investor's "Payout rail" card and the "Add a wallet" flow on POR-05. A51.1 adds the travel-rule console frame (see Design references).

  These three do not block this node's build or gate: every investor action here is API-complete and tested through the SDK. They do block switching the flag on (HG-LAUNCH), because an act a person can only do through an agent breaks D04 parity in reverse.

**2. Adapters (`packages/adapters/stablecoin/`, on A08's frame)**

- The `stablecoin` manifest calls A08's `defineAdapter` with `implementations: { sandbox }` only and `swap: { humanGate: 'HG-VENDOR', candidates: ['AUDM (Macropod, AFSL 566313)', 'AUDD (Novatti)', 'A$DC (ANZ)'], whatChanges: 'issuer mint and redeem calls, the token contract address per chain and the issuer status feed; the transfer path, custody signing, R9 and the API are unchanged' }`. Its verbs, each with a zod contract, and none signing or holding key material:
  - `listAssets()`: the payment-asset allow-list, meaning symbol, issuer, chain id, contract address, decimals, status (`active`, `suspended`, `delisted`) and the issuer's latest reserve-attestation date. This is the research pack's cross-cutting control 19.
  - `mint({ asset, vaultCode, amountCents, fundingRef })` and `redeem({ asset, vaultCode, amountCents, bankRef })`: issuer operations against AUD moved on A17's rail.
  - `buildTransfer({ asset, fromVault, to, amountCents })`: returns unsigned calldata for A15.
  - `getBalance(asset, address, blockTag)` and `getTransfers(asset, address, fromBlock, toBlock)`: both read through A27's chain adapter and indexer.
  - `issuerStatus(asset)`: whether the issuer has frozen an address, paused the token, or let its attestation go stale past `stablecoin.attestation_max_age_days`, default 45.
  - `health()`.
- Amounts: every amount in and out is integer cents. Base units = cents × 10^(decimals − 2). An asset with fewer than 2 decimals is refused at registration (`SC_DECIMALS_UNSUPPORTED`). A chain amount that is not a whole number of cents is never rounded. An inbound one is quarantined (§6), and an outbound one cannot be built.
- The sandbox is a test ERC-20 (`TestAUDStable`, 6 decimals) in `sandbox/contracts/`. It uses OpenZeppelin ERC20, Pausable and AccessControl, imported unchanged, plus an issuer-only `mint`, `burn`, `freeze(address)` and `pause`, so the tests meet the failure modes of a real issuer-controlled coin. It is deployed only to the local Hardhat chain (and, at A86's discretion, Base Sepolia) through A27's deploy path. It carries no value and is never a mainnet artefact. The sandbox issuer mints when A17's sandbox bank shows the AUD payout to the issuer's account, with the same `fundingRef`. It redeems by burning and asking A17's sandbox to credit the client-money account.
- Magic inputs live only inside `sandbox/`:
  - `SIM-ISSUER-FREEZE-<address>` freezes an address at the issuer.
  - `SIM-ISSUER-PAUSE` pauses the token.
  - `SIM-MINT-DELAY-<s>` delays a mint.
  - `SIM-ATTESTATION-STALE` makes the issuer attestation stale.
  - Addresses beginning `0x5A5A` score high risk in A22.1's `screenWallet`, and `0x5B5B` score review.

  A `SandboxControl` registers `trigger('issuer-freeze', {address})`, `trigger('inbound-transfer', {to, amountBaseUnits, from})`, `advance(duration)`, `reset(seed)` and `state()`. A09 mounts them under `/sandbox/stablecoin/*` in `local|test` only.
- The `travel-rule` manifest has verbs:
  - `discoverVasp({ address } | { vaspId })`.
  - `sendTransferData(payload)`: an IVMS101-shaped originator and beneficiary record, returning `messageRef`.
  - `getMessage(ref)`.
  - `acceptInbound(ref)`, `declineInbound(ref, reasonCode)` and `requestMissingData(ref, fields[])`.
  - `subscribe(handler)`, for the signed, idempotent webhook.
  - `health()`.

  The sandbox is a small in-process network of four fixture VASPs:
  - `VASP-SIM-AU-OK`: Australian, acknowledges within 5 s.
  - `VASP-SIM-FOREIGN-OK`: foreign jurisdiction, acknowledges.
  - `VASP-SIM-DECLINE`: declines with a reason code.
  - `VASP-SIM-NO-SUNRISE`: its jurisdiction does not yet enforce the travel rule, and it never acknowledges.

  A `VASP-SIM-TIMEOUT` marker makes any VASP stay silent. Each manifest has a `conformance/` suite written against the interface. A human runs it against the chosen vendor's sandbox after HG-VENDOR, before that implementation may be registered.

**3. Vaults V5 and V6 (custody sandbox, via A15's own paths)**

- A15 seeds V5 and V6 as `reserved` with empty allow-lists. This node never writes `custody_accounts`, and A15's P8 trigger would refuse it anyway. The deploy script `fill-vaults` drafts two A15 `custody.policy.change` requests (`6E+Q`, SECO maker, V1 owner quorum 3-of-5, 24 h timelock), which the test fixtures' signers approve. The V1 slots are CEO, CFO, CCO, SECO and the independent member, and P9 removes the maker and both checkers from the V1 signer set. So the two `6E` checkers of these requests are TRSY and CCO (proposed; at most one V1 member among the checkers), which leaves CEO, CFO and the independent member eligible, three for a 3-of-5. A test asserts that at least three V1 signers stay eligible for each `fill-vaults` request. If A14's seeded checker sets for `custody.policy.change` cannot be satisfied that way, the build escalates it as an A03/A14 data change and does not work around it.
- The proposed policy, which the founder and counsel confirm at Specify as spec point (a) and which stays policy data whatever the answer. The rule that shapes it: for every action code that can sign on a vault, the signer set must hold at least the quorum plus the largest number of signers that P9 can exclude for that code (the maker, every checker and, for A39's `6E`, the second approver). The largest exclusion here is 3: A14's `withdrawal.release` `6E` row admits "TRSY or System threshold" as maker with CFO plus CCO or CEO as checkers, so a TRSY-made `6E` excludes TRSY, CFO and CCO (A39's flow always uses System threshold, which excludes only the two checkers). So a 2-of-N set needs N ≥ 5. The proposal is 2-of-5 for both human quorums:
  - **V5 distribution:** warm. Signers: the `SVCA` `AUTO`-only slot (A15.1) plus the human set TRSY, CFO, CCO, CEO and SECO, 2-of-5. Allow-list, all on the allow-listed asset's address only:
    - `transfer(address,uint256)` to a distribution line's destination, signed `AUTO` by the SVCA within policy, and only when the destination and amount are a line of an approved distribution run's file with matching `file_sha256`. The per-approval cap equals the run's stablecoin total (A15.1's P3 counter for transfers).
    - `transfer` to the issuer's redeem address (`stablecoin.vault.redeem` from V5) and `transfer` to V6 (`stablecoin.v5.sweep`), each `4E` TRSY → CFO and signed by the human 2-of-5.
    - `transfer` of a quarantined inbound back to its sender (`travel_rule.inbound.return`, `4E` MLRO → CFO), signed by the human 2-of-5.
    Nothing else, and nothing but distribution lines is ever `AUTO` on V5.
  - **V6 treasury:** warm. Signers: the `SVCA` `AUTO`-only slot plus TRSY, CFO, CCO, CEO and SECO, 2-of-5. The allow-list is `transfer` to investor destinations for withdrawals, `transfer` to V5 for rebalancing, `transfer` to the issuer's redeem address and `transfer` of a quarantined inbound back to its sender. `AUTO` SVCA signing is allowed only for withdrawal transfers inside A39's `AUTO` band (the same `auto_max_cents` row). Everything else needs the human quorum.
  - **Both:** P6 business-hours window, P7 KYT, P9 (the SVCA is never the maker of what it signs, and no agent ever signs), and P10 with no timelock on transfers. Rotation class `warm`.
  - **Signer-eligibility test:** for every action code that can sign on V5 or V6 (`withdrawal.release` `AUTO`, `4E` and `6E`; the run approval for V5 lines; `stablecoin.vault.fund`, `.redeem`, `.rebalance`, `stablecoin.v5.sweep`; `travel_rule.inbound.return`), and for every maker and checker combination A14's rows allow, a test computes the signer set after P9 and asserts it holds at least `quorum_m` eligible human signers (or the SVCA for an `AUTO` op). Worked cases: vault operations TRSY → CFO leave CCO, CEO and SECO; A39's `4E` System threshold → CFO leaves TRSY, CCO, CEO and SECO; A39's `6E` → CFO + CCO leaves TRSY, CEO and SECO; a TRSY-made `6E` → CFO + CCO leaves CEO and SECO; the inbound return MLRO → CFO leaves four.
- Spec point (b), the on-chain binding: proposed, V5 and V6 are custody-held addresses that sign plain ERC-20 transfers. The approval id is enforced before signing by A15's P1 attestation check and proven after the fact by R9. R9 matches every Transfer event from a vault to exactly one `CustodyOp` and its approval id, and flags anything unmatched as an `unbound_transfer` Sev1. The alternative is a V5 and a V6 instance of A25's `AgentGateway` holding the balances, so `Executed(approvalId)` is emitted on-chain. That is a deploy-script change and no new Solidity, but it moves coins into a contract that production custody vendors do not usually hold. The user decides. Either way this node writes no Solidity beyond the sandbox test token.
- Vault operations (TRSY maker, `4E` → CFO, registered through A14's action and policy registries as data rows):
  - `stablecoin.vault.fund`: an A17 payout of AUD to the issuer from the Distributions or Treasury client-money account, then the adapter's `mint` into V5 or V6. The `stablecoin.fund` journal is posted only on the indexer's confirmed Transfer from the issuer mint to the vault.
  - `stablecoin.vault.redeem`: from V5 or V6 to the issuer's redeem address; the `stablecoin.redeem` journal is posted when the burn is confirmed and A17's credit to the named client-money account is matched.
  - `stablecoin.vault.rebalance`: V6 to V5 `unallocated`.
  - `stablecoin.v5.sweep`: V5 (`unallocated`, or a closed run's leftover earmark) to V6.
  - `stablecoin.run.earmark` is not a staff request: this node's `StablecoinPayoutChannel.earmarkRun({ runId, totalCents, approvalId })` posts it, and A40.1 calls it before `funding → funded`. It is idempotent per run and refuses if V5 `unallocated` is short (`SC_V5_UNFUNDED`), so the run stays `funding` until TRSY funds V5.

  Emergency stop is `stablecoin.rail.suspend` (`SGL+R`: CCO or CFO acts with a hardware key and the other ratifies within 2 h). A lapse escalates through A13 and never resumes the rail. Its effector calls A15's `suspendVault` for V5 and V6 and sets the channel to `suspended`. A48's R6 runbook (stablecoin half, seeded `phase: P3`) resolves to this action code. Resuming is `stablecoin.rail.resume` (`4E` CFO → CCO).

**4. Payout destinations and preferences (always the investor's own)**

- `payout_destination` rows (id `PDS-<yyyymmdd>-<6 base32>`) carry:
  - `investor_id` and `kind`:
    - `registered_wallet`: the investor's passkey smart wallet from A29, identity-registered. The worker job `stablecoin.destinations.bootstrap` creates it in `screening` (never directly `verified`) when the flag turns on, and it becomes `verified` only after a KYT result under `kyt.review_at_score`, with the same `review` and `blocked` outcomes as any destination. It is exempt from the cooling period, and no "new destination" notice is sent, because the address is not new: the person created it in person with their first passkey (a non-delegable act) and A31 registered it to their verified identity, so there is no account-takeover window to cool. The exemption is a policy row (`payout_destination.cooling_exempt_kinds = ['registered_wallet']`, counsel to confirm), and tests cover it both ways.
    - `self_hosted`: an external wallet the investor controls.
    - `vasp_account`: an account at another VASP.
  - `chain_id`, `address` (stored in clear because it is public chain data, shown truncated), `vasp_id` and the beneficiary name as the VASP holds it (`vasp_account` only).
  - `destination_sha256`: the SHA-256 of the canonical `(kind, chain_id, address, vasp_id, beneficiary_name)`, computed at insert.
  - `ownership_proof_id` (`self_hosted`), `kyt_result_id`, `state` (`pending_cosign`, `pending_proof`, `screening`, `verified`, `review`, `blocked`, `removed`), `verified_until` and, for joint "both to sign" adds, `cosign_due_at`.
  - Immutability: `kind`, `chain_id`, `address`, `vasp_id`, beneficiary name, `investor_id` and `destination_sha256` never change after insert. A database trigger allows updates only to `state`, `verified_until`, `kyt_result_id`, `cosign_due_at` and, while `state = pending_proof`, `ownership_proof_id`; any other update is refused. A different address is a new row.
- Adding a destination is the stablecoin equivalent of changing the withdrawal bank account, which D04 lists as non-delegable. `POST /v1/me/payout-destinations` and `POST /v1/me/payout-destinations/{id}/proof` therefore carry `requiresHuman: true` and a fresh passkey assertion (A11, purpose `payout_destination_add`, over `destination_sha256`). An AGNT principal gets `AGNT_NOT_ALLOWED` before scope evaluation.
  - **Who must be present, by investor type** (the rule A12 uses for token creation, read through A12's `SignatoryPort`; counsel to confirm that the same rule fits a payout-destination change):
    - individual: the holder's assertion.
    - joint, signing rule `either`: one holder's assertion; the other holder is notified through A43 at once.
    - joint, signing rule `both`: the first holder's add creates the row in `pending_cosign`; the second holder's assertion over the same `destination_sha256` within 24 hours moves it on, otherwise it becomes `removed` (`PDS_COSIGN_EXPIRED`).
    - SMSF, company or trust: only a principal the `SignatoryPort` reports as a verified signatory of the entity may add (`PDS_SIGNATORY_REQUIRED` otherwise); every other verified signatory is notified.
  - **Beneficiary name for `vasp_account`, by investor type** (exact after A17's normalisation, no fuzzy score; counsel to confirm):
    - individual: the holder's verified legal name.
    - joint: either holder's verified legal name, or the two verified names joined as A17's normalisation table forms a joint name.
    - SMSF, company or trust: the entity's registered name or the fund or trust name only (A30's KYB record, A30.1), never an individual trustee's, director's or signatory's personal name, because that would be another person's account.
  - **Self-hosted proof for an entity:** the proof is signed by the wallet's key and presented by a verified signatory in person; the challenge text names the entity account id and the signatory's principal id, and the signatory confirms the wallet is controlled by the entity. Whether an entity may use a self-hosted wallet at all is an HG-LEGAL question; the default is allowed.
  - Tests: one per investor type (individual, joint `either`, joint `both` with and without the cosign, SMSF, company, trust), each with a name that matches and one that does not.
  - A `self_hosted` wallet needs an ownership proof. The server issues a challenge ("RealX ownership proof · INV-… · <address> · <nonce> · <issued at>", valid 15 minutes). The investor signs it with that wallet, and the module verifies EIP-191 for an externally owned account or EIP-1271 against the local chain for a contract wallet. The proof is valid for `travel_rule.self_hosted_proof_months` (default 12, set with counsel). The micro-transfer ("Satoshi") test is not built; whether counsel requires it is recorded in HG-LEGAL.
  - A `vasp_account` needs the VASP in the approved directory (§5) and a beneficiary name that equals one of the investor's verified names under A17's normalisation table. The match is exact after normalisation, with no fuzzy score. Otherwise `PDS_BENEFICIARY_NAME_MISMATCH`, because money only ever goes to the investor's own account.
  - Every new destination, `registered_wallet` included, is KYT-screened before it becomes `verified`: score at or above `kyt.block_at_score` is `blocked`, with an AML intake line to A42 and no investor-facing reason; at or above `kyt.review_at_score` it goes to `review` for KYCA.
  - A new destination (except a kind in `payout_destination.cooling_exempt_kinds`) cannot receive a transfer for `payout_destination.cooling_hours` (default 24, counsel to confirm). The person is notified through A43 on every add, and so are their other verified channels (the account-takeover defence the bank-account change also uses).
- `DELETE /v1/me/payout-destinations/{id}` narrows, so it is delegable under the money-movement scope and audited "agent <token prefix> on behalf of <person>". `PUT /v1/me/payout-preferences` (`{ distributions: 'aud_wallet' | 'stablecoin', destination_id }`) has the scope `stablecoin.preferences.write`, whose `delegable.ts` cell is seeded `"counsel"` for this build (HG-LEGAL). While it is `"counsel"`, A12 refuses to issue a token carrying that scope and refuses an agent call to the route with `SCOPE_PENDING_COUNSEL`; the person can always make the change in person. The proposal to counsel is yes, because the preference chooses only among destinations the person added in person. The hidden tests flip the data row: `true` → the agent call succeeds under the scope and raises an A101 sensitive-action notice; `false` → `G22_SCOPE_NON_DELEGABLE`. Either way the change applies to runs whose snapshot is locked after it, never to a run already `calculated`.
- **Investor-safe destination states** (every investor and investor-agent read of a destination, and every notice about one): `pending_cosign` → `waiting_for_co_holder`; `pending_proof` → `needs_proof`; `screening` → `checking`; `verified` → `ready` (with `usable_from` while cooling); `review` and `blocked` → `unavailable`, with the message "This wallet can't be used for payouts right now" and byte-equal bodies after id substitution; `removed` → `removed`. The KYT score, result and internal state never appear in these reads.
- Destinations are re-screened nightly (worker job `stablecoin.kyt_rescreen`, 03:30 Sydney, idempotent) and on A22's list-update event. A `blocked` result on a destination with an open transfer holds that transfer (§5). Removing a destination with a transfer in flight is `409 PDS_IN_USE`.

**5. Transfers, the G15 gate and travel-rule messages**

- `stablecoin_transfer` (id `SCT-<yyyymmdd>-<6 base32>`) carries:
  - `direction` (`out` or `in`, immutable after insert) and `purpose`: outbound `distribution_line`, `withdrawal`, `vault_redeem`, `rebalance`, `v5_sweep`, `quarantine_return`; inbound `vault_fund` (the issuer's mint) and `return` (a counterparty sending back a transfer this node sent).
  - `source_ref` (line id, WDR id, APR id or quarantine id), `retry_of` (the SCT id a retry replaces), `approval_id`, `asset`, `vault_code`, `destination_id`, `cleared_destination_sha256` (copied from the destination at `cleared`), `amount_cents` and `amount_base_units`.
  - `kyt_result_id`, `kyt_screened_at`, `travel_rule_message_id`, `g15_evaluation_id` and `custody_op_id`.
  - `tx_hash`, `block_number`, `ledger_journal_id`, `settle_journal_id`, `hold_reason` and `state`.

  The transition lists are local data in A03's engine format, like A12's token table, not one of the 14 product-map machines. Every pair not listed is refused with `SCT_ILLEGAL_TRANSITION`, a self-transition included, and every transition writes one audit event. A failed or cancelled line that A40 retries is a new SCT row with `retry_of`, never a transition out of a terminal state.

  **Outbound (`direction = out`), 13 states, 21 allowed pairs:**

  | # | From → to | When |
  |---|---|---|
  | O1 | `requested → screening` | worker picks it up (external destinations and `quarantine_return`) |
  | O2 | `requested → cleared` | internal purposes only (`vault_redeem`, `rebalance`, `v5_sweep`): the counterparty is the issuer's redeem address or RealX's own vault, so no screening or travel-rule message (counsel confirms for the issuer) |
  | O3 | `requested → held` | `flag_off`, `rail_suspended`, `recon_not_clear`, `aml_restricted`, `destination_cooling` |
  | O4 | `screening → travel_rule_pending` | person re-screen clear and fresh KYT under the review score |
  | O5 | `screening → held` | `kyt_review`, `kyt_blocked`, `aml_restricted`, `flag_off`, `rail_suspended`, `recon_not_clear` |
  | O6 | `travel_rule_pending → cleared` | G15 passes; `cleared_destination_sha256` is written |
  | O7 | `travel_rule_pending → held` | any `travel_rule_*` reason, `vasp_suspended`, `kyt_blocked`, `aml_restricted`, `flag_off`, `rail_suspended`, `recon_not_clear` |
  | O8 | `cleared → custody_submitted` | the channel submits to A15 and posts the pay journal in one transaction |
  | O9 | `cleared → held` | the VASP asks for missing data or declines after clearance, the VASP is suspended, a re-screen blocks the destination, an A42 restriction, `flag_off`, `rail_suspended`, `recon_not_clear` |
  | O10 | `custody_submitted → signed` | A15's atomic signing point succeeds |
  | O11 | `custody_submitted → held` | a recoverable refusal at signing (see "Signing-time re-check"); the op is cancelled and the pay journal reversed |
  | O12 | `custody_submitted → failed` | an unrecoverable refusal at signing; the op is cancelled and the pay journal reversed |
  | O13 | `signed → broadcast` | the bundle is broadcast |
  | O14 | `signed → failed` | the bundle cannot be broadcast (expired, issuer pause or freeze seen before broadcast); pay journal reversed |
  | O15 | `broadcast → confirmed` | A27's indexer match (tx hash, from, to, amount, confirmations per A27's config); the settle journal is posted |
  | O16 | `broadcast → failed` | the transaction reverts (for example an issuer freeze); pay journal reversed |
  | O17 | `confirmed → reconciled` | the next clean R9 run includes it |
  | O18 | `confirmed → returned` | a matched inbound `return` is confirmed before R9 ran |
  | O19 | `reconciled → returned` | a matched inbound `return` is confirmed after R9 ran |
  | O20 | `held → screening` | release (see the hold table) |
  | O21 | `held → cancelled` | hold past `travel_rule.hold_max_hours`, a `decline` decision, a `flag_off` hold on the next tick, or a removed destination |

  Terminal: `failed`, `returned`, `cancelled`. A staff decision on a terminal transfer, or on a `reconciled` one, is `409 SCT_CLOSED`.

  **Inbound (`direction = in`), 6 states, 5 allowed pairs:**

  | # | From → to | When |
  |---|---|---|
  | I1 | `observed → matched` | an issuer mint to V5 or V6 that a `stablecoin.vault.fund` request expects (amount and funding ref), or a return matched by `return_ref` to an outbound SCT |
  | I2 | `observed → quarantined` | anything else (§6); an `inbound_quarantine` row is written |
  | I3 | `matched → confirmed` | A27's confirmations; the `stablecoin.fund` or `stablecoin.return` journal is posted |
  | I4 | `confirmed → reconciled` | the next clean R9 run includes it |
  | I5 | `quarantined → resolved` | its `quarantine_return` SCT is confirmed, or LEGL records a final decision |

  The forbidden-pair test derives its set from these two lists and nothing else: outbound 13 × 12 − 21 = 135 forbidden ordered pairs of distinct states, inbound 6 × 5 − 5 = 25, plus every self-pair and every cross-direction pair, each refused with `SCT_ILLEGAL_TRANSITION`.

  **Hold reasons, where they come from and how they end** (11 reasons):

  | `hold_reason` | Entered from | Released by (`held → screening`) | Investor-safe state |
  |---|---|---|---|
  | `travel_rule_missing_data` | `travel_rule_pending`, `cleared` | `release-hold` (`4E` KYCA → MLRO), only if G15 now passes | `processing`, message "The receiving provider has asked for more details. We're handling it." |
  | `travel_rule_no_ack` | `travel_rule_pending` | `release-hold`, or automatically when the acknowledgement arrives | `processing`, message "Waiting for the receiving provider to confirm your details" |
  | `travel_rule_declined` | `travel_rule_pending`, `cleared` | not released; `decline` or hold expiry cancels | `under_review`, no reason |
  | `vasp_suspended` | `travel_rule_pending`, `cleared` | `release-hold` after the VASP is re-approved | `under_review`, no reason |
  | `kyt_review` | `screening`, `cleared` (re-screen) | KYCA clears the review through A42, then automatic | `under_review`, no reason |
  | `kyt_blocked` | `screening`, `travel_rule_pending`, `cleared`, `custody_submitted` | never; cancelled at hold expiry or by `decline` | `under_review`, no reason |
  | `aml_restricted` | `requested`, `screening`, `travel_rule_pending`, `cleared`, `custody_submitted` | automatically on A42's restriction-lifted event (re-screen follows) | `under_review`, no reason |
  | `destination_cooling` | `requested` (distribution lines only; a withdrawal is refused at submit with `PDS_COOLING`) | automatically when cooling ends | `processing`, message "New wallets can receive payouts after <usable_from>" |
  | `rail_suspended` | `requested`, `screening`, `travel_rule_pending`, `cleared`, `custody_submitted` | automatically on `stablecoin.rail.resume` | `processing`, message "Stablecoin payouts are paused for now" |
  | `flag_off` | `requested`, `screening`, `travel_rule_pending`, `cleared`, `custody_submitted` | never; cancelled on the next tick, and the money falls back | `processing`, then `not_completed` |
  | `recon_not_clear` | `requested`, `screening`, `travel_rule_pending`, `cleared`, `custody_submitted` | automatically after a clean R9 and A20 `assertClear` | `processing`, no message |

  The investor-safe transfer states are exactly `processing`, `under_review`, `sent` (`signed`, `broadcast`), `completed` (`confirmed`, `reconciled`), `returned` and `not_completed` (`failed`, `cancelled`; the money is back in the AUD wallet or paid there). Every `under_review` body is byte-equal after id substitution whatever the reason, and a `not_completed` body is byte-equal whether the cause was AML, KYT, a decline, a failure or the flag. Every hold is still cancelled at `travel_rule.hold_max_hours` (default 72) if not released first.
- `StablecoinPayoutChannel.submit({ purpose, sourceRef, investorId, destinationId, amountCents, approvalId })` is the one entry A39.1 and A40.1 call. In one transaction it:
  - re-asserts G21 (`isEnabled('payouts.stablecoin')` and `evaluateG21('stablecoin_payments')`);
  - re-asserts A20's `assertClear(null, 'payout')` and the R9 status;
  - re-asserts G14 through A42's `restrictionStatus` (`payouts_held` or `full` refuses with the same generic hold A39 uses);
  - checks the destination's state and cooling period;
  - then creates the transfer in `requested` (a distribution line whose destination is still cooling is created and moved to `held: destination_cooling`, so the run proceeds).

  The worker job `stablecoin.transfers.advance` (every 15 s) runs the rest. On every tick it first re-reads the flag, the rail state, A20's status and A42's restriction for each non-terminal transfer before `signed`, so an auto-revert, a suspension, a recon break or a restriction holds every unsubmitted transfer within one tick. Then:
  - `screening`: a fresh person re-screen through A42's `screenForPayout`, plus a KYT result younger than `kyt.max_age_minutes` (default 60) for the destination.
  - `travel_rule_pending`: G15.
  - `cleared`: `cleared_destination_sha256` is copied from the destination.
  - Submit to A15 with the approval id and `cleared_destination_sha256` in the op's metadata. V5 transfers ride the run's approval, V6 withdrawal transfers the withdrawal's.

  **Ledger timing.** The pay journal (`stablecoin.distribution.pay` or `stablecoin.withdrawal.pay`) is posted in the same transaction as the custody submit, with the outbox carrying the custody call, so a crash never double-submits. It moves the value from `distribution_payable` or `investor_cash` into `stablecoin_in_transit`, not out of the vault mirror, because the coins are still in the vault. Only the indexer's confirmation posts `stablecoin.transfer.settle`, moving `stablecoin_in_transit` to the vault mirror, with the Transfer's block number. Any exit from `custody_submitted` or `signed` to `held` or `failed`, and `broadcast → failed`, posts `stablecoin.return` reversing the pay journal in the same transaction as the transition. So at every step: Σ pay journals − Σ returns = Σ settle journals (confirmed Transfers) + `stablecoin_in_transit` balance, and `stablecoin_in_transit` equals the Σ of transfers in `custody_submitted`, `signed` or `broadcast`.
- **Signing-time re-check.** This node's `DestinationClearanceProvider.clearDestination(op)` (A15.1) runs at `submitOp` and again inside A15's atomic signing point, for every V5 and V6 op. It returns `{ ok: false, code }` unless all of these hold at that moment:
  - `isEnabled('payouts.stablecoin')` and `evaluateG21('stablecoin_payments')` (`flag_off`, recoverable only by cancellation);
  - the rail is not suspended (`rail_suspended`) and A20's `assertClear(null, 'payout')` and the R9 status are clean (`recon_not_clear`);
  - A42's `restrictionStatus` for the investor is not `payouts_held` or `full` (`aml_restricted`);
  - the destination is `verified`, not removed, out of cooling, and its `destination_sha256` equals the transfer's `cleared_destination_sha256` (`P4_DESTINATION_CHANGED`, unrecoverable → `failed`);
  - the destination's KYT result is younger than `kyt.max_age_minutes` and under the block score (a stale result is recoverable: `held` then an automatic re-screen; a block is `kyt_blocked`);
  - `issuerStatus(asset)` shows no freeze of the vault or destination, no pause and no stale attestation (`SC_ISSUER_FROZEN`, `SC_ISSUER_PAUSED`, `SC_ATTESTATION_STALE`, unrecoverable → `failed`).

  Internal purposes (`vault_redeem`, `rebalance`, `v5_sweep`) check only the flag, the rail, R9 and the issuer status. On a refusal A15 leaves the op in `signing` with the refusal recorded; the channel then calls A15's `cancelOp` as the op's maker (the SVCA for `AUTO` ops, the request's maker otherwise, through the system path A15 provides for its own makers), moves the transfer by O11 or O12, and reverses the pay journal in the same transaction. A bundle already `signed` cannot be recalled (A15's rule): it goes on to `broadcast` and `confirmed` or `failed`.
- **Fallback money sequence.** When an outbound line ends `cancelled` or finally `failed` and its coins never left the vault:
  - **Withdrawal (V6):** `stablecoin.return` has already put the value back in `investor_cash`; A39's cancel path releases the reservation, and the investor sees `not_completed`. The coins stay in V6 as treasury inventory, so V6's mirror and `investor_cash` both stay whole; TRSY may redeem them later through `stablecoin.vault.redeem`.
  - **Distribution line (V5):** `stablecoin.return` has put the value back in `distribution_payable`, and the coins are still in the run's V5 earmark. The channel then drafts one `stablecoin.vault.redeem` request per run for the unpaid stablecoin total (`4E` TRSY → CFO, V5 human 2-of-5), sourced from the run's earmark and paying the AUD back into the Distributions client-money account. The `stablecoin.redeem` journal is posted when the burn is confirmed and A17's credit is matched. Only then does A40's `4E` manual payout credit `investor_cash` from `distribution_payable`, so the Distributions bank account never goes short. If the redeem has not settled within `stablecoin.fallback.redeem_max_hours` (default 24), TRSY may instead fund the manual payout from float through A39's float path (`4E`, journal `float.advance`), and the redeem's credit repays it (`float.repay`); A20's R10 sees both. A `v5_sweep` to V6 is the alternative only when the AUD comes from float.
  - A line or withdrawal `returned` after confirmation follows the same sequence: the inbound `return` lands the coins in the run's earmark while the run is open (else V5 `unallocated`) or in V6, and restores `distribution_payable` or `investor_cash`.
  - The run's earmark must be zero at `reconciling → closed`: every coin in it was either settled to an investor or redeemed or swept out.
- `evaluateG15(input)` in the module's `gate/` folder is a pure function, registered with A19's evaluator registry so G15 stops returning `not_applicable` once `PHASE=P3`. It returns `{ pass, reason_codes[], rule_version }`, and A19 writes the `GateEvaluation` row.
- **Parties for each purpose and investor type** (counsel to confirm the originator for distributions and the field sets; all rows are policy data). RealX is always the ordering institution (the sending VASP: legal name, ABN, AUSTRAC registration reference). For an inbound message RealX is the beneficiary institution.

  | Purpose | Investor type | Originator (IVMS101 party type) | Beneficiary (IVMS101 party type) |
  |---|---|---|---|
  | `distribution_line` | individual | the trust's trustee or RE (legal person) | the investor (natural person) |
  | `distribution_line` | joint | the trustee or RE (legal person) | both holders (two natural persons) |
  | `distribution_line` | SMSF, company, trust | the trustee or RE (legal person) | the investor entity (legal person) |
  | `withdrawal` | individual | the investor (natural person) | the investor (natural person) |
  | `withdrawal` | joint | both holders (natural persons) | both holders (natural persons) |
  | `withdrawal` | SMSF, company, trust | the investor entity (legal person) | the investor entity (legal person) |
  | `quarantine_return` | any | RealX as the vault holder (legal person) | the original sender as given in the inbound message (its party type), or the sending address only if none came |
  | inbound `return` | any | as received from the returning VASP; must reference `return_ref` | as received; must name the original originator |
  | `vault_redeem`, `rebalance`, `v5_sweep` | none | no message (issuer or RealX's own vault; counsel confirms for the issuer) | no message |

  Field sets, as `required_fields` policy rows keyed by party type and role: a natural person needs the name, the account or wallet number and one of address, national id, or date and place of birth; a legal person needs the legal name, the registration number (ABN, ACN or foreign equivalent), the registered address and the account or wallet number. The trustee's or RE's details come from A33's entity record for the vehicle; the investor entity's from A30's KYB record.
- The decision table:
  - **`registered_wallet`:** passes when the wallet is identity-registered to the same investor (A31's claim) and not frozen. No message is sent for a withdrawal, because both ends are the same verified person. For a `distribution_line` the originator is the trustee or RE, so whether a message or record is still required is a counsel question; the default is to keep an internal IVMS101 record with the party table's fields, without sending one. Counsel confirms at Specify whether RealX's non-custodial passkey wallets count as self-hosted wallets under the reformed rules. If they do, the row changes to the `self_hosted` path with the proof satisfied by the passkey registration.
  - **`self_hosted`:** passes with a valid, unexpired ownership proof, a KYT result under the review score, an internal IVMS101 record with the party table's fields complete, and an amount at or below `travel_rule.self_hosted_max_cents` if counsel sets one (default none). Refusals: `G15_SELF_HOSTED_PROOF_MISSING`, `G15_SELF_HOSTED_PROOF_EXPIRED`, `G15_SELF_HOSTED_CAP`, and the missing-data codes below.
  - **`vasp_account`:** passes only when a message with complete originator and beneficiary data for the party types in the table has been sent and the counterparty has acknowledged it before broadcast. The ordering rule "before or at the same time" is counsel's; the default is before. Refusals:
    - `G15_COUNTERPARTY_UNKNOWN` and `G15_VASP_NOT_APPROVED`;
    - `G15_MISSING_ORIGINATOR_DATA` and `G15_MISSING_BENEFICIARY_DATA`, each naming the field and the `party_type` (`natural_person` or `legal_person`);
    - rows for legal-person data missing: a distribution with the trustee's registration number or registered address missing (`G15_MISSING_ORIGINATOR_DATA`, `legal_person`, field named), and an SMSF, company or trust beneficiary with its legal name, registration number or registered address missing (`G15_MISSING_BENEFICIARY_DATA`, `legal_person`, field named);
    - `G15_MESSAGE_NOT_ACKNOWLEDGED` after `travel_rule.ack_timeout_minutes` (default 60);
    - `G15_COUNTERPARTY_DECLINED`;
    - `G15_SUNRISE_UNSUPPORTED`, applied by a policy row (default hold for the MLRO, counsel to confirm).
  - **Thresholds:** `travel_rule.threshold_cents` (default 0, meaning no de minimis, pending counsel on the reformed Australian rules). An aggregation window is policy data with default none.
- A refusal moves the transfer to `held` with its `hold_reason` from the hold table above. An A13 queue clock `travel_rule_hold_72h` goes to KYCA, and breach raises a Sev3. A hold past `travel_rule.hold_max_hours` (default 72) cancels the transfer (O21), and the money falls back through the fallback money sequence:
  - A withdrawal's reservation is released to `investor_cash` through A39's cancel path, and the investor is told the transfer could not be completed.
  - A distribution line fails into A40's retry (a new SCT with `retry_of`, same destination and amount, only once `issuerStatus` is clear) and then its `manual_payout` path, where the `4E` manual payout is the AUD credit funded as above, so paid still equals approved.
- `travel_rule_message` (id `TRM-<yyyymmdd>-<6 base32>`) carries:
  - `direction`, `transfer_id`, `counterparty_vasp_id`, `protocol_ref`, `return_ref`;
  - `originator_party_type` and `beneficiary_party_type` (`natural_person` or `legal_person`, one or more parties each, per the party table);
  - the IVMS101 originator and beneficiary fields, only in `_enc` columns, with the natural-person or legal-person field set named above;
  - `required_fields` (policy data per party type and role), `missing_fields[]`, `state` (`draft`, `sent`, `acknowledged`, `data_requested`, `complete`, `held`, `declined`, `returned`), timestamps and `retention_until` (default seven years, counsel).

  Every read masks through A04's `mask()`. `POST /v1/travel-rule/messages/{id}/unmask` is reason-coded and `requiresHuman`, grants 15 minutes, and is audited, as A30 does it.
- VASP directory (`travel_rule_vasp`): name, jurisdiction, registration reference, protocol endpoint (sandbox id), `due_diligence_state` (`pending`, `approved`, `suspended`), `sunrise` (whether the jurisdiction enforces the travel rule) and the last review date. Adding or approving a VASP is `travel_rule.vasp.approve` (`4E`, KYCA maker → MLRO checker). Suspending one is `SGL+R` (MLRO acts, CCO ratifies within 2 h), and the effector holds every open transfer to that VASP. A transfer to a VASP whose jurisdiction is not Australia writes an AML intake line with `international: true`, so A42 opens its IFTI item under `ifti_10bd`. This is what A42's note "A96 makes it real" refers to.
- Staff actions on a held transfer or message:
  - `request-data`: KYCA, no checker, audited.
  - `release-hold`: `travel_rule.transfer.release_hold` (`4E`, KYCA maker → MLRO checker), allowed only when G15 now passes on re-evaluation.
  - `decline`: `travel_rule.transfer.decline` (`4E`, KYCA → MLRO), with the money falling back as above.

  Each is registered through A14 with the checks attached as `GateRow` labels: "Originator data complete", "Beneficiary name matches verified investor", "Counterparty acknowledged", "KYT score below review", "No AML restriction (G14)".

**6. Inbound transfers and quarantine**

- The rail accepts inbound coins only from the issuer (a mint to V5 or V6 that a `stablecoin.vault.fund` request expects, matched by amount and funding ref) and from a counterparty returning a transfer this node sent (matched by `return_ref` in the travel-rule message).
- Anything else the indexer sees arriving at V5 or V6 is written as an `inbound_quarantine` row (`from`, amount, tx hash, KYT result, inbound travel-rule message if one arrives) with `state` `quarantined`. It is never a ledger credit to anyone and never counted in a vault's `stablecoin_mirror`. Note that R9 compares the mirror with the on-chain balance less quarantined amounts; see §7. That includes sub-cent dust and any amount of an asset not on the allow-list.
- A quarantined item raises an AML intake line (`unsolicited_inbound`) and a Sev2 for TRSY and MLRO. It can only be returned: `travel_rule.inbound.return` (`4E`, MLRO maker → CFO checker), which creates an outbound `quarantine_return` SCT from the vault that received it (V5 or V6, signed by that vault's human 2-of-5; never `AUTO`), destination = the sending address or the originating VASP's return address, through screening and G15 like any external transfer. Or, for an asset the rail cannot move, it is held for LEGL with a recorded decision. There is no route that credits it, and a test asserts it.

**7. R9, reconciliation and read models**

- `StablecoinSnapshotSource` (exported from `index.ts`) gives A20 at the recon cut block, per vault and asset:
  - the on-chain `balanceOf`;
  - the ledger `stablecoin_mirror` balance in cents (all sub-accounts), counting only settle, fund, redeem, rebalance and sweep journals whose Transfer block is at or before the cut block;
  - the `stablecoin_in_transit` balance and the list of transfers in `custody_submitted`, `signed` or `broadcast`, and of confirmed transfers whose block is after the cut block;
  - the Σ of `inbound_quarantine` amounts not yet returned;
  - every outbound Transfer event with its tx hash;
  - every `stablecoin_transfer` in `confirmed` or later, with approval id and journal id;
  - issuer mint and burn events against `stablecoin.fund` and `stablecoin.redeem` journals.

  R9 (A20.1) is clean only when all of the following hold:
  - on-chain balance − quarantined = ledger mirror × scale, exactly, at the cut block. Outbound transfers in flight do not break this: their value sits in `stablecoin_in_transit`, not in the mirror, and the mirror moves only on the settle journal with the Transfer's block number. R9 runs only once A27's indexer has confirmed past the cut block; until then the run is `deferred`, not a break (Sev2 if deferred past `recon.r9.max_defer_minutes`, default 60);
  - `stablecoin_in_transit` = Σ amounts of transfers in `custody_submitted`, `signed` or `broadcast` at the cut, exactly;
  - Σ pay journals − Σ `stablecoin.return` = Σ settle journals + `stablecoin_in_transit`, per vault and asset;
  - every outbound Transfer matches exactly one transfer row and one custody op with an approval id (unmatched is `unbound_transfer` Sev1);
  - every confirmed transfer has exactly one pay journal and one settle journal with the same approval id and amount;
  - for every closed distribution run, its V5 earmark is zero;
  - mint and burn totals equal the fund and redeem journals.

  Any break blocks every stablecoin submit (and every signing, through the provider) through `assertClear(null, 'payout')` scoped to the rail, and leaves the AUD rail untouched. R8 for a run with stablecoin lines counts those lines' journals too (A20.1). A40's `reconciling → closed` therefore needs R8 and R9 clean and the run's earmark at zero.
- Read routes (cursor pagination, `x-realx-scope`):
  - Investor side:
    - `GET /v1/me/payout-destinations`, with only the investor-safe destination states of §4 (`review` and `blocked` both read `unavailable` with byte-equal bodies);
    - `GET /v1/me/payout-preferences`;
    - `GET /v1/me/stablecoin-transfers`, with only the investor-safe states and messages of the hold table in §5 (`processing`, `under_review`, `sent`, `completed`, `returned`, `not_completed`). `under_review` covers the KYT, AML, VASP-suspension and counterparty-decline holds and never carries a reason, because a counterparty's decline may itself be AML-driven. A missing counterparty acknowledgement or a data request may be said plainly because it is a data matter, the same distinction A39 draws for the name mismatch.
  - Staff side:
    - `GET /v1/travel-rule/messages?direction=&state=&counterparty=&age=`, returning the panel's table shape;
    - `GET /v1/travel-rule/messages/{id}`;
    - `GET /v1/travel-rule/vasps`;
    - `GET /v1/travel-rule/self-hosted-proofs?state=`;
    - `GET /v1/travel-rule/quarantine`;
    - `GET /v1/stablecoin/vaults`: V5 and V6 on-chain balance, ledger mirror, quarantined, Δ, last R9 run id and state;
    - `GET /v1/stablecoin/transfers?state=&purpose=`;
    - `GET /v1/travel-rule/kpis`: messages today, missing-data holds, holds past SLA, self-hosted proofs pending, quarantine count, V5 and V6 Δ.
- Empty and edge states:
  - With the flag off, every write returns `403 G21_FLAG_OFF` (or `G21_PHASE_NOT_REACHED` before `PHASE=P3`). Every read returns `200` with `rail_state: "off"`, `reason` from A21 and empty lists, so the console shows the P3 preview and never a broken page.
  - No messages is `{ items: [], next_cursor: null }`.
  - An unknown id is 404.
  - A decision on a terminal or `reconciled` transfer is `409 SCT_CLOSED`.
  - A sponsor, partner, SUPP or IRCM principal gets 404 on every travel-rule and quarantine route, because the need-to-know ACL is A42's second RLS policy applied to this module's tables through A04's registry.

**8. Agents and tokens (D04, G22)**

- Every route carries `x-realx-scope` in M16 or M20. The new scope cells (`stablecoin.destinations.read`, `stablecoin.destinations.remove`, `stablecoin.preferences.write`, `stablecoin.transfers.read`, `travel_rule.read`, `travel_rule.draft`, `stablecoin.vault.draft`) are data rows added by A03.1 (§1) and read through A12's documented extension list, as A42 did for its M20 scopes. Every route passes A12's `agentAuth` middleware and G22 evaluator unchanged.
- An investor's agent can:
  - read destinations, preferences and transfers;
  - remove a destination;
  - set the payout preference among verified destinations, only once counsel's `delegable.ts` cell for `stablecoin.preferences.write` reads `true`; while it is seeded `"counsel"` the agent gets `SCOPE_PENDING_COUNSEL` (§4);
  - request a stablecoin withdrawal through A39.1's route within its mandate's amount band. G22 refuses above the band, and an above-cap confirmable withdrawal returns A12's `428 CONFIRMATION_REQUIRED`.

  It can never add a destination, prove ownership, unmask, or act on a V5 or V6 vault. On-chain it acts only through its delegated session key on the passkey smart wallet (A29). The session-key policy never lists the vaults or the stablecoin contract's `transfer` from the investor's wallet unless counsel and the founder add investor-initiated coin transfers later, which is not in this node.
- A staff agent can list, read and draft: `request-data`, and drafts of `release-hold`, `decline`, `vasp.approve`, `inbound.return`, `vault.fund`, `vault.redeem`, `vault.rebalance` and `v5.sweep` as maker within its human's role. A14 refuses it as checker, ratifier or approver before scope evaluation (`APR_CHECKER_MUST_BE_HUMAN`). A15's P9 refuses any token or MCP credential at signing (`P9_TOKEN_CREDENTIAL`). Segregation of duties counts the agent as its human, so a KYCA's agent's draft cannot be released by that KYCA.
- `requiresHuman: true` routes: `POST /v1/me/payout-destinations`, `POST …/proof`, `POST /v1/travel-rule/messages/{id}/unmask`, the `SGL+R` act on `stablecoin.rail.suspend` and every approve, ratify or sign. The MCP tool catalogue (A104) lists the delegable tools and none of these. Every agent action is audited as "agent <token prefix> on behalf of <person>", appears in the person's activity, and is visible on ADM-35 (A77).

**9. Audit, events and errors**

- One hash-chained audit event (A10) per destination change, proof, KYT result, preference change, transfer transition, message state change, VASP decision, quarantine item, vault operation, rail suspension and unmask grant. Each event carries `actor_id`, `on_behalf_of_id`, `approval_id` where one exists, and the SCT, TRM, PDS or APR id as `object_ref`. KYT scores and AML hold reasons carry `sensitivity = need_to_know`.
- Outbox events: `stablecoin.destination.added|cosign_pending|verified|blocked|removed`, `stablecoin.run.earmarked`, `stablecoin.preference.changed`, `stablecoin.transfer.requested|held|cleared|confirmed|failed|returned|cancelled`, `travel_rule.message.sent|acknowledged|declined|data_requested`, `travel_rule.vasp.approved|suspended`, `stablecoin.inbound.quarantined`, `stablecoin.vault.funded|redeemed|rebalanced|swept`, `stablecoin.rail.suspended|resumed` and `payment.aml_intake` (source `stablecoin`) for A42. Events carry ids and codes, never PII or IVMS101 data.
- Stable error codes in OpenAPI:
  - `SC_DECIMALS_UNSUPPORTED`, `SC_ASSET_NOT_ALLOWED`, `SC_ISSUER_FROZEN`, `SC_ISSUER_PAUSED`, `SC_ATTESTATION_STALE`, `SC_RAIL_SUSPENDED`, `SC_RECON_NOT_CLEAR`, `SC_V5_UNFUNDED`;
  - `PDS_BENEFICIARY_NAME_MISMATCH`, `PDS_PROOF_INVALID`, `PDS_PROOF_EXPIRED`, `PDS_COOLING`, `PDS_IN_USE`, `PDS_BLOCKED`, `PDS_COSIGN_EXPIRED`, `PDS_SIGNATORY_REQUIRED`, `PDS_IMMUTABLE_FIELD`;
  - `P4_DESTINATION_CHANGED` (returned by this node's `DestinationClearanceProvider` and surfaced by A15's P4);
  - `SCT_ILLEGAL_TRANSITION`, `SCT_CLOSED`, `SCT_SUBCENT_AMOUNT`;
  - `TRM_UNMASK_GRANT_INVALID`;
  - the `G15_*` codes above;
  - passed through: A21's `G21_*`, A42's generic hold, A15's `P*` codes, A11's `AGNT_NOT_ALLOWED` and `FRESH_ASSERTION_*`, and A12's `G22_*`, `SCOPE_PENDING_COUNSEL` and `CONFIRMATION_REQUIRED`.

**10. Console panel (`apps/console/app/(travel-rule)/` and `src/features/travel-rule/`)**

- One page group under the "Risk & records" sidebar group, built from A05's components and A53's `ConsoleTable`, `MaskedCell`, `WhenCell`, `GateRow` and `HardwareKeyConfirm`, and nothing new. It has five tabs:
  - **Messages:** a KPI strip, then the table with columns Transfer / Direction / Counterparty / Originator ✓ / Beneficiary ✓ / Rule / State / Age; the detail view has masked IVMS101 fields, a timeline and the actions.
  - **Holds:** transfers in `held`, with the A13 clock.
  - **VASP directory.**
  - **Self-hosted proofs.**
  - **Vaults and quarantine:** the V5 and V6 strip with on-chain, ledger, quarantined and Δ, the last R9 run, the quarantine list with Return, and the rail suspend and resume controls.
- Every decision goes through `HardwareKeyConfirm`, with the server's payload hash, the checks as `GateRow`s and the SoD note. The maker sees no decision control (`not-eligible`). An AGNT persona sees the grey "Agents prepare. A person approves." banner in place of every Approve, Release, Decline, Return, Suspend and Resume.
- With the flag off, every tab shows the P3 preview: a brick pill "P3", the sentence from A21's reason ("Stablecoin payouts are off · no licence binding (G21)"), empty tables and no action controls.
- Route: `/admin/travel-rule` (folder `app/(travel-rule)/travel-rule/`, with tab routes `/admin/travel-rule/holds`, `/vasps`, `/proofs` and `/vaults`), a sibling of A75's `/admin/aml` so neither node edits the other's folder. Its nav item sits under ADM-25 in the "Risk & records" group through A51.1's `screens.json` entry.
- Pages export `screen: 'ADM-25'`, `subsurface: 'TR'` and `designKey: 'design-ext:ADM-25-TR'`. ADM-25 is owned by A75; the panel is a P3 sub-surface of M20, and the `subsurface` field tells the Screen Coverage Matrix not to count it as a second ADM-25 or as a new screen, so the 91-screen count does not change.
- States drawn and tested: default, empty, flag-off preview, hold past SLA (amber clock, then red), quarantine present (red strip), R9 break (red Δ and the rail-blocked banner), rail suspended (the ratify countdown), not-eligible, agent persona, loading and error.

## Owned paths (other nodes do not edit these)

- <workspace>\app\packages\adapters\stablecoin\
- <workspace>\app\services\api\src\modules\travel-rule\
- <workspace>\app\apps\console\app\(travel-rule)\
- <workspace>\app\apps\console\src\features\travel-rule\

Inside the adapter package, by convention:
- `manifest.ts`, `interface.ts`, `sandbox/` (with `contracts/TestAUDStable.sol`), `conformance/`;
- `travel-rule/` with the same four parts;
- `README.md`, which states both swap points in plain words.

Inside the module:
- `destinations/`, `preferences/`, `transfers/` (the channel, transition table and worker), `kyt/`, `messages/`, `vasps/`, `quarantine/`, `vaults/`, `gate/` (`evaluateG15`), `recon/` (`StablecoinSnapshotSource`), `jobs/`, `events.ts`, `errors.ts`, `index.ts` (the only import surface).

Tables are registered through A04's module-schema registry. Cross-node changes are the loops in §1, never edits here.

## Design references

- `design-ext:ADM-13 vaults card` from [[A51 - Design extension B- operator console screens not in the prototype]] (ADM-13 Wallets, custody and recovery). It draws one `KeyValueList` card per vault: purpose, quorum ("V2 token agent · 2-of-3"), rotation, allow-list, timelocks, state. The design shows V5 and V6 as absent, and A72 renders them as grey "reserved (P3)" cards from A15's `state = reserved`. This node's contract is that, once `fill-vaults` executes, A15's `GET /custody/vaults` returns V5 and V6 with every field the V2 card shows (purpose "Distribution payouts (P3)" or "Treasury (P3)", quorum, rotation class `warm`, allow-list, P6 and P7 parameters, state `active`). A72's card then renders them with no change to A72's code. The node QA checks this by screenshot against the V2 card's layout. The Risk column on ADM-13's wallet table stops reading "not in P1" for destinations this node screened, because A22.1 returns a score in P3.
- `design:adToken` (`<workspace>\design\RealX App.dc.html`, lines 706-716, screen key `adToken`, ADM-14). The "Waiting for signatures" row pattern (mono `id · approval id`, the op name, the mono detail, the signature bar and the Sign button) is how a V5 or V6 human-quorum (2-of-5) transfer, a vault fund, redeem or sweep, or a quarantine return appears in A72's signing queue. This node supplies the op kind `stablecoin_transfer` with detail "<destination short> · <SCT id>" so that row renders unchanged. The `adToken` sub-line "approval ID and a 2-of-3 custody quorum before broadcast" (line 884) is the pattern V5 and V6 follow (an approval ID and a custody quorum before broadcast); their quorum is 2-of-5 per spec point (a), so the signature bar shows 5 slots, which the row pattern already supports.
- `design:adAml` (line 887, ADM-25) is the DataTable and tone pattern the Messages tab follows, and its sub-line "Owned by the MLRO" is the ownership the need-to-know ACL enforces. `adApprovals` (line 655, ADM-30) has the checks list and the note "Maker-checker (G13): you are not the maker, so you can approve. Approval ID is bound to the signing request." These are the wording standard for the release-hold, decline, VASP and return requests. `adTreasury` (line 886, ADM-18) KpiTiles are the pattern for the V5 and V6 balance strip.
- The undesigned travel-rule panel has no product-map screen code. The research pack's admin spec §20.6 "Travel Rule console" (messages in and out, missing-data holds, self-hosted proofs, states pending / complete / held / declined / returned) is its content source. A51.1 adds one frame `design-ext:ADM-25-TR` in the dark admin shell, with its `screens.json` entry in "Risk & records" at `phase: P3`, so A53's generated `nav.ts` picks it up with no hand edit. If the frame is not drawn by Specify, the builder composes the panel only from A05's components and A51's `blocks.md`, and the node QA records a DF finding against A51.
- Tokens: cream `#F5F2EC`, ink `#17191C`, green `#1F5E4B`, brick `#9A4524`, alert red `#A3312A`, the `TONE` map at line 740, and the admin sidebar `#1C1E22`. Tones map as follows:
  - `complete`, `acknowledged`, `verified`, `confirmed`: green.
  - `held`, `data_requested`, `review`, `pending_proof`: amber.
  - `declined`, `blocked`, `quarantined` and any Δ ≠ 0: red.
  - `returned`, `cancelled`, `removed`: grey.
  - The P3 preview pill: brick.

  Geist Mono is used for SCT, TRM, PDS, APR and VASP ids, addresses (truncated `0x5e…03`), tx hashes and amounts; Newsreader for the page title.
- Flag label finding: A51's ADM-34 mock says `stablecoin.rails` where A08's key is `payouts.stablecoin`. That is A69's recorded finding (e). This node uses A08's key only.
- Investor surfaces this node feeds: the POR-05 "Payout rail" card, the "Add a wallet" flow with the ownership-proof step, and the investor-safe transfer states. They are `design-ext` frames requested from [[A50 - Design extension A- investor and sponsor screens not in the prototype]] (A50.3) and built in A59.1. This node fixes their data contracts only.

## Requirements covered

- **Product-map screens:** none owned. ADM-13's V5 and V6 cards (A72), ADM-14's signing queue for stablecoin ops (A72), ADM-18's treasury figures (A74, which sees the vault funds as client-money-adjacent balances once A16.1 lands) and ADM-35's agent view (A77) consume this node. "Done" for those screens means every field they need exists here, checked by contract tests they reuse unchanged. The travel-rule panel is a P3 sub-surface of ADM-25 owned by this node.
- **Journeys:** none owned. "Done" means the P3 variants of J06 (a monthly run with some lines paid in coins from V5) and J15 (a stablecoin withdrawal from V6) run at service level in Docker with the flag bound to a sandbox licence record. The failure paths (missing travel-rule data, KYT block, AML restriction, issuer freeze, R9 break) end in a hold or the AUD fallback, never in a lost or doubled payment.
- **Gates:**
  - **G14 AML monitoring and reports:** done when:
    - every stablecoin payout re-screens the person, KYT-screens the destination and checks A42's `restrictionStatus` before custody submit;
    - a restriction placed after clearance refuses at submit and at signing;
    - blocked destinations and unsolicited inbound coins reach A42 as intake lines;
    - transfers to foreign VASPs create IFTI items;
    - no KYT score, hold reason or case fact reaches an investor, sponsor, SUPP or IRCM response.
  - **G15 Travel rule:** done when:
    - A19 evaluates `evaluateG15` for every stablecoin transfer in P3;
    - no transfer to a VASP broadcasts without complete data and an acknowledgement;
    - no transfer to a self-hosted wallet broadcasts without a valid ownership proof;
    - every missing-data hold has a clock, a KYCA owner and a `4E` release;
    - inbound unsolicited transfers are quarantined and only ever returned;
    - message PII is encrypted, masked and retained per policy.
  - **G21 Licence perimeter:** done when:
    - every route and job refuses while `payouts.stablecoin` is off, unbound or before `PHASE=P3`;
    - an auto-revert of the flag on a licence lapse stops every unsubmitted transfer within one clock tick;
    - no seed, env override outside `local|test` or direct write can turn the rail on.
- **Modules:**
  - **M16 Treasury and client money:** done when V5 and V6 funding, redemption and rebalancing exist as `4E` requests with ledger journals, stablecoin payouts for withdrawals and distribution lines go through one channel bound to their approval ids, and the vault balances reconcile under R9 with Δ 0.
  - **M20 AML cases:** done when KYT results, wallet-risk intake, the travel-rule message store, the VASP directory, the quarantine and the IFTI hand-off exist as this API under the need-to-know ACL, and A42 needs nothing more from this rail.
- **State machine SM-08 Distribution run:** done when a run with stablecoin lines still satisfies every SM-08 invariant.
  - `funded` requires the run's own V5 earmark (`stablecoin_mirror` V5 `run:<run id>`, posted by `earmarkRun`) to equal the run's stablecoin total, in addition to A40's `distribution_payable` check (A40.1). Earmarks are per run, so two concurrent runs, of the same or different tenants, can never count the same coins.
  - `executing` pays stablecoin lines only through the run's approval id and file hash.
  - A stablecoin line that fails or is held past policy moves the run to `partially_failed`, then `retrying` with the same destination and amount, then `manual_payout`, where the AUD credit is the fallback.
  - `reconciling → closed` requires R8 and R9 clean, paid equal to approved and the run's earmark at zero (every unpaid coin redeemed or swept by the fallback money sequence of §5).
  - This node adds no SM-08 state or transition, and A40's forbidden-transition tests still pass with stablecoin lines present.
- **Founder requirement D04:** done when every delegable action of this node works under a scoped token with parity to the screens, the destination and proof acts refuse agents before scope evaluation, staff agents can only draft, investor agents move coins only through the withdrawal mandate band and never through a vault, and every agent action is audited on behalf of its human.

## Gauntlet loop for this node

- **Specify.** The frozen spec is this note plus:
  - the settled spec points (a) V5 and V6 quorums and signing policy and (b) the on-chain binding choice;
  - the loops in §1, merged and their hashes recorded;
  - A15's P1-P10 contracts with A15.1;
  - A16's catalogue with A16.1;
  - A20's rule interface with A20.1;
  - A21's activity-to-flag map (`stablecoin_payments` → `payouts.stablecoin`);
  - A22.1's `screenWallet` P3 contract and magic prefixes;
  - A39.1's and A40.1's destination contracts;
  - A42's `restrictionStatus`, `screenForPayout`, intake shape and IFTI policy row;
  - A14's action and policy registries;
  - A13's `travel_rule_hold_72h` row;
  - the policy rows named here.

  The node QA records `specVersion` and those hashes in the manifest. It then writes the hidden tests to `<workspace>\gauntlet\hidden\A96\` as Vitest suites over the module against Docker Postgres, the local Hardhat chain with the test token, A15's custody sandbox, A17's sandbox bank, A22.1's sanctions sandbox and the travel-rule sandbox network, under the injectable clock.
- **Build (Grok 4.7 xhigh, split by the planner).** Suggested shape: four parallel sub-agents on disjoint folders, then integration.
  - (1) The adapter package: both manifests, the test token and its deploy through A27, the sandbox issuer tied to A17's sandbox bank, the VASP network, magic inputs, `SandboxControl`, both conformance suites and the README swap points.
  - (2) `destinations/`, `preferences/`, `kyt/` and `vaults/`: proofs (EIP-191 and EIP-1271 on the local chain), the name match through A17's table, cooling and notices, the nightly re-screen, the `fill-vaults` script through A15's P8 path, the fund, redeem, rebalance and rail suspend and resume requests and effectors.
  - (3) `transfers/`, `gate/`, `messages/`, `vasps/` and `quarantine/`: the channel, the transition table, the advance worker with the journal-plus-outbox pattern, `evaluateG15` and its A19 registration, holds and clocks, release, decline and return requests, IFTI intake, masking and unmask grants.
  - (4) `recon/`, the HTTP layer and the console panel: `StablecoinSnapshotSource`, routes with `x-realx-scope` and `requiresHuman`, OpenAPI entries and error catalogue, the five tabs with every state in §10, the contract tests A72, A74 and A77 reuse, and fixtures for the personas (an investor with each destination kind, a joint account, an SMSF signatory, KYCA, MLRO, CCO, CFO, TRSY, SECO, SUPP, IRCM, an SP-OWN, and an AGNT of each).

  Integration seeds a sandbox licence record covering `stablecoin_payments` and binds the flag through A21's `4E` path. It then drives, end to end:
  - a run of 186 lines with 40 stablecoin lines;
  - a V6 withdrawal inside the `AUTO` band and one above it;
  - a VASP transfer that holds for missing data and is released;
  - a foreign VASP transfer with its IFTI item;
  - a self-hosted transfer;
  - an unsolicited inbound return;
  - a vault fund and redeem;
  - an R9 clean run and a planted break.

  It finishes with `npm run verify` and `app/evidence/A96.json`.
- **Challenge (node QA, Opus 5.5 high).** Runs the hidden tests, then works through the node-specific challenges below by hand against the two-process stack. Then:
  - a design-fidelity pass: screenshots of the five tabs in each state against the A51.1 frame, or against A51's blocks if the frame is missing, and of A72's V5 and V6 cards against the V2 card;
  - greps of API and worker logs, the outbox, the audit table, `gate_evaluation` inputs, OpenAPI examples, SDK types and the evidence folder for planted IVMS101 names, dates of birth and KYT scores;
  - the `Date.now()` grep;
  - the OpenAPI document diffed against the routes;
  - an MCP catalogue check that no `requiresHuman` route is a tool;
  - a check that only the owned paths and the merged §1 loops changed.

  It writes `app/evidence/A96.qa.md`.
- **Gate.** Passing publishes the hidden tests into `app/`. It unlocks no node, but it is a precondition for HG-LAUNCH's P3 switch-on review. Max three fix loops, and each loop re-runs the full hidden set and the failing challenges.
- **Escalation trigger.** Any of these escalates to the user:
  - the same assumption failing three times;
  - a fix that needs a change beyond the §1 loops in A15, A16, A20, A39, A40 or A42 (a spec defect; reopen upstream);
  - any coin leaving V5 or V6 without an approval id, to a destination that is not the investor's own, or while the flag is off (Critical, stops the milestone);
  - any agent approving, releasing, ratifying or signing by any route (Critical);
  - any correctness question that depends on a counsel answer not yet given (record in HG-LEGAL and ask the user).

## Node-specific challenges (node QA)

- **Third-party destination by the side door (money).** An investor adds a `vasp_account` whose beneficiary name is their spouse's, then a `self_hosted` wallet with a proof signed by a different key, then asks their agent to add a destination. The node QA then edits a `payout_destination.address` by SQL after verification, and asks a V6 withdrawal to a raw address in the body. Expected, in order:
  - `PDS_BENEFICIARY_NAME_MISMATCH`;
  - `PDS_PROOF_INVALID`;
  - `AGNT_NOT_ALLOWED` before scope evaluation;
  - the address edit is refused by the immutability trigger of §4 (`PDS_IMMUTABLE_FIELD`); in a copy of the database with the trigger dropped, the edited row's recomputed hash no longer equals the transfer's `cleared_destination_sha256`, so this node's `DestinationClearanceProvider` refuses at A15's P4 with `P4_DESTINATION_CHANGED`, the transfer goes `failed` and the pay journal is reversed;
  - `422` with no address field accepted.

  Any coin reaching an address that is not the investor's own fails the node.
- **Travel-rule race (compliance, G15).** A transfer to `VASP-SIM-AU-OK` is cleared. Before custody submit, the VASP sends `requestMissingData` for the beneficiary's date of birth. Separately, a transfer to `VASP-SIM-TIMEOUT` sits for 59 and then 61 minutes, and one to `VASP-SIM-NO-SUNRISE` waits for an acknowledgement that never comes. Expected:
  - the first moves back to `held` and nothing broadcasts;
  - the second holds at 61 minutes and not at 59;
  - the third follows the `G15_SUNRISE_UNSUPPORTED` policy row;
  - after 72 hours the withdrawal's money is back in `investor_cash` and the distribution line has gone through A40's retry to `manual_payout` with an AUD credit;
  - Σ paid equals approved in both cases.
- **Flag lapses mid-run (G21, money).** A run of 186 lines with 40 stablecoin lines is `executing` at line 20 when the licence record's CAR ends. A21 auto-reverts `payouts.stablecoin` within one tick. Expected: no stablecoin line after the lapse is submitted to custody; lines already `signed` still `confirm` or `fail` (a signed bundle cannot be recalled, matching A15's rule) and are reconciled; the unsent lines fall to the AUD fallback through `manual_payout`; R8 and R9 are clean at close. Then a SUPER tries to turn the flag back on with no binding, and gets `G21_NO_BINDING`.
- **Unsolicited inbound and dust (money, G14).** The node QA sends V6 a sub-cent amount of the test token, 5,000 whole coins from an address beginning `0x5A5A`, and a coin of a second, non-allow-listed test token. None may become a ledger credit or count in the mirror. Each must be a quarantine row with an AML intake line. R9 must stay clean, because it subtracts quarantine. The return must need a `4E` MLRO → CFO decision and V6's 2-of-3, and a staff agent's attempt to approve the return must be refused by A14.
- **Issuer freezes the vault (money).** With `SIM-ISSUER-FREEZE` on V5 in the middle of a run, the next transfer reverts. Expected:
  - the transfer goes to `failed`, and the journal is reversed by `stablecoin.return` in the same transaction as the failure;
  - `SC_ISSUER_FROZEN` is raised with a Sev1 to TRSY and CFO;
  - A20's R9 sees no unmatched transfer and no ledger delta;
  - no retry is attempted against a frozen vault until `issuerStatus` clears;
  - the rail suspend (`SGL+R`) can be acted by CCO with the CFO ratifying.
- **R9 zero tolerance (money).** The node QA plants a 1-base-unit transfer out of V5 signed directly with the sandbox kms key, outside custody. R9 must report `unbound_transfer` Sev1 on the next run, every stablecoin submit must refuse with `SC_RECON_NOT_CLEAR`, AUD withdrawals must continue, and ADM-17 must show the break through A20. A second plant, a ledger journal edited by 1 cent, must be refused by A16's immutability before R9 ever sees it.
- **Tipping-off through the new rail (compliance).** Two identical investors each request a stablecoin withdrawal. One's destination scores `0x5B5B` (review), and the other is restricted by A42 under `SGL+R` after clearance. The node QA diffs every investor-facing response, the notification outbox, status codes and latency over a scripted hour, as each investor and as their agents. Both must read `under_review` with byte-equal bodies after id substitution. SUPP and IRCM see nothing from this module. A KYT score appearing in any non-need-to-know read fails the node.
- **Agent authority (D04, G22).** An investor's agent with a A$2,000 daily band requests two A$1,500 stablecoin withdrawals 10 ms apart: exactly one passes. The agent then tries `POST /payout-destinations`, `POST …/proof` and a session-key call to the stablecoin contract from the passkey wallet, and all are refused. A KYCA's agent drafts `release_hold` on a transfer, and the KYCA tries to approve their own agent's draft: refused as maker (the agent counts as its human). An MLRO approves it in person with a hardware key, and both audit rows carry `on_behalf_of`.
- **Design fidelity.** With the flag off, every tab must render the P3 preview with A21's reason and no action control. With it on and the standard seed, the Messages table must use the `adAml` column rhythm and tone pills. A72's V5 and V6 cards must match the V2 card's structure field for field. The signing-queue row for a V6 transfer must render in `adToken`'s "Waiting for signatures" layout with "<destination short> · <SCT id>". Any new status string, any bare 4xx, or any field the A72, A74 and A77 contract tests need that is missing fails the node.

## Pass criteria

- The hidden tests in `<workspace>\gauntlet\hidden\A96\` pass 100 % on three consecutive runs with different seeds. They include:
  - a fast-check property over at least 10,000 random sequences of destination adds, proofs, KYT results, travel-rule events, holds, restrictions, flag toggles, issuer freezes and custody outcomes, asserting that no coin leaves V5 or V6 without an approval id, to a destination that is not the investor's own, without a passing G15 evaluation, or while the flag is off;
  - Σ ledger stablecoin payouts equals Σ confirmed vault Transfers at every step;
  - every cancelled or failed payout ends with its money back in `investor_cash` or on A40's manual path, never lost or doubled.
- Transition coverage: every state and allowed transition of the `stablecoin_transfer` table is exercised at least once. 100 % of non-listed pairs are refused with `SCT_ILLEGAL_TRANSITION`, and the count of forbidden pairs equals the count derived from the table.
- G15 decision table: one test per row and per reason code (each destination kind × data complete or missing × acknowledged, declined or timed out × sunrise or not × proof valid, expired or absent). Every refusal names its code and field, and A19 stores a replayable `GateEvaluation` for each.
- R9: for 1,000 random vault histories the rule is clean exactly when the four equalities in §7 hold. Each planted class (unbound transfer, 1-base-unit balance delta, missing journal, mint without journal, quarantine counted as balance) yields its break type and severity. A break blocks the stablecoin rail and not the AUD rail.
- G21: with the flag off, unbound, or before `PHASE=P3`, 100 % of write routes and jobs refuse with A21's code and read routes return `rail_state: "off"`. An auto-revert stops unsubmitted transfers within one clock tick, with exactly one audit event and no further custody op.
- Vaults: V5 and V6 reach `active` only through A15's P8 path (asserted by the absence of any direct `custody_accounts` write in the module and by P8's trigger). The fund, redeem, rebalance and return actions each create their `4E` request with the checks named here. Every custody op carries its approval id and signs only under the vault's quorum or `AUTO` policy.
- Agent parity: every delegable route succeeds under a token whose scope covers it and fails with `G22_SCOPE_MISSING` under one that does not. Every `requiresHuman` route refuses an AGNT before scope evaluation. No agent reaches an approve, ratify or sign step through any route or MCP tool. Every agent-made event carries `on_behalf_of_id`, and A100's parity catalogue lists every route of this module with its UI counterpart or `api-only-until-A59.1`.
- PII and tipping-off: a scan of logs, the outbox, audit, `gate_evaluation` inputs, OpenAPI examples, SDK types and the evidence folder finds zero planted IVMS101 names, dates of birth or KYT scores. Masked fields equal A04's `mask()` snapshots. Unmask grants work only within 15 minutes, from their session, for their fields. The investor-facing bodies of a KYT-review hold and an AML-restriction hold are byte-equal after id substitution.
- Console: the five tabs render every state in §10, and each state's screenshot is reviewed against the A51.1 frame (or A51's blocks with the DF finding recorded). A72's V5 and V6 cards match the V2 card's field list. The axe check shows zero serious violations, and the production-bundle grep finds no sandbox route.
- Conformance and swap points: both adapter conformance suites pass against the sandbox. `SWAP-POINTS.md` contains the `stablecoin` and `travel-rule` rows with HG-VENDOR. The README names what changes when AUDM, AUDD or A$DC and a travel-rule provider are selected.
- Line coverage is at least 90 % on the module and the adapter package. The module boundary lint passes. `npm run verify` and `openapi:check` are green. Only the owned paths and the merged §1 loops changed. `app/evidence/A96.json` and `A96.qa.md` exist and match disk.

## Human gates touched

- None crossed. The node runs on the local chain with a test token that carries no value, A15's custody sandbox with regenerable keys, A17's sandbox bank, A22.1's sanctions sandbox and the in-process travel-rule network. No issuer, KYT vendor, travel-rule provider, AUSTRAC system or mainnet is touched.
- **HG-LEGAL (counsel questions recorded, not decided here):**
  - which licence or relief covers stablecoin payouts (ASIC Instrument 2025/867 for distributing eligible stablecoins, and whether the CAR arrangement or the P2 RE's licence covers the activity `stablecoin_payments`);
  - AUSTRAC registration for the reformed virtual-asset designated services;
  - the Australian travel-rule threshold, the required IVMS101 fields, the "before or at the same time" ordering, the aggregation window and the treatment of counterparties in jurisdictions without the rule;
  - whether RealX's non-custodial passkey smart wallets are self-hosted wallets for travel-rule purposes;
  - whether the signature proof is enough or a micro-transfer test is required, and the proof validity period;
  - whether coins in V5 and V6 are client money or client property, and the custody standard that applies;
  - whether each sponsor's trust deed permits distributions in a stablecoin;
  - IFTI treatment of transfers to foreign VASPs;
  - the retention period for travel-rule data;
  - whether setting the payout preference to stablecoin is delegable to an agent (proposed yes; the cell is `"counsel"` in A03's `delegable.ts` until answered, and is refused to agents with `SCOPE_PENDING_COUNSEL` if counsel says no).
- **HG-VENDOR (later):** the stablecoin issuer relationship (AUDM, AUDD or A$DC), the KYT vendor (Chainalysis, TRM Labs or Elliptic behind A22's `screenWallet`), the travel-rule provider and the custody vendor's support for V5 and V6. Each implementation must pass this node's conformance suites and hidden tests unchanged before it is registered.
- **HG-KEYS (later):** the V5 and V6 key ceremonies, witnessed by SECO and the custodian, with the quorums settled at spec point (a).
- **HG-PROD and HG-AUDIT (later):** the production stablecoin contract address and chain are a manifest entry after mainnet approval. The test token is never deployed to mainnet. If spec point (b) chooses the `AgentGateway` instances, their deployment is inside A25's audit scope, and no new Solidity is audited for this node.
- **HG-PENTEST (later):** destination adds and proofs, the channel, the quarantine and the console panel are in scope of the external penetration test before the P3 switch-on.
- **HG-LAUNCH (later):** turning `payouts.stablecoin` on is a founder act after the licence binding exists, the questions above are answered, A50.3, A59.1 and A51.1 have landed, and a P3 launch update like A94's has passed. This node passes on sandbox evidence and records the gate as open.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
