Цель: Настроить IS-IS для Underlay сети

1. Настроить IS-IS в Underlay сети, для IP связанности между всеми устройствами NXOS.
2. Проверка связности месжду сетями расположенных в одной локации, так и между локациями.
3. Предоставить вывод таблиц маршрутизации с утройств.

![](img/isis-chema.png)

Пояснение. На схеме отражена распределенная сеть построенная на базе протокола OSPF. Все маршрутизаторы находятся в единой Area 0. Связь между сетями происходит через единый маршрутизатор R-8.
Сеть имеет ряд недостатков:

1. Единая точка отказа. Маршрутизацтор R-8.
2. Отсутствие резервных линий связи от устройств NXOS 4-6 к R-8.
3. Не оптимальная настройки протокала OSPF.
4. Для подключения маршрутизатора R-10 используется протокол VRRP.

! В выводе убраны все настройки не относящиеся к поставленной задаче.

Настройка маршрутизаторов R-1 и R-2 Area 0:
<details>
<summary>R-1</summary>
<pre><code>

interface Loopback0
 ip address 1.1.1.255 255.255.255.255
 ip router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/0
 ip address 10.10.10.0 255.255.255.254
 ip router isis 1
 isis circuit-type level-2-only
 isis network point-to-point
!
interface Ethernet0/1
 ip address 10.10.10.2 255.255.255.254
 ip router isis 1
 isis circuit-type level-2-only
 isis network point-to-point
!
interface Ethernet0/2
 ip address 10.10.10.4 255.255.255.254
 ip router isis 1
 isis circuit-type level-2-only
 isis network point-to-point
!
interface Ethernet0/3
 no ip address
!
router isis 1
 net 49.0000.0000.0000.0255.00
 is-type level-2-only
 metric-style wide
 log-adjacency-changes
 maximum-paths 8

</code></pre>
</details>

<details>
<summary>R-2</summary>
<pre><code>

interface Loopback0
 ip address 1.1.1.254 255.255.255.255
 ip router isis 1
 isis circuit-type level-2-only
!
interface Ethernet0/0
 ip address 10.10.11.0 255.255.255.254
 ip router isis 1
 isis circuit-type level-2-only
 isis network point-to-point
!
interface Ethernet0/1
 ip address 10.10.11.2 255.255.255.254
 ip router isis 1
 isis circuit-type level-2-only
 isis network point-to-point
!
interface Ethernet0/2
 ip address 10.10.11.4 255.255.255.254
 ip router isis 1
 isis circuit-type level-2-only
 isis network point-to-point
!
interface Ethernet0/3
 no ip address
!
router isis 1
 net 49.0000.0000.0000.0254.00
 is-type level-2-only
 priority 127
 metric-style wide
 log-adjacency-changes
 maximum-paths 8


</code></pre>
</details>

Настройка маршрутизаторов NX-5 NX-9:
<details>
<summary>NX-5</summary>
<pre><code>

feature isis

interface Ethernet1/1
  no switchport
  ip address 10.10.10.5/31
  isis network point-to-point
  isis circuit-type level-2
  ip router isis 1
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.10.11.5/31
  isis network point-to-point
  isis circuit-type level-2
  ip router isis 1
  no shutdown

interface Ethernet1/3
  no switchport
  medium p2p
  ip unnumbered loopback0
  isis network point-to-point
  isis circuit-type level-1
  ip router isis 1
  no shutdown

  interface loopback0
    ip address 1.1.1.3/32
    ip router isis 1
  cli alias name wr copy running-config startup-config
  line console
  line vty
  no feature signature-verification
  router isis 1
    net 49.0010.0000.0000.0003.00
    distribute level-1 into level-2 all
    log-adjacency-changes

</code></pre>
</details>

<details>
<summary>NX-9</summary>
<pre><code>

feature isis

ip prefix-list redistribute_list seq 5 permit 10.0.2.0/24
route-map ISIS-redistribute permit 10
  match ip address prefix-list redistribute_list
vrf context management


interface Ethernet1/1
  no switchport
  medium p2p
  ip unnumbered loopback0
  isis network point-to-point
  isis circuit-type level-1
  ip router isis 1
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.0.2.1/24
  no shutdown

  interface loopback0
    ip address 1.1.1.7/32
    isis circuit-type level-1
    ip router isis 1
  cli alias name wr copy running-config startup-config
  line console
  line vty
  no feature signature-verification
  router isis 1
    net 49.0010.0000.0000.0007.00
    is-type level-1
    redistribute direct route-map ISIS-redistribute
    log-adjacency-changes


</code></pre>
</details>
