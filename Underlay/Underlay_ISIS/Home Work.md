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

Настройка маршрутизаторов R-1 и R-2:
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
