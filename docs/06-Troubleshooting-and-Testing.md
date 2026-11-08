# 06 – Troubleshooting and Testing

## 1. Overview

During the configuration of my WireGuard VPN home lab, I encountered several issues involving cryptographic keys, firewall rules, DNS resolution, and network routing.

This section documents the troubleshooting methods I used to investigate and resolve these issues.

The objective was to establish reliable remote access to my home network using split tunneling and to route IPv4 internet traffic through my OPNsense firewall using full tunneling.

## 2. WireGuard Handshake Failure

### Problem

During the initial setup, my CachyOS laptop was unable to establish a successful WireGuard handshake with the OPNsense server.

Although the WireGuard interface could be activated, no recent handshake appeared when checking the connection status.

### Investigation

I used the following command to inspect the WireGuard connection:

```bash
sudo wg show
```

I checked the public keys, endpoint address, UDP listening port, and peer configuration on both devices.

### Cause

The OPNsense WireGuard instance's private key had mistakenly been entered into the Linux client's `[Peer] PublicKey` field.

WireGuard requires each peer to use the other device's **public key** when establishing a connection.

### Resolution

I corrected the client's configuration to use the OPNsense instance's public key.

After restarting the VPN connection, a successful handshake was established.

### Verification

```bash
sudo wg show
```

The output displayed a recent handshake and transfer statistics, confirming communication between the two peers.

## 3. Firewall Rules Blocking VPN Traffic

### Problem

After establishing a successful handshake, some traffic between the CachyOS laptop and OPNsense was still blocked.

This demonstrated that a successful WireGuard handshake does not automatically mean all network traffic is permitted through the firewall.

### Investigation

I reviewed the firewall rules under:

**Firewall → Rules → HomeVPN**

I also examined:

**Firewall → Log Files → Live View**

The logs helped identify traffic blocked by the firewall.

### Resolution

I configured rules on the assigned HomeVPN interface to permit the required traffic, including:

- ICMP traffic from `10.10.10.2` to `10.10.10.1`.
- DNS traffic using TCP/UDP port `53`.
- IPv4 traffic from the VPN client to permitted destinations.

### Verification

```bash
ping -c 4 10.10.10.1
```

Successful replies confirmed connectivity between the CachyOS client and the OPNsense WireGuard interface.

## 4. DNS Resolution Failure

### Problem

During full-tunnel testing, the VPN connection established successfully, but DNS queries failed.

This meant that network connectivity and name resolution needed to be investigated separately.

### Investigation

I tested DNS resolution directly against the OPNsense WireGuard interface:

```bash
dig @10.10.10.1 google.com
```

The query initially failed.

I then inspected the OPNsense firewall logs using:

**Firewall → Log Files → Live View**

The logs showed DNS traffic from the VPN client to `10.10.10.1` on port `53` being blocked by a default deny/state violation rule.

### Resolution

I added a firewall rule on the assigned HomeVPN interface permitting TCP/UDP DNS traffic from `10.10.10.2` to `10.10.10.1` on port `53`.

I also configured Unbound DNS under:

**Services → Unbound DNS → Access Lists**

An access list was added to allow the WireGuard network:

`10.10.10.0/24`

### Verification

```bash
dig @10.10.10.1 google.com
```

After applying the changes, DNS resolution succeeded.

This demonstrated the importance of checking both firewall permissions and DNS resolver access controls.

## 5. Split-Tunnel Routing Issues

### Problem

While testing split tunneling, I observed that traffic destined for my home LAN was sometimes routed through the laptop's Wi-Fi interface rather than the WireGuard interface.

### Investigation

I used:

```bash
ip route get 192.168.1.1
```

One test returned:

```text
192.168.1.1 dev wlan0 src 192.168.1.174
```

This indicated that the destination was being reached through the local Wi-Fi network rather than `wg0`.

### Possible Cause

The local Wi-Fi network and home LAN may both use the `192.168.1.0/24` subnet.

