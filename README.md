# NIP-RP: Reservation Protocol

This repository contains the protocol specification and JSON schemas for NIP-RP (Reservation Protocol).

This NIP defines a protocol to manage reservations via Nostr. The term "reservations" is used as a broad term and could be applied to restaurants, hotels, or any other business offering appointments. This NIP also defines a business transaction attestation event associated with a succesfully completed reservation so that customers can issue a **verified business review**.

## Contents

- [`rp.md`](./rp.md) - Complete protocol specification
- [`schemas/`](./schemas/) - JSON Schema definitions for reservation message kinds (9901-9904)

## Usage

Schemas can be used for validation in any implementation. See [`schemas/README.md`](./schemas/README.md) for details.

## Reference Implementation

- See [Synvya Client](https://github.com/Synvya/client) for a reference implementation of the business side of the protocol.
- See [AI Concierge](https://github.com/Synvya/ai-concierge) for a reference implementation of the consumer side of the protocol.
