# Настройка протокола eBGP сети Дата-центра DeLFI-UP

## Цель:
***1.Настроить протокол eBGP в Underlay сети датацентра, для IP связанности между всеми сетевыми устройствами и автономными системами;<br>***

## План работ

1. Настроить route-map для ананосирования Loopback адресов;
2. Сконфигурировать протокол динамической маршрутизации eBGP;
3. Проверить работу Underlay сети дата-центра DeLFI-UP;

## Описание 
После внедрения протокола IS-IS дата-центр DeLFI-UP успешно работает. Появляются все новые и новые клиенты, которые используют серверные мощности нашего ЦОД.  
Среди клиентов появились крупные заказчики такие как банки и государственные органы. Совсем скоро Дата-центр DeLFI-UP готовится получить сертификацию Tier III.  Вместе с клиентами растет и количество трафика поступающего в наш ЦОД. Исходя из этого в пятницу вечером было принято решение о внедрении протокола eBGP. Рассказав сетевым инженерам обслуживающим наш дата-центр о будущей идеи внедрении протокола  eBGP и показав будущую схему сети  автору было рекомендовано сходить поспать. Но показав схему будущей сети инвесторам было получено финансирование на приобретение трех коммутаторов уровня ***SuperSpine***.<br> К сожалению, поставщик сетевого оборудования смог предоставить только один такой коммутатор остальные будут изготавливаться на заводе Cisco специально для нашего дата-центра.Однако из-за сильной загруженности завода изготовителя коммутаторы будут готовы не скоро. Ориентировочные сроки поставки коммутаторов конец года. <br>
Глубоко вздохнув, берем ноутбук и консольный кабель и идем к телекоммуникационной стойке заниматься монтажом и настраивать наше оборудование… 


## Схема сети
Для орагнизации Underlay-сети по протоколу eBGP в нашем дата-центре нам необходимо настроить ***route-map*** для ананоса Loopback адреса другим маршрутизаторам в дата-центре. Также условимся что Ethernet и Loopback адреса уже настроенны согласно нашему IP плану и схемы сети. Далее потребуется активировать процесс eBGP и сконфигурировать его на каждом L3-коммутаторе<br>


