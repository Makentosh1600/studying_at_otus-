# Лабораторная работа. Просмотр таблицы MAC-адресов коммутатор 
**Топология**  

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/1.png)  

| Устройство    | Интерфейс          | IP-адрес | маска под сети |
| ------------- |:------------------:| -----:| -----:|
| **S1**     | VLAN1   | 192.168.1.11 | 255.255.255.0 |
| **S2**     | VLAN1   | 192.168.1.12 | 255.255.255.0 |
| **PC-A**    | NIC |   192.168.1.1 | 255.255.255.0 |
| **PC-B**    | NIC |   192.168.1.2 | 255.255.255.0 |

**Цели**  
Часть 1. Создание и настройка сети
Часть 2. Изучение таблицы МАС-адресов коммутатора

## Часть 1. Создание и настройка сети  
### Шаг 1. Подключите сеть в соответствии с топологией.     
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/1.png)    
### Шаг 2. Настройте узлы ПК. 
На примере PC-A:   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/2.png) 
### Шаг 3. Выполните инициализацию и перезагрузку коммутаторов.   
### Шаг 4. Настройте базовые параметры каждого коммутатора  
a.	Настройте имена устройств в соответствии с топологией.  
b.	Настройте IP-адреса, как указано в таблице адресации.  
c.	Назначьте cisco в качестве паролей консоли и VTY.  
d.	Назначьте class в качестве пароля доступа к привилегированному режиму EXEC.  
**Пример настройки коммутатора S1:**  
Switch>enab  
Switch#conf  
Configuring from terminal, memory, or network [terminal]?   
Enter configuration commands, one per line. End with CNTL/Z.  
Switch(config)#hostname S1  
S1(config)#no ip domain-lookup  
S1(config)#service password-encryption  
S1(config)#enable secret class   
S1(config)#banner motd #   
Enter TEXT message. End with the character '#'.  
Unauthorized access is strictly prohibited. #  
S1(config)#interface vlan 1   
S1(config-if)#ip address 192.168.1.11 255.255.255.0  
S1(config-if)#no shutdown  

  
S1(config-if)#  
%LINK-5-CHANGED: Interface Vlan1, changed state to up   
S1(config-if)#   
%LINK-5-CHANGED: Interface FastEthernet0/6, changed state to up  
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/6, changed state to up   
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up   
%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up   
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up   
S1(config-if)#   
S1#   
%SYS-5-CONFIG_I: Configured from console by console   
S1#conf   
Configuring from terminal, memory, or network [terminal]?    
Enter configuration commands, one per line. End with CNTL/Z.   
S1(config)#line con 0   
S1(config-line)#logging synchronous   
S1(config-line)#password cisco    
S1(config-line)#login   
S1#conf   
Configuring from terminal, memory, or network [terminal]?    
Enter configuration commands, one per line. End with CNTL/Z.   
S1(config)#line vty 0 4   
S1(config-line)#password class   
S1(config-line)#transport input telnet   
S1(config-line)#login   

## Часть 2. Изучение таблицы МАС-адресов коммутатора    
Как только между сетевыми устройствами начинается передача данных, коммутатор выясняет МАС-адреса и строит таблицу.  
### Шаг 1. Запишите МАС-адреса сетевых устройств.   
a.	Откройте командную строку на PC-A и PC-B и введите команду **ipconfig /all**.    
Открытие окна командной строки Windows     
**Вопрос: Назовите физические адреса адаптера Ethernet.**  
MAC-адрес компьютера **PC-A**: *00D0.BCA3.9B0A*  
MAC-адрес компьютера **PC-B**: *00E0.F780.8407*  
Закройте окно командной строки.   
b.	Подключитесь к коммутаторам S1 и S2 через консоль и введите команду **show interface F0/1** на каждом коммутаторе.  
Откройте окно конфигурации  
**Вопросы: Назовите адреса оборудования во второй строке выходных данных команды (или зашитый адрес — bia).**  
МАС-адрес коммутатора **S1 Fast Ethernet 0/1**: *0001.9707.7901*  
МАС-адрес коммутатора **S2 Fast Ethernet 0/1**: *00d0.ff19.4401*  

