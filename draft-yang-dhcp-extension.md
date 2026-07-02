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

This document defines a Dynamic Host Configuration Protocol (DHCP) option extension designed to advertise the reachability and runtime parameters of an upstream intelligent control plane during the initial bootstrap phase. This mechanism allows client devices to perceive the intelligence profiles of upstream control plane prior to establishing sessions, mitigating control-plane latency and redundant session negotiation overhead.

--- middle

# Introduction

This document specifies a DHCP extension tailored for smart campus networks to automate intent-driven configurations. In a typical campus deployment, a centralized master device (such as a core switch) acts as the high-compute intelligence hub, while massive downstream elements (such as access switches and Wi-Fi APs) serve as lightweight client elements. Due to strict cost ceilings and power limits at the edge, neural processing units/graphics processing units (NPUs/GPUs) is exclusively centralized on the master device rather than distributed to every edge node.

Traditionally, these downstream clients remain blind to upstream computational profiles during bootup, relying on manual, static command-line configuration templates. Also, they acquire only basic IP parameters via standard DHCP and remains completely blind to the runtime computational profiles or presence of the upstream master device. Without early-stage capability awareness, edge client devices blindly attempt to establish higher protocol sessions (such as TLS, NETCONF, or Model Context Protocol (MCP) channels) with upstream controllers. If a selected controller lacks the required model active status, necessary parameter scale, or budget alignment required for specific local automation tasks, the session must be torn down and renegotiated with an alternative controller. This blind interconnectivity wastes link bandwidth and introduces severe control-plane setup latency.

To address this gap, this document specifies a DHCP option extension designed to advertise the reachability and runtime parameters of the upstream intelligent control plane during the initial bootstrap phase. By inserting metrics, such as model capability status, parameter scale, deployment hierarchy roles, and API pricing attributes—directly into early DHCP negotiations, client devices to instantly discover the profile of the upstream intelligent control plane. 

# Conventions and Definitions

{::boilerplate bcp14-tagged}


This document defines the following terms:

**Master Device**:
: Master Device could be a core switch or gateway equipped with hardware neural processing units. It hosts and executes the llm to perform configuration inference and network troubleshooting.

**Client Device**:
: Client Device could be an aggregation switche, access switche, or Wi-Fi Access Point (distributive deployed). Since they are constrained by hardware cost and power limits, they delegate heavy text and logic processing to the Master Device.
Note that Master Device is for generating policies while Client Device is for executing policies.

# Target Deployment Topology

The diagram below illustrates a smart campus network topology.

~~~~
                     +---------------------------------------+
                     |   Upstream Master Device (Core/GW)    |
                     |          [Centralized NPU / Model]    |
                     |         ====== DHCP Server ======     |
                     +---------------------------------------+
                                         |
               __________________________|__________________________
              |                                                     |
   +----------------------+                              +----------------------+
   | Aggregation Switch A |                              | Aggregation Switch B |
   +----------------------+                              +----------------------+
              |                                                     |
        ______|________________                               ______|________________
       |                      |                              |                      |
+------------------+  +------------------+            +------------------+  +------------------+
|  Access Switch   |  |    Wi-Fi7 AP     |            |  Access Switch   |  |    Wi-Fi7 AP     |
| [DHCP Client]    |  | [DHCP Client]    |            | [DHCP Client]    |  | [DHCP Client]    |
+------------------+  +------------------+            +------------------+  +------------------+
~~~~
## Topology Description

The deployment model implements an architecture structured as follows:

**Master Device**: The Upstream Master Device (Core/GW) at the root of the network acts as the centralized intelligence, deploying physical hardware acceleration  to run the intelligent model. It simultaneously operates as the DHCP Server.

**Client Device**: The downstream elements, including the Access Switches and Wi-Fi7 APs at the network edge, operate as lightweight DHCP Clients. 

Note: The intermediate Aggregation Switches serve as transparent layer-2 or layer-3 transport elements only for transporting traffic.

During the initial bootstrap phase, Client Devices broadcast standard DHCP Discover and Request messages up through the aggregation layer. These frames encapsulate the Parameter Request List (PRL), signaling their intent to discover upstream model intelligence profiles. In response, the Master Device transmits DHCP Offer and ACK messages down to the clients. This mechanism carries the newly extended option metadata, including model presence, parameter scale, deployment hierarchy roles, and operational cost structures back to the clients.

# Protocol Flow

