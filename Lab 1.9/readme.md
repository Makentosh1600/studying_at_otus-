# Лабораторная работа - Конфигурация безопасности коммутатора

---

## Топология

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/01.png)   

---

## Таблица адресации

| Устройство | Интерфейс | IP-адрес | Маска подсети |
|-----------|-----------|----------|---------------|
| **R1** | G0/0/1 | 192.168.10.1 | 255.255.255.0 |
| **R1** | Loopback0 | 10.10.1.1 | 255.255.255.0 |
| **S1** | VLAN 10 | 192.168.10.201 | 255.255.255.0 |
| **S2** | VLAN 10 | 192.168.10.202 | 255.255.255.0 |
| **PC-A** | NIC | DHCP | 255.255.255.0 |
| **PC-B** | NIC | DHCP | 255.255.255.0 |

---

## Настройка VLAN

| VLAN ID | Имя |
|---------|------|
| 1 | Default |
| 10 | Management |
| 333 | Native |
| 999 | ParkingLot |

---

## ЧАСТЬ 1: Настройка основных параметров

### 1.1 Построение физической топологии
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/02.png) 


### 1.2 Настройка маршрутизатора R1

Выполните следующий конфигурационный скрипт на R1:

```
enable
configure terminal

hostname R1

no ip domain lookup

ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.10.201 192.168.10.202

ip dhcp pool Students
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
domain-name CCNA2.Lab-11.6.1
exit

interface Loopback0
ip address 10.10.1.1 255.255.255.0
exit

interface GigabitEthernet0/1
description Link to S1
ip dhcp relay information trusted
ip address 192.168.10.1 255.255.255.0
no shutdown
exit

line con 0
logging synchronous
exec-timeout 0 0
exit

exit
```

### 1.3 Проверка конфигурации R1

```
R1# show ip interface brief
```
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/03.png)  

### 1.3 Базовая конфигурация коммутаторов S1 и S2

Выполните на обоих коммутаторах:

```cisco
enable
configure terminal

hostname S1          ! (или S2 для второго коммутатора)

no ip domain lookup

ip default-gateway 192.168.10.1

! Для S1 настройка интерфейсов
interface F0/5
description Link to R1
exit

interface F0/6
description Link to PC-A
exit

interface F0/1
description Link to S2
exit

! Для S2 описани используемых интерфейсов
! interface F0/1
! description Link to S1
! exit

! interface F0/18
! description Link to PC-B
! exit

exit
```

---

## ЧАСТЬ 2: Настройка сетей VLAN на коммутаторах

### 2.1 Создание VLAN

На обоих коммутаторах (S1 и S2):

```cisco
configure terminal

vlan 10
name Management
exit

vlan 333
name Native
exit

vlan 999
name ParkingLot
exit

exit
```

### 2.2 Конфигурирование SVI для VLAN 10

**На S1:**

```cisco
configure terminal

interface vlan 10
description Management Interface S1
ip address 192.168.10.201 255.255.255.0
no shutdown
exit

exit
```

**На S2:**

```cisco
configure terminal

interface vlan 10
description Management Interface S2
ip address 192.168.10.202 255.255.255.0
no shutdown
exit

exit
```
---

## ЧАСТЬ 3: Реализация комплексных функций безопасности

### 3.1 Конфигурация магистральных соединений 

На обоих коммутаторах настройте интерфейс F0/1:

```cisco
configure terminal

interface F0/1
description Trunk Link to Other Switch
switchport mode trunk
switchport trunk native vlan 333
switchport trunk allowed vlan 10,333,999
switchport nonegotiate
no shutdown
exit

exit
```

### Проверка конфигурации trunk

На обоих коммутаторах:

```cisco
show interface trunk
```
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/04.png)  
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/05.png)       

### 3.3 Проверка отключения DTP

На обоих коммутаторах:

```cisco
show interfaces F0/1 switchport | include Negotiation
```

**Ожидаемый результат:**
```
Negotiation of Trunking: Off
```

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/06.png)     
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/07.png)     


### 3.2 Настройка портов доступа 

**На S1:**

```cisco
configure terminal

interface F0/5
switchport mode access
switchport access vlan 10
description Link to R1
no shutdown
exit

interface F0/6
switchport mode access
switchport access vlan 10
description Link to PC-A
no shutdown
exit

exit
```

**На S2:**

```cisco
configure terminal

interface F0/18
switchport mode access
switchport access vlan 10
description Link to PC-B
no shutdown
exit

exit
```

### 3.3 Обеспечение безопасности неиспользуемых портов

**На S1:**

```cisco
configure terminal

interface range F0/2-4, F0/7-24, G0/1-2
switchport mode access
switchport access vlan 999
shutdown
exit

exit
```

**На S2:**

```cisco
configure terminal

interface range F0/2-17, F0/19-24, G0/1-2
switchport mode access
switchport access vlan 999
shutdown
exit

exit
```

### Проверка конфигурации портов

На обоих коммутаторах:

