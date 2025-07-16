# Лабораторная работа. Доступ к сетевым устройствам по протоколу SSH 
**Топология**  

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.5/JPG/01.png)     

**Таблица адресации**    

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.5/JPG/02.png)   

**Задачи**    
Часть 1. Настройка основных параметров устройства   
Часть 2. Настройка маршрутизатора для доступа по протоколу SSH   
Часть 3. Настройка коммутатора для доступа по протоколу SSH   
Часть 4. SSH через интерфейс командной строки (CLI) коммутатора   

## Часть 1. Настройка основных параметров устройств  
### Шаг 1. Создайте сеть согласно топологии.   
### Шаг 2. Выполните инициализацию и перезагрузку маршрутизатора и коммутатора.   
### Шаг 3. Настройте маршрутизатор.
a.	Подключитесь к маршрутизатору с помощью консоли и активируйте привилегированный режим EXEC.   
b.	Войдите в режим конфигурации.   
c.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.   
d.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.  
e.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.   
f.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.   
g.	Зашифруйте открытые пароли.   
h.	Создайте баннер, который предупреждает о запрете несанкционированного доступа.  
i.	Настройте и активируйте на маршрутизаторе интерфейс G0/0/1, используя информацию, приведенную в таблице адресации.   
j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.      
**РЕЗУЛЬТАТ:**   
*Router#conf t*   
*Router(config)#hostname R1*   
*R1(config)#no ip domain-lookup*   
*R1(config)#service password-encryption*   
*R1(config)#enable secret class*   
*R1(config)#banner motd #*   
*Enter TEXT message. End with the character '#'.*   
*Unauthorized acces is atrictly prohibited. #*  
*R1(config)#interface G0/0/1*  
*R1(config-if)#ip address 192.168.1.1 255.255.255.0*   
*R1(config-if)#no shutdown*   
*R1(config-line)#logging synchronous*   
*R1(config-line)#password cisco*   
*R1(config-line)#login*   
*R1(config-line)#exit*    
*R1(config)#exit*   
*R1#copy running-config startup-config*   
*Destination filename [startup-config]?*    
*Building configuration...*    
*[OK]*   

    ### Шаг 1. Настройте компьютер PC-A.    
a.	Настройте для PC-A IP-адрес и маску подсети.    
b.	Настройте для PC-A шлюз по умолчанию.    
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.5/JPG/03.png)     

### Шаг 2. Проверьте подключение к сети.    
Пошлите с PC-A команду Ping на маршрутизатор R1. Если эхо-запрос с помощью команды ping не проходит, найдите и устраните неполадки подключения.   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.5/JPG/04.png)    

## Часть 2. Настройка маршрутизатора для доступа по протоколу SSH   
Шаг 1. Настройте аутентификацию устройств.    
При генерации ключа шифрования в качестве его части используются имя устройства и домен. Поэтому эти имена необходимо указать перед вводом команды **crypto key**.   
Откройте окно конфигурации   
a.	Задайте имя устройства.   
b.	Задайте домен для устройства.   
Шаг 2. Создайте ключ шифрования с указанием его длины.    
Шаг 3. Создайте имя пользователя в локальной базе учетных записей.   
Настройте имя пользователя, используя **admin** в качестве имени пользователя и **Adm1nP @55** в качестве пароля.   
Шаг 4. Активируйте протокол SSH на линиях VTY.   
a.	Активируйте протоколы Telnet и SSH на входящих линиях VTY с помощью команды transport input.   
b.	Измените способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.   
**РЕЗУЛЬТАТ:**     
*R1#conf t*   
*Enter configuration commands, one per line. End with CNTL/Z.*   
*R1(config)#ip domain-name otus.ru*    
*R1(config)#crypto key generate rsa*    
*The name for the keys will be: R1.otus.ru*   
*Choose the size of the key modulus in the range of 360 to 2048 for your*   
*General Purpose Keys. Choosing a key modulus greater than 512 may take*   
*a few minutes.*    
*How many bits in the modulus [512]:*   
*% Generating 512 bit RSA keys, keys will be non-exportable...[OK]*    
*R1(config)#ip ssh version 2*   
**Mar 1 1:32:49.659: RSA key size needs to be at least 768 bits for ssh version 2*   
**Mar 1 1:32:49.659: %SSH-5-ENABLED: SSH 1.5 has been enabled*   
*Please create RSA keys (of at least 768 bits size) to enable SSH v2.*   
*R1(config)#username admin privilege 15 secret Adm1nP @55*    
*R1(config)#line vty 0 4*   
*R1(config-line)#login local*    
*R1(config-line)#transport input ssh*    

