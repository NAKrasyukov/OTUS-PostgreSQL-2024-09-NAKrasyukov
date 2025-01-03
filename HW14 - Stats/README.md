# Домашнее задание - Работа с join'ами, статистикой (Сбор и использование статистики)

## Подготовка тестовой среды

  1) Создаю 3 таблицы с тестовыми данными:

 ```
        -- Создаём таблицу клиентов
        CREATE TABLE customers (
            customer_id SERIAL PRIMARY KEY,
            name VARCHAR(50),
            city VARCHAR(50)
        );
        
        -- Вставляем тестовые данные
        INSERT INTO customers (name, city)
        VALUES
            ('Иван Иванов', 'Москва'),
            ('Ольга Петрова', 'Санкт-Петербург'),
            ('Анна Сидорова', 'Новосибирск');
        
        -- Создаём таблицу заказов
        CREATE TABLE orders (
            order_id SERIAL PRIMARY KEY,
            customer_id INT,
            amount DECIMAL(10, 2),
            order_date DATE,
            FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        );
        
        -- Вставляем тестовые данные
        INSERT INTO orders (customer_id, amount, order_date)
        VALUES
            (1, 1500.50, '2025-01-01'),
            (2, 2000.00, '2025-01-02'),
            (3, 3000.00, '2025-01-03'),
            (1, 500.00, '2025-01-04');
        
        -- Создаём таблицу продуктов
        CREATE TABLE products (
            product_id SERIAL PRIMARY KEY,
            name VARCHAR(50),
            price DECIMAL(10, 2)
        );
        
        -- Вставляем тестовые данные
        INSERT INTO products (name, price)
        VALUES
            ('Продукт А', 100.00),
            ('Продукт Б', 200.00),
            ('Продукт В', 300.00);
 ```

  ## Работа с запросами

  1)  Прямое соединение (INNER JOIN). Прямое соединение возвращает строки, у которых есть соответствия в обеих таблицах:
```
-- Получить заказы клиентов с их именами
SELECT 
    c.name AS customer_name,
    o.order_id,
    o.amount,
    o.order_date
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

<img src="https://github.com/user-attachments/assets/b67211d8-26b8-426c-b74f-779587b19cc3" alt="drawing" width="500"/>

  2) Левостороннее соединение (LEFT JOIN). Левостороннее соединение возвращает все строки из левой таблицы и соответствующие строки из правой. Если соответствия нет, поля из правой таблицы будут NULL:
```
-- Получить всех клиентов, включая тех, кто не сделал заказы
SELECT 
    c.name AS customer_name,
    o.order_id,
    o.amount,
    o.order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

<img src="https://github.com/user-attachments/assets/838f35c4-53c3-4b1f-aabf-317a38e1bbcd" alt="drawing" width="500"/>

  3) Правостороннее соединение (RIGHT JOIN). Правостороннее соединение возвращает все строки из правой таблицы и соответствующие строки из левой:
```
-- Получить все заказы, включая тех, у кого нет клиентов (хотя тут такого нет)
SELECT 
    c.name AS customer_name,
    o.order_id,
    o.amount,
    o.order_date
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

<img src="https://github.com/user-attachments/assets/3e159fcb-c64e-497b-841c-bf3017ab0280" alt="drawing" width="500"/>

  4) Кросс соединение (CROSS JOIN). Кросс соединение создаёт декартово произведение таблиц:
```
-- Получить все возможные комбинации клиентов и продуктов
SELECT 
    c.name AS customer_name,
    p.name AS product_name
FROM customers c
CROSS JOIN products p;
```

<img src="https://github.com/user-attachments/assets/a0c896e7-6825-40a7-982c-219429b8fda6" alt="drawing" width="500"/>

  5) Полное соединение (FULL JOIN). Полное соединение объединяет строки из обеих таблиц, даже если нет соответствий:
```
-- Получить всех клиентов и заказы, включая несоответствия
SELECT 
    c.name AS customer_name,
    o.order_id,
    o.amount,
    o.order_date
FROM customers c
FULL JOIN orders o ON c.customer_id = o.customer_id;
```

<img src="https://github.com/user-attachments/assets/25ec6f6a-6350-4855-98eb-ce4aa71eed08" alt="drawing" width="500"/>

  6) Использование разных типов соединений в одном запросе. В данном запросе используется левостороннее соединение и кросс соединение:
```
-- Получить заказы с клиентами и продуктами
SELECT 
    c.name AS customer_name,
    o.order_id,
    o.amount,
    p.name AS product_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
CROSS JOIN products p;
```

<img src="https://github.com/user-attachments/assets/89a228fa-2dd4-4e21-a3cf-dff4f7a625a1" alt="drawing" width="500"/>

## Задание со *. Создать 3 свои метрики

  1) Общая сумма заказов каждого клиента:
```
SELECT 
    c.name AS customer_name,
    SUM(o.amount) AS total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name;
```

<img src="https://github.com/user-attachments/assets/2615f832-298d-402f-90a7-bd6c25064ca8" alt="drawing" width="500"/>

  2) Средняя стоимость продукта:
```
SELECT 
    AVG(price) AS average_price
FROM products;
```

<img src="https://github.com/user-attachments/assets/532a9af5-30ca-49b2-b3f4-29d101c07485" alt="drawing" width="500"/>

  3) Количество заказов за определённый период:
```
SELECT 
    COUNT(*) AS total_orders
FROM orders
WHERE order_date BETWEEN '2025-01-01' AND '2025-01-03';
```

<img src="https://github.com/user-attachments/assets/e619e76a-f7d7-4281-a00f-8034176562b5" alt="drawing" width="500"/>


**Готово! И на этом домашнее задание выполнено!**
