# Проектная работа

## Тема: Проектирование VxLAN\EVPN сети для повышения отказоустойчивости программного беспроводного контроллера Ubiquiti UniFI 
## Цель и задачи проекта
#### <u>**Обеспечить возможность миграции виртуальной машины беспроводного контроллера между гипервизорами VMware ESXi  находящихся в разных местах**</u><br>

![Схема сети](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/Proejct.jpeg?raw=true "Схема организации подключения")

### Задачи проекта 
1.Рассмотреть текущую схему работы беспроводной сети;<br>
2.Рассмотреть схему работы сети после внедрения VxLAN\EVPN;<br>
3.Настроить Overlay сеть;<br>
4.Произвести тест доступности удаленного гипервизора 


### Какие технологии использовались?
1. Cisco Nexus9000v 
2. OSPF\BGP (Underlay)
3. VxLAN\EVPN (Overlay)
4. VMware ESXi 6.5

### Ссылки:
1. [Презентация](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/%D0%9F%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%20VxLAN-EVPN%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%B4%D0%BB%D1%8F%20%D0%BF%D0%BE%D0%B2%D1%8B%D1%88%D0%B5%D0%BD%D0%B8%D1%8F%20%D0%BE%D1%82%D0%BA%D0%B0%D0%B7%D0%BE%D1%83%D1%81%D1%82%D0%BE%D0%B9%D1%87%D0%B8%D0%B2%D0%BE%D1%81%D1%82%D0%B8%20%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%B1%D0%B5%D1%81%D0%BF%D1%80%D0%BE%D0%B2%D0%BE%D0%B4%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BA%D0%BE%D0%BD%D1%82%D1%80%D0%BE%D0%BB%D0%BB%D0%B5%D1%80%D0%B0%20Ubiqutu%20UniFI.pptx "Проектирование VxLAN\EVPN сети для повышения отказоустойчивости программного беспроводного контроллера Ubiqutu UniFi
") проекта;
2. [Файлы](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/_Exports_unetlab_export-20250513-201541.zip "EVE-NG проект") проекта;
3. [Файл конфигурации Spine-коммутатора](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/Spine-1.txt)
4. [Файл конфигурации Leaf-1-коммутатора](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/Leaf-1.txt)
5. [Файл конфигурации Leaf-3-коммутатора](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/Leaf-3.txt)
6. [Схема существующей сети](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/NAT-UniFi.jpeg)
7. [Схема сети с использованием VxLAN\EVPN](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/VxLAN-L2-VNI-UniFI.jpeg)
8. [Общая схема решения](https://github.com/DeLFI901/OTUS_NETWORK_DATACENTER/blob/main/LABS/PROJECT/Proejct.jpeg)