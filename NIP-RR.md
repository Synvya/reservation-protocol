NIP-RR
======

Restaurant Reservation and Verified Reviews Protocol
----------------------------------------------------

`draft` `optional`

This NIP defines a protocol to manage restaurant reservations and verified reviews via Nostr. 

The reservation process uses 4 different messages, each with its own kind, that are sent unsigned, sealed, and gift wrapped between the parties to maintain the privacy of the customer. Only the customer and the restaurant are aware of the reservation.

The review flow uses 2 different messages, each with its own kind, and it ensures that only verified customers can issue a review.

The customer's identify, reservation date, and time becomes public when issuing a review. 

## Overview

The Restaurant Reservation and Verified Reviews Protocol uses four event kinds to support a complete negotiation flow and 2 event kinds to support verified reviews:

- `reservation.request` - `kind:9901`: Initial message sent  by the customer to make a reservation request
- `reservation.response` - `kind:9902`: Message sent by the restaurant or the customer to finalize the exchange of messages with status `confirmed`, `declined`, or `cancelled`
- `reservation.modification.request` - `kind:9903`: Message sent to modify a firm reservation or a reservation under negotiation
- `reservation.modification.response` - `kind:9904`: Message sent in response to a `reservation.modification.request`
- `visit.attestation` - `kind:9905`: Message sent by the restaurant to attest that a specific customer visited with a given reservation
- `verified.review` - `kind:9906`: Public review written by the customer and optionally marked as “verified” by embedding a prior `kind:9905` visit attestation


Clients must support `kind:9901` and `kind:9902` messages. Support for `kind:9903`, `kind:9904`, `kind:9905`, and `kind:9906` is optional but strongly recommended.

## Kind Definitions

### Reservation Request - Kind:9901

**Rumor Event Structure:**
```yaml
{
  "id": "<32-byte hex of unsigned event hash>",
  "pubkey": "<senderPublicKey>",
  "created_at": <unix timestamp in seconds>,
  "kind": 9901,
  "tags": [
    ["p", "<restaurantPublicKey>", "<relayUrl>"]
    // Additional tags MAY be included
  ],
  "content": "<content-in-plain-text>"
  // Note: No signature field - this is an unsigned rumor
}
```

**Content Structure:**
```yaml
{
  "party_size": <integer between 1 and 20>,
  "iso_time": "<ISO8601 datetime with timezone>",
  "notes": "<optional string, max 2000 chars>",
  "contact": {
    "name": "<optional string, max 200 chars>",
    "phone": "<optional string, max 64 chars>",
    "email": "<optional email>"
  },
  "constraints": {
    "earliest_iso_time": "<optional ISO8601 datetime>",
    "latest_iso_time": "<optional ISO8601 datetime>",
  }
}
```

**Required Fields:**
- `party_size`: Integer between 1 and 20
- `iso_time`: ISO8601 datetime string with timezone offset

**Optional Fields:**
- `notes`: Additional notes or special requests (max 2000 characters)
- `contact`: Contact information object
- `constraints`: Preferences for negotiation

---

### Reservation Response - Kind:9902

**Rumor Event Structure:**
```yaml
{
  "id": "<32-byte hex of unsigned event hash>",
  "pubkey": "<senderPublicKey>",
  "created_at": <unix timestamp in seconds>,
  "kind": 9902,
  "tags": [
    ["p", "<recipientPublicKey>", "<relay-url>"],
    ["e", "<unsigned-9901-rumor-id>", "", "root"]
    // Additional tags MAY be included
  ],
  "content": "<content-in-plain-text>"
  // Note: No signature field - this is an unsigned rumor
}
```

**Content Structure:**
```yaml
{
  "status": "<confirmed|declined|cancelled>",
  "iso_time": "<ISO8601 datetime with timezone> | null",
  "message": "<optional string, max 2000 chars>",
  "table": "<optional string | null>",
}
```

