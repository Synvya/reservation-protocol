# Synvya Nostr Schema and Example Reference

This directory defines the **Synvya reservation message schemas**, plus example payloads that conform to the Nostr-based restaurant reservation protocol.

The schemas formalize the JSON structure of each payload, enabling validation, linting, and auto-generation of SDK models.

For complete protocol documentation, see [`../NIP-RR.md`](../NIP-RR.md).

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
└── reservation.modification.response.schema.json
```

---

## 🧩 Schema Overview

| Schema File | Purpose |
|--------------|----------|
| `reservation.request.schema.json` | Schema for reservation.request message (kind 9901) |
| `reservation.response.schema.json` | Schema for reservation.response message (kind 9902) |
| `reservation.modification.request.schema.json` | Schema for reservation.modification.request message (kind 9903) |
| `reservation.modification.response.schema.json` | Schema for reservation.modification.response message (kind 9904) |

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

---

## 🧠 Design Notes

- Rumor `content` is **plain text JSON** (not encrypted).
- Encryption occurs at the **seal** (kind 13) and **gift wrap** (kind 1059) layers using NIP-44.
- All inter-party messages **must** use **NIP-59 Gift Wrap** for privacy.
- **Replaceable events** follow **NIP-01** and use `a` tags, not deprecated `d` tags.
- **Threading** (root/reply) follows **NIP-10**.
- Light **Proof of Work (NIP-13)** is recommended for anti-spam.

---

## 🪴 Extending the Model

To introduce new message types (e.g., `order.request`, `order.response`):

1. Assign a kind in the 9905–9999 range.
2. Create a new JSON Schema in `/schemas/`.
3. Include a valid example in `/schemas/examples/`.
4. Update `README.md` and your SDK registry.

---

## 🧾 License

© 2025 Synvya, Inc. — Schemas released under the MIT License. See [LICENSE](../LICENSE) for details.