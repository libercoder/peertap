# PeerTap Specification

draft status: R&D

date: 2025-07-07

version: 0.2.0

# Overview

PeerTap is a decentralized, censorship-resistant Layer 2 overlay network that links participants into cryptographically isolated broadcast domains called CryptoVLANs. Each VLAN is protected using a combination of TreeKEM-based MLS group encryption and end-to-end hybrid encryption.

All traffic between nodes is transmitted over encrypted libp2p trunk connections. VLAN traffic is encapsulated as raw Ethernet frames and tunneled through the mesh network, providing transparent L2 connectivity between participants.

Since all traffic is encrypted and anonymized uniformly using VLAN Forwarding Tokens (VFTs), there is no technical or privacy-based reason to fragment the trunk connectivity into isolated segments. On the contrary, the design encourages connecting all nodes into a global relay mesh. This enhances availability, reduces the risk of partitioning, and increases resilience against network-level censorship. Logical isolation and segmentation are strictly enforced at the VLAN level via cryptographic means, allowing the underlying infrastructure to remain flat, decentralized, and robust—regardless of dynamic changes in the trunk mesh topology.

Each CryptoVLAN is assigned a static 128-bit VLAN-ID, derived from a creator's public key or multisignature and a label, which identifies it within the switching fabric. A separate dynamic MLS-ID reflects the state of the group encryption key.

Forwarding across the mesh is handled using VLAN Forwarding Tokens (VFT), which allow all types of traffic—unicast, broadcast, and multicast—to be routed anonymously and uniformly. Relays forward traffic without inspecting or distinguishing payload types, based solely on VFT mappings.

A node is defined as a physical or virtual device running a PeerTap instance. The application:

- Establishes libp2p connections with other peers
- Manages VLAN membership and cryptographic state
- Creates TAP interfaces in the OS for each joined VLAN
- Forwards tagged traffic locally or across trunks
- Uses internal untagged Management VLAN (MVLAN), which have not TAP/TUN interfaces, for monitoring, coordination and discovery

---

# Terminology

| TermDescription |                                                                                                                                                                                                      |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| VLAN-ID         | Static 128-bit tag derived from creator public key and label: BLAKE3(PubKey\_Creator + Label). The PubKey\_Creator may belong to a single participant, a multisig group, or configuration authority. |
| MLS-ID          | Dynamic identifier derived from current TreeKEM group state: BLAKE3(GroupInitKey)                                                                                                                    |
| CryptoVLAN      | A broadcast domain identified by VLAN-ID, protected by MLS group keys                                                                                                                                |
| CTag            | Field in packet carrying VLAN-ID                                                                                                                                                                     |
| Relay           | Any Node configured to forward tagged traffic                                                                                                                                                        |
| MVLAN           | Special VLAN with untagged frames, used for control traffic                                                                                                                                          |
| VFT             | VLAN Forwarding Token, used to route all packet types (unicast, broadcast, multicast) anonymously and efficiently through the relay mesh                                                             |
| Node            | A device (physical or virtual) running an instance of the PeerTap application                                                                                                                        |

---

# Layer Model (Data Link Only)

## Switching Layer (Layer 2)

- Responsible for forwarding frames within CryptoVLANs
- Forwards based on static VLAN-ID
- Forwards untagged packets only within MVLAN
- Trunk-like behavior is handled entirely at the switching level

---

# Packet Format

| Field           | Size       | Description                              |
| --------------- | ---------- | ---------------------------------------- |
| VLAN-ID (CTag)  | 16 bytes   | Static identifier of the VLAN            |
| MLS-ID          | 16 bytes   | Current MLS group key hash               |
| VFT             | 8–16 bytes | Forwarding token used by relays          |
| Nonce / IV      | 12 bytes   | Used for authenticated encryption        |
| Encrypted Frame | variable   | Full Ethernet frame (L2+L3)              |
| Auth Tag        | variable   | Authentication tag for payload integrity |

- Encrypted payload contains a complete raw Ethernet frame.
- IP addressing inside VLAN is handled entirely within the encrypted L2 frame.

---

# Cryptographic Primitives

- Trunk Encryption: All libp2p trunk links between nodes are protected using end-to-end encryption, independent of VLAN-level encryption. This ensures full confidentiality and integrity of all overlay traffic across the mesh.
- VLAN-ID: BLAKE3-128(PubKey\_Creator + Label)
- MLS-ID: BLAKE3-128(GroupInitKey)
- VFT:
  - Unicast: BLAKE3-128(MAC\_TAP + VLAN-ID + salt)
  - Broadcast: BLAKE3-128("FFFFFFFFFFFF" + VLAN-ID)
  - Multicast: BLAKE3-128(multicast\_MAC\_mapped\_from\_IP + VLAN-ID)
- Group Encryption: MLS (TreeKEM-based), with forward secrecy

---

# Discovery Protocols

## VLAN Discovery

- Management VLAN is used to broadcast presence (HELLO) with:
  - Supported VLAN-IDs
  - Supported Relay Capabilities
  - Latency metrics

