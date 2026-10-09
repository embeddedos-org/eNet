# Device bridges: the agent-fabric hardware lane for eNet

The agent fabric's hardware-bridge lane gives AI agents real-device
reach — serial, MQTT, TCP, Modbus, debug probes — through MCP. Each
bridge below plugs into a specific eNet surface; the trust note is the
same everywhere, because bridges are network-facing by construction.

## The bridges

| Bridge | What it is | eNet surface it plugs into |
|---|---|---|
| **mcp2serial** (mcp2everything) | Serial-to-MCP bridge with Raspberry Pi Pico support | eNet's device story: agents talk to physical Pico 2s (pairs with the eos RP2350 board def `boards/rp2350.yaml`) |
| **mcp2mqtt** / **mcp2tcp** | MQTT/TCP device bridges | eNet's device story: the pub/sub and stream transports agents use to reach fleets |
| **modbus-mcp** | Modbus for industrial IoT | eNet's industrial angle: deterministic-comms lane (`docs/tsn-ethercat-lane.md`) extended to legacy industrial fieldbus |
| **embedded-debugger-mcp** (probe-rs) | Embedded debugging over MCP | Debug transport: flash/verify/inspect on real targets through the same fabric agents use for everything else |
| **xds110_mcp_server** | TI debug probes | The TI board queue: XDS110 probes as the agent-visible debug surface for TI targets |

## Trust notes (apply to every bridge)

Bridges are network-facing, so the eSec hostile-protocol posture
(`embeddedos-org/eSec` `docs/mcp-hostile-protocol-hardening.md`)
applies at each one:

- **Allowlisted destinations** — a bridge may only reach the devices
  and brokers it was configured for; discovery-on-the-network is off.
- **Identity binding** — devices authenticate to the bridge; the bridge
  authenticates to the gateway. No implicit trust because "it's on the
  bench."
- **Pinned versions + config-change approval** — the fabric's standing
  rules, per the 200k-stdIO-instances lesson.

## Relationship to the deterministic-comms lane

`docs/tsn-ethercat-lane.md` covers deterministic on-wire comms
(TSN/EtherCAT for the FRDM-IMXRT1186-class hardware). The bridges in
this document are the *agent-reachable* complement: TSN is how devices
talk to each other deterministically; the bridges are how agents talk
to devices at all.
