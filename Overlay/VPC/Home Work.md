Цель: Настроить Overlay на основе с использованием VPC;
      Маршрутизация в Overlay.

1. Организована связность между сетями 172.17.12.0/24 и 172.17.122.0/24 расположенными между разными LEAF коммутаторами. Локальный vlan-id 12 и 122.
2. Схема подключения к SPINE осталась без изменения, ее показывать не буду.
3. Для маршрутизации трафика внутри L3VNI используется
- vlan 66 на NX-1, NX-3, NX-11.
- vlan 99 на NX-9 и NX-10.

![](img/vxlan-route.png)

! В выводе убраны все настройки не относящиеся к поставленной задаче.

<details>
<summary>NX-1</summary>
<pre><code>
vlan 1,11-12,66,99
vlan 11
  vn-segment 11000
vlan 12
  vn-segment 12000
vlan 66
  name For-VXLAN-Routing
  vn-segment 6666

vrf context Vlan12-VRF
  vni 6666
  address-family ipv4 unicast
    route-target import 6666:6666
    route-target import 6666:6666 evpn
    route-target export 6666:6666
    route-target export 6666:6666 evpn
    route-target both auto
    route-target both auto evpn
vrf context management

interface Vlan1

interface Vlan11
  no shutdown
  ip address 172.17.11.254/24
  fabric forwarding mode anycast-gateway

interface Vlan12
  no shutdown
  vrf member Vlan12-VRF
  ip address 172.17.12.254/24
  fabric forwarding mode anycast-gateway

interface Vlan66
  no shutdown
  vrf member Vlan12-VRF
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 6666 associate-vrf
  member vni 11000
    ingress-replication protocol bgp
  member vni 12000
    ingress-replication protocol bgp

</code></pre>
</details>

<details>
<summary>NX-3</summary>
<pre><code>
vlan 1,11-12,66
vlan 11
  vn-segment 11000
vlan 12
  vn-segment 12000
vlan 66
  name For-VXLAN-Routing
  vn-segment 6666

vrf context Vlan12-VRF
  vni 6666
  address-family ipv4 unicast
    route-target import 6666:6666
    route-target import 6666:6666 evpn
    route-target export 6666:6666
    route-target export 6666:6666 evpn
    route-target both auto
    route-target both auto evpn

interface Vlan11
  no shutdown
  ip address 172.17.11.254/24
  fabric forwarding mode anycast-gateway

interface Vlan12
  no shutdown
  vrf member Vlan12-VRF
  ip address 172.17.12.254/24
  fabric forwarding mode anycast-gateway

interface Vlan66
  no shutdown
  vrf member Vlan12-VRF
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 6666 associate-vrf
  member vni 11000
    ingress-replication protocol bgp
  member vni 12000
    ingress-replication protocol bgp

</code></pre>
</details>

<details>
<summary>NX-11</summary>
<pre><code>
vlan 1,66,122
vlan 66
  name For-VXLAN-Routing
  vn-segment 6666
vlan 122
  vn-segment 12000

vrf context Vlan12-VRF
  vni 6666
  address-family ipv4 unicast
    route-target import 6666:6666
    route-target import 6666:6666 evpn
    route-target export 6666:6666
    route-target export 6666:6666 evpn
    route-target both auto
    route-target both auto evpn


interface Vlan66
  no shutdown
  vrf member Vlan12-VRF
  ip forward

interface Vlan122
  no shutdown
  vrf member Vlan12-VRF
  ip address 172.17.122.254/24
  fabric forwarding mode anycast-gateway

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 6666 associate-vrf
  member vni 12000
    ingress-replication protocol bgp

interface Ethernet1/1
  switchport access vlan 122

</code></pre>
</details>

<details>
<summary>NX-9</summary>
<pre><code>

vlan 12
  vn-segment 12000
vlan 99
  name For-VXLAN-Routing
  vn-segment 6666

vrf context VPC
vrf context Vlan12-VRF
  vni 6666
  address-family ipv4 unicast
    route-target import 6666:6666
    route-target import 6666:6666 evpn
    route-target export 6666:6666
    route-target export 6666:6666 evpn
    route-target both auto
    route-target both auto evpn

interface Vlan12
  no shutdown
  vrf member Vlan12-VRF
  ip address 172.17.12.254/24
  fabric forwarding mode anycast-gateway
interface Vlan99
  no shutdown
  vrf member Vlan12-VRF
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 6666 associate-vrf
  member vni 11000
    ingress-replication protocol bgp
  member vni 12000
    ingress-replication protocol bgp

</code></pre>
</details>

<details>
<summary>NX-10</summary>
<pre><code>
vlan 12
  vn-segment 12000
vlan 99
  name For-VXLAN-Routing
  vn-segment 6666

vrf context VPC
vrf context Vlan12-VRF
  vni 6666
  address-family ipv4 unicast
    route-target import 6666:6666
    route-target import 6666:6666 evpn
    route-target export 6666:6666
    route-target export 6666:6666 evpn
    route-target both auto
    route-target both auto evpn
vrf context management

interface Vlan11
  no shutdown
  ip address 172.17.11.254/24
  fabric forwarding mode anycast-gateway

interface Vlan12
  no shutdown
  vrf member Vlan12-VRF
  ip address 172.17.12.254/24
  fabric forwarding mode anycast-gateway

interface Vlan99
  no shutdown
  vrf member Vlan12-VRF
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 6666 associate-vrf
  member vni 11000
    ingress-replication protocol bgp
  member vni 12000
    ingress-replication protocol bgp
</code></pre>
</details>

! Проверка доступности маршрутизации внутри VNI
![](img/check-ping.png)
