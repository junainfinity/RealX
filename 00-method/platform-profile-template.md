# Platform Profile Template (used for every platform file)

Every platform file in `02-platforms/<region>/` follows this structure so platforms can be compared line-by-line.

## Evidence labels (mandatory on every non-trivial claim)

| Label | Meaning |
|---|---|
| **[C#n]** | Confirmed — stated by a primary/credible source; `n` = source number in the Sources section |
| **[I]** | Inferred — reasoned from confirmed facts (job posts, contract behaviour, docs, UI, regulatory necessity). The reasoning is given inline or in a footnote |
| **[S]** | Speculative — plausible industry-standard assumption with no direct evidence |

---

```markdown
# <Platform name> — <Region> (<HQ country>)

| Field | Value |
|---|---|
| Website | |
| HQ / legal entity | |
| Founded | |
| Status as of Sep 2026 | active / paused / wound down / pivoted |
| Category | on-chain tokenized fractional / fractional (off-chain ledger) / tokenization infra (white-label) / hybrid |
| Asset focus | residential rental / CRE / off-plan / debt / land … |
| Scale (AUM, properties, investors, volume) | with sources + dates |
| Min. investment | |
| Why it is in the Top 5 | ranking rationale |
| Research confidence | High / Medium / Low |

## 1. Business model snapshot
## 2. Legal & compliance structure
 2.1 Issuing vehicle (per-property LLC / Series LLC / SPV / fund / trust / bond / DLD-registered fraction …)
 2.2 What the token legally represents
 2.3 Securities exemptions / licences / registrations used
 2.4 Regulators & licensed partners (broker-dealer, transfer agent, custodian, ATS/MTF, escrow, trustee)
 2.5 KYC / AML / KYB / accreditation / sanctions / travel rule (vendors if known)
 2.6 Investor eligibility & geo-restrictions
 2.7 Tax handling & reporting
 2.8 Disclosures, investor protections, known regulatory actions
## 3. User types (personas & system roles)
## 4. Actions each user type can exercise (table: role → actions → channel/UI → on-chain or off-chain)
## 5. Core flows (numbered steps; add a mermaid sequence diagram for at least primary investment + distribution)
 5.1 Onboarding & KYC  5.2 Primary purchase  5.3 Income distribution  5.4 Secondary trading / transfer
 5.5 Redemption / exit / property sale  5.6 Governance / voting (if any)  5.7 Issuer / property listing onboarding
## 6. Reverse-engineered architecture (mermaid component diagram + narrative; on-chain vs off-chain split)
## 7. Tech stack (table: layer → technology → evidence label → source/reasoning)
  Layers: blockchain/L2, token standard, compliance/permissioning contracts, identity, wallets/custody,
  fiat rails, stablecoin rails, KYC vendor, marketplace/AMM/order book, oracles, frontend, backend,
  cloud/infra, data/analytics, CRM/support, e-signature, property management integration
## 8. Money flows & revenue model (fees, spreads, distribution currency and cadence)
## 9. Secondary market & liquidity mechanics
## 10. Risks, incidents, controversies, enforcement
## 11. Extrapolated platform-owner ADMIN PANEL (clearly labelled extrapolation)
  - Modules / pages (list), key widgets per page
  - Actions available to admins (with maker-checker where plausible)
  - Admin roles (RBAC) and what each can do
  - Alerts / automations / reports
  - Evidence driving each module (e.g., "they freeze tokens → token-control module")
## 12. Builder takeaways (copy / avoid / differentiate)
## 13. Sources (numbered, URL + title + date accessed)
```
