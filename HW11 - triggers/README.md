# Домашнее задание - Триггеры, поддержка заполнения витрин (Хранимые функции и процедуры)

## Подготовка тестовой среды

  1) Скачиваю приложенный к ДЗ файл со структурой схемы.
  2) Отрабатываю код из файла:

     <img src="https://github.com/user-attachments/assets/6f9ec3e3-a524-453e-9c39-d3a9ddefdc03" alt="drawing" width="500"/>

## Создание функции и триггера

  1) Назначение функции-триггера.
     **Функция-триггер должна:**
      - Автоматически обновлять витрину (good_sum_mart) при изменении таблицы ``sales``.
      - Работать для всех типов операций: ``INSERT``, ``UPDATE`` и ``DELETE``.
      - Учитывать зависимость от таблицы ``goods`` для вычисления цены товара.

     **Разделение логики по типу операции:**
      - INSERT — добавление новой записи в sales.
      - UPDATE — изменение существующей записи в sales.
      - DELETE — удаление записи из sales.

     **INSERT**
      
      - Определяем ``good_name`` и цену (``good_price``) из таблицы ``goods``, связанной через ``good_id``.
      - Вычисляем сумму (``good_price * sales_qty``) для нового товара.
      - Обновляем или добавляем данные в таблицу ``good_sum_mart``:
          Если good_name уже существует, увеличиваем значение ``sum_sale``.
          Если good_name отсутствует, добавляем новую строку.

     **UPDATE**

      - Вычитаем старую сумму (``good_price * OLD.sales_qty``) из ``good_sum_mart`` на основании предыдущих данных.
      - Добавляем новую сумму (``good_price * NEW.sales_qty``) в ``good_sum_mart``.

     **DELETE**

      - Определяем, какую сумму нужно вычесть из ``good_sum_mart`` на основании удаляемой записи (``OLD.sales_qty``).
      - Вычитаем сумму (``good_price * OLD.sales_qty``) из ``good_sum_mart``.
      - Если результат суммы стал <= 0, удаляем строку из ``good_sum_mart``.

  2) Итоговая структура функции:
     ```
      CREATE OR REPLACE FUNCTION update_good_sum_mart()
      RETURNS TRIGGER AS $$
      BEGIN
          -- INSERT
          IF (TG_OP = 'INSERT') THEN
              INSERT INTO good_sum_mart (good_name, sum_sale)
              VALUES (
                  (SELECT good_name FROM goods WHERE goods_id = NEW.good_id),
                  (SELECT good_price * NEW.sales_qty FROM goods WHERE goods_id = NEW.good_id)
              )
              ON CONFLICT (good_name) DO UPDATE
              SET sum_sale = good_sum_mart.sum_sale + EXCLUDED.sum_sale;
      
          -- UPDATE
          ELSIF (TG_OP = 'UPDATE') THEN
              -- Удаляем старое значение
              UPDATE good_sum_mart
              SET sum_sale = sum_sale - (SELECT good_price * OLD.sales_qty FROM goods WHERE goods_id = OLD.good_id)
              WHERE good_name = (SELECT good_name FROM goods WHERE goods_id = OLD.good_id);
      
              -- Добавляем новое значение
              INSERT INTO good_sum_mart (good_name, sum_sale)
              VALUES (
                  (SELECT good_name FROM goods WHERE goods_id = NEW.good_id),
                  (SELECT good_price * NEW.sales_qty FROM goods WHERE goods_id = NEW.good_id)
              )
              ON CONFLICT (good_name) DO UPDATE
              SET sum_sale = good_sum_mart.sum_sale + EXCLUDED.sum_sale;
      
          -- DELETE
          ELSIF (TG_OP = 'DELETE') THEN
              UPDATE good_sum_mart
              SET sum_sale = sum_sale - (SELECT good_price * OLD.sales_qty FROM goods WHERE goods_id = OLD.good_id)
              WHERE good_name = (SELECT good_name FROM goods WHERE goods_id = OLD.good_id);
      
              -- Удаляем строки с нулевой или отрицательной суммой
              DELETE FROM good_sum_mart WHERE sum_sale <= 0;
          END IF;
      
          RETURN NULL;
      END;
      $$ LANGUAGE plpgsql;
     ```

  3) Создание триггера:
     ```
      CREATE TRIGGER trg_update_good_sum_mart
      AFTER INSERT OR UPDATE OR DELETE ON sales
      FOR EACH ROW
      EXECUTE FUNCTION update_good_sum_mart();
     ```
  4) Добавляю констрейнт на таблицу ``good_sum_mart`` чтобы название товара в этой таблице было уникальным: ``ALTER TABLE good_sum_mart ADD CONSTRAINT good_sum_mart_good_name_unique UNIQUE (good_name);``

