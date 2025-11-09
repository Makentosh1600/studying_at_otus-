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


### 2.3 Проверка SLAAC на PC-A

1. Включите PC-A
2. Убедитесь, что сетевой адаптер настроен на автоматическую конфигурацию IPv6
3. Выполните команду `ipconfig` на PC-A

**Ожидаемый результат:**

```
FastEthernet0 Connection (default port)
Connection-specific DNS Suffix: 
Link-local IPv6 Address........: FE80::204:9AFF:FE1B:78D4
IPv6 Address...................: 2001:DB8:ACAD:1:204:9AFF:FE1B:78D4
Autoconfiguration IPv4 Address.: 169.254.120.212
Subnet Mask.....................: 255.255.0.0
Default Gateway.................: FE80::1
```

---

## Часть 3: DHCPv6 без состояния (Stateless)

### 3.1 Концепция Stateless DHCPv6

В режиме **Stateless DHCPv6**:
- ✅ Адреса генерируются клиентом через SLAAC
- ✅ Параметры конфигурации (DNS, домен) предоставляются сервером
- ✅ Сервер **не отслеживает** распределение адресов
- ✅ Подходит для сетей с достаточным адресным пространством

### 3.2 Настройка Stateless DHCPv6 на R1

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

**Объяснение команд:**

| Команда | Описание |
|---------|---------|
| `ipv6 dhcp pool R1-STATELESS` | Создание пула для Stateless DHCPv6 |
| `dns-server 2001:db8:acad::254` | Адрес DNS сервера (IPv6) |
| `domain-name STATELESS.com` | Суффикс DNS домена |
| `ipv6 nd other-config-flag` | Установка флага O (Other Config) в Router Advertisement |
| `ipv6 dhcp server R1-STATELESS` | Привязка пула к интерфейсу |

### 3.3 Проверка на PC-A

1. Перезагрузите PC-A
2. Выполните `ipconfig /all`

**Ожидаемый результат:**

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

## Часть 4: DHCPv6 с состоянием (Stateful)

### 4.1 Концепция Stateful DHCPv6

В режиме **Stateful DHCPv6**:
- ✅ **Полное** назначение адресов (как в DHCPv4)
- ✅ Сервер **отслеживает** все распределенные адреса
- ✅ Клиент не использует SLAAC
- ✅ Подходит для сетей с ограниченным адресным пространством

### 4.2 Настройка Stateful DHCPv6 на R1

```bash
R1# configure terminal

# Создание пула DHCPv6 для сети PC-B
R1(config)# ipv6 dhcp pool R2-STATEFUL
R1(config-dhcp)# address prefix 2001:db8:acad:3:aaa::/80
R1(config-dhcp)# dns-server 2001:db8:acad::254
R1(config-dhcp)# domain-name STATEFUL.com
R1(config-dhcp)# exit

# Привязка пула к интерфейсу G0/0/0 (для Relay из R2)
R1(config)# interface gigabitethernet 0/0/0
R1(config-if)# ipv6 dhcp server R2-STATEFUL
R1(config-if)# exit

R1(config)# end
R1# write memory
```

**Параметры пула:**

| Параметр | Значение | Описание |
|----------|---------|---------|
| Пул имя | R2-STATEFUL | Идентификатор пула |
| Префикс адреса | 2001:db8:acad:3:aaa::/80 | Диапазон назначения адресов |
| Длина префикса | /80 | Позволяет 2^16 адресов (65 536) |
| DNS сервер | 2001:db8:acad::254 | Адрес DNS сервера |
| Домен | STATEFUL.com | DNS суффикс |

**Адресный диапазон:**
```
2001:db8:acad:3:aaa:0:0:1     (первый адрес)
...
2001:db8:acad:3:aaa:ffff:ffff (последний адрес)
```

---

## Часть 5: DHCPv6 Relay

### 5.1 Концепция DHCPv6 Relay

**DHCPv6 Relay** решает проблему доставки DHCPv6 запросов на разные подсети:

```
┌─────────────┐
│   PC-B      │ ── DHCPv6 SOLICIT ──┐
│ (клиент)    │                     │
└─────────────┘                     │
                                    ↓
                              ┌──────────────┐
                              │     R2       │
                              │   (Relay)    │ ── DHCPv6 RELAY-FORWARD ──┐
                              │ г0/0/0       │                          │
                              └──────────────┘                          │
                                                                        ↓
                                                                ┌──────────────┐
                                                                │     R1       │
                                                                │  (Сервер)    │
                                                                │ г0/0/0       │
                                                                └──────────────┘
                                                                        ↑
                                                          ── DHCPv6 REPLY ──
                                        
```

### 5.2 Настройка DHCPv6 Relay на R2

```bash
R2# configure terminal

# Конфигурация интерфейса G0/0/1 (сторона клиента)
R2(config)# interface gigabitethernet 0/0/1
R2(config-if)# ipv6 nd managed-config-flag
R2(config-if)# ipv6 dhcp relay destination 2001:db8:acad:2::1 gigabitethernet 0/0/0
R2(config-if)# exit

R2(config)# end
R2# write memory
```

**Объяснение команд:**

| Команда | Описание |
|---------|---------|
| `ipv6 nd managed-config-flag` | Установка флага M в Router Advertisement (переход в Stateful режим) |
| `ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0` | Перенаправление DHCPv6 запросов на сервер R1 через интерфейс g0/0/0 |

### 5.3 Проверка конфигурации Relay

```bash
# На R2 проверить конфигурацию
R2# show ipv6 dhcp interface gigabitethernet 0/0/1

GigabitEthernet0/0/1 is in relay mode
  Relay Information option is enabled
  DHCPv6 relay destination: 2001:db8:acad:2::1 via GigabitEthernet0/0/0
```