**Required Fields:**
- `status`: One of `"confirmed"`, `"declined"`, or `"cancelled"`
- `iso_time`: ISO8601 datetime string with timezone offset

**Optional Fields:**
- `message`: Human-readable message to the customer
- `table`: Table identifier (e.g., "A5", "12", "Patio 3")

**Threading:**
- MUST include an `e` tag with `["e", "<unsigned-9901-rumor-id>", "", "root"]` referencing the unsigned rumor ID of the original request (kind:9901).

---

### Reservation Modification Request - Kind:9903

**Rumor Event Structure:**
```yaml
{
  "id": "<32-byte hex of unsigned event hash>",
  "pubkey": "<senderPubKey>",
  "created_at": <unix timestamp in seconds>,
  "kind": 9903,
  "tags": [
    ["p", "<recipientPublicKey>", "<relay-url>"],
    ["e", "<unsigned-9901-rumor-id>", "", "root"],
    // Additional tags MAY be included
  ],
  "content": "<content-in-plain-text>"
  // Note: No signature field - this is an unsigned rumor
}
```

**Content Structure:**
```yaml
{
  "party_size": <integer between 1 and 20>,
  "iso_time": "<ISO8601 datetime with timezone>",
  "notes": "<optional string, max 2000 chars>",
  "contact": {
    "name": "<optional string, max 200 chars>",
    "phone": "<optional string, max 64 chars>",
    "email": "<optional email>"
  },
  "constraints": {
    "earliest_iso_time": "<optional ISO8601 datetime>",
    "latest_iso_time": "<optional ISO8601 datetime>",
  }
}
```

**Required Fields:**
- `party_size`: Integer between 1 and 20
- `iso_time`: ISO8601 datetime string with timezone offset

**Optional Fields:**
- `notes`: Additional notes or special requests (max 2000 characters)
- `contact`: Contact information object
- `constraints`: Preferences for negotiation

**Threading:**
- MUST include an `e` tags with `["e", "<unsigned-9901-rumor-id>", "", "root"]` referencing the unsigned rumor ID of the original request.

---

### Reservation Modification Response - Kind:9904

**Rumor Event Structure:**
```yaml
{
  "id": "<32-byte hex of unsigned event hash>",
  "pubkey": "<senderPublicKey>",
  "created_at": <unix timestamp in seconds>,
  "kind": 9904,
  "tags": [
    ["p", "<recipientPublicKey>", "<relay-url>"],
    ["e", "<unsigned-9901-rumor-id>", "", "root"],
    // Additional tags MAY be included
  ],
  "content": "<content-in-plain-text>"
  // Note: No signature field - this is an unsigned rumor
}
```

**Content Structure:**
```yaml
{
  "status": "<confirmed|declined>",
  "iso_time": "<ISO8601 datetime with timezone> | null",
  "message": "<optional string, max 2000 chars>",
}
```

**Required Fields:**
- `status`: One of `"confirmed"` or `"declined"`
- `iso_time`: ISO8601 datetime string with timezone offset

**Optional Fields:**
- `message`: Human-readable message to the customer


**Threading:**
- MUST include an `e` tags with `["e", "<unsigned-9901-rumor-id>", "", "root"]` referencing the unsigned rumor ID of the original request.

---

### Visit Attestation – Kind:9905

`kind:9905` is a **signed** nostr event created by the restaurant to attest that a visit occurred under a given reservation thread. The event is created by the restaurant system when a reservation is fulfilled (for example, when the check is closed) and sent privately to the customer using an encrypted direct message (`kind:14`) and is **never** published to public relays to maintain the restaurant visit private until the customer decides to publish a review.

The event is intended as a visit token that the customer MAY later embed in a public review. If the customer does not publish a review, then the restaurant visit will remain private. 

