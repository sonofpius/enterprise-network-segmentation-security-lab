# Enterprise Network Segmentation \& DMZ Security Lab



## Overview



This project demonstrates the design, implementation and validation of a segmented enterprise network using VMware virtualization and a firewall/router.



The objective was not simply to configure a firewall, but to design and test security controls that address realistic enterprise problems such as lateral movement, excessive network access, guest-device exposure and insecure placement of public-facing services.



The environment contains multiple departmental networks, a dedicated IT network, a Guest network, an isolated DMZ and an Ubuntu web server.



---



## Project Objectives



* Design a segmented enterprise network.

* Separate departments into security zones.

* Implement a deny-by-default firewall policy.

* Apply least-privilege network access.

* Isolate guest devices from internal resources.

* Place a public-facing web server inside a DMZ.

* Allow only HTTP/HTTPS traffic from the WAN to the DMZ web server.

* Prevent DMZ systems from accessing internal networks.

* Test segmentation using Kali Linux, ICMP and Nmap.

* Capture firewall logs as evidence that security controls are working.



---



## Enterprise Problem



A flat enterprise network can allow an attacker who compromises one workstation to move laterally across the organization.






```text

┌──────────────────────────────┐
│  User Workstation Compromised│
└───────────────┬──────────────┘
                ↓
┌────────────────────────────────────────┐
│     Flat Network (No Segmentation)     │
└───────────────┬────────────────────────┘
                ↓
┌────────────────────────────────────────┐
│  Departmental Systems (HR / Finance)   │
└───────────────┬────────────────────────┘
                ↓
┌──────────────────────────────┐
│   Admin & Executive Systems  │
└───────────────┬──────────────┘
                ↓
┌──────────────────────────────┐
│ High-Value Assets & Data     │
└──────────────────────────────┘


```



This creates unnecessary exposure to sensitive systems and increases the potential impact of a successful compromise.



A second problem is the placement of public-facing applications. An internet-facing web server should not have unrestricted access to internal corporate systems.



This project addresses these risks by separating the network into security zones and controlling communication between those zones through firewall policy.



---



## Security Architecture



                                                         ┌──────────┐
                                                         │ internet │
                                                         └──────────┘
                                                               |
                                                         ┌──────────┐
                                                         │  WAN/NAT │
                                                         └──────────┘
                                                               |
                                                    ┌────────────────────┐
                                                    │ Opnsense firewall  │________________________________________________________________________________
                                                    └──────────┬─────────┘                                                                               |
                                                    /    /     |  \  \  \  \______________________________________________________________________       |
                                                  /    /       |    \  \  \__________________________________________________                    |       |
                 ┌──────────────────┐           /     /        |     \  \________________________________                    |                   |       |
                 │     IT           │_________/      /         |      \_____________                     |                   |                   |       |
                 │ (windowsever/AD) │┌──────────────────┐┌──────────────────┐       |                    |                   |                   |       |
                 └──────────────────┘│   Executives     ││        HR        │┌──────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌────────────┐ |   
                                     │ (windows 11 pc)  ││ (windows 11 pc)  ││    FINANCE       │ │  SALES          │ │ ENGINEERING     │ │    DMZ     │ |
                                     └──────────────────┘└──────────────────┘│ (windows 11 pc)  │ │(windows 11 pc)  │ │(Ubuntu endpoint)│ │ubuntuserver│ |
                                                                             └──────────────────┘ └─────────────────┘ └─────────────────┘ └────────────┘ |
                                                                                                                                                         | 
                                                                                                                                               ┌────────────┐
                                                                                                                                               │    GUEST   │ 
                                                                                                                                               │     BYOD   │     
                                                                                                                                               └────────────┘

                   
---



## Network Segmentation



| Segment         | Network         | Purpose                  |
| --------------- | --------------- | ------------------------ |
| IT              | 192.168.10.0/24 | Administration           |
| Executive       | 192.168.20.0/24 | Executive users          |
| HR              | 192.168.30.0/24 |  Human Resources         |
| Finance         | 192.168.40.0/24 | Financial system         |
| Sales           | 192.168.50.0/24 | Sales operations         |
| Engineering     | 192.168.60.0/24 |Engineering/Development   |
|  DMZ            | 192.168.70.0/24 |  Public-facing web server|
|  Guest          | 192.168.80.0/24 | Guest/BYOD devices       |



---



## Security Controls Implemented



### 1. Network Segmentation



Each department is placed into a separate network segment.



This reduces unnecessary communication between departments and limits the potential for lateral movement following a compromise.



### 2. Deny-by-Default Firewall



The firewall follows a deny-by-default model.



Only explicitly required traffic is permitted.







ALLOW required traffic
         +
DENY everything else





### 3. Guest Network Isolation



Guest devices are allowed to access the Internet but are prevented from accessing internal corporate networks.




Guest - Internet       ALLOW

Guest - Internal LAN   BLOCK




### 4. Department Isolation



Sensitive departments such as HR and Finance are prevented from communicating where there is no legitimate business requirement.



### 5. DMZ Isolation



The public-facing Ubuntu web server is placed inside a dedicated DMZ.




