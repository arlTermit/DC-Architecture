Цель: Настроить BGP для Underlay сети

1. Настроить BGP в Underlay сети, для IP связанности между всеми устройствами NXOS

1. Настроить BGP в Underlay сети, для IP связанности между всеми устройствами NXOS.
2. Проверка связности месжду сетями расположенных в одной локации, так и между локациями.
3. Предоставить вывод таблиц маршрутизации с утройств.

![](img/bgp_schema.png)
Пояснение. На схеме отражена распределенная сеть построенная на базе протокола OSPF. Все маршрутизаторы находятся в единой Area 0. Связь между сетями происходит через единый маршрутизатор R-8.
Сеть имеет ряд недостатков:

1. Единая точка отказа. Маршрутизацтор R-8.
2. Отсутствие резервных линий связи от устройств NXOS 4-6 к R-8.
3. Не оптимальная настройки протокала OSPF.
4. Для подключения маршрутизатора R-10 используется протокол VRRP.

! В выводе убраны все настройки не относящиеся к поставленной задаче.

Настройка маршрутизатора R-8:
<details>
<summary>R-8</summary>
<pre><code>

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
router bgp 64512
 bgp router-id 1.1.1.255
 bgp log-neighbor-changes
 network 1.1.1.255 mask 255.255.255.255
 neighbor 10.10.10.1 remote-as 64515
 neighbor 10.10.10.1 soft-reconfiguration inbound
 neighbor 10.10.10.3 remote-as 64516
 neighbor 10.10.10.3 soft-reconfiguration inbound
 neighbor 10.10.10.5 remote-as 64513
 neighbor 10.10.10.5 soft-reconfiguration inbound


</code></pre>
</details>

Настройка SPINE коммутаторов:

<details>
<summary>NXOS-1</summary>
<pre><code>

feature ospf
feature bgp
feature interface-vlan
feature hsrp
feature lacp
feature vpc

interface Ethernet1/1
  no switchport
  ip address 10.10.10.1/31
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip address 10.1.4.0/31
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip address 10.1.5.0/31
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/4
  no switchport
  medium p2p
  ip address 10.1.6.0/31
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
router bgp 64515
  router-id 1.1.1.1
  bestpath as-path multipath-relax
  log-neighbor-changes
  address-family ipv4 unicast
    network 1.1.1.1/32
  template peer LEAF
    address-family ipv4 unicast
      maximum-prefix 100
  neighbor 10.1.4.1
    inherit peer LEAF
    remote-as 64517
  neighbor 10.1.5.1
    inherit peer LEAF
    remote-as 64518
  neighbor 10.1.6.1
    inherit peer LEAF
    remote-as 64519
  neighbor 10.10.10.0
    remote-as 64512
    address-family ipv4 unicast
      maximum-prefix 200

</code></pre>
</details>

<details>
<summary>NXOS-2</summary>
<pre><code>

feature ospf
feature bgp
feature interface-vlan
feature hsrp
feature lacp
feature vpc

interface Ethernet1/1
  no switchport
  ip address 10.10.10.3/31
  no shutdown

interface Ethernet1/2
  no switchport
  medium p2p
  ip address 10.2.4.0/31
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip address 10.2.5.0/31
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/4
  no switchport
  medium p2p
  ip address 10.2.6.0/31
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
router bgp 64516
  router-id 1.1.1.2
  bestpath as-path multipath-relax
  log-neighbor-changes
  address-family ipv4 unicast
    network 1.1.1.2/32
    redistribute direct route-map direct
    maximum-paths 2
  template peer LEAF
    address-family ipv4 unicast
      maximum-prefix 100
  neighbor 10.2.4.1
    inherit peer LEAF
    remote-as 64517
  neighbor 10.2.5.1
    inherit peer LEAF
    remote-as 64518
  neighbor 10.2.6.1
    inherit peer LEAF
    remote-as 64519
  neighbor 10.10.10.2
    remote-as 64512
    address-family ipv4 unicast
      maximum-prefix 200

</code></pre>
</details>

<details>
<summary>NXOS-3</summary>
<pre><code>

</code></pre>
</details>
