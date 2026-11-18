# 🗄️ Лабораторные работы по базам данных

![MySQL](https://img.shields.io/badge/MySQL-26.10.0-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![macOS](https://img.shields.io/badge/macOS-000000?style=flat-square&logo=apple&logoColor=white)

> **Дисциплина:** Основы проектирования баз данных
> **Студент:** ChrisRedfield48
> **Дата:** ноябрь 2026

---

## 📚 Содержание

- [🔬 Лабораторная №1 — Основные SQL-операции](https://github.com/ChrisRedfield48/products-mysql-sql-lab)
- [🔬 Лабораторная №2 — _название_] (#-лабораторная-2--название)
- [🔬 Лабораторная №3 — _название_](#-лабораторная-3--название)

---

## 🛠 Среда выполнения

| Компонент | Версия / Значение |
|-----------|-------------------|
| ОС | macOS (Apple Silicon) |
| Терминал | zsh |
| Docker | Docker Desktop |
| Образ MySQL | `mysql:latest` → **MySQL Community Server 26.10.0** |
| Кодировка | `utf8mb4` / `utf8mb4_unicode_ci` |
| Клиент | `mysql` CLI внутри контейнера |

---

# 🔬 Лабораторная №1 — Основные SQL-операции

> **Задание:** Основные SQL-операции на примере двух таблиц «products1» и «products2».
> **Дата выполнения:** ноябрь 2026

## 🎯 Цель работы

Освоить базовые операции реляционной алгебры в SQL на примере двух таблиц с одинаковой структурой:

| Операция | Реализация в SQL |
|----------|------------------|
| Выборка | `WHERE` |
| Проекция | `SELECT столбец` |
| Объединение | `UNION` |
| Разность | `EXCEPT` |
| Пересечение | `INTERSECT` |
| Декартово произведение | `CROSS JOIN` |
| Соединение | `JOIN` / `LEFT JOIN` |
| Деление | _(требует 3 таблиц — не рассматривается)_ |

Дополнительно: научиться работать с **MySQL в Docker**, создавать **дамп** БД и **восстанавливать** из него.

## 📁 Структура лабораторной

```
lab-1-sql-operations/
├── 📄 my.cnf          # конфиг MySQL (utf8mb4)
├── 💾 dump.sql        # дамп БД shop
├── 📝 log.txt         # лог всех выполненных команд
└── 📸 screenshots/
    ├── 01-docker-ps.png
    ├── 02-tables.png
    ├── 03-except-intersect.png
    ├── 04-joins.png
    ├── 05a-restore-command.png
    └── 05b-restore-check.png
```

## 🚀 Ход выполнения

### Шаг 1. Запуск контейнера

Создан файл `my.cnf` для корректной работы с кириллицей:

```ini
[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4
```

Запуск контейнера MySQL с монтированием конфига:

```bash
docker run --name products-mysql \
  -e MYSQL_ROOT_PASSWORD=123 \
  -e LANG=C.UTF-8 \
  -v mysql_data:/var/lib/mysql \
  -v "$(pwd)/my.cnf:/etc/mysql/conf.d/my.cnf" \
  -d mysql:latest
```

**Результат:** контейнер `products-mysql` в статусе `Up`:

![docker ps](lab-1-sql-operations/screenshots/01-docker-ps.png)

---

### Шаг 2. Создание БД и таблиц

```sql
CREATE DATABASE shop
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

USE shop;

CREATE TABLE products1 (
    name  VARCHAR(100) NOT NULL,
    price INT NOT NULL
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE TABLE products2 (
    name  VARCHAR(100) NOT NULL,
    price INT NOT NULL
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

---

### Шаг 3. Заполнение данными

```sql
INSERT INTO products1 (name, price) VALUES
  ('Дыни', 800),
  ('Клубника', 150),
  ('Яблоки', 120),
  ('Лимоны', 200);

INSERT INTO products2 (name, price) VALUES
  ('Дыни', 800),
  ('Клубника', 150),
  ('Каштаны', 200),
  ('Хурма', 350);
```

**Таблица `products1`:**

| name | price |
|------|------:|
| Дыни | 800 |
| Клубника | 150 |
| Яблоки | 120 |
| Лимоны | 200 |

**Таблица `products2`:**

| name | price |
|------|------:|
| Дыни | 800 |
| Клубника | 150 |
| Каштаны | 200 |
| Хурма | 350 |

![Таблицы и данные](lab-1-sql-operations/screenshots/02-tables.png)

---

### Шаг 4. Основные операции

#### 4.1. Выборка (SELECT + WHERE)

**Задача:** товары из `products1` дороже 150 ₽.

```sql
SELECT * FROM products1 WHERE price > 150;
```

| name | price |
|------|------:|
| Дыни | 800 |
| Лимоны | 200 |

---

#### 4.2. Проекция (SELECT столбца)

**Задача:** только названия товаров из `products1`.

```sql
SELECT name FROM products1;
```

**Результат:** `Дыни`, `Клубника`, `Яблоки`, `Лимоны`.

---

#### 4.3. Объединение (UNION)

**Задача:** единый список уникальных названий из обеих таблиц.

```sql
SELECT name FROM products1
UNION
SELECT name FROM products2;
```

| name |
|------|
| Дыни |
| Клубника |
| Яблоки |
| Лимоны |
| Каштаны |
| Хурма |

> `UNION` удаляет дубликаты. `UNION ALL` — сохраняет все строки, включая повторы.

---

#### 4.4. Разность (EXCEPT)

**Задача:** товары, которые есть в `products1`, но отсутствуют в `products2`.

```sql
SELECT name, price FROM products1
EXCEPT
SELECT name, price FROM products2;
```

| name | price |
|------|------:|
| Яблоки | 120 |
| Лимоны | 200 |

---

#### 4.5. Пересечение (INTERSECT)

**Задача:** товары, которые есть и в `products1`, и в `products2` (с одинаковой ценой).

```sql
SELECT name, price FROM products1
INTERSECT
SELECT name, price FROM products2;
```

| name | price |
|------|------:|
| Дыни | 800 |
| Клубника | 150 |

![EXCEPT и INTERSECT](lab-1-sql-operations/screenshots/03-except-intersect.png)

---

#### 4.6. Декартово произведение (CROSS JOIN)

**Задача:** все возможные пары «товар из первой таблицы — товар из второй» (4 × 4 = **16 строк**).

```sql
SELECT t1.name AS name1, t1.price AS price1,
       t2.name AS name2, t2.price AS price2
FROM products1 t1
CROSS JOIN products2 t2;
```

| name1 | price1 | name2 | price2 |
|-------|------:|-------|------:|
| Дыни | 800 | Дыни | 800 |
| Дыни | 800 | Клубника | 150 |
| Дыни | 800 | Каштаны | 200 |
| Дыни | 800 | Хурма | 350 |
| Клубника | 150 | Дыни | 800 |
| … | … | … | … |

---

#### 4.7. Соединение (INNER JOIN)

**Задача:** совпадающие по названию товары с ценами из обеих таблиц.

```sql
SELECT t1.name, t1.price AS price1, t2.price AS price2
FROM products1 t1
JOIN products2 t2 ON t1.name = t2.name;
```

| name | price1 | price2 |
|------|------:|------:|
| Дыни | 800 | 800 |
| Клубника | 150 | 150 |

---

#### 4.8. Левое соединение (LEFT JOIN)

**Задача:** все товары из `products1` с ценами из `products2` (если пары нет — `NULL`).

```sql
SELECT t1.name, t1.price AS price1, t2.price AS price2
FROM products1 t1
LEFT JOIN products2 t2 ON t1.name = t2.name;
```

| name | price1 | price2 |
|------|------:|------:|
| Дыни | 800 | 800 |
| Клубника | 150 | 150 |
| Яблоки | 120 | NULL |
| Лимоны | 200 | NULL |

![JOIN-операции](lab-1-sql-operations/screenshots/04-joins.png)

---

### Шаг 5. Дамп и восстановление

#### 5.1. Создание дампа

```bash
docker exec -e MYSQL_PWD=123 products-mysql mysqldump -uroot shop > dump.sql
```

Результат — файл `dump.sql` (~3 KB).

#### 5.2. Удаление и восстановление БД

```sql
DROP DATABASE shop;
```

Создаём пустую БД и импортируем дамп:

```sql
CREATE DATABASE shop
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

```bash
docker exec -i products-mysql mysql -uroot -p123 --force shop < dump.sql
```

> ⚠️ **Ошибка `ERROR 3546 (HY000): @@GLOBAL.GTID_PURGED cannot be changed`** — ожидаема. Флаг `--force` продолжает импорт, данные восстанавливаются корректно.

![Команда восстановления](lab-1-sql-operations/screenshots/05a-restore-command.png)

#### 5.3. Проверка после восстановления

```sql
USE shop;
SHOW TABLES;
SELECT * FROM products1;
SELECT * FROM products2;
```

Обе таблицы восстановлены с полным набором строк.

![Проверка после восстановления](lab-1-sql-operations/screenshots/05b-restore-check.png)

---

## 📊 Сводная таблица операций

| # | Операция | SQL-оператор | Результат |
|---|----------|--------------|-----------|
| 1 | Выборка | `WHERE` | 2 строки (Дыни, Лимоны) |
| 2 | Проекция | `SELECT name` | 4 названия |
| 3 | Объединение | `UNION` | 6 уникальных названий |
| 4 | Разность | `EXCEPT` | 2 строки (Яблоки, Лимоны) |
| 5 | Пересечение | `INTERSECT` | 2 строки (Дыни, Клубника) |
| 6 | Декартово произведение | `CROSS JOIN` | 16 строк (4 × 4) |
| 7 | Соединение | `INNER JOIN` | 2 совпадающие строки |
| 8 | Левое соединение | `LEFT JOIN` | 4 строки (2 с `NULL`) |
| 9 | Дамп | `mysqldump` | `dump.sql` (~3 KB) |
| 10 | Восстановление | `mysql < dump.sql` | БД восстановлена |

---

## ✅ Вывод по лабораторной №1

В ходе работы:

1. ✅ Развёрнут **MySQL 26.10.0** в Docker-контейнере с поддержкой кириллицы (`utf8mb4`).
2. ✅ Создана БД `shop` и две таблицы `products1`, `products2` с тестовыми данными.
3. ✅ Освоены **8 базовых SQL-операций** над множествами.
4. ✅ Проверена работа **дампов**: создание (`mysqldump`), удаление БД, восстановление из дампа.
5. ✅ Все результаты задокументированы скриншотами и логом команд.

**Ключевой вывод:** SQL предоставляет полный набор операций реляционной алгебры. `UNION`, `EXCEPT`, `INTERSECT` работают на уровне множеств строк; `JOIN`-операции позволяют связывать данные по ключу. `EXCEPT` и `INTERSECT` доступны в MySQL начиная с версии 8.0.31.

## 📎 Приложения к лабораторной №1

- 📄 [Лог выполнения команд](lab-1-sql-operations/log.txt)
- 💾 [Дамп БД `shop`](lab-1-sql-operations/dump.sql)
- ⚙️ [Конфиг MySQL](lab-1-sql-operations/my.cnf)

---

# 🔬 Лабораторная №2 — _название_

> **Задание:** _краткое описание задания_
> **Дата выполнения:** _дата_

## 🎯 Цель работы

_Описание цели._

## 🚀 Ход выполнения

_Разделы по мере выполнения._

## ✅ Вывод по лабораторной №2

_Вывод._

---

# 🔬 Лабораторная №3 — _название_

> **Задание:** _краткое описание задания_
> **Дата выполнения:** _дата_

## 🎯 Цель работы

_Описание цели._

## 🚀 Ход выполнения

_Разделы по мере выполнения._

## ✅ Вывод по лабораторной №3

_Вывод._

---

<div align="center">

_© 2026_

</div>