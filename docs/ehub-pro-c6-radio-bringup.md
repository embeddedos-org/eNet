# eHub-Pro hub-radio bring-up contract (SH9)

Status: contract 2026-11-23 (SH9, growth backlog; first build-stage
slice for smart-home-iot). Refs eNet #7 (#7 stays open across the
track's slices by design). Home repo: eNet, as owner of the
home-systems track. Parent spec:
[ehub-pro-spec.md](ehub-pro-spec.md) (§8.2 pins the platform;
this document is the bring-up contract for that pin). Gate:
SH7 merged 2026-11-23 as eNet #20 (merge `f3be912e`,
Closes #21) — ESP32-C6 pinned as the certified platform for
the hub radio. This contract names the silicon, the wire, and
the host side; it builds no radio and claims no certification.

Pipeline position: on merge of this contract, smart-home-iot
moves gap-mapped → building — the first build-stage slice.
The radio row of the spec §8 gap map (Hub radio hardware)
moves OPEN → PARTIAL: the platform is pinned (SH7) and the
bring-up contract now exists (SH9); what remains is the
module/integration on the hub, the host-side daemon, and the
hardware bring-up itself.

## 1. Decision in one paragraph

The ESP32-C6 runs OpenThread Radio Co-Processor (RCP)
firmware and carries the 802.15.4 radio only. The NXP i.MX 8M
Mini hub SoC (Linux) runs the full OpenThread stack plus the
OpenThread Border Router daemon and the Matter controller.
The two are joined by a serial link carrying the Spinel
protocol — UART first (460800 baud, the esp-thread-br
default), SPI as the latency-preferred alternative. This is
the RCP split: the radio is dumb, the host is smart, and the
contract between them is Spinel. The single-SoC C6 hub
alternative was evaluated and is NOT the default (§10).

## 2. Architecture

```
                i.MX 8M Mini (Linux host)                ESP32-C6 (RCP)
  ┌──────────────────────────────────────────┐   ┌──────────────────────┐
  │ OpenThread stack (host)                  │   │ OpenThread RCP fw  │
  │ OpenThread Border Router daemon          │   │  (ot_rcp example)  │
  │ Matter controller (see §6)               │   │  802.15.4 radio    │
  │ IPv6 forwarding + mDNS / SRP             │   │  only — no Thread  │
  │ Dataset provisioning + RCP update        │   │  stack, no Wi-Fi,  │
  │ Hub app + MQTT cloud chain (later)       │   │  no Matter         │
  └───────────────┬──────────────────────────┘   └──────────┬───────────┘
                  │ Spinel over HDLC (UART)                │
                  │ or Spinel over SPI (preferred)         │
                  │ + RESET / BOOT for RCP update          │
                  └────────────────────────────────────────┘
  Home IP network (GbE / Wi-Fi 6) ◄── host ──► Thread mesh (802.15.4)
```

Why this split, in the spec's own terms: the hub SoC is
Linux-capable (§1 of the spec) and already carries GbE, the
hub app, and the MQTT cloud chain (§7); putting the Thread
stack on the host keeps the Matter controller, the border
router, and the cloud chain in one place, under one OS, with
one update story. The C6 contributes exactly one thing — a
certified 802.15.4 radio — and contributes it without
sharing its RF path with Wi-Fi, because in this split the C6
runs no Wi-Fi at all.

## 3. Version pins

