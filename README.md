# infra-labs

A home lab for learning Linux, networking, cloud, and security by building things, breaking them on purpose, and writing up what happened, including the mistakes.

> Nothing here is finished. Each lab records what worked, what didn't, and what I learned from the gap.

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

### In progress
- [ ] **DHCP reservation:** pin `client` to `10.0.0.139` and resolve `client.lab.internal`
- [ ] **Time server:** `chrony` on `server` as the lab's NTP source, handed out over DHCP
- [ ] **Central logging:** `client` sends its logs to `server` with rsyslog

---

## Labs

### 🌐 Network
| Lab | What it covers |
|---|---|
| [01 · A Small Network, Imperfectly Built](network/lab-01-mini-network/) | Building the topology above: DHCP and DNS with dnsmasq, routing and NAT with firewalld zones and policies |
| [01 · Part 2: Break It on Purpose](network/lab-01-mini-network/PART-2.md) | Break-and-fix challenges: tracing a forwarding failure through packet captures and firewall logs, plus a boot-time race in dnsmasq |

---

## Roadmap

**Phase 1: Linux and networking** *(now)*
- [x] Two-host routed network with DHCP, DNS, and NAT
- [ ] Remaining break-and-fix challenges: NAT, packet loss, MTU
- [ ] DHCP reservation, NTP, central logging
- [ ] Arduino UNO Q joins the network as a real device: scan, audit, harden
- [ ] Dynamic routing with FRRouting: static routes → OSPF → BGP → a small leaf-spine fabric
- [ ] Python automation that checks network state against the intended design

**Phase 2: Cloud**
- [ ] WireGuard tunnel connecting this lab to an AWS VPC (hybrid network)

**Phase 3: Security**
- [ ] Attack the lab from a Kali VM
- [ ] Detect it with Suricata/Zeek and a SIEM
- [ ] Respond and clean up

---

## How each lab is written

Every lab follows the same shape:
- **What I built**, with a diagram
- **The build**: configs and commands
- **Proof it works**, with screenshots and captures
- **Where it cracked**: each problem as symptom, cause, fix, and lesson
- **Still unfinished**

The "Where it cracked" sections are the most important part. Most of what I've learned came from things going wrong.
