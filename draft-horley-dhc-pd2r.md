---
title: "Reserved Interface Identifier Sub-Range Delegation for IPv6 Endpoints"
abbrev: "DHCPv6-PD Sub-Range Delegation"
category: info

docname: draft-horley-dhc-pd2r-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Dynamic Host Configuration"
keyword:

- Internet Draft
venue:
  group: "Dynamic Host Configuration"
  type: "Working Group"
  mail: "dhcwg@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dhcwg/"
  github: "hexabuild/draft-horley-dhc-pd2r"
  latest: "https://hexabuild.github.io/draft-horley-dhc-pd2r/draft-horley-dhc-pd2r.html"

author:

-
    ins: E. Horley
    fullname: Ed Horley
    organization: HexaBuild
    email: ed@hexabuild.io

normative:
RFC8174:
RFC4862:
RFC8415:
RFC9663:
RFC6877:

informative:

--- abstract
This document specifies a mechanism allowing DHCPv6 servers to allocate small Interface Identif(IID) sub-ranges (e.g., /96 or /120 blocks) from within the lower 64 bits of an IPv6 on-link /64. These sub-ranges provide hosts with deterministic, host-specific address pools while preserving normal Neighbor Discovery (ND) and Router Advertisement (RA) behavior. Clients continue to treat the prefix as a /64 for all purposes, including ND, and may freely assign addresses from the reserved sub-range to any local interface (including a CLAT internal interface). This specification maintains architectural requirements for IPv6 subnetting, avoids introducing new routing semantics, and enables hosts to perform richer internal addressing without disrupting the network.
--- middle

# Introduction

IPv6 addressing architecture defines a /64 prefix boundary between the network prefix and Interface Identifier (IID). Hosts typically obtain IPv6 addresses through SLAAC, DHCPv6 IA_NA, DHCPv6 IA_PD, or combinations thereof. However, emerging endpoint architectures—multi-interface nodes, CLAT translators (RFC 6877), virtual machine hosting, and container networks—require access to *multiple deterministic IPv6 addresses belonging to the same on-link /64.

Prefix Delegation (PD) is not appropriate in these situations because:

- PD produces a routed prefix, not intended for assignment on the link.
- PD requires router participation, not available on access links like Wi-Fi or residential broadband.
- Hosts need the additional addresses locally, not as routed subnets.

This document introduces **IID Sub-Range Reservation**, allowing a DHCPv6 server to reserve a block of IIDs in the lower 64 bits of an on-link /64 and deliver it to the client—*without altering router advertisements or ND behavior. Hosts continue to perceive the network prefix as a normal /64. The reserved block is not a delegated prefix, not routed, and not advertised externally.

This mechanism extends conceptual models from RFC 9663 by specifying concrete DHCPv6 behavior, algorithms, interoperability rules, and examples.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

Key words (MUST, MUST NOT, etc.) are per RFC 8174.

**Parent Prefix:** The /64 prefix advertised via RA.

**IID Sub-Range:** A reserved block of the lower 64 bits, commonly /96 or /120.

**Reserved IID Range:** The usable IPv6 addresses created by concatenating ParentPrefix[0:64] with the IID sub-range.

**CLAT Host:** A Customer-side translator using IPv6 addresses for NAT64/CLAT internal mappings.

# Requirements and Constraints

This specification MUST meet the following:

1. **Do not alter SLAAC.**
SLAAC MUST operate normally. Host MUST consider the on-link prefix to be /64.

2. **No changes to Router Advertisements.**
RAs MUST NOT be extended with additional options related to reserved sub-ranges.

3. **Server-managed uniqueness.**
DHCPv6 server MUST ensure IID ranges do not overlap between clients.

4. **Clients are not required to advertise these prefixes.**
The reserved range is local-only and MUST NOT be treated as a routed prefix.

5. **Host-local multi-interface use allowed.**
Clients MAY assign sub-range addresses to any local interface.

6. **If DHCPv6 and SLAAC are run on the same on-link via the RA, then simple duplicate address detection will be used for the rare cases where a SLAAC address is dynamically generated in one of the IID Sub-Ranges that is in use. The host that is allocated the IID Sub-Range MUST do the DAD response on behalf of the entire range.

# Mechanism Overview

The mechanism operates in the following phases:

```text
+---------+       DHCPv6 IA_NA + OPTION_IID_SUBRANGE        +---------+
| Client  | <------------------------------------------------> | Server |
+---------+                                                     +---------+

1. Client sends SOLICIT.
2. Server allocates and returns IID sub-range via OPTION_IID_SUBRANGE.
3. Client configures any number of addresses from that range.
4. Client still processes RA and SLAAC normally.
```

The DHCPv6 server never advertises a prefix and never signals a non-/64 boundary.

# Protocol Overview

A new DHCPv6 option, **OPTION_IID_SUBRANGE**, provides:

- ParentPrefix (always a /64)
- StartIID
- EndIID
- Lifetime

The client treats addresses formed from this range as *additional IPv6 addresses*. These addresses:

- testing
- belong to the on-link /64
- are not SLAAC addresses and do not follow IID rules (stable/private)
- MUST pass DAD
- MAY be used by any local interface

# DHCPv6 OPTION_IID_SUBRANGE

## Format

