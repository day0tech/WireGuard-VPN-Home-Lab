# WireGuard VPN Home Lab

## Project Overview

This project documents the deployment and configuration of a WireGuard VPN using an OPNsense firewall and a CachyOS Linux client.

The aim was to establish secure remote access to my home network and configure two different VPN tunneling methods: split tunneling and full tunneling.

The project involved configuring WireGuard peers, public/private key authentication, firewall rules, routing, outbound NAT, and DNS services.

## Project Objectives

- Deploy a WireGuard VPN server on OPNsense.
- Configure a CachyOS Linux laptop as a WireGuard client.
- Implement split tunneling for remote access to the home LAN.
- Implement full tunneling to route internet traffic through the home firewall.
- Configure firewall rules and outbound NAT for VPN connectivity.
- Troubleshoot VPN handshakes, routing issues, and DNS connectivity.
- Verify VPN operation using Linux networking tools.

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
