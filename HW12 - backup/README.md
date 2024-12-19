# Домашнее задание - Бэкапы (Резервное копирование и восстановление)

## Подготовка тестовой среды

  1) Создаю в БД схему и вней таблицу: 
    ```
      CREATE SCHEMA example_schema;

      CREATE TABLE example_schema.example_table (
          id SERIAL PRIMARY KEY,
          name TEXT NOT NULL,
          value INTEGER NOT NULL
      );
    ```
