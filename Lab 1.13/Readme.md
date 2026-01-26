# Лабораторная работа: Настройка протоколов CDP, LLDP и NTP

## Топология
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/01.jpg)   

## Таблица адресации

| Устройство | Интерфейс | IP-адрес | Маска подсети | Шлюз по умолчанию |
|------------|-----------|----------|---------------|-------------------|
| R1 | Loopback1 | 172.16.1.1 | 255.255.255.0 | --- |
| R1 | G0/0/1 | 10.22.0.1 | 255.255.255.0 | --- |
| S1 | SVI VLAN 1 | 10.22.0.2 | 255.255.255.0 | 10.22.0.1 |
| S2 | SVI VLAN 1 | 10.22.0.3 | 255.255.255.0 | 10.22.0.1 |

## Часть 1. Создание сети и настройка основных параметров устройства

### Шаг 1: Создание сети согласно топологии
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/02.jpg)       

### Шаг 2: Настройка базовых параметров маршрутизатора R1

```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# no ip domain-lookup
R1(config)# enable secret class
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit
R1(config)# service password-encryption
R1(config)# banner motd #Unauthorized access is prohibited!#
R1(config)# interface Loopback1
R1(config-if)# ip address 172.16.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/0/1
R1(config-if)# ip address 10.22.0.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# exit
R1# copy running-config startup-config
```
### Шаг 3: Настройка базовых параметров коммутатора S1

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname S1
S1(config)# no ip domain-lookup
S1(config)# enable secret class
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
S1(config)# line vty 0 15
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
S1(config)# service password-encryption
S1(config)# banner motd #Unauthorized access is prohibited!#
S1(config)# interface range FastEthernet0/2-4, FastEthernet0/6-24, GigabitEthernet0/1-2
S1(config-if-range)# shutdown
S1(config-if-range)# exit
S1(config)# exit
S1# copy running-config startup-config
```
### Шаг 4: Настройка базовых параметров коммутатора S2

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname S2
S2(config)# no ip domain-lookup
S2(config)# enable secret class
S2(config)# line console 0
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# exit
S2(config)# line vty 0 15
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# exit
S2(config)# service password-encryption
S2(config)# banner motd #Unauthorized access is prohibited!#
S2(config)# interface range FastEthernet0/2-24, GigabitEthernet0/1-2
S2(config-if-range)# shutdown
S2(config-if-range)# exit
S2(config)# exit
S2# copy running-config startup-config
```

---

## Часть 2. Обнаружение сетевых ресурсов с помощью протокола CDP

### Шаг 1: Определение интерфейсов CDP на R1

**Команда:**
```cisco
R1# show cdp interface
```
**ОТВЕТ R1:**    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/03.jpg)    

**Команда:**
```cisco
R1# show cdp interface neighbors
```
**ОТВЕТ R1:**    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/04.jpg)    

**Команда:**
```cisco
R1# show cdp entry S1
```
**ОТВЕТ R1:**    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/05.jpg)   

**Версия IOS на S1 - Version 15.0(2)SE4**    

**Команда:**
```cisco
R1# show cdp traffic
```
**Команда CPT не подерживается (((**    

По команде **show cdp entry S1** дополнительнь становится известно о типе и модели подключенного устройства, порт подключения к оборудованию с которого производится запрос, IP адрес.

**Команда для отключения CDP глобально:**
```cisco
R1(config)# no cdp run
```
## Часть 3. Обнаружение сетевых ресурсов с помощью протокола LLDP

### Шаг 1: Включение LLDP на всех устройствах
**На R1:**
```cisco
R1# configure terminal
R1(config)# lldp run
R1(config)# exit
```

**На S1:**
```cisco
S1# configure terminal
S1(config)# lldp run
S1(config)# exit
```

**На S2:**
```cisco
S2# configure terminal
S2(config)# lldp run
S2(config)# exit
```
### Шаг 2: Получение детальной информации о S2 с коммутатора S1

**Команда:**
```cisco
S1# show lldp entry S2
```
**Вопрос: Что такое chassis ID для коммутатора S2**   
*Ответ: Это уникальный идентификатьр, который используется для однозначного определения физического устройства в сети.*

### Шаг 3: Просмотр физической топологии с помощью LLDP

**На R1:**
```cisco
R1# show lldp neighbors
```
**ОТВЕТ R1:**   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/06.jpg)     

**На S1:**
```cisco
S1# show lldp neighbors
```
**ОТВЕТ S1:**   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/07.jpg)      

**На S2:**
```cisco
S2# show lldp neighbors
```
**ОТВЕТ S2:**   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/08.jpg)      

**Топология, построенная на основе LLDP:**
```
R1 (Gi0/0/1) <---> (Fa0/5) S1 (Fa0/1) <---> (Fa0/1) S2
```

---
## Часть 4. Настройка и проверка NTP

### Шаг 1: Вывод текущего времени на R1

**Команда:**
```cisco
R1# show clock
```
**ОТВЕТ:**
```
*2:53:39.19 UTC Mon Mar 1 1993
```
**Таблица текущего времени:**

| Дата | Время | Часовой пояс | Источник времени |
|------|-------|--------------|------------------|
| Mon Mar 1 1993 | 02:53:39.19 | UTC | Внутренние часы (не синхронизированы) |   

### Шаг 2: Установка времени на R1

**Команда:**
```cisco
R1# clock set 21:50:00 26 January 2026
```
**Проверка:**
```cisco
R1# show clock
```
**Вывод:**
```
21:50:41.159 UTC Mon Jan 26 2026
```
### Шаг 3: Настройка R1 как NTP-сервера

**Команда:**
```cisco
R1# configure terminal
R1(config)# ntp master 4
R1(config)# exit
```

**Проверка:**
```cisco
R1# show ntp status
```

**ОТВЕТ R1:**   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.13/JPG/09.jpg)  










