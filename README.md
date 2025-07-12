# PeerTap

PeerTap is a decentralized, censorship-resistant Layer 2 mesh network built on libp2p. It connects nodes into encrypted Ethernet-level broadcast domains called CryptoVLANs. Each VLAN is cryptographically isolated, providing anonymous, resilient data-link connectivity.

## Key Features

- Cryptographic VLANs (CryptoVLAN) based on BLAKE3 and TreeKEM (MLS)
- Raw Ethernet tunneling over libp2p encrypted trunks
- Transparent TAP/TUN interfaces for L2/L3 support
- VLAN-based switching and VFT-based forwarding (anonymized unicast/multicast/broadcast)
- Full support for loop prevention, relay coordination, and spanning-tree topology
- Optional GUI (Tauri) and CLI (Rust)
- Cross-platform (Linux, Windows, macOS, Android, iOS)

## Repositories

- Core implementation: [`peertap`](https://github.com/libercoder/peertap)
- Android wrapper: [`peertap-android`](https://github.com/libercoder/peertap-android)
- iOS wrapper: [`peertap-ios`](https://github.com/libercoder/peertap-ios)

## Documentation

- [Specification](./SPECIFICATION.md)
- [Roadmap](./ROADMAP.md)
- [Contribution Guide](./CONTRIBUTE.md)

## Status

PeerTap is in active research and prototyping stage.

---

For questions or contributions, see [CONTRIBUTE.md](./CONTRIBUTE.md).

---

> PeerTap aims to be the Ethernet of anonymous, censorship-resistant overlay networks.

