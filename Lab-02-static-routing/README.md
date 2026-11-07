# Lab 02: Static Routing with Basic Device Security

A Packet Tracer lab that connects two LANs through three routers with static routes, and applies basic access security and housekeeping settings to the routers.

## Objective

1. Connect PC1 (10.10.1.0/24) and PC2 (10.10.3.0/24) across R1, R2 and R3 using static routes.
2. Apply basic security to the routers: privileged-mode secret, console and remote-line passwords, password encryption and a login banner.
3. Set the time zone and clock. 

## Topology


PC1 --- Sw1 --- R1 --- R2 --- R3 --- Sw2 --- PC2




## Cabling

|From|Interface|To|Interface|
|-|-|-|-|
|Sw1|g0/1|R1|g0/1|
|R1|g0/0|R2|g0/1|
|R2|g0/1|R3|g0/0|
|R3|g0/1|Sw2|g0/1|

## Addressing

|Device|Interface|IP address|Mask|Gateway|
|-|-|-|-|-|
|PC1|NIC|10.10.1.1|255.255.255.0|10.10.1.254|
|PC2|NIC|10.10.1.2|255.255.255.0|10.10.1.254|
|PC3|NIC|10.10.3.1|255.255.255.0|10.10.2.254|
|PC4|NIC|10.10.3.2|255.255.255.0|10.10.3.254|
|R1|to Sw1|10.10.1.254|255.255.255.0|-|
|R1|to R2|192.168.20.1|255.255.255.0|-|
|R2|to R1|192.168.20.2|255.255.255.0|-|
|R2|to R3|192.168.35.1|255.255.255.0|-|
|R3|g0/0 (to R2)|192.168.35.2|255.255.255.0|-|
|R3|g0/1 (to Sw2)|10.10.3.254|255.255.255.0|-|

Addressing plan: 10.10.x.0/24 for the LANs and 192.168.x.0/24 for the router-to-router links. All addresses are private lab addresses and do not belong to any real network.

## Static routes


R1(config)# ip route 10.10.3.0 255.255.255.0 192.168.20.2

R1(config)# ip route 192.168.35.0 255.255.255.0 192.168.20.2

&#x20;

R2(config)# ip route 10.10.1.0 255.255.255.0 192.168.20.1 

R2(config)# ip route 10.10.3.0 255.255.255.0 192.168.35.2



R3(config)# ip route 10.10.1.0 255.255.255.0 192.168.35.1 

R3(config)# ip route 192.168.20.0 255.255.255.0 192.168.35.1 



Each router has a route to every LAN it is not directly connected to, and the next hop is always the neighbouring router's address on the shared link.

Full device configs are in [`configs/`](configs/).

## Device security configured

|Setting|Command (example)|Purpose|
|-|-|-|
|Hostname|`hostname R1`|Identify each device|
|Privileged-mode secret|`enable secret <password>`|Protects privileged EXEC mode; stored as a hash|
|Console password|`line console 0` / `password <password>` / `login`|Protects the physical console|
|Remote-line password|`line vty 0 4` / `password <password>` / `login`|Protects remote logins|
|Password encryption|`service password-encryption`|Hides line passwords in the config (weak type 7 encoding, not strong protection)|
|Login banner|`banner motd # Authorized access only #`|Legal warning to anyone connecting|
|Disable DNS lookup|`no ip domain-lookup`|Stops the router stalling on mistyped commands|

Devices configured: `<R1 / R2 / R3, list only the ones you actually configured>`

## Time and logging

```
R1(config)# clock timezone EAT 3
R1# clock set <hh:mm:ss> <day> <month> <year>
R1# show clock
```

1. East Africa Time (UTC+3). The timezone is set first, then the clock. (The timezone is set but packet tracer has an issue)



## Troubleshooting: what went wrong

**1. Clock showed 3 hours ahead**

* **Symptom:** `show clock` displayed a time 3 hours later than the real local time.
* **Cause:** The clock was set before the timezone. The router stored the typed time as UTC, then added the +3 offset when displaying it.
* **Fix:** With the timezone already configured, set the clock again with local time.
* **Lesson:** Set the timezone first, then the time.

**2. PC1 could ping R3 but not reach PC2**

* **Symptom:** Pings from PC1 reached R3's address but not anything behind R3.
* **Cause:** A static route on R2 had the wrong destination network. It pointed at PC1's LAN via R3, so R2 had no route to PC2's LAN.
* **Fix:** Removed the wrong route and added a route to 10.10.3.0/24 via R3's link address.
* **Lesson:** In a static route, the destination is the network you want to reach, and the next hop is the neighbouring router. Check `show ip route` one router at a time.

## Not configured in this lab

* SSH (remote access is by console or the VTY line password only)
* Per-user accounts and privilege levels
* ACLs and port security
* Shutting down unused ports

These are planned for later labs.

## Files

|File|Description|
|-|-|
|`lab-02-router-basic-security.pkt`|Packet Tracer file|
|`configs/R1.txt`, `R2.txt`, `R3.txt`|`show running-config` output|
|`screenshots/`|Topology and verification screenshots|

## What I learned

* Static routes need a correct destination network and next hop on every router, in both directions.
* Time zone must be configured before the clock is set.
* `service password-encryption` is not strong security; `enable secret` is the proper way to protect privileged mode.
* Basic passwords are only a starting point. Real hardening needs SSH, individual accounts and access control.
* The command is enable secret <password> not enable secret "class" <password>. 

