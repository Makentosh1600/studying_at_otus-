# Лабораторная работа: Настройка DHCPv6

---

## Топология сети

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/01.png)


---

## Таблица адресации

| Устройство | Интерфейс | IPv6-адрес |
|-----------|-----------|-----------|
| R1 | G0/0/0 | 2001:db8:acad:2::1/64 |
| R1 | G0/0/0 | fe80::1 |
| R1 | G0/0/1 | 2001:db8:acad:1::1/64 |
| R1 | G0/0/1 | fe80::1 |
| R2 | G0/0/0 | 2001:db8:acad:2::2/64 |
| R2 | G0/0/0 | fe80::2 |
| R2 | G0/0/1 | 2001:db8:acad:3::1/64 |
| R2 | G0/0/1 | fe80::1 |
| PC-A | NIC | DHCP |
| PC-B | NIC | DHCP |

---

## Задачи лабораторной работы

- ✅ Создание сети и настройка основных параметров устройства    
- ✅ Проверка назначения адреса SLAAC от R1
- ✅ Настройка и проверка сервера DHCPv6 без гражданства на R1
- ✅ Настройка и проверка состояния DHCPv6 сервера на R1
- ✅ Настройка и проверка DHCPv6 Relay на R2

---

## Часть 1: Создание сети и настройка базовых параметров

### 1.1 Физическая топология

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.8/JPG/02.png)

### 1.2 Базовая конфигурация коммутаторов

#### Коммутатор S1 (и аналогично S2):

# Настройка поддержки IPv6
S1(config)# sdm prefer dual-ipv4-and-ipv6 default
S1(config)# end
S1# reload
     
# Отключение DNS lookup
S1(config)# no ip domain-lookup

# Пароль привилегированного режима (enable)
S1(config)# enable secret class

# Конфигурация консольного доступа
S1(config)# line con 0
S1(config-line)# logging synchronous
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit

# SSH конфигурация для VTY
S1(config)# ip ssh version 2
S1(config)# username admin privilege 15 secret AdminSSH
S1(config)# line vty 0 4
S1(config-line)# login local
S1(config-line)# transport input ssh
S1(config-line)# exit

# Шифрование открытых паролей
S1(config)# service password-encryption

# Баннер предупреждения
S1(config)# banner motd #
Enter TEXT message. End with the character '#'.
Unauthorized access is strictly prohibited.#

# Сохранение конфигурации
S1(config)# end
S1# write memory
```

### 1.3 Настройка маршрутизаторов

#### Маршрутизатор R1 – Конфигурация интерфейсов:

```bash
R1> enable
R1# configure terminal

# Активация IPv6 маршрутизации
R1(config)# ipv6 unicast-routing

# Конфигурация G0/0/0 (Link к R2)
R1(config)# interface gigabitethernet 0/0/0
R1(config-if)# ipv6 address 2001:db8:acad:2::1/64
R1(config-if)# ipv6 address fe80::1 link-local
R1(config-if)# no shutdown
R1(config-if)# exit

# Конфигурация G0/0/1 (Link к PC-A)
R1(config)# interface gigabitethernet 0/0/1
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64
R1(config-if)# ipv6 address fe80::1 link-local
R1(config-if)# no shutdown
R1(config-if)# exit

# Маршрут по умолчанию к R2
R1(config)# ipv6 route ::/0 2001:db8:acad:2::2
R1(config)# end
R1# write memory
```

#### Маршрутизатор R2 – Конфигурация интерфейсов:

```bash
R2> enable
R2# configure terminal

# Активация IPv6 маршрутизации
R2(config)# ipv6 unicast-routing

# Конфигурация G0/0/0 (Link к R1)
R2(config)# interface gigabitethernet 0/0/0
R2(config-if)# ipv6 address 2001:db8:acad:2::2/64
R2(config-if)# ipv6 address fe80::2 link-local
R2(config-if)# no shutdown
R2(config-if)# exit

