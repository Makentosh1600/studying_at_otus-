# Лабораторная работа. Развертывание коммутируемой сети с резервными каналами  
**Топология**    

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/01.png)   
   **Таблица адресации**     
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/02.png)   
	**Цели**   
Часть 1. Создание сети и настройка основных параметров устройства   
Часть 2. Выбор корневого моста   
Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов   
Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов   

## Часть 1:	Создание сети и настройка основных параметров устройства  
### Шаг 1:	Создайте сеть согласно топологии.   
Подключите устройства, как показано в топологии, и подсоедините необходимые кабели.   
### Шаг 2:	Выполните инициализацию и перезагрузку коммутаторов.   
### Шаг 3:	Настройте базовые параметры каждого коммутатора.   
a.	Отключите поиск DNS.   
b.	Присвойте имена устройствам в соответствии с топологией.   
c.	Назначьте class в качестве зашифрованного пароля доступа к привилегированному режиму.   
d.	Назначьте cisco в качестве паролей консоли и VTY и активируйте вход для консоли и VTY каналов.   
e.	Настройте logging synchronous для консольного канала.   
f.	Настройте баннерное сообщение дня (MOTD) для предупреждения пользователей о запрете несанкционированного доступа.   
g.	Задайте IP-адрес, указанный в таблице адресации для VLAN 1 на всех коммутаторах.   
h.	Скопируйте текущую конфигурацию в файл загрузочной конфигурации.   
**На примере S2:**   
*Switch(config)#hostname S2*    
*S2(config)#no ip domain-lookup*   
*S2(config)#ip domain-name otus.ru*  
*S2(config)#service password-encryption*   
*S2(config)#enable secret class*   
*S2(config)#banner motd #*    
*Enter TEXT message. End with the character '#'.*  
*Unauthorized access is strictly prohibited.#*   
*S2(config)#line con 0*   
*S2(config-line)#logging synchronous*   
*S2(config-line)#password cisco*  
*S2(config-line)#login*   
*S2(config-line)#exit*  
*S2(config)#interf vlan 1*   
*S2(config-if)#ip address 192.168.1.2 255.255.255.0*  
*S2(config-if)#no shutdown*  
*S2(config-if)#crypto key generate rsa*  
*The name for the keys will be: S2.otus.ru*  
*Choose the size of the key modulus in the range of 360 to 2048 for your General Purpose Keys. Choosing a key modulus greater than 512 may take
a few minutes.*   
*How many bits in the modulus [512]: 2048*   
*% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]*   
*S2(config)#do sh ip ssh*   
**Mar 2 4:59:40.759: %SSH-5-ENABLED: SSH 1.99 has been enabled*    
*SSH Enabled - version 1.99*    
*Authentication timeout: 120 secs; Authentication retries: 3*   
*S2(config)#ip ssh version 2*   
*S2(config)#username admin privilege 15 secret AdminSSH*    
*S2(config)#line vty 0 4*    
*S2(config-line)#login local*     
*S2(config-line)#transport input ssh*    
*S2#copy running-config startup-config*
*Destination filename [startup-config]?*    
*Building configuration...*   
### Шаг 4:	Проверьте связь.   
Проверьте способность компьютеров обмениваться эхо-запросами.   
**Вопрос. Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S2?**    
*-Да.*    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/03.png)    
**Вопрос. Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S3?**   
*-Да.*   

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/04.png)   
**Вопрос. Успешно ли выполняется эхо-запрос от коммутатора S2 на коммутатор S3?**    
*-Да.*    

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/05.png)   
## Часть 2:	Определение корневого моста
### Шаг 1: Отключите все порты на коммутаторах.   
**На примере S2:**     
*S2#conf t*   
*S2(config)#inter rang f0/1-24*   
*S2(config-if-range)#shutd*   

### Шаг 2: Настройте подключенные порты в качестве транковых.   
**На примере S1:**  
*S1(config)#inter rang f0/1-4*   
*S1(config-if-range)#switchport mode trunk*   
*S1(config-if-range)#switchport trunk native vlan 1*   
*S1(config-if-range)#switchport trunk allowed vlan 1*   

### Шаг 3: Включите порты F0/2 и F0/4 на всех коммутаторах.      
**На примере S2:**   
*S2(config)#inter rang f0/2, f0/4*   
*S2(config-if-range)#no shutd*   
*%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to down
S2(config-if-range)#*    
*%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to up*   
*%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up*   

### Шаг 4: Отобразите данные протокола spanning-tree.
**S1#show spanning-tree**   
*VLAN0001*   
*Spanning tree enabled protocol ieee*   
*Root ID Priority 32769*   
*Address 000B.BE03.CAD9*   
*Cost 19*   
*Port 4(FastEthernet0/4)*   
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*
 
*Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)*   
*Address 00E0.F71C.22C7*  
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*   
*Aging Time 20*   

*Interface Role Sts Cost Prio.Nbr Type*   
*---------------- ---- --- --------- -------- --------------------------------*  
*Fa0/4 Root FWD 19 128.4 P2p*   
*Fa0/2 Altn BLK 19 128.2 P2p*   

**S2#show spanning-tree**   
*VLAN0001*   
*Spanning tree enabled protocol ieee*   
*Root ID Priority 32769*   
*Address 000B.BE03.CAD9*   
*Cost 19*   
*Port 4(FastEthernet0/4)*   
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*   

*Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)*  
*Address 000B.BE6B.D81A*   
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*  
*Aging Time 20*   

*Interface Role Sts Cost Prio.Nbr Type*  
*---------------- ---- --- --------- -------- --------------------------------*  
*Fa0/4 Root FWD 19 128.4 P2p*  
*Fa0/2 Desg FWD 19 128.2 P2p*  

