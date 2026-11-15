# CCNA IOS CLI, SSH and Password Recovery Lab

**Author:** Suraj Pun Magar
**Program:** CCNA Networking Internship, Week 1, Task 3
**Tool:** Cisco Packet Tracer (free with a Cisco Networking Academy account)

## Overview

This lab practises the basics of the Cisco IOS command line on a router and a switch: configuration modes, device hardening, interface addressing, SSH remote access, the key `show` commands, saving configuration, and recovering a router whose password is lost.

## Topology

```
PC0 ---- SW1 (2960-24TT) ---- R1 (2911)
         192.168.10.0/24
```

| Device | Interface | IP address | Default gateway |
|--------|-----------|------------|-----------------|
| PC0 | NIC | 192.168.10.10 /24 | 192.168.10.1 |
| R1 | GigabitEthernet0/0 | 192.168.10.1 /24 | n/a |
| SW1 | VLAN 1 (SVI) | 192.168.10.2 /24 | 192.168.10.1 |

## What was configured

On both R1 and SW1:

- Hostname, login banner (`banner motd`)
- `enable secret` for privileged mode
- `service password-encryption`
- Console password with `login` and `logging synchronous`
- Local user `admin`, domain name `lab.local`
- 1024-bit RSA keys and **SSH version 2**
- VTY lines restricted to SSH (`transport input ssh`, `login local`)

On R1: `GigabitEthernet0/0` addressed and enabled.
On SW1: VLAN 1 addressed and a default gateway set.

## Verification

- `show ip interface brief` on R1: G0/0 is `up/up` with 192.168.10.1
- `show ip interface brief` on SW1: Vlan1 is `up/up` with 192.168.10.2
- `show ip ssh` on both devices: `SSH Enabled - version 2.0`
- Ping from PC0 to R1: 4 sent, 4 received, 0% loss
- `copy running-config startup-config` completed with `[OK]` on both devices

## Troubleshooting notes

| Problem | Cause | Fix |
|---------|-------|-----|
| Setup dialog opened by mistake | Answered `yes` at boot | Ctrl+C to abort, confirmed config was unchanged |
| `% Invalid input` on `crypto key generate rsa modulus 1024` | This Packet Tracer IOS does not accept the `modulus` keyword | Used `crypto key generate rsa` and entered 1024 at the prompt |
| `show ip ssh` showed `SSH Disabled` | No RSA keys existed yet | Generated the keys, then `ip ssh version 2` |
| Pings stuck, packets not moving | Packet Tracer was in Simulation mode | Switched to Realtime |
| `opy running-config` rejected | Typo | Retyped as `copy` |

## Password recovery (router)

The configuration register tells the router how to boot. `0x2102` loads the saved `startup-config`; `0x2142` skips it so the router starts with a blank config and no passwords.

1. `reload`, then press Ctrl+C (or Ctrl+Break) during boot to reach ROMMON
2. `confreg 0x2142` then `reset`
3. Answer `no` to the setup dialog, then `enable`
4. `copy startup-config running-config`
5. Set a new `enable secret` and console password, run `no shutdown` on the interface
6. `config-register 0x2102`, then `copy running-config startup-config`
7. `reload` and log in with the new password

## Progress checklist

- [x] Topology built and addressed
- [x] Hostname, banner, secrets and console passwords on R1 and SW1
- [x] SSH version 2 enabled on R1 and SW1
- [x] Connectivity tested with ping
- [x] Configuration saved on R1 and SW1
- [ ] SSH login tested from PC0
- [ ] Reload test (saved vs unsaved config)
- [ ] Password recovery performed on R1

## Repository contents

```
README.md
report/        PDF report
configs/       R1 and SW1 running-config text files
screenshots/   Packet Tracer screenshots from this lab
lab.pkt        Packet Tracer file
```

## Skills practised

Cisco IOS CLI, device hardening, IP addressing, SSH, `show` commands, configuration management, password recovery, technical documentation.
