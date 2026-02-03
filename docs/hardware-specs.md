# Hardware Specifications & Resource Allocation

## Physical Hardware
*   **Compute:** Intel Core i5-7500 (4 Cores / 4 Threads).
*   **Memory:** 48 GB DDR4 2666 MHz (Non-ECC).
*   **Network:** Intel i350 Dual-Port Gigabit NIC + Onboard Intel NIC.
*   **Storage Tiering:**
    *   **Boot:** 256 GB NVMe (Proxmox Host).
    *   **VM Storage:** 1 TB NVMe (ZFS Encrypted Pool).
    *   **Mass Storage:** 3x 2TB HDD (ZFS RAIDZ1 passed to TrueNAS).
    *   **Backup:** External USB HDD (PASSTHROUGH to PBS).

## Resource Overcommitment Strategy
I utilize a widespread overprovisioning strategy (approx 3:1 vCPU ratio), managing 4 physical cores against various workloads.

High-resource VMs (Windows/Fedora) are treated as mutually exclusive to prevent CPU contention.

| VM | vCPUs | CPU Units | RAM | Function |
| :--- | :--- | :--- | :--- | :--- |
| **Proxmox (Host)** | - | - | 2 GB | Hypervisor & ZFS ARC (Limited to 1GB) |
| **OPNsense** | 2 | 4096 | 4 GB | Edge Firewall & Routing |
| **TrueNAS** | 2 | 2048 | 10 GB | Storage (ZFS) |
| **Debian** | 2 | 2048 | 12 GB | Docker Host (Immich, Nextcloud) |
| **PBS (Backup)** | 2 | 1024 | 3 GB | Proxmox Backup Server |
| **Windows 11*** | 3 | 6144 | 12 GB | Vivado/Engineering Tools |
| **Fedora*** | 3 | 6144 | 15 GB | Linux Development Environment |

*\*Windows 11 and Fedora are never run simultaneously.*
