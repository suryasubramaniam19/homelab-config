# Backup & Disaster Recovery Strategy

My data protection strategy follows a modified **3-2-1 methodology**, ensuring data resiliency against hardware failure, corruption, and ransomware.

## 1. Virtual Machine Backups (Proxmox Backup Server)
All VMs (Debian, Windows, OPNsense) are backed up to a dedicated **Proxmox Backup Server (PBS)** instance.

*   **Target:** Encrypted Datastore on a passed-through 1 TB USB HDD.
*   **Encryption:** Client-side encryption (keys stored in Bitwarden).
*   **Schedule:** Daily incremental backups.
*   **Retention Policy:**
    *   Keep last 9 backups
    *   Keep 7 daily, 4 weekly, 6 monthly, 1 yearly.
*   **Secondary Copy:** A "Disaster Only" copy is sent via NFS to TrueNAS (non-incremental) for immediate access to raw VM disk files if PBS fails.
*   *Note:* The PBS VM itself is excluded from self-backup to prevent "freeze/thaw" loops.

## 2. NAS Data Protection (TrueNAS/ZFS)
Data integrity is maintained through ZFS checksumming and automated tasks.

### Local Snapshots (Rollback)
*   **Frequency:** Hourly/Daily recursive snapshots on all datasets.
*   **Retention:** 1 week.
*   **Exclusions:** Volatile data (e.g., `Devices/Proxmox`) is excluded to preserve storage space.

### Offsite Cloud Backup (Backblaze B2)
To protect against site-level disasters (fire, theft), data is pushed to the cloud.
*   **Tool:** Cloud Sync Tasks (Rclone).
*   **Destination:** Backblaze B2 Bucket (`Gringotts-<ID>`).
*   **Encryption:** File name and content encryption enabled before upload.
*   **Ransomware Protection:** Lifecycle rules retain file versions for 30 days, allowing recovery from accidental deletion or malicious encryption.

## 3. Data Integrity Verification
I automate hardware health checks to detect "bit rot" or drive failure early:
*   **SMART Tests:** Short tests daily; Long tests weekly (Sundays).
*   **ZFS Scrub:** Weekly data scrub (Sundays) to verify checksums and repair corrupted blocks.
