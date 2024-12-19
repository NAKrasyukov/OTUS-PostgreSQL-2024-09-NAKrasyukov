# Домашнее задание - Бэкапы (Резервное копирование и восстановление)

## Подготовка тестовой среды

  1) Создаю в БД схему и вней таблицу: 
  ```
      CREATE SCHEMA example_schema;

      CREATE TABLE example_schema.example_table (
          id SERIAL PRIMARY KEY,
          name TEXT NOT NULL,
          value INTEGER NOT NULL
      );
  ```

  2) Заполняю таблицу автосгенерированными 100 записями:
  ```
    INSERT INTO example_schema.example_table (name, value)
    SELECT 'Name_' || i, (RANDOM() * 100)::INTEGER
    FROM generate_series(1, 100) AS i;
  ```

  3) Убеждаюсь в том что данные сели в таблицу: ``SELECT * FROM example_table;``

     <img src="https://github.com/user-attachments/assets/32727107-89b1-4439-8fb6-fc9867e0bcdd" alt="drawing" width="500"/>

  4) Создаю каталог для бэкапов ``sudo -u postgres mkdir /var/lib/postgres/data/backups`` и выдаю права ``sudo -u postgres chmod 700 /var/lib/postgres/data/backups``

## Логический бэкап (COPY)

  1) Создаю бэкап: ``COPY example_schema.example_table TO '/var/lib/postgres/data/backups/example_table.csv' CSV HEADER;``

     <img src="https://github.com/user-attachments/assets/0f8e1069-20f2-4b66-bdb9-a5d50596f111" alt="drawing" width="500"/>

  2) Создаю вторую таблицу:
  ```
    CREATE TABLE example_schema.example_table_copy (
        id SERIAL PRIMARY KEY,
        name TEXT NOT NULL,
        value INTEGER NOT NULL
    );
  ```

  3) Восстанавливаю данные из бэкапа во вторую таблицу: ``COPY example_schema.example_table_copy FROM '/var/lib/postgres/data/backups/example_table.csv' CSV HEADER;``
  4) Убеждаюсь в том что данные сели в таблицу: ``SELECT * FROM example_table;``

     <img src="https://github.com/user-attachments/assets/b336f3fb-eeaa-46e5-b999-c83af4a5d529" alt="drawing" width="500"/>

## Создать бэкап двух таблиц в кастомном сжатом формате с помощью pg_dump

  1) Создаю бэкап в кастомном сжатом формате двух таблиц: ``pg_dump -U postgres -t example_schema.example_table -t example_schema.example_table_copy -Fc postgres > /var/lib/postgres/data/backups/new_backup.dump``

      <img src="https://github.com/user-attachments/assets/1b838718-4bc5-43c5-ab56-f1deac778de2" alt="drawing" width="500"/>

  2) Создаю новую базу данных: ``CREATE DATABASE new_database;``
  3) Создаю в ней схему ``CREATE SCHEMA example_schema;``
  4) Восстанавливаю только вторую таблицу на новой базе данных: ``pg_restore --dbname new_database --table=example_table_copy /var/lib/postgres/data/backups/two_tables_backup.dump``
  5) Убеждаюсь в том что таблица создалась и данные сели в нее: ``SELECT * FROM example_table;``
  
      <img src="https://github.com/user-attachments/assets/cb60608c-0539-44c5-b292-a353741cc81a" alt="drawing" width="500"/>

**Готово! И на этом домашнее задание выполнено!**
