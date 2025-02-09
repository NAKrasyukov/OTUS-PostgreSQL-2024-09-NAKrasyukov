# Проектная работа по курсу - Оптимизация настроек кластера, структуры БД и запросов, для повышения производительности при работе с большими данными.

## Введение

Современные базы данных играют ключевую роль в обработке и анализе больших объемов информации. Одной из наиболее популярных систем управления базами данных (СУБД) является PostgreSQL, известная своей расширяемостью, мощными возможностями индексирования и поддержкой сложных аналитических запросов. Однако производительность PostgreSQL по умолчанию не всегда является оптимальной и требует дополнительной настройки в зависимости от характеристик сервера и специфики нагрузки.

В данной работе рассматривается процесс оптимизации кластера PostgreSQL и запросов к нему на примере обработки данных из набора "Animes Dataset 2023" с платформы Kaggle. Данный дата-сет представляет собой выгрузку открытых данных с сайта myanimelist.net. Оптимизация включает в себя настройку параметров PostgreSQL, создание индексов, анализ и улучшение запросов, а также другие мероприятия, направленные на повышение производительности системы.

**Исходные данные загружаются в базу данных из трех CSV-файлов:**
- `anime-transformed-dataset-2023.csv` - 23748 записей. Представляет список всех анмие и информацию по ним. 
- `users-details-transformed-2023.csv` - 731282 записей. Представляет список всех пользователей и открытую информацию по ним.
- `users-scores-transformed-2023.csv`  - 23796586 записей. Является сводным списком, в котором предоставлена информация о том какой пользователь какому аниме какую оценку поставил.  

Анализ эффективности оптимизации проводится с использованием инструмента `EXPLAIN ANALYZE`, позволяющего оценить планы выполнения запросов и время их обработки. Основная цель работы — на практике продемонстрировать, как изменения в настройках кластера и структуры запросов влияют на производительность базы данных.

**Цели и задачи исследования**

Целью данной работы является изучение и практическое применение методов оптимизации PostgreSQL для повышения производительности обработки данных. Для достижения поставленной цели необходимо решить следующие задачи:
1. Развернуть кластер PostgreSQL на виртуальной машине с ОС Manjaro Linux в VirtualBox.
2. Загрузить в базу данных информацию из предоставленных CSV-файлов.
3. Разработать SQL-запросы для анализа данных.
4. Провести первоначальную оценку их производительности с помощью `EXPLAIN ANALYZE`.
5. Оптимизировать конфигурацию кластера PostgreSQL в соответствии с доступными ресурсами виртуальной машины.
6. Создать и настроить индексы для повышения эффективности выполнения запросов.
7. Проанализировать и оптимизировать сами SQL-запросы.
8. Провести повторное тестирование и сравнить результаты до и после оптимизации.

Результаты работы позволят наглядно продемонстрировать влияние различных мер оптимизации на скорость выполнения запросов, а также получить практический опыт настройки и улучшения производительности PostgreSQL.



## Шаг 1 - Подготовка виртуальной машины и кластера постгрес.

  1) Скачиваю образ дистрибутива Linux с Официального сайта
  2) Создаю новую вм со стандартными настройками:
     
     <img src="https://github.com/user-attachments/assets/5e1d6cd0-f5fb-49e8-b9b5-cd742df7c2ea" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/15b0e3c6-3b76-4924-93be-17608f1566a5" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/c9523dd1-5490-4c54-a784-4e50908c7899" alt="drawing" width="500"/>

  3) Запускаю ВМ и устанавливаю ОС

     <img src="https://github.com/user-attachments/assets/4219c519-c9b7-4beb-898b-fc67e6145904" alt="drawing" width="500"/>

  4) Устанавливаю Postgres на ВМ: ``sudo pacman -S postgresql``
  5) Инициализирую кластер базы данных: ``sudo -u postgres initdb --locale=en_US.UTF-8 -D /var/lib/postgres/data``
  6) Запускаю PostgreSQL: ``sudo systemctl start postgresql``
  7) Проверяю статус кластера: ``sudo -u postgres pg_ctl status -D /var/lib/postgres/data``

     <img src="https://github.com/user-attachments/assets/4fb9418a-c40f-471f-8b06-e2ca6452e5f2" alt="drawing" width="500"/>

