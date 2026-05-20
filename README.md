# Hybrid-SD-WAN-MPLS-Architecture-Deployment

> Academic project — Design and deployment of a hybrid SD-WAN + MPLS network architecture using FortiGate firewalls and Cisco vIOS routers in a virtualized EVE-NG environment.

---

##  Project Overview

| Field | Details |
|-------|---------|
| **Institution** | ENSA Kénitra — Ibn Tofail University |
| **Program** | Réseaux et Systèmes Télécommunications |
| **Module** | Technologies de Réseaux WAN |
| **Authors** | EL-ACHOURI Chaima · ZAOUI Ihssane |
| **Supervisor** | Mme. C. Kissi |
| **Academic Year** | 2024–2025 |
| **Simulator** | EVE-NG |

---

##  Objectives

- Design and deploy a **hybrid MPLS + SD-WAN** network infrastructure
- Implement **intelligent routing** — critical traffic prioritized over MPLS, less sensitive traffic over Internet
- Ensure **high availability** with automatic failover between MPLS and Internet links
- Secure inter-site communications via **IPsec VPN tunnels**
- Provide **centralized management** using FortiGate firewalls

---

##  Infrastructure Deployed

### Components

| Equipment | Quantity | Role |
|-----------|----------|------|
| **FortiGate** (VM64-KVM) | 4 | SD-WAN, VPN, Firewall |
| **Cisco vIOS** | 4 | MPLS backbone routers |
| **VPC** (virtual PCs) | 8 | User traffic simulation |
| **Subnets** | 4 | Net1, Net2, Net3, Net4 |

### Sites

| Site | Role |
|------|------|
| **Tokyo** | Hub — centralized Internet exit, MPLS core |
| **Sapporo** | Spoke — MPLS + Internet (SD-WAN) |
| **Okayama** | Spoke — MPLS + Internet (SD-WAN) |
| **Nagoya** | Spoke — MPLS + Internet (SD-WAN) |

---

##  IP Addressing Plan

| Equipment | Interface | IP Address |
|-----------|-----------|------------|
| VPC5 | eth0 | 172.16.254.2/24 |
| VPC6 | eth0 | 172.16.0.2/24 |
| VPC7 | eth0 | 172.16.1.2/24 |
| VPC8 (PC4) | eth0 | 172.16.2.2/24 |
| Fortinet Net1 | port2 | 172.16.254.1/24 |
| Fortinet Net1 | port3 (MPLS) | 172.20.254.1/30 |
| Fortinet Net1 | port4 (WAN) | 100.64.254.1/28 |
| Fortinet Net2 | port2 | 172.16.0.1/24 |
| Fortinet Net2 | port3 (MPLS) | 172.20.0.1/30 |
| Fortinet Net2 | port4 (WAN) | 100.64.1.1/28 |
| Fortinet Net3 | port2 | 172.16.1.1/24 |
| Fortinet Net3 | port3 (MPLS) | 172.20.1.1/30 |
| Fortinet Net4 | port2 | 172.20.2.1/30 |
| Fortinet Net4 | port3 | 172.16.2.1/24 |
| vIOS9 (R1) | Gi0/0 | 172.20.254.2/30 |
| vIOS9 (R1) | Gi0/1 | 172.20.0.2/30 |
| vIOS9 (R1) | Gi0/2 | 172.20.1.2/30 |
| vIOS9 (R1) | Gi0/3 | 172.20.2.1/30 |
| vIOS11 (R2) | Gi0/0 | 100.64.1.2/28 |
| vIOS11 (R2) | Gi0/1 | 100.64.3.2/28 |
| vIOS12 (R3) | Gi0/0 | 100.64.0.1/28 |
| vIOS12 (R3) | Gi0/1 | 100.64.2.2/28 |

---

##  Architecture

### Two Complementary Layers

