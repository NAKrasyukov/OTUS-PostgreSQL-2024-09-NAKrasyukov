# Домашнее задание - Установка и настройка PostgteSQL в контейнере Docker

## Создание виртуальной машины в VirtualBox

  1) Скачиваю образ дистрибутива Linux с Официального сайта
  2) Создаю новую вм со стандартными настройками:
     
     <img src="https://github.com/user-attachments/assets/5e1d6cd0-f5fb-49e8-b9b5-cd742df7c2ea" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/15b0e3c6-3b76-4924-93be-17608f1566a5" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/c9523dd1-5490-4c54-a784-4e50908c7899" alt="drawing" width="500"/>

  3) Запускаю ВМ и устанавливаю ОС

     <img src="https://github.com/user-attachments/assets/4219c519-c9b7-4beb-898b-fc67e6145904" alt="drawing" width="500"/>

## Работа с Postgres и Docker

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

9) Создаю и подключаюсь к контейнеру с клиентом postgres ``sudo docker run -it --rm --name postgres_client --link postgres_server:postgres postgres psql -h postgres -U postgres``
10) Создаю тестовую таблицу и заполняю данными:
    
    ``CREATE TABLE test (id SERIAL PRIMARY KEY, name TEXT);``

    ``INSERT INTO test (name) VALUES ('Vasiliy'), ('Anton'), ('Egor');``

    ``SELECT * FROM test;``

    <img src="https://github.com/user-attachments/assets/842577da-b2d4-4f0a-a2a9-8eb79721ddc3" alt="drawing" width="500"/>

11) Подключение к Postgres извне ВМ:
    Для этого меняю конфигурацию pg_hba.conf, добавляю строку ``host all all 0.0.0.0/0 md5``. Тк в моем докер имадже нету никакого текстового редактора (vi, vim, nano и ed). Делаю это следующим образом:
    Копирую файл из докера на ВМ ``sudo docker cp postgres_server:/var/lib/postgresql/data/pg_hba.conf ./pg_hba.conf``, меняю права доступа ``sudo chmod 777 /home/nakrasyukov/postgresql.conf``, добавляю необходимые параметры, восстанавливаю права доступа обратно ``sudo chmod 700 /home/nakrasyukov/postgresql.conf`` и копирую файл обратно в контейнер докера ``sudo docker ./pg_hba.conf cp postgres_server:/var/lib/postgresql/data/pg_hba.conf``.
    Убеждаюсь что посгрес слушает все адреса ``sudo docker exec -it postgres_server cat /var/lib/postgresql/data/postgresql.conf | grep listen_addresses``

    <img src="https://github.com/user-attachments/assets/70eb7ce8-a145-4870-9c88-46b019413f01" alt="drawing" width="500"/>

    Для безопастности и для того чтобы не открывать порты и не настраивать фаерволл, в настройках ВМ выставляю сетевой адаптер "Сетевой мост" вместо "NAT", тем самым открывая доступ по портам только для локальной сети.
    Перезапускаю ВМ и прописываю ``ip addr show`` чтобы узнать адрес виртуальной машины в локальной сети.

    <img src="https://github.com/user-attachments/assets/1604ca67-4603-4adb-8a0f-a66bc24ed298" alt="drawing" width="500"/>

    На своей хост машине на Windows создаю подключение в DBeaver с указанием этого ip адреса, портом 5432 и паролем от postgres который задавал при создании контейнера.

    <img src="https://github.com/user-attachments/assets/6528aeb0-eb06-4c00-b070-e5fda6bd7009" alt="drawing" width="500"/>

    Тест соединения проходит. Можно сделать выборку из тестовой таблицы, которую создал ранее ``select * from test``:

    <img src="https://github.com/user-attachments/assets/701f858b-6f1c-4efa-b8b5-c0dac56db410" alt="drawing" width="500"/>

    12) Удаляю контейнер с сервером ``sudo docker stop postgres_server`` ``sudo docker rm postgres_server``
    13) Создаю контейнер с сервером заново ``sudo docker run -d --name postgres_server -e POSTGRES_PASSWORD=yourpassword -v /var/lib/postgres_new:/var/lib/postgresql/data -p 5432:5432 postgres``
    14) Поднимаю контейнер с клиентом ``sudo docker run -it --rm --name postgres_client --link postgres_server:postgres postgres psql -h postgres -U postgres`` и проверяю данные в тестововй таблице ``select * from test``

    <img src="https://github.com/user-attachments/assets/f8985bfc-bb3e-414e-a929-bd383f7b07b7" alt="drawing" width="500"/>

    **Все работает! И на этом домашнее задание выполнено!**