Шаг 1. Сохраните текущую конфигурацию в файл загрузочной конфигурации.   
Шаг 2. Установите соединение с маршрутизатором по протоколу SSH.   
Установите SSH-подключение к R1. Use the username admin and password Adm1nP @55. У вас должно получиться установить SSH-подключение к R1.   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.5/JPG/05.png)     
## Часть 3. Настройка коммутатора для доступа по протоколу SSH      
### Шаг 1. Настройте основные параметры коммутатора.   
Откройте окно конфигурации   
a.	Подключитесь к коммутатору с помощью консольного подключения и активируйте привилегированный режим EXEC.   
b.	Войдите в режим конфигурации.    
c.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.   
d.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.   
e.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.   
f.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.    
g.	Зашифруйте открытые пароли.   
h.	Создайте баннер, который предупреждает о запрете несанкционированного доступа.   
i.	Настройте и активируйте на коммутаторе интерфейс VLAN 1, используя информацию, приведенную в таблице адресации.   
j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.   
**РЕЗУЛЬТАТ:**     
*Switch#conf t*  
*Switch(config)#hostname S1*   
*S1(config)#no ip domain-lookup*   
*S1(config)#service password-encryption*   
*S1(config)#enable secret class*   
*S1(config)#banner motd #*  
*Enter TEXT message. End with the character '#'.*
*Unauthorized access is strictly prohibited. #*
*S1(config)#interface vlan 1*   
*S1(config-if)#ip address 192.168.1.11 255.255.255.0*   
*S1(config-if)#no shutdown*  
*S1(config-if)#*   
*%LINK-5-CHANGED: Interface Vlan1, changed state to up*   
*S1(config-if)#ip default-gateway 192.168.1.1*    
*S1(config)#line con 0*   
*S1(config-line)#logging synchronous*  
*S1(config-line)#password cisco*   
*S1(config-line)#login*   
*S1(config-line)#exit*    
*S1(config)#exit*   
*S1#copy running-config startup-config*   
*Destination filename [startup-config]?*  
*Building configuration...*   
*[OK]*   
*S1#*   

### Шаг 2. Настройте коммутатор для соединения по протоколу SSH.   
Для настройки протокола SSH на коммутаторе используйте те же команды, которые применялись для аналогичной настройки маршрутизатора в части 2.   
a.	Настройте имя устройства, как указано в таблице адресации.   
b.	Задайте домен для устройства.    
c.	Создайте ключ шифрования с указанием его длины.   
d.	Создайте имя пользователя в локальной базе учетных записей.
e.	Активируйте протоколы Telnet и SSH на линиях VTY.   
f.	Измените способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.   
**РЕЗУЛЬТАТ:**       
*S1#conf t*   
*S1(config)#ip domain-name otus.ru*   
*S1(config)#crypto key generate rsa*   
*The name for the keys will be: S1.otus.ru*   
*Choose the size of the key modulus in the range of 360 to 2048 for your*   
*General Purpose Keys. Choosing a key modulus greater than 512 may take*   
*a few minutes.*   
*How many bits in the modulus [512]:*   
*% Generating 512 bit RSA keys, keys will be non-exportable...[OK]*     
*S1(config)#do sh ip ssh*   
**Mar 1 1:17:24.818: RSA key size needs to be at least 768 bits for ssh version 2*   
**Mar 1 1:17:24.818: %SSH-5-ENABLED: SSH 1.5 has been enabled*   
*SSH Enabled - version 1.5*   
*Authentication timeout: 120 secs; Authentication retries: 3*   
*S1(config)#ip ssh version 2*   
*Please create RSA keys (of at least 768 bits size) to enable SSH v2.*  
*S1(config)#username admin privilege 15 secret Adm1nP @55*   
*S1(config)#line vty 0 4*   
*S1(config-line)#login local*   
*S1(config-line)#transport input ssh*   

### Шаг 3. Установите соединение с коммутатором по протоколу SSH.   
Запустите программу Tera Term на PC-A, затем установите подключение по протоколу SSH к интерфейсу SVI коммутатора S1.   
**Вопрос: Удалось ли вам установить SSH-соединение с коммутатором?**   
![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.5/JPG/06.png)     
## Часть 4. Настройка протокола SSH с использованием интерфейса командной строки (CLI) коммутатора    
a.	Чтобы подключиться к маршрутизатору R1 по протоколу SSH, введите команду –l admin. Это позволит вам войти в систему под именем admin. При появлении приглашения введите в качестве пароля Adm1nP @55   
S1# ssh -l admin 192.168.1.1   
Password:    
Authorized Users Only!   
R1>   
b.	Чтобы вернуться к коммутатору S1, не закрывая сеанс SSH с маршрутизатором R1, нажмите комбинацию клавиш Ctrl+Shift+6. Отпустите клавиши Ctrl+Shift+6 и нажмите x. Отображается приглашение   привилегированного режима EXEC коммутатора.  
R1>    
S1#   
c.	Чтобы вернуться к сеансу SSH на R1, нажмите клавишу Enter в пустой строке интерфейса командной строки. Чтобы увидеть окно командной строки маршрутизатора, нажмите клавишу Enter еще раз.   
S1#   
[Resuming connection 1 to 192.168.1.1 ... ]   
R1>   
d.	Чтобы завершить сеанс SSH на маршрутизаторе R1, введите в командной строке маршрутизатора команду exit.   
R1# exit   
[Connection to 192.168.1.1 closed by foreign host]   
S1#     
**РЕЗУЛЬТАТ:**    

![](https://github.com/Makentosh1600/studying_at_otus-/blob/main/Lab%201.5/JPG/07.png)     

**Вопрос1. Какие версии протокола SSH поддерживаются при использовании интерфейса командной строки?**   
*- Версия 1.хх и 2*
## Вопрос для повторения   
**Как предоставить доступ к сетевому устройству нескольким пользователям, у каждого из которых есть собственное имя пользователя?**   
*-	Необходимо настроить аутентификацию пользователей на устройстве командой «username»*
 





