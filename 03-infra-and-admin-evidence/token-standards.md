# Token Standards for Tokenised Real Estate — Compliance Functions & Admin-Panel Mapping

**Scope:** EVM standards (ERC-20/721/1155, ERC-1400 family, ERC-1404, ERC-3643, ERC-4626/7540, ERC-7518, ERC-7943) and non-EVM token models (Algorand ASA, Solana Token-2022, Stellar, XRPL, Hedera HTS + Asset Tokenization Studio, Polymesh, Canton/Daml), plus chain-selection notes (Avalanche L1s, Base/Polygon/Gnosis).
**Currency check:** 23 Sep 2026. **Evidence labels:** [C#n] = confirmed by source n (see Sources); [I] = inferred; [S] = speculative.

---

## 0. Key takeaways (for RealX)

1. **ERC-3643 (T-REX) is the de-facto permissioned-token standard.** It is Final on eips.ethereum.org, bundles on-chain identity (ONCHAINID), modular compliance, freeze/partial-freeze, pause, forced transfer, wallet recovery and batch ops, and is backed by an association with 90+ members including DTCC; ~US$32B of assets are claimed on it, and an ISO standardisation initiative is underway [C#1][C#7][C#9]. Every function maps cleanly to an admin-panel button.
2. **ERC-7943 "uRWA" (created Jun 2025) is the 2025–26 newcomer.** It is a *minimal* interface (canSend/canReceive/canTransfer, getFrozenTokens/setFrozenTokens, forcedTransfer) that works across ERC-20/721/1155/6909; it is shown as Final on eips.ethereum.org and was reported "Final" by May 2026, with CMTA (CMTAT), Chainlink ACE, Brickken, DigiShares, Stobox, Propchain and others in its coalition [C#2][C#8]. It is an *interop surface*, not a full compliance stack — it complements (does not replace) ERC-3643 [I].
3. **ERC-1400 is stagnant but still in production** (Draft, never finalised) — mainly via Hedera's Asset Tokenization Studio (ATS), which now ships **dual ERC-1400 + ERC-3643** support (Nov 2025) [C#4][C#9][C#17]. Its partitions (1410), documents (1643) and controller ops (1644) are the richest "corporate-actions-native" design [C#4].
4. **ERC-7518 (DyCIST, Zoniqx)** is still in review; it is the only standard with native `payout`/`batchPayout`, time-locks and partitions (ERC-1155-based), and it explicitly cites floor-by-floor real-estate fractionalisation [C#3].
5. **Non-EVM chains now all have native issuer controls**: XRPL added Clawback (2024), DID (2024), Credentials (Sep 2025), Multi-Purpose Tokens (Oct 2025) and Permissioned Domains/DEX (Feb 2026) [C#12][C#13]; Solana Token-2022 has transfer hooks, permanent delegate, default-frozen accounts, confidential transfers, **pausable** and **scaled-UI-amount** extensions [C#10]; Hedera HTS has KYC/freeze/wipe/pause keys [C#15]; Stellar has auth-required/revocable/clawback flags [C#14]; Canton published CIP-56 (Oct 2025) with registry-controlled, multi-step transfers and DvP [C#19].
6. **Account abstraction landed at protocol level**: EIP-7702 shipped in Pectra (7 May 2025); ERC-4337 is production-grade (bundlers/paymasters) — this matters for gasless investor UX and for admin-panel "gas sponsorship" settings [C#7].
7. **Tokenised real estate is still small on-chain**: RWA.xyz showed ~US$226M *distributed* real-estate value (US$1.34B "represented"), 2,851 holders, 25 assets on the day of access; top platforms Groma, Reental, DigiShares, Securitize, MetaWealth; networks include Ethereum, Polygon, Base, Solana, Stellar, Hedera, MANTRA, Injective [C#21]. (Dashboard coverage is partial — e.g., it may not include every Gnosis/Algorand issuer [I].)

---

## 1. EVM standards

### 1.1 ERC-20 / ERC-721 / ERC-1155 (base standards)
- **What they give:** fungible balances (20), unique tokens (721), multi-token/semi-fungible IDs (1155). No identity, whitelist, freeze or forced transfer in the base spec [C#9].
- **How RWA issuers use them:** "Permissioned ERC-20" — an ERC-20 with issuer-added allowlist/pausable/mint-burn roles, compliance handled off-chain by a transfer agent. BlackRock BUIDL (Securitize) is cited as the largest example (~US$2.5B) [C#9]. ERC-721 is used for deed/SPV-share NFTs; ERC-1155 for per-property or per-tranche IDs [I].
- **Admin implications:** all compliance functions are custom (OpenZeppelin `Pausable`, `AccessControl`, `ERC20Burnable`, allowlist mappings) — must be audited per deployment [I].

### 1.2 ERC-1400 family (Security Token Standard) — **Draft, stale**
Umbrella proposal (GitHub issue #1411, Sep 2018; never finalised) [C#4]:

| Sub-standard | Function | Key methods / concepts |
|---|---|---|
| **ERC-1410** Partially fungible tokens | Partitions (tranches) within one holder's balance, e.g., primary-issuance vs secondary-purchased lots, lock-up classes | `balanceOfByPartition`, `transferByPartition`, operators per partition [C#4] |
| **ERC-1594** Core security token | Issue/redeem, pre-transfer checks with reason codes (EIP-1066, e.g., 0x51 success, 0x55 funds locked), off-chain data injection (`transferWithData`) | `issue`, `redeem`, `canTransfer`, `isIssuable` [C#4] |
| **ERC-1643** Document management | Attach/update/remove documents (hash + URI) to the token | `setDocument`, `getDocument`, `removeDocument` [C#4] |
| **ERC-1644** Controller operations | Forced transfer/redeem by issuer/regulator/transfer agent (fraud, court order) if `isControllable()` | `controllerTransfer`, `controllerRedeem` [C#4] |

- **Compliance functions:** partitions/lock-ups ✔, forced transfer ✔, documents ✔, transfer-rule checks ✔ (off-chain signed data), identity ✘ (left to implementer) [C#4][C#9].
- **Status 2026:** "effectively stagnant", originator Polymath moved to Polymesh L1; still best for partition-heavy structures and live in Hedera ATS [C#9][C#16].

### 1.3 ERC-1404 Simple Restricted Token — Draft (Jul 2018)
- ERC-20 + `detectTransferRestriction(from,to,value)` → code (0 = OK) and `messageForTransferRestriction(code)` → human-readable reason [C#5].
- Use: lock-ups, KYC/AML allowlists, securities restrictions; lightweight and widely copied by early STO platforms [C#5]. No standard freeze/forced-transfer/recovery [I].
- **Admin implication:** a "restriction code dictionary" page (codes ↔ messages) and allowlist/lock-up editor [I].

### 1.4 ERC-3643 (T-REX) — **Final** (created Jul 2021)
Authors from Tokeny; requires ERC-20 and ERC-173 [C#1]. Six contract components [C#1]:

1. **Token** (ERC-20 compatible, compliance-checked transfers)
2. **Identity Registry** (whitelist of verified investor identities + ISO-3166 country)
3. **Identity Registry Storage** (shareable across tokens — one KYC, many tokens)
4. **Trusted Issuers Registry** (which KYC providers/claim issuers are trusted)
5. **Claim Topics Registry** (which claims — e.g., KYC, accreditation — are required)
6. **Modular Compliance** (pluggable modules: investor caps, country restrictions, transfer limits, etc.)

**Agent/owner functions** [C#1]:
- Transfers: `forcedTransfer`, `batchForcedTransfer`, `batchTransfer`
- Supply: `mint`, `burn`, `batchMint`, `batchBurn`
- Recovery: `recoveryAddress(lostWallet, newWallet, investorOnchainID)`
- Freeze: `setAddressFrozen`, `freezePartialTokens`, `unfreezePartialTokens` (+ batch versions)
- Pause: `pause`, `unpause`
- Config: `setOnchainID`, `setIdentityRegistry`, `setCompliance`
- **Roles:** Owner (issuer; appoints agents per ERC-173) and Agent (operational; EOA or smart contract — e.g., a multisig for maker-checker) [C#1]. Agent management is specified in the EIP's `IAgentRole` interface: `addAgent(address)`, `removeAgent(address)`, `isAgent(address)` [C#1, QA re-check 23 Sep 2026].
- **Naming note (QA):** the EIP names the sixth component simply "Compliance". "Modular Compliance" and its `addModule` / `removeModule` / `callModuleFunction` calls are the T-REX reference implementation (`IModularCompliance.sol`, github.com/ERC-3643/ERC-3643, GPL-3.0), not text of the EIP itself.

**Ecosystem (2025–26):** ERC3643 Association 90+ members incl. DTCC; presented to US SEC (2025); Project Guardian pilots; ISO standardisation initiative; ~US$32B assets claimed [C#7][C#9]. Integrated into Hedera ATS (Nov 2025) [C#17].

### 1.5 ERC-4626 & ERC-7540 (vaults)
- **ERC-4626** tokenised vault: deposit/withdraw/convertToShares; production-ready and used for institutional yield wrappers [C#7].
- **ERC-7540** async vaults — **Final**; `requestDeposit`/`requestRedeem` → Pending → Claimable → Claimed; operators. Designed for RWA redemption queues; used by Centrifuge, Ondo, Backed [C#6][C#9].
- **Relevance to RealX:** a property-fund or "income vault" can wrap ERC-3643 units behind a 7540 subscription/redemption queue; the admin panel then needs a **"process pending requests / set NAV / fulfil redemptions"** screen [I].

### 1.6 ERC-7518 DyCIST (Zoniqx) — in review (created Sep 2023)
- ERC-1155-based; each `tokenId` = partition (share class, Reg D vs Reg S, floor of a building) [C#3].
- Functions: `restrictTransfer`/`removeRestriction`/`canTransfer`, `lockTokens`/`unlockToken` (time-based holding periods), `freezeAddress`, `forceTransfer` (recovery, AML), `payout`/`batchPayout` (dividends/interest), voucher-based off-chain-signed compliance, `wrapToken`/`unwrapToken`/`wrapTokenFromPartition` for interop [C#3].
- Real estate explicitly cited ("floor-by-floor fractional ownership") [C#3].

### 1.7 ERC-7943 uRWA — Universal RWA Interface (created 10 Jun 2025; shown **Final**)
- View: `canSend(addr)`, `canReceive(addr)`, `canTransfer(from,to,amount|tokenId)`, `getFrozenTokens(addr)`; state-changing: `forcedTransfer(from,to,amount|tokenId)`, `setFrozenTokens(account, amount|tokenId, status)`; events `ForcedTransfer`, `Frozen` [C#2].
- Variants for ERC-20, ERC-721, ERC-1155, ERC-6909; deliberately minimal and unopinionated (no identity model) [C#2].
- Adoption signals: CMTA integrated in CMTAT; Chainlink ACE demonstrated compatibility; coalition includes Brickken, DigiShares, Stobox, Propchain, RealEstate.Exchange, Zoth, Casper, Forte, Bit2Me [C#8]. Co-author Dario Lo Buglio also submitted written input to the SEC Crypto Task Force (Mar 2026 file name) [C#22 — search-result only].
- **Implication:** a RealX token that is ERC-3643 *and* exposes ERC-7943 views maximises DeFi/venue compatibility [I].

### 1.8 Account abstraction (wallet layer, not token)
- **EIP-7702** shipped with Pectra on 7 May 2025 (EOAs can delegate to smart-contract code → batching, sponsored gas); **ERC-4337** production-ready (bundlers, paymasters) [C#7].
- Admin implication: a "gas sponsorship / paymaster policy" setting and a "smart-wallet recovery" workflow distinct from ERC-3643 `recoveryAddress` [I].

---

## 2. Non-EVM token models

### 2.1 Algorand Standard Assets (ASA)
- Four mutable role addresses [C#11]:
  - **Manager** — reconfigures/destroys the asset and can re-assign other roles.
  - **Reserve** — informational holder of non-circulating supply (not protocol-enforced).
  - **Freeze** — freeze/unfreeze a specific account's holding (frozen accounts can't send or receive).
  - **Clawback** — move assets between any two opted-in accounts (compliance enforcement, lost-key recovery).
- **Default frozen + mandatory opt-in** = allowlist pattern (unfreeze only after KYC) [C#11].
- Setting a role to empty **permanently** removes it (progressive decentralisation; irreversible — admin panel must warn) [C#11].
- ARC-20 "Smart ASA" and ARC-71 "Non-transferable ASA" exist for richer rules [C#23 — search-result titles].
- Real-estate usage: Lofty (US) is a known Algorand issuer [I — to be confirmed by US platform agent].

### 2.2 Solana Token-2022 (Token Extensions)
Extensions relevant to regulated RWAs [C#10]:
| Extension | Compliance use |
|---|---|
| **Transfer Hook** | CPI to a custom program on every transfer → whitelist/KYC/holding-limit checks (ERC-3643-like rules engine) |
| **Permanent Delegate** | Issuer authority can transfer/burn from any account → forced transfer, court orders, wallet recovery |
| **Default Account State** | New token accounts start *frozen* → unfreeze after KYC (allowlist) |
| **Confidential Transfer** (+ confidential mint/burn) | Encrypted balances/amounts with auditor key → privacy with regulator view |
| **Metadata Pointer / Token Metadata** | On-mint metadata (property ID, docs URI) |
| **Pausable** | Pause mint/burn/transfer globally |
| **Scaled UI Amount / Interest-Bearing** | Display-level rebasing (splits, accrual) |
| **Non-Transferable** | Soulbound credentials (e.g., KYC badge) |
| **Memo Required, CPI Guard, Transfer Fees, Groups/Members, Close Mint, Immutable Owner** | Audit memo, safety, fees, series grouping |
- Some extensions are mutually incompatible (e.g., on the same mint) → choose at deploy time [C#10]. Freeze authority (core SPL) gives per-account freeze [I].

### 2.3 Stellar assets
- **AUTH_REQUIRED** — issuer must approve each trustline (allowlist); two levels: AUTHORIZED and AUTHORIZED_TO_MAINTAIN_LIABILITIES [C#14].
- **AUTH_REVOCABLE** — revoke/downgrade authorisation → freeze + cancel open orders [C#14].
- **AUTH_CLAWBACK_ENABLED** — issuer can claw back (requires revocable) [C#14].
- **AUTH_IMMUTABLE** — locks flags permanently [C#14].
- "Authorization sandwich" pattern: grant full auth around each approved tx then drop back — effectively per-transfer approval (cf. SEP-8 regulated-asset approval servers) [C#14][I for SEP-8].
- Supply lock: set issuer master weight to 0 [C#14].

### 2.4 XRP Ledger
- **Trust-line tokens** (legacy IOUs): RequireAuth, freeze (individual/global), no-freeze; **Clawback** amendment (Feb 2024) [C#13][I for freeze flags].
- **Multi-Purpose Tokens (MPTokensV1, ~Oct 2025)**: flags CanTransfer, **RequireAuth** (allowlist), CanTrade, CanEscrow, **CanLock** (freeze individual/global), **CanClawback**; up to 1,024-byte metadata with asset-class taxonomy incl. `rwa → real_estate`; max supply cap; transfer fee 0–50% (burned); the DynamicMPT amendment would allow later mutation of metadata/fees/flags [C#12][C#13] — **QA note (Fable):** DynamicMPT was *not yet enabled* on XRPL mainnet per the xrpscan amendments API on 23 Sep 2026, so treat MPT metadata as immutable at issuance for now.
- **DID** (Oct 2024), **Credentials** (Sep 2025), **Permissioned Domains** & **Permissioned DEX** (Feb 2026) — an order book only accessible to holders of specified credentials [C#13]. QA re-check (xrpscan amendments API, 23 Sep 2026): Clawback enabled 8 Feb 2024; DID 30 Oct 2024; Credentials 4 Sep 2025; MPTokensV1 1 Oct 2025; PermissionedDomains 4 Feb 2026; PermissionedDEX 18 Feb 2026; TokenEscrow 12 Feb 2026.
- Admin implication: "issue credential", "manage domain membership", "lock/clawback MPT" screens [I].

### 2.5 Hedera Token Service (HTS) & Asset Tokenization Studio (ATS)
- **HTS keys:** Admin, Supply (mint/burn), **Freeze**, **KYC** (grant/revoke per account), **Wipe** (remove tokens from an account), **Pause**, Fee-Schedule, Metadata; accounts must *associate* a token before holding it [C#15].
- **ATS (open source):** web UI + SDK + contracts; ERC-1400-based equity and bond types; since 12 Nov 2025 **dual ERC-1400 + ERC-3643** [C#16][C#17].
  - Layers: Core (access control, control lists, supply caps, pause, locks, snapshots) → Define (security type, corporate actions) → Customize (jurisdiction rules) [C#16].
  - Roles: Admin, Minter/Issuer, Controller (forced transfer), Pauser, KYC Manager (+ others) [C#16].
  - Actions: create/deploy, mint (capped), redeem, forced transfer, allow/block lists, pause, locks (vesting/escrow), snapshots, freeze/unfreeze, external management lists, documents, SSI verification; corporate actions: dividends, coupons, voting rights, balance adjustments, **Mass Payout** [C#16].
  - Custody integrations: Dfns, Fireblocks, AWS KMS; wallets MetaMask, HashPack, Blade [C#16].

### 2.6 Polymesh (purpose-built securities L1)
- Asset creation with type (Equity/Bond/Fund), divisibility, metadata, ISIN/CUSIP identifiers; `asset::issue`/`asset::redeem` by asset **agents** [C#18].
- Built-in transfer restrictions: max investor count, max ownership %, claim-based rules; compliance requirements + trusted claim issuers at chain level [C#18].
- **Checkpoints** (balance snapshots) → **corporate actions**: distributions and ballots/voting [C#18].
- Portfolios & custody, settlement via venues/instructions (DvP) [C#18][I for details].
- Polymesh is where Polymath's ERC-1400 lineage moved [C#9].

### 2.7 Canton Network / Daml
- Privacy-first, permissioned-participant network; **CIP-56** token standard (published 2 Oct 2025) = "institutional ERC-20": balances, transfers, history, plus **registry-controlled transfers** (asset admin decides allowed counterparties), **multi-step transfers** (receiver accept/reject), **preapprovals**, atomic **DvP** [C#19].
- Holdings/transfers shared need-to-know (sub-transaction privacy) [C#19].
- Users: Dfns (custody), CantonSwap, Bitsafe (CBTC), Tradeweb (treasury financing vs stablecoins) [C#19]. Canton positions for institutional RWA (report Dec 2025: >US$36B on-chain RWA excl. stablecoins, 1–3% cross-chain pricing gaps) [C#20].

### 2.8 Chain-selection notes
- **Avalanche L1s (formerly subnets):** Avalanche promotes "Evergreen" permissioned L1s for institutions [C#24 — search-result titles]; the Avalanche9000 upgrade (Dec 2024) cut L1 deployment cost [I].
- **Base / Polygon / Ethereum:** all appear in RWA.xyz's tokenised real-estate network list [C#21]. Polygon PoS is a long-standing low-fee home for property tokens (e.g., Reental, Brickken users) [I]; Base (Coinbase L2) gives native Coinbase Smart Wallet / USDC rails [I].
- **Gnosis Chain:** RealT runs its property tokens and YAM secondary market on Gnosis [I — confirm in US platform profile].

---

## 3. Compliance-function matrix

Legend: ✔ native in standard · ◐ possible via extension/module/partition · ✘ not provided (custom code/off-chain)

| Function | ERC-20 (permissioned) | ERC-1400 | ERC-1404 | ERC-3643 | ERC-7518 | ERC-7943 | Algorand ASA | Solana T-2022 | Stellar | XRPL MPT | Hedera HTS/ATS | Polymesh | Canton CIP-56 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Identity / whitelisting | ◐ custom | ◐ off-chain data | ◐ | ✔ ONCHAINID + registries | ◐ vouchers | ◐ `canSend/canReceive` | ✔ default-frozen + opt-in | ✔ default-frozen / hook | ✔ auth-required | ✔ RequireAuth + Credentials | ✔ KYC key / control lists | ✔ claims | ✔ registry-controlled |
| Transfer rules (country, caps, limits) | ✘ | ◐ reason codes | ✔ restriction codes | ✔ modular compliance | ✔ restrictions | ◐ `canTransfer` | ◐ ARC-20 | ✔ transfer hook | ◐ approval server | ◐ domains | ◐ ATS "customize" layer | ✔ count/% rules | ✔ Daml logic |
| Freeze (wallet) | ◐ | ◐ | ◐ | ✔ | ✔ | ✔ | ✔ | ✔ (freeze auth) | ✔ revoke | ✔ CanLock | ✔ freeze key | ◐ | ◐ |
| Partial freeze / lock | ✘ | ✔ partitions | ◐ | ✔ `freezePartialTokens` | ✔ `lockTokens` | ✔ `setFrozenTokens` | ✘ | ✘ | ✘ | ◐ escrow | ✔ locks | ◐ | ◐ |
| Pause (global) | ◐ Pausable | ◐ | ✘ | ✔ | ◐ | ✘ | ◐ global freeze | ✔ pausable | ◐ | ✔ global lock | ✔ pause key | ✔ freeze asset [I] | ◐ |
| Forced transfer | ◐ | ✔ 1644 | ✘ | ✔ | ✔ | ✔ | ✔ clawback | ✔ permanent delegate | ✔ clawback | ✔ clawback | ✔ controller / wipe | ✔ controller [I] | ✔ admin [I] |
| Lost-wallet recovery | ✘ | ◐ via 1644 | ✘ | ✔ `recoveryAddress` | ✔ via forceTransfer | ◐ via forcedTransfer | ◐ clawback | ◐ perm. delegate | ◐ clawback | ◐ clawback | ◐ controller | ◐ identity-bound keys [I] | ◐ |
| Partitions / classes | ✘ | ✔ 1410 | ✘ | ✘ (one token per class) | ✔ tokenId partitions | ✘ | ✘ | ◐ groups | ✘ | ✘ | ◐ | ◐ portfolios | ◐ |
| Lock-ups / vesting | ◐ | ✔ | ✔ | ✔ module / partial freeze | ✔ | ◐ | ✘ | ◐ hook | ✘ | ◐ escrow | ✔ locks | ◐ | ◐ |
| Corporate actions / dividends | ✘ | ◐ | ✘ | ✘ (off-chain / separate contract) | ✔ `payout` | ✘ | ✘ | ◐ interest-bearing/scaled UI | ✘ | ✘ | ✔ dividends/coupons/voting/Mass Payout | ✔ checkpoints + distributions + ballots | ◐ |
| Snapshots | ◐ ERC20Snapshot | ◐ | ✘ | ✘ [I] | ◐ | ✘ | ✘ | ✘ | ✘ | ✘ | ✔ | ✔ checkpoints | ◐ |
| Document management | ✘ | ✔ 1643 | ✘ | ◐ (ONCHAINID/claims; docs off-chain) | ✘ | ✘ | ◐ metadata URL | ✔ metadata | ◐ home domain | ✔ metadata URIs | ✔ documents | ✔ documents [I] | ◐ |
| Status 2026 | Final | Draft (stale) | Draft | **Final** | Review | **Final** | Live | Live | Live | Live (Oct 2025) | Live; dual std Nov 2025 | Live | Live (CIP-56 Oct 2025) |

Sources for the matrix: [C#1]–[C#19]; cells marked [I] are inferred from general protocol knowledge.

---

## 4. Standard function → admin-panel action (mapping table)

| Admin-panel action (page → button) | ERC-3643 | ERC-1400 | ERC-7943 | ERC-7518 | Algorand | Solana T-2022 | Stellar | XRPL | Hedera | Maker-checker recommended |
|---|---|---|---|---|---|---|---|---|---|---|
| Token Studio → **Deploy token / set compliance** | T-REX factory; `setIdentityRegistry`, `setCompliance`, `setOnchainID` | deploy + partitions | expose views | deploy 1155 | AssetConfig (roles, default-frozen) | init mint + extensions | issuer account + flags | `MPTokenIssuanceCreate` | ATS create / HTS keys | Yes (2 approvers) [I] |
| Investors → **Approve investor / add to whitelist** | register identity in Identity Registry (claims from trusted issuer) | off-chain signer data | `canSend/canReceive` true | voucher | unfreeze holding | thaw account / hook allowlist | `SetTrustLineFlags` authorize | MPT authorize / issue Credential | KYC grant / control list | Optional (KYC reviewer + compliance officer) |
| Compliance Rules → **Add rule (country ban, max holders, max % holding, lock-up)** | add compliance module | partition rules / signer | custom | `restrictTransfer`, `lockTokens` | ARC-20 app | transfer-hook config | approval server | permissioned domain | ATS customize layer | Yes |
| Primary issuance → **Mint / issue to investor** | `mint` / `batchMint` | `issue`/`issueByPartition` | n/a | mint | send from reserve | `mintTo` | payment from issuer | `Payment` from issuer | mint (supply key) | **Yes** |
| **Burn / redeem** | `burn` / `batchBurn` | `redeem`, `controllerRedeem` | n/a | burn | clawback to reserve | burn / perm. delegate | payment back / clawback | clawback/burn | redeem / wipe | **Yes** |
| Holder detail → **Freeze wallet** | `setAddressFrozen` | controller | `setFrozenTokens` | `freezeAddress` | AssetFreeze | freeze account | revoke auth | lock (CanLock) | freeze key | Yes |
| Holder detail → **Freeze partial amount / lock** | `freezePartialTokens` | partition move | `setFrozenTokens(amount)` | `lockTokens` | ✘ | ✘ | ✘ | escrow | ATS lock | Yes |
| Token → **Pause / unpause all transfers** | `pause`/`unpause` | custom | ✘ | custom | global freeze [I] | pausable ext. | ✘ | global lock | pause key | **Yes** (emergency: single approver + post-hoc review) [I] |
| Holder detail → **Forced transfer** (court order, estate, error) | `forcedTransfer` | `controllerTransfer` | `forcedTransfer` | `forceTransfer` | clawback | permanent delegate | clawback | clawback | controller / wipe | **Yes** (+ legal doc upload) |
| Holder detail → **Recover lost wallet** | `recoveryAddress` | `controllerTransfer` | `forcedTransfer` | `forceTransfer` | clawback + re-send | perm. delegate | clawback + re-issue | clawback + re-issue | controller | **Yes** (+ identity re-verification) |
| Documents → **Attach doc to token** | off-chain + claim/metadata | `setDocument` | ✘ | ✘ | asset URL/metadata hash | metadata ext. | stellar.toml | MPT metadata URI | ATS documents | Optional |
| Distributions → **Snapshot + pay dividend/rent** | off-chain snapshot + stablecoin batch transfer | partition-based | ✘ | `payout`/`batchPayout` | ✘ (off-chain) | scaled UI/interest-bearing or batch | ✘ | ✘ | ATS snapshot + dividends/Mass Payout | **Yes** |
| Governance → **Create ballot** | off-chain (Snapshot-style) | ✘ | ✘ | ✘ | ✘ | ✘ | ✘ | ✘ | ATS voting rights | Optional |
| Roles → **Add/remove agent** | `addAgent`/`removeAgent` (`IAgentRole`, called by the ERC-173 owner) [C#1] | operators/controllers | ✘ | roles | rekey role addresses | set authorities | signer weights | issuer keys | update keys / ATS roles | **Yes** |
| Roles → **Renounce power permanently** | renounce ownership [I] | `finalizeIssuance` / renounce control [I] | ✘ | ✘ | set role to empty (irreversible) | set authority None | AUTH_IMMUTABLE / master weight 0 | ✘ | remove keys | **Yes** + warning |

---

## Sources

| # | URL | Title | Accessed |
|---|---|---|---|
| 1 | https://eips.ethereum.org/EIPS/eip-3643 | ERC-3643: T-REX – Token for Regulated EXchanges | 23 Sep 2026 |
| 2 | https://eips.ethereum.org/EIPS/eip-7943 | ERC-7943: uRWA – Universal Real World Asset Interface | 23 Sep 2026 |
| 3 | https://eips.ethereum.org/EIPS/eip-7518 | ERC-7518: Dynamic Compliant Interop Security Token | 23 Sep 2026 |
| 4 | https://github.com/ethereum/EIPs/issues/1411 | ERC 1400: Security Token Standard (GitHub issue) | 23 Sep 2026 |
| 5 | https://github.com/ethereum/EIPs/issues/1404 | ERC-1404: Simple Restricted Token Standard (GitHub issue) | 23 Sep 2026 |
| 6 | https://eips.ethereum.org/EIPS/eip-7540 | ERC-7540: Asynchronous ERC-4626 Tokenized Vaults | 23 Sep 2026 |
| 7 | https://www.xcapit.com/en/blog/ethereum-new-standards-2026-enterprise | New Ethereum Standards in 2026: Enterprise Guide (Xcapit, 13 Feb 2026) | 23 Sep 2026 |
| 8 | https://en.cryptonomist.ch/2026/05/26/erc-7943-the-new-universal-standard-for-the-tokenization-of-real-world-assets-on-ethereum/ | ERC-7943: the new universal standard… (The Cryptonomist, 26 May 2026) | 23 Sep 2026 |
| 9 | https://protofire.io/guides/rwa-token-standards/ | ERC-3643 vs ERC-1400 vs ERC-20: RWA & Security Token Standards Compared (2026) (Protofire, Jul 2026) | 23 Sep 2026 |
| 10 | https://solana.com/docs/tokens/extensions | Solana Docs – Token Extensions | 23 Sep 2026 |
| 11 | https://dev.algorand.co/concepts/assets/overview/ | Algorand Standard Assets (ASAs) – Algorand Developer Portal | 23 Sep 2026 |
| 12 | https://xrpl.org/docs/concepts/tokens/fungible-tokens/multi-purpose-tokens | XRPL – Multi-Purpose Tokens | 23 Sep 2026 |
| 13 | https://thecryptobasic.com/2026/03/16/why-recent-xrpl-upgrades-like-mpts-and-permissioned-domains-make-xrp-ideal-for-rwa-tokenization/ | Why recent XRPL upgrades like MPTs and Permissioned Domains… (The Crypto Basic, 16 Mar 2026) | 23 Sep 2026 |
| 14 | https://developers.stellar.org/docs/tokens/control-asset-access | Stellar Docs – Control Access to an Asset | 23 Sep 2026 |
| 15 | https://docs.hedera.com/hedera/core-concepts/tokens/hedera-token-service-hts-native-tokenization | Hedera Token Service (HTS) – Native Tokenization | 23 Sep 2026 |
| 16 | https://docs.hedera.com/hedera/open-source-solutions/asset-tokenization-studio-ats | Asset Tokenization Studio (ATS) – Hedera Docs | 23 Sep 2026 |
| 17 | https://hedera.com/blog/hedera-integrates-erc-3643-token-standard-into-asset-tokenization-studio/ | Hedera integrates ERC-3643 into Asset Tokenization Studio (12 Nov 2025) | 23 Sep 2026 |
| 18 | https://developers.polymesh.network/core/assets/fungible/ | Fungible Assets – Polymesh Documentation Portal | 23 Sep 2026 |
| 19 | https://www.canton.network/blog/what-is-cip-56-a-guide-to-cantons-token-standard | What is CIP-56? A Guide to Canton's Token Standard (2 Oct 2025) | 23 Sep 2026 |
| 20 | https://www.canton.network/blog/state-of-rwa-tokenization-2026 | State of RWA Tokenization 2026 (18 Dec 2025) | 23 Sep 2026 |
| 21 | https://app.rwa.xyz/real-estate | RWA.xyz – Tokenized Real Estate (live dashboard snapshot) | 23 Sep 2026 |
| 22 | https://www.sec.gov/files/ctf-written-sec-crypto-task-force-buglio-030926.pdf | SEC Crypto Task Force written input – Buglio (search-result listing only; not fetched) | 23 Sep 2026 |
| 23 | https://dev.algorand.co/arc-standards/arc-0020/ ; https://dev.algorand.co/arc-standards/arc-0071/ | Smart ASA (ARC-20); Non-Transferable ASA (ARC-71) (search-result titles only) | 23 Sep 2026 |
| 24 | https://www.avax.network/about/blog/avalanche-launches-evergreen-subnets-for-institutional-blockchain-deployments | Avalanche Launches 'Evergreen' Subnets for Institutional Blockchain Deployments (search-result title only) | 23 Sep 2026 |
| 25 | https://www.erc3643.org/news/iso-standardization-initiative-for-the-t-rex-erc-3643-permissioned-tokens ; https://www.ftfnews.com/dtcc-joins-tokenization-standards-group/ | ERC3643 Association ISO initiative; DTCC joins tokenization standards group (search-result titles; corroborated by #9) | 23 Sep 2026 |

Blocked: https://developer.algorand.org/docs/get-details/asa/ returned HTTP 403 → replaced by source #11.
