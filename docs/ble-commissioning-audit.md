# SH1 — BLE commissioning-bearer audit + H4 port-vs-simulate decision

> Status: decision record, 2026-11-08. Audit of the eos BLE stack against
> prerequisite 1 of `docs/matter-thread-note.md`. Growth-backlog item SH1
> (2026-11-06). Home: eNet. No code in this slice; docs/decision only.

## Decision (one sentence)

**SIMULATE — H4 (EoSim Thread border-router + Matter commissioning sim)
proceeds as a simulation-only bearer, decoupled from the eos BLE stack,
because the eos BLE stack cannot carry Thread commissioning today; porting
it is a separate, larger prerequisite H4 must not wait on.**

## What was audited

- Prerequisite 1 of `docs/matter-thread-note.md`: "BLE commissioning
  bearer — Matter commissioning happens over BLE (GATT). eNet's BLE link
  layer must support the commissioning window (advertising the
  discriminator, GATT service for PASE)."
- The prerequisite's evidence base: eos `examples/ble-sensor/` (live tree,
  master, read 2026-11-08).

## Findings (each claim cites its source; nothing here is inferred)

1. **The eos BLE API is declarations only — there is no GATT API.**
   `eos/hal/include/eos/hal_extended.h:199-221` declares nine BLE
   functions (`eos_ble_init`, `eos_ble_deinit`, `eos_ble_advertise_start`,
   `eos_ble_advertise_stop`, `eos_ble_connect`, `eos_ble_disconnect`,
   `eos_ble_is_connected`, `eos_ble_send`, `eos_ble_set_rx_callback`)
   and a config struct with only `device_name`, `tx_power_dbm`,
   `adv_interval_ms`. There is no GATT service/characteristic API, no
   advertising-payload control, no scan API, no pairing/security API,
   and no MTU control anywhere in the header. A full-tree listing of
   eos master (1,064 entries, read 2026-11-08) contains zero paths
   mentioning "gatt".
2. **The only BLE implementation in eos is stubs that fail.**
   `eos/hal/src/hal_extended_stubs.c:267-285` implements every
   `eos_ble_*` function as a stub: `eos_ble_init` returns -1,
   `eos_ble_advertise_start`/`eos_ble_advertise_stop`/`eos_ble_connect`/
   `eos_ble_disconnect`/`eos_ble_send`/`eos_ble_set_rx_callback` return
   -1, `eos_ble_is_connected` returns false. No other file in the eos
   tree implements these functions: `eos/hal/src/hal_extended.c`
   (313 lines) and `eos/hal/src/hal_linux.c` (291 lines) contain no BLE
   code, and no `hal/src/backends/*ble*` file exists.
3. **The ble-sensor example therefore cannot demonstrate a working BLE
   bearer.** `eos/examples/ble-sensor/main.c:119-126` calls
   `eos_ble_init`, `eos_ble_set_rx_callback`, and
   `eos_ble_advertise_start`, and `main.c:70` calls `eos_ble_send` —
   all of which return -1 under the stub implementation (finding 2).
   The example compiles against the declarations; it does not prove a
   BLE radio transmits anything.
4. **The example's hardware claim is unsupported by the tree.**
   `eos/examples/ble-sensor/README.md:25` says "MCU: nRF52840 (or any
   BLE-capable MCU)", and `eos/examples/ble-sensor/eos.yaml:12` lists
   layer `layers/bsp/nrf52` — but the eos tree has no `layers/bsp/nrf52`
   directory (the `layers/bsp/` entries are `generic-x86`, `qemu-arm64`,
   `raspberrypi4`, `stm32h7`, `stm32mp1`). This is an eos-side
   documentation defect, flagged here for eos, not fixed in this eNet
   slice.
5. **There is no Thread/802.15.4 presence in eos to commission into.**
   The same full-tree listing contains no `thread`, `openthread`, or
   `802.15.4` paths (the only "thread" hits are POSIX thread helpers in
   `services/posix/`). Prerequisite 2 of the matter-thread note (a
   Thread stack) is therefore also unmet; it is out of scope for this
   audit but recorded because the commissioning bearer has nothing to
   hand off to yet.
6. **EoSim has no Thread or border-router model today, and only a
   peripheral-level BLE stub.** A full-tree listing of EoSim master
   (read 2026-11-08) contains no `thread` or `border` paths; the only
   "ble" hit is `eosim/engine/native/simulators/wearable.py`, which
   attaches a `BLEModule` peripheral
   (`eosim/engine/native/peripherals/wireless.py`) and tracks a boolean
   `ble_connected` state. The H4 sim therefore starts from a boolean
   connection flag, not from a commissioning protocol model.

