# Customer Attestation Credential · Design Deep-Dive

**Status**: Draft · 2026-04-24
**Parent spec**: [v0.1.md](./v0.1.md) · credential #16
**Classification under eIDAS**: EAA (private electronic attestation)

## Why this credential matters

From 2026 VC pain-point research:
- Customer existence is the most-faked claim by fraudulent founders ($175M case with fabricated customer list)
- VCs spend 4-8 hours per startup doing reference calls manually
- Current proxies (LinkedIn connections, G2 reviews, Featured Customers testimonials) are either weak (unsigned) or gameable

**Pitch line**: "Your customers sign that they're your customers. Cryptographically. No slide decks, no fake testimonials, no reference call marathons."

## Credential structure

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://schema.cloudpany.eu/founder-passport/v0.1"
  ],
  "type": ["VerifiableCredential", "CustomerAttestationCredential"],
  "issuer": {
    "id": "did:web:cloudpany.eu:customer:c_3f2a1e8b",
    "name": "Customer (pseudonymous)",
    "type": "CustomerEntity"
  },
  "validFrom": "2026-04-15T00:00:00Z",
  "validUntil": "2026-10-15T00:00:00Z",
  "credentialSubject": {
    "id": "did:web:cloudpany.eu:fp:fernando-gonzalez",
    "company": {
      "companyId": "did:web:cloudpany.eu:company:plot-twist-sl",
      "legalName": "Plot Twist SL"
    },
    "relationship": {
      "type": "paying-customer",
      "startDate": "2025-11-01",
      "endDate": null,
      "active": true,
      "mrrRange": "100-500",
      "currency": "EUR",
      "plan": "team",
      "billingVerified": true,
      "billingVerificationMethod": "psd2-transfer-match"
    },
    "satisfaction": {
      "nps": 9,
      "optionalComment": null,
      "wouldRecommend": true
    },
    "consentScope": {
      "visibleTo": ["verified-investors-only"],
      "revocableBy": "customer",
      "dataRetention": "P180D"
    }
  },
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-rdfc-2022",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "did:web:cloudpany.eu:customer:c_3f2a1e8b#key-1",
    "created": "2026-04-15T14:22:00Z",
    "proofValue": "z5XyKLm..."
  }
}
```

## Four issuance flows (ordered by complexity and fidelity)

### Flow 1 · Pseudonymous with billing match (MVP v1)

**Use case**: customers don't have DIDs yet (2026-2027 reality). Lowest friction.

**UX (founder)**:
1. Founder connects Stripe / Mollie / PSD2 bank via OAuth
2. Cloudpany lists active customers (from billing data)
3. Founder selects up to 10 customers to invite
4. Sends batched email invites

**UX (customer)**:
1. Receives email: "Fernando (Plot Twist) asks you to confirm you're a customer. 2 min."
2. Clicks link → Cloudpany landing (no account creation)
3. Sees exactly what will be signed (editable preview)
4. Signs with magic-link email (custodial DID, Cloudpany holds key)
5. Done

**Fidelity**: ⭐⭐⭐⭐ (high)
**Customer friction**: ⭐⭐ (low)
**Default for v0.1**

### Flow 2 · Sovereign DID attestation (v1.0+)

**Use case**: customer has own passport / W3C VC wallet (EUDI Wallet, Microsoft Authenticator, Trinsic, Dock).

Customer signs with their own private key. No custody by Cloudpany.

**Fidelity**: ⭐⭐⭐⭐⭐
**Customer friction**: ⭐⭐⭐ (requires wallet adoption)
**Timeline**: available when EUDI Wallet reaches critical mass (2027-2028)

### Flow 3 · Dual-sign with escrow (high-stakes deals)

**Use case**: marketplace transactions > €500K, or investor due diligence requiring maximum trust.

1. Customer signs attestation (signature A)
2. Escrow service (Mercury, Qonto, notario) independently verifies via bank records and co-signs (signature B)
3. Verifier sees A + B → near-absolute certainty

**Cost**: €20-50 per attestation, chargeable to founder
**Fidelity**: ⭐⭐⭐⭐⭐

### Flow 4 · Aggregate-only (privacy-maximal)

**Use case**: founder wants to display "47 attested customers" without revealing individual customer identities.

Uses BBS+ signature aggregation to prove N signatures exist without disclosing signer identities.

```json
{
  "type": ["VerifiableCredential", "AggregateCustomerAttestationCredential"],
  "credentialSubject": {
    "aggregatedAttestations": {
      "count": 47,
      "avgNPS": 8.4,
      "avgTenureDays": 420,
      "mrrSumRange": "15000-30000"
    },
    "individualSignatures": [
      "0x3a...", "0x7f...", "..."
    ],
    "proofMethod": "bbs-plus-aggregation"
  }
}
```

**Fidelity**: ⭐⭐⭐ (proves existence, not individual verification)
**Privacy**: ⭐⭐⭐⭐⭐

## Anti-gaming · 5-layer defense

The risk: a founder fabricates customers. Mitigations:

### Layer 1 · Billing match (blocking)
The attestation is only issued if the customer's email matches a real `customer_email` in the founder's Stripe / Mollie / PSD2 transaction history. No match = no attestation.

### Layer 2 · Customer ID cross-check
Payload includes hash of the `stripe_customer_id` / `mollie_customer_id` / PSD2 transaction reference. Cloudpany maintains the mapping under custody. If two "customers" resolve to the same billing ID → flag.

### Layer 3 · Rate limits + device fingerprinting
- Max 3 attestations from same IP subnet in 30 days
- Device fingerprint (IP + user agent + timezone) logged per attestation
- 5+ "customers" signing from same fingerprint → flag + notification to Cloudpany Trust team

### Layer 4 · Payment method uniqueness
Payment processors expose unique `payment_method_id` per card / SEPA / bank account. Two "customers" paying with same payment method → same person.

### Layer 5 · Tenure floor
Billing relationship must be ≥ 30 days old before attestation can be issued. Prevents "customer created yesterday to attest today."

## Consent model (GDPR + eIDAS)

### Customer's explicit consent

```
☑ Confirmo que soy cliente pagante de Plot Twist SL
☑ Autorizo que Cloudpany comparta esta confirmación con 
  investidores verificados (no general public)
