
# Домашнее задание Сартисона Евгения N15 #

(1) Установите Tarantool одним из доступных способов: через виртуальную машину (ВМ) или Docker.

(2) Создайте в Box спейс (таблицу) для хранения данных о поисках авиабилетов с полями:

id (уникальный идентификатор),

airline (авиакомпания),

departure_date (дата вылета),

departure_city (город вылета),

arrival_city (город прилета),

min_price (минимальная стоимость авиабилета).

(3) Добавьте первичный индекс для спейса.

(4) Создайте вторичный индекс на поля departure_date, airline и departure_city.

(5) Вставьте несколько записей в спейс.

(6) Напишите запрос для выборки минимальной стоимости авиабилета на рейсы с датой вылета 01.01.2025.

(7) Реализуйте функцию на Lua для вывода списка рейсов с минимальной стоимостью билета менее 3000 рублей.

На проверку отправьте текстовый документ, содержащий:
текст выполняемых запросов и настроек для каждого этапа задания;
код функции на Lua;
скриншоты вывода запросов и результата работы функции.


## (1) Установите Tarantool одним из доступных способов: через виртуальную машину (ВМ) или Docker. ## 

Использую пример из [Otus-demo tarantool-1](https://gitlab.com/otus-demo/nosql/course/tarantool-1) 

Поднял docker контейнер
```
student:~/tarantool/tarantool-1-main$ sudo chmod -R 777 /my/own/datadir
student:~/tarantool/tarantool-1-main$ docker run --name mytarantool \
  -p 3301:3301 \
  -v /my/own/datadir:/var/lib/tarantool \
  -d tarantool/tarantool
6ef94d2ad8e765cdf3c31d936c7878c0e4b966ed94f07fef30f89056ab8795bd
student:~/tarantool/tarantool-1-main$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS                            PORTS                                         NAMES
6ef94d2ad8e7   tarantool/tarantool   "/docker-entrypoint.…"   5 seconds ago   Up 5 seconds (health: starting)   0.0.0.0:3301->3301/tcp, [::]:3301->3301/tcp   mytarantool
student:~/tarantool/tarantool-1-main$ docker exec -it mytarantool console
   • Connecting to the instance...
   • Connected to /var/run/tarantool/sys_env/default/instance-001/tarantool.control

/var/run/tarantool/sys_env/default/instance-001/tarantool.control> 
```



## (2) Создайте в Box спейс (таблицу) для хранения данных о поисках авиабилетов ## 

Создайте в Box спейс (таблицу) для хранения данных о поисках авиабилетов с полями:
id (уникальный идентификатор),
airline (авиакомпания),
departure_date (дата вылета),
departure_city (город вылета),
arrival_city (город прилета),
min_price (минимальная стоимость авиабилета).
```
local flight_searches = box.schema.space.create('flight_searches', {
    if_not_exists = true,
    -- Описываем структуру полей (формат данных)
    format = {
        {name = 'id',             type = 'unsigned'}, -- уникальный идентификатор
        {name = 'airline',        type = 'string'},   -- авиакомпания
        {name = 'departure_date', type = 'string'},   -- дата вылета (обычно хранится как ISO строка 'YYYY-MM-DD')
        {name = 'departure_city', type = 'string'},   -- город вылета
        {name = 'arrival_city',   type = 'string'},   -- город прилета
        {name = 'min_price',      type = 'number'}    -- минимальная стоимость (поддерживает целые и дробные числа)
    }
})
```


## (3) Добавьте первичный индекс для спейса. ##
Создаем первичный индекс по полю 'id' (первое поле, индекс 1)
```
flight_searches:create_index('primary', {
    type = 'TREE',
    parts = {{'id', 'unsigned'}},
    if_not_exists = true
})
```

## (4) Создайте вторичный индекс на поля departure_date, airline и departure_city. ##

```
flight_searches:create_index('date_airline_city', {
    type = 'TREE',
    parts = {
        {'departure_date'},
        {'airline'},
        {'departure_city'}
    },
    if_not_exists = true
})
```


## (5) Вставьте несколько записей в спейс. ##

Сделал это через питон script с помощью кода
```
box.space.flight_searches:insert({
    1, 
    'Aeroflot', 
    '2026-10-15', 
    'Moscow', 
    'Rostov-on-Don', 
    5500.50
})

box.space.flight_searches:insert({
    2, 
    'Aeroflot', 
    '2026-10-15', 
    'Moscow', 
    'Samara', 
    5900.50
})

box.space.flight_searches:insert({
    3, 
    'Aeroflot', 
    '2026-10-15', 
    'Moscow', 
    'Saratov', 
    5000.50
})

box.space.flight_searches:insert({
    4, 
    'Aeroflot', 
    '2026-10-15', 
    'Moscow', 
    'Kazan', 
    7500.50
})

box.space.flight_searches:insert({
    5, 
    'Aeroflot', 
    '2026-10-15', 
    'Moscow', 
    'Tula', 
    2000.50
})

box.space.flight_searches:insert({
    6, 
    'Aeroflot', 
    '2026-10-15', 
    'Moscow', 
    'Tver', 
    1500.50
})


box.space.flight_searches:insert({
    7, 
    'Aeroflot', 
    '2027-01-01', 
    'Moscow', 
    'Perm', 
    1500.50
})
```


## (6) Напишите запрос для выборки минимальной стоимости авиабилета на рейсы с датой вылета 01.01.2027. ##

Сделал это через питон script с помощью кода
```
local results = box.space.flight_searches.index.date_departure:select({'2027-01-01'})
---
- [7, 'Aeroflot', '2027-01-01', 'Moscow', 'Perm', 1500.5]
...


```

## (7) Реализуйте функцию на Lua для вывода списка рейсов с минимальной стоимостью билета менее 3000 рублей. ##

Сделал это через питон script с помощью кода
```
local results = box.space.flight_searches.index.price:select({3000}, {iterator = 'LT'})
---
- [5, 'Aeroflot', '2026-10-15', 'Moscow', 'Tula', 2000.5]
- [6, 'Aeroflot', '2026-10-15', 'Moscow', 'Tver', 1500.5]
- [7, 'Aeroflot', '2027-01-01', 'Moscow', 'Perm', 1500.5]
...

```
