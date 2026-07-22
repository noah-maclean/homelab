---
title: Tailscale LXC
status: running
type: container
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
