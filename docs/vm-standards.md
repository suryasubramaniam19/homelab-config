# VM Configuration Standards & Passthrough Strategy

To optimize performance on the HP EliteDesk hardware, I utilize specific virtualization configurations for different workloads, specifically focusing on PCI passthrough and GVT-g graphics virtualization.

## 1. General Hypervisor Settings
*   **BIOS Type:** SeaBIOS (Default) for stability; OVMF (UEFI) used strictly for Windows/Fedora.
*   **Storage Policy:** All VM boot disks reside on the `Encrypted-VMs` ZFS pool.
*   **Caching:** `SSD Emulation` and `Discard` enabled to support TRIMMing on the underlying SSDs.
*   **Memory Ballooning:** Explicitly **DISABLED** for OPNsense and TrueNAS to ensure stable memory address allocation for PCI devices and ZFS ARC.

## 2. TrueNAS Scale (Storage Controller Passthrough)
To grant TrueNAS direct access to the drives for ZFS safety:
1.  **IOMMU Grouping:** Verified isolation of the SATA controller (`00:1f.2`) to ensure no interrupt conflicts.
2.  **VFIO Configuration:**
    *   Kernel Parameters: `intel_iommu=on iommu=pt`
    *   Modules: `vfio`, `vfio_pci`, `vfio_virqfd` loaded via `/etc/modules`.
    *   Driver Blacklisting: `ahci` drivers blacklisted on the generic host to prevent Proxmox from claiming the drives before the VM starts.

## 3. Graphics Virtualization (Intel GVT-g)
Rather than passing the entire GPU to a single VM, I utilize **Intel GVT-g** to split the iGPU into multiple virtual instances (vGPUs).
*   **Configuration:** `i915.enable_gvt=1` enabled in GRUB.
*   **MDEV Profile:** `i915-GVTg_V5_8` (allows 2 simultaneous VMs to share the GPU).
*   **Allocation:**
    *   **Slice 1:** Debian 13 (Jellyfin/Immich Hardware Transcoding).
    *   **Slice 2:** Windows 11 (GUI acceleration).
*   *Benefit:* Allows for hardware transcoding in Docker containers while simultaneously maintaining a console output on the Proxmox host.
