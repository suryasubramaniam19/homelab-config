# Network Segmentation & Firewall Rules

My network is managed via **OPNsense** with strict traffic shaping rules to ensure IoT devices and backup servers operate with least-privilege access.

## 1. OPNsense Configuration
*   **RFC1918 Aliases:** Created aliases for private IP ranges to easily manage internal vs. external traffic rules.
*   **IoT Isolation:**
    *   Created `WAN_Blocked_Hosts` alias for smart bulbs and Wi-Fi AP administrative interfaces.
    *   **Rule:** Block `WAN_Blocked_Hosts` -> Destination `!RFC1918` (Prevent calling home to cloud servers).
    *   Verified via firewall live view logs that IoT devices cannot reach vendor cloud APIs.

## 2. TrueNAS Firewalling (Data Protection)
Given TrueNAS hosts critical data, I implement a "Deny All" strategy for WAN, with scheduled exceptions for offsite backups.

### Automated Backup Schedule
To protect the NAS from constant internet exposure while allowing for "daily automated cloud backups" (as claimed in resume):
1.  **Schedule:** Firewall rules open ONLY on Sunday 00:00 - 06:00.
2.  **Logic:**
    *   **Rule 1:** Allow NAS -> LAN (Always active).
    *   **Rule 2:** Allow NAS -> WAN (Active ONLY during backup window via Schedule object).
    *   **Rule 3:** Block NAS -> WAN (Default state).

## 3. IPv6 Hardening
Since my internal infrastructure relies heavily on IPv4 NAT and specific alias blocking, IPv6 is disabled network-wide on OPNsense and DHCPv6 is deactivated to prevent traffic leakage or firewall bypass.
