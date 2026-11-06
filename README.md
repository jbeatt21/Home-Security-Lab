# Home Cybersecurity Lab

A virtualized security lab built on repurposed laptop hardware for practicing
network segmentation, attack simulation, log analysis, and system hardening.

## Goals
- Build an isolated environment to safely run offensive security tools
- Detect simulated attacks using a SIEM (Wazuh)
- Practice hardening Linux systems and verifying controls

## Hardware
| Component | Spec |
|---|---|
| Device | HP 14-dq1043cl laptop |
| CPU | Intel Core i3-1005G1 (2 cores / 4 threads, VT-x enabled) |
| RAM | 8 GB (6.9 GiB usable; second slot open for upgrade to 16 GB) |
| Host OS | Ubuntu 26.04 LTS |

## Progress
- [x] Hardware assessment and system updates
- [x] KVM/libvirt hypervisor + Cockpit web management
- [x] Headless operation with remote access over Tailscale
- [x] SSH hardening (key-only authentication)
- [ ] Isolated lab network with OPNsense firewall
- [ ] Vulnerable target (Metasploitable 2)
- [ ] Attacker VM (Kali Linux)
- [ ] Wazuh SIEM + custom detection rules

## Build Log

### 1. Hardware Assessment
Confirmed CPU virtualization support (`/proc/cpuinfo` vmx flags) and checked
memory layout with `dmidecode`. With 8 GB of RAM, the lab runs VMs in rotation
rather than all at once.

### 2. Hypervisor Setup
Installed KVM/libvirt and Cockpit for web-based VM management.
**Troubleshooting:** `qemu-kvm` has no installation candidate on newer Ubuntu
releases; installed `qemu-system-x86` instead.

### 3. Remote Access
**Problem:** Campus Wi-Fi uses client isolation, which blocks
device-to-device traffic, so direct SSH timed out.
**Solution:** Deployed Tailscale (a WireGuard-based mesh VPN) to reach the lab
without exposing any ports to the network. Disabled lid-close suspend through a
systemd-logind drop-in so the lab runs headless.

### 4. SSH Hardening
- Generated an ed25519 key pair; the private key stays on the client only
- Disabled password and keyboard-interactive auth and direct root login
  through a drop-in config (`/etc/ssh/sshd_config.d/10-hardening.conf`)
- Validated the config with `sshd -t` before restarting the service
- **Verified:** forcing password authentication returns
  `Permission denied (publickey)`

```
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin no
```

## Security Notes
IP addresses, usernames, and credentials are intentionally left out of this repository.
Offensive tools are confined to an isolated virtual network with no route to the campus network (in progress).

## What I Learned
Password-based SSH can be an easy target for automated brute force attacks; thus, I switched to key-only authentication and further confirmed that password logins are rejected. Testing a control matters just as much as configuring it. I also learned that campus wifi uses client isolation to block device-to-device traffic, so I used Tailgate, which is a WireGuard-based VPN, to securely connect my own devices without opening any port on the school network.
