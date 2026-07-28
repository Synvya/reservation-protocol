Reservation Protocol

This repository contains the protocol specification and YAML schemas for NIP-RP (Reservation Protocol).

This NIP defines a protocol to manage reservations via Nostr. The term "reservations" is used as a broad term and could be applied to restaurants, hotels, or any other business offering appointments. This NIP also defines a business transaction attestation event associated with a succesfully completed reservation so that customers can issue a **verified business review**.

## Contents

- [`nostr-protocols/nips/rp.md`](./nostr-protocols/nips/rp.md) - Complete protocol specification
- [`nostrability/schemata/nips/nip-rp/`](./nostrability/schemata/nips/nip-rp/) - JSON Schema definitions (authored in YAML) for reservation message kinds (9901-9905)

## Usage

Schemas can be used for validation in any implementation. Each schema is located in its respective kind directory (e.g., `kind-9901/schema.yaml`).

## Reference Implementation

- See [Synvya Client](https://github.com/Synvya/client) for a reference implementation of the business side of the protocol.
- See [Synvya Diners](https://github.com/Synvya/diners) for a reference implementation of the consumer side of the protocol.
