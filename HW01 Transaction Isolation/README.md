# Домашнее задание - Работа с уровнями изоляции транзакции в PostgreSQL

## Создание виртуальной машины в VirtualBox

  1) Скачиваю образ дистрибутива Linux с Официального сайта
  2) Создаю новую вм со стандартными настройками:
     
     <img src="https://github.com/user-attachments/assets/5e1d6cd0-f5fb-49e8-b9b5-cd742df7c2ea" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/15b0e3c6-3b76-4924-93be-17608f1566a5" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/c9523dd1-5490-4c54-a784-4e50908c7899" alt="drawing" width="500"/>

  3) Запускаю ВМ и устанавливаю ОС

     <img src="https://github.com/user-attachments/assets/4219c519-c9b7-4beb-898b-fc67e6145904" alt="drawing" width="500"/>
     
## Установка Postgres

  1) Обновил базы данных пакетов: ``sudo pacman -Syu``
  2) Устанавливаю Докер: ``sudo pacman -S docker``
  3) Запускаю сервис Докера ``sudo systemctl start docker`` ``sudo systemctl enable docker``
  4) Проверил что сервис докера встал ``sudo docker ps``

     <img src="https://github.com/user-attachments/assets/fa432667-3c86-4acf-9a0d-9cd0852c5ce6" alt="drawing" width="500"/>

  5) Создаю каталог ``sudo mkdir /var/lib/postgresql``
  6) Создаю новый каталог для хранения данных ``sudo mkdir /var/lib/postgres_new``
  7) Создаю контейнер с сервером postgers ``sudo docker run -d --name postgres_server -e POSTGRES_PASSWORD=yourpassword -v /var/lib/postgres_new:/var/lib/postgresql/data -p 5432:5432 postgres``
  8) Проверяю создание контейнера ``sudo docker ps``

     <img src="https://github.com/user-attachments/assets/eaa5900c-03ba-4e41-b20f-c5dd6d9b61e0" alt="drawing" width="500"/>

## Подключение и работа с Postgres

  Для удобства работы буду использовать DBeaver. 

  1) Выключаю auto commit

     <img src="https://github.com/user-attachments/assets/b81b7ae8-482f-48a7-bf5d-05169d2e914f" alt="drawing" width="500"/>

  2) Создаю новую таблицу и наполняю её данными. В первой сессии выполняю следующие команды:
     ```
     CREATE TABLE persons (id SERIAL PRIMARY KEY, first_name TEXT, second_name TEXT);
     INSERT INTO persons (first_name, second_name) VALUES ('ivan', 'ivanov');
     INSERT INTO persons (first_name, second_name) VALUES ('petr', 'petrov');
     COMMIT;
     ```

     <img src="https://github.com/user-attachments/assets/ad4a8b97-400a-4430-b2a7-e95946b4261a" alt="drawing" width="500"/>

  2) Проверяю текущий уровень изоляции. Выполняю команду ``SHOW TRANSACTION ISOLATION LEVEL;``
     
     <img src="https://github.com/user-attachments/assets/62ad1952-5a55-405e-9bc6-48ef95407084" alt="drawing" width="500"/>
     
  5) Начинаю новую транзакцию в обоих сессиях. В обеих сессиях выполняю: ``BEGIN;``
  6) Добавить новую запись в первой сессии. Выполняю в первой сессии: ``INSERT INTO persons (first_name, second_name) VALUES ('sergey', 'sergeev');``

     <img src="https://github.com/user-attachments/assets/7f1be640-3525-4234-b1da-374bf5f1ed15" alt="drawing" width="500"/>

  8) Сделать SELECT из второй сессии. Выполняю в второй сессии: ``SELECT * FROM persons;``

     <img src="https://github.com/user-attachments/assets/f78a69a9-a393-4a60-9209-390f504fc340" alt="drawing" width="500"/>

  9) Я не вижу новую запись ('sergey', 'sergeev') во второй сессии, так как транзакция в первой сессии ещё не завершена (не сделали COMMIT). Во второй сессии видны только те изменения, которые были зафиксированы (committed).
  10) Завершить первую транзакцию. В первой сессии выполняю: ``COMMIT;``

      <img src="https://github.com/user-attachments/assets/962cc266-ae19-4a9f-8e71-aafbac5c2fb6" alt="drawing" width="500"/>

  11) Сделать SELECT из второй сессии. Выполняю в второй сессии: ``SELECT * FROM persons;``

      <img src="https://github.com/user-attachments/assets/07570425-063b-4dbd-8920-b54748f5534b" alt="drawing" width="500"/>

  12) Теперь запись ('sergey', 'sergeev') отображается, так как первая транзакция была завершена, и изменения были зафиксированы.
  13) Завершить транзакцию во второй сессии. В этой сессии выполняю: ``COMMIT;``
  14) Устанавливаю уровень изоляции "repeatable read" и начинаю новые транзакции в сессиях: ``BEGIN;``

      <img src="https://github.com/user-attachments/assets/dd209c7d-480c-40b9-a803-07d2fdee3926" alt="drawing" width="500"/>

  15) Добавить новую запись в первой сессии: Выполняю в первой сессии: ``INSERT INTO persons (first_name, second_name) VALUES ('sveta', 'svetova');``

      <img src="https://github.com/user-attachments/assets/1ca1c398-80e2-4c36-b50f-cd1648227c80" alt="drawing" width="500"/>

  16) Сделать SELECT из второй сессии: Выполняю в второй сессии: ``SELECT * FROM persons;``

      <img src="https://github.com/user-attachments/assets/894a7e1a-5339-4f55-978e-c9c0e747274d" alt="drawing" width="500"/>

  17) Новая запись ('sveta', 'svetova') не отображается во второй сессии, так как уровень изоляции repeatable read предотвращает отображение изменений, сделанных другими транзакциями, до тех пор, пока транзакция не будет завершена.
  18) Завершить первую транзакцию: В первой сессии выполняю: ``COMMIT;``
  19) Сделать SELECT из второй сессии: Выполняю в второй сессии: ``SELECT * FROM persons;``

      <img src="https://github.com/user-attachments/assets/c6ef57e8-af1b-47ec-864b-bf2e5f199a85" alt="drawing" width="500"/>

  20) Новой записи ('sveta', 'svetova') все еще не видно. Потому что при уровне изоляции "repeatable read", если вторая сессия начала транзакцию и сделала SELECT, она "замораживает" состояние данных на момент начала транзакции. То есть даже если первая сессия завершит свою транзакцию и зафиксирует изменения (вставку записи 'sveta', 'svetova'), вторая сессия не увидит этих изменений, пока не завершит свою транзакцию.
  21) Завершить вторую транзакцию: В этой сессии выполняю: ``COMMIT;``
  22) Сделать SELECT * из второй сессии: Выполняю в второй сессии: ``SELECT * FROM persons;``

      <img src="https://github.com/user-attachments/assets/c7d12eb2-c434-4721-85c6-6b687dfcd35d" alt="drawing" width="500"/>

  23) Теперь новая запись ('sveta', 'svetova') отображается, так как транзакция завершена и вторая сессия обновила своё состояние.

**Готово! И на этом домашнее задание выполнено!**