| Piece | Pin | Source (first-party unless noted) |
|---|---|---|
| Matter | 1.6.1 | CSA Core Specification document 23-27349, accepted by the CSA Board 2026-09-16 (spec §3 target lock; unchanged by this contract) |
| Thread | 1.4 (border router) | Thread Group, formally released Sep 4, 2024; BR-certification-only rule in force since Jan 1, 2026 (spec §3; unchanged) |
| ESP32-C6 platform | ESP32-C6 as the certified platform for the hub radio | Spec §8.2 (SH7, eNet #20 merged 2026-11-23) |
| ESP-IDF (RCP firmware build) | v5.5.6 (recommended by the esp-thread-br docs at read time) | ESP Thread BR docs, Build and Run (docs.espressif.com, read 2026-11-23); the ESP-IDF v6.1.1 OpenThread guide was also read the same day and describes the same RCP mode — the v5.5.6 pin follows the esp-thread-br recommendation, not the newest IDF |
| RCP firmware | `esp-idf/examples/openthread/ot_rcp`, target `esp32c6` | ESP-IDF OpenThread guide (docs.espressif.com, v6.1.1, read 2026-11-23): RCP mode is available on ESP32-H2 and ESP32-C6; transport SPI or UART |
| Border-router reference | esp-thread-br (Espressif ESP Thread Border Router SDK) — reference for the RCP side and the BR feature set, NOT a drop-in host stack (see §4) | esp-thread-br README + Build and Run docs (docs.espressif.com, read 2026-11-23) |
| OpenThread release | NOT pinned here | The OpenThread version rides with the ESP-IDF-bundled OpenThread at build time; pin it at implementation from the IDF release notes, do not assert a version in this contract (see §11) |
| Host-side border-router daemon | NOT pinned here | The Linux host runs the OpenThread Border Router daemon (otbr-agent) against the RCP; the otbr release to track is an implementation decision (§5), not asserted here |

No row in this table is `shipped` or `supported`. Every pin
is a design target, not a stack that exists.

## 4. Reference implementation — esp-thread-br, and its limit

esp-thread-br is the official ESP Thread Border Router SDK,
built on ESP-IDF and OpenThread, and it is the reference for
this contract's RCP side: it defines the two-SoC shape (a
Wi-Fi host SoC running the Thread stack + an 802.15.4 SoC
running RCP firmware), the RCP firmware source (`ot_rcp`),
the default UART0 460800-baud link, the RESET/BOOT pins for
RCP update, and the border-router feature set (bi-directional
IPv6 connectivity, service discovery delegate, service
registration server).

Its limit, stated plainly because it decides the host work:
**esp-thread-br's host is an ESP32 Wi-Fi SoC running
ESP-IDF, not a Linux host.** Its reference board is an
ESP32-S3 host + ESP32-H2 RCP. The eHub-Pro host is a Linux
i.MX 8M Mini. So esp-thread-br is the reference for what the
RCP must do and what the border router must provide — it is
not the host stack this hub will run. The Linux host stack
is the OpenThread Border Router daemon against the same RCP
(§5). Any sentence that presents esp-thread-br as running on
the i.MX host is wrong; this contract does not make that
claim.

What transfers from esp-thread-br to this hub, unchanged:
the RCP firmware build (`ot_rcp`, target `esp32c6`), the
Spinel-over-serial contract (§5), the 460800-baud UART
default, the RESET/BOOT RCP-update wiring, and the
border-router feature checklist. What does not transfer: the
ESP-IDF host application, the ESP32-S3 host SoC, and the
Wi-Fi-based backbone link (the hub's backbone is GbE, per
spec §1).

## 5. RCP ↔ host protocol and wiring

### 5.1 Protocol: Spinel

Spinel is the host–co-processor protocol: the host sends
commands, the RCP replies with responses or asynchronous
notifications. In the RCP design the full Thread stack runs
on the host; the RCP performs timing-sensitive MAC/PHY
operations only (OpenThread co-processor designs,
openthread.io, read 2026-11-23). The RCP presents a defined
Spinel command/property interface; the host configures the
radio, sends and receives 802.15.4 frames, and controls
Thread operations through it. The protocol ships with
OpenThread at `src/lib/spinel`; Pyspinel is the test CLI.

### 5.2 Framing and transport

- **UART (default):** Spinel frames are HDLC-encoded over
  UART. Default baud 460800 (esp-thread-br default). Hardware
  flow control (CTS/RTS) is REQUIRED at 460800 — a 2-wire
  UART at that rate is a known failure mode (§8). Fallback
  baud 230400 if the link is unreliable on the hub's wiring
  (the esp-thread-br docs record exactly this reduction for
  long cables / weak drive).