Internet

   |

   | HTTP / HTTPS

  ↓

Firewall

  |

  ↓

DMZ

  |

  ↓

Web Server




The DMZ is prevented from initiating unrestricted connections to internal corporate networks.



### 6. NAT / Port Forwarding



Only required public services are exposed:




WAN:80   - DMZ Web Server:80

WAN:443  - DMZ Web Server:443






---



## Enterprise Problems Solved

| Problem                               | Security Control            | Result                               |
|---------------------------------------|------------------------------|---------------------------------------|
| Flat network                          | Network segmentation         | Reduced lateral movement              |
| Guest access to internal systems      | Guest isolation              | Internal resources protected          |
| Excessive inter‑department access     | Firewall ACLs / rules        | Enforced least privilege              |
| Public web server exposure            | DMZ placement                | Reduced blast radius                  |
| DMZ‑to‑LAN pivoting                   | DMZ isolation                | Prevented internal network pivoting   |
| Unrestricted administrative access    | Dedicated IT/Admin segment   | Centralized and controlled admin ops  |
| Unknown effectiveness of controls     | Kali Linux testing           | Controls validated through attack sims|
| Lack of security evidence             | Firewall logging             | Documented security outcomes          |



---



## Testing \& Validation



Security controls were validated instead of simply being configured.



### Test 1 — HR to Finance



Expected result:





HR - Finance = BLOCKED





Evidence:



* [ICMP test](Screenshots/11-icmp-hr2finance.png)


* [Firewall log](Screenshots/12-firewall-log-hr2finance.png)



### Test 2 — Guest to HR



Expected result:





Guest - HR = BLOCKED





Evidence:



* [Nmap scan](Screenshots/13-nmapscan-guest2HR.png)

* [Firewall log](Screenshots/14-forewall-log-guest2HR.png)



### Test 3 — Guest to Internet

Expected result:

Guest - Internet = ALLOWED

Evidence:
* [guest internet access](Screenshots/15-guest2internet.png)



This confirms that the firewall is enforcing policy rather than simply blocking all traffic.



### Test 4 — WAN to Web Server



Expected result:





WAN - DMZ:80/443 = ALLOWED





The web server should be reachable through the approved web services.





This demonstrates that unnecessary services are not exposed.



### Test 5 — IT Access



Expected result:




IT → required enterprise networks = ALLOWED



Evidence:
* [IT administrative access](Screenshots/16-IT-administrative-access.png)




This validates the administrative access model.



---



## Security Framework Mapping



### NIST Cybersecurity Framework 2.0



| Function | Project Application                                      |
| -------- | -------------------------------------------------------- |
| Govern   | Network security requirements and policy                 |
| Identify | Identification of departments, systems and network zones |
| Protect  | Segmentation, firewall controls, DMZ and least privilege |
| Detect   | Firewall logs and blocked connection monitoring          |
| Respond  | Unauthorized traffic is automatically blocked by policy  |
| Recover  | Not implemented as part of the current project           |



### CIS Controls



**CIS Control 12 — Network Infrastructure Management**



The project applies principles of secure network architecture, segmentation and least privilege.



### NIST SP 800-41 Rev. 1



The project applies firewall policy concepts involving traffic control between networks with different security postures, policy design, configuration and testing.



---



## Tools \& Technologies



* VMware Workstation

* OPNsense 

* Ubuntu Server

* Ubuntu Desktop

* Windows endpoints

* Kali Linux

* Nginx

* DHCP

* DNS

* NAT

* Firewall rules

* ICMP

* Nmap



---



## Evidence



### Network Architecture



[Network Topology](Screenshots/01-topology.png)



### Firewall Interfaces



[Firewall rules](Screenshots/04-firewall-rules.png)



### Guest Isolation



[Guest blocked traffic](Screenshots/09-guest-blocked-traffic.png)



### DMZ Configuration



[DMZ webserver](Screenshots/06-dmz-webserver.png)



### NAT Port Forwarding



[NAT Port Forward](Screenshots/08-nat-portforwarding.png)



### Firewall Logs



[Firewall Logs](Screenshots/05-firewall-logs.png)



---



## Lessons Learned



This project demonstrated that network security is not only about deploying a firewall. Effective security requires defining trust boundaries, limiting communication between systems, applying least privilege and validating that the controls actually enforce the intended policy.



The testing phase was particularly important because it provided evidence that prohibited traffic was actually being blocked while legitimate business traffic remained available.



---



## Future Improvements



* Deploy Suricata IDS/IPS.

* Integrate splunk/wazuh for centralized security monitoring.

* Add VPN-based remote access.

* Implement centralized logging.

* Add vulnerability scanning.

* Add a second firewall for an advanced DMZ architecture.

* Simulate compromised endpoints and investigate lateral-movement attempts.



---



## Security Disclaimer



All scanning and security testing performed in this project was conducted inside an isolated lab environment under my control.



Kali Linux was used only to validate the security controls implemented in the lab.



---



## Project Status



**Completed:** Core segmentation, firewall policy, DMZ isolation, NAT configuration and security validation.



**Future work:** IDS/IPS, centralized monitoring and additional attack/defense simulations.



