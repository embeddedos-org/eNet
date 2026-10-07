# Sub-GHz three-way plan

Date: 2026-10-07. Sibling to `docs/lorawan-study.md` and
`docs/matter-thread-note.md`.

## The three lanes

Sub-GHz IoT is splitting into three standards lanes, and eNet needs a
position on all three:

1. **Wi-Fi HaLow (802.11ah)** — long-range Wi-Fi for IP-native sensor
   networks; simplest application-layer story.
2. **Thread Sub-GHz** — Thread's 802.15.4 radio moved down from 2.4 GHz;
   Mesh over IP for smart-home/industrial.
3. **Zigbee "Suzi"** — the CSA's brand for 802.15.4 sub-GHz (EU 868 /
   NA 915 MHz). Certification opened in 2026; already shipping in products
   (pierregode/ragnar, Sept 26).

## The design question: coexistence

HaLow and Suzi share the same 868/915 MHz bands. Running both stacks on one
device — or in one deployment — is not a free lunch. Required before eNet
commits to a dual-stack story:

- **Duty-cycle accounting** per band/region (EU 868 has strict duty-cycle
  caps; NA 915 has dwell-time rules).
- **Channel planning** so HaLow's wider channels don't deafen Suzi/Thread
  reception.
- **Interference testing in EoSim** — simulate the mixed-band environment
  before it is built (pairs with the M5Stack SX1262 radio platform def).

## Watch: Apple Oct 13

Apple's Oct-13 home-hub event is expected to cover smart-home hardware and
LG-built Thread/Matter accessories. If Apple ships Thread as the default
home radio, the **Thread Border Router story becomes table stakes** for any
eNet smart-home positioning. Revisit this doc after the event.
