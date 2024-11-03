# Домашнее задание - Нагрузочное тестирование и тюнинг PostgreSQL (Настройка PostgreSQL)

## Создание виртуальной машины в VirtualBox

  1) Скачиваю образ дистрибутива Linux с Официального сайта
  2) Создаю новую вм со стандартными настройками:
     
     <img src="https://github.com/user-attachments/assets/5e1d6cd0-f5fb-49e8-b9b5-cd742df7c2ea" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/15b0e3c6-3b76-4924-93be-17608f1566a5" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/c9523dd1-5490-4c54-a784-4e50908c7899" alt="drawing" width="500"/>

  3) Запускаю ВМ и устанавливаю ОС

     <img src="https://github.com/user-attachments/assets/4219c519-c9b7-4beb-898b-fc67e6145904" alt="drawing" width="500"/>

  Примечание: после создания вм я добавил ядер (до 5) и памяти (до 10968)

  <img src="https://github.com/user-attachments/assets/720fa0ec-1771-46a9-9f91-2b1fd95d59ab" alt="drawing" width="500"/>
   
## Установка Postgres

  1) Устанавливаю Postgres на ВМ: ``sudo pacman -S postgresql``
  2) Инициализирую кластер базы данных: ``sudo -u postgres initdb --locale=en_US.UTF-8 -D /var/lib/postgres/data``
  3) Запускаю PostgreSQL: ``sudo systemctl start postgresql``
  4) Проверяю статус кластера: ``sudo -u postgres pg_ctl status -D /var/lib/postgres/data``

     <img src="https://github.com/user-attachments/assets/4fb9418a-c40f-471f-8b06-e2ca6452e5f2" alt="drawing" width="500"/>

## Первый запуск pgbench со стандартными настройками

  1) Переключаюсь на пользователя постгрес: ``sudo -iu postgres``
  2) Задаю базу постгрес для тестирования в пгбенч: ``pgbench -i postgres``
  3) Запускаю тест: ``pgbench -c 50 -j 2 -P 10 -T 60 postgres``

     <img src="https://github.com/user-attachments/assets/2d00a028-cbb0-43d4-83bb-b910d7ac4088" alt="drawing" width="500"/>

  4) TPS со стандартными настройками равен 796.945803

## Настройка Postgres

  Для подбора оптимальных настроек Postgres буду использовать сервис pgtune (https://pgtune.leopard.in.ua/)

  1) Указываю конфигурацию хоста Postgres и нажимаю "Generate":

     <img src="https://github.com/user-attachments/assets/297c233e-3d98-4146-8d8a-3f7be5fbf78e" alt="drawing" width="500"/>

  2) Сервис предложил следующие параметры конфигурации:

     <img src="https://github.com/user-attachments/assets/0eab9ea7-3bc8-465f-b185-5d6b121ae9d0" alt="drawing" width="500"/>

  3) Открываю файл конфигурации постгрес: ``nano /var/lib/postgres/data/postgresql.conf``
  4) Вставляю предложенные сервисом настройки в конец файла:

     <img src="https://github.com/user-attachments/assets/a7b4af2e-77a2-43d5-90c0-09840dc77a0e" alt="drawing" width="500"/>

     Так как домашнее задание допускает пренебрежение надежностью кластера постгрес я добавил параметры ``fsync = off`` и ``synchronous_commit = off`` которые отключают запись данных на диск до завершения транзакции и запись транзакций в журнал (WAL) перед подтверждением соответсвенно.

  5) Сохраняю файл и перезапускаю кластер.

## Второй запуск pgbench с модифицированными настройками

  1) Переключаюсь на пользователя постгрес: ``sudo -iu postgres``
  2) Задаю базу постгрес для тестирования в пгбенч: ``pgbench -i postgres``
  3) Запускаю тест: ``pgbench -c 50 -j 2 -P 10 -T 60 postgres``

     <img src="https://github.com/user-attachments/assets/33d7fb0c-8ea8-440a-bfb6-251371aabe4c" alt="drawing" width="500"/>

  4) Теперь значение TPS составляет 2038.511413. После применения данных настроек удалось достич значительного прироста в производительности кластера постгрес. Эффективность выросла на 155.79% по сравнению со стандартными настройками.

**Готово! И на этом домашнее задание выполнено!**
