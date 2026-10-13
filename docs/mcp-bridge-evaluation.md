# MCP bridge evaluation — mcp2mqtt, mcp2tcp, modbus-mcp

**Status:** evaluation (2026-10-13). This page evaluates three
MCP bridges against eNet's protocol surface and recommends a trial
order. The bridge lane itself — what each bridge is and where it
plugs in — is specified in [`device-bridges.md`](device-bridges.md);
this page does not restate it, it judges it.

## What is being evaluated

An MCP bridge puts a device protocol behind the Model Context
Protocol so an agent can reach real devices through the same fabric
it uses for everything else. The three candidates map onto eNet's
surface as follows:

| Bridge | Protocol | eNet surface | Fit for the device story |
|---|---|---|---|
| mcp2mqtt | MQTT publish/subscribe to a broker | eNet's MQTT device story: fleet telemetry, command topics | Highest. MQTT is the org's primary device-messaging surface; a bridge here reaches the most devices with the least new machinery, and the broker is a natural policy choke point. |
| mcp2tcp | Raw TCP streams to configured endpoints | eNet's TCP transport: legacy devices, bench instruments, stream protocols | Medium. Useful and simple, but raw TCP has no topic or session semantics — every safeguard (framing, auth, timeouts) is the bridge's own problem. |
| modbus-mcp | Modbus register/coil access | eNet's industrial angle (the TSN/EtherCAT lane's legacy-fieldbus complement) | High value, highest care. Modbus has no authentication by design and writes move physical equipment; the bridge is a safety boundary, not just a protocol adapter. |

## Provenance checklist (applies to every bridge)

No bridge is trialled, let alone adopted, until each row is answered
in writing — the eVera policy rules applied at selection time:

| Check | What "pass" means |
|---|---|
| Pinned source | The bridge's source is pinned to a specific commit in our configuration; "latest" is never a version. |
| Reproducible build | The pinned commit builds reproducibly in CI from source; no prebuilt binary blobs from the internet at run time. |
| Endpoint ownership | Every endpoint the bridge can reach is configured by us and allowlisted; the bridge cannot discover or wander to new endpoints on its own. |
| Credential isolation | The bridge holds device/broker credentials; the agent never does (below). |
| Read/write separation | Read and write tools are distinct, so write access can be granted (or withheld) per deployment. |

## Credential isolation — the broker rule

A bridge must be able to **broker credentials**: it pulls the broker
password, device key, or token from its own protected store at call
time and presents it to the device side, while the agent's side of
the conversation contains no credential material at all. Consequences:

- An agent (or a prompt-injection attack riding one) cannot leak a
  token it never held.
- Credential rotation happens in one place — the bridge's store —
  not in every agent configuration.
- Audit logs can attribute device actions to the bridge's identity
  and the requesting session separately.

A bridge that requires the agent to supply credentials per call
fails this check regardless of its other merits.

## Recommendation

1. **Trial mcp2mqtt first.** MQTT is eNet's main device surface, the
   broker gives a single policy and credential choke point, and the
   read/write split maps cleanly onto topic ACLs. Trial scope: one
   bench broker, read-only topics plus one non-safety write topic,
   credentials held only by the bridge.
2. **modbus-mcp second, read-first.** Trial with read tools only
   (register/coil reads against a bench PLC or simulator); write
   tools stay disabled until the trial report covers interlocks and
   failure modes for physical actuation.
3. **mcp2tcp last, bench-only.** Its lack of protocol semantics
   makes it the weakest fit for a general device story; trial only
   against a named bench instrument with a documented framing
   protocol.

A trial needs: the provenance checklist answered for the candidate,
a pinned deployment under the gateway, an allowlist containing only
bench endpoints, and a short trial report in this directory before
any production endpoint is added.

## Cross-references

- [`device-bridges.md`](device-bridges.md) — the hardware-bridge lane this evaluates.
- [`tsn-ethercat-lane.md`](tsn-ethercat-lane.md) — the deterministic-comms lane modbus-mcp complements.
- eSec `docs/mcp-hostile-protocol-hardening.md` — the hostile-protocol posture every bridge inherits.