**Event Structure:**
```yaml
{
  "id": "<32-byte hex of event hash>",
  "pubkey": "<restaurantPublicKey>",
  "created_at": <unix timestamp in seconds>,
  "kind": 9905,
  "tags": [
    ["rr", "<unsigned-9901-rumor-id>"],  # Reservation thread id 
    ["p", "<customerPublicKey>"]
  ],
  "content": "",
  "sig": "<signed by restaurantPrivateKey>"
}
```

**Required Tags:**
	•	`["rr", "<unsigned-9901-rumor-id>"]`: MUST reference the unsigned rumor ID of the original reservation.request `kind:9901` message. This value is the reservation thread id.
	•	`["p", "<customerPublicKey>"]`: MUST reference the customer associated with the visit.

---

### Verified Review – Kind:9906

`kind:9906` is a **signed** public event created by the customer to publish a review for a past visit. Reviews are “verified” by embedding the associated `visit attestation` `kind:9905` event received directly from the restaurant via direct message.

**Event Structure:**
```yaml
{
  "id": "<32-byte hex of event hash>",
  "pubkey": "<customerPublicKey>",
  "created_at": <unix timestamp in seconds>,
  "kind": 9906,
  "tags": [
    ["p", "<restaurantPublicKey>"],
    ["rr", "<unsigned-9901-rumor-id>"],          # Reservation thread id 
    ["rating", "<1-5>"],
    ["verified", "<base64-encoded-visit-9905>"] 
    # Additional tags MAY be included
  ],
  "content": "<free-form review text>",
  "sig": "<signed by customerPrivateKey>"
}
```

**Required Tags**:
	•	`["p", "<restaurantPublicKey>"]`: MUST specify the restaurant being reviewed.
	•	`["rr", "<unsigned-9901-rumor-id>"]`: MUST reference the same reservation thread id T used during the reservation flow.
	•	`["rating", "<1-5>"]`: Numeric rating for the visit. The rating scale MUST be between 1 and 5.
	•	`["verified", "<base64-encoded-visit-9905>"]`: Tag used to mark a review as a verified visit. The value MUST be a base64 encoding of the full serialized kind:9905 visit attestation event.

A client MAY treat a `verified.review` `kind:9906` event as a **verified review** if all of the following conditions are met:
  1.	The event has a `["verified", "<payload>"]` tag.
	2.	Decoding `<payload>` from base64 yields a valid nostr `visit.attestation` `kind:9905` event with the following properties:
	  •	`pubkey` matches the restaurant public key indicated by the `verified.review` `kind:9906` `["p", "<restaurantPublicKey>"]` tag
	  •	There is a `["p", "<customerPublicKey>"]` tag where `<customerPublicKey>` matches the `pubkey` field of the `verified.review` `kind:9906` event
	  •	There is a `["rr", "<unsigned-9901-rumor-id>"]` tag matching the `["rr", "<unsigned-9901-rumor-id>"]` tag of the `verified.review` `kind:9906` event
	  •	The `visit.attestation` `kind:9905` event has a valid signature computed according to NIP-01.

If verification succeeds, clients MAY label the review as a “verified visit”. If verification fails, clients MUST NOT display the review. 


---

## Encryption, Wrapping, and Threading

All reservation messages MUST follow the [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) Gift Wrap protocol:

1. **Create Rumor**: Build an unsigned event of the appropriate kind (9901, 9902, 9903, or 9904) with plain text content
2. **Create Seal**: Wrap the rumor in a `kind:13` seal event, encrypted with [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md)
3. **Create Gift Wrap**: Wrap the seal in a `kind:1059` gift wrap event, addressed to the recipient via `p` tag


**Gift Wrap**
```yaml
{
  "id": "<usual hash>",
  "pubkey": randomPublicKey,
  "created_at": randomTimeUpTo2DaysInThePast(),
  "kind": 1059, // gift wrap
  "tags": [
    ["p", receiverPublicKey, "<relay-url>"] // receiver
  ],
  "content": nip44Encrypt(
    {
      "id": "<usual hash>",
      "pubkey": senderPublicKey,
      "created_at": randomTimeUpTo2DaysInThePast(),
      "kind": 13, // seal
      "tags": [], // no tags
      "content": nip44Encrypt(unsignedKind990x, senderPrivateKey, receiverPublicKey),
      "sig": "<signed by senderPrivateKey>"
    },
    randomPrivateKey, receiverPublicKey
  ),
  "sig": "<signed by randomPrivateKey>"
}
```

