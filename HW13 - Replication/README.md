# Домашнее задание - Репликация (Виды и устройство репликации в PostgreSQL.)

## Подготовка тестовой среды (Все шаги аналогичны предыдущим ДЗ)

  1) Создаю Виртуальную машину и накатываю на нее ОС Manjaro Linux (Подробнее в ДЗ1)
  2) Загружаю на нее PostgreSQL и DBeaver (Подробнее в ДЗ2 и ДЗ3)
  3) Настраиваю pg_hba.conf и postgresql.conf для подключения извне (Подробнее в ДЗ2)
  4) Клонирую эту ВМ еще два раза, чтобы получилось в общем 3 виртуальынх машины 

     <img src="https://github.com/user-attachments/assets/4f262298-0d4d-490e-804c-e55d3f8b3c60" alt="drawing" width="500"/>

  5) Узнаю IP каждой машины при помощи команды ``ip addr show``
     Получается что IP у ВМ1: 192.168.1.86;
     У ВМ2: 192.168.1.55;
     И у ВМ3: 192.168.1.40
  7) Проверяю что машины видят другдруга при помощи утилиты ``ping``

     ВМ1:
     
     <img src="https://github.com/user-attachments/assets/97449af4-3e57-4107-82db-f306c09dcb97" alt="drawing" width="500"/>
     
     ВМ2:
     
     <img src="https://github.com/user-attachments/assets/bc6e2921-84cf-419c-9cfa-453c735c547c" alt="drawing" width="500"/>
     
     ВМ3:
     
     <img src="https://github.com/user-attachments/assets/8907c358-dbad-477a-95e9-8fc19f0907f5" alt="drawing" width="500"/>

  9) Убеждаюсь что на каждой ВМ в настройках постгреса установлены следующие параметры ``wal_level = logical``, ``max_wal_senders = 10``, ``max_replication_slots = 10``

     <img src="https://github.com/user-attachments/assets/e118fecb-3e49-41ad-918b-952c4d83916a" alt="drawing" width="500"/>

## Настройка таблиц и публикаций

  1) Создание таблиц для чтения и записи на ВМ1 (192.168.1.86): ``CREATE TABLE test (id SERIAL PRIMARY KEY, value TEXT);``, ``CREATE TABLE test2 (id SERIAL PRIMARY KEY, value TEXT);``
  2) Создание публикации для таблицы test на ВМ1: ``CREATE PUBLICATION pub_test FOR TABLE test;``
  3) Создание таблиц для чтения и записи на ВМ2 (192.168.1.55): ``CREATE TABLE test (id SERIAL PRIMARY KEY, value TEXT);``, ``CREATE TABLE test2 (id SERIAL PRIMARY KEY, value TEXT);``
  4) Создание публикации для таблицы test на ВМ2: ``CREATE PUBLICATION pub_test2 FOR TABLE test2;``
  5) Создание подписки на публикацию для таблицы test2 на ВМ1: ``CREATE SUBSCRIPTION sub_test2 CONNECTION 'host=192.168.1.55 dbname=postgres user=postgres password=1234' PUBLICATION pub_test2;``
  6) Создание подписки на публикацию для таблицы test2 на ВМ2: ``CREATE SUBSCRIPTION sub_test1 CONNECTION 'host=192.168.1.86 dbname=postgres user=postgres password=1234' PUBLICATION pub_test;``
  7) Создание  таблиц для чтения на ВМ3 (192.168.1.40): ``CREATE TABLE test (id SERIAL PRIMARY KEY, value TEXT);``, ``CREATE TABLE test2 (id SERIAL PRIMARY KEY, value TEXT);``
  8) Создание подписки на публикации на ВМ3: ``CREATE SUBSCRIPTION sub_from_vm1 CONNECTION 'host=192.168.1.86 dbname=postgres user=postgres password=1234' PUBLICATION pub_test;``
``CREATE SUBSCRIPTION sub_from_vm2 CONNECTION 'host=192.168.1.55 dbname=postgres user=postgres password=1234' PUBLICATION pub_test2;``

## Проверка репликации

  1) На ВМ1 создаю запись в таблицу test: ``INSERT INTO test (value) VALUES ('Hello from VM1');``
  2) На ВМ2 создаю запись в таблицу test2: ``INSERT INTO test2 (value) VALUES ('Hello from VM2');``
  3) Проверяю данные в таблице test2 на ВМ1: ``select * from test2;``

     <img src="https://github.com/user-attachments/assets/c653e55b-b795-495f-8a55-198cc0ffee2a" alt="drawing" width="500"/>

  4) Проверяю данные в таблице test на ВМ2: ``select * from test;``

     <img src="https://github.com/user-attachments/assets/a754d86c-7150-4f01-bc78-feccbbf1a45d" alt="drawing" width="500"/>

  5) Проверяю данные в таблицах test и test2 на ВМ3:

     <img src="https://github.com/user-attachments/assets/2800b856-2899-4874-86f6-0e7e5ea647b3" alt="drawing" width="500"/>

## Задание со *.

Задание со * я, к сожалению, сделать не могу, так как не имею достаточных ресурсов для одновременного запуска сразу четырех виртуальных машин. Даже запуск этих трех машин мне дался с горем пополам( 

Хотя в теории это сделать не сложно. Для этого на ВМ3 в файе postgresql.conf нужно утсановить параметр ``wal_level = replica``, и создать слот репликации ``SELECT * FROM pg_create_physical_replication_slot('vm4_replica_slot');``.

А на ВМ4 в файе postgresql.conf установить параметры ``hot_standby = on``
``primary_conninfo = 'host=192.168.1.40 port=5432 user=пользователь_для_реплики password=пароль'``
``primary_slot_name = 'vm4_replica_slot'``

И восстановить постгрес на ВМ4 с ВМ3: ``pg_basebackup -h 192.168.1.40 -D /mnt/data/postgresql/14/data/ -U репликация_пользователь -Fp -Xs -P -R``

**Готово! И на этом домашнее задание выполнено!**
