# WireGuard VPN Implementation (OPNsense)

To provide secure remote access to the homelab without exposing critical management ports (SSH/Proxmox GUI) to the public internet, I implemented a **WireGuard** server directly within the OPNsense router.

This implementation follows the official [road warrior setup](https://docs.opnsense.org/manual/how-tos/wireguard-client.html) to ensure adherence to security best practices.

## 1. Server Configuration (Instance Setup)
The WireGuard service rests on the `os-wireguard` plugin architecture.

*   **Instance Name:** `Floo Network` (Primary VPN Interface)
*   **Listen Port:** `51820` (UDP)
*   **Tunnel Addresses:** `10.10.10.1/24`
    *   *Engineering Decision:* I selected a dedicated `/24` subnet separate from the main LAN (`192.168.1.0/24`) to allow for granular firewall rules. This creates a clear boundary between local devices and remote clients.
*   **Private Key:** Generated purely on the router side (never exported).

## 2. Peer Configuration (Access Control)
Each client device (Phone, Laptop) requires a distinct peer verify identity via public key cryptography.

*   **Authentication:** Ed25519 Public Keys.
*   **Pre-Shared Key (PSK):** Generated for each peer to provide a second layer of symmetric encryption, creating quantum-resistant forward secrecy properties.
*   **Allowed IPs:** Defined specific tunnel IPs for each peer (e.g., `10.10.10.2/32`) to enforce static IP assignment for logging purposes.

## 3. Interface Assignment & Routing
WireGuard interfaces in OPNsense are virtual. To apply packets filters, the instance was assigned as a logical interface.

1.  **Interfaces -> Assignments:** Mapped `wg0` to a logical interface named `WIREGUARD`.
2.  **Enable Interface:** Set `IPv4 Configuration` to nothing (handled by the WireGuard config itself) while ensuring the interface state is "Up."

## 4. Firewall & NAT Rules
This is the most critical security step. Two distinct rule sets were required to facilitate traffic flow:

### A. WAN Interface (The Entry Point)
*   **Action:** Pass
*   **Protocol:** UDP
*   **Source:** Any (Remote Road Warriors)
*   **Destination:** WAN address
*   **Port:** `51820`
*   *Purpose:* Allows the encrypted handshake to reach the router.

### B. WireGuard Interface (Internal Routing)
Rather than a "Pass All" rule, I implemented restricted routing based on aliases:

*   **Reference:** LAN Access Rule
*   **Source:** WireGuard Net
*   **Destination:** `LAN_Net`
*   **Action:** Pass
*   *Purpose:* Allows remote clients to access local servers (Proxmox, TrueNAS) but filters access to other VLANs if necessary.

## 5. Verification
*   **Handshake Status:** Verified in **VPN -> WireGuard -> Status** that the "Latest Handshake" timestamp updates, confirming successful key exchange.
*   **Traffic Graphing:** Monitored the Dashboard Traffic Graph to ensure bits were flowing over the virtual `wg1` interface during extensive file transfers.
