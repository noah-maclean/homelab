---
title: Pi-hole (Raspberry Pi)
status: running
type: container
author: noah, hermes
tags:
  - container
  - networking
  - dns
  - adblocking
  - tailscale
---

# Pi-hole (Raspberry Pi)

Runs on a Raspberry Pi 3B (hostname: `rasppi`) on the LAN at `192.168.1.10`.

## Tailscale Setup — Ad-blocking Everywhere

Pi-hole is joined to the tailnet as `rasppi` (`100.75.28.2`). DNS queries from any Tailscale-connected device are routed to the Pi-hole for ad-blocking, even when off-network.

### Config (Tailscale admin console)

- **Global nameservers**: `100.75.28.2`
- **"Use with exit node"**: ✅ ticked (so ad-blocking persists when routing through an exit node)
- **Override local DNS**: ✅ ticked

### Upstream DNS

Upstream DNS providers set in Pi-hole web UI:

| Provider | Notes |
|---|---|
| **Quad9 (filtered, DNSSEC)** — `9.9.9.9` | Primary. Blocks malware + DNSSEC validation |
| **Cloudflare (DNSSEC)** — `1.1.1.2` | Secondary. DNSSEC validated, malware-blocking variant |

Previously used Google DNS (`8.8.8.8`). Switched to Quad9 + Cloudflare for better privacy and malware filtering.

### Config (Pi-hole web UI, v6.6.1)

- Under **Settings → All Settings → DNS**, set **Interface listening behavior** to **Permit all origins**.
- This allows the dnsmasq backend to accept queries from Tailscale IPs (`100.x.x.x`). The default "Allow only local requests" setting silently drops non-local queries with `ignoring query from non-local network` warnings.

### MagicDNS

Tailscale's MagicDNS relay (`100.100.100.100`) forwards queries to the Pi-hole. This means client devices can use MagicDNS as their resolver and still get Pi-hole filtering — the relay acts as a proxy.

### Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| DNS queries fail over Tailscale with `connection timed out` from Mac, but Pi-hole works locally | Pi-hole dnsmasq blocking non-local networks | Set **Permit all origins** in Pi-hole settings |
| `dig` TCP, UDP queries time out from client | Pi-hole dnsmasq rejecting Tailscale subnet | Check `pihole.log` for `ignoring query from non-local network` warnings |
| Direct queries work (`dig @100.75.28.2`) from Pi itself | Client-to-Pi routing issue or Pi-hole host config | Test with `tailscale ping --c 3 <client_ip>` to confirm Tailscale connectivity |

### Key addresses

| Address | Purpose |
|---|---|
| `192.168.1.10` | LAN IP (Raspberry Pi 3B) |
| `100.75.28.2` | Tailscale IP (rasppi) |
| `100.100.100.100` | Tailscale MagicDNS relay |

## Related

- [[tailscale]]
- [[todo_containers]]
- [[goals]]

### macOS Tailscale quirk

The Tailscale network extension on macOS (`systemextensionsctl`) persists across reboots and intercepts traffic to any known tailnet peer, even when the Tailscale app is quit. This means:

- `ssh user@192.168.1.23` (LAN IP) may hang if Hermes is a tailnet member
- `ssh user@100.94.37.116` (Tailscale IP) always works when Tailscale is on

If you need LAN IP access without Tailscale: `sudo systemextensionsctl deactivate W5364U7YZB io.tailscale.ipn.macsys.network-extension`
