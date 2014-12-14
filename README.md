# Palaver IRC Capability

This repository contains the specification for the Palaver IRC Capability. An extension to the IRC protocol providing support for the Palaver client to negotiate preferences and capability to receive push notifications.

The specification is completely open-source, which allows anyone to build their own server-side implementation to support push notifications with Palaver. If you’re building an implementation, we would love to hear from you.

## Capabilities

The main capability is named `palaverapp.com`, please consult the [specification](Specification.md) for details on how it works.

Outside of this specification, Palaver also supports the following capabilities:

- [server-time](http://ircv3.atheme.org/extensions/server-time-3.2)
- [sasl](http://ircv3.atheme.org/extensions/sasl-3.1)
- [multi-prefix](http://ircv3.atheme.org/extensions/multi-prefix-3.1)
