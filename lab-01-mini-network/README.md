# Lab 01: Small Network

Two RHEL 10.2 VMs, where one is the other's entire network. Built by hand, then broken on purpose one piece at a time.

← [homelab](../)

---

## What this lab is

`client` boots with no network configuration and gets everything from `server`:

- an IP address over **DHCP** (dnsmasq)
- name lookups through **DNS** (dnsmasq, `lab.internal`)
- internet access through **routing and NAT** (firewalld zones + a forwarding policy)

It's a small version of what a home router does.

```
     Internet
        │
 VirtualBox NAT
        │
   ┌────┴─────┐  enp0s3 10.0.2.15   (external zone, masquerade)
   │  server  │
   └────┬─────┘  enp0s8 10.0.0.1    (internal zone, dnsmasq)
        │   labnet 10.0.0.0/24
   ┌────┴─────┐
   │  client  │  enp0s3 10.0.0.139  (DHCP, default via 10.0.0.1)
   └──────────┘
```

---

## Contents

| Part | What happens | Key lesson |
|---|---|---|
| [**The build**](lab1.md) | VMs, cloning, `nmcli`, firewalld zones and policy, dnsmasq, verification with `dig`, `curl`, `tracepath` | A clone carries hidden identity: hostname, machine ID, registration, MAC, SSH host keys |
| [**Challenge 1:** Remove forwarding](challenge1.md) | Disable the `int2ext` policy and trace where packets stop | **Reject vs. drop:** rejected traffic fails instantly (40 ms); the firewall log shows exactly why |
| [**Challenge 2:** Turn off NAT](challenge2.md) | Disable masquerade and watch the source address on both sides of `server` | **Routing gets packets out; NAT gets replies back.** Silent failures time out (10 s) |
| Challenge 5: Packet loss *(next)* | 10% loss with `netem`; watch TCP retransmit | |
| Challenge 6: MTU mismatch *(planned)* | Small packets work, large ones silently fail | |

---

## Skills practiced

- **Linux:** `nmcli`, `firewall-cmd`, `sysctl`, `systemctl`, systemd drop-ins, `journalctl`, `vi` and `nano`, SSH jump hosts
- **Networking:** DHCP (full exchange vs. lease renewal), DNS forwarding, routing, NAT and connection tracking, ICMP error types
- **Troubleshooting:** packet captures on both sides of a router, decoding kernel firewall logs, reading curl error codes, testing reboots

---

## The best cracks

The full list is in each writeup. These taught the most:

- **It worked until I rebooted.** dnsmasq started before `enp0s8` existed. Fixed with `bind-dynamic` and a systemd ordering override. The packaged unit file had a comment suggesting the same fix.
- **`1.1.1.1` became `1.1.1.0`.** A stray keypress in vi. DNS kept working through backup resolvers, so the only clue was a line in the logs.
- **The wildcard that wouldn't expand.** `sudo rm internal-*.lease` failed because *my* shell expands `*` before `sudo` runs, and my user can't read the directory.
- **Empty captures.** I pinged before the captures were running. Now I start captures first, and confirm they work with a ping to `server` before trusting an empty result.

---

## Status

- [x] Build: DHCP, DNS, routing, NAT
- [x] Full DHCP exchange captured
- [x] Challenge 1: forwarding
- [x] Challenge 2: NAT
- [x] Challenge 3: packet loss
- [ ] Challenge 4: MTU mismatch

---

## Files

```
lab-01-mini-network/
├── README.md        ← you are here
├── lab1.md          the build
├── challenge1.md    remove forwarding (+ the boot-time race)
├── challenge2.md    turn off NAT
└── screenshots/
```
