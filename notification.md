# Notification Protocol: Developer Reference

This protocol lets Android send notification events to Desktop, and Desktop send interaction commands (reply, dismiss, action) back to Android. Here’s how it works and what each message looks like.

---

## Who Sends What?

- **Android → Desktop:** Sends notification events.
- **Desktop → Android:** Sends interaction commands (reply, dismiss, action).

---

## Message Types & Structures

### Notification Event (`notification`)

Sent from Android to Desktop to inform about a new or updated notification.

```jsonc
{
  "type": "notification",
  "payload": {
    "id": "abc123",                // Unique notification key (see Notes)
    "title": "Messenger",
    "body": "Hey, what's up?",
    "app": "Messenger",
    "package": "com.facebook.orca",
    "timestamp": 1729612345,
    "can_reply": true,
    "actions": [
      {
        "title": "Reply",
        "type": "remote_input",
        "key": "quick_reply"
      },
      {
        "title": "Mark as Read",
        "type": "action",
        "key": "mark_read"
      }
    ]
  }
}
```

#### Payload Fields

| Field        | Type         | Required | Description                                 |
| ------------ | ------------ | -------- | ------------------------------------------- |
| `id`         | string       | Yes      | Unique notification key                     |
| `title`      | string       | Yes      | Notification title                          |
| `body`       | string       | Yes      | Notification body/text                      |
| `app`        | string       | Yes      | App name                                    |
| `package`    | string       | Yes      | App package name                            |
| `timestamp`  | int          | Yes      | UNIX timestamp (seconds, UTC)               |
| `can_reply`  | boolean      | Yes      | Can the notification be replied to?         |
| `actions`    | array        | Yes      | List of available actions                   |

Each action in `actions`:

| Field    | Type   | Required | Description                      |
| -------- | ------ | -------- | -------------------------------- |
| `title`  | string | Yes      | Button/action label              |
| `type`   | string | Yes      | "remote_input" or "action"       |
| `key`    | string | Yes      | Key for reply/action             |

---

### Notification Action (`notification_action`)

Sent from Desktop to Android to trigger a notification action. This covers both button actions and remote input (reply) actions.

```jsonc
{
  "type": "notification_action",
  "payload": {
    "id": "abc123",                 // Notification ID
    "key": "quick_reply",           // Action or RemoteInput key
    "type": "remote_input",          // "remote_input" or "action"
    "body": "Sure, talk later"       // Optional; only for remote_input
  }
}
```

---

### Notification Dismiss (`notification_dismiss`)

Sent from Desktop to Android to dismiss the notification.

```json
{
  "type": "notification_dismiss",
  "payload": {
    "id": "abc123"
  }
}
```

---

## Examples

**Notification from Android:**

```json
{
  "type": "notification",
  "payload": {
    "id": "abc123",
    "title": "Messenger",
    "body": "Hey, what's up?",
    "app": "Messenger",
    "package": "com.facebook.orca",
    "timestamp": 1729612345,
    "can_reply": true,
    "actions": [
      { "title": "Reply", "type": "remote_input", "key": "quick_reply" },
      { "title": "Mark as Read", "type": "action", "key": "mark_read" }
    ]
  }
}
```

**Reply from Desktop (remote_input):**

```json
{
  "type": "notification_action",
  "payload": {
    "id": "abc123",
    "key": "quick_reply",
    "type": "remote_input",
    "body": "Sure, talk later"
  }
}
```

**Action from Desktop (button/action):**

```json
{
  "type": "notification_action",
  "payload": {
    "id": "abc123",
    "key": "mark_read",
    "type": "action"
  }
}
```

**Dismiss from Desktop:**

```json
{
  "type": "notification_dismiss",
  "payload": {
    "id": "abc123"
  }
}
```

---

## How to Handle

- On Android, keep a map of active notifications with their `id` and action metadata.
- On `notification_action`:
  - If `type` is `remote_input` and `body` is present, use `RemoteInput` and `PendingIntent` to send the reply.
  - If `type` is `action`, call `PendingIntent.send()` for the specified action.
- On `notification_dismiss`, use `NotificationManager.cancel()` with the tag/id.

---

## Notes on `id`

- Use Android’s `StatusBarNotification.getKey()` if available—it's a stable unique ID across updates/dismissals.
- If not, fallback to a custom hash of `package + timestamp`.

---

## Security Tips

- Anyone on the LAN could send a fake JSON to act like a dismiss or reply.
- Recommended: Pairing with a key/fingerprint, auth tokens, MACs, or optional encryption.
