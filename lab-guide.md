# Enterprise Network Segmentation \& DMZ Security Lab — Lab Guide



This guide documents the technical steps used to build and validate the Enterprise Network Segmentation \& DMZ Security Lab.



---



# 1. Prerequisites





Depending on the Hardware capability Not all VMs need to run simultaneously. VMs that are not being tested can be suspended.



## Required Software



* VMware Workstation

* OPNsense

* Ubuntu Server

* Ubuntu Desktop 

* Kali Linux

* Windows Server

* Windows 11 pro



Download all required software from their official sites



---



# 2. Lab Architecture



The lab contains the following virtual machines and network segments.



| #  | System                  | Role                | Network                 |
| -- | ----------------------- | ------------------- | ----------------------- |
| 1  | OPNsense                | Firewall/Router     | WAN + Internal Networks |
| 2  | IT                      | Administration / DC | 192.168.10.0/24         |
| 3  | Executive Workstation   | Executive           | 192.168.20.0/24         |
| 4  | HR Workstation          | HR                  | 192.168.30.0/24         |
| 5  | Finance Workstation     | Finance             | 192.168.40.0/24         |
| 6  | Sales Workstation       | Sales               | 192.168.50.0/24         |
| 7  | Engineering Workstation | Engineering         | 192.168.60.0/24         |
| 8  | Ubuntu Web Server       | DMZ                 | 192.168.70.0/24         |
| 9  | Guest Device            | Guest/BYOD          | 192.168.80.0/24         |
| 10 | Kali Linux              | Security Testing    | Guest/Testing Networks  |



---



# 3. Network Addressing



| Network     | Subnet          | Gateway      |
| ----------- | ------------    | ---------    |
| IT          | 192.168.10.0/24 | 192.168.10.1 |
| Executive   | 192.168.20.0/24 | 192.168.20.1 |
| HR          | 192.168.30.0/24 | 192.168.30.1 |
| Finance     | 192.168.40.0/24 | 192.168.40.1 |
| Sales       | 192.168.50.0/24 | 192.168.50.1 |
| Engineering | 192.168.60.0/24 | 192.168.60.1 |
| DMZ         | 192.168.70.0/24 | 192.168.70.1 |
| Guest       | 192.168.80.0/24 | 192.168.80.1 |



---



# 4. Phase 1 — Create VMware Virtual Networks



Open:



**VMware Workstation → Edit → Virtual Network Editor**



Select:



**Change Settings**



Create the required host-only networks.



```text

VMnet2

VMnet3

VMnet4

VMnet5

VMnet6

VMnet7

VMnet9

VMnet10

```



Leave:



```text

VMnet8 = NAT / WAN

```



For each internal VMnet:



* Set the network type to **Host-only**

* Disable VMware's local DHCP service

* Allow OPNsense to provide DHCP



This prevents two DHCP servers from operating on the same segment.



---



# 5. Phase 2 — Deploy OPNsense



Create a new virtual machine.



starting configuration:



* 1–2 vCPU

* 1–2 GB RAM

* 8–20 GB disk

* FreeBSD 64-bit or equivalent

* OPNsense ISO attached



---



## Network Adapters



Configure the OPNsense VM with:



```text

Adapter 1 → NAT / VMnet8 → WAN



Adapter 2 → VMnet2 → IT

Adapter 3 → VMnet3 → Executive

Adapter 4 → VMnet4 → HR

Adapter 5 → VMnet5 → Finance

Adapter 6 → VMnet6 → Sales

Adapter 7 → VMnet7 → Engineering

Adapter 8 → VMnet9 → DMZ

Adapter 9 → VMnet10 → Guest

```



The exact interface names may appear as `emX` or `vtnetX`.



---



# 6. Phase 3 — Assign OPNsense Interfaces



Boot OPNsense and assign the network interfaces.



Assign:



```text

WAN

IT 

Executive

HR

Finance

Sales

Engineering

DMZ

Guest



```



Configure the internal gateways:



```text

IT            192.168.10.1       

Executive     192.168.20.1   

HR            192.168.30.1

Finance       192.168.40.1

Sales         192.168.50.1

Engineering   192.168.60.1      

DMZ           192.168.70.1

Guest         192.168.80.1

```



---



# 7. Phase 4 — Access the OPNsense Web Interface



Connect a VM to the IT  network.



