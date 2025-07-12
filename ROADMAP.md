# PeerTap Project Roadmap

## ✅ Phase 0 – Design Finalization (DONE)
- ✓ VLAN Forwarding Token (VFT) model finalized
- ✓ Uniform handling of unicast, broadcast, multicast
- ✓ Anonymous relay forwarding
- ✓ MVLAN discovery protocol defined
- ✓ TAP & TUN mode compatibility
- ✓ Final specification v0.2.0 published

---

## 💪 Phase 1 – MVP Implementation
**Goal:** Build a functional prototype for TAP-based nodes with encrypted libp2p trunk mesh.

### Core Components:
- [ ] libp2p trunk mesh with fallback transports (QUIC, TCP, WebSocket, etc.)
- [ ] VLAN creation via static VLAN-ID (BLAKE3)
- [ ] MLS group creation with TreeKEM
- [ ] L2 frame encryption with ECIES + AES-GCM
- [ ] VFT generation logic for all traffic types
- [ ] Stateless relay forwarding based on VFT
- [ ] ACL-based relay filtering and enforcement
- [ ] MVLAN presence/discovery messaging
- [ ] TAP interface setup with CryptoVLAN switching logic

### Developer Tools:
- [ ] CLI tool with JSON config
- [ ] Log system and debug trace mode
- [ ] Basic network topology print/debug info

---

## 📱 Phase 2 – TUN/Mobile Support
**Goal:** Enable PeerTap deployment on mobile and desktop platforms without TAP access.

### TUN Interface Adaptation:
- [ ] Generate MAC address for each TUN interface
- [ ] Emulate L2 semantics in user space
- [ ] Unified VFT derivation from virtual MAC
- [ ] Internal handling of broadcast/multicast for TUN

### Platform Integration:
- [ ] Android: VpnService tunnel mode
- [ ] iOS: NEPacketTunnelProvider
- [ ] Desktop fallback mode (TUN if no TAP support)

---

## ↺ Phase 3 – Fragmentation & Transport Robustness
**Goal:** Ensure compatibility with MTU limits and duplex flows.

- [ ] Fragmentation before encryption
- [ ] Reassembly logic on receiving peers
- [ ] Stateless relay forwarding of fragments
- [ ] Full duplex message flow with symmetric VFTs
- [ ] ICMP echo/ping handling (for debugging and path validation)
- [ ] TCP stream compatibility testing

---

## 🛡️ Phase 4 – Optimization & Scalability
**Goal:** Strengthen relay mesh stability under churn and scale.

- [ ] Peer latency and relay scoring metrics
- [ ] Multi-path relay redundancy and selection
- [ ] CRST: Crypto Rapid Spanning Tree evaluation
- [ ] Suppression of redundant broadcast/multicast floods
- [ ] Dynamic relay load adaptation

---

## 🧠 Phase 5 – Research & Future Extensions
**Goal:** Expand protocol capabilities and resilience.

- [ ] Graph-based multicast tree generation
- [ ] Anonymous NAT traversal enhancements
- [ ] Fuzz testing for relay behavior
- [ ] Security review and formal audit
- [ ] Experimental routing strategies (DHT/pubsub)
- [ ] Optional: relay incentive/disincentive models

---

## 🚀 Milestone – v1.0 Stable Release
- [ ] Binaries for major OS platforms
- [ ] PeerTap protocol version negotiation
- [ ] Automated mesh self-healing under node churn
- [ ] Full CI/CD and reproducible builds
- [ ] Minimal UI for VLAN selection and interface status

---

## Long-Term Direction
- [ ] Open mesh switching fabric on global scale
- [ ] Native L3-over-L2 encapsulation bridge
- [ ] Interop modules for legacy network compatibility
- [ ] Research-driven evolution of anti-censorship overlay

