# 01 – WireGuard Overview

## Introduction

WireGuard is a modern VPN protocol designed to provide secure, encrypted communication between devices over a network.

For this project, WireGuard was deployed on an OPNsense firewall, with a CachyOS Linux laptop configured as the VPN client.

Two VPN configurations were implemented:

- **Split Tunnel:** Routes traffic destined for the home network through WireGuard, while other internet traffic uses the client's normal connection.
- **Full Tunnel:** Routes all IPv4 internet traffic through WireGuard and the home OPNsense firewall.

## How WireGuard Works

WireGuard uses public-key cryptography to authenticate peers and establish shared encryption keys.

Each device has its own public and private key pair.

The private key remains on its respective device, while the public key is shared with the other peer.

WireGuard uses Curve25519 for key agreement and ChaCha20-Poly1305 for authenticated encryption.

## Network Addressing

| Device | IP Address |
|---|---|
| OPNsense LAN | `192.168.1.1/24` |
| Home LAN Network | `192.168.1.0/24` |
| WireGuard Network | `10.10.10.0/24` |
| OPNsense WireGuard | `10.10.10.1` |
| CachyOS Client | `10.10.10.2/32` |
| WireGuard Listening Port | UDP `51820` |

## Split Tunnel vs Full Tunnel

| Split Tunnel | Full Tunnel |
|---|---|
| Home network traffic uses VPN | All IPv4 traffic uses VPN |
| Normal internet traffic stays local | Internet traffic exits through home WAN |
| Useful for remote LAN access | Useful for securing traffic on public Wi-Fi |
| `AllowedIPs = 10.10.10.0/24, 192.168.1.0/24` | `AllowedIPs = 0.0.0.0/0` |

## Project Architecture

The VPN server runs on the OPNsense firewall, which is connected to the internet through the home ISP.

The CachyOS laptop connects to the OPNsense WireGuard instance over an encrypted tunnel.

Depending on the selected client configuration, traffic is either routed to the home LAN or forwarded to the internet through the OPNsense WAN interface.
