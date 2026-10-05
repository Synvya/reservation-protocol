# Reservation Protocol (NIP-RP) — archived

**This repository is archived.** NIP-RP has been superseded by the **Open Markets Bookings** proposal, which Synvya has submitted to the Open Markets Foundation specification. Bookings covers the same flows (request, confirm, decline, cancel, modify) plus availability queries and payments, on a single private message kind (`1331`) instead of kinds 9901–9905.

## Where to go

- **The proposal:** [OpenMarketsFoundation/specification#17](https://github.com/OpenMarketsFoundation/specification/pull/17) adds `proposals/bookings.md` and its schemas under `proposals/bookings/schemata/`.
- **Verified reviews** (formerly `kind:9905` / `kind:31555`): will be proposed separately to Open Markets as a lane-neutral transaction attestation.
- **Offer redemption** (`redemption-event.md`, `kind:9906`): not part of bookings; kept here only as history.

## What is here

The last NIP-RP specification and its JSON Schemas remain in this repository unchanged, for implementations that still read the legacy kinds during their transition:

- [`nostr-protocols/nips/rp.md`](./nostr-protocols/nips/rp.md) — the NIP-RP specification (kinds 9901–9905, `kind:31555` verified reviews)
- [`nostrability/schemata/nips/nip-rp/`](./nostrability/schemata/nips/nip-rp/) — JSON Schemas (YAML) for kinds 9901–9905 and the `verified` tag

No further changes will be made here.
