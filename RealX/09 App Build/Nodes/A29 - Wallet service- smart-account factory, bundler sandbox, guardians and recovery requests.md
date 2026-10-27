---
title: "A29 - Wallet service: smart-account factory, bundler sandbox, guardians and recovery requests"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am2]
node: A29
milestone: AM2
lane: "Wallets"
oracle: yes
human_gate: none
---

# A29 - Wallet service: smart-account factory, bundler sandbox, guardians and recovery requests

**The service side of the design's "no seed phrase" promise. It covers: wallet rows of four kinds; a passkey smart account whose address exists the moment onboarding asks for it (counterfactual first, deployed later); a sandbox bundler that submits `handleOps` from a funded local key, behind the same adapter a real bundler will use on testnet; the paymaster policy signer, with on-chain limits kept equal to the approved policy; guardian set-up on A26's two guardian slots, behind re-verification and a waiting period; the smart-wallet recovery request, which runs on its own internal machine with the 48 h on-chain timelock, notices, cancel from any device and an off-chain block on disputed proposals; the handoff of an unrecoverable wallet to A28, which runs SM-13's token-level fallback; and delegated session keys that mirror A12 mandates, so an investor's agent can act on-chain without ever holding the owner key. The whole mechanism sits behind one interface, so counsel's DAP answer can change it without touching the register, ledger or token layers.**

**Build status:** not started. **Milestone:** AM2 (Token and wallet core). **Lane:** Wallets. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 8-12 agent-hours (build, challenge and verify loops). This holds because the token-level fallback belongs to A28. If the fallback ever moves back here, the estimate becomes 10-14 h, or the bundler and paymaster split into a node of their own.

**Depends on:** [[A26 - Passkey smart wallet, paymaster and guardian recovery contracts]], [[A27 - Chain adapter, deployment manifests and indexer]], [[A14 - Approvals engine and maker-checker (SM-00)]], [[A12 - API tokens, scopes, agent principals and mandates for every user type (G22)]], [[A15 - Custody sandbox adapter and policy engine P1-P10]], [[A22 - Identity-DVS and sanctions sandbox adapters]], [[A18 - Unit register (legal record) and registry sandbox adapter]], [[A20 - Three-way reconciliation engine with zero tolerance]], [[A06 - Seed data and scenario simulator]] A28 is not a dependency: A28 depends on this node (it reads `getRecoveryRequest` and the wallet identity link), so an edge the other way would be a cycle. This node reaches A28 only over A28's public route `POST /v1/token-ops/transfer-requests` and A28's transfer-request events on the outbox, never by import. Its own tests and hidden oracle run the handoff against a recorded fake of that route and those events, built from A28's frozen contract; the real round trip is proven at A28's gate and in A49 (graph note, 24 Sep 2026).
**Unlocks:** [[A49 - Money-path smoke through the SDK (AM3 exit and founder checkpoint)]], [[A56 - Onboarding flow ONB-01 to ONB-10 with entity branch]], [[A59 - AUD wallet, token wallet and account screens]], [[A72 - Console- token operations, forced transfer and recovery desk, wallets and custody, token registry]]

**Oracle rule:** The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A29\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder. The cases are published into the repo only after the gate passes. The oracle scope is the node summary, corrected by the 24 Sep review. The fallback checks in the graph summary (old balance zero after the fallback, and the CCO + CEO court-order waiver) belong to A28's oracle, because A28 runs SM-13. This oracle covers:
- a smart-wallet recovery never reaches `executing` before its `timelock_until`, never after a dispute, and has no waiver path at all;
- a recovery is handed to A28 only after the request records `unrecoverable` with its reason and evidence, and only for a new wallet that is already `registered`; the link between the two requests is recorded on both sides;
- a session key can never do anything its mandate does not name;
- no session-key operation is admitted after the mandate or token revocation commits, and the on-chain revoke is recorded within `SESSION_KEY_REVOKE_SLA`;
- the owner key never reaches an agent;
- every custody call this module causes carries its own approval id, and no approval id is bound to two custody calls.

This node draws no screen. A56 builds ONB-08. A59 builds POR-05, POR-06 and ACC-02. A72 builds ADM-13 and ADM-15. All of them consume the contracts frozen here. The contracts themselves (smart account, factory, RecoveryModule, SessionKeyModule, RealXPaymaster) are A26. The chain transport, manifests and Holding read model are A27. Custody signing is A15, reached only through A14's execution path. The token-level fallback belongs to A28, on its `transfer_requests` table: SM-13 one to one, the old-wallet freeze, the 48 h queue, the court-order waiver, `recoveryAddress` and the A18 register step. On-chain identity registration is A31 (AM3), which writes this node's `registry_status` through the function named in §1. The sources are requirements catalogue D03 and architecture brief §4.4. This plan is not legal advice. Every sentence about what counsel requires is a counsel question.

## Build brief

**1. Wallet rows, wallet passkeys and the mechanism interface (`services/api/src/modules/wallets/`)**

- **Tables.** A04's `wallets.ts` provides `wallet`, `wallet_key`, `guardian`, `recovery_request`, and the `service_account` rows for the paymaster (V7) and the RealX guardian. This node owns their behaviour. It adds, by migration in its own folder, only:
  - `session_key`: id, wallet id, mandate id from A12, public key, policy hash, policy json, registered tx, revoked tx, revoke approval id, state;
  - `user_operation`: userOp hash, wallet id, signer kind `owner_passkey | session_key`, target, selector, value, paymaster policy version, bundler ref, state, receipt;
  - `paymaster_policy`: versioned data, `approval_id`;
  - `recovery_notice`;
  - `wallet_key.key_role` (`owner | recovery`).
  No table is created outside the module's schema file.
- **Wallet types.** `wallet.wallet_type` takes exactly four values:
  - `passkey_aa`: an investor's ERC-4337 smart account. There is one active account per investor principal. A second one exists only as the `new_wallet_id` of a recovery handed to A28.
  - `external`: a self-custodied address linked by signed-message proof. The phase flag `wallets.external` (A08 flag defaults) is off in P1, so the create route refuses with `EXTERNAL_WALLET_NOT_ENABLED`. The row type exists so that P2 and P3 add no schema.
  - `vault`: read-only mirrors of A15's V1-V7 custody accounts, once A27's manifests give them addresses. ADM-13's wallet registry lists them beside investor wallets. A test checks that they can never hold investor units.
  - `paymaster`: the RealXPaymaster contract and its V7 funding key, with the runway fields below.
- **Wallet passkeys.** Wallet passkeys are `wallet_key` rows with A11's `purpose = wallet`. A wallet credential presented to a login route is refused with `PASSKEY_WRONG_PURPOSE`, and a login credential cannot sign a UserOp.
  - `key_role = owner` rows are owners of the account. This module runs the WebAuthn registration ceremony through A11's server library and stores the P-256 public key as an account owner key. `device_label`, `created_at`, `last_used_at` and `counter` are kept, so POR-06's passkeys-and-devices list and the design's "3 passkeys" chip are plain reads (`passkey_count` counts owner keys only).
  - A further owner device is added under a fresh assertion (A11 §8, purpose `wallet_key_add`, 5 minutes). It becomes an owner through an owner-signed `addOwner` UserOp. Removing the last owner key is refused (`WALLET_LAST_KEY`).
  - `key_role = recovery` is the separate, non-owner credential that fills A26's `recoveryPasskey` slot (§4). It can never sign a UserOp.
- **Mechanism interface.** `mechanism/WalletMechanism.ts` is the interface that the rest of the module and every screen depend on. It speaks only the verbs A26 exposes:
  - `computeAddress(investorId, ownerKey, realxGuardian, recoveryPasskey?)`
  - `createAccount(ownerKey, realxGuardian, recoveryPasskey?)`
  - `buildUserOp(wallet, calls, signerKind)`, `submit(userOp)`
  - `setRecoveryPasskey(wallet, key)` and `clearRecoveryPasskey(wallet)`: both are owner-signed UserOps calling A26's `setRecoveryPasskey`. The clear sets the zero key, which A26 allows at creation; if A26's setter refuses a zero key, that is an A26 escalation.
  - `proposeOwner(account, newOwnerKey)`: guardian calldata for a custody op.
  - `vetoRecovery(account, proposalId, ownerSig)`: relayed by the sandbox relayer through A27's transport. No custody op is needed, because the authority is the owner's signature.
  - `executeRecovery(account, proposalId)`: guardian calldata for a custody op.
  - `getProposal(account)`: returns `proposalId`, `proposer`, `executableAt`, `expiresAt`, vetoed.
  - `registerSessionKey(key, policy)`: an owner UserOp.
  - `revokeSessionKey(account, keyId)` and `revokeAllSessionKeys(account)`: either an owner UserOp or guardian calldata.
  - `isDeployed`, `health`.

  The only implementation built is `passkeyAA`, over A26's contracts and A27's transport. A `custodial` implementation is registered as a stub. It refuses every verb with `MECHANISM_NOT_ENABLED` and is selected by `WALLET_MECHANISM=custodial`. So the open design point the founder can see in [[Decisions]] (RealX as guardian, passkey sync or delegate key-sharing, and whether the guardian model is a DAP under the 2027 framework) is a configuration change, not a rewrite. A module-boundary lint rule fails if any file outside `mechanism/` imports A26's ABIs or A27's transport directly.
