Цель: Настроить OSPF для Underlay сети.

1. Настроить OSPF в Underlay сети, для IP связанности между всеми устройствами NXOS.
2. Проверка связности месжду сетями расположенных в одной локации, так и между локациями.
3. Предоставить вывод таблиц маршрутизации с утройств.

Пояснение. На схеме отражена распределенная сеть построенная на базе протокола OSPF. Все маршрутизаторы находятся в единой Area 0. Связь между сетями происходит через единый маршрутизатор R-8.
Сеть имеет ряд недостатков:

1. Единая точка отказа. Маршрутизацтор R-8.
2. Отсутствие резервных линий связи от устройств NXOS 1-3 к R-8.
3. Не оптимальная настройки протокала OSPF.
4. Для подключения маршрутизатора R-10 используется протокол VRRP.

! В выводе убраны все настройки не относящиеся к поставленной задаче.

Настройка маршрутизатора R-8:
<details>
<summary>R-8</summary>
<pre><code>
Router#show run

interface Loopback0
 ip address 1.1.1.255 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.10.0 255.255.255.254
!
interface Ethernet0/1
 ip address 10.10.10.2 255.255.255.254
!
interface Ethernet0/2
 ip address 10.10.10.4 255.255.255.254
!
interface Ethernet0/3
 no ip address
!
router ospf 1
 router-id 1.1.1.255
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 network 1.1.1.255 0.0.0.0 area 0
 network 10.10.10.0 0.0.0.1 area 0
 network 10.10.10.2 0.0.0.1 area 0
 network 10.10.10.4 0.0.0.1 area 0

</code></pre>
</details>

Настройка SPINE коммутаторов:
<details>
<summary>NXOS-1</summary>
<pre><code>
SPINE-1# show run

feature ospf
feature interface-vlan
feature hsrp
feature lacp
feature vpc

interface Ethernet1/1
  no switchport
  ip address 10.10.10.1/31
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/4
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback0
  ip address 1.1.1.1/32
  ip router ospf UNDERLAY area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDERLAY
  router-id 1.1.1.1
  log-adjacency-changes detail

</code></pre>
</details>

<details>
<summary>NXOS-2</summary>
<pre><code>
SPINE-2# show run
feature ospf
feature interface-vlan
feature hsrp
feature lacp
feature vpc

interface Ethernet1/1
  no switchport
  ip address 10.10.10.3/31
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/4
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback0
  ip address 1.1.1.2/32
  ip router ospf UNDERLAY area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDERLAY
  router-id 1.1.1.2
  log-adjacency-changes detail

</code></pre>
</details>

<details>
<summary>NXOS-3</summary>
<pre><code>

SPINE-3# show run
feature ospf
feature interface-vlan
feature hsrp
feature lacp
feature vpc

interface Ethernet1/1
  no switchport
  ip address 10.10.10.5/31
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback0
  ip address 1.1.1.3/32
  ip router ospf UNDERLAY area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDERLAY
  router-id 1.1.1.3
  log-adjacency-changes detail

</code></pre>
</details>

Настройка LEAF коммутаторов:

<details>
<summary>NXOS-4</summary>
<pre><code>

feature ospf
feature interface-vlan
feature hsrp
feature lacp
feature vpc

ip prefix-list redistribute_list seq 5 permit 10.0.0.0/30
route-map OSPF-redistribute permit 10
  match ip address prefix-list redistribute_list

interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.0.0.1/30
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

  interface loopback0
  ip address 1.1.1.4/32
  ip router ospf UNDERLAY area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDERLAY
  router-id 1.1.1.4
  redistribute direct route-map OSPF-redistribute
  log-adjacency-changes detail

</code></pre>
</details>

<details>
<summary>NXOS-5</summary>
<pre><code>

feature vrrp
cfs eth distribute
feature ospf
feature interface-vlan
feature hsrp
feature lacp
feature vpc

