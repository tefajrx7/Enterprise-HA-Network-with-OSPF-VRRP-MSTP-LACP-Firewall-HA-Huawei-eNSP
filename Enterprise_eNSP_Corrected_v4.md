# Enterprise HA Network with OSPF, VRRP, MSTP, LACP & Firewall HA — Huawei eNSP

------------------------------------------------------------------------
------------------------------------------------------------------------

# PHASE 1 — HQ L2 switch + HQ routers

## 4.1 LSW-HQ (L2 فقط)


system-view
sysname LSW-HQ
vlan batch 900

interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 900
quit

interface GigabitEthernet0/0/2
 port link-type access
 port default vlan 900
quit

interface GigabitEthernet0/0/3
 port link-type access
 port default vlan 900
quit

interface GigabitEthernet0/0/4
 port link-type access
 port default vlan 900
quit

interface GigabitEthernet0/0/5
 port link-type access
 port default vlan 900
quit

return
save


## 4.2 HQ-1

``` text
system-view
sysname HQ-1

interface LoopBack0
 ip address 10.255.15.15 255.255.255.255
quit

interface GigabitEthernet0/0/0
 ip address 10.254.254.2 255.255.255.240
 vrrp vrid 254 virtual-ip 10.254.254.1
 vrrp vrid 254 priority 150
 vrrp vrid 254 preempt-mode timer delay 20
quit

interface Serial3/0/0
 ip address 10.255.101.1 255.255.255.252
quit

ip route-static 10.0.0.0 255.255.0.0 10.254.254.7
ip route-static 10.254.0.0 255.255.255.0 10.254.254.7
ip route-static 10.10.10.0 255.255.255.0 10.254.254.7

ospf 1 router-id 10.255.15.15
 import-route static
 area 0.0.0.0
  network 10.255.15.15 0.0.0.0
  network 10.254.254.0 0.0.0.15
  network 10.255.101.0 0.0.0.3
 quit
quit

return
save


## 4.3 HQ-2


system-view
sysname HQ-2

interface LoopBack0
 ip address 10.255.14.14 255.255.255.255
quit

interface GigabitEthernet0/0/0
 ip address 10.254.254.3 255.255.255.240
 vrrp vrid 254 virtual-ip 10.254.254.1
 vrrp vrid 254 priority 130
 vrrp vrid 254 preempt-mode timer delay 20
quit

interface Serial3/0/0
 ip address 10.255.102.1 255.255.255.252
quit

ip route-static 10.0.0.0 255.255.0.0 10.254.254.7
ip route-static 10.254.0.0 255.255.255.0 10.254.254.7
ip route-static 10.10.10.0 255.255.255.0 10.254.254.7

ospf 1 router-id 10.255.14.14
 import-route static
 area 0.0.0.0
  network 10.255.14.14 0.0.0.0
  network 10.254.254.0 0.0.0.15
  network 10.255.102.0 0.0.0.3
 quit
quit

return
save


## 4.4 HQ-3


system-view
sysname HQ-3

interface LoopBack0
 ip address 10.255.13.13 255.255.255.255
quit

interface GigabitEthernet0/0/0
 ip address 10.254.254.4 255.255.255.240
 vrrp vrid 254 virtual-ip 10.254.254.1
 vrrp vrid 254 priority 110
 vrrp vrid 254 preempt-mode timer delay 20
quit

interface Serial3/0/0
 ip address 10.255.103.1 255.255.255.252
quit

ip route-static 10.0.0.0 255.255.0.0 10.254.254.7
ip route-static 10.254.0.0 255.255.255.0 10.254.254.7
ip route-static 10.10.10.0 255.255.255.0 10.254.254.7

ospf 1 router-id 10.255.13.13
 import-route static
 area 0.0.0.0
  network 10.255.13.13 0.0.0.0
  network 10.254.254.0 0.0.0.15
  network 10.255.103.0 0.0.0.3
 quit
quit

return
save


**اختبار:** `display vrrp brief` على الثلاثة → HQ-1 = Master. `display ospf peer brief` → الجيران على `10.254.254.0/28` لازم Full (والـ ISP لسه ما اتعملش، عادي).

------------------------------------------------------------------------

# PHASE 2 — HQ core switches (LSW-HQ-1 / LSW-HQ-2)

## 5.1 LSW-HQ-1


system-view
sysname LSW-HQ-1

dhcp enable
vlan batch 10 20 30 40 50 60 70 80 90 100 110 120 130 140 901

# --- Inter-switch Eth-Trunk1 (3 links) 
interface Eth-Trunk1
 mode lacp-static
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140 901
quit
interface GigabitEthernet0/0/4
 eth-trunk 1
quit
interface GigabitEthernet0/0/5
 eth-trunk 1
quit
interface GigabitEthernet0/0/6
 eth-trunk 1
quit

# --- Firewall-facing Eth-Trunk2 (2 links) 
interface Eth-Trunk2
 mode lacp-static
 port link-type access
 port default vlan 901
quit
interface GigabitEthernet0/0/1
 eth-trunk 2
quit
interface GigabitEthernet0/0/2
 eth-trunk 2
quit

# --- Access uplinks
interface GigabitEthernet0/0/7
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit
interface GigabitEthernet0/0/8
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit
interface GigabitEthernet0/0/9
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit
interface GigabitEthernet0/0/10
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit
interface GigabitEthernet0/0/11
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit

# --- MSTP ---
stp mode mstp
stp region-configuration
 region-name ENTERPRISE
 revision-level 1
 instance 1 vlan 10 30 50 70 90 110 130
 instance 2 vlan 20 40 60 80 100 120 140
 active region-configuration
quit
stp root primary
stp instance 1 root primary
stp instance 2 root secondary

# --- Firewall transit Vlanif (VRRP VIP 10.254.0.1 = للسويتشات بس) ---
interface Vlanif901
 ip address 10.254.0.4 255.255.255.0
 vrrp vrid 150 virtual-ip 10.254.0.1
 vrrp vrid 150 priority 120
quit

# --- User gateways + DHCP relay ---
interface Vlanif10
 ip address 10.0.10.2 255.255.255.0
 vrrp vrid 10 virtual-ip 10.0.10.1
 vrrp vrid 10 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif20
 ip address 10.0.20.2 255.255.255.0
 vrrp vrid 20 virtual-ip 10.0.20.1
 vrrp vrid 20 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif30
 ip address 10.0.30.2 255.255.255.0
 vrrp vrid 30 virtual-ip 10.0.30.1
 vrrp vrid 30 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif40
 ip address 10.0.40.2 255.255.255.0
 vrrp vrid 40 virtual-ip 10.0.40.1
 vrrp vrid 40 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif50
 ip address 10.0.50.2 255.255.255.0
 vrrp vrid 50 virtual-ip 10.0.50.1
 vrrp vrid 50 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif60
 ip address 10.0.60.2 255.255.255.0
 vrrp vrid 60 virtual-ip 10.0.60.1
 vrrp vrid 60 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif70
 ip address 10.0.70.2 255.255.255.0
 vrrp vrid 70 virtual-ip 10.0.70.1
 vrrp vrid 70 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif80
 ip address 10.0.80.2 255.255.255.0
 vrrp vrid 80 virtual-ip 10.0.80.1
 vrrp vrid 80 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif90
 ip address 10.0.90.2 255.255.255.0
 vrrp vrid 90 virtual-ip 10.0.90.1
 vrrp vrid 90 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif100
 ip address 10.0.100.2 255.255.255.0
 vrrp vrid 100 virtual-ip 10.0.100.1
 vrrp vrid 100 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif110
 ip address 10.0.110.2 255.255.255.0
 vrrp vrid 110 virtual-ip 10.0.110.1
 vrrp vrid 110 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif120
 ip address 10.0.120.2 255.255.255.0
 vrrp vrid 120 virtual-ip 10.0.120.1
 vrrp vrid 120 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif130
 ip address 10.0.130.2 255.255.255.0
 vrrp vrid 130 virtual-ip 10.0.130.1
 vrrp vrid 130 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif140
 ip address 10.0.140.2 255.255.255.0
 vrrp vrid 140 virtual-ip 10.0.140.1
 vrrp vrid 140 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit

return
save


## 5.2 LSW-HQ-2


system-view
sysname LSW-HQ-2

dhcp enable
vlan batch 10 20 30 40 50 60 70 80 90 100 110 120 130 140 901

# --- Inter-switch Eth-Trunk1 (3 links) ---
interface Eth-Trunk1
 mode lacp-static
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140 901
quit
interface GigabitEthernet0/0/4
 eth-trunk 1
quit
interface GigabitEthernet0/0/5
 eth-trunk 1
quit
interface GigabitEthernet0/0/6
 eth-trunk 1
quit

# --- Firewall-facing Eth-Trunk2 (2 links) ---
interface Eth-Trunk2
 mode lacp-static
 port link-type access
 port default vlan 901
quit
interface GigabitEthernet0/0/1
 eth-trunk 2
quit
interface GigabitEthernet0/0/2
 eth-trunk 2
quit

# --- Access uplinks ---
interface GigabitEthernet0/0/7
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit
interface GigabitEthernet0/0/8
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit
interface GigabitEthernet0/0/9
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit
interface GigabitEthernet0/0/10
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit
interface GigabitEthernet0/0/11
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100 110 120 130 140
quit

# --- MSTP ---
stp mode mstp
stp region-configuration
 region-name ENTERPRISE
 revision-level 1
 instance 1 vlan 10 30 50 70 90 110 130
 instance 2 vlan 20 40 60 80 100 120 140
 active region-configuration
quit
stp root secondary
stp instance 1 root secondary
stp instance 2 root primary

# --- Firewall transit Vlanif (VRRP VIP 10.254.0.1 = للسويتشات بس) ---
interface Vlanif901
 ip address 10.254.0.5 255.255.255.0
 vrrp vrid 150 virtual-ip 10.254.0.1
 vrrp vrid 150 priority 100
quit

# --- User gateways + DHCP relay ---
interface Vlanif10
 ip address 10.0.10.3 255.255.255.0
 vrrp vrid 10 virtual-ip 10.0.10.1
 vrrp vrid 10 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif20
 ip address 10.0.20.3 255.255.255.0
 vrrp vrid 20 virtual-ip 10.0.20.1
 vrrp vrid 20 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif30
 ip address 10.0.30.3 255.255.255.0
 vrrp vrid 30 virtual-ip 10.0.30.1
 vrrp vrid 30 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif40
 ip address 10.0.40.3 255.255.255.0
 vrrp vrid 40 virtual-ip 10.0.40.1
 vrrp vrid 40 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif50
 ip address 10.0.50.3 255.255.255.0
 vrrp vrid 50 virtual-ip 10.0.50.1
 vrrp vrid 50 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif60
 ip address 10.0.60.3 255.255.255.0
 vrrp vrid 60 virtual-ip 10.0.60.1
 vrrp vrid 60 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif70
 ip address 10.0.70.3 255.255.255.0
 vrrp vrid 70 virtual-ip 10.0.70.1
 vrrp vrid 70 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif80
 ip address 10.0.80.3 255.255.255.0
 vrrp vrid 80 virtual-ip 10.0.80.1
 vrrp vrid 80 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif90
 ip address 10.0.90.3 255.255.255.0
 vrrp vrid 90 virtual-ip 10.0.90.1
 vrrp vrid 90 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif100
 ip address 10.0.100.3 255.255.255.0
 vrrp vrid 100 virtual-ip 10.0.100.1
 vrrp vrid 100 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif110
 ip address 10.0.110.3 255.255.255.0
 vrrp vrid 110 virtual-ip 10.0.110.1
 vrrp vrid 110 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif120
 ip address 10.0.120.3 255.255.255.0
 vrrp vrid 120 virtual-ip 10.0.120.1
 vrrp vrid 120 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif130
 ip address 10.0.130.3 255.255.255.0
 vrrp vrid 130 virtual-ip 10.0.130.1
 vrrp vrid 130 priority 100
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit
interface Vlanif140
 ip address 10.0.140.3 255.255.255.0
 vrrp vrid 140 virtual-ip 10.0.140.1
 vrrp vrid 140 priority 120
 dhcp select relay
 dhcp relay server-ip 10.255.15.15
quit

return
save


> الـ default route على السويتشين **مش هنا**: بيتضاف في Phase 5 بعد ما الـ Firewall HA يقوم.

**اختبار:** `display eth-trunk 1` (3 members = Selected)، `display vrrp brief` (LSW-HQ-1 Master للـ VLANs الفردية + 901، LSW-HQ-2 Master للزوجية)، `display stp brief`.

------------------------------------------------------------------------

# PHASE 3 — HQ access switches

كل access switch: **`GE0/0/1` و `GE0/0/2` = uplinks للـ core (trunk)**، وباقي الـ trunks = interlinks على `Ethernet`.

## 6.1 HQ-LSW1


system-view
sysname HQ-LSW1
vlan batch 10 20 30 40 50 60 70 80 90 100

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface Ethernet0/0/3
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit

interface Ethernet0/0/1
 port link-type access
 port default vlan 10
 stp edged-port enable
quit
interface Ethernet0/0/2
 port link-type access
 port default vlan 20
 stp edged-port enable
quit

# --- MSTP (نفس الـ region بتاع الـ core بالحرف) ---
stp mode mstp
stp region-configuration
 region-name ENTERPRISE
 revision-level 1
 instance 1 vlan 10 30 50 70 90 110 130
 instance 2 vlan 20 40 60 80 100 120 140
 active region-configuration
quit
stp bpdu-protection

return
save


## 6.2 HQ-LSW2


system-view
sysname HQ-LSW2
vlan batch 10 20 30 40 50 60 70 80 90 100

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface Ethernet0/0/4
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface Ethernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit

interface Ethernet0/0/2
 port link-type access
 port default vlan 40
 stp edged-port enable
quit
interface Ethernet0/0/3
 port link-type access
 port default vlan 30
 stp edged-port enable
quit

# --- MSTP (نفس الـ region بتاع الـ core بالحرف) ---
stp mode mstp
stp region-configuration
 region-name ENTERPRISE
 revision-level 1
 instance 1 vlan 10 30 50 70 90 110 130
 instance 2 vlan 20 40 60 80 100 120 140
 active region-configuration
quit
stp bpdu-protection

return
save


## 6.3 HQ-LSW3


system-view
sysname HQ-LSW3
vlan batch 10 20 30 40 50 60 70 80 90 100

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface Ethernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface Ethernet0/0/4
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit

interface Ethernet0/0/2
 port link-type access
 port default vlan 60
 stp edged-port enable
quit
interface Ethernet0/0/3
 port link-type access
 port default vlan 50
 stp edged-port enable
quit

# --- MSTP (نفس الـ region بتاع الـ core بالحرف) ---
stp mode mstp
stp region-configuration
 region-name ENTERPRISE
 revision-level 1
 instance 1 vlan 10 30 50 70 90 110 130
 instance 2 vlan 20 40 60 80 100 120 140
 active region-configuration
quit
stp bpdu-protection

return
save


## 6.4 HQ-LSW4


system-view
sysname HQ-LSW4
vlan batch 10 20 30 40 50 60 70 80 90 100

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface Ethernet0/0/4
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface Ethernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit

interface Ethernet0/0/2
 port link-type access
 port default vlan 80
 stp edged-port enable
quit
interface Ethernet0/0/3
 port link-type access
 port default vlan 70
 stp edged-port enable
quit

# --- MSTP (نفس الـ region بتاع الـ core بالحرف) ---
stp mode mstp
stp region-configuration
 region-name ENTERPRISE
 revision-level 1
 instance 1 vlan 10 30 50 70 90 110 130
 instance 2 vlan 20 40 60 80 100 120 140
 active region-configuration
quit
stp bpdu-protection

return
save


## 6.5 HQ-LSW5


system-view
sysname HQ-LSW5
vlan batch 10 20 30 40 50 60 70 80 90 100

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit
interface Ethernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80 90 100
quit

interface Ethernet0/0/2
 port link-type access
 port default vlan 90
 stp edged-port enable
quit
interface Ethernet0/0/3
 port link-type access
 port default vlan 100
 stp edged-port enable
quit

# --- MSTP (نفس الـ region بتاع الـ core بالحرف) ---
stp mode mstp
stp region-configuration
 region-name ENTERPRISE
 revision-level 1
 instance 1 vlan 10 30 50 70 90 110 130
 instance 2 vlan 20 40 60 80 100 120 140
 active region-configuration
quit
stp bpdu-protection

return
save


**اختبار (مهم قبل ما تكمل):**

```text
PC6:  ip 10.0.10.10 255.255.255.0 gw 10.0.10.1   ← يدوي
ping 10.0.10.1   ping 10.0.10.2   ping 10.0.10.3
```
لو الثلاثة نجحوا → الـ access layer سليم. بعدها `display stp brief` على LSW-HQ-1 (لازم Root) وقارن `display stp region-configuration` (الـ digest لازم يتطابق مع الـ access switches).

## 6.6  HQ — VLAN 110 (Wireless clients) + تحويل ports الـ APs إلى trunk

> VLAN 110 موجودة أصلًا على الـ core (Vlanif110 + VRRP + relay + `HQ_VLAN110` على HQ-1، وuplinks الـ core بتسمح بيها). الناقص إنها توصل لسويتشات الـ access وports الـ APs.
> نفذ ده **بعد** 6.1 → 6.5.
> الـ AP بيفضل ياخد IP إدارته من VLAN بتاعه (20/40/60/80/100) untagged، والـ clients بتعدّي tagged على VLAN 110.

**HQ-LSW1** (AP على `E0/0/2`، VLAN إدارة 20):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/3
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/2
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 20
 port trunk allow-pass vlan 20 110
quit
return
save


**HQ-LSW2** (AP على `E0/0/2`، VLAN إدارة 40):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/1
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/4
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/2
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 40
 port trunk allow-pass vlan 40 110
quit
return
save


**HQ-LSW3** (AP على `E0/0/2`، VLAN إدارة 60):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/1
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/4
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/2
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 60
 port trunk allow-pass vlan 60 110
quit
return
save


**HQ-LSW4** (AP على `E0/0/2`، VLAN إدارة 80):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/1
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/4
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/2
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 80
 port trunk allow-pass vlan 80 110
quit
return
save


**HQ-LSW5** (AP على `E0/0/3`، VLAN إدارة 100):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/1
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/3
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 100
 port trunk allow-pass vlan 100 110
quit
return
save


------------------------------------------------------------------------

# PHASE 4 — DHCP server (HQ-1)


system-view
dhcp enable

# --- لازم على كل interface ممكن يوصلها طلب الـ relay ---
interface GigabitEthernet0/0/0
 dhcp select global
quit
interface Serial3/0/0
 dhcp select global
quit
interface LoopBack0
 dhcp select global
quit

ip pool HQ_VLAN10
 gateway-list 10.0.10.1
 network 10.0.10.0 mask 255.255.255.0
 excluded-ip-address 10.0.10.1 10.0.10.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN20
 gateway-list 10.0.20.1
 network 10.0.20.0 mask 255.255.255.0
 excluded-ip-address 10.0.20.1 10.0.20.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN30
 gateway-list 10.0.30.1
 network 10.0.30.0 mask 255.255.255.0
 excluded-ip-address 10.0.30.1 10.0.30.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN40
 gateway-list 10.0.40.1
 network 10.0.40.0 mask 255.255.255.0
 excluded-ip-address 10.0.40.1 10.0.40.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN50
 gateway-list 10.0.50.1
 network 10.0.50.0 mask 255.255.255.0
 excluded-ip-address 10.0.50.1 10.0.50.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN60
 gateway-list 10.0.60.1
 network 10.0.60.0 mask 255.255.255.0
 excluded-ip-address 10.0.60.1 10.0.60.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN70
 gateway-list 10.0.70.1
 network 10.0.70.0 mask 255.255.255.0
 excluded-ip-address 10.0.70.1 10.0.70.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN80
 gateway-list 10.0.80.1
 network 10.0.80.0 mask 255.255.255.0
 excluded-ip-address 10.0.80.1 10.0.80.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN90
 gateway-list 10.0.90.1
 network 10.0.90.0 mask 255.255.255.0
 excluded-ip-address 10.0.90.1 10.0.90.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN100
 gateway-list 10.0.100.1
 network 10.0.100.0 mask 255.255.255.0
 excluded-ip-address 10.0.100.1 10.0.100.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN110
 gateway-list 10.0.110.1
 network 10.0.110.0 mask 255.255.255.0
 excluded-ip-address 10.0.110.1 10.0.110.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN120
 gateway-list 10.0.120.1
 network 10.0.120.0 mask 255.255.255.0
 excluded-ip-address 10.0.120.1 10.0.120.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN130
 gateway-list 10.0.130.1
 network 10.0.130.0 mask 255.255.255.0
 excluded-ip-address 10.0.130.1 10.0.130.20
 dns-list 8.8.8.8 1.1.1.1
quit
ip pool HQ_VLAN140
 gateway-list 10.0.140.1
 network 10.0.140.0 mask 255.255.255.0
 excluded-ip-address 10.0.140.1 10.0.140.20
 dns-list 8.8.8.8 1.1.1.1
quit

return
save


> لو `dhcp select global` اترفض على `LoopBack0` (Unrecognized command) تجاهله، الاتنين التانيين كفاية غالبًا. لو الـ DHCP لسه مش شغال بعد Phase 5 شوف قسم 8 (Fallback).

## 4.5 🆕 Option 43 على HQ-1 (عشان APs الـ HQ تعرف عنوان AC-HQ)

> بنعمل `ip pool` تاني بنفس الأسماء (مفيش تعديل على اللي فات، بس بنضيف Option). ده للـ VLANs اللي عليها APs فقط: 20 / 40 / 60 / 80 / 100.


system-view

ip pool HQ_VLAN20
 option 43 sub-option 3 ascii 10.10.10.20
quit
ip pool HQ_VLAN40
 option 43 sub-option 3 ascii 10.10.10.20
quit
ip pool HQ_VLAN60
 option 43 sub-option 3 ascii 10.10.10.20
quit
ip pool HQ_VLAN80
 option 43 sub-option 3 ascii 10.10.10.20
quit
ip pool HQ_VLAN100
 option 43 sub-option 3 ascii 10.10.10.20
quit

return
save


------------------------------------------------------------------------

# PHASE 5 — HQ DMZ switch + HQ Firewalls (HA)

## 7.1 LSW-HQ-DMZ


system-view
sysname LSW-HQ-DMZ
vlan 300

interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 300
quit
interface GigabitEthernet0/0/2
 port link-type access
 port default vlan 300
quit
interface GigabitEthernet0/0/3
 port link-type access
 port default vlan 300
quit
interface GigabitEthernet0/0/4
 port link-type access
 port default vlan 300
quit
interface GigabitEthernet0/0/5
 port link-type access
 port default vlan 300
quit

return
save


Servers (من eNSP → Server → Basic Config): `Server1 10.10.10.10/24` ، `Server2 10.10.10.11/24` ، `Server3 10.10.10.12/24` ، Gateway `10.10.10.1`.

## 7.2 🆕 LSW-HQ-DMZ — port الـ AC-HQ


system-view
interface GigabitEthernet0/0/6
 port link-type access
 port default vlan 300
quit
return
save


الـ AC-HQ هيتظبط في Phase 9 (W.2) بعد ما الشبكة كلها تشتغل.

## 8.1 FW-HQ-1 — base (من غير HRP)


system-view
sysname FW-HQ-1

# --- (1) الأول: فك الـ management binding عن GE0/0/0 ---
interface GigabitEthernet0/0/0
 undo ip binding vpn-instance default
 ip address 10.254.254.5 255.255.255.240
 service-manage enable
 service-manage ping permit
quit

interface GigabitEthernet1/0/0
 ip address 10.255.254.1 255.255.255.252
 service-manage enable
 service-manage ping permit
quit

# --- (2) Inside Eth-Trunk (2 links) ---
interface Eth-Trunk1
 mode lacp-static
 ip address 10.254.0.2 255.255.255.0
 service-manage enable
 service-manage ping permit
quit
interface GigabitEthernet1/0/1
 eth-trunk 1
quit
interface GigabitEthernet1/0/2
 eth-trunk 1
quit

# --- (3) DMZ ---
interface GigabitEthernet1/0/4
 ip address 10.10.10.2 255.255.255.0
 service-manage enable
 service-manage ping permit
quit

# --- (4) Zones ---
firewall zone untrust
 set priority 5
 add interface GigabitEthernet0/0/0
quit
firewall zone trust
 set priority 85
 add interface Eth-Trunk1
quit
firewall zone dmz
 set priority 50
 add interface GigabitEthernet1/0/4
quit

# --- (5) Routes ---
ip route-static 0.0.0.0 0.0.0.0 10.254.254.1
ip route-static 10.0.0.0 255.255.0.0 10.254.0.1

# --- (6) VRRP (HA) ---
interface GigabitEthernet0/0/0
 vrrp vrid 10 virtual-ip 10.254.254.7 255.255.255.240 active
quit
interface Eth-Trunk1
 vrrp vrid 160 virtual-ip 10.254.0.10 255.255.255.0 active
quit
interface GigabitEthernet1/0/4
 vrrp vrid 30 virtual-ip 10.10.10.1 255.255.255.0 active
quit

return
save


## 8.2 FW-HQ-2 — base (من غير HRP) — اعمله **قبل** HRP


system-view
sysname FW-HQ-2

# --- (1) الأول: فك الـ management binding عن GE0/0/0 ---
interface GigabitEthernet0/0/0
 undo ip binding vpn-instance default
 ip address 10.254.254.6 255.255.255.240
 service-manage enable
 service-manage ping permit
quit

interface GigabitEthernet1/0/0
 ip address 10.255.254.2 255.255.255.252
 service-manage enable
 service-manage ping permit
quit

# --- (2) Inside Eth-Trunk (2 links) ---
interface Eth-Trunk1
 mode lacp-static
 ip address 10.254.0.3 255.255.255.0
 service-manage enable
 service-manage ping permit
quit
interface GigabitEthernet1/0/1
 eth-trunk 1
quit
interface GigabitEthernet1/0/2
 eth-trunk 1
quit

# --- (3) DMZ ---
interface GigabitEthernet1/0/4
 ip address 10.10.10.3 255.255.255.0
 service-manage enable
 service-manage ping permit
quit

# --- (4) Zones ---
firewall zone untrust
 set priority 5
 add interface GigabitEthernet0/0/0
quit
firewall zone trust
 set priority 85
 add interface Eth-Trunk1
quit
firewall zone dmz
 set priority 50
 add interface GigabitEthernet1/0/4
quit

# --- (5) Routes ---
ip route-static 0.0.0.0 0.0.0.0 10.254.254.1
ip route-static 10.0.0.0 255.255.0.0 10.254.0.1

# --- (6) VRRP (HA) ---
interface GigabitEthernet0/0/0
 vrrp vrid 10 virtual-ip 10.254.254.7 255.255.255.240 standby
quit
interface Eth-Trunk1
 vrrp vrid 160 virtual-ip 10.254.0.10 255.255.255.0 standby
quit
interface GigabitEthernet1/0/4
 vrrp vrid 30 virtual-ip 10.10.10.1 255.255.255.0 standby
quit

return
save


## 8.2b 🆕 Cross-links HA على الـ Firewalls (اعمله **قبل** HRP)

> الـ `GE1/0/3` هنا **routed** (مش جزء من `Eth-Trunk1` ومش L2)، فمفيش loop ومفيش تعارض مع `10.254.0.0/24`. كل cross-link ليه /30 لوحده.
> الـ `GE1/0/3` داخل **trust zone** فكل الـ security-policy والـ NAT الموجودين بيتطبقوا عليه تلقائي.
> **لو HRP شغال بالفعل:** على FW-HQ-2 اكتب الأول `hrp standby config enable` لو رفض الأوامر، وبعد الانتهاء `undo hrp standby config enable`.

**FW-HQ-1** (X1 ↔ LSW-HQ-2 G0/0/3):


system-view
interface GigabitEthernet1/0/3
 ip address 10.254.1.1 255.255.255.252
 service-manage enable
 service-manage ping permit
quit
firewall zone trust
 add interface GigabitEthernet1/0/3
quit
# رجوع احتياطي لشبكة الـ users عن طريق الـ cross-link (Preference أعلى من الأساسي)
ip route-static 10.0.0.0 255.255.0.0 10.254.1.2 preference 100
return
save


**FW-HQ-2** (X2 ↔ LSW-HQ-1 G0/0/3):


system-view
interface GigabitEthernet1/0/3
 ip address 10.254.2.1 255.255.255.252
 service-manage enable
 service-manage ping permit
quit
firewall zone trust
 add interface GigabitEthernet1/0/3
quit
ip route-static 10.0.0.0 255.255.0.0 10.254.2.2 preference 100
return
save


## 8.3 HRP — FW-HQ-1 أولًا ثم FW-HQ-2

**FW-HQ-1:**

system-view
hrp mirror session enable
hrp interface GigabitEthernet1/0/0 remote 10.255.254.2
hrp track interface GigabitEthernet0/0/0
hrp track interface Eth-Trunk1
hrp enable
return
save

**FW-HQ-2** (آخر حاجة، بعدها مفيش config تاني عليه):

system-view
hrp standby-device
hrp mirror session enable
hrp interface GigabitEthernet1/0/0 remote 10.255.254.1
hrp enable
return
save

لو hrp مشتغلش اذا اعمل هذه الخطوه 
FW-HQ-1
system-view
firewall zone trust
add interface GigabitEthernet1/0/0
quit

ثم:

interface GigabitEthernet1/0/0
service-manage ping permit
quit
FW-HQ-2
system-view
firewall zone trust
add interface GigabitEthernet1/0/0
quit

ثم:

interface GigabitEthernet1/0/0
service-manage ping permit
quit


**اختبار:**

```text
FW-HQ-1: display hrp state          → Role: active,  peer: standby
FW-HQ-2: display hrp state          → Role: standby, peer: active
FW-HQ-1: display vrrp brief         → vrid 10 / 30 / 160 = Master
FW-HQ-1: ping 10.254.254.1          ← HQ routers
FW-HQ-1: ping 10.254.0.4            ← LSW-HQ-1
```

## 8.4 Security policy + NAT (على FW-HQ-1 بس، بعد ما HRP يبقى active)


system-view

security-policy
 rule name TRUST_TO_UNTRUST
  source-zone trust
  destination-zone untrust
  action permit
 quit
 rule name TRUST_TO_DMZ
  source-zone trust
  destination-zone dmz
  action permit
 quit
 rule name TRUST_TO_LOCAL
  source-zone trust
  destination-zone local
  action permit
 quit
 rule name LOCAL_TO_TRUST
  source-zone local
  destination-zone trust
  action permit
 quit
 rule name LOCAL_TO_UNTRUST
  source-zone local
  destination-zone untrust
  action permit
 quit
 rule name UNTRUST_TO_LOCAL
  source-zone untrust
  destination-zone local
  action permit
 quit
 rule name DMZ_TO_LOCAL
  source-zone dmz
  destination-zone local
  action permit
 quit
 rule name LOCAL_TO_DMZ
  source-zone local
  destination-zone dmz
  action permit
 quit
 rule name DMZ_TO_UNTRUST
  source-zone dmz
  destination-zone untrust
  action permit
 quit
 rule name UNTRUST_TO_DMZ
  source-zone untrust
  destination-zone dmz
  destination-address 10.10.10.0 mask 255.255.255.0
  action permit
 quit
 rule name UNTRUST_INTERNAL_TO_TRUST
  source-zone untrust
  destination-zone trust
  source-address 10.0.0.0 mask 255.0.0.0
  action permit
 quit
 rule name UNTRUST_TO_TRUST
  source-zone untrust
  destination-zone trust
  action deny
 quit
quit

# الترتيب مهم: NO_NAT الأول، وبعده Easy-IP
nat-policy
 rule name NO_NAT_INTERNAL
  source-zone trust
  destination-zone untrust
  destination-address 10.0.0.0 mask 255.0.0.0
  action no-nat
 quit
 rule name HQ_USERS_TO_INTERNET
  source-zone trust
  destination-zone untrust
  source-address 10.0.0.0 mask 255.255.0.0
  action source-nat easy-ip
 quit
quit

return
save


> لو `source-nat easy-ip` مش موجود على إصدار الـ USG بتاعك: اكتب `?` جوه الـ rule وشوف الصيغة المتاحة (زي ما الملف القديم كان بيقول).
> لو مفيش وقت للـ Internet NAT دلوقتي، الـ rule الأولى (`NO_NAT_INTERNAL`) كفاية، والتانية تتأجل.

**اختبار:** `display hrp configuration check all` → security-policy و nat-policy = `Same Configuration`.

## 8.5 Default route على السويتشين (بعد ما HRP active)


# على LSW-HQ-1 وعلى LSW-HQ-2
system-view
ip route-static 0.0.0.0 0.0.0.0 10.254.0.10
return
save


> لو حصلت مشكلة في VRRP الـ Firewall الداخلي (`vrid 160`)، للاختبار بس استخدم مؤقتًا `10.254.0.2` كـ next-hop (FW-HQ-1 مباشرة) وارجع لـ `.10` بعدين.

## 8.6 🆕 Cross-links HA — ناحية سويتشات الـ core (بعد 8.5)

> VLAN 911 و 912 **مش** بيتعدّوا على أي trunk (مش في `allow-pass` بتاع Eth-Trunk1 ولا الـ access uplinks)، فمفيش L2 loop.

**LSW-HQ-2** (X1 ↔ FW-HQ-1 `G1/0/3`):


system-view
vlan batch 911
interface GigabitEthernet0/0/3
 port link-type access
 port default vlan 911
quit
interface Vlanif911
 ip address 10.254.1.2 255.255.255.252
quit
ip route-static 0.0.0.0 0.0.0.0 10.254.1.1 preference 100
return
save


**LSW-HQ-1** (X2 ↔ FW-HQ-2 `G1/0/3`):


system-view
vlan batch 912
interface GigabitEthernet0/0/3
 port link-type access
 port default vlan 912
quit
interface Vlanif912
 ip address 10.254.2.2 255.255.255.252
quit
ip route-static 0.0.0.0 0.0.0.0 10.254.2.1 preference 100
return
save


**إزاي بيشتغل:**

- الـ default route الأساسي `10.254.0.10` (Preference 60) هو المستخدم في الوضع الطبيعي.
- الـ default الاحتياطي (Preference 100) عن طريق الـ cross-link بيدخل الشغل **تلقائي** لما `Vlanif901` تقع، يعني لما اللينكات اللي بتوصّل السويتش بالـ FW (Eth-Trunk2) وبالسويتش التاني (Eth-Trunk1) يقعوا مع بعض.
- لو FW-HQ-1 بس وقع والسويتشات لسه متوصلين ببعض، الـ VRRP الداخلي (`vrid 160`) بيغطيها زي ما هو موجود في Test B.

**اختبار:**


LSW-HQ-2: ping 10.254.1.1
LSW-HQ-1: ping 10.254.2.1
LSW-HQ-1: display ip routing-table 0.0.0.0     ← الاتنين ظاهرين: Pre 60 (نشط) و Pre 100 (احتياطي)

اختبار فشل على LSW-HQ-1: اعمل shutdown لـ Eth-Trunk1 و Eth-Trunk2 الاتنين
LSW-HQ-1: display ip routing-table 0.0.0.0     ← لازم الـ default يبقى عن طريق 10.254.2.1
(وبعدها undo shutdown للاتنين)
```

