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
  github: "epml-spec/draft-ietf-patterns-protocol "
  latest: "https://epml-spec.github.io/draft-ietf-patterns-protocol/draft-ietf-patterns-protocol.html"

author:
 -
    fullname: Someone
    organization: SomeOrg
    email: "some@email.com"

normative:
  RFC8259:
  RFC4632:

informative:
  Quamina:
    title: Quamina
    target: https://github.com/timbray/quamina
  Ruler:
    title: Event Ruler
    target: https://github.com/aws/event-ruler



--- abstract

This document describes patterns as supported within a matching libraries with the goal standardizing querying fields within data objects through an easy to express query language across many different libraries; promoting interoperability, predictability, and expectations for future changes to improve developer experience.

**NOTE**: This is a work in progress. It is not ready for consumption or reference. Prior art can be found at [Quamina Patterns.md](https://github.com/timbray/quamina/blob/main/PATTERNS.md).

--- middle

Introduction        {#intro}
============

Event matching libraries are software components that enable developers to perform advanced querying and filtering operations on data structures in real-time. These libraries provide powerful mechanisms to then search, extract, and manipulate specific information within complex data objects. They are commonly used in a variety of domains, including data processing, event-driven systems, and data integration.

## Current State

At present {{Quamina}} and {{Ruler}} are two known examples that are optimized for fast processing. While both has similar origins, they were built independently and have gaps in features supported. Quamina exclusively supports shell-style matching while Event-Ruler does the same for CIDR and Numeric matching. Event Ruler also has [reserved keywords](https://github.com/aws/event-ruler/blob/main/src/main/software/amazon/event/ruler/Constants.java) for future matchers against regular-expressions, dates, and strings isn't the case for Quamina.

## Need for Standardization

While more work is necessary to identify additional libraries with similar intent, the current landsacpe is expected to be a fragmented across different libraries and langauges. With no guide for standardization, each library organically introduced its own syntax and semantics for specifying field queries, leading to inconsistencies and limited interoperability between libraries.

This poses several challenges. Developers often need to learn and adapt to different query languages when switching between libraries, increasing development effort and creating a barrier to entry. Furthermore, the lack of interoperability hampers collaboration and limits the reusability of code across different matching libraries such as common test and benchmarking framework.

To address these challenges, there is a clear need for a standardized query language for field querying in matching libraries.

## Scope

The document's primary goal is to define a common syntax and semantics for patterns used in matching libraries. By establishing a shared understanding of how fields are queried, developers can write portable and reusable code across different libraries. Additionally, this standardization aims to foster collaboration, simplify integration, and enhance the overall ecosystem of matching libraries.

This document focuses on patterns used for querying fields within data objects, with an emphasis on JSON as described in {{RFC8259}}. While other querying mechanisms (such as regular expressions and SQL) and formats (such as Avro and Ion) exists, this standardization effort specifically addresses patterns expressed in JSON syntax and leaves others for future revisions.

## Conventions and Definitions

{::boilerplate bcp14-tagged}

The terminology used in this document follows the definitions specified in {{RFC8259}} for JSON constructs. Additionally, specific terms introduced within the context of patterns and matching libraries will be defined as they arise.

Throughout the document, examples will be provided to illustrate pattern syntax and usage. These examples are not normative and serve only to aid comprehension.

## Document Structure

The remainder of this document is organized as follows: Section 2 provides background information on matching libraries and the need for standardization. Section 3 introduces the concept of patterns and their role in matching libraries. Sections 4 to 9 discuss various aspects of patterns, including extended patterns, standardization approach, implementation guidelines, and security considerations. Finally, the document concludes with acknowledgments, references, and any relevant appendices.

# Patterns

A pattern in the context of matching libraries is a query expression used to identify and match specific fields within data objects. A field represents a specific value within a data object, identified by its path from the root of the object. Patterns provides a concise and expressive way to specify the desired criteria for field matching. Patterns MAY be applied to various types of data structures, but this document primarily focuses on JSON-based patterns.

## Syntax and Semantics

Patterns are expressed as JSON objects, conforming to the syntax defined in {{RFC8259}}. The syntax allows for the composition of complex patterns by combining objects, arrays, strings, numbers, and boolean values. Leaf values in a pattern must be enclosed within arrays.

The semantics of patterns dictate how they are evaluated and matched against fields in data objects. A pattern must contain an exact path specification, represented as a list of strings, leading to a leaf value in the target object. The leaf value MAY be either a direct match or an extended pattern, which further refines the matching criteria.

## Field Matching and Path Specification

Field matching involves comparing the values of fields in data objects with the patterns defined. Each field consists of a leaf value and a path, which is a sequence of strings representing the hierarchy of field names leading to the leaf value from the root of the object.

Path specifications in patterns omit arrays, focusing solely on the field names. By matching the path and comparing the leaf value of a field, patterns determine whether a field in a data object satisfies the specified criteria.

Consider the following pattern:

~~~
{"alpha": {"beta": [ 1 ]}}
~~~

In this pattern, "alpha" is the field name, "beta" is the subfield name that defines the path as "alpha.beta", and [1] is the leaf value. This pattern would match an event object with the field structure:

~~~
{"alpha": {"beta": 1}}
~~~

It would also match a pattern where event object was wrapped within an array structure:

~~~
{"alpha":  [ {"beta": 1} , {"beta": 2}, {"gamma": false} ]}
~~~

Patterns MAY also directly specify the path that needs to be travelled for simplicity as shown in the following pattern:

~~~
{"alpha.beta": [ 1 ]}
~~~

# Extended Patterns

This section describes several types of extended patterns commonly supported by matching libraries, including prefix, suffix, exists, anything-but, CIDR, and wildcard patterns. Extended patterns expand the capabilities of basic patterns by introducing additional types of matching criteria. These criteria allow for more flexible and expressive field querying within data objects. Extended patterns are represented as JSON objects with a designated field called the "Pattern Type" that specifies the type of extended pattern being used. Each pattern type has its own syntax and semantics for specifying matching conditions.

Libraries MAY implement additional extendend patterns such as "shell-style" that are outside of the current spe but could be included in future revisions.

## Prefix Pattern

The prefix pattern allows developers to match fields based on a prefix string. It is useful when partial matching or searching for fields with a common prefix is desired. The pattern type for the prefix pattern is "prefix" and its value is a string representing the desired prefix.

Example:

Consider the following event:

~~~
{"name": "John Doe"}
~~~

A prefix pattern to match the "name" field with a prefix of "John" would be:

~~~
{"name": {"prefix": "John"}}
~~~

This pattern would successfully match the event because the field value "John Doe" has a prefix of "John."

## Suffix Pattern

The suffix pattern matches fields whose values have a specific suffix. The "Pattern Type" for a suffix pattern is set to "suffix", and its value is a string representing the desired suffix.

Example:
Consider the following event:

~~~
{"email": "johndoe@example.com"}
~~~

To match fields where the value ends with "@example.com", the suffix pattern would be:

~~~
{"email": {"suffix": "@example.com"}}
~~~

This pattern would match the example event, as the field "email" has a value that ends with the suffix "@example.com".

## Wildcard Pattern

The wildcard pattern matches fields using wildcard characters for partial matches. The "Pattern Type" for a wildcard pattern is set to "wildcard", and its value is a string containing wildcard characters. Wildcard characters MAY stand in for any number of characters inluding zero characters.

Example:
Consider the following event:

~~~
{"file": "document.txt"}
~~~

To match fields where the value matches the wildcard pattern "docent", the wildcard pattern would be:

~~~
{"file": {"wildcard": "doc*ent*"}}
~~~

This pattern would match the example event, as the field "file" matches the wildcard pattern.

To match the asterisk character specifically, a wildcard character SHOULD be escaped with a backslash. Two consecutive backslashes, that is a backslash escaped with a backslash, represents the actual backslash character. A backslash escaping any character other than asterisk or backslash is not allowed.

## Exists Pattern

The exists pattern matches fields based on their existence or non-existence. The "Pattern Type" for an exists pattern is set to "exists", and its value is a boolean indicating whether the field should exist or not.

Example:
Consider the following event:

~~~
{"name": "John Doe"}
~~~

To match fields where the field "name" exists, the exists pattern would be:

~~~
{"name": {"exists": true}}
~~~

This pattern would match the example event, as the field "name" exists. Conversely, to match fields where the field "age" does not exist, the exists pattern would be:

~~~
{"age": {"exists": false}}
~~~

This pattern would also match the example event, as the field "age" does not exist.

## CIDR Matching

The CIDR matching allows for matching IPv4 and IPv6 address ranges. The "Pattern Type" for an exists pattern is set to "cidr", and its value is a CIDR notation as described within {{RFC4632}}.

Example:
Consider the following event:

~~~
{"ip": "192.0.2.1"}
~~~

To match fields where the value is within the CIDR range "192.0.2.0/24", the CIDR pattern would be:

~~~
{"ip": {"cidr": "192.0.2.0/24"}}
~~~

This pattern would match the example event, as the field "ip" falls within the specified CIDR range.

## Numeric Matching

Numeric matching patterns allow for specifying conditions based on JSON number values. Numeric matching is works for values between -5.0e9 and +5.0e9 inclusive, with 15 digits of precision or six digits to the right of the decimal point. Numeric values outside this range MAY not be supported for matching.

The following example demonstrates numeric matching for an event pattern that matches events based on numeric conditions for multiple fields:

~~~
{
  "detail": {
    "c-count": [ { "numeric": [ ">", 0, "<=", 5 ] } ],
    "d-count": [ { "numeric": [ "<", 10 ] } ],
    "x-limit": [ { "numeric": [ "=", 3.018e2 ] } ]
  }
}
~~~

In this pattern, the following conditions are specified for the respective fields:

* The "detail.c-count" field should have a numeric value greater than 0 and less than or equal to 5.
* The "detail.d-count" field should have a numeric value less than 10.
* The "detail.x-limit" field should have a numeric value equal to 3.018e2.

It will match following event

~~~
{
  "detail": {
    "c-count": 3,
    "d-count": 5,
    "x-limit": 301.8
  }
}
~~~

# Complex Matching Patterns

This section describes examples of complex matching patterns and their usage. Complex matching patterns allow for creation more sophisticated event patterns. They MAY allow for multiple rules to be combined for logical operations. They also support nesting for matching against a complex but singular criteria for a part of an event. These patterns provide increased flexibility and granularity in specifying the conditions under which an event should be matched.

## And Matching

When multiple multiple patterns are specified together, this effectively acts as logical and operator. This mean the aggregate complex pattern will only be a match if all rules within the pattern match.

Consider the following example rule:

~~~
{
  "time": [ { "prefix": "2017-10-02" } ],
  "detail": {
    "state": [ { "anything-but": "initializing" } ],
    "c-count": [ { "numeric": [ ">", 0, "<=", 5 ] } ],
    "d-count": [ { "numeric": [ "<", 10 ] } ],
    "x-limit": [ { "anything-but": [ 100, 200, 300 ] } ]
  }
}
~~~

This pattern specifies the following matching rules for different fields:

* The "time" field should have a prefix of "2017-10-02".
* The "detail.state" field should not match the value "initializing".
* The "detail.c-count" field should have a numeric value greater than 0 and less than or equal to 5.
* The "detail.d-count" field should have a numeric value less than 10.
* The "detail.x-limit" field should not match the values 100, 200, or 300.

It will match following event as it matches all rules:

~~~
{
  "time": "2017-10-02T00:00:00Z" ,
  "detail": {
    "state": "terminating",
    "c-count": 5,
    "d-count": 5,
    "x-limit": 400
  }
}
~~~

And fail to match following event as it failed to match the last rule :

~~~
{
  "time": "2017-10-02T00:00:00Z" ,
  "detail": {
    "state": "terminating",
    "c-count": 5,
    "d-count": 5,
    "x-limit": 200
  }
}
~~~

Note, as is the case specified {{RFC8259}}, the behavior of pattern matching is unpredictable when the fields within a rule are not unique. {{Ruler}} implementations only respect the last name/value pair only but {{Quamina}} reports an error.

## Or Matching

Complex event patterns MAY also be used to match any of the values across multiple fields using the "$or" construct. This allows for more flexible matching conditions based on various field values.

The following example demonstrates an event pattern that matches if any of the specified conditions are met:

~~~
{
  "detail": {
    "$or": [
      { "c-count": [ { "numeric": [ ">", 0, "<=", 5 ] } ] },
      { "d-count": [ { "numeric": [ "<", 10 ] } ] },
      { "x-limit": [ { "numeric": [ "=", 3.018e2 ] } ] }
    ]
  }
}
~~~

In this pattern, the "$or" construct is used to check if any of the following conditions are met:

* The "c-count" field has a numeric value greater than 0 and less than or equal to 5.
* The "d-count" field has a numeric value less than 10.
* The "x-limit" field has a numeric value equal to 3.018e2.

Complex matching patterns with the "$or" construct provide a powerful way to create flexible matching conditions across multiple fields.

Consider following example rule:

~~~
{
  "source": [ "aws.cloudwatch" ],
  "$or": [
    { "metricName": [ "CPUUtilization", "ReadLatency" ] },
    { "namespace": [ "AWS/EC2", "AWS/ES" ] }
  ]
}
~~~

It will match following event:

~~~
{
  "source": "aws.cloudwatch" ,
  "metricName": "CPUUtilization"
}
~~~

and  following event :

~~~
{
  "source": "aws.cloudwatch" ,
  "metricName": "CPUUtilization"
}
~~~

## Matching Anything But

The anything-but matching pattern allows for excluding specific values or patterns from matching. You MAY consider it logical NOT operator applied to single string or numeric values, lists of values, as a prefix, or suffix match.

The following examples illustrate different usages of the anything-but matching rule:

~~~
{
  "detail": {
    // anything-but (string)
    "state": [{ "anything-but": "initializing" }],
    // anything-but (number)
    "x-limit": [{ "anything-but": 123 }],
    // Anything-but list (strings)
    "status": [{ "anything-but": [ "stopped", "overloaded" ]}],
    // Anything-but list (numbers)
    "y-limit": [{ "anything-but": [ 100, 200, 300 ]}],
    // Anything-but prefix
    "x-reason": [{ "anything-but": { "prefix": "init" }}],
    // Anything-but suffix
    "y-reason": [{ "anything-but": { "suffix": "1234" }}],
    // Anything-but-ignore-case list (strings)
    "error-code": [{ "anything-but": {"equals-ignore-case": "RETRY" }}]
  }
}
~~~


## Nesting


Complex matching patterns MAY be combined to further allowing for fine-grained control over event filtering and processing. This is illustrated by following example that in effect implements the logic shown as the top-level comment within the example.

~~~
// Effect of ("source" && ("metricName" || ("metricType && "namespace" && ("metricId" || "spaceId")) || "scope") || ( "status" != "initializing" )))
{
  "source": [ "aws.cloudwatch" ],
  "$or": [
    { "metricName": [ "CPUUtilization", "ReadLatency" ] },
    {
      "metricType": [ "MetricType" ] ,
      "namespace": [ "AWS/EC2", "AWS/ES" ],
      "$or" : [
        { "metricId": [ 1234 ] },
        { "spaceId": [ 1000 ] }
      ]
    },
    { "scope": [ "Service" ] },
    {
        { "status": [ { "anything-but": "initializing" } ] }
    }
  ]
}
~~~

# Examples

To better understand the practical application of standardized patterns, the following examples demonstrate their usage in different scenarios, showcasing various matchers:

## Field Matching and Path Specification

Consider the following pattern:

~~~
{
  "field1": "value1",
  "field2": { "prefix": "abc" },
  "field3": { "suffix": "xyz" },
  "field4": { "exists": true },
  "field5": { "anything-but": "value2" },
  "field6": { "equals-ignore-case": "Value3" }
}
~~~

This pattern matches data objects where:

* "field1" exactly matches "value1".
* "field2" starts with the prefix "abc".
* "field3" ends with the suffix "xyz".
* "field4" exists in the data object.
* "field5" does not match the value "value2".
* "field6" matches the value "Value3" regardless of case sensitivity.

## Numeric Matching and CIDR Matching

The pattern below showcases numeric and CIDR matching:

~~~
{
  "field1": { "numeric": [ ">", 10, "<=", 20 ] },
  "field2": { "cidr": "192.168.0.0/16" }
}
~~~

This pattern matches data objects where:

* "field1" is a numeric value greater than 10 and less than or equal to 20.
* "field2" is an IP address that falls within the CIDR block "192.168.0.0/16".

## Complex Matching with Anything-But and $or

The pattern below demonstrates complex matching using the anything-but and $or matchers:

~~~
{
  "$or": [
    { "field1": "value1" },
    { "field2": { "anything-but": [ "value2", "value3" ] } },
    { "field3": { "exists": true } }
  ]
}
~~~

This pattern matches data objects where:

* Either "field1" exactly matches "value1" OR
* "field2" does not match the values "value2" or "value3" OR
* "field3" exists in the data object.

These examples showcase the versatility of standardized patterns and the ability to express complex matching requirements using a combination of matchers. Developers MAY leverage these patterns to effectively query and filter data objects based on various criteria.

# 7 Use Cases

1. **Filtering**: Organizations can use patterns to filter and process events based on specific criteria. For example, an event processing system can match events that contain a specific keyword in the event description or events generated by a particular source. Log management systems can employ patterns to extract relevant lines from log files.
1. **Stream Routing**: Patterns can be used to route relevant streams of information to domain specific subsystems. For example, an eventbus can be receive all events for an e-commerce platform and then route to domain specific events to microservices that serve payment, shipping, and ordering. Also, By defining patterns that match specific sensor readings or patterns of sensor events, organizations can derive insights and trigger actions based on IoT data.
1. **Data Validation**: Patterns facilitate data validation by allowing organizations to define rules for validating incoming data. For instance, an e-commerce platform can define patterns to ensure that user-provided information, such as dates being within valid ranges. IT systems can verify ip-address being valid range.
1. **Fraud and Anomaly Detection**: Organizations can leverage patterns to detect suspicious activities and potential fraud. By combining multiple matchers, such as numeric ranges, exclusion rules, and existence checks, systems can identify abnormal patterns of transactions or behavior. Patterns that match known attack signatures, anomalous network behavior, or malicious file patterns, organizations can enhance their security posture.
1. **Monitoring**: Patterns can be used to create sophisticated monitoring systems that track and analyze data streams in real-time. Multiple streams can be monitored for governance, compliance, auditing, resource monitoring.
1. **Geolocation-based Matching:** Applications that rely on geolocation data can utilize patterns to match data based on specific geographic regions. This can be useful for regional content delivery, or compliance with local regulations.

These use cases highlight the broad applicability of standardized patterns in diverse domains, providing a flexible and powerful toolset for data querying, filtering, and analysis.

# Security Considerations

Generally, there are security issues when parsing text. When designing and implementing systems that utilize pattern matching, validate and sanitize input patterns to prevent injection attacks or unintended consequences.

Patterns with complex or inefficient matching rules can potentially lead to resource exhaustion or performance degradation, and implementers should consider limiting the complexity of patterns (suc as length/depth of inputs) or enforcing timeouts to mitigate the risk of denial-of-service attacks.

When implementing patterns for matching extended patterns that necessisate references, any security considerations listed within those document MUST be accounted for.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

This document has been motivated by the [discussion](https://github.com/timbray/quamina/issues/210) within {{Quamina}} and is based on the [Patterns.md](https://github.com/timbray/quamina/blob/main/PATTERNS.md) file within the package. It also takes inspiration from the [README](https://github.com/aws/event-ruler/blob/main/README.md) within {{Ruler}}.
