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

TODO Introduction

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
