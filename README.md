# homelab

A home lab for learning Linux, networking, cloud, and security by building things, breaking them on purpose, and writing up what happened, including the mistakes.

> Nothing here is finished. Each lab records what worked, what didn't, and what I learned from the gap.

**Labs:** [Lab 01: A Small Network](lab-01-mini-network/)

---

## Current topology

Two RHEL 10.2 VMs on VirtualBox. `server` is the whole network for `client`: its router, firewall, DHCP server, and DNS server.

```
                  Internet
                     │
          VirtualBox NAT (10.0.2.2)
                     │
 ┌───────────────────┴────────────────────────────────┐
 │ server                              RHEL 10.2        │
 │                                                      │
 │  enp0s3  10.0.2.15    zone: external  ← NAT (masquerade)
 │  enp0s8  10.0.0.1/24  zone: internal                 │
 │                                                      │
 │  • firewalld   zones + int2ext forwarding policy     │
 │  • dnsmasq     DHCP 10.0.0.100–.200, DNS for         │
 │                lab.internal, upstream 1.1.1.1        │
 │  • sshd        reached from Windows via port 2222    │
 └───────────────────┬────────────────────────────────┘
                     │  labnet (internal network, 10.0.0.0/24)
 ┌───────────────────┴────────────────────────────────┐
 │ client                              RHEL 10.2        │
 │                                                      │
 │  enp0s3  10.0.0.139 (DHCP)   default via 10.0.0.1   │
 │  • sshd        reached via `ssh -J` through server   │
 └──────────────────────────────────────────────────────┘
```

| Host | Role | Interfaces | Access |
|---|---|---|---|
| `server` | Router, NAT, firewall, DHCP, DNS | `enp0s3` NAT uplink · `enp0s8` 10.0.0.1/24 | `ssh -p 2222 sakthi@127.0.0.1` |
| `client` | Lab workstation | `enp0s3` 10.0.0.139 (DHCP) | `ssh -J sakthi@127.0.0.1:2222 sakthi@10.0.0.139` |

**Stack:** RHEL 10.2 · VirtualBox · NetworkManager (`nmcli`) · firewalld · dnsmasq · tcpdump · systemd

---

## The plan

### Phase 1: Linux and networking *(now)*
Build a real network, and understand every packet that crosses it.

- [x] Two-host routed network with DHCP, DNS, and NAT
- [x] Break-and-fix: forwarding, NAT
- [x] Break-and-fix: packet loss, MTU
- [ ] **Arduino UNO Q** joins as a real physical device: scan it, audit it, harden it
- [ ] **Dynamic routing with FRRouting:** static routes → OSPF → BGP → a small leaf-spine data-center fabric
- [ ] **Python automation** that pulls state from every device and checks it against the intended design

### Phase 2: Cloud
Extend the lab into AWS, so home and cloud act like one network.

- [ ] **WireGuard tunnel** from `server` to a small hub instance in an AWS VPC
- [ ] Routing across the tunnel in both directions: lab ↔ private cloud subnet
- [ ] Problems to work through on purpose: overlapping address ranges, NAT traversal, and AWS's source/destination check

### Phase 3: Security
Attack everything built in Phases 1 and 2, detect it, and clean it up.

- [ ] **Attack:** a Kali VM scanning and exploiting intentionally vulnerable targets on the lab network
- [ ] **Detect:** an IDS (Suricata or Zeek) on `server`, since all traffic already passes through it, plus a SIEM collecting logs from every machine
- [ ] **Respond:** find how the attacker got in, remove them, and close the hole. Practice with safe stand-ins first (EICAR, Atomic Red Team)

---

## Where it's headed

A rough sketch of the lab once all three phases are in place:

```
                         Internet
                            │
                ┌───────────┴───────────┐
                │        server         │──── WireGuard ────► AWS VPC
                │ router · NAT · fw     │                    ├─ hub (public subnet)
                │ DHCP · DNS · NTP      │                    └─ app server (private)
                │ IDS · log collector   │
                └───┬──────────────┬────┘
                    │              │
              labnet 10.0.0.0/24   │   routing lab (containerlab + FRR)
                    │              │      spine1 ── spine2
         ┌──────────┼─────────┐    │        │  ╲  ╱  │
         │          │         │    │        │   ╳    │      BGP everywhere
      client     UNO Q      Kali   │      leaf1   leaf2
      (RHEL)   (Debian +  (attacker)          │       │
               STM32)                       hosts   hosts

      monitoring: Prometheus + Grafana scraping every box
      automation: Python checks for "is the network what I designed?"
```

Hardware comes later: the plan is to move from VirtualBox on a laptop to a small always-on mini PC running Proxmox, plus a managed switch for real VLANs.