------------------------------------------------------------------------

# 9. HQ verification (بالترتيب)


# 1) من LSW-HQ-1
ping 10.254.0.2                      ← FW-HQ-1 (inside)
ping 10.254.0.10                     ← FW VIP
ping 10.254.254.5                    ← FW outside
ping -a 10.0.10.1 10.254.254.1       ← HQ routers VIP
ping -a 10.0.10.1 10.255.15.15       ← DHCP server

# 2) من الـ Firewall
ping 10.254.254.1
ping 10.255.15.15
display ip routing-table 10.255.15.15   ← لازم يخرج من GigabitEthernet0/0/0

# 3) من HQ-1
ping 10.254.254.5
display ip routing-table 10.0.10.1      ← Static عن طريق 10.254.254.7
```

**DHCP:** على LSW-HQ-1 (User view):


reset dhcp relay statistics

على PC6 اضغط **DHCP → Apply** وبعد 10 ثواني:

```text
LSW-HQ-1: display dhcp relay statistics    → sent to servers > 0 ، received from servers > 0
HQ-1    : display dhcp server statistics   → Discover > 0 ، Offer > 0
PC6     : ipconfig                         → 10.0.10.x


## 9.1 Fallback لو HQ-1 فضل Discover = 0 (رغم إن الـ ping شغال)