### 5.4 Проверка на PC-B

1. Перезагрузите PC-B
2. Выполните `ipconfig /all`

**Ожидаемый результат:**

```
FastEthernet0 Connection (default port)
Connection-specific DNS Suffix..: STATEFUL.com
Physical Address..................: 00E0.8F16.7B53
Link-local IPv6 Address...........: FE80::2E0:8FFF:FE16:7B53
IPv6 Address......................: 2001:DB8:ACAD:3:AAA:XXXX:XXXX:XXXX
Autoconfiguration IP Address......: (не используется)
Subnet Mask........................: 255.255.0.0
Default Gateway....................: FE80::1
DHCPv6 IAID........................: (номер)
DHCPv6 Client DUID................: (DUID от MAC адреса)
DNS Servers........................: 2001:DB8:ACAD::254
```

### 5.5 Проверка подключения через Relay

```bash
# На PC-B проверить связность
C:\> ping 2001:db8:acad:1::1

Pinging 2001:db8:acad:1::1 with 32 bytes of data:
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254

Ping statistics for 2001:DB8:ACAD:1::1:
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

---

## Проверочные команды и диагностика

### Проверка конфигурации DHCPv6

```bash
# На R1: Просмотр всех пулов DHCPv6
R1# show ipv6 dhcp pool

DHCPv6 pool: R1-STATELESS
  DNS server: 2001:DB8:ACAD::254
  Domain name: STATELESS.com
  Active clients: 1

DHCPv6 pool: R2-STATEFUL
  Prefix: 2001:DB8:ACAD:3:AAA::/80
  Prefix duration: infinite (preferred)
  DNS server: 2001:DB8:ACAD::254
  Domain name: STATEFUL.com
  Active clients: 1
```

### Проверка привязки DHCPv6

```bash
# На R1: Проверка привязок пулов к интерфейсам
R1# show ipv6 dhcp interface

GigabitEthernet0/0/0 is in server mode
  Server: R2-STATEFUL
  
GigabitEthernet0/0/1 is in server mode
  Server: R1-STATELESS
  Other config flag is set
```

### Проверка статистики DHCPv6

```bash
# На R1: Статистика DHCPv6
R1# show ipv6 dhcp statistics

DHCPv6 Global Statistics:
  Messages received:  2
    SOLICIT: 1
    REQUEST: 1
  Messages sent:      2
    ADVERTISE: 1
    REPLY: 1
```

### Отладка на клиентах

```bash
# На PC-A или PC-B: Полная информация о конфигурации
C:\> ipconfig /all

# Проверка IPv6 маршрутов
C:\> route print -6

# Проверка DNS резолюции
C:\> nslookup www.example.com
```

---

## Типичные проблемы и решения

| Проблема | Причина | Решение |
|----------|---------|---------|
| PC не получает адрес SLAAC | IPv6 маршрутизация не активирована | `ipv6 unicast-routing` на маршрутизаторе |
| Отсутствует DNS в Stateless режиме | Флаг O не установлен | `ipv6 nd other-config-flag` на интерфейсе |
| PC-B не получает адреса через Relay | Relay не настроен на R2 | Настроить `ipv6 dhcp relay destination` |
| Маршрутизаторы не видят друг друга | Маршруты по умолчанию не настроены | Добавить `ipv6 route ::/0 ...` |
| Link-local адреса некорректные | Нарушен формат команды | Использовать `ipv6 address fe80::X link-local` |

---

## Структура IPv6 адреса в лабораторной работе

```
┌─────────────────────────────────────────────────────────────┐
│           2001:db8:acad:1:204:9aff:fe1b:78d4               │
├─────────────────────────────────────────────────────────────┤
│  Глобальный префикс    │ Subnet ID │      Interface ID      │
│   2001:db8:acad:1      │     0     │  204:9aff:fe1b:78d4    │
├─────────────────────────────────────────────────────────────┤
│  (первые 48 бит)       │  16 бит   │      64 бита           │
└─────────────────────────────────────────────────────────────┘
```

---

## Справочные материалы

### IPv6 адресация
- **2001:db8::/32** – документационный префикс (используется в примерах)
- **fe80::/10** – link-local адреса (автоматически генерируются)
- **ff00::/8** – multicast адреса

### DHCPv6 режимы

| Режим | Адреса | Параметры | Использование |
|-------|--------|-----------|----------------|
| **SLAAC** | Генерируются клиентом | Нет параметров | Автоматическое конфигурирование |
| **Stateless DHCPv6** | Генерируются клиентом | DNS, Domain | Большие подсети |
| **Stateful DHCPv6** | Назначаются сервером | Полная конфигурация | Управляемые подсети |

### Флаги Router Advertisement

- **M-флаг** (Managed Address Configuration) = 1: Использовать Stateful DHCPv6
- **M-флаг** = 0, **O-флаг** (Other Stateful Configuration) = 1: SLAAC + Stateless DHCPv6
- **M-флаг** = 0, **O-флаг** = 0: Только SLAAC

---

## Заключение

Данная лабораторная работа демонстрирует:
- ✅ Полный цикл настройки IPv6 сети
- ✅ Три различные методики получения адресов клиентами
- ✅ Использование DHCPv6 Relay для многосетевых окружений
- ✅ Практические навыки диагностики и отладки IPv6

Полученные знания применяются в реальных корпоративных сетях, где IPv6 становится стандартным протоколом маршрутизации.

---

**Автор:** Cisco Networking Academy  
**Последнее обновление:** 2025  
**Лицензия:** Creative Commons Attribution License
