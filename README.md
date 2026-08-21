# Lab 01: Inter-VLAN Routing, DHCP, and NAT Overload Setup

## 📌 Project Overview
Lab นี้เป็นการจำลองระบบเครือข่ายองค์กรขนาดเล็กที่ใช้ Cisco Router และ Switch โดยประกอบด้วยการแบ่ง VLAN, การแจก IP แบบอัตโนมัติ (DHCP), การเชื่อมต่อออกอินเทอร์เน็ตผ่าน NAT Overload และการกำหนด Default Route

## 📐 Network Topology
<img width="1920" height="1040" alt="Topology" src="https://github.com/user-attachments/assets/c33c168f-85b2-4c0c-9891-8fa8af14a11f" />

* **VLAN 10:** Admin (`192.168.10.0/24`)
* **VLAN 20:** HR (`192.168.20.0/24`)
* **VLAN 30:** Sales (`192.168.30.0/24`)
* **WAN Link:** `200.1.1.0/30`
* **Internet Server:** `8.8.8.8/24`

## 🛠️ Key Configurations & Concepts Learned
### 🔹 Switch-A Configuration
```text
Current configuration : 2143 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Switch
!
!
!
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/2
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/3
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/4
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/5
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/6
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/7
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/8
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/9
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/10
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/11
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/12
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/13
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/14
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/15
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/16
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/17
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/18
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/19
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/20
 switchport access vlan 200
 switchport mode access
!
interface FastEthernet0/21
!
interface FastEthernet0/22
!
interface FastEthernet0/23
!
interface FastEthernet0/24
!
interface GigabitEthernet0/1
 switchport mode trunk
!
interface GigabitEthernet0/2
!
interface Vlan1
 no ip address
 shutdown
!
!
!
!
line con 0
!
line vty 0 4
 login
line vty 5 15
 login
!
!
!
!
end
```
### 🔹 Router-HQ Configuration
```text
Current configuration : 1407 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Router-HQ
!
!
!
!
ip dhcp excluded-address 172.16.10.1 172.16.10.10
ip dhcp excluded-address 172.16.20.1 172.16.20.10
!
ip dhcp pool VLAN100_POLL
 network 172.16.10.0 255.255.255.0
 default-router 172.16.10.1
 dns-server 8.8.8.8
ip dhcp pool VLAN200_POOL
 network 172.16.20.0 255.255.255.0
 default-router 172.16.20.1
 dns-server 8.8.8.8
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX15248MG2-
!
!
!
!
!
!
!
!
!
!
!
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/0.100
 encapsulation dot1Q 100
 ip address 172.16.10.1 255.255.255.0
 ip nat inside
!
interface GigabitEthernet0/0.200
 encapsulation dot1Q 200
 ip address 172.16.20.1 255.255.255.0
 ip nat inside
!
interface GigabitEthernet0/1
 ip address 100.64.1.1 255.255.255.252
 ip nat outside
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
ip nat inside source list 1 interface GigabitEthernet0/1 overload
ip classless
ip route 0.0.0.0 0.0.0.0 100.64.1.2 
!
ip flow-export version 9
!
!
access-list 1 permit 172.16.0.0 0.0.255.255
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```
* **Inter-VLAN Routing (Router-on-a-Stick):** ใช้ Sub-interface แยก VLAN บนพอร์ตเดียว
* **DHCP Server:** แจก IP ให้ client แต่ละ VLAN พร้อมเว้นช่วง IP สถิต (`excluded-address`)
* **NAT Overload (PAT):** แปลง Private IP เป็น Public IP ขา WAN โดยใช้ **Wildcard Mask (`0.0.255.255`)** เพื่อคุมทุก VLAN ในบรรทัดเดียว
* **Default Route:** ชี้ `0.0.0.0 0.0.0.0` ออกไปยัง Router-ISP (`200.1.1.2`)

## ✅ Verification
ทดสอบ Ping จาก PC ในวง LAN ไปยัง Server ด้านนอก (`8.8.8.8`) ผลลัพธ์คือการส่งแพ็กเก็ตสำเร็จ 100% (TTL=126)

<img width="752" height="570" alt="ping" src="https://github.com/user-attachments/assets/efb605f0-7d4a-463d-aac4-ece3fb61aedf" />