# Конфигурация G0/0/1 (Link к PC-B)
R2(config)# interface gigabitethernet 0/0/1
R2(config-if)# ipv6 address 2001:db8:acad:3::1/64
R2(config-if)# ipv6 address fe80::1 link-local
R2(config-if)# no shutdown
R2(config-if)# exit

# Маршрут по умолчанию к R1
R2(config)# ipv6 route ::/0 2001:db8:acad:2::1
R2(config)# end
R2# write memory
```

### 1.4 Проверка маршрутизации

Проверьте связность между маршрутизаторами:

```bash
# На R1 проверить доступ к интерфейсу R2
R1# ping 2001:db8:acad:3::1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

---

## Часть 2: Проверка назначения адреса SLAAC от R1

Через несколько минут результаты команды ipconfig должны показать, что PC-A присвоил себе адрес из сети 2001:db8:1::/64.       

```
FastEthernet0 Connection (default port)
Connection-specific DNS Suffix: 
Link-local IPv6 Address........: FE80::204:9AFF:FE1B:78D4
IPv6 Address...................: 2001:DB8:ACAD:1:204:9AFF:FE1B:78D4
Autoconfiguration IPv4 Address.: 169.254.120.212
Subnet Mask.....................: 255.255.0.0
Default Gateway.................: FE80::1
```
Откуда взялась часть адреса с идентификатором хоста?   

### Ответ: генерация идентификатора интерфейса (IID)   
Преобразование MAC-адреса в 64-битный IID:

| Этап | Значение | Описание |
|------|---------|---------|
| MAC-адрес | `00:04:9A:1B:78:D4` | Исходный MAC |
| Шаг 1 | `0004:9A1B:78D4` | Представление в hex |
| Шаг 2 | `0004:9AFF:FE1B:78D4` | Вставка `FFFF` в середину |
| Шаг 3 | Бит 7 инвертирован | `0204:9AFF:FE1B:78D4` | Финальный IID |

**Формула:** `2001:db8:acad:1:` + `0204:9AFF:FE1B:78D4` = `2001:DB8:ACAD:1:204:9AFF:FE1B:78D4`

---

## Часть 3: Настройка и проверка сервера DHCPv6 на R1
В части 3 выполняется настройка и проверка состояния DHCP-сервера на R1. Цель состоит в том, чтобы предоставить PC-A информацию о DNS-сервере и домене.

### 3.1 Более подробно изучите конфигурацию PC-A.
Выполните команду ipconfig /all на PC-A и посмотрите на результат.

```
FastEthernet0 Connection:(default port)

Connection-specific DNS Suffix..: 
Physical Address................: 0004.9A1B.78D4
Link-local IPv6 Address.........: FE80::204:9AFF:FE1B:78D4
IPv6 Address....................: 2001:DB8:ACAD:1:204:9AFF:FE1B:78D4
Autoconfiguration IP Address....: 169.254.120.212
Subnet Mask.....................: 255.255.0.0
Default Gateway.................: FE80::1
0.0.0.0
DHCP Servers....................: 0.0.0.0
DHCPv6 IAID.....................: 
DHCPv6 Client DUID..............: 00-01-00-01-47-5B-26-96-00-04-9A-1B-78-D4
DNS Servers.....................: :: 	0.0.0.0

```
Обратите внимание, что основной DNS-суффикс отсутствует. Также обратите внимание, что предоставленные адреса DNS-сервера являются адресами «локального сайта anycast», а не одноадресные адреса, как ожидалось.


### 3.2 Настройте R1 для предоставления DHCPv6 без состояния для PC-A.

```bash
R1# configure terminal

# Создание пула DHCPv6
R1(config)# ipv6 dhcp pool R1-STATELESS
R1(config-dhcp)# dns-server 2001:db8:acad::254
R1(config-dhcp)# domain-name STATELESS.com
R1(config-dhcp)# exit

# Привязка пула к интерфейсу G0/0/1
R1(config)# interface gigabitethernet 0/0/1
R1(config-if)# ipv6 nd other-config-flag
R1(config-if)# ipv6 dhcp server R1-STATELESS
R1(config-if)# exit

R1(config)# end
R1# write memory
```


