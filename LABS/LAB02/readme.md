# Настройка протокола OSPF сети Дата-центра DeLFI-UP

## Цель:
***1.Настроить протокол OSPF в Underlay сети датацентра, для IP связанности между всеми сетевыми устройствами;<br>***

## План работ

1. Сконфигурировать Loopback адреса на всех устройствах;
2. Сконфигурировать Ethernet интерфейсы на всех устройствах;
3. Сконфигурировать протокол динамической маршрутизации OSPF;
4. Проверить работу Underlay сети дата-центра DeLFI-UP;


## Схема сети
Для орагнизации Underlay-сети в нашем датацентре нам необходимо настроить сетевые интерфейсы и Loopback адреса согласно схемы. Руководствуясь  IP планом и приведенной схеммой. 


![Схема сети](https://raw.githubusercontent.com/DeLFI901/OTUS_NETWORK_DATACENTER/refs/heads/main/LABS/LAB02/DC-NETWORK-OSPF.jpeg "Схема организации подключения Underlay сети дата-центра DeLFI-UP")

## Конфигурация устройств

Для корректной работы Underlay сети в первую очередь нам необходимо сконфигурировать Loopback адреса на всех устройствах. Для это подключаемся консольным кабелем к коммутаторам  переходим в режим глобальной конфигурации устройства и руководствуясь нашим IP-планом производим конфигурации Loopback адресов: В этом примере мы будем использовать коммутатор ***Spine-1*** для настроек. На остальных устройствах необходимо выполнить аналогичные команды учитывая необходимую IP адресацию. 


``` Spine-1(config)#interface loopback 0```<br> 
``` ip address 10.101.101.1 255.255.255.255.255```

После настройки всех Loopback адресов произведем настройку сетевых интерфейсов которые будут использоваться для подключения к Spine\Leaf коммутаторам. 

Также для коректной работы маршрутизации на устройствах нам потребуется явно включить IP маршрутизацию на устройве командой:

```Spine-1(config)#ip routing```

Производим настройку IP адресации на физических интерфейсах устройства:

```
Spine-1(config)#interface ethernet 1
Spine-1(config-if-Et1)#no switchport (отключим L2 коммутацию трафика на порту)
Spine-1(config-if-Et1)#ip address 10.101.109.1 255.255.255.252
Spine-1(config-if-Et1#description LINK-TO-Leaf-1 (добавим описание интерфейса) 
```

После настройки вех необходимых для работы  интерфейсов проверим их видимость в системе. Для этого выполним команду:

```
Spine-1#show ip interface brief
Interface              IP Address         Status     Protocol         MTU
Ethernet1              10.101.109.1/30    up         up              1500
Ethernet2              10.101.109.5/30    up         up              1500
Ethernet3              10.101.109.9/30    up         up              1500
Loopback0              10.101.101.1/32    up         up             65535
Management1            unassigned         up         up              1500
```

Переходим к конфигурации протокола OSPF:

```router ospf 100 ``` <br>

Назначим Router-ID для коректной работы Overlay сети

``` router-id 10.101.101.1 ```

Отключим рассылку ***Hello*** сообщений протокола OSPF на всех интерфейсах по-умолчанию командой

```passive-interface default```

И включим на необходимых нам интерфейсах командой:

```  
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
```
Для корректной работы Overlay сети нам потребуется информация о Loopback адресах других устройств. Для этого потребуется включить редистрибьюцию IP подсетей в протоколе OSPF подключеных к устройству. Для этого выполним команду:

  ```redistribute connected```
  
  Нам необходимо проанасировать подсети другим соседним маршрутизаторам с котрых будет просходить установление соседтва. Для этого выполним команды:
  
  ```
   network 10.101.109.0/30 area 0.0.0.0
   network 10.101.109.4/30 area 0.0.0.0
   network 10.101.109.8/30 area 0.0.0.0
```

После выполнения всех действий проверим установления соседства с Leaf-коммутаторами. Для этого выполним следующие команды:<br>
```show ip ospf neighbor```

В выводе должна отобразиться информация о состоянии установившехся соседств с другими маршрутизаторами: 
```
Spine-1#sh ip ospf neighbor
Neighbor ID     VRF      Pri State                  Dead Time   Address         Interface
10.201.101.1    default  1   FULL/BDR               00:00:34    10.101.109.2    Ethernet1
10.202.101.1    default  1   FULL/BDR               00:00:36    10.101.109.6    Ethernet2
10.203.101.1    default  1   FULL/BDR               00:00:29    10.101.109.10   Ethernet3
```

Проверим состояние базы данных протокола OSPF:
```
Spine-1#sh ip ospf database

            OSPF Router with ID(10.101.101.1) (Process ID 100) (VRF default)


                 Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.102.101.1    10.102.101.1    1721        0x8000000b   0x5498   3
10.202.101.1    10.202.101.1    7           0x80000009   0x8482   2
10.203.101.1    10.203.101.1    1658        0x80000008   0x9362   2
10.201.101.1    10.201.101.1    472         0x80000009   0x77a1   2
10.101.101.1    10.101.101.1    1711        0x8000000b   0xe311   3

                 Network Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum
10.102.109.1    10.102.101.1    462         0x80000004   0x4d26
10.102.109.5    10.102.101.1    1841        0x80000003   0x333c
10.102.109.9    10.102.101.1    1721        0x80000003   0x1753
10.101.109.5    10.101.101.1    1831        0x80000003   0x3f33
10.101.109.9    10.101.101.1    1711        0x80000003   0x234a
10.101.109.1    10.101.101.1    511         0x80000004   0x591d

                 Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#         Checksum Tag
10.102.101.1    10.102.101.1    222         0x80000004   0xf60a   0
10.101.101.1    10.101.101.1    211         0x80000004   0xbf7    0

```

Проверим состояние таблицы маршрутизации:

```
Spine-1#sh ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - Dhcp client installed default route

Gateway of last resort is not set

 C      10.101.101.1/32 is directly connected, Loopback0
 C      10.101.109.0/30 is directly connected, Ethernet1
 C      10.101.109.4/30 is directly connected, Ethernet2
 C      10.101.109.8/30 is directly connected, Ethernet3
 O E2   10.102.101.1/32 [110/1] via 10.101.109.2, Ethernet1
                                via 10.101.109.6, Ethernet2
                                via 10.101.109.10, Ethernet3
 O      10.102.109.0/30 [110/20] via 10.101.109.2, Ethernet1
 O      10.102.109.4/30 [110/20] via 10.101.109.6, Ethernet2
 O      10.102.109.8/30 [110/20] via 10.101.109.10, Ethernet3
 O E2   10.201.101.1/32 [110/1] via 10.101.109.2, Ethernet1
 O E2   10.202.101.1/32 [110/1] via 10.101.109.6, Ethernet2
 O E2   10.203.101.1/32 [110/1] via 10.101.109.10, Ethernet3
 ```
 
 Проверим достпность Spine-2 и Leaf коммутаторов: <br>
 ***Spine-2*** 
 ```
 Spine-1#ping 10.102.101.1
PING 10.102.101.1 (10.102.101.1) 72(100) bytes of data.
80 bytes from 10.102.101.1: icmp_seq=1 ttl=63 time=10.3 ms
80 bytes from 10.102.101.1: icmp_seq=2 ttl=63 time=9.30 ms
80 bytes from 10.102.101.1: icmp_seq=3 ttl=63 time=9.39 ms
80 bytes from 10.102.101.1: icmp_seq=4 ttl=63 time=9.07 ms
80 bytes from 10.102.101.1: icmp_seq=5 ttl=63 time=9.25 ms

--- 10.102.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 40ms
rtt min/avg/max/mdev = 9.070/9.465/10.307/0.446 ms, ipg/ewma 10.178/9.868 ms
```
***Leaf-1*** <br>

```
Spine-1#ping 10.201.101.1
PING 10.201.101.1 (10.201.101.1) 72(100) bytes of data.
80 bytes from 10.201.101.1: icmp_seq=1 ttl=64 time=5.35 ms
80 bytes from 10.201.101.1: icmp_seq=2 ttl=64 time=4.81 ms
80 bytes from 10.201.101.1: icmp_seq=3 ttl=64 time=4.92 ms
80 bytes from 10.201.101.1: icmp_seq=4 ttl=64 time=4.78 ms
80 bytes from 10.201.101.1: icmp_seq=5 ttl=64 time=4.72 ms

--- 10.201.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 21ms
rtt min/avg/max/mdev = 4.726/4.921/5.358/0.228 ms, ipg/ewma 5.300/5.129 ms
```

***Leaf-2*** <br>
```
Spine-1#ping 10.202.01.1
PING 10.202.101.1 (10.202.101.1) 72(100) bytes of data.
80 bytes from 10.202.101.1: icmp_seq=1 ttl=64 time=5.18 ms
80 bytes from 10.202.101.1: icmp_seq=2 ttl=64 time=4.49 ms
80 bytes from 10.202.101.1: icmp_seq=3 ttl=64 time=4.61 ms
80 bytes from 10.202.101.1: icmp_seq=4 ttl=64 time=4.55 ms
80 bytes from 10.202.101.1: icmp_seq=5 ttl=64 time=4.63 ms

--- 10.202.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20ms
rtt min/avg/max/mdev = 4.498/4.698/5.189/0.257 ms, ipg/ewma 5.145/4.937 ms
```

***Leaf-3*** <br>

```
Spine-1#ping 10.203.01.1
PING 10.203.101.1 (10.203.101.1) 72(100) bytes of data.
80 bytes from 10.203.101.1: icmp_seq=1 ttl=64 time=11.1 ms
80 bytes from 10.203.101.1: icmp_seq=2 ttl=64 time=11.9 ms
80 bytes from 10.203.101.1: icmp_seq=3 ttl=64 time=11.7 ms
80 bytes from 10.203.101.1: icmp_seq=4 ttl=64 time=6.62 ms
80 bytes from 10.203.101.1: icmp_seq=5 ttl=64 time=5.34 ms

--- 10.203.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 5.347/9.372/11.978/2.808 ms, pipe 2, ipg/ewma 11.503/10.051 ms
```

Исходя из вывода команд мы можем сделать вывод что маршрутизация работает корекктно и наша Underlay сеть настроена корректно.



***Конфигурация Spine-1 коммутатора:***<br>

```
spanning-tree mode mstp
!
no aaa root
!
interface Ethernet1
   description LINK-TO-Leaf-1
   no switchport
   ip address 10.101.109.1/30
!
interface Ethernet2
   description LINK-TO-Leaf-2
   no switchport
   ip address 10.101.109.5/30
!
interface Ethernet3
   description LINK-TO-Leaf-3
   no switchport
   ip address 10.101.109.9/30
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 10.101.101.1/32
!
interface Management1
!
ip routing
!
router ospf 100
   router-id 10.101.101.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   redistribute connected
   network 10.101.109.0/30 area 0.0.0.0
   network 10.101.109.4/30 area 0.0.0.0
   network 10.101.109.8/30 area 0.0.0.0
   max-lsa 12000
!
end
```

***Конфигурация Liaf-1 коммутатора:***<br>
```

!
transceiver qsfp default-mode 4x10G
!
hostname Leaf-1
!
spanning-tree mode mstp
!
no aaa root
!
interface Ethernet1
   description LINK-TO-Spine-1
   no switchport
   ip address 10.101.109.2/30
!
interface Ethernet2
   description LINK-TO-Spine-2
   no switchport
   ip address 10.102.109.2/30
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 10.201.101.1/32
!
interface Management1
!
ip routing
!
router ospf 100
   router-id 10.201.101.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   redistribute connected
   network 10.101.109.0/30 area 0.0.0.0
   network 10.102.109.0/30 area 0.0.0.0
   max-lsa 12000
!
end
```
