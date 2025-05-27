# Net363 Final v1.1
## Scoring
- Configurations: 70%
- Connectivity 30%
## Abstract
**Congratulations**, your company has won the bid to configure the network of our new corporate HQ and first brick-and-mortar store!

Since we already have an online presence and existing cloud infrastructure, we only need you to set up the network at the HQ and store for employees and guest to do their work. Right now, the team is small and our needs are basic, so we only need static routing, basic security and redundancy set up by the time the doors are ready to open.

## Requirements

### Corporate Office
#### VLANs
| VLAN Number | NAME        | IPv4 Network | IPv6 Network            | Hosts Needed incl. gateway(s)  |
| ---         | ---         | ---          | ---                     | ---                            |    
| 100         | HR          | 10.0.10.0/? | 2001:ACAD:BEEF:100::/64  | 12                             |
| 200         | Accounting  | 10.0.20.0/? | 2001:ACAD:BEEF:200::/64  | 24                             |
| 300         | Staff       | 10.0.30.0/? | 2001:ACAD:BEEF:300::/64  | 200                            |
| 400         | Executives  | 10.0.40.0/? | 2001:ACAD:BEEF:400::/64  | 9                              |
| 500         | Management  | 10.0.50.0/? | 2001:ACAD:BEEF:500::/64  | 8                              |

- We know the maximum amount of devices that will be in each network, and in order to conserve IPv4 address space for later growth we require that you use the smallest possible network sizes.
- Since IPv6 addresses are basically unlimited, we would like the standard /64 prefixes implemented.

#### Routers
| Router  | Interface | IPv4                | IPv6                   | VIPv4              |
| ---     | ---       | ---                 | ---                    | ---                |
| Corp_R1 | g0/0      | 10.0.0.1/30         | 2001:ACAD:BEEF:1::1/64 | N/A                |
|         | g0/1      | 10.0.0.5/30         | 2001:ACAD:BEEF:2::1/64 | N/A                |
|         | g0/2      | 10.0.0.9/30         | 2001:ACAD:BEEF:3::1/64 | N/A                |
|         | s0/0/0    | 10.0.0.14/30        | 2001:ACAD:BEEF:4::2/64 | N/A                |
| Corp_R2 | g0/0      | 10.0.0.10/30        | 2001:ACAD:BEEF:3::2/64 | N/A                |
|         | g0/1      | VLAN 500 (first)    | VLAN 500 (first)       | VLAN 500 (third)   |
|         | g0/1.100  | VLAN 100 (first)    | VLAN 100 (first)       | VLAN 100 (third)   |
|         | g0/1.200  | VLAN 200 (first)    | VLAN 200 (first)       | VLAN 200 (third)   |
|         | g0/1.300  | VLAN 300 (first)    | VLAN 300 (first)       | VLAN 300 (third)   |
|         | g0/1.400  | VLAN 400 (first)    | VLAN 400 (first)       | VLAN 400 (third)   |
| Corp_R3 | g0/0      | 10.0.0.6/30         | 2001:ACAD:BEEF:2::2/64 | N/A                |
|         | g0/1      | VLAN 500 (second)   | VLAN 500 (second)      | VLAN 500 (third)   |
|         | g0/1.100  | VLAN 100 (second)   | VLAN 100 (second)      | VLAN 100 (third)   |
|         | g0/1.200  | VLAN 200 (second)   | VLAN 200 (second)      | VLAN 200 (third)   |
|         | g0/1.300  | VLAN 300 (second)   | VLAN 300 (second)      | VLAN 300 (third)   |
|         | g0/1.400  | VLAN 400 (second)   | VLAN 400 (second)      | VLAN 400 (third)   |

- General
    - Configure hostnames on all routers matching the table above

- Addressing:
    - Assign router IP addresses per the above schema. 
        - Where an address shows VLAN XXX, assign it the proper IP to the VLAN in the VLAN table. (first), (second), or (third) corresponds to the first, second, or third usable IP address in the VLAN subnet respectively.
        - dot1q encapsulation type should be used on vlan sub-interfaces
- Point-to-point links:
    - Serial point-to-point links should be configured to use PPP and CHAP authentication.
    - Configure a CHAP username and password for DC R1 to authenticate to Corp R1
        - Username: DC_R1
        - Password: DC_R1_Corp_R1_363f1n4l
