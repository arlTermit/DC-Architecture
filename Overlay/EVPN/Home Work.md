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

Настройка BLeaf NSOS-1 NXOS-3 и NXOS-11:

<details>
<summary>NSOS-1</summary>
<pre><code>

</code></pre>
</details>

<details>
<summary>NSOS-3</summary>
<pre><code>

</code></pre>
</details>

<details>
<summary>NSOS-11</summary>
<pre><code>

</code></pre>
</details>
