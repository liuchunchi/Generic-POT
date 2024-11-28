---
title: Generic Proof of Transit Mechanism
abbrev: Generic POT
category: info

docname: draft-nasr-liu-generic-pot-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
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
  github: "liuchunchi/Generic-POT"
  latest: "https://liuchunchi.github.io/Generic-POT/draft-nasr-liu-generic-pot.html"

author:
 -
    fullname: "Chunchi Liu"
    organization: Your Organization Here
    email: "131236634+liuchunchi@users.noreply.github.com"

normative:

informative:

--- abstract

This document describes a generic abstraction of Proof of Transit (PoT) mechanisms. It provides abstract roles and conceptual messages to help different PoT mechanism designs work together.

--- middle

# Introduction {#intro}

The Proof-of-Transit (PoT) mechanism provides a cryptographically secure record of a message's forwarding trail at a specified granularity, serving as a security complement to packet-steering techniques such as Segment Routing (SR), Traffic Engineering (TE) and Policy-Based Routing (PBR).

Emerging client requirements, such as regulatory compliance and customized security assurance, necessitate the capability to record a packet's exact forwarding history. In response, various Proof-of-Transit mechanisms have been proposed. To facilitate interoperability across domains, this document distills the common characteristics of existing PoT mechanisms, enabling clients to deploy diverse designs while maintaining cross-domain compatibility.

# Terminology {#term}

* Proof-of-Transit (PoT): A PoT is a cryptographically verifiable proof that a message has been processed by a specific network element, transmitted either in-band as a message tag or out-of-band via a separate channel.
* PoT Mechanism: A PoT mechanism specifies the algorithms and procedures for calculating and verifying PoT, along with the requisite protocol extensions for its transmission.

# Roles

- Node: A forwarding network element on the predetermined path. It inspects, processes or verifies the packet header of a certain level so as to be visible.

- Producer: Produces a PoT.

- Verifier: Verifies a PoT for a message. The verifier can be a network element on the path, a network element at the end of a path, a controlling element like a border gateway, an end-client or a controller.

- Decision-making Point: The decision-making point can make further packet-level processing decisions according to PoT verification result. It can choose to accept, forward or drop a message.

- Setup: Produces necessary auxillary information to produce or verify PoTs. Usually the setup is the network controller or an equivalence.

# Messages

- Input: The necessary information that uniquely identifies the message at this granularity. For example, if it is at stream level, the input can be the five-tuple. If it is at packet level, the input can be the thumbprint of the IP header and its payload. To add device processing details, the input should also include ingress interface, egress interface, and other device-specific information.

- Output:
  * In-situ Tag: The PoT is attached to the message as a tag. The end node or the intermediate nodes can verify and/or update it. This tag can be carried using In Situ Operations, Administration, and Maintenance (IOAM), Alternate Marking, or other In-Band Network Telemetry techniques. The in-situ tag has two calculation types:

  * Out-of-band Message: The PoT is sent out-of-band via a separate channel. The verifier or decision-making point not on the path can verify it. This message can be sent using IPFIX, NetFlow, Netconf/YANG, SNMP, gNMI/gRPC, etc.

- Auxillary Information: The auxillary information is used to calculate a PoT. It includes public cryptographic parameters, keys, secret values, profiles etc.

- Verification Reference: The verification reference is a deterministic reference value used during the PoT verification process. It can be identiical to the expected PoT, or an input to PoT verification algorithm. The Verifier or the Decision-making Point must possess a verification reference.

# Steps of a Typical PoT Mechanism

## Setup Reference Baseline

- Controller Setup: The controller computes the path, obtaining the forwarding baseline. It then computes the verification reference for each individual verifier node on the path, and directly configure them to the respective node. The controller may have the keys or secret value of the producers/verifiers to compute the verification references. It should be considered a trusted setup.
- Distributed Setup: When no controller is present, the verification references are generated via a dial-test like method and distributed along the path.

Each verifier or producer also requires an identification mechanism that triggers verification or production of PoT. During the setup, packet or stream characteristics should be recorded along with the verification reference or auxillary information.

## Calculation

- Calculate-and-Replace: The Producer calculates its PoT and replaces the current PoT on the packet. It can either be a direct replacement (usually needs verification) or an aggregation with the current PoT (may not need verification).

~~~~~~~~~~
+-------------+         +-------------+
| header      |         | header      |
| +---------+ |         | +---------+ |
| | POT X   | |   ----> | | POT Y   | |
| +---------+ |         | +---------+ |
+-------------+         +-------------+
~~~~~~~~~~