الاحتمال إن الـ AR بيتجاهل طلب الـ relay الموجّه لـ Loopback. الحل: خلّي الـ relay يبعت لـ IP الـ interface بتاع HQ-1 (`10.254.254.2`) بدل الـ Loopback. على **LSW-HQ-1 و LSW-HQ-2 الاتنين**:

system-view
interface Vlanif10
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif20
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif30
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif40
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif50
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif60
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif70
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif80
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif90
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif100
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif110
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif120
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif130
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
interface Vlanif140
 undo dhcp relay server-ip 10.255.15.15
 dhcp relay server-ip 10.254.254.2
quit
return
save


الـ `NO_NAT_INTERNAL` بيغطي `10.254.254.2` أصلًا (جزء من `10.0.0.0/8`) فمفيش حاجة تانية تتغير.

لو لسه صفر: اعمل Capture على `GE0/0/0` بتاع HQ-1 بفلتر `bootp` وابعتلي النتيجة، وتأكد إن `display vrrp brief` على HQ-1 بيقول **Master**.

------------------------------------------------------------------------

# PHASE 6 — BR1

## 10.1 BR1-1


system-view
sysname BR1-1

interface LoopBack0
 ip address 10.255.11.1 255.255.255.255
quit

interface GigabitEthernet0/0/0
 ip address 10.254.11.2 255.255.255.248
 vrrp vrid 11 virtual-ip 10.254.11.1
 vrrp vrid 11 priority 150
 vrrp vrid 11 preempt-mode timer delay 20
