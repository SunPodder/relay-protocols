# [Draft] SMS Protocol (`sms`): Developer Reference

This protocol lets Android and Desktop send and receive SMS messages. It works both ways and supports delivery status updates.

---

## Who Sends What?

- **Android → Desktop:** Notifies about incoming SMS.
- **Desktop → Android:** Sends outgoing SMS.

---

## Message Structure

Here's what an `sms` message looks like:

```jsonc
{
  "type": "sms",
  "id": "sms-uuid-001",
  "payload": {
    "direction": "incoming",        // "incoming" or "outgoing"
    "from": "+880123456789",        // Incoming only
    "to": "+880123456789",          // Outgoing only
    "body": "Hey, how are you?",
    "timestamp": 1729578000,         // UNIX epoch seconds (UTC)
    "status": "sent"                // Optional: sent, delivered, failed
  }
}
```

### Payload Fields

| Field       | Type   | Required | Description                                 |
| ----------- | ------ | -------- | ------------------------------------------- |
| `direction` | string | Yes      | "incoming" or "outgoing"                   |
| `from`      | string | Incoming | Sender phone number (incoming only)         |
| `to`        | string | Outgoing | Recipient phone number (outgoing only)      |
| `body`      | string | Yes      | SMS message text                            |
| `timestamp` | int    | Yes      | UNIX timestamp (seconds, UTC)               |
| `status`    | string | No       | Delivery status: sent, delivered, failed    |

---

## Examples

**Incoming SMS (Android → Desktop):**

```json
{
  "type": "sms",
  "id": "sms-uuid-002",
  "payload": {
    "direction": "incoming",
    "from": "+880123456789",
    "body": "Hello!",
    "timestamp": 1729579000
  }
}
```

**Outgoing SMS (Desktop → Android):**

```json
{
  "type": "sms",
  "id": "sms-uuid-003",
  "payload": {
    "direction": "outgoing",
    "to": "+880123456789",
    "body": "Hi, this is a test.",
    "timestamp": 1729579050
  }
}
```

---

## How to Handle

- Use `direction` to know if the message is incoming or outgoing.
- Use `status` to track delivery state (`sent`, `delivered`, `failed`).
- Timestamps should always be UNIX epoch seconds (UTC).

---

## Notes

- For incoming messages, `from` is required; for outgoing, `to` is required.
- Delivery status updates may be sent as additional `sms` messages with the same `id`.