Overlapping subnets can create routing conflicts because the operating system has competing routes to the same destination network.

### Troubleshooting Approach

To investigate this, I checked the active interfaces and routes:

```bash
ip -4 addr
ip -4 route
```

I also checked the WireGuard interface:

```bash
sudo wg show
```

A reliable split-tunnel test should be performed from an external network that does not overlap with the home LAN subnet.

### Expected Result

```text
192.168.1.1 dev wg0 src 10.10.10.2
```

This would demonstrate that traffic to the home LAN is being routed through WireGuard.

**Note:** The earlier `wlan0` result is evidence of a routing issue, not proof of successful split-tunnel LAN routing. A new test from a non-overlapping external network is required to verify that behavior.

## 6. Full-Tunnel Internet Connectivity

### Problem

For full tunneling to work, the OPNsense firewall must allow traffic from the WireGuard network to reach the internet.

The VPN connection alone does not provide internet access without appropriate forwarding, firewall permissions, and outbound NAT.

### Investigation

I reviewed the OPNsense outbound NAT configuration:

**Firewall → NAT → Outbound**

I also checked the HomeVPN firewall rules and tested external IPv4 connectivity.

### Resolution

I used Automatic outbound NAT on OPNsense and confirmed that the generated rules included the WireGuard network.

Firewall rules were configured to allow the VPN client to access external IPv4 destinations.

The CachyOS full-tunnel configuration used:

`AllowedIPs = 0.0.0.0/0`

This directs IPv4 traffic through WireGuard.

### Verification

I tested internet connectivity:

```bash
ping -c 4 1.1.1.1
```

I then checked the public IPv4 address:

```bash
curl -4 https://ifconfig.me
```

The result matched my home internet connection's public IPv4 address when connected from an external network.

This confirmed that IPv4 internet traffic was exiting through the home OPNsense firewall.

## 7. Final Testing Commands

The following commands were used throughout the project to investigate and verify connectivity.

| Command | Purpose |
|---|---|
| `sudo wg show` | Check WireGuard peers, handshakes, and transfer statistics |
| `ping -c 4 10.10.10.1` | Test connectivity to the VPN server |
| `ip route get 192.168.1.1` | Check the selected route to the home LAN |
| `ip -4 route` | Inspect IPv4 routing entries |
| `ip -4 rule show` | Inspect IPv4 policy routing rules |
| `dig @10.10.10.1 google.com` | Test DNS resolution through OPNsense |
| `ping -c 4 1.1.1.1` | Test external IPv4 connectivity |
| `curl -4 https://ifconfig.me` | Check the public IPv4 address |

## 8. Lessons Learned

This project improved my understanding of how VPN connectivity depends on multiple networking components working together.

The key lessons included:

- **Cryptographic key configuration:** WireGuard requires the correct public and private key relationships between peers.
- **Firewall troubleshooting:** A successful VPN handshake does not guarantee that traffic is permitted through the firewall.
- **DNS troubleshooting:** DNS failures can result from firewall rules or resolver access restrictions, even when basic IP connectivity works.
- **Network routing:** Overlapping subnets can interfere with split-tunnel routing.
- **Network Address Translation:** Outbound NAT is essential when private VPN client addresses need to access the public internet through the home gateway.
- **Evidence-based troubleshooting:** Commands, firewall logs, and connectivity tests help identify the cause of networking problems.

## 9. Conclusion

I successfully configured and tested a WireGuard VPN using OPNsense as the VPN server and CachyOS Linux as the client.

The project demonstrated both split-tunnel remote access and full-tunnel IPv4 internet connectivity.

Troubleshooting handshake failures, firewall restrictions, DNS resolution, and routing behavior provided practical experience in Linux networking, firewall administration, and VPN security.

Further verification of split-tunnel routing from a non-overlapping external network and IPv6 behavior remains useful for completing the testing evidence.
