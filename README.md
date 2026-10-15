# Michel's Office Network

A small-business network lab built in **Cisco Packet Tracer**. It has four department VLANs, inter-VLAN routing, DHCP pools, NAT to a simulated internet, and basic device hardening.

- **Lab file:** `Small-Business-Network.pkt`
- **Platform:** Cisco Packet Tracer 8.x / 9.x
- **Devices:** 2 routers (2911), 1 switch (2960-24TT), 8 endpoints

## Topology

```
Internet (8.8.8.8 loopback)
        │
      [ISP]  Gi0/0 ── 203.0.113.1/30
        │
      [R1]   Gi0/1 ── 203.0.113.2/30   (NAT overload)
        │    Gi0/0 ── 802.1Q trunk     (router-on-a-stick)
      [SW1]  Gi0/1 ── trunk to R1
        │
   ┌────┴─────┬──────────┬──────────┐
 VLAN 10    VLAN 20    VLAN 30    VLAN 40
 Executive  Sales      Accounting IT
```

## Addressing plan

| VLAN | Department | Subnet | Gateway (R1) | Switch ports | Hosts |
|---|---|---|---|---|---|
| 10 | Executive | 192.168.10.0/24 | 192.168.10.1 | Fa0/1–2 | Michel-PC (.11), Exec-Laptop (.12) |
| 20 | Sales | 192.168.20.0/24 | 192.168.20.1 | Fa0/3–4 | Sales-PC1 (.11), Sales-PC2 (.12) |
| 30 | Accounting | 192.168.30.0/24 | 192.168.30.1 | Fa0/5–6 | Acct-PC1 (.11), Acct-PC2 (.12) |
| 40 | IT | 192.168.40.0/24 | 192.168.40.1 | Fa0/7–8 | IT-PC1 (.11), IT-PC2 (.12) |
| 99 | Network management | 192.168.99.0/24 | 192.168.99.1 | SW1 `Vlan99` = .2 | Kept off the internet (not NATed) |
| WAN | R1 ↔ ISP | 203.0.113.0/30 | ISP = .1, R1 = .2 | — | — |

All hosts use DNS `8.8.8.8`, which is the ISP router's loopback and acts as the test "internet".

## Features

- **VLAN segmentation:** each department gets its own broadcast domain.
- **Inter-VLAN routing:** R1 uses dot1Q subinterfaces on `Gi0/0`.
- **DHCP:** R1 has pools for VLANs 10–40, with `.1–.50` excluded for static devices. The hosts are currently static, so the pools are available but unused.
- **NAT (PAT overload):** user VLANs 10–40 share R1's WAN address. The management VLAN is deliberately excluded.
- **Hardening:** `enable secret`, a local `admin` user, `login local` on the VTY lines, a console password and `service password-encryption`, with unique credentials per device.

## Usage

1. Open `Small-Business-Network.pkt` in Cisco Packet Tracer.
2. Log in to a device CLI with the credentials you set (see Security notes).
3. Test connectivity from any PC's Command Prompt:

   ```
   ping 192.168.20.1     # own gateway
   ping 192.168.30.11    # another department (inter-VLAN routing)
   ping 8.8.8.8          # internet (NAT)
   ```

## Why the segments are separated

| VLAN | Reason |
|---|---|
| Executive | Sensitive files and communications stay away from other departments. |
| Sales | Customer-facing staff, kept apart from finance data. |
| Accounting | Financial systems get their own broadcast domain, which makes access control easier. |
| IT | Administrators have their own segment and can be given broader access later. |
| Management | Switch and router management traffic is kept off user and internet paths. |

## Security notes

- Passwords are **not** stored in this repo. Choose unique, strong values for each device.
- `enable secret` and `username … secret` are hashed (type 5). `service password-encryption` only weakly obfuscates the console line (type 7).
- For production, prefer SSH over Telnet and a stronger hash type than type 5.

## Known limitations / ideas

- The hosts use static IPs. Switch some to DHCP to exercise the pools.
- Add SSH (`transport input ssh` plus RSA keys). Packet Tracer needs a `reload` for crypto on a 2911.
- Add ACLs between VLANs, for example to stop Sales reaching Accounting.
- Add a second switch, wireless access or a redundant uplink.

## Change log

- **Fixed:** SW1 access-port VLANs now match the cabling and each host's subnet. Previously 6 of 8 ports were in the wrong VLAN.
- **Added:** device password hardening on R1, ISP and SW1.
- **Changed:** the NAT ACL is now limited to VLANs 10–40, so management traffic is no longer translated.

## Author

Michel
