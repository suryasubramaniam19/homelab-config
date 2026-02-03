# Docker Service Catalog

Below is the documentation for the containerized services running on the Debian host (`192.168.1.4`) and TrueNAS (`192.168.1.3`).

## Infrastructure & Monitoring
| Service | Purpose | Configuration Highlights |
| :--- | :--- | :--- |
| **Nginx Proxy Manager** | Reverse Proxy | Manages wildcard SSL certificates (`*.floonetwork.xyz`) via Let's Encrypt and Cloudflare DNS challenge. Points `vpn` subdomain to dynamic WAN IP. |
| **Portainer** | Container Management | Used primarily for log analysis and rapid troubleshooting. |
| **Uptime Kuma** | Monitoring | Monitors VM heartbeats (ping) and Service HTTP status. Integrated with **Ntfy** for push alerts to mobile. |
| **Ntfy** | Notifications | Self-hosted push notification server. configured largely to bypass email alerts. *Constraint:* Payload limited to 4096 bytes (logs truncated). |

## Applications
### 📸 Immich (Photo Management)
*   **Function:** Self-hosted replacement for iCloud Photos.
*   **Hardware Acceleration:** Utilizes split iGPU passthrough (Intel GVT-g) for transcoding.
*   **Customization:** Implemented `.immichignore` patterns to exclude system directories from library scans.

### ☁️ Nextcloud (Productivity)
*   **Function:** CalDAV/CardDAV server for syncing calendars between macOS, iPadOS, and Android (via DAVx5).
*   **Setup:** Custom Docker Compose (Non-AIO) for granular control over network bridges.

### 🧠 Open WebUI (LLM Interface)
*   **Function:** Local interface for interacting with LLMs (OpenRouter/OpenAI).

### 🏠 Homarr (Dashboard)
*   **Function:** Central landing page for the "Floo Network" homelab.
*   **Integrations:** API connections to Proxmox, TrueNAS, and OPNsense for real-time status widgets.

### 🔄 Syncthing
*   **Host:** TrueNAS (Scale/Chart).
*   **Purpose:** Continuous low-latency file synchronization between MacBook, NAS, and Phone.

### 🎮 Minecraft
*   **Image:** `itzg/docker-minecraft-server` (Java 21).
*   **Security:** Container is stopped when not in use to minimize WAN attack surface.
