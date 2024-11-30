# Домашнее задание - Секционирование таблицы (Секционирование)

## Подготовка тестовой среды

  1) Скачиваю тестовую базу данных demo-small.zip с сайта https://postgrespro.com/community/demodb
  2) Распаковываю архив
  3) Поднимаю базу данных на своем кластере постгрес ``sudo -u postgres psql -d postgres -f /tmp/demo-small-en-20170815.sql -c 'alter database demo set search_path to bookings'``
  4) Проверяю доступность базы

     <img src="https://github.com/user-attachments/assets/a5d6882d-b9ab-4e3a-a080-63a33152c922" alt="drawing" width="500"/>

## Секционирование 

  Самая большая таблица этой базы данных - "ticket_flights". В этой таблице храняться данные о билетах на рейсы. Эта таблица содержит 1045726 записей, состоит из 4 столбцов, среди которых ticket_no(bpchar(13)); flight_id(int4); fare_conditions(varchar(10)); amount(numeric(10,2)).

  Предположительно чаще всего обращения к такой базе будет происходить по полям "ticket_no" и "flight_id". Исходя из этого сделаю партицирование по полю flight_id.

  1) Оценим диапазоны flight_id - ``SELECT MIN(flight_id), MAX(flight_id), COUNT(*) FROM ticket_flights;``

     <img src="https://github.com/user-attachments/assets/02b4307f-50c0-41cf-a0b1-11e33980e129" alt="drawing" width="500"/>

     Видим, что диапазон flight_id между 1 и 33121. По полю flight_id таблицу можно разделить на 7 партиций по 5000 записей
     
  2) Создаем новую секционированную таблицу:

     ```
     CREATE TABLE ticket_flights_partitioned (
        ticket_no bpchar(13),
        flight_id int,
        fare_conditions varchar(10),
        amount numeric(10, 2)
     ) PARTITION BY RANGE (flight_id);

     ```

  3) Создаем разделы в этой таблице:

     ```
      CREATE TABLE ticket_flights_p1
      PARTITION OF ticket_flights_partitioned
      FOR VALUES FROM (1) TO (5000);
      
      CREATE TABLE ticket_flights_p2
      PARTITION OF ticket_flights_partitioned
      FOR VALUES FROM (5000) TO (10000);
      
      CREATE TABLE ticket_flights_p3
      PARTITION OF ticket_flights_partitioned
      FOR VALUES FROM (10000) TO (15000);
      
      CREATE TABLE ticket_flights_p4
      PARTITION OF ticket_flights_partitioned
      FOR VALUES FROM (15000) TO (20000);
      
      CREATE TABLE ticket_flights_p5
      PARTITION OF ticket_flights_partitioned
      FOR VALUES FROM (20000) TO (25000);
      
      CREATE TABLE ticket_flights_p6
      PARTITION OF ticket_flights_partitioned
      FOR VALUES FROM (25000) TO (30000);
      
      CREATE TABLE ticket_flights_p7
      PARTITION OF ticket_flights_partitioned
      FOR VALUES FROM (30000) TO (35000);
     ```
  4) Переносим данные ``INSERT INTO ticket_flights_partitioned SELECT * FROM ticket_flights;``
  5) Сравним производительность не секционированной базы с секционированной

     ``explain  analyze select * from ticket_flights where flight_id between 5000 and 7000 order by flight_id;``

     <img src="https://github.com/user-attachments/assets/72909469-564e-4eab-8713-d6476cab0b5d" alt="drawing" width="500"/>

     ``explain  analyze select * from ticket_flights_partitioned where flight_id between 5000 and 7000 order by flight_id;``

     <img src="https://github.com/user-attachments/assets/f0204584-1753-43de-b832-caf3919567d9" alt="drawing" width="500"/>

  По результатам команд ``explain analyze`` можно увидеть, что Execution Time у партицированной таблицы значительно меньше, чем у не партицированной. Так же видно что Parallel Seq Scan происходит только по одной партиции в данном случае, а не по всей таблице.

**Готово! И на этом домашнее задание выполнено!**
