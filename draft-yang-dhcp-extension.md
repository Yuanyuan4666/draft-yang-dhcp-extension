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

This document defines a Dynamic Host Configuration Protocol (DHCP) option extension designed to advertise the reachability and runtime computational parameters of an upstream intelligent control plane during the initial bootstrap phase. The specified metadata includes LLM capability status, parameter scale, deployment hierarchy roles, and API pricing attributes. This mechanism allows client devices to perceive the intelligence profiles of upstream controllers prior to establishing high-layer agent sessions, mitigating control-plane latency and redundant session negotiation overhead.

--- middle

# Introduction

This document specifies a DHCP extension tailored for smart campus networks to automate intent-driven configurations. In a typical campus deployment, a centralized master device (such as a core switch) acts as the high-compute intelligence hub, while massive downstream elements (such as access switches and Wi-Fi APs) serve as lightweight client elements. Due to strict cost ceilings and power limits at the edge, neural processing hardware (NPUs/GPUs) is exclusively centralized on the master device rather than distributed to every edge node.

Traditionally, these downstream clients remain blind to upstream computational profiles during bootup, relying on manual, static command-line configuration templates. They often waste link bandwidth and introduce latency by blindly attempting full-stack protocol sessions with unaligned controllers. By inserting key model metrics directly into early DHCP lease negotiations, this extension allows edge client devices to instantly discover the processing profile of their upstream master device. 

# Conventions and Definitions

{::boilerplate bcp14-tagged}


This document defines the following terms:

Master Device:
: The central network hub (such as a core switch or gateway) equipped with hardware neural processing units. It hosts and executes the llm to perform configuration inference and network troubleshooting.
Client Device:
: The downstream edge elements (such as aggregation switches, access switches, and Wi-Fi Access Points) that are constrained by hardware cost and power limits. They delegate heavy text and logic processing to the Master Device.

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

# Problem Statement

## Current Practice vs. Target Architecture

Current Practice: Characterized by traditional element management and legacy data forwarding, coupled with manual, static CLI dependencies.

Target Architecture: Transitioning to an intelligent control plane equipped with high performance NPUs to execute policy inference and automated orchestration, leveraging decoupled operational skills for configuration and troubleshooting.

## Technical Gap

During initial network bootstrapping, a client device acquires only basic IP parameters via standard DHCP and cannot perceive the LLM capabilities of the upstream controller. Existing DHCP options cannot encapsulate runtime computational and intelligence capabilities, such as model presence, scale, or operational costs.

Without early-stage capability awareness, client devices blindly establish full protocol sessions (such as TLS, NETCONF, or MCP) with the controller. If the selected controller lacks the required LLM capability status, model scale, or budget alignment necessary for specific local tasks, the session must be torn down and renegotiated with an alternative controller. This blind interconnectivity wastes network bandwidth and introduces severe control-plane setup latency.

# Protocol Flow

There are two viable implementation methods to carry the required LLM metadata parameters within the protocol payload: a standalone new DHCP Option or a sub-option extension embedded within the existing Vendor-Specific Information Option (Option 43). The sequence below illustrates the interaction and extraction procedure:

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

The interaction diagram above shows how the client device discovers the parameters. Upon receiving the final DHCPACK response, the client device terminates the state machine and extracts the target metadata. Depending on the deployment approach, the parameters are extracted either via direct structural field offsets (for the standalone new option) or parsed sequentially through pointer iterations using while/for loops to traverse the internal sub-TLVs (for the Option 43 extension pathway). The extracted boundary values define the session properties: Cap (0x01 (LLM Active)), Scale (0x0048 (72B Model)), Role (0x01 (Primary Master)), and Price (Budget Verified).

# Message Formats

DHCP New Option Extension options convey the LLM Address, LLM Capability Level (Scale-Based), API Pricing, and LLM Primary/Standby Roles. The implementation methods include the following two approaches, both of which are viable options for product deployment:

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