The `visit.attestation` `kind:9905` message is sent from the restaurant to the customer as the content of a direct message `kind:14`. 

{
  "kind": 14,
  "pubkey": "<restaurantPublicKey>",
  "created_at": <unix timestamp in seconds>,
  "tags": [
    ["p", "<customerPublicKey>"],
    ["e", "<unsigned-9901-rumor-id>", "", "root"]
  ],
  "content": "<plain-text serialized and signed kind-9905 event>"
  // no sig field – this is a rumor, encrypted and transported via NIP-59
}

## Protocol Flow

### Simple Reservation Request 
1. Customer sends `reservation.request` `kind:9901` message to the restaurant
2. Restaurant responds with `reservation.response` `kind:9902` message with `"status":"confirmed"` or `"status":"declined"`
3. Message exchange ends

### Reservation Request With Restaurant Suggesting Alternative Time
1. Customer sends `reservation.request` `kind:9901` message to the restaurant
2. Restaurant sends with `reservation.modification.request` `kind:9903` message with proposed new time
3. Customer responds with `reservation.modification.response` `kind:9904` message with `"status":"confirmed"` or `"status":"declined"`
4. Restaurant responds with `reservation.response` `kind:9902` message with matching status `confirmed` or `declined`
5. Message exchange ends

### Succesful Reservation Modification by Customer
1. Customer sends `reservation.modification.request` `kind:9903` message with proposed new time
2. Restaurant sends `reservation.modification.response` `kind:9904` message with `"status":"confirmed"` to indicate availability for the new time
3. Customer sends `reservation.response` `kind:9902` message with status `confirmed`
4. Message exchange ends

Note: *This flow assumes that there is an existing confirmed reservation initiated by a `reservation.request` `kind:9901` message to the restaurant. All messages should include the `"e"` tag with the rumor ID of the original `reservation.request` message to match the modification to the original reservation.*

### Unsuccesful Reservation Modification by Customer
1. Customer sends `reservation.modification.request` `kind:9903` message with proposed new time
2. Restaurant sends `reservation.modification.response` `kind:9904` message with `"status":"declined"` to indicate lack of availability for the new time
3. Customer sends `reservation.response` `kind:9902` message with original time and status `"status":"confirmed"` to maintain original reservation or `"status":"cancelled"` to cancel the original reservation
4. Message exchange ends

Note: *This flow assumes that there is an existing confirmed reservation initiated by a `reservation.request` `kind:9901` message to the restaurant. All messages should include the `"e"` tag with the rumor ID of the original `reservation.request` message to match the modification to the original reservation.*

### Reservation Cancellation Initated by the Restaurant
1. Restaurant sends `reservation.response` `kind:9902` message with `"status":"cancelled"` to the customer. Including a message is highly encouraged. 
2. Message exchange ends

No further action is expected from the customer. 

Note: *This flow assumes that there is an existing confirmed reservation initiated by a `reservation.request` `kind:9901` message to the restaurant. All messages should include the `"e"` tag with the rumor ID of the original `reservation.request` message to match the modification to the original reservation.*

### Reservation Cancellation Initated by the Customer
1. Customer sends `reservation.response` `kind:9902` message with `"status":"cancelled"` to the restaurant. Including a message is highly encouraged. 
2. Message exchange ends

No further action is expected from the restaurant.

Note: *This flow assumes that there is an existing confirmed reservation initiated by a `reservation.request` `kind:9901` message to the restaurant. All messages should include the `"e"` tag with the rumor ID of the original `reservation.request` message to match the modification to the original reservation.*

