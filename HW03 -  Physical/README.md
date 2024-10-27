# Домашнее задание - Физический уровень PostgreSQL (Установка и настройка PostgreSQL)

## Создание виртуальной машины в VirtualBox

  1) Скачиваю образ дистрибутива Linux с Официального сайта
  2) Создаю новую вм со стандартными настройками:
     
     <img src="https://github.com/user-attachments/assets/5e1d6cd0-f5fb-49e8-b9b5-cd742df7c2ea" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/15b0e3c6-3b76-4924-93be-17608f1566a5" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/c9523dd1-5490-4c54-a784-4e50908c7899" alt="drawing" width="500"/>

  3) Запускаю ВМ и устанавливаю ОС

     <img src="https://github.com/user-attachments/assets/4219c519-c9b7-4beb-898b-fc67e6145904" alt="drawing" width="500"/>

## Установка Postgres

  1) Устанавливаю Postgres на ВМ: ``sudo pacman -S postgresql``
  2) Инициализирую кластер базы данных: ``sudo -u postgres initdb --locale=en_US.UTF-8 -D /var/lib/postgres/data``
  3) Запускаю PostgreSQL: ``sudo systemctl start postgresql``
  4) Проверяю статус кластера: ``sudo -u postgres pg_ctl status -D /var/lib/postgres/data``

     <img src="https://github.com/user-attachments/assets/4fb9418a-c40f-471f-8b06-e2ca6452e5f2" alt="drawing" width="500"/>

  5) Переключаюсь на пользователя postgres: ``sudo -i -u postgres``
  6) Запускаю ``psql``, создаю тестовую базу ``CREATE TABLE test(c1 TEXT);``, записываю в нее тестовые данные ``INSERT INTO test VALUES('1');`` и проверяю содержимое ``SELECT * FROM test;``:

     <img src="https://github.com/user-attachments/assets/6b4b39f7-b04a-4211-98a0-b16a1bc58d78" alt="drawing" width="500"/>
     
  7) Остановка кластера PostgreSQL: ``sudo systemctl stop postgresql``

## Создание и подключение нового диска

  1) Выключаю ВМ.
  2) В VirtualBox, в настройках ВМ, в разделе "Носители" добавляю новый виртуальный диск размером 10 ГБ:

     <img src="https://github.com/user-attachments/assets/ae04f24f-a311-408e-95f3-ff2aff3e7323" alt="drawing" width="500"/>

     <img src="https://github.com/user-attachments/assets/e1e71383-835c-4be6-ba7d-240adfb5457a" alt="drawing" width="500"/>
     
  3) Запускаю ВМ и нахожу новый диск: ``lsblk``:

     <img src="https://github.com/user-attachments/assets/26f77c36-7f50-4e34-bb44-6e05d665d5c0" alt="drawing" width="500"/>

  4) Размечаю диск при помощи утилиты ``fdisk``:

     <img src="https://github.com/user-attachments/assets/6d3ed55c-a835-4f1c-9f4e-168b3f1d71ce" alt="drawing" width="500"/>

  5) Задаю файловую систему: ``sudo mkfs.ext4 /dev/sdb1``

     <img src="https://github.com/user-attachments/assets/73f81add-34ef-48a0-b735-1258ff2d94a8" alt="drawing" width="500"/>

  6) Создаю точку монтирования ``sudo mkdir -p /mnt/data`` и монтирую в нее диск ``sudo mount /dev/sdb1 /mnt/data``
  7) Для автоматического монтирования добавляю запись в fstab: ``echo '/dev/sdb1 /mnt/data ext4 defaults 0 2' | sudo tee -a /etc/fstab``
  8) Перезагружаю ВМ
  9) Проверяю что диск остался примонтированным: ``df -h | grep /mnt/data``

      <img src="https://github.com/user-attachments/assets/fe52dc24-22f9-490c-9716-651309c0ec52" alt="drawing" width="500"/>

## Перемещение данных Postgres

  1) Делаю postgres владельцем каталога: ``sudo chown -R postgres:postgres /mnt/data``
  2) Перемещаю данные PostgreSQL: ``sudo mv /var/lib/postgres /mnt/data``
  3) Пытаюсь запустить сервис PostgreSQL: ``sudo systemctl start postgresql``
  4) Запустит сервис не удалось. Все потому что в конфигурации сервиса все еще указан старый каталог ``PGROOT``
  5) Меняю в ``/usr/lib/systemd/system/postgresql.service`` каталоги для ``PGROOT`` и ``PIDFile``

     <img src="https://github.com/user-attachments/assets/8a0fcb14-fa63-48de-a4a0-685cd24dd209" alt="drawing" width="500"/>

  6) Перезагружаю сессию и пробую запустить сервис: ``sudo systemctl start postgresql``. Теперь сервис запускается.
  7) Запускаю ``sudo -u postgres psql`` и делаю выборку из тестовой таблицы ``select * from test;``:

     <img src="https://github.com/user-attachments/assets/d7ea9dc6-09d5-4ab1-afc2-27c61a226bec" alt="drawing" width="500"/>

  Данные отображаются корректно!
  Примечание: мне не пришлось менять файл конфигурации ``postgresql.conf`` так как при установке он встал в ``/var/lib/postgres/data/postgresql.conf`` (возможно зависит от версии Postgres или пакетного менеджера) и переменная ``data_directory`` в нем указывала на текущий каталог, который был польностью перенесен на новый жесткий диск. 
## Задание со звездочкой *

  1) Создаю новую виртуалную машину аналогично первой
  2) Устанавливаю Postgres на ВМ: ``sudo pacman -S postgresql``
  3) Инициализирую кластер базы данных в стандартном каталоге: ``sudo -u postgres initdb --locale=en_US.UTF-8 -D /var/lib/postgres/data``
  4) Удаляю каталог с данными: ``sudo rm -rf /var/lib/postgres``
  5) Выключаю ВМ
  6) Подключаю ранее созданный жесткий диск к этой ВМ (Настройки ВМ -> Носители -> Контроллер SATA -> Добавить жесткий диск, выбираю ранее созданный жесткий диск):

     <img src="https://github.com/user-attachments/assets/4906a88b-2790-4e82-bd4c-ec66a87de562" alt="drawing" width="500"/>

  7) Запускаю ВМ, создаю точку монтирования ``sudo mkdir -p /mnt/data`` и монтирую в нее диск ``sudo mount /dev/sdb1 /mnt/data``, для автоматического монтирования добавляю запись в fstab: ``echo '/dev/sdb1 /mnt/data ext4 defaults 0 2' | sudo tee -a /etc/fstab``
  8) Делаю postgres владельцем каталога: ``sudo chown -R postgres:postgres /mnt/data``
  9) Меняю в ``/usr/lib/systemd/system/postgresql.service`` каталоги для ``PGROOT`` и ``PIDFile``
  10) Перезагружаю ВМ и пытаюсь запустить сервис ``sudo systemctl start postgresql``
  11) Запускаю ``sudo -u postgres psql`` и делаю выборку из тестовой таблицы ``select * from test;``:

      <img src="https://github.com/user-attachments/assets/706f6f4c-e14c-45ad-8111-3269d89a32fd" alt="drawing" width="500"/>

**Готово! И на этом домашнее задание выполнено!**