## Проверка работы функции и триггера

  1) Проверка ``INSERT``: ``INSERT INTO sales (good_id, sales_qty) VALUES (1, 5);``
  2) ``SELECT * FROM good_sum_mart;``

     <img src="https://github.com/user-attachments/assets/7fc9e4cd-66bf-4865-8f43-af0570cc9251" alt="drawing" width="500"/>

  3) Проверка ``UPDATE``: ``UPDATE sales SET sales_qty = 15 WHERE sales_id = 1;``
  4) ``SELECT * FROM good_sum_mart;``

     <img src="https://github.com/user-attachments/assets/9286d346-f1f3-44bd-8afd-4a68c6797aa4" alt="drawing" width="500"/>

  5) Проверка ``DELETE``: ``DELETE FROM sales;``
  6) ``SELECT * FROM good_sum_mart;``

     <img src="https://github.com/user-attachments/assets/2dc3b347-590e-45f3-8a9a-c1372df07ff5" alt="drawing" width="500"/>

## Задание со звездочкой*

  Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?

  1) Автоматическое обновление данных
      - Витрина всегда содержит актуальную информацию, так как обновляется автоматически при изменении данных (добавление, обновление, удаление записей в исходных таблицах).
      - В случае отчета "по требованию" информация может устареть, если изменения в исходных данных произошли между моментами генерации отчета.
        
  2) Реакция на изменения цен
     Если цены товаров изменяются со временем, отчет "по требованию" должен учитывать изменения. Это требует дополнительной логики:
      - Учет исторических цен (например, хранение цены продажи на момент транзакции).
      - Динамическое пересчет суммы с учетом новых цен (что увеличивает сложность SQL-запроса).
     Схема с витриной позволяет: 
      - Сохранить корректную сумму продаж в момент выполнения транзакции.
      - Избежать влияния изменений цен на исторические данные, так как сумма фиксируется при каждой операции.
    
  3) Снижение нагрузки на основную базу
      - Витрина: Основные вычисления (агрегации, группировки) происходят в момент изменения данных (через триггер). Таким образом, чтение витрины становится быстрым, даже при большом объеме данных.
      - Отчет "по требованию": Каждый раз, когда выполняется запрос, требуется доступ к основным таблицам (goods, sales), что может привести к значительным накладным расходам (особенно при большом объеме данных).

  4) Упрощение отчетности
      - Схема с витриной облегчает генерацию отчетов. Отчет можно построить на основе уже готовой агрегированной таблицы (good_sum_mart), не создавая сложных запросов с JOIN и группировками.
      - Отчеты "по требованию" требуют написания сложных запросов, особенно если данные распределены по нескольким таблицам.

  5) Поддержка сложных сценариев
     В реальной жизни могут быть сложные сценарии, например:
      - Удаление товара из ассортимента.
      - Возвраты покупок, корректировки продаж.
      - Периодические скидки или акции, влияющие на итоговую сумму.
     В таких случаях триггер может быть доработан для обработки всех изменений, чтобы витрина всегда оставалась актуальной. А отчет "по требованию" усложняется с каждым новым требованием.

**Готово! И на этом домашнее задание выполнено!**
