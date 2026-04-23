# Build & Usage — Reservation Protocol (NIP-RP)

This repository contains the specification for the Synvya Nostr Reservation Protocol (NIP-RP). It is a documentation-only repository and does not require a compilation step.

## Repository Structure

- **`nostr-protocols/nips/`**: Contains the draft NIP (Nostr Improvement Possibility) specifications defining the message flow and event types for restaurant reservations.
- **`nostrability/schemata/`**: Contains JSON Schema definitions for validating reservation events (Kinds 9901–9905).

## Usage

### 1. Protocol Reference
The primary specification defining the reservation lifecycle (Request -> Response -> Modification) is located in the `nostr-protocols/nips/` directory.

### 2. Event Validation
If you are implementing a client or server that interacts with NIP-RP, you can use the JSON Schemata in `nostrability/schemata/` to validate events before signing or processing them.

### 3. Navigation
This repository is intended to be used as a reference for:
- Building `diners` app components that request reservations.
- Building `server` components that route and process reservation state.
- Building `client` (business) dashboards that respond to reservation requests.
