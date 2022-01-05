Цель: Настроить BGP для Underlay сети


1. Настроить BGP в Underlay сети, для IP связанности между всеми устройствами NXOS.
2. Проверка связности между сетями расположенных в одной локации, так и между локациями.
3. Предоставить вывод таблиц маршрутизации с утройств.

![](img/bgp_schema.png)
Пояснение. На схеме отражена распределенная сеть построенная на базе протокола OSPF. Все маршрутизаторы находятся в единой Area 0. Связь между сетями происходит через единый маршрутизатор R-8.
Сеть имеет ряд недостатков:

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
!
router bgp 64512
 bgp router-id 1.1.1.255
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 1.1.1.255 mask 255.255.255.255
 neighbor 10.10.10.1 remote-as 64515
 neighbor 10.10.10.1 soft-reconfiguration inbound
 neighbor 10.10.10.3 remote-as 64516
 neighbor 10.10.10.3 soft-reconfiguration inbound
 neighbor 10.10.10.5 remote-as 64513
 neighbor 10.10.10.5 soft-reconfiguration inbound
 maximum-paths eibgp 4

</code></pre>
</details>

-----------------------------------------------------------------
<details>
<summary>Суммарная информация о BGP соседях с маршрутизатора R-8</summary>
<pre><code>

R-8#show ip bgp summary
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.1      4        64515      19      24       20    0    0 00:07:25       13
10.10.10.3      4        64516      19      24       20    0    0 00:07:24       13
10.10.10.5      4        64513      15      24       20    0    0 00:07:23        5

</code></pre>
</details>

<details>
<summary>Маршрутная таблица с R-8</summary>
<pre><code>
1.0.0.0/32 is subnetted, 8 subnets
B        1.1.1.1 [20/0] via 10.10.10.1, 00:08:54
B        1.1.1.2 [20/0] via 10.10.10.3, 00:08:54
B        1.1.1.3 [20/0] via 10.10.10.5, 00:08:54
B        1.1.1.4 [20/0] via 10.10.10.3, 00:08:54
           [20/0] via 10.10.10.1, 00:08:54
B        1.1.1.5 [20/0] via 10.10.10.3, 00:08:54
           [20/0] via 10.10.10.1, 00:08:54
B        1.1.1.6 [20/0] via 10.10.10.3, 00:08:54
           [20/0] via 10.10.10.1, 00:08:54
B        1.1.1.7 [20/0] via 10.10.10.5, 00:08:54
C        1.1.1.255 is directly connected, Loopback0
10.0.0.0/8 is variably subnetted, 16 subnets, 4 masks
B        10.0.0.0/30 [20/0] via 10.10.10.3, 00:08:54
               [20/0] via 10.10.10.1, 00:08:54
B        10.0.1.0/24 [20/0] via 10.10.10.3, 00:08:54
               [20/0] via 10.10.10.1, 00:08:54
B        10.0.2.0/30 [20/0] via 10.10.10.5, 00:08:54
B        10.1.4.0/31 [20/0] via 10.10.10.3, 00:08:54
               [20/0] via 10.10.10.1, 00:08:54
B        10.1.5.0/31 [20/0] via 10.10.10.3, 00:08:54
               [20/0] via 10.10.10.1, 00:08:54
B        10.1.6.0/31 [20/0] via 10.10.10.3, 00:08:54
               [20/0] via 10.10.10.1, 00:08:54
B        10.2.4.0/31 [20/0] via 10.10.10.3, 00:08:54
               [20/0] via 10.10.10.1, 00:08:54
B        10.2.5.0/31 [20/0] via 10.10.10.3, 00:08:54
               [20/0] via 10.10.10.1, 00:08:54
B        10.2.6.0/31 [20/0] via 10.10.10.3, 00:08:54
               [20/0] via 10.10.10.1, 00:08:54
B        10.3.7.0/31 [20/0] via 10.10.10.5, 00:08:54
C        10.10.10.0/31 is directly connected, Ethernet0/0
L        10.10.10.0/32 is directly connected, Ethernet0/0
C        10.10.10.2/31 is directly connected, Ethernet0/1
L        10.10.10.2/32 is directly connected, Ethernet0/1
C        10.10.10.4/31 is directly connected, Ethernet0/2
L        10.10.10.4/32 is directly connected, Ethernet0/2

</code></pre>
</details>
