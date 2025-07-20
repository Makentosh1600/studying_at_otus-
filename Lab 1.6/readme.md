# Лабораторная работа - Внедрение маршрутизации между виртуальными локальными сетями  
**Топология**  

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.6/JPG/01.png)    

   **Таблица адресации**    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.6/JPG/02.png)   
**Таблица VLAN**
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.6/JPG/03.png)

**Задачи**  
Часть 1. Создание сети и настройка основных параметров устройства  
Часть 2. Создание сетей VLAN и назначение портов коммутатора   
Часть 3. Настройка транка 802.1Q между коммутаторами.   
Часть 4. Настройка маршрутизации между сетями VLAN   
Часть 5. Проверка, что маршрутизация между VLAN работает   

## Часть 1. Создание сети и настройка основных параметров устройства   
В первой части лабораторной работы вам предстоит создать топологию сети и настроить базовые параметры для узлов ПК и коммутаторов.   
### Шаг 1. Создайте сеть согласно топологии.   
Подключите устройства, как показано в топологии, и подсоедините необходимые кабели.   
### Шаг 2. Настройте базовые параметры для маршрутизатора.   
a.	Подключитесь к маршрутизатору с помощью консоли и активируйте привилегированный режим EXEC.   
Откройте окно конфигурации   
b.	Войдите в режим конфигурации.   
c.	Назначьте маршрутизатору имя устройства.   
d.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.  
e.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.   
f.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.   
g.	Установите cisco в качестве пароля виртуального терминала и активируйте вход.   
h.	Зашифруйте открытые пароли.   
i.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.   
j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.   
k.	Настройте на маршрутизаторе время.   
**Настройка R1**   
*Router#conf t*  
*Router(config)#hostname R1*   
*R1(config)#no ip domain-lookup*   
*R1(config)#service password-encryption*   
*R1(config)#enable secret class*   
*R1(config)#banner motd #*   
*Enter TEXT message. End with the character '#'.*
*Unauthorized access is strictly prohibited. #*   
*R1(config)#*   
*R1(config)#ip domain-name otus.ru*   
*R1(config)#line con 0*   
*R1(config-line)#logging synchronous*   
*R1(config-line)#password cisco*   
*R1(config-line)#login*   
*R1(config-line)#exit*   
*R1#clock set 14:45:00 20 July 2025*   
*R1#show clock*   
*14:45:11.121 UTC Sun Jul 20 2025*   
*R1#copy running-config startup-config*   
*Destination filename [startup-config]?*    
*Building configuration...*    
*[OK]*   
### Шаг 3. Настройте базовые параметры каждого коммутатора.   
a.	Присвойте коммутатору имя устройства.   
b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.   
c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.    
d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.    
e.	Установите cisco в качестве пароля виртуального терминала и активируйте вход.    
f.	Зашифруйте открытые пароли.   
g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.    
h.	Настройте на коммутаторах время.    
i.	Сохранение текущей конфигурации в качестве начальной.   
**НА ПРИМЕРЕ S1** 
*Switch(config)#hostname S1*   
*S1(config)#no ip domain-lookup*   
*S1(config)#service password-encryption*   
*S1(config)#enable secret class*   
*S1(config)#banner motd #*   
*Enter TEXT message. End with the character '#'.*  
*unauthorized access is strictly prohibited. #*   
*S1(config)#ip domain-name otus.ru*   
*S1(config)#line con 0*   
*S1(config-line)#logging synchronous*  
*S1(config-line)#password cisco*   
*S1(config-line)#login*  
*S1(config-line)#exit*  
*S1(config)#exit*   
*S1#clock set 14:55:00 20 Jul 2025*   
*S1#copy running-config startup-config*  
*Destination filename [startup-config]?*   
*Building configuration...*  
*[OK]*     
### Шаг 4. Настройте узлы ПК.   
Адреса ПК можно посмотреть в таблице адресации.   
## Часть 2. Создание сетей VLAN и назначение портов коммутатора  
Во второй части вы создадите VLAN, как указано в таблице выше, на обоих коммутаторах. Затем вы назначите VLAN соответствующему интерфейсу и проверите настройки конфигурации. Выполните следующие задачи на каждом коммутаторе.    
### Шаг 1. Создайте сети VLAN на коммутаторах.   
a.	Создайте и назовите необходимые VLAN на каждом коммутаторе из таблицы выше.   
Откройте окно конфигурации   
b.	Настройте интерфейс управления и шлюз по умолчанию на каждом коммутаторе, используя информацию об IP-адресе в таблице адресации.    
c.	Назначьте все неиспользуемые порты коммутатора VLAN Parking_Lot, настройте их для статического режима доступа и административно деактивируйте их.   
*Примечание. Команда interface range полезна для выполнения этой задачи с минимальным количеством команд.*   
### Шаг 2. Назначьте сети VLAN соответствующим интерфейсам коммутатора.   
a.	Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.   
b.	Убедитесь, что VLAN назначены на правильные интерфейсы.    
**НА ПРИМЕРЕ КОММУТАТОРА S2**   
*S2(config)#vlan 10*   
*S2(config-vlan)#name Management*   
*S2(config-vlan)#exit*   
*S2(config)#vlan 20*   
*S2(config-vlan)#name Sales*  
*S2(config-vlan)#exit*   
*S2(config)#vlan 30*   
*S2(config-vlan)#name Operations*  
*S2(config-vlan)#exit*  
*S2(config)#inter f0/18*   
*S2(config-if)#switchport mode access*  
*S2(config-if)#switchport access vlan 30*  
*S2(config-if)#switchport nonegotiate*  
*S2(config-if)#exit*   
*S2(config)#vlan 999*   
*S2(config-vlan)#name Parking_Lot*   
*S2(config-vlan)#exit*  
*S2(config)#int ra fa0/2-17, fa0/19-24,G0/1-2*  
*S2(config-if-range)#switchport mode access*   
*S2(config-if-range)#switchport access vlan 999*   
*S2(config-if-range)#switchport nonegotiate*    
*S2(config-if-range)#shutdown*  
*S2(config-if-range)#exit*   
*S2(config)#vlan 1000*  
*S2(config-vlan)#name Privat*  
*S2(config-vlan)#exit*  
*S2(config)#inter vlan 10*   
*S2(config-if)#***
*S2(config-if)#ip address 192.168.10.12 255.255.255.0*   
*S2(config-if)#no shutdown*   
*S2(config-if)#ip default-gateway 192.168.10.1*   
*S2(config)#exit*  
*S2#show vlan br*  
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.6/JPG/04.png)     
## Часть 3. Конфигурация магистрального канала стандарта 802.1Q между коммутаторами   
В части 3 вы вручную настроите интерфейс F0/1 как транк.   
### Шаг 1. Вручную настройте магистральный интерфейс F0/1 на коммутаторах S1 и S2.   
a.	Настройка статического транкинга на интерфейсе F0/1 для обоих коммутаторов.   
Откройте окно конфигурации   
b.	Установите native VLAN 1000 на обоих коммутаторах.   
c.	Укажите, что VLAN 10, 20, 30 и 1000 могут проходить по транку.   
d.	Проверьте транки, native VLAN и разрешенные VLAN через транк.   
**НА ПРИМЕРЕ S1**    
*S1(config)#inter fa0/1*   
*S1(config-if)#switchport mode trunk*   
*S1(config-if)#switch trunk native vlan 1000*   
*S1(config-if)#switchport trunk allowed vlan 10,20,30,1000*   
### Шаг 2. Вручную настройте магистральный интерфейс F0/5 на коммутаторе S1.   
a.	Настройте интерфейс S1 F0/5 с теми же параметрами транка, что и F0/1. Это транк до маршрутизатора.   
b.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.   
c.	Проверка транкинга.   
**НАСТРОЙКА F0/5 НА КОММУТАТОРЕ S1**   
*S1(config)#interface fa0/5*   
*S1(config-if)#switchport mode trunk*  
*S1(config-if)#switchport trunk native vlan 1000*   
*S1(config-if)#switchport trunk allowed vlan 10,20,30,1000*   
*S1(config-if)#*   
*S1(config-if)#exit*    


