# Homelab

A private homelab running on Proxmox VE with a Raspberry Pi 3B for lightweight services. This repository is the knowledge vault — configuration notes, logs, and to-do lists for everything in the stack.

## Hardware

| Device           | Specs                 | Role                                |
| ---------------- | --------------------- | ----------------------------------- |
| Proxmox host     | Ryzen 3 3200G, 24 GB  | VM/LXC hypervisor                   |
| Raspberry Pi 3B  | 1 GB RAM              | Pi-hole DNS, lightweight services   |

### Storage

| Disk          | Size   | Mount         | Purpose                                                    |
| ------------- | ------ | ------------- | ---------------------------------------------------------- |
| Kingston SSD  | 240 GB | `local-lvm`   | Proxmox OS + VM/container disks                           |
| Seagate HDD   | 1 TB   | `media-storage0` | Bulk media (mergerfs → `/mnt/storage`)                   |
| Toshiba HDD   | 500 GB | `app-data`     | Application data (mergerfs → `/mnt/storage`)              |

Additional HDDs auto-merge into `/mnt/storage` via mergerfs.

## Network

- **Tailscale subnet router** — advertises `192.168.1.0/24` so all Tailscale devices can reach LAN services
- **Pi-hole** (Raspberry Pi 3B) — DNS-level ad blocking for the whole LAN

## Services

### Running

| Service                          | Type          | Purpose                                      |
| -------------------------------- | ------------- | -------------------------------------------- |
| [Hermes Agent](vault/containers/hermes_agent.md) | LXC           | AI assistant with tool-use and memory        |
| [Glance Dashboard](vault/containers/glance.md)   | LXC           | Self-hosted dashboard (192.168.1.22)         |
| [Tailscale](vault/containers/tailscale.md)        | LXC           | Mesh VPN + subnet routing (192.168.1.21)    |
| Pi-hole                          | Raspberry Pi  | DNS ad blocking                              |
| Home Assistant                   | LXC           | Home automation                              |
| Crafty Controller                | LXC           | Minecraft/game server manager                |

### Planned

- Immich — photo backup and management
- Jellyfin + \*Arr stack — media server with Sonarr, Radarr, Prowlarr, etc.
- Uptime Kuma — uptime monitoring
- Nginx Proxy Manager or Caddy — reverse proxy

### Maybe

- Nextcloud or similar — file sharing / NAS OS
- Vaultwarden — self-hosted password manager

## Vault Structure

```text
vault/
├── AGENTS.md          # Guidelines for AI agents editing this repo
├── goals.md           # Long-term priorities
├── containers/        # Service configuration notes
├── logs/              # Daily changelogs (YYYY-MM-DD.md)
└── todo/              # Task lists by topic
```

Every note has YAML front matter with `title`, `type`, `author`, and `tags` for easy querying in Obsidian (Dataview, graph view, tag pane).

## Usage

This is a knowledge repository, not an application. Browse the notes for setup instructions, IPs, and configuration notes. Copy commands from the vault rather than typing from memory — they've been tested and corrected.