☑ Entiendo que puedo revocar unilateralmente en cualquier momento
☑ Acepto Cloudpany Privacy Policy

Campos a revelar:
[x] MRR range (€100-500)
[x] Relationship start date (2025-11-01)
[ ] NPS score        ← opt-in
[ ] Written comment  ← opt-in
```

### Revocation

Customer can revoke at any time via:
1. Direct link (included in all emails they receive)
2. Personal dashboard at Cloudpany (if account created)
3. Email to `privacy@cloudpany.eu` with credential DID

Revocation updates StatusList2021 in <60s. All verifiers querying the credential see `revoked: true`.

### GDPR specifics

- **Controller/Processor**: Cloudpany is data processor for customer's attestation, founder is separate controller for their passport. Documented in DPA.
- **Retention**: 180 days default, renewable with re-consent.
- **Right to be forgotten**: revocation + purge of PII from issuance logs. Only hash + timestamp retained for audit.
- **Data minimization**: credential does NOT include customer email, full name, or address. Only DID + MRR range + tenure.

## Economic model

### Free for customers, paid for founders

| Tier | What | Price |
|---|---|---|
| Free | 5 attestations/month per passport | €0 |
| Cloudpany plan | Unlimited attestations + priority emission | included in €49/mo |
| Enterprise | API for automated requests + analytics dashboard | negotiate |
| Escrow (Flow 3) | Dual-signed with independent verifier | €20-50 per attestation |

**Why customer never pays**: would violate trust premise. Asking customers to pay to validate a founder feels extractive.

**Why rate limit on founder**: prevents spam (founders don't send to 500 random customers if it costs them).

## Integration with Cloudpany Market

Passport Score in listing UI incorporates attested customers as a high-weight signal:

```
Plot Twist · €50K
─────────────────────────────
Identity              ✓
Incorporation         ✓
Revenue (€10-25K MRR) ✓
Customers             ✓ 127 total, 12 attested, avg NPS 8.4
Churn                 ✓ 4.2% monthly, 108% NRR
Concentration         ✓ Top 5 = 18.1% (healthy)
IP                    ✓ 1 trademark EU
─────────────────────────────
Passport Score: 94/100
Saves buyer ~22h of DD
```

The qualitative difference between "seller says they have customers" and "12 out of 127 customers proactively signed cryptographic attestations confirming they pay" is massive.

## Open risks

1. **Low customer adoption**: if only 30% of invited customers sign, signal is weaker than expected.
   **Mitigation**: brutally simple UX + emotional copy ("help Fernando get his first investment")

2. **Paid attestations abuse**: founder pays 5 customers €50 each to sign. Legal but borderline.
   **Mitigation**: ToS prohibits explicitly; detection via Stripe patterns (customer who paid once then signed is more suspicious than customer paying 6 months)

3. **Privacy backlash**: customer perceives "Fernando shared my email with Cloudpany" as violation.
   **Mitigation**: email used ONLY for invite; Cloudpany doesn't store, market, or share. Made explicit in privacy policy + email notice.

4. **Competitive copy**: Affinity / Evertrace implement Stripe-direct attestation.
   **Mitigation**: this is a protocol, not a feature. If competitors implement, they use our schema. Network wins.

## MVP scope

For Cloudpany Market MVP v1:
- **Flow 1 only** (pseudonymous with billing match)
- **Layers 1, 2, 3, 5** of anti-gaming (skip Layer 4 payment-method check initially; add if abuse detected)
- **Free tier: 3 attestations/month** (not 5, to encourage Cloudpany plan uptake)
- **PSD2 + Stripe + Mollie** as supported billing sources
- **No aggregate flow, no dual-sign, no sovereign DID** — all deferred

Engineering estimate: 3-4 weeks with 1 engineer (frontend + backend + anti-gaming rules + email flow + signing infrastructure).
