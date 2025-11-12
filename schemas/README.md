# Synvya Nostr Schema and Example Reference

This directory defines the **Synvya reservation message schemas**, plus example payloads that conform to the Nostr-based reservation protocol.

The schemas formalize the JSON structure of each payload, enabling validation, linting, and auto-generation of SDK models.

For complete protocol documentation, see [`../rp.md`](../rp.md).

---

## 📚 Directory Structure

```
schemas/
├── examples/
│   ├── reservation.request.example.json
│   ├── reservation.response.confirmed.example.json
│   ├── reservation.response.declined.example.json
│   ├── reservation.response.cancelled.example.json
│   ├── reservation.modification.request.example.json
│   ├── reservation.modification.response.confirmed.example.json
│   └── reservation.modification.response.declined.example.json
├── reservation.request.schema.json
├── reservation.response.schema.json
├── reservation.modification.request.schema.json
├── reservation.modification.response.schema.json
├── transaction.attestation.schema.json
├── verified.review.schema.json
└── README.md
```

---

## 🧩 Schema Overview

We provide **one schema per event kind** that validates the complete event structure including tags.

| Schema File | Purpose |
|--------------|----------|
| `reservation.request.schema.json` | Full event schema for kind 9901 - unsigned rumor (includes `p` tag validation) |
| `reservation.response.schema.json` | Full event schema for kind 9902 - unsigned rumor (includes `p` and `e` tag validation) |
| `reservation.modification.request.schema.json` | Full event schema for kind 9903 - unsigned rumor (includes `p` and `e` tag validation) |
| `reservation.modification.response.schema.json` | Full event schema for kind 9904 - unsigned rumor (includes `p` and `e` tag validation) |
| `transaction.attestation.schema.json` | Full event schema for kind 9905 - signed event (includes `e` and `p` tag validation) |
| `verified.review.schema.json` | Full event schema for kind 31555 - signed public event (includes `d`, `e`, `rating`, and `verified` tag validation) |

### Tag Requirements

Schemas validate required tags:

- **Kind 9901**: Requires `p` tag (business public key)
- **Kind 9902**: Requires `p` tag (recipient) AND `e` tag with `["e", "<unsigned-9901-rumor-id>", "", "root"]`
- **Kind 9903**: Requires `p` tag (recipient) AND `e` tag with `["e", "<unsigned-9901-rumor-id>", "", "root"]`
- **Kind 9904**: Requires `p` tag (recipient) AND `e` tag with `["e", "<unsigned-9901-rumor-id>", "", "root"]`
- **Kind 9905**: Requires `e` tag (reservation thread ID) AND `p` tag (customer public key) - signed event
- **Kind 31555**: Requires `d` tag (business identifier), `e` tag (reservation thread ID), `rating` tag (primary rating with "thumb"), and `verified` tag (base64-encoded transaction attestation) - signed public event

---

## ✅ How to Validate

### CLI Validation

You can validate any example against its schema using a JSON Schema validator (e.g. `ajv-cli` or `python-jsonschema`):

#### Using `ajv-cli` (Node.js)
```bash
npx ajv validate -s reservation.request.schema.json -d examples/reservation.request.example.json
```

#### Using Python
```bash
python -m jsonschema -i examples/reservation.response.confirmed.example.json reservation.response.schema.json
```

All provided examples should pass validation successfully.

### Programmatic Validation (TypeScript)

The business client implements validation using `ajv` and `ajv-formats`:

```typescript
import Ajv from "ajv";
import addFormats from "ajv-formats";
import requestSchema from "./reservation.request.schema.json";
import responseSchema from "./reservation.response.schema.json";

// Initialize AJV with format support (for date-time validation)
const ajv = new Ajv({ allErrors: true });
addFormats(ajv);

// Compile schemas
const validateRequest = ajv.compile(requestSchema);
const validateResponse = ajv.compile(responseSchema);

// Validate a request
const requestPayload = {
  party_size: 2,
  iso_time: "2025-10-20T19:00:00-07:00",
  notes: "Window seat please"
};

const isValid = validateRequest(requestPayload);
if (!isValid) {
  console.error("Validation errors:", validateRequest.errors);
  // Errors include: field path, message, and failing value
}

// Validate a response
const responsePayload = {
  status: "confirmed",
  iso_time: "2025-10-20T19:00:00-07:00",
  message: "See you then!"
};

if (validateResponse(responsePayload)) {
  console.log("Valid response!");
}
```

**Full Implementation:**  
See [synvya-client-2](https://github.com/Synvya/synvya-client-2) `client/src/lib/reservationEvents.ts` for complete validation, encryption, and parsing logic.

**Key Features:**
- Type-safe validation with detailed error messages
- Date-time format validation via `ajv-formats`
- Automatic schema compilation at module load
- Integration with NIP-44 encryption for payload security
- Support for both unsigned rumor events (kinds 9901-9904) and signed events (kinds 9905, 31555)

---

## 🧾 License

© 2025 Synvya, Inc. — Schemas released under the MIT License. See [LICENSE](../LICENSE) for details.