quit

interface Serial3/0/0
 ip address 10.255.111.1 255.255.255.252
quit

ip route-static 10.1.0.0 255.255.0.0 10.254.11.4

ospf 1 router-id 10.255.11.1
 import-route static
 area 0.0.0.0
  network 10.255.11.1 0.0.0.0
  network 10.254.11.0 0.0.0.7
  network 10.255.111.0 0.0.0.3
 quit
quit

return
save


## 10.2 BR1-2


system-view
sysname BR1-2

interface LoopBack0
 ip address 10.255.11.2 255.255.255.255
quit

interface GigabitEthernet0/0/0
 ip address 10.254.11.3 255.255.255.248
 vrrp vrid 11 virtual-ip 10.254.11.1
 vrrp vrid 11 priority 130
 vrrp vrid 11 preempt-mode timer delay 20
quit

interface Serial3/0/0
 ip address 10.255.112.1 255.255.255.252
quit

ip route-static 10.1.0.0 255.255.0.0 10.254.11.4

ospf 1 router-id 10.255.11.2
 import-route static
 area 0.0.0.0
  network 10.255.11.2 0.0.0.0
  network 10.254.11.0 0.0.0.7
  network 10.255.112.0 0.0.0.3
 quit
quit

return
save


## 10.3 LSW-BR1-1 (L2)