## Consequence for prerequisite 1

Prerequisite 1 is **NOT MET**. The matter-thread note's wording is
corrected in this same PR:

- The note said "eNet's BLE link layer must support the commissioning
  window". The BLE link layer lives in eos, not eNet (the note itself
  says the implementation lives in `eos/net/`), and eos has no working
  BLE link layer — only declarations plus failing stubs. The note now
  says so explicitly.
- The note's stack table said the radio row covers "link technologies
  eNet already claims". That claim is qualified: BLE is claimed, not
  shipped.
- The note's sequencing ("BLE link → OpenThread port → simulated border
  router") implied a working BLE link comes first. It does not exist,
  so the sequence is corrected: the simulated bearer comes first
  (H4, decoupled), and the real BLE link is a separate prerequisite
  tracked below.

## Port-vs-simulate decision for H4

| Option | Verdict | Reason |
|---|---|---|
| Port the eos BLE stack, then build H4 on it | **Rejected for H4** | Requires a real BLE backend + a new GATT service API + a BLE-capable BSP (no nRF52 layer exists) — a multi-slice eos program, not a prerequisite H4 can wait on. |
| Simulate the commissioning bearer in EoSim, decoupled from eos BLE | **Chosen** | H4 can model the commissioning window (advertising the discriminator, GATT service for PASE, PASE exchange) at the protocol level in the sim, using the existing `BLEModule` peripheral pattern as the attachment point. No eos BLE code is on the H4 critical path. |

## What H4 needs next (not done here)

- H4 sim scope: model the commissioning window (discriminator
  advertising, GATT PASE service, PASE exchange) at protocol level;
  record explicitly that the sim models the bearer, not the eos HAL.
- Separate eos prerequisite (not H4-gating): a real BLE backend for at
  least one target, a GATT service/characteristic API in
  `eos/hal_extended.h`, and a BLE-capable BSP layer; plus the
  eos-side README/layer fix from finding 4. Recorded here for the eos
  queue; filing it as an eos issue is left to the run's stewardship
  pass, which owns cross-repo issue filing.
- DAC/PASE co-design with eSec (note prerequisite 4, CY1) is unchanged
  by this decision; the sim must not invent attestation behavior.

> Follow-up filing note: finding 4 (ble-sensor README claims nRF52840
> support and eos.yaml references a nonexistent `layers/bsp/nrf52`
> layer) is a pre-existing eos defect surfaced by this audit. It is
> recorded here for the eos queue; filing it as an eos issue is left
> to the run's stewardship pass, which owns cross-repo issue filing.

## Verification limits (what this audit did NOT establish)

- No BLE code was built or run; no radio behavior was tested. The
  stub finding is from reading the implementation, not from a build.
- No first-party Matter/CSA source was consulted for the PASE-over-GATT
  shape; the sim's protocol model must be grounded in one before H4
  claims protocol fidelity. Until then, H4 models the bearer at the
  level this record describes and claims nothing about CSA conformance.
- EoSim's `BLEModule` internals were not read beyond the wearable
  simulator's use of it; the H4 design must read
  `eosim/engine/native/peripherals/wireless.py` before reusing it.

## References

- `docs/matter-thread-note.md` (prerequisite 1; corrected in this PR)
- `docs/ehub-pro-spec.md` §5 (BLE caveat: "GATT PASE service
  UNVERIFIED" — now answered: no GATT service exists) and §8 gap-map
  row "BLE PASE bearer — Example exists; GATT PASE service UNVERIFIED"
  (now answered; the gap-map row itself is updated by the gap-map
  slice, not here)
- eos master, read 2026-11-08: `examples/ble-sensor/main.c`,
  `examples/ble-sensor/README.md`, `examples/ble-sensor/eos.yaml`,
  `hal/include/eos/hal_extended.h`, `hal/src/hal_extended_stubs.c`,
  `hal/src/hal_extended.c`, `hal/src/hal_linux.c`, full-tree listing
- EoSim master, read 2026-11-08: full-tree listing,
  `eosim/engine/native/simulators/wearable.py`
- Growth backlog SH1 (2026-11-06): port-vs-simulate decision recorded,
  H4 unblocked (simulate) / re-scoped (decoupled from eos BLE)
