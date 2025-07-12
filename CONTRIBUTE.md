# Contributing to PeerTap

PeerTap is in early-stage development. Contributions are welcome.

## How to Contribute

- Fix typos or improve docs
- Suggest ideas via issues
- Prototype features in alignment with the spec
- Report bugs, inconsistencies, or missing parts

## Workflow

1. Fork → Branch → PR
2. Keep commits small and clear
3. Link to spec when relevant

## Stack

- Core (repo: `peertap`): Rust (`libp2p`, `blake3`, `mls-rs`, `tun`, `wintun`, `utun`)
- CLI: Rust (`clap`, `dialoguer`, `indicatif`)
- GUI (optional): Tauri + Vanilla JS + SCSS modules
- Mobile:
  - Android: separate repo (`peertap-android`), Kotlin + Rust via FFI
  - iOS: separate repo (`peertap-ios`), Swift + Rust via FFI

## Security

Report vulnerabilities privately: `security@peertap.net`

## Communication

Open issues for questions, ideas, or feedback.

---

Thank you for helping improve PeerTap.

