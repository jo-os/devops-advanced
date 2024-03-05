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
- consul validate /etc/consul.d/consul.hcl - проверка конфига
- consul join <ip-address or node name> - присоединится к кластеру указав участника
- consul leave - исключить ноду из кластера
- consule maint -enable/disable - ключить режим обслуживания, вернутся в рабочий режим
- consul members - показать участников кластера
- cinsul info - общая информация о кластере
- consul reload - перечитать конфигурацию агента
- consul catalog services - показать список зарегестрированных сервисов
- consul services register/deregister be.json - зарегистрировать/дерегистрировать сервис описанный в файле be.json

Конфигурация клиента
```
data_dir = "/opt/consul"  # - каталог для хранения данных - /opt/consul по умолчанию
client_addr = "192.168.0.1 127.0.0.1" # - интерфейс на который будут приходит запросы от клиентов, 127.0.0.1 для обращения к api из коммандной строки - иначе надо будет постоянно явно указывать адрес
bind_addr = "192.168.0.1" # - интерфейс для общения консул серверов в кластере
encrypt = "xxxxxxxxxxxx" # - ключ для создания шифрованного соединения - создается коммандой - consul keygen
enable_local_script_checks = true
```
Регистрация сервиса
```
"service":
{ "name": "be",
  "tags": ["be"],
  "check":
  {
    "id":"NGINX",
    "name": "Check",
    "http": "http://localhost",
    "method": "GET",
    "interval": "10s",
    "timeout": "1s"
  }
}
```
Веб интерфейс
```
ui_config{
enabled = true # - включаем в consul.hcl
}
consul reload
ssh -L 8500:localhost:8500 ip-address - пробрасываем порт
```
**Ansible**

https://github.com/ansible-collections/ansible-consul

https://github.com/nginxinc/ansible-role-nginx

https://github.com/joos-net/consul-role - простая своя роль для тестов

**Consule-template**
```
sudo apt-install consul-template # - ставим на балансироващик нагрузки
mkdir /etc/consul-template.d/ # - создадим каталог для конфига
chown -R consul:consul /etc/consul-template.d
```
Работа с key-value-хранилищем
```
Побавить ключ mykey со значением 5
curl -X PUT -d 5 http://127.0.0.1:8500/v1/kv/mykey
consul kv put mykey 5
Получить значение ключа mykey
curl -X GET http://127.0.0.1:8500/v1/kv/mykey
consul kv get mykey
```
Consule-template написан на языке go для работы с шаблонами используется шаблоны go (go-templates)

Запросы в каталог
```
{{ service "<TAG>.<NAME>@<DATACENTER>~<NEAR>|<FILTER>" }}
{{ services "@<DATACENTER>" }}
```
```
{{ range service "be" }}
server {{ .Name }} {{ .Address }}:{{ .Port }}
{{ end }}
```
```  
{{ key "<PATH>@<DATACENTER>" }}
{{ keyExists "<PATH>@<DATACENTER>" }}
{{ keyOrDefault "<PATH>@<DATACENTER>" "<DEFAULT>"}}
```
## CMS (СУК)
**Ansible**

**Минусы**
- хороший GUI с коммерческой лицензией
- не отслеживает состояние машины, так как не агентов
- yaml сложно дебажить

**Плюсы**
- написан на Python
- работает по ssh
- есть GUI AWX - хотя отстает от коммерческой версии
- yaml - легко читать
- для шаблонов есть jinja2
- есть много готовых решений
- можно создать свой модуль или дописать существующий

**Общие минусы СУК**
- сложно внедрить в существующиую инфраструктуру
- требуется время на создание конфигураций
- есть рись убить все разом

**Общие плюсы СУК**
- все сервера имеют стандарт настроек
- можем управлять конфигурацией сотен серверов
- больше не требуется ручных действий

- **Task** - описание действий которые нужно выполнить
- **Play** - набор ролей и тасков и список целевых хостов
- **Role** - структурированный набор плей,таск и переменных (все разложено по каталогам)
- **Playbook** - набор play - может быть один или много

**Порядок запуска playbook**
- pre_task
- handlers for pre_task
- roles/task (или по порядку или сначало зависимые)
- handlers for roles/task
- post_task
- handlers for post_task

Стратегии выполнения задачи или ролей
- Linear
- Free
```
[defaults]
strategy=free
----------------
Playbook:
  host: all
  strategy: free
```
- fork=5 - параллельный запуск на многих хостах
- serial: 5 - выполним на 5 хостах и после завершения перейдет к следующим 5 хостам
- run_once: true - выполнится один раз на первой машине
- dlegate_to: hostname - выполнится на указанном хосте - столько раз сколько хостов, c run_once - один раз

https://dzen.ru/a/ZQCA9_MxXR4GAnVK

- handlers - выполняются после play
- meta: flush_handlers - если нужно выполнить handler в task

- ansible-playbook site.yml --list-tasks - проверка какие таски будут выполняться
- ansible-playbook site.yml --list-hosts - проверка на каких хостаз будет все выполняться

Если есть роли то используем pre_task и post_task. 

Tags - метки в playbook которые позволяют выборочно запускать tasks
- always - всегда, если не пропускаем
- never - никогда, если не указываем запуск
--tags all - run all tasks, ignore tags (default behavior)
--tags tag1,tag2 - run only tasks with either the tag tag1 or the tag tag2
--skip-tags tag3,tag4 - run all tasks except those with either the tag tag3 or the tag tag4
--tags tagged - run only tasks with at least one tag
--tags untagged - run only tasks with no tags

inventory-файл
- описывает инфарстпуктуру, используется ansible для выполнения
- может быть в формате ini, json, yaml
- может быть статическим или динамическим

