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

    


