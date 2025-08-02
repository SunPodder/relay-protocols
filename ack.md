# Acknowledgment (`ack`): Developer Reference

The `ack` message is how you confirm that you received and processed a previous message (like a handshake, command, or error). It's a simple way to say "got it" or "something went wrong."

---

## Who Sends What?

- **Server → Client:** Usually sends `ack` to confirm a connection or command.
- **Client → Server:** Can also send `ack` to confirm server commands (optional).

---

## Message Structure

Here's what an `ack` message looks like:

```jsonc
{
  "type": "ack",
  "payload": {
    "ref_id": "conn-001",         // ID of the message being acknowledged
    "status": "ok",               // "ok" or "error"
    "reason": "Invalid token"     // Optional, only if status is "error"
  }
}
```

### Payload Fields

| Field      | Type   | Required | Description                                 |
| ---------- | ------ | -------- | ------------------------------------------- |
| `ref_id`   | string | Yes      | ID of the message being acknowledged        |
| `status`   | string | Yes      | "ok" for success, "error" for failure       |
| `reason`   | string | No       | Reason for error (if status is "error")     |

---

## Example

```json
{
  "type": "ack",
  "payload": {
    "ref_id": "conn-001",
    "status": "ok"
  }
}
```

---

## How to Handle

- `ref_id` must match the `id` of the message you're acknowledging.
- `status` is `ok` for success, `error` for failure.
- If `status` is `error`, always include a `reason` to help with debugging.

---

## Best Practices

- Always send an `ack` for important protocol messages to ensure reliability.
- Use clear `reason` messages for errors to help developers debug issues quickly.

---

## Summary

The `ack` message is your protocol's way of confirming receipt and result. It's simple, reliable, and helps both sides know what's going on.
