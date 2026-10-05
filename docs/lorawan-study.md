# LoRaWAN study — native backend pattern + hardware-bridge pairing

> Status: research note, not a roadmap commitment. eNet has no code yet;
> the implementation lives in `eos/net/`. This note records what a
> LoRaWAN story for eNet could look like, following the pattern that
> landed in Zephyr 4.5.

## Why LoRaWAN for eNet

Matter/Thread covers the home; LoRaWAN covers the field — kilometers, not
meters. For eos-health-adjacent sensing (remote patient monitoring
gateways), eos-aero (ground-station telemetry), and eApps device targets
in agriculture/infrastructure, LoRaWAN is the long-range uplink that
Wi-Fi and Thread cannot be. It belongs in eNet's protocol landscape next
to the Matter/Thread note.

## The Zephyr 4.5 pattern (what to copy)

Zephyr 4.5 shipped a **native LoRaWAN backend**: LoRaWAN 1.0.x Class A
running directly on the LoRa radio driver (EU868), dropping the Semtech
LoRaMac-node dependency. The design lessons for eNet:

1. **Backend, not stack.** LoRaWAN is a MAC layer above the LoRa radio —
   structure it as a backend (`lorawan`) over eNet's radio abstraction,
   not as a separate stack. The radio driver owns the PHY; the backend
   owns join procedure, frame counters, ADR, and duty-cycle bookkeeping.
2. **Class A first.** Class A (uplink-driven, two short receive windows)
   is the lowest-power, widest-supported class — the right default for
   battery sensors. Class B/C are follow-ups, not day-one.
3. **Region as configuration.** EU868 vs US915 vs AS923 differ in channels,
   duty cycle, and dwell time — make the region a build-time config with
   the channel plan as data, not code branches.
4. **No Semtech dependency.** The old approach (LoRaMac-node as an
   external component) is exactly what Zephyr removed. A native backend
   keeps the security story (eSec: key storage for AppKey/NwkKey) inside
   the org's audit boundary.

## What eNet would need

- A `LoRaMac` backend in `eos/net/` implementing join (OTAA), confirmed/
  unconfirmed uplink, and the Class A receive windows.
- Region channel-plan data files (EU868 first — the Zephyr-validated one).
- Key provisioning story: AppKey/NwkKey via eSec key management, never
  baked into firmware images in the repo.
- Duty-cycle enforcement in the backend (regulatory, not optional).

## Hardware-bridge pairing (agent fabric)

For the device story, two MCP bridges from the agent-fabric plan pair
naturally with eNet protocols:

- **cramen/modbus_connector** (Sept 2026, `--read-only` flag) — the
  best-maintained Modbus MCP server. Modbus/TCP gateways are how
  industrial LoRaWAN network servers expose device data; a read-only
  bridge lets agents poll registers without write risk.
- **mcp2mqtt** — LoRaWAN application servers (ChirpStack, TTN) speak MQTT
  on the application side. An mcp2mqtt bridge gives agents the downlink/
  uplink topic surface for device management.

Pattern: radio/MAC in firmware (`eos/net/`), application-server
integration via MCP bridges — the same split as the KB plan's
docs-MCP/tools-MCP separation.

## Open questions

- LoRaWAN 1.1 security context (Join Server split) vs 1.0.x simplicity —
  the Zephyr backend chose 1.0.x; follow unless a deployment needs 1.1.
- FUOTA (firmware update over LoRaWAN) fragmentation story — deferred,
  but the eBoot update path should know the MTU constraints exist.