```text
0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      OPTION_IID_SUBRANGE      |         option-len           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                       ParentPrefix (128 bits)                 |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         RangeStartIID (64 bits)               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          RangeEndIID (64 bits)                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Flags (16 bits)          |       Lifetime (32 bits)  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Field Requirements

- ParentPrefix MUST be a /64 (low 64 bits = 0).
- RangeStartIID and RangeEndIID MUST satisfy `Start ≤ End`.
- Lifetime MUST be treated similar to IA_NA lifetimes.

# Server Operation

This section defines **normative algorithms**.

## Server Allocation Algorithm

### Allocation Preconditions

Server MUST know:

- The RA-advertised /64 for the link.
- The administrative IID allocation pool (e.g., 0x0000:0000:0000:0000–0x0000:0000:00FF:FFFF).

### Algorithm (Normative)

Python example:

```python
function allocate_iid_subrange(client_duid):
    pool = get_available_iid_ranges()
    if pool is empty:
        return ERROR_NoAvailableRange

    range = select_smallest_available_block(pool)  # SHOULD pick /120
    mark_range_as_reserved(range, client_duid)
    return range
```

### Rebinding

If a client repeats SOLICIT with the same DUID:

```python
if existing_assignment_for(client_duid):
    return existing_assignment
else:
    return allocate_iid_subrange(client_duid)
```

### Collision Prevention

Server MUST guarantee:

`ReservedRange(clientA) ∩ ReservedRange(clientB) = ∅`.

## Example Server Policy

A server MAY divide the lower IID /80–/120 into blocks:

```text
2001:db8:1000:200::0000/120  → Client 1
2001:db8:1000:200::0100/120  → Client 2
2001:db8:1000:200::0200/120  → Client 3
"..."
```

# Client Operation

Client MUST follow these rules:

1. Continue processing RAs and SLAAC as usual.
2. Treat ParentPrefix as /64.
3. Validate ParentPrefix against local RAs.
4. Create addresses:
example:

```text
IPv6Address = ParentPrefix[0:64] || IID_value
```

5. Perform DAD normally.
6. MAY assign any sub-range address to:
- primary interface
- CLAT internal interface
- loopback
- virtual NICs
- containers

# Examples

## Example DHCPv6 Exchange

```text
Client → SOLICIT

Server → ADVERTISE
  IA_NA: 2001:db8:1000:200::a8f1
  OPTION_IID_SUBRANGE:
    ParentPrefix: 2001:db8:1000:200::/64
    StartIID:     0x0000000000000100
    EndIID:       0x00000000000001FF
    Lifetime:     7200

Client → REQUEST

Server → REPLY (same contents)
```

Client may now assign:

```text
2001:db8:1000:200::100
2001:db8:1000:200::101
"..."
2001:db8:1000:200::1FF
```

# CLAT Use Cases

## CLAT IID Reservation

A CLAT host MUST use a deterministic IPv6 address for its NAT64-binding IPv6 endpoint.

Today, implementations typically:

- derive the CLAT IPv6 address from EUI-64 or stable IID
- rely on RFC 7217 private addresses
- limit the host to one CLAT address

With IID sub-ranges, the host MAY allocate:

- one address for CLAT internal translation
- other addresses for per-flow, per-container, or per-service mapping

### Example CLAT Assignment

```text
Reserved range: 2001:db8:1000:200::200–::2FF

CLAT internal interface:
  2001:db8:1000:200::200

Container A (CLAT-enhanced):
  2001:db8:1000:200::210

Container B:
  2001:db8:1000:200::220

Host loopback:
  2001:db8:1000:200::2FF
```

CLAT behavior is unchanged; the host simply has more stable options.

# State Machines

## Server State Machine

```text
          +----------------+
          |   INIT         |
          +--------+-------+
                   |
                   v
          +--------+-------+
          | WAIT_SOLICIT   |
          +--------+-------+
                   |
     SOLICIT       v
          +--------+-------+
          | ALLOCATE_RANGE |
          +--------+-------+
                   |
                   v
          +--------+-------+
          | SEND_ADVERTISE |
          +--------+-------+
                   |
  REQUEST          v
          +--------+-------+
          | SEND_REPLY     |
          +----------------+
```

### Client State Machine

```text
 +-------------+
 | INIT        |
 +------+------+
        |
        v
 +------+------+     RA arrives
 | LISTEN_RA   |------------------------------+
 +------+------+                              |
        |                                      |
        v                                      |
 +------+------+                                |
 | SOLICIT     |                                |
 +------+------+                                |
        | DHCPv6 ADVERTISE                      |
        v                                      |
 +------+-------+                               |
 | PROCESS_OPT  |  ← Validate ParentPrefix ←----+
 +------+-------+
        |
        v
 +------+-------+
 | CONFIG_IIDS  |
 +------+-------+
        |
        v
 +------+-------+
 | OPERATE       |
 +--------------+
```

# Validation Logic

Client MUST:

1. Confirm ParentPrefix matches a prefix advertised via RA.
2. Reject options referencing unknown or non-/64 prefixes.
3. Ensure `RangeEndIID ≥ RangeStartIID`.
4. Ensure generated addresses succeed in DAD.
5. Ignore invalid or overlapping sub-ranges.

Server MUST:

1. Guarantee exclusivity of IID ranges.
2. Validate ParentPrefix is an on-link /64.
3. Reject attempts to allocate ranges outside administrative pools.

# Security Considerations

- Predictable IID ranges may reveal host identity patterns; operators SHOULD allow randomized distribution.
- DHCPv6 authentication SHOULD be used when available.
- No new attacks on SLAAC or RAs are introduced.

# IANA Considerations

IANA is requested to assign a DHCPv6 option code for:

```text
OPTION_IID_SUBRANGE
```
--- back

# Acknowledgments

{:numbered="false"}

The author(s) would like to acknowledge the valuable input and contributions from Tim Winters, Nick Buraglio, and Tommy Jensen.
