# Домашнее задание - Настройка autovacuum с учетом особеностей производительности (MVCC, vacuum и autovacuum)

## Создание виртуальной машины в VirtualBox

  1) Скачиваю образ дистрибутива Linux с Официального сайта
  2) Создаю новую вм со стандартными настройками:
     
     <img src="https://github.com/user-attachments/assets/5e1d6cd0-f5fb-49e8-b9b5-cd742df7c2ea" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/15b0e3c6-3b76-4924-93be-17608f1566a5" alt="drawing" width="500"/>

  3) Запускаю ВМ и устанавливаю ОС

     <img src="https://github.com/user-attachments/assets/4219c519-c9b7-4beb-898b-fc67e6145904" alt="drawing" width="500"/>

## Установка Postgres

  1) Устанавливаю Postgres на ВМ: ``sudo pacman -S postgresql``
  2) Инициализирую кластер базы данных: ``sudo -u postgres initdb --locale=en_US.UTF-8 -D /var/lib/postgres/data``
  3) Запускаю PostgreSQL: ``sudo systemctl start postgresql``
  4) Проверяю статус кластера: ``sudo -u postgres pg_ctl status -D /var/lib/postgres/data``

     <img src="https://github.com/user-attachments/assets/4fb9418a-c40f-471f-8b06-e2ca6452e5f2" alt="drawing" width="500"/>

## Оптимизация Postgres

  1) Выполнил команду инициализации данных для тестирования: ``pgbench -i postgres``
  2) Запустил тест производительности: ``pgbench -c 8 -P 6 -T 60 -U postgres postgres``

     <img src="https://github.com/user-attachments/assets/2a371a68-32df-4570-95ac-7ae6e51a6094" alt="drawing" width="500"/>

  3) Вношу параметры из файла приложенного к ДЗ в свой файл конфигурации: ``nano /var/lib/postgres/data/postgresql.conf``

     <img src="https://github.com/user-attachments/assets/7b3a93b8-f55a-4148-a766-22608f2cbbcc" alt="drawing" width="500"/>

  4) Перезапускаю кластер: ``sudo systemctl restart postgresql``
  5) Заново провожу тестирование кластера: ``pgbench -c 8 -P 6 -T 60 -U postgres postgres``

     <img src="https://github.com/user-attachments/assets/962be211-90f9-4b44-9ac9-8674e90d9569" alt="drawing" width="500"/>

  6) Количество транзакций в секунду несколько уменьшилось, предположительно из-за того что в приложенном к ДЗ файле описаны настройки Postgres для иной конфигурации системы. А именно несоответсвуют количество ядер из файла и в первом пункте задания, а также не соответсвует тип накопителя, из-за чего предложенные настройки могут не являться оптимальными для конкретной системы.

## Работа с Автовакуум

  Задание со *, процедура для обновления строк в таблице:

  ```
    CREATE OR REPLACE FUNCTION update_rows_in_table(table_name TEXT, iterations INT)
    RETURNS VOID
    LANGUAGE plpgsql
    AS $$
    DECLARE
        i INT := 1; -- Переменная для отслеживания шага цикла
    BEGIN
        -- Цикл выполняется заданное количество раз
        FOR i IN 1..iterations LOOP
            -- Выполнение динамического SQL для обновления строк в указанной таблице
            EXECUTE format('UPDATE %I SET text_field = text_field || ''*''', table_name);
            
            -- Вывод текущего шага цикла
            RAISE NOTICE 'Шаг цикла: %', i;
        END LOOP;
    END;
    $$;
  ```

  1)  Создаю таблицу с текстовым полем и заполняю данными: ``CREATE TABLE test_data (text_field TEXT);``, ``INSERT INTO test_data (text_field) SELECT md5(random()::text) FROM generate_series(1, 1000000);``
  2)  Смотрю размер файла таблицы: ``SELECT pg_size_pretty(pg_total_relation_size('test_data'));``

      <img src="https://github.com/user-attachments/assets/59f66d3c-2286-4e67-ba30-366e42046a4c" alt="drawing" width="500"/>
    
  3) Пользуясь ранее созданной процедурой обновляю все строки таблицы 5 раз: ``SELECT update_rows_in_table('test_data', 5);``

      <img src="https://github.com/user-attachments/assets/d8a0eebb-12b2-4c9b-a4c5-7ed29ddbf3a5" alt="drawing" width="500"/>

  4) Посмотрел количество мертвых строк: ``SELECT relname, n_dead_tup FROM pg_stat_user_tables WHERE relname = 'test_data';``

      <img src="https://github.com/user-attachments/assets/a5d532ff-1122-46ac-8801-e7fb9842ff55" alt="drawing" width="500"/>

  5) Пользуясь ранее созданной процедурой обновляю все строки таблицы 5 раз: ``SELECT update_rows_in_table('test_data', 5);``
  6) Посмотрел размер файла с таблицей ``SELECT pg_size_pretty(pg_total_relation_size('test_data'));``

      <img src="https://github.com/user-attachments/assets/d286a617-227f-4f2c-be94-e0a1680d732f" alt="drawing" width="500"/>

  7) Отключил автовакуум для этой таблицы: ``ALTER TABLE test_data SET (autovacuum_enabled = false);``
  8) Пользуясь ранее созданной процедурой обновляю все строки таблицы 10 раз: ``SELECT update_rows_in_table('test_data', 10);``
  9) Посмотрел размер файла с таблицей ``SELECT pg_size_pretty(pg_total_relation_size('test_data'));``

      <img src="https://github.com/user-attachments/assets/c43dad26-1953-4673-a63a-a7fcc8265d82" alt="drawing" width="500"/>

  10) После этой оперции размер таблицы значительно увеличился. Увеличение размера таблицы после нескольких обновлений связано с тем, что в PostgreSQL операции UPDATE фактически создают новые версии строк, а старые версии помечаются как «мертвые». Эти мертвые строки занимают место в таблице и приводят к увеличению её размера.

**Готово! И на этом домашнее задание выполнено!**