### 3.3 Проверка на PC-A

Проверьте вывод ipconfig /all и обратите внимание на изменения.

```
FastEthernet0 Connection (default port)
Connection-specific DNS Suffix..: STATELESS.com
Physical Address..................: 0004.9A1B.78D4
Link-local IPv6 Address...........: FE80::204:9AFF:FE1B:78D4
IPv6 Address......................: 2001:DB8:ACAD:1:204:9AFF:FE1B:78D4
Autoconfiguration IP Address......: 169.254.120.212
Subnet Mask........................: 255.255.0.0
Default Gateway....................: FE80::1
DHCPv6 IAID........................: 921531058
DHCPv6 Client DUID................: 00-01-00-01-47-5B-26-96-00-04-9A-1B-78-D4
DNS Servers........................: 2001:DB8:ACAD::254
```

### 3.4 Проверка связности

```bash
C:\> ping 2001:db8:acad:3::1
Pinging 2001:db8:acad:3::1 with 32 bytes of data:

Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254

Ping statistics for 2001:DB8:ACAD:3::1:
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Approximate round trip times in milli-seconds:
Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

---

## Часть 4: Настройка сервера DHCPv6 с сохранением состояния на R1

### Настройка Stateful DHCPv6 на R1

```bash
R1# configure terminal

# Создание пула DHCPv6 на R1 для сети 2001:db8:acad:3:aaa::/80. Это предоставит адреса локальной сети, подключенной к интерфейсу G0/0/1 на R2. В составе пула задайте DNS-сервер 2001:db8:acad: :254 и задайте доменное имя STATEFUL.com.
R1(config)# ipv6 dhcp pool R2-STATEFUL
R1(config-dhcp)# address prefix 2001:db8:acad:3:aaa::/80
R1(config-dhcp)# dns-server 2001:db8:acad::254
R1(config-dhcp)# domain-name STATEFUL.com
R1(config-dhcp)# exit

# Назначение только что созданного пула DHCPv6 интерфейсу g0/0/0 на R1 
R1(config)# interface gigabitethernet 0/0/0
R1(config-if)# ipv6 dhcp server R2-STATEFUL
R1(config-if)# exit

R1(config)# end
R1# write memory
```

---

## Часть 5: Настройка и проверка ретрансляции DHCPv6 на R2

### 5.1 Включите PC-B и проверьте адрес SLAAC, который он генерирует

```
FastEthernet0 Connection:(default port)

Connection-specific DNS Suffix..: 
Physical Address................: 00E0.8F16.7B53
Link-local IPv6 Address.........: FE80::2E0:8FFF:FE16:7B53
IPv6 Address....................: 2001:DB8:ACAD:3:2E0:8FFF:FE16:7B53
Autoconfiguration IP Address....: 169.254.123.83
Subnet Mask.....................: 255.255.0.0
Default Gateway.................: FE80::1
0.0.0.0
DHCP Servers....................: 0.0.0.0
DHCPv6 IAID.....................: 
DHCPv6 Client DUID..............: 00-01-00-01-16-EE-42-95-00-E0-8F-16-7B-53
DNS Servers.....................: ::
0.0.0.0

```

### 5.2 Настройте R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1

```bash
R2# configure terminal

# Конфигурация интерфейса G0/0/1 (сторона клиента)
R2(config)# interface gigabitethernet 0/0/1
R2(config-if)# ipv6 nd managed-config-flag
R2(config-if)# ipv6 dhcp relay destination 2001:db8:acad:2::1 gigabitethernet 0/0/0
!!! НЕ ПОДЕРЖИВАЕТ PACKET TRACER  ipv6 dhcp relay
R2(config-if)# exit

R2(config)# end
R2# write memory
```

### 5.3 Попытка получить адрес IPv6 из DHCPv6 на PC-B

Не удалось, т.к. PACKET TRACER НЕ ПОДЕРЖИВАЕТ  ipv6 dhcp relay
