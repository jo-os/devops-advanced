# devops-advanced
Продвинутый курс

## YAML
**Можно указать тип данных**
```
bool: !!bool yes
num: !!int 3
name: !!str string
ex: !!map {...}
...
```
**Поддерживаются типы данных из Python:**
```
!!python/none
!!python/bool
!!python/int
!!python/float
...
```
Запись длинного текста:

После пробела ставим знак больше и со следующей строки пишем текст - все будет записано в одну строку
```
long_text: >
Support for reading
and
writing
```
После двоеточия ставим пайп (вертикальная черта) и пишем текст - будут учтены переносы - характерно для Kubernetes
```
long_text: |
Support for reading and writing
```
Переменные (environments) - можем сделать ключ, поместить в него текст, списки, словари и использовать этот ключ в разных местах YAML файла.
```
foo: &defautl_settings
  env:dev
  db:
    host: localhost
    db_name: test
    db_port: 5432
```
```
dev: *default_settings - использование
или
stage:
  <<: *default_settings
```
Описание нескольких YAML в одном файле
```
---
foo: "Hello"
---
foo: "Bye"
```

## Балансировка нагрузки
**Балансировка нагрузки** - метод распределения заданий между несколькими сетевыми устройствами

**Основные задачи:**
- оптимизирование ресурсов
- ускорение обслуживания запросов
- обеспечение отказоустойчивости
- максимизация пропускной способности
- уменьшение времени отклика
- предотвращение перегрузки одного ресурса

**Nginx, HAProxy** - одни из самых быстрых и популярных балансировщиков нагрузкт

Nginx (HTTP трафик, трафик на сайты)
```
upstream mysite_backend{
  server mysite-backend:8081 weight=8;
  server 127.0.0.1:4213;
}
server {
    listen 80;
    server_name mysite.ru;
    access_log /var/log/nginx/mysite-access.log json;
    error_log /var/log/nginx/mysite-error.log;
  location / {
    proxy_pass http://mysite_backend
    include proxy_params;
  }
}
```
HAProxy (балансировка TCP трафика) 
```
frontend mysite-api
  bind *:80
    mode tcp
    option tcplog
    default_backend mysite-api

backend mysite-api
    mode tcp
    balance roundrobin
  server mysite0master01-prod mysite-master01.prod.local:8001 check
  server mysite0master02-prod mysite-master02.prod.local:8001 check
  server mysite0master03-prod mysite-master03.prod.local:8001 check
```
```
listen mysite-api
  bind *:80
    option tcp-check
    default-server on-marked-down shutdown-sessions on-marked-up
shutdown-backup-sessions
  server mysite0master01-prod mysite-master01.prod.local:8001 check
  server mysite0master02-prod mysite-master02.prod.local:8001 check
  server mysite0master03-prod mysite-master03.prod.local:8001 check

```
## Прокси сервера
Прямой прокси-сервер - промежуточный сервер, который перенаправляет запрос/трафик от клиента к конечному пункту, например, веб-сайту.
- используется для обхода ограничений
- выступает в роли кеш-сервера во внутренней сети