Open:



```text

https:// 192.168.10.1

```



Log in using the credentials configured during installation.



Complete the initial configuration wizard.



---



# 8. Phase 5 — Configure DHCP



Configure DHCP for each internal interface.



Example for HR:



```text

Network:    192.168.20.0/24

Gateway:    192.168.20.1

DHCP Range: 192.168.20.100 – 192.168.20.200

```

**NOTE: The IT network  will get a static IP because it is hosting active directory and you want the other clients to locate active directory domain.**



Repeat for the other internal networks.



Verify that each endpoint receives an address from the correct subnet.



Example:



```text

HR workstation       → 192.168.30.x

Finance workstation  → 192.168.40.x

Guest workstation    → 192.168.80.x

```



---



# 9. Phase 6 — Configure DNS



Configure the OPNsense DNS resolver if required.



For this environments, i am using Active Directory,so i  configured DNS so that clients can resolve the domain controller and required services.



Verify DNS resolution from the endpoints.



---



# 10. Phase 7 — Deploy Department VMs



Create the department VMs.



Each VM should have one network adapter connected to its assigned VMware network.



Examples:



```text

HR-WS01       → HR VMnet

FIN-WS01      → Finance VMnet

SALES-WS01    → Sales VMnet

ENG-WS01      → Engineering VMnet

EXEC-WS01     → Executive VMnet

GUEST-WS01    → Guest VMnet

```



Verify that each machine receives an IP address from the correct subnet.



---



# 11. Phase 8 — Configure IT



install Windows Server and configure it as a domain controller.



Create the required organizational units and join selected endpoints to the domain.







Verify:



* DNS resolution

* Domain connectivity

* Authentication

* Required AD services



---



# 12. Phase 9 — Deploy the DMZ Web Server



Create an Ubuntu Server VM.



Connect its network adapter to the DMZ VMnet.



Configure an address in:



```text

192.168.70.0/24

```



Example:



```text

IP:       192.168.70.150

Gateway:  192.168.70.1

```



Install Nginx:



```bash

sudo apt update

sudo apt install nginx

```



Check the service:



```bash

sudo systemctl status nginx

```



Test from the DMZ:



```text

http://192.168.70.150

```



---



# 13. Phase 10 — Configure Firewall Rules



Configure the firewall using a deny-by-default approach.



Only required traffic should be explicitly permitted.



## IT



Configure the required administrative access.



```text

IT → Required Networks = ALLOW

IT → Internet = ALLOW

```



## Department Networks



Allow Internet access and required internal services.



Example:



```text

HR → Internet = ALLOW

HR → Required IT/DNS/AD services = ALLOW

```



Apply equivalent rules to the other department networks according to the required access.



---



# 14. Phase 11 — Configure Guest Isolation



Configure:



```text

Guest → Internet = ALLOW

```



Do not create rules allowing Guest access to internal corporate networks.



Where applicable, enable:



```text

Block private networks

Block bogon networks

```



for this lab, i enabled a rule that block guest from internal network



Verify that the Guest network cannot communicate with:



```text

IT

HR

Finance

Sales

Engineering

Executive

```



---



# 15. Phase 12 — Configure DMZ Rules



Allow the required web traffic:



```text

WAN → DMZ TCP/80  = ALLOW

WAN → DMZ TCP/443 = ALLOW

```



Do not expose unnecessary services.



For example:



```text

WAN → DMZ TCP/22 = BLOCK

```



Do not create rules allowing unrestricted DMZ access to internal networks.



---



# 16. Phase 13 — Configure NAT / Port Forwarding



Create the HTTP port forward:



```text

WAN TCP/80

&#x20;     ↓

DMZ Web Server TCP/80

```



Create the HTTPS port forward:



```text

WAN TCP/443

   ↓

DMZ Web Server TCP/443

```



Do not create an SSH port forward unless it is specifically required for an approved lab test.



---



# 17. Phase 14 — Test Basic Connectivity



Before testing security controls, confirm that the networks are functioning.



From a workstation:



```bash

ip addr

```



Verify the assigned subnet.



Test the gateway:



```bash

ping <gateway-IP>

```



Examples:



```bash

ping 192.168.20.1

ping 192.168.30.1

ping 192.168.40.1

```



---



# 18. Phase 15 — Test Department Isolation



## HR → Finance



