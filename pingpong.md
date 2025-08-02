# Heartbeat Protocol (`ping`/`pong`): Developer Reference

This protocol keeps the connection alive and healthy between server and client (Android/Desktop). It uses `ping` and `pong` messages to check if the other side is still there.

---

## Who Sends What?

- **Server → Client:** The server must periodically send `ping` messages to each client. The client must reply with a `pong` (using the same `id`). If the client doesn't reply in time, the server drops the connection.
- **Client → Server:** Clients can also send a `ping` to the server if they want to check if the server is alive. The server must reply with a `pong` (using the same `id`).
- **Both sides** should implement both `ping` and `pong` handling, but the server is always responsible for regular heartbeat pings.

---

## Message Structure

### `ping`

```jsonc
{
  "type": "ping",
  "id": "ping-001", // Required: unique for each ping
  "payload": {
    "device": "Sun-PC" // Optional: device name for context
  }
}
```

### `pong`

```jsonc
{
  "type": "pong",
  "id": "ping-001", // Required: must match the ping's id
  "payload": {
    "device": "Sun-Phone" // Optional: device name for context
  }
}
```

#### Fields

| Field      | Type   | Required | Description                                 |
| ---------- | ------ | -------- | ------------------------------------------- |
| `type`     | string | Yes      | "ping" or "pong"                           |
| `id`       | string | Yes      | Unique for each ping; pong must match ping  |
| `payload`  | object | Yes      | Can include `device` (optional, for context)|

---

## Examples

**Ping from Server:**

```json
{
  "type": "ping",
  "id": "ping-001",
  "payload": { "device": "Pixel 7" }
}
```

**Pong from Client:**

```json
{
  "type": "pong",
  "id": "ping-001",
  "payload": { "device": "Relay PC" }
}
```

---

## How to Handle

- Always use the same `id` for the `ping` and its corresponding `pong`.
- If the server doesn't get a `pong` back within a timeout, it should disconnect the client.
- Both server and client should implement both `ping` and `pong` handling.
- The server must send regular pings to all clients.

---

## Why Heartbeat?

- Detects dropped or dead connections quickly.
- Keeps the connection healthy and responsive.

---

## Summary

The heartbeat protocol is simple but essential. The server pings, the client pongs. If you don't get a pong, drop the connection. Both sides should be able to send and reply to pings, but the server is always responsible for regular heartbeat checks.