- **Counterfactual address first.** `POST /v1/wallets` computes the CREATE2 address with A26's `getAddress(salt, owner, realxGuardian, recoveryPasskey)`. The salt is `keccak256(investor principal id)`, as A26 specifies, and the factory address comes from A27's manifest for the current chain. The row is stored in wallet state `counterfactual` with `deployed_at = null`. So ONB-08 shows the address and its "Created" check at once, and onboarding never waits on a transaction.
  - Deployment happens on the wallet's first UserOp (the op carries `initCode` and is sponsored by the paymaster). It happens eagerly, through the worker job `wallets.deploy`, when `packages/config` says `WALLET_EAGER_DEPLOY=true`. That is the default for testnet runs, so every address carries code before any mint.
  - An invariant test proves the deployed address equals the counterfactual one for 1,000 random investor ids.
  - `POST /v1/wallets` is idempotent per investor: a second call returns the same wallet, never a second row.
- **Wallet state.** This is internal, in A03's engine format, and not one of the 14 product-map machines. Transitions:
  - `counterfactual -> deployed`;
  - `deployed -> recovery_pending -> deployed` (owner change through the smart-wallet recovery of §5);
  - `recovery_pending -> recovered_out`, when A28 reports that the linked `lost_wallet` transfer request is `closed`. The row stays readable with `frozen = true`, and its zero balance is A28's assertion.

  These are flags, not states: `frozen` (mirrored from the chain by A27's indexer and from A28's freeze ops), `registry_status`, `guardian_status` and `kyt_score`. A transition not in the table is refused with `WALLET_ILLEGAL_TRANSITION`.
- **Identity registration record (G01).** `registry_status` uses A31's vocabulary exactly: {`unregistered`, `registering`, `registered`, `stale`, `deregistered`}. Every wallet starts `unregistered`.
  - A31 writes this field only through `setRegistryStatus(walletId, status, { txHash, event })`, exported from this module's `index.ts`. That function enforces `unregistered -> registering -> registered`, `registered -> stale -> registered`, and `registered | stale -> deregistered`. Anything else is refused with `WALLET_REGISTRY_ILLEGAL`. `registered` needs an `IdentityRegistered` event with its tx hash, and this node never sets it by hand.
  - Eligibility is not a status. It is the computed field `can_register`: the KYC case is `approved` and unexpired, sanctions are clear, the wallet is not frozen, and the status is `unregistered`.
  - KYC state is read through an injected `KycStatusProvider` port. It fails closed with no provider, and a fixture provider is used in this node's tests. A30 (AM3) registers the real provider over its public interface. The module never reads the `kyc_case` table.
  - The exported function `isRegisteredDestination(chainId, address)` returns true only for `registered`. It is this node's reader of the off-chain registration record. It passes A15's `IdentityRegistryReader` provider conformance cases, so A30 can register it (or wrap it) as that port's provider unchanged. This node does not claim to be what A15's P4 or A28's destination guard call at run time. The check that the running stack wires them to this record belongs to A49.
  - The labels for ONB-08's second row are A31's: "Linked" for `registered`, "Pending identity review" while the case is in manual or EDD review, and "Not yet linked" for `unregistered`. They are read from A03's `labels.ts` or A50's extension, never written here. A wallet whose identity is not verified is created but cannot receive tokens. This node makes that true off-chain through the function above, and the T-REX identity registry makes it true on-chain.
- **Wallet risk screening at creation (J02 step 9).** The module calls A22's sanctions adapter `screenWallet` through its interface. In P1 and P2 the adapter returns `not_applicable_in_phase`. That value is stored as `kyt_score` and shown as such, never as a pass. The product map's J02 step 9 ("wallet risk screened") and the ADM-13 risk column expect a P1 score. So storing `not_applicable_in_phase` is a **proposed default** that must be recorded in [[Decisions]] before Specify (see the Gauntlet loop). It is not a silent choice.

**2. Bundler adapter (`packages/adapters/bundler/`), built on A08's frame**

- `manifest.ts` calls A08's `defineAdapter` with `name: 'bundler'`, the zod interface contract, `implementations: { sandbox }` only, and `swap: { humanGate: 'HG-VENDOR', candidates: ['Pimlico', 'Alchemy'], whatChanges: 'client only; UserOp construction, paymaster policy and receipts unchanged' }`. A08's generated `SWAP-POINTS.md` and its cross-check test pick the row up unchanged.
- `interface.ts` verbs use the ERC-4337 v0.7 vocabulary, typed with zod. No verb takes or returns private key material. The verbs are `supportedEntryPoints()`, `estimateUserOperationGas(userOp)`, `sendUserOperation(userOp, entryPoint)` (returns the v0.7 `userOpHash`), `getUserOperationByHash(hash)`, `getUserOperationReceipt(hash)` and `health()`.
- `sandbox/` is selected by `ADAPTER_BUNDLER=sandbox` through A08's registry. A08's config refuses any other selector while `RUN_MODE=local|test`. The sandbox:
  - validates the op shape and runs the EntryPoint's simulation against A27's Hardhat transport;
  - submits `handleOps([op], beneficiary)` from a funded local EOA whose key lives in the kms sandbox (A08; `kms.sign` only, no key bytes in the package);
  - computes the `userOpHash` exactly as EntryPoint v0.7 does (a test compares it with the `UserOperationEvent` the chain emits) and builds receipts from the events;
  - receives A08's injected `Clock` and `SeededRandom`;
  - honours A08's magic inputs only inside `sandbox/`. A UserOp whose `callData` memo prefix is `SIM-BUNDLER-REJECT` is refused at send. `SIM-BUNDLER-DELAY-<seconds>` delays inclusion. `SIM-BUNDLER-DROP` is accepted and never included, which is how the stuck-op detector is tested;
  - registers a `SandboxControl` with `trigger('include', {hash})`, `trigger('drop', {hash})`, `advance(duration)`, `reset(seed)` and `state()`. `state()` never serialises key refs.

  The same relayer key submits A26's `veto(account, proposalId, ownerSig)` as a plain transaction when §5 needs it. A09 mounts the sandbox controls under `/sandbox/bundler/*` in `local|test` only, and the ADM-32 sandbox panel (A69) exposes them by hand.
- `conformance/*.conf.ts` is written against the interface, not the sandbox, and runs through A08's `runConformance` harness. It has one case per verb, a deploy-on-first-op case (`initCode` present), a paymaster-sponsored case, a rejected case, a delayed case, a dropped case, and determinism under `reset(seed)`. It is the artefact a human runs against Pimlico's or Alchemy's Base Sepolia endpoint after HG-VENDOR, before that implementation may be registered. Nothing in this node talks to either.
- **Stuck operations.** A `user_operation` in state `sent` without a receipt after 30 minutes raises one Sev3 alert per op to ENGO through A13. This is the same threshold A27 uses for plain transactions. The wallet's screen state shows "pending" with the hash, never a silent failure.

**3. Paymaster policy signer, runway and the gas-sponsorship policy change**

