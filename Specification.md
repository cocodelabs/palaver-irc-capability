# Palaver Capability

Copyright (c) 2012-2019 Cocode <support@cocode.org>.

Unlimited redistribution and modification of this document is allowed
provided that the above copyright notice and this permission notice
remains intact.

## The `palaverapp.com` client capability

This specification describes the capability which was build for push
notifications in the Palaver IRC Client.

The name of this capability is `palaverapp.com` and can be enabled using
[capability
negotiation](http://ircv3.atheme.org/specification/capability-negotiation-3.1).

## PALAVER Command

### `BACKGROUND`

When the Palaver client goes into the background, this command will be emitted.
This command may only be sent from iOS with multi-tasking.

```
PALAVER BACKGROUND
```

### `FOREGROUND`

When the Palaver client comes back into the foreground from the background, this command will be emitted.

```
PALAVER FOREGROUND
```

### `IDENTIFY <client-token> <client-preference-version> <network-uuid>`

When a client connects and the capability is enabled. It may send an `IDENTIFY`
command to identify it via a unique token and the current version number. A
network UUID may also be sent.

For example:

```
PALAVER IDENTIFY 9167e47b01598af7423e2ecd3d0a3ec4 611d3a30a3d666fc491cdea0d2e1dd6e b758eaab1a4611a310642a6e8419fbff
```

After a client has identified, you can request the preferences for a client if
you do not have them. The parameter `client-preference-version` is provided for
this benefit.

### `REQ`

REQ can be sent by the server to the client to request the preferences for the
client.

The client will then send a `BEGIN` to indicate it has started sending
preferences. Once the references have been sent, an `END` will be sent from the
client.

A server should reset it's preferences for a client when the BEGIN command is
received and then set the new values. This will prevent any from being left
behind.

```
PALAVER BEGIN <client-token> <client-preference-version>
PALAVER SET PUSH-TOKEN 605b64f5addc408fcfa7ff0685e2d065fdecb127
PALAVER SET PUSH-ENDPOINT https://api.palaverapp.com/1/push
PALAVER ADD MENTION-KEYWORD cocode
PALAVER ADD MENTION-KEYWORD {nick}
PALAVER END
```

#### `SET <key> <value>`

The `SET` command is used to set a key-value property. The following properties
may be sent:

- `PUSH-TOKEN` (string)
- `PUSH-ENDPOINT` (string)
- `SHOW-MESSAGE-PREVIEW` (either `true` or `false`)
    - Default: true

#### `ADD <key> <value>`

The add subcommand is to add a preference to a unique list. For example, a list
of mention keywords.

The following keys may be sent:

- `MENTION-KEYWORD`
- `MENTION-CHANNEL`
- `MENTION-NICK`
- `IGNORE-KEYWORD`
- `IGNORE-CHANNEL`
- `IGNORE-NICK`

Values can be substituted if they use curly braces. For example, {nick} should
be substituted with the user's current nickname on comparison.

## PUSH notifications

Push notifications should be sent to the client using the `PUSH-ENDPOINT` when
a mention happens. The following rules should be used to determine if a push
notification should be sent:

- The message contains any occurrence `MENTION-KEYWORD` matching should be
  done using a case-insensitive comparison using word-boundaries.
- The message is sent to a channel in `MENTION-CHANNEL`.
- The message is sent by a nick in `MENTION-NICK`.

### Ignore

After a message has been matched, it should then be check against the ignore
list. If it matches any of the following criteria, a push notification should
not be sent to the client.

- The message contains a any occurrence of an `IGNORE-KEYWORD` matching should be
  done using a case-insensitive comparison using word-boundaries.
- The message is sent to a channel in `IGNORE-CHANNEL`.
- The message is sent from a nick in `IGNORE-NICK`.

### Sending a notification

Once a message meets the above criteria, you should send it a push notification
using the HTTP API in `PUSH-ENDPOINT` if it's set.

If the URL is set to `https://api.palaverapp.com/1/push`, the request should
look something like:

```
POST /1/push HTTP/1.1
Host: api.palaverapp.com
Authorization: Bearer <PUSH-TOKEN>
Content-Type: application/json
```

```javascript
{
    "badge": 4,
    "message": "Hello Kyle!",
    "sender": "Dennis",
    "channel": "#palaver",
    "network": "b758eaab1a4611a310642a6e8419fbff"
}
```

When the notification is no longer relevant, such as when the user has read the
message on another device. You can reset it's notifications and badge count
using:

```
POST /1/push HTTP/1.1
Host: api.palaverapp.com
Authorization: Bearer <PUSH-TOKEN>
Content-Type: application/json
```

```javascript
{
    "badge": 0
}
```

If `SHOW-MESSAGE-PREVIEW` is set to false, then the message should not be sent
to the server. Instead `private` should be set to `true`:

```javascript
{
    "badge": 4,
    "private": true,
    "sender": "Dennis",
    "channel": "#palaver",
    "network": "b758eaab1a4611a310642a6e8419fbff"
}
```
