# SSH & System Access Hardening

To secure the hypervisor (Proxmox) and VMs (Debian, TrueNAS), I rely entirely on cryptographic key authentication rather than passwords.

## 1. Authentication Strategy
*   **Algorithm:** Moved away from RSA to **Ed25519** keys for better security and performance.
*   **Password Disabling:** SSH password authentication is explicitly disabled in `/etc/ssh/sshd_config` on all hosts.
    ```bash
    PasswordAuthentication no
    PubkeyAuthentication yes
    PermitRootLogin no # Enforced on Debian hosts
    ```
*   **Root Locking:** On Debian VMs, the root account password is locked (`passwd -l root`) to force usage of `sudo` escalation, creating an audit trail.

## 2. Client Side Workflow
I manage multiple hosts using a structured `~/.ssh/config` file to prevent identity file confusion and streamline access:

```text
Host Proxmox
   HostName 192.168.1.x
   User root
   IdentityFile ~/.ssh/id_ed25519_homelab

Host Debian-Docker
   HostName 192.168.1.y
   User username
   IdentityFile ~/.ssh/id_ed25519_homelab
