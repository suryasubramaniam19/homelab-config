# Emergency Out-of-Band Access

In the event of an OPNsense firewall failure or network configuration error, I have configured a physical "Backdoor" interface to access the Proxmox console.

## Interface Configuration (`vmbr2`)
*   **Physical Port:** `enp3s0f1` (Dedicated unused ethernet port on the Intel i350 card).
*   **Network Type:** Linux Bridge (Isolated).
*   **Static IP:** `172.16.99.1/24`.
    *   *Note:* This subnet is deliberately distinct from LAN (`192.168.1.x`) and WireGuard (`10.10.10.x`) ranges to prevent routing conflicts.
*   **Gateway:** None (Local access only).

## Access Protocol
1.  Connect client laptop via ethernet to Port 3.
2.  Manually assign client IP to `172.16.99.2`.
3.  SSH directly to `root@172.16.99.1`.

This ensures that even if the virtualized router (OPNsense) crashes or VLAN tagging breaks, I retain root shell access to the hypervisor to perform repairs.