system-view
sysname LSW-BR1-1
vlan 11

interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 11
quit
interface GigabitEthernet0/0/4
 port link-type access
 port default vlan 11
quit

interface Eth-Trunk1
 mode lacp-static
 port link-type access
 port default vlan 11
quit
interface GigabitEthernet0/0/2
 eth-trunk 1
quit
interface GigabitEthernet0/0/3
 eth-trunk 1
quit

return
save


## 10.4 FW-BR1-1


system-view
sysname FW-BR1-1

# GE0/0/0 لازم يتفك من الـ management binding قبل ما يدخل Eth-Trunk
interface GigabitEthernet0/0/0
 undo ip binding vpn-instance default
quit

# --- Outside: Eth-Trunk1 (GE0/0/0 + GE1/0/0) ---
interface Eth-Trunk1
 mode lacp-static
 ip address 10.254.11.4 255.255.255.248
 service-manage enable
 service-manage ping permit
quit
interface GigabitEthernet0/0/0
 eth-trunk 1
quit
interface GigabitEthernet1/0/0
 eth-trunk 1
quit

# --- Inside: Eth-Trunk2 (GE1/0/1 + GE1/0/2) ---
interface Eth-Trunk2
 mode lacp-static
 ip address 10.1.0.1 255.255.255.0
 service-manage enable
 service-manage ping permit
quit
interface GigabitEthernet1/0/1
 eth-trunk 2
quit
interface GigabitEthernet1/0/2
 eth-trunk 2
quit

firewall zone untrust
 set priority 5
 add interface Eth-Trunk1
quit
firewall zone trust
 set priority 85
 add interface Eth-Trunk2
quit

ip route-static 0.0.0.0 0.0.0.0 10.254.11.1
ip route-static 10.1.0.0 255.255.0.0 10.1.0.2

security-policy
 rule name BR1_TO_WAN
  source-zone trust
  destination-zone untrust
  action permit
 quit
 rule name BR1_TRUST_TO_LOCAL
  source-zone trust
  destination-zone local
  action permit
 quit
 rule name BR1_LOCAL_TO_TRUST
  source-zone local
  destination-zone trust
  action permit
 quit
 rule name BR1_LOCAL_TO_UNTRUST
  source-zone local
  destination-zone untrust
  action permit
 quit
 rule name BR1_UNTRUST_TO_LOCAL
  source-zone untrust
  destination-zone local
  action permit
 quit
 rule name BR1_INTERNAL_TO_TRUST
  source-zone untrust
  destination-zone trust
  source-address 10.0.0.0 mask 255.0.0.0
  action permit
 quit
quit

return
save


> **لو `GE0/0/0` رفض `eth-trunk 1`** (لأنه port الـ management): سيب `Eth-Trunk1` واعمل الـ outside على `GE1/0/0` لوحده:
> `interface GigabitEthernet1/0/0` ثم `ip address 10.254.11.4 255.255.255.248` + `service-manage enable` + `service-manage ping permit`، وضيفه لـ `untrust` بدل `Eth-Trunk1`.
> وعلى LSW-BR1-1 شيل `GE0/0/2` من الـ Eth-Trunk (وخلّي `GE0/0/3` بس على `port default vlan 11`).

## 10.5 LSW-BR1-2 (Distribution)


system-view
sysname LSW-BR1-2
dhcp enable
vlan batch 10 20 30 40 50 60 70 80 1000

interface Eth-Trunk1
 mode lacp-static
 port link-type access
 port default vlan 1000
quit
interface GigabitEthernet0/0/1
 eth-trunk 1
quit
interface GigabitEthernet0/0/2
 eth-trunk 1
quit
interface Vlanif1000
 ip address 10.1.0.2 255.255.255.0
quit

interface GigabitEthernet0/0/3
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit
interface GigabitEthernet0/0/4
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit
interface GigabitEthernet0/0/5
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit

interface Vlanif10
 ip address 10.1.10.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif20
 ip address 10.1.20.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif30
 ip address 10.1.30.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif40
 ip address 10.1.40.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif50
 ip address 10.1.50.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif60
 ip address 10.1.60.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif70
 ip address 10.1.70.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif80
 ip address 10.1.80.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit

stp mode mstp
stp region-configuration
 region-name BR1
 revision-level 1
 active region-configuration
quit
stp root primary

ip route-static 0.0.0.0 0.0.0.0 10.1.0.1

return
save


## 10.6 BR1-LSW1


system-view
sysname BR1-LSW1
vlan batch 10 20 30 40 50 60 70 80

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit
interface Ethernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit

