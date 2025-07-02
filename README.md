# 🌐 Summary: Wakunet fully Decentralized L2 Virtual Network

This project aims to build a **fully decentralized, cryptographically secure Layer 2 (L2) virtual networking system**, designed for both desktop and mobile devices, and capable of forming **virtual Ethernet broadcast domains** over a libp2p overlay.

---

## ⚙️ Core Concepts

- **Peer-to-peer (P2P) networking** is built on top of [**libp2p**](https://libp2p.io), using encrypted persistent connections ("**trunks**") between peers.
- Peers **do not rely on centralized servers**, and all discovery, connection, and routing is handled through decentralized protocols and cryptographic authentication.
- The network supports **VLAN-style segmentation**, where each virtual network (VLAN) functions like a broadcast domain with optional group encryption.

---

## 🧱 Network Layers

### 1. **libp2p Transport Layer**

- Trunk connections are configurable **end-to-end encrypted** using ECDH-derived keys and symmetric ciphers like AES-GCM or ChaCha20-Poly1305.
- Peer discovery is **continuous**. Trunks are maintained with a limit (`connectionLimit`), while other known peers are kept "alive" via lightweight heartbeat messages.

### 2. **Trunks**

- Persistent, secure libp2p streams established between peers.
- Trunks exist independently of VLANs and are used to route control and data packets.

---

## 🛰 VLAN Overlay Network

- A **VLAN** is identified by a BLAKE3-128 hash of a public key.
- VLANs can be:
  - **Open**: Anyone can join using the VLAN ID.
  - **Private**: Require cryptographic authorization (e.g., via [MLS](https://messaginglayersecurity.rocks)).
- On joining a VLAN:
  - A **tap interface** is created on the host system (for Ethernet-level interaction).
  - The peer receives an IP address (via distributed DHCP or manual config).
  - All intra-VLAN traffic is optionally encrypted using **group keying** (e.g., MLS).

---

## 🔄 Switching & Routing Logic

The network emulates a virtual Ethernet switch:

| Traffic Type | Handling |
|--------------|----------|
| **Unicast**  | Directed using MAC/peer forwarding tables |
| **Broadcast**| Routed only to peers in the same VLAN using a spanning tree |
| **Multicast**| Routed based on group subscription tables (IGMP-style) |

Each peer maintains:

- A **MAC forwarding table**
- A **spanning tree per VLAN**
- A **multicast group subscription table**
- A **frame hash cache** to prevent loops and duplicates

---

## 🔐 Cryptographic Privacy

- **All trunks** are encrypted.
- **VLANs** can optionally use **Messaging Layer Security (MLS)** for group encryption.
- Each VLAN maintains its own cryptographic group state — adding/removing members triggers rekeying.
- VLAN metadata (e.g., name, icon, policy) can be published and browsed over the network.

---

## 🌍 Gateway and Blockchain Integration

Three **independent roles** may exist in the network:

### 1. **Blockchain Node (Proof-of-Stake)**

- Participates in a PoS blockchain for governance, staking, or consensus.
- Has no direct relation to VLAN or gateway responsibilities.

### 2. **Gateway Node**

- Provides **exit-to-internet services** for overlay traffic.
- Access to gateway features may be gated via:
  - Token payments
  - ZK proofs
  - VLAN policies

### 3. **Regular node**

- Can start a gateway role

> **Important**: A node **cannot** act as both a blockchain node and a gateway simultaneously.

---

## 🧪 Use Cases

### 🔐 Globally-Available, Cryptographically-Secure Broadcast Domains

Unlike typical decentralized systems where only apps implementing the same protocol can talk to each other, this network supports **true full-stack interconnection**.

This architecture makes it possible to create **private VLANs that span the planet**, with full L2 broadcast/multicast/unicast behavior — secured by modern cryptographic group schemes (like MLS).


> Any software capable of using a network interface — including browsers, office tools, games, or IoT services — can operate across the libp2p overlay transparently.

This means:
- Alice in Alaska can print a document from her word processor directly to a printer connected to Bob’s machine in Dubai — **as if they were on the same office network**, sharing broadcast ARP, DHCP, or mDNS traffic securely.
- Networked multiplayer games can work over encrypted P2P links with no central servers.
- Custom business or industrial systems can integrate globally, over a trustless, decentralized network.

The network provides **virtual Ethernet interfaces (tap devices)**, making it fully compatible with traditional networking stacks. This brings global physical separation and local network semantics into one consistent, secure abstraction.

---

### 🌍 Decentralized Routing, Proxying & VPN Infrastructure

Peers in the overlay can act as **exit nodes, proxies, or intermediate routers**, forming the basis of a **truly decentralized VPN infrastructure**.

- Gateways may allow outbound traffic to the global Internet, according to configurable policies or token-based access control.
- No centralized providers or trust assumptions are needed.
- All routing logic is handled peer-to-peer, and traffic is encrypted both on link and group levels.

This enables use cases such as:
- Bypassing censorship and surveillance.
- Creating secure hybrid networks across organizations.
- Enabling private access to internet services via community-controlled infrastructure.

---
