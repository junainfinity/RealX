# 02 — RBAC, Approvals (Maker-Checker), Segregation of Duties, Break-Glass and Key Management

> **EXTRAPOLATION.** Built from:
> - the role sets observed in vendor stacks: Tokeny Identities/Assets write domains; DS Protocol Master/Issuer/Exchange/Transfer Agent; Hedera ATS Admin/Minter/Controller/Pauser/KYC Manager; ERC-3643 Owner/Agent; Vertalo Owner/Transfer Controller/Allowance Controller [`APE` M15; `WLD` §18.2];
> - custody-layer maker-checker: the Fireblocks Policy Engine and Taurus function-level governance [`WLD` §12–13];
> - the maker-checker and RBAC suggestions in all 20 platform §11s;
> - regulatory duties [`XCC`].
>
> Aliases, confidence scale and module IDs are in `00-overview.md`. Date: 23 Sep 2026.

## 0. Design principles

1. **Approval in the app, signing in custody, one ID.** Every privileged action is an `ApprovalRequest` in M25. The custody policy engine only signs transactions that carry an approved request ID. Vendors put the "checker" in custody [`APE` headline 2]; RealX puts it in both layers so neither can be bypassed.
2. **No self-approval; no money + configuration in one person.**
   - `SUPER` configures but never moves money or signs.
   - Signers never draft the action they sign.
   - Pattern evidence: PRYPCO "Super-admin (CTO): config, role management; no money movement" [`mena/01-prypco` §11.3]; Ctrl Alt "super-admin (no signing rights)" [`mena/02-ctrl-alt` §11.3]; BrickX "Super Admin: platform config only; cannot move money" [`australia/01-brickx` §11.3].
3. **On-chain roles mirror app roles.** App roles with signing power map 1:1 to on-chain roles:
   - identity agent → ERC-3643 Identity Registry agent;
   - token agent → Assets agent (mint/burn/freeze/forced transfer);
   - pauser;
   - owner → multisig.
   The reference is the Tokeny split between Identities and Assets write domains [`WLD` §2].
4. **External and machine principals are scope-limited.** Issuer tenants, property managers, distributors, transfer agents, custodians, auditors and regulators see only their own objects:
   - `X-AUD` and `X-REG` are read-only and time-boxed;
   - `AGNT` (AI agents) act **only as makers**, always on behalf of a named human, and **never as checkers**. This follows Vertalo "delegation on every write" and Juniper "same permissions, same audit trail" [`WLD` §8, §17].
5. **Emergency speed with accountability.** Only *protective* actions (pause, halt, freeze, suspend, inflow freeze) may be `SGL+R`: one authorised actor acts at once, then ratification is mandatory. *Restorative* actions (unpause, unfreeze, resume) always need four eyes.
6. **Thresholds are jurisdiction-pack parameters** (M27/M10), not hard-coded.
7. **Hardware-backed MFA** (FIDO2) for every approver and signer. 2FA is mandatory for all admin users (DigiShares [`WLD` §1]; Domoblock made it mandatory for users in Mar 2026 [`eu/05-domoblock` header "Status"]).
8. **Quarterly access recertification** and a 30-day dormancy lock on privileged accounts.

---

## 1. Admin role catalogue (41 roles)

> **QA note (Fable):** the original catalogue had 40 roles but referenced "CEO" as a checker in §3 rows 9, 13, 20, 29, 33, 52–53, 55 and in §6.1 without defining it. A `CEO` row is added below so that every approver named in the policy table is a defined principal.

Legend: **Approver** = can act as checker in M25 · **Signer** = holds a custody/MPC or multisig share · **Scope** = default data scope.

### 1.1 Internal roles