## Шаг 2 - Импорт данных в PostgreSQL

  Данный дата-сет изначально создавался для обучения нейронных сетей, а потому структура данных не совсем подходит для работы с ними в БД, а также дата-сет содержит лишнюю (в рамках данной работы) информацию, такую как ссылки на изображения с обложками, подробное описание каждого сериала и др.
  По этой причине необходимо произвести нормализацию и очистку данных. Я решил сделать это на этапе импорта данных, чтобы не возвращаться к этому в дальнейшем. 
  **Перечень полей для импорта по каждому файлу: **

  - Файл: ``anime-transformed-dataset-2023.csv``; Столбцы: ``id,title,genres,type,episodes,status,producers,licensors,studios,source,duration,rating,rank,popularity,favorites,scored_by,members,is_hentai``.
  - Файл: ``users-details-transformed-2023.csv``; Столбцы: ``id,name,gender,joined,days_watched,mean_score,watching,completed,on_hold,dropped,plan_to_watch,total_entries,rewatched,episodes_watched``.
  - Файл: ``users-scores-transformed-2023.csv``;  Столбцы: ``user_id,anime_id,rating``.

  Перед началом импорта данных необходимо создать в БД подходящую структуру:

  1) Создание новой базы данных: ``CREATE DATABASE anime_db;``
  2) Создание таблицы ``anime``:
```
     CREATE TABLE anime (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    genres TEXT,
    type TEXT,
    episodes INT,
    status TEXT,
    producers TEXT,
    licensors TEXT,
    studios TEXT,
    source TEXT,
    duration TEXT,
    rating TEXT,
    rank INT,
    popularity INT,
    favorites INT,
    scored_by INT,
    members INT,
    is_hentai BOOLEAN
    );
```
  3) Создание таблицы ``users``:
```
     CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    gender TEXT,
    joined TIMESTAMP WITH TIME ZONE,
    days_watched NUMERIC,
    mean_score NUMERIC(4,2),
    watching INT,
    completed INT,
    on_hold INT,
    dropped INT,
    plan_to_watch INT,
    total_entries INT,
    rewatched INT,
    episodes_watched INT
    );
```
  4) Создание таблицы ``user_scores``:
```
    CREATE TABLE user_scores (
    user_id INT REFERENCES users(id) ON DELETE CASCADE,
    anime_id INT REFERENCES anime(id) ON DELETE CASCADE,
    rating SMALLINT CHECK (rating BETWEEN 0 AND 10),
    PRIMARY KEY (user_id, anime_id)
    );
```

  Теперь, когда в БД подготовлена структура данных можно приступать к импорту данных. Так как из файлов нужно импортировать не все, а только определенные столбцы, а стандартная утилита ``COPY`` это не поддерживает, то мною было принято решение сначала загрузить все данные во временную таблицу, а потом перелить только нужные данные в основную (чуть позже выяснилось, что это не лучшее решение). 

  **Скрипт для импорта данных из файла ``anime-transformed-dataset-2023.csv``:**
```
    CREATE TEMP TABLE anime_raw (
    col1 TEXT, col2 TEXT, col3 TEXT, col4 TEXT, col5 TEXT, col6 TEXT, col7 TEXT, col8 TEXT, col9 TEXT, col10 TEXT, 
    col11 TEXT, col12 TEXT, col13 TEXT, col14 TEXT, col15 TEXT, col16 TEXT, col17 TEXT, col18 TEXT, col19 TEXT, col20 TEXT, col21 TEXT
    );
    
    COPY anime_raw FROM '/mnt/data/anime-transformed-dataset-2023.csv'
    DELIMITER ',' CSV HEADER QUOTE '"' ESCAPE '"';
    
    INSERT INTO anime (id, title, genres, type, episodes, status, producers, licensors, studios, source, duration, rating, rank, popularity, favorites, scored_by, members, is_hentai)
    SELECT col1::INT, col2, col4, col6::INT, col7, col8, col9, col10, col11, col12, col13, col14, col15::INT, col16::INT, col17::INT, col18::INT, col19::INT,col21::BOOLEAN
    FROM anime_raw;
```

  Данные из файла ``users-details-transformed-2023.csv`` не нуждаются в очистке и нормализации, поэтому их можно импортировать полностью.
  
  **Скрипт для импорта данных из файла ``users-details-transformed-2023.csv``:**
