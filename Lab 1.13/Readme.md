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