- **Sponsorship.** `paymaster/sponsor(userOp, wallet)` decides whether RealX pays the gas. When it does, it asks A15 to sign the VerifyingPaymaster data with the V7 key, under A15's standing `AUTO` approval for the effective policy version. This module holds no key.
  - The policy is data in `paymaster_policy` (versioned, immutable once effective). It holds: allowed targets (contract kinds from A27's address book: T-REX tokens, the ClaimIssuer and identity contracts, the RecoveryModule, the SessionKeyModule, and the resale settlement contract in P2); allowed selectors per target; the per-wallet daily gas cap in wei; the global daily cap; a `frozen_wallets_sponsored = false` rule; and `initCode` allowed once per wallet.
  - A refusal carries a stable code (`PAYMASTER_TARGET_NOT_ALLOWED`, `PAYMASTER_SELECTOR_NOT_ALLOWED`, `PAYMASTER_WALLET_CAP`, `PAYMASTER_GLOBAL_CAP`, `PAYMASTER_WALLET_FROZEN`, `PAYMASTER_POLICY_DRIFT`) and is recorded on the `user_operation` row.
  - Investors never buy gas. The API has no route that accepts a self-paid UserOp from an investor wallet, and a UserOp without a paymaster field is refused with `USEROP_UNSPONSORED`.
- **Changing the gas-sponsorship policy.** This is the product map's screen-level row "Change gas-sponsorship policy (ENGO to CFO)".
  - `POST /v1/wallets/paymaster/policy` creates an `ApprovalRequest` through A14 with `action_code = wallet.gas_policy.change`, under policy `4E` (maker ENGO, checker CFO).
  - The effector registered with A14's `registerEffector` does three things on `confirmed`: it inserts the new `paymaster_policy` version; it updates A15's V7 record (`daily_cap`) through A15's ENGO → CFO policy path; and it submits the on-chain paymaster owner call (`setCaps`, `addTarget` or `removeTarget`) as a custody op bound to the same approval id, against the account that A27's manifest records as the paymaster `owner`.
  - A15 binds one approval to one call. So a change that needs several on-chain calls is split into one linked request per call, and the new DB version becomes effective only when every linked request's postcondition has passed.
  - The postcondition compares the approved payload with three things: the effective version's hash, A15's V7 record, and the on-chain `allowedTargets`, `perWalletDailyCapWei` and `dailyCapWei` read through A27. All three must match.
  - A direct write to `paymaster_policy` outside that path is blocked by a trigger.
  - The worker job `paymaster.drift` runs every 15 minutes and repeats that comparison. On any drift, it raises a Sev2 alert to ENGO and SECO through A13, and sponsorship fails closed with `PAYMASTER_POLICY_DRIFT` until the values match again.
  - In deployment, A26 names the V1 owner multisig as the paymaster owner, while A15's ENGO → CFO path forbids a V1 authorisation. That conflict is flagged to A15 and A26 (see Escalate). Locally the owner is A07's deployer account, reached through the sandbox custody path.
- **Runway.** The worker job `paymaster.runway` runs every 15 minutes (on A09's scheduler until A46 exists). It reads the paymaster's EntryPoint deposit and `runway()` through A27, and computes the trailing seven-day average daily sponsored gas from `user_operation` receipts. It writes `deposit_wei`, `spent_today_wei` and `runway_days` to the paymaster wallet row. It also registers this module as A15's `PaymasterBalance` provider, after re-running A15's provider conformance cases against it. A15 owns the V7 funding key, the daily cap, top-ups and the one-Sev2-alert-per-breach rule. This node's tests assert, through A15, that the alert fires exactly once when runway falls below 3 days, and again only after it recovers above 3 days and falls again. `GET /v1/wallets/paymaster` returns exactly the fields listed in ADM-13's "paymaster runway" panel.

**4. UserOps and guardian set-up on A26's two slots**

- **UserOps.** `POST /v1/wallets/me/userops` accepts a UserOp signed by one of the wallet's owner passkeys, or by a registered session key (§6). An owner signature is a WebAuthn assertion over the `userOpHash`, verified server-side with the stored P-256 key before anything is spent.
  - The module checks that the wallet is not `frozen` and not `recovery_pending` (`WALLET_FROZEN`, `WALLET_RECOVERY_PENDING`), runs the paymaster policy, and submits through the bundler adapter. `GET /v1/wallets/me/userops/{hash}` returns the state and receipt.
  - Every UserOp is one audit event with the signer kind. A session-key op is audited as "agent <token prefix> on behalf of <person>" through A12's middleware.
  - At launch an investor's own UserOps are rare, because subscriptions and distributions are AUD movements and V2 mints. So these routes exist now for claim acceptance and for guardian and session-key management, and later for the P2 resale board.
- **The guardian model follows A26 exactly.**
  - `realx` is A26's `realxGuardian` slot. It is set at creation and `active` from day one. It is an address: A25's V2 AgentGateway in deployment, and A26's `TestRealXGuardian` 2-of-3 stub locally. Every RealX guardian act (`proposeOwner`, `executeRecovery`, a guardian session-key revoke) is a custody transaction signed through A14 and A15, with its own approval id. It is never a UserOp. The guardian alone can never drain the account (A26).
  - `second_device` is A26's `recoveryPasskey` slot. It is a separate, non-owner P-256 credential (`wallet_key.key_role = recovery`) created on a second device. It can propose an owner but can never sign a UserOp. It is set with `setRecoveryPasskey` and cleared with `clearRecoveryPasskey`. Both are owner-signed UserOps, and A26 allows them only while no proposal is open.
  - `trusted_contact` is **off-chain only**, because A26 has no slot for it. It is an identity-verified RealX investor whom the person names. The contact may countersign a recovery request (recorded as evidence for KYCA) and receives the recovery notices. The contact cannot propose, veto or execute anything on-chain. ONB-08 in the product map lists "an optional trusted contact", so the off-chain form is a **proposed default** to record in [[Decisions]]. An on-chain trusted-contact guardian would mean reopening A26.
- **Adding or removing an investor-side guardian.** The catalogue requires "recovery set-up needs identity re-verification and a waiting period".
  - The route carries `requiresHuman: true` and a fresh assertion.
  - The module opens a liveness re-verification through A22's identity adapter, with `sessionAssurance = passkey_present`. The sandbox name-suffix scenarios approve, refer or reject.
  - On a pass, the guardian row enters `waiting_period` with `effective_at = now + GUARDIAN_WAITING_PERIOD`. The value is data in `packages/config`. Its **proposed default is 24 h**: the sources say only "a waiting period", so the value is recorded in [[Decisions]] as founder-visible. Notices go to every registered channel with a one-click cancel.
  - **Nothing is pre-signed.** At `effective_at`, the worker job `wallets.guardians.due` moves the row to `awaiting_confirmation` and sends the person a "confirm guardian" notice. `POST /v1/wallets/me/guardians/{id}/confirm` builds the `setRecoveryPasskey` or `clearRecoveryPasskey` UserOp at that moment, with the current nonce and fresh paymaster validity, and the person signs it with an owner passkey.
  - A `trusted_contact` change is off-chain, so it activates at `effective_at` without a signature. An unconfirmed on-chain change lapses to `expired` after `GUARDIAN_CONFIRM_WINDOW` (data, default 7 days).
  - A refer or reject leaves the row `pending_reverification`, and the screen shows why.
  - Any guardian change while a `recovery_request` is open is refused with `GUARDIAN_LOCKED_DURING_RECOVERY`, and A26 reverts one anyway.
  - `guardian_status` on the wallet (`realx_only`, `pending`, `active`) is what POR-06's guardians list and ADM-13's registry show.

**5. Smart-wallet recovery on its own internal machine, and the handoff to A28's SM-13 fallback**

- **Why a separate machine.** The product map's SM-13 note says "Smart-wallet recovery comes first; this is the token-level fallback". A28 runs SM-13 one to one on `transfer_requests`, and a `RecoveryRequest` that this node marks `unrecoverable` is what opens A28's `lost_wallet` transfer request. So the smart-wallet path runs on an internal machine, `WR` (wallet recovery). It is written in A03's engine format inside this module, like the wallet state above, and is not one of the 14 product-map machines.
  - This node adds no row to A03's SM-13 table or to `extensions.ts`, and it never edits A03.
  - `old_wallet_frozen` keeps its SM-13 meaning in A28.
  - This is a **plan-level decision** to record in [[Decisions]] before Specify. The alternative is reopening A03 to route the smart-wallet path through SM-13 with a path-conditional guard.
- **`recovery_request` rows.** Each row carries:
  - `recovery_type`: only `passkey_lost`. Estate, court-order, divorce, error and sanctions-directive cases are A28 transfer requests opened by LEGL, and never pass through this machine.
  - `recovery_path`: `smart_wallet` or `handed_off`.
  - `wallet_id`, `new_owner_key_id`, `proposal_id`, `executable_at` and `expires_at` (read from A26), and `timelock_until`.
  - `channel_snapshot`: every email and phone on record when the request was raised, plus every channel changed or removed in the 30 days before it.
  - `approval_id` (the `wallet.recovery.approve` request) and `execute_approval_id` (the `wallet.recovery.execute` request).
  - `unrecoverable_reason` and `unrecoverable_evidence_ids[]`.
  - `new_wallet_id`, `transfer_request_id` and a mirrored `transfer_request_state` (handoff only).
  - `state` (WR).
- **WR transitions** (anything else is refused with `WR_ILLEGAL_TRANSITION`):

| From → to | Trigger | Who |
|---|---|---|
| `[*] → requested` | `POST /v1/wallets/recovery` | the investor, signed in (after A11's login recovery if the login passkey was lost too), `requiresHuman`; or SUPP intake from ADM-15 |
| `requested → triage` | `POST /recovery/{id}/triage` | SUPP |
| `triage → identity_reverification` | triage complete, and the account has a working RecoveryModule (`getProposal` readable and the RealX guardian set) | SUPP |
| `triage → unrecoverable` | no working RecoveryModule (reason and evidence recorded) | SUPP, then KYCA confirms |
| `identity_reverification → awaiting_new_key` / `→ rejected` | liveness and document through A22 pass, or fail (KYCA decides a refer) | the person present; KYCA |
| `awaiting_new_key → compliance_approval` | the person registers the new wallet passkey (`POST .../new-key`, stored as `new_owner_key_id`, not yet an owner), and KYCA submits the A14 request `wallet.recovery.approve` (`4E`, KYCA → CCO; the product map row "Approve smart-wallet recovery") | the person; KYCA as maker |
| `compliance_approval → proposing` / `→ rejected` | CCO approves or rejects in A14; on approval, A14's execution creates one custody op, `proposeOwner(account, newKey)` by the RealX guardian, bound to that approval id | CCO |
| `proposing → waiting_period` | A27's indexer sees `RecoveryProposed`; `proposal_id`, `executable_at` and `expires_at` are recorded | system |
| `waiting_period → executing` | at `timelock_until`, an `SVCA` creates the A14 request `wallet.recovery.execute` (`AUTO`, referencing the parent approval id and the proposal id), whose single custody op is `executeRecovery(account, proposalId)` | system |
| `waiting_period → disputed` | cancel from any signed-in session, the notice's cancel link, an on-chain `RecoveryVetoed`, or an objection SUPP records | the person; SUPP |
| `executing → confirmed` | A27's indexer sees `RecoveryExecuted` | system |
| `executing → unrecoverable` | execution reverted twice, or the proposal reached `expiresAt` | system |
| `confirmed → closed` | the register's wallet reference still equals the account address (A18 read; the address never changes on this path), and A20's targeted recon for the wallet is clean; otherwise `RECOVERY_NOT_RECONCILED` and the request stays open | system |
| `disputed → rejected` / `→ compliance_approval` / `→ unrecoverable` | LEGL records the dispute review, then CCO decides; a return to `compliance_approval` needs the holder's passkey confirmation and a fresh re-verification record | LEGL, CCO |
| `unrecoverable → handed_off` | see the handoff bullet | KYCA or SUPP |

- **Executing only after the timelock and the notices.** `timelock_until = max(executable_at, the time the first notice for each channel kind in the snapshot was sent + 48 h)`. It is recomputed from the audit-anchored `RecoveryProposed` and `wallet.recovery.notice_sent` events on every read and every transition, so editing the column cannot shorten the wait.
  - This module registers an A14 precheck hook for `wallet.recovery.execute` that fails when: the request is not in `waiting_period`; `now < timelock_until` (`RECOVERY_TIMELOCK_ACTIVE`); a notice per channel kind is missing (`RECOVERY_NOTICE_NOT_SENT`); or the request is `disputed` (`RECOVERY_DISPUTED`).
  - There is **no waiver of any kind** on this path. A26's RecoveryModule enforces its own 48 h, and a court order is an A28 `court_order` transfer request. The API returns `RECOVERY_NO_WAIVER` with that sentence to any waiver attempt.
- **Cancel and dispute: an off-chain block.**
  - `POST /v1/wallets/recovery/{id}/cancel` can come from any signed-in session of the same principal, or through the single-use cancel link in the notice (valid 72 h). It moves `waiting_period -> disputed`.
  - In the same transaction it cancels, through A14's cancel, any `wallet.recovery.execute` request not yet signed. A15's `ApprovalReader` then reports it cancelled, so any op under it is refused with `P1_CANCELLED`, and the precheck hook refuses any new one.
  - Only the RealX guardian can execute a proposal on A26, so a disputed proposal can never mature into an owner change. It lapses on-chain at `expires_at`.
  - **The veto is used only when an owner passkey exists.** If the person still has a current owner passkey, `POST /recovery/{id}/veto` (`requiresHuman`) asks for a WebAuthn assertion over A26's veto digest and relays `veto(account, proposalId, ownerSig)`. The on-chain proposal ends at once, and a new proposal is possible immediately.
  - Otherwise, a new proposal is blocked on-chain until `expires_at` (A26 `ProposalAlreadyOpen`). `GET /recovery/{id}` shows `new_proposal_blocked_until = expires_at`, and a return to `compliance_approval` is refused with `RECOVERY_PROPOSAL_STILL_OPEN`, naming that date. If the founder wants a faster re-proposal, the answer is to reopen A26 for a guardian `cancelProposal`, not to work around it here.
  - A cancel raises a Sev2 alert to MLRO and SECO, because a cancel usually means someone else started the recovery. The wallet stays `recovery_pending` until the dispute is resolved.
- **Protective acts get their own approval ids.** Starting a request, and any dispute, revoke the wallet's session keys (§6) through the guardian's `revokeAllSessionKeys`. That is a custody op under its own `wallet.session_key.revoke` request (`AUTO`, restrict-only, `SVCA` actor, `trigger_ref` = the recovery request id), never under the recovery's approval id. `executeRecovery` also revokes every session key on-chain (A26), and this module records that from the event.
- **Handoff to A28 (token-level fallback).** When a request reaches `unrecoverable`, the reason and the failed attempt's evidence (reverted tx hashes, the proposal's expiry, the dispute outcome) are recorded. That makes "smart-wallet recovery is attempted first" provable from the row.
  - A new `passkey_aa` wallet is created for the person, counterfactual and then deployed. It must reach `registry_status = registered` first. Until then, `POST /recovery/{id}/handoff` is refused with `RECOVERY_NEW_WALLET_UNREGISTERED`. A31 does the registration, and this node's fixture sets it through `setRegistryStatus` with a fixture event.
  - The handoff then calls A28's public route `POST /v1/token-ops/transfer-requests` (over HTTP through the SDK, never an import; a recorded fake of the route at this node's gate) with `request_type = lost_wallet`, `recovery_request_id`, `old_wallet`, `new_wallet`, the re-verification record id and the evidence ids. It stores the returned `transfer_request_id` and moves to `handed_off`. The link is recorded on both rows.
  - From then on, A28 owns the old-wallet freeze (the J09 step-6 TOKO freeze, under A28's own approval), the 48 h queue, the SM-13 approvals, `recoveryAddress`, the A18 register step and `closed`. A lost-wallet recovery cannot be waived at all (A28's `waivable_types`).
  - This module consumes A28's transfer-request events and mirrors their state into `transfer_request_state`, so ACC-02 shows `fallback: true` with the right step. On A28's `closed`, the old wallet moves to `recovered_out`.
  - This module exports `getRecoveryRequest(id)` from `index.ts` for A28's check that the request really is `unrecoverable`. A28 depends on this node, so it imports it directly.
- **Notices.** Notices are rendered by templates inside this module, behind a `NoticeSender` interface.
  - The implementation built here is a sandbox sender. It writes each notice to a `recovery_notice` row (channel kind, address hash, sent-at, provider ref, template version) and to a local inbox, readable at `GET /sandbox/wallets/notices?to=` (mounted by A09 in `local|test` only).
  - It does not use A08's email or sms adapters, and it is not A43. A43 (AM3) later registers its comms implementation behind the same interface, using its `recovery.started` and `recovery.vetoable` notice types. That swap must be written into A43's own brief and challenges.
  - The request cannot leave `waiting_period` for `executing` until at least one notice per channel kind in the snapshot is recorded as sent.
- **Disclosure.** The disclosure that RealX can reconstitute control (the pack's "quasi-custodial" note) is a counsel question under HG-LEGAL. The API exposes `recovery_disclosure_version` on `GET /v1/wallets/me`, so the screens show the approved wording and never a hard-coded sentence.

**6. Session keys for agent mandates (founder requirement D04, gate G22)**

- **Registration.** When A12 creates or widens a mandate whose scope includes a wallet cell (P2 asks and bids on the resale board, claim acceptance, and ballots in P2), the mandate carries a `session_key_policy` derived from its caps. The policy holds allowed contracts and selectors (from A27's address book), per-transaction and daily unit ceilings, allowed vehicles, and an expiry no later than the mandate's.
  - `POST /v1/wallets/me/session-keys` takes the mandate id and the agent's session public key. The key is generated by the agent's own runtime. RealX never holds the session private key, and the owner key never leaves the wallet passkey.
  - The route checks, through A12's `introspect`, that the mandate is `active`. It checks that the policy hash inside the owner-passkey-signed `registerSessionKey` UserOp equals the mandate's `session_key_policy` hash, and that the key's expiry is no later than the mandate's.
  - It then sponsors and submits the UserOp, and stores the `session_key` row as `registering`, then `active` on receipt.
  - Authority comes from the owner passkey signature on that UserOp. This node does not rely on the A11 fresh assertion that A12 consumed for the mandate.
  - The route carries `requiresHuman: true`, so an agent cannot register its own key.
- **Checked twice.**
  - **Off-chain.** `session-keys/check.ts` first calls A12's `introspect` synchronously on every session-key op, for the token and mandate state (revoked, expired, suspended or kill-switched). A key whose mandate or token is no longer live fails closed with `SESSION_KEY_MANDATE_INACTIVE`. It then checks the op against the policy: target, selector, units, the daily total under the `mandate_usage` row lock A12 defined, expiry, the wallet not frozen and not `recovery_pending`. A violation is refused with `SESSION_KEY_POLICY_VIOLATION`, naming the field. All of this runs after A12's G22 middleware has already admitted the call.
  - **On-chain.** A26's SessionKeyModule checks again. That is the guarantee when someone bypasses the API on a public testnet.
  - The key can never call the account's owner, guardian or key-management functions, on either layer.
- **Revocation.** A12 only emits outbox events. It does not call this module, and this module does not ask it to.
  - Because `check.ts` reads A12's state synchronously, the next op after a mandate or token revocation commits is refused at once.
  - An outbox consumer in this module handles `agent.mandate.revoked`, `agent.token.revoked` and `auth.recovery.completed`, as well as this module's own recovery start, a freeze seen by A27's indexer or A28, and the nightly expiry sweep. It marks the `session_key` row `revoked`, then submits the on-chain revoke as the RealX guardian. That revoke is a custody transaction through A14 and A15 under its own `wallet.session_key.revoke` request (`AUTO`, restrict-only, `SVCA` actor, `trigger_ref` = the event id, one approval id per revoke).
  - The tx hash and approval id are recorded on confirmation, within `SESSION_KEY_REVOKE_SLA` (data, default 60 s on the local stack). A breach raises a Sev2 alert to SECO.
  - When the person revokes a key themselves from POR-06 or ACC-05, it is an owner-signed `revoke` UserOp through the bundler.
  - The staff kill switches (A101, A77) reach the guardian revoke through their own `SGL+R` rows, not this node's.
  - Session keys are listed on POR-06 and ACC-05 with the mandate name, limits, expiry and last use.

**7. API module surface, agent rules, audit, errors and empty states**

- **Investor routes** under `/v1/wallets`:
  - `POST /` (create).
  - `GET /me` returns: address; `address_short` by the 6+4 rule; chain; `deployed`; `registry_status` and `can_register`; `frozen` with reason; `guardian_status`; `passkey_count`; keys and devices; guardians; session keys; `tokens_held_total`; `trust_count`; tokens by trust with `frozen_amount` and `frozen_reason` from A27's Holding read model; `recovery_disclosure_version`; the open recovery, if any.
  - `POST /me/keys`, `DELETE /me/keys/{id}`, `POST /me/guardians`, `DELETE /me/guardians/{id}`, `POST /me/guardians/{id}/confirm`, `POST /me/userops`, `GET /me/userops/{hash}`, `POST /me/session-keys`, `DELETE /me/session-keys/{id}`, `POST /recovery`.
  - `GET /recovery/{id}` returns `step` in {`verify`, `guardian`, `wait`, `new_passkey`}, mapped from WR: `verify` ← `requested`, `triage`, `identity_reverification`, `awaiting_new_key`; `guardian` ← `compliance_approval`, `proposing`; `wait` ← `waiting_period`, `disputed`; `new_passkey` ← `executing`, `confirmed`, `closed`. It also returns `fallback: true | false` (true once `handed_off`, with `transfer_request_state`), `timelock_until` and `new_proposal_blocked_until`, so ACC-02's stepper and banner need no client rules.
  - `POST /recovery/{id}/reverify`, `POST /recovery/{id}/new-key`, `POST /recovery/{id}/cancel`, `POST /recovery/{id}/veto`.
- **Staff routes:**
  - `GET /`: the wallet registry with `registry_status`, `kyt_score`, `frozen`, `guardian_status` and a tenant filter (ADM-13).
  - `GET /{id}`.
  - `GET /recovery?state=`: the ADM-15 desk rows for WR requests. Each row has id, path, units at stake, evidence, `timelock_until`, state, the approvals chain, notices sent, and `transfer_request_id` when handed off. A72 lists A28's transfer requests beside them.
  - `POST /recovery/{id}/triage`, `POST /recovery/{id}/reverification-result`, `POST /recovery/{id}/submit-approval`, `POST /recovery/{id}/objection`, `POST /recovery/{id}/dispute-review`, `POST /recovery/{id}/dispute-decision`, `POST /recovery/{id}/unrecoverable`, `POST /recovery/{id}/handoff`.
  - `GET /paymaster`, `POST /paymaster/policy`.

  RBAC comes from A03's matrix: SUPP intake and objection only; KYCA re-verification and the recovery maker; LEGL dispute review; CCO, CFO and CEO decide in A14; ENGO is the paymaster maker; TP-OBS and TP-AUD read-only. There is one internal route: `GET /v1/internal/wallets/{chainId}/{address}/registration`, which is `isRegisteredDestination` over HTTP for the provider A30 registers.
- **Investor and partner agents (D04, G22).** Every read route can be called with an investor, sponsor, partner or staff token within scope. These routes carry `requiresHuman: true`, and a token gets `AGNT_NOT_ALLOWED` plus a G22 gate evaluation: `POST /`, `POST /me/keys`, `DELETE /me/keys/{id}`, `POST /me/guardians`, `DELETE /me/guardians/{id}`, `POST /me/guardians/{id}/confirm`, `POST /recovery`, `POST /recovery/{id}/reverify` (liveness; non-delegable in A03's `delegable.ts`), `POST /recovery/{id}/new-key`, `POST /recovery/{id}/veto` and `POST /me/session-keys`. These make up the founder's non-delegable list: first passkey and wallet, identity and liveness, guardian changes, recovery, and widening authority. `cancel` is allowed to a token acting for the owner, because it only ever narrows: it is an off-chain block and never a veto. A session-key UserOp is the one on-chain act an agent can perform. An investor token on any staff route is refused by scope with `403 G22_SCOPE_MISSING`, naming the cell.
- **Staff agents (D04, the AI-agent principal rule).** A staff agent may act as maker only, within its human's role. It is never a checker, approver or custody signer. Segregation of duties counts the agent as its human: the agent's human joins the request's maker identity set, so that human cannot then decide it (`checkSod`).

| Staff route | Human role | Staff agent |
|---|---|---|
| `triage`, `objection` | SUPP | yes, as maker (draft or submit the record) |
| `reverification-result` | KYCA | no, `AGNT_NOT_ALLOWED`: a decision on a person's identity |
| `submit-approval` (`wallet.recovery.approve` draft and submit) | KYCA | yes, as maker |
| A14 decide on `wallet.recovery.approve` | CCO | no, `APR_CHECKER_MUST_BE_HUMAN` and `AGNT_NOT_ALLOWED` |
| `dispute-review` | LEGL | no, `AGNT_NOT_ALLOWED`: legal-review decisions are never agent acts |
| `dispute-decision` | CCO | no, `AGNT_NOT_ALLOWED` |
| `unrecoverable` | SUPP or KYCA | draft only; the transition fires when the human KYCA confirms |
| `handoff` | KYCA or SUPP | yes, as maker (A28's table allows an agent as maker on `POST /transfer-requests`) |
| `POST /paymaster/policy` | ENGO | yes, as maker; the CFO decision in A14 is never an agent's |
| every `AUTO` act (`wallet.recovery.execute`, `wallet.session_key.revoke`) | SVCA | not an agent act; no token can trigger it |

- **Audit.** There is one hash-chained event per transition, decision, notice, UserOp and refusal: `wallet.created`, `wallet.deployed`, `wallet.key.added`, `wallet.key.removed`, `wallet.guardian.pending`, `wallet.guardian.awaiting_confirmation`, `wallet.guardian.active`, `wallet.guardian.removed`, `wallet.guardian.expired`, `wallet.userop.sent`, `wallet.userop.included`, `wallet.userop.refused`, `wallet.session_key.registered`, `wallet.session_key.revoked`, `wallet.recovery.<state>` per WR state, `wallet.recovery.notice_sent`, `wallet.recovery.cancelled`, `wallet.recovery.vetoed`, `wallet.recovery.handed_off`, `wallet.registry_status.changed`, `wallet.paymaster.policy_changed`, `wallet.paymaster.drift` and `wallet.paymaster.runway`. Each event carries `actor_id`, `on_behalf_of_id`, `approval_id` where one exists and `tx_hash` where one exists.
- **Empty and error states:**
  - An investor with no wallet gets `{ wallet: null, can_create, reason }` (reason `identity_pending` or `ready`), so ONB-08 renders the DropBox.
  - No tokens gives `tokens: []` and `tokens_held_total = 0`. No recovery gives `[]`.
  - A refusal is A09's envelope with a stable code and a plain-English message, never a bare 4xx.
  - A `GET` on a closed, rejected or handed-off recovery returns 200 with its terminal state, never 404.
  - If the bundler is down, the route returns `503 BUNDLER_UNAVAILABLE` and the op stays `prepared` for the person to retry. It is never retried silently.

## Owned paths (other nodes do not edit these)

- <workspace>\app\packages\adapters\bundler\
- <workspace>\app\services\api\src\modules\wallets\

Inside the module, by convention:
- `mechanism/` (`WalletMechanism.ts`, `passkey-aa/`, `custodial-stub/`);
- `keys/`, `guardians/`;
- `recovery/` (the WR machine, notices and `NoticeSender`, cancel and veto, the A28 handoff client and event consumer);
- `session-keys/`, `paymaster/`, `userops/`, `registry-status/`;
- `schema.ts`, `events.ts`;
- `index.ts`: the only import surface for A28, A30, A31 and the screen nodes. It exports `setRegistryStatus`, `isRegisteredDestination`, `getRecoveryRequest`, and the read contracts.

This node only uses:
- A26's ABIs and A27's transport (inside `mechanism/`);
- A14's approvals API, `registerEffector`, `registerPrecheck` and cancel;
- A15's custody ops, V7 record, `PaymasterBalance` port and `IdentityRegistryReader` conformance cases;
- A12's `introspect`, outbox events and `mandate_usage` lock;
- A11's WebAuthn library and fresh-assertion primitive;
- A22's identity and sanctions interfaces;
- A28's `POST /v1/token-ops/transfer-requests` and transfer-request events;
- A18's register read and A20's targeted recon result;
- A06's seed personas;
- A08's adapter frame and kms;
- A13's alert API and A10's audit interface.

It changes nothing outside the two paths above. The new action-code rows it needs (`wallet.recovery.execute` and `wallet.session_key.revoke` as `AUTO`) are seeded by A03 and A14, not written here.

## Design references

- `design:onb ONB-08`: `<workspace>\design\RealX App.dc.html`, screen `onb`, step ONB-08 "Wallet".
  - Title "Set up your token wallet", sub-line "Your tokens live in a wallet secured by a passkey on your device. No seed phrase to lose.".
  - The **DropBox** has a dashed `#B9B4AA` border, the Newsreader 22 title "Create passkey wallet" and the caption "Uses Face ID, Touch ID or Windows Hello".
  - Below it, the **CheckList** has two **CheckRow**s: "Wallet address" with sub-line "0x7a3f…c91e" and a green StatusPill "Created"; and "Linked to your verified identity" with sub-line "Required to receive tokens" and a green StatusPill "Linked".
  - `POST /v1/wallets` and `GET /v1/wallets/me` return `address_short` (first 6 plus last 4, the single truncation rule from design-analysis §7 bug 3), `created_at` and `registry_status`, so A56 renders both rows without client-side rules. The amber and grey forms of the second row take A31's labels, not strings written here.
  - The counterfactual design point is what makes the "Created" pill honest before any transaction exists.
- `design:wallet Token wallet card`: screen `wallet` (POR-05 with the POR-06 card), the third **WalletCard**.
  - It has the title Newsreader 22 "Token wallet"; the 13px text "Secured by your passkey. RealX can't move your tokens without trustee approval."; a mono value chip on cream `#F5F2EC` reading "0x7a3f…c91e · 3 passkeys"; and the KPI "Tokens held" "280" with sub-line "Across 3 trusts".
  - `GET /v1/wallets/me` carries `address_short`, `passkey_count` (owner keys), `tokens_held_total` and `trust_count` with exactly those meanings.
  - The sentence about trustee approval is the plain-English form of two things: SM-13's checker step (CFO and trustee) in A28's fallback, and A26's "guardian alone cannot drain".
  - A06's seed investor (Alex Morgan: 3 passkeys, 2 guardians, 120 + 60 + 100 units) must produce those exact numbers.
- **Undesigned surfaces this node feeds**, drawn in the design-extension nodes and built later:
  - POR-06 Token wallet and recovery, and ACC-02 Recover account (`design-ext:POR-06`, `design-ext:ACC-02` from [[A50 - Design extension A- investor and sponsor screens not in the prototype]]). These show an expanded WalletCard with tokens by trust and grey frozen pills carrying a reason; a passkeys-and-devices CheckList; a guardians list (RealX recovery; second device; trusted contact); a "Start recovery" outline button; the HorizontalStepper "Verify you / Guardian / Wait 48 h / New passkey"; a DropBox-style centred status; and the fallback banner.
  - ADM-13 Wallets, custody and recovery, and ADM-15 Forced transfer and recovery desk (`design-ext:ADM-13`, `design-ext:ADM-15` from [[A51 - Design extension B- operator console screens not in the prototype]]). These show the wallet DataTable with identity status, risk and frozen columns; the paymaster runway panel; the recovery card with a waiting-period Timeline; and the desk DataTable with the approvals chain and the "notice sent" banner.
  - This node supplies only their data contracts. "Done" for design here means those payloads contain exactly the fields the screens list, and validate against the SDK schema.
- **Status labels and tones.** This node returns state ids only. Labels and tones come from A03's `labels.ts` where the prototype shows the state, and otherwise from A50's and A51's design-extension entries. A state id with no source label is a gap to report to A50 or A51, not a string to invent. The contract test fails on any state id that has no label in those sources.

## Requirements covered

- **Product-map screens:** none owned. Done means the data contracts for ONB-08 (A56), POR-05's token wallet card, POR-06 and ACC-02 (A59), and ADM-13 and ADM-15 (A72) are published in the SDK, with contract tests that the screen nodes reuse unchanged. The seed investor must render the prototype's "0x7a3f…c91e · 3 passkeys" and "280" "Across 3 trusts".
- **Journeys:** none owned. Done means:
  - J02 step 9 (create wallet, risk screened, address shown at once) runs at service level;
  - J09 steps 1-5 and 10 run end to end on the compose stack on the smart-wallet path;
  - J09's fallback runs from `unrecoverable` through the handoff to an A28 `lost_wallet` transfer request, for a new wallet that was registered first, with the link recorded on both sides (J09 steps 6-9, and the zero old balance, are A28's to prove);
  - J04 step 12 and J07 hold at this node's boundary, because `isRegisteredDestination` is false for every wallet that is not `registered`. The running-stack wiring check is A49's.
- **Gate G01 Identity verified:** done means:
  - a wallet can be created before identity is verified;
  - `registry_status` follows A31's vocabulary and reaches `registered` only through `setRegistryStatus` with an `IdentityRegistered` event;
  - `isRegisteredDestination` passes A15's `IdentityRegistryReader` conformance cases;
  - no handoff to A28 happens for an unregistered new wallet.
- **Gate G13 Maker-checker:** done means:
  - every smart-wallet recovery approval, execution, guardian act by RealX, protective session-key revoke and gas-policy change is an `ApprovalRequest` through A14, with the policy codes named above;
  - every custody call carries its own approval id, and no approval id is bound to two custody calls;
  - no route in this module can move an owner, a guardian, a token or a policy without one.
- **Gate G22 Agent authority:** done means:
  - an agent can read wallet state and submit session-key UserOps strictly inside its mandate's policy;
  - an agent is refused on every non-delegable route with `AGNT_NOT_ALLOWED` and a G22 gate evaluation;
  - a staff agent is maker only, per the table in §7, with segregation of duties counting it as its human;
  - an agent never receives or registers the owner key;
  - once a revocation commits, the agent's next session-key op fails at once, and the on-chain guardian revoke is recorded within `SESSION_KEY_REVOKE_SLA`.
- **Module M08 Wallets and custody:** done means these exist as the product map's M08 objects: the wallet registry (four types, identity status, risk, frozen, guardians); the passkey smart-account factory with counterfactual addresses; the bundler and the paymaster, with on-chain limits equal to the approved policy and runway feeding V7; guardian set-up on A26's two slots with re-verification and a waiting period; and the recovery desk data. N14 (non-custodial wallets: RealX never holds investor keys; paymaster caps and a runway alert below 3 days; recovery requires re-verification and a waiting period) is met by tests.
- **State machine SM-13 Wallet recovery or forced transfer:** done means:
  - this node proves SM-13's first invariant, "smart-wallet recovery is attempted first", with the WR machine and its recorded `unrecoverable` evidence;
  - SM-13 is entered only through the handoff to A28, and ACC-02 mirrors its state;
  - every WR transition has a test and every forbidden pair has a refusal test.
  A28 runs SM-13 itself one to one, including the waiver, `old_wallet_frozen`, the timelock and `register_updated`.

## Gauntlet loop for this node

- **Specify.** The frozen spec is this note plus:
  - A26's contract ABIs and the RecoveryModule, SessionKeyModule and RealXPaymaster interfaces (two guardian slots, `veto` needing an owner signature, `expiresAt`, the paymaster owner setters);
  - A27's manifest and Holding read model;
  - A14's approvals contract (action codes, `registerEffector`, `registerPrecheck`, cancel, the `AUTO` path);
  - A15's custody ops, V7 record and provider ports;
  - A12's `introspect`, `session_key_policy` type and outbox events;
  - A11's WebAuthn and fresh-assertion primitives;
  - A22's `startVerification` and `screenWallet`;
  - A28's transfer-request API and events;
  - A06's seed personas;
  - D03 and §4.4 of the sources.

  Before Specify, these **proposed defaults** must be written to [[Decisions]], where the founder can see them:
  1. the WR internal machine for the smart-wallet path, with SM-13 left to A28's fallback (the alternative is reopening A03);
  2. `trusted_contact` as off-chain only;
  3. `GUARDIAN_WAITING_PERIOD = 24 h` and `GUARDIAN_CONFIRM_WINDOW = 7 days`;
  4. `kyt_score = not_applicable_in_phase` in P1 and P2, against the product map's J02 step 9 and the ADM-13 risk column;
  5. `SESSION_KEY_REVOKE_SLA = 60 s`;
  6. the new `AUTO` rows `wallet.recovery.execute` and `wallet.session_key.revoke`, seeded by A03 and A14.

  The node QA writes the hidden oracles to `<workspace>\gauntlet\hidden\A29\` first. They are Vitest and fast-check cases over the mechanism interface, the WR service, the handoff client, the session-key checker and revocation consumer, and the paymaster policy and drift job. They run with a controllable clock, the local Hardhat chain, A15's sandbox custody and the recorded fake of A28's transfer-request route and events.
- **Build (Grok 4.7 xhigh, split by the planner).** Four parallel sub-agents work on disjoint folders:
  1. `packages/adapters/bundler`: manifest, interface, the sandbox on kms and A27's transport (including the veto relay), magic inputs, sandbox control and the conformance suite.
  2. The module core: the mechanism interface and `passkeyAA`; wallet rows and states; counterfactual creation; wallet passkeys with key roles; `setRegistryStatus` and `isRegisteredDestination`; UserOp submission; the paymaster policy signer; the runway and drift jobs; the gas-policy change with its on-chain owner call through A14.
  3. Guardians and recovery: two-slot guardians with confirmation; the WR machine; the A14 action codes, effectors and precheck hook; the notice sandbox and channel snapshot; cancel, veto and dispute; the handoff client and the A28 event mirror; the ADM-13 and ADM-15 read contracts.
  4. Session keys with A12's `introspect` and outbox consumer, fixtures for every investor sub-type, and the screen contract tests for ONB-08, POR-05, POR-06, ACC-02, ADM-13 and ADM-15.

  Then one integration pass on the compose stack:
  - create a wallet and deploy it on the first op;
  - add a second-device guardian through re-verification, the waiting period and confirmation;
  - run a lost-passkey recovery on the smart-wallet path;
  - run a disputed recovery and prove execution is refused;
  - run an unrecoverable case to a registered new wallet, and hand it to A28;
  - register, use and revoke a session key;
  - change the gas policy and check the on-chain caps;
  - drive the runway below 3 days;
  - run `npm run verify`, and write `app/evidence/A29.json`.
- **Challenge (node QA, Opus 5.5 high).** The node QA:
  - runs the hidden oracles and replays every challenge below against the running stack;
  - tries to execute a recovery at hour 47, after a dispute, without notices, and after editing `timelock_until`;
  - submits session-key UserOps outside the policy, both through the API and straight to the EntryPoint on the local chain;
  - presents investor, sponsor, partner and staff API tokens and MCP sessions on every non-delegable and checker route;
  - checks that only the owned paths changed, and that nothing outside `mechanism/` imports A26 or A27 directly;
  - scans the repo, logs, evidence and `state()` output for key bytes, key refs, session private keys or WebAuthn private material;
  - checks that every refusal has a stable code and a plain-English message;
  - compares `GET /v1/wallets/me` and `GET /recovery/{id}` with the ONB-08 CheckList, the Token wallet card and the ACC-02 stepper fields;
  - writes `app/evidence/A29.qa.md`.
- **Gate.** A pass unlocks A49, A56, A59 and A72. A fail returns to Build with the node QA's findings. There are at most 3 fix loops, and each re-runs the full hidden oracle and the failing challenges.
- **Escalate** after the third failed loop on the same assumption. Escalate at once if a fix would:
  - need a change to A26's contracts (for example a guardian `cancelProposal`, an on-chain trusted contact, or a zero-key refusal in `setRecoveryPasskey`), A03's machines or `extensions.ts`, A14's approval contract, A12's mandate type, or A28's transfer-request API. Each of these is a spec defect upstream, reopened there;
  - let any route move an owner, guardian, token or policy without an approval id;
  - let an agent hold or register an owner key;
  - change the 48 h timelock or the 3-day runway, or depart from the proposed defaults recorded in [[Decisions]].

  In any of these cases: split the node, reopen the spec, or ask the user. The paymaster owner conflict is flagged here now. In deployment, A26 names the V1 owner multisig as owner, while A15's `wallet.gas_policy.change` path forbids a V1 authorisation. A15 and A26 must settle which custody account owns the paymaster's `setCaps`, `addTarget` and `removeTarget` before HG-AUDIT. Locally, the deployer account stands in.

## Node-specific challenges (node QA)

- **Handoff to an unregistered wallet (money and holdings).**
  - A recovery that reached `unrecoverable` is handed off while its new wallet is `unregistered`, and again while it is `registering`. Both must be refused with `RECOVERY_NEW_WALLET_UNREGISTERED`, and no A28 transfer request may exist.
  - After A31's fixture event makes the wallet `registered`, the handoff must create exactly one `lost_wallet` transfer request, with `recovery_request_id` set and `transfer_request_id` stored on this row.
  - A second handoff call must return the same transfer request, never a second one.
  - A handoff from any state other than `unrecoverable` must be refused with `WR_ILLEGAL_TRANSITION`.
  - The row's `unrecoverable_reason` and evidence must be present before the call.
- **Timelock, no waiver.**
  - `wallet.recovery.execute` requested at 47 h 59 min must be refused. So must a request with one channel kind's notice missing, and a direct edit of `timelock_until` to the past.
  - Any waiver attempt on a WR request must be refused with `RECOVERY_NO_WAIVER`. None may reach `executing`.
  - The recomputed `timelock_until` from the audit-anchored events must win over the column.
  - On-chain, A26 must also revert (`TimelockNotElapsed`) when the precheck is bypassed on the local chain.
- **Hijacked recovery (security).** An attacker with a stolen login session changes the account email, then starts a wallet recovery with their own new passkey.
  - Notices must reach the previous email and phone from the channel snapshot.
  - The one-click cancel from the old phone, and a cancel from the investor's other signed-in device, must each move the request to `disputed` and raise the Sev2 alert.
  - Execution must then be refused at three layers: the route (`RECOVERY_DISPUTED`), A14's precheck, and A15 (`P1_CANCELLED` for any execute request created before the cancel). `executeRecovery` must never be signed.
  - A re-proposal before `expires_at` must be refused with `RECOVERY_PROPOSAL_STILL_OPEN`, naming the date.
  - With an owner passkey present, the veto must end the on-chain proposal at once.
  - No route may let the attacker skip re-verification or the 48 h wait.
- **Session-key overreach and revocation (G22).** A session key is registered under a mandate limited to token X, 10 units per day and a 7-day expiry.
  - These UserOps are submitted: one calling token Y; one moving 11 units; one arriving after expiry; one calling `RecoveryModule.proposeOwner`; one calling `SessionKeyModule.registerSessionKey`; and one sent straight to the EntryPoint, bypassing the API. Each API call must be refused with `SESSION_KEY_POLICY_VIOLATION` naming the field, and must leave a G22 gate evaluation. The bypass must revert on-chain.
  - Revoking the mandate in A12 must make the next op fail with `SESSION_KEY_MANDATE_INACTIVE` as soon as the revocation commits. This must hold even with the outbox consumer paused.
  - The guardian's on-chain revoke must be recorded within `SESSION_KEY_REVOKE_SLA` with its own `wallet.session_key.revoke` approval id. The same must hold for `agent.token.revoked` and `auth.recovery.completed`.
- **Investor agent on non-delegable acts.**
  - An API token carrying every investor scope calls: create wallet, add key, add guardian, confirm guardian, start recovery, reverify, new-key, veto and register session key. Every one must be refused with `AGNT_NOT_ALLOWED` and a G22 gate evaluation.
  - A `cancel` from the same token must succeed as an off-chain block, and must never produce a veto.
  - The same token calling the staff `triage` route must be refused with `G22_SCOPE_MISSING`.
  - `GET /v1/wallets/me` must still succeed within scope, and the person's activity list must show the refusals.
- **Staff agents.**
  - A KYCA token posting `reverification-result`, a LEGL token posting `dispute-review`, and a CCO token deciding `wallet.recovery.approve` in A14 must each be refused (`AGNT_NOT_ALLOWED`, or `APR_CHECKER_MUST_BE_HUMAN` in A14).
  - A KYCA token submitting `wallet.recovery.approve` as maker, and an ENGO token drafting a gas-policy change, must be accepted, with `maker_principal` showing the agent on behalf of its human.
  - A test-fixture principal who holds both ENGO and CFO, and whose agent drafted the gas-policy change, must be refused as its checker by `checkSod`.
- **Guardian confirmation.**
  - A `setRecoveryPasskey` UserOp pre-signed at request time and submitted at `effective_at` must be refused, both by the route (it accepts only an op built by `confirm`) and on-chain (stale nonce or expired paymaster validity).
  - A confirmation after `GUARDIAN_CONFIRM_WINDOW` must leave the row `expired`.
  - A guardian add during an open recovery, a guardian add without liveness, and a `second_device` credential that is also an owner key must each be refused.
  - A `trusted_contact` must never appear in any on-chain call.
- **Paymaster, policy drift and runway.**
  - A UserOp targeting a non-RealX contract, one from a wallet over its daily gas cap, one from a frozen wallet, and one without a paymaster field must be refused with the right codes.
  - An approved ENGO → CFO cap change must leave three things equal to the approved payload: the DB version, A15's V7 `daily_cap`, and the on-chain `perWalletDailyCapWei` and `dailyCapWei`.
  - A direct on-chain `setCaps` from the local deployer key, outside A14, must raise the drift Sev2 alert within one poll, and sponsorship must fail closed with `PAYMASTER_POLICY_DRIFT`.
  - An ENGO change without the CFO's decision must leave all three unchanged.
  - The runway alert must fire exactly once per breach through A15: once when the deposit falls below 3 days of spend across four consecutive polls, and a second time only after recovery and a second fall.
- **Counterfactual and idempotency.**
  - The address on ONB-08 before deployment must equal the address after the first UserOp with `initCode`.
  - A mint to the counterfactual address before deployment must survive deployment.
  - Two calls to `POST /v1/wallets` for the same investor must return one wallet.
  - 1,000 random investor ids must yield no address collision.
  - Removing the last owner key must be refused.
- **Design fidelity of the data contract.**
  - For the seed investor, `GET /v1/wallets/me` must return `address_short = "0x7a3f…c91e"`, `passkey_count = 3`, `tokens_held_total = 280`, `trust_count = 3`, and `registry_status = registered`, which maps to the label "Linked".
  - `GET /recovery/{id}` must expose `step` and `fallback`, so ACC-02's stepper and banner render without client rules.
  - A `registry_status` value outside A31's five fails the contract test. So does any address truncated by another rule, and any bare 4xx.

## Pass criteria

- The node QA's hidden oracles in `<workspace>\gauntlet\hidden\A29\` pass 100% on three consecutive runs with different seeds. They include:
  - no WR request reaches `executing` before `timelock_until`, after a dispute, or without a notice per channel kind;
  - no WR path offers a waiver;
  - no handoff happens without a recorded `unrecoverable` reason and evidence, and a `registered` new wallet;
  - no session-key UserOp outside its policy is admitted, off-chain or on-chain;
  - no session-key op is admitted after the mandate or token revocation commits;
  - every guardian revoke is recorded on-chain within `SESSION_KEY_REVOKE_SLA` and carries an approval id;
  - no route returns or stores an owner private key or a session private key;
  - no approval id is bound to two custody calls.
- **WR machine.** Every allowed transition has a passing test, and every forbidden pair has a refusal test. 10,000 fast-check sequences of recovery, cancel, veto, dispute, clock, revert, registration and role operations never produce any of the following:
  - `executing` before the timelock, or after a dispute;
  - `closed` without the register check and a clean recon;
  - `handed_off` without `unrecoverable` and a `registered` new wallet;
  - a second open proposal before `expires_at` unless a veto ended the first.
- **Bundler and coverage.**
  - The bundler conformance suite, through A08's harness, covers every verb, deploy-on-first-op, and the sponsored, rejected, delayed and dropped cases, plus determinism under `reset(seed)`.
  - The sandbox's `userOpHash` equals the chain's `UserOperationEvent` hash for 200 random ops.
  - Statement coverage is at least 90% on the adapter and the module, and 100% on `paymaster/policy.ts`, `session-keys/check.ts` and the WR transition executor.
- **Addresses, runway and drift.**
  - The counterfactual address equals the deployed address for 1,000 random investor ids, with no collision.
  - `POST /v1/wallets` is idempotent per investor.
  - The runway alert fires exactly once per breach episode in a 200-poll simulation with two breaches.
  - After every approved gas-policy change, the on-chain paymaster limits, A15's V7 record and the DB version equal the approved payload.
- **Audit, secrets and scope of change.**
  - Every transition, decision, notice, UserOp and refusal leaves exactly one hash-chained audit event.
  - After the integration run the audit chain verifies, and the event count equals the transition and decision count.
  - Every agent-initiated call carries `on_behalf_of`.
  - A scan of the repo, logs, evidence, database dump and `state()` output finds no key material, key refs, session private keys or WebAuthn private material.
  - `npm run verify` passes repo-wide, with only the owned paths changed and the module-boundary lint green (no import of A26 or A27 outside `mechanism/`).
- **Data contracts.** `GET /v1/wallets/me`, `GET /recovery/{id}`, `GET /` (registry), `GET /recovery` (desk) and `GET /paymaster` validate against the SDK schema, and contain exactly the fields that ONB-08, the Token wallet card, POR-06, ACC-02, ADM-13 and ADM-15 list. The seed investor returns the prototype's exact values. A56, A59 and A72's later screenshot comparisons depend on nothing outside these fields.
- **End-to-end runs and evidence.**
  - The end-to-end runs on the compose stack complete three runs in a row with identical audit event counts: create and deploy; guardian set-up through re-verification, waiting period and confirmation; smart-wallet recovery; disputed recovery refused; handoff to A28 for a registered new wallet; session key register, use and revoke; gas-policy change with an on-chain check. The handoff step runs against the recorded A28 fake here; the same run against the real A28 is part of A28's gate and A49.
  - `app/evidence/A29.json` and `A29.qa.md` exist and match disk, with no open critical or high finding.

## Human gates touched

- **HG-LEGAL** (counsel questions, not rules; logged, and they do not block this node):
  - Do the RealX guardian and the recovery power make the wallet a custodial or DAP arrangement under the 2027 framework, and what must the disclosure say? `recovery_disclosure_version` carries counsel's wording, and the `custodial` mechanism stub is the fallback if counsel requires it.
  - Must any recovery, guardian or trusted-contact declaration be personal rather than delegable? These are already non-delegable here. Counsel may relax that, never tighten it, through data in A03's `delegable.ts`.
  - Do the 48 h timelock and the notice wording satisfy counsel?

  All are recorded as open in `app/evidence/A29.json`. The court-order waiver question belongs to A28.
- **HG-VENDOR / HG-ACCOUNTS** (bundler and RPC): a Pimlico or Alchemy account for Base Sepolia is a human step. The conformance suite is the artefact a human runs against it before the implementation may be registered. No vendor call is made in this node, and it passes on sandbox evidence alone.
- **HG-KEYS:** the production RealX guardian key, the paymaster policy key, the paymaster owner and the bundler submitter key are human ceremonies. The kms sandbox stands in for them, and its keys carry no value.
- **HG-AUDIT:** the RecoveryModule, SessionKeyModule and RealXPaymaster that this node drives are custom Solidity in A26, and must be audited before mainnet. This node's tests are software evidence only.
- **HG-PENTEST** (later): the wallet passkey ceremonies, the recovery flow, notices and cancel links, session keys and the paymaster policy are in scope for the external penetration test before go-live. No production keys, real money, mainnet deployment or real investor wallets are touched.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
