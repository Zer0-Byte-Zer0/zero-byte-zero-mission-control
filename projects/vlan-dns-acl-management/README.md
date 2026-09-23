\# VLAN Segmentation, DNS, and ACL Management Lab



\## Project Overview



This Cisco Packet Tracer lab demonstrates network segmentation, inter-VLAN routing, switch management, internal DNS, web services, and service-specific access control. The project also documents the troubleshooting and correction of an overly broad ACL that initially blocked all employee-to-server communication.



\## Network Design



The network uses one Cisco 2960 switch and one Cisco 2911 router to connect two logical networks:



| Network  | VLAN | Subnet            | Purpose                                    |

| -------- | ---: | ----------------- | ------------------------------------------ |

| EMPLOYEE |   10 | `192.168.10.0/24` | Employee workstation and switch management |

| SERVER   |   20 | `192.168.20.0/24` | DNS and web server                         |



\### Addressing



| Device        | Address         | Role                    |

| ------------- | --------------- | ----------------------- |

| ZBZ-RTR-01    | `192.168.10.1`  | VLAN 10 default gateway |

| ZBZ-RTR-01    | `192.168.20.1`  | VLAN 20 default gateway |

| ZBZ-SW-01     | `192.168.10.2`  | Switch management SVI   |

| ZBZ-WS-01     | `192.168.10.10` | Employee workstation    |

| ZBZ-SRV-UBU01 | `192.168.20.20` | DNS and web server      |



\## Implementation



\* Assigned the employee workstation port to VLAN 10.

\* Assigned the server port to VLAN 20.

\* Configured an 802.1Q trunk between the switch and router.

\* Used router subinterfaces to provide inter-VLAN routing.

\* Configured a VLAN 10 switch virtual interface at `192.168.10.2`.

\* Set the switch default gateway to `192.168.10.1`.

\* Enabled DNS, HTTP, and HTTPS services on the server.

\* Created the DNS record `portal.zbz.local`, resolving to `192.168.20.20`.

\* Applied an extended ACL inbound on the employee VLAN.



\## Troubleshooting and Remediation



Initial testing showed that the employee workstation could not reach the server. The workstation received “Destination host unreachable” responses from its default gateway.



Testing from the router successfully reached the server, proving that VLAN 20 and the server connection were operational. Inspection of the router revealed that the `EMPLOYEE\_TO\_SERVER` ACL denied all IP traffic from VLAN 10 to VLAN 20:



```text

deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255

```



This rule unintentionally blocked ICMP, DNS, HTTP, HTTPS, and all other IP services. I replaced it with a service-specific rule:



```text

10 deny tcp 192.168.10.0 0.0.0.255 host 192.168.20.20 eq www

20 permit ip any any

```



The revised ACL blocks unencrypted HTTP access to the server while preserving DNS resolution, ICMP testing, HTTPS access, and other permitted connectivity.



\## Verification



The completed configuration produced the following results:



\* The workstation successfully pinged the switch management address.

\* The workstation successfully pinged the server across VLAN boundaries.

\* `portal.zbz.local` resolved to `192.168.20.20`.

\* HTTP access timed out as required by the ACL.

\* HTTPS successfully loaded the Cisco Packet Tracer webpage.

\* VLANs 10 and 20 were active across the trunk.

\* Router and switch configurations were saved to startup configuration.



\## Security Concepts Demonstrated



\* Network segmentation

\* Router-on-a-stick

\* 802.1Q trunking

\* Switch management through an SVI

\* Inter-VLAN routing

\* Internal DNS

\* Extended access control lists

\* Least-privilege access

\* Layered troubleshooting

\* Configuration verification and change management



\## Key Takeaway



A functioning network is not necessarily a correctly secured network. The original ACL stopped communication, but it was too broad to support legitimate business services. The corrected rule enforced a specific security requirement while maintaining required connectivity.



This project demonstrates a practical workflow:



\*\*Build → Test → Isolate the failure → Correct the control → Verify the result → Save and document\*\*