vlan 1-2
vlan 2
  name Client-Vlan2

ip prefix-list redistribute_list seq 5 permit 10.0.1.0/24
route-map OSPF-redistribute permit 10
  match ip address prefix-list redistribute_list
vrf context VPC
vrf context management
vpc domain 1
  role priority 100
  peer-keepalive destination 10.200.100.2 source 10.200.100.1 vrf VPC


interface Vlan1

interface Vlan2
  no shutdown
  ip address 10.0.1.254/24
  vrrp 2
    priority 1
    address 10.0.1.1
    no shutdown

interface port-channel1
  description *** VPC PEERLINK ***
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel2
  switchport access vlan 2
  vpc 1

interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  switchport access vlan 2
  channel-group 2 mode active

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/4
  description *** VPC KEEPALIVE LINK ***
  no switchport
  vrf member VPC
  ip address 10.200.100.1/24
  no shutdown

interface Ethernet1/5
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  switchport mode trunk
  channel-group 1 mode active

interface loopback0
  ip address 1.1.1.5/32
  ip router ospf UNDERLAY area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDERLAY
  router-id 1.1.1.5
  redistribute direct route-map OSPF-redistribute
  log-adjacency-changes detail

</code></pre>
</details>

<details>
<summary>NXOS-6</summary>
<pre><code>

feature vrrp
cfs eth distribute
feature ospf
feature interface-vlan
feature hsrp
feature lacp
feature vpc

vlan 1-2
vlan 2
  name Client-Vlan2

ip prefix-list redistribute_list seq 5 permit 10.0.1.0/24
route-map OSPF-redistribute permit 10
  match ip address prefix-list redistribute_list
vrf context VPC
vrf context management
vpc domain 1
  role priority 1
  peer-keepalive destination 10.200.100.1 source 10.200.100.2 vrf VPC


interface Vlan1

interface Vlan2
  no shutdown
  ip address 10.0.1.253/24
  vrrp 2
    priority 2
    address 10.0.1.1
    no shutdown

interface port-channel1
  description *** VPC PEERLINK ***
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel2
  switchport access vlan 2
  vpc 1

interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  switchport access vlan 2
  channel-group 2 mode active

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/4
  description *** VPC KEEPALIVE LINK ***
  no switchport
  vrf member VPC
  ip address 10.200.100.2/24
  no shutdown

interface Ethernet1/5
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  switchport mode trunk
  channel-group 1 mode active

interface loopback0
  ip address 1.1.1.6/32
  ip router ospf UNDERLAY area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDERLAY
  router-id 1.1.1.6
  redistribute direct route-map OSPF-redistribute
  log-adjacency-changes detail

</code></pre>
</details>


<details>
<summary>NXOS-7</summary>
<pre><code>

cfs eth distribute
feature ospf
feature interface-vlan
feature hsrp
feature lacp
feature vpc

vlan 1

ip prefix-list redistribute_list seq 5 permit 10.0.2.0/30
route-map OSPF-redistribute permit 10
  match ip address prefix-list redistribute_list

interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.0.2.1/30
  no shutdown

interface loopback0
  ip address 1.1.1.7/32
  ip router ospf UNDERLAY area 0.0.0.0
cli alias name wr copy running-config startup-config
line console
line vty
boot nxos bootflash:/nxos.9.2.2.bin
router ospf UNDERLAY
  router-id 1.1.1.7
  redistribute direct route-map OSPF-redistribute
  log-adjacency-changes detail

</code></pre>
</details>


Настройка клиентских устройств:
<details>
<summary>R-9</summary>
<pre><code>

interface Ethernet0/0
 ip address 10.0.0.2 255.255.255.252

ip route 0.0.0.0 0.0.0.0 10.0.0.1

</code></pre>
</details>

<details>
<summary>R-11</summary>
<pre><code>

</code></pre>
</details>

<details>
<summary>R-10</summary>
<pre><code>

</code></pre>
</details>
