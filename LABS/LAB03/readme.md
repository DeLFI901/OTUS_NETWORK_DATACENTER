# Настройка протокола IS-IS сети Дата-центра DeLFI-UP

## Цель:
***1.Настроить протокол IS-IS в Underlay сети датацентра, для IP связанности между всеми сетевыми устройствами;<br>***

## План работ

1. Сконфигурировать Loopback адреса на всех устройствах;
2. Сконфигурировать Ethernet интерфейсы на всех устройствах;
3. Сконфигурировать протокол динамической маршрутизации IS-IS;
4. Проверить работу Underlay сети дата-центра DeLFI-UP;

## Описание 
После успешного запуска протокола OSPF в сети дата-центра DeLFI-UP от инвесторов было получено финансирование и были приобретены коммутаторы Cisco Nexus. Побывав на конференции одного из ведущих производителей сетевого оборудования руководство ознакомилось с современными технологиями и захотело непременно внедрить такие вещи как Segment-Routing MPLS  и VxLAN в нашем дата-центре, отказавшись от использовании протокола LDP.<br> 

Просмотрев презентации и рекламные брошюры с конференции было принято решения что дальше жить без протокола IS-IS мы не можем, где тут-же и была поставлена задача развернуть базовую конфигурацию протокола IS-IS в сети дата-центра.<br> ***Доводы сетевых инженеров  что в протоколе OSPF также все будет работать хорошо были отклонены***  :(

## Схема сети
Для орагнизации Underlay-сети в нашем дата-центре нам необходимо настроить сетевые интерфейсы и Loopback адреса согласно схемы. Руководствуясь  IP планом и приведенной схеммой.  <br>


![Схема сети](https://raw.githubusercontent.com/DeLFI901/OTUS_NETWORK_DATACENTER/refs/heads/main/LABS/LAB03/DC-NETWORK-IS-IS.jpeg "Схема организации подключения Underlay сети дата-центра DeLFI-UP по протоколу IS-IS")

## Конфигурация устройств

Для правильной работы Underlay сети в первую очередь нам необходимо сконфигурировать Loopback адреса на всех устройствах. Для этого идем к телекоммуникационным стойкам и подключаемся консольным кабелем к коммутаторам.  Переходим в режим глобальной конфигурации устройства и руководствуясь нашим IP-планом производим конфигурации Loopback адресов: В этом примере мы будем использовать коммутатор ***Spine-2*** для настроек. На остальных устройствах необходимо выполнить аналогичные команды учитывая необходимую IP адресацию и адреса NET которые отображены на схеме. 


``` Spine-2(config)# interface loopback 0```<br> 
``` ip address 10.102.101.1/32```

После настройки всех Loopback адресов произведем настройку сетевых интерфейсов которые будут использоваться для подключения к Spine\Leaf коммутаторам. 

Производим настройку IP адресации на физических интерфейсах устройства:

```
Spine-2(config)# interface ethernet 1/1
Spine-2(config-if)#no switchport (отключим L2 коммутацию трафика на порту)
Spine-2(config-if)#ip address 10.102.109.1 255.255.255.252
Spine-2(config-if)#description LINK-TO-Leaf-1 (добавим описание интерфейса) 
```

После настройки вех необходимых для работы  интерфейсов проверим их видимость в системе. Для этого выполним команду:

```
Spine-2# show ip int brief
IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  10.102.101.1    protocol-up/link-up/admin-up
Eth1/1               10.102.109.1    protocol-up/link-up/admin-up
Eth1/2               10.102.109.5    protocol-up/link-up/admin-up
Eth1/3               10.102.109.9    protocol-up/link-up/admin-up
```

Переходим к конфигурации протокола IS-IS:

```router ISIS DeLFI-UP (где DeLFI-UP имя процесса IS-IS) ``` <br>

Назначим версию is-type  для коректной работы Overlay сети

``` is-type level-2 ```

Укажем NET адрес нашего устройства

```net 49.0001.0000.0000.0003.00```

Включим отображение изменений в процессе работы в log-файлы:

```  
log-adjacency-changes
```
Для возможности процессу IS-IS создавать и принимать как объекты Type Length Value (TLV) в узком метрическом стиле, так и объекты TLV в широком метрическом стиле выполним команду:

  ```metric-style transition```
  
 Сконфигурируем семейство адресов IPv4 в протоколе IS-IS:
  
  ```
  address-family ipv4 unicast
```
Назначим Router-ID для корректной работы 

```
router-id 10.102.101.1
```
Теперь нам необходимо настроить сетевые интерфейсы для работы с протоколом IS-IS. Для этого перейдем в настройки сетевых интерфейсов и выполним команду сразу на всех трех интерфейсах:
```
Spine-2(config)# interface ethernet 1/1-3
Spine-2(config-if-range)# ip router isis DeLFI-UP
```

Также нам необходимо выполнить эту команду на Loopback интерфейсе:

```
Spine-2(config)# interface loopback 0
Spine-2(config-if)# ip router isis DeLFI-UP
```

Проверим состояние базы данных протокола IS-IS:
```
Spine-2# sh isis DeLFI-UP database
IS-IS Process: DeLFI-UP LSP database VRF: default
IS-IS Level-1 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T

IS-IS Level-2 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
  Spine-1.00-00         0x000000AB   0x20E7    946        0/0/0/3
  Spine-1.01-00         0x00000009   0x9016    1042       0/0/0/3
  Spine-1.02-00         0x000000A3   0xB452    965        0/0/0/3
  Spine-1.03-00         0x0000000E   0x0990    882        0/0/0/3
  Leaf-1.00-00          0x00000009   0x1743    1051       0/0/0/3
  Leaf-1.02-00          0x00000009   0xF7A8    996        0/0/0/3
  Spine-2.00-00       * 0x0000001C   0xC4B2    1142       0/0/0/3
  Leaf-2.00-00          0x0000001A   0xD251    829        0/0/0/3
  Leaf-2.02-00          0x0000000F   0x682C    1071       0/0/0/3
  Leaf-3.00-00          0x00000018   0xE122    827        0/0/0/3
  Leaf-3.02-00          0x0000000E   0xA8E9    1017       0/0/0/3

```

Проверим состояние таблицы маршрутизации:

```
Spine-2# sh ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.101.101.1/32, ubest/mbest: 3/0
    *via 10.102.109.2, Eth1/1, [115/81], 01:19:48, isis-DeLFI-UP, L2
    *via 10.102.109.6, Eth1/2, [115/81], 01:19:57, isis-DeLFI-UP, L2
    *via 10.102.109.10, Eth1/3, [115/81], 01:19:57, isis-DeLFI-UP, L2
10.101.109.0/30, ubest/mbest: 1/0
    *via 10.102.109.2, Eth1/1, [115/80], 01:19:57, isis-DeLFI-UP, L2
10.101.109.4/30, ubest/mbest: 1/0
    *via 10.102.109.6, Eth1/2, [115/80], 02:14:20, isis-DeLFI-UP, L2
10.101.109.8/30, ubest/mbest: 1/0
    *via 10.102.109.10, Eth1/3, [115/80], 02:07:42, isis-DeLFI-UP, L2
10.102.101.1/32, ubest/mbest: 2/0, attached
    *via 10.102.101.1, Lo0, [0/0], 02:31:16, local
    *via 10.102.101.1, Lo0, [0/0], 02:31:16, direct
10.102.109.0/30, ubest/mbest: 1/0, attached
    *via 10.102.109.1, Eth1/1, [0/0], 02:30:00, direct
10.102.109.1/32, ubest/mbest: 1/0, attached
    *via 10.102.109.1, Eth1/1, [0/0], 02:30:00, local
10.102.109.4/30, ubest/mbest: 1/0, attached
    *via 10.102.109.5, Eth1/2, [0/0], 02:30:00, direct
10.102.109.5/32, ubest/mbest: 1/0, attached
    *via 10.102.109.5, Eth1/2, [0/0], 02:30:00, local
10.102.109.8/30, ubest/mbest: 1/0, attached
    *via 10.102.109.9, Eth1/3, [0/0], 02:30:00, direct
10.102.109.9/32, ubest/mbest: 1/0, attached
    *via 10.102.109.9, Eth1/3, [0/0], 02:30:00, local
10.201.101.1/32, ubest/mbest: 1/0
    *via 10.102.109.2, Eth1/1, [115/41], 01:19:57, isis-DeLFI-UP, L2
10.202.101.1/32, ubest/mbest: 1/0
    *via 10.102.109.6, Eth1/2, [115/41], 02:14:20, isis-DeLFI-UP, L2
10.203.101.1/32, ubest/mbest: 1/0
    *via 10.102.109.10, Eth1/3, [115/41], 02:07:42, isis-DeLFI-UP, L2

```
Посмотрим на маршруты полученные от процесса IS-IS:
```
Spine-2# sh ip route isis-DeLFI-UP
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.101.101.1/32, ubest/mbest: 3/0
    *via 10.102.109.2, Eth1/1, [115/81], 01:20:42, isis-DeLFI-UP, L2
    *via 10.102.109.6, Eth1/2, [115/81], 01:20:51, isis-DeLFI-UP, L2
    *via 10.102.109.10, Eth1/3, [115/81], 01:20:51, isis-DeLFI-UP, L2
10.101.109.0/30, ubest/mbest: 1/0
    *via 10.102.109.2, Eth1/1, [115/80], 01:20:51, isis-DeLFI-UP, L2
10.101.109.4/30, ubest/mbest: 1/0
    *via 10.102.109.6, Eth1/2, [115/80], 02:15:14, isis-DeLFI-UP, L2
10.101.109.8/30, ubest/mbest: 1/0
    *via 10.102.109.10, Eth1/3, [115/80], 02:08:36, isis-DeLFI-UP, L2
10.201.101.1/32, ubest/mbest: 1/0
    *via 10.102.109.2, Eth1/1, [115/41], 01:20:51, isis-DeLFI-UP, L2
10.202.101.1/32, ubest/mbest: 1/0
    *via 10.102.109.6, Eth1/2, [115/41], 02:15:14, isis-DeLFI-UP, L2
10.203.101.1/32, ubest/mbest: 1/0
    *via 10.102.109.10, Eth1/3, [115/41], 02:08:36, isis-DeLFI-UP, L2
```
 
 
 Проверим достпность Spine-1 и Leaf коммутаторов: <br>
 ***Spine-1*** 
 ```
Spine-2# ping 10.101.101.1
PING 10.101.101.1 (10.101.101.1): 56 data bytes
64 bytes from 10.101.101.1: icmp_seq=0 ttl=253 time=4.465 ms
64 bytes from 10.101.101.1: icmp_seq=1 ttl=253 time=2.727 ms
64 bytes from 10.101.101.1: icmp_seq=2 ttl=253 time=3.546 ms
64 bytes from 10.101.101.1: icmp_seq=3 ttl=253 time=1.568 ms
64 bytes from 10.101.101.1: icmp_seq=4 ttl=253 time=1.505 ms

--- 10.101.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 1.505/2.762/4.465 ms
```
***Leaf-1*** <br>

```
Spine-2# ping 10.201.101.1
PING 10.201.101.1 (10.201.101.1): 56 data bytes
64 bytes from 10.201.101.1: icmp_seq=0 ttl=254 time=1.238 ms
64 bytes from 10.201.101.1: icmp_seq=1 ttl=254 time=0.892 ms
64 bytes from 10.201.101.1: icmp_seq=2 ttl=254 time=0.722 ms
64 bytes from 10.201.101.1: icmp_seq=3 ttl=254 time=0.608 ms
64 bytes from 10.201.101.1: icmp_seq=4 ttl=254 time=0.771 ms

--- 10.201.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.608/0.846/1.238 ms
```

***Leaf-2*** <br>
```
Spine-2# ping 10.202.101.1
PING 10.202.101.1 (10.202.101.1): 56 data bytes
64 bytes from 10.202.101.1: icmp_seq=0 ttl=254 time=1.463 ms
64 bytes from 10.202.101.1: icmp_seq=1 ttl=254 time=0.673 ms
64 bytes from 10.202.101.1: icmp_seq=2 ttl=254 time=1.12 ms
64 bytes from 10.202.101.1: icmp_seq=3 ttl=254 time=0.693 ms
64 bytes from 10.202.101.1: icmp_seq=4 ttl=254 time=1.232 ms

--- 10.202.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.673/1.036/1.463 ms
```

***Leaf-3*** <br>

```
Spine-2# ping 10.203.101.1
PING 10.203.101.1 (10.203.101.1): 56 data bytes
64 bytes from 10.203.101.1: icmp_seq=0 ttl=254 time=1.249 ms
64 bytes from 10.203.101.1: icmp_seq=1 ttl=254 time=0.935 ms
64 bytes from 10.203.101.1: icmp_seq=2 ttl=254 time=0.76 ms
64 bytes from 10.203.101.1: icmp_seq=3 ttl=254 time=0.482 ms
64 bytes from 10.203.101.1: icmp_seq=4 ttl=254 time=0.627 ms

--- 10.203.101.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.482/0.81/1.249 ms
```

Исходя из вывода команд мы можем сделать вывод что маршрутизация работает корекктно и наш протокол IS-IS базово настроен. По мере роста сети и получения опыта сетевыми инженерами дата-центра будут произведены тонкие настройки данного протокола. Но уже сейчас можно сказать что базовые функции Underlay сети по передаче IP трафика данная сеть может выполнять успешно.




***Конфигурация Spine-2 коммутатора:***<br>

```
Spine-2# sh run

!Command: show running-config
!Running configuration last done at: Mon Mar 24 11:26:47 2025
!Time: Mon Mar 24 11:40:58 2025

version 10.4(2) Bios:version
hostname Spine-2
vdc Spine-2 id 1
  limit-resource vlan minimum 16 maximum 4094
  limit-resource vrf minimum 2 maximum 4097
  limit-resource port-channel minimum 0 maximum 511
  limit-resource m4route-mem minimum 58 maximum 58
  limit-resource m6route-mem minimum 8 maximum 8

feature ospf
feature bgp
feature isis
feature bfd

no password strength-check
username admin password 5 $5$NMEMHK$zK9Ny/8spbWB8maaYZ3wb6tSVKAeSzw8371n6FfE431
 role network-admin
ip domain-lookup
copp profile strict
snmp-server user admin network-admin auth md5 2144191D7C160027F0FA0EB78B4260A269
5C priv aes-128 043E353D4F30482BE0F62EAD9C132AF3704D localizedV2key
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO

vlan 1

vrf context management

interface Ethernet1/1
  description LINK-TO-Leaf-1
  ip address 10.102.109.1/30
  ip router isis DeLFI-UP
  no shutdown
  interface Ethernet1/2
  description LINK-TO-Leaf-2
  ip address 10.102.109.5/30
  ip router isis DeLFI-UP
  no shutdown

interface Ethernet1/3
  description LINK-TO-Leaf-3
  ip address 10.102.109.9/30
  ip router isis DeLFI-UP
  no shutdown

interface mgmt0
  vrf member management

interface loopback0
  ip address 10.102.101.1/32
  ip router isis DeLFI-UP
icam monitor scale

line console
line console
line vty
boot nxos bootflash:/nxos64-cs.10.4.2.F.bin
router isis DeLFI-UP
  net 49.0001.0000.0000.0003.00
  is-type level-2
  metric-style transition
  log-adjacency-changes
  address-family ipv4 unicast
    router-id 10.102.101.1
```

***Конфигурация Leaf-2 коммутатора:***<br>
```
!Command: show running-config
!Running configuration last done at: Mon Mar 24 11:46:50 2025
!Time: Mon Mar 24 11:47:32 2025

version 10.4(2) Bios:version
hostname Leaf-2
vdc Leaf-2 id 1
  limit-resource vlan minimum 16 maximum 4094
  limit-resource vrf minimum 2 maximum 4097
  limit-resource port-channel minimum 0 maximum 511
  limit-resource m4route-mem minimum 58 maximum 58
  limit-resource m6route-mem minimum 8 maximum 8

feature ospf
feature bgp
feature isis
feature bfd

no password strength-check
username admin password 5 $5$MLFKHL$JmDmNm3PyV1lynA/tihorCHLiILMKxqkzPIK0PFVnJ7
 role network-admin
ip domain-lookup
copp profile strict
snmp-server user admin network-admin auth md5 0044F98ED6D5AF6C9ED453004F02ABBB03
59 priv aes-128 165AB9D3F3A68A52BCD23F0E1051DEB45D09 localizedV2key
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO

vlan 1

vrf context management

interface Ethernet1/1
  description LINK-TO-Spine-1
  ip address 10.101.109.6/30
  ip router isis DeLFI-UP
  no shutdown

interface Ethernet1/2
  description LINK-TO-Spine-2
  ip address 10.102.109.6/30
  ip router isis DeLFI-UP
  no shutdown
  
  interface mgmt0
  vrf member management

interface loopback0
  ip address 10.202.101.1/32
  ip router isis DeLFI-UP
icam monitor scale

line console
line vty
boot nxos bootflash:/nxos64-cs.10.4.2.F.bin
router isis DeLFI-UP
  net 49.0001.0000.0000.0004.00
  is-type level-2
  metric-style transition
  log-adjacency-changes
  address-family ipv4 unicast
    router-id 10.202.101.1
```
