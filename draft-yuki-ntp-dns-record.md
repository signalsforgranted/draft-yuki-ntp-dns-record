---
title: "NTP DNS Resource Record"
category: info

docname: draft-yuki-ntp-dns-record-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Network Time Protocols"
keyword: internet-draft

author:
 -
    fullname:
      :: 後藤ゆき
      ascii: Yuki Goto
    organization: independent
    email: minami.hiroy@gmail.com

normative:
  RFC9460:

informative:
  NTPv5:
    title: "Network Time Protocol Version 5"
    seriesinfo:
      Internet-Draft: draft-ietf-ntp-ntpv5-07

...

--- abstract

This document defines a new NTP DNS Resource Record, similar in concept to the HTTPS DNS Resource Record specified in [RFC9460].

This record enables an NTP server to indicate, via DNS, the versions of the NTP protocol it supports prior to the initiation of any NTP message exchange.

--- middle

# Introduction

[NTPv5] is currently under standardization, and there are concerns regarding how clients select the newer NTP protocol version to use.

The server will drop NTP packets with unsupported versions. Therefore, clients need to select an NTP version that the server can receive; however, clients have no reliable way to know the server’s supported versions in advance. Accordingly, clients commonly initiate communication using NTPv4, assuming that NTPv4 is supported by the server, even if the server also implements NTPv5. Servers then attempt to advertise NTPv5 support to clients using the NTPv4 reference timestamp.

The version of NTP used in the first request is therefore effectively based on implicit assumptions rather than explicit information. This creates challenges for the deployment of future NTP protocol versions.

To address this challenge, this document defines a DNS-based mechanism similar to the HTTPS Resource Record ([RFC9460]). This mechanism enables a server to advertise the NTP protocol versions it supports before a client initiates any NTP communication.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# NTP Resource Record
The NTP Resource Record (RR type code TBD) is a SVCB-compatible RR type, as defined in [RFC9460].
It uses the same RDATA wire format as the SVCB RR, but its semantics are specialized for discovery and configuration of Network Time Protocol (NTP) services.

## Example
The following example illustrates the presentation format of the NTP RR, showing how a server advertises support for multiple NTP protocol versions using the ntp-version SvcParamKey.

~~~
ntp.example.com. 300 IN NTP 1 . ntp-version=4,5
~~~

## ntp-version SvcParams
The ntp-version SvcParamKey indicates the set of NTP protocol versions supported by the service endpoint. Its value is a comma-separated list of ASCII version identifiers. Each version identifier consists of a numeric version number, optionally followed by a hyphen and an alphanumeric label (e.g., “5-draft5”), allowing servers to indicate support for development, experimental, or otherwise distinguished variants of a protocol version.

ABNF:

~~~
version           = 1*DIGIT *( "-" version-label )
version-label     = 1*( ALPHA / DIGIT )
ntp-version-value = version *( "," version )
~~~

The wire-format consists of one or more version identifiers, each encoded as a length-prefixed byte sequence. These length–value pairs are concatenated to form the SvcParamValue.

## NTS

Time service providers offering NTS {{RFC8915}} can use SVCB records to announce the location of their NTS-KE services. This record must specify the ALPN value, however if no port number is specified clients should assume service availability on TCP 4460.

The following example shows an NTS-KE service address, and implied port number.
```
time.example.com 3600 IN SVCB 3 nts.example.com. ( alpn="ntske/1" )
```

# Operational Sequence and Client Behavior
The following list outlines the typical operational flow for deploying and using the NTP Resource Record, from server-side configuration to client-side version selection and communication. In practice, clients MAY perform NTP RR resolution in parallel with their default NTP initiation behavior (typically NTPv4, or NTPv5 when configured) and use the result when available.

- The server operator publishes an NTP RR in the authoritative DNS zone, including the appropriate SvcParams such as ntp-version="4,5".
- The client initiates DNS resolution for the NTP RR corresponding to the target domain name; if the NTP RR is not available, the client falls back to its default behavior (typically NTPv4).
- The client processes the DNS response using the rules for SVCB-compatible RR types.
  - If the response is in AliasMode (SvcPriority = 0), the client follows the alias target and repeats resolution.
  - If the response is in ServiceMode, the client proceeds to evaluate the associated SvcParams.
- The client parses known SvcParams; unknown parameters are ignored.
- If the ntp-version SvcParamKey is present, the client parses its value into a list of supported version identifiers.
- The client determines the intersection between its locally supported NTP versions and the versions listed in the ntp-version SvcParam.
- The client selects the highest mutually supported version and uses it as the version for the initial NTP request.
- If no mutually supported version exists, or if the NTP RR or its parameters are malformed, the client falls back to its default behavior (typically initiating communication using NTPv4).
- The client sends the first NTP request using the selected version.
- The server accepts the packet if the version is supported; packets using unsupported versions are dropped according to normal NTP behavior.
- NTP communication proceeds using the selected protocol version.

# Security Considerations

TODO


# IANA Considerations

## NTP RR Type

TODO

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