| Code | Role | Mandate (what they own) | Approver | Signer | Scope | Evidence (platform §11 role lists) |
|---|---|---|---|---|---|---|
| `SUPER` | Platform super-admin | Configuration, tenants, flags (maker only); **no money, no keys** | No | No | Platform | PRYPCO, BrickX, Ctrl Alt, Brickken §11.3 |
| `SECO` | Security officer / CISO | Users/roles/SoD, custody policies, keys, integrations, break-glass co-holder | Yes | Admin-quorum member | Platform | Lofty "Eng on-call", Redbelly CISO checker [`australia/02-redbelly` §11.2] |
| `CCO` | Chief compliance officer | Offerings approval, rules, freezes, filings, complaints oversight | Yes | Pauser; owner-quorum member | Platform | All §11.3 |
| `MLRO` | Money laundering reporting officer | Screening, EDD, cases, SAR/STR, restrictions, BWRA | Yes | Pauser (freeze only) | Platform | Reental, PRYPCO, BrickX |
| `KYCA` | KYC/onboarding analyst | KYC/KYB review (maker), L1 AML triage | Limited (standard KYC) | No | Assigned queues | Lofty, Reental, PRYPCO |
| `CLSR` | Classification/accreditation reviewer | Accreditation, wholesale, professional, qualified evidence | Limited | No | Assigned | RedSwan "KYC/accreditation analyst" |
| `LEGL` | Legal / company secretary | Entities, docs, title verification, forced-transfer maker, meetings, filings | Yes | No | Platform | BrickX co-sec, Propvest board, Arrived legal |
| `SHRO` | Shariah officer | Shariah screening of assets and offerings; fatwa repository | Yes (Shariah only) | No | Islamic products | Saudi operator RBAC [`mena/04-saudi` §11.4] |
| `INVM` | Investments / acquisitions | Pipeline, underwriting, IC submissions | No | No | Pipeline | Arrived acquisitions, mogul underwriting |
| `ASTM` | Asset manager | Asset data, income confirmation, exits, PM oversight | Limited (work orders) | No | Assigned assets | Reental, PRYPCO, Assetora |
| `VALO` | Valuation officer | Valuations, NAV, band reference, oracle inputs | No (committee approves) | No | Assets | Arrived valuation, BrickX investment manager |
| `CAPM` | Capital-markets / offering manager | Offering builder, raise ops, allocation (maker) | No | No | Offerings | Arrived capital markets, Propvest product |
| `PROP` | Property-operations manager | Rent roll, work orders, municipal/title compliance | Limited | No | Assigned assets | Lofty property ops; RealT municipal-compliance officer ⚑ |
| `TOKO` | Token-operations officer | Deploy, mint, burn, freeze execution (maker/initiator) | No | Initiator only (not a co-signer on own tx) | Tokens | Lofty, RedSwan issuance engineer, Ctrl Alt issuance officer |
| `REGO` | Registrar / TA officer (internal) | Golden record, recon, snapshots, lock-ups, register adjustments (maker) | Limited (conditional transfers) | No | Tokens | RealT registry, Tokeniser registry officer, Redbelly registry manager |
| `FINA` | Fund accountant / finance (maker) | Income ledger, distribution runs, refunds, redemptions | No | No | Assigned products | All §11.3 |
| `CFO` | Finance approver (checker) | Approves money-moving actions, valuations committee, admin quorum | Yes | Owner-quorum member | Platform | All §11.3 |
| `CEO` | Chief executive (board-level co-approver) | Second checker on large or exceptional actions only: mints outside an allocation file, distributions > $1M, withdrawals > $250k, timelock waivers, admin-quorum and break-glass membership, fee-schedule and feature-flag sign-off. **Never a maker**; no operational queues | Yes (co-approver only) | Owner-quorum and admin-quorum member | Platform | Added in QA; mirrors "management board" approvals at Propvest [`eu/03-propvest` §11.3] and the Ctrl Alt / PRYPCO pattern of keeping the top executive out of day-to-day money movement |
| `TRSY` | Treasury operations | Pay-ins/outs, escrow, stablecoin ops, paymaster | No | Initiator | Accounts | Lofty, Reental, PRYPCO |
| `MKTO` | Market ops / surveillance | Venue ops, listings, halts, windows, surveillance | Limited (conditional/DvP) | Pauser (market contracts) | Markets | Lofty, BrickX, Propvest |
| `RISK` | DeFi / liquidity risk manager | Pools, lending params, oracles, liquidations | No (risk committee approves) | Pool pauser | DeFi | RealT DeFi risk manager, Tokeniser risk |
| `TAXO` | Tax operations | Tax profiles, withholding rules, forms, filings | No | No | Platform | Lofty, RealT tax ops |
| `IRCM` | Investor relations / comms | Announcements, campaigns, reports, transfer-consent intake | No | No | Platform | Brickken issuer IR, mogul IR |
| `MKTG` | Marketing | Promotion drafts (maker) | No | No | Marketing | RedSwan (FINRA 2210 flow) |
| `SUPP` | Support / member services | Tickets, assisted actions, recovery intake; masked PII by default | No | No | Assigned | All §11.3 |
| `CMPH` | Complaints handler | Complaints register, investigations, final responses (maker) | No | No | Complaints | PRYPCO complaints, Propvest ADR |
| `PART` | Partner / distribution manager | Sponsors, distributors, advisers, clubs, rewards (maker) | No | No | Partners | Reental partner manager, Blocksquare protocol ops |
| `ENGO` | Engineering / blockchain ops | Deployments (sandbox), upgrades (maker), connectors, RPC; **no business approvals** | No | Deployer (sandbox); not prod signer | Tech | Lofty "Eng on-call", Reental blockchain ops |

### 1.2 External, limited and machine principals