```
    COPY users(id, name, gender, joined, days_watched, mean_score, watching, completed, on_hold, dropped, plan_to_watch, total_entries, rewatched, episodes_watched)
    FROM '/mnt/data/users-details-transformed-2023.csv'
    DELIMITER ','
    CSV HEADER QUOTE '"' ESCAPE '"';
```

  Изначально я попробовал импортировать данные из файла ``users-scores-transformed-2023.csv`` таким же образом, как и для первого файла, однако, так как временные таблицы в PostgreSQL храняться в оперативной памяти, а данный файл содержит больше всего информации, я столкнулся с переполнением оперативной памяти на виртуальной машине. Поэтому, вместо создания временной таблицы, будет создана обычная таблица для импорта всех данных из файла, а потом только необходимые данные будут загружены в основную таблицу. 
  
  **Скрипт для импорта данных из файла ``users-scores-transformed-2023.csv``:**
```
    CREATE TABLE usr_scr_tmp (
    col1 TEXT, col2 TEXT, col3 TEXT, col4 TEXT, col5 TEXT
    );

    COPY usr_scr_tmp FROM '/mnt/data/users-scores-transformed-2023.csv'
    DELIMITER ',' CSV HEADER QUOTE '"' ESCAPE '"';

    INSERT INTO user_scores (user_id, anime_id, rating)
    SELECT col1::INT, col3::INT, col5::SMALLINT
    FROM anime_raw;

    DROP TABLE usr_scr_tmp;
```

## Шаг 3 - Оптимизация кода и контрольный замер.

  Перед началом процесса оптимизации необходимо составить запрос, на примере которого будет производится вся последующая работа по оптимизации. 
  Для этой цели я составил следуюший запрос, который выводит 10 самых высоко-оцененных серилалов у определенного пользователя: 

```
    SELECT u.name, a.title, us.rating  -- Вывод имени пользователя, названия сериала, оценка пользователя
    FROM users u, user_scores us, anime a  -- используемые таблицы
    WHERE us.user_id = u.id
    AND us.anime_id = a.id
    AND (u.id = 3611)  -- указание пользователя
    AND us.rating > 0
    ORDER BY us.rating DESC, a.popularity DESC  -- сортировка
    LIMIT 10; -- ограничение 10 строк
```

  Данный запрос хорош тем, что использует сразу три имеющиеся таблицы, а значит последующая оптимизация затронет их все. 

  **Контрольный замер производительности с EXPLAIN ANALYZE:**

  <img src="https://github.com/user-attachments/assets/7db51444-2ad7-46bf-b378-340dde9d15c0" alt="drawing" width="500"/>

  Как видно, данный запрос имеет Planning Time: 1.921 ms, а также Execution Time: 114.510 ms.
  
  Однако в данном случае сам запрос не является оптимальным, так как составлен не лучшим образом. А именно, данный запрос использует устаревший синтаксис с WHERE вместо JOIN, что хуже оптимизируется,
  нет явного указания соединений JOIN, что увеличивает вероятность неэффективного выполнения.
  
  
  **Оптимизированный запрос:**
```
    SELECT u.name, a.title, us.rating
    FROM user_scores us
    JOIN users u ON us.user_id = u.id
    JOIN anime a ON us.anime_id = a.id
    WHERE (u.id = 123 OR u.name = 'Пример_Пользователя')
    AND us.rating > 0
    ORDER BY us.rating DESC, a.popularity DESC
    LIMIT 10;
```
  Данный запрос выполняет туже задачу, но написан уже с учетом выявленных недостатков. 
  
  **Результат выполнения оптимизированного запроса:**

  <img src="https://github.com/user-attachments/assets/8a4fcf2a-865c-4cca-8a5e-35c9f074837e" alt="drawing" width="500"/>

  Обновленный, оптимизированный запрос уже имеет Planning Time: 0.320 ms (в 6 раз быстрее), а также Execution Time: 46.154 ms (в 2,5 раза быстрее). На данном примере было выявлено, что правильное написание запроса позволяет повысить его производительность более чем в 2 раза. 
  