### Шаг 2. Просмотрите таблицу МАС-адресов коммутатора.   
Подключитесь к коммутатору S2 через консоль и просмотрите таблицу МАС-адресов до и после тестирования сетевой связи с помощью эхо-запросов.  
a.	Подключитесь к коммутатору S2 через консоль и войдите в привилегированный режим EXEC.  
Откройте окно конфигурации   
b.	В привилегированном режиме EXEC введите команду show mac address-table и нажмите клавишу ввода.  
S2# show mac address-table  
Даже если сетевая коммуникация в сети не происходила (т. е. если команда ping не отправлялась), коммутатор может узнать МАС-адреса при подключении к ПК и другим коммутаторам.  
**Записаны ли в таблице МАС-адресов какие-либо МАС-адреса?**  
*Да. MAC адрес коммутатора S1*   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/3.png)     
*После команды ping со стороны PC-A, появились MAC адреса PC-A и PC-B*  
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/4.png)      
**Какие МАС-адреса записаны в таблице? С какими портами коммутатора они сопоставлены и каким устройствам принадлежат? Игнорируйте МАС-адреса, сопоставленные с центральным процессором.**
*Записаны MAC адреса PC-A, PC-B, S1.*  

**Если вы не записали МАС-адреса сетевых устройств в шаге 1, как можно определить, каким устройствам принадлежат МАС-адреса, используя только выходные данные команды show mac address-table? Работает ли это решение в любой ситуации?**
*Можно использовать данные из таблицы МАС адресов, так как есть указания номеров портов. Но если к порту подключено не единичное устройство, а допустим коммутатор с подключенными многочисленными устройствами, то они будут иметь в таблице один и тот же порт.*

### Шаг 3. Очистите таблицу МАС-адресов коммутатора S2 и снова отобразите таблицу МАС-адресов.   
a.	В привилегированном режиме EXEC введите команду **clear mac address-table dynamic** и нажмите клавишу Enter.  
S2# clear mac address-table dynamic   
b.	Снова быстро введите команду **show mac address-table**.  
**Вопросы: Указаны ли в таблице МАС-адресов адреса для VLAN 1? Указаны ли другие МАС-адреса?**  
*МАС-адрес VLAN 1 не указан.*
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/5.png)   
Через 10 секунд введите команду **show mac address-table** и нажмите клавишу ввода. Появились ли в таблице МАС-адресов новые адреса?  
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/6.png)    
### Шаг 4. С компьютера PC-B отправьте эхо-запросы устройствам в сети и просмотрите таблицу МАС-адресов коммутатора.   
a.	На компьютере PC-B откройте командную строку и еще раз введите команду **arp -a**.  
Откройте командную строку. 
**Вопрос: Не считая адресов многоадресной и широковещательной рассылки, сколько пар IP- и МАС-адресов устройств было получено через протокол ARP?**   
*Три: S1 (VLAN 1), S2 (VLAN 2), PC-A*
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/7.png)    
b.	Из командной строки PC-B отправьте эхо-запросы на компьютер PC-A, а также коммутаторы S1 и S2.  
**Вопрос: От всех ли устройств получены ответы? Если нет, проверьте кабели и IP-конфигурации.**   
*Да, от всех*   
c.	Подключившись через консоль к коммутатору S2, введите команду **show mac address-table**.  
Откройте окно конфигурации   
**Вопрос: Добавил ли коммутатор в таблицу МАС-адресов дополнительные МАС-адреса? Если да, то какие адреса и устройства?**   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/8.png)    
*Добавился порт VLAN 1 коммутатора S1 с MAC адресом 0001.с978.c88b*   
На компьютере PC-B откройте командную строку и еще раз введите команду **arp -a**.  
**Вопрос: Появились ли в ARP-кэше компьютера PC-B дополнительные записи для всех сетевых устройств, которым были отправлены эхо-запросы?**   
*НЕТ*  
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.2/JPG/9.png)     

## Вопрос для повторения  

**В сетях Ethernet данные передаются на устройства по соответствующим МАС-адресам. Для этого коммутаторы и компьютеры динамически создают ARP-кэш и таблицы МАС-адресов. Если компьютеров в сети немного, эта процедура выглядит достаточно простой. Какие сложности могут возникнуть в крупных сетях?**  
*Переполнение памяти коммутатора МАС адресов. Снижение безопасности сети из-за применения широковещательного трафика*


