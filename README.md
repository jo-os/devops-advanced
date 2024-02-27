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




