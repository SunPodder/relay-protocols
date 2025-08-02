# Protocol Envelope Format: Developer Reference

This document defines the base protocol structure for all communication between clients and servers (e.g., Android ↔ Desktop). If you're building or consuming a protocol using this format, **read this first**—it covers everything you need to know to send, receive, and understand messages.

---

## Message Framing (How Data is Sent)

Every message is sent as:

```text
----------------------------------------------------
| 4-byte length (big-endian) | JSON message bytes  |
----------------------------------------------------
```

- The first 4 bytes are an unsigned integer (big-endian) telling you how many bytes the JSON message is.
- The JSON message comes right after, encoded in UTF-8.
- The length **does not** include the 4-byte prefix—just the JSON part.

**Why?** This makes it easy to know exactly where each message starts and ends, even if you send lots of them over a socket.

---

## Message Structure (What a Message Looks Like)

Every message is a JSON object with this structure:

```jsonc
{
  "type": "<type>",           // Required. What kind of message is this? (e.g., ping, sms, notification)
  "id": "uuid-123",           // Optional. Unique ID for tracking or matching requests/responses
  "timestamp": 1729577323,    // Optional. UNIX time (seconds, UTC)
  "payload": { }          // Required. The actual data for this message type
}
```

### Field Reference

| Field       | Type    | Required | Description                                                |
| ----------- | ------- | -------- | ---------------------------------------------------------- |
| `type`      | string  | Yes      | What kind of message? (`conn`, `ack`, `ping`, `sms`, etc.) |
| `id`        | string  | No       | Unique ID for matching requests/responses                  |
| `timestamp` | int     | No       | UNIX timestamp (seconds, UTC)                              |
| `payload`   | object  | Yes      | The actual message data                                    |

---

## Example

Here's a real message you might send:

```json
{
  "type": "ping",
  "id": "ping-001",
  "timestamp": 1729577323,
  "payload": {
    "device": "Sun-PC"
  }
}
```

**On the wire:**

```java
[0x00 0x00 0x00 0x58]   // 88-byte length prefix (big-endian)
{"type":"ping","id":"ping-001","timestamp":1729577323,"payload":{"device":"Sun-PC"}}
```

---

## Rules & Best Practices

- **Always** use the 4-byte length prefix framing for every message.
- The JSON must be UTF-8 encoded.
- All messages must follow the structure above—no exceptions.
- Only read as many bytes as the length prefix says, then parse the JSON.
- The `type` field is how you know what kind of message it is.
- The `payload` field is where all the real data goes (see other docs for type-specific payloads).
- You can add `id` and `timestamp` if you need them for tracking or ordering, but they're optional.

---

## Why This Format?

- **Simple:** Easy to implement in any language.
- **Extensible:** You can add new message types or payload fields without breaking old clients.
- **Reliable:** No guessing where messages start/end.

---

## Need More?

See the other protocol docs for details on specific message types and their payloads. This doc is the foundation—**all protocol messages must follow this envelope format.**
