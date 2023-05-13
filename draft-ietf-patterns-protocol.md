---
title: "Patterns - Event Pattern Matching Language: Flexible event matching through rules"
abbrev: "Patterns"
category: info

docname: draft-ietf-patterns-protocol-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "baldawar/draft-ietf-rule-patterns"
  latest: "https://baldawar.github.io/draft-ietf-rule-patterns/draft-ietf-patterns-protocol.html"

author:
 -
    fullname: Rishi Baldawa
    organization: TBC
    email: "baldawar@amazon.com"

normative:

informative:


--- abstract

This document describes patterns as supported within a matching libraries with the goal standardizing querying fields within data objects through an easy to express query language across many different libraries.

# About This Document

This is a work in progress. It is not ready for consumption or reference. Prior art can be found at https://github.com/timbray/quamina/blob/main/PATTERNS.md 

--- middle

Introduction        {#intro}
============

The patterns provides a way to query JSON documents as described in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259.html). The patterns are defined using a simple JSON syntax to make it easy to read, write, and manipulate such that they can be prefered over ad-hoc parsing and manipulation of code for further operations such as filtering, selection, and transfomring of data based on specific patterns. Additionally, This document describes the syntax and semantics of the patterns, along with examples of how they can be used in practice. Finally, the document provides guidelines for creating new patterns that conform to the existing syntax and semantics. Overall, this RFC aims to guide the functionality of the JSON-based library by providing a powerful pattern matching capability that enables users to work with JSON data in a more flexible and intuitive way.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge [Quamina](https://github.com/timbray/quamina) and [Ruler](https://github.com/aws/event-ruler)
