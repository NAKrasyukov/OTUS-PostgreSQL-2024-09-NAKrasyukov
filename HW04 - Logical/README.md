# Домашнее задание - Работа с базами данных, пользователями и правами (Логический уровень PostgreSQL)

## Установка Postgres

  1) Устанавливаю Postgres на ВМ: ``sudo pacman -S postgresql``
  2) Инициализирую кластер базы данных: ``sudo -u postgres initdb --locale=en_US.UTF-8 -D /var/lib/postgres/data``
  3) Запускаю PostgreSQL: ``sudo systemctl start postgresql``
  4) Проверяю статус кластера: ``sudo -u postgres pg_ctl status -D /var/lib/postgres/data``

     <img src="https://github.com/user-attachments/assets/4fb9418a-c40f-471f-8b06-e2ca6452e5f2" alt="drawing" width="500"/>

## Работа с Postgres

  1) Подключаюсь через psql под пользователем postgres: ``sudo -iu postgres psql``
  2) Создаю базу данных testdb ``CREATE DATABASE testdb;`` и подключаюсь к ней ``\c testdb``
  3) Создаю схему и таблицу в ней ``CREATE SCHEMA testnm;`` ``CREATE TABLE testnm.t1 (c1 INTEGER);``, а затем вставляю значение в таблицу ``INSERT INTO testnm.t1 (c1) VALUES (1);``
  4) Создаю роль "readonly" и выдаю права доступа:
     
     ```
     CREATE ROLE readonly;
     GRANT CONNECT ON DATABASE testdb TO readonly;
     GRANT USAGE ON SCHEMA testnm TO readonly;
     GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
     ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES TO readonly; 
     ```

  5) Создаю пользователя testread ``CREATE USER testread WITH PASSWORD 'test123';`` и назначаю ему роль readonly ``GRANT readonly TO testread;``
  6) Выхожу из psql и вхожу снова под пользователем testread: ``psql -U testread -d testdb``
  7) Делаю ``SELECT * FROM testnm.t1;``:

     <img src="https://github.com/user-attachments/assets/a8fddfc6-c6ad-4251-ba3f-eae8615b88da" alt="drawing" width="500"/>
     
  8) Все получилось. Проблемы могли возникнуть, если бы я не указал схему "testnm" при создании таблицы. Если схема не указана, то PostgreSQL создаст таблицу в схеме, указанной в параметре search_path, который по умолчанию равен "public".
  9)  Попробую создать новую таблицу: ``create table t2(c1 integer);``

      <img src="https://github.com/user-attachments/assets/83b6cb93-6070-41ed-ac15-c2df57c766b7" alt="drawing" width="500"/>

      Ничего не получилось. Так как у пользователя нет на это прав. Это можно проверить командой ``\du testread``

      <img src="https://github.com/user-attachments/assets/b820e63a-836f-4b22-bff7-9279e25b6aba" alt="drawing" width="500"/>
      
      Хоть в этой базе данных и есть схема public, у testread нет к ней доступа, тк он не имеет соответсвующей роли.

**Готово! И на этом домашнее задание выполнено!**
