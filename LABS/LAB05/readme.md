# Настройка протоколов VxLAN\EVPN (L2 VNI) в сети Дата-центра DeLFI-UP

## Цель:
***1.Настроить протоколы VxLAN\EVPN в Overlay сети датацентра, для организации L2 связанности между сервером и удаленным рабочим местом расположенных на разных гипервизорах VWware ESXI;<br>***

## План работ

1. Настроить протокол BGP для работы с VxLAN\EVPN;
2. Сконфигурировать протоколы VxLAN\EVPN;
3. Проверить работу Overlay сети дата-центра DeLFI-UP;

## Описание 
Прошло немало времени с того времени как дата-центр DeLFI-UP начал свою работу. Сегодня это один из ведущих дата-центров в регионе. О нем известно многим. Даже тем кто связан сетями и IT технологиями лишь по наслышке. Во многом эта заслуга сетевых инженеров дата-центра.  
Однажды в дата-центр обратился один клиент который захотел организовать доступ к 1С удаленно. К сожалению у него нет своих IT специалистов которые бы настроили сервер и организовали бы VPN туннели. <br> От дата-центра требуется организовать L2 связность между двумя виртуальными машинами расположенными на разных гипервизорах. 

-***"Это просто"*** с улыбкой сказал один из начинающих сотрудников работающий сетевым техником на пол ставки. <br>

-***"Просто пробросить VLAN между Spine к Leaf. И можно пить чай".***  Глубоко вздохнув, другой сетевой инженер который не первый год работает с сетями передачи данных сказал в слух <br>
-***"Ага ты еще Микротик поставь между Spine и Leaf специально для VLAN-ов" А лучше пачку Микротиков, для резервирования"*** 
 
 -***"А что плохого в Микротике"?*** спросил молодой техник 
 
 -***"Такие задачи решаются соврешенно другими технологиями. И одни из них это VxLAN\EVPN"***, попровляя очки сказал опытный инженер.<br>
  -***Vx-Что? Виски-влан?*** повторил молодой техник удивленнным голосом.<br>
 -***"Ты только о виски и думаешь"*** пробурчал инженер. 
 ***Линк пропадет со Spine как ты доступность виртуалки обеспечишь с другого коммутатора?***<br>
 -"***Я как-то не подумал***" грустно ответил техник понимая что сказал глупость.<br>
 -***"Поэтому бери стул и садись поближе будем настраивать Виски-Vlan твой"...***<br>  Сказал инженер и запустил SSH клиент на ноутбуке....
 


## Схема сети
Для орагнизации Overlay-сети по протоколам VxLAN\EVPN в нашем дата-центре нам необходимо настроить протоколы ***BGP, EVPN, VxLAN*** Также условимся что Ethernet и Loopback адреса уже настроенны согласно нашему IP плану и схемы сети. Имеется IP связность по протоколу OSPF <br>


