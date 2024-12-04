
# "Цель: Создание функционального индекса для ускорения запросов с использованием функции LOWER() на колонке title."

Запрос до создания индекса
```sql
EXPLAIN ANALYZE
SELECT * FROM plyushch.t_books WHERE LOWER(title) = 'Book_1';
```
План выполнения:
```mathematica
Seq Scan on t_books  (cost=0.00..3530.00 rows=750 width=33) (actual time=117.517..117.518 rows=0 loops=1)
  Filter: (lower((title)::text) = 'Book_1'::text)
  Rows Removed by Filter: 150000
Planning Time: 0.111 ms
Execution Time: 117.556 ms
```
Вывод:
Без индекса использовался Seq Scan (последовательное сканирование всей таблицы).
Планировалось проверить 150000 строк (Rows Removed by Filter: 150000).
Общее время выполнения: 117.556 ms.
Последовательное сканирование занимает много времени из-за необходимости проверять каждую строку.

Создание индекса

```sql
CREATE INDEX idx_lower_title ON plyushch.t_books (LOWER(title));
```
Запрос после создания индекса
```sql
EXPLAIN ANALYZE
SELECT * FROM plyushch.t_books WHERE LOWER(title) = 'Book_1';
```
План выполнения:

```mathematica
Bitmap Heap Scan on t_books  (cost=18.23..1179.18 rows=750 width=33) (actual time=0.080..0.081 rows=0 loops=1)
  Recheck Cond: (lower((title)::text) = 'Book_1'::text)
  ->  Bitmap Index Scan on idx_lower_title  (cost=0.00..18.05 rows=750 width=0) (actual time=0.057..0.058 rows=0 loops=1)
        Index Cond: (lower((title)::text) = 'Book_1'::text)
Planning Time: 0.448 ms
Execution Time: 0.121 ms
```
Вывод:

После создания функционального индекса запрос стал использовать Bitmap Index Scan, который значительно быстрее.
Планировалось проверить те же 150000 строк, но благодаря индексу фильтрация выполнялась быстрее.
Общее время выполнения: 0.121 ms.
Время выполнения уменьшилось почти в 1000 раз.
 Выводы по эффективности функционального индекса
Функциональный индекс позволяет эффективно ускорить запросы, которые применяют функции (LOWER() в данном случае) к колонкам.
Последовательное сканирование (Seq Scan) заменяется на Bitmap Index Scan, который работает быстрее за счёт предварительной фильтрации через индекс.
Применение индекса особенно полезно для таблиц с большим количеством записей.

# Составной индекс

## До создания индекса

Запрос:

```sql
EXPLAIN ANALYZE
SELECT * FROM plyushch.t_books WHERE title = 'Book_2';
```
Результаты:

```mathematica
Seq Scan on t_books  (cost=0.00..3155.00 rows=1 width=33) (actual time=0.013..19.628 rows=1 loops=1)
  Filter: ((title)::text = 'Book_2'::text)
  Rows Removed by Filter: 149999
Planning Time: 0.763 ms
Execution Time: 19.653 ms
```
Вывод:

Используется Seq Scan.
Полный перебор строк приводит к относительно долгому времени выполнения (19.653 ms).
После создания составного индекса

## Создание индекса:

```sql
CREATE INDEX idx_author_title ON plyushch.t_books (author, title);
```
Запрос:

```sql
EXPLAIN ANALYZE
SELECT * FROM plyushch.t_books WHERE title = 'Book_2';
```
Результаты:
```mathematica
Seq Scan on t_books  (cost=0.00..3155.00 rows=1 width=33) (actual time=0.018..44.411 rows=1 loops=1)
  Filter: ((title)::text = 'Book_2'::text)
  Rows Removed by Filter: 149999
Planning Time: 0.510 ms
Execution Time: 44.461 ms
```
## Вывод:

Составной индекс не был использован для запроса WHERE title = 'Book_2'.
PostgreSQL продолжает использовать Seq Scan, так как составной индекс на колонках (author, title) не оптимален для фильтрации только по title.
Время выполнения даже увеличилось (до 44.461 ms), что может быть связано с затратами на планирование.
