# Reservation Protocol (NIP-RP) — archived

**This repository is archived.** NIP-RP has been superseded by the **Open Markets Bookings** proposal, which Synvya is submitting to the Open Markets Foundation specification. Bookings covers the same flows (request, confirm, decline, cancel, modify) plus availability queries and payments, on a single private message kind (`1331`) instead of kinds 9901–9905.

## Where to go

- **The proposal:** `proposals/bookings.md` in [OpenMarketsFoundation/specification](https://github.com/OpenMarketsFoundation/specification) (pull request pending). Until it is open, the current draft is at [Synvya/docs → open-markets-bookings.md](https://github.com/Synvya/docs/blob/alejandro-dev/06-domains-and-services/specs/open-markets-bookings.md).
- **Migrating an implementation from NIP-RP:** [Synvya/docs → migration-nip-rp-to-bookings.md](https://github.com/Synvya/docs/blob/alejandro-dev/06-domains-and-services/specs/migration-nip-rp-to-bookings.md) maps every NIP-RP kind, tag, and field to its Bookings equivalent.
- **Verified reviews** (formerly `kind:9905` / `kind:31555`): will be proposed separately to Open Markets as a lane-neutral transaction attestation.
- **Offer redemption** (`redemption-event.md`, `kind:9906`): not part of bookings; kept here only as history.

## What is here

The last NIP-RP specification and its JSON Schemas remain in this repository unchanged, for implementations that still read the legacy kinds during their transition:

- [`nostr-protocols/nips/rp.md`](./nostr-protocols/nips/rp.md) — the NIP-RP specification (kinds 9901–9905, `kind:31555` verified reviews)
- [`nostrability/schemata/nips/nip-rp/`](./nostrability/schemata/nips/nip-rp/) — JSON Schemas (YAML) for kinds 9901–9905 and the `verified` tag

No further changes will be made here.
