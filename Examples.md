# **API MikroBILL**

## **Примеры скриптов**

### **Пример 1. Выводим имена всех клиентов на выбранном тарифе**
>
> // Example script №1
>
> // Скрипт совместим с версией MikroBILL 1.5.1 и выше
>
> **var** UsersCount // Количество клиентов на выбранном тарифе
>
> **var** c // счётчик цикла
>
> UsersCount = *GetCountClientsInTarif*(**"testys"**) // Получаем кол-во
> юзеров на тарифе
>
> c = 0
>
> loop1: // начало цикла
>
> *alert*(*GetClientFromTarif*(**"testys"**,c))
>
> c=c+1
>
> **if** (c &lt; UsersCount) **then** **goto** loop1
>
### **Пример 2. Пополняем баланс всем клиентам на выбранном тарифе**
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> //\*\* Доступ в интернет для клиентов, чей баланс \*\*\*
>
> //\*\* выше заданного значения. \*\*\*
>
> //\*\*\* Скрипт совместим с версией MikroBILL 1.7.1 и выше. \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> **var** UsersCount // Количество клиентов на выбранном тарифе
>
> **var** Money // Сколько будем добавлять
>
> **var** Tarif // С каким тарифом будем работать
>
> **var** Clients\[0\] // Массив с клиентами
>
> **var** c // Счётчик
>
> Money = inputbox(**"Сколько денег вы хотите положить
> клиентам?"**,**""**,**"0"**)
>
> Tarif = inputbox(**"Клиентам из какого тарифа будем класть
> деньги?"**,**""**,**"test2"**)
>
> **if** **not** isnumeric(money) **then**
>
> *alert*(**"Вы ввели некорректное число!"**)
>
> **else**
>
> c = 0
>
> UsersCount = *GetCountClientsInTarif*(Tarif) // Получаем кол-во юзеров
> на тарифе
>
> **if** UsersCount = 0 **then**
>
> *alert*(**"Не найдено клиентов на указанном тарифе!"** & vbNewLine &
> **"Возможно, вы ошиблись с написанием тарифа."**)
>
> **end**
>
> **end** **if**
>
> *GetClientsFromTarif*(Tarif, Clients)
>
> loop: // начало цикла
>
> *addmoney*(Money, **"Пополнение баланса!"** , Clients\[c\])
>
> c=c+1
>
> **if** (c &lt; UsersCount) **then** **goto** loop
>
> *alert*(**"Баланс успешно пополнен у "** & UsersCount & **"
> клиентов!"**)
>
> **end** **if**
>
> **  
### Пример 3. Сбрасываем трафик 1 числа**
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> //\*\*\* Сбрасываем наработанный трафик заданного числа \*\*\*
>
> //\*\*\* Необходимо поставить запуск скрипта в планировщик \*\*\*
>
> //\*\*\* MikroBILL на нужное число, время запуска - 0 часов,\*\*\*
>
> //\*\*\* интервал запуска 60 мин. \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> **var** c // Счётчик цикла
>
> **var** UsersCount // Количество клиентов
>
> UsersCount = *GetCountClients*()
>
> c = 0
>
> loop1: // начало цикла
>
> *ResetMonthTraffic*(*GetClient*(c))
>
> c=c+1
>
> **if** (c &lt; UsersCount) **then** **goto** loop1
>
### **Пример 4. Включаем Интернет только по выходным дням**
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> //\*\*\* Доступ в Интернет только по выходным дням \*\*\*
>
> //\*\*\* Скрипт совместим с версией MikroBILL 1.7.1 и выше. \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> **var** TarifName // Название тарифа, который будем обрабатывать
>
> **var** Clients\[0\] // Массив с клиентами
>
> **var** StartOrStop // Будем останавливать, или запускать клиентов
>
> **var** c // Счётчик цикла
>
> // Если сегодня выходной
>
> // PS: В американском календаре Воскр.=1, суббота = 7
>
> **if** ((Weekday(date) = 1) **or** (Weekday(date) = 7)) **then**
>
> StartOrStop = **true**
>
> **else**
>
> StartOrStop = **false**
>
> **end** **if**
>
> // Тариф, с которым работаем
>
> TarifName = **"testys"**
>
> *GetClientsFromTarif*(TarifName, Clients)
>
> c = 0
>
> loop: // начало цикла
>
> **if** (StartOrStop = **true**) **then**
>
> *StartClients*(Clients\[c\])
>
> **else**
>
> *StopClients*(Clients\[c\])
>
> **end** **if**
>
> c=c+1
>
> **if** (c &lt; *GetArraySize*(Clients)) **then** **goto** loop
>
### **Пример 5. Снижение скорости в зависимости от нагрузки**
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> //\*\* При превышении нагрузки на канал временно \*\*\*
>
> //\*\* снижаем скорость самым качающим абонентам \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> // Скрипт совместим с версией MikroBILL 1.5.4 и выше
>
> **var** TarifName // Название тарифа, который будем обрабатывать
>
> **var** LineRx // Ширина входящего канала, кбит/сек.
>
> **var** LineTx // Ширина исходящего канала, кбит/сек.
>
> **var** LoadThreshold // Нагрузка в % после которой срабатывает
> ограничение
>
> **var** LoadThreshold2 // Нагрузка в % после которой ограничения
> вернуться к предыдущему значению
>
> **var** UsersCount // Количество клиентов на тарифе
>
> **var** CurrUser // Текущий клиент
>
> **var** CurrLoadRx // Текущая входящая скорость, кбит/сек.
>
> **var** CurrLoadTx // Текущая исходящая скорость, кбит/сек.
>
> **var** BaseLimitRx // Ограничение входящей скорости для клиентов,
> кбит/сек.
>
> **var** BaseLimitTx // Ограничение исходящей скорости для клиентов,
> кбит/сек.
>
> **var** c // Счётчик цикла
>
> **var** UserIndex // Номер клиента
>
> // Название тарифа
>
> TarifName = **"Test"**
>
> // Ширина канала в Интернет (Кбит/сек.)
>
> LineRx = 9000
>
> LineTx = 4000
>
> // Рекомендуемые ограничения скорости для клиентов (Кбит/сек.)
>
> BaseLimitRx = 5000
>
> BaseLimitTx = 2000
>
> // Процент загрузки канала, после которой сработают ограничения
>
> LoadThreshold = 85
>
> // Процент загрузки канала, после которой ограничения будут возвращены
>
> LoadThreshold2 = 35
>
> CurrLoadRx = 0
>
> CurrLoadTx = 0
>
> UsersCount = *GetCountClientsInTarif*(TarifName)
>
> // Считаем нагрузку на канал
>
> c = 0
>
> loop1: // начало цикла
>
> CurrUser = *GetClientFromTarif*( TarifName , c)
>
> CurrLoadRx = CurrLoadRx + *GetRxSpeed*(CurrUser)
>
> CurrLoadTx = CurrLoadTx + *GetTxSpeed*(CurrUser)
>
> c=c+1
>
> **if** (c &lt; UsersCount) **then** **goto** loop1
>
> // Определяем % загрузки канала
>
> CurrLoadRx = CurrLoadRx / (LineRx / 100)
>
> CurrLoadTx = CurrLoadTx / (LineTx / 100)
>
> **var** RxOverload
>
> **var** TxOverload
>
> RxOverload = **false**
>
> TxOverload = **false**
>
> // Проверяем, превысила ли нагрузка макс. допустимую
>
> **if** (CurrLoadRx &gt; LoadThreshold) **then** RxOverload = **true**
>
> **if** (CurrLoadTx &gt; LoadThreshold) **then** TxOverload = **true**
>
> **if** (len(*ReadFileLine*(**"Overload"**,0)) &gt; 0) **then**
>
> **if** (*ReadFileLine*(**"Overload"**,0) = **true**) **then**
>
> RxOverload = **false**
>
> **if** (CurrLoadRx &gt; LoadThreshold2) **then** RxOverload = **true**
>
> **end** **if**
>
> **end** **if**
>
> **if** (len(*ReadFileLine*(**"Overload"**,1)) &gt; 0) **then**
>
> **if** (*ReadFileLine*(**"Overload"**,1) = **true**) **then**
>
> TxOverload = **false**
>
> **if** (CurrLoadTx &gt; LoadThreshold2) **then** TxOverload = **true**
>
> **end** **if**
>
> **end** **if**
>
> //Запоминаем была ли перегрузка канала
>
> *WriteLineInNewFile*(**"Overload"**,RxOverload)
>
> *AppendLineInFile*(**"Overload"**,TxOverload)
>
> // Уменьшаем ограничения скор. вдвое всем, у кого скорость &gt;
> заданного %
>
> c = 0
>
> loop2: // начало цикла
>
> CurrUser = *GetClientFromTarif*(TarifName,c)
>
> **if** ((*GetRxSpeed*(CurrUser) / BaseLimitRx) \* 100 &gt;
> LoadThreshold2) **then**
>
> **if** (RxOverload) **then**
>
> *SetLimits*(BaseLimitRx / 2, -1, CurrUser)
>
> **else**
>
> *SetLimits*(BaseLimitRx, -1, CurrUser)
>
> **end** **if**
>
> **else**
>
> *SetLimits*(BaseLimitRx, -1, CurrUser)
>
> **end** **if**
>
> **if** ((*GetTxSpeed*(CurrUser) / BaseLimitTx) \* 100 &gt;
> LoadThreshold2) **then**
>
> **if** (TxOverload) **then**
>
> *SetLimits*(-1, BaseLimitTx /2, CurrUser)
>
> **else**
>
> *SetLimits*(-1, BaseLimitTx, CurrUser)
>
> **end** **if**
>
> **else**
>
> *SetLimits*(-1, BaseLimitTx, CurrUser)
>
> **end** **if**
>
> c=c+1
>
> **if** (c &lt; UsersCount) **then** **goto** loop2
>
> **  
### Пример 6. Удаляем неактивных клиентов**
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> //\*\*\* Удаляем клиентов, которые неактивны более N дней \*\*\*
>
> //\*\*\* Скрипт совместим с версией MikroBILL 2.10.0 и выше. \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> **var** UsersCount
>
> **var** MoneyLim
>
> **var** Tarif
>
> **var** c
>
> **var** Clients\[0\]
>
> **var** Days
>
> Days = 6 \* 86400 // Через сколько секунд без активности удалять
> клиентов
>
> Tarif = **"test2"** // Тариф, с которым работаем
>
> UsersCount = *GetCountClientsInTarif*(Tarif)
>
> *GetClientsFromTarif*(Tarif, Clients)
>
> c = *GetCountClientsInTarif*(TarifName) - 1
>
> loop:
>
> // Если не было активности больше 7 дней, удалить клиента
>
> **if** *ToUnixTime*(Now) -
> *ToUnixTime(GetClientLastActivity*(Clients\[c\]))&gt; Days) **then**
>
> *RemoveClients*(Clients\[c\])
>
> **end** **if**
>
> c=c-1
>
> **if** (c &gt; -1) **then** **goto** loop
>
> **  
### Пример 7. Проверяем доступность MikroTik**
>
> // \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> // \*\*\* Проверяем, подключен ли MikroTik к MikroBILL \*\*\*
>
> // \*\*\* если нет, отправляем СМС \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> **var** TikName // Имя подключения
>
> **var** PrevState // Предыдущее состояние подключения
>
> **var** CurState // Текущее состояния подключения
>
> **var** c // Счётчик цикла
>
> **var** TiksCount // Количество клиентов
>
> **var** AdminName // Имя администратора
>
> // Логин пользователя MikroBILL, которому будет отправляться СМС
>
> AdminName = **"Admin"**
>
> TiksCount = *GetCountMikrotiks*()
>
> c = 0
>
> loop1: // начало цикла
>
> TikName = *GetMikroTikName*(c)
>
> PrevState = 0
>
> **if** (len(*ReadFileLine*(TikName,0)) &gt; 0) **then** PrevState =
> *ReadFileLine*(TikName, 0)
>
> CurState = *IsMikroTikConnected*(TikName)
>
> **if** (CurState = **true**) **then**
>
> CurState = 0
>
> **else**
>
> CurState = 1 + PrevState
>
> **end** **if**
>
> **if** (CurState = 5) **then** *SendSMS*(TikName & **" - Отсутствует
> соединение!"**, **"Admin"**)
>
> *WriteLineInNewFile*(TikName,CurState)
>
> c=c+1
>
> **if** (c &lt; TiksCount) **then** **goto** loop1
>
### **Пример 8. Проверяем температуру оборудования MikroTik**
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> //\*\*\* Устанавливаем скрипт в планировщик с интервалом \*\*\*
>
> //\*\*\* запуска 5 мин на все дни недели. \*\*\*
>
> //\*\*\* Скрипт совместим с версией MikroBILL 1.6.4 и выше \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> **var** TelnetConn
>
> **var** Temperature
>
> **var** AlertTemp
>
> AlertTemp = 65 // Температура тревоги
>
> // Создаём новое Telnet подключение
>
> TelnetConn =
> *CreateTelnetConnection*(**"IP"**,23,**"Login"**,**"Password"**)
>
> Temperature = *TelnetSendSync*(TelnetConn,**":put \[system health get
> value-name=temperature\]"**) // Telnet подключение должно быть
> добавлено в MikroBILL перед использованием
>
> **if** (Temperature &gt; AlertTemp) **then**
>
> **if** (*ReadFileLine*(**"overheat"**, 0) = 0) **then**
>
> // Раскомментируйте ниже строку на выбор!
>
> //SendSMS("Danger! Overheat!", "Admin") // Отправляем SMS.
> Пользователь Admin должен существовать с телефоном в профиле.
>
> //alert("Danger! Overheat!") // Отображаем уведомление на ПК с
> MikroBILL
>
> *WriteLineInNewFile*(**"overheat"**,1)
>
> **end** **if**
>
> **end** **if**
>
> **if** (Temperature &lt; (AlertTemp-10)) **then**
>
> *WriteLineInNewFile*(**"overheat"**, 0)
>
> **end** **if**
>
> **  
### Пример 9. Доступ в Интернет для людей с балансом выше заданной
> величины**
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> //\*\* Доступ в интернет для клиентов, чей баланс \*\*\*
>
> //\*\* выше заданного значения. \*\*\*
>
> //\*\*\* Скрипт совместим с версией MikroBILL 1.7.1 и выше. \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> **var** UsersCount
>
> **var** MoneyLim
>
> **var** Tarif
>
> **var** c
>
> **var** Clients\[0\]
>
> Tarif = **"test2"** // Тариф, с которым работаем
>
> MoneyLim = 50 // Минимальная сумма, ниже которой абонент будет
> остановлен
>
> UsersCount = *GetCountClientsInTarif*(Tarif)
>
> *GetClientsFromTarif*(Tarif, Clients)
>
> c=0
>
> loop:
>
> **if** (*GetMoney*(Clients\[c\]) &gt; MoneyLim) **then**
>
> *StartClients*(Clients\[c\])
>
> **else**
>
> *StopClients*(Clients\[c\])
>
> **end** **if**
>
> c=c+1
>
> **if** (c &lt; UsersCount) **then** **goto** loop
>
### **Пример 10. Подключение к произвольному хосту по Telnet и выполнение команд**
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* MikroBILL Script
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> //\*\*\* Соединяемся с произвольным оборудованием по Telnet \*\*\*
>
> //\*\*\* и отправляем команду. \*\*\*
>
> //\*\*\* Скрипт совместим с версией MikroBILL 1.6.9 и выше. \*\*\*
>
> //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> **var** TC
>
> TC = *CreateTelnetConnection*(**"10.64.1.1"**, 23, **"login"**,
> **"pass"**) // Создаём новое подключение по Telnet
>
> *TelnetSendSync*(TC,**"ip firewall address-list set \[/ip firewall
> address-list find list=test\] address=1.2.3.4"**) // Отправляем
> команду через соединение TC
>
### **Пример 11. Первые несколько месяцев после подключения ниже стоимость**
>
> **var** i
>
> **var** Clients\[0\]
>
> **var** Tarif
>
> **var** MonthsCount
>
> **var** ActionPrice
>
> // Название тарифа
>
> Tarif = **"test"**
>
> // Количество месяцев (в секундах)
>
> MonthsCount=1 \* 30 \* 86400
>
> // Размер оплаты по акции в первые месяцы
>
> ActionPrice=100
>
> *GetClients*(Clients)
>
> i=0
>
> loop:
>
> **If** *GetTarif*(Clients\[i\])=Tarif **then**
>
> **if** *ToUnixTime*(Now) -
> *ToUnixTime*(*GetClientCreationDate*(Clients\[i\])) &gt; MonthsCount
> **then**
>
> *SetPersonalFee*(Clients\[i\], 2, ActionPrice)
>
> **end** **if**
>
> **End** **if**
>
> i=i+1
>
> **if** (i&lt;*GetArraySize*(Clients)) **then** **goto** loop
>
### **Пример 12. Загрузка файла на FTP-сервер**
>
> **var** FTP
>
> FTP = *CreateFTPConnection*(**"ftp://localhost"**,
> **"Anonymous"**,**""**) // Создаём новое FTP подключение
>
> *FTPUploadFile*(FTP, **"test.txt"**,**""**) // Загружаем файл на FTP
> сервер. Файл test.txt должен быть расположен в папке
> %allusersprofile%\MikroBILL\Scripts\Files\\
>
### **Пример 13. Скачивание файла с FTP-сервера**
>
> **var** FTP
>
> FTP = *CreateFTPConnection*(**"ftp://localhost"**,
> **"Anonymous"**,**""**) // Создаём новое FTP подключение
>
> *FTPDownloadFile*(FTP, **"test.txt"**, **"test.txt"**) // Скачиваем
> файл с FTP сервера. Файл test.txt будет скачан в папку
> %allusersprofile%\MikroBILL\Scripts\Files\\