### Visit Attestation Issued By The Restaurant
1. After the customer has dined and the visit is considered fulfilled, the restaurant marks the reservation as completed.
2. A `visit.attestation` `kind:9905` message is automatically sent to the customer via a `kind:14` direct message. The `visit.attestation` `kind:9905` message is **never** published to a relay.
3. Message exchange ends

No further action is expected from the restaurant or the customer. 

### Customer Elects to Not Publish a Review
1. Customer receives the `visit.attestation` `kind:9905` message from the restaurant.
2. Customer chooses to not issue a review
3. Message exchange ends

The fact that the customer dined at the restaurant remains a private fact known only to the customer and the restaurant. 

### Customer Elects to Publish a Review
1. Customer receives the `visit.attestation` `kind:9905` message from the restaurant.
2. Customer wrrites a review and publishes it as a `verified.review` `kind:9906` event that includes the `["verified", "<base64-encoded-visit-9905>"]` tag 
3. Message exchange ends

The fact that the customer dined at the restaurant becomes a public fact. 


---


## JSON Schema Validation

Clients MUST validate payloads against JSON schemas before processing:

- Kind 9901: Validate against `reservation.request.schema.json`
- Kind 9902: Validate against `reservation.response.schema.json`
- Kind 9903: Validate against `reservation.modification.request.schema.json`
- Kind 9904: Validate against `reservation.modification.response.schema.json`
- Kind 9905: Validate against `visit.attestation.schema.json`
- Kind 9906: Validate against `review.schema.json`

Invalid payloads MUST be rejected and not processed further.

### Encryption Details

- **Content Encryption**: The JSON payload MUST be encrypted using [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) version 2 encryption
- **Seal Encryption**: The serialized rumor JSON MUST be encrypted using [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) version 2 encryption with a conversation key derived from the sender's private key and recipient's public key
- **Gift Wrap Encryption**: The serialized seal JSON MUST be encrypted using [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) version 2 encryption with a conversation key derived from a random ephemeral private key and recipient's public key

### Self CC Pattern

Following [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) and [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md), senders SHOULD publish gift wraps to both the recipient AND themselves (self-addressed). This ensures:
- Senders can retrieve their own messages across devices
- Full conversation history is recoverable with the sender's private key
- Each recipient gets a separately encrypted gift wrap

### Timestamp Randomization

Per [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md), `created_at` timestamps SHOULD be randomized up to 2 days in the past for both seal and gift wrap events to prevent metadata correlation attacks.

### Threading

All messages in a reservation conversation after the original `reservation.request` `kind:9901` message MUST be threaded using the **unsigned rumor ID** of the original request as the root.

---

## Restaurant Discovery 