There are two viable implementation methods to carry the required LLM metadata parameters within the protocol payload: a standalone new DHCP Option or a sub-option extension embedded within the existing Vendor-Specific Information Option (Option 43). They follow the identical standard DHCP sequence and the sequence below illustrates the interaction and extraction procedure:

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
     +--> Scale = 0x0048 (72B Model)
     +--> Role = 0x01 (Primary Master)
     +--> Price = Budget Verified
~~~~

## Operational Protocol Sequence

1. **DHCP Discover**: The client device broadcasts a DHCP Discover message. The Parameter Request List (PRL) includes either the newly allocated standalone Option code or Option 43, signaling its intent to perceive upstream capability profiles.
2. **DHCP Offer**: The DHCP Server (hosted on the Master Device) replies with a DHCP Offer encapsulating the initial intelligence profiles in its option payload.
3. **DHCP Request**: The client device selects the offer and transmits a DHCP Request to the DHCP server.
4. **DHCP ACK**: The server commits the allocation via a DHCP ACK message to the client.
5. **Extraction**:Upon receiving the final DHCPACK response, the client device terminates the state machine and extracts the target metadata.

## Where the Mechanisms Diverge

# Message Formats

DHCP New Option Extension options convey the LLM Address, LLM Capability Level (Scale-Based), API Pricing, and LLM Primary/Standby Roles. The implementation methods include the following two approaches, they only differ in the message formats as follows:

## New DHCP Option Format (Standalone Option)

~~~~
0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+---------------+-------------------------------+
|        OPTION_LLM_META        |       Option-Length           |
+---------------+---------------+---------------+---------------+
|    LLM_Cap    |        LLM_Scale              |  LLM_Role     |
+---------------+-------------------------------+---------------+
|                      API_Price                                |
+-------------------------------+--------------+----------------+
|       LLM_Dest_Port           |   Addr_Type  |                |
+-------------------------------+--------------+                |
|                                                               |
|           Address / Domain Name (Variable Length...)          |
|                                                               |
+---------------------------------------------------------------+
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
| SubLen=0x02   |          LLM_Scale (2 Bytes)  | SubType=0x03  | <-- Sub-TLV 2
+-------------------------------+-------------------------------+
| SubLen=0x01   | LLM_Role      | SubType=0x04  | SubLen=0x04   | <-- Sub-TLV 3
+-------------------------------+-------------------------------+
|                    API_Price (4 bytes)                        | <-- Sub-TLV 4
+-------------------------------+-------------------------------+
| SubType=0x05  | SubLen=0x02   |     LLM_Dest_Port             | <-- Sub-TLV 5
+-------------------------------+-------------------------------+
| SubType=0x06  | SubLen=Var    | Address/Domain (Variable...)  | <-- Sub-TLV 6
+-------------------------------+-------------------------------+
~~~~

## Field Attribute Interpretations

The two implementation methods share the same payload formats as follows:

LLM_Cap:
: 1 byte. 0x01 indicates Active; 0x00 indicates Baseline.

LLM_Scale:
: 2 bytes. Unsigned integer directly representing the model scale size in units of Billions (B). This continuous range accommodates diverse deployment sizes.

LLM_Role:
: 1 byte. 0x01 indicates Primary LLM; 0x02 indicates Backup LLM.

Addr_Type:
: 1 byte. 0x01 indicates IPv4 (4 bytes); 0x02 indicates IPv6 (16 bytes); 0x03 indicates FQDN.

LLM_Dest_Port:
: 2 bytes. 0x0000 defaults to port 443 (HTTPS); otherwise specifies the active port.

API_Price:
: 4 bytes. 0x0000000A represents the monetary cost per million tokens.

# Deployment Architecture Comparison

## Pros of Standalone New DHCP Option

* Cleaner design without nesting attributes inside complex structures.
* Fixed field offsets allow network chipsets to parse LLM metadata directly, eliminating pointer iterations that introduce latency and hardware memory consumption.
* No requirement to allocate 2 bytes for single attribute containers (subtype + sublen) across every field, resulting in lower overall packet size overhead.

## Pros of New Extension via Option 43

* Does not require a scarce global Option assignment from IANA.
* Legacy devices (e.g., intermediate DHCP relays) can natively recognize and forward Option 43 by treating it as an existing vendor container.
* Provides highly extensible fields; easy to add, remove, or modify new LLM attributes without breaking the main protocol container structure.

# Security Considerations

TBD

# IANA Considerations

This document requests IANA to allocate a new DHCP Option code from the standardized range (224 to 254) for the standalone option format, or alternatively register the sub-option definitions within the Option 43 internal sub-option registry space.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
