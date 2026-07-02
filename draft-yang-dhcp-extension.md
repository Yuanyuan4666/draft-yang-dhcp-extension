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

This document specifies a DHCP option extension designed for campus networks to help client devices connect to a master device with Artificial Intelligence (AI) capabilities. The mechanism extends two specific parameters within the DHCP payload: the master device address information and the master device intelligent attribute identity. This allows client devices to identify and register to an AI-capable master device during the bootstrap phase, enabling them to utilize upstream AI capabilities and preventing the waste of master device processing resources.

--- middle

# Introduction

A campus network refers to a local area network established within a specific area (such as an enterprise, science park, school, or hospital) to provide specific services or meet specific requirements. Network elements within a campus network are divided into master devices and client devices. client devices must discover and register to a master device to complete networking, while the master device manages multiple registered client devices. With the development of AI, at least one master device in the campus network possesses AI capabilities.

In existing discovery and registration schemes, a client device selects a master device from multiple available options based solely on the discovery sequence or the current load conditions of the master devices. However, under these current practices, client devices cannot determine whether a master device possesses AI capabilities and may register to a non-AI-capable device. Consequently, client devices cannot request or utilize the AI capabilities of the master device and can only accept basic management, leading to a waste of the master device's AI resources.

To address this limitation, this document specifies a method for connecting to an AI-capable master device. The solution extends two distinct elements within the DHCP protocol payload:
1. **Master device address information**
2. **Master device intelligent attribute identity**

By delivering these two extensions during initial negotiation, client devices can successfully identify and connect to a master device with AI capabilities, allowing them to utilize upstream AI resources and avoiding the waste of computational capabilities.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


This document defines the following terms:

**Master Device**:
: Master Device could be a core switch or gateway equipped with hardware neural processing units. It hosts and executes the llm to perform configuration inference and network troubleshooting.

**Client Device**:
: Client Device could be an aggregation switche, access switche, or Wi-Fi Access Point (distributive deployed). Since they are constrained by hardware cost and power limits, they delegate heavy text and logic processing to the Master Device.

Note that Master Device is for generating policies while Client Device is for executing policies.

**Master Device Address Information**:
: The network coordinates used by a Client Device to reach the Master Device. It includes:
  * **Addr_Type**: Master devices' IPv4, IPv6, or FQDN addresses.
  * **LLM_Dest_Port**: Specifies the destination transport port for the Master device with llm.

**Master Device Intelligent Attribute Identity**:
: The capability profile of the Master Device's llm. It includes:
  * **LLM_Cap**: Indicates whether the model capability is active or at baseline.
  * **LLM_Scale**: Represents the llm parameter size in billions (B).
  * **LLM_Role**: Identifies the master device's role in a high-availability setup (Primary or Backup).
  * **API_Price**: Indicates the financial cost per million tokens.
    


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

# Protocol Flow

Two implementation methods can carry the required parameters within the protocol payload: a standalone new DHCP Option or a sub-option extension within the existing Vendor-Specific Information Option (Option 43). Both methods follow the identical standard DHCP sequence below, differing only in the specific Option code requested and returned:

~~~~
Client Device                                                 DHCP Server
     |                                                             |
     |--- DHCP Discover ------------------------------------------>|
     |    (PRL includes New Option or Option 43)                    |
     |                                                             |
     |<-- DHCP Offer ----------------------------------------------|
     |    (Carries Master Device Address Information               |
     |     & Master Device Intelligent Attribute Identity)         |
     |                                                             |
     |--- DHCP Request ------------------------------------------->|
     |                                                             |
     |<-- DHCP ACK ------------------------------------------------|
     |    (Carries Master Device Address Information               |
     |     & Master Device Intelligent Attribute Identity)         |
     |                                                             |
     v                                                             v
[Extract direct / Extract through while or for using pointer]
     |
     +--> Cap = 0x01 (Model Active)
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

DHCP extensions convey the Master Device Address Information and the Master Device Intelligent Attribute Identity. The two implementation methods only differ in their message formats as follows:

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

### Field Attribute Interpretations

**Master Device Address Information Parameters:**

Addr_Type:
: 1 byte. 0x01 indicates IPv4 (4 bytes); 0x02 indicates IPv6 (16 bytes); 0x03 indicates FQDN.

LLM_Dest_Port:
: 2 bytes. 0x0000 defaults to port 443 (HTTPS); otherwise specifies the active port.

**Master Device Intelligent Attribute Identity Parameters:**

LLM_Cap:
: 1 byte. 0x01 indicates Active; 0x00 indicates Baseline.

LLM_Scale:
: 2 bytes. Unsigned integer representing the model scale size in units of Billions (B).

LLM_Role:
: 1 byte. 0x01 indicates Primary Master; 0x02 indicates Backup Master.

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
