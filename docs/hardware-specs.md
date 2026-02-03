# Hardware Specifications & Resource Allocation

## Physical Hardware
*   **Compute:** Intel Core i5-7500 (4 Cores / 4 Threads).
*   **Memory:** 48 GB DDR4 2666 MHz (Non-ECC).
*   **Network:** Intel i350 Dual-Port Gigabit NIC + Onboard Intel NIC.
*   **Storage Tiering:**
    *   **Boot:** 256 GB NVMe.
    *   **VM Storage:** 1 TB NVMe (ZFS).
    *   **Mass Storage:** 3x 2TB HDD (ZFS RAIDZ1 passed to TrueNAS).

## Resource Overcommitment Strategy
I utilize a documented overprovisioning strategy (approx 2.5:1 vCPU ratio) managing 4 physical cores against ~10 allocated vCPUs.

| VM | vCPUs | RAM Assignment | Primary Function |
| :--- | :--- | :--- | :--- |
| **Proxmox (Host)** | Reserved | 1 GB (ARC) + 1 GB (System) | Hypervisor & ZFS Management |
| **OPNsense** | 2 | 4 GB (Fixed) | Edge Firewall / Routing |
| **TrueNAS** | 2 | 10 GB (Fixed) | Storage & SMB Shares |
| **Debian** | 2 | 12 GB | Docker Host (Immich, Nextcloud) |
| **Windows 11** | 3 | 12 GB | Vivado/Engineering Tools |

*Note: Windows and Fedora instances remain powered off when not in active use to free up CPU cycles for the container stack.*
