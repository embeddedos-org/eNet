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
2026-09-16). Pin refresh 2026-11-11 (SH3): Thread 1.4
certification requirement — the Thread Group's
border-router-certification-only rule has been in force since
Jan 1, 2026 (deadline passed); Matter 1.6.1 certification lag
— no ledger entry states 1.6.1 yet, newest certified on the
ledger is 1.6 (research brief 2026-11-11 §4/§6).

Pin refresh 2026-11-21 (SH7): Espressif ESP32-C6 pinned as the certified platform for the eHub-Pro hub radio — the Thread border router + Zigbee bridge bearer (see §8.2). This pin names the radio platform only; the hub SoC stays the NXP i.MX 8M Mini (§1) and the Thread border router + Zigbee bridge roles (§2) are unchanged.

Use case, in one line: eHub-Pro as the household's Matter
controller + Thread border router + Zigbee bridge, commissioning
and controlling its own device family.

Pipeline position: smart-home-iot moves selected → specified on
this landing. The gap-map stage is COMPLETE as of 2026-11-15
(SH4, this slice): §8 is now the completed gap table, every row
grounded in evidence already in the repo or a cited first-party
source; rows that could not be verified are marked blocked/unknown
with the blocker named, never invented. Self-check status:
selected + specified; H4 (EoSim Thread border-router + Matter
commissioning sim) landed 2026-11-10, so the §7 chain can now be
walked in sim — the Verify-stage walk itself is a later slice and
is not claimed here.

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
| eSpeaker-AI | Wi-Fi 6, BLE | Voice front end (see §9) |

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
| Matter | 1.6.1 | target (certification lag: no ledger entry states 1.6.1 yet; newest certified is 1.6) |
| Thread | 1.4 (border router) | target (BR-certification-only rule in force since Jan 1, 2026) |
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
  v1 device set. Cluster IDs are pinned at gap-map stage in §8
  (§8.1): On/Off 0x0006 + Level Control 0x0008 and Thermostat
  0x0201 are first-party-verified; the doorbell/camera device
  types are UNKNOWN (no Camera/Doorbell device type in the CSA
  Device Library read) and stay unpinned.
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
(`examples/ble-sensor/` in eos), but the SH1 audit
(`docs/ble-commissioning-audit.md`; eNet #9 merged 2026-11-09)
has answered the audit question: eos BLE is declarations-only
— the API in `hal/include/eos/hal_extended.h:199-221` has no
GATT service/characteristic API and the only implementation
returns -1 for every BLE call. No GATT PASE bearer exists, and
real-hardware commissioning is blocked until eos ships a real
BLE backend plus a GATT API (and an nrf52 BSP). This spec
claims no BLE PASE bearer.

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

Self-check status: selected + specified. H4 landed 2026-11-10
(EoSim `docs/thread-border-router.md`: ThreadBorderRouter +
MatterCommissioning, three scenarios — `commission_device`,
`mesh_map`, `commission_timeout`); the §7 chain can now be walked
in sim. The Verify-stage walk itself is a later slice, not
claimed here. Second self-check target note: the
ESP32-H2 Matter end device (already in the self-check set)
is the natural SECOND target for this domain — the end-device
side of the same mesh — once the hub side specifies. It is
recorded as a walk addition, NOT a second selected use case:
one domain, one use case.

## 8. Gap map (COMPLETE 2026-11-15, SH4)

Gap-map stage complete: every row below is grounded in evidence
already in the repo or a cited first-party source, verified
2026-11-15. Rows that could not be verified are marked
blocked/unknown with the blocker named — nothing is invented.
Status key: OPEN = gap known, work not started; BLOCKED = a named
prerequisite must land first; UNKNOWN = the evidence to decide
does not exist yet; PARTIAL = a sim or partial piece exists, the
hardware/real piece does not.

| Area | Has today (evidence) | Missing | Owner repo | Status |
|---|---|---|---|---|
| Matter stack | Nothing shipped (home-systems plan §1; [matter-thread-note.md](matter-thread-note.md): "no stack, no port") | Full stack — Matter SDK or a subset; PASE/CASE via eSec; On/Off + LevelControl first (plan §2 step 5) | eos / eNet | OPEN — blocked on BLE prerequisite 1 + OpenThread port (see rows below) |
| Thread stack | Nothing shipped (same sources) | OpenThread port; ESP32-H2 is the natural first target (802.15.4 + BLE 5, no Wi-Fi); BR-certification-only rule in force since Jan 1, 2026, so any border-router claim is a Thread 1.4 claim | eos / eNet | OPEN |
| Matter application clusters (v1) | Pinned — see §8.1 | Doorbell/camera device types (UNKNOWN, see §8.1) | eNet | PARTIAL |
| MQTT client | Codec core, slice 1 (`eos/net/src/mqtt_codec.c`, 14,106 B) + slice 2 connect/disconnect state machine (eos #212; handle table, DISCONNECTED → CONNECTING → CONNECTED → DISCONNECTING, fake-broker tests in `eos/tests/test_net.c`) | Slice 3 — publish / subscribe / loop + keep-alive (`eos/net/src/net.c`: `TODO(#199 slice 3)`; publish, subscribe, unsubscribe, loop still return -1). No open tracker: eos #199 closed 2026-10-30 after slice 2, although its design doc (`eos/docs/eos-mqtt-client-design.md` §7) says it closes when slice 3 lands. Until slice 3 lands nothing sends PINGREQ, so a broker drops an idle connection after 1.5 × keepalive_sec (MQTT-3.1.2-24). | eos | OPEN — slice 3 needs a tracker |
| DAC / attestation | Requirements only (§5). eSec owns attestation, key management, device identity and secure storage as scope (eSec README); eos `services/crypto/` has AES + SHA-256/512 passing NIST vectors, RSA/ECC verification still stubs | Provisioning + certificate storage design — the eSec CY1 co-design, still pending (EoSim `docs/thread-border-router.md`: "Device identity provisioning is the eSec CY1 co-design, still pending") | eSec + eNet | BLOCKED on CY1 co-design |
| Radio simulation | H4 LANDED 2026-11-10 (EoSim `docs/thread-border-router.md`; `ThreadBorderRouter` + `MatterCommissioning` in `eosim/engine/native/peripherals/thread.py`; product simulator `thread_border_router` with scenarios `commission_device`, `mesh_map`, `commission_timeout`; `tests/unit/test_thread_border_router.py`) | Real 802.15.4 radio (explicitly not simulated); PASE cryptography (the passcode comparison is a sim placeholder); DAC behavior; NFC as a second commissioning bearer (documented, not implemented) | EoSim | PARTIAL — sim exists, hardware path still open |
| BLE PASE bearer | Audit answered (SH1, eNet #9, [ble-commissioning-audit.md](ble-commissioning-audit.md)): declarations-only, no GATT API; H4 SIMULATE decision decouples the sim from eos BLE | Real BLE backend + GATT API + nrf52 BSP in eos (prerequisite 1, still unmet) | eos / eNet | BLOCKED on prerequisite 1 |
| Hub radio hardware | Datasheet protocol rows only (Matter 1.6.1 target, Zigbee 3.0, Z-Wave 700, BLE 5.3, Wi-Fi 6, Thread 1.4 target); hub connectivity lists GbE + USB only. **SH7 pin (2026-11-21): Espressif ESP32-C6 is the certified platform for the hub radio** — the Thread border router + Zigbee bridge bearer (see §8.2) | Radio module/integration on the hub + certification evidence for the ESP32-C6 in this role — the platform is pinned, the module and its certification record are still to be specified | eCAD + eNet | OPEN — platform pinned 2026-11-21 (SH7); was UNKNOWN (blocker: eCAD datasheet radio-module line or a USB-radio decision) |
| Zigbee bridge | Hub/coordinator first mode chosen (§2); the bridge pattern exists for other protocols ([device-bridges.md](device-bridges.md): mcp2serial, mcp2mqtt, modbus) | No Zigbee coordinator/bridge implementation; native stack explicitly undecided; closing evidence (a named deployment the hub story cannot serve) does not exist | eNet | OPEN |
| Z-Wave | Datasheet row + [home-systems-plan.md](home-systems-plan.md) §3.3 research gap | Study in the shape of the LoRaWAN study (licensing + silicon + stack read from first-party sources); sequenced after plan §2 steps 1–2 | eNet | UNKNOWN — research gap |
| Hub app | Queue item only (§4, §7); no tracker found (eApps title search "hub": 0 hits), no code | The app itself | eApps | OPEN — blocked on the Matter lane (nothing to show yet) |
| Voice (H6) | Scoping done 2026-11-13 (eVera `docs/hub-voice-split-scoping.md`, eVera #87 / PR #86); local pipeline read (webrtcvad VAD, faster-whisper STT, "hey vera" wake word, pyttsx3 TTS); HA 2026.11 two-way audio is camera/doorbell talk-back only, not a voice-command path (§9) | Intent → Matter-cluster mapping (does not exist); capability-scoping per eSec Controls 7–9 | eVera + eNet | OPEN |

### 8.1 Matter application clusters — pinned IDs (first-party)

Cluster IDs pinned from the CSA Matter Device Library
Specification R1.1, document 22-27351 (Board-accepted May 17,
2023; csa-iot.org). The IDs are ZCL-heredity values stable across
Matter revisions; the citation is the 1.1 Device Library because
it is the CSA document read first-party for this slice — a
1.6.1 Device Library re-read is still open for the
doorbell/camera question below.

| Device / surface | Device type | Clusters (server, mandatory unless noted) |
|---|---|---|
| On/Off Light (the plan §2 step 5 smallest certifiable surface) | — | On/Off 0x0006 (M); Level Control 0x0008 (O) — §4.1.4 |
| eThermostat | Thermostat 0x0301 | Thermostat 0x0201 (M) — §9.2.4 |
| eSpeaker-AI (volume only) | Speaker 0x0022 | Volume control via the Speaker endpoint — §10.4 |

UNKNOWN — eDoorbell / eCamera-Home: no Camera or Doorbell
device type exists in the CSA Device Library R1.1 read for this
slice (text search of the document returns no match for either).
Their Matter device types and clusters stay UNPINNED. Blocker:
read the Matter 1.6.1 Device Library for camera/doorbell device
types before pinning anything for the doorbell or the camera. Do
not map them to Door Lock or any other device type by analogy.

### 8.2 Certified platform pin — Espressif ESP32-C6 (SH7, 2026-11-21)

- **Pin:** Espressif ESP32-C6 is the certified platform for the eHub-Pro hub radio — the silicon carrying the Thread border router (§2, Thread 1.4) and the Zigbee bridge (§2, hub/coordinator first mode).
- **Scope of the pin:** the radio platform only. It does not change the hub SoC (NXP i.MX 8M Mini, §1), the Matter 1.6.1 / Thread 1.4 target lock (§3), or the Thread border router + Zigbee bridge roles (§2).
- **What this does not claim:** no certification is claimed by this pin — no Matter or Thread certification record for the ESP32-C6 in this hub role is cited here, and none is asserted. The module/integration and its certification evidence are still to be specified (see the Hub radio hardware row above).
- **No conflict:** §8 named no other certified platform before this pin — the Hub radio hardware row previously recorded the radio as unspecified (UNKNOWN). Nothing was replaced.
- **Bring-up contract (SH9, 2026-11-23):** the hub-radio bring-up contract for this pin is [ehub-pro-c6-radio-bringup.md](ehub-pro-c6-radio-bringup.md) — C6 as 802.15.4 RCP to the i.MX 8M Mini host, esp-thread-br as reference, Thread 1.4 + Matter 1.6.1. On its merge, smart-home-iot moves gap-mapped → building.

## 9. Voice pointer (H6, scoping done 2026-11-13 — one section)

The plan §3.1 distinction, now recorded as the H6 scoping
outcome: Matter-local control (no cloud endpoint, nothing for
the org to operate) vs an Alexa Smart Home Skill cloud endpoint
(a separate, authenticated surface; a product and operations
commitment, not a protocol task). No Alexa endpoint is proposed
here.

H6 voice-split scoping (SH3, 2026-11-11; research brief §6;
scoped 2026-11-13 in eVera `docs/hub-voice-split-scoping.md`,
eVera #87 / PR #86): eVera's voice service source HAS now been
read — local pipeline (webrtcvad VAD, local faster-whisper STT,
"hey vera" wake word by transcription + fuzzy match, session
state machine, local pyttsx3 TTS with optional cloud edge-tts;
mobile push-to-talk `voice.ts`). That local pipeline is the
candidate Matter-local voice command path; the intent →
Matter-cluster mapping it needs does not exist yet, and any
command it issues must be capability-scoped per eSec Controls
7–9. The Alexa cloud-endpoint path is explicitly NOT proposed.

Home Assistant 2026.11 correction (drift fixed 2026-11-13):
two-way audio SHIPPED in 2026.11 (released 2026-11-04,
first-party release notes,
`home-assistant.io/blog/2026/11/04/release-202611/`) as a Labs
preview — enable in Settings > System > Labs — not "(Labs)"
as an unshipped label. Scope limit, from the same release
notes: it is camera/doorbell talk-back (a talk button on a live
stream), NOT a voice-command path, and it covers Matter cameras
and Reolink cameras over RTSP only, over HTTPS only. It is
evidence a hub-local voice path can exist without a cloud
endpoint; it is not the Matter-local voice command path.

## 10. Must NOT be claimed (checklist, part of this spec)

A future reader must not mistake this spec's targets for
shipments. This spec claims NONE of the following: a Matter
stack; a Thread stack; a Zigbee stack (beyond the datasheet
row + hub/coordinator first mode); a Z-Wave stack; an Alexa
endpoint; any certification (Matter, Thread, or otherwise);
any Matter 1.6.1 Groupcast/Chime deltas; any BLE PASE bearer existence claim;
any "supports Matter/Thread" shipped language; any change to
the site's smart-home-iot entry (it stays at `design()`
maturity until a bundle verifies — program step 7, not this
spec).


