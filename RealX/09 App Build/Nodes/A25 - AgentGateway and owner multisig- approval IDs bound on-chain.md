---
title: "A25 - AgentGateway and owner multisig: approval IDs bound on-chain"
created: 2026-09-23
updated: 2026-09-24
status: planned
tags: [realx, app-build, node, am2]
node: A25
milestone: AM2
lane: "Chain"
oracle: yes
human_gate: none
---

# A25 - AgentGateway and owner multisig: approval IDs bound on-chain

**The two custom contracts that make "one approval ID binds the admin approval, the custody signature and the chain transaction" a rule the chain itself enforces. `AgentGateway` is the ERC-3643 Agent for V2, with a second, restrict-only instance for V3. It executes a call, or a bounded atomic batch of calls under one approval (`executeBatch`, for sequences such as unpause plus activation or a multi-trust recovery), only when it has a valid, unexpired, unused and unrevoked EIP-712 attestation from the API's attestation key, every target is registered and its (contract kind, selector) pair is allow-listed, a quorum of distinct signers has signed and none of them is the attested maker, and, for mints, the running total stays within the approval's cap on the one token the approval first minted. Its targets include A26's wallet contracts, so the V2 gateway is the smart-wallet recovery guardian and the V3 gateway the session-key revoker for investor agents (D04). It emits `Executed(approvalId, ...)` for the indexer. `RealXOwnerMultisig` is V1 (3-of-5). Only owners may propose, and a proposal counts as the proposer's own approval unless the proposer is the attested maker; the maker can never approve, so any three non-maker owners can act. Every approval id is used once. Its own allow-list means it can never call a value-moving function. Every owner-level change waits a timelock (48 h by default), and any owner can veto during the queue unless the change affects that owner. About 400 lines of custom Solidity at most, invariant-tested, and audited before mainnet.**

**Build status:** not started. **Milestone:** AM2 (Token and wallet core). **Lane:** Chain. **Planner:** Opus 5.5 (high). **Builder:** Grok 4.7 (xhigh). **Node QA:** Opus 5.5 (high, every loop). **Graph QA:** Fable 5.1 (high, at milestone close).
**Estimate:** 8-12 agent-hours (build, challenge and verify loops).

**Depends on:** [[A24 - ERC-3643 suite and RealX compliance modules]], [[A15 - Custody sandbox adapter and policy engine P1-P10]], [[A26 - Passkey smart wallet, paymaster and guardian recovery contracts]]. The A15 → A25 edge is already in `app-graph.json`; `final-graph.json` is to be regenerated from it, and A15's Unlocks list must name A25. The A26 → A25 edge is **not yet in the graph** (pending graph reconciliation: `app-graph.json` must add A26 to A25's deps and move A26 before A25 in `order`, and A26's Unlocks list must name A25). There is no cycle: A26 depends only on A07, and both nodes are in AM2. A14, A11 and A03 then come in transitively.
**Unlocks:** [[A27 - Chain adapter, deployment manifests and indexer]], [[A106 - IAM admin API for ADM-37]]