| Code | Role | Mandate | Approver | Signer | Scope | Evidence |
|---|---|---|---|---|---|---|
| `X-TA` | External transfer agent / registrar | Register certification; checker on forced transfers/recovery; lock-up administration | Yes (register) | Optional TA agent key | Assigned issuers | SEC TA proposal: TA "exclusive control" of DLT master file [`REG-US` §5]; Securitize/Vertalo TA [`VND` §7]; RedSwan registrar [`us/02-redswan` §2.4] |
| `X-BDP` | Broker-dealer registered principal | FINRA supervision: comms (2210), suitability/Reg BI, allocations, ATS windows | Yes | No | BD activity | RedSwan BD console [`us/02-redswan` §11 #4]; Arrived NCPS [`us/03-arrived` §11] |
| `X-CUS` | Custodian signer | Co-signs per custody policy; key ceremonies | Custody only | Yes | Vaults | Fireblocks/Zodia/Ripple Custody [`VND` §4]; Ctrl Alt custody signer [`mena/02-ctrl-alt` §11.3] |
| `X-AUD` | External auditor | Read-only everything plus exports within engagement | No | No | Engagement, time-boxed | All §11.3 "auditor read-only" |
| `X-REG` | Regulator observer | Read-only scoped data room; selective disclosure | No | No | Regulator grant | Canvas regulator observer [`australia/04-canvas` §11 #10]; SEC TA proposal regulator access [`XCC` #45] |
| `X-PM` | Property manager | Own properties: rent roll, work orders, notices, update drafts | No | No | Own assets | Lofty PM, mogul PM, RealT PMs |
| `X-ISA` | Issuer / sponsor tenant admin | Own entities/assets/offerings/tokens; tenant sign-off | Tenant-level | Tenant's own custody (A5) | Own tenant | Brickken issuer owner, Blocksquare CP admin, DigiShares corporate admin |
| `X-ISO` | Issuer tenant operator | Maker within tenant | No | No | Own tenant | Brickken issuer operator/compliance |
| `X-DST` | Distributor / adviser / club leader | Own clients' orders and onboarding submissions; own commissions | No | No | Own clients | Ctrl Alt distributors, Assetora advisers, mogul Chief Mogul |
| `X-RE` | Responsible entity / trustee / SPV director | Statutory sign-offs: offerings (PDS), results, register certification, exits | Yes | Optional owner-quorum member | Own schemes/SPVs | BrickX RE board delegate; Tokeniser trustee (IQ-EQ) sign-off |
| `X-VAL` | Independent valuer | Upload valuations only | No | No | Assigned assets | BrickX/Stake independent valuers |
| `AGNT` | AI agent principal | Drafts, triage, calculations, maker-level submissions **on behalf of a named user** | **Never** | **Never** | Delegator's scope ∩ agent mandate | Vertalo, Juniper, Brickken ERC-8226 mandates [`WLD` §8, §17; `eu/04-brickken` §11 A] |
| `SVCA` | Integration service account | Automated jobs: eligibility grants, recon, snapshots, screening, syncs, rule-based payouts | No | Policy-limited hot key (e.g., identity agent) | Job-specific | Brickken prepare/sign/send; Ctrl Alt DLD service account |

---

## 2. Role × module permission matrix

**Legend:** `V` view · `C` create/draft · `E` edit · `A` approve (checker) · `O` execute on-chain (initiate or sign within custody policy) · `X` export · `–` none.

- Every role can **view its own approvals inbox** (M25.5) and its own notifications.
- PII is masked unless the role has `E` or `A` on M06/M07.
- `A` on a module does not mean every action in it: the specific action → checker mapping is in §3.

### 2.1 Internal roles — governance, compliance and front office

| Module | `SUPER` | `SECO` | `CCO` | `MLRO` | `KYCA` | `CLSR` | `LEGL` | `SHRO` | `INVM` | `ASTM` | `VALO` | `CAPM` | `PROP` |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| M01 Command Center | V | V | VX | V | V | V | V | – | V | V | V | V | V |
| M02 Issuers/SPVs/Entities | VE | V | VEA | V | V | – | VCEX | V | VC | V | – | V | V |
| M03 Properties & Assets | V | – | VA | V | – | – | VE | VA | VCE | VCE | VCE | V | VE |
| M04 Offerings | V | – | VA | V | V | V | VEA | VA | V | V | V | VCEOX | – |
| M05 Orders/Allocations/Redemptions | – | – | VA | V | VEA | V | V | – | – | V | V | VEX | – |
| M06 Investors CRM | – | – | VEAX | VEA | VEA | VE | VE | – | – | V | – | V | – |
| M07 Onboarding Review | – | – | VAX | VEA | VCEA | VEA | V | – | – | – | – | V | – |
| M08 Wallets & Custody | – | VCEA | VA | VEA | VA | – | – | – | – | – | – | – | – |
| M09 Token Operations | – | VA | VA | VO | V | – | VCEA | – | – | – | – | – | – |
| M10 Compliance Rules | – | VA | VCEA | V | V | V | VEA | – | – | – | – | – | – |
| M11 Register & Recon | – | – | VAX | – | – | – | V | – | – | V | – | – | – |
| M12 Distributions | – | – | V | – | – | – | VA | – | – | VE | – | – | V |
| M13 Tax | – | – | VA | – | – | – | – | – | – | – | – | – | – |
| M14 Secondary Market | – | – | VEA | V | – | – | VA | – | – | – | VE | – | – |
| M15 Lending/DeFi Risk | – | V | VA | – | – | – | – | – | – | – | VE | – | – |
| M16 Treasury & Client Money | – | V | VEA | V | – | – | – | – | – | VC | – | – | – |
| M17 Governance | – | – | VA | – | – | – | VCEA | – | – | VCE | – | – | – |
| M18 Documents & E-sign | – | – | VA | – | – | – | VCEAX | V | – | – | – | VC | – |
| M19 Reporting & Filings | – | VA | VCEAX | V | – | – | VCEA | – | – | – | – | – | – |
| M20 AML Cases | – | – | VA | VCEAX | VCE | – | V | – | – | – | – | – | – |
| M21 Comms & IR | – | VC | VCA | – | – | – | VA | – | – | VC | – | – | – |
| M22 Property Ops | – | – | VA | – | – | – | VE | – | V | VCEA | V | – | VCEX |
| M23 Support & Complaints | – | – | VAX | V | – | – | V | – | – | – | – | – | – |
| M24 Partners | – | – | VEA | – | – | – | VA | – | – | – | – | – | – |
| M25 Access/Approvals/Audit | VCE | VCEAX | VA | – | – | – | – | – | – | – | – | – | – |
| M26 Integrations/API | V | VA | V | – | – | – | – | – | – | – | – | – | – |
| M27 Settings/Tenancy | VCE | VA | VEA | – | – | – | VA | – | – | – | – | – | – |

### 2.2 Internal roles — operations, finance, markets and engagement

| Module | `TOKO` | `REGO` | `FINA` | `CFO` | `TRSY` | `MKTO` | `RISK` | `TAXO` | `IRCM` | `MKTG` | `SUPP` | `CMPH` | `PART` | `ENGO` |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| M01 Command Center | V | V | V | VX | V | V | V | V | V | V | V | V | V | V |
| M02 Issuers/SPVs/Entities | VO | V | V | VA | VE | – | – | V | – | – | – | – | V | – |
| M03 Properties & Assets | V | V | V | VA | – | V | V | V | V | V | V | V | V | – |
| M04 Offerings | VO | V | V | VA | V | V | – | V | V | V | V | V | V | – |
| M05 Orders/Allocations/Redemptions | VO | V | VEOX | VA | VE | V | – | V | V | – | VC | V | V | – |
| M06 Investors CRM | V | V | V | V | V | V | V | VE | VE | V | VCE | V | V | – |
| M07 Onboarding Review | V | V | – | – | – | – | – | – | – | – | V | – | – | – |
| M08 Wallets & Custody | VEO | V | V | VA | VEO | V | V | – | – | – | VC | – | – | VE |
| M09 Token Operations | VCEO | VEO | V | VA | – | VO | – | – | VC | – | VC | – | – | VCO |
| M10 Compliance Rules | VEO | VE | – | – | – | V | – | – | – | – | – | – | – | – |
| M11 Register & Recon | V | VCEX | VE | VA | – | V | – | V | – | – | V | – | – | – |
| M12 Distributions | VO | V | VCEX | VAX | VO | – | – | V | V | – | V | – | – | – |
| M13 Tax | – | – | V | VA | VE | – | – | VCEX | – | – | V | – | – | – |
| M14 Secondary Market | VO | – | – | VA | – | VCEOX | V | – | V | – | V | – | – | – |
| M15 Lending/DeFi Risk | VO | – | – | VA | – | VCE | VCEO | – | – | – | – | – | – | – |
| M16 Treasury & Client Money | – | – | VEO | VAX | VCEO | – | – | V | – | – | V | – | – | – |
| M17 Governance | – | V | – | V | – | VO | – | – | VE | – | V | – | – | – |
| M18 Documents & E-sign | VO | – | – | – | – | – | – | – | VC | – | V | – | – | – |
| M19 Reporting & Filings | – | – | VC | VCAX | – | – | – | VC | VCX | – | – | – | – | – |
| M20 AML Cases | – | – | – | – | VE | VC | – | – | – | – | – | – | – | – |
| M21 Comms & IR | – | – | – | – | – | – | – | – | VCEX | VCE | VC | V | V | – |
| M22 Property Ops | – | – | V | VA | V | – | – | – | V | – | V | – | – | – |
| M23 Support & Complaints | – | – | – | VA | – | – | – | – | V | – | VCE | VCEX | – | – |
| M24 Partners | – | – | V | VA | – | – | – | – | V | V | – | – | VCEX | – |
| M25 Access/Approvals/Audit | – | – | – | A | – | – | – | – | – | – | – | – | – | – |
| M26 Integrations/API | – | – | – | – | – | – | – | – | – | – | – | – | VC | VCE |
| M27 Settings/Tenancy | – | – | VC | VEA | – | – | – | – | – | – | – | – | VC | – |

### 2.3 External, limited and machine principals (always scope-restricted: own tenant / own assets / own clients)

| Module | `X-TA` | `X-BDP` | `X-CUS` | `X-AUD` | `X-REG` | `X-PM` | `X-ISA` | `X-ISO` | `X-DST` | `X-RE` | `X-VAL` | `AGNT` | `SVCA` |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| M01 Command Center | – | V | – | – | – | – | V | V | V | V | – | V | – |
| M02 Issuers/SPVs/Entities | V | V | – | VX | V | – | VCE | VC | – | VA | – | V | – |
| M03 Properties & Assets | – | V | – | VX | V | V | VCE | VC | V | VA | VC | VC | – |
| M04 Offerings | V | VA | – | VX | V | – | VCEA | VCE | V | VA | – | VC | – |
| M05 Orders/Allocations/Redemptions | V | VA | O | VX | V | – | V | V | VC | V | – | VC | O |
| M06 Investors CRM | V | V | – | V | V | – | V | V | V | V | – | V | – |
| M07 Onboarding Review | V | VA | – | V | V | – | VEA | VE | V | – | – | VC | O |
| M08 Wallets & Custody | V | – | VAO | V | V | – | V | V | – | – | – | – | O |
| M09 Token Operations | VAO | – | AO | VX | V | – | VC | VC | – | VA | – | VC | O |
| M10 Compliance Rules | VE | V | – | V | V | – | V | – | – | V | – | V | – |
| M11 Register & Recon | VEAX | – | – | VX | VX | – | VX | – | – | VA | – | V | O |
| M12 Distributions | V | – | O | VX | V | – | VC | VC | – | VA | – | VC | O |
| M13 Tax | – | – | – | VX | V | – | V | – | – | – | – | V | – |
| M14 Secondary Market | V | VA | – | VX | VX | – | V | – | V | – | – | V | O |
| M15 Lending/DeFi Risk | – | – | O | V | V | – | – | – | – | – | – | V | – |
| M16 Treasury & Client Money | – | – | VAO | VX | V | – | V | – | – | V | – | V | O |
| M17 Governance | – | – | – | V | V | VC | VC | VC | – | VA | – | VC | – |
| M18 Documents & E-sign | V | V | – | VX | VX | – | VCE | VC | – | V | – | VC | – |
| M19 Reporting & Filings | – | V | – | VX | VX | – | VX | – | – | VA | – | VC | – |
| M20 AML Cases | – | V | – | V | – | – | – | – | – | – | – | – | O |
| M21 Comms & IR | – | VA | – | V | V | VC | VC | VC | V | – | – | VC | – |
| M22 Property Ops | – | – | – | V | V | VCE | VCE | VC | – | V | – | VC | O |
| M23 Support & Complaints | – | – | – | V | V | – | V | – | V | – | – | VC | – |
| M24 Partners | – | – | – | V | V | – | V | – | V | – | – | V | – |
| M25 Access/Approvals/Audit | – | – | – | VX | VX | – | – | – | – | – | – | – | – |
| M26 Integrations/API | – | – | – | V | – | – | – | – | – | – | – | – | – |
| M27 Settings/Tenancy | – | – | – | V | V | – | VE | – | – | – | – | – | – |

**Reading notes.**
- QA note (Fable): `CCO` was given `E` on M14/M16/M24/M27 and `C` on M21 so that the actions where `CCO` is the *maker* (change venue mode, payment-asset allow-list, suspend partner, tenant guardrails and suspension, security broadcast) are permitted by the matrix; the four-eyes checker on those actions remains `LEGL`/`CFO`/CEO per §3.
- `CFO` has `A` on M25 only as a member of the **admin quorum** for approval-policy and custody-policy changes.
- `X-CUS` has `O` wherever a custody co-signature is part of the policy (M05 delivery, M08, M09, M12 payouts, M15, M16).
- `SVCA` has `O` only for policy-limited automated calls (e.g., identity registration after an approved KYC case; distribution pushes that match an approved file hash).

---

## 3. Maker-checker policy (defaults; parameterised per jurisdiction pack in M27)

**Notation:**
- `n-of-m` = custody quorum.
- **TTL** = approval expiry.
- **TL** = on-chain or operational timelock before execution.
- Monetary thresholds are in USD equivalents; the admin can localise them per pack (AED, EUR, AUD, SAR).

| # | Action (module) | Maker | Checker(s) | Threshold / condition | Custody quorum / multisig | TTL · TL | Notes & evidence |
|---|---|---|---|---|---|---|---|
| 1 | Approve standard KYC (M07) | `KYCA` | — (QA sample 5–10% by `MLRO`) | Provider-approved, no hits, low risk | — | — | Blocksquare auto-whitelist mode [`APE` M4] |
| 2 | KYC override / high-risk approval (M07) | `KYCA` | `MLRO` | Provider reject/refer, or high risk | — | 72h | Lofty/Reental "override KYC → MLRO" |
| 3 | PEP / EDD onboarding (M07) | `KYCA` | `MLRO` + senior manager | Always | — | 7d | AML regimes [`XCC` #6–8] |
| 4 | Clear sanctions false positive (M07/M20) | `KYCA` | `MLRO` | Always | — | 24h | [`XCC` #20] |
| 5 | Accreditation via minimum-investment path or professional opt-up (M07) | `CLSR` | `CCO` | $200k/$1M NAL path; EU 2-of-3 opt-up | — | 72h | [`REG-US` §3; `REG-EU` §3] |
| 6 | Grant on-chain eligibility (M07) | `SVCA` | Auto (after #1–#5) | Rate limit N/min; only IR functions | **Identity-agent hot key**, function allow-list | — | Tokeny Identities domain [`WLD` §2] |
| 7 | Whitelist batch pushing holders > 80% of cap (M08) | `KYCA` | `CCO` | Holder count after batch ≥ 80% of 12(g) / 3(c)(1) / scheme cap | As #6 | 72h | RealT 2,000 cap [`us/04-realt` §11.2] |
| 8 | Publish valuation / NAV (M03) | `VALO` | Valuation committee: `CFO` + independent member | Always; independent valuer report attached | Oracle push via policy (if on-chain) | 7d | Arrived NAV committee; RealT oracle conflict ⚑ |
| 9 | IC approval of acquisition (M03) | `INVM` | 2 of {`CFO`, `CCO`, CEO} | Always | — | 14d | Domoblock/mogul investment committee |
| 10 | Approve offering for launch (M04) | `CAPM` | `CCO` + `LEGL` (+`X-BDP` US; +`X-RE` AU; +`SHRO` Islamic; +platform `CCO` for tenant offerings) | Always | — | 14d | Every §11 "publish listing" 4-eyes |
| 11 | Go live (deploy sale/escrow) (M04) | `CAPM` | `CCO` | Always | Token-agent vault **2-of-3** | 72h | Brickken `newSto` [`eu/04-brickken` §4] |
| 11a | Deploy token contract to production (M09) | `TOKO` | `CCO` + `SECO` | Sandbox deploy, test evidence and audit report attached; identity registry, compliance and agent set reviewed | **Owner vault 3-of-5** | 14d | Added in QA to match M09 "Deploy token" (`6E+Q`) and the token lifecycle `deploy_requested → deployed : owner quorum 3-of-5` (`03` §6); Tokeny wizard [`WLD` §2]; Taurus deploy-time governance [`WLD` §13] |
| 12 | Finalise & mint allocation (M04/M05/M09) | `CAPM` / `TOKO` | `CFO` + `CCO` | Allocation file locked; escrow = allocation | Token-agent vault 2-of-3 (`TOKO` initiator, `CFO` delegate, `X-CUS`/`X-TA`) | 72h | Blocksquare finalise vs revert [`APE` M3] |
| 13 | Mint outside a locked allocation file (M09) | `TOKO` | `CFO` + `CCO` + CEO | Any | Owner vault **3-of-5** | 24h · TL 24h | Exceptional; alert Sev2 |
| 14 | Burn / redeem (M09) | `TOKO` | `CFO` | Matches redemption or exit file | Token-agent 2-of-3 | 72h | [`TSD` §4] |
| 15 | Freeze wallet: sanctions / true match (M09) | `MLRO` | Ratify by `CCO` ≤ 2h | Sanctions/court directive | **Pauser/freezer key 1-of-N** | — | Protective; `SGL+R` |
| 16 | Freeze wallet / partial freeze: other (M09) | `CCO` | `LEGL` | Always | Token-agent 2-of-3 | 24h | Lofty "freeze/clawback → compliance lead + eng" |
| 17 | Unfreeze (M09) | `MLRO` | `CCO` | Always | Token-agent 2-of-3 | 24h | Restorative ⇒ 4E |
| 18 | Pause token (M09) | `CCO` / `SECO` / on-call pauser | Ratify ≤ 2h by the other of `CCO`/`SECO` | Incident | Pauser 1-of-N | — | Reental "emergency single-signer then ratified" |
| 19 | Unpause token (M09) | `CCO` | `SECO` + `CFO` | Always; post-incident review attached | Token-agent 2-of-3 | 24h | Restorative |
| 20 | Forced transfer: court order / estate / divorce / error (M09) | `LEGL` | `CCO` + `CFO` (+`X-TA` if appointed) | Always; evidence bundle mandatory | Token-agent **2-of-3 incl. `X-TA` or `X-CUS`** | 7d · **TL 48h** (court-ordered immediacy may waive, with `CCO` + CEO) | Tokeny/DS `seize`/ATS Controller [`APE` M6] |
| 21 | Lost-wallet recovery (M09) | `SUPP` intake → `KYCA` re-verification | `CCO` | Always; new wallet must be verified | Token-agent 2-of-3 | 7d · **TL 48h** plus notice to old channels | ERC-3643 `recoveryAddress`; Securitize "destroy & reissue" |
| 22 | Add/remove agent or role; change compliance/IR pointers (M09) | `SECO` / `TOKO` | Owner quorum | Always | **Owner multisig 3-of-5** | 7d · **TL 48h** | Fireblocks admin quorum [`WLD` §12] |
| 23 | Contract upgrade (M09) | `ENGO` | `SECO` + `CCO` | Audit report attached | Owner 3-of-5 | 14d · **TL 72h** | UUPS/1967 risk [`eu/04-brickken` §10] |
| 24 | Rule-pack publish / token rule change (M10) | `CCO` | `LEGL` (+`SECO` for trusted issuers) | Simulation attached | Token-agent 2-of-3 (module changes) | 7d | Taurus function-level governance [`WLD` §13] |
| 25 | Investor-specific eligibility override (M10) | `CCO` | `LEGL` | Legal basis mandatory; time-boxed | — | 72h | [`XCC` #15] |
| 26 | Register adjustment / break resolution (M11) | `REGO` | `CCO` + `CFO` (+`X-TA`) | Any quantity change | If corrective tx: token-agent 2-of-3 | 72h | TA golden record [`REG-US` §5] |
| 27 | Lock snapshot for record date (M11/M12) | `FINA` | `CFO` | Always | — | Until record date | RealT average-balance method |
| 28 | Approve distribution run (M12) | `FINA` | `CFO` | ≤ $1M | — | 72h | Every §11 "payout → CFO" |
| 29 | Approve distribution run, large or anomalous (M12) | `FINA` | `CFO` + CEO | > $1M, or variance > 25% vs prior | — | 72h | — |
| 30 | Execute payout push / claim root (M12) | `TRSY` / `TOKO` | Auto if file hash = approved run | Destinations ⊆ approved file | **Distribution vault 2-of-3** (policy: amount ≤ run total; allow-listed contract/addresses) | 24h | Blocksquare `pushRevenueToUser`; ATS Mass Payout |
| 31 | Zero or reduced distribution declaration (M12) | `ASTM` | `CFO` + management board | Below forecast by > X% | — | — | Propvest zero-payout [`eu/03-propvest` §11.2] |
| 32 | Exit waterfall / sale proceeds (M12) | `FINA` | `CFO` + `LEGL` + `X-RE`/SPV director | Always | Distribution vault 2-of-3 | 14d | Reental/Propvest/PRYPCO waterfalls |
| 33 | Withdrawal (M16) | Investor / `TRSY` | Auto ≤ $25k; `CFO` ≤ $250k; `CFO` + CEO > $250k | Allow-listed destination; travel-rule data (crypto) | Treasury vault 2-of-3 (crypto) | 24h | Lofty "treasury transfers above threshold" |
| 34 | Match deposit manually (M16) | `TRSY` | `CFO` above $10k | — | — | 72h | Reental ">€10k" [`eu/01-reental` §11.2] |
| 35 | Release escrow (M16) | `FINA` | `CFO` + `CCO` | After finalise | Escrow vault/contract 2-of-3 | 72h | [`XCC` #18] |
| 36 | Mint/redeem stablecoin; move treasury funds (M16) | `TRSY` | `CFO` | Any | Treasury 2-of-3 | 24h | [`VND` §5–6] |
| 37 | Change payment-asset allow-list (M16) | `CCO` | `CFO` | Always | — | 7d | GENIUS/MiCA/CBUAE [`XCC` #19] |
| 38 | List instrument / change venue mode (M14) | `MKTO` / `CCO` | `CCO` / `LEGL` | Licence flag bound | Token-agent 2-of-3 (venue address whitelist) | 7d | [`XCC` #30] |
| 39 | Set/modify price band or floor (M14) | `MKTO` | `CCO` + `CFO` | Always | — | 7d | PRYPCO ±15%, BrickX 80% |
| 40 | Halt instrument (M14) | `MKTO` (or auto) | Ratify `CCO` ≤ 2h | Incident / meeting / valuation event | Market pauser 1-of-N | — | BrickX halts |
| 41 | Resume trading / bust trade (M14) | `MKTO` | `CCO` | Always | — (bust: token-agent 2-of-3) | 24h | Restorative |
| 42 | Set window price (M14) | `VALO` | `CFO` (US ATS: `X-BDP`) | Always | — | Before window open | Stake exit-window pricing |
| 43 | Pool / lending parameter change (M15) | `RISK` | Risk committee (2) + `CCO` for LTV | Always | DeFi admin 2-of-3 | 7d · **TL 24h** | RealT/Reental lending |
| 44 | Pause pool / lending market (M15) | `RISK` | Ratify `CCO` ≤ 2h | Incident | Pool pauser 1-of-N | — | Protective |
| 45 | Oracle price update / source change (M15) | Automation / `VALO` | `RISK` | Deviation > X% or manual | DeFi admin 2-of-3 | 24h | Issuer-set oracle ⚑ [`us/04-realt` §11.2] |
| 46 | Publish governance notice; certify result (M17) | `LEGL` | `CCO`; certification by scrutineer / `X-RE` | Always | Optional result-hash anchor | — | BrickX co-sec → director |
| 47 | Approve marketing item (M21) | `MKTG` | `CCO` (US BD: `X-BDP` registered principal) | Always | — | Approved until expiry | FINRA 2210 [`us/02-redswan` §11] |
| 48 | Material announcement (M21) | `IRCM` | `CCO` + `LEGL` | Price-sensitive | — | 24h | Ad-hoc disclosure |
| 49 | Redress payment (M23) | `CMPH` | `CCO` (+`CFO` > $5k) | Always | — | 7d | Complaint SLAs |
| 50 | Partner commission / reward rules (M24) | `PART` | `CFO` + `CCO` (+`LEGL` if transaction-based) | Always | — | 14d | mogul clubs risk |
| 51 | Grant role / invite user (M25) | `SUPER` | `SECO` | Always; SoD check passes | Mirror on-chain if a signing role | 72h | — |
| 52 | Change approval policy or custody policy (M25/M08) | `SECO` | **Admin quorum**: `CCO` + `CFO` + CEO | Always | Custody admin quorum **3-of-5** | 7d · **TL 24h** | Fireblocks admin quorum |
| 53 | Activate break-glass (M25) | Any 2 of {`CCO`, `SECO`, CEO} | Post-hoc board review ≤ 5 business days | Incident declared | Emergency signer set | ≤ 4h session | §5 |
| 54 | API key with write scope; MCP write tool (M26) | `ENGO` / `PART` | `SECO` (+`CCO` for agent tools) | Always | — | 72h | Brickken agent mandates |
| 55 | Feature flag / fee schedule / tenant guardrail (M27) | `SUPER` / `CFO` / `CCO` | `CCO` / CEO / `LEGL` | Licence-bound flags only | — | 7d | Brickken guardrails |
| 56 | Global inflow freeze / marketplace suspend (M27) | `CCO` or RE board delegate | Board ratification ≤ 24h | Incident / regulatory | Optional pause (1-of-N) | — | Assetora 2024 freeze [`australia/05-assetora` §2.8] |
| 57 | Dissolve SPV / wind down (M02) | `LEGL` | `CFO` + `X-RE`/SPV director (+ court fiduciary where appointed) | Always | Owner 3-of-5 for final burns | 30d | RealT liquidation waterfall |

---

## 4. Segregation-of-duties (SoD) conflicts

**Hard conflicts** block the grant or action; the system enforces them. **Soft conflicts** are allowed only with a time-boxed exception approved by `CCO` + `SECO` and are reported monthly.

| # | Conflict | Type | Rationale | Enforcement |
|---|---|---|---|---|
| 1 | Maker = checker on the same ApprovalRequest | Hard | Four-eyes | Checker list excludes maker, including via delegation or agent |
| 2 | `AGNT` as checker, or as signer | Hard | Agents act only on delegation | Agents cannot hold `A`/`O` |
| 3 | `SUPER` with `TRSY`/`FINA`/`CFO`/`TOKO` or any custody signer | Hard | Configuration vs money/keys | PRYPCO / Ctrl Alt / BrickX pattern |
| 4 | `TOKO` initiator also co-signing the same custody tx | Hard | Initiation vs authorisation | Custody policy: initiator ≠ approver |
| 5 | `KYCA` approving own-submitted assisted order, or own relative's case | Hard | Conflict of interest | Relationship register check |
| 6 | `FINA` (run maker) with `CFO` (run checker) | Hard | Four-eyes on money | Role exclusivity |
| 7 | `VALO` with `MKTO` (sets band reference and runs market) | Hard | Price-setting vs market operation | Role exclusivity |
| 8 | `VALO` with `RISK` oracle approver | Hard | Oracle independence (RealT ⚑) | Role exclusivity |
| 9 | `REGO` register adjustment and break-closure approval | Hard | Register integrity | Checker must be `CCO` + `CFO` |
| 10 | `ASTM` proposing sale and certifying vote result | Hard | Governance integrity | Certification by `LEGL` + scrutineer/`X-RE` |
| 11 | `MLRO` case content visible to `SUPP`/`IRCM` | Hard | Tipping-off prohibition | Case ACL, reason masking |
| 12 | `SECO` granting roles to self; `SUPER` inviting self as approver | Hard | Privilege escalation | No-self-grant rule |
| 13 | `ENGO` deploying to prod and approving the upgrade | Hard | Change management | Upgrade checkers `SECO` + `CCO` |
| 14 | `CAPM` approving own offering compliance review | Hard | Product vs compliance | Checker set excludes offering team |
| 15 | `X-ISA` approving platform-level compliance of own offering (A5) | Hard | Tenant vs platform guardrail | Platform `CCO` must approve tenant offerings |
| 16 | `PART` approving commissions for partners they manage | Hard | Incentive conflict | `CFO` + `CCO` checkers |
| 17 | `MKTO` trading or holding in instruments they supervise | Soft | Market abuse | Staff dealing policy, pre-clearance, insider list |
| 18 | `ASTM` also `X-PM` for the same asset (related PM) | Soft | Related-party (mogul manager = PM) | RPT register + independent review |
| 19 | Investment committee member with sponsor affiliation | Soft | Related-party purchases (Arrived affiliate premium) | Recusal + independent reviewer |
| 20 | `LEGL` as both maker of forced transfer and `X-TA` liaison approving it | Hard | Transfer-agent independence | `X-TA` must be a separate entity or officer |

---

## 5. Break-glass and emergency procedures

**Break-glass accounts** are sealed, split-knowledge credentials held by `CCO`, `SECO` and the CEO. Activation needs any **2 of 3** (M25). The session is time-boxed (≤ 4h), screen-recorded and logged as `iam.breakglass.*`, and triggers board notification and a post-incident review within 5 business days. Runbooks are drilled quarterly on testnet (M09.8).

| Runbook | Trigger | Immediate actions (≤ 15 min) | Next 24h | Restore criteria | Comms |
|---|---|---|---|---|---|
| **R1 Global pause** | Smart-contract exploit; mass unauthorised transfers; unknown-holder spike (M11 Sev1); regulator directive | Pauser `pause` on affected tokens (1-of-N); M14 halt all venues; M27 inflow freeze; stop payout jobs; snapshot state (M11) | Forensics; register vs chain reconciliation; legal assessment; regulator notification per licence conditions | Root cause fixed and patched (or migrated, R3); reconciliation clean; **unpause 4E+Q** (`CCO` + `SECO` + `CFO`) | Status banner; investor email; regulator (VARA/ASIC/SEC TA/NCA per licence) |
| **R2 Key compromise: agent/operational key** | Suspicious signing, custodian alert, device loss | Freeze affected vault in custody; revoke device/signer; pause tokens where that key holds agent rights | Owner quorum `removeAgent` then add new agent (TL may be waived by owner 3-of-5 + board); rotate hot keys; review all tx since last known-good | New agent keys ceremony-created; policies re-baselined | Internal; regulator if client assets affected |
| **R3 Key compromise: owner key / smart-contract flaw requiring migration** | Owner multisig threshold at risk; critical vulnerability | Pause; freeze owner-controlled functions; engage custodian and auditor | Deploy new contract suite (sandbox → prod); **snapshot → batch mint on new → forced-transfer/burn on old**; update registry refs, venues, pools, oracles | Register reconciled 1:1; old contract paused permanently; documents updated (ERC-1643) | Public notice; investor FAQs; venue notices |
| **R4 Registry sync failure (DLD/RER/TA)** | Sync backlog > SLA; registry rejects | Halt secondary settlement for affected assets (M14); queue instructions; **no on-chain moves without registry ack** | Manual reconciliation with registry; force re-sync (4E) | Zero pending; 3-way recon clean | Distributor tenants; affected investors |
| **R5 Valuation/oracle failure** | Reference price stale or deviation > X% | Halt instruments using the reference; set collateral factor 0 (M15) | Re-valuation (independent); committee publish | New published valuation; bands recomputed | Market notice |
| **R6 Stablecoin or payment-asset event** | Depeg > 0.5%, issuer sanction/regulatory change, bank or EMI failure | Suspend payouts and pay-ins on the affected asset/rail; switch to fallback rail per allow-list | Treasury rebalancing; investor election for payout asset | Peg restored or asset replaced | Investors (payout asset change) |
| **R7 Sanctions hit on a large holder** | True match | Freeze wallet(s) (`SGL+R`); hold distributions (M12 exceptions) | Report (OFAC/EOCN/DFAT/EU); legal review | Delisting or licence | **No tipping-off**; regulator only |
| **R8 Court order / legal hold / receivership** | Court order (e.g., rent into escrow, RealT TRO), fiduciary appointment | Asset → `legal_hold`; pause distributions for asset; halt instrument | Engage counsel; comply with order; disclosure | Order discharged or executed | Holders (disclosure); regulator as required |
| **R9 Operator cessation / wind-down** | Insolvency risk, licence loss | Execute cessation plan: freeze inflows; appoint administrator/replacement manager; export investor register | Property sale programme; waterfall (M12) | Final distribution; tokens burned; entities dissolved | Per cessation plan (PRYPCO/Stake patterns [`mena/01-prypco` §2.8; `mena/03-stake` §2.8]) |

---

## 6. Key-management and signer policy (Fireblocks-style policy engine)

### 6.1 Vault / key architecture (RealX default)

| Vault | Holds / on-chain role | Custody model | Quorum | Policy highlights | Evidence |
|---|---|---|---|---|---|
| **V1 Owner** | ERC-3643 **Owner** (appoints agents, sets compliance/IR, upgrades) | Cold MPC or Safe multisig | **3-of-5**: CEO, `CFO`, `CCO`, `SECO`, independent director or `X-CUS` | Contract-call allow-list (owner functions only); **24–72h timelock**; no value transfers | ERC-3643 Owner/Agent [`TSD` §1.4] |
| **V2 Token agent** | Assets agent: mint/burn/forced transfer/recovery/partial freeze | Warm MPC | **2-of-3**: `TOKO` initiates; approvers `CFO`-delegate, `CCO`-delegate, `X-TA` or `X-CUS` | Function allow-list; per-tx and daily supply limits; destination = identity-registered wallets; requires approval ID | Tokeny Assets domain; Fireblocks × Tokeny [`WLD` §2] |
| **V3 Pauser / freezer** | `pause`, `setAddressFrozen(true)` only | Warm | **1-of-N** (`CCO`, `SECO`, `MLRO`, on-call) | Can only *restrict*; unpause/unfreeze are routed to V2 | Emergency controls (Stobox) [`APE` M6] |
| **V4 Identity agent** | Identity Registry agent: `registerIdentity`, `updateCountry`, `deleteIdentity` | Hot (service account) | Policy auto-sign | Only IR functions; rate limit; only for approved KYC cases | Tokeny Identities domain |
| **V5 Distribution** | Stablecoin payouts | Hot/warm | 2-of-3 or auto on hash match | Amount ≤ approved run; destinations ⊆ approved file; time window | Blocksquare push; ATS Mass Payout |
| **V6 Treasury** | Operating stablecoin/fiat-bridge funds | Warm | 2-of-3 | Allow-listed destinations; travel-rule payload; daily limits | Fireblocks rules [`WLD` §12] |
| **V7 Paymaster** | Gas sponsorship deposits | Hot | Auto within cap | Top-up ≤ cap/day; alerts on runway | Reental paymaster [`eu/01-reental` §11 #15] |
| **V8 DeFi admin** (A1 only) | Pool/lending configurator, oracle updater | Warm | 2-of-3 | 24h timelock; parameter bounds (e.g., LTV ≤ 75%) | RealT/Reental [`us/04-realt` §11.2] |
| **V9 Escrow** | Offering escrow contracts / accounts | Warm or contract | 2-of-3 | Release only after finalise approval | [`XCC` #18] |

### 6.2 Policy-engine rule set (illustrative defaults)

| Rule | Scope | Condition | Required approvals | Action if violated |
|---|---|---|---|---|
| P1 Approval-ID binding | All vaults except V7 | Tx note must carry an approved M25 request ID; hashes must match | — | Block |
| P2 Function allow-list | V1–V5, V8 | Contract + selector ∈ allow-list for the vault | — | Block |
| P3 Supply guard | V2 | Mint ≤ remaining allocation of the referenced file; daily mint ≤ X% of supply | 2-of-3 | Block over limit (escalate to V1 3-of-5) |
| P4 Destination guard | V2, V5, V6 | Destination ∈ identity-registered wallets (V2), approved file (V5), allow-listed address book (V6) | — | Block |
| P5 Amount bands | V6 | ≤ $25k auto; ≤ $250k 2-of-3; > $250k 3-of-5 | As listed | Escalate |
| P6 Time window | V2, V6 | Business hours unless the break-glass flag is set | — | Hold for approval |
| P7 KYT screening | V5, V6 | Counterparty risk score < threshold | — | Block + M20 alert |
| P8 Admin quorum | Policy changes | Any change to P1–P7, signers or vaults | 3-of-5 (V1 set) | Block |
| P9 Initiator ≠ approver | All | Signer identity differs from initiator | — | Block |
| P10 Timelock | V1, V8 | Owner or DeFi parameter calls queued ≥ TL | — | Queue |

### 6.3 Chain analogues for the same controls

| Control | EVM (ERC-3643) | Algorand (ASA) | XRPL | Hedera (HTS) | Stellar | Solana (Token-2022) |
|---|---|---|---|---|---|---|
| Owner quorum | Safe multisig / MPC owner | Manager address = **multisig account** (msig 3-of-5) | Issuer account **SignerListSet** quorum; master key disabled | Admin key = **KeyList threshold** | Signer weights + high threshold | Squads multisig as authority [I] |
| Agent (mint/burn/forced transfer) | Agent role held by V2 | Clawback + reserve addresses held by multisig | Issuer with CanClawback (MPT) | Supply + wipe keys (threshold) | Issuer with clawback flag | Mint authority + permanent delegate |
| Freezer / pauser | Agent with pause/freeze rights (V3) | Freeze address | CanLock holder / global lock | Freeze + pause keys | Revocable authorisation | Freeze authority / pausable ext. |
| Identity / KYC grant | IR agent (V4) | Unfreeze after opt-in (default-frozen) | `MPTokenAuthorize` / Credentials | KYC key `TokenGrantKyc` | `SetTrustLineFlags` authorise | Thaw / transfer-hook allow-list |
| Irreversibility hazards | Renounce ownership | **Setting a role address to empty is permanent** | Disabling master without a signer list | Removing keys | `AUTH_IMMUTABLE`; master weight 0 | Setting an authority to None |

Sources: [`TSD` §2, §4]. On Algorand, Lofty's likely use of clawback/freeze addresses is [I] [`us/01-lofty` §7].

### 6.4 Key lifecycle

- **Ceremonies** for V1/V2/V8 are witnessed by `SECO` + `X-CUS` and recorded in M08.4 (`custody.ceremony.recorded`).
- **Rotation:** hot keys every 90 days; warm keys every 12 months or on staff change; owner quorum reviewed at each board change.
- **Signer offboarding** happens on the same day as HR offboarding. The quorum is re-baselined under the P8 admin quorum.
- **Custodian due diligence** (M02 licences), aligned to the relevant regime:
  - US BD 15c3-3 "five conditions": access, DLT assessment, no known material weakness, key safeguarding, continuity [`REG-US` §5];
  - ECSPR Art 10 qualified custodian [`REG-EU` §5];
  - VARA licensed custodian and ARVA reserve segregation [`REG-ME` §5];
  - ASIC RG 133 and omnibus relief 2025/871 [`REG-AU` §5].
- **Omnibus model (A3):** per-investor sub-ledger attestations (M08.5) mitigate the "single warm MPC omnibus" concentration risk [`mena/02-ctrl-alt` §10].