**Вопрос: Что произойдет, если G0/0/1 на R1 будет отключен?**   
*-	Будет отсутствовать связь между VLAN 10, 20 и 30*    
## Часть 3. Настройка маршрутизации между сетями VLAN   
### Шаг 1. Настройте маршрутизатор.   
Откройте окно конфигурации    
a.	При необходимости активируйте интерфейс G0/0/1 на маршрутизаторе.   
b.	Настройте подинтерфейсы для каждой VLAN, как указано в таблице IP-адресации. Все подинтерфейсы используют инкапсуляцию 802.1Q. Убедитесь, что подинтерфейсу для native VLAN не назначен IP-адрес. Включите описание для каждого подинтерфейса.    
c.	Убедитесь, что вспомогательные интерфейсы работают    
*R1(config)#interf g0/1*   
*R1(config-if)#exit*   
*R1(config)#interf g0/1.10*   
*R1(config-subif)#description defual Gateway for VLAN 10*   
*R1(config-subif)#encapsulation dot1Q 10*   
*R1(config-subif)#ip add 192.168.10.1 255.255.255.0*  
*R1(config-subif)#exit*   
*R1(config)#interf g0/1.20*   
*R1(config-subif)#description defual gateway for VLAN 20*   
*R1(config-subif)#encapsulation dot1Q 20*   
*R1(config-subif)#ip add 192.168.20.1 255.255.255.0*   
*R1(config-subif)#exit*   
*R1(config)#interf g0/1.30*   
*R1(config-subif)#description defual gateway for VLAN 30*   
*R1(config-subif)#encapsulation dot1Q 30*   
*R1(config-subif)#ip add 192.168.30.1 255.255.255.0*   
*R1(config-subif)#exit*
*R1(config)#interf g0/1*   
*R1(config-if)#description trunk link to S1*   
*R1(config-if)#no shut*   
*R1(config-if)#end*  
*R1#*   
## Часть 5. Проверьте, работает ли маршрутизация между VLAN    
### Шаг 1. Выполните следующие тесты с PC-A. Все должно быть успешно.   
a.	Отправьте эхо-запрос с PC-A на шлюз по умолчанию.   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.6/JPG/05.png)   
b.	Отправьте эхо-запрос с PC-A на PC-B.   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.6/JPG/06.png)    
c.	Отправьте команду ping с компьютера PC-A на коммутатор S2.    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.6/JPG/07.png)   
### Шаг 2. Пройдите следующий тест с PC-B   
В окне командной строки на PC-B выполните команду **tracert** на адрес PC-A.   
**Вопрос: Какие промежуточные IP-адреса отображаются в результатах?**
*192.168.30.1 – Маршрутизатор S1, порт G0/1*   
*192.168.20.3 – Компьютер PC-A*    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.6/JPG/08.png)   









