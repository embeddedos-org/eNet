# OCPP target decision (ev-charging lane)

Status: decision note, 2026-11-01 (Sunday consolidation, morning domain
step: ev-charging). Home repo: eNet (protocol track). Companion docs:
`docs/home-systems-plan.md` (Matter/Thread home-systems track),
`docs/ehub-pro-spec.md` (eHub-Pro Specify-stage spec). Hardware source
of truth for the BMS: the eBMS-100A datasheet + BOM in
eCAD-Hardware-Products (`eEnergy_CAD_Design/battery_products/`).

Seam served: Stage 6 (integrating/shipping) primarily, Stage 3
(building) via the EV2 contract. A charger built to the wrong OCPP
major version is an integration product that cannot talk to its
ecosystem; this decision exists before any charger code so inertia
does not pick a version by accident.

## 1. Decision

- **OCPP 2.0.1 is the floor.** Any charger work in this org targets
  OCPP 2.0.1 at minimum.
- **OCPP 2.1 is the forward target.** 2.1 is the version new work
  grows toward; it is not a v1 claim for anything.
- **OCPP 1.6 is excluded from the target, not deferred.** 1.6 and
  2.0.1 are not compatible with each other, so "support 1.6 for now"
  is not a stepping stone to 2.x — it is a different, incompatible
  implementation.

Ground (first-party anchor): the Open Charge Alliance protocol page
(openchargealliance.org/protocols/open-charge-point-protocol, read
2026-11-01). Three versions are available. OCPP 2.0.1 was released in
2020 and its Edition 3 was approved as IEC standard IEC 63584 in
2024. OCPP 2.1 was released in 2025, builds on 2.0.1 with all 2.0.1
application logic kept compatible, and adds: ISO 15118-20 support
with bidirectional power transfer, a bidirectional-charging
functional block (V2X), a DER control block, improved smart charging,
extended transaction options (fixed costs/energy/time, resume after
forced reboot), battery swapping, local cost calculation, and new
authorisation options (prepaid cards, ad-hoc payment, secure dynamic
QR codes).

## 2. Boundary statement (three boundaries, not one blur)

- **OCPP = charger ↔ CSMS.** The protocol between a charge point and
  its Charging Station Management System. This is the surface EV1
  decides.
- **OCPI = roaming.** Roaming between charge-point operators and
  mobility service providers is OCPI's job, not OCPP's. This note
  does not decide OCPI.
- **ISO 15118 = vehicle link.** The vehicle-to-charger link
  (including the ISO 15118-20 support that OCPP 2.1 carries) is a
  separate surface from OCPP itself. This note records the
  relationship; it does not specify the vehicle link.

Blurring these three into "the charging protocol" is how a BMS
datasheet ends up described as a charging stack. It is not one.

## 3. What exists today (honest inventory)

- **eBMS-100A BMS package** (eCAD-Hardware-Products,
  `eEnergy_CAD_Design/battery_products/`): product datasheet + BOM,
  STM32G474 + LTC6813 cell monitoring + active balancing, CAN FD
  (plus SMBus / RS-485 / BLE per the datasheet comms line). A
  battery management system. Not a charger.
- **`eos/products/ev.h` product-profile header.** A profile header,
  not an implementation. Its existence is not EV-charging support.
- **eBMS-100A's peer role in the automotive lane.** The automotive
  eVCU-Pro use case (selected 2026-10-31) already consumes the BMS
  as a peer device on its CAN FD vehicle network. That peer role and
  this lane's charger-controller role consume the SAME BMS contract
  (EV2) — one contract, not two.
- **No charger. No EVSE controller. No OCPP code. No CSMS.** None
  of these exists anywhere in the charging-relevant repos today.
  The site's ev-charging entry bundles three products under one
  reference name (BMS + 10 kWh pack + 22 kW motor drive); the
  package on disk is the BMS. Any sentence implying otherwise is
  wrong.

## 4. Sequencing

1. **EV1 (this note)** — version + boundary decision.
2. **EV2 (shared BMS contract)** — the CAN FD surface (pack state,
   pack limits, fault signalling, contactor/inhibit commands) written
   once, consumed by both the automotive and charging lanes, with
   the safe-state table as a Specify-stage deliverable. EV2 must not
   define a second CAN dialect.
3. **EV3 (charger-side gap map)** — charger power stage, EVSE
   controller, and CSMS surface as three separately-gated pieces.
4. **Any charger code** — only after EV1–EV3. No OCPP code before
   EV1; no charger hardware before EV3.

The CSMS transport story waits on eos tracker #199 Slice 3 (the MQTT
cloud leg): the depot chain is cells → BMS → CAN FD → charger
controller → OCPP/CSMS → fleet visibility → charge-session
analytics, and the cloud leg runs over the eos MQTT chain.

## 5. Selected use case this decision serves

ev-charging, selected 2026-11-01 (roster step, register pointer
6 → 7): **eBMS-100A as the battery guardian of a depot charging
post.** The 100 A BMS (STM32G474 + LTC6813 + active balancing)
watches a 10 kWh eLiPack pack, reports pack state and limits over
CAN FD to a charger controller, and the charger — speaking OCPP
2.0.1 to its CSMS, with 2.1's DER/bidirectional blocks as a
documented forward target, not a v1 claim — runs a managed charging
session (authorise → charge under a CSMS charging profile → meter →
stop), with safe state on any BMS fault (contactor open / charge
inhibit; the exact table is a Specify-stage deliverable shared with
the automotive lane). Board class: constrained MCU for the BMS; the
charger controller's MCU is TBD at Specify stage and is not assumed
to be the same chip.

## 6. Open verification tasks (questions, NOT asserted facts)

(a) OCPP 2.1 Edition 2 / IEC 63584-210:2025 / EN IEC 63584-210:2026 —
secondary-reported (2026 guides disagree on the 2026 layer); re-verify
against OCA/IEC primary text before any committed doc cites those
numbers. (b) NEVI OCPP 2.0.1 mandate (23 CFR Part 680) —
secondary-reported; re-verify against the CFR text before citing it
as a funding floor. (c) OCPP 1.6 WebSocket security exposure —
secondary (US DOE / Idaho National Laboratory via a secondary guide);
a security argument, not a design input, until read first-party.
(d) The site note's 22 kW motor-drive identity — no matching named
product was found in the BMS datasheet; reconcile at Specify stage.
(e) Which standards attach to which product — the site lists IEC
62619 / UL 1973 / IEC 62477-1 while the datasheet family adds IEC
62933 / IEC 62368-1; a single "ev-charging compliance" claim would be
wrong for all five products. Resolve per product at Specify stage.

## 7. Non-claims box (explicit)

No OCPP implementation. No charger hardware. No EVSE controller.
No CSMS. No certification of any kind — IEC 62619, UL 1973, and
IEC 62477-1 are targets, not achievements. No "EoS supports EV
charging" on the basis of `ev.h`. No Edition-2 / IEC 63584-210
numbers, no NEVI citation as fact, no 1.6 security claim as
first-party fact. This note decides a version and a boundary; it
ships no charging capability.
