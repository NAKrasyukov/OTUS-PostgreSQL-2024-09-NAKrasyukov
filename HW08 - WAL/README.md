# Домашнее задание - Работа с журналами (Журналы)

## Настройка сервера

  1) Настраиваю выполнение контрольной точки раз в 30 секунд - ``nano /var/lib/postgres/data/postgresql.conf`` ``checkpoint_timeout = 30s`` ``log_checkpoints = on``

      <img src="https://github.com/user-attachments/assets/e78fb002-5884-48ec-8d1f-850f7acd0aad" alt="drawing" width="500"/>

  2) Перезапускаю PostgreSQL для применения настроек ``sudo systemctl restart postgresq``

## 10 минут подавайте нагрузку утилитой pgbench

  1) Инициализирую тестовую базу данных - ``pgbench -i postgres``
  2) Запуск теста на 10 минут - ``pgbench -c 4 -j 2 -T 600 postgres``

     <img src="https://github.com/user-attachments/assets/7cd243f9-5cf8-4780-baf2-225051d642e9" alt="drawing" width="500"/>

## Измерьте объем сгенерированных журнальных файлов

  1) Смотрю общий обьем WAL (80мб)
     
     <img src="https://github.com/user-attachments/assets/b41d462a-afd5-4b54-931e-7a87f34d8721" alt="drawing" width="500"/>

  2) Количество контрольных точек за 10 минут (20) -``количество_контрольных_точек = 10 минут / 30 секунд = 20``
  3) Средний объем на одну контрольную точку (4мб) - ``средний_объем = общий_объем_WAL / количество_контрольных_точек`` => `` 80/20 = 4``

## Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию.

  1) Проверяю записи в журнале сервиса PostgreSQL - ``journalctl -xeu postgresql.service``

     <img src="https://github.com/user-attachments/assets/0ba8509f-9a4b-4fbc-a9af-065dc96e2906" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/b9909392-f44e-4a50-bb65-7a20c49338cf" alt="drawing" width="500"/>

     У меня получилось так, что контрольные точки создавались примерно за 27 секунд, при этом между завершением предыдущей контрольной точки и началом следующей есть интервал примерно в 3 секунды.
     Судя по всему это нормальное поведение PostgreSQL. Контрольная точка включает запись большого количества данных в ``pg_data``. Время завершения зависит от размера данных и скорости диска. PostgreSQL старается растянуть процесс записи контрольной точки по времени, чтобы избежать резкого пикового использования диска. 
     В настройках PostgreSQL есть параметр ``checkpoint_completion_target`` Значение по умолчанию: 0.9. Это означает, что процесс контрольной точки должен завершиться к 90% времени следующей точки (как раз создает интервал в 3 секунды для моего случая). 

## Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

  1) По стандарту постгрес работает в синхронном режиме, тест в этом режиме был проведен выше (tps = 805). Отключаю синхронный режим ``synchronous_commit = off``

     <img src="https://github.com/user-attachments/assets/3407865b-9982-4803-bbe4-f60222ad6620" alt="drawing" width="500"/>

  2) Перезапускаю ``sudo systemctl restart postgresq`` и запускаю тест заново ``pgbench -i postgres`` ``pgbench -c 4 -j 2 -T 600 postgres``

     <img src="https://github.com/user-attachments/assets/51cbc1d1-5a59-4d07-adf7-7cf38786fd60" alt="drawing" width="500"/>

  В асинхронном режиме TPS значительно выше, так как транзакции не дожидаются подтверждения записи WAL на диск.

## Изменение байт в таблице и выборка из этой таблицы

  1) Отанавливаю кластер ``sudo systemctl stop postgresql``
  2) Удалаю имеющийся кластер ``rm -rf /var/lib/postgres/data/*`` и создаю на его месте новый с контрольной суммой страниц ``initdb -D /var/lib/postgres/data/ --data-checksums``
  3) Запускаю кластер ``sudo systemctl start postgresql``
  4) Создаю тестовую таблицу и вношу несколько строк - ``CREATE TABLE test_table (id SERIAL, data TEXT);`` ``INSERT INTO test_table (data) VALUES ('test1'), ('test2');``

     <img src="https://github.com/user-attachments/assets/b6375a48-f6f7-4a62-9038-c024843d8dd3" alt="drawing" width="500"/>

  5) Определяю OID этой таблицы ``SELECT oid FROM pg_class WHERE relname = 'test_table';`` (16389)
  6) Отанавливаю кластер ``sudo systemctl stop postgresql``
  7) Перехожу в каталог ``/var/lib/postgres/data/base/5/`` и открываю файл таблицы ``16389``

     <img src="https://github.com/user-attachments/assets/48eafe5c-1123-40c3-96e8-76c4d4f428f2" alt="drawing" width="500"/>

  8) Меняю несколько символов в записи ``test1`` => ``tesT3`` и сохраняю файл

     <img src="https://github.com/user-attachments/assets/25248a6c-7d94-4154-9921-7c51c853e1dc" alt="drawing" width="500"/>

  9) Запускаю кластер ``sudo systemctl start postgresql``
  10) Делаю выборку, и возникает ошибка 

      <img src="https://github.com/user-attachments/assets/0b1a5f36-81cb-4a79-a00f-83577087a21b" alt="drawing" width="500"/>

  11) Для игнорирования ошибки можно изменить параметр ``SET zero_damaged_pages = on;``, после этого делаю выборку, но испорченные данные в таблице к сожалению или к стчастью уже не отображаются

      <img src="https://github.com/user-attachments/assets/726d37b0-619d-47e4-8831-6e30b10b8def" alt="drawing" width="500"/>

  В теории до сделующего перезапуска сервера (а точнее до перезаписи этой таблицы) можно попробовать сделать дамп файла поврежденной таблицы, чтобы попытаться хоть как-то восстановить поврежденные данные. 

  **Готово! И на этом домашнее задание выполнено!**

  ## Примечание 

  В процессе работы, для удобства, научился открывать программы с графическим интерфейсом от имени пользователя postgres. Для этого установил утилиту конфигурации сервера графического интерфейса Xorg ``sudo pacman -S xorg-xhost``, выдал права на использование Xorg для пользователя postgres ``xhost +SI:localuser:postgres`` запустил зашел под ним в терминале ``sudo su postgres`` и установил переменную среды указывающую на неодходимый дисплей ``export DISPLAY=:0``. Теперь можно из этой же сесии терминала запускать программы с графическим интерфейсом от имени пользователя postgres, например файловый менеджер ``dolphin`` или текстовый редактор ``kate``. Тоже интересный опыт однако) (хоть и бесполезный в реальной серверной среде)
