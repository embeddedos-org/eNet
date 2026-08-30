# eNet

Networking subsystem for the EmbeddedOS platform — link technologies, protocols, and discovery.

**Status: Planned.** There is no implementation in this repository yet. The
code that will become eNet lives in [`eos`](https://github.com/embeddedos-org/eos)
today, at `net/`.

This repository exists so the component has a home, an issue tracker, and a
place to record decisions before any code moves. It is deliberately not a
mirror: duplicating the sources here would give the platform two copies to
keep in step, and §24 of the architecture document is explicit that internal
modules should not be promoted into separate brands until they have stable
interfaces and users.

## What eNet owns

- Link technologies: Ethernet, Wi-Fi, BLE, Thread, Zigbee, CAN/CAN-FD
- Network protocols: TCP/IP, UDP, DHCP, DNS, MQTT, CoAP, HTTP, WebSocket
- Security integration through eSec/TLS
- Device discovery and network configuration

Today `eos/net/` is the EoS-facing API with a POSIX socket backend for host
builds. There is no IP stack for bare-metal targets — `eos_net_connect()`
returns `-1` there. ADR-014 selects lwIP for the Nano and Edge profiles.

## Where the code is now

| | |
|---|---|
| Implementation | [`eos`](https://github.com/embeddedos-org/eos) → `net/` |
| Maturity | [`eos/STATUS.md`](https://github.com/embeddedos-org/eos/blob/master/STATUS.md) |
| Decision of record | [ADR-014 — TCP/IP via lwIP](https://github.com/embeddedos-org/eos/blob/master/docs/adr/ADR-014-tcpip-via-lwip.md) |

## When code moves here

The architecture document sets one condition, and it has not been met:

> eNet should initially remain a subsystem of EoS unless it needs a release cycle independent of the OS. A separate repository is justified only when it becomes reusable across multiple runtimes. — §12

Until then, work on eNet happens in `eos`. Opening the split earlier would
cost a release cycle, a CI pipeline and a versioning story for a component
whose interface is still changing.

## Reference

- Architecture & Ecosystem Design Document — §12 (Platform / Core)
- [Repository taxonomy](https://github.com/embeddedos-org/eos) — §22
- [Organization model](https://github.com/embeddedos-org/.github) — §23

## License

MIT. See [LICENSE](LICENSE).
