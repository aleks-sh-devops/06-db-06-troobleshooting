## Задача 1

Перед выполнением задания ознакомьтесь с документацией по [администрированию MongoDB](https://docs.mongodb.com/manual/administration/).

Пользователь (разработчик) написал в канал поддержки, что у него уже 3 минуты происходит CRUD операция в MongoDB и её 
нужно прервать. 

Вы как инженер поддержки решили произвести данную операцию:
- напишите список операций, которые вы будете производить для остановки запроса пользователя
- предложите вариант решения проблемы с долгими (зависающими) запросами в MongoDB

## Ответ  
3 минуты = 180 секунд.  
1. Посылаем mongo-shell запрос db.currentOp({"secs_running": {$gte: 180}}) для вычисления деградирующего запроса.  
2. Убиваем зависший запрос командой db.killOp(opid)

Вообще необходимо понимать этот случай носит единичный или массовый характер. Поэтому важно мониторить БД. Соответственно, ставим на мониторинг с помощью прометеуса и графаны. Нужно отслеживать состояние БД. Возможно дело в нехватке ресурсов, поэтому как один из вариантов - увеличение производительности сервера или ВМ. Кроме того, саму БД можно потюнить с помощью настроек индексов шарнирования и репликации. Ну и как вариант задать максимальное время выполнения запросов с помощью параметра maxTimeMS

## Задача 2

Перед выполнением задания познакомьтесь с документацией по [Redis latency troobleshooting](https://redis.io/topics/latency).

Вы запустили инстанс Redis для использования совместно с сервисом, который использует механизм TTL. 
Причем отношение количества записанных key-value значений к количеству истёкших значений есть величина постоянная и
увеличивается пропорционально количеству реплик сервиса. 

При масштабировании сервиса до N реплик вы увидели, что:
- сначала рост отношения записанных значений к истекшим
- Redis блокирует операции записи

Как вы думаете, в чем может быть проблема?

## Ответ  
Вероятно, это связано с тем что количество истекших ключей превысило отметку в 25% Необховимо следить чтобы у них не было одинаковое время жизни. В документации раздел Latency generated by expires как раз про это.
Также вероятная причина может быть связана с тем, что Redis -одномоточное приложение. Другими словами все входящие запросы обслуживает один поток. И когда этих запросов большое количество Редис начинает "тупить". Как вариант поиграться с конфигом (https://redis.io/docs/management/config-file/), а именно в сторону изменения частоты. Но слишком борщить не стоит так как это приводит к излишней нагрузке на проц даже при простаивании Redis.

## Задача 3

Перед выполнением задания познакомьтесь с документацией по [Common Mysql errors](https://dev.mysql.com/doc/refman/8.0/en/common-errors.html).

Вы подняли базу данных MySQL для использования в гис-системе. При росте количества записей, в таблицах базы,
пользователи начали жаловаться на ошибки вида:
```python
InterfaceError: (InterfaceError) 2013: Lost connection to MySQL server during query u'SELECT..... '
```

Как вы думаете, почему это начало происходить и как локализовать проблему?

Какие пути решения данной проблемы вы можете предложить?

## Ответ  
Вообще эта ошибка говорит о долгих запросах. Универсального решения нет, но можно попробовать:  
- Возможно, дело в нехватке ресурсов, поэтому как один из вариантов - увеличение производительности сервера или ВМ.  
- Возможно, стандартное значение connect_timeout (количество секунд, в течение которых сервер mysql ожидает пакет подключения, прежде чем прервать соединение) слишком маленькое и его необходимо увеличить.  
- Можно попробовать оптимизировать запросы  

А вообще начинать нужно с того, что мы включаем слоу_лог и смотрим, что нам отвечает БД  

## Задача 4

Перед выполнением задания ознакомтесь со статьей [Common PostgreSQL errors](https://www.percona.com/blog/2020/06/05/10-common-postgresql-errors/) из блога Percona.

Вы решили перевести гис-систему из задачи 3 на PostgreSQL, так как прочитали в документации, что эта СУБД работает с 
большим объемом данных лучше, чем MySQL.

После запуска пользователи начали жаловаться, что СУБД время от времени становится недоступной. В dmesg вы видите, что:

`postmaster invoked oom-killer`

Как вы думаете, что происходит?

Как бы вы решили данную проблему?

## Ответ  
Когда у сервера или процесса заканчивается память, Linux предлагает 2 пути решения: обрушить всю систему или завершить процесс (приложение), который съедает память. Лучше, конечно, завершить процесс и спасти ОС от аварийного завершения. В двух словах, Out-Of-Memory Killer — это процесс, который завершает приложение, чтобы спасти ядро от сбоя. Он жертвует приложением, чтобы сохранить работу ОС (источник: https://habr.com/ru/company/southbridge/blog/464245/).  

Возможные решения - добавление оперативы, включение свопа (но с этим нужно аккуратно ибо может привести к другой проблеме - замедлению БД) и ограничение потребляемой памяти сервисом PostgreSQL.  
