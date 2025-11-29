# Лабораторная работа: Настройка протокола OSPFv2 для одной области

## Сетевая топология   

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/LAB%210.1/JPG/01.jpg)    

## Таблица адресации

| Устройство | Интерфейс | IP-адрес | Маска подсети | 
|-----------|-----------|----------|---------------|
| **R1** | G0/0/1 | 10.53.0.1 | 255.255.255.0 |
| **R1** | Loopback1 | 172.16.1.1 | 255.255.255.0 |
| **R2** | G0/0/1 | 10.53.0.2 | 255.255.255.0 |
| **R2** | Loopback1 | 192.168.1.1 | 255.255.255.0 |

## ЧАСТЬ 1. Создание сети и настройка основных параметров устройства
### 1.1 Построение физической топологии
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/LAB%210.1/JPG/02.jpg)    

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

## ЧАСТЬ 2: Настройка и проверка базовой работы протокола OSPFv2 для одной области

### 2.1 Настройка IP-адресов интерфейсов (R1)

```cisco
configure terminal

interface G0/1
ip address 10.53.0.1 255.255.255.0
no shutdown
exit

interface Loopback1
ip address 172.16.1.1 255.255.255.0
no shutdown
exit

exit
```

### 2.2 Настройка IP-адресов интерфейсов (R2)

```cisco
configure terminal

interface G0/1
ip address 10.53.0.2 255.255.255.0
no shutdown
exit

interface Loopback1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit

exit
```

### 2.3 Базовая конфигурация OSPFv2 на R1

```cisco
configure terminal

router ospf 56
router-id 1.1.1.1
network 10.53.0.0 0.0.0.255 area 0
exit

exit
```

### 2.4 Базовая конфигурация OSPFv2 на R2

```cisco
configure terminal

router ospf 56
router-id 2.2.2.2
network 10.53.0.0 0.0.0.255 area 0
network 192.168.1.1 0.0.0.0 area 0
exit

exit
```

