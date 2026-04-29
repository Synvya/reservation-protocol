NIP-RP Extension: Offer Redemption
==================================

`draft` `optional`

This document extends [NIP-RP](./rp.md) with an event kind for **offer redemptions** — the act of a customer redeeming a previously-published offer at a business.

## Motivation

Businesses publish offers as `kind:30402` (parameterised replaceable event for individual offers) and `kind:30405` (replaceable for offer collections). Today there is no spec'd way to record that a specific customer redeemed a specific offer at a specific time. A signed redemption event provides:

1. A verifiable record for the customer's loyalty wallet or review tooling.
2. A deterministic count for analytics consumers (e.g. Synvya `systemtools` Pulse).
3. A reactive trigger for loyalty programs (e.g. award a stamp on each redemption).

## Event — `offer.redemption` — `kind:9906` (proposed)

The kind number is **proposed** pending NIP-RP acceptance and may be reassigned by the community. Implementers should treat the kind number as the only field at risk of churn before NIP-RP acceptance.

**Event Structure:**

```yaml
{
  "id": "<32-byte hex of event hash>",
  "pubkey": "<businessPublicKey>",            # the restaurant emits the redemption
  "created_at": <unix timestamp in seconds>,
  "kind": 9906,
  "tags": [
    ["p", "<customerPublicKey>"],              # the diner who redeemed
    ["a", "30402:<businessPublicKey>:<d-tag>"],# the redeemed offer (replaceable address); '30405:...' for collection-scope
    ["e", "<offer-event-id>"],                 # optional: pin to a specific publish of the offer
    ["e", "<reservation-thread-id>", "", "reservation"], # optional: NIP-RP reservation thread (kind:9901 id) if the redemption happened during a reservation
    ["redeemed_at", "<unix timestamp seconds>"],
    ["value", "<optional decimal>"],           # economic value of the redemption (e.g. discount applied)
    ["currency", "<optional ISO 4217 code>"]   # required if 'value' present
  ],
  "content": "<optional human-readable note>",
  "sig": "<schnorr signature per NIP-01>"
}
```

The event is **signed** by the business. Recipients verify the signature and that `event.pubkey` matches the offer's publishing business pubkey (the second segment of the `a` tag).

## Issuance Rules

- A redemption event MUST be issued by the business that published the original offer (`event.pubkey == a-tag.businessPublicKey`).
- A redemption event MUST reference the offer being redeemed via an `a` tag in the form `30402:<businessPublicKey>:<d-tag>` (or `30405:...` for collection-scope redemptions).
- A redemption event SHOULD include a `p` tag identifying the customer pubkey when the customer's identity is known. If the redemption is anonymous (e.g. cash-paid walk-in claiming a public offer), the `p` tag MAY be omitted.
- The event SHOULD be published to relays the business writes to per NIP-65, and MAY additionally be wrapped via NIP-17 private direct message to the customer if the redemption is privacy-sensitive (in which case the public publish carries an aggregate-only payload — see Privacy Considerations).
- Multiple redemption events MAY reference the same offer. An offer that permits multiple redemptions per customer or across customers produces one redemption event per redemption.

## Counting Semantics

For analytics aggregation:

- **# offers redeemed** in a scope (e.g. for a given week, restaurant, or customer) is the count of distinct redemption event ids in scope.
- An offer redeemed twice produces two redemption events; both count.
- A redemption event re-broadcast to additional relays does **not** produce a new id and counts once.
- Aggregators MUST de-duplicate by event id before counting.

## Privacy Considerations

When the redemption is sent as a NIP-17 private direct message (gift-wrapped), only the business and customer can decrypt the payload. Servers and relays wishing to count redemptions must therefore either be participants in the message exchange (e.g. an analytics service holding a relay key) or rely on businesses publishing a separate public signal.

For Synvya v1, businesses publish redemption events to public relays the Synvya `server` subscribes to, so server-side projection is straightforward. A future revision MAY define a privacy-preserving count attestation that lets aggregators count without learning customer identities.

## Open Questions

- **Kind number assignment.** `kind:9906` is proposed; the NIP-RP community may pick a different number when the spec is accepted. Implementers should treat the kind number as the only field at risk of churn before NIP-RP acceptance.
- **Linkage to loyalty point spend.** A redemption event may or may not coincide with a loyalty-point spend. If it does, an additional `["loyalty", "<points>"]` tag is reasonable; spec'd later.
- **Idempotency.** Whether a business should be allowed to re-emit a redemption event for the same redemption (e.g. for re-publishing) is not specified. Aggregators de-duplicate by event id; in-protocol idempotency is left to a future revision.
- **Anonymous redemptions.** When the `p` tag is omitted, how should distinct-diner-reach metrics treat the redemption? v1 analytics excludes anonymous redemptions from distinct-diner counts but includes them in raw redemption counts.
