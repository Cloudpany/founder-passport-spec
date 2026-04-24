# Founder Passport

**Open protocol for verifiable founder identity and startup credentials.**

Founder Passport is a [W3C Verifiable Credentials](https://www.w3.org/TR/vc-data-model-2.0/) based protocol that lets founders accumulate cryptographically-signed claims about their identity, their companies, revenue, team, customers, fundraising history — and lets investors, marketplaces, and AI agents consume that data directly via API.

Built EU-native: aligned with [eIDAS 2.0](https://eur-lex.europa.eu/eli/reg/2024/1183), [EUDI Wallet](https://ec.europa.eu/digital-building-blocks/sites/display/EUDIGITALIDENTITYWALLET), and [PSD2 Open Banking](https://www.eba.europa.eu/regulation-and-policy/payment-services-and-electronic-money).

## Why this exists

Investors spend 20+ hours per deal on mechanical verification — founder identity, company existence, revenue reality, previous round verification — before making a judgment call. Almost all of that work can be replaced with cryptographically-signed credentials from authoritative issuers.

Founder Passport is the protocol that makes that possible.

**For founders**: one portable, verifiable profile. Stop repeating yourself across investor data rooms.

**For investors / marketplaces**: instant pre-screening with verified signals, not self-reports. Most of the mechanical due diligence work becomes API calls.

**For AI agents**: structured, signed data consumable via MCP and REST — native to the agent era.

**For the EU ecosystem**: aligned with regulatory tailwinds (PSD2, eIDAS 2.0, EUDI Wallet), not another US-centric solution bolted onto Europe.

## Status

**Draft v0.1** · April 2026 · not production-ready

The specification is drafted. Reference implementation and canonical issuers are forthcoming. Breaking changes are possible before v1.0.

## Read the spec

- [**Specification v0.1**](./spec/v0.1.md) — main document: 19 credential types, JSON-LD schema, eIDAS mapping, PSD2 architecture, MVP scope
- [**Customer Attestation Flow**](./spec/customer-attestation-flow.md) — deep dive on the most novel credential: how customers cryptographically attest to being paying customers, with 5-layer anti-gaming

## Credential types at a glance

The 19 credential types in v0.1, organized in 4 tiers of importance:

### Foundational
- `IdentityCredential` — issued by EUDI Wallet, Cl@ve, or KYC providers
- `IncorporationCredential` — issued by registros mercantiles, Companies House, Stripe Atlas
- `RoleCredential` — founder/director relationship to a company
- `WorkHistoryCredential` — past employment

### Growth signals
- `RevenueCredential` — issued by PSD2 AISP (default), Mollie, Stripe, accounting software
- `FundraisingCredential` — round history
- `TeamSizeCredential` — issued by TGSS / URSSAF / Sozialversicherung (government) or payroll providers
- `ExitCredential` — liquidity events

### Product & community
- `ProductShipmentCredential` — public launch evidence
- `OpenSourceContributionCredential` — GitHub / GitLab contributions
- `AcceleratorCredential` — YC, Techstars, Antler, Seedcamp, EF
- `EducationCredential` — universities via Europass / OpenBadges

### Operational
- `ComplianceCredential` — SOC2, ISO27001, GDPR compliance
- `LegalGoodStandingCredential` — registry lookup
- `CreditRepaymentCredential` — lender repayment history

### Verification enhancers (novel to v0.1)
- `CustomerAttestationCredential` — customer signs that they are a paying customer
- `CustomerConcentrationCredential` — top-N customer breakdown from PSD2 / processor
- `ChurnMetricsCredential` — monthly churn and net revenue retention
- `IntellectualPropertyCredential` — EUIPO / OEPM / USPTO / WIPO registered IP

## Design principles

1. **Open protocol, freely implementable** — MIT licensed. Fork it, extend it, compete with us on implementation.
2. **EU-sovereign by default** — PSD2 + eIDAS 2.0 + EUDI Wallet from day one. Not a retrofit.
3. **Agent-native** — MCP server alongside REST API for AI consumption.
4. **W3C compliant** — VC Data Model 2.0, DIDs, OpenID4VCI/VP, SD-JWT. Never invents new standards.
5. **Holder-controlled** — founders own their DID and passport. Hosted services never own the data.

## Architecture at a glance

```
┌───────────────────────┐
│   Founder Passport    │  ← Container (VP-like), holder-controlled
│  did:web:...:fp:alice │
│                       │
│  ┌─────────────────┐  │
│  │ IdentityCred    │  │  ← signed by EUDI Wallet / Cl@ve
│  ├─────────────────┤  │
│  │ RevenueCred     │  │  ← signed by PSD2 AISP / Stripe
│  ├─────────────────┤  │
│  │ CustomerAttest1 │  │  ← signed by customer (pseudonymous)
│  │ CustomerAttest2 │  │
│  │ CustomerAttest3 │  │
│  ├─────────────────┤  │
│  │ ... (more)      │  │
│  └─────────────────┘  │
└───────────────────────┘
           │
           │ OpenID4VP / REST / MCP
           ▼
┌───────────────────────┐
│   Verifier            │  ← VC, marketplace, AI agent
│   Consumes passport   │
└───────────────────────┘
```

## Quick start for integrators

Reference implementation coming. In the meantime:

- Read [spec/v0.1.md](./spec/v0.1.md) top-to-bottom (~30 min).
- Check the canonical `IncorporationCredential` example in section 4 of the spec.
- If you're a potential **issuer** (payment processor, identity provider, registry): contact [hola@cloudpany.eu](mailto:hola@cloudpany.eu) about joining the certified issuer program.
- If you're a potential **verifier** (VC deal-flow tool, marketplace): the REST API and MCP server specs are coming; reach out early to shape them.

## Relationship to existing standards

Founder Passport is a schema and protocol built on top of ratified standards. It does not invent cryptography, identity, or transport — it specifies which credential types matter for the founder/startup domain.

Stack:
- **Data model**: [W3C VC Data Model 2.0](https://www.w3.org/TR/vc-data-model-2.0/)
- **Identifiers**: [W3C DIDs](https://www.w3.org/TR/did-core/) (`did:web` primary, `did:ebsi` for EU institutional)
- **Signatures**: [Data Integrity Proofs](https://www.w3.org/TR/vc-data-integrity/) with `eddsa-rdfc-2022`
- **Selective disclosure**: [SD-JWT VC](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/)
- **Issuance / presentation**: [OpenID4VCI](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html), [OpenID4VP](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html)
- **Revocation**: [StatusList2021](https://www.w3.org/TR/vc-status-list/)
- **EU compatibility**: [eIDAS 2.0 ARF](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework)

## Roadmap

- **v0.1 (now)** — draft specification, 19 credential types, EU-first architecture
- **v0.2 (Q3 2026)** — reference implementation, TrueLayer PSD2 bridge, Cl@ve identity integration
- **v0.3 (Q4 2026)** — production pilot with Cloudpany Market, first external issuers certified
- **v1.0 (2027)** — EBSI registration, EUDI Wallet certified, cross-EU coverage

## Contributing

This is an early-stage spec. High-leverage contributions right now:

- Review the spec for ambiguity, gaps, or compatibility issues with W3C / eIDAS standards
- Propose new credential types or modifications via GitHub Issues
- Implement experimental issuers or verifiers and share feedback
- Share pain points: are you a VC, founder, or marketplace frustrated with current verification workflows? [Tell us.](https://github.com/Cloudpany/founder-passport-spec/issues/new)

## License

[MIT License](./LICENSE) — use, modify, fork, commercialize freely.

## Credits

Authored by [Fernando González](https://github.com/fergp92) for [Cloudpany](https://cloudpany.eu).

Informed by:
- 2026 VC due diligence pain point research (see research synthesis inside Cloudpany internal docs)
- W3C Credentials Community Group ongoing work
- EU Digital Identity Wallet Consortium reference materials
- Interviews with early-stage founders and European VCs
