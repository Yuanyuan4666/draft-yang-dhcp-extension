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
 - DHCP Extension
 - LLM Capability
 - Increase Automation
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
  RFC2119:
    title: "Key words for use in RFCs to Indicate Requirement Levels"
  RFC8174:
    title: "Ambiguity of Uppercase %BCP14 Keywords should be Avoided"
  RFC2132:
    title: "DHCP Options and BOOTP Vendor Extensions"
  RFC1035:
    title: "Domain Names - Implementation and Specification"
  RFC3118:
    title: "Authentication for DHCP Messages"

informative:

...

--- abstract

This document specifies a DHCP option extension designed for campus networks to help client devices distinguish and connect to a master device with the LLM (Large Language Model). The mechanism extends a new DHCP option containing two specific parameters within the DHCP payload: the master device's LLM address and the master device's LLM configuration. This allows client devices to identify and register to LLM-enabled master device during the bootstrap phase.

--- middle

# Introduction

A campus network refers to a network established within a specific area, such as an enterprise, science park, school, or hospital. Network elements within a campus network are divided into master devices (such as a core switch or a gateway) and client devices (such as an access switch or an AP). Client devices must discover and register to a master device to complete networking, while the master device manages multiple registered client devices.

Centralized campus Artificial Intelligence for IT Operations (AIOps) relies on cloud data centers. Local devices upload logs and alarms to the cloud for LLM analysis. This introduces two pitfalls:
1. **Data Privacy**: Regulations prohibit uploading internal network topology and business traffic data to public clouds.
2. **High Latency**: Cloud interactions over WAN (Wide Area Network) introduce high latency, failing the real-time requirements for network self-healing.

To eliminate these bottlenecks, shifting LLM inference to the network edge is the current trend. Core switches and gateways are now equipped with NPU/GPU hardware. This distributed architecture keeps sensitive data within the campus and eliminates cloud latency, enabling real-time root-cause analysis and troubleshooting directly at the edge.

Currently, the master device's LLM address is manually configured via CLI or hardcoded into client devices. Although standard DHCP automatically assigns basic parameters like IP addresses, subnets, and gateways, it cannot indicate whether a master device possesses LLM capabilities. This means that client devices cannot automatically identify which master devices are LLM-enabled. Consequently, they may register to a non-LLM-enabled device and cannot request or utilize the LLM capabilities of the master device for network configuration or troubleshooting, which also leads to a waste of the master device's LLM resources.

To address this limitation, this document extends two distinct elements within the DHCP protocol payload through a new DHCP option, to help client devices distinguish and connect to a master device with LLM capabilities:
1. **the master device's LLM address**
2. **the master device's LLM configuration**

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

This document defines the following roles:

**Master Device**:
: The network element that hosts and executes the LLM to perform configuration and network troubleshooting inference, which operates as the DHCP Server. A master device could be a core switch or a gateway equipped with hardware neural processing units.

**Client Device**:
: The network element that delegates heavy text and logic processing to the Master Device due to hardware cost and power limits, which operates as lightweight DHCP Client. Client Device could be an aggregation switch, access switch, or a Wi-Fi Access Point (distributive deployed).

# Typical Deployment Topology

The diagram below illustrates a typical smart campus network topology.

~~~~
                     +---------------------------------------+
                     |   Upstream Master Device (Core/GW)    |
                     |   [Centralized NPU or NPU / Model]    |
                     |         ====== DHCP Server ======     |
                     +---------------------------------------+
                                         |
             ____________________________|___________________________
             |                                                       |
   +----------------------+                               +----------------------+
   | Aggregation Switch A |                               | Aggregation Switch B |
   +----------------------+                               +----------------------+
              |                                                      |
        ______|________________                                ______|________________
       |                      |                               |                      |
