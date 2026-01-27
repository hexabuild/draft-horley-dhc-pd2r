---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Reserved Interface Identifier Sub-Range Delegation for IPv6 Endpoints"
abbrev: "DHCPv6-PD Sub-Range Delegation"
category: info

docname: draft-horley-dhc-pd2r-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "int"
workgroup: "Dynamic Host Configuration"
keyword:
 - Internet Draft
venue:
  group: "Dynamic Host Configuration"
  type: "Working Group"
  mail: "dhcwg@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dhcwg/"
  github: "hexabuild/draft-horley-dhc-pd2r"
  latest: "https://github.com/hexabuild/draft-horley-dhc-pd2r"

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

...

--- abstract

This document specifies a mechanism allowing DHCPv6 servers to allocate small Interface Identifier (IID) sub-ranges (e.g., /96 or /120 blocks) from within the lower 64 bits of an IPv6 on-link /64. These sub-ranges provide hosts with deterministic, host-specific address pools while preserving normal Neighbor Discovery (ND) and Router Advertisement (RA) behavior. Clients continue to treat the prefix as a /64 for all purposes, including ND, and may freely assign addresses from the reserved sub-range to any local interface (including a CLAT internal interface). This specification maintains architectural requirements for IPv6 subnetting, avoids introducing new routing semantics, and enables hosts to perform richer internal addressing without disrupting the network.

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

**Reserved IID Range:** The usable IPv6 addresses created by concatenating ParentPrefix\[0:64] with the IID sub-range.

**CLAT Host:** A Customer-side translator using IPv6 addresses for NAT64/CLAT internal mappings.

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
