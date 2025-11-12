# Лабораторная работа - Реализация DHCPv4
---

## Топология

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/03.png)


## Таблица адресации
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/04.png)

---

## 🔗 Конфигурация VLAN

| VLAN | Имя | Назначенный интерфейс |
|------|------|----------------------|
| 1 | Default | S2: F0/18 |
| 100 | Clients_VLAN | S1: F0/6 |
| 200 | Management_VLAN | S1: VLAN 200 |
| 999 | Parking_Lot | S1: F0/1-4, F0/7-24, G0/1-2 |
| 1000 | Native_VLAN | --- |

### Часть 1: Создание сети и настройка основных параметров устройства

#### 1.1 Создание схемы адресации
##### Подсеть A (Клиенты R1)
- **Требуемое количество хостов:** 58
- **Минимальные биты хостов:** 6 (2^6 = 64)
- **Маска подсети:** 255.255.255.192 (/26)
- **Адрес сети:** 192.168.1.0
- **Первый полезный адрес (R1 G0/0/1.100):** 192.168.1.1
- **Последний полезный адрес:** 192.168.1.62
- **Broadcast:** 192.168.1.63

##### Подсеть B (Управление)
- **Требуемое количество хостов:** 28
- **Минимальные биты хостов:** 5 (2^5 = 32)
- **Маска подсети:** 255.255.255.224 (/27)
- **Адрес сети:** 192.168.1.64
- **Первый полезный адрес (R1 G0/0/1.200):** 192.168.1.65
- **Второй полезный адрес (S1 VLAN 200):** 192.168.1.66
- **Последний полезный адрес:** 192.168.1.94
- **Broadcast:** 192.168.1.95

##### Подсеть C (Клиенты R2)
- **Требуемое количество хостов:** 12
- **Минимальные биты хостов:** 4 (2^4 = 16)
- **Маска подсети:** 255.255.255.240 (/28)
- **Адрес сети:** 192.168.1.96
- **Первый полезный адрес (R2 G0/0/1):** 192.168.1.97
- **Второй полезный адрес (S2 VLAN 1):** 192.168.1.98
- **Последний полезный адрес:** 192.168.1.110
- **Broadcast:** 192.168.1.111

#### 1.2 Создание сети согласно топологии   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/05.png)

#### 1.3 Базовая конфигурация маршрутизаторов (R1 и R2)

```
enable
configure terminal
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
hostname R1
exit
copy running-config startup-config
```

#### Настройка времени
```
clock set 12:00:00 12 November 2025
```

#### 1.4 Настройка маршрутизации между VLAN на маршрутизаторе R1

```
interface G0/1
no shutdown
exit

interface G0/1.100
description Clients_VLAN
encapsulation dot1Q 100
ip address 192.168.1.1 255.255.255.192
exit

interface G0/1.200
description Management_VLAN
encapsulation dot1Q 200
ip address 192.168.1.65 255.255.255.224
exit

interface G0/1.1000
description Native_VLAN
encapsulation dot1Q 1000 native
exit
```

#### 1.5 Настройка интерфейсов на R1 и R2

**На R1:**
```
interface G0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
exit
```

**На R2:**
```
interface G0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
exit

interface G0/1
ip address 192.168.1.97 255.255.255.240
no shutdown
exit
```

#### Настройка статической маршрутизации

**На R1:**
```
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

**На R2:**
```
ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

#### Проверка маршрутизации
```
ping 192.168.1.97
```
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/06.png)
#### 1.6 Базовая конфигурация коммутаторов (S1 и S2)

*Аналогично маршрутизаторам:*
```
enable
configure terminal
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
hostname S1
exit
copy running-config startup-config

clock set 12:00:00 12 November 2025

```

#### 1.7 Создание VLAN на коммутаторе S1

```
vlan 100
name Clients_VLAN
exit

vlan 200
name Management_VLAN
exit

vlan 999
name Parking_Lot
exit

vlan 1000
name Native
exit
```

#### Настройка интерфейса управления на S1

```
interface vlan 200
ip address 192.168.1.66 255.255.255.224
no shutdown
exit

ip default-gateway 192.168.1.65
```

#### Настройка интерфейса управления на S2

```
interface vlan 1
ip address 192.168.1.98 255.255.255.240
no shutdown
exit

ip default-gateway 192.168.1.97
```

#### Назначение VLAN портам и настройка trunk на S1

```
interface F0/6
switchport mode access
switchport access vlan 100
no shutdown
exit

interface range F0/1-4, F0/7-24, G0/1-2
switchport mode access
switchport access vlan 999
shutdown
exit

interface F0/5
switchport mode trunk
switchport trunk native vlan 1000
switchport trunk allowed vlan 100,200,1000
no shutdown
exit
```

#### Назначение VLAN портам на S2

```
interface F0/18
switchport mode access
switchport access vlan 1
no shutdown
exit

interface range F0/1-4, F0/6-17, F0/19-24, G0/1-2
shutdown
exit
```

#### 1.14 Проверка конфигурации trunk

```
show interfaces F0/5 switchport
show interfaces trunk
```

---

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/07.png)

Почему интерфейс F0/5 указан в VLAN1? - По умолчанию всем портам присваивается VLAN1
Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP? - Первый свободный из таблицы возможных на сервере DHCP

### Часть 2: Настройка и проверка двух серверов DHCPv4 на R1

#### 2.1 Создание пула DHCP для подсети A

```
configure terminal

ip dhcp excluded-address 192.168.1.1 192.168.1.5

ip dhcp pool R1_Client_LAN
network 192.168.1.0 255.255.255.192
default-router 192.168.1.1
domain-name CCNA-lab.com
lease 2 12 30 (команда в Packet Trace не подкрживается)
exit
```

#### 2.2 Создание пула DHCP для подсети C

```
ip dhcp excluded-address 192.168.1.97 192.168.1.101

ip dhcp pool R2_Client_LAN
network 192.168.1.96 255.255.255.240
default-router 192.168.1.97
domain-name CCNA-lab.com
lease 2 12 30 (команда в Packet Trace не подкрживается)
exit

exit
copy running-config startup-config
```

#### 2.3 Проверка конфигурации DHCP

show ip dhcp pool
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/08.png)

show ip dhcp binding  
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/09.png)  

show ip dhcp server statistics (команда в Packet Trace не подкрживается)


#### 2.4 Получение IP-адреса на PC-A

**На компьютере PC-A:**

ipconfig
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/10.png)  

ping 192.168.1.1
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/11.png) 

---

### Часть 3: Настройка  и проверка DHCP Relay на R2

#### 3.1 Конфигурация DHCP Relay

```
configure terminal
interface G0/1
ip helper-address 10.0.0.1
no shutdown
exit

exit
copy running-config startup-config
```

#### 3.2 Получение IP-адреса на PC-B

**На компьютере PC-B:**

ipconfig
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/12.png)   

ping 192.168.1.1
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/13.png) 


#### 3.3 Проверка DHCP на R1

show ip dhcp binding
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/14.png) 

show ip dhcp server statistics (команда в Packet Trace не подкрживается)

---