Обратный прокси-сервер - ретранслирует запросы клиентов из внешней сети на один или несколько серверов расположенных во внутренней сети.
- Подключение SSL к сайту
- Защита от DoS и DDoS атак
- Уменьшение нагрузки на сервер за счет кеширования
- Скрытие опрашиваемых серверов и их характеристик
- Выполнение функций балансировщика
- Отказоустойчивость в виде резервирования серверов
```
server {
  listen 80;
  server_name mysite.ru;
  return 301 https://$host$request_url;
}

server {
  listen 443;
  server_name mysite.ru
  access_log /var/log/nginx/mysite-access.log json;
  error_log /var/log/nginx/mysite-error.log;
  ssl on;
  ssl_certificate /etc/ssl/certs/cert.pem;
  ssl_certificate_key /etc/ssl/private/key.pem;
  location / {
    proxy_pass http://127.0.0.1:9097;
    include proxy_params;
  }
}
```
## Виды БД
- Реляционные (SQL) - таблицы
- Нереляционные (NoSQL)
  - бд временных рядов - значение времени - данные - удобно для мониторинга
  - столбцовые - ClickHouse, Cassandra - аналитика, анализ данных, онлайн игры, мессенджеры - высокая скорость обработки запросов
  - графовые - гибкие - могут расти бесконечно - имеют высокую скорость поиска - это узлы и ребра (отношения между узлами) - соцсети, биоинформатика
  - документоориентированные - высокая доступность и гибкость данных (используются как каталоги) - единица хранения документ - xml, json, bson - данные как ключ-значение - каждая строка имеет определенные характеристики
  - ключ-значение - простая структура - высокая скорость - легко шардируются (= масштабируются) - кеш, хранение изображений

## Consul
**Consul** применяется для
- Service Discovery - обнаруживаеи новые сервисы и добавляет их в кластер
- Key-value хранилище - позволяет хранить конфиги
- Health checking - проверка доступности сервиса для принятия решения об исключении сервиса из кластера
- Multi datacenter - позволяет запускать кластер в нескольких дата-центрах

**Особенности**
- может работать в режиме server или client
- написан на языке Go, один исполняемый файл без зависимостей
- количество серверос должно быть нечетным - 3 или 5
- внутри реализован алгоритм RAFT для выбора master
- для обмена данными между нодами используется протокол gossip
- можно динамически менять конфигурацию кластера
- позволяет создавать геораспределенные отказоустойчивые системы
- имеет HTTP API и DNS API для запросов о состоянии кластера

**Server**
- запущен на отдельных машинах так как более ресурсоемкий
- образуют отказоустойчивый кластер
- обращаются с Consul-клиентами

**Client**
- работают на машинах где запущен сервис
- проверяют состояние сервиса
- отправляют информацию на сервер

**Обнаружение отказов**
- consul сам опрашивает сервис например через http запрос
- приложение само сообщает что работает, например методом put
- стороннее приложение или скрипт проверяет работу сервиса и сообщает о его работоспособности

**Конфигурирование и запуск Consul**
- запускаем сервер на отдельной машине
- запускаем агент в режиме клиента на машине с сервисом
- присоединяем клиента к серверу командой consul join. (присоединение можно автоматизировать)

Плюсы подхода
- добваление ноды в кластер с минимальными затратами
- надежно
- возможность управлять конфигурацией кластера

Минусы
- требует пересмотр архитектуры
- требуется дополнительный компонент для переконфигурирования load balancer
- установке и конфигурирование новой ноды в зависимости от ее роли сложнее реализовать

Установка
```
https://www.consul.io/downloads

unzip consul_1.11.1_linux_amd64.zip
cp consul ~/bin/
export PATH=$PATH:~/bin
```
```
apt install software-properties-common
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release - cs) main"
sudo apt-get update && sudo apt-get install consul
```
/etc/consul.d/consul.hcl
```
datacenter = "dc" # - можно указать любое
server = true  # - агент работает в режиме сервер - если клиент то срока будет закомментирована
data_dir = "/opt/consul"  # - каталог для хранения данных - /opt/consul по умолчанию
client_addr = "192.168.0.1 127.0.0.1" # - интерфейс на который будут приходит запросы от клиентов, 127.0.0.1 для обращения к api из коммандной строки - иначе надо будет постоянно явно указывать адрес
bind_addr = "192.168.0.1" # - интерфейс для общения консул серверов в кластере
encrypt = "xxxxxxxxxxxx" # - ключ для создания шифрованного соединения - создается коммандой - consul keygen
```
Полезные комманды
- consul validate /etc/consul.d/consul.hcl
- consul join <ip-address or node name>
- consul leave
- consule maint -enable/disable
- consul members
- cinsul info
- consul reload
- consul catalog services
- consul register/deregister fe.json




