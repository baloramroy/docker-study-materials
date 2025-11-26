# Docker Builtin Network Drivers Explanation

## Docker built in drivers

Docker comes with **built-in** (or native) drivers:

- **bridge** — Single-host local networks
- **host** — Container uses the host’s network stack directly (no isolation)
- **overlay** — Multi-host networks for Swarm (uses VXLAN)
- **macvlan** — Give containers real LAN IPs directly
- **ipvlan** — Similar to macvlan but more efficient; assigns IPs without creating new MACs
- **none** — Disable all networking for the container

> **Note:** You can also use third-party drivers for more advanced setups.
