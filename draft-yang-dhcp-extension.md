---
title: "DHCP New Option Extension based on LLM Capability"
abbrev: "DHCP LLM Extension"
category: info

docname: draft-yang-dhcp-extension-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date: 2026-06-30
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
  github: "Yuanyuan4666/draft-yang-dhcp-extension"
  latest: "https://Yuanyuan4666.github.io/draft-yang-dhcp-extension/draft-yang-dhcp-extension.html"

author:
 -
    fullname: "Yuanyuan Yang"
    organization: Huawei
    email: "yangyuanyuan55@huawei.com"

normative:

informative:

...

--- abstract

This document defines a DHCP option extension designed to advertise the reachability and runtime computational parameters of an upstream Large Language Model (LLM) control plane during the initial bootstrap phase. The specified metadata includes LLM capability status, parameter scale, deployment hierarchy roles, and API pricing attributes. This mechanism allows client devices to perceive the intelligence profiles of upstream controllers prior to establishing high-layer agent sessions, mitigating control-plane latency and redundant session negotiation overhead.

--- middle

# Introduction

Traditional network operations and management architectures heavily rely on legacy data forwarding to manage client devices (such as switches, routers, firewalls, Access Controllers, Access Points, and wired or wireless endpoints). Establishing stable, capacity-matched agent sessions between the master device and client devices requires a lightweight discovery mechanism at the link layer.

Modern network control planes are evolving toward an LLM Control Plane. Equipped with high performance NPUs, these controllers execute policy inference and orchestration, leveraging decoupled operational skills for configuration and troubleshooting. To anchor these architectures, the client device bootstraps and dynamically discovers the control plane's LLM parameters, enabling stable agent sessions.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Problem Statement

## Current Practice vs. Target Architecture

Current Practice: Characterized by traditional element management and legacy data forwarding, coupled with manual, static CLI dependencies.

Target Architecture: Transitioning to an LLM Control Plane equipped with high performance NPUs to execute policy inference and automated orchestration, leveraging decoupled operational skills for configuration and troubleshooting.

## Technical Gap

During initial network bootstrapping, a client device acquires only basic IP parameters via standard DHCP and cannot perceive the LLM capabilities of the upstream controller. Existing DHCP options cannot encapsulate runtime computational and intelligence capabilities (such as model presence, scale, or operational costs).

## Control-Plane Performance Bottlenecks

Without early-stage capability awareness, client devices blindly establish full protocol sessions (such as TLS, NETCONF, or MCP) with the controller. If the selected controller lacks the required LLM capability status, model scale, or budget alignment necessary for specific local tasks, the session must be torn down and renegotiated with an alternative controller. This blind interconnectivity wastes network bandwidth and introduces severe control-plane setup latency.

# Protocol Flow

When implementing the Option 43 extension pathway, the client parses the sub-option fields sequentially through pointer iterations (e.g., utilizing while or for loops based on sub-option lengths). The sequence below illustrates the programmatic exchange and data extraction boundary:

~~~~
Client Device                                                 DHCP Server
     |                                                             |
     |--- DHCP Discover ------------------------------------------>|
     |    (PRL includes New Option / Option 43)                    |
     |                                                             |
     |<-- DHCP Offer ----------------------------------------------|
     |    (Carries New Option / Option 43 payload)                 |
     |                                                             |
     |--- DHCP Request ------------------------------------------->|
     |                                                             |
     |<-- DHCP ACK ------------------------------------------------|
     |                                                             |
     v                                                             v
[Extract direct / Extract through while or for using pointer]
     |
     +--> Cap = 0x01 (LLM Active)
     +--> Scale = 0x48 (72B Model)
     +--> Role = 0x01 (Primary Master)
     +--> Price = Budget Verified
~~~~

The interaction shows how the client device discovers the parameters. Upon receiving the DHCPACK response, it extracts the target values including Cap (0x01 (LLM Active)), Scale (0x48 (72B Model)), Role (0x01 (Primary Master)), and Price (Budget Verified) via direct offsets or sub-option loop pointers.

# Message Formats

To facilitate product commercialization, this specification defines two viable deployment options: a standalone new DHCP Option or a sub-option extension embedded within the existing Vendor-Specific Information Option (Option 43).