**SD-WAN Layer (FortiGate)**
- Direct interconnection between FortiGate firewalls
- Intelligent routing based on link performance
- Dual VPN tunnels per site: `VPN-INET` (Internet) + `VPN-MPLS` (MPLS)
- SD-WAN zone: `virtual-wan-link` grouping both tunnels
- Health check `MPLS_HC` — ICMP ping to 8.8.8.8 every 1s, failover after 5 failures

**MPLS Layer (vIOS Routers)**
- MPLS backbone with LDP (Label Distribution Protocol)
- Dynamic routing via BGP (AS 65001 / AS 65002)
- OSPF for intra-domain routing
- Static routes for LAN-to-MPLS reachability

---

##  Key Configurations

### MPLS Routers (vIOS)
```
router bgp 65001
  bgp router-id <loopback-ip>
  neighbor <peer-ip> remote-as 65002

mpls ldp router-id Loopback0
mpls ip
```

### FortiGate — SD-WAN
```
config system sdwan
  set status enable
  config zone
    edit "virtual-wan-link"
  config members
    edit 1
      set interface "VPN-MPLS"
    edit 2
      set interface "VPN-INET"
  config health-check
    edit "MPLS_HC"
      set server "8.8.8.8"
      set protocol ping
      set interval 1000
      set failtime 5
```

### FortiGate — IPsec VPN (Hub)
```
config vpn ipsec phase1-interface
  edit "VPN-INET"
    set type dynamic
    set ike-version 2
    set proposal des-sha1
    set dhgrp 14
    set peertype any
    set dpd on-demand
    set add-route disable
  edit "VPN-MPLS"
    set type dynamic
    set ike-version 2
    set proposal des-sha1
    set dhgrp 14
    set auto-discovery-sender enable
```

### FortiGate — BGP
```
config router bgp
  set as 65002
  set router-id 172.16.254.1
  config neighbor
    edit "172.20.254.2"
      set remote-as 65001
      set ebgp-enforce-multihop enable
```

### Firewall Policy
```
config firewall policy
  edit 1
    set name "LAN to WAN"
    set srcintf "port2"
    set dstintf "virtual-wan-link"
    set srcaddr "all"
    set dstaddr "all"
    set action accept
    set schedule "always"
    set service "ALL"
```

---

##  Connectivity Tests

| Test | Result |
|------|--------|
| Ping PC1 → PC2 (172.16.254.1) |  Success — avg ~2.6ms |
| Ping PC2 → Router 1 (172.20.254.2) |  Success — avg ~3.5ms |
| Ping Router 3 → Router 1 (100.64.254.1) |  Success — 100% (5/5) — avg 2/3/5ms |
| Inter-site BGP peering |  Neighbors Up |
| VPN-INET tunnel |  Established |
| VPN-MPLS tunnel |  Established |

---

##  SD-WAN vs MPLS Comparison

| Criteria | MPLS | SD-WAN |
|----------|------|--------|
| Cost | High | Low |
| Flexibility | Limited | Very high |
| Security | Good but costly | Native via IPsec |
| Performance | Low latency, optimized QoS | Variable per link |
| Cloud integration | Difficult | Native |

---

##  Migration Approach

1. **Start:** Full MPLS — centralized Internet exit via Tokyo Hub
2. **Step 1:** Configure Hub (Tokyo) — dual dynamic IPsec tunnels (VPN-INET + VPN-MPLS), SD-WAN zone, BGP
3. **Step 2:** Migrate each Spoke — add local Internet link, dual VPN tunnels, SD-WAN zone, BGP
4. **Result:** All sites on MPLS + Internet hybrid — automatic path selection via SD-WAN policies

---

##  Documentation

| File | Description |
|------|-------------|
| `Projet_sdwan_mpls.pdf` | Full project report with configurations and screenshots |

---

##  Disclaimer

This project was conducted in a **virtualized academic environment (EVE-NG)** for educational purposes only. It does not represent a production deployment. All configurations are for learning purposes within the context of the WAN Technologies module at ENSA Kénitra.
