# Домашнее задание - Работа с индексами (Виды индексов. Работа с индексами и оптимизация запросов)

## Подготовка тестовой среды

  1) С сайта https://neon.tech/postgresql/postgresql-getting-started/postgresql-sample-database тестовую открытую базу "dvdrental"
  2) Распаковал архив с базой
  3) Поднял базу на своем инстансе Postgres ``pg_restore -U postgres -d postgres -1 /home/nakrasyukov/Загрузки/dvdrental.tar``
  4) Выбрал таблицу "film" в качестве подопытного, удалил с этой таблицы все имеющиеся индексы, убедился что на таблице не осталогсь индексов ``SELECT indexname AS index_name, tablename AS table_name FROM pg_indexes WHERE tablename = 'film';``

     <img src="https://github.com/user-attachments/assets/4685188c-5256-4d73-b02b-faa0e3841aa3" alt="drawing" width="500"/>

## Работа с Инексами 

  **Создать индекс к какой-либо из таблиц вашей БД**

  1) Создаю индекс на поле film_id -  ``CREATE INDEX indx_film_id ON film(film_id);``

     <img src="https://github.com/user-attachments/assets/6696dd15-1bba-4965-bacd-6e8806713abc" alt="drawing" width="500"/>

  2) Результат команды ``EXPLAIN SELECT * FROM film WHERE film_id in (1,54,195,327,864,620);``:
     
     ```
     QUERY PLAN                                                                |
     --------------------------------------------------------------------------+
     Index Scan using indx_film_id on film  (cost=0.28..21.84 rows=6 width=384)|
       Index Cond: (film_id = ANY ('{1,54,195,327,864,620}'::integer[]))       |

     ```

  **Реализовать индекс для полнотекстового поиска**

  1) Создание индекса для полнотекстового поиска по полю description в таблице film - ``CREATE INDEX indx_film_description ON film USING gin(to_tsvector('english', description));``
  2) Проверяю работу поиска по индексу ``SELECT * FROM film WHERE to_tsvector('english', description) @@ to_tsquery('epic & drama');``

     <img src="https://github.com/user-attachments/assets/d4ff1476-5345-44c8-a55e-2b5892babbb7" alt="drawing" width="500"/>

  3) Результат команды ````:

     ```
     QUERY PLAN                                                                                              |
     --------------------------------------------------------------------------------------------------------+
     Bitmap Heap Scan on film  (cost=13.21..17.72 rows=1 width=384)                                          |
       Recheck Cond: (to_tsvector('english'::regconfig, description) @@ to_tsquery('epic & drama'::text))    |
       ->  Bitmap Index Scan on "indx_film_description"  (cost=0.00..13.21 rows=1 width=0)                   |
             Index Cond: (to_tsvector('english'::regconfig, description) @@ to_tsquery('epic & drama'::text))|
     ```