- Calculate-and-Concatenate: The producer calculates its PoT and concatenates it to the current PoT on the packet.

~~~~~~~~~~
+-------------+         +-------------+
| header      |         | header      |
| +---------+ |         | +---------+ |
| | POT X   | |   ----> | | POT X   | |
| +---------+ |         | +---------+ |
+-------------+         | | POT Y   | |
                        | +---------+ |
                        |     ...     |
                        +-------------+
~~~~~~~~~~

- Calculate-and-send: The producer calculates its PoT and send it out-of-band via a separate channel.

## Verification

- End-verify: This happens when the end-node or end-client verifies the PoT. It saves intermediate verification time, but possible deviation will not be found in real-time. No immediate responses possible.

- Mid-verify: This happens when the intermediate node verifies the PoT. Each mid-verifier can also make packet decisions. This increases overhead but can discover deviation in real-time, and immediate responses like packet drops can take place. This can minimize attack window and the risk of data leakage to non-secure devices.

# Existing Instances

Frank Brockners et al. produced a PoT mechanism based on revised Shamir's Secret Sharing Scheme {{?I-D.ietf-sfc-proof-of-transit-08}}.
Luigi Iannone et al. produced a PoT mechanism based on Hash-based Message Authentication Codes {{?I-D.iannone-spring-srv6-pot-00}}.

# Use Case Analysis

- Geofencing:

In the context of data transmission, geofencing refers to the practice of forwarding specific data within a virtual perimeter that corresponds to a defined real-world geographic area. In this use case, the border gateway device is the verifier and decision making point. If the carried POT does not pass verification, it can drop the packet to protect data confidentiality.

~~~~~~~~~~
                  Jurisdiction A    Jurisdiction B
                                   |
                                   |  Border
                                   |  Gateway
      +--------+   +--------+  +---+------+
      |        |   |        |  | Verifier,|
... --> Node 1 +---> Node 2 +--> Decision |
      |        |   |        |  |  Point   |
      +--------+   +--------+  +---+------+
                                   |
                                   |
                                   |
~~~~~~~~~~

- SRv6 Path Validation

When in SRv6 strict mode, the forwarding must strictly adhere to the segment list. In this case, each node is also a verifier (potentially decision making point), each verifies the carried POT before calculating its own.

~~~~~~~~~~
+----------+  +----------+  +----------+
| Node 1,  |  | Node 2,  |  | Node 3,  |
| Verifier +--> Verifier +--> Verifier |
+----------+  +----------+  +----------+
~~~~~~~~~~

# Cross-domain Interoperability

When the domain A and domain B has implemented differnet PoT mechanisms, they are obviously not interoperable. But there are two ways to enable interoperability:

- Pass-down Results: The border gateway of domain A act as an end-verifier. It verifies the PoT, gets a result, sign it, remove PoT details, and pass the verification result down to domain B. The verification result can be a binary result, or contains deviation logs. The border gateway can choose to drop the packet if its policy mandates.

- Pass-down Details: The border gateway of domain A pass down the original PoT and necessary auxillary information of verification with a signature. The gateway or other verifiers at domain B conduct the verification. Due to the large size of the auxillary information, it should be sent via APIs or other OOB methods.

However, when operating in one single domain, different vendors should agree on implementing and using one same PoT mechanism.

# Security Considerations

## Replay Attacks

A replay attack occurs when a PoT is used more than once. An attacker can intercept the PoT tag and use it again. It usually occurs when the PoT is static or not using nonces.

## Forgery Attacks

We should assume a POT should require some unique secret value or keys to compute. When proper key rotation or key derivation methods are applied, forgery attacks can be avoided.

## Removal

Since PoT can reflect potential deviation, a malicious attacker may try to remove a PoT tag from the packet that indicates a reroute.

## L2 stealth devices

A L2 stealth device is usually a switch that only process L2 headers, such that they do not leave a mark on the L3 IP header. It is to the opposite to the definition of node in this document since they have no visibility to be perceived. In the list, we discussed and agreed that a PoT mechanism is only able to record forwarding trail of a specific level. For now when dealing with SRv6 or other L3 use cases, L2 is out-of-scope.

Another way to mitigate such threat is to extend PoT data field on L2 headers. For a controlled domain, the controller usually is aware of the real L2 topology, thus the L2 device between L3 devices. Using similar algorithms and requiring L2 devices to do the same, L2 visibility is doable in the future.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
