---
title: Tailscale LXC
status: running
type: container
author: noah, hermes
tags:
  - container
  - networking
  - vpn
---

# Tailscale LXC

## Setup

- Debian LXC (default settings)
       - 1 core
       - 512 MB RAM and swap
       - 8GB HDD (could be lowered)
- Resources -> Add -> Device Passthrough -> `/dev/net/tun`
- IP: 192.169.1.21

## Post Creation

- install `curl`
- install Tailscale (`curl -fsSL https://tailscale.com/install.sh | sh`)
- start Tailscale (`tailscale up --ssh)
- install `sudo` so commands can just be copied and pasted
- advertise subnet routes:

```shell
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf


sudo tailscale set --advertise-routes=192.168.1.0/24
```

- approve route in admin console
- disable expiry

- now, each container can be accessed using their local IP (192.168.1.xx)

## Subnet Routing — How It Works

Two separate Tailscale features that work together:

| Feature | Used by | What it does |
|---|---|---|
| `--advertise-routes=192.168.1.0/24` | **Tailscale LXC** (the subnet router) | "I can be the gateway to the `192.168.1.x` LAN — route traffic for those IPs through me." |
| `--accept-routes` | **Remote devices** (Mac, phone, etc.) | "If another node advertises routes, I'll use them to reach those networks." |

**The flow when you're away from home:**

```text
   Your Mac (coffee shop)          Tailscale LXC (at home)            Hermes (at home)
   --accept-routes ✅              --advertise-routes ✅              --accept-routes ❌
         │                                  │                               │
         │  I want to reach 192.168.1.23    │                               │
         │──────────────────────────────►   │                               │
         │                                  │── forwards to 192.168.1.23──►│
         │◄─── response via tunnel ─────────│◄──────────────────────────────│
```

- **Your Mac** has `--accept-routes` → it knows to send `192.168.1.x` traffic through the Tailscale LXC subnet router. This is correct.
- **Tailscale LXC** has `--advertise-routes` → it tells the tailnet "I can reach the LAN". This is correct.
- **Other nodes on the LAN** (Hermes, Pi, etc.) **should NOT** have `--accept-routes` — they don't need to route through anyone else to reach their own LAN. If enabled, the `192.168.1.0/24` route gets added to their table, and LAN responses go through `tailscale0` instead of `eth0`, breaking local access.

### `--accept-routes` breaks LAN access to tailnet members

Other containers on the LAN with Tailscale installed **must not** have `--accept-routes` enabled. If they do, they'll accept the `192.168.1.0/24` subnet route into Tailscale's table 52, routing LAN responses through `tailscale0` instead of `eth0`. This makes them unreachable via their LAN IP from other containers.

- **Symptoms**: SSH/pings to the container's LAN IP hang from other LAN devices, but work via Tailscale IP
- **Fix**: `sudo tailscale set --accept-routes=false` on the affected node
- **The Tailscale LXC** advertises the route; **no other device** should accept it

## Related

- [[todo_tailscale]]
- [[todo_reverse_proxy]]
- [[glance]]
- [[hermes_agent]]
- [[goals]]
- [[pihole]]
