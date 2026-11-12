# Matter / Thread landscape note (KB seed)

> Status: research note, not a roadmap commitment. eNet has no code yet;
> the implementation lives in `eos/net/`. This note records what eNet's
> commissioning story would need if/when Matter support is pursued.

## Why Matter matters for eNet

Matter is the application-layer standard the smart-home industry converged
on. For an embedded OS with health- and home-adjacent ambitions (eos-health,
eApps device targets), speaking Matter is table stakes for "works with"
ecosystems (Apple Home, Google Home, Alexa, SmartThings).

## The stack, bottom-up

| Layer | Technology | eNet relevance |
|---|---|---|
| Radio | 802.15.4 (Thread), Wi-Fi, Ethernet | Link technologies eNet claims; BLE is claimed, not shipped (see audit note below) |
| Mesh | Thread 1.3/1.4 | Needs a Thread stack — OpenThread is the reference |
| Commissioning | BLE (Matter commissioning) | Every Matter device commissions over BLE first |
| Application | Matter (application clusters, data model) | Needs the Matter SDK or a subset |
| Security | PASE/CASE session establishment, DAC attestation | Ties directly to eSec (key management, attestation) |

Key point: **Thread is not a Matter alternative — it is Matter's
low-power transport.** A "Matter over Thread" device uses BLE to get
commissioned, then operates over the Thread mesh. eNet cannot do Matter
without both a BLE story and a Thread story.

## Version pins (SH3, 2026-11-11)

- **Thread 1.4** is the target version. The Thread Group's
  border-router-certification-only rule has been in force since
  Jan 1, 2026 — the deadline has passed, so any border-router
  claim in this bundle is a Thread 1.4 claim.
- **Matter 1.6.1** is the target version (CSA Core Specification
  document 23-27349, accepted by the CSA Board 2026-09-16).
  Certification lag: no entry on the CSA certification ledger
  states Matter 1.6.1 yet; the newest certified version on the
  ledger is 1.6. The bundle targets 1.6.1 on the strength of the
  published spec, not on a certified product. Home Assistant's
  matter.js server supports 1.6.1 features (Groupcast,
  battery-device handling, OTA controls, Matter network map) —
  secondary-reported (maison-et-domotique, Nov 3, 2026).
- Pins are bundle target decisions, refreshed 2026-11-11 (SH3,
  research brief 2026-11-11 §4/§6); they are not certification
  claims.

## Silicon worth watching

- **ESP32-H2** — 802.15.4 + BLE 5 (LE); Espressif's Thread/Zigbee vehicle.
  No Wi-Fi: a pure 15.4 end-device chip.
- **ESP32-C5** — 802.15.4 + Wi-Fi 6 + BLE; the first Espressif chip that can
  be a Thread end-device *and* a Wi-Fi Matter device in one SKU.
- **nRF54 series** (Nordic) — Thread + BLE + Matter SDK support is Nordic's
  home turf; relevant if eNet ever targets Nordic silicon.

## What eNet's commissioning story would need

1. **BLE commissioning bearer** — Matter commissioning happens over BLE
   (GATT). Status, audited 2026-11-08 (SH1, `docs/ble-commissioning-audit.md`;
   eNet #9 merged 2026-11-09): NOT MET. The BLE link layer lives in eos, not eNet, and eos has no working
   BLE link layer: `hal/include/eos/hal_extended.h:199-221` declares the BLE API
   with no GATT service/characteristic API, and the only implementation,
   `hal/src/hal_extended_stubs.c:267-285`, returns -1 for every BLE call. The
   ble-sensor example therefore cannot demonstrate a working bearer. Until eos
   ships a real BLE backend plus a GATT API, the commissioning window
   (advertising the discriminator, GATT service for PASE) cannot be carried by
   the eos stack. Prerequisite 1 (real BLE backend + GATT API in eos, plus an
   nrf52 BSP) is still unmet post-SH1: real-hardware commissioning remains
   blocked. H4 decision (same audit): SIMULATE — the EoSim border-router
   sim models the bearer at protocol level, decoupled from eos BLE; porting the
   eos BLE stack is a separate, larger prerequisite H4 does not wait on.
2. **Thread stack** — OpenThread port for at least one 15.4-capable target
   (ESP32-H2 is the natural first target). This is the biggest lift.
3. **Border router story** — Thread meshes need a border router to reach IP
   networks. EoSim could simulate one before hardware exists.
4. **Device Attestation (DAC)** — Matter requires a Device Attestation
   Certificate chain at commissioning. That is eSec's domain (key
   provisioning, certificate storage); eNet and eSec must design this
   together.
5. **Application clusters** — start with On/Off + LevelControl (lights),
   the smallest certifiable surface, before sensor clusters.

## Suggested sequencing (if pursued)

Simulated commissioning bearer in EoSim (H4, decoupled from eos BLE) → real BLE link + GATT API in eos (separate prerequisite, not H4-gating) → OpenThread port (H2) → simulated border router in EoSim →
PASE/CASE via eSec → minimal Matter application (on/off light) →
certification pre-test. Each step is independently shippable and testable;
do not attempt the full stack at once.

## References

- Matter specification — Connectivity Standards Alliance
- Thread 1.4 specification — Thread Group
- OpenThread — https://openthread.io (Apache-2.0)
- Espressif ESP-Matter SDK — the fastest path to a working demo on H2/C5
- eSec — Device attestation and key management (co-design required)