- **SPI (preferred for latency):** Espressif's own guidance
  is explicit — "for the sake of latency, we recommend using
  SPI as the underlying transport" (ESP-IDF OpenThread guide,
  v6.1.1). SPI is the preferred transport for this hub; UART
  is the default because it is the reference's default and the
  simplest to bring up first. The bring-up sequence is:
  UART first (prove the Spinel link), SPI second (prove the
  latency-preferred path), then pick one for the module.
  Both transports carry Spinel; the choice is wiring + driver,
  not a protocol change.
- **RCP update wiring:** RESET and BOOT pins from the host
  to the C6, per the esp-thread-br board pattern — the host
  must be able to put the C6 into download mode and re-flash
  RCP firmware without physical access. A module without
  host-driven RESET/BOOT does not meet this contract.
- **Coexistence:** no Wi-Fi/15.4 coexistence handling is
  needed on the C6 in this split — the C6 runs no Wi-Fi.
  (The 3-wire PTA coexistence in the esp-thread-br board
  exists because its host SoC and RCP SoC share RF
  neighbourhood on an ESP32 host; it does not apply here.)

### 5.3 Host side — i.MX 8M Mini responsibilities

The host owns everything the RCP does not:

1. **OpenThread stack + border-router daemon** — runs the
   full Thread stack and the OpenThread Border Router daemon
   (otbr-agent) against the RCP over Spinel. Provides IPv6
   connectivity between the Thread mesh and the home IP
   network, service registration (SRP), and service discovery
   (mDNS) delegation.
2. **Serial link driver** — the UART (or SPI) device node,
   baud/flow-control configuration, and the Spinel/HDLC
   framing on the host side. The device node path is a
   bring-up fact (record it when the module lands); this
   contract does not invent one.
3. **Dataset provisioning** — generates and stores the Thread
   network dataset (network key, PAN ID, channel, mesh-local
   prefix). The dataset MUST be freshly generated per hub —
   never ship the example/default dataset (community reports
   note the ESP-IDF `ot_br` example's default network key
   ships with every copy of ESP-IDF — secondary-reported, re-
   verify against the example before relying on it; the rule
   stands regardless: never ship an example dataset). Dataset storage lives with the hub
   credential store (see the SH5 token-audit note in the
   growth backlog).
4. **RCP firmware update** — flashes RCP firmware to the C6
   via RESET/BOOT on first boot and on update, mirroring the
   esp-thread-br flow (RCP image included in the host image
   and flashed to the RCP on first boot). RCP firmware is
   signed like any other firmware the hub ships (foundation
   signing story; a bitstream/firmware-blob distinction does
   not apply — the RCP image is code).
5. **Watchdog + recovery** — detects a dead Spinel link
   (§8) and recovers it without operator action where
   possible.
6. **Matter controller** — commissions and controls Matter
   devices over the Thread mesh (§6). The controller
   implementation on the Linux host (connectedhomeip
   chip-tool lineage vs an EoS-native controller) is an
   implementation decision, NOT pinned here — no controller
   exists in the org today, and this contract does not invent
   one.
7. **Zigbee bridge (first mode)** — the hub/coordinator story
   (§7). The C6 is the Zigbee radio candidate; the coordinator
   software runs on the host.

