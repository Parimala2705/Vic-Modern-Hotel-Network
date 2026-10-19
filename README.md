# Vic Modern Hotel Network (Cisco Packet Tracer)

An end-of-year networking project: design and implementation of an enterprise network for a three-floor hotel, built in Cisco Packet Tracer. It uses VLANs, inter-router serial links, OSPF routing, DHCP on the routers, SSH remote management, and port security.



## Scenario

The hotel has three floors, each with its own router and switch. The three routers sit together in the server room in the IT department.

| Floor | Departments |
|-------|-------------|
| 1st | Reception, Store, Logistics |
| 2nd | Finance, HR, Sales/Marketing |
| 3rd | IT, Admin |

## Requirements

1. Three routers, one per floor, all placed in the server room in the IT department.
2. Routers connected to each other with serial DCE cables.
3. Router-to-router links use `10.10.10.0/30`, `10.10.10.4/30` and `10.10.10.8/30`.
4. One switch per floor, placed on that floor.
5. WiFi on each floor for laptops and phones.
6. A printer in every department.
7. Each department in its own VLAN (table below).
8. OSPF as the routing protocol.
9. All devices get their IP address dynamically, with the floor's router acting as DHCP server.
10. All devices can communicate with each other.
11. SSH configured on all routers for remote login.
12. A PC named `Test-PC` on port `fa0/1` of the IT switch, used to test remote login.
13. Port security on the IT switch so only `Test-PC` can use `fa0/1` (sticky MAC, violation mode `shutdown`).

## VLAN and addressing plan

| Floor | Department | VLAN | Network |
|-------|------------|------|---------|
| 1st | Reception | 80 | 192.168.8.0/24 |
| 1st | Store | 70 | 192.168.7.0/24 |
| 1st | Logistics | 60 | 192.168.6.0/24 |
| 2nd | Finance | 50 | 192.168.5.0/24 |
| 2nd | HR | 40 | 192.168.4.0/24 |
| 2nd | Sales/Marketing | 30 | 192.168.3.0/24 |
| 3rd | Admin | 20 | 192.168.2.0/24 |
| 3rd | IT | 10 | 192.168.1.0/24 |

Router-to-router links:

| Link | Network |
|------|---------|
| 1 | 10.10.10.0/30 |
| 2 | 10.10.10.4/30 |
| 3 | 10.10.10.8/30 |

> Fill in your router names and interface IPs here (for example F1-Router, F2-Router, F3-Router and the serial and sub-interface addresses from `show ip interface brief`).

## How it works

- **VLANs and trunking:** each floor switch holds that floor's VLANs. The link to the router is a trunk, and the router uses sub-interfaces (router-on-a-stick) as the gateway for each VLAN.
- **OSPF:** all three routers advertise their VLAN networks and the `10.10.10.x` links, so every department can reach every other one.
- **DHCP:** each router has DHCP pools for the VLANs on its floor, so PCs, laptops, phones and printers get addresses automatically.
- **SSH:** every router has a domain name, an RSA key, a local user, SSH version 2, and `transport input ssh` with `login local` on the vty lines. `Test-PC` is used to test remote login.
- **Port security:** on the IT switch, `fa0/1` is restricted to `Test-PC` using sticky MAC learning, and a violation shuts the port down.

## Example commands

SSH setup on a router:

```
hostname F3-Router
ip domain-name hotel.local
username gtech privilege 15 secret <password>
enable secret <password>
crypto key generate rsa modulus 1024
ip ssh version 2
line vty 0 15
 transport input ssh
 login local
```

Port security on the IT switch:

```
interface fa0/1
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

## Testing

- Ping between PCs in different VLANs and on different floors.
- From `Test-PC`, log in to each router: `ssh -l gtech <router IP>`
- Check routing with `show ip route` and `show ip ospf neighbor`.
- Check port security with `show port-security interface fa0/1`.

## Troubleshooting notes

SSH login failed with `% Login invalid` even though ping worked. The cause was that each router needs its own SSH setup and local user. A router that was missing the `gtech` user, the RSA key, or `login local` on the vty lines rejected the login. Configuring each router separately fixed it.

## How to open

1. Install Cisco Packet Tracer.
2. Open `Hotel_Mangagement_Networking_Project.pkt`.

The usernames and passwords in this project are lab-only examples.

## Tools

Cisco Packet Tracer

## Author

Parimal, CSE student, SRM University