+------------------+  +------------------+             +------------------+  +------------------+
|   Access Switch  |  |     Wi-Fi7 AP    |             |   Access Switch  |  |     Wi-Fi7 AP    |
| [DHCP Client]    |  | [DHCP Client]    |             | [DHCP Client]    |  | [DHCP Client]    |
+------------------+  +------------------+             +------------------+  +------------------+
~~~~

**Master Device**: The Upstream Master Device (Core/GW) at the root of the network acts as the centralized intelligence center, utilizing hardware acceleration to run the LLM.

**Client Device**: The downstream elements, including the Access Switches and Wi-Fi7 APs at the network edge.

Note: The intermediate Aggregation Switches serve as transparent layer-2 or layer-3 transport elements only for transporting traffic.

# Message Formats

DHCP extensions convey the master device's LLM address and the master device's LLM configuration. The format of the new DHCP option (OPTION_LLM_META) is defined as follows:

## New DHCP Option Format (OPTION_LLM_META)

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
|            Dest_Port          |   Addr_Type  |                |
+-------------------------------+--------------+                |
|                                                               |
|           Address / Domain Name (Variable Length...)          |
|                                                               |
+---------------------------------------------------------------+
~~~~

### Field Attribute Interpretations

**the master device's LLM address parameters:**

Addr_Type:
: 1 byte. Indicates the format of the following Address/Domain Name. 0x01 indicates IPv4 (4 bytes); 0x02 indicates IPv6 (16 bytes); 0x03 indicates fully qualified domain name (FQDN).

Dest_Port:
: 2 bytes. Indicates the port used to access the LLM service. 0x0000 defaults to port 443 (HTTPS); otherwise specifies the active port.

Address / Domain Name:
: Variable length. Contains the IPv4 address/IPv6 address/FQDN of the master device. If Addr_Type is 0x01, it MUST be a 4-byte IPv4 address; If Addr_Type is 0x02, it MUST be a 16-byte IPv6 address; If Addr_Type is 0x03, it MUST be a DNS-encoded FQDN(as specified in [RFC1035]).

**the master device's LLM configuration parameters:**

LLM_Cap:
: 1 byte. 0x01 indicates Active; 0x00 indicates Baseline.

LLM_Scale:
: 2 bytes. Unsigned integer representing the model scale size in units of Billions (B).

LLM_Role:
: 1 byte. 0x01 indicates Primary Master; 0x02 indicates Backup Master.

API_Price:
: 4 bytes. 0x0000000A represents the monetary cost per million tokens.

# Client Behavior

If a DHCP client requires the LLM metadata, it MUST include OPTION_LLM_META in the Parameter Request List (PRL) option, as described in [RFC2132]. When a DHCP client receives OPTION_LLM_META, it MUST perform the following validation checks:
* Verify that the `Option-Length` matches the required structure minimums defined in this document.
* If `Addr_Type` is 0x03 (FQDN), verify that the Address / Domain Name field does not exceed 255 octets and represents a properly formatted domain name as specified in [RFC1035].

# Server Behavior

A DHCP server supporting this specification MUST be capable of configuring and storing the LLM metadata, including the master device's LLM address and the master device's LLM configuration parameters. This extension does not introduce any new DHCP message types. The server processing logic MUST comply with the followings:

* Upon receiving a DHCP DISCOVER or DHCP REQUEST message, the server MUST inspect the PRL option. If OPTION_LLM_META is specified in the PRL, the server SHOULD append this option into its corresponding DHCP OFFER and DHCP ACK responses.
* If a client does not request OPTION_LLM_META in its PRL, or if the server itself is not configured with LLM capabilities, the server MUST NOT include OPTION_LLM_META in its reply messages.

# Security Considerations

The communication between the DHCP client and the DHCP server for exchanging LLM address and configuration parameters is security sensitive and requires server authentication and integrity protection. DHCPv4 authentication mechanisms specified in [RFC3118] can be used for this purpose.

# IANA Considerations

IANA is requested to assign a new DHCP Option code for OPTION_LLM_META from the "BOOTP Vendor Extensions and DHCP Options" registry maintained at http://www.iana.org/.

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
