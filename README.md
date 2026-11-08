# WireGuard VPN Home Lab

## Project Overview

This project documents the deployment and configuration of a WireGuard VPN using an OPNsense firewall and a CachyOS Linux client.

The aim was to establish secure remote access to my home network and configure two different VPN tunneling methods: split tunneling and full tunneling.

The project involved configuring WireGuard peers, public/private key authentication, firewall rules, routing, outbound NAT, and DNS services.


## Project Documentation

- [01 – WireGuard Overview](docs/01-WireGuard-Overview.md)  - A look a what Wireguard is.
- [02 – OPNsense Server Setup](docs/02-OPNsense-Server-Setup.md)  - Deploy a WireGuard VPN server on OPNsense.
- [03 – Linux Client Setup](docs/03-Linux-Client-Setup.md)  - Configure a CachyOS Linux laptop as a WireGuard client.
- [04 – Split-Tunnel Configuration](docs/04-Split-Tunnel-Configuration.md)  - Implement split tunneling for remote access to the home LAN.
- [05 – Full-Tunnel Configuration](docs/05-Full-Tunnel-Configuration.md)  - Implement full tunneling to route internet traffic through the home firewall.
- [06 – Troubleshooting and Testing](docs/06-Troubleshooting-and-Testing.md)  - Troubleshoot VPN handshakes, routing issues, and DNS connectivity.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| OPNsense | Firewall, VPN server and network gateway |
| WireGuard | Encrypted VPN tunnel |
| CachyOS Linux | VPN client |
| Unbound DNS | DNS resolution through the VPN |
| Linux networking tools | Connectivity and routing verification |

## VPN Configurations

### Split Tunnel

The split-tunnel configuration routes traffic destined for the home network through WireGuard while allowing normal internet traffic to use the client's existing connection.

### Full Tunnel

The full-tunnel configuration routes all IPv4 internet traffic through the WireGuard tunnel and out through the home OPNsense firewall.

## Project Results

Both configurations were successfully tested using the CachyOS Linux client.

The split tunnel provided remote connectivity to the OPNsense VPN address, while the full tunnel successfully routed internet traffic through the home connection.

DNS connectivity issues were identified and resolved through firewall rule configuration.

## Documentation

Detailed installation, configuration, troubleshooting, and testing instructions will be added to the `docs` directory.

## Security Considerations

WireGuard uses public-key cryptography for peer authentication and key agreement, and symmetric encryption to protect VPN traffic.

The full-tunnel setup has been verified for IPv4. IPv6 leak protection and kill-switch behavior require separate verification.
