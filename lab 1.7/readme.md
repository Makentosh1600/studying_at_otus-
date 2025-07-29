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