![Схема сети](https://raw.githubusercontent.com/DeLFI901/OTUS_NETWORK_DATACENTER/refs/heads/main/LABS/LAB04/DC-NETWORK-eBGP.jpeg "Схема организации подключения Underlay сети дата-центра DeLFI-UP по протоколу eBGP")

## Конфигурация устройств

Для правильной работы  протокола eBGP в первую очередь нам необходимо сконфигурировать route-map на всех устройствах.  Переходим в режим глобальной конфигурации устройства и руководствуясь нашим IP-планом производим конфигурацию route-mapa  для возможности анансирования его другим устройствам. В этом примере мы будем использовать коммутатор ***Spine-1*** для настроек. На остальных устройствах необходимо выполнить аналогичные команды учитывая необходимую IP адресацию и номера автономных систем которые отображены на схеме. Также условимся что коммутатор ***SuperSpine-1*** уже базово настроен на нем корерректно происходит редистрбьюция Loopback адреса и он ожидает подключение по протоколу eBGP от соседних маршрутизаторов.  

Произведем установку компонентов протокола eBGP в коммутатор:

``` Spine-1(config)# feature bgp```<br> 


После установки в систему проверим доступные протоколы для работы:

```
Spine-1(config)# show feature                                                                                                                                                                                    
Feature Name          Instance  State                                                                                                                                                                            
---------------------------------------------
bgp                    1        enabled
---------------------------------------------
```
В списке доступных компонентов находим BGP и его статус ***Enabled*** которые говорят о том что данный протокол доступен для работы на нашем оборудовании.

Производим настройку route-map:

```
Spine-1(config)# route-map REDIST_LOOPBACK permit 10
Spine-1(config-route-map)# match interface loopback 0  
```

После настройки route-map переходим к конфигурации протокола eBGP. Данный коммутатор будет работать в приватной автономной системе AS65000:

```
Spine-1(config)# router bgp 65000
```
Настроим Router-id
```
Spine-1(config-router)# router-id 10.101.101.1
```

Настроим интервал повторного переподключения:

```
Spine-1(config-router)# reconnect-interval 12 
```

Сконфигурируем семейство адресов IPv4 глобально для всего процесса 

```
Spine-1(config-router)# address-family ipv4 unicast
```

Проиведем редистрибьюцию подклюеного IP адреса Loopback интерфейса в протокол eBGP:

```
Spine-1(config-router-af)# redistribute direct route-map REDIST_LOOPBACK 
```

Активируем соседство с ***Leaf-1*** коммутатором с ***AS65001*** и добавим описание данного соеднинения

```  
neighbor 10.101.109.2
remote-as 65001
description Peer-to-Leaf-1
```
Чтобы никто посторонний не смог подключится к нашему процессу eBGP активируем его  работу через аутенфикацию

  ```
  password cisco2008 (где cisco2008 наш секретный пароль)
  ```
Немного иземеним таймеры для более быстрого реагирования процесса eBGP на изменения в сети:

```
timers 3 9 (где первое значение Keepalive interval в секундах а второе время Holdtime)
```

Активируем поддержку ECMP

```
 maximum-paths 10  (значение 10 установленно с заделом на будущее. В данный момент у нас будет 2 активных маршрута от Spine коммутаторов к Leaf коммутаторам).
```

Сконфигурируем семейство адресов IPv4  для данного соседа:
  
  ```
  address-family ipv4 unicast
```

Передадим адрес шлюза-по умолчанию нашему соседу. Данный коммутатор будет являтся шлюзом последней надежды для Leaf-коммутаторов

```
default-originate
```
По аналогии настраиваем соседей с Leaf-2 и Leaf-3 коммутатором. Убеждаемся что соседство поднялось со всеми соседними коммутаторами. 

Проверяем состояние протокола BGP на Leaf-1 коммутаторе: 
```
Leaf-1# sh ip bgp
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 36, Local Router ID is 10.201.101.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*|e0.0.0.0/0          10.101.109.1                                   0 65000 i
*>e                   10.102.109.1                                   0 65000 i
*>e10.101.101.1/32    10.101.109.1             0                     0 65000 ?
*>e10.102.101.1/32    10.102.109.1             0                     0 65000 ?
*|e10.111.101.1/32    10.102.109.1                                   0 65000 645
12 ?
*>e                   10.101.109.1                                   0 65000 645
12 ?
*>r10.201.101.1/32    0.0.0.0                  0        100      32768 ?
*|e10.202.101.1/32    10.101.109.1                                   0 65000 650
02 ?                                                                                                                                                                                                             *>e                   10.102.109.1                                   0 65000 650                                                                                                                                 02 ?                                                                                                                                                                                                             *|e10.203.101.1/32    10.101.109.1                                   0 65000 650                                                                                                                                 03 ?                                                                                                                                                                                                             *>e                   10.102.109.1                                   0 65000 650                                                                                                                                 03 ?                                                                                                                                                                                                                                                                                                                                                                                                                              

```
Проверим соседство со Spine-коммутаторами
```
Leaf-1# sh ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.201.101.1, local AS number 65001
BGP table version is 36, IPv4 Unicast config peers 2, capable peers 2
7 network entries and 11 paths using 2428 bytes of memory
BGP attribute entries [6/2208], BGP AS path entries [4/36] 
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.101.109.1    4 65000        568        550       36    0    0 00:10:59 5

10.102.109.1    4 65000        310        300       36    0    0 00:10:56 5
```

Посмотрим на маршруты полученные от протокола eBGP 
```
Leaf-1# sh ip route bgp-65001
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via outp0.0.0.0/0, ubest/mbest: 2/0
    *via 10.101.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000
    *via 10.102.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000ut denotes VRF <string>
10.101.101.1/32, ubest/mbest: 1/0
    *via 10.101.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000
10.102.101.1/32, ubest/mbest: 1/0 
 *via 10.102.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000
10.111.101.1/32, ubest/mbest: 2/0
    *via 10.101.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000
    *via 10.102.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000
10.202.101.1/32, ubest/mbest: 2/0 
   *via 10.101.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000
    *via 10.102.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000 
10.203.101.1/32, ubest/mbest: 2/0 
 *via 10.101.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000
 *via 10.102.109.1, [20/0], 00:22:32, bgp-65001, external, tag 65000 
  ```

Проверим доступность ***SuperSpine-1*** коммутатора

```
Spine-1# ping 10.111.101.1
PING 10.111.101.1 (10.111.101.1): 56 data bytes
64 bytes from 10.111.101.1: icmp_seq=0 ttl=254 time=1.663 ms
64 bytes from 10.111.101.1: icmp_seq=1 ttl=254 time=1.164 ms
64 bytes from 10.111.101.1: icmp_seq=2 ttl=254 time=1.033 ms
64 bytes from 10.111.101.1: icmp_seq=3 ttl=254 time=0.77 ms
64 bytes from 10.111.101.1: icmp_seq=4 ttl=254 time=0.658 ms

--- 10.111.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.658/1.057/1.663 ms
```

Проверим доступность ***Spine-2*** коммутатора
```
Spine-1# ping 10.102.101.1
PING 10.102.101.1 (10.102.101.1): 56 data bytes
64 bytes from 10.102.101.1: icmp_seq=0 ttl=253 time=1.79 ms
64 bytes from 10.102.101.1: icmp_seq=1 ttl=253 time=1.124 ms
64 bytes from 10.102.101.1: icmp_seq=2 ttl=253 time=1.142 ms
64 bytes from 10.102.101.1: icmp_seq=3 ttl=253 time=1.059 ms
64 bytes from 10.102.101.1: icmp_seq=4 ttl=253 time=1.12 ms

--- 10.102.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 1.059/1.247/1.79 ms
--- 10.102.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.659/0.86/1.408 ms
```
Проверим доступность ***Leaf-1*** коммутатора
```
Spine-1# ping 10.201.101.1
PING 10.201.101.1 (10.201.101.1): 56 data bytes
64 bytes from 10.201.101.1: icmp_seq=0 ttl=254 time=1.282 ms
64 bytes from 10.201.101.1: icmp_seq=1 ttl=254 time=0.916 ms
64 bytes from 10.201.101.1: icmp_seq=2 ttl=254 time=0.814 ms
64 bytes from 10.201.101.1: icmp_seq=3 ttl=254 time=0.638 ms
64 bytes from 10.201.101.1: icmp_seq=4 ttl=254 time=0.608 ms

--- 10.201.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.608/0.851/1.282 ms
```
Проверим доступность ***Leaf-2*** коммутатора
```
Spine-1# ping 10.202.101.1
PING 10.202.101.1 (10.202.101.1): 56 data bytes
64 bytes from 10.202.101.1: icmp_seq=0 ttl=254 time=1.117 ms
64 bytes from 10.202.101.1: icmp_seq=1 ttl=254 time=0.664 ms
64 bytes from 10.202.101.1: icmp_seq=2 ttl=254 time=0.537 ms
64 bytes from 10.202.101.1: icmp_seq=3 ttl=254 time=0.61 ms
64 bytes from 10.202.101.1: icmp_seq=4 ttl=254 time=0.567 ms

--- 10.202.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.537/0.699/1.117 ms
```
Проверим доступность ***Leaf-3*** коммутатора

```
Spine-1# ping 10.203.101.1
PING 10.203.101.1 (10.203.101.1): 56 data bytes
64 bytes from 10.203.101.1: icmp_seq=0 ttl=254 time=1.327 ms
64 bytes from 10.203.101.1: icmp_seq=1 ttl=254 time=0.985 ms
64 bytes from 10.203.101.1: icmp_seq=2 ttl=254 time=0.948 ms
64 bytes from 10.203.101.1: icmp_seq=3 ttl=254 time=0.684 ms
64 bytes from 10.203.101.1: icmp_seq=4 ttl=254 time=0.7 ms

--- 10.203.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.684/0.928/1.327 ms
```

Конфигурация ***Spine-1*** коммутатора:
```
Spine-1# sh run                                                                                                                                                                                                                                                                                                                                                                                                                   !Command: show running-config                                                                                                                                                                                    !Running configuration last done at: Sat Mar 29 14:52:50 2025                                                                                                                                                    !Time: Sat Mar 29 15:46:59 2025                                                                                                                                                                                                                                                                                                                                                                                                   version 10.4(2) Bios:version                                                                                                                                                                                     hostname Spine-1                                                                                                                                                                                                 vdc Spine-1 id 1                                                                                                                                                                                                   limit-resource vlan minimum 16 maximum 4094                                                                                                                                                                      limit-resource vrf minimum 2 maximum 4097                                                                                                                                                                        limit-resource port-channel minimum 0 maximum 511                                                                                                                                                                limit-resource m4route-mem minimum 58 maximum 58                                                                                                                                                                 limit-resource m6route-mem minimum 8 maximum 8                                                                                                                                                                                                                                                                                                                                                                                  feature ospf
feature bgp
feature isis
feature bfd

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
  description LINK-TO-Leaf-1
  no ip redirects
  ip address 10.101.109.1/30
  no shutdown

interface Ethernet1/2
  description LINK-TO-Leaf-2
  ip address 10.101.109.5/30
  no shutdown

interface Ethernet1/3
  description LINK-TO-Leaf-3
  ip address 10.101.109.9/30
  no shutdown
  
  interface Ethernet1/7
  description LINK-TO-SupeSpine-1
  ip address 10.111.109.2/30
  no shutdown
  
  interface mgmt0 
  vrf member management                                                                                                                                                                                                                                                                                                                                                                                                           interface loopback0                                                                                                                                                                                                ip address 10.101.101.1/32                                                                                                                                                                                     icam monitor scale                                                                                                                                                                                                                                                                                                                                                                                                                line console                                                                                                                                                                                                     line vty                                                                                                                                                                                                         boot nxos bootflash:/nxos64-cs.10.4.2.F.bin                                                                                                                                                                      

router bgp 65000 
	router-id 10.101.101.1                                                                                                                                                                                           
	reconnect-interval 12                                                                                                                                                                                            
	address-family ipv4 unicast
    redistribute direct route-map REDIST_LOOPBACK
    maximum-paths 10
  neighbor 10.101.109.2
    remote-as 65001
    description Peer-to-Leaf-1
    password 3 a01efc0492d53b40c0b3aa9b18731403
    timers 3 9
    address-family ipv4 unicast
      default-originate
  neighbor 10.101.109.6
    remote-as 65002
    description Pear-to-Leaf-2
    password 3 a01efc0492d53b40c0b3aa9b18731403
    timers 3 9
    address-family ipv4 unicast
      default-originate
  neighbor 10.101.109.10
    remote-as 65003
    description Pear-to-Leaf-3
    password 3 a01efc0492d53b40c0b3aa9b18731403
    timers 3 9
    address-family ipv4 unicast
      default-originate
  neighbor 10.111.109.1
    remote-as 64512
    description Pear-to-SuperSpine-1
    password 3 a01efc0492d53b40c0b3aa9b18731403
    timers 3 9
    address-family ipv4 unicast
 ```
 
 Конфигурация ***Leaf-1*** коммутатора
 
 ```
 !Command: show running-config
!Running configuration last done at: Sat Mar 29 15:02:53 2025
!Time: Sat Mar 29 15:51:29 2025

version 10.4(2) Bios:version
hostname Leaf-1                                                                                                                                                                                                  
vdc Leaf-1 id 1                                                                                                                                                                                                    
limit-resource vlan minimum 16 maximum 4094                                                                                                                                                                      
limit-resource vrf minimum 2 maximum 4097                                                                                                                                                                        
limit-resource port-channel minimum 0 maximum 511                                                                                                                                                                
limit-resource m4route-mem minimum 58 maximum 58                                                                                                                                                                 
limit-resource m6route-mem minimum 8 maximum 8                                                                                                                                                                                                                                                                                                                                                                                  feature ospf                                                                                                                                                                                                     feature bgp                                                                                                                                                                                                      feature isis                                                                                                                                                                                                     feature bfd                                                                                                                                                                                                                                                                                                                                                                                                                       no password strength-check
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

vlan 1

route-map REDIST_LOOPBACK permit 10
  match interface loopback0
vrf context management


interface Ethernet1/1
  description LINK-TO-Spine-1
  no ip redirects
  ip address 10.101.109.2/30
  no shutdown

interface Ethernet1/2
  description LINK-TO-Spine-2
  ip address 10.102.109.2/30
  no shutdown
  
  interface mgmt0
  vrf member management

interface loopback0
  ip address 10.201.101.1/32
icam monitor scale

line console
line vty
boot nxos bootflash:/nxos64-cs.10.4.2.F.bin

router bgp 65001
  router-id 10.201.101.1
  reconnect-interval 12
  address-family ipv4 unicast
    redistribute direct route-map REDIST_LOOPBACK
    maximum-paths 10
  neighbor 10.101.109.1
    remote-as 65000
    description Peer-to-Spine-1
    password 3 a01efc0492d53b40c0b3aa9b18731403
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.102.109.1
    remote-as 65000
    description Pear-to-Spine-2
    password 3 a01efc0492d53b40c0b3aa9b18731403
    timers 3 9
    address-family ipv4 unicast

