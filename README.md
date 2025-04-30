# 🔐 Site-to-Site IPSec VPN Lab (Cisco 1921 & 2921)

This lab demonstrates a secure **Site-to-Site IPSec VPN** tunnel between two Cisco routers (1921 & 2921), allowing encrypted communication between two remote LANs.

---

## 🧭 Lab Objective

- Create an encrypted tunnel over the public WAN
- Securely connect:
  - **R1 LAN:** 192.168.10.0/24
  - **R2 LAN:** 192.168.20.0/24
- Use **ISAKMP (IKEv1)** and **IPSec ESP**
---

## 🔧 Router Configurations

### 🔹 R1 (Cisco 1921)

## hostname R1
## !
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
## !
interface GigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
## !
ip access-list extended VPN-TRAFFIC
 permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
## !
crypto isakmp policy 10
 encryption aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
## !
crypto isakmp key VPNKEY address 10.0.0.2
## !
crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
 mode tunnel
## !
crypto map VPNMAP 10 ipsec-isakmp 
 set peer 10.0.0.2
 set transform-set VPN-SET
 match address VPN-TRAFFIC
## !
interface GigabitEthernet0/1
 crypto map VPNMAP
## !
ip route 192.168.20.0 255.255.255.0 10.0.0.2



## hostname R2
!
interface GigabitEthernet0/0
 ip address 192.168.20.1 255.255.255.0
 no shutdown
## !
interface GigabitEthernet0/1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
## !
ip access-list extended VPN-TRAFFIC
 permit ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
## !
crypto isakmp policy 10
 encryption aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
## !
crypto isakmp key VPNKEY address 10.0.0.1
## !
crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
 mode tunnel
## !
crypto map VPNMAP 10 ipsec-isakmp 
 set peer 10.0.0.1
 set transform-set VPN-SET
 match address VPN-TRAFFIC
## !
interface GigabitEthernet0/1
 crypto map VPNMAP
## !
ip route 192.168.10.0 255.255.255.0 10.0.0.1
## !
 📌 Notes
VPN traffic is encrypted using AES with SHA authentication
NAT is not configured in this lab; traffic is routed directly
This configuration is suitable for lab simulations and portfolio projects 

## ✅ Status:
Tunnel successfully established. Ping verified across LANs.

⚠️ Real-World Observation: Tunnel Did Not Initially Trigger
In this lab, the IPSec VPN tunnel did not immediately become active after the configuration was completed.

🧪 Issue
The tunnel was not triggered (inactive) until we manually initiated interesting traffic (such as a ping) from R1's LAN to R2's LAN.

✅ Resolution
After executing the following command:

 R1# ping 192.168.20.1 source 192.168.10.1

The VPN tunnel was established, and the show crypto isakmp sa and show crypto ipsec sa outputs confirmed the tunnel was ACTIVE (QM_IDLE).

🧠 Explanation
This is expected behavior in many policy-based IPSec VPN setups:

IPSec does not bring the tunnel up automatically; instead, it waits for "interesting traffic" defined by the access list (in our case, VPN-TRAFFIC).

When such traffic is detected, the ISAKMP Phase 1 and then IPSec Phase 2 negotiations begin, forming the encrypted tunnel.

📌 Takeaway
Always remember:
"In policy-based VPNs, no interesting traffic = no tunnel."
This is a key behavior to know when troubleshooting real-world VPN issues.

## Contact 
https://github.com/IlkinNureddinov 
