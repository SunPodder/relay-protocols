# Connection Handshake (`conn`): Developer Reference

The `conn` message is how a client (like a desktop app) says "hello" to the server (like an Android device). It tells the server who you are, what you support, and (optionally) provides an authentication token.

---

## Who Sends What?

- **Client → Server:** Sends the `conn` message to start the handshake.
- **Server:** Checks the info, then replies with an `ack` message (see `ack.md`).

---

## Message Structure

Here's what a `conn` message looks like:

```jsonc
{
  "type": "conn",
  "id": "conn-001",
  "payload": {
    "device_name": "Sun-PC",              // Human-readable device name
    "platform": "linux",                  // "android" or "linux"
    "version": "0.1.0",                   // Client version
    "supports": ["sms", "ping", "pong"],  // List of supported protocols
    "auth_token": "optional-token"        // Optional authentication token (string, optional)
  }
}
```

### Payload Fields

| Field         | Type     | Required | Description                                      |
| ------------- | -------- | -------- | ------------------------------------------------ |
| `device_name` | string   | Yes      | Human-readable device name                       |
| `platform`    | string   | Yes      | "android" or "linux"                            |
| `version`     | string   | Yes      | Client version                                   |
| `supports`    | string[] | Yes      | List of supported protocol types                 |
| `auth_token`  | string   | No       | Optional authentication token                    |

---

## Example

```json
{
  "type": "conn",
  "id": "conn-001",
  "payload": {
    "device_name": "Pixel 7",
    "platform": "android",
    "version": "1.2.3",
    "supports": ["sms", "ping", "pong", "notification"],
    "auth_token": "my-secret-token"
  }
}
```

---

## How to Handle

- The server should check the handshake and reply with an `ack` message.
- If `auth_token` is present, the server should verify it before accepting the connection.
- The `supports` array lets the client and server know what features they both support (feature negotiation).

---

## Security Tips

- Use `auth_token` for authentication if possible (recommended for security).
- For sensitive data, consider using TLS or message authentication.

---

## Summary

The `conn` message is the first thing a client sends. It sets up identity, features, and (optionally) authentication. The server must always reply with an `ack`.
