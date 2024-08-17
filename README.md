# jmeter-custom-influxdb

Дашбоард для мониторинга запусков сценариев НТ написанных с помощью интсрумента нагрузочного
тестирования [Jmeter](https://jmeter.apache.org/).  
Отправка метрик осуществляется с помощью 
[Backend listener](https://jmeter.apache.org/usermanual/component_reference.html#Backend_Listener)
в InfluxDB (Query Language: InfluxQL).

---
## С чего начать?
Мини инструкция перед началом эксплуатации:
1. Импортировать дашборд
2. Настроить подключение к InfluxDB через Data sources
3. Изменить параметры дашборда
   * Открыть Home --> Dashboards --> jmeter-custom-influxdb --> Settings --> Variables
   * UID - указать uid-дашборда (если был изменен при импорте)
   * dashboardName - указать наименование дашборда (если было изменено при импорте)
4. Внести несколько изменений в Backend Listener
   * **application = ${__TestPlanName}** - название jmx-файла
   * **measurement = jmeter** - наименование Database в настройках подключения к InfluxDB
   * **testTitle = ${__P(nameTest)}** - название теста (либо другая ~~полезная~~ информация)
   * **eventTags = ${__time()}** - уникальный RunID сценария
   * **influxdbToken = {influxdbToken}** - 
   ваш [API Token](https://docs.influxdata.com/influxdb/cloud/admin/tokens/create-token/)

![Backend Listener - картинка](https://raw.githubusercontent.com/promokk/jmeter-custom-influxdb/main/data/Backend_Listener.png)

---
## Возможности дашборда
**!!! ПРИМЕЧАНИЕ !!!**
* На всех панелях отображается одновременно информация о запросах и транзакциях. Учитывайте это при
просмотре **_Summary_**.  
* Параметр _transaction_ позволяет выбрать необходимые запросы / транзакции для просмотра в **_RPS / Response Times_**.
Также во всех таблицах есть фильтры, которые позволяют сделать тоже самое.  
* Использование _Assertion_ влияет на отображение ошибок 4xx и 5xx на панелях. В случае неудачной проверки
вместо кода ошибки будет отображаться сообщение - Assertion errors.

---
Проверить поведение при распределенном тестировании и написать об этом. То что при маленьком rps метрика
отображается некорректно.

---

### Run Test
Таблица Run Test - статус запущенных / завершенных тестов. Для работы данной таблицы нужно обязательно передавать
уникальное значение eventTags в Backend listener.  
1. **Status** - отображает состояние запуска, RUN или END. Запуск переходит в стаутс END при завершии теста.
Тест обязательно должен завершиться корректно. Либо по истичении времени,
либо должен быть остановлен с помощью /jmeter/bin/shutdown.sh.  
2. **Test** - выводит значение testTitle из Backend listener.
3. **RunID** - выводит значение eventTags из Backend listener. Значение RunID кликабельно. Открывает временной
интервал проводимого теста.

![Run Test - гифка](https://raw.githubusercontent.com/promokk//jmeter-custom-influxdb/main/data/Run_Test.gif)

---
### Summary
Содержит основную информацию для первичного анализа:
1. Активные / Завершившиеся потоки
2. Пропускная спобоность 
3. Кол-во / Процент ошибок
4. Кол-во запросов
5. Сеть

![Summary - картинка](https://raw.githubusercontent.com/promokk/jmeter-custom-influxdb/main/data/Summary.png)

---
### RPS / Response Times
Отображает успешные и неудачные запросы, времена отклика.
1. Кол-во успешных / неудачных запросов в секунду
2. Время отклика успешных / неудачных запросов
3. Кол-во запросов 4xx и 5xx в секунду
4. Количесво / Процент неудачных запросов по кодам

![RPS/Response Time - картинка](https://raw.githubusercontent.com/promokk/jmeter-custom-influxdb/main/data/RPS_Response_Time.png)

---
### Error
Таблица Error Info - краткая информация об ошибках.  
Поле Response Message отображает наименование ошибки. Полное сообщение об ошибки (тело ответа) **не выводится**.

![Error - картинка](https://raw.githubusercontent.com/promokk/jmeter-custom-influxdb/main/data/Error.png)

---
### Transaction Table
Таблица Transaction Table - статистика запросов / транзакций.
1. Наименование запроса / транзакции
2. Кол-во запросов (общее / успешное / неудачное)
3. Кол-во запросов в секунду (среднее / максимальное)
4. Время отклика (95th percentile / 99th percentile)
5. Процент неудачных запросов

![Transaction Table - картинка](https://raw.githubusercontent.com/promokk/jmeter-custom-influxdb/main/data/Transaction_Table.png)

---
### Network Traffic
Отображает полученные байты / отправленные байты

![Transaction Table - картинка](https://raw.githubusercontent.com/promokk/jmeter-custom-influxdb/main/data/Network_Traffic.png)

---
### Correlation
Аннотация _Сorrelation_ позволяет поставить маркер на панели. Данный маркер будет отображаться на всех панелях.  
**Обязательное условие** - при создании аннотации небоходимо указать tags --> **lt**.

![Correlation - гифка](https://raw.githubusercontent.com/promokk/jmeter-custom-influxdb/main/data/Correlation.gif)

---
## Пример запуска теста в Non-GUI mode
Пример для [распределенного запуска](https://jmeter.apache.org/usermanual/remote-test.html).
Обратите внимание при распределенном запуске _properties_ передаются с помощью -G{param}.  
Чтобы передать _properties_ в Backend Listener необходимо использовать -D{param} (используется для указания testTitle).  

С параметрами командной строки можно ознакомиться по
[ссылке](https://jmeter.apache.org/usermanual/get-started.html#non_gui).

~~~shell
nohup /opt/jmeter/bin/jmeter -n -t scriptExample.jmx -X -R localhost,192.168.1.101 -DtestName=example > /dev/null 2>&1&
~~~