Restaurants MUST advertise their capability to handle reservation messages using [NIP-89](https://github.com/nostr-protocol/nips/blob/master/89.md) Application Handlers.

### Handler Information Event (kind:31990)

Restaurants MUST publish a `kind:31990` handler information event that declares support for all reservation message kinds:

```yaml
{
  "kind": 31990,
  "pubkey": "<restaurantPublicKey>",
  "tags": [
    ["d", "synvya-restaurants-v1.0"],
    ["k", "9901"],
    ["k", "9902"],
    ["k", "9903"],
    ["k", "9904"]
  ],
  "content": ""
}
```

- The `d` tag MUST use the identifier `"synvya-restaurants-v1.0"`
- The `k` tags MUST include all four supported kinds: `9901`, `9902`, `9903`, and `9904`
- The `content` field MAY be empty (clients will use the restaurant's `kind:0` profile for display)

### Handler Recommendation Events (kind:31989)

Restaurants MUST publish four `kind:31989` handler recommendation events, one for each supported event kind:

**For kind:9901 (reservation.request):**
```yaml
{
  "kind": 31989,
  "pubkey": "<restaurantPublicKey>",
  "tags": [
    ["d", "9901"],
    ["a", "31990:<restaurantPublicKey>:synvya-restaurants-v1.0", "<relayUrl>", "all"]
  ],
  "content": ""
}
```

**For kind:9902 (reservation.response):**
```yaml
{
  "kind": 31989,
  "pubkey": "<restaurantPublicKey>",
  "tags": [
    ["d", "9902"],
    ["a", "31990:<restaurantPublicKey>:synvya-restaurants-v1.0", "<relayUrl>", "all"]
  ],
  "content": ""
}
```

**For kind:9903 (reservation.modification.request):**
```yaml
{
  "kind": 31989,
  "pubkey": "<restaurantPublicKey>",
  "tags": [
    ["d", "9903"],
    ["a", "31990:<restaurantPublicKey>:synvya-restaurants-v1.0", "<relayUrl>", "all"]
  ],
  "content": ""
}
```

**For kind:9904 (reservation.modification.response):**
```yaml
{
  "kind": 31989,
  "pubkey": "<restaurantPublicKey>",
  "tags": [
    ["d", "9904"],
    ["a", "31990:<restaurantPublicKey>:synvya-restaurants-v1.0", "<relayUrl>", "all"]
  ],
  "content": ""
}
```

- Each `kind:31989` event MUST have a `d` tag with the event kind it recommends (`"9901"`, `"9902"`, `"9903"`, or `"9904"`)
- Each `kind:31989` event MUST include an `a` tag referencing the restaurant's `kind:31990` handler information event
- The `a` tag format MUST be: `"31990:<restaurantPublicKey>:synvya-restaurants-v1.0"`
- The second value of the `a` tag SHOULD be a relay URL hint for finding the handler
- The third value of the `a` tag SHOULD be `"all"` to indicate the recommendation applies to all platforms

### Publishing Requirements

- Restaurants MUST publish the `kind:31990` handler information event when first setting up their reservation system
- Restaurants MUST publish all four `kind:31989` recommendation events when first setting up their reservation system
- Restaurants SHOULD republish these events whenever their handler configuration changes or when updating their business profile
- All handler events MUST be published to the same relays where reservation messages are expected to be received

### Client Discovery

Clients discovering restaurants that support reservations SHOULD:

1. Query for `kind:31989` events with `#d` filters for `["9901"]`, `["9902"]`, `["9903"]`, and `["9904"]`
2. Extract the `a` tag values from recommendation events to find handler information events
3. Query for the corresponding `kind:31990` handler information events using the `a` tag coordinates
4. Verify that the handler information event includes all four `k` tags (`9901`, `9902`, `9903`, `9904`) before considering the restaurant as fully supporting the protocol

### Namespace discovery (Optional)
Restaurants that do not support NIP-RR but want to still be found in basic searches should publish a label ```restaurant``` within the namespace ```com.synvya.merchant```.

Include the following tags in the ```kind:0``` event for the restaurant:
```yaml
{
  "id": "<32-byte hex of unsigned event hash>",
  "pubkey": "<restaurantPublicKey>",
  "created_at": <unix timestamp in seconds>,
  "kind": 0,
  "tags": [
    ["L", "com.synvya.merchant"],
    ["l", "restaurant", "com.synvya.merchant"]
    // Additional tags MAY be included
  ],
  "content": "<content-in-plain-text>"
  // additional fields
  "sig": "<signed by restaurantPrivateKey>"  
}
```

## Verified Reviews Discovery

A client searching for verified reviews for a given restaurant with `restaurantPublicKey` SHOULD:
  1. Query for `kind:9906` events with:
	  •	`["p", "<restaurantPublicKey>"]` to select reviews for this restaurant
	  •	For each review, check for a `["verified", "<payload>"]` tag and, if present, perform the verification steps defined in section [Verified Review - Kind:9906](#verified-review–kind:9906).

If verification succeeds, the review MAY be marked as “Verified Review" and shown to the user. Unverified reviews should be ignored. 