interface Ethernet0/0/2
 port link-type access
 port default vlan 10
 stp edged-port enable
quit
interface Ethernet0/0/3
 port link-type access
 port default vlan 20
 stp edged-port enable
quit
interface GigabitEthernet0/0/2
 port link-type access
 port default vlan 30
 stp edged-port enable
quit

stp mode mstp
stp region-configuration
 region-name BR1
 revision-level 1
 active region-configuration
quit
stp bpdu-protection

return
save


## 10.7 BR1-LSW2


system-view
sysname BR1-LSW2
vlan batch 10 20 30 40 50 60 70 80

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit
interface Ethernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit

interface Ethernet0/0/3
 port link-type access
 port default vlan 40
 stp edged-port enable
quit
interface Ethernet0/0/2
 port link-type access
 port default vlan 50
 stp edged-port enable
quit

stp mode mstp
stp region-configuration
 region-name BR1
 revision-level 1
 active region-configuration
quit
stp bpdu-protection

return
save


## 10.8 BR1-LSW3


system-view
sysname BR1-LSW3
vlan batch 10 20 30 40 50 60 70 80

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 70 80
quit

interface Ethernet0/0/2
 port link-type access
 port default vlan 60
 stp edged-port enable
quit
interface Ethernet0/0/3
 port link-type access
 port default vlan 70
 stp edged-port enable
quit
interface Ethernet0/0/1
 port link-type access
 port default vlan 80
 stp edged-port enable
quit

stp mode mstp
stp region-configuration
 region-name BR1
 revision-level 1
 active region-configuration
quit
stp bpdu-protection

return
save


Endpoints BR1: `PC1 VLAN10` ، `PC2 VLAN20` ، `AP1 VLAN30` ، `PC3 VLAN40` ، `AP2 VLAN50` ، `PC4 VLAN60` ، `PC5 VLAN70` ، `AP3 VLAN80`.

## 10.9 🆕 LSW-BR1-2 — تكملة الـ DHCP (بعد الـ interface pool الموجود في 10.5)

> الـ DHCP شغال أصلًا على LSW-BR1-2. هنا بنضيف: excluded range (.2 → .20 زي الـ HQ)، lease يوم، و**Option 43** لـ VLANs الـ APs (30 / 50 / 80) بعنوان **AC-BR1 المحلي** (`10.1.100.20`).


system-view
interface Vlanif10
 dhcp server excluded-ip-address 10.1.10.2 10.1.10.20
 dhcp server lease day 1
quit
interface Vlanif20
 dhcp server excluded-ip-address 10.1.20.2 10.1.20.20
 dhcp server lease day 1
quit
interface Vlanif30
 dhcp server excluded-ip-address 10.1.30.2 10.1.30.20
 dhcp server lease day 1
 dhcp server option 43 sub-option 3 ascii 10.1.100.20
quit
interface Vlanif40
 dhcp server excluded-ip-address 10.1.40.2 10.1.40.20
 dhcp server lease day 1
quit
interface Vlanif50
 dhcp server excluded-ip-address 10.1.50.2 10.1.50.20
 dhcp server lease day 1
 dhcp server option 43 sub-option 3 ascii 10.1.100.20
quit
interface Vlanif60
 dhcp server excluded-ip-address 10.1.60.2 10.1.60.20
 dhcp server lease day 1
quit
interface Vlanif70
 dhcp server excluded-ip-address 10.1.70.2 10.1.70.20
 dhcp server lease day 1
quit
interface Vlanif80
 dhcp server excluded-ip-address 10.1.80.2 10.1.80.20
 dhcp server lease day 1
 dhcp server option 43 sub-option 3 ascii 10.1.100.20
quit
return
save


**اختبار:** `display ip pool interface Vlanif10 used` على LSW-BR1-2 بعد ما الـ PC الأول ياخد DHCP.

## 10.10 🆕 BR1 — جاهزية الـ Wireless (VLAN 100 للـ AC + VLAN 110 للـ clients + ports الـ APs)

> - **VLAN 100** (`10.1.100.0/24`): شبكة AC-BR1، بيتوصل على `LSW-BR1-2 G0/0/6`. الـ AC جوه الـ trust zone، فمفيش حاجة بتعدّي على الـ Firewall.
> - **VLAN 110** (`10.1.110.0/24`): الـ Wireless clients، والـ DHCP عليها من LSW-BR1-2.
> - نفذ ده **بعد** الـ base بتاع السويتشات (10.5 → 10.8).

**LSW-BR1-2:**


system-view
vlan batch 100 110

interface GigabitEthernet0/0/3
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/4
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/5
 port trunk allow-pass vlan 110
quit

# port الـ AC-BR1
interface GigabitEthernet0/0/6
 port link-type access
 port default vlan 100
quit
interface Vlanif100
 ip address 10.1.100.1 255.255.255.0
quit

# Gateway + DHCP الـ Wireless clients
interface Vlanif110
 ip address 10.1.110.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
 dhcp server excluded-ip-address 10.1.110.2 10.1.110.20
 dhcp server lease day 1
quit
return
save


**BR1-LSW1** (AP على `G0/0/2`، VLAN إدارة 30):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/1
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface GigabitEthernet0/0/2
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 30
 port trunk allow-pass vlan 30 110
quit
return
save


**BR1-LSW2** (AP على `E0/0/2`، VLAN إدارة 50):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/1
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/2
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 50
 port trunk allow-pass vlan 50 110
quit
return
save


**BR1-LSW3** (AP على `E0/0/1`، VLAN إدارة 80):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/1
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 80
 port trunk allow-pass vlan 80 110
quit
return
save


------------------------------------------------------------------------

# PHASE 7 — BR2

## 11.1 BR2-1


system-view
sysname BR2-1

interface LoopBack0
 ip address 10.255.12.1 255.255.255.255
quit

interface GigabitEthernet0/0/0
 ip address 10.254.12.2 255.255.255.248
 vrrp vrid 12 virtual-ip 10.254.12.1
 vrrp vrid 12 priority 150
 vrrp vrid 12 preempt-mode timer delay 20
quit

interface Serial3/0/0
 ip address 10.255.121.1 255.255.255.252
quit

ip route-static 10.2.0.0 255.255.0.0 10.254.12.4

ospf 1 router-id 10.255.12.1
 import-route static
 area 0.0.0.0
  network 10.255.12.1 0.0.0.0
  network 10.254.12.0 0.0.0.7
  network 10.255.121.0 0.0.0.3
 quit
quit

return
save


## 11.2 BR2-2


system-view
sysname BR2-2

interface LoopBack0
 ip address 10.255.12.2 255.255.255.255
quit

interface GigabitEthernet0/0/0
 ip address 10.254.12.3 255.255.255.248
 vrrp vrid 12 virtual-ip 10.254.12.1
 vrrp vrid 12 priority 130
 vrrp vrid 12 preempt-mode timer delay 20
quit

interface Serial3/0/0
 ip address 10.255.122.1 255.255.255.252
quit

ip route-static 10.2.0.0 255.255.0.0 10.254.12.4

ospf 1 router-id 10.255.12.2
 import-route static
 area 0.0.0.0
  network 10.255.12.2 0.0.0.0
  network 10.254.12.0 0.0.0.7
  network 10.255.122.0 0.0.0.3
 quit
quit

return
save


## 11.3 LSW-BR2-1 (L2)


system-view
sysname LSW-BR2-1
vlan 12

interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 12
quit
interface GigabitEthernet0/0/2
 port link-type access
 port default vlan 12
quit

interface Eth-Trunk1
 mode lacp-static
 port link-type access
 port default vlan 12
quit
interface GigabitEthernet0/0/3
 eth-trunk 1
quit
interface GigabitEthernet0/0/4
 eth-trunk 1
quit

return
save


## 11.4 FW-BR2-1


system-view
sysname FW-BR2-1

interface GigabitEthernet0/0/0
 undo ip binding vpn-instance default
quit

# --- Outside: Eth-Trunk1 (GE0/0/0 + GE1/0/0) ---
interface Eth-Trunk1
 mode lacp-static
 ip address 10.254.12.4 255.255.255.248
 service-manage enable
 service-manage ping permit
quit
interface GigabitEthernet0/0/0
 eth-trunk 1
quit
interface GigabitEthernet1/0/0
 eth-trunk 1
quit

# --- Inside: Eth-Trunk2 (GE1/0/1 + GE1/0/2) ---
interface Eth-Trunk2
 mode lacp-static
 ip address 10.2.0.1 255.255.255.0
 service-manage enable
 service-manage ping permit
quit
interface GigabitEthernet1/0/1
 eth-trunk 2
quit
interface GigabitEthernet1/0/2
 eth-trunk 2
quit

firewall zone untrust
 set priority 5
 add interface Eth-Trunk1
quit
firewall zone trust
 set priority 85
 add interface Eth-Trunk2
quit

