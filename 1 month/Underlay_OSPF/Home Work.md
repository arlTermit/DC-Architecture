Цель: Настроить OSPF для Underlay сети

1. Настроить OSPF в Underlay сети, для IP связанности между всеми устройствами NXOS.
2. Проверка связности месжду сетями расположенных в одной локации, так и между локациями.
3. Предоставить вывод таблиц маршрутизации с утройств.

Пояснение. На схеме отражена сеть построенная на базе протокола OSPF. Все маршрутизаторы находятся в единой Area 0. Связь между сетями происходит через единый маршрутизатор R-8.
Сеть имеет ряд недостатков:

1.
2.
3.
4.

! В выводе убраны все настройки не относящиеся к поставленной задаче.

Настройка R-8:
 <details>
<summary>R-8</summary>
<pre><code>
Router>en
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

Router#
</code></pre>
</details>
