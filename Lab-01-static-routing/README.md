# Lab 01: Static Routing (3 Routers, 2 Switches and 2 PCs)

A Packet Tracer lab where two LANs communicate through three routers using static routes only (no routing protocol).

## Objective

To configure static routes so PC1 (192.168.1.0/24) and PC2 (192.168.3.0/24) can communicate to each other through R1, R2 and R3.

## Topology



PC1 --- Sw1 --- R1 --- R2 --- R3 --- Sw2 --- PC2



## Cabling

|From|Interface|To|Interface|
|-|-|-|-|
|Sw1|g0/1|R1|g0/1|
|R1|g0/0|R2|g0/0|
|R2|g0/1|R3|g0/0|
|R3|g0/1|Sw2|g0/1|

## Addressing

|Device|Interface|IP address|Mask|Gateway|
|-|-|-|-|-|
|PC1|NIC|192.168.1.1|255.255.255.0|192.168.1.254|
|PC2|NIC|192.168.3.1|255.255.255.0|192.168.3.254|
|R1|g0/1 (to Sw1)|192.168.1.254|255.255.255.0|-|
|R1|g0/0 (to R2)|192.168.12.1|255.255.255.0|-|
|R2|g0/0 (to R1)|192.168.12.2|255.255.255.0|-|
|R2|g0/1 (to R3)|192.168.13.2|255.255.255.0|-|
|R3|g0/0 (to R2)|192.168.13.3|255.255.255.0|-|
|R3|g0/1 (to Sw2)|192.168.3.254|255.255.255.0|-|

## Static routes

Each router needs a route to every network it is not directly connected to.



R1(config)# ip route 192.168.13.0 255.255.255.0 192.168.12.2
R1(config)# ip route 192.168.3.0 255.255.255.0 192.168.12.2

R2(config)# ip route 192.168.1.0 255.255.255.0 192.168.12.1
R2(config)# ip route 192.168.3.0 255.255.255.0 192.168.13.3

R3(config)# ip route 192.168.1.0 255.255.255.0 192.168.13.2
R3(config)# ip route 192.168.12.0 255.255.255.0 192.168.13.2



## **Verification**

* PC1 to PC2 ping: !\[Ping](screenshots/ping-pc1-to-pc2.png)
* Routing table on R2: !\[Routes](screenshots/show-ip-route-r2.png)
* Path taken (tracert 192.168.3.1 from PC1 and PC2)

## **Troubleshooting: what went wrong**

**Symptom:** PC1 could ping R1's LAN interface (192.168.1.254) but not R1's other interface or anything beyond it. PC2 had the same problem.

**Cause:** The default gateway on both PCs was set to the network address (192.168.1.0 and 192.168.3.0) instead of the router interface address. The PCs could still reach their own subnet, but traffic for other networks was sent to an address no device owns.

**Fix:** Set the gateway to the router interface in the same subnet: 192.168.1.254 for PC1 and 192.168.3.254 for PC2.

**Lesson:** Reaching your own subnet does not prove the gateway is correct. Test the gateway first, then work outward one hop at a time.



## **What I learned**

* A router only knows its directly connected networks until you add routes.
* Static routes must exist in both directions, or replies never return.
* The PC default gateway must match the router interface address in the same subnet.





&#x20;   **All IP addresses are private lab addresses and do not belong to any real network.**

