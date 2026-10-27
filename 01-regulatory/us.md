# United States — Regulatory Landscape for Tokenised Fractional Real Estate

> **Research, not legal advice.** Snapshot as of 23 Sep 2026; verify with US securities/tax counsel before acting.

**Evidence labels:** [C#n] confirmed by numbered source n (list at end) · [I] inferred (reasoning or well-established law not re-fetched in this session) · [S] speculative.

**Bottom line.** In the US a fractional real-estate token is a **security** (an LLC/LP/REIT interest or investment contract "formatted as or represented by a crypto asset"). Tokenisation changes the record-keeping technology, not the legal analysis [C#1]. A new entrant therefore lives inside the ordinary exempt-offering regime (Reg D / Reg S / Reg A+ / Reg CF). It also needs a registered transfer agent (or issuer self-transfer), a FINRA broker-dealer, and, if it runs secondary trading, an ATS. 2025–26 brought a lot of guidance (SEC staff tokenisation taxonomy, DTC pilot, the transfer-agent modernisation proposal, the innovation exemption) but **no statute that changes the treatment of tokenised real estate**. The CLARITY Act failed cloture on 15 Sep 2026 [C#4].

---

## 1. Classification of a fractional real-estate token

| Structure | Likely classification | Test / basis |
|---|---|---|
| Token = membership unit in a single-property LLC / Series LLC (the most common US model) | Security (LLC interest / investment contract) | *SEC v. W.J. Howey* (1946): investment of money, common enterprise, expectation of profit, from the efforts of others (sponsor/property manager) [I]. SEC staff: "the format in which a security is issued or the methods by which holders are recorded does not affect application of the federal securities laws" [C#1] |
| Token = share in a REIT (private or Reg A) | Security (stock) | Enumerated "stock" in Securities Act §2(a)(1) [I] |
| Token = debt/note secured on property | Security (note) unless a *Reves* "family resemblance" exception applies | *Reves v. Ernst & Young* (1990) [I] |
| Token = direct tenancy-in-common deed fraction with no management arrangement | Possibly not a security; usually becomes one once pooled management or leasing is bundled | Howey "efforts of others" [I] |
| Third-party "wrapper" token over an existing security | "Custodial tokenized security" (security entitlement) or "synthetic/linked security", still a security | SEC staff taxonomy of 28 Jan 2026 [C#1] |

- **SEC staff taxonomy (28 Jan 2026).** Joint statement of Corporation Finance, Investment Management and Trading & Markets. It defines a *tokenized security* as "a financial instrument enumerated in the definition of 'security'… formatted as or represented by a crypto asset". It splits them into **issuer-sponsored** tokens (the master securityholder file is kept on-chain by the issuer or its transfer agent) and **third-party-sponsored** tokens (custodial or synthetic) [C#1]. RealX-style SPV tokens are **issuer-sponsored** [I].
- **SEC/CFTC interpretation (17 Mar 2026).** Sets out a five-way taxonomy: digital commodities, digital collectibles, digital tools, stablecoins and **digital securities**. It confirms that most crypto assets are not securities, but digital securities remain securities [C#14].
- **Proposed Regulation Crypto Assets (18 Aug 2026).** Creates offering exemptions for crypto-asset investment contracts ($5M over 4 years, and $20M/$75M tiers) plus a Rule 400 "investment contract ends" safe harbour. **Tokenized stocks, bonds and other instruments that are themselves securities remain under the federal securities laws and outside the proposed exemptions** [C#12][C#13]. A real-estate token cannot use it [I].
- **Investment Company Act.** A single-property SPV holds real estate, not securities, so it is usually not an investment company. A multi-asset or fund-of-SPVs vehicle typically relies on §3(c)(5)(C) (primarily holding real-estate interests) or §3(c)(1)/(7) (the last two restrict investor counts or types) [I].
- **Exchange Act §12(g).** An issuer with ≥$10M in assets and **2,000 holders of record, or 500 non-accredited holders**, must register. Tokenisation can inflate holder counts because each wallet on an issuer-sponsored ledger is plausibly a holder of record. Reg A Tier 2 and Reg CF have conditional 12(g) exemptions [I]. **Admin control: holder-count monitor per SPV.**
- **Tax classification (for structuring).** If LLC tokens trade on an ATS, the LLC may be a *publicly traded partnership* (IRC §7704). Rental income is "qualifying income", so it usually avoids corporate tax, but the K-1 and §1446(f) withholding burden is heavy [I].

## 2. Licence / exemption pathways for a new platform

### Licence pathway options for a new entrant

| Pathway | Who can invest | Raise cap | Investor caps | Gen. solicitation | Resale / lock-up | Partners typically required | SEC / state filings | Indicative cost / time |
|---|---|---|---|---|---|---|---|---|
| **Reg D 506(b)** | Unlimited accredited investors + ≤35 sophisticated non-accredited investors (per 90 days) [C#24] | Unlimited [C#24] | None for accredited investors; non-accredited investors must receive Reg D disclosure package [I] | **No** [C#24] | Restricted securities; Rule 144: 1 yr for non-reporting issuer [I] | BD for paid solicitation; TA or issuer-kept register; escrow optional [I] | Form D ≤15 days after first sale [C#24]; state notice filings and fees (NSMIA pre-empts registration) [I] | Legal $25–75k per SPV [S]; weeks |
| **Reg D 506(c)** | Accredited only, with **reasonable-steps verification** [C#24] | Unlimited | None; verification via documents/third-party letter, **or $200k minimum (natural person) / $1M (entity) + written reps + no third-party financing** (NAL 12 Mar 2025) [C#8] | **Yes** [C#24] | As 506(b) | Verification vendor; BD if paid for placement; TA [I] | As 506(b) | As 506(b), plus verification ~$50–100 per investor [S] |
| **Reg S** (offshore tranche, often run alongside Reg D) | Non-US persons offshore | Unlimited | None federal; local rules of investor's country apply [I] | No directed selling efforts into US [I] | Distribution-compliance period (up to 1 yr for equity of non-reporting US issuer, Cat. 3) [I] | Geo-blocking and US-person screening; FIRPTA/ECI withholding agent [I] | None with SEC [I] | Low incremental cost [I] |
| **Reg A+ Tier 1** | Anyone | $20M / 12 mo [C#26] | None federal | Yes (test the waters) [I] | Unrestricted federally, but state blue-sky registration required [I] | BD optional; TA [I] | Form 1-A qualification + **state registration** [I] | Rarely used [I] |
| **Reg A+ Tier 2** | Anyone; non-accredited investors capped at 10% of greater of income/net worth unless the security is listed [I] | $75M / 12 mo [C#26] | Yes (non-accredited) | Yes | **Freely tradable** federally; state secondary-trading exemptions vary [I] | BD (often), registered TA (needed for the 12(g) conditional exemption), auditor (audited FS required) [C#26][I] | Form 1-A qualification (SEC review ~3–6 mo [S]); 1-K, 1-SA, 1-U ongoing [I]; state pre-emption for offering [I] | $100–300k first offering [S]; 4–9 months [S] |
| **Reg CF** | Anyone, with investment limits for non-accredited investors [C#25] | **$5M / rolling 12 mo** (C&DI Feb 2026: rolling from each closing) [C#25][C#27] | Non-accredited: greater of $2,500 or 5% of income/NW (if <$124k), else 10% capped at $124k [I] | Limited (tombstone off-platform; full info on portal) [I] | **1-year** resale restriction [C#25] | **Must use a registered funding portal or BD** [C#25]; qualified third-party escrow [I] | Form C, C-AR annual [I] | Portal fee 5–8% [S]; 1–3 months |
| **Rule 144 / 4(a)(7)** (resale, not raise) | Resale to accredited investors (4(a)(7)) or public after holding period | n/a | n/a | 4(a)(7): no general solicitation [I] | 144: 6 mo (reporting) / 1 yr (non-reporting) [I] | BD/ATS for matched trading [I] | None [I] | n/a |
| **Own broker-dealer (FINRA NMA)** | Required to receive transaction-based compensation or run an ATS | n/a | n/a | FINRA 2210 comms rules | n/a | Clearing / custody arrangements, FinOps, CCO | Form BD, Form NMA; FINRA must decide within 180 days of a substantially complete application [C#23] | Application fee $7.5k–55k; **4–9 months typical**; ~$125–175k initial capital excl. advisers [C#23] |
| **ATS (secondary market)** | BD-registered operator | n/a | n/a | n/a | Enables secondary trading of restricted securities where eligible | Must be a registered BD | Form ATS / ATS-N (NMS only) [I] | Add 3–6 months beyond BD [S] |
| **Registered transfer agent** (Rule 17Ad) | Issuer's agent for master securityholder file | n/a | n/a | n/a | Controls transfer and lock-ups on chain | — | Form TA-1; Sept 2026 proposal would permit a DLT master file and add Form TA-2 tokenisation disclosures [C#11] | Weeks (registration); ongoing exam risk [I] |
| **"Rent-a-stack" (partner model)** | Depends on partner exemption | — | — | — | — | Partner BD/ATS (e.g., tokenisation-native BDs), partner TA, qualified custodian | Partner files | Fastest route (weeks); revenue share [I] |
| **FinCEN MSB registration** | If platform transmits fiat/stablecoin value outside a BD | — | — | — | — | Bank/stablecoin partner | FinCEN Form 107; state money-transmitter licences [I] | State MTLs are costly (≈50 states) [I] |

**Practical sequence for a new entrant [I]:**
1. Start with 506(c) + Reg S SPVs, using a partner BD and a registered TA.
2. Add Reg CF (via partner portal) or Reg A Tier 2 for retail.
3. Add a partner or own ATS for secondary trading after the Rule 144 holding periods.
4. Consider an own BD/TA once volume justifies it.

## 3. Investor eligibility, suitability and limits

- **Accredited investor (Rule 501(a)).** Income >$200k (or $300k joint) in each of the last two years; net worth >$1M excluding primary residence; Series 7/65/82 holders; entities with >$5M assets; family offices; knowledgeable employees [I]. The thresholds have not changed since 1982 [C#10].
- **506(c) verification.** Reasonable steps: tax returns, bank/brokerage statements, CPA/attorney/RIA/BD letter, or the **minimum-investment NAL** ($200k natural person / $1M entity, written reps, no third-party financing, no actual knowledge to the contrary) [C#8].
- **Pending reform.** The House passed the INVEST Act on 11 Dec 2025 (302–123). It would add inflation-adjusted thresholds, professional/education criteria and an **SEC-administered exam pathway** to accredited status, and raise the Reg CF accountant-review threshold to $250k. It sits in Senate committees [C#9]. No SEC rule change had been adopted as of mid-2026 [C#10].
- **Reg A Tier 2.** Non-accredited natural persons may invest ≤10% of the greater of annual income or net worth, unless the securities are listed on a national exchange [I].
- **Reg CF.** Aggregate 12-month limits across all Reg CF offerings for non-accredited investors [C#25]. "Annual income" means calendar year (C&DI, Feb 2026) [C#27].
- **Suitability.** A BD making recommendations to retail customers must follow Reg Best Interest and Form CRS. For non-retail, FINRA Rule 2111 suitability applies. Reg CF portals may not give investment advice [I].
- **Qualified purchaser** ($5M investments) applies if a fund uses §3(c)(7) [I].

## 4. KYC / AML / CFT, sanctions, travel rule, beneficial ownership

| Topic | Rule | Status / note |
|---|---|---|
| AML programme | BSA; FINRA Rule 3310 for BDs; funding portals have AML obligations as FINRA members [I] | BDs: CIP (31 CFR 1023.220), CDD/beneficial-ownership rule (31 CFR 1010.230, 25% ownership + control prong), SAR filing, CTRs [I] |
| Beneficial-ownership **registry** (CTA/BOI) | FinCEN final rule of 11 Aug 2026 (effective 14 Aug 2026) **permanently exempts domestic companies and US persons**; only foreign-registered entities report [C#22] | The CDD rule at account opening **is not changed**; FinCEN says it will revisit the CDD rule separately [C#22] |
| Residential real estate reporting (RRE) rule | Would have required closing agents to report non-financed transfers to entities/trusts from 1 Mar 2026 | **Vacated** 19 Mar 2026 (*Flowers Title v. Bessent*, E.D. Tex.); FinCEN says no filing is currently required [C#15]. SPV acquisitions in cash may again face Geographic Targeting Orders [I] |
| Sanctions | OFAC (strict liability): screen investors, wallets, counterparties, sellers of property [I] | DTC's tokenisation pilot screens every registered wallet for OFAC and allows transfers only between whitelisted wallets, a useful benchmark [C#7] |
| Travel rule | BSA Recordkeeping & Travel Rule, 31 CFR 1010.410(f), ≥$3,000 transmittals; FinCEN applies it to convertible virtual currency [I] | Applies to the platform when it transmits stablecoin/fiat on behalf of customers [I] |
| MSB | Platform transmitting value outside a BD/bank model may be an MSB (FinCEN 2019 CVC guidance) + state MTLs [I] | Avoid by routing funds through BD, bank escrow or a qualified third party [I] |
| Stablecoins | **GENIUS Act** signed 18 Jul 2025; effective the earlier of **18 Jan 2027** or 120 days after final regulations [C#2][C#6]; OCC NPRM published Mar 2026 [C#6 search] | Permitted payment-stablecoin issuers become BSA financial institutions. After the transition, US digital-asset service providers may only offer permitted payment stablecoins [I]. Admin control: accepted-stablecoin allow-list |
| Investment advisers | FinCEN IA AML rule (postponed) [I] | Relevant only if RealX advises a fund [I] |

## 5. Custody and client money

- **Issuer-sponsored model.** The transfer agent (or issuer) keeps the master securityholder file; the chain can be the file of record [C#1]. The **Sept 2026 transfer-agent proposal** (comments due 3 Nov 2026) would permit DLT as "all or part" of the master file only if the TA keeps **exclusive control** of the authoritative record. It would also impose integrity, redundancy and audit-trail controls, six-year retention, and Form TA-2 disclosure of tokenisation agents and DLT platforms. Wallet addresses **cannot yet replace** names and mailing addresses [C#11]. **Admin controls:** off-chain golden record ↔ on-chain reconciliation, and TA-controlled forced-transfer / re-issue.
- **Broker-dealer custody.** The SEC T&M statement of 17 Dec 2025 sets five conditions for a BD to be in "physical possession" of crypto-asset securities under Rule 15c3-3(b)(1): (1) access and ability to transfer; (2) a documented DLT assessment; (3) no known material weaknesses; (4) private-key safeguarding; (5) continuity plans for forks, 51% attacks and bankruptcy. The 2020 special-purpose BD statement remains valid [C#19].
- **Investment-adviser custody** (if RealX advises a fund): Rule 206(4)-2 requires a qualified custodian [I].
- **Offering funds.** Reg CF requires a qualified third-party escrow (bank) until the target is met. BD-run all-or-none/min-max offerings follow Rule 15c2-4 escrow [I].
- **Self-custody.** Permitted: investors may hold tokens in their own wallets if transfer restrictions are enforced (whitelist / ERC-3643-type permissioning) and the TA can re-issue or force-transfer [I]. The DTC pilot likewise lets DTC Participants (only) register blockchain addresses as "Registered Wallets" for which "the relevant Participant would be fully responsible", with tokens transferable only between Registered Wallets [C#7]; the letter does not state who holds the private keys, so "self-held" is [I].

## 6. Secondary trading

- Securities from Reg D, Reg S and Reg CF are **restricted**. Resale needs Rule 144 (1 yr for non-reporting issuers), §4(a)(7) (accredited buyers, information requirements), or §4(a)(1½) [I]. Reg CF resale is barred for 1 year [C#25].
- **Platform-run matching = exchange / ATS / broker activity.** A new entrant needs a partner ATS or its own BD+ATS (Reg ATS; fair-access rules above volume thresholds) [I]. Direct P2P transfers between whitelisted wallets *without* platform matching or compensation are lower risk, but the TA must still record them [I].
- Reg A Tier 2 shares are freely tradable federally, but broker-assisted secondary trading needs state "manual" or blue-sky exemptions [I].
- **SEC "innovation exemption" (17 Sep 2026).** A five-year conditional exemption for "Tokenized Securities Venues" trading **tokenized NMS stock only**, with volume and symbol limits, auditable public smart contracts, and trading halts synced to the primary market. It **does not cover private or real-estate tokens** [C#5].
- **DTC tokenisation NAL (11 Dec 2025).** Covers Russell 1000 stocks, Treasuries and major-index ETFs. Pilot H1 2026, then broader rollout in H2 2026. Transfers only between DTC-registered, OFAC-screened wallets [C#7]. It is not available to private real-estate tokens but is the regulatory template for whitelisted-wallet design [I].
- **Tax constraint.** Active secondary trading can make the LLC a PTP (see §1) [I].

## 7. Disclosure, ongoing reporting and marketing

- **Offering disclosure.**
  - Reg D: no mandated document for accredited-only offerings, but anti-fraud (Rule 10b-5) drives a full PPM. Form D within 15 days [C#24].
  - Reg A: Form 1-A offering circular; Tier 2 ongoing Form 1-K (annual audited), 1-SA (semi-annual), 1-U (current events) [I].
  - Reg CF: Form C; annual C-AR; stale financials must be updated for offerings open past 30 April (C&DI 2026) [C#27].
- **Marketing.**
  - 506(b): no general solicitation.
  - 506(c) and Reg A: general solicitation allowed (Reg A test-the-waters materials must be filed or legended) [I].
  - Reg CF: advertising outside the portal is limited to tombstone-style notices [I].
  - Reg S: no directed selling efforts into the US [I].
  - BD communications: FINRA Rule 2210 [I].
- **Misrepresentation risk.** Texas SSB's June 2026 emergency order against E-Estate Group targeted claims that tokens were "backed by real estate", that "capital was protected", and that the offering was compliant [C#18]. **Admin control:** marketing-claims review workflow.
- **Tokenisation-specific disclosure** (smart-contract risk, chain choice, forced-transfer powers, who keeps the golden record) is expected by the SEC staff statement's logic and the TA proposal [I].

## 8. Tax reporting

| Item | Rule | Platform action |
|---|---|---|
| Broker reporting on dispositions | Tokenized securities that are dual-classification assets are reported on **Form 1099-DA, not 1099-B**, **unless** traded on a *limited-access regulated network* (then 1099-B) [C#20]. Gross proceeds from 2025; **basis reporting mandatory from 2026** [C#21] | Decide which network status applies; build 1099-DA/1099-B engine and cost-basis lots [I] |
| Distributions | LLC/partnership SPV → Schedule **K-1**; REIT → **1099-DIV** [I] | K-1 generation per holder per SPV (heavy if many holders) [I] |
| Foreign investors | FDAP / effectively connected income withholding: §1441/1442 (30%) or §1446(a) partnership ECI withholding; **FIRPTA** (§897/§1445: 15% withholding on dispositions of USRPIs); **§1446(f)** 10% withholding on transfers of partnership interests [I] | W-8BEN/W-8BEN-E collection, withholding engine, Form 1042-S / 8804-8805 [I] |
| FATCA | W-9/W-8 self-certification; FATCA status of entity investors [I] | Tax-form collection and expiry tracking [I] |
| CRS | The US is not a CRS jurisdiction; non-US platforms must still report US-sourced accounts under their own CRS [I] | — |
| State | Nonresident partner withholding and composite returns in property states [I] | State-tax module [I] |

## 9. Land registry, title linkage, foreign ownership

- Title stays in the **county recorder** system; no US state recognises a token as title. The SPV holds the deed and the token represents SPV equity [I]. Delaware permits blockchain stock ledgers (DGCL §224) and Wyoming has DLT-friendly entity laws, but neither links to land title [I].
- **Foreign ownership.**
  - No general federal bar [I].
  - CFIUS reviews real-estate purchases near designated military installations [I].
  - AFIDA reporting applies to agricultural land [I].
  - A growing set of **state "foreign adversary" land laws** (e.g., Florida SB 264, Texas 2025) restrict purchases by certain nationals and may reach indirect interests [I].
  - **Admin control:** nationality screen per property state.
- **Operational risk from absentee ownership.** Detroit sued RealT (2 Jul 2025) over 408 properties, 53 "unquestionably harmful", citing unpaid taxes and blight fines and "165 different corporate entities" [C#16]. A court appointed an independent fiduciary in April 2026, and RealT announced **voluntary liquidation on 2 Jul 2026**: ~$140M raised, ~14–22k investors, payouts halted by end-2025 [C#17].

## 10. 2025–2026 developments (status on 23 Sep 2026)

| Date | Development | Status / relevance |
|---|---|---|
| 12 Mar 2025 | 506(c) minimum-investment verification NAL | In force; enables lighter 506(c) onboarding for tickets ≥$200k [C#8] |
| 18 Jul 2025 | **GENIUS Act** signed | Effective the earlier of 18 Jan 2027 or 120 days after final rules; OCC NPRM Mar 2026 [C#2][C#6] |
| 17 Jul 2025 | **CLARITY Act** passed House (294–134) | Senate Agriculture Committee 29 Jan 2026 (12–11); Senate Banking Committee 14 May 2026 (15–9); **cloture failed 49–50 on 15 Sep 2026**; enactment in 2026 unlikely [C#2][C#3][C#4]. It would not re-classify tokenized securities [I] |
| 31 Jul 2025 | SEC **Project Crypto** launched (Chair Atkins) | Agenda: token taxonomy, offering exemptions, "super-app" BD/ATS/custody modernisation [C#28] |
| 11 Dec 2025 | **DTC tokenisation NAL** | Pilot H1 2026 [C#7] |
| 11 Dec 2025 | INVEST Act passed House | Senate pending [C#9] |
| 17 Dec 2025 | T&M statement on BD custody of crypto-asset securities | In force (staff view) [C#19] |
| 28 Jan 2026 | **SEC staff Statement on Tokenized Securities** | Taxonomy; "format doesn't matter" [C#1] |
| Feb 2026 | Reg CF C&DIs (rolling cap, platform switching, stale financials) | In force [C#27] |
| 17 Mar 2026 | SEC/CFTC joint interpretation: crypto-asset taxonomy | "Digital securities" remain securities [C#14] |
| 19 Mar 2026 | FinCEN RRE rule vacated | No filing currently required [C#15] |
| 11/14 Aug 2026 | FinCEN BOI final rule | Domestic companies permanently exempt [C#22] |
| 18 Aug 2026 | **Proposed Regulation Crypto Assets** | Excludes tokenized securities; comment period 60 days after Federal Register publication [C#12][C#13] |
| 1 Sep 2026 | **Transfer-agent modernisation proposal** | Comments due 3 Nov 2026; DLT master file, TA-2 tokenisation fields [C#11] |
| 17 Sep 2026 | **Innovation exemption** (tokenized NMS stock venues) | Five years; NMS only [C#5] |
| 2026 | FINRA 2026 Digital-Asset Activity Information Request to members | Expect exam focus [C#29] |

## 11. Enforcement and litigation relevant to real-estate tokenisation

| Case | Date | Relevance |
|---|---|---|
| **City of Detroit v. RealToken (RealT)** (3rd Circuit Court, MI — i.e., Wayne County Circuit Court, as `02-platforms/us/04-realt.md` names it) | Filed 2 Jul 2025 | Nuisance/blight action over 408 tokenised rentals; shows landlord liability sits with the SPVs and sponsor, not token holders [C#16] |
| **RealT liquidation** | Fiduciary Apr 2026; voluntary liquidation 2 Jul 2026 | ~$140M, 14–22k holders, ~400 French investors pursuing claims. Shows cross-border investor-protection and operational-reserve risk [C#17] |
| **Texas SSB v. E-Estate Group** (Emergency C&D ENF-26-CDO-1896) | 1 Jun 2026 | "Tokenized real estate" from $10 with daily distributions; unregistered securities and dealers, MLM, misleading "backed by real estate" and "capital protected" claims [C#18] |
| SEC enforcement posture | FY2025–26 | Shifted from registration-only crypto cases to fraud [I]. Unregistered offerings of real-estate tokens to retail remain actionable at federal and state level [I] |

---

## Compliance obligations checklist

| Obligation | Legal basis | Who's responsible | Platform control needed |
|---|---|---|---|
| Choose and document an exemption per SPV | Securities Act §5; Reg D/S/A/CF | Issuer (SPV/sponsor) | Offering set-up wizard with exemption selector, caps and investor-type rules |
| Accredited verification (506(c)) | Rule 506(c)(2)(ii); NAL 12 Mar 2025 | Issuer (may outsource) | Accreditation queue: document upload, third-party letter, min-ticket ≥$200k/$1M path with reps, 90-day or annual expiry |
| Non-accredited investment limits (Reg CF / Reg A T2) | 17 CFR 227.100(a)(2); Reg A 251(d)(2)(i)(C) | Portal/BD + issuer | Income/NW self-cert, cross-offering 12-month limit tracker, calendar-year income field |
| Offering cap tracking | Reg CF $5M rolling; Reg A $20M/$75M | Issuer / portal | Raise-cap meter with rolling 12-month window per issuer |
| Form D / Form C / 1-A / 1-K / 1-SA / 1-U filings | Reg D Rule 503; Reg CF; Reg A | Issuer | Filing calendar and EDGAR task tracker; C-AR / 1-K reminders |
| Blue-sky notice filings | NSMIA; state law | Issuer | State filing tracker keyed to investor residence |
| AML programme, CIP, CDD beneficial owners | BSA; 31 CFR 1023.220 & 1010.230; FINRA 3310 | BD / portal (and platform if MSB) | KYC/KYB workflow, UBO ≥25% + control person capture, risk scoring, periodic refresh |
| OFAC sanctions screening (people and wallets) | IEEPA/OFAC regulations | Everyone (strict liability) | Real-time name and wallet screening, blocked-wallet list, re-screen on list updates |
| SAR / suspicious-activity escalation | 31 CFR 1023.320 | BD / MSB | Case-management queue, 30-day SAR clock, confidentiality controls |
| Travel rule ≥$3,000 | 31 CFR 1010.410(f) | Transmitting institution | Originator/beneficiary data capture on fiat/stablecoin transfers |
| Transfer-restriction enforcement (lock-ups, Reg S, Rule 144) | Securities Act; Reg S Rule 903; Rule 144 | Issuer / TA | Token compliance rules engine: whitelist, jurisdiction flags, lock-up end dates, per-holder restrictions |
| Holder-of-record count | Exchange Act §12(g) | Issuer | Holder-count dashboard per SPV with 500/2,000 alerts |
| Master securityholder file and reconciliation | Rule 17Ad; TA proposal Sept 2026 | Transfer agent | Golden-record ledger, daily on-chain reconciliation, forced-transfer / re-issue with dual approval, 6-year retention |
| Custody of tokens / keys | Rule 15c3-3 (BD); Rule 206(4)-2 (RIA) | BD / custodian / RIA | Custody-provider integration, key-management policies, DLT assessment record, BCP |
| Escrow of subscription funds | Reg CF 227.303(e); Rule 15c2-4 | Portal / BD | Escrow-status tracker, min-target logic, refund automation |
| Secondary-trading venue | Exchange Act §§5, 15; Reg ATS | ATS operator (BD) | Partner-ATS integration or order book gated to eligible holders; post-trade TA update |
| Marketing / communications review | Rule 10b-5; FINRA 2210; 506(b) ban; Reg S | Issuer / BD | Content-approval workflow, claims library, jurisdiction-based page gating |
| Reg BI / Form CRS (if BD recommends) | Exchange Act Rule 15l-1 | BD | Best-interest record per recommendation, CRS delivery log |
| Tax forms and withholding | IRC §§6045 (1099-DA/B), 1441/1446(a)/(f), 1445 (FIRPTA), FATCA | Broker / withholding agent / partnership | W-9/W-8 collection and expiry, withholding engine, K-1 / 1099-DA / 1042-S generation, cost-basis lots |
| Foreign-ownership screens | CFIUS real-estate regulations; state foreign-adversary land laws | Sponsor / SPV | Nationality plus property-location rule check before allocation |
| Property operations compliance | Local housing codes and taxes | SPV manager | Property-ops tracker: taxes, code violations, insurance, reserves (RealT lesson) |
| Stablecoin acceptance | GENIUS Act (from ~Jan 2027) | Platform | Stablecoin allow-list (permitted issuers only), issuer-status monitoring |

---

## Sources (accessed 23 Sep 2026)

1. SEC Divisions of CF/IM/TM, "Statement on Tokenized Securities", 28 Jan 2026 — https://www.sec.gov/newsroom/speeches-statements/corp-fin-statement-tokenized-securities-012826-statement-tokenized-securities
2. Latham & Watkins, "US Crypto Policy Tracker: Legislative Developments" (updated Aug 2026) — https://www.lw.com/en/us-crypto-policy-tracker/legislative-developments
3. CoinDesk, "U.S. Senate opens first stage of crypto Clarity Act voting…", 8 Aug 2026 — https://www.coindesk.com/policy/2026/08/08/u-s-senate-opens-first-stage-of-crypto-clarity-act-voting-to-give-bill-a-chance-next-month
4. The Crypto Times, "CLARITY Act Fails in Senate as Cloture Vote Falls Short of 60 Votes", 16 Sep 2026 — https://www.cryptotimes.io/2026/09/16/clarity-act-fails-in-senate-as-cloture-vote-falls-short-of-60-votes/ (CNBC original returned 403)
5. SEC Press Release 2026-90, "SEC Issues 'Innovation Exemption' to Facilitate the Trading of Tokenized NMS Stock", 17 Sep 2026 — https://www.sec.gov/newsroom/press-releases/2026-90-sec-issues-innovation-exemption-facilitate-trading-tokenized-nms-stock-request-comment
6. Chapman and Cutler, "GENIUS Act Rulemaking and Reporting Tracker" (as of 16 Jul 2026) — https://www.chapman.com/publication-genius-act-rulemaking-tracker ; OCC Bulletin 2026-3 NPRM (search result) — https://www.occ.gov/news-issuances/bulletins/2026/bulletin-2026-3.html
7. Carlton Fields, "SEC Staff No-Action Letter to DTC for Tokenization Services", Dec 2025 — https://www.carltonfields.com/insights/publications/2025/sec-staff-no-action-letter-to-dtc-for-tokenization-services ; letter: https://www.sec.gov/files/tm/no-action/dtc-nal-121125.pdf
8. Ropes & Gray, "SEC Issues No-Action Letter Clarifying Rule 506(c) Accredited Investor Verification", Mar 2025 — https://www.ropesgray.com/en/insights/alerts/2025/03/sec-issues-no-action-letter-clarifying-rule-506c-accredited-investor-verification
9. ABA Business Law Today, "House Passes Bipartisan Capital Formation Package: The INVEST Act", Jan 2026 — https://www.americanbar.org/groups/business_law/resources/business-law-today/2026-january/house-passes-bipartisan-capital-formation-package-invest-act/
10. Augment, "Accredited investor rule updates: what's changed and what to watch in 2026", Jun 2026 (secondary) — https://augment.market/pulse/accredited-investor-rule-updates
11. Skadden, "SEC Proposes Modernization of Transfer Agent Rules…", Sep 2026 — https://www.skadden.com/insights/publications/2026/09/sec-proposes-modernization-of-transfer-agent-rules
12. Greenberg Traurig, "SEC Proposes 'Regulation Crypto Assets'…", Aug 2026 — https://www.gtlaw.com/en/insights/2026/8/sec-proposes-regulation-crypto-assets-creating-tailored-crypto-offering-exemptions-and-investment-contract-safe-harbor
13. SEC Press Release 2026-76, "SEC Proposes New Regulation Crypto Assets", 18 Aug 2026 — https://www.sec.gov/newsroom/press-releases/2026-76-sec-proposes-new-regulation-crypto-assets
14. SEC Press Release 2026-30, "SEC Clarifies the Application of Federal Securities Laws to Crypto Assets", 17 Mar 2026 — https://www.sec.gov/newsroom/press-releases/2026-30-sec-clarifies-application-federal-securities-laws-crypto-assets
15. Burr & Forman, "FinCEN's Residential Real Estate Rule… PAUSED" (updated for 19 Mar 2026 vacatur) — https://www.burr.com/newsroom/articles/fincens-residential-real-estate-rule-applicable-to-non-financed-residential-entity-real-estate-transfers-paused
16. Outlier Media, "Detroit sues crypto landlord RealT over blight, tenant harm", Jul 2025 — https://outliermedia.org/realt-lawsuit-detroit-sues-crypto-landlord/
17. Crypto Briefing, "Crypto real estate empire collapses as $140 million tokenized property venture enters liquidation", Jul 2026 — https://cryptobriefing.com/realt-crypto-real-estate-collapse/
18. Texas State Securities Board, "…Orders E-Estate Group and Promoters to Halt Alleged Fraudulent Tokenized Real Estate Investment Scheme", 1 Jun 2026 — https://www.ssb.texas.gov/news-publications/texas-state-securities-board-orders-e-estate-group-and-promoters-halt-alleged
19. SEC Division of Trading and Markets, "Statement on the Custody of Crypto Asset Securities by Broker-Dealers", 17 Dec 2025 — https://www.sec.gov/newsroom/speeches-statements/trading-markets-121725-statement-custody-crypto-asset-securities-broker-dealers
20. IRS, "Instructions for Form 1099-DA (2026)" — https://www.irs.gov/instructions/i1099da
21. The Tax Adviser, "Navigating the Form 1099-DA reporting maze", Mar 2026 — https://www.thetaxadviser.com/issues/2026/mar/navigating-the-form-1099-da-reporting-maze/
22. Mayer Brown, "The Final Chapter: FinCEN Permanently Eliminates BOI Reporting Requirements for US Companies and US Persons", Aug 2026 — https://www.mayerbrown.com/en/insights/publications/2026/08/the-final-chapter-fincen-permanently-eliminates-boi-reporting-requirements-for-us-companies-and-us-persons
23. FirstMark Regulatory Solutions, "FINRA NMA: Timeline, Costs, and What to Expect" — https://firstmarksolutions.com/finra-new-member-application-nma-guide/
24. SEC, "Exempt Offerings" overview — https://www.sec.gov/resources-small-businesses/exempt-offerings
25. SEC, "Regulation Crowdfunding" — https://www.sec.gov/resources-small-businesses/exempt-offerings/regulation-crowdfunding
26. SEC, "Regulation A" — https://www.sec.gov/resources-small-businesses/exempt-offerings/regulation
27. Crowdfunding & FinTech Law Blog (M. Roderick), "SEC Issues New Reg CF Guidance", 17 Mar 2026 — https://crowdfundingattorney.com/2026/03/17/sec-issues-new-reg-cf-guidance/
28. A&O Shearman, "SEC Chairman Atkins announces 'Project Crypto'", Aug 2025 (search-result confirmation of date/title) — https://www.aoshearman.com/en/insights/ao-shearman-on-fintech-and-digital-assets/sec-chairman-atkins-announces-project-crypto
29. FTI Consulting, "What Broker-Dealers Need to Know About Digital Assets in 2026" — https://www.fticonsulting.com/insights/articles/what-broker-dealers-need-know-about-digital-assets-2026
