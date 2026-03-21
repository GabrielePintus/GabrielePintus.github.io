---
title: "Home Lab"
status: "Ongoing"
tags: [ESXi, VMware, Networking, pfSense, TrueNAS, NAS, Syncthing]
key: homelab
related_posts:
  - obsidian-sync
---

At home, I run a custom-built NAS on TrueNAS. It handles primary storage and backups, and sits quietly next to the router.

For heavier workloads, I rely on a Dell PowerEdge R710 hosted off-site at a friend's family business. In exchange for the rack space, I help out with occasional IT tasks. The facility runs on solar power, so electricity is cheap, and the noise of an enterprise server is far less of a problem in an industrial environment than it would be at home.

I have also repurposed my old desktop PC as an additional node on the same network as the R710.

Together, these three machines form a compact but flexible lab where I can experiment with virtualization, self-hosted services, and networking. It is still a work in progress — not everything is fully set up yet, but the core infrastructure is in place and I am gradually building on top of it.

The two sites are currently isolated and accessed separately. I am planning to connect them with a WireGuard site-to-site VPN in the future.

---

## Home

### Hardware

**NAS**

| | |
|---|---|
| **Motherboard** | ASRock J4105M |
| **CPU** | Intel Quad-Core J4105 |
| **RAM** | 8 GB DDR4 2400 MHz |
| **Disks** | 3× 4 TB WD Red |
| **OS** | TrueNAS CE |

A low-power dedicated NAS. The J4105 is fanless-friendly and draws very little power, making it suitable for always-on duty. This is the machine I rely on the most, as it holds all my important data and backups.

### Network

The home network (`192.168.10.0/24`) is a standard setup managed by the router. The NAS is reachable at `192.168.10.5`.

### Storage

TrueNAS manages the three WD Red disks in a **RAIDZ1** configuration, providing redundancy against a single disk failure. ZFS handles checksumming and copy-on-write, protecting against bit rot and silent data corruption.

### Services

#### Syncthing

Continuous file synchronization across devices. It runs directly on TrueNAS and is used, among other things, to sync my Obsidian vault across devices. The NAS acts as the always-on hub that keeps the laptop and desktop in sync without relying on any cloud service or paid subscription.

---

## Off-site

### Hardware

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

My former primary PC (`172.16.1.11`), repurposed as a lab node running **AlmaLinux 10**. It is connected to the same physical switch as the R710's LAN port, placing it within the `172.16.1.0/24` subnet.

### Network

ESXi uses three virtual switches to isolate traffic:

```
vmnic0 ──► vSwitch0 (WAN) ──► Internet
vmnic1 ──► vSwitch1 (LAN) ──► 172.16.1.0/24
                                    │
                               Physical Switch
                                    │
                               Desktop PC (172.16.1.11)
vmnic2 ──► vSwitch2 (DMZ) ──► 172.16.2.0/24
```

All traffic is routed through **pfSense**, running as a dedicated VM (4 vCPUs, 8 GB RAM). It manages routing, firewalling, and DHCP across all three segments. A deny-by-default security policy is enforced, with explicit rules controlling what can cross between LAN, DMZ, and WAN.

The DMZ (`172.16.2.0/24`) is where internet-facing services live. It is isolated from the LAN, so a compromised public-facing service cannot directly reach internal machines.

Remote access is handled via **OpenVPN** on pfSense. This secure tunnel (`172.16.10.0/24`) allows me to reach the off-site LAN from home without exposing internal services to the public internet.

### Storage

A **TrueNAS VM** (`172.16.1.5`, 8 vCPUs, 16 GB RAM) handles storage for the off-site environment. The Dell PERC 6/i controller is passed through directly to the VM via VMware DirectPath I/O, allowing ZFS to manage the raw disks in a **RAIDZ1** configuration.

Although the PERC 6/i is older hardware, this setup provides two layers of protection:

- **Hardware level:** The integrated battery (BBU) protects the controller's write cache in case of power failure.
- **Software level:** ZFS avoids the write hole issue through its copy-on-write architecture and uses checksums to detect and repair bit rot (silent data corruption).

### Services

Services are split between the LAN and the DMZ. A dedicated **AlmaLinux 10** VM in the LAN serves as the container host for internal services. Internet-facing services run in the DMZ, keeping them isolated from the internal network.

Public traffic enters through **Nginx**, acting as a reverse proxy in the DMZ. pfSense forwards incoming web traffic to Nginx, which serves as the single entry point for all public-facing services. This simplifies SSL certificate management and internal routing.

#### Apache Guacamole

A clientless remote desktop gateway accessible via browser. It centralizes RDP and SSH sessions for the entire lab, eliminating the need for separate clients or complex port forwarding.

#### Bitwarden (Vaultwarden)

A self-hosted password manager. It is currently the only service exposed to the internet through the reverse proxy, ensuring that sensitive credentials stay on hardware I physically control.

---

*Project ongoing — updates to follow.*