What the host does NOT do: run the 802.15.4 MAC/PHY (that is
the RCP's job), or speak to Thread devices except through
the RCP and the border router.

## 6. Commissioning path

The commissioning sequence is the spec §5 sequence (PASE →
attestation → NOC → CASE, BLE bearer first), with the radio
leg now named:

1. **BLE PASE bearer (gated):** the BLE bearer is the plan's
   first bearer, but the SH1 audit (eNet #9) answered that
   eos BLE is declarations-only — no GATT API, no PASE bearer
   on real hardware. Real-hardware Matter commissioning over
   BLE is therefore BLOCKED on the eos BLE backend + GATT API
   + nrf52 BSP (spec §8, BLE PASE bearer row). This contract
   does not unblock it and does not claim otherwise.
2. **Sim path (exists today):** the H4 sim (EoSim, landed
   2026-11-10) walks commissioning in simulation
   (`commission_device` scenario). The SH9 bring-up
   acceptance tests (§9) are written so the sim path can
   carry them until hardware exists.
3. **Thread join:** once PASE completes, the device joins the
   Thread mesh through this border router: dataset
   provisioning (§5.3.3), mesh attach, SRP registration,
   then CASE session establishment with the Matter
   controller on the host.
4. **DAC / attestation:** requirements only (spec §5) —
   provisioning + certificate storage is the eSec CY1
   co-design, still pending (spec §8, BLOCKED). The radio
   bring-up does not depend on DAC; commissioning of
   production devices does.
5. **Credential sharing:** Thread credentials are shared to
   the commissioner's phone per Thread 1.4 credential
   sharing (the H4 sim models this shape). The exact
   sharing UI is a hub-app concern, not this contract's.

## 7. Zigbee bridge — the second radio role

The C6 is also the Zigbee bridge bearer (spec §2, hub/
coordinator first mode; §8.2 pin scope). What this contract
does and does not say:

- **Said:** the C6's 802.15.4 radio is the shared radio for
  Thread and Zigbee; the hub/coordinator story keeps the
  Zigbee coordinator software on the host and the radio on
  the C6, mirroring the Thread RCP split. One radio, two
  protocols — time-shared, never simultaneous on the same
  channel.
- **NOT said (unverified):** that the C6 Zigbee coordinator
  firmware, its coexistence with the Thread RCP role on one
  chip, or the channel/time-share policy are solved. The
  esp-thread-br docs mention a Zigbee Gateway sub-board in
  their Ethernet variant, which suggests a Zigbee gateway
  shape exists in that ecosystem — but that shape was NOT
  read first-party for this slice, and no Zigbee coordinator
  behavior is asserted here. The Zigbee-on-C6 validation
  (coordinator firmware source, Thread/Zigbee time-share,
  channel plan) is an open follow-up, recorded here rather
  than claimed.

## 8. Failure modes and fallback

| Failure | Detection | Fallback / safe state |
|---|---|---|
| Spinel link dead (UART/SPI silent) | Host daemon heartbeat / Spinel timeout | Host resets the C6 via RESET pin, re-attaches; if re-attach fails after N retries (N set at implementation), hub reports the Thread mesh DOWN and stops advertising the border router — it does not pretend the mesh is up |
| RCP firmware mismatch (host expects a newer RCP image) | Version handshake at attach | Host re-flashes the RCP via BOOT mode (§5.3.4); if flashing fails, hub stays in "radio unavailable" state — no Thread service is offered |
| Host daemon crash (otbr-agent down) | Process supervisor / systemd | Restart the daemon; re-attach to the RCP; re-form or re-attach to the existing dataset (never silently generate a new dataset — a new dataset orphans every commissioned device) |
| Thread mesh partition (devices unreachable) | Mesh map / SRP expiry (H4 sim models the shape) | Hub reports affected devices unreachable; no synthesized state, no fabricated readings — the hub app shows "offline", never a stale value presented as live |
| Dataset loss (host storage failure) | Dataset load failure at boot | Hub refuses to form a new mesh automatically; operator re-provisioning is required. Forming a fresh network silently would orphan commissioned devices |
| C6 hardware fault (radio unresponsive after reset) | RCP attach failure after reset + re-flash attempt | Hub reports radio fault; Thread + Zigbee legs down; Wi-Fi/Ethernet legs unaffected; the failure is logged with the RCP firmware version and link parameters for diagnosis |
| Power loss mid-flash (RCP update interrupted) | Boot-mode check on next attach | C6 download mode + re-flash from the host image; the host image always carries a known-good RCP image for exactly this case |

Safe-state rule for the whole hub, carried from the gap map:
no synthesized data, no control output from a stale reading,
no silent re-provisioning. A hub that cannot reach its mesh
says so.

## 9. Acceptance criteria — what "brought up" means

The radio is brought up when ALL of the following are
observable — in sim first (H4), on hardware when the module
lands. Each criterion names its observable, not a feeling:

1. **RCP attach:** the host daemon reports the RCP firmware
   version and Spinel protocol version on attach; the link
   survives 24 h of idle + traffic without a reset.
2. **Dataset formed:** a freshly generated dataset (never the
   example default) is stored; the mesh-local prefix is
   reachable from the host (`ping6` to the mesh-local
   address succeeds through the border router).
3. **Device join:** a Thread end device (ESP32-H2 Matter end
   device in the self-check set, or the H4 sim device)
   joins the mesh, registers via SRP, and appears in the
   mesh map.
4. **Cross-network reachability:** an IPv6 host on the home
   LAN reaches the joined device through the border router,
   and the device reaches the LAN host — bi-directional, not
   one-way.
5. **Matter commissioning (sim now, hardware when the BLE
   bearer exists):** the H4 `commission_device` scenario
   passes against this contract's dataset shape; on hardware,
   a Matter-over-Thread device commissions via the §6 path
   once the BLE bearer unblocks.
6. **RCP update:** the host re-flashes the C6 RCP firmware
   via RESET/BOOT and re-attaches with no physical access.
7. **Failure drill:** killing the Spinel link recovers via
   §8 (reset + re-attach) with the mesh re-forming and
   previously commissioned devices re-attaching — no silent
   new dataset, no orphaned devices.
8. **Zigbee (follow-up, not a gate):** §7 validation lands as
   its own slice; the Thread bring-up above is NOT gated on
   it.

Until criterion 5 runs on hardware, the honest status is
"brought up in sim, hardware bring-up pending" — the sim
leg (H4) is real, the hardware leg is not yet run.

## 10. Alternative evaluated — the single-SoC C6 hub

The alternative: run the whole hub on one ESP32-C6 — full
OpenThread stack + border router on the C6 itself
(`esp-idf/examples/openthread/ot_br`, native 802.15.4 radio,
Wi-Fi/Thread coexistence on by default for the C6), with the
C6 as the hub. This is a real, demonstrated shape: ESPHome
2026.11.0 ships an experimental single-chip ESP32 Thread
border router, and community builds pair Matter devices
through it in Home Assistant. It was evaluated honestly and
is NOT the default. Gains and losses:

**What it gains**

- One chip instead of two: lower BOM, simpler hardware,
  no host↔RCP serial link to wire or debug.
- Proven quickly: `ot_br` is an in-tree ESP-IDF example;
  ESPHome's experimental OTBR shows the shape works for
  small networks.
- No Linux host needed for the radio leg at all.

**What it loses**

- **The RF path is shared.** The C6 has one radio: Wi-Fi
  and Thread take turns and cannot both receive at the same
  moment. The `ot_br` README is reported to recommend two
  chips (a Wi-Fi SoC running `ot_br` plus an H2 RCP) —
  secondary-reported via a community build report; re-verify
  against the README at implementation — and Espressif does
  not recommend the single-chip shape for production or
  larger networks (secondary-reported community experience,
  consistent with the first-party two-chip recommendation —
  the "not for production" wording itself is secondary and
  is labelled as such).
- **The hub stops being a hub.** The eHub-Pro is defined by
  its Linux host: GbE, the Matter controller, the hub app,
  the MQTT cloud chain, 2 GB of RAM for local analytics
  (spec §1, §4, §7). A C6 has no Ethernet, and no room for a
  Matter controller + hub app + cloud chain alongside a
  Thread stack. The single-SoC shape is a border router, not
  a household Matter controller + bridge + app.
- **The certification path changes.** The SH7 pin inherits
  certification through the C6 as a radio platform; a
  single-SoC C6 hub would need its own Matter controller
  certification story, which does not exist in this program.
- **The Zigbee leg gets harder, not easier.** One radio
  already time-shares Thread and Zigbee in the RCP split;
  adding Wi-Fi to the same radio is a third time-share
  claimant on one RF path.

**Decision:** the RCP split stays the default. The
single-SoC C6 hub is recorded as a legitimate fallback for
a minimal border-router-only product — if a future product
needs only a Thread border router with no Matter controller,
no hub app, and no cloud chain, the `ot_br` shape is the
starting point, not this contract. It is not the eHub-Pro.

## 11. Sources and unverified list

**Read first-party for this slice (2026-11-23)**

- ESP-IDF Programming Guide v6.1.1, OpenThread — ESP32-C6
  (docs.espressif.com): RCP mode on ESP32-H2/C6; transport
  SPI or UART, SPI recommended for latency; `ot_br` example
  and `esp_openthread_border_router_init()`.
- ESP Thread BR docs, Build and Run (docs.espressif.com,
  latest): recommended ESP-IDF v5.5.6; RCP image from
  `ot_rcp`; default UART0 460800 baud; RCP flashed on first
  boot; basic_thread_border_router example.
- esp-thread-br README (Espressif, via GitHub raw):
  official ESP Thread Border Router SDK; two-SoC Wi-Fi
  design; UART + SPI, RESET/BOOT for RCP update, 3-wire PTA;
  `ot_br` example recommended for standalone modules.
- OpenThread co-processor designs (openthread.io): RCP
  keeps only a minimal MAC layer on the radio; host ↔ RCP
  via Spinel; Spinel ships at `src/lib/spinel`.
- eNet spec at live master `f3be912e` (post-#20):
  §8.2 C6 pin, §8 gap map, §5 commissioning caveats.

**Carried from the spec (already first-party-verified
there, not re-read here)**

- Matter 1.6.1 (CSA 23-27349, Board-accepted 2026-09-16);
  Thread 1.4 (Thread Group, Sep 4, 2024); BR-certification-
  only rule in force since Jan 1, 2026.

**Secondary-reported (labelled, not asserted as fact)**

- ESPHome 2026.11.0 experimental single-chip ESP32 OTBR
  (ESP32-C5/C6 native, external-RCP mode hung in soak) —
  from the 2026-11-20 research brief, first-party ESPHome
  notes cited there.
- "Espressif doesn't recommend single-chip for production
  or larger networks" — community build reports; consistent
  with the first-party two-chip recommendation in the
  `ot_br` README, but the production wording itself is
  secondary.

**Unverified — explicitly not claimed**

- OpenThread release version to pin (rides with ESP-IDF;
  pin at implementation).
- Host-side otbr-agent release and packaging on the i.MX
  8M Mini (implementation decision).
- Matter controller implementation on the Linux host
  (chip-tool lineage vs EoS-native; no controller exists
  in the org today).
- Zigbee coordinator firmware on the C6, Thread/Zigbee
  time-share, and channel plan (§7 follow-up).
- Any certification record for the ESP32-C6 in this hub
  role (none cited; none asserted — SH7 pin inherits, it
  does not certify).
- UART vs SPI final choice for the hub module (UART first
  for bring-up, SPI preferred for latency; pick at module
  time).
- The hub module itself: part number, antenna, USB vs
  direct-UART/SPI wiring to the i.MX 8M Mini, power — all
  TBD at module specification (eCAD).

## 12. Must NOT be claimed (checklist, part of this contract)

This contract claims NONE of the following: a Matter stack;
a Thread stack on hardware; a Zigbee stack; a BLE PASE
bearer on hardware; any certification (Matter, Thread, or
otherwise); esp-thread-br running on the i.MX host; a pinned
OpenThread or otbr-agent version; a Matter controller
implementation; a hub module part number; any "supports
Matter/Thread" shipped language. The radio is contracted,
not built. The next slice is the module specification
(eCAD) + host daemon bring-up — not a claim that either
exists.

— av_mando
