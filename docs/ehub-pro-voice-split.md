# eHub-Pro voice split scoping (H6)

Status: scoping 2026-11-24 (H6, growth backlog; smart-home-iot domain step).
Home repo: eNet (home-systems track owner). Voice surface owner: eVera.
Refs: eNet #7 (#7 stays open across the track's slices by design);
`docs/home-systems-plan.md` §3.1 (the distinction this scopes);
`docs/ehub-pro-spec.md` (hub spec); `docs/ehub-pro-c6-radio-bringup.md`
(SH9, eNet #22 — see open dependency below). No Alexa endpoint is
proposed by this document. Nothing here is `shipped` or `supported`.

## 1. The split, in one paragraph

Voice reaches an eHub-Pro home through two **distinct surfaces that must
not be conflated**: a **Matter-local** surface, where commands and talk-back
stay on the home network inside the Matter fabric, and an **Alexa Smart
Home Skill** surface, where a voice request travels through Amazon's cloud
to an endpoint the org hosts. The first is a protocol task that rides the
Matter lane the track is already building. The second is a product and
operations commitment — a hosted service, an account-linking story, and a
certification relationship — and it is **not decided here**.

## 2. Surface A — Matter-local

| Question | Answer |
|---|---|
| Transport | Local Matter fabric: Thread mesh (802.15.4) via the hub radio + home IP (GbE / Wi-Fi 6) to the controller. Device control is Matter CASE-session traffic on the LAN. Camera talk-back is WebRTC between browser and camera (see §4). |
| Auth | Matter fabric credentials: PASE commissioning, attestation, NOC, CASE. No Amazon account, no OAuth, no vendor cloud credential. |
| On-device vs cloud | On-device / on-LAN. The hub (i.MX 8M Mini host + ESP32-C6 RCP radio, per SH9) runs the Matter controller and border router; nothing leaves the home for a command to execute. |
| What the org operates | Nothing beyond the hub software itself. No hosted service, no account system. |
| What it requires | The SH9 radio row (open dependency, §5); the Matter controller on the hub host; the Matter 1.6.1 pin (spec §3); a Matter server recent enough for the device classes used (HA's own caveat for cameras, §4). |
| What it is for | Controlling EoS Matter devices from any Matter controller (Alexa included, as a fabric member — no skill, no cloud endpoint), and talking back through Matter cameras/doorbells. |

## 3. Surface B — Alexa Smart Home Skill endpoint (cloud)

| Question | Answer |
|---|---|
| Transport | User speaks to an Alexa device → Amazon cloud (wake-word + NLU in Amazon's cloud) → Alexa directive over HTTPS to a Smart Home Skill endpoint the org hosts → endpoint drives the hub/devices. The voice path leaves the home by design. |
| Auth | Amazon account linking (OAuth) between the user's Alexa account and the org's account system, plus Alexa skill certification. The org becomes an identity provider for its users. |
| On-device vs cloud | Cloud. The hub is a downstream actuator of a cloud decision; if the endpoint or the internet is down, voice stops even though the devices and the LAN are fine. |
| What the org operates | A hosted API-layer service (uptime, scaling, security patching, incident response), an account-linking service, and the Alexa certification relationship. This is an operations commitment with a recurring cost, not a one-time protocol task. |
| What it requires | A product decision (is Alexa voice a product the org ships?), a hosted-service design, account-linking + data-handling story, Alexa certification, and eSec Controls 7–9 capability scoping for any voice tool the MCP layer exposes (voice commands must be capability-scoped; MCP never exposes raw mic audio). |
| What it is for | Voice control for users who speak to Alexa as their primary interface, including commands the Matter-local surface cannot express — at the price of the cloud dependency. |

## 4. Live reference: Home Assistant 2026.11 two-way audio — a Labs preview, not a shipped surface

**Correction recorded here:** the scoping brief for this step described
HA 2026.11 two-way audio as "shipped, not Labs". That is wrong. Verified
against the first-party source — Home Assistant release notes 2026.11,
"2026.11: A lot in store!", 2026-11-04
(https://www.home-assistant.io/blog/2026/11/04/release-202611/, §"Talk
back through your cameras") — two-way audio is **"a new Labs preview
feature"**, enabled in Settings > System > Labs. It is live and usable,
but it is explicitly a preview, and this document treats it as such.

What the release notes establish (first-party, read 2026-11-24):

- Matter 1.5 cameras appear in Home Assistant with live video and
  snapshots, streamed with WebRTC directly between browser and camera.
- The Labs two-way-audio preview adds a talk button to the live stream,
  so you can talk to whoever is near the camera or doorbell.
- At launch it covers Matter cameras and Reolink (RTSP only — RTMP/FLV
  cannot carry audio back to the camera).
- Browsers allow microphone access only over a secure connection, so
  two-way audio works over HTTPS for now; the HA apps over HTTP are
  still in progress.
- Matter cameras need a Matter server recent enough to support them.

What this reference is, and is not, for the eHub-Pro:

- **It is** the live Matter-local talk-back reference: a concrete,
  shipping-in-preview demonstration that local talk-back to a Matter
  camera works over WebRTC with no vendor cloud in the audio path.
  The hub's doorbell/camera talk-back should be judged against this
  shape (local WebRTC audio, HTTPS-gated browser mic, preview-grade
  maturity).
- **It is not** a voice-command reference. Talk-back is push-to-talk
  audio to a person near a camera; it is not a voice assistant that
  parses commands. Matter defines no general voice-command cluster
  for arbitrary commands, and this document claims none. Do not cite
  HA two-way audio as evidence that "Matter does voice commands".
- **It is not** a stable surface to build a committed feature on yet:
  Labs-preview status means the feature can change shape or
  requirements. The bundle may prototype against it; it may not
  promise it.

## 5. Open dependency: eNet #22 (SH9 radio bring-up) is still open

At writing time (2026-11-24) eNet #22 — the SH9 ESP32-C6 hub-radio
bring-up contract, `docs/ehub-pro-c6-radio-bringup.md` — is **open, not
merged** (head `loop/2026-11-23-sh9-c6-radio` @ `09e808b1`). The
bring-up doc exists only on that PR branch, not on master. This scoping
therefore:

- asserts **nothing** from #22's contents as merged fact;
- scopes **around** it: Surface A's bearer is the Thread mesh the SH9
  contract will define (RCP split: ESP32-C6 radio, i.MX 8M Mini host,
  Spinel link — as proposed on the PR branch), and the spec §8 gap-map
  radio row moves OPEN → PARTIAL only when #22 merges;
- stays valid either way: if #22's radio split changes at review, only
  §2's "what it requires" row changes, not the surface distinction.

The bundle's pipeline position is unchanged by this scoping: on #22's
merge, smart-home-iot moves gap-mapped → building.

## 6. What eVera has today (dependency read, now verified)

H6's backlog dependency was a read of `eVera/mobile/src/services/voice.ts`.
Read live on eVera master (blob `afb8f3cf`, 3,553 bytes): it is a
**phone-local** speech service — speech-to-text via
`@react-native-voice/voice` and text-to-speech via `react-native-tts`,
with a native microphone permission request. It is the on-device voice
surface for the eVera mobile app. It contains **no** hub path, **no**
Matter path, and **no** Alexa/Skill path. Consequence for this split:
the eVera voice surface and the eHub-Pro voice surfaces are separate
pieces that will need a defined meeting point (which surface answers a
spoken command, on which device) — that meeting point is a later slice,
not this scoping.

## 7. Seam + journey stage

Seam 6 — voice that only works via one vendor's cloud is a shipping
dependency. Stage 6 (integrating/shipping): a hub that ships with only
the Skill surface ships with a cloud dependency its buyers did not ask
for; a hub that ships the Matter-local surface first keeps voice working
when the internet does not. Scoping the split now is what lets the
bundle build the local surface first and treat the Skill as a separate,
later, explicitly-decided commitment.

## 8. Explicitly NOT decided

- **No Alexa endpoint is proposed or approved.** Building a Skill
  endpoint, hosting it, and certifying it is a product/operations
  decision for Aswin, not a consequence of this scoping.
- **No voice-assistant / NLU choice.** No wake-word engine, no
  on-device NLU, no cloud NLU provider is selected. HA two-way audio
  is a talk-back reference only (§4), not an NLU selection.
- **No hub microphone hardware.** Whether the hub itself listens
  (mic array on the hub) or voice arrives via phones, speakers, and
  cameras is undecided; §6's eVera phone-local service is the only
  listening surface that exists today.
- **No Matter voice-command story.** None is claimed; if a future
  Matter release defines one, that is a new scoping input, not an
  assumption here.
- **No MCP voice-tool contract.** Voice tools under eSec Controls 7–9
  are named as the layer where they belong (§3), but no tool shapes,
  capability names, or permission model are specified here.
- **No HA dependency.** Home Assistant is a reference implementation
  for talk-back, not a bundle dependency; the hub does not require HA.

## 9. Next slices (not commitments)

1. On #22's merge: re-confirm §2's "what it requires" row against the
   merged radio contract (one-line check, no new doc).
2. Matter-local talk-back prototype against the HA 2026.11 Labs
   preview shape (WebRTC audio to a Matter camera/doorbell) — only
   after the bundle has a hub and a camera to talk through.
3. Skill-endpoint decision input for Aswin (hosting shape, account
   linking, cost) — only when a device exists that Alexa cannot
   already reach locally (plan §3.1's own trigger).

— av_mando
