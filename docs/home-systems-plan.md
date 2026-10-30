# Home-systems plan — protocols and platforms, sequenced

Status: scoping doc for the home-systems track (tracker: eNet#7).

Target lock 2026-10-30: Matter 1.6.1 (CSA Core Specification document 23-27349, accepted by the CSA Board 2026-09-16); Thread 1.4 is the border-router target. These are TARGETS for the sequence in §2, not shipped stacks: the org has no Matter stack, no Thread stack, and no Alexa endpoint today.
Date: 2026-10-24. This plan turns the track's two planning notes
and one study into a sequenced programme. It commits the org to
an order of work, not to a stack: where a choice is still open,
this doc records it as a decision input with options, and says
what evidence would close it.

Seeds (in-repo authority for everything below):

- [Matter / Thread landscape note](matter-thread-note.md) — the
  stack, the silicon, the five commissioning prerequisites, the
  sequencing.
- [Sub-GHz three-way plan](subghz-three-way-plan.md) — HaLow /
  Thread Sub-GHz / Zigbee "Suzi" and the coexistence work.
- [LoRaWAN study](lorawan-study.md) — the adjacent long-range
  lane, out of this track's scope but sharing its radio
  questions.

## 1. Inventory, split by kind

### Protocols (the wire)

| Protocol | State in the org today | Lane in this plan |
| --- | --- | --- |
| Thread / Matter | Planning note only; no stack, no port | **First lane** — §2's sequence |
| Zigbee | Named in the sub-GHz plan (2.4 GHz installed base + sub-GHz "Suzi"); no stack, no coordinator story | Decision input — §3 |
| Z-Wave | Named in the 2026-10-24 directive; no note, no stack | Decision input — §3 (research gap) |
| Sub-GHz generally | Three-way plan: HaLow / Thread Sub-GHz / Zigbee "Suzi" share 868/915 MHz | Coexistence work (§4) precedes any dual-stack claim |

### Platforms (the ecosystems)

Platforms integrate **through the API layer**, not through an
agent: a platform integration is a programmatic surface a
third-party ecosystem calls, per the org's MCP/API rule.

| Platform | Integration surface (API layer) | Depends on |
| --- | --- | --- |
| Alexa | Two distinct surfaces — §3.1 | Matter lane, or a cloud endpoint decision |
| Google Home | Matter (local) / Google Home APIs | Matter lane |
| Apple HomeKit | Matter (local) / HomeKit accessory protocol | Matter lane |
| Home Assistant | Matter (local) / MQTT + API integrations | Matter lane; eos MQTT client (eos#199) for the MQTT surface |
| SmartThings | Matter (local) / SmartThings APIs | Matter lane |

The pattern in the table is the plan's core claim: **four of
five platforms, and Alexa's local surface, are reached through
the same substrate — Matter.** That is why §2 sequences Matter
first and treats the platform-specific cloud surfaces as later,
separate decisions.

## 2. The Matter-first sequence

Quoted in substance from the landscape note, whose warning this
plan carries as a rule: **do not attempt the full stack at
once.** Each step is independently shippable and testable:

1. **BLE commissioning bearer** — Matter commissioning happens
   over BLE (GATT): advertising the discriminator, the GATT
   service for PASE. eNet's BLE story must support the
   commissioning window before anything above it can exist.
2. **OpenThread port** — for at least one 802.15.4-capable
   target; the note names ESP32-H2 as the natural first target
   (802.15.4 + BLE 5, no Wi-Fi — a pure 15.4 end device).
   ESP32-C5 (15.4 + Wi-Fi 6 + BLE) is the second silicon in the
   note; the nRF54 series is the Nordic option.
3. **Simulated border router in EoSim** — Thread meshes need a
   border router to reach IP networks; EoSim simulates one
   before hardware exists, so steps 4–5 can be developed
   against it. The simulated border router targets Thread 1.4. Thread 1.4 is the target version; any certification-cutoff claim is out of scope for this lock until verified against a Thread Group certification notice.
4. **PASE/CASE via eSec** — session establishment, plus the
   Device Attestation Certificate (DAC) chain commissioning
   requires. This is eSec's domain (key provisioning,
   certificate storage): eNet and eSec design it together.
5. **Minimal application** — On/Off + LevelControl clusters
   (lights), the smallest certifiable surface, before any
   sensor clusters.

One dated external note, recorded because it widens step 1's
options later (first-party, Connectivity Standards Alliance,
Matter 1.6 announcement): as of Matter 1.6, **NFC is an
additional commissioning bearer alongside BLE**. It does not
change the BLE-first sequence above; it is a future bearer
option for step 1. The plan's target is now Matter 1.6.1 (see target lock above). 1.6.1 does not change §2's five prerequisites and does not change the BLE-first sequence; NFC remains a future bearer option, not a reason to delay BLE-first. The detailed commissioning step sequence
(PASE → attestation → NOC → CASE) is **not** specified in this
plan: it was not verified against a first-party CSA source at
writing time, and the landscape note's five prerequisites are
the in-repo authority until it is. Closing that gap is a
research task for the slice that reaches step 4.

## 3. Decision inputs (not decisions)

### 3.1 Alexa's two surfaces

- **Matter (local):** an EoS Matter device is controllable by
  Alexa through the local Matter fabric — no cloud endpoint,
  no skill, nothing for the org to operate. This surface comes
  free with §2's lane.
- **Alexa Smart Home Skill (cloud):** a skill needs a cloud
  endpoint — an API-layer service the org would host, with an
  account-linking story. That is a product and operations
  commitment, not a protocol task.

First-slice recommendation (input to Aswin, not a decision):
target the **Matter surface first** — it is the same work as
every other platform's local surface — and let the Skill
question wait until a device exists that Alexa cannot already
reach locally.

### 3.2 Zigbee

Two honest options, neither chosen here:

- **Hub/coordinator story:** EoS talks to existing Zigbee
  networks through a coordinator or bridge (the pattern eNet's
  device-bridge work already uses for other protocols). Small
  org-side surface; inherits the hub's limits.
- **Native stack:** an in-tree or ported Zigbee stack on
  802.15.4 silicon (the H2 is Espressif's Thread/Zigbee
  vehicle). Full control; the largest lift of any lane in this
  plan, and it competes with the Thread port for the same
  radio time on the same chips.

Evidence that would close it: a named deployment the hub story
cannot serve. Until one exists, Zigbee stays a lane with a
plan (§4's coexistence work covers its sub-GHz form), not a
stack commitment.

### 3.3 Z-Wave

**Research gap, recorded as such.** Z-Wave entered this track
through the 2026-10-24 directive; the org has no note on it,
and this plan makes no licensing, silicon or stack claim from
memory. The closing work is a study in the shape of the
LoRaWAN study (native backend pattern, hardware-bridge
pairing, licensing read from first-party sources) — a named
future slice, sequenced after §2's steps 1–2 exist, since any
Z-Wave position depends on the same radio and coexistence
answers.

## 4. The sub-GHz coexistence work (precedes any commitment)

From the three-way plan, required before eNet commits to any
dual-stack sub-GHz story:

- **Duty-cycle accounting** per band/region (EU 868 duty-cycle
  caps; NA 915 dwell-time rules).
- **Channel planning** so HaLow's wider channels do not deafen
  Suzi / Thread Sub-GHz reception.
- **Interference testing in EoSim** — simulate the mixed-band
  environment before it is built.

## 5. Neighbouring repos — one role each

- **EoSim** — radio and border-router simulation before
  hardware (§2 step 3, §4 interference testing).
- **eSec** — DAC attestation and PASE/CASE co-design (§2
  step 4); the security controls any exposed surface must meet.
- **eVera** — the on-device voice surface: the local endpoint
  a voice interaction lands on, once a device exists to hear it.
- **eos** — where the net implementation lives today
  (`eos/net/`); this repo is the component's design record.
  That split — eNet holds the docs, eos holds the code — is
  the current state, and it is input to Aswin's eNet
  merge-vs-keep call. This plan records it as a fact of the
  layout and makes no recommendation on it.

## 6. Modular-adoption check

Could a third party lift only the home-systems piece? The
piece is a set of API-layer contracts (§1's platform surfaces)
over the Matter lane — a lifter would need: the `eos` net API
(the contract in `net.h`, including the MQTT client tracked
as eos#199 for the Home Assistant MQTT surface), an OpenThread
port for their silicon, and eSec's attestation surface for
DAC/PASE/CASE. The coupling that honesty requires naming: the
attestation dependency is real design coupling, not an
accident — Matter commissioning cannot be lifted without *an*
attestation story, though nothing binds a lifter to eSec's
specifically if the contract is honoured. No other hidden
coupling was found in the notes; if a slice discovers one, it
is recorded here, not papered over.

## 7. First implementable slice (named, not landed here)

**Slice 1: BLE commissioning-bearer audit.** A read-only audit
of what BLE support actually exists today in `eos` (advertising,
GATT services, the `net.h` transport claims) against §2 step
1's requirements, written up as a gap note in this repo.
Preconditions: none — it reads the tree. Sized for one daily
run. It converts step 1 from a note's claim into a measured
gap list, which is what step 2's port then works against.

**Slice 2 (after slice 1): EoSim border-router simulation
spec** — the §2 step 3 design, in EoSim's repo, against the
audit's findings.

No protocol-stack or endpoint code is part of this scoping
slice, and none of §3's inputs is presented as decided.
