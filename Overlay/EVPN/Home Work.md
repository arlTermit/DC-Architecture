Цель:
Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами

В этой  самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроить BGP peering между Leaf и Spine в AF l2vpn evpn
2. Spine работает в качестве route-reflector
3. Настроена связанность между клиентами в первой зоне

![](img/L2VPN.png)


! В выводе убраны все настройки не относящиеся к поставленной задаче.


Настройка BGP RR NSOS-2 и NXOS-4:

<details>
<summary>NSOS-2</summary>
<pre><code>
NX-2# show run

nv overlay evpn
feature ospf
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface loopback0
  ip address 2.2.2.2/32
  ip router ospf UNDER area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDER
  router-id 2.2.2.2
router bgp 65001
  template peer LEAF
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
  neighbor 1.1.1.1
    inherit peer LEAF
  neighbor 3.3.3.3
    inherit peer LEAF
  neighbor 5.5.5.5
    inherit peer LEAF

</code></pre>
</details>

<details>
<summary>NSOS-4</summary>
<pre><code>
NX-4# show run

nv overlay evpn
feature ospf
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface loopback0
  ip address 4.4.4.4/32
  ip router ospf UNDER area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDER
  router-id 4.4.4.4
router bgp 65001
  template peer LEAF
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
  neighbor 1.1.1.1
    inherit peer LEAF
  neighbor 3.3.3.3
    inherit peer LEAF
  neighbor 5.5.5.5
    inherit peer LEAF

</code></pre>
</details>

Настройка Leaf NSOS-1 NXOS-3 и NXOS-11:

<details>
<summary>NSOS-1</summary>
<pre><code>
NX-1# show run

nv overlay evpn
feature ospf
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

vlan 1,11-12
vlan 11
  vn-segment 11000
vlan 12
  vn-segment 12000

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 11000
    ingress-replication protocol bgp
  member vni 12000
    ingress-replication protocol bgp

interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/3
  switchport access vlan 11

interface Ethernet1/4
  switchport access vlan 12

interface loopback0
  ip address 1.1.1.1/32
  ip router ospf UNDER area 0.0.0.0

interface loopback1
  ip address 10.1.1.1/32
  ip router ospf UNDER area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
no feature signature-verification
router ospf UNDER
  router-id 1.1.1.1
router bgp 65001
  template peer SPINE
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 2.2.2.2
    inherit peer SPINE
  neighbor 4.4.4.4
    inherit peer SPINE

</code></pre>
</details>

<details>
<summary>NSOS-3</summary>
<pre><code>
NX-3# show run

nv overlay evpn
feature ospf
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 11000
    ingress-replication protocol bgp
  member vni 12000
    ingress-replication protocol bgp

interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/3
  switchport access vlan 11

interface Ethernet1/4
  switchport access vlan 12

interface loopback0
  ip address 3.3.3.3/32
  ip router ospf UNDER area 0.0.0.0

interface loopback1
  ip address 30.3.3.3/32
  ip router ospf UNDER area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
no feature signature-verification
router ospf UNDER
  router-id 3.3.3.3
router bgp 65001
  template peer SPINE
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 2.2.2.2
    inherit peer SPINE
  neighbor 4.4.4.4
    inherit peer SPINE

</code></pre>
</details>

<details>
<summary>NSOS-11</summary>
<pre><code>
NX-11# show run

nv overlay evpn
feature ospf
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

vlan 1,12
vlan 12
  vn-segment 12000

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 12000
    ingress-replication protocol bgp

interface Ethernet1/1
  switchport access vlan 12

interface Ethernet1/2
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDER area 0.0.0.0
  no shutdown

interface loopback0
  ip address 5.5.5.5/32
  ip router ospf UNDER area 0.0.0.0

interface loopback1
  ip address 50.5.5.5/32
  ip router ospf UNDER area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
no feature signature-verification
router ospf UNDER
  router-id 5.5.5.5
router bgp 65001
  template peer SPINE
    remote-as 65001
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 2.2.2.2
    inherit peer SPINE
  neighbor 4.4.4.4
    inherit peer SPINE

</code></pre>
</details>