From HR:



```bash

ping 192.168.40.x

```



Expected:



```text

BLOCKED

```



Check the firewall logs for the blocked connection.



---



## HR → IT



Test the required IT/Admin connectivity:



```bash

ping 192.168.10.x

```



Expected:



```text

ALLOWED

```



if ICMP has been permitted by the firewall policy.



---



# 19. Phase 16 — Test Guest Isolation with Kali



Connect Kali Linux to the Guest network.



Check its address:



```bash

ip addr

```



Scan the Finance subnet:



```bash

nmap -sn 192.168.40.x/24

```



Expected:



```text

Finance network should not be reachable from Guest.

```



Review the firewall logs to verify the blocked traffic.



---



# 20. Phase 17 — Test Guest Internet Access



From Kali or the Guest workstation:



```bash

ping 8.8.8.8

```



or browse to an Internet website.



Expected:



```text

Guest → Internet = ALLOWED

```



---



# 21. Phase 18 — Test WAN → DMZ



From the appropriate external/WAN test position, browse to:



```text

http://<WAN-IP>

```



Expected:



```text

DMZ web page loads

```



If HTTPS is configured:



```text

https://<WAN-IP>

```



Expected:



```text

HTTPS connection succeeds

```



---



# 22. Phase 19 — optional Test Unauthorized WAN Services



Test an unapproved service such as SSH:



```bash

ssh <WAN-IP>

```



Expected:



```text

Connection blocked

```



This verifies that unnecessary services are not exposed.



---



# 23. Phase 20 — Review Firewall Logs



Open the OPNsense firewall log/live view.



Run the security tests while monitoring the logs.



Check:



* Source IP

* Destination IP

* Protocol

* Source port

* Destination port

* Action

* Interface

* Timestamp



Capture screenshots of relevant blocked and permitted traffic.



---



# 24. Phase 21 — Evidence Collection



Capture evidence for the project documentation.



## Infrastructure



* VMware VM configuration

* VMware Virtual Network Editor

* OPNsense interface assignments

* IP addressing



## Firewall



* Department firewall rules

* Guest rules

* DMZ rules

* NAT/port forwarding



## DMZ



* Ubuntu server IP

* Nginx status

* Web page

* WAN → DMZ access



## Testing



* HR → Finance blocked

* Guest → Finance blocked

* Guest → Internet allowed

* WAN → Web Server allowed

* WAN → SSH blocked

* Firewall log evidence



---



# 25. Final Validation Checklist



Use this checklist after completing the lab.



* [ ] OPNsense WAN is operational

* [ ] All internal interfaces are configured

* [ ] DHCP is working

* [ ] DNS is working

* [ ] Department VMs receive correct IP addresses

* [ ] IT/Admin access works

* [ ] HR → Finance is blocked

* [ ] Guest → Internal networks is blocked

* [ ] Guest → Internet works

* [ ] WAN → HTTP works

* [ ] WAN → HTTPS works

* [ ] WAN → SSH is blocked

* [ ] DMZ → Internal networks is blocked

* [ ] Firewall logs show blocked traffic

* [ ] Required screenshots have been captured



---



# 26. Troubleshooting



## VM has no network connection



Check:



```text

VM Settings

→ Network Adapter

→ Correct VMnet

```



Make sure the VM is not accidentally connected to NAT or Bridged networking.



---



## Firewall has no Internet access



Verify:



```text

OPNsense WAN

     ↓

VMnet8

     ↓

VMware NAT

```



Confirm that the WAN adapter is connected to the correct VMware network.



---



## DHCP is not working



Verify that VMware's DHCP service is disabled on the host-only network.



OPNsense should be the DHCP server for the internal segment.



---



## Traffic is unexpectedly blocked



Check:



1. Source interface

2. Destination IP

3. Destination port

4. Firewall rule order

5. Gateway

6. NAT configuration

7. Firewall logs



---



## Web server is unreachable



Check:



```text

1. Web server IP

2. Default gateway

3. Nginx status

4. DMZ firewall rules

5. NAT/port-forward rule

6. WAN firewall rule

7. Firewall logs

```



---



# 27. Lab Safety



All security testing must remain inside the isolated laboratory environment.



Kali Linux and Nmap should only be used against systems and networks that you own or have explicit authorization to test.



Do not scan public IP addresses or external networks without authorization.