![Схема сети](https://raw.githubusercontent.com/DeLFI901/OTUS_NETWORK_DATACENTER/refs/heads/main/LABS/LAB05/DC-NETWORK-VxLAN-L2-VNI.jpeg "Схема организации подключения Underlay сети дата-центра DeLFI-UP по протоколу eBGP")

## Конфигурация устройств

Для правильной работы  протоколов VxVLAN и EVPN  в первую очередь нам необходимо включить поддержку коммутатором этих протоколов а затем сконфигурировать их на всех устройствах. В этом примере мы будем использовать коммутаторы ***Spine-1***, ***Leaf-1*** и ***Leaf-3*** для настроек. На остальных устройствах необходимо выполнить аналогичные команды учитывая необходимую IP адресацию и идентификаторы сети ***(VLAN,VNI,VTEP)*** которые отображены на схеме.  

Произведем установку компонентов протокола VxLAN и EVPN в коммутатор:

``` Spine-1(config)# feature nv overlay``` <br>
``` Spine-1(config)# feature vn-segment-vlan-based```<br>
``` Spine-1(config)# nv overlay evpn```

После установки в систему проверим доступные протоколы для работы:

```
Spine-1(config)# show feature                                                                                                                                                                                    
Feature Name          Instance  State                                                                                                                                                                            
---------------------------------------------
hmm                    1          enabled 
icam                   1          enabled
nve                    1          enabled 
---------------------------------------------
```
В списке доступных компонентов находим hmm,nve  и их статус ***Enabled*** которые говорят о том что данные протоколы доступны для работы на нашем оборудовании.

Производим настройку протокола BGP для работы с EVPN на коммутаторе Leaf-1: Данный коммутатор будет работать в приватной автономной системе AS65000:

```
Leaf-1(config)# router bgp 65000
```
Настроим Router-id
```
Leaf-1(config-router)# router-id 10.201.101.1
```
Настроим тамеры:
```
Leaf-1(config-router)# timers bgp 3 9 
```

Настроим интервал повторного переподключения:

```
Leaf-1(config-router)# reconnect-interval 12 
```

Настроим логирование событий:
```
Leaf-1(config-router)# log-neighbor-changes
```

Сконфигурируем семейство адресов для работы L2 EVPN:

```
Leaf-1(config-router)#  address-family l2vpn evpn
```

Включим поддержу ECMP в EVPN:

```
Leaf-1(config-router-af)# maximum-paths 2
```

Активируем соседство с ***Leaf-3*** коммутатором, добавим описание данного соеднинения и будем строить соседство с Loopback адреса 

```  
neighbor 10.203.101.1
remote-as 65000
description Peer-to-Leaf-3
update-source loopback0 
```
Включим подержку L2VPN EVPN для данного соседа:

```
address-family l2vpn evpn
send-community
send-community extended
```

Также по аналогии сделаем соседство со ***Spine-1*** коммутатором:

```  
neighbor 10.101.101.1
remote-as 65000
description Peer-to-Spine-1
address-family l2vpn evpn
send-community
send-community extended
```

Вы заметите, что ваш обычный запрос show ip bgp summary не возвращает никакой информации о ваших соседях. Это ожидаемое поведение, поскольку вы подключаетесь к семейству адресов “evpn”.
Проверим поднялось ли соседство в bgp address-family l2vpn:
```
Leaf-1# sh bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.201.101.1, local AS number 65000
BGP table version is 15, L2VPN EVPN config peers 2, capable peers 2
3 network entries and 4 paths using 876 bytes of memory
BGP attribute entries [3/1104], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [1/4]
Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.101.101.1    4 65000       4553       4552       15    0    0 03:47:22 1
10.203.101.1    4 65000       4427       4428       15    0    0 03:41:09 1

Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5     Type-12
10.101.101.1    I 65000 1          0          1          0          0          0

10.203.101.1    I 65000 1          0          1          0          0          0

```

Переходим к конфигурации VxLAN. Создадим VLAN10 и свяжем его с VNI-10
```
vlan 10 
name EVPN-VLAN-10
vn-segment 10
```

Настроим Ethernet интерфейс куда у нас подключен сервер 1С:
```
interface Ethernet1/5
description LINK-TO-Client1
switchport
switchport access vlan 10
no shutdown
```
Настроим EVPN для работы с протокололом BGP:
```
evpn  
vni 10 l2
rd auto
route-target import auto
route-target export auto
```
Настроим VTEP:
```
interface nve1
host-reachability protocol bgp
source-interface loopback0
member vni 10
ingress-replication protocol bgp
```
На коммутаторе ***Leaf-3*** делаем аналогичные настройки и проверяем работу VxLAN\EVPN:

```
Leaf-1# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac
--------- --------------------------------------  ----- --------- -------- ----------------
nve1      10.203.101.1                            Up    CP        03:59:52 n/a
```
```
Leaf-1# show nve vni
Codes: CP - Control Plane        DP - Data Plane
       UC - Unconfigured         SA - Suppress ARP
       S-ND - Suppress ND
       SU - Suppress Unknown Unicast
       Xconn - Crossconnect
       MS-IR - Multisite Ingress Replication
       HYB - Hybrid IRB mode
       
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
nve1      10       UnicastBGP        Up    CP   L2 [10]
```
```
Leaf-1# show vxlan
Vlan            VN-Segment
====            ==========
10              10
```
```
Leaf-1# show l2route evpn mac all 
Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Asy):Asymmetric (Gw):Gateway
(Bh):Blackhole
(Pf):Permanently-Frozen, (Orp): Orphan 

(PipOrp): Directly connected Orphan to PIP based vPC BGW 
(PipPeerOrp): Orphan connected to peer of PIP based vPC BGW 
Topology    Mac Address    Prod   Flags              Seq No     Next-Hops
----------- -------------- ------ ------------------- ---------- ---------------
10          0050.7966.6807 Local  L,                 0          Eth1/5 
10          0050.7966.6808 BGP    Rcv                0          10.203.101.1 (Label: 10)

```

Проверим доступность клиента с сервера 1С во VLAN-10 находящегося на другом гипервизоре:
```
VPCS> ping 192.168.10.20 
84 bytes from 192.168.10.20 icmp_seq=1 ttl=64 time=8.478 ms 
84 bytes from 192.168.10.20 icmp_seq=2 ttl=64 time=2.375 ms
84 bytes from 192.168.10.20 icmp_seq=3 ttl=64 time=2.465 ms
84 bytes from 192.168.10.20 icmp_seq=4 ttl=64 time=3.271 ms 
84 bytes from 192.168.10.20 icmp_seq=5 ttl=64 time=7.287 ms
```


Конфигурация ***Leaf-1*** коммутатора:

```
!Command: show running-config 
!Running configuration last done at: Sun Apr 13 14:56:57 2025 
!Time: Sun Apr 13 15:50:19 2025                                                                                                                                                                                                                                                                                                                                                                                                   version 10.4(2) Bios:version                                                                                                                                                                                     hostname Leaf-1                                                                                                                                                                                                  vdc Leaf-1 id 1                                                                                                                                                                                                    limit-resource vlan minimum 16 maximum 4094                                                                                                                                                                      limit-resource vrf minimum 2 maximum 4097                                                                                                                                                                        limit-resource port-channel minimum 0 maximum 511                                                                                                                                                                limit-resource m4route-mem minimum 58 maximum 58                                                                                                                                                                 limit-resource m6route-mem minimum 8 maximum 8                                                                                                                                                                                                                                                                                                                                                                                  nv overlay evpn
feature ospf
feature bgp
feature isis
feature vn-segment-vlan-based
feature bfd
feature nv overlay

no password strength-check
username admin password 5 $5$EHLMCA$yjiBuJ2MMKuB2wINWzsyYhVzwUbTRvxO.BSbFlZRYI9
 role network-admin
ip domain-lookup
copp profile strict
bfd interval 500 min_rx 500 multiplier 3
snmp-server user admin network-admin auth md5 52324031F6D8DE1E331D54DAD6245D4AD8
D8 priv aes-128 21653F449ED1D9443C512980802C4E4BE0B9 localizedV2key
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO

vlan 1,10
vlan 10
  name EVPN-VLAN-10
  vn-segment 10
route-map REDIST_LOOPBACK permit 10
  match interface loopback0
vrf context management


interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 10
    ingress-replication protocol bgp

interface Ethernet1/1
  description LINK-TO-Spine-1
  no ip redirects
  ip address 10.101.109.2/30
  ip ospf message-digest-key 0 md5 3 a01efc0492d53b40c0b3aa9b18731403
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 100 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  description LINK-TO-Spine-2
  ip address 10.102.109.2/30
  ip ospf message-digest-key 0 md5 3 a01efc0492d53b40c0b3aa9b18731403
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 100 area 0.0.0.0
  no shutdown
  
interface Ethernet1/5
  description LINK-TO-Client1
  switchport
  switchport access vlan 10
  no shutdown
  
interface loopback0 
ip address 10.201.101.1/32 

icam monitor scale

line console
line vty
boot nxos bootflash:/nxos64-cs.10.4.2.F.bin
router ospf 100
router-id 10.201.101.1
redistribute direct route-map REDIST_LOOPBACK 
log-adjacency-changes
maximum-paths 2
passive-interface default
router bgp 65000
  router-id 10.201.101.1
  timers bgp 3 9
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 2
  neighbor 10.101.101.1
    remote-as 65000
    description Peer-to-Spine-1
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.203.101.1
    remote-as 65000
    description Peer-to-Leaf-3
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
evpn
  vni 10 l2
    rd auto
route-target import auto
route-target export auto
```

Конфигурация ***Spine-1***

```
!Command: show running-config 
!No configuration change since last restart 
!Time: Sun Apr 13 16:03:11 2025                                                                                                                                                                                                                                                                                                                                                                                                   version 10.4(2) Bios:version                                                                                                                                                                                     hostname Spine-1                                                                                                                                                                                                 vdc Spine-1 id 1                                                                                                                                                                                                   limit-resource vlan minimum 16 maximum 4094                                                                                                                                                                      limit-resource vrf minimum 2 maximum 4097                                                                                                                                                                        limit-resource port-channel minimum 0 maximum 511                                                                                                                                                                limit-resource m4route-mem minimum 58 maximum 58                                                                                                                                                                 limit-resource m6route-mem minimum 8 maximum 8                                                                                                                                                                                                                                                                                                                                                                                  nv overlay evpn                                                                                                                                                                                                  feature ospf
feature bgp
feature isis
feature vn-segment-vlan-based
feature bfd
feature nv overlay

no password strength-check
username admin password 5 $5$OJIDIK$zPPAGeIeiUidwYquVdIJgfcG2tyUiIPUT0nbI9Ti9A4
 role network-admin
ip domain-lookup
copp profile strict
bfd interval 500 min_rx 500 multiplier 3
snmp-server user admin network-admin auth md5 5331FAF6799A7DEB896C355F9F45DA50AF
D8 priv aes-128 174896B88D60970C7286B09944DD3ED3204E localizedV2key
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO

vlan 1

route-map REDIST_LOOPBACK permit 10
  match interface loopback0
vrf context management

interface Ethernet1/1 
  no ip redirects
  description LINK-TO-Leaf-1 
  ip address 10.101.109.1/30
  ip ospf message-digest-key 0 md5 3 a01efc0492d53b40c0b3aa9b18731403
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 100 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  description LINK-TO-Leaf-2
  ip address 10.101.109.5/30
  ip ospf message-digest-key 0 md5 3 a01efc0492d53b40c0b3aa9b18731403
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 100 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description LINK-TO-Leaf-3
  ip address 10.101.109.9/30
  ip ospf message-digest-key 0 md5 3 a01efc0492d53b40c0b3aa9b18731403
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf 100 area 0.0.0.0
  no shutdown
  
interface Ethernet1/7
  description LINK-TO-SupeSpine-1
  ip address 10.111.109.2/30
  no shutdown
  
interface mgmt0 
  vrf member management                                                                                                                                                                                                                                                                                                                                                                                                           interface loopback0                                                                                                                                                                                                ip address 10.101.101.1/32                                                                                                                                                                                     icam monitor scale                                                                                                                                                                                                                                                                                                                                                                                                                line console
line vty
boot nxos bootflash:/nxos64-cs.10.4.2.F.bin
router ospf 100
  router-id 10.101.101.1
  redistribute direct route-map REDIST_LOOPBACK
  log-adjacency-changes
  maximum-paths 2
  passive-interface default
router bgp 65000
  router-id 10.101.101.1
  timers bgp 3 9
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 2
  neighbor 10.201.101.1
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
  neighbor 10.203.101.1
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
      route-reflector-client
```
