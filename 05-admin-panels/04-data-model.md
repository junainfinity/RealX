# 04 — Core Data Model (Admin-Panel Back End)

> **EXTRAPOLATION.** This is the logical data model behind modules M01–M27. The `state` enumerations follow `03-workflows-and-state-machines.md`. The model is chain-agnostic: on-chain objects are **mirrored** and reconciled, never assumed. Date: 23 Sep 2026.

**Conventions.**
- Every table carries `tenant_id`, `jurisdiction_pack_version` (where rules apply), `created_at/by` and `updated_at/by`.
- Deletes are soft deletes plus an audit event.
- PII is encrypted at field level (`_enc`). **No PII goes on-chain.** The chain sees addresses, claim/credential references and document hashes only, following the DS Registry Service pattern of hashed PII [`APE` M4 C#13] and Redbelly/Averer ZK credentials [`australia/02-redbelly` §2.2].
- Diagrams use Mermaid `erDiagram` notation:
  - `||--o{` = one-to-zero-or-many;
  - `||--|{` = one-to-one-or-many;
  - `|o--o|` = zero-or-one on both sides;
  - `}o--o{` = many-to-many.

---

## 1. Overview (entities and relationships)

```mermaid
erDiagram
    Tenant ||--o{ Issuer : hosts
    Issuer ||--o{ SPV : sponsors
    SPV ||--o{ Licence : "holds or relies on"
    SPV ||--o{ Property : "holds title to"
    Property ||--o{ Valuation : "valued by"
    Property ||--o{ Lease : "let under"
    Property ||--o{ IncomePeriod : "earns"
    SPV ||--o{ Offering : "raises via"
    Offering ||--|{ Tranche : "split into"
    SPV ||--o{ TokenContract : "issuer of"
    TokenContract ||--|{ TokenClass : defines
    Tranche }o--|| TokenClass : issues
    RulePack ||--o{ ComplianceRule : parameterises
    TokenContract ||--o{ ComplianceRule : enforces
    Investor ||--o{ InvestorClassification : "classified as"
    Investor ||--o{ KYCCase : "verified by"
    Investor ||--o{ AccreditationRecord : evidences
    Investor ||--o{ TaxForm : files
    Investor ||--o{ Wallet : controls
    CustodyAccount ||--o{ Wallet : "holds sub-accounts"
    Investor ||--o{ Subscription : places
    Subscription }o--|| Tranche : for
    Subscription ||--o{ Payment : "funded by"
    Offering ||--o| EscrowAccount : uses
    Payment }o--o| EscrowAccount : "lands in"
    Subscription ||--o| Allocation : "results in"
    Allocation ||--o{ Holding : creates
    TokenClass ||--o{ Holding : "recorded in"
    Wallet ||--o{ Holding : "holds"
    TokenClass ||--o{ TransferRequest : "subject of"
    Investor ||--o{ Order : places
    Order ||--o{ Trade : "fills into"
    TokenClass ||--o{ Snapshot : "captured by"
    TokenClass ||--o{ ReconBreak : "breaks on"
    TokenClass ||--o{ Distribution : "pays on"
    Distribution ||--|{ PayoutLine : contains
    PayoutLine }o--|| Investor : "paid to"
    SPV ||--o{ Proposal : concerns
    Proposal ||--o{ Vote : receives
    Vote }o--|| Investor : "cast by"
    Document ||--o{ Signature : "signed via"
    Signature }o--|| Investor : "signed by"
    Investor ||--o{ AMLAlert : triggers
    AMLAlert }o--o| Case : "escalated to"
    Investor ||--o{ Complaint : raises
    AdminUser ||--o{ RoleAssignment : has
    Role ||--o{ RoleAssignment : "granted by"
    AdminUser ||--o{ ApprovalRequest : makes
    ApprovalRequest ||--o{ OnchainTx : authorises
    ApprovalRequest ||--o{ AuditEvent : "logged as"
    AdminUser ||--o{ AuditEvent : "acts in"
    Integration ||--o{ AuditEvent : emits
    Investor ||--o{ Notification : receives
```

---

## 2. Domain A — Tenancy, entities, assets and offerings

```mermaid
erDiagram
    Tenant ||--o{ Issuer : hosts
    Issuer ||--o{ SPV : sponsors
    SPV ||--o{ Licence : "relies on"
    SPV ||--o{ Property : "holds title to"
    Property ||--o{ Valuation : "valued by"
    Property ||--o{ Lease : "let under"
    Property ||--o{ IncomePeriod : earns
    SPV ||--o{ Offering : raises
    Offering ||--|{ Tranche : "split into"
    Offering ||--o{ Document : "discloses via"
    Tenant {
        uuid id PK
        string name
        string tenant_type "platform issuer operator distributor"
        string plan "tier and capacity limits"
        string jurisdictions_allowed
        int investor_cap
        string whitelist_mode "auto_on_kyc or admin_confirm"
        json guardrails "kyc_required locked on in prod"
        string status
    }
    Issuer {
        uuid id PK
        uuid tenant_id FK
        string legal_name
        string jurisdiction
        string reg_no
        string kyb_status
        string onchain_identity "ONCHAINID or DID"
        decimal retained_equity_pct "seller skin in the game"
        string status
    }
    SPV {
        uuid id PK
        uuid issuer_id FK
        string legal_name
        string vehicle_type "series_llc dao_llc difc_spv unit_trust sub_fund gmbh sl fund"
        string jurisdiction
        string reg_no
        string tax_class "c_corp reit partnership amit"
        string register_of_record "chain ta unit_register dld rer"
        string chain_is_register_clause "OA clause ref if any"
        string bank_account_ref
        string state
    }
    Licence {
        uuid id PK
        uuid entity_id FK
        string regulator
        string licence_no
        string activities
        string conditions
        date expires_on
        string feature_flags_bound
        string status
    }
    Property {
        uuid id PK
        uuid spv_id FK
        string address
        string country
        string zone_flags "freehold holy_city firb_residential"
        string property_type
        decimal area_sqm
        string registry_ref "county DLD RER Torrens"
        string title_status
        date title_verified_on
        decimal acquisition_price
        decimal lvr
        int compliance_score
        string lifecycle_state
    }
    Valuation {
        uuid id PK
        uuid property_id FK
        date valuation_date
        string method
        string valuer
        decimal value
        decimal nav_per_token
        string status "draft committee published"
        uuid approval_id FK
        string oracle_tx_hash
    }
    Lease {
        uuid id PK
        uuid property_id FK
        string unit_ref
        string tenant_ref_masked
        string lease_ref "Ejari Ejar"
        decimal rent
        date start_date
        date end_date
        decimal arrears
        string source_system "appfolio yardi buildium csv"
    }
    IncomePeriod {
        uuid id PK
        uuid property_id FK
        string period
        decimal gross_rent
        decimal opex
        decimal reserve_topup
        decimal platform_fees
        decimal smoothing_adj
        decimal net_distributable
        string state "draft confirmed locked"
    }
    Offering {
        uuid id PK
        uuid spv_id FK
        string regime "reg_d_506c reg_s reg_a_t2 reg_cf ecsp vara_arva difc_cf au_mis s708 rega"
        string jurisdictions
        decimal target_amount
        decimal min_ticket
        decimal max_ticket
        decimal soft_cap
        decimal hard_cap
        decimal per_investor_cap_pct
        datetime opens_at
        datetime closes_at
        string failure_mode "finalise_or_revert all_or_nothing"
        uuid rule_pack_id FK
        string state
    }
    Tranche {
        uuid id PK
        uuid offering_id FK
        uuid token_class_id FK
        string name "reg_d reg_s presale general"
        decimal unit_price
        decimal size
        string eligibility
    }
    Document {
        uuid id PK
        string owner_type
        uuid owner_id
        string doc_type "ppm form_c kiis pds tmd whitepaper oa resolution certificate"
        int version
        string sha256
        string uri "or IPFS CID"
        date effective_from
        string state
    }
```

---

## 3. Domain B — Investors, onboarding, wallets and custody

```mermaid
erDiagram
    Investor ||--o{ InvestorClassification : "classified as"
    Investor ||--o{ KYCCase : "verified by"
    Investor ||--o{ AccreditationRecord : evidences
    Investor ||--o{ TaxForm : files
    Investor ||--o{ Wallet : controls
    CustodyAccount ||--o{ Wallet : "holds sub-accounts"
    Investor {
        uuid id PK
        uuid tenant_id FK
        string investor_type "individual joint smsf trust company ira"
        string name_enc
        string residency
        string nationality
        string religion_attr_enc "only where law requires, e.g. KSA holy cities"
        string risk_rating
        bool pep
        date kyc_valid_until
        string onchain_identity "ONCHAINID or DID"
        string tier "presale superreentel pro"
        string state "active suspended restricted estate offboarded"
    }
    InvestorClassification {
        uuid id PK
        uuid investor_id FK
        string jurisdiction
        string investor_class "retail accredited professional qualified wholesale institutional"
        string basis
        date valid_until
        decimal limit_used_12m "Reg CF or DFSA annual caps"
    }
    KYCCase {
        uuid id PK
        uuid investor_id FK
        string provider "sumsub idwise equifax nafath uaepass entrust"
        string provider_ref
        string provider_decision
        int risk_score
        json screening_hits
        string sof_sow_status
        string state
        uuid decided_by FK
        datetime decided_at
    }
    AccreditationRecord {
        uuid id PK
        uuid investor_id FK
        string regime_test "us_506c eu_pro_optup au_wholesale vara_qualified dfsa_pro"
        string method "third_party_letter docs accountant_cert min_ticket_reps"
        string evidence_sha256
        date valid_from
        date valid_until
        string claim_ref "on-chain claim or credential id"
        string state
    }
    TaxForm {
        uuid id PK
        uuid investor_id FK
        string form_type "w9 w8ben w8bene crs tfn"
        string tin_enc
        string tax_residency
        decimal treaty_rate
        date valid_until
        string state
    }
    Wallet {
        uuid id PK
        uuid investor_id FK
        uuid custody_account_id FK
        string chain
        string address UK
        string wallet_type "passkey_aa external omnibus_sub vault pool treasury"
        string ownership_proof "signature satoshi_test none"
        int kyt_score
        bool frozen
        string registry_status "registered pending removed"
        string guardian_status
    }
    CustodyAccount {
        uuid id PK
        string custodian "fireblocks bitgo ripple_custody zodia tangany coinbase dfns"
        string purpose "owner agent pauser identity distribution treasury paymaster escrow"
        string chain
        string vault_ref
        string policy_id
        string quorum "e.g. 2-of-3"
        bool omnibus
    }
```

---

## 4. Domain C — Token, register, rules and market

```mermaid
erDiagram
    SPV ||--o{ TokenContract : "issuer of"
    TokenContract ||--|{ TokenClass : defines
    RulePack ||--o{ ComplianceRule : parameterises
    TokenContract ||--o{ ComplianceRule : enforces
    TokenClass ||--o{ Holding : "recorded in"
    Wallet ||--o{ Holding : holds
    TokenClass ||--o{ TransferRequest : "subject of"
    TokenClass ||--o{ Order : "traded as"
    Order ||--o{ Trade : "fills into"
    TokenClass ||--o{ Snapshot : "captured by"
    TokenClass ||--o{ ReconBreak : "breaks on"
    ApprovalRequest ||--o{ OnchainTx : authorises
    TokenContract {
        uuid id PK
        uuid spv_id FK
        string chain
        string address UK
        string standard "erc3643 erc7518 erc1400 asa xrpl_mpt hts stellar t22 custom"
        string owner_vault
        string identity_registry
        string compliance_contract
        string upgradeability "none uups erc1967"
        bool paused
        string state
    }
    TokenClass {
        uuid id PK
        uuid token_contract_id FK
        string partition "class series reg_d reg_s floor"
        decimal total_supply
        decimal supply_cap
        int decimals
        string rights "equity debt revenue_share title_fraction"
        string supply_rule "10000_per_sqm 100000_eq_100pct 10000_per_trust"
    }
    RulePack {
        uuid id PK
        string code "US-506c AE-DLD AU-MIS-retail"
        string jurisdiction
        int version
        date effective_from
        json parameters
        uuid approval_id FK
    }
    ComplianceRule {
        uuid id PK
        uuid token_contract_id FK
        uuid rule_pack_id FK
        string module "country max_holders max_pct lockup transfer_limit price_band"
        json params
        int version
        string params_sha256
        string onchain_module
        string state
    }
    Holding {
        uuid id PK
        uuid token_class_id FK
        uuid investor_id FK
        uuid wallet_id FK
        decimal balance
        decimal frozen_amount
        date lockup_until
        string legal_record_ref "TA entry DLD certificate unit register RER deed share"
        string state "active locked partially_frozen frozen"
        datetime as_of
    }
    TransferRequest {
        uuid id PK
        uuid token_class_id FK
        uuid from_investor_id FK
        uuid to_investor_id FK
        string request_type "recovery forced court estate divorce error consent conditional"
        decimal quantity
        string evidence_sha256
        datetime timelock_until
        uuid approval_id FK
        string state
    }
    Order {
        uuid id PK
        uuid investor_id FK
        uuid token_class_id FK
        string side
        decimal quantity
        decimal limit_price
        decimal reference_price
        string validation_result
        string state
    }
    Trade {
        uuid id PK
        uuid buy_order_id FK
        uuid sell_order_id FK
        decimal quantity
        decimal price
        decimal fees
        string settlement_state
        string tx_hash
        string registry_ack
    }
    Snapshot {
        uuid id PK
        uuid token_class_id FK
        string purpose "distribution vote tax audit"
        string method "point_in_time average_balance"
        int block_number
        string file_sha256
        string state
    }
    ReconBreak {
        uuid id PK
        uuid token_class_id FK
        string break_type "qty unknown_holder missing_registration pending_tx bridge_drift custodian"
        decimal delta_qty
        string chain_value
        string register_value
        string custodian_value
        uuid resolution_approval_id FK
        string state
    }
    OnchainTx {
        uuid id PK
        uuid approval_id FK
        string chain
        string tx_hash UK
        string function_called
        string state "prepared signed broadcast confirmed failed"
        int confirmations
    }
    ApprovalRequest {
        uuid id PK
        string action
        string payload_sha256
        string state
    }
```

---

## 5. Domain D — Subscriptions, money and distributions

```mermaid
erDiagram
    Investor ||--o{ Subscription : places
    Tranche ||--o{ Subscription : receives
    Subscription ||--o{ Payment : "funded by"
    Offering ||--o| EscrowAccount : uses
    EscrowAccount ||--o{ Payment : holds
    Subscription ||--o| Allocation : "results in"
    Allocation ||--o{ Holding : creates
    IncomePeriod ||--o{ Distribution : funds
    Snapshot ||--o{ Distribution : "sizes"
    Distribution ||--|{ PayoutLine : contains
    Investor ||--o{ PayoutLine : "paid via"
    Subscription {
        uuid id PK
        uuid investor_id FK
        uuid tranche_id FK
        decimal amount
        decimal quantity
        string channel "direct distributor adviser club"
        datetime reflection_ends_at
        string consent_ref "ECSP warning acknowledgement"
        string state
    }
    Payment {
        uuid id PK
        uuid subscription_id FK
        uuid escrow_account_id FK
        string direction "in out refund"
        string rail "ach sepa wire card payid lean mada usdc usdt aecoin"
        string asset
        decimal amount
        string reference
        int kyt_score
        string tx_hash
        string state
    }
    EscrowAccount {
        uuid id PK
        uuid offering_id FK
        string escrow_type "bank qualified_third_party smart_contract"
        string agent
        decimal balance
        string state
    }
    Allocation {
        uuid id PK
        uuid subscription_id FK
        decimal quantity
        decimal refund_amount
        string allocation_file_sha256
        uuid mint_tx_id FK
        string state
    }
    Distribution {
        uuid id PK
        uuid token_class_id FK
        uuid income_period_id FK
        uuid snapshot_id FK
        string cadence "daily weekly monthly quarterly bullet"
        decimal gross
        decimal withholding
        decimal net
        string payout_asset
        string rail "push claim wallet_credit bank"
        string claim_root
        uuid approval_id FK
        string state
    }
    PayoutLine {
        uuid id PK
        uuid distribution_id FK
        uuid investor_id FK
        uuid wallet_id FK
        decimal units
        decimal gross
        decimal wht_rate
        decimal net
        string exception_code "frozen sanctions missing_form dust invalid_address"
        string tx_hash
        string state
    }
    Holding {
        uuid id PK
        decimal balance
    }
    IncomePeriod {
        uuid id PK
        decimal net_distributable
    }
```

---

## 6. Domain E — Governance, documents, compliance cases and administration

```mermaid
erDiagram
    SPV ||--o{ Proposal : concerns
    Proposal ||--o{ Vote : receives
    Investor ||--o{ Vote : casts
    Document ||--o{ Signature : "signed via"
    Investor ||--o{ Signature : signs
    Investor ||--o{ AMLAlert : triggers
    AMLAlert }o--o| Case : "escalated to"
    Investor ||--o{ Complaint : raises
    AdminUser ||--o{ RoleAssignment : has
    Role ||--o{ RoleAssignment : "granted by"
    AdminUser ||--o{ ApprovalRequest : makes
    ApprovalRequest ||--o{ AuditEvent : "logged as"
    AdminUser ||--o{ AuditEvent : "acts in"
    Integration ||--o{ AuditEvent : emits
    Investor ||--o{ Notification : receives
    Proposal {
        uuid id PK
        uuid spv_id FK
        string proposal_type "sale extension pm_change capex rent wind_up protocol"
        string origin "holder pm auto_trigger statutory fiduciary"
        datetime discussion_ends
        datetime vote_closes
        decimal quorum_pct
        decimal threshold_pct
        bool default_to_manager "disallowed by default"
        uuid snapshot_id FK
        string result_sha256
        string state
    }
    Vote {
        uuid id PK
        uuid proposal_id FK
        uuid investor_id FK
        decimal weight
        string choice
        string channel
        datetime cast_at
    }
    Signature {
        uuid id PK
        uuid document_id FK
        uuid investor_id FK
        string provider "docusign dropbox_sign uaepass"
        string envelope_ref
        string signed_sha256
        datetime signed_at
    }
    AMLAlert {
        uuid id PK
        uuid investor_id FK
        string scenario
        int score
        decimal amount
        uuid case_id FK
        string state
    }
    Case {
        uuid id PK
        string case_type "aml surveillance legal"
        string subjects
        string stage
        datetime clock_due
        string filing_ref "SAR STR SMR goAML"
        string need_to_know_acl
        string state
    }
    Complaint {
        uuid id PK
        uuid investor_id FK
        string category
        datetime received_at
        datetime ack_due
        datetime final_response_due
        decimal redress
        string external_body "afca ombudsman difc_courts finra"
        string state
    }
    AdminUser {
        uuid id PK
        string user_type "staff external service agent"
        string email
        uuid delegator_id FK "required for AI agents"
        bool mfa_hardware
        string status
    }
    Role {
        uuid id PK
        string code UK
        json permissions "module x V C E A O X"
        bool can_approve
        bool can_sign
    }
    RoleAssignment {
        uuid id PK
        uuid admin_user_id FK
        uuid role_id FK
        string scope "tenant entity asset jurisdiction"
        datetime expires_at
    }
    ApprovalRequest {
        uuid id PK
        string action
        string object_ref
        uuid maker_id FK
        string required_checkers
        string payload_sha256
        datetime expires_at
        string custody_tx_ref
        string state
    }
    AuditEvent {
        uuid id PK
        datetime ts
        uuid actor_id FK
        uuid on_behalf_of_id FK
        string action
        string object_ref
        uuid approval_id FK
        string before_sha256
        string after_sha256
        string prev_event_sha256 "hash chain"
        string tx_hash
    }
    Notification {
        uuid id PK
        string recipient_type
        uuid recipient_id
        string template
        string channel
        string severity
        string state
    }
    Integration {
        uuid id PK
        string provider
        string category "kyc kyt custody payments registry venue oracle pm esign comms"
        string environment
        string credential_ref
        string health
        datetime last_sync
    }
```

---

## 7. Which entities mirror on-chain state

**System of record by archetype** (`00-overview.md` §6):
- **A1:** the chain.
- **A2/A4:** the TA or unit register.
- **A3:** the sovereign registry (DLD/RER).
- **A5:** the chain per tenant token.
- **A6:** the on-chain registry (CDSR) plus the trustee.

RealX default: an **off-chain golden record controlled by the registrar/TA**, reconciled to the chain. This is compatible with the SEC TA proposal's "exclusive control" requirement [`REG-US` §5] and with registry-as-truth markets.

| Entity | Off-chain record | On-chain mirror (default ERC-3643) | Mirrored fields | Reconciliation rule |
|---|---|---|---|---|
| TokenContract | Config, approved role set, rule version | Contract, owner, agents, identity registry, compliance, `paused` | Owner/agent addresses, paused flag, compliance address | **R1** on-chain role holders = approved role set (M09.5); paused flag = app state |
| TokenClass | Supply ledger (allocations − burns) | `totalSupply` (per chain; bridged supply summed) | total_supply, supply_cap | **R2** Σ chains `totalSupply` = register supply = Σ allocations − Σ burns; tolerance 0 |
| Holding | Golden record (register entry, lots, legal-record ref) | `balanceOf`, frozen amounts, `isFrozen` | balance, frozen_amount, state | **R3** per holder: chain balance = register balance = custodian sub-ledger (omnibus); **unknown holder = Sev1** |
| Wallet | Wallet ↔ investor map, KYT, proof | Identity Registry entry and country; token-level frozen flag | registry_status, country, frozen | **R4** IR members = {wallets where investor active ∧ KYC valid}; stale entries flagged |
| KYCCase / AccreditationRecord | Decision and validity | ONCHAINID claims / XRPL Credentials / VCs | claim_ref, validity | **R5** claim present ⇔ record `verified` ∧ `valid_until ≥ today` (daily sweep revokes expired) |
| ComplianceRule | Approved parameter set (+ hash) | Compliance-module parameters | params_sha256 | **R6** hash(on-chain params) = approved version hash; drift = Sev1 |
| Allocation / Subscription | Allocation file (hash) | Mint transactions | mint_tx_id | **R7** Σ minted per offering = locked allocation file; each mint references its file |
| Distribution / PayoutLine | Approved run file and hash | Transfers / claim root / claim-contract balance | tx_hash, claim_root | **R8** Σ paid on-chain = Σ approved net; claim root = Merkle root of file; unclaimed = contract balance |
| Payment (stablecoin) | Payment record | Stablecoin transfer | tx_hash, amount | **R9** each inbound transfer matched to one payment (or unmatched queue); KYT recorded |
| Payment (fiat) / EscrowAccount | Ledger | — (bank statements) | — | **R10** bank balance = Σ ledger (client money) daily; escrow = Σ commitments |
| Trade | Trade record | Settlement tx; registry ack (A3) | settlement_state, tx_hash, registry_ack | **R11** trade `registered` only when both chain tx and registry ack exist |
| Valuation | Published NAV | Oracle value (if used) | oracle_tx_hash | **R12** oracle value = published NAV ± tolerance; stale > 24h → halt / collateral 0 |
| Snapshot | File and method | Block number; optional checkpoint | file_sha256 | **R13** recompute from indexer at block N reproduces file hash |
| Document | Versioned doc | ERC-1643 `setDocument` / IPFS CID | sha256, uri | **R14** on-chain document hash = latest effective version |
| Proposal / Vote | Tally | Optional result hash | result_sha256 | **R15** anchored hash = certified result |
| Omnibus sub-ledger (A3) | Sub-accounts per investor | Omnibus address balance | — | **R16** Σ sub-accounts = omnibus on-chain balance (PRYPCO/Ctrl Alt pattern [`mena/01-prypco` §2.4]) |
| Registry (A3 DLD/RER, external TA) | Instruction log | — (registry API) | registry_ack | **R17** every ownership change has a registry ack; pending > SLA → halt secondary for that asset |
| AuditEvent | Hash-chained log | Optional daily anchor of chain head [L] | prev_event_sha256 | **R18** hash chain verifies end-to-end; anchor matches |

**Reconciliation operations.**
- **Frequency:**
  - nightly full run (M11.3);
  - targeted run after every privileged on-chain action;
  - intraday runs for payouts and registry-sync-heavy assets.
- **Tolerance:** zero for quantities. Cash tolerance applies only to FX/fee rounding, is documented per rail, and is never applied to client money.
- **Break handling:** break → owner → proposed fix → approval (`REGO` → `CCO` + `CFO`, + `X-TA`) → corrective action (register adjustment, or M09 corrective tx) → re-run.
- **Registry-truth markets (DLD/RER):** **no on-chain movement without a registry acknowledgement**. The registry has "absolute probative force" and "settlement = registry update, not token transfer" [`mena/04-saudi` §2.2].

---

## 8. Other data-design notes

> **QA note (Fable) — coverage gap.** The model covers the core lifecycle (entities, assets, offerings, investors, wallets, tokens, register, money, distributions, governance, cases, RBAC and audit) but does **not** yet define entities that `01-reference-admin-spec.md` relies on in these modules: M14 (`TradingWindow`, `Halt`, `SurveillanceAlert`, `VenueConfig`), M15 (`Pool`, `LendingMarket`, `OracleFeed`, `Position`, `Liquidation`, `RepoAgreement`), M16 (`FeeLedger`, `Reserve`/`GuaranteeFund`, `PaymentAssetAllowlist`), M18 (`Template`, `Envelope`, `RetentionHold`), M19 (`RegulatoryObligation`, `Filing`, `MetricDefinition`), M21 (`Announcement`, `Campaign`, `MarketingItem`, `Claim`), M22 (`WorkOrder`, `MunicipalObligation`, `TitleCheck`, `InsurancePolicy`, `PropertyManager`), M23 (`Ticket`), M24 (`Partner`, `Distributor`, `Reward`, `Commission`), M25 (`ApprovalPolicy`, `SoDRule`, `BreakGlassSession`, `AgentMandate`), M26 (`ApiKey`, `WebhookSubscription`, `Job`) and M27 (`FeatureFlag`, `FeeSchedule`, `JurisdictionBinding`, `Invoice`). Add these (with `tenant_id`, state enums from `03` and `approval_id` links where an action is privileged) before implementation; the reconciliation rules R1–R18 below are unaffected.

- **Multi-tenancy.** `tenant_id` is on every row, with row-level security. Platform staff get "view as tenant" with audit. Shared KYC across tenants (the Blocksquare "one account, many marketplaces" pattern [`eu/02-blocksquare` §2.5]) uses a platform-level `Investor` with tenant-scoped consent records.
- **Jurisdiction packs.** `RulePack` versions are immutable once effective. Objects store the pack version applied when an action was evaluated, which lets audits replay decisions.
- **Time-travel.** `Holding` is bi-temporal (valid time plus record time). This supports Tokeny-style historical look-back [`WLD` §2] and point-in-time register extracts.
- **GDPR vs immutability.**
  - PII stays off-chain.
  - Erasure pseudonymises the investor record and seals the address ↔ person link.
  - Retention holds (AML 5y EU; TA 6y US; VARA 8y; AU 7y [`XCC` #44]) override erasure until they expire.
- **Data residency.** Storage location is configurable per pack. Saudi national infrastructure runs "in-Kingdom" [`mena/04-saudi` §7]. Assetora reports no overseas recipients [`australia/05-assetora` §7]. Canvas processes data in five countries [`australia/04-canvas` §2.6].
- **Sensitive attributes.** Religion is stored **only** where a transfer-eligibility law requires it (KSA holy-cities rule [`REG-ME` §9]). It is encrypted, visible to `CCO`/`MLRO` only, and evaluated as an eligibility flag, never displayed.
