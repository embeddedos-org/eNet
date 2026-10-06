# eNet

Networking subsystem for the EmbeddedOS platform — link technologies, protocols, and discovery.

**Status: Implemented — in [`eos`](https://github.com/embeddedos-org/eos), at
`net/`.** Not here.

Under §28 of the master design, *Implemented* means "feature exists and is
usable", evidenced by code and functional tests. eNet meets that: 26 public
functions in `net/include/eos/net.h`, a POSIX backend in `net/src/net_posix.c`,
and `test_net` passing in the eos suite. What it does not have is a separate
repository containing any of it, and that is deliberate.

This repository exists so the component has an issue tracker and a place to
record decisions. It is deliberately not a mirror: duplicating the sources here
would give the platform two copies to keep in step.

The v2.0 master design is direct about why:

> §10: **Repositories should not be the dependency API.**

> §21.1: Do not create a separate repository merely to create a branded name. A
> subsystem earns a separate repository when it has a stable interface,
> independent release lifecycle, clear maintainers and multiple consumers.

Depend on eNet through a **component manifest**, not through this repository.
See `manifests/enet-wifi.yml` in
[embeddedos-stack](https://github.com/embeddedos-org/embeddedos-stack) for a
worked example.

The cost of getting this wrong is visible elsewhere in the platform. There are
two independent Ed25519 implementations (eos and eBoot) and two OTA
implementations (eos and eos-health), and each pair has drifted: the same
low-order-key bypass exists in both Ed25519 copies, and eos-health's OTA calls a
verification function that is defined nowhere. Splitting code across
repositories before there is a component model is how that happens.

## When this repository would earn code

All four of §21.1's conditions, not one:

- a stable interface — `eos/net.h` is at 0.x and still moving
- an independent release lifecycle — eNet currently ships when eos ships
- clear maintainers distinct from the eos maintainers
- multiple consumers depending on it *as a component*, which needs the component
  manifest to exist first

Until then the honest arrangement is code in one place and a manifest that
points at it.

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