- First-hop Redundancy:
    - Configure HSRP on Corp R2 and Corp R3 for each VLAN use group 1
    - Configure HSRP v2
    - Configure a Virtual IPv4 address for each VLAN, refer to the table above for address assignments
    - Corp R2 shall be the primary router, configured to assume the active role when it is online. Configure it to have a priority of 150
- Routing
    - Enable IPv6 Unicast Routing
    - Corp R1 should be configured with a static default route for IPv4 and IPv6 with a next-hop of ISP Router
    - Corp R1 should be configured with specific IPv4 routes for each Store 1 network with a next-hop of DC_R1. Use interface routes on point-to-point links
    - Corp R1 should be configured with a route for DC Network (172.16.0.0/12) with a next-hop of DC_R1. Use interface routes on point-to-point links
    - Corp R1 should be configured with specific IPv4 routes for each link between routers with a next-hop of DC_R1. Use interface routes on point-to-point links
    - Corp R1 should be configured with static IPv4 and IPv6 routes to all Corporate Office VLANs via Corp_R2
    - Corp R1 should be configured with floating static IPv4 and IPv6 routes to all Corporate Office VLANs via Corp_R3 with an administrative Distance of 5
    - Corp R2 and Corp R3 should be configured with a static default route for IPv4 and IPv6 with a next-hop of Corp R1

- DHCP
    - For VLANs 100,200,300, and 400 configure DHCP relay for ipv4 on Corp_R2. The IP helper address should be DC Server1.

#### Switches
- Configure Host Names to match diagram
- Configure the VLANS according to the table above
- STP
    - Configure Corp_S1 to be the root bridge for all VLANs with a priority of 0
    - Configure Corp_S2 to be the secondary root bridge with a priority of 4096
- Configure port-channels on all reduntant links. Configure all port-channel modes to be active LACP.
    - Corp_S1
        - po1
            - members: fa0/1 - 2
    - Corp_S2
        - po1
            - members: fa0/1 - 2
    - Corp_S3
        - po1
            - members: fa0/1 - 2
        - po2
            - members: fa0/3 - 4
    - Corp_S4
        - po1
            - members: fa0/1 - 2
        - po2
            - members: fa0/3 - 4
- Access Ports
    - Configure port fa0/8 on Corp_S3 to be an access port assigned to vlan 300
    - Configure port fa0/5 on Corp_S4 to be an access port assigned to vlan 400
    - Configure port fa0/6 on Corp_S4 to be an access port assigned to vlan 100
    - Enable Spanning-tree portfast on ports fa0/5 - 10 on Corp_S3 and Corp_S4
    - Enable Spanning-tree bpduguard on ports fa0/5 - 10 on Corp_S3 and Corp_S4

- Trunk Ports
    - Configure trunk ports to use industry-standard dot1q encapsulation on Corp_S1 and Corp_S2
    - Configure all links between switches as trunk ports with a Native VLAN of 500
    - Configure trunk ports from Corp_S1 and Corp_S2 to Corp_R1 and Corp_R2 with a Native VLAN of 500

#### Wireless Networks
- Configure Corp_AP1
    - Configure the SSID on Corp_AP1 to be Corp_Execs
    - Configure the Authentication to be WPA2-PSK and the Pass Phrase to be net363execs!
- Ensure Corp Laptop1 can connect and get an IP address via DHCP

### DataCenter
## The DataCenter Router and Server are preconfigured, this information is for reference
#### Routers
| Router  | Interface | IPv4                | IPv6                      |
| ---     | ---       | ---                 | ---                       |
| DC_R1   | fa0/0     | 172.16.0.1/16       | 2001:ACAD:BEEF:1000::1/64 |
|         | s0/0/0    | 10.0.0.13/30        | 2001:ACAD:BEEF:4::1/64    |
|         | s0/0/1    | 10.0.0.21/30        | 2001:ACAD:BEEF:5::1/64    |

- Routing
    - DC_R1 should be configured with a route for 10.0.0.0/8 using the interface s0/0/0
    - DC_R1 should be configured with a route for 192.168.0.0/16 using the interface s0/0/1

