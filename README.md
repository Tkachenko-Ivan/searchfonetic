# Описание

Создание Docker контейнера для тестирования фонетических алгоритмов.

Ход выполнения. 

1. Запускаю [docker-compose.yaml](/docker-compose.yaml). Поднимаются два контейнера, один с БД [postgres](https://hub.docker.com/_/postgres), второй с [manticore](https://hub.docker.com/r/manticoresearch/manticore). В БД будут храниться индексируемые данные.
2. Порт для работы с postgres 5432 - открыт, поэтому создаю подключение, и импортирую данные в таблицу `fias_addrobj`. Данные в файле [fias_addrobj_tymen.csv](/fias_addrobj_tymen.csv).
3. Через `docker exec`, в контейнере `manticore`, выполняю `gosu manticore indexer --all --rotate`. Это создаёт навые индексы на основе конфига [manticore.conf](/manticore/config/manticore.conf).
4. Контейнер `manticore` перезапускаю `restart`.

Выполнив в контейнер `manticore`, через `docker exec`, команды: 
```
cd data
ls
```
Вы должны увидеть:
![ExecWindow](https://user-images.githubusercontent.com/10295935/112434309-3d9f5e80-8d65-11eb-9098-eb2350cf3a16.png)

Если картинка не соответветствует, например в именах файла присутствует `.new.`, значит что-то пошло не так. Возможно контейнер не был перезапущен, либо индексы созданы под другим пользователем и служба `manticore` не может получить к ним доступ.

После перезапуска, можно подключиться к `manticore` через mysql или по HTTP.

* mysql
```
mysql -h 127.0.0.1 -P 9306 --default-character-set=utf8
```

```
mysql> select * from rus_metaphone where match('Линейная');
+------+--------------------------------------+-----------+------------------+
| id   | aoguid                               | shortname | offname          |
+------+--------------------------------------+-----------+------------------+
|    5 | 01339f2b-6907-4cb8-919b-b71dbed23f06 | ул        | Линейная         |
+------+--------------------------------------+-----------+------------------+
1 row in set (0.00 sec)
```

* HTTP
```
POST localhost:9308/search -d '{"index":"rus_metaphone","query":{"match":{"*":"Линейная"}}}'
```

```JSON
{
    "took": 0,
    "timed_out": false,
    "hits": {
        "total": 1,
        "hits": [
            {
                "_id": "5",
                "_score": 1727,
                "_source": {
                    "aoguid": "01339f2b-6907-4cb8-919b-b71dbed23f06",
                    "shortname": "ул",
                    "offname": "Линейная"
                }
            }
        ]
    }
}
```

Полученный из контейнера `manticore` образ, можно найти на Docker Hub: [tkachenkoivan/searchfonetic](https://hub.docker.com/r/tkachenkoivan/searchfonetic)

# См. также
* [Manticore Search Manual](https://manual.manticoresearch.com/Introduction)
* [manticore.conf](https://github.com/manticoresoftware/docker/blob/master/manticore.conf)
