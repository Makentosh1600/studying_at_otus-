# Лабораторная работа: Настройка ACL в сети Cisco

## Топология сети

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/LAB%201.11/JPG/01.jpg)

## Таблица адресации

| Устройство | Интерфейс    | IP-адрес   | Маска подсети  | Шлюз по умолчанию |
|------------|--------------|------------|----------------|-------------------|
| R1         | G0/0/1       | ---        | ---            | ---               |
| R1         | G0/0/1.20    | 10.20.0.1  | 255.255.255.0  | ---               |
| R1         | G0/0/1.30    | 10.30.0.1  | 255.255.255.0  | ---               |
| R1         | G0/0/1.40    | 10.40.0.1  | 255.255.255.0  | ---               |
| R1         | G0/0/1.1000  | ---        | ---            | ---               |
| R1         | Loopback1    | 172.16.1.1 | 255.255.255.0  | ---               |
| R2         | G0/0/1       | 10.20.0.4  | 255.255.255.0  | ---               |
| S1         | VLAN 20      | 10.20.0.2  | 255.255.255.0  | 10.20.0.1         |
| S2         | VLAN 20      | 10.20.0.3  | 255.255.255.0  | 10.20.0.1         |
| PC-A       | NIC          | 10.30.0.10 | 255.255.255.0  | 10.30.0.1         |
| PC-B       | NIC          | 10.40.0.10 | 255.255.255.0  | 10.40.0.1         |

## Таблица VLAN

| VLAN | Имя         | Назначенные интерфейсы                          |
|------|-------------|-----------------------------------------------|
| 20   | Management  | S2: F0/5                                      |
| 30   | Operations  | S1: F0/6                                      |
| 40   | Sales       | S2: F0/18                                     |
| 999  | ParkingLot  | S1: F0/2-4, F0/7-24, G0/1-2<br>S2: F0/2-4, F0/6-17, F0/19-24, G0/1-2 |
| 1000 | Native      | ---                                           |

## ЧАСТЬ 1. Создание сети и настройка основных параметров устройства
### 1.1 Построение физической топологии
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/LAB%201.11/JPG/01.jpg)  
### 1.2 Базовая конфигурация маршрутизаторов (R1 и R2)

```
configure terminal
hostname R1    # или R2 для второго маршрутизатора
no ip domain-lookup
enable password class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd #WARNING: Unauthorized access is prohibited!#
exit
copy running-config startup-config
```
### 1.3 Базовая конфигурация коммутаторов (S1 и S2)

```
enable
configure terminal
hostname S1    # или S2 для второго коммутатора
no ip domain-lookup
enable password class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd #WARNING: Unauthorized access is prohibited!#
exit
copy running-config startup-config
```
## ЧАСТЬ 2. Настройка сетей VLAN на коммутаторах
### 2.1 Создание сетей VLAN на коммутаторах (R1 и R2)
```
vlan 20
 name Management
vlan 30
 name Operations
vlan 40
 name Sales
vlan 999
 name ParkingLot
vlan 1000
 name Native
```
### 2.2 Настройка интерфейсов управления и шлюзов по умолчанию на коммутаторах (R1 и R2)   
Для S1:
```
interface vlan 20
 ip address 10.20.0.2 255.255.255.0
 exit
ip default-gateway 10.20.0.1
```
Для S2:
```
interface vlan 20
 ip address 10.20.0.3 255.255.255.0
 exit
ip default-gateway 10.20.0.1
```


