# Cisco Packet Tracer Labs: Router & Switch Configuration

Two network installation labs built and tested in **Cisco Packet Tracer** on Cisco ISR 4300-series routers and Catalyst switches. They cover IPv4/IPv6 addressing, SSH hardening, static routing and a backup (floating) default route, all configured from the IOS command line.

Completed for *ICTTEN312 Install telecommunications network equipment* (Advanced Diploma of Electronics and Communications Engineering, RMIT, 2026).

| Lab | Topics | Packet Tracer file |
|---|---|---|
| [1. Router setup, SSH and IPv6](#lab-1--router-setup-ssh-and-ipv6) | Dual-stack addressing, password policy, SSH-only VTY access, IPv6 routing | [`lab1-ssh-ipv6.pkt`](lab1-router-ssh-ipv6/lab1-ssh-ipv6.pkt) |
| [2. Static and floating routes](#lab-2--static-and-floating-static-routes) | Static routes, default routes, floating static backup route, traceroute | [`lab2-static-routing.pkt`](lab2-static-floating-routes/lab2-static-routing.pkt) |

Open the `.pkt` files in Cisco Packet Tracer 8.x.

---

## Lab 1: Router setup, SSH and IPv6

![Topology](lab1-router-ssh-ipv6/docs/topology.png)

### Addressing

| Device | Interface | IPv4 | IPv6 | Link-local |
|---|---|---|---|---|
| R1 | G0/0/0 | 192.168.0.1/24 | 2001:db8:acad::1/64 | fe80::1 |
| R1 | G0/0/1 | 192.168.1.1/24 | 2001:db8:acad:1::1/64 | fe80::1 |
| R1 | Loopback0 | 10.0.0.1/24 | 2001:db8:acad:2::1/64 | fe80::1 |
| PC-A | NIC | 192.168.1.10/24 (gw .1) | 2001:db8:acad:1::10/64 | gw fe80::1 |
| PC-B | NIC | 192.168.0.10/24 (gw .1) | 2001:db8:acad::10/64 | gw fe80::1 |

### What I configured on R1

- Hostname, `no ip domain-lookup`, and `service password-encryption`.
- `security passwords min-length 12`.
- A local user `SSHadmin` with an encrypted secret, and a 1024-bit RSA key for SSH.
- VTY lines 0–4 set to `login local` and `transport input ssh` (SSH only, no Telnet), with `exec-timeout 4 0` to log idle sessions out after 4 minutes.
- `ipv6 unicast-routing`.
- IPv4, IPv6 global and IPv6 link-local addresses on all three interfaces, with descriptions.
- Configuration saved to startup-config.
- On S1: hostname, and configuration saved.

| VTY / SSH config | Interfaces up |
|---|---|
| ![VTY](lab1-router-ssh-ipv6/docs/vty-ssh-config.png) | ![Interfaces](lab1-router-ssh-ipv6/docs/show-ip-interface-brief.png) |

### Verification

- **Ping:** PC-A → PC-B across the router, 4/4 replies, 0% loss.
- **SSH:** PC-A logged in to R1 at 192.168.1.1 as `SSHadmin`.
- **`show version`:** ISR 4300, IOS XE 03.16.05.S. I used it to find the system image file and the model.

| Ping PC-A → PC-B | SSH from PC-A |
|---|---|
| ![Ping](lab1-router-ssh-ipv6/docs/ping-pc-b.png) | ![SSH](lab1-router-ssh-ipv6/docs/ssh-client.png) |

---

## Lab 2: Static and floating static routes

![Topology](lab2-static-floating-routes/docs/topology.png)

### Addressing

| Device | Interface | IPv4 |
|---|---|---|
| R1 | G0/0/0 | 172.16.1.1/24 |
| R1 | G0/0/1 | 192.168.1.1/24 |
| R1 | Loopback1 | 10.1.0.1/24 |
| R1 | Loopback2 | 209.165.200.225/27 |
| R2 | G0/0/0 | 172.16.1.2/24 |
| R2 | G0/0/1 | 192.168.1.2/24 |
| R2 | Loopback1 | 10.2.0.1/24 |
| R2 | Loopback2 | 209.165.200.193/27 |

The two routers are joined directly on G0/0/0, and through switches S1 and S2 on G0/0/1. That gives two paths between them.

### Routes configured

```
! R1: static route to R2's Loopback1, via the switched link
ip route 10.2.0.0 255.255.255.0 192.168.1.2

! R1: floating static default route via the switched link (AD 5 = backup)
ip route 0.0.0.0 0.0.0.0 192.168.1.2 5

! R2: default route back to R1
ip route 0.0.0.0 0.0.0.0 172.16.1.1
```

A **floating static route** has a higher administrative distance (5 here) than a normal static route (1). It stays out of the routing table until the main route fails, so it acts as a backup path.

### Verification

| `show ip route` on R1 | `traceroute 10.2.0.1` |
|---|---|
| ![Routing table](lab2-static-floating-routes/docs/show-ip-route.png) | ![Traceroute](lab2-static-floating-routes/docs/traceroute.png) |

R1's routing table shows the connected networks, the static route to `10.2.0.0/24` via 192.168.1.2, and a default route. `traceroute 10.2.0.1` goes through **192.168.1.2** in one hop, as required.

---

## Skills shown

- Building a topology from a plan, with correct cabling between routers, switches and PCs.
- Cisco IOS CLI: interface configuration, IPv4 and IPv6 dual-stack, saving configs.
- Device hardening: password policy, encrypted passwords, SSH-only remote access, idle timeouts.
- Static, default and floating static routing, and administrative distance.
- Checking the network with `show ip route`, `show ip interface brief`, `show version`, ping, traceroute and SSH.
- Working to WHS requirements for network installation: hazard identification, ESD control, cable management.
