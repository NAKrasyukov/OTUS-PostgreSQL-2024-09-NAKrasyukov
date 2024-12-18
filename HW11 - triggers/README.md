# Домашнее задание - Триггеры, поддержка заполнения витрин (Хранимые функции и процедуры)

## Подготовка тестовой среды

  1) Скачиваю приложенный к ДЗ файл со структурой схемы.
  2) Отрабатываю код из файла:

     <img src="https://github.com/user-attachments/assets/6f9ec3e3-a524-453e-9c39-d3a9ddefdc03" alt="drawing" width="500"/>

## Создание функции-триггера

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
     
