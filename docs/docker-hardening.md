# Docker Security & Network Hardening

To ensure container security within the Proxmox environment, I implement strict isolation policies using non-root users and custom iptables rules.

## 1. Runtime Security
Most containers are configured to run with minimal privileges to prevent escalation attacks:

*   **Non-Root Execution:** Containers run as user `1000:1000` where possible.
*   **Capability Dropping:** Unneeded Linux capabilities are dropped using `cap_drop`.
*   **Security Options:** `no-new-privileges:true` is enforced to prevent processes from gaining more privileges than they started with.
*   **Read-Only Mounts:** Bind mounts (e.g., config files, libraries) are mounted as read-only to prevent container compromise from affecting the host filesystem.

## 2. Network Isolation (LAN vs. WAN)
To mitigate the risk of supply chain attacks or compromised containers, I segment Docker networks into "LAN Only" and "WAN Access" tiers.

### Network Segmentation Strategy
*   **WAN Network (`172.20.x.x`):** Only for containers requiring external API access (e.g., Nginx Proxy Manager, OpenWebUI).
*   **LAN Network (`172.19.x.x`):** For internal services (Immich, Nextcloud, Database). These are strictly blocked from the internet.
*   **IPv6 Disabled:** To prevent routing circumvention.

### Custom IPTables Firewall Rules
I utilize the `DOCKER-USER` chain to enforce these rules at the host level:

```bash
# Flush existing rules
sudo iptables -F DOCKER-USER

# 1. Allow WAN for specific authorized containers (NPM)
sudo iptables -A DOCKER-USER -s 172.20.0.0/16 -j RETURN 

# 2. Allow Internal Traffic (LAN/VPN/Loopback) for secure containers
sudo iptables -A DOCKER-USER -s 172.19.0.0/16 -d 10.10.10.0/24   -j ACCEPT   # WireGuard VPN
sudo iptables -A DOCKER-USER -s 172.19.0.0/16 -d 192.168.0.0/16  -j ACCEPT   # Home LAN
sudo iptables -A DOCKER-USER -s 172.19.0.0/16 -d 127.0.0.0/8     -j ACCEPT   # Loopback

# 3. DROP everything else (Prevents WAN access for internal containers)
sudo iptables -A DOCKER-USER -s 172.19.0.0/16 -j REJECT

# 4. Persistence
# Rules are saved via netfilter-persistent to survive reboots.
