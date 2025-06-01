# Шардирование таблиц в кластере Citus

Ниже приведена краткая инструкция, как превратить обычную таблицу в распределённую (шардированную) в вашем Helm-кластерe Citus.

---

## 1. Подключаемся к PostgreSQL

```bash
# Замените <host> при необходимости (по-умолчанию citus-coordinator)
PGPASSWORD=$POSTGRES_PASSWORD psql -U $POSTGRES_USER -h <host> -p 5432
```

## 2. Переходим в нужную базу

```sql
\c post_service
```

## 3. Регистрируем воркеры

```sql
-- пример для двух worker-подов
SELECT citus_add_node('citus-worker-0.citus-worker', 5432);
SELECT citus_add_node('citus-worker-1.citus-worker', 5432);
-- при необходимости добавьте другие по аналогии
```

## 4. (Опционально) Балансируем шарды

```sql
SELECT rebalance_table_shards();
```

Эта команда равномерно распределит существующие шарды по всем доступным воркерам.

## 5. Делаем таблицу распределённой

```sql
-- распределяем по колонке uuid
SELECT create_distributed_table('posts', 'uuid');

-- удаляем локальные данные, чтобы освободить место
SELECT truncate_local_data_after_distributing_table('public.posts');
```

---

### Полезные запросы

* Проверить активные воркеры
  ```sql
  SELECT * FROM citus_get_active_worker_nodes();
  ```
* Узнать, где лежит конкретный шард
  ```sql
  SELECT p.shardid, n.nodename
  FROM pg_dist_placement p
  JOIN pg_dist_node n ON n.groupid = p.groupid AND n.noderole = 'primary'
  WHERE p.shardid = <shard_id>;
  ```

Теперь таблица `posts` распределена, и кластер готов к масштабируемым нагрузкам. Удачной работы!

```bash
    #Зайти в нужную базу
    \c post_service
```


```bash
    SELECT citus_add_node('citus-worker-0.citus-worker', 5432);
    SELECT citus_add_node('citus-worker-1.citus-worker', 5432);
    # если нужно больше воркеров то по анлогоии
```

```bash
    # и затем перебалансировать шард-плейсменты
    SELECT rebalance_table_shards();
```