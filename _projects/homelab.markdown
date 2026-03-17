---
title: "Home Lab — Personal Server"
status: "Ongoing"
tags: [ESXi, VMware, Networking, pfSense]
---

I run a personal home lab on a **Dell PowerEdge R710**. Despite its age, this enterprise-grade hardware remains more cost-effective and capable than most modern consumer alternatives for virtualization.

## Hardware

**Server — Dell PowerEdge R710**

| | |
|---|---|
| **CPU** | 2× Intel Xeon X5650 (12 cores / 24 threads @ 2.67 GHz) |
| **RAM** | 72 GB DDR3 ECC |
| **Hypervisor** | VMware ESXi 6.7 |

The X5650 CPUs (Westmere-EP) still handle virtualization workloads effectively. With 72 GB of ECC RAM, I can run multiple concurrent VMs without hitting memory bottlenecks.

**Desktop PC**

| | |
|---|---|
| **CPU** | Intel Core i7-4770 |
| **RAM** | 12 GB DDR3 |
| **GPU** | NVIDIA GTX 1070 |

My former primary PC (`172.16.1.11`) has been repurposed as a lab node running **AlmaLinux 10**. It is connected to the same physical switch as the server's LAN port, placing it within the `172.16.1.0/24` subnet.

---

## Network

ESXi uses two virtual switches to isolate traffic:

```
vmnic0 ──► vSwitch0 (WAN) ──► Internet
vmnic1 ──► vSwitch1 (LAN) ──► 172.16.1.0/24
                                    │
                               Physical Switch
                                    │
                               Desktop PC (172.16.1.11)
```

The `vmnic1` interface connects to a physical switch, bridging the virtual environment with physical hardware on the local network.

All traffic is routed through **pfSense**, running as a dedicated VM (4 vCPUs, 8 GB RAM). It manages routing, firewalling, and DHCP (serving the `.101` to `.200` range). Moving to a dedicated firewall has allowed me to implement a deny-by-default security policy for the entire lab.

Remote access is handled via **OpenVPN** on pfSense. This secure tunnel (`172.16.10.0/24`) allows me to access the LAN without exposing internal services to the public internet.

---

## Storage

Storage is managed by a **TrueNAS** VM (`172.16.1.5`, 8 vCPUs, 16 GB RAM).

The Dell PERC 6/i controller is passed through directly to the VM via VMware DirectPath I/O. This allows ZFS to manage the raw disks in a **RAIDZ1** configuration.

Although the PERC 6/i is older hardware, this setup provides two layers of protection:

- **Hardware level:** The integrated battery (BBU) protects the controller's write cache in case of power failure.
- **Software level:** ZFS avoids the write hole issue through its copy-on-write architecture and uses checksums to detect and repair bit rot (silent data corruption).

---

## Services

A dedicated **AlmaLinux 10** VM serves as the container host. This separation of concerns ensures that the hypervisor handles the hardware, the VM provides the OS, and Docker manages the applications.

Public traffic is handled by **Nginx**, acting as a reverse proxy. pfSense forwards incoming web traffic to Nginx, which serves as the single entry point. This simplifies the management of SSL certificates and internal routing.

### Apache Guacamole

A clientless remote desktop gateway accessible via browser. It centralizes RDP and SSH sessions for the entire lab, eliminating the need for separate clients or complex port forwarding.

### Bitwarden (Vaultwarden)

A self-hosted password manager. It is currently the only service exposed to the internet through the reverse proxy, ensuring that sensitive credentials stay on hardware I physically control.

---

*Project ongoing — updates to follow.*
