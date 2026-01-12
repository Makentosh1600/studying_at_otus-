# Лабораторная работа: Настройка NAT для IPv4
## Топология   
![Топология сети](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/01.jpg)   

## Таблица адресации

| Устройство | Интерфейс | IP-адрес | Маска подсети |
| :--- | :--- | :--- | :--- |
| **R1** | G0/0/0 | 209.165.200.230 | 255.255.255.248 |
| | G0/0/1 | 192.168.1.1 | 255.255.255.0 |
| **R2** | G0/0/0 | 209.165.200.225 | 255.255.255.248 |
| | Lo1 | 209.165.200.1 | 255.255.255.224 |
| **S1** | VLAN 1 | 192.168.1.11 | 255.255.255.0 |
| **S2** | VLAN 1 | 192.168.1.12 | 255.255.255.0 |
| **PC-A** | NIC | 192.168.1.2 | 255.255.255.0 |
| **PC-B** | NIC | 192.168.1.3 | 255.255.255.0 |

## Общие сведения и сценарий    
Преобразование (NAT) — это процесс, при котором сетевое устройство, например маршрутизатор Cisco, назначает публичный адрес узлам в пределах частной сети. NAT используют для сокращения количества публичных IP-адресов, используемых организацией, поскольку количество доступных публичных IPv4-адресов ограничено.
Интернет-провайдер выделил компании общедоступное пространство IP-адресов 209.165.200.224/29. Эта сеть используется для обращения к каналу между маршрутизатором ISP (R2) и шлюзом компании (R1). Первый адрес (209.165.200.225) назначается интерфейсу g0/0 на R2, а последний адрес (209.165.200.230) назначается интерфейсу g0/0/0 на R1. Остальные адреса (209.165.200.226-209.165.200.229) будут использоваться для предоставления доступа в Интернет хостам компании. Маршрут по умолчанию используется от R1 до R2. Подключение интернет-провайдера к Интернету смоделировано loopback-адресом на маршрутизаторе интернет-провайдера.
В этой лабораторной работе  вы будете настраивать различные типы NAT. Вы выполните тестирование, отображение и проверку осуществления всех преобразований и проанализируете статистику NAT/PAT для контроля процесса.

## Часть 1. Создание сети и настройка базовых конфигурация   
### Шаг 1.1. Создание модели в СРТ
![Топология сети](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/02.jpg)   
### Шаг 1.2. Базовая настройка маршрутизаторов (R1, R2)    
```
hostname R1 (для второго R2)
no ip domain-lookup
enable secret class
line console 0
 password cisco
 login
line vty 0 4
 password cisco
 login
service password-encryption
banner motd "Unauthorized Access Prohibited"
```
Для R1:
```
interface g0/0
 ip address 209.165.200.230 255.255.255.248
 no shutdown
interface g0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit
ip route 209.165.200.0 255.255.255.224 209.165.200.225
```
Для R2:   
```
interface g0/0
 ip address 209.165.200.225 255.255.255.248
 no shutdown
ip route 0.0.0.0 0.0.0.0 209.165.200.230
write memory
```
Настройка Loopback  
```
interface loopback1
 ip address 209.165.200.1 255.255.255.224
```

### Шаг 1.3. Базовая настройка коммутаторов (S1, S2)   
```
hostname S1 (для второго S2)
no ip domain-lookup
enable secret class
line console 0
 password cisco
 login
line vty 0 15
 password cisco
 login
service password-encryption
banner motd "Unauthorized Access Prohibited"
```
Для S1:
```
interface vlan 1
 ip address 192.168.1.11 255.255.255.0
 no shutdown
ip default-gateway 192.168.1.1

interface range f0/2-4, f0/7-24, g0/1-2
 shutdown
write memory
```
Для S2:
```
interface vlan 1
 ip address 192.168.1.12 255.255.255.0
 no shutdown
ip default-gateway 192.168.1.1

interface range f0/2-17, f0/19-24, g0/1-2
 shutdown
write memory
```
## Часть 2. Настройка и проверка NAT для IPv4   
### Шаг 2.1. Настройка NAT на R1, используя пул из трех адресов 209.165.200.226 - 209.165.200.228  
```
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 netmask 255.255.255.248
ip nat inside source list 1 pool PUBLIC_ACCESS

interface g0/1
 ip nat inside
interface g0/0
 ip nat outside
```
### Шаг 2.2. Проверка конфигурации   
![Топология сети](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/03.jpg)    

#### Во что был транслирован внутренний локальный адрес PC-B?   
- 209.165.200.226
#### Какой тип адреса NAT является переведенным адресом?
- 192.168.1.3
### Шаг 2.3. Переполнение таблица выдачи IP адресов под NAT
С **PC-A** выполним ping до `209.165.200.1`.
С **S1** выполним ping до `209.165.200.1`.
С **S2** выполним ping до `209.165.200.1`.    
Комманда ping выполняется. При попытке выполнить на **PC-B** команда не выполнена    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/05.jpg)     

Проверка таблицы NAT:    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/04.jpg)    

Команда **show ip nat translations verbose** не выполняется в CPT
Команда **clear ip nat statistics** не выполняется в CPT

Командой clear ip nat translation * выполняем очистку таблиции NAT

## Часть 3: Настройка и проверка PAT для IPv4  
### Шаг 3.1. Переход от NAT к PAT (Address Pool) 
Удаляем старые настройки NAT на R1:
```cisco
no ip nat inside source list 1 pool PUBLIC_ACCESS
```
Добавляем настройку с ключевым словом **overload** (PAT):
```cisco
ip nat inside source list 1 pool PUBLIC_ACCESS overload
```
**Проверка**: Запустите ping с PC-A и PC-B одновременно. Проверка таблица NAT:   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/06.jpg)    

#### Во что был транслирован внутренний локальный адрес PC-B?   
- 209.165.200.226:Port
#### Какой тип адреса NAT является переведенным адресом?
- 192.168.1.3
#### Чем отличаются выходные данные команды show ip nat translations из упражнения NAT?   
- формат `IP:Port`
#### Как маршрутизатор отслеживает, куда идут ответы? 
- по `IP:Port` по **Port** 

### Шаг 3.2. Переход к PAT (Interface Overload)
Удаляем старые настройки NAT на R1:
```cisco
no ip nat inside source list 1 pool PUBLIC_ACCESS overload
no ip nat pool PUBLIC_ACCESS
```
Добавляем настройку для настройки PAT (Interface Overload) (PAT):
```cisco
ip nat inside source list 1 interface g0/0 overload
```
## Часть 4: Настройка статического NAT   
Настройка статического NAT для PC-A на R1:
```cisco 
ip nat inside source static 192.168.1.2 209.165.200.229 
```
**Проверка**:  
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/07.jpg)   

Пинг с R2 на 209.165.200.229:    

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/08.jpg)   

На R1 отобразим таблицу NAT с помощью команды *show ip nat translations*:   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.12/JPG/09.jpg)    