**S3#show spanning-tree**    
*VLAN0001*   
*Spanning tree enabled protocol ieee*   
*Root ID Priority 32769*   
*Address 000B.BE03.CAD9*   
*This bridge is the root*  
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*  

*Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)*  
*Address 000B.BE03.CAD9*   
Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*  
*Aging Time 20*   

*Interface Role Sts Cost Prio.Nbr Type*   
*---------------- ---- --- --------- -------- --------------------------------*  
*Fa0/2 Desg FWD 19 128.2 P2p*   
*Fa0/4 Desg FWD 19 128.4 P2p*  

**Вопрос. В схему ниже запишите роль и состояние (Sts) активных портов на каждом коммутаторе в топологии.**    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/06.png)   
**Вопрос. Какой коммутатор является корневым мостом?**   
*-S3*    
**Вопрос. Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?**    
*- Значение MAC адреса самый малый*   
**Вопрос. Какие порты на коммутаторе являются корневыми портами?**   
*- На S1: F0/2; на S2: F0/4*   
**Вопрос. Какие порты на коммутаторе являются назначенными портами?**  
*- На S3: F0/4, F0/2;  на S2: F0/2*   
**Вопрос. Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?**   
*- На S1: F0/4*   
**Вопрос. Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?**   
*- На основе вычисления BID и Port ID так как стоимость порта равна (Path Cost)*   

## Часть 3: Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов  

### Шаг 1: Определите коммутатор с заблокированным портом.   
*- На S1: F0/4*   
### Шаг 2: Измените стоимость порта.   
*S1#show span*  
*VLAN0001*   
*Spanning tree enabled protocol ieee*   
*Root ID Priority 32769*   
*Address 000B.BE03.CAD9*   
*Cost 19*  
*Port 4(FastEthernet0/4)*   
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*   

*Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)*  
*Address 00E0.F71C.22C7*   
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*  
*Aging Time 20*   

*Interface Role Sts Cost Prio.Nbr Type*   
*---------------- ---- --- --------- -------- --------------------------------*  
*Fa0/4 Root FWD 19 128.4 P2p*   
*Fa0/2 Altn BLK 19 128.2 P2p*  

*S1#conf t*  
*Enter configuration commands, one per line. End with CNTL/Z.*  
*S1(config)#inter f0/4*   
*S1(config-if)#spanning-tree vlan 1 cost 18*   

### Шаг 3: Просмотрите изменения протокола spanning-tree.   
**S1#show span**   
*VLAN0001*   
*Spanning tree enabled protocol ieee*   
*Root ID Priority 32769*  
*Address 000B.BE03.CAD9*   
*Cost 18*   
*Port 4(FastEthernet0/4)*  
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*   

*Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)*  
*Address 00E0.F71C.22C7*   
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*   
*Aging Time 20*  

*Interface Role Sts Cost Prio.Nbr Type*   
*---------------- ---- --- --------- -------- --------------------------------*  
*Fa0/4 Root FWD **18** 128.4 P2p*   
*Fa0/2 **Desg FWD** 19 128.2 P2p*      

**S2#show span**   
*VLAN0001*   
*Spanning tree enabled protocol ieee*   
*Root ID Priority 32769*   
*Address 000B.BE03.CAD9*  
*Cost 19*   
*Port 4(FastEthernet0/4)*  
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*   

*Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)*   
*Address 000B.BE6B.D81A*  
*Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec*  
*Aging Time 20*  

*Interface Role Sts Cost Prio.Nbr Type*  
*---------------- ---- --- --------- -------- --------------------------------*  
*Fa0/4 Root FWD 19 128.4 P2p*   
*Fa0/2 **Altn BLK** 19 128.2 P2p*  

**Вопрос. Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?**   
*- Из-за того, что изменилась стоимость порта, изменилась и стоимость пути до корневого коммутатора. Произошел обмен кадрами BPDU и перерасчет пути.*   

## Шаг 4:	Удалите изменения стоимости порта.   

a.	Выполните команду **no spanning-tree vlan 1 cost 18** режима конфигурации интерфейса, чтобы удалить запись стоимости, созданную ранее.   
*S1#conf t*   
*Enter configuration commands, one per line. End with CNTL/Z.*   
*S1(config)#inter f0/4*   
*S1(config-if)#no spanning-tree vlan 1 cost 18*   
b.	Повторно выполните команду **show spanning-tree**, чтобы подтвердить, что протокол STP сбросил порт на коммутаторе некорневого моста, вернув исходные настройки порта. Протоколу STP требуется примерно 30 секунд, чтобы завершить процесс перевода порта.   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/07.png)     

## Часть 4: Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов   

a.	Включите порты F0/1 и F0/3 на всех коммутаторах.   
**На примере S3:**     
*S3#conf t*   
*S3(config)#interf range f0/1, f0/3*   
*S3(config-if-range)#no shutd*   

b.	Подождите 30 секунд, чтобы протокол STP завершил процесс перевода порта, после чего выполните команду **show spanning-tree** на коммутаторах некорневого моста. Обратите внимание, что порт корневого моста переместился на порт с меньшим номером, связанный с коммутатором корневого моста, и заблокировал предыдущий порт корневого моста.    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/08.png)    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/lab%201.7/JPG/09.png)      

**Вопрос. Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?**   
*- на S1: F0/3; на S2: F0/3*    
**Вопрос. Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?**   
*- C учетом BID и стоимости пути.*   
## Вопросы для повторения   
 **Вопрос. 1.	Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?**    
 *- BID поле*   
 **Вопрос. 2.	Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?**   
 *- Стоимость порта (Path Cost)*   
 **Вопрос. 3.	Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?**   
 *- идентификатор порта (Port ID), обычно номер порта.*   
 
 

