**Oracle rule:** The node QA writes hidden oracle tests from this spec to `<workspace>\gauntlet\hidden\A25\` before the build starts ([[App Gauntlet]] §2). The builder never reads that folder. The cases are published into the repo only after the gate passes. The node summary fixes the oracle scope:
- a replayed approval ID is refused, including reuse across the mint and non-mint shapes and across `execute` and `executeBatch`;
- a revoked approval ID is refused;
- a batch runs atomically and consumes its id once, and every inner call passes the same allow-list and restrict-only checks as a single call;
- a mint approval cannot mint on a second token;
- an attestation for one calldata cannot execute another;
- a mint total over the cap fixed at first use reverts;
- a maker among the signers reverts;
- the V3 restrict-only rule holds;
- the multisig proposer and maker binding, its allow-list and denylist (including the inner call of `callModuleFunction`), its liveness with two owner keys lost, and its timelock and veto rules hold.

This node is the on-chain half of the approval-ID binding. [[A15 - Custody sandbox adapter and policy engine P1-P10]] enforces the same binding off-chain (architecture brief §4.3 and §6; requirements catalogue N05 and SM-00). ERC-3643 agent functions take no memo. Without this contract the binding would live only as a convention inside a custody vendor.

Making the Agent a contract means the chain refuses a call that has no valid attestation from the attestation key, no quorum, a reused or revoked id, or a mint over its cap, even if the policy engine is bypassed. The chain does **not** enforce every off-chain rule. In particular, the 48 h timelock on V2 forced transfers and recoveries is enforced off-chain: A15 signs the attestation only after `executable_at`, and A14 refuses execution before `timelock_until`. On-chain, the defence is `revokeApproval`, which lets any gateway signer or the owner multisig kill a signed bundle during the queue. This residual risk is written into the HG-AUDIT pack.

Everything runs on the local Hardhat chain, and on Base Sepolia for integration runs. Mainnet deployment, key ceremonies and the audits are human gates. Every custom line written here goes to the auditors, so smallness is a requirement, not a preference. This plan is not legal advice.

## Build brief

**0. Toolchain pins and encodings (fixed once, used by every part below)**

- **Compiler and libraries.** The solc version is A24's pin (`0.8.17` unless A07's `contracts/TOOLCHAIN.md` says otherwise). OpenZeppelin Contracts is pinned to **4.9.x**, because OZ 5.x needs a later compiler. The gateway and the multisig import OZ `EIP712`, `ECDSA`, `ReentrancyGuard` and `Ownable2Step` unchanged; these are not counted as custom code.
- **Signature recovery.** Every recovery uses `ECDSA.tryRecover`, never `recover`. A malformed, high-s or zero-address result from the attestation signature maps to `BadAttestation`. The same result from a signer signature maps to `BadSigner`. No OZ error string ever surfaces.
- **Selectors and ABIs.** Every selector and function signature comes from A24's `contracts/deploy/suite/exports.json` and the compiled ABIs, never typed by hand. A test regenerates each selector in `gateway-config.json` from the ABI and fails on any difference.
- **Approval id encoding.** A14 issues ids as the ASCII string `APR-<yyyymmdd>-<6 Crockford base32>`, which is 19 characters. The on-chain `bytes32 approvalId` is those ASCII bytes, left-aligned and right-padded with zero bytes to 32. The mapping is reversible, so A27 can print "APR-…" from the log alone. A15's encoder produces this value when it builds the attestation (reconciliation item R4 below). This node fixes the on-chain interpretation. A test decodes `Executed.approvalId` back to the exact APR string, and refuses an id that is not valid padded ASCII. The contracts do not validate the format; only the tests and A15 do.
- **Principal ids.** `principalId = keccak256(bytes(<A11 staff principal id>))`. This is the same value A15 puts in the attestation's `makerId`.
- **Maker id when an agent is the maker (D04, G22).** Segregation of duties counts an agent as its human. When the op's maker is an `AGNT` or `AGT-U` agent acting through a token, `makerId = keccak256(bytes(<the on_behalf_of human's A11 principal id>))`, never the agent's or the token's own id. So the agent's human can never sign or approve that op (`MakerCannotSign`). A `makerId` of zero is refused by the gateway and by the multisig (`MakerRequired`), because a zero id would exclude nobody. A15 must encode this (reconciliation item R4).

**1. `AgentGateway.sol` (`contracts/src/gateway/AgentGateway.sol`, target about 220 lines excluding imports and comments)**

- **Constructor, with no seeding after deploy.** The constructor is `(bytes32 vaultCode, address owner, address attestationSigner, address[] signers, bytes32[] principalIds, uint8 quorum, bool restrictOnly, bytes32[] allowKinds, bytes4[] allowSelectors, address[] targets, bytes32[] targetKinds)`. `owner` is the `RealXOwnerMultisig` address from the moment of deployment, so there is no deployer ownership and no handover of the gateway.
  - The constructor applies the same checks as the setters: quorum between 1 and the signer count, restrict-only pairs only, no zero addresses, and no target equal to the gateway itself or to the owner.
  - Two instances are deployed per chain. **V2** has `vaultCode = "V2"`, `quorum = 2` and `restrictOnly = false`. **V3** has `vaultCode = "V3"`, `quorum = 1` and `restrictOnly = true`.
  - The owner multisig adds both gateways as Agents on every A24 Token. On each compliance, V3 is set as the `HaltModule` halter, and V2 is set as the `HaltModule` lifter and the `LockInModule` lifter, exactly the roles A24 defines. On A26's `RealXGuardianRegistry`, V2 is the `guardian` and V3 the `revoker`, exactly the roles A26 defines. No other role or key is ever granted to a gateway.
- **Signer set, allow-list and target registry are data.** The setters are `addSigner(address signer, bytes32 principalId)`, `removeSigner(address)`, `setQuorum(uint8)`, `setAllowList(bytes32 kind, bytes4 selector, bool allowed)`, `setTargetKind(address target, bytes32 kind)` (kind `0` unregisters a target) and `setAttestationSigner(address)`. All are `onlyOwner`, so every change goes through the V1 timelock.
  - `setTargetKind` refuses `address(this)` and `owner()` (`InvalidTarget`).
  - The kinds are `TOKEN`, `HALT_MODULE`, `LOCKIN_MODULE` and `DOCUMENT_ANCHOR` (A24), plus `RECOVERY_MODULE`, `SESSION_KEY_MODULE` and `GUARDIAN_REGISTRY` (A26). The A26 selectors come from A26's `contracts/deploy/wallet/out/<chainId>.wallet.json` ("selectors A25 must allow-list") and its compiled ABIs, never typed by hand.
  - Because the allow-list is keyed by (kind, selector) and not by address, onboarding a new trust needs only `setTargetKind(token, TOKEN)` on each gateway. The modules, `DocumentAnchor` and A26's modules and registry are shared per chain and are registered once. Smart accounts themselves are never targets: every A26 function the gateways call takes the account as an argument.
  - The seeded sets follow A15's vault records exactly: V2 has three signers; V3 has the CCO, SECO, MLRO and on-call ENGO signers. Quorum can never exceed the signer count and can never be zero.
  - No `AGNT`, `AGT-U`, token or service principal may be a signer. The contract cannot know a principal's type, so `40-verify.ts` and a unit test enforce this against A15's vault fixture (see §3).
- **Attestation.**
  - The EIP-712 domain is `{name: "RealXAgentGateway", version: "1", chainId, verifyingContract}`.
  - The struct is `CustodyAttestation(bytes32 approvalId, address target, bytes32 dataHash, uint256 expiry, bytes32 makerId, uint256 cap, uint256 nonce)`, byte for byte the struct A15 signs. This node never changes it.
  - Because `verifyingContract` is in the domain, an attestation for the V2 gateway is meaningless on the V3 gateway and on any other chain.
  - The digest and recovery logic lives in one internal library, `AttestationLib`, which the gateway calls. The same library backs the test-only vector harness in §4.
  - `verifyAttestation(bytes sig, CustodyAttestation att) → bool` is a public view under the gateway's own domain.
- **`execute`.** The signature is `execute(bytes32 approvalId, address target, bytes calldata data, bytes[] calldata signerSigs, bytes calldata attestation, CustodyAttestation calldata att)`. The whole function is `nonReentrant`. It runs these checks in order and reverts with a named custom error at the first failure:
  1. `att.approvalId == approvalId`, `att.target == target` and `att.dataHash == keccak256(data)`, else `AttestationMismatch`.
  2. `block.timestamp <= att.expiry`, else `AttestationExpired`.
  3. `att.makerId != 0`, else `MakerRequired`.
  4. The attestation signature recovers to `attestationSigner`, else `BadAttestation`.
  5. The per-call checks below pass for `(target, data)`: registered target, allow-listed pair, approval binding inside the calldata.
  6. The replay, revocation and cap guard below passes.
  7. `signerSigs` recover, over the same EIP-712 digest, to distinct addresses in strictly ascending order. Any address that is out of order, duplicated or not a current signer reverts `BadSigner`. That includes a signer removed after signing: the rule is defined once, as `BadSigner`, and such a signer is never silently skipped.
  8. There are at least `quorum` valid signatures, else `QuorumNotMet`.
  9. No recovered signer's `principalId` equals `att.makerId`, else `MakerCannotSign`.
  10. The restrict-only rule below passes.
  11. `target.call(data)` runs, with the revert reason bubbled (`TargetReverted`). All state is written before the call.
- **Per-call checks (one internal function, used by `execute` and by every inner call of `executeBatch`).**
  - `targetKind[target] != 0`, else `TargetNotRegistered`.
  - The allow-list has `(targetKind[target], bytes4(data[:4]))`, else `SelectorNotAllowed`.
  - **Approval binding inside the calldata.** For `HaltModule.forcedTransferOverride(compliance, from, to, value, expiry, approvalId)` the decoded `approvalId` argument must equal the approval id being executed (`OverrideApprovalMismatch`), so an override can never be armed under one approval and consumed under another. For A26's `RecoveryModule.proposeOwner`, `cancelProposal` and `executeRecovery`, the decoded `approvalRef` must equal `keccak256(abi.encodePacked(approvalId))` over the 32-byte id of §0 (`ApprovalRefMismatch`), so the reference A27 matches is the id the chain verified (reconciliation item R9 fixes this encoding with A26).
- **`executeBatch`, a bounded attested multi-call under one approval.** Some operations are several gateway calls that A24 and A28 define under **one** approval id: arming `HaltModule.forcedTransferOverride` and then `forcedTransfer` on a paused token; A24 item 3a's recovery (per token: override if paused, `forcedTransfer`, `setAddressFrozen(new, true)`, and `freezePartialTokens` when a partial freeze was recorded); and `HaltModule.activate` with the first `unpause`. `execute` consumes a non-mint id at first use, so these need one atomic call.
  - The signature is `executeBatch(bytes32 approvalId, Call[] calldata calls, bytes[] calldata signerSigs, bytes calldata attestation, CustodyAttestation calldata att)`, with `Call = (address target, bytes data)` and 1 to 32 calls (`BatchSizeInvalid`). 32 covers A24 item 3a's four calls on each of the eight seed trusts; a larger recovery is split into several approvals by A28.
  - The attestation struct is unchanged. For a batch, `att.target == address(this)` and `att.dataHash == keccak256(abi.encode(calls))`, else `AttestationMismatch`. A15 must sign exactly this (reconciliation item R7).
  - The checks run in the same order as `execute`, with step 5 applied to every inner call and step 10 to every inner call. A mint selector inside a batch is refused (`MintInBatch`): mints stay single `execute` calls with their own nonces and cap. `att.cap` must be zero (`CapShapeInvalid`).
  - The id is recorded as `SINGLE` and consumed once for the whole batch. The inner calls run in order; the first revert reverts the whole batch with its reason bubbled (`TargetReverted`), so a batch never half-applies.
  - Events: one `BatchCall(approvalId, index, target, selector, dataHash)` per inner call, then one `Executed(approvalId, att.nonce, address(this), executeBatch.selector, keccak256(abi.encode(calls)), executor)`. So `Executed` still appears exactly once per approved operation, and A27 can bind it to the op's stored batch calldata by the same rule it uses for single calls (reconciliation item R8).
  - An id used by `execute` cannot be used by `executeBatch`, and the reverse (`ApprovalAlreadyUsed`).
- **Replay, revocation and cap guard.** On first use, each approval id records `kindOf[approvalId]` (`MINT` or `SINGLE`) and, for `MINT`, `capOf[approvalId] = att.cap`.
  - **SINGLE (non-mint selectors).** The id is consumed at first use. Any second `execute` with that id, whatever the nonce, calldata or selector, reverts `ApprovalAlreadyUsed`.
  - **MINT (`mint(address,uint256)` and `batchMint(address[],uint256[])`).** The pair `(approvalId, nonce)` is consumed, so a large allocation can be minted in several chunks under one approval. A replayed pair reverts `NonceAlreadyUsed(approvalId, nonce)`. At first use the gateway also records `targetOf[approvalId] = target`; every later chunk must name the same token (`TargetMismatch`), so a later chunk cannot mint on another trust under the same cap. On every chunk after the first, `att.cap` must equal `capOf[approvalId]` (`CapMismatch`), so a later attestation cannot raise the ceiling. The running total `mintedUnder[approvalId] + amount(data)` must not exceed `capOf[approvalId]` (`MintCapExceeded`).
  - **No reuse across shapes.** An id recorded as `SINGLE` presented with a mint selector, or an id recorded as `MINT` presented with a non-mint selector, reverts `ApprovalAlreadyUsed`.
  - **Cap shape.** `cap` must be zero for non-mint selectors and batches, and non-zero for mint selectors (`CapShapeInvalid`).
  - **Decoding.** `amount(data)` decodes the calldata itself and sums `batchMint` amounts with checked arithmetic. A mint-selector calldata that does not decode exactly, including trailing bytes or mismatched array lengths, is refused with `MalformedMintCalldata`.
  - **Upper bound only.** The gateway enforces only `≤ cap`. The architecture brief's "minted total equals the allocation-file total" is checked off-chain by A28 when it closes the allocation and by A20's reconciliation. It is not checked here.
- **Revocation, the on-chain veto.** `revokeApproval(bytes32 approvalId)` may be called by any current signer of this gateway or by the owner. It marks the id revoked. Any later `execute` or `executeBatch` with that id, including remaining mint chunks, reverts `ApprovalAlreadyUsed`. It emits `ApprovalRevoked(approvalId, by)`. Revoking an id already consumed as `SINGLE` reverts `ApprovalAlreadyUsed`. A15 or A14 call this path when a queued approval is vetoed after its bundle was signed (reconciliation item R5).
- **Restrict-only (V3).** When `restrictOnly` is true:
  - `setAllowList` (and the constructor) accept only seven pairs: `(TOKEN, pause())`, `(TOKEN, setAddressFrozen(address,bool))`, `(TOKEN, batchSetAddressFrozen(address[],bool[]))`, `(HALT_MODULE, <A24 halt selector>)`, and, for the D04 kill switches on investor agents' session keys, `(SESSION_KEY_MODULE, revoke)`, `(SESSION_KEY_MODULE, revokeAll)` and `(GUARDIAN_REGISTRY, pauseSessionKeys)`, with exact signatures from A26's ABI. Anything else is refused with `RestrictOnlyViolation`.
  - `execute` and `executeBatch` decode the boolean argument of `setAddressFrozen`, and every element of `batchSetAddressFrozen`'s `bool[]`, and refuse any `false` (`RestrictOnlyViolation`). V3 can add a restriction but never lift one.
  - A24's `halt(cause)` and A26's `revoke`, `revokeAll` and `pauseSessionKeys` take no boolean, so they need no decoding. Their exact parameter lists are taken from the ABIs. A26 makes revocation irreversible and lets only the guardian or the registry owner call `unpauseSessionKeys`, so V3 can stop an agent but never restore one.
  - These can never be allow-listed on V3: `lift`, `activate`, `unpause`, `setUnlockAt`, `forcedTransferOverride`, `unfreezePartialTokens`, `freezePartialTokens`, `mint`, `batchMint`, `burn`, `batchBurn`, `forcedTransfer`, `batchForcedTransfer`, `recoveryAddress`, `setDocument`, `unpauseSessionKeys`, `proposeOwner`, `cancelProposal` and `executeRecovery`.
- **Events, the indexer's contract with A27.**
  - `Executed(bytes32 indexed approvalId, uint256 nonce, address indexed target, bytes4 selector, bytes32 dataHash, address executor)`.
  - `BatchCall(bytes32 indexed approvalId, uint256 index, address indexed target, bytes4 selector, bytes32 dataHash)` for each inner call of `executeBatch`.
  - `ApprovalRevoked(bytes32 indexed approvalId, address by)`, `SignerAdded(address signer, bytes32 principalId)`, `SignerRemoved`, `QuorumSet`, `AllowListSet`, `TargetKindSet(address target, bytes32 kind)` and `AttestationSignerSet`.
  - `Executed` is emitted once per successful `execute` or `executeBatch`, never on revert. Its `approvalId` is the value the design's adToken tokenOps row prints next to the op id (`o.id · o.appr`).
  - The full event list of both contracts is exported, from the compiled ABIs, as `contracts/deploy/gateway/out/<chainId>/gateway.json#events`, which A27 imports into its `src/events.ts` (reconciliation item R8). A test decodes every event a full integration run emits with an independent viem decoder built only from that export, and fails on any log it cannot decode.
- **Who may call `execute`: anyone.** Authority comes from the attestation and the signer signatures, not from `msg.sender`. The A15 sandbox relayer, a real bundler or an on-call ENGO key can submit an already-signed bundle without holding any power. A bundle cannot be altered in flight, because every field is under the signatures.
- **`SGL+R` through V3 needs two people.** This follows A15's rule. The `SGL+R` actor (CCO, SECO or MLRO) is the op's maker, so the actor's `principalId` is `att.makerId` and `MakerCannotSign` excludes them. A different V3 member signs: 1 of the remaining 3. Whether a one-person emergency action is wanted is already a founder question recorded by A15. This node does not build it.
- **Deliberately absent:** no unattested or unbounded batch (the only multi-call is the attested `executeBatch` above), no delegatecall, no ETH value and no upgradeability. A new version is a new deployment plus an owner-multisig `addAgent`/`removeAgent`. There is no admin bypass and no "emergency" path that skips the attestation: an emergency is a V3 op with an `SGL+R` approval, still attested.

**2. `RealXOwnerMultisig.sol` (`contracts/src/gateway/RealXOwnerMultisig.sol`, V1 owner 3-of-5 with timelock, target about 180 lines)**

- **Constructor, with no seeding after deploy.** The constructor is `(address[] owners, bytes32[] principalIds, uint8 threshold, uint32 defaultDelay, bytes4[] delaySelectors, uint32[] delaySeconds, bytes4[] allowSelectors, address[] permittedAgents)`.
  - There are five owners, each with a `principalId`: CEO, CFO, CCO, SECO and the independent owner, from A15's V1 record. `threshold = 3`.
  - Owner changes and threshold changes are themselves proposals through the same timelocked path. A proposal that would leave fewer owners than the threshold reverts.
- **Owner allow-list, so V1 never moves value.**
  - `allowedSelector[selector]` is seeded from `gateway-config.json`'s V1 list. That list mirrors A15's V1 `function_allow_list`: `addAgent`, `removeAgent`, `setCompliance`, `setIdentityRegistry`, `setOnchainID`, `addModule`, `removeModule`, the vendored implementation authority's version-change selectors and `TREXFactory.deployTREXSuite`. It adds the owner-only configuration calls this node and J12 need: `callModuleFunction` (with its inner call restricted, below), the registries' trusted-issuer and claim-topic setters, ONCHAINID `addKey`/`removeKey`, and every gateway and multisig setter. It also adds A26's owner-only setters, because A26 makes the V1 multisig the owner of the `RealXGuardianRegistry` (`setGuardian`, `setRevoker`, and `acceptOwnership` for its two-step ownership) and of the `RealXPaymaster` (`addTarget`, `removeTarget`, `setCaps`, `setSigner`, and `acceptOwnership` if A26's ABI makes its ownership two-step). A26's paymaster `withdrawTo`, `unlockStake` and `withdrawStake` are **not** allow-listed at launch; adding them is a founder choice through a timelocked `setAllowedSelector`. A15 must add these to its V1 list (reconciliation item R2).
  - The allow-list is changed only by a timelocked proposal whose target is the multisig itself (`setAllowedSelector`).
  - `propose` and `execute` refuse any selector not on the list (`SelectorNotAllowed`).
  - **Hard-coded denylist: every gateway-only selector.** The denylist is the union of every selector on the V2 and V3 gateway allow-lists (§3) plus every ERC-3643 Token agent function, all generated from the ABIs at compile time: the Token agent functions (`mint`, `batchMint`, `burn`, `batchBurn`, `forcedTransfer`, `batchForcedTransfer`, `recoveryAddress`, `setAddressFrozen`, `batchSetAddressFrozen`, `freezePartialTokens`, `batchFreezePartialTokens`, `unfreezePartialTokens`, `batchUnfreezePartialTokens`, `pause`, `unpause`), A24's `HaltModule.halt`, `lift`, `activate` and `forcedTransferOverride`, `LockInModule.setUnlockAt`, `DocumentAnchor.setDocument`, and A26's `proposeOwner`, `cancelProposal`, `executeRecovery`, `revoke`, `revokeAll`, `pauseSessionKeys` and `unpauseSessionKeys`. A24 lets the compliance owner and token owner (the multisig) call several of these directly, so without this list the multisig would be an unattested second route. A denylisted selector can never be allow-listed, even by a self-proposal (`setAllowedSelector` reverts `ValueMovingSelector`), and any proposal naming one reverts `ValueMovingSelector`. A build-time test regenerates the denylist from the ABIs and fails on any difference.
  - **`callModuleFunction` is decoded.** For `ModularCompliance.callModuleFunction(bytes callData, address module)` the multisig decodes the inner selector `bytes4(callData[:4])` at `propose` and at `execute`. It must be on a `moduleConfigSelector` set generated from A24's `exports.json` (module role setters such as the halter and lifter setters, and rule parameters such as holder caps), and must not be on the denylist; otherwise the call reverts `ValueMovingSelector`. So `callModuleFunction(lift)`, `callModuleFunction(setUnlockAt)` or `callModuleFunction(forcedTransferOverride)` can never pass.
  - `addAgent(address agent)` is accepted only when `agent` is in `permittedAgents` (`AgentNotPermitted`). The set starts as the V2 and V3 gateways, and is changed only by timelocked self-proposal.
  - V4 becomes the IdentityRegistry agent at vehicle deploy, through the factory's agent parameters, and in `30-handover.ts` for the seed trusts. It is never added to a Token through the multisig.
  - The multisig cannot decode `deployTREXSuite`'s agent parameters on-chain. A new token deployed by three owners with an unexpected agent is caught by `40-verify.ts`, A27's `verifyConfigured` and A20's reconciliation, not by this contract. This residual risk is written into the HG-AUDIT pack.
- **Proposal lifecycle (the V1 slice of SM-00).**
  - `propose(Call[] calls, bytes32 approvalId, bytes32 makerId) → id` sets the state to `proposed`. It is callable **only by an owner** (`NotOwner`). `Call` is `(address target, bytes data)`. A batch of 1 to 16 calls runs atomically under one approval, so onboarding a new trust or rotating a gateway signer set is one proposal, not many.
  - Each call must pass the allow-list, denylist and agent checks at `propose` and again at `execute`.
  - `approvalId` must never have been used by any earlier proposal, whatever that proposal's final state (`ApprovalAlreadyUsed`). An executed, vetoed, cancelled or expired proposal's id can never be proposed again.
  - The proposal records `proposerId` = the proposer's own `principalId`, set on-chain and not taken from a parameter, plus the declared `makerId`, which is A14's maker as carried by A15 (for an agent-made op, the agent's human, §0). `makerId == 0` reverts `MakerRequired`.
  - **The proposal counts as the proposer's approval, unless the proposer is the maker.** If `proposerId != makerId`, `propose` records the proposer's approval at once. If `proposerId == makerId`, the proposer's proposal counts for nothing and the proposer can never approve it (`MakerCannotSign`). A proposer who calls `approve` on their own counted proposal gets `AlreadyApproved`.
  - `approve(id)` is by an owner who is not the declared maker (`MakerCannotSign`), once each (`AlreadyApproved`). Only the maker is ever excluded, so **any three non-maker owners can act**: with two of the five owner keys lost, the three remaining owners (one proposing, two approving) can still replace the lost owners, and when the maker is one of the five, the other four leave one spare. This is the product map's 3-of-5 with A15's P9 rule ("maker is never a checker") applied, and it is recorded as founder-visible. The earlier rule (proposer never approves) was dropped because, with the maker also excluded, it needed four or five owners and bricked V1 permanently after two key losses.
  - **Residual risk, written into the HG-AUDIT pack.** The multisig cannot verify `makerId`. A proposer who is really the maker could declare another owner as maker and so count themselves. The defences are off-chain and on-chain together: A15's P1 check proposes V1 ops only from A14's recorded request; A27 binds every `Proposed.makerId` to A14's maker for that approval and raises the Sev1 `chain.unbound_change` on a mismatch (reconciliation item R8); and every V1 change still waits its timelock, during which any other owner can veto.
  - The multisig holds no attestation key. The approval id is bound on-chain by uniqueness and by the three owners' approvals. The binding to A14's request is A15's P1 check before the V1 op is proposed.
  - At `threshold` counted approvals the proposal is `queued`, with `executableAt = block.timestamp + max(delayFor(selector) over its calls)`.
  - `veto(id)` by any owner while `queued` moves it to `vetoed`, which is terminal. The one exception is an owner-set change that removes or replaces the vetoing owner. That owner cannot veto it (`ConflictedVeto`), so one compromised owner key cannot block its own removal.
  - `cancel(id)` by the proposer before `queued` moves it to `cancelled`.
  - `execute(id)` by anyone at or after `executableAt` and before `executableAt + 7 days` moves it to `executed` (`NotExecutableYet`, `ProposalExpired`). The calls are made in order, with the first revert reason bubbled.
  - A proposal whose owner set changed since approval recomputes valid approvals at `execute`, and reverts if the threshold no longer holds.
  - Events: `Proposed(id, approvalId, proposerId, makerId, callsHash)` with `callsHash = keccak256(abi.encode(calls))`, then one `ProposedCall(id, index, target, selector, dataHash)` per call with `dataHash = keccak256(data)`, so A27 can join a V1 op to the one call it cares about (for example the `TREXFactory` deploy) without re-deriving the batch; `Approved(id, owner)` (including the proposer's counted approval); `Queued(id, executableAt)`; `Vetoed(id, owner)`; `Cancelled(id)`; `Executed(id, approvalId)`; and `OwnerAdded(owner, principalId)`, `OwnerRemoved(owner)`, `ThresholdSet(threshold)`, `DelaySet(selector, seconds)`, `AllowedSelectorSet(selector, allowed)` and `PermittedAgentSet(agent, allowed)`.
  - **Principal types after deploy.** The contracts cannot know whether a principal id is staff, so a later `addSigner` or owner change could seed an `AGNT`, `AGT-U`, `SVCA` or token principal. `40-verify.ts --principals-only` checks every current signer and owner `principalId` against A11's principal types (through A15's vault fixture locally, or the API's principal lookup when it is up) and exits non-zero on any non-staff or unknown principal. A27 must run this check on every `SignerAdded` and `OwnerAdded` it indexes and raise a Sev1 on failure (reconciliation item R10). A15's P-checks must also refuse to build such a proposal (reconciliation item R4).
- **Delays as data, every value listed in `gateway-config.json` as (target kind, selector, delay).**
  - **48 h (default).** Agent changes (`addAgent`, `removeAgent`), compliance and registry changes (`setCompliance`, `setIdentityRegistry`, `setOnchainID`, `addModule`, `removeModule`, `callModuleFunction`, trusted-issuer and claim-topic setters, ONCHAINID `addKey`/`removeKey`), A26's registry and paymaster owner setters (`setGuardian`, `setRevoker`, `addTarget`, `removeTarget`, `setCaps`, `setSigner`, `acceptOwnership`; A26 already states a 48 h timelock for guardian and revoker rotation), and the multisig's own owner, threshold, delay, allow-list and permitted-agent changes.
  - **72 h.** The implementation authority's version-change selectors and `TREXFactory.deployTREXSuite` ("72 h upgrades" in the product map; factory deploys keep the 72 h the earlier spec set).
  - **24 h.** The gateway configuration setters `addSigner`, `removeSigner`, `setQuorum`, `setAllowList`, `setTargetKind` and `setAttestationSigner`. These are the on-chain fields of a custody policy change, and the product map sets "24 h policy changes". A15 says the multisig delay is that change's clock. Raising them to 48 h is a founder choice, not a builder choice.
  - `setDelay(selector, seconds)` works only through a proposal to the multisig itself. It has a hard floor of 24 h and no zero (`DelayBelowFloor`).
  - There is no fast path and no on-chain delay waiver. A waiver is an off-chain decision that shortens nothing here.
  - Every value in this list is founder-visible. A test asserts each one.
- **Not a Safe fork.** The contract is written small because every line is audited (HG-AUDIT). If the founder later prefers a Safe with a timelock module in production, the swap is one deploy-script change and a re-pointing of ownership. The gateway does not care who its owner is.

**3. Deploy scripts (`contracts/deploy/gateway/`)**

- **`10-owner-and-gateways.ts`** deploys, in three consecutive deployer nonces, `RealXOwnerMultisig`, then the V2 gateway, then the V3 gateway.
  - The multisig constructor receives the V2 and V3 addresses predicted from `(deployer, nonce)` as its `permittedAgents`. Plain CREATE address prediction is independent of the constructor arguments, which breaks the circular dependency. The script asserts that the deployed addresses equal the predictions and aborts otherwise.
  - All constructor data comes from one `gateway-config.json` plus A15's vault fixture: V1 owners and principal ids (from A15's `listVaults()` output when the API is up, or the fixture file with the same keys for pure contract runs), V2 and V3 signers and principal ids, quorums, A15's attestation public key, per-vault (kind, selector) allow-lists, delays, and the target registry built from A24's `deploy/suite/out/<chainId>/shared.json` and `<vehicleCode>.json` manifests plus A26's `contracts/deploy/wallet/out/<chainId>.wallet.json` (RecoveryModule, SessionKeyModule and RealXGuardianRegistry addresses, and A26's list of selectors A25 must allow-list).
  - Allow-lists, from A24's and A26's exact signatures:
    - **V2:** `mint`, `batchMint`, `burn`, `batchBurn`, `forcedTransfer`, `batchForcedTransfer`, `setAddressFrozen`, `batchSetAddressFrozen`, `freezePartialTokens`, `unfreezePartialTokens`, `unpause` on `TOKEN`; `lift`, `activate` and `forcedTransferOverride` on `HALT_MODULE`; `setUnlockAt` on `LOCKIN_MODULE`; `setDocument` on `DOCUMENT_ANCHOR`; `proposeOwner`, `cancelProposal` and `executeRecovery` on `RECOVERY_MODULE`; `revoke` and `revokeAll` on `SESSION_KEY_MODULE`; `pauseSessionKeys` and `unpauseSessionKeys` on `GUARDIAN_REGISTRY`.
    - **Not on V2:** the vendored `recoveryAddress`. A24 item 3a shows it cannot work on the shared identity storage for a holder of more than one trust and says A28 must not call it. Token-level recovery is A24 item 3a's sequence as one `executeBatch` (§1). A config test fails if `recoveryAddress` appears on any gateway list; adding it back is an escalation, not a builder choice.
    - **V3:** `pause`, `setAddressFrozen`, `batchSetAddressFrozen` on `TOKEN`; `halt` on `HALT_MODULE`; `revoke` and `revokeAll` on `SESSION_KEY_MODULE`; `pauseSessionKeys` on `GUARDIAN_REGISTRY`.
    - **V1:** as in §2.
  - Addresses are **per environment**. The signer sets and attestation keys differ between the local chain, Base Sepolia and production, so addresses are not claimed to be equal across environments. They are reproducible on two fresh chains of the same environment with the same config.
- **`30-handover.ts`** runs while the deployer still holds A24's local roles. It adds both gateways as Agents on each deployed Token. It sets V3 as halter, and V2 as lifter, on each compliance's `HaltModule` and `LockInModule`, through `callModuleFunction` or A24's setter. It registers A26's RecoveryModule, SessionKeyModule and RealXGuardianRegistry as targets on both gateways (in the constructor data). Then it moves **every A24 and A26 privileged slot**, read from A24's `deploy/suite/out/<chainId>/` manifests and A26's `deploy/wallet/out/<chainId>.wallet.json`, to its final holder:

  | A24 or A26 slot | Holder after handover |
  |---|---|
  | `IdentityRegistry` agent (each vehicle) | V4 identity agent added; deployer removed |
  | `IdentityRegistry`, `IdentityRegistryStorage`, `TrustedIssuersRegistry`, `ClaimTopicsRegistry`, `ModularCompliance`, `Token` and compliance-module owners, and `DocumentAnchor` owner if it is ownable | multisig |
  | `RealXClaimIssuer` management key (ONCHAINID purpose 1) | multisig added; deployer removed |
  | `RealXClaimIssuer` claim key | stays V4, as A24 set it |
  | `TREXFactory` and `TREXImplementationAuthority` owners | multisig (72 h selectors) |
  | Each token's ONCHAINID management key | multisig added; deployer removed |
  | `IdFactory` owner | V4 identity agent (reason below) |
  | A26 `RealXGuardianRegistry` `guardian` / `revoker` | V2 gateway / V3 gateway, set by `setGuardian` and `setRevoker` while the deployer still owns the registry (replacing A26's `TestRealXGuardian` and `TestRealXRevoker` stubs) |
  | A26 `RealXGuardianRegistry` owner (Ownable2Step) | multisig: the deployer calls `transferOwnership(multisig)`; the multisig's first batch proposal calls `acceptOwnership` |
  | A26 `RealXPaymaster` owner | multisig (two-step as above if A26's ABI makes it two-step); the policy signer stays as A26 and A29 set it |

  A31 needs a management key on the ClaimIssuer, because `revokeClaim` is manager-only on the vendored ClaimIssuer. It also needs `IdFactory.createIdentity`, which is `onlyOwner`, once per investor. So:
  - the `IdFactory` owner is V4 and not the multisig, and V4 is additionally a ClaimIssuer management key if A24's ABI makes revocation manager-only;
  - A15's V4 allow-list must carry `createIdentity`, and A31 must confirm this split (reconciliation item R3);
  - V4 holding `addTokenFactory` on the IdFactory is a residual risk recorded in the HG-AUDIT pack.

  The vendored T-REX v4 contracts and ONCHAINID use one-step ownership transfers and direct key changes, so no acceptance step is needed. A26's registry uses two-step ownership, and any A24 or A26 contract that does is handled the same way: its `acceptOwnership` is in the multisig's first batch proposal, and `40-verify.ts` reports "pending acceptance" until that proposal executes. The deployer's agent and key roles are then renounced. A26 says A27 later rotates the guardian and revoker; with this table, A27 only verifies them (reconciliation item R9).
- **`40-verify.ts`** reads every slot in the table back and fails if:
  - any deployer role or key remains;
  - V4 is not the IdentityRegistry agent;
  - any V3 allow-list entry could lift a restriction or restore a session key;
  - `recoveryAddress` is on any gateway list;
  - A26's registry `guardian` is not V2, its `revoker` is not V3, or its or the paymaster's owner is not the multisig (or still pending acceptance, reported as such);
  - the multisig allow-list contains a denylisted selector, or its `moduleConfigSelector` set contains one;
  - any on-chain signer or owner `principalId` maps, in A15's vault fixture, to an `AGNT`, `AGT-U`, `SVCA` or token principal, or to no staff principal at all;
  - the config hash differs from `gateway-config.json`.
- **Output.** Every script writes `contracts/deploy/gateway/out/<chainId>/gateway.json` (inside this node's owned paths, mirroring A24's `deploy/suite/out`). The file holds the multisig and gateway addresses, deploy blocks, the config hash, ABI hashes and the handover result. A27 imports this file into its manifest; A25 does not call A27's `writeManifest` or `contracts:export`, which do not exist yet. A15 learns the deployed gateway addresses, and so its attestation domain's `verifyingContract`, from A27's manifest, replacing its placeholder.
- **Safety.** Every script is idempotent: re-running it against an already-deployed chain changes nothing and reports "no change". Every script refuses to run when the configured chain id is a mainnet id while `RUN_MODE=local|test`. No private key is read from the repo: local runs use Hardhat's deterministic accounts, and testnet runs read a faucet-funded key from the `kms` adapter.

**4. Tests (`contracts/test/gateway/`)**

- **Unit tests** (Hardhat, Chai), one per custom error and per event. A shared fixture deploys the A24 suite for one trust plus this node's contracts through the real deploy scripts, and registers three identity-verified wallets.
- **Property and model-based tests** (fast-check over the Hardhat network, with a controllable clock). Thousands of random sequences of attestations, signer bundles, single calls and batches, revocations, allow-list and target changes, signer removals and multisig proposals check the invariants in the pass criteria. The state model is the SM-00 slice (`queued_for_custody → signed → broadcast → confirmed`) plus the multisig proposal machine (`proposed → queued → executed | vetoed | cancelled | expired`).
- **Attestation vectors.** A15's `attestation-vectors.json` is consumed unchanged, and each vector carries its own domain (chain id, pinned name and version, per-vault `verifyingContract`). A test-only `GatewayVectorHarness` in `contracts/test/gateway/` calls the gateway's own `AttestationLib` with the vector's domain, so every valid vector verifies and every mutated vector is refused by the exact code path `execute` uses. Separately, end-to-end tests issue fresh attestations through A15's `issueAttestation` code path, signed with the fixture attestation key, against the deployed gateways' real domain.
- **Cross-checks.** The V1, V2 and V3 allow-lists in `gateway-config.json` are checked against A15's vault `function_allow_list` records, and every selector is checked against A24's `exports.json` ABIs.
- **Event contract.** The independent-decoder test of §1 (every event of both contracts from a full integration run, decoded only from the exported event list) and a check that the export's event table equals the compiled ABIs.
- **Evidence and static analysis.** A gas report for `execute` (mint, a 100-line batchMint, pause) and for the multisig path is written to evidence. `solhint` runs. When A07's toolchain provides slither in Docker, a static-analysis run records any finding; otherwise "not run" is recorded rather than claimed.

## Owned paths (other nodes do not edit these)

- <workspace>\app\contracts\src\gateway\
- <workspace>\app\contracts\test\gateway\
- <workspace>\app\contracts\deploy\gateway\ (including `out/<chainId>/gateway.json`)

This node only *uses*:
- A24's compiled suite, its `exports.json`, its `deploy/suite/out/` manifests and its `HaltModule` and `LockInModule` role hooks;
- A26's compiled wallet contracts, ABIs and `deploy/wallet/out/<chainId>.wallet.json` (addresses, owners, and the selectors A25 must allow-list);
- A07's Hardhat configuration and deterministic accounts;
- A15's attestation vector file, vault-config fixture and `issueAttestation` code path (read-only, in tests).

It changes nothing under `contracts/src/token`, `contracts/src/compliance`, `contracts/deploy/suite`, A26's `contracts/src/wallet` and `contracts/deploy/wallet`, or any TypeScript package. The API-side `chain` module and the manifests belong to A27.

**Cross-node reconciliation items (raised by this node; the owners change their own specs; A25 escalates rather than editing them):**
- **R1 (A15).** A15's V2 and V3 `function_allow_list` and its P2 wording use `HaltModule.setHalt(address,bool)`, which A24 does not have. They must use A24's `halt(cause)` on V3 and `lift()` on V2. V2 must add `HaltModule.forcedTransferOverride`, `HaltModule.activate`, `LockInModule.setUnlockAt`, `DocumentAnchor.setDocument` and A26's wallet selectors (§3), and must drop `recoveryAddress`. V3 must add A26's `revoke`, `revokeAll` and `pauseSessionKeys`. The allow-lists must be keyed by (kind, selector) as in §3. Until A15 matches, the cross-check test fails by design and the node escalates.
- **R2 (A15).** A15's V1 list must add the owner configuration selectors in §2: `callModuleFunction`, registry setters, ONCHAINID key changes, the gateway and multisig setters, and A26's registry and paymaster owner setters and `acceptOwnership`.
- **R3 (A15, A31, A24).** `IdFactory` ownership and ClaimIssuer management by V4, and `createIdentity` on V4's allow-list, as in §3's handover table. A31's "V4 is the ClaimIssuer management key in the sandbox" and A24's "management key is the multisig after A25" must agree with that table.
- **R4 (A15, A14).** The bytes32 approval id encoding in §0 (ASCII right-padded) must be stated in A15's encoder and vectors. A15 must also state the agent maker rule of §0 (for an agent-made op, `makerId` is the `on_behalf_of` human's principal id hash, never zero), add a vector where the agent's human signs and the gateway must revert `MakerCannotSign`, and refuse to build any `addSigner` or owner-change proposal whose principal is not a staff principal.
- **R5 (A15, A14).** A queued-approval veto after signing must call `revokeApproval` on the op's gateway through a signer key. A15's placeholder `gateway_address` wording ("derived from A25's CREATE2 salts") becomes "read from A27's manifest".
- **R6 (A27).** A27 says A25 calls `writeManifest()` and `contracts:export`; it must instead import `contracts/deploy/gateway/out/<chainId>/gateway.json`. A27's `sandbox_deployed` path attaches a new token to the gateways "with A07's deployer account 0" after handover. That contradicts "no admin bypass". After handover, a sandbox deploy of a new trust must go through one multisig batch proposal (§2), or run before `30-handover.ts` in a fresh local chain only.
- **R7 (A15).** For `executeBatch`, A15 signs the unchanged `CustodyAttestation` struct with `target = <gateway address>` and `dataHash = keccak256(abi.encode(calls))`, where `calls` is the ordered `(address target, bytes data)[]`, and stores that encoded batch as the op's calldata. A15's vectors must include a batch vector and a mutated batch vector (one inner call changed, calls reordered). A15's per-op policy checks run on every inner call.
- **R8 (A27).** A27's event table must match §1 and §2: V1 ops are joined through `ProposedCall(id, index, target, selector, dataHash)` (not a `Proposed` carrying target and selector), so the J12 deploy join reads the `ProposedCall` whose target is the manifest `trexFactory`; A27 must decode `BatchCall`, `ApprovalRevoked`, `TargetKindSet`, `OwnerAdded`, `OwnerRemoved`, `ThresholdSet`, `DelaySet`, `AllowedSelectorSet` and `PermittedAgentSet`, bind each `ApprovalRevoked` to the vetoed `CustodyOp`, bind every `Proposed.makerId` to A14's maker for that approval (Sev1 `chain.unbound_change` on a mismatch), and import the event list from `gateway.json#events`. As written today, every V1 op would fail the join and every revocation would trip `chain.indexer_inconsistent`.
- **R9 (A26, A27).** A26's `approvalRef` is `keccak256(abi.encodePacked(bytes32 approvalId))` over the §0 encoding, so the gateway's `ApprovalRefMismatch` check and A27's match agree. The guardian and revoker are set to the V2 and V3 gateways by A25's `30-handover.ts` while the deployer still owns the registry (not rotated later by A27, which only verifies). A26's Unlocks list must name A25.
- **R10 (A27, A20).** On every indexed `SignerAdded` or `OwnerAdded`, A27 runs A25's `40-verify.ts --principals-only` check (or the same rule in its own code) and raises a Sev1 when a signer or owner is an `AGNT`, `AGT-U`, `SVCA` or token principal, or no staff principal at all.
- **R11 (A28, A24).** A28 must compose the multi-call operations as one `executeBatch` under one approval: a forced transfer on a paused token (`forcedTransferOverride` with the same approval id, then `forcedTransfer`); A24 item 3a's recovery per token; and `HaltModule.activate` with the first `unpause`. A28's `recovery` row must name A24 item 3a's sequence, not `recoveryAddress` (A24 already raises this). A28's J09 smart-wallet path uses A26's `proposeOwner` and `executeRecovery` through V2, each under its own approval id, as A26 states.
- **Graph (app-graph.json).** Add A26 to A25's deps and place A26 before A25 in `order`. The A25 summary says "About 200 lines of custom Solidity"; it should read "at most 400 lines (gateway about 220, multisig about 180)" to match this spec.

## Design references

- `design:adToken approval id per op` — `<workspace>\design\RealX App.dc.html`, admin shell `ad`, screen `adToken` (ADM-14). The adToken **tokenOps row** (an `sc-for` over `tokenOps` printing `{{ o.id }} · {{ o.appr }}`) prints "OP-7730 · APR-2279" in Geist Mono, and the "Recent" table has an Approval column (APR-2270, APR-2268, APR-2261) beside a Tx column (0xa1…9f). Nothing in this node renders a pixel. "Done" for design means that the `Executed(approvalId, nonce, target, selector, dataHash, executor)` event carries exactly what lets A27's indexer fill those two columns from the transaction log alone. The approval ID a checker saw on `adApprovals` ("Approval ID is bound to the signing request") is the same value the chain emitted, recovered from the 32 bytes by the §0 encoding.
- The sub-line "Every on-chain operation needs an approval ID and a 2-of-3 custody quorum before broadcast" is the plain-English statement of the V2 gateway's two checks. The toast "{id} broadcast · register sync queued" marks the moment the `Executed` event exists and reconciliation has not yet closed the op.
- Undesigned surfaces this node feeds are drawn in [[A51 - Design extension B- operator console screens not in the prototype]] and built by [[A72 - Console- token operations, forced transfer and recovery desk, wallets and custody, token registry]]. They are ADM-13's custody KeyValueList (V1 3-of-5 with "48 h timelock", V2 2-of-3, V3 1-of-N restrict-only) and ADM-15's Timelock column and `executable_at` countdown. For owner-level changes these read the multisig's `Queued(id, executableAt)` event. Visual palette and type come from A05; this node supplies only on-chain facts.

## Requirements covered

- **Product-map screens:** none owned. Done means the on-chain events give ADM-14, ADM-13 and ADM-15 the approval id, nonce, selector, executor, revocation and `executableAt` they display, with no field the screens need missing from the event log.
- **Journeys:** none owned. Done means each of these steps has an allow-listed route through the right gateway or the multisig, proven by one test per step against the A24 suite:
  - J04 step 11: `batchMint` per locked allocation file.
  - J08 step 10 and J21 step 6: burns.
  - J09 steps 6 and 8, token level: freeze through V3 (one `executeBatch` across the holder's trusts), then, after the off-chain timelock, A24 item 3a's sequence as one V2 `executeBatch` under one approval (per token: `forcedTransferOverride` if paused, `forcedTransfer`, `setAddressFrozen(new, true)`, `freezePartialTokens` if a partial freeze was recorded), proven on a two-trust holder with one token paused. Never `recoveryAddress`.
  - J09, smart-wallet level (the primary path): A26's `RecoveryModule.proposeOwner` through V2 under one approval, then `executeRecovery` through V2 at 48 h under a second approval, with the investor's session keys revoked in the same transaction.
  - J22 and D04: V3 revokes one investor agent's session key (`revoke`), all of an investor's keys (`revokeAll`), and pauses every session key (`pauseSessionKeys`); V3 can never unpause them.
  - Token activation (SM-06 `issuing → active`): `HaltModule.activate` plus the first `unpause` as one V2 `executeBatch`.
  - J10 step 3: pause under `SGL+R` through V3, with the CCO as the named maker and the SECO as the named signer, and the CCO's own signature refused.
  - J12 step 9: deploy and configure a ninth trust as **one** multisig batch proposal under the V1 owner quorum, followed by V2 and V3 target registration, which is part of the same batch.
- **Gate G13 Maker-checker:** done means the chain refuses:
  - any agent call without a valid attestation from the attestation key;
  - a reused or revoked approval ID, including reuse across the mint and non-mint shapes;
  - a signer whose principal is the attested maker, including the human behind an agent maker (G22: the agent counts as its human), and a zero maker id;
  - fewer than the quorum of distinct current signers;
  - a multisig proposal from a non-owner, a counted approval by the declared maker, or a second approval by the proposer;
  - any owner-level change that has not waited its timelock.

  Every one of those refusals is a named custom error that a test exercises.
- **Module M09 Token operations:** done means every token operation the product map lists has exactly one allow-listed route, the V3 route can only restrict, the multisig can never call a value-moving selector, and the three allow-lists equal A15's vault allow-lists by a cross-check test (after reconciliation item R1). The operations are:
  - for V2 and V3: mint, batch mint, burn, freeze, partial freeze, unfreeze, pause, unpause, activation (`HaltModule.activate`), halt, lift, lock-up change, forced-transfer override, forced transfer, recovery (A24 item 3a's sequence, and A26's smart-wallet recovery), session-key revocation and pause, and document anchoring;
  - for V1: deploy, configure, agent change, compliance change, upgrade, and A26's guardian, revoker and paymaster configuration.

  "Exactly one route" is checked against both lists: a test fails if any gateway selector is also reachable through the multisig, directly or inside `callModuleFunction`.
- **State machine SM-00 Privileged action:** done means:
  - `queued_for_custody → signed → broadcast → confirmed` corresponds one to one to "bundle assembled by A15 → `execute` or `executeBatch` mined → `Executed` observed";
  - `broadcast → failed` corresponds to a mined transaction that reverted with one of this node's named custom errors (or a bubbled `TargetReverted`), with no `Executed` event; a test maps each named revert to SM-00 `failed`;
  - the invariant "the approval ID travels in the custody transaction" holds, because it is a field of the transaction the chain verified;
  - "approvals expire" holds on-chain through `expiry`, and a vetoed approval is killed on-chain through `revokeApproval`;
  - "maker ≠ any checker" extends to signers through `makerId`, and to multisig approvers through `makerId` (the proposer's proposal counts as an approval only when `proposerId != makerId`);
  - the owner-level machine `proposed → queued → executed | vetoed | cancelled | expired` refuses every transition not in that list.

## Gauntlet loop for this node

- **Specify.** The frozen spec is:
  - this note;
  - A24's deployed interfaces, `exports.json` and `deploy/suite/out/` manifests (Token agent functions, `HaltModule.halt`, `lift` and `forcedTransferOverride`, `LockInModule.setUnlockAt`, `DocumentAnchor.setDocument`, and module, registry, factory and key owners);
  - A26's wallet contracts' ABIs and `deploy/wallet/out/<chainId>.wallet.json` (RecoveryModule, SessionKeyModule and RealXGuardianRegistry functions, registry and paymaster owner setters, and the selectors A25 must allow-list);
  - A15's attestation struct, vector file, vault allow-lists and vault fixture;
  - A14's approval id format;
  - A03's SM-00 table;
  - §4.2–§4.3 and §6 of the architecture brief, and the key-hierarchy paragraph of the requirements catalogue (D03).

  The node QA writes the hidden oracles to `<workspace>\gauntlet\hidden\A25\` first: Hardhat test files with their own attestation signer key, synthetic signer and owner sets with principal ids, mutated vectors and a time-travel harness.
- **Build (Grok 4.7 xhigh, split by the planner).** Three parallel sub-agents work on disjoint files:
  1. `AgentGateway.sol` and `AttestationLib`, with their unit tests, the vector harness and the vector test;
  2. `RealXOwnerMultisig.sol` with its unit tests;
  3. `gateway-config.json`, the three deploy scripts, the A15 and A24 allow-list cross-checks and the fast-check model-based suite.

  Then one integration pass on Hardhat:
  - deploy the A24 suite for two trusts and A26's wallet contracts, and run the scripts;
  - execute a chunked `batchMint` of 9,100 units under one approval in two nonces, then replay both;
  - run `activate` plus `unpause` as one V2 `executeBatch`, then replay it;
  - run a V3 pause signed by a non-maker and a refused V3 unpause;
  - run a two-trust token-level recovery (A24 item 3a) with one token paused as one V2 `executeBatch`, then replay it;
  - run an A26 smart-wallet recovery (`proposeOwner`, 48 h, `executeRecovery`) through V2, and a V3 `revoke`, `revokeAll` and `pauseSessionKeys` on an investor agent's session key;
  - run a V2 `lift` and a `setUnlockAt`;
  - revoke a signed bundle;
  - propose an `addAgent` through the multisig;
  - veto one proposal and execute another after 48 h of time travel;
  - onboard a ninth trust as one batch proposal;
  - run the gas report and static analysis, and run `npm run verify`;
  - write `app/evidence/A25.json` with line counts of custom Solidity, ABI hashes and the config hash.
- **Challenge (node QA, Opus 5.5 high).**
  - Runs the hidden oracles and replays every challenge below.
  - Re-derives the EIP-712 digest independently in TypeScript and checks it equals the contract's.
  - Fuzzes `execute` with random selectors, malformed mint calldata and mixed-case signature orderings.
  - Checks that the deployer holds no role after deployment, and diffs the three allow-lists against A15's vault records.
  - Counts custom Solidity lines, and confirms that only the owned paths changed and no A24 file was touched.
  - Checks that every revert is a named error and every success emits exactly one `Executed`.
  - Writes `app/evidence/A25.qa.md`.
- **Gate.** A pass unlocks A27 and A106, and through A27 it unlocks A28, where the AM2 exit test lives. A fail returns to Build with the node QA's findings. There are at most 3 fix loops, each re-running the full hidden oracle, the model-based suite and the failing challenges.
- **Escalate** after the third failed loop on the same assumption, or at once if a fix would:
  - change the attestation struct (breaking A15's vectors), including adding an on-chain `notBefore` for timelocked V2 ops;
  - add an admin bypass or an untimelocked path;
  - grow custom Solidity past 400 lines;
  - change a quorum or timelock value from the product map or from the delay table in §2 (these are founder-visible);
  - need edits to A24's or A26's contracts;
  - need A15, A26, A27 or A28 to change their specs (reconciliation items R1–R11);
  - put `recoveryAddress` back on a gateway list, or allow a batch larger than 32 calls.

  The response is to split the node, reopen the spec or ask the user.

## Node-specific challenges (node QA)

- **Money and supply.**
  - An approval attests `cap = 9,100`. Nonce 1 mints 5,000 and nonce 2 mints 4,100. Nonce 3 mints 1 and must revert `MintCapExceeded`. Replaying nonce 2 must revert `NonceAlreadyUsed(approvalId, 2)`.
  - Chunk 2 attested with `cap = 20,000` under the same id must revert `CapMismatch`.
  - Chunk 2 under the same id and cap, but with `target` set to a second trust's token, must revert `TargetMismatch`, and the second token's supply stays zero.
  - An id first used for a `burn` and then presented with a `mint` must revert `ApprovalAlreadyUsed`. So must an id first used for a mint and then presented with a `burn`.
  - A `batchMint` whose amounts sum to exactly `cap`, but whose calldata is padded with an extra recipient or trailing bytes, is refused with `MalformedMintCalldata`.
  - A `batchMint` whose amounts sum past `2^256` reverts on checked arithmetic, not by wrapping to a small number.
  - After every sequence, total supply equals the sum of accepted mints and never exceeds the cap fixed at first use.
- **Multi-call operations under one approval (`executeBatch`).**
  - Forced transfer on a paused token: a V2 batch `[forcedTransferOverride(compliance, lost, new, v, expiry, APR-x), forcedTransfer(lost, new, v)]` under APR-x succeeds and emits one `OverrideArmed`, one `OverrideConsumed`, two `BatchCall` and one `Executed`. The same pair sent as two `execute` calls fails at the second (`ApprovalAlreadyUsed`). An override whose calldata carries APR-y inside an APR-x batch reverts `OverrideApprovalMismatch`. Replaying the batch reverts `ApprovalAlreadyUsed`.
  - Activation: `[activate(compliance), unpause()]` under one approval moves the token to SM-06 `active` (A24's `explainToken` shows no `UNPAUSED_NOT_ACTIVATED` flag). Replaying it reverts `ApprovalAlreadyUsed`. `activate` proposed to V3's `setAllowList` reverts `RestrictOnlyViolation`.
  - Two-trust recovery: A24 item 3a's sequence for a holder of two trusts, one paused and one with a partial freeze, as one batch under one id: both tokens recover, the new wallet ends frozen on both, and the partial freeze is re-applied. Replaying it reverts `ApprovalAlreadyUsed`.
  - Atomicity: the same batch with its last inner call made to revert (for example `forcedTransfer` over the balance) leaves every balance, freeze, override and `usedApproval` unchanged.
  - A batch containing a `mint` reverts `MintInBatch`; a batch of 0 or 33 calls reverts `BatchSizeInvalid`; a batch with one inner call not allow-listed, or to an unregistered target, reverts `SelectorNotAllowed` or `TargetNotRegistered`; a V3 batch with one `setAddressFrozen(x, false)` among restricting calls reverts `RestrictOnlyViolation`; a batch attestation whose `dataHash` covers the calls in a different order reverts `AttestationMismatch`.
  - An id used by `execute` presented to `executeBatch`, and the reverse, reverts `ApprovalAlreadyUsed`.
- **Wallet recovery and agent kill switches (A26, D04).**
  - J09 smart-wallet recovery through the real V2 gateway: `proposeOwner` under APR-a, then `executeRecovery` under APR-b after 48 h of time travel, succeeds; the account has the new passkey and no live session key. `executeRecovery` whose `approvalRef` is not the hash of the executed id reverts `ApprovalRefMismatch`. Replaying either step reverts `ApprovalAlreadyUsed`.
  - D04 through the real V3 gateway: `revoke(account, keyId)` of an investor agent's session key, signed by one non-maker V3 member, stops that agent's next UserOp; `revokeAll` and `pauseSessionKeys` do the same for all keys; `unpauseSessionKeys`, `proposeOwner` and `executeRecovery` on V3's `setAllowList` revert `RestrictOnlyViolation`.
  - After handover, the registry's `guardian` is V2 and `revoker` is V3, and a direct `setGuardian` from the deployer reverts.
- **Compliance and segregation of duties.**
  - The CCO made APR-2279 and is also a V2 signer in a synthetic set. A bundle with the CCO's signature plus one other reverts `MakerCannotSign`, even though two signatures are present.
  - A bundle with the same signer's signature twice reverts `BadSigner`.
  - A signer removed by the multisig after signing, but before `execute` is mined, makes the bundle revert `BadSigner`.
  - J10: the CCO as `SGL+R` maker signing their own V3 pause reverts `MakerCannotSign`, and the SECO's signature executes it.
  - On V3, an attestation for `setAddressFrozen(wallet, false)`, or a `batchSetAddressFrozen` with one `false` among the elements, reverts `RestrictOnlyViolation` even with a valid attestation and signature.
  - The multisig cannot allow-list `unpause`, `lift`, `setUnlockAt` or `forcedTransferOverride` on V3.
  - A signer or owner seeded with an `AGNT` principal id from the fixture makes `40-verify.ts` fail.
  - Agent maker (G22): an agent token acting for the CFO is the maker, so `makerId` is the CFO's principal hash; a bundle the CFO signs as a V2 signer reverts `MakerCannotSign`. A bundle with `makerId = 0` reverts `MakerRequired`, on the gateway and on the multisig.
  - After deploy, a timelocked multisig proposal `addSigner(<address>, <AGNT principal id>)` executes on-chain (the contract cannot know principal types), and `40-verify.ts --principals-only` then exits non-zero naming that signer.
- **Security.**
  - An attestation for calldata A presented with calldata B of the same length.
  - An attestation signed by a V2 signer's key instead of the attestation key.
  - An attestation for the V2 gateway presented to the V3 gateway (the domain's `verifyingContract` differs).
  - An attestation for chain id 31337 presented on a fork with chain id 84532.
  - An expired attestation.
  - A high-s malleated signature, which must yield `BadAttestation` or `BadSigner` and never an OZ error.
  - A target contract that re-enters `execute` with the same bundle during the call.
  - An `execute` whose `target` is unregistered, the gateway or the multisig (`TargetNotRegistered`), and `setTargetKind(gateway or owner)` (`InvalidTarget`).
  - A revoked approval presented with a valid bundle (`ApprovalAlreadyUsed`), and `revokeApproval` from a stranger.

  Each must revert with the expected error and leave `usedApproval`, `usedNonce`, `kindOf`, `capOf`, `targetOf` and `mintedUnder` unchanged.
- **Timelock and governance.**
  - An `addAgent` proposal approved by 3 of 5 at hour 0 is executed at hour 47 (`NotExecutableYet`).
  - The same proposal is vetoed at hour 47 and executed at hour 49 (`vetoed` is terminal).
  - It is executed at day 9 (`ProposalExpired`).
  - A `setDelay` proposal to 0 or to 12 h (`DelayBelowFloor`).
  - An owner-set change to two owners with threshold 3.
  - A non-owner proposes (`NotOwner`); the proposer approves their own counted proposal again (`AlreadyApproved`); an owner who is the declared maker proposes, the proposal shows zero counted approvals, and their `approve` reverts `MakerCannotSign`; a proposal with `makerId = 0` reverts `MakerRequired`.
  - Liveness with two owner keys lost: the three remaining owners (one proposing, two approving) replace a lost owner after 48 h. With SECO as the maker on "Change signer policy", the three non-maker owners among the remaining four pass it, and SECO's approval reverts `MakerCannotSign`.
  - Unattested second route: proposals of `callModuleFunction(<HaltModule.lift calldata>, haltModule)`, a direct `HaltModule.lift`, `HaltModule.activate`, `LockInModule.setUnlockAt`, `DocumentAnchor.setDocument` and `RecoveryModule.executeRecovery`, and `setAllowedSelector(setUnlockAt)` as a self-proposal, each revert `ValueMovingSelector`. A `callModuleFunction` whose inner call is a module role setter on the `moduleConfigSelector` set passes.
  - An executed proposal's `approvalId` is proposed again (`ApprovalAlreadyUsed`).
  - A proposal of `Token.mint` or `forcedTransfer` (`ValueMovingSelector`), and a proposal of `addAgent(multisig)` or `addAgent(<EOA>)` (`AgentNotPermitted`).
  - An owner vetoes their own replacement (`ConflictedVeto`), and a compromised owner vetoes every other proposal while the proposal removing them still executes after its delay.
  - A call to `AgentGateway.setAllowList` from the deployer key after handover.
  - Each delay in the §2 table is asserted by a test.
- **Handover.** After `30-handover.ts`, V4's `registerIdentity` on the IdentityRegistry succeeds and the deployer's reverts. Every slot in the handover table holds its listed holder.
- **Design fidelity.** For the seeded OP-7730 mint bound to APR-2279, the `Executed` event decoded by an independent viem script must give a 32-byte id that decodes, by the §0 rule, to the exact APR string A14 assigned, together with the selector `batchMint`. `Queued(id, executableAt)` for a V1 change must give the countdown ADM-15 shows. A builder that stores the approval id as a string, hashes it, truncates it or emits it unindexed fails.
- **Operations.**
  - Re-running the deploy scripts twice changes no state.
  - Running them with a mainnet chain id under `RUN_MODE=local` is refused.
  - `40-verify.ts` fails when a test deliberately leaves the deployer as an Agent or as a ClaimIssuer management key.
  - `gateway.json` exists under `out/<chainId>/` and matches the chain.
  - The custom Solidity line count in the evidence file equals a fresh count by the node QA.

## Pass criteria

- The node QA's hidden oracles in `<workspace>\gauntlet\hidden\A25\` pass 100% on three consecutive runs, including:
  - a replayed approval id, a replayed mint nonce, a reuse across the two shapes and a revoked id are each refused;
  - an attestation for one calldata cannot execute another;
  - the mint total under one approval never exceeds the cap fixed at first use;
  - a maker among the signers reverts;
  - V3 never lifts a restriction;
  - a batch runs atomically, consumes its id once, and cannot be replayed through `execute` or `executeBatch`;
  - a forced transfer on a paused token, activation plus unpause, and a two-trust recovery each succeed under one approval id, and an override armed under another id is refused;
  - a mint approval cannot mint on a second token;
  - A26's smart-wallet recovery runs through V2, and V3 can revoke and pause investor agents' session keys but never restore them;
  - no multisig proposal counts an approval by its declared maker, calls a gateway-only selector (directly or inside `callModuleFunction`), or executes inside its delay or after a veto; and three non-maker owners can always act.
- The model-based suite runs at least 10,000 random operation sequences on Hardhat with a controllable clock and never observes any of these:
  - an `Executed` event without a valid attestation and quorum;
  - `mintedUnder[id] > capOf[id]`, or a mint chunk on a token other than `targetOf[id]`;
  - the same `(approvalId, nonce)` or `SINGLE` `approvalId` executed twice (through `execute`, `executeBatch` or both), or one id used as both shapes;
  - a batch that applied some inner calls and not others;
  - a `forcedTransferOverride` executed under an approval id different from the one in its calldata;
  - a multisig proposal queued with a counted approval from its declared maker, or any multisig call to a gateway-only selector;
  - an execution after `ApprovalRevoked`;
  - a signer counted twice;
  - a signer whose `principalId == makerId`;
  - a V3 call with a `false` restriction argument;
  - a multisig `execute` before `executableAt` or after a veto.

  Line and branch coverage of both contracts is 100% (`solidity-coverage`), and every custom error and event is hit by at least one test.
- A15's `attestation-vectors.json` is consumed unchanged. Every valid vector verifies through the gateway's `AttestationLib`, and every mutated vector (one field changed) is refused. The EIP-712 digest computed by an independent TypeScript implementation equals the contract's for all vectors.
- After the deploy scripts run on a fresh Hardhat chain, all of these hold:
  - both gateways are Agents on the deployed Tokens;
  - V3 is the `HaltModule` halter, and V2 is the `HaltModule` and `LockInModule` lifter;
  - V2 is A26's registry `guardian`, V3 its `revoker`, and the multisig owns the registry and the paymaster (after its first batch proposal accepts ownership);
  - every slot in the handover table has its listed holder;
  - the deployer holds no role or key anywhere;
  - no signer or owner maps to an `AGNT`, `AGT-U`, `SVCA` or token principal;
  - the on-chain allow-lists equal `gateway-config.json` by hash and A15's vault allow-lists by the cross-check (after R1 and R2);
  - addresses are reproducible on a second fresh chain with the same config;
  - `contracts/deploy/gateway/out/<chainId>/gateway.json` is written;
  - a second run reports no change.
- Custom Solidity totals at most 400 lines across both contracts and `AttestationLib`, excluding imports, comments and blank lines; the gateway is about 220 and the multisig about 180. The count is recorded in `app/evidence/A25.json`, alongside ABI hashes, gas figures for `execute` (mint, 100-line batchMint, pause) and the multisig path, the `solhint` result, and the static-analysis result or an explicit "not run".
- `npm run verify` passes repo-wide with only the owned paths changed. No file under `contracts/src/token`, `contracts/src/compliance` or `contracts/deploy/suite` differs from A24's gate commit. `app/evidence/A25.json` and `A25.qa.md` exist and match disk, with no open critical or high finding.

## Human gates touched

- **HG-AUDIT:** both contracts are custom code and must pass two independent audits before mainnet. This node produces the audit package: source, spec, invariants, test report, gas and static-analysis results, and the residual-risk list. The residual risks are:
  - the V2 forced-transfer and recovery timelock is enforced off-chain, and `revokeApproval` is the on-chain veto;
  - `deployTREXSuite` agent parameters are not decoded on-chain;
  - V4 owns the `IdFactory`;
  - the multisig cannot verify the declared `makerId`, so a proposer who lies about being the maker is caught off-chain (A15 P1, A27's maker binding) and by the timelock veto, not by the contract;
  - principal types (staff versus agent or service) are checked by `40-verify.ts` and A27, not by the contracts.

  The node passes on local evidence with the gate recorded as open.
- **HG-KEYS:** the V1 owner keys, the V2 and V3 signer keys and the attestation key exist here only as no-value sandbox keys from A15. The production ceremonies are human. Recorded as open.
- **HG-PROD:** mainnet deployment (Base or Redbelly) is human. The scripts refuse a mainnet chain id in local mode. Production addresses come from the production deploy's own `gateway.json` and A27 manifest; they are not predicted from local runs.
- **HG-VENDOR** (custody): in production a rented policy engine signs the gateway bundles behind A15's adapter. The gateway's checks stay the same, so no vendor is needed to pass this node.
- **HG-LEGAL** (counsel questions, not rules):
  - whether the T-REX interfaces this node compiles against carry GPL-3.0 obligations into these two contracts (the clean-room fallback keeps the gateway unchanged, since it calls interfaces only);
  - whether an on-chain timelock with owner veto, plus an off-chain V2 timelock with on-chain revocation, satisfies counsel's view of trustee control over the token;
  - whether any owner-level act must be personal rather than delegable.

  The build enforces "never an agent signs" regardless. Gateway signers and multisig owners are the staff principals of A15. No `AGNT`, `AGT-U`, token or service principal is ever seeded as one, and `40-verify.ts` plus a unit test prove it.

Every node also meets the standing pass criteria in [[App Gauntlet]] §4.

---
[[App Build Plan]] · [[App Development Graph]] · [[App Gauntlet]] · [[Screen Coverage Matrix]]
