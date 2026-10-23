# MCP bridge trial plan — mcp2mqtt first, mcp2tcp bench-only

**Status:** trial plan (2026-10-16). This plan makes the trial
recommended by [`mcp-bridge-evaluation.md`](mcp-bridge-evaluation.md)
concrete: scope, safety shape, success criteria, and the exit
decision each trial informs. It adopts nothing — a bridge is
adopted only when its trial report says so, in writing, in this
directory.

## Scope

Two trials, in the evaluation's order:

1. **mcp2mqtt (first).** One simulated device behind mcp2mqtt,
   running against EoSim, publishing and consuming through one
   bench MQTT broker. Topic set: read-only telemetry topics plus
   exactly one non-safety write topic (the evaluation's trial
   scope). The broker requirements in
   [`mqtt-broker-hardening.md`](mqtt-broker-hardening.md) apply to
   the bench broker unchanged — the trial does not get a weaker
   broker than production would.
2. **mcp2tcp (last, bench-only).** One bench instrument behind
   mcp2tcp, against a stub endpoint with a documented framing
   protocol — no production endpoint, no discovery. Raw TCP has
   no topic or session semantics, so the trial exists to measure
   how much safeguard machinery the bridge itself must carry.

**Out of scope for this plan:** modbus-mcp. The evaluation
sequences it second and read-first (register/coil reads only,
write tools disabled until interlocks and failure modes for
physical actuation are covered). It gets its own trial plan when
the mcp2mqtt report is in.

## Preconditions

The evaluation's provenance checklist, answered in writing for
the candidate before the trial starts: pinned source commit,
reproducible build, endpoint ownership by allowlist, credential
isolation, read/write tool separation. A candidate that cannot
answer all five is not trialled.

## Safety shape

- **Allowlisted surface only.** The mcp2mqtt trial's topic
  allowlist and the mcp2tcp trial's endpoint allowlist are the
  whole reachable world of each bridge. Anything outside the
  allowlist is refused by configuration, not by agent discipline.
- **No unauthenticated bridge.** Both bridges run authenticated
  end to end. The lesson of the gateway auth-bypass class
  (CWE-290) applies to bridges exactly as to gateways: a bridge
  that lets an unauthenticated caller reach device commands is
  an open channel to physical equipment.
- **No unauthenticated fallback.** Every failure path — rejected
  or expired credential, broker reconnect, bridge restart,
  configuration reload — fails closed: no anonymous session, no
  default credential, no default-permissive topic/endpoint set.
  A bridge that cannot authenticate does not bridge.
- **The agent never holds credentials.** The bridge brokers
  credentials at call time from its own protected store (the
  evaluation's broker rule); the agent side of the conversation
  contains no credential material, so there is nothing for a
  prompt injection riding the agent to leak.
- **Bounded commands.** Each trial defines its command set in
  advance; a command outside the set is a trial failure, not a
  feature request.
- **Teardown.** Ending a trial leaves no open listeners, no
  running bridge processes, and no credentials on the bench
  broker beyond its own store.

## Success criteria

Per bridge, all of:

1. The bridge's tools are listable behind the org's MCP gateway
   alongside the org's own servers.
2. A bounded command from the trial's command set executes
   against the simulated device (mcp2mqtt) or the stub endpoint
   (mcp2tcp), and its result returns through the fabric to the
   caller.
3. An out-of-allowlist request is refused by the bridge's
   configuration, with the refusal visible in the bridge's log.
4. Every unauthenticated shape is refused, and the trial report
   records each one individually: (i) no credential; (ii) a wrong
   credential; (iii) a fabricated `Authorization` header on an
   otherwise unauthenticated connection; (iv) a reconnect after a
   broker restart without re-authentication; (v) an
   out-of-allowlist request made while unauthenticated. Each
   refusal must be visible in the bridge's log (the visibility
   standard of criterion 3).
5. Teardown leaves the bench in the state the safety shape
   requires.

## Exit decision

Each trial ends with a short report in this directory recording,
per bridge: **adopt**, **defer**, or **reject** — with the reason
tied to the success criteria above and to the provenance
checklist. An observed unauthenticated fallback on any path is an
automatic **reject** for that bridge (not defer), because the
failure is in the bridge's trust shape, not its feature set. The decision is recorded in this plan's status line
when the report lands. Silence is not adoption: a bridge with no
report stays untrialled for planning purposes, however well the
demo went.

## Cross-references

- [`mcp-bridge-evaluation.md`](mcp-bridge-evaluation.md) — the evaluation and trial order this plan executes.
- [`device-bridges.md`](device-bridges.md) — the hardware-bridge lane both trials belong to.
- [`mqtt-broker-hardening.md`](mqtt-broker-hardening.md) — the bench broker's requirements for the mcp2mqtt trial.

## Feasibility probe — 2026-10-22: trial not runnable in the daily-loop sandbox

A time-boxed probe (2026-10-22) asked whether the mcp2mqtt
trial above could execute in the environment the daily loop
runs in. **Answer: no — for that sandbox, not for the bridge.**
What the environment lacks, recorded so a trial day is not
re-probed from scratch:

- **No container runtime:** Docker is unavailable, so neither
  the broker nor the bridge can be run as the pinned,
  reproducible containers the provenance checklist requires.
- **No broker binary:** no `mosquitto` (or any MQTT broker) is
  installed, so there is no bench broker to harden to
  `mqtt-broker-hardening.md` — and the trial does not get a
  weaker broker than production would.
- **No MQTT client library** is installed for the harness
  side, and installing unpinned tooling to force a "yes" was
  ruled out of bounds — a trial run on unpinned components
  would not evidence the pinned candidate this plan trials.

**What a trial day needs (provisioning ask):** an environment
with (1) a container runtime or a natively installed broker
binary, (2) the bench broker provisioned at the hardening
standard in `mqtt-broker-hardening.md`, (3) an MQTT client
library for the harness, and (4) the mcp2mqtt candidate itself
at a pinned source commit per the provenance checklist. With
those four in place the trial above runs as written,
including criterion 4's five-case negative set. Until then
the mcp2mqtt trial is **deferred by environment, not by
evidence about the bridge** — no adopt/defer/reject decision
about mcp2mqtt itself is taken or implied by this entry.