- Point-to-point links:
    - Serial point-to-point links should be configured to use PPP and CHAP authentication.
        - Configure a CHAP username and password for Corp_R1 to authenticate to DC_R1
            - Username: Corp_R1
            - Password: DC_R1_Corp_R1_363f1n4l
        - Configure a CHAP username and password for Store_R1 to authenticate to DC_R1
            - Username: Store_R1
            - Password: DC_R1_Store_R1_363f1n4l

#### Servers
| Server      | Interface         | IPv4            | IPv6                        |
| ---         | ---               | ---             | ---                         |
| DC Server 1 | fastethernet0     | 172.16.0.100/16 | 2001:ACAD:BEEF:1000::100/64 |


### Store 1
#### VLANs

| VLAN Number | NAME        | IPv4 Network    | IPv6 Network            |
| ---         | ---         | ---             | ---                     |    
| 10          | Store       | 192.168.1.0/24  | 2001:ACAD:BEEF:7::/64   |
| 99          | Guest       | 192.168.99.0/24 | N/A                     |

#### Routers
| Router   | Interface | IPv4                | IPv6                   |
| ---      | ---       | ---                 | ---                    |
| Store_R1 | g0/0      | 192.168.254.1/30    | 2001:ACAD:BEEF:6::1/64 |
|          | g0/1      | 192.168.1.1/24      | 2001:ACAD:BEEF:7::1/64 |
|          | g0/1.99   | 192.168.99.1/24     | N/A                    |
|          | s0/0/1    | 10.0.0.22/30        | 2001:ACAD:BEEF:5::2/64 |

- Configure Hostname of Store_R1

- Confgure Store R1 to be a DHCP server for vlan 99
    - Configure a DHCP pool named Guest
    - Configure the Default gateway to be g0/1.99
    - Configure the network for vlan 99
    - Configure the DHCP scope to exclude the IP address of R1
    - Configure the DNS server to be DC_Server

- Configure Store_R1 to be a DHCP relay for the store VLAN. Use DC Server1 as the IP Helper address
- dot1q encapsulation type should be used on vlan sub-interfaces


- Point-to-point links:
    - Serial point-to-point links should be configured to use PPP and CHAP authentication.
        - Configure a CHAP username and password for DC_R1 to authenticate to Store_R1
            - Username: DC_R1
            - Password: DC_R1_Store_R1_363f1n4l

- Routing
    - Store_R1 should be configured with a static default route for IPv4 and IPv6 with a next-hop of ISP Router
    - Store_R1 should be configured with a static route for 10.0.0.0/8 and 172.16.0.0/12 Use interface routes via DC_R1


#### Switches
- Configure hostname to Store_S1
- Configure VLANS in the table above
- Configure a VLAN 10 interface on Store_S1
    - ip 192.168.1.254
    - Configure the default gateway for Store_S1 to be Store_R1

- Trunk Ports
    - Configure trunk ports to use industry-standard dot1q encapsulation on Store_S1
    - Port g0/1 should be configured as a trunk port, use vlan 10 as the native-vlan

- Access Ports
    - Port g0/2 should be configured as an acc port on vlan 99
    - Ports fa0/1 - 2 should be configured as access ports on vlan 10
    - Enable Spanning-tree portfast on ports g0/2, fa0/1 - 2 Store_S1
    - Enable Spanning-tree bpduguard on ports g0/2, fa0/1 - 2 Store_S1
    - Configure port security on Store_S1
        - Ports fa0/1 - 2
        - Configure fa0/1 - 2 ot use sticky mac addresses
        - Configure the port security mode to restrict
        - Disable all other disconnected ports

#### Wireless Networks
- Configure Store_AP1
    - Configure the SSID on Corp_AP1 to be Guest
    - Configure the Authentication to be Disabled
- Ensure Guest Laptop 1 can connect and get an IP address via DHCP

#### Remote Management
- Configure SSH for remote management on Store_R1 and Store_S1
    - use RSA key modulus size 2048
    - Use net363.local as the domain name
    - Configure vty 0 - 4 to only allow ssh
        - Configure vty lines to use local user accounts
    - configure a username and secret for SSH authentication
        - username: net363admin
        - secret:   net363admin!
    - configure priveliged exec secret to be net363admin!