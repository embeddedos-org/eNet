# The deterministic-comms lane: TSN + EtherCAT

**Status:** direction (2026-10-08). eNet's wireless work (LoRaWAN study,
Matter/Thread note, sub-GHz three-way plan) covers the IoT edge. This
document opens the *deterministic* lane: Time-Sensitive Networking and
EtherCAT for industrial control, robotics, and onboard-AI flight
systems -- where "best effort" is not an option.

## Why now: named hardware

Two October 2026 boards make this lane concrete instead of aspirational:

- **NXP FRDM-IMXRT1186** -- M7 @ 800MHz + M33 @ 300MHz, **dual GbE TSN**
  plus 2x Fast Ethernet for EtherCAT/TSN, motor-control headers. One
  board carries both the TSN endpoint and the EtherCAT master story,
  plus the motor it would drive.
- **Upbeat Bluemag Pi** -- SiFive E3 (flight-critical) + E2 (AI/system)
  RISC-V flight controller with onboard AI, demoing at CEATEC Oct 13-16.
  Onboard-AI flight control needs deterministic comms between the
  real-time core and the AI core -- exactly the TSN use case.

## Lane scope

| Layer | eNet position |
|---|---|
| TSN endpoint (802.1AS time sync, 802.1Qbv scheduling) | Planned; FRDM-IMXRT1186 is the reference target |
| EtherCAT master | Planned; same board, 2x Fast Eth |
| CAN FD | Noted (Arduino VENTUNO Q carries it; the robotics lane overlaps) |
| Deterministic inter-core comms (dual-brain boards) | Design input from Bluemag Pi / VENTUNO Q |

## Non-goals

eNet does not implement a full TSN switch stack on day one. The lane
starts at the *endpoint*: time sync + scheduled transmission on the
reference board, with the switch-side work explicitly out of scope
until an endpoint ships.

## Watch items

- **CEATEC Oct 13-16**: Bluemag Pi demo -- watch for published
  latency/jitter numbers on the E3/E2 split; they become eNet's
  reference budgets.
- **Apple Oct-13 hub event**: Thread/Matter watch (separate lane, but
  the hub is a TSN-adjacent time-sync story for the home).

## Cross-references

- EoSim `eosim/platforms/nxp-frdm-imxrt1186/platform.yml` -- the board def.
- EoSim `eosim/platforms/bluemag-pi/platform.yml` -- the board def.
- `docs/subghz-three-way-plan.md` -- the wireless lane (complementary, not competing).
