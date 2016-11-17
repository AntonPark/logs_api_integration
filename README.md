# Интеграция с Logs API

Данный скрипт реализует функциональность интеграции Logs API от Яндекс.Метрики с ClickHouse.

## Требования
Скрипт написан на Python 2.7, в нем используются библиотеки `requests` и `pandas`, которые можно установить с помощью менеджера пакетов [pip](https://pip.pypa.io/en/stable/installing/)
```bash
pip install requests pandas
```
Кроме того, для работы необходима СУБД ClickHouse, инструкцию по ее установке можно найти на [официальном сайте](https://clickhouse.yandex/).

## Настройка
Прежде всего, необходимо заполнить [config](./configs/config.json)
```javascript
{
	"token" : "<your_token>", // токен для доступа к API Яндекс.Метрики
	"counter_id": "<your_counter_id>", // номер счетчика
	"visits_fields": [ // список параметров визитов
	    "ym:s:counterID",
	    "ym:s:dateTime",
	    "ym:s:date",
	    "ym:s:firstPartyCookie"
	],
	"hits_fields": [ // список параметров хитов
	    "ym:pv:counterID",
	    "ym:pv:dateTime",
	    "ym:pv:date",
	    "ym:pv:firstPartyCookie"
	],
	"log_level": "INFO", // уровень логирования
	"retries": 1, // количество попыток перезапустить скрипт в случае ошибки
	"retries_delay": 60, // перерыв между попытками
	"clickhouse": {
		"host": "http://localhost:8123", // адрес поднятого инстанса ClickHouse
		"user": "", // логин для доступа к БД
		"password": "", // пароль для доступа в БД
		"visits_table": "visits_all", // имя таблицы для хранения визитов
		"hits_table": "hits_all", // имя таблицы для хранения хитов
		"database": "default" // имя базы данных для таблиц
	}
}
```

## Запуск программы
При запуске программы необходимо указать источник данных (хиты или визиты) с помощью опции `-source`.

Скрипт умеет работать в нескольких режимах:
 * __history__ - скрипт выгрузит все данные с даты создания счетчика Метрики до позавчерашнего дня
 * __regular__ - в таком режиме будут выгружены данные за позавчера (рекомендуется использовать такой режим для регулярных выгрузок)
 * __regular_early__ - будут получены данные за вчерашний день
 
Пример запуска программы:
```bash
python metrica_logs_api.py -mode history -source visits
```

Кроме того, есть возможность получить данные за конкретный промежуток времени:
```bash
python metrica_logs_api.py -source hits -start_date 2016-10-10 -end_date 2016-10-18
```