ip route-static 0.0.0.0 0.0.0.0 10.254.12.1
ip route-static 10.2.0.0 255.255.0.0 10.2.0.2

security-policy
 rule name BR2_TO_WAN
  source-zone trust
  destination-zone untrust
  action permit
 quit
 rule name BR2_TRUST_TO_LOCAL
  source-zone trust
  destination-zone local
  action permit
 quit
 rule name BR2_LOCAL_TO_TRUST
  source-zone local
  destination-zone trust
  action permit
 quit
 rule name BR2_LOCAL_TO_UNTRUST
  source-zone local
  destination-zone untrust
  action permit
 quit
 rule name BR2_UNTRUST_TO_LOCAL
  source-zone untrust
  destination-zone local
  action permit
 quit
 rule name BR2_INTERNAL_TO_TRUST
  source-zone untrust
  destination-zone trust
  source-address 10.0.0.0 mask 255.0.0.0
  action permit
 quit
quit

return
save


> نفس ملاحظة FW-BR1-1: لو `GE0/0/0` رفض الـ `eth-trunk`، استخدم `GE1/0/0` لوحده واشيل `GE0/0/3` من الـ Eth-Trunk على LSW-BR2-1.

## 11.5 LSW-BR2-2 (Distribution)


system-view
sysname LSW-BR2-2
dhcp enable
vlan batch 10 20 30 40 50 60 1000

interface Eth-Trunk1
 mode lacp-static
 port link-type access
 port default vlan 1000
quit
interface GigabitEthernet0/0/1
 eth-trunk 1
quit
interface GigabitEthernet0/0/2
 eth-trunk 1
quit
interface Vlanif1000
 ip address 10.2.0.2 255.255.255.0
quit

interface GigabitEthernet0/0/3
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit
interface GigabitEthernet0/0/4
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit
interface GigabitEthernet0/0/5
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit

interface Vlanif10
 ip address 10.2.10.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif20
 ip address 10.2.20.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif30
 ip address 10.2.30.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif40
 ip address 10.2.40.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif50
 ip address 10.2.50.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit
interface Vlanif60
 ip address 10.2.60.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
quit

stp mode mstp
stp region-configuration
 region-name BR2
 revision-level 1
 active region-configuration
quit
stp root primary

ip route-static 0.0.0.0 0.0.0.0 10.2.0.1

return
save


## 11.6 BR2-LSW1


system-view
sysname BR2-LSW1
vlan batch 10 20 30 40 50 60

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit

interface Ethernet0/0/2
 port link-type access
 port default vlan 10
 stp edged-port enable
quit
interface Ethernet0/0/1
 port link-type access
 port default vlan 20
 stp edged-port enable
quit

stp mode mstp
stp region-configuration
 region-name BR2
 revision-level 1
 active region-configuration
quit
stp bpdu-protection

return
save


## 11.7 BR2-LSW2


system-view
sysname BR2-LSW2
vlan batch 10 20 30 40 50 60

interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit
interface Ethernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit

interface Ethernet0/0/3
 port link-type access
 port default vlan 30
 stp edged-port enable
quit
interface Ethernet0/0/2
 port link-type access
 port default vlan 40
 stp edged-port enable
quit

stp mode mstp
stp region-configuration
 region-name BR2
 revision-level 1
 active region-configuration
quit
stp bpdu-protection

return
save


## 11.8 BR2-LSW3


system-view
sysname BR2-LSW3
vlan batch 10 20 30 40 50 60

interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit
interface Ethernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60
quit

interface Ethernet0/0/3
 port link-type access
 port default vlan 50
 stp edged-port enable
quit
interface Ethernet0/0/2
 port link-type access
 port default vlan 60
 stp edged-port enable
quit

stp mode mstp
stp region-configuration
 region-name BR2
 revision-level 1
 active region-configuration
quit
stp bpdu-protection

return
save


Endpoints BR2: `PC11 VLAN10` ، `AP1 VLAN20` ، `PC12 VLAN30` ، `AP2 VLAN40` ، `PC13 VLAN50` ، `AP3 VLAN60`.

## 11.9 🆕 LSW-BR2-2 — تكملة الـ DHCP (بعد الـ interface pool الموجود في 11.5)

> الـ DHCP شغال أصلًا على LSW-BR2-2. هنا بنضيف: excluded range (.2 → .20 زي الـ HQ)، lease يوم، و**Option 43** لـ VLANs الـ APs (20 / 40 / 60) بعنوان **AC-BR2 المحلي** (`10.2.100.20`).


system-view
interface Vlanif10
 dhcp server excluded-ip-address 10.2.10.2 10.2.10.20
 dhcp server lease day 1
quit
interface Vlanif20
 dhcp server excluded-ip-address 10.2.20.2 10.2.20.20
 dhcp server lease day 1
 dhcp server option 43 sub-option 3 ascii 10.2.100.20
quit
interface Vlanif30
 dhcp server excluded-ip-address 10.2.30.2 10.2.30.20
 dhcp server lease day 1
quit
interface Vlanif40
 dhcp server excluded-ip-address 10.2.40.2 10.2.40.20
 dhcp server lease day 1
 dhcp server option 43 sub-option 3 ascii 10.2.100.20
quit
interface Vlanif50
 dhcp server excluded-ip-address 10.2.50.2 10.2.50.20
 dhcp server lease day 1
quit
interface Vlanif60
 dhcp server excluded-ip-address 10.2.60.2 10.2.60.20
 dhcp server lease day 1
 dhcp server option 43 sub-option 3 ascii 10.2.100.20
quit
return
save


**اختبار:** `display ip pool interface Vlanif10 used` على LSW-BR2-2 بعد ما الـ PC الأول ياخد DHCP.

## 11.10 🆕 BR2 — جاهزية الـ Wireless (VLAN 100 للـ AC + VLAN 110 للـ clients + ports الـ APs)

> - **VLAN 100** (`10.2.100.0/24`): شبكة AC-BR2، بيتوصل على `LSW-BR2-2 G0/0/6`. الـ AC جوه الـ trust zone، فمفيش حاجة بتعدّي على الـ Firewall.
> - **VLAN 110** (`10.2.110.0/24`): الـ Wireless clients، والـ DHCP عليها من LSW-BR2-2.
> - نفذ ده **بعد** الـ base بتاع السويتشات (11.5 → 11.8).

**LSW-BR2-2:**


system-view
vlan batch 100 110

interface GigabitEthernet0/0/3
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/4
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/5
 port trunk allow-pass vlan 110
quit

# port الـ AC-BR2
interface GigabitEthernet0/0/6
 port link-type access
 port default vlan 100
quit
interface Vlanif100
 ip address 10.2.100.1 255.255.255.0
quit

# Gateway + DHCP الـ Wireless clients
interface Vlanif110
 ip address 10.2.110.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 8.8.8.8
 dhcp server excluded-ip-address 10.2.110.2 10.2.110.20
 dhcp server lease day 1
quit
return
save


**BR2-LSW1** (AP على `E0/0/1`، VLAN إدارة 20):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/1
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 20
 port trunk allow-pass vlan 20 110
quit
return
save


**BR2-LSW2** (AP على `E0/0/2`، VLAN إدارة 40):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface GigabitEthernet0/0/2
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/1
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/2
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 40
 port trunk allow-pass vlan 40 110
quit
return
save


**BR2-LSW3** (AP على `E0/0/2`، VLAN إدارة 60):


system-view
vlan batch 110
interface GigabitEthernet0/0/1
 port trunk allow-pass vlan 110
quit
interface Ethernet0/0/1
 port trunk allow-pass vlan 110
quit
# port الـ AP: من access إلى trunk (PVID = VLAN إدارة الـ AP untagged، و 110 للـ clients tagged)
interface Ethernet0/0/2
 undo port default vlan
 port link-type trunk
 port trunk pvid vlan 60
 port trunk allow-pass vlan 60 110
quit
return
save


------------------------------------------------------------------------

# PHASE 8 — ISP

## 12.1 ISP-1  (اتصلحت مطابقة لـ ports.txt)


system-view
sysname ISP-1

interface LoopBack0
 ip address 10.255.19.19 255.255.255.255
quit

interface Serial3/0/0
 ip address 10.255.201.1 255.255.255.252
quit
interface Serial2/0/0
 ip address 10.255.111.2 255.255.255.252
quit
interface Serial2/0/1
 ip address 10.255.112.2 255.255.255.252
quit

ospf 1 router-id 10.255.19.19
 area 0.0.0.0
  network 10.255.19.19 0.0.0.0
  network 10.255.201.0 0.0.0.3
  network 10.255.111.0 0.0.0.3
  network 10.255.112.0 0.0.0.3
 quit
quit

return
save


## 12.2 ISP-2


system-view
sysname ISP-2

interface LoopBack0
 ip address 10.255.17.17 255.255.255.255
quit

interface Serial2/0/0
 ip address 10.255.201.2 255.255.255.252
quit
interface Serial2/0/1
 ip address 10.255.202.1 255.255.255.252
