+++ 
draft = true
date = 2021-01-31T22:47:02+06:00
title = "Бесплатный VPN сервис Wireguard на AWS"
description = "Инструкция по созданию бесплатного на 1 год VPN сервиса Wireguard на базе AWS"
tags = ["vpn", "wireguard", "aws", "aws ec2"]
categories = ["Информационная безопасность", "Cетевые технологии", "VPN"]
series = ["vpn", "wireguard", "aws"]
+++

## Для чего?

С ростом цензурирования интернета авторитарными режимами, блокируются все большее количество полезных интернет ресурсов и сайтов. В том числе с технической информацией.
Таким образом, становится невозможно полноценно пользоваться интернетом и нарушается фундаментальное право на свободу слова, закрепленное во [Всеобщей декларации прав человека](https://www.un.org/ru/documents/decl_conv/declarations/declhr.shtml).
> Статья 19
Каждый человек имеет право на свободу убеждений и на свободное выражение их; это право включает свободу беспрепятственно придерживаться своих убеждений и свободу искать, получать и распространять информацию и идеи любыми средствами и независимо от государственных границ

В данном руководстве мы за 6 этапов развернем свой собственный бесплатный* [VPN сервис](https://ru.bmstu.wiki/VPN_(Virtual_Private_Network)) на базе технологии [Wireguard](https://www.wireguard.com/), в облачной инфраструктуре [Amazon Web Services](https://aws.amazon.com/ru/) (AWS), с помощью бесплатного аккаунта (на 12 месяцев), на инстансе (виртуальной машине) под управлением [Ubuntu Server 18.04 LTS](https://www.ubuntu.com/server).

Я старался сделать это пошаговое руководство как можно более дружественным к людям, далеким от ИТ. Единственное что требуется - это усидчивость в повторении описанных ниже шагов.   


> Примечание
* AWS предоставляет [бесплатный уровень использования](https://aws.amazon.com/ru/free/faqs/) сроком на 12 месяцев, с ограничением на 15 гигабайт трафика в месяц.

## Этапы 
1. Регистрация бесплатного аккаунта AWS
2. Создание инстанса AWS
3. Подключение к инстансу AWS
4. Конфигурирование Wireguard
5. Конфигурирование VPN клиентов
6. Проверка корректности установки VPN

## Полезные ссылки
* [Скрипты автоматической установки Wireguard на AWS](https://github.com/pprometey/wireguard_aws)
* [Дискуссия на Habr.com (RU)](https://habr.com/ru/post/448528/#comments)
* [Дискуссия на Habr.com (EN)](https://habr.com/en/post/449234/#comments)

# 1. Регистрация аккаунта AWS

Для регистрации бесплатного аккунта AWS требуется реальный номер телефона и платежеспособная кредитная карта Visa или Mastercard. Рекомендую воспользоваться виртуальными картами которые бесплатно предоставляет [Яндекс.Деньги](https://money.yandex.ru/cards/virtual) или [Qiwi кошелек](https://qiwi.com/cards/qvc). Для проверки валидности карты, при регистрации списывается 1$ который в дальнейшем возвращается. 

## 1.1. Открытие консоли управления AWS
Необходимо открыть браузер и перейти по адресу: [https://aws.amazon.com/ru/](https://aws.amazon.com/ru/)
Нажать на кнопку "Регистрация"

![Стартовая страница AWS Amazon](/images/wireguard-aws/ru/register1.jpg)

## 1.2. Заполнение персональных данных
Заполнить данные и нажать на кнопку "Продолжить"

![Регистрации аккаунта AWS Amazon](/images/wireguard-aws/ru/register2.jpg)

## 1.3. Заполнение контактных данных
Заполнить контактные сведения. 

![Указание контактных данных при регистрации аккаунта AWS Amazon](/images/wireguard-aws/ru/register3.jpg)

## 1.4. Указание платежной информации. 
Номер карты, срок окончания и имя держателя карты. 

![Указание платежных данных при регистрации аккаунта AWS Amazon](/images/wireguard-aws/ru/register4.jpg)

## 1.5. Подтверждение аккаунта
На этом этапе идет подтверждение номера телефона и непосредственное списание 1$ с платежной карты. На экране компьютера отображается 4х значный код, и на указанный телефон поступает звонок из Amazon. Во время звонка необходимо набрать код, указанный на экране. 

![Подтверждение аккаунта AWS Amazon](/images/wireguard-aws/ru/register5.jpg)

## 1.6. Выбор тарифного плана. 
Выбираем - Базовый план (бесплатный)

![Выбор тарифного плана AWS Amazon](/images/wireguard-aws/ru/register6.jpg)

## 1.7. Вход в консоль управления

![Вход в консоль управления AWS Amazon](/images/wireguard-aws/ru/register7.jpg)

## 1.8. Выбор расположения дата-центра

![Консоль управления AWS Amazon](/images/wireguard-aws/ru/console1.jpg)

### 1.8.1. Тестирование скорости
Прежде чем выбирать датацентр, рекомендуется протестировать через [https://speedtest.net](https://speedtest.net) скорость доступа к ближайшим датацентрам, в моей локации такие результаты:

- Сингапур
![Сингапур](/images/wireguard-aws/ru/st_singapore.jpg)
- Париж
![Париж](/images/wireguard-aws/ru/st_paris.jpg)
- Франкфурт
![Франкфурт](/images/wireguard-aws/ru/st_frankfurt.jpg)
- Стокгольм
![Стокгольм](/images/wireguard-aws/ru/st_stockholm.jpg)
- Лондон
![Лондон](/images/wireguard-aws/ru/st_london.jpg)

Лучшие результаты по скорости показывает датацентр в Лондоне. Поэтому я выбрал его для дальнейшей настройки.

# 2. Создание инстанса AWS

## 2.1 Создание виртуальной машины (инстанса)

### 2.1.0. Запуск пошагового мастера создания инстанса

#### 2.1.0.1. Переход на страницу запуска инстанса 

![Переход на страницу запуска инстанса](/images/wireguard-aws/ru/instance0.jpg)

#### 2.1.0.2. Запуск пошагового мастера создания инстанса

![Запуск пошагового мастера создания инстанса](/images/wireguard-aws/ru/instance0_1.jpg)

#### 2.1.0.3. Выбор типа операционной стистемы инстанса

![Выбор типа операционной стистемы](/images/wireguard-aws/ru/instance0_2.jpg)

### 2.1.1. Выбор типа инстанса
По умолчанию выбран инстанс t2.micro, он нам и нужен, просто нажимаем кнопку **Next: Configure Instance Detalis**

![Выбор типа инстанса](/images/wireguard-aws/ru/instance1.jpg)

### 2.1.2. Настройка параметров инстанса
В дальнейшем мы подключим к нашему инстансу постоянный публичный IP, поэтому на этом этапе мы отключаем автоназначение публичного IP, и нажимаем кнопку **Next: Add Storage**

![Настройка параметров инстанса](/images/wireguard-aws/ru/instance2.jpg)

### 2.1.3. Подключение хранилища
Указываем размер "жесткого диска". Для наших целей достаточно 16 гигабайт, и нажимаем кнопку **Next: Add Tags**

![Подключение хранилища](/images/wireguard-aws/ru/instance3.jpg)

### 2.1.4. Настройка тегов
Если бы мы создавали несколько инстансов, то их можно было бы группировать по тегам, для облегчения администрирования. В данном случае эта фукнциональность излишняя, сразу нажимаем кнопку **Next: Configure Security Gorup**

![Настройка тегов](/images/wireguard-aws/ru/instance4.jpg)

### 2.1.5. Открытие портов
На этом этапе мы настраиваем брандмауэр, открывая нужные порты. Набор открытых портов называется "Группа безопасности" (Security Group). Мы должны создать новую группу безопасности, дать ей имя, описание, добавить порт UDP (Custom UDP Rule), в поле Rort Range необходимо назначить номер порта, из диапазона [динамических портов](https://ru.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D0%BF%D0%BE%D1%80%D1%82%D0%BE%D0%B2_TCP_%D0%B8_UDP) 49152—65535. В данном случае я выбрал номер порта 54321.

![Открытие портов](/images/wireguard-aws/ru/instance5.jpg)

После заполнения необходимых данных, нажимаем на кнопку **Review and Launch**

### 2.1.6. Обзор всех настроек инстанса
На данной странице идет обзор всех настроек нашего инстанса, проверяем все ли настройки в порядке, и нажимаем кнопку **Launch**

![Обзор всех настроек инстанса](/images/wireguard-aws/ru/instance6.jpg)

### 2.1.7. Создание ключей доступа
Дальше выходит диалоговое окно, предлагающее либо создать, либо добавить существующий SSH ключ, с помощью которого мы в дальнейшем будет удаленно подключатся к нашему инстансу. Мы выбираем опцию "Create a new key pair" чтобы создать новый ключ. Задаем его имя, и нажимаем кнопку **Download Key Pair**, чтобы скачать созданные ключи. Сохраните их в надежное место на диске локального компьютера. После того как скачали - нажимаете кнопку **Launch Instances**

![Создание ключей доступа](/images/wireguard-aws/ru/instance7.jpg)

### 2.1.7.1. Сохранение ключей доступа
Здесь показан, этап сохранения созданных ключей из предыдущего шага. После того, как мы нажали кнопку **Download Key Pair**, ключ сохраняется в виде файла сертификата с расширением *.pem. В данном случае я дал ему имя ***wireguard-awskey.pem***

![Сохранение ключей доступа](/images/wireguard-aws/ru/instance8.jpg)

### 2.1.8. Обзор результатов создания инстанса
Далее мы видим сообщение об успешном запуске только что созданного нами инстанса. Мы можем перейти к списку наших инстансов нажав на кнопку **View instances**

![Обзор результатов создания инстанса](/images/wireguard-aws/ru/instance9.jpg)

## 2.2. Создание внешнего IP адреса

### 2.2.1. Запуск создания внешнего IP
Дальше нам необходимо создать постоянный внешний IP адрес, через который мы и будем подключатся к нашему VPN серверу. Для этого в навигационной панели в левой части экрана необходимо выбрать пункт **Elastic IPs** из категории ***NETWORK & SECTURITY*** и нажать кнопку **Allocate new address**

![Запуск создания внешнего IP](/images/wireguard-aws/ru/elasticip1.jpg)

### 2.2.2. Настройка создания внешнего IP
На следующем шаге нам необходима чтобы была включена опция ***Amazon pool*** (включена по умолчанию), и нажимаем на кнопку **Allocate**

![Настройка создания внешнего IP](/images/wireguard-aws/ru/elasticip2.jpg)

### 2.2.3. Обзор результатов создания внешнего IP адреса
На следующем экране отобразится полученный нами внешний IP адрес. Рекомендуется его запомнить, а лучше даже записать. он нам еще не раз пригодиться в процессе дальнейшей настройки и использования VPN сервера. В данном руководстве в качестве примера я использую IP адрес ***4.3.2.1***. Как записали адрес, нажимаем на кнопку **Close**

![Обзор результатов создания внешнего IP адреса](/images/wireguard-aws/ru/elasticip3.jpg)

### 2.2.4. Список внешних IP адресов
Далее нам открывается список наших постоянных публичных IP адресов (elastics IP).

![Список внешних IP адресов](/images/wireguard-aws/ru/elasticip4.jpg)

### 2.2.5. Назначение внешнего IP инстансу
В этом списке мы выбираем полученный нами IP адрес,  и нажимаем правую кнопку мыши, чтобы вызвать выпадающее меню. В нем выбираем пункт **Associate address**, чтобы назначить его ранее созданному нами инстансу. 

![Назначение внешнего IP инстансу](/images/wireguard-aws/ru/elasticip5.jpg)

### 2.2.6. Настройка назначения внешнего IP
На следующем шаге выбираем из выпадающего списка наш инстанс, и нажимаем кнопку **Associate**

![Настройка назначения внешнего IP](/images/wireguard-aws/ru/elasticip6.jpg)

### 2.2.7. Обзор результатов назначения внешнего IP 
После этого, мы можем увидеть, к нашему постоянному публичному IP адресу привязан наш инстанс и его приватный IP адрес. 

![Обзор результатов назначения внешнего IP](/images/wireguard-aws/ru/elasticip7.jpg)

Теперь мы можем подключиться к нашему вновь созданному инстансу из вне, со своего компьютера по SSH.

# 3. Подключение к инстансу AWS

[SSH](https://ru.wikipedia.org/wiki/SSH) - это безопасный протокол удаленного управления компьютерными устройствами. 

## 3.1. Подключение по SSH c компьютера на Windows

Для подключения к компьютера с Windows, прежде необходимо скачать и установить программу [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). 

### 3.1.1. Импорт приватного ключа для Putty
3.1.1.1. После установки Putty, необходимо запустить утилиту PuTTYgen идущую с ней в комплекте, для импорта ключа сертификата в формате PEM, в формат, пригодный для использования в программе Putty. Для этого в верхнем меню выбираем пункт **Conversions->Import Key**

![Импорт приватного ключа для Putty](/images/wireguard-aws/ru/ssh1.jpg)

#### 3.1.1.2. Выбор ключа AWS в формате PEM
Далее, выбираем ключ, который мы ранее сохранили на этапе 2.1.7.1, в нашем случае его имя ***wireguard-awskey.pem***

![Выбор ключа AWS в формате PEM](/images/wireguard-aws/ru/ssh2.jpg)

#### 3.1.1.3. Задание параметров импорта ключа
На этом шаге нам необходимо указать комментарий для этого ключа (описание) и задать для безопасности пароль и его подтверждение. Он будет запрашиваться при каждом подключении. Таким образом мы защищаем ключ паролем от не целевого использования. Пароль можно не задавать, но это менее безопасно, в случае, если ключ попадет в чужие руки. После нажимаем кнопку **Save private key**

![Задание параметров импорта ключа](/images/wireguard-aws/ru/ssh3.jpg)

#### 3.1.1.4. Сохранение импортированного ключа
Открывается диалоговое окно сохранения файла, и мы сохраняем наш приватный ключ в виде файла с расширением `.ppk`, пригодного для использования в программе ***Putty***.
Указываем имя ключа (в нашем случае `wireguard-awskey.ppk`) и нажимаем кнопку **Сохранить**. 

![Сохранение импортированного ключа](/images/wireguard-aws/ru/ssh4.jpg)

### 3.1.2. Создание и настройка соединения в Putty

#### 3.1.2.1. Создание соединения
Открываем программу Putty, выбираем категорию **Session** (она открыта по умолчанию) и в поле **Host Name** вводим публичный IP адрес нашего сервера, который мы получили на шаге 2.2.3. В поле **Saved Session** вводим произвольное название нашего соединения (в моем случае *wireguard-aws-london*), и далее нажимаем кнопку **Save** чтобы сохранить сделанные нами изменения. 

![Создание соединения](/images/wireguard-aws/ru/ssh5.jpg)

#### 3.1.2.2. Настройка автологина пользователя
Дальше в категории ***Connection***, выбираем подкатегорию ***Data*** и в поле **Auto-login username** водим имя пользователя **ubuntu** - это стандартный пользователь инстанса на AWS с Ubuntu. 

![Настройка автологина пользователя](/images/wireguard-aws/ru/ssh6.jpg)

#### 3.1.2.3. Выбор приватного ключа для соединения по SSH
Затем переходим в подкатегорию ***Connection/SSH/Auth*** и рядом с полем **Private key file for authentication** нажимаем на кнопку **Browse...** для выбора файла с сертификатом ключа. 

![Выбор приватного ключа для соединения по SSH](/images/wireguard-aws/ru/ssh7.jpg)

#### 3.1.2.4. Открытие импортированного ключа
Указываем ключ, импортированный нами ранее на этапе 3.1.1.4, в нашем случае это файл *wireguard-awskey.ppk*, и нажимаем кнопку **Открыть**.

![Открытие импортированного ключа](/images/wireguard-aws/ru/ssh8.jpg)


#### 3.1.2.5. Сохранение настроек и запуск подключения
Вернувшись на страницу категории ***Session*** нажимаем еще раз кнопку **Save**, для сохранения сделанных ранее нами изменений на предыдущих шагах (3.1.2.2 - 3.1.2.4). И затем нажимаем кнопку **Open** чтобы открыть созданное и настроенное нами удаленное подключение по SSH. 

![Сохранение настроек и запуск подключения](/images/wireguard-aws/ru/ssh10.jpg)

#### 3.1.2.7. Настройка доверия между хостами
На следующем шаге, при первой попытке подключиться, нам выдается предупреждение, у нас не настроено доверие между двумя компьютерами, и спрашивает, доверять ли удаленному компьютеру. Мы нажимем кнопку **Да**, тем самым добавляя его в список доверенных хостов. 

![Настройка доверия между хостами](/images/wireguard-aws/ru/ssh11.jpg)

#### 3.1.2.8. Ввод пароля для доступа к ключу
После этого открывается окно терминала, где запрашивается пароль к ключу, если вы его устанавливали ранее на шаге 3.1.1.3. При вводе пароля никаких действий на экране не происходит. Если ошиблись, можете использовать клавишу *Backspace*.

![Ввод пароля для доступа к ключу](/images/wireguard-aws/ru/ssh12.jpg)

#### 3.1.2.9. Приветственное сообщение об успешном подключении
После успешного ввода пароля, нам отображается в терминале текст приветствия, который сообщает что удаленная система готова к выполнению наших команд.

![Приветственное сообщение об успешном подключении](/images/wireguard-aws/ru/ssh13.jpg)

# 4. Конфигурирование сервера Wireguard

Наиболее актуальную инструкцию по установке и использованию Wireguard с помощью описанных ниже скриптов можно посмотреть в репозитории: [https://github.com/pprometey/wireguard_aws](https://github.com/pprometey/wireguard_aws)

## 4.1. Установка Wireguard
В терминале вводим следующие команды (можно копировать в буфер обмена, и вставлять в терминале нажатием правой клавиши мыши):

### 4.1.1. Клонирование репозитория
Клонируем репозиторий со скриптами установки Wireguard
```
git clone https://github.com/pprometey/wireguard_aws.git wireguard_aws
```
### 4.1.2. Переход в каталог со скриптами
Переходим в каталог с клонированным репозиторем 
```
cd wireguard_aws
```

### 4.1.3  Запуск скрипта инициализации 
Запускаем от имени администратора (root пользователя) скрипт установки Wireguard
```
sudo ./initial.sh
```
В процессе установки будут запрошены определенные данные, необходимые для настройки Wireguard

### 4.1.3.1. Ввод точки подключения
Введите внешний IP адрес и открытый порт Wireguard сервера. Внешний IP адрес сервера мы получили на шаге 2.2.3, а порт открыли на шаге 2.1.5. Указываем их слитно, разделяя двоеточием, например `4.3.2.1:54321`, и после этого нажимает клавишу ***Enter***  
*Пример вывода:*
```
Enter the endpoint (external ip and port) in format [ipv4:port] (e.g. 4.3.2.1:54321): 4.3.2.1:54321
```

### 4.1.3.2. Ввод внутреннего IP адреса
Введите IP адрес сервера Wireguard в защищенной VPN подсети, если не знаете что это такое, просто нажмите клавишу Enter для установки значения по умолчанию (`10.50.0.1`)  
*Пример вывода:*
```
Enter the server address in the VPN subnet (CIDR format) ([ENTER] set to default: 10.50.0.1):
```

### 4.1.3.3. Указание сервера DNS
Введите IP адрес DNS сервера, или просто нажмите клавишу Enter для установки значения по умолчанию `1.1.1.1` (Cloudflare public DNS)  
*Пример вывода:*
```
Enter the ip address of the server DNS (CIDR format) ([ENTER] set to default: 1.1.1.1):
```

### 4.1.3.4. Указание WAN интерфейса
Дальше требуется ввести имя внешнего сетевого интерфейса, который будет прослушивать внутренний сетевой интерфейс VPN. Просто нажмите Enter, чтобы установить значение по умолчанию для AWS (`eth0`)  
*Пример вывода:*
```
Enter the name of the WAN network interface ([ENTER] set to default: eth0):
```

### 4.1.3.5. Указание имени клиента
Введите имя VPN пользователя. Дело в том, что VPN сервер Wireguard не сможет запуститься, пока не добавлен хотя бы один клиент. В данном случае я ввел имя `Alex@mobile`  
*Пример вывода:*
```
Enter VPN user name: Alex@mobile
```
После этого на экране должен отобразится QR код с конфигурацией только что добавленного клиента, который надо считать с помощью мобильного клиента Wireguard на Android либо iOS, для его настройки. А также ниже QR кода отобразится текст конфигурационного файла в случае ручной конфигурации клиентов. Как это сделать будет сказано ниже. 

![Завершение установки Wireguard](/images/wireguard-aws/ru/install1.jpg)

## 4.2. Добавление нового VPN пользователя

Чтобы добавить нового пользователя, необходимо в терминале выполнить скрипт `add-client.sh` 
```
sudo ./add-client.sh
```
Скрипт запрашивает имя пользователя:  
*Пример вывода:*
```
Enter VPN user name: 
```
Также, имя пользователям можно передать в качестве параметра скрипта (в данном случае `Alex@mobile`):
```
sudo ./add-client.sh Alex@mobile
```
В результате выполнения скрипта, в каталоге с именем клиента по пути `/etc/wireguard/clients/{ИмяКлиента}` будет создан файл с конфигурацией клиента `/etc/wireguard/clients/{ИмяКлиента}/{ИмяКлиента}.conf`, а на экране терминала отобразится QR код для настройки мобильных клиентов и содержимое файла конфигурации.

### 4.2.1. Файл пользовательской конфигурации
Показать на экране содержимое файла .conf, для ручной настройки клиента, можно с помощью команды `cat`
```
sudo cat /etc/wireguard/clients/Alex@mobile/Alex@mobile.conf
```
результат выполнения:
```
[Interface]
PrivateKey = oDMWr0toPVCvgKt5oncLLRfHRit+jbzT5cshNUi8zlM=
Address = 10.50.0.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = mLnd+mul15U0EP6jCH5MRhIAjsfKYuIU/j5ml8Z2SEk=
PresharedKey = wjXdcf8CG29Scmnl5D97N46PhVn1jecioaXjdvrEkAc=
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = 4.3.2.1:54321
```
Описание файла конфигурации клиента:
```
[Interface]
PrivateKey = Приватный ключ клиента
Address = IP адрес клиента
DNS = ДНС используемый клиентом

[Peer]
PublicKey = Публичный ключ сервера
PresharedKey = Общи ключ сервера и клиента
AllowedIPs = Разрешенные адреса для подключения (все -  0.0.0.0/0, ::/0)
Endpoint = IP адрес и порт для подключения
```

### 4.2.2. QR код конфигурации клиента
Показать на экране терминала QR код конфигурации для ранее созданного клиента можно с помощью команды `qrencode -t ansiutf8` (в данном примере используется клиент с именем Alex@mobile):
```
sudo cat /etc/wireguard/clients/Alex@mobile/Alex@mobile.conf | qrencode -t ansiutf8
```

# 5. Конфигурирование VPN клиентов

## 5.1. Настройка мобильного клиента Андроид
Официальный клиент Wireguard для Андроид можно [установить из официального магазина GooglePlay](https://play.google.com/store/apps/details?id=com.wireguard.android)

После чего, необходимо импортировать конфигурацию, считав QR код с конфигурацией клиента (см. пункт 4.2.2) и дать ему имя:

![Настройка Андроид клиента Wireguard](/images/wireguard-aws/ru/android1.jpg)

После успешного импорта конфигурации, можно включить VPN тоннель. Об успешном подключении скажет заначок ключика в системной панели Андроид

![Работающий Андроид клиент Wireguard](/images/wireguard-aws/ru/android2.jpg)

## 5.2. Настройка клиента Windows
Первоначально необходимо скачать и установить программу [TunSafe for Windows](https://tunsafe.com/download) - это клиент Wireguard для Windows.

### 5.2.1. Создание файла конфигурации для импорта
Правой кнопкой мышки создаем текстовый файл на рабочем столе. 

![Создание текстового файла](/images/wireguard-aws/ru/windows1.jpg)

### 5.2.2. Копирование содержимого файла конфигурации с сервера
Дальше возвращаемся к терминалу Putty и отображаем содержимое конфигурационного файла нужного пользователя, как это описано на шаге 4.2.1.   
Далее выделяем правой кнопкой мыши текст конфигурации в терминале Putty, по окончании выделения он автоматически скопируется в буфер обмена. 

![Копирование текста с конфигурацией](/images/wireguard-aws/ru/windows2.jpg)

### 5.2.3. Копирование конфигурации в локальный файл конфигурации
Поле этого возвращаемся к созданному нами ранее на рабочем столе текстовому файлу, и вставляем в него из буфера обмена текст конфигурации. 

![Копирование текста с конфигурацией](/images/wireguard-aws/ru/windows3.jpg)

### 5.2.4. Сохранение локального файла конфигурации
Сохраняем файл, с расширением **.conf** (в данном случае с именем `london.conf`)

![Сохранения файла с конфигурацией](/images/wireguard-aws/ru/windows4.jpg)

### 5.2.5. Импорт локального файла конфигурации
Далее необходимо импортировать файл конфигурации в программу TunSafe.

![Импорт файла конфигурации в TunSafe](/images/wireguard-aws/ru/windows5.jpg)

### 5.2.6. Установка VPN соединения
Выбрать этот файл конфигурации и подключиться, нажав кнопку **Connect**.
![Подключение к серверу VPN через TunSafe](/images/wireguard-aws/ru/windows6.jpg)

# 6. Проверка успешности подключения 

Чтобы проверить успешность подключения через VPN тоннель, необходимо открыть браузер и перейти на сайт [https://2ip.ua/ru/](https://2ip.ua/ru/)

![Подключение к серверу VPN через TunSafe](/images/wireguard-aws/ru/check1.jpg)

Отображаемый IP адрес должен совпадать с тем, который мы получили на этапе 2.2.3.
Если это так, значит VPN тоннель работает успешно. 

Из терминала в Linux можно проверить свой IP адрес, введя команду: 
```
curl http://zx2c4.com/ip
```

Или можно просто зайти на порнохаб, если вы находитесь в Казахстане.