---

# Topology Management

## Relay Coordination

- Relays announce VLAN-ID forwarding capabilities in MVLAN
- Each relay maintains an Access Control List (ACL) specifying:
  - Whether to forward all VLANs (open relay mode)
  - Only VLANs the relay is a member of (restricted mode)
  - Specific allowed or denied VLAN-ID patterns (whitelist/blacklist mode)
- ACLs can be defined statically or updated dynamically via signed control messages
- This enables granular control over relay participation and forwarding policy
- Peers may elect to route via relays if direct connection is:
  - Not possible (Firewalls/DPI)
  - Inferior (latency/bandwidth)

## Loop Avoidance

- Uses a variant of Rapid Spanning Tree (CRST)
- Periodic signed root announcements
- Paths are evaluated and weighted

---

# Switching Strategy

- Peers evaluate best paths using VLAN-ID, latency and path cost
- Each packet includes both CTag and MLS-ID
- Packets are encrypted as full Ethernet frames, with IP addressing managed locally
- Relay nodes forward traffic based on universal forwarding logic using VFT

---

# Forwarding Model

All packets (unicast, broadcast, multicast) are forwarded using the same anonymous mechanism based on the VFT. The relay does not distinguish packet type and routes each frame using a consistent table-based method:

VFT → [list of peerIDs to forward to]

## Unicast (1:1)

- Sender generates VFT = BLAKE3(MAC\_TAP + VLAN-ID + salt)
- This corresponds to a unique recipient
- Relay sees only the VFT and forwards according to learned or announced destination peerID

## Broadcast (1\:N to all VLAN members)

- A reserved MAC address "FFFFFFFFFFFF" is used to derive a broadcast VFT
- Relay forwards to all known peers in the VLAN’s spanning tree

## Multicast (1\:N or N\:N subsets)

- VFT = BLAKE3(multicast\_MAC\_mapped\_from\_IP + VLAN-ID), according to standard IPv4/IPv6 IP→MAC mapping rules
- Relays maintain forwarding trees based on subscriptions or precomputed delivery graphs
- Enables efficient scoped delivery within a VLAN without revealing traffic class

## Full Duplex

- Replies use symmetric VFTs
- Routing remains symmetric and stateless

---

# Fragmentation

In cases where the encrypted Ethernet frame exceeds the MTU (typically 1500 bytes), PeerTap supports simple fragmentation **before encryption**. This allows large payloads to be transmitted securely without relying on underlying IP-level fragmentation.

- Each fragment is encapsulated in its own PeerTap packet
- Shared fields:
  - Fragment-ID: 128-bit unique identifier (e.g., BLAKE3 of payload)
  - Offset: position in the original frame (in bytes)
  - More flag or total size to indicate reassembly completion
- Reassembly happens only at the receiving node
- Relays do **not** buffer or track fragments

This approach is stateless for the network and maintains end-to-end encryption without exposure of internal structure.

---

# Security & Privacy

- Replay Protection: Nonce-based, with TTL expiration
- Authentication: All frames signed by sender
- Anonymity: Based on VFT and indirect relay paths
- Censorship Resistance: Mesh overlay, path selection
- MAC Collision Mitigation:
  - MAC = (BLAKE3(PubKey + VLAN-ID + RandomNonce) & 0xFFFFFFFFFFFF) | 0x020000000000
  - Local administration bit is set
  - Conflicts resolved by regenerating MAC with new salt
  - Detection via encrypted probe/response within CryptoVLAN

---

# Mobile / TUN Support (Experimental)

Mobile platforms (e.g., Android, iOS) restrict raw TAP access without root permissions. PeerTap can optionally support TUN-mode operation for these platforms:

- Each TUN interface is assigned a locally generated MAC address
- VFTs are computed using the assigned MAC address just like in TAP mode:
  - Unicast: BLAKE3(MAC\_TAP + VLAN-ID + salt)
  - Broadcast: BLAKE3("FFFFFFFFFFFF" + VLAN-ID)
  - Multicast: BLAKE3(multicast\_MAC\_mapped\_from\_IP + VLAN-ID)
- L2 semantics (e.g., MAC addressing, broadcast domains) are emulated locally per node
- TUN-mode VLANs may implement support for unicast, broadcast, and multicast in user space
- Interoperability with TAP-mode is transparently handled by the local PeerTap instance through internal encapsulation logic.
- Used via platform VPN APIs (e.g., VpnService on Android, NEPacketTunnelProvider on iOS)
- **On macOS**, TAP interfaces are not natively supported; only TUN interfaces via `utunX` are available. Layer-2 behavior is emulated over TUN in user space.

This mode enables deployment on mobile platforms and macOS without compromising the encrypted trunk model or general anonymity.

---

# Open Questions / Research Areas

- Best discovery strategy: Gossip vs DHT vs pubsub
- Relay trust scoring or incentives
- Trunks encryption

---

# Research & Extension Ideas

- Optimized broadcast trees
- Load-aware relay balancing