quit

ospf 1 router-id 10.255.17.17
 area 0.0.0.0
  network 10.255.17.17 0.0.0.0
  network 10.255.201.0 0.0.0.3
  network 10.255.202.0 0.0.0.3
 quit
quit

return
save


## 12.3 ISP-3


system-view
sysname ISP-3

interface LoopBack0
 ip address 10.255.22.22 255.255.255.255
quit

interface Serial2/0/0
 ip address 10.255.202.2 255.255.255.252
quit
interface Serial2/0/1
 ip address 10.255.203.1 255.255.255.252
quit
interface Serial3/0/0
 ip address 10.255.101.2 255.255.255.252
quit
interface Serial3/0/1
 ip address 10.255.102.2 255.255.255.252
quit
interface Serial4/0/0
 ip address 10.255.103.2 255.255.255.252
quit

ospf 1 router-id 10.255.22.22
 area 0.0.0.0
  network 10.255.22.22 0.0.0.0
  network 10.255.202.0 0.0.0.3
  network 10.255.203.0 0.0.0.3
  network 10.255.101.0 0.0.0.3
  network 10.255.102.0 0.0.0.3
  network 10.255.103.0 0.0.0.3
 quit
quit

return
save


## 12.4 ISP-4


system-view
sysname ISP-4

interface LoopBack0
 ip address 10.255.18.18 255.255.255.255
quit

interface Serial3/0/0
 ip address 10.255.203.2 255.255.255.252
quit
interface Serial3/0/1
 ip address 10.255.204.1 255.255.255.252
quit

ospf 1 router-id 10.255.18.18
 area 0.0.0.0
  network 10.255.18.18 0.0.0.0
  network 10.255.203.0 0.0.0.3
  network 10.255.204.0 0.0.0.3
 quit
quit

return
save


## 12.5 ISP-5


system-view
sysname ISP-5

interface LoopBack0
 ip address 10.255.20.20 255.255.255.255
quit

interface Serial2/0/0
 ip address 10.255.204.2 255.255.255.252
quit
interface Serial2/0/1
 ip address 10.255.121.2 255.255.255.252
quit
interface Serial3/0/0
 ip address 10.255.122.2 255.255.255.252
quit

ospf 1 router-id 10.255.20.20
 area 0.0.0.0
  network 10.255.20.20 0.0.0.0
  network 10.255.204.0 0.0.0.3
  network 10.255.121.0 0.0.0.3
  network 10.255.122.0 0.0.0.3
 quit
quit

return
save


# PHASE 9 🆕 — Wireless (AC لكل موقع)

## W.1 Firewall rule لـ AC-HQ (على FW-HQ-1 بس، بعد HRP active)

> الـ APs بتبدأ الاتصال (trust → dmz، وده مسموح)، لكن الـ AC بيبعت للـ APs رسائل من عنده (dmz → trust) ومفيش rule ليها في الأصل. **الفروع مش محتاجة ده** لأن AC الفرع جوه الـ trust zone.

system-view
security-policy
 rule name DMZ_AC_TO_TRUST
  source-zone dmz
  destination-zone trust
  source-address 10.10.10.20 mask 255.255.255.255
  action permit
 quit
quit
return
save



## W.2 AC-HQ (يدير HQ-AP1 → HQ-AP5)


system-view
sysname AC-HQ

vlan batch 110 300

interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 300
quit
interface Vlanif300
 ip address 10.10.10.20 255.255.255.0
quit
ip route-static 0.0.0.0 0.0.0.0 10.10.10.1

capwap source interface vlanif 300

wlan
 regulatory-domain-profile name DOMAIN_CN
  country-code cn
 quit
 security-profile name SEC_ENT
  security wpa2 psk pass-phrase Huawei@123 aes
 quit
 ssid-profile name SSID_ENT
  ssid Enterprise-WiFi
 quit
 vap-profile name VAP_V110
  forward-mode direct-forward
  service-vlan vlan-id 110
  security-profile SEC_ENT
  ssid-profile SSID_ENT
 quit
 ap-group name AP-HQ-W
  regulatory-domain-profile DOMAIN_CN
  vap-profile VAP_V110 wlan 1 radio all
 quit
 ap auth-mode mac-auth
 ap-id 1 ap-mac <MAC-HQ-AP1>
  ap-name HQ-AP1
  ap-group AP-HQ-W
 quit
 ap-id 2 ap-mac <MAC-HQ-AP2>
  ap-name HQ-AP2
  ap-group AP-HQ-W
 quit
 ap-id 3 ap-mac <MAC-HQ-AP3>
  ap-name HQ-AP3
  ap-group AP-HQ-W
 quit
 ap-id 4 ap-mac <MAC-HQ-AP4>
  ap-name HQ-AP4
  ap-group AP-HQ-W
 quit
 ap-id 5 ap-mac <MAC-HQ-AP5>
  ap-name HQ-AP5
  ap-group AP-HQ-W
 quit
quit
return
save


## W.3 AC-BR1 (يدير BR1-AP1 → BR1-AP3)

> موصل على `LSW-BR1-2 G0/0/6` (VLAN 100). الـ gateway `10.1.100.1`.


system-view
sysname AC-BR1

vlan batch 110 100

interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 100
quit
interface Vlanif100
 ip address 10.1.100.20 255.255.255.0
quit
ip route-static 0.0.0.0 0.0.0.0 10.1.100.1

capwap source interface vlanif 100

wlan
 regulatory-domain-profile name DOMAIN_CN
  country-code cn
 quit
 security-profile name SEC_ENT
  security wpa2 psk pass-phrase Huawei@123 aes
 quit
 ssid-profile name SSID_ENT
  ssid Enterprise-WiFi-BR1
 quit
 vap-profile name VAP_V110
  forward-mode direct-forward
  service-vlan vlan-id 110
  security-profile SEC_ENT
  ssid-profile SSID_ENT
 quit
 ap-group name AP-BR1-W
  regulatory-domain-profile DOMAIN_CN
  vap-profile VAP_V110 wlan 1 radio all
 quit
 ap auth-mode mac-auth
 ap-id 1 ap-mac <MAC-BR1-AP1>
  ap-name BR1-AP1
  ap-group AP-BR1-W
 quit
 ap-id 2 ap-mac <MAC-BR1-AP2>
  ap-name BR1-AP2
  ap-group AP-BR1-W
 quit
 ap-id 3 ap-mac <MAC-BR1-AP3>
  ap-name BR1-AP3
  ap-group AP-BR1-W
 quit
quit
return
save


## W.4 AC-BR2 (يدير BR2-AP1 → BR2-AP3)

> موصل على `LSW-BR2-2 G0/0/6` (VLAN 100). الـ gateway `10.2.100.1`.


system-view
sysname AC-BR2

vlan batch 110 100

interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 100
quit
interface Vlanif100
 ip address 10.2.100.20 255.255.255.0
quit
ip route-static 0.0.0.0 0.0.0.0 10.2.100.1

capwap source interface vlanif 100

wlan
 regulatory-domain-profile name DOMAIN_CN
  country-code cn
 quit
 security-profile name SEC_ENT
  security wpa2 psk pass-phrase Huawei@123 aes
 quit
 ssid-profile name SSID_ENT
  ssid Enterprise-WiFi-BR2
 quit
 vap-profile name VAP_V110
  forward-mode direct-forward
  service-vlan vlan-id 110
  security-profile SEC_ENT
  ssid-profile SSID_ENT
 quit
 ap-group name AP-BR2-W
  regulatory-domain-profile DOMAIN_CN
  vap-profile VAP_V110 wlan 1 radio all
 quit
 ap auth-mode mac-auth
 ap-id 1 ap-mac <MAC-BR2-AP1>
  ap-name BR2-AP1
  ap-group AP-BR2-W
 quit
 ap-id 2 ap-mac <MAC-BR2-AP2>
  ap-name BR2-AP2
  ap-group AP-BR2-W
 quit
 ap-id 3 ap-mac <MAC-BR2-AP3>
  ap-name BR2-AP3
  ap-group AP-BR2-W
 quit
quit
return
save


## W.5 الاختبار


AC-HQ  : ping 10.10.10.1          ← الـ gateway (FW VIP)
AC-BR1 : ping 10.1.100.1          ← LSW-BR1-2
AC-BR2 : ping 10.2.100.1          ← LSW-BR2-2

على كل AC:
display ap all                    ← لازم State = nor لكل الـ APs بتاعته
display vap all                   ← VAPs بحالة ON


**لو AP مش ظاهر `nor`:**


AC     : display ap all                         ← idle = الـ MAC غلط | لو مش موجود أصلًا = الـ AP ماخدش IP أو ما وصلش للـ AC
AC     : display ap unauthorized record         ← بيوريك MAC الـ AP اللي وصل ومش مسجّل
سويتش الـ distribution : display ip pool interface Vlan<id> used   ← الـ AP ماخدش IP؟ (وراجع Option 43)