```cisco
show interfaces status
```
 
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/08.png)    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/09.png)  

---


### 3.4 Документирование и реализация функции безопасности портов

На S1:

```cisco
show port-security interface F0/6
```

| Параметр | Значение |
|----------|---------|
| Защита портов | Disabled |
| Максимальное количество MAC-адресов | 1 |
| Режим нарушения | Secure-down |
| Aging Time | 0 mins |
| Aging Type | Absolute |
| Secure Static Address Aging | Disabled |
| Sticky MAC Address | 0 |

### Конфигурация Port Security на S1 F0/6


```cisco
configure terminal

interface F0/6
switchport port-security
switchport port-security maximum 3
switchport port-security violation restrict
switchport port-security aging time 60
switchport port-security aging type inactivity (!!! НЕ ПОДЕРЖИВАЕТ CPT)
exit

exit
```

```cisco
show port-security interface F0/6
```
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/10.png)  

### Просмотр таблицы безопасных MAC-адресов

```cisco
show port-security address
```
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/11.png)  

### Конфигурация Port Security на S2 F0/18 (Sticky MAC)

```cisco
configure terminal

interface F0/18
switchport port-security
switchport port-security maximum 2
switchport port-security violation protect
switchport port-security mac-address sticky
switchport port-security aging time 60
exit

exit
```

### Проверка Port Security на S2 F0/18

```cisco
show port-security interface F0/18
```

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/12.png)  

### Просмотр таблицы безопасных MAC-адресов на S2

```cisco
show port-security address
```
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/13.png)  

---

### 3.5 Включение DHCP Snooping на S2

```cisco
configure terminal

ip dhcp snooping
ip dhcp snooping vlan 10
interface F0/1
ip dhcp snooping trust
exit

interface F0/18
ip dhcp snooping limit rate 5
exit

exit
```

### Проверка конфигурации DHCP Snooping на S2

```cisco
show ip dhcp snooping
```
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/14.png)  

### Проверка привязки DHCP Snooping на S2

После того как PC-B получит IP-адрес через DHCP:

```cisco
show ip dhcp snooping binding
```
**HELP!!!**   
коммутатор S2 не пропускает запросы DHCP. Комьютер PC-B не получает IP адрес после выполнение команды ip dhcp snooping на S2. Отключает работает. Не нашел причину.

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/15.png)  

---

### 3.6 Конфигурация PortFast и BPDU Guard


**На S1:**

```cisco
configure terminal

interface range F0/5-6
spanning-tree portfast
exit

exit
```

**На S2:**

```cisco
configure terminal

interface F0/18
spanning-tree portfast
exit

exit
```

### Включение BPDU Guard

**На S1:**

```cisco
configure terminal

interface range F0/5-6
spanning-tree bpduguard enable
exit

exit
```

**На S2:**

```cisco
configure terminal

interface F0/18
spanning-tree bpduguard enable
exit

exit
```

### Проверка PortFast и BPDU Guard

**На S1:**

```cisco
show spanning-tree interface F0/6 detail
```
   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/16.png)  

! CPT не отображает информацию о состоянии BPDU на портах

---

### Проверка сквозной связности
На PC-B отключен ip dhcp snooping.

**На PC-B:** 
```cmd
ping 192.168.10.1
```
 
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/17.png)  

```cmd
ping 192.168.10.201
```

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/18.png) 

```cmd
ping 192.168.10.10
```
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/19.png) 

```cmd
ping 10.10.1.1
```
 
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.9/JPG/20.png) 

---

## Вопросы для повторения 

### Вопрос 1: С точки зрения безопасности порта на S2, почему нет значения таймера для оставшегося возраста в минутах, когда было сконфигурировано динамическое обучение - sticky?

**Ответ:** Sticky MAC-адреса имеют тип **Static** в конфигурации и хранятся как постоянные записи в конфигурационном файле. 

### Вопрос 2: Что касается безопасности порта на S2, если вы загружаете скрипт текущей конфигурации на S2, почему порту 18 на PC-B никогда не получит IP-адрес через DHCP?

**Ответ:** При загрузке конфигурации со Sticky MAC-адресами в конфигурационный файл включается команда `switchport port-security mac-address sticky [MAC]`. Когда эта конфигурация загружается, коммутатор ограничивает порт F0/18 только указанными MAC-адресами. Если PC-B имеет другой MAC-адрес, он будет заблокирован порталом. Необходимо либо удалить статические MAC-записи, либо обновить их.


### Вопрос 3: 3.	Что касается безопасности порта, в чем разница между типом абсолютного устаревания и типом устаревание по неактивности?

**Ответ:**

- **Absolute (Абсолютное):** MAC-адрес удаляется после истечения установленного времени (60 минут), независимо от активности. Таймер не сбрасывается, если устройство активно.

- **Inactivity (По неактивности):** MAC-адрес удаляется только если в течение установленного времени (60 минут) нет трафика от этого MAC-адреса. Если устройство активно, таймер постоянно сбрасывается.

