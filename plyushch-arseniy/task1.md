 # Простой индекс
## До создания индекса

Запрос:

```sql
EXPLAIN ANALYZE
SELECT * FROM plyushch.t_books WHERE title = 'Book_1';
```
Результаты:
```mathematica
Seq Scan on t_books  (cost=0.00..3155.00 rows=1 width=33) (actual time=0.019..32.229 rows=1 loops=1)
  Filter: ((title)::text = 'Book_1'::text)
  Rows Removed by Filter: 149999
Planning Time: 0.225 ms
Execution Time: 32.264 ms
```
Вывод:
Используется Seq Scan (последовательное сканирование всей таблицы).
Полный перебор строк, что приводит к относительно долгому времени выполнения (32.264 ms).
Условие фильтрации (title = 'Book_1') применяется к каждой строке, что неэффективно для больших таблиц.
После создания индекса

## Создание индекса:
```sql
CREATE INDEX idx_title ON plyushch.t_books (title);
```
Запрос:
```sql
EXPLAIN ANALYZE
SELECT * FROM plyushch.t_books WHERE title = 'Book_1'
```
Результаты:

```mathematica
Index Scan using idx_title on t_books  (cost=0.42..8.44 rows=1 width=33) (actual time=0.040..0.041 rows=1 loops=1)
  Index Cond: ((title)::text = 'Book_1'::text)
Planning Time: 0.248 ms
Execution Time: 0.060 ms
```
## Вывод:

После создания индекса используется Index Scan, который значительно быстрее.
Индекс позволяет напрямую находить строки, соответствующие условию (title = 'Book_1'), вместо полного перебора.
Время выполнения снизилось с 32.264 ms до 0.060 ms — увеличение производительности примерно в 500 раз.
