# Лабораторная работа: Настройка ACL в сети Cisco

## Топология сети


![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.11/JPG/01.jpg)

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
 
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.11/JPG/02.jpg)
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
### 2.1 Создание сетей VLAN на коммутаторах (S1 и S2)
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
### 2.2 Настройка интерфейсов управления и шлюзов по умолчанию на маршрутизаторов (S1 и S2)   
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
### 2.3 Назначение не используемых портов на коммутаторах (S1 и S2)
Для S1:
```
interface range f0/2-4, f0/7-24, g0/1-2
 switchport mode access
 switchport access vlan 999
 shutdown
exit
Interface f0/6
switchport mode access
switchport access vlan 30
switchport nonegotiate

```

Для S2:
```
interface range f0/2-4, f0/6-17, f0/19-24, g0/1-2
 switchport mode access
 switchport access vlan 999
 shutdown
exit
Interface f0/18
switchport mode access
switchport access vlan 40
switchport nonegotiate

```
### 2.4 Проверка настройки VLAN и интерфейсов на коммутаторах (S1 и S2)   
Для S1:  

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.11/JPG/03.jpg)   

Для S2:   

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.11/JPG/04.jpg)

### 2.5 Настройка магистральных интерфейсов F0/1 на коммутаторах (S1 и S2)    
```
interface f0/1
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 20,30,40,1000
```

### 2.6 Настройка магистральных интерфейсов F0/5 на коммутаторах (S1 и S2) 
```
interface f0/5
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 20,30,40,1000
```
Проверка настройки командой show interfaces trunk
Для S1:
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.11/JPG/08.jpg)
Для S2:
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.11/JPG/09.jpg)   

## ЧАСТЬ 3. Настройка маршрутизаторов 
### 3.1 Настройка маршрутизатора R1
```
interface g0/1
 no shutdown
exit
interface g0/1.20
 encapsulation dot1q 20
 ip address 10.20.0.1 255.255.255.0
 description Management VLAN
interface g0/1.30
 encapsulation dot1q 30
 ip address 10.30.0.1 255.255.255.0
 description Operations VLAN
interface g0/1.40
 encapsulation dot1q 40
 ip address 10.40.0.1 255.255.255.0
 description Sales VLAN
interface g0/1.1000
 encapsulation dot1q 1000 native
 description Native VLAN
```
Настройка Loopback  
```
interface loopback1
 ip address 172.16.1.1 255.255.255.0
```
Проверка настройки командой show ip interfaces brief
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.11/JPG/07.jpg)   

### 3.2 Настройка маршрутизатора R2
```
interface g0/1
 ip address 10.20.0.4 255.255.255.0
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 10.20.0.1
```
## ЧАСТЬ 4. Настройка удаленого доступа
### 4.1 Настройка SSH на всех сетевых устройствах 
```
ip ssh version 2
username SSHadmin secret $cisco123!
ip domain-name ccna-lab.com
crypto key generate rsa general-keys modulus 1024
line vty 0 4
 transport input ssh
 login local
```
### 4.2 Включение HTTPS на R1
```
ip http secure-server
ip http authentication local
```
! Комманды не подерживаются CPT (((