DHCP New Option Extension: LLM Address, LLM Capability Level (Scale-Based), API Pricing, and LLM Primary/Standby Roles. The implementation methods include the following two approaches (both are viable options for product commercialization):

## New DHCP Option Format (Standalone Option)

~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------------------------------+-------------------------------+
| OPTION_LLM_METADATA (Range)   | Option-Length (Variable)      |
+-------------------------------+-------------------------------+
|  LLM_Cap (1B) |     LLM_Scale (2 Bytes)       |   LLM_Role    |
+-------------------------------+-------------------------------+
|                     LLM_Dest_Port (2 Bytes)                   |
+-------------------------------+-------------------------------+
|                      LLMA_Price (4 Octets)                    |
+-------------------------------+-------------------------------+
|   Addr_Type   | Address / Domain Name (Variable Length...)    |
+-------------------------------+-------------------------------+
~~~~

## Alternative Sub-option Format (Extension via Option 43)

~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------------------------------+-------------------------------+
| Type = 43 (Vendor-Specific)   | Main Option-Length            |
+-------------------------------+-------------------------------+
| SubType=0x01  | SubLen=0x01   | LLM_Cap(0x01) | SubType=0x02  | <-- Sub-TLV 1
+-------------------------------+-------------------------------+
| SubLen=0x02   |          LLM_Scale (2 Bytes)                  | <-- Sub-TLV 2
+-------------------------------+-------------------------------+
| SubType=0x03  | SubLen=0x01   | LLM_Role      | SubType=0x04  | <-- Sub-TLV 3
+-------------------------------+-------------------------------+
| SubLen=0x04   |          LLMA_Price (4 Octets)                | <-- Sub-TLV 4
+-------------------------------+-------------------------------+
| SubType=0x05  | SubLen=0x02   |     LLM_Dest_Port             | <-- Sub-TLV 5
+-------------------------------+-------------------------------+
| SubType=0x06  | SubLen=Var    | Address/Domain (Variable...)  | <-- Sub-TLV 6
+-------------------------------+-------------------------------+
~~~~

## Field Attribute Interpretations

LLM_Cap:
: 1 octet. 0x01 indicates Active; 0x00 indicates Baseline.

LLM_Scale:
: 2 octets. Unsigned integer directly representing the model scale size in units of Billions (B). This continuous range accommodates diverse deployment sizes.

LLM_Role:
: 1 octet. 0x01 indicates Primary LLM; 0x02 indicates Backup LLM.

Addr_Type:
: 1 octet. 0x01 indicates IPv4 (4 octets); 0x02 indicates IPv6 (16 octets); 0x03 indicates FQDN.

LLM_Dest_Port:
: 2 octets. 0x0000 defaults to port 443 (HTTPS); otherwise specifies the active port.

LLMA_Price:
: 4 octets. 0x0000000A represents the monetary cost per million tokens.

# Deployment Architecture Comparison

## Pros of Standalone New DHCP Option

* Cleaner design without nesting attributes inside complex structures.
* Fixed field offsets allow network chipsets to parse LLM metadata directly, eliminating pointer iterations that introduce latency and hardware memory consumption.
* No requirement to allocate 2 octets for single attribute containers (subtype + sublen) across every field, resulting in lower overall packet size overhead.

## Pros of New Extension via Option 43

* Does not require a scarce global Option assignment from IANA.
* Legacy devices (e.g., intermediate DHCP relays) can natively recognize and forward Option 43 by treating it as an existing vendor container.
* Provides highly extensible fields; easy to add, remove, or modify new LLM attributes without breaking the main protocol container structure.

# Security Considerations

Without early-stage capability awareness, client devices blindly establish full protocol sessions with unverified controllers. If malicious nodes spoof DHCP responses and advertise falsified LLM attributes, they can trick client devices into joining fraudulent control planes or consuming high-cost computational APIs. Mitigating these risks requires validation via standard network access control mechanisms and higher-layer channel authentication.

# IANA Considerations

This document requests IANA to allocate a new DHCP Option code from the standardized range (224 to 254) for the standalone option format, or alternatively register the sub-option definitions within the Option 43 internal sub-option registry space.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
