---
title: "Marketplace Notes"
created: 2026-09-23
updated: 2026-09-23
status: active
tags: [realx, research, marketplace, build-vs-buy]
---

# Marketplace Notes

**How a marketplace for other sponsors' property trusts works in Australia: precedents and build-vs-buy options.** These notes follow decision D02 in [[Decisions]].

Labels: [C] confirmed, [I] inferred and [S] speculative come from the pack. [I Claude] marks Claude's own inference.

## 1. Two tracks

| | Wholesale (now) | Retail (around 8 Apr 2027) |
|---|---|---|
| Who issues | Each sponsor's own wholesale trust, under the sponsor's own AFSL | One umbrella registered scheme run by a hired RE, with each sponsor's property as a sub-fund |
| RealX's role | Arranges deals as a CAR of a licence holder (weeks) [I]; runs the platform; vets sponsors | Same, plus DDO distributor duties for each TMD [I] |
| Precedent | Tokeniser: "the issuer keeps its own vehicle and licence" [C] | Assetora: one ARSN, 130+ sub-funds; sponsors propose assets and the RE vets them [C] |

## 2. Precedents

### Tokeniser + Liquidise (Australia): [[03-tokeniser-liquidise]]
- **Product:** B2B SaaS covering registry, KYC, investor portal, subscriptions and distributions. Liquidity, leverage and API are optional add-ons [C].
- **Pricing:** "Emerging" funds pay A$11,000 a year + A$2,500 setup, for about 50 investors. "Enterprise" is tailored (SPVs, retail). Company plans are priced per investor [C].
- **Scale (self-reported):** A$1.2B issued, 160+ issuers, A$2B+ in transactions [C].
- **Licensing:** the issuer's AFSL "covers the offer". Tokeniser's own AFSL is not disclosed. Liquidise holds AFSL 547681 [C].
- **Wallets and KYC:** non-custodial Averer wallets, apparently designed to stay outside the DAP "holding" trigger [I]. KYC uses reusable zero-knowledge credentials [C].
- **Risks:** its trading pools rest on an "exempt public protocol" theory that is untested under s767A [I]. It settles in its own LIQ token, which the pack says to avoid [C].
- **Partners:** IQ-EQ for trustee and registry [C].
- **Gap:** no named property-fund client and no property-specific product [C].

### Assetora (DomaCom) + Bricklet (Australia): [[05-assetora-domacom-bricklet]]
- **Structure:** one registered MIS (ARSN 167 020 626) with a sub-fund per asset. Perpetual holds title [C].
- **Listing flow** [C]:
  1. A sponsor or adviser proposes an asset.
  2. The RE does due diligence and sets a fixed price.
  3. A campaign opens.
  4. At target, a supplementary PDS is issued.
  5. A buyer's agent buys the property, the custodian takes title, and units are issued.
- **Freeze:** in Jul 2024 the market froze after NDIS sub-funds could not find tenants. Inflows and launches stayed suspended for about 13 months, and the listed parent is suspended on the ASX [C].

### Other patterns relevant to a marketplace

| Pattern | From | Copy or avoid | Source |
|---|---|---|---|
| Seller keeps at least 10% of the property | Lofty | Copy (skin in the game) | [[implications-for-realx]] §1.1 |
| One KYC across many marketplaces | Blocksquare | Copy | [[implications-for-realx]] §1.1 |
| Public planned-vs-actual track record | Domoblock | Copy | [[implications-for-realx]] §1.1 |
| Escrow with soft/hard caps and automatic refunds | Brickken | Copy | [[implications-for-realx]] §1.1 |
| Syndicate sponsors creating private syndicates | BrickX "Select & Syndicate" | Reference | [[01-brickx]] §3 |
| Fees paid in the property token | Blocksquare | Avoid | [[implications-for-realx]] §1.2 |
| Settling in a proprietary token (LIQ) | Tokeniser | Avoid | [[03-tokeniser-liquidise]] §12 |
| Letting issuers skip KYC | Brickken (the pack's QA found Brickken already blocks `needKyc=false` in production; see [[Pack Issues]] I-01) | Avoid: copy Brickken's hard block | [[04-brickken]] §12 |

## 3. Admin panel
- **Two tiers:** a RealX (operator) console over a sponsor (tenant) console. This is admin archetype A5 in [[00-overview]] §6.
- **Hard problems:** tenant isolation, guardrails against non-compliant tenants, and shared KYC across tenants [I].
- **Modules:** M27 (tenancy) becomes core and M24 (partners and distributors) matters. Every object carries a `tenant_id` from day one ([[00-overview]] §9).

## 4. Build vs buy

Founder question 12 asks: in-house ERC-3643, or Tokeny, Fireblocks, Brickken or Tokeniser? The pack gives options but no recommendation ([[implications-for-realx]] §5).

| Option | What it means | Trade-off | Evidence |
|---|---|---|---|
| 1. Build on open standards and rent the services **(Claude's recommendation)** | ERC-3643 contracts (reference implementation T-REX, GPL-3.0); Fireblocks for signing and policy; embedded wallets (Privy or Dynamic); rented KYC (Sumsub, or in Australia, Averer credentials or Equifax eIDV). RealX builds the marketplace, sponsor console and admin panel | Most engineering, full control. Matches the pack's default stack and its "own the experience" principle | [[tech-stack-comparison]]; [[token-standards]] §1.4 |
| 2. White-label the token layer | Tokeny's T-REX Platform (servicing and investor apps; part of Apex Group; 120+ issuers). RealX builds the marketplace on top | Less token work, but RealX depends on the vendor's roadmap [I Claude]. Pricing is not in the pack | [[white-label-issuer-dashboards]] §2 |
| 3. Run on Tokeniser + Averer + Redbelly | Use Tokeniser's registry, KYC, portal and distributions. RealX focuses on origination, the RE relationship and UX | Fastest. But Tokeniser sells to the same sponsors [I Claude], and its trading pools rest on an untested theory [I] | [[03-tokeniser-liquidise]] §12 |

Other vendor prices in the pack: Blocksquare charges US$5,990 per tokenisation plus US$69 a month per property; DigiShares Advanced costs €50,000+ ([[white-label-issuer-dashboards]]).

**Why option 1 [I Claude]:**
- For a marketplace, the sponsor and investor experience is the product, so owning it matters.
- The token contracts are an open standard.
- The regulated parts (licences, custody, KYC) are rented anyway.

**To check:** the T-REX reference implementation is GPL-3.0. Confirm the licence implications with counsel before deploying modified contracts.
