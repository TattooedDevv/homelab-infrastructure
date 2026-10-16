# Homelab Infrastructure

Engineering notes, reproducible experiments, architecture decisions, and troubleshooting from a self-hosted homelab.

## Documentation principles
- Build in dependency order, verify each change, and record what changed.
- Explain commands and trade-offs rather than presenting unexplained command dumps.
- Publish **sanitized examples only**. Replace real endpoints, keys, hostnames, addresses, usernames, hardware identifiers, and screenshots as appropriate.
- Keep operational inventory, precise network topology, recovery instructions, and live configurations in a separate private repository.
- Never commit VPN private keys, SSH private keys, preshared keys, passwords, authentication tokens, or backups containing those secrets.

## Projects
| Area | Project | Status |
| --- | --- | --- |
| Networking | [Remote homelab access with pfSense, WireGuard, and SSH](docs/networking/wireguard-remote-access/README.md) | Implemented and verified on Ubuntu client; ongoing reliability checks |

## Learning workflow
For each infrastructure milestone: identify applicable certification objectives, consult official objectives, implement and test the feature, capture evidence, document problems/fixes, and write a short retrospective. Current Linux/networking exercises overlap with RHCSA and LFCS skills, but are not substitutes for full exam preparation.

## Repository scope
This repository contains learning-oriented public documentation and portable examples. Machine-specific procedures and privileged access information belong in the private operational runbooks.
