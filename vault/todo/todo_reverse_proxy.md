---
title: Reverse Proxy Setup TODO
created: 2026-07-22
type: todo
author: hermes
tags:
  - todo
  - networking
  - reverse-proxy
  - caddy
  - tailscale
  - dns
---

# Reverse Proxy Setup

Goal: nice domain names instead of IPs. Some services public (Glance, Jellyfin), most stay on Tailscale.

## Two options

### Option A: Tailscale Funnel (no router config, starts now)

Jellyfin video goes through Tailscale relay — fine to test, slower for 4K.

```shell
# Enable funnel
sudo tailscale set --funnel

# Expose services publicly at https://<port>.your-tailnet.ts.net
sudo tailscale funnel --bg 8080   # Glance
sudo tailscale funnel --bg 8096   # Jellyfin
```

**Pros:** Zero port forwarding, works now, built-in Tailscale auth.
**Cons:** *.ts.net URLs, video relayed through Tailscale.

---

### Option B: Caddy + DuckDNS + Port Forwarding (recommended for Jellyfin)

Full control, direct traffic, pretty subdomains. Swap DuckDNS for a real domain later trivially.

#### 1. Get a DuckDNS domain

1. Go to [duckdns.org](https://duckdns.org) — sign in with GitHub/Twitter/Google
2. Pick a subdomain, e.g. `noah-homelab.duckdns.org`
3. Note your **token**

#### 2. Install Caddy

```shell
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update && sudo apt install caddy
```

#### 3. DuckDNS auto-updater (keeps IP current if it changes)

```shell
cat > /home/hermes/duckdns-update.sh << 'EOF'
#!/bin/bash
echo url="https://www.duckdns.org/update?domains=noah-homelab&token=TOKEN_HERE&ip=" | curl -k -o /home/hermes/duckdns.log -s -
EOF
chmod +x /home/hermes/duckdns-update.sh

# Add to crontab (every 5 min)
(crontab -l 2>/dev/null; echo "*/5 * * * * /home/hermes/duckdns-update.sh") | crontab -
```

#### 4. Write the Caddyfile

Replace ports with whatever your services actually run on.

```caddy
# /etc/caddy/Caddyfile

glance.noah-homelab.duckdns.org {
    reverse_proxy localhost:8080
}

jellyfin.noah-homelab.duckdns.org {
    reverse_proxy localhost:8096
}

ha.noah-homelab.duckdns.org {
    reverse_proxy localhost:8123
}
```

```shell
sudo systemctl reload caddy
```

Caddy auto-fetches Let's Encrypt certs — no certbot needed.

#### 5. Port forwarding on router (if you can)

1. Find your router IP: `ip route | grep default`
2. Log into the router admin page in a browser
3. Look for: "Port Forwarding" / "Virtual Server" / "NAT"
4. Add two rules:

| External | Internal IP (your LXC) | Internal Port | Protocol |
|----------|----------------------|---------------|----------|
| 80       | LXC IP (`hostname -I`) | 80 | TCP |
| 443      | LXC IP | 443 | TCP |

#### 6. Test

```shell
curl -I https://glance.noah-homelab.duckdns.org
```

Expect `200 OK` with a valid cert.

### Recommended path

1. **Start with Tailscale Funnel** — 5 min test drive
2. **Switch to Caddy + DuckDNS** if Jellyfin streaming feels slow or you want cleaner URLs
3. **Buy a real domain later** — just point it at the same Caddy config

### If port forwarding doesn't work

- Some UK ISPs block 80/443 — check yours
- Your LXC's firewall (ufw) or Proxmox firewall may need port rules
- Try Tailscale Funnel instead — no ports needed
- Or use Cloudflare Tunnel (`cloudflared`) — no open ports, but Jellyfin video violates their ToS on the free plan

## Related

- [[containers/tailscale]]
- [[containers/glance]]
- [[todo/todo_jellyfin_arr]]
- [[todo/todo_containers]]
- [[goals]]
