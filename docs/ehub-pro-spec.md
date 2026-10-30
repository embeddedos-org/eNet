# eHub-Pro use-case specification (Specify stage)

Status: selected 2026-10-30 (research brief §3, theme pull-forward
for smart-home-iot); specified by this document. Refs eNet #7
(#7 stays open across the track's slices by design). Home repo:
eNet, as owner of the home-systems track. Hardware source of
truth: the eCAD product datasheet
(`eCAD-Hardware-Products/eConsumer_CAD_Design/smart_home/
product_datasheet.md`, v1.0, 2026-06-03, Design Phase), cited —
not duplicated — below. Target lock: Matter 1.6.1 / Thread 1.4
(eNet home-systems plan, target lock 2026-10-30; CSA Core
Specification document 23-27349, accepted by the CSA Board
2026-09-16).

Use case, in one line: eHub-Pro as the household's Matter
controller + Thread border router + Zigbee bridge, commissioning
and controlling its own device family.

Pipeline position: smart-home-iot moves selected → specified on
this landing. The gap-map is the NEXT pipeline stage and is NOT
claimed by this spec — §9 is a gap-map pointer, not a completed
gap-map. Self-check status: selected + specified, not walkable
until the EoSim radio simulation (H4) lands — the full chain in
§8 cannot be walked even in sim before then.

## 1. Hardware (datasheet facts only)

Hub (datasheet, Electrical Specifications — eHub-Pro): NXP i.MX
8M Mini (quad Cortex-A53); 2 GB LPDDR4 + 16 GB eMMC; GbE;
USB 3.0 ×2; USB-C; 12 VDC / 10 W. Board class: Linux-capable —
already a hardware self-check class, so no self-check set
addition is needed for this use case.

Cast (datasheet, Product Family rows):

| Device | Connectivity (datasheet) | Role in this use case |
|---|---|---|
| eThermostat | Wi-Fi, BLE | Thermostat end device |
| eDoorbell | Wi-Fi 6, BLE | Video doorbell end device |
| eCamera-Home | Wi-Fi 6 | Indoor camera end device |
| eSpeaker-AI | Wi-Fi 6, BLE | Voice front end (see §10) |

No hardware fact beyond the datasheet is asserted in this spec;
anything not in the datasheet is marked "to be specified" rather
than invented.

## 2. Roles (with honesty labels)

- **Matter controller** (target): the hub commissions and
  controls Matter devices in the home, and is itself the
  household's Matter control point.
- **Thread border router** (target, Thread 1.4): the hub bridges
  the Thread mesh to the home IP network.
- **Zigbee bridge** (first mode, see below): the hub reaches
  Zigbee devices through a coordinator/bridge.

Zigbee mode note: the home-systems plan (§3.2) records two
honest options and chooses neither org-wide — the
hub/coordinator story and a native stack. For THIS use case,
the hub/coordinator story is the first mode (small org-side
surface, the pattern eNet's device-bridge work already uses
for other protocols); the native-stack alternative is
explicitly still undecided, and stays undecided until the
plan's closing evidence exists: a named deployment the hub
story cannot serve. Z-Wave is recorded as a datasheet protocol
row plus a plan §3.3 research gap — no Z-Wave stack position
is taken by this spec.

## 3. Protocols + target versions

| Protocol | Version / bearer | Status |
|---|---|---|
| Matter | 1.6.1 | target |
| Thread | 1.4 (border router) | target |
| Zigbee | 3.0 | datasheet fact |
| Z-Wave | 700 | datasheet fact |
| BLE | 5.3 | datasheet fact; BLE commissioning bearer FIRST per plan §2 step 1 (see §5 caveat) |
| Wi-Fi | 6 | datasheet fact |
| MQTT | eos #199 client | dependency (eos#199) — cloud chain only, see §7 |

No row in this table is `shipped` or `supported`. Every
`target` row is a design target, not a stack that exists.

## 4. Feature list v1 (full, but bounded)

- Controller + border-router + bridge roles (§2).
- Commissioning of the cast (§5).
- On/Off + LevelControl clusters FIRST (plan §2 step 5:
  lights as the smallest certifiable surface).
- Thermostat / doorbell / camera device control as the cast's
  v1 device set. Cluster IDs per device are to be pinned at
  gap-map stage — this spec names the devices, not cluster
  IDs it has not sourced.
- Hub-local visibility: an eApps hub-app queue item (not
  built by this spec; queued for the bundle's app step).
- Hub-local analytics (v1; see §7).

Explicitly NOT v1: NFC commissioning (a documented later
variant per plan §2); cloud analytics; a native Zigbee stack;
a Z-Wave stack.

## 5. Commissioning flow (the plan's sequence, labelled as such)

Bearer first: BLE, per plan §2 step 1. Sequence: PASE →
attestation → NOC → CASE, as the plan's sequence.

Caveat, carried from the plan: the detailed commissioning step
sequence was not verified against a first-party CSA source at
plan-writing time; it is the plan's sequence, not a quoted CSA
sequence.

BLE caveat (mandatory): eos has a BLE example
(`examples/ble-sensor/` in eos), but whether a GATT PASE
service exists is UNVERIFIED and is plan §7 Slice 1's audit
question. This spec says neither "BLE exists" as a
commissioning bearer nor "BLE is missing" — the audit answers
it.

DAC provisioning flow — requirements only: eSec co-design
(key provisioning + certificate storage, per plan §2 step 4).
Product Security 1.1 system-scope view is listed as a
requirements input, secondary-reported — not asserted as a
quoted standard revision.

## 6. Standards constraints

Site-declared targets only: ETSI EN 303 645 + IEC 62368-1
(the www site's industries entry for smart-home-iot). Wording
rule for this spec: "site-declared targets; no revision
currency is asserted beyond the site entry in this pass."
No other standards claim is made here. No certification claim
of any kind is made here.

## 7. Cloud chain + self-check chain (Verify-stage definition)

Chain: eHub-Pro → peer devices (Thread mesh + Wi-Fi) →
cloud messaging (MQTT via eos #199 — sequenced AFTER #199
Slice 3; this spec does not assume MQTT exists) → user
visibility (hub app, eApps queue item) → hub-local analytics
(v1; cloud analytics explicitly out of v1).

Self-check status: selected + specified, not walkable until
H4 (EoSim Thread border-router + Matter commissioning
simulation) lands. Second self-check target note: the
ESP32-H2 Matter end device (already in the self-check set)
is the natural SECOND target for this domain — the end-device
side of the same mesh — once the hub side specifies. It is
recorded as a walk addition, NOT a second selected use case:
one domain, one use case.

## 8. Gap-map pointer (NOT a completed gap-map)

Each row is marked "gap-map stage to complete". The next
pipeline slice (gap-map) completes this table; this spec does
not claim it complete. Seeded only with facts already
established in the research brief and this spec:

| Area | Has today | Missing | Owner repo |
|---|---|---|---|
| Matter stack | Nothing shipped | Full stack (gap-map stage to complete) | eos / eNet |
| Thread stack | Nothing shipped | Full stack (gap-map stage to complete) | eos / eNet |
| MQTT client | Codec core + Slice 2 state machine (eos #199) | Slice 3+; cloud chain (gap-map stage to complete) | eos |
| DAC / attestation | Requirements only (§5) | Provisioning + storage design (gap-map stage to complete) | eSec + eNet |
| Radio simulation | Nothing shipped | Thread BR + commissioning sim H4 (gap-map stage to complete) | EoSim |
| BLE PASE bearer | Example exists; GATT PASE service UNVERIFIED | Slice 1 audit (gap-map stage to complete) | eos / eNet |
| Hub app | Queue item only | App (gap-map stage to complete) | eApps |

## 9. Voice pointer (H6, scoping only — one section)

The plan §3.1 distinction only: Matter-local control (no
cloud endpoint, nothing for the org to operate) vs an Alexa
Smart Home Skill cloud endpoint (a separate, authenticated
surface; a product and operations commitment, not a protocol
task). No eVera voice capability is claimed here (the voice
service source has not been read for this spec); no Alexa
endpoint is proposed here. Full H6 scoping (local-voice
command path, capability-scoped per eSec Controls 7–9) is a
later slice that MUST read eVera's voice service source first.

## 10. Must NOT be claimed (checklist, part of this spec)

A future reader must not mistake this spec's targets for
shipments. This spec claims NONE of the following: a Matter
stack; a Thread stack; a Zigbee stack (beyond the datasheet
row + hub/coordinator first mode); a Z-Wave stack; an Alexa
endpoint; any certification (Matter, Thread, or otherwise);
any Thread 1.4 certification-cutoff date; any Matter 1.6.1
Groupcast/Chime deltas; any BLE PASE bearer existence claim;
any "supports Matter/Thread" shipped language; any change to
the site's smart-home-iot entry (it stays at `design()`
maturity until a bundle verifies — program step 7, not this
spec).
