# Remote homelab access: pfSense + WireGuard + SSH

## Goal
Use a laptop on an external network to securely administer Linux systems at home, without exposing the servers' SSH ports to the public internet.

## Design
```mermaid
flowchart LR
    A[Ubuntu laptop] -->|Wi-Fi or hotspot| B[Internet]
    B -->|WireGuard UDP 51820| C[pfSense VPN gateway]
    C --> D[Homelab private LAN]
    D --> E[Linux server via SSH]
```

A single WireGuard interface on pfSense can accept multiple **peers** (clients). Each client gets its own key pair and distinct tunnel address. A tunnel's description is a label, not a binding to one laptop. Use distinct peers for a Windows workstation and an Ubuntu workstation; do not share client private keys.

## Example addressing (documentation only)
| Purpose | Example |
| --- | --- |
| Home VPN DNS | `vpn.example.net` |
| pfSense WireGuard interface | `10.60.0.1/24` |
| Ubuntu client | `10.60.0.3/32` |
| Home LAN | `10.50.0.0/24` |
| SSH target | `10.50.0.53` |
| Server listen port | UDP `51820` |

These are illustrative values, **not deployment instructions for a live network**. Confirm permitted peer addresses, firewall rules, and routing before assigning them.

## Configuration concepts
1. Install WireGuard tools and use NetworkManager to create a WireGuard profile on Ubuntu.
2. Generate a client key pair with restrictive permissions. Keep the private key on the client; register only its public key in the pfSense peer configuration.
3. Set the client interface address to its dedicated VPN address (`/32`).
4. Set the pfSense public key, DDNS hostname plus UDP port, and peer Allowed IPs to the desired VPN and home-LAN prefixes. Leave the client interface listen port automatic.
5. For **split tunneling**, route only the homelab prefixes over WireGuard; let ordinary internet traffic use the local Wi-Fi default route. In NetworkManager, verify `wireguard.peer-routes=yes` and that the VPN does not become the default route.
6. Use an SSH key and a host entry in `~/.ssh/config` to reach the Linux server. Verify the SSH server host key fingerprint out-of-band before accepting a first connection.
7. Enable NetworkManager autoconnect and test it **after a reboot**.
8. Configure a DDNS updater at the home gateway. A client whose saved endpoint uses a hostname will resolve it when activated; WireGuard does not continuously re-resolve it. A periodically run, idempotent endpoint refresh script can improve recovery after home public-IP changes.

## The routing issue observed
The client could handshake with pfSense and ping the home gateway on the local network, yet `ip route get <home-lan-address>` chose Wi-Fi/Ethernet rather than `wg0`. Inspection showed `wireguard.peer-routes: no`.

After enabling NetworkManager peer routes and preserving the ordinary internet default route, `ip route get <home-lan-address>` selected `dev wg0` with the client VPN source address. **Allowed IPs alone do not prove the Linux route was installed.**

Helpful diagnostics:
```bash
nmcli connection show --active
sudo wg show
ip -4 route show table all
ip route get 10.50.0.53
systemctl list-timers --all
journalctl -u wireguard-ddns-refresh.service --no-pager
```

## Verification evidence
- Recent handshake and traffic counters in `wg show`.
- Route selection through `wg0`.
- ICMP to the pfSense LAN interface through VPN routing, subject to firewall rules.
- Public-key-only SSH to the Linux server, with the server observing the client's tunnel address.
- Connection profile auto-activated after a reboot.
- DDNS timer remained enabled after reboot.
- A controlled endpoint mismatch was corrected by the local refresh script, followed by a successful SSH reconnection.

**Not yet proven:** recovery during a real ISP public-address change, including upstream DNS propagation and firewall/ISP reachability. External Wi-Fi/hotspot testing must also be explicitly documented as external, rather than inferred from route output alone.

## Operations and security
- Keep the original working peers intact when onboarding another device.
- WireGuard's `PersistentKeepalive=25` helps clients behind NAT; it is not DNS failover.
- A successful handshake is not proof that firewall policy permits SSH to all internal machines.
- Don't publish real public endpoints, key files, live rule sets, screenshots exposing identity/network data, or complete private configurations.
- On an actual public IP change, confirm pfSense DDNS status, hostname resolution from the client, the active WireGuard endpoint, and reachability in that order.

## Learning outcomes
Linux permissions/umask, NetworkManager profiles, IPv4 routing and split tunnels, SSH public-key authentication, DNS, Bash scripting, systemd oneshot services/timers, journal inspection, and troubleshooting methodology.