Ускорить выполнение playbook
- gather_facts: no
```
[defaults]
gathering = smart
fact_caching = redis
fact_caching_timeout = 7200

[defaults]
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/facts_cache
fact_caching_timeout = 7200
```
Хранение чувствительных данных
- хранить в KV-хранилище
- внешний файл по отношению к ansible
- переменные окружения
- ansible-vault
```
ansible-vault create file
ansible-vault view file
ansible-vault edit file
ansible-vault encript file
ansible-vault decrypt file
ansible-vault rekey file
ansible-vault playbook.yml --ask-vault-pass
ansible-vault encrypt_string
```
## Docker
**Мультисборка**
- создать контейнер для сборки
- перенести собранное приложение в пустой контейнер
- удалить контейнер для сборки
```
COPY --from=build /my-app /new-app
```
**Оптимизация образа**
- использовать меньший базовый образ
- мультисборка
- отключить кеширование, если используем данные извне
- минимизировать количество слоев
- не ставить ненужное (например для дебага)
- если возможно использовать образы distroless (java, python...)
- данные храним на внешнем диске
- вместо shared objects использовать статичные библиотеки

**Docker Compose** - инструмент, позволяющий запускать приложения, требующие несколько контейнеров или имеющие зависимости
```
docker compose up
docker compose down
docker compose logs -f [service_name]
```
**Docker Swarm** - инструмент оркестрации, для создания на базе контейнеров Docker и для управления ими

## CI/CD
Набор практик для создания процесса.
- быстрее пилим фичи
- быстрее тестируем и закрываем дыры безопасности
- быстее выкатываем пользователям

**Jenkins** - система по обеспечению непрерывной интеграции

https://plugins.jenkins.io - плагины Jenkins

- **Jobs** - задачи состоящие из других подзадач или тасок (типы - freestyle, pipeline, multi configurtion project)
- **Task** - элемент джобы, то есть атомарное действие

**Плюсы**
- большое комьюнити
- почти под все есть плагины
**Минусы**
- Голый Jenkins после деплоя
- Плагины нужны почти под все
- Неудобный интерфейс
- Скопировать конфигурацию Jenkins будет проблематично

Установка - https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

**GitLab**

**Переменные**
- Переиспользование кода
- Больше контроля над джобами
- Нет хардкода в конфиг файлах

**Типы переменных**
- Предопределенные - получаем метаинформацию о сборочнои окружении - https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
- Кастомные - объявляем сами для своих нужд

**Тэг** - метка для указания на определенный раннер
```
tags:
  - docker
  - python3
```
**Rules** - правила фильтры - например - коммит в мастер ветку - выполняем или нет определенные джобы

**Синтаксис**
- условия - if, change, exists
- операторы - ==, !=, ||, &&, =~ и другие
- следствие - when (always, never, on_success, on_failure, manual, Delayed), allow_failure, start_in

**Джоба запустится, если**
- if, change, exists совпадают с условием
- с when используются always, on_success, delayed

**Джоба не запустится, если**
- нет правил которые подходят и нет when c always, on_success, delayed
- when never

**Артефакты**

Варианты использования
- Выгрузка на машину
- Деплой в arifactory
- Продолжение работы внутри пайплайны
```
arifacts:
  paths:
    - ./
    - ./debs/*.deb # все deb пакеты в debs
  expire_in: 30 days
tags:
  - gitlab-org-docker
```
Получившиеся артифакты можно скачать в рамках джобы.
```
path:
  - ./*.sh # попкадут в arifacts
exclude: # в секции arifacts исключаем ненужные файлы
  - '*.deb'
  - '*.txt'
```
Имя для артифактов
```
artifacts:
  name: "${CI_JOB_STAGE}-src"
```

## Мониторинг
**VictoriaMetrics**
- TSDB
- Мониторинг
- Хранение метрик
- Prometheus, InfluxDB, Graphite как источники данных

**Плюсы**
- Простота эксплуатации
- Долгосрочное хранилище
- MetricsQL (обратно совмести с Promql)
- Эффективное использование ресурсов

**Vmagent**
- собирать метрики pull/push
- фильтровать метрики
- управлять потоком
- сохранять метрики в случае отказа remote store

**Consul + Prometheus** (Service Discovery)
```
scrape_configs:
  - job_name: consul
    consul_sd_configs:
      - server: "localhost:8500"
        tags: [test] 
```
Основные паттерны:
- U - utilisation - насколько загружено железо
- S - saturation - загрузка ноды в целом (ресурс имеет дополнительную работу, которую он не может обслужить, часто в очереди.)
- E - errors - как часто - что и где

- R - Rate - сколько поступает запросов
- E - Errors - сколько ошибок и почему
- D - Duration - сколько обрабатывается запрос

**Grafana**
- Визуализация
- Траблшутинг
- Поиск узких мест

**Основные идеи**
- дашборд отвечает на конкретный вопрос
- есть стратегия и цель для мониторинга
- документация которая описывает как устроен мониторинг

**Основные практики**
- лаконичные имена
- удаление ненужного
- суффиксы temp/study/test для временных дашбордов
- адекватные скрейп интервалы
- подробное документирование
- избегать избыточного копирования
- отсутствие повторяющихся имен
- очищение тегов при копировании

**Аннтотации** - для информативности и наглядности
- через ctrl выбираем или точну или область и подписываем для наглядности
- автоматическое срабатываение по тригеру

**Алерт** - уведомления для информирования
- Alerting - Notification Chanal - Telegramm
- на дашборде метрики выбираем владку alert
  - Conditions - условие при котором сработает алерт




