# MQTT broker hardening — spec and test vectors

## Scope

MQTT is eNet's device-messaging lane (bridges, telemetry,
fleet management). The broker-side code this note governs is
wherever an EmbeddedOS component accepts MQTT connections from
other devices — today that surface is small, and the time to
write its requirements down is before it grows. The pattern
this note defends against is the unauthenticated-broker class:
a broker that lets an unauthenticated connection reach
publish/subscribe state turns every topic it carries into an
open read/write channel, and brokers have repeatedly shipped
in exactly that configuration.

This is a spec for implementers, in the same shape as
`docs/dns-hardening.md`: numbered requirements, then a
test-vector table written so it can be transcribed into tests
verbatim.

## Requirements

1. **No session without authentication.** A CONNECT that
   carries no credentials — or credentials the broker cannot
   verify — is refused with the protocol's authorization
   refusal (MQTT 3.1.1 CONNACK 0x04/0x05; MQTT 5 reason code
   0x86/0x87) and the connection is closed. There is no
   anonymous mode in a shipped configuration; a build-time
   anonymous option, if one exists for bench testing, is off
   in release builds and its state is visible in the broker's
   startup log.
2. **Authorization is per-topic, not per-connection.**
   Authentication answers "who is this"; it does not make
   every topic writable. A client authorized for its own
   telemetry prefix cannot publish to another device's prefix
   or to broker-internal (`$SYS`-class) topics, and cannot
   subscribe outside its grant. Wildcard subscriptions (`#`,
   and `+` above the granted prefix) are refused unless the
   grant explicitly includes them — a wildcard is how one
   compromised device reads the whole fleet.
3. **Retained messages are publications.** A retained publish
   is stored and replayed to future subscribers, so it passes
   the same authorization as a live publish, and a zero-length
   retained publish (the retained-clear operation) is honored
   only from a client authorized for that topic.
4. **CONNECT floods are bounded before session state.**
   Connection attempts are rate-limited and half-open CONNECT
   handshakes time out in seconds, so a flood costs the
   attacker more than the broker; per-source and global
   pending-handshake caps are enforced before any session or
   subscription state is allocated.
5. **Oversized and malformed packets fail closed.** A packet
   header declaring a remaining length beyond the broker's
   maximum is refused without allocating the declared size;
   malformed fixed headers close the connection without
   affecting other sessions. (The same fail-closed discipline
   as `docs/dns-hardening.md` requires of the resolver.)
6. **Last-will messages are the client's own.** A will is
   published on ungraceful disconnect with the client's
   identity and only to topics the client could have published
   to itself; a will must never become a way to publish as
   someone else after disconnect.

## Test vectors

| # | Vector | Input shape | Expected broker behaviour |
|---|---|---|---|
| MQTTV-01 | Anonymous CONNECT | CONNECT with no username/password against a credentialed broker | Refused (auth refusal code), connection closed, no session state |
| MQTTV-02 | Wrong credentials | CONNECT with a well-formed but incorrect password | Refused as MQTTV-01; repeated attempts rate-limited |
| MQTTV-03 | Cross-prefix publish | Client granted `fleet/device-a/#` publishes to `fleet/device-b/telemetry` | Publish refused/not routed; connection state unchanged |
| MQTTV-04 | Wildcard overreach | Client granted `fleet/device-a/#` subscribes to `#` or `fleet/#` | Subscription refused for the out-of-grant filter |
| MQTTV-05 | Retained replay attack | Unauthorized client sets a retained message on a victim's topic, then a legitimate subscriber connects | Retained publish refused at publish time (MQTTV-03 rule); subscriber never receives it |
| MQTTV-06 | CONNECT flood | 1,000 half-open CONNECTs (no second packet) from one source | Pending-handshake cap + timeout engage; established sessions unaffected |
| MQTTV-07 | Declared-length abuse | Packet header declaring the maximum remaining length with a truncated body | Refused without proportional allocation; connection closed; broker serves other clients |
| MQTTV-08 | Will impersonation | Client A's will names a topic under client B's prefix | Will is not published on A's disconnect (or published only within A's grant) |

## Adoption

Whoever next touches an MQTT broker or broker-like component
in the org implements the vectors as tests named after their
ids (`MQTTV-01`…) alongside the code, the way `eos/net` is
expected to adopt the `DNSV-*` vectors from
`docs/dns-hardening.md`. If a conforming implementation
already exists for part of this surface, the adoption step is
still the vectors — run them, and record the result against
this note rather than re-litigating the requirements.
