# Azure Configuration

1. Спочатку треба зареєструвати та зробити безкоштовну підписку на Azure Services

2. Далі, створюємо resource group. 
3. Створюємо EventHub та event hub instance та налаштовуємо для нього policy
![alt](../images/eventhub.png)
4.  Так само робимо з redis cache
![alt](../images/redis.png)

# Code

1. В папці Service в файлі SendDataConsoleImpl.java заміняємо CACHE_HOSTNAME, CACHE_KEY на відповідні дані з Azure.

2. В папці Service в файлі SendDataEventHubImpl.java міняємо CACHE_HOSTNAME, CACHE_KEY, namespace, hub name, Connection string–primary key, Primary key на відповідні з Azure.

3. Run project
![alt](../images/lab5_code.png)

# Post Data

1. Для запису даних в Event Hub або Redis, необхідно виконати POST запит на localhost:9000/url

2. Виконуємо запит через Postman. 
body:
```
{
 "url": "url",
 "strategy": "eventHub/redis" // відповідна стратегія
}
```
![alt](../images/lab5_postman_event.png)
![alt](../images/lab5_postman_redis.png)


3. Переходимо у Event Hub instance -> proccess data і дивимось результат
Для EventHub:
![alt](../images/eventhub_result.png)
Для redis: робимо реквест в терміналі hgetAll ConsoleLog
![alt](../images/redis_result.png)
