---
title: Tailscale LXC
status: running
type: container
author: noah
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

## Pitfalls

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
