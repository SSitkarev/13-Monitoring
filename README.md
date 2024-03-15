# Домашнее задание к занятию "13.Системы мониторинга" - Сергей Ситкарёв

1. Вас пригласили настроить мониторинг на проект. На онбординге вам рассказали, что проект представляет из себя платформу для вычислений с выдачей текстовых отчетов, которые сохраняются на диск. Взаимодействие с платформой осуществляется по протоколу http. Также вам отметили, что вычисления загружают ЦПУ. Какой минимальный набор метрик вы выведите в мониторинг и почему?

### Ответ

- Нагрузка на процессор
- Нагрузка на диск (скорость записи/чтения, индексные дескрипторы inodes)
- Мониторинг API (доступность, ошибки)
- Мониторинг RAM

2. Менеджер продукта посмотрев на ваши метрики сказал, что ему непонятно что такое RAM/inodes/CPUla. Также он сказал, что хочет понимать, насколько мы выполняем свои обязанности перед клиентами и какое качество обслуживания. Что вы можете ему предложить?

### Ответ

RAM - утилизация оперативной памяти; 
inodes - кол-во индексных дескрипторов, на каждый файл используется 1 inode. Если будет большое кол-во лекгковесных файлов-отчётов, то в какой-то момент inodes могут закончиться, при этом на диске может оставаться свободное пространство, но файлы не смогут создаваться;
CPUia - показатель средней загрузки CPU за промежуток времени.

Конечно, такие технические детали больше помогут админам найти проблемные и "узкие" места. Менеджерам и клиентам важны не технические показатели, а что они получат в результате работы сервиса. 
Для определения качества обслуживания и выполнения обязанностей перед клиентом необходимо определить целевые показатели производительности. Для данного можно выделить следующие показатели:

- Допустимое время недоступности сервиса (единоразовое и/или допустимое время недоступности в определённый промежуток времени)
- Допустимый процент ошибок при подготовке отчётов
- Среднее время ожидания ответа от сервиса при подготовке отчёта

3. Вашей DevOps команде в этом году не выделили финансирование на построение системы сбора логов. Разработчики в свою очередь хотят видеть все ошибки, которые выдают их приложения. Какое решение вы можете предпринять в этой ситуации, чтобы разработчики получали ошибки приложения?

### Ответ

Можно настроить open source решение для мониторинга, например, zabbix, prometeus, elastic stack, TICK stack и т.д.

4. Вы, как опытный SRE, сделали мониторинг, куда вывели отображения выполнения SLA=99% по http кодам ответов. Вычисляете этот параметр по следующей формуле: summ_2xx_requests/summ_all_requests. Данный параметр не поднимается выше 70%, но при этом в вашей системе нет кодов ответа 5xx и 4xx. Где у вас ошибка?

### Ответ

В формуле учтены только 2xx статус коды. Http запрос может завершаться с другими "не ошибочными" кодами, которые тоже необходимо учесть:

- Информационные 100 - 199
- Перенаправления 300 - 399

5. Опишите основные плюсы и минусы pull и push систем мониторинга.

### Ответ

#### push-модель

Плюсы:

- упрощение репликации данных в разные системы мониторинга или их резервные копии
- более гибкая настройка отправки пакетов данных с метриками
- UDP — это менее затратный способ передачи данных, из-за чего может возрасти производительность сбора метрик

Минусы:

- UDP не гарантирует доставку данных
- Необходимость проверки подлинности данных
- Необходимость настройки агента

#### pull-модель

Плюсы:

- легче контролировать подлинность данных
- можно настроить единый proxy server до всех агентов с TLS
- упрощённая отладка получения данных с агентов

Минусы:

- Необходимость настройки узлов мониторинга

6. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

Prometheus - pull
TICK - push
Zabbix - pull/push
VictoriaMetrics - pull/push
Nagios - push

7. Склонируйте себе репозиторий и запустите TICK-стэк, используя технологии docker и docker-compose.
В виде решения на это упражнение приведите скриншот веб-интерфейса ПО chronograf (http://localhost:8888).

![Задание7](https://github.com/SSitkarev/13-Monitoring/blob/main/img/7.jpg)

8. Перейдите в веб-интерфейс Chronograf (http://localhost:8888) и откройте вкладку Data explorer.

- Нажмите на кнопку Add a query
- Изучите вывод интерфейса и выберите БД telegraf.autogen
- В measurments выберите cpu->host->telegraf-getting-started, а в fields выберите usage_system. Внизу появится график утилизации cpu.
- Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации cpu из веб-интерфейса.

![Задание8](https://github.com/SSitkarev/13-Monitoring/blob/main/img/8.jpg)

9. Изучите список telegraf inputs. Добавьте в конфигурацию telegraf следующий плагин - docker:

```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```
  
Дополнительно вам может потребоваться донастройка контейнера telegraf в docker-compose.yml дополнительного volume и режима privileged:

```
  telegraf:
    image: telegraf:1.4.0
    privileged: true
    volumes:
      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
      - /var/run/docker.sock:/var/run/docker.sock:Z
    links:
      - influxdb
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"
```

После настройке перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список measurments в веб-интерфейсе базы telegraf.autogen . 
Там должны появиться метрики, связанные с docker.

![Задание9](https://github.com/SSitkarev/13-Monitoring/blob/main/img/9.jpg)