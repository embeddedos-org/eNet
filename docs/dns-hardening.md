# DNS resolver hardening — malformed-query requirements and test vectors

**Status:** requirements note for the DNS resolver in `eos`
(`eos/net`). eNet is a docs-only repo — the resolver does not live
here; this note is the contract its implementers and reviewers work
against. **Pattern source:** CVE-2015-5477 (ISC BIND, TKEY query
DoS, added to CISA KEV 2026-10-08): a single crafted query of an
unusual type reached an `assert` and killed the server. The resolver
equivalent for EoS is: **no wire input may assert, abort, panic, or
hang the resolver — malformed queries are dropped, never fatal.**

## Requirements

1. **Unknown or unusual QTYPEs are data, not errors of state.**
   A query for a type the resolver does not implement (including
   TKEY, type 249) is answered with an appropriate rcode (NotImp /
   FormErr) or dropped per local policy — it must never reach a
   code path that assumes the type is known.
2. **No assertion on wire-controlled values.** Header counts,
   label lengths, name compression pointers, and section sizes all
   come from the packet. Bounds are checked with ordinary error
   returns; `assert`/panic on any of them is a defect of the
   CVE-2015-5477 class.
3. **Compression pointers must terminate.** Pointer loops and
   pointers outside the packet are rejected; total name-expansion
   work per packet is bounded.
4. **Truncation is routine.** A packet shorter than its header
   claims, or cut mid-question/mid-record, is dropped whole — never
   parsed partially into resolver state.
5. **Oversized label chains are bounded.** Names are expanded
   against the 255-octet DNS limit; anything beyond is malformed
   and dropped, without allocating attacker-sized buffers.

## Test vectors (adopt verbatim as resolver tests in `eos`)

| # | Input | Expected behaviour |
|---|---|---|
| 1 | Well-formed A query for a known name | Normal resolution (control) |
| 2 | Query with QTYPE = 249 (TKEY), otherwise well-formed | Rcode NotImp/FormErr or policy drop; resolver alive, next query (vector 1) still resolves |
| 3 | Query with QTYPE = 0 and QTYPE = 65535 | Same as vector 2 — no type is special-cased into a crash |
| 4 | Header claims QDCOUNT = 1, packet ends inside the question | Dropped whole; no state change; resolver alive |
| 5 | Question name is a compression pointer to itself | Rejected; expansion terminates; resolver alive |
| 6 | Compression pointer beyond the packet end | Rejected; resolver alive |
| 7 | Label chain expanding past 255 octets | Rejected as malformed; no oversized allocation |
| 8 | Response with ANCOUNT larger than the packet carries | Dropped; cache unchanged (no partial poisoning) |

"Resolver alive" in every row means: the process/task continues and
a following control query resolves correctly — the regression these
vectors exist to pin is a resolver that dies or wedges on the first
crafted packet.

## Relationship to the KEV gate

This note is the eNet-side disposition for the Oct 8 KEV batch's
BIND entry: the vulnerable component (BIND) is not in the org, but
the *pattern* — one crafted query type reaching an assertion — is
exactly what a small resolver must be tested against. The stack's
KEV doc records the component disposition; this note records the
pattern work.
