PG-Инженер по отказоустойчивости
1. 1. Выполнил развертывание рабочего окружения(https://gist.github.com/pkonotopov/c1e048e217217dcb2e5d7bed1739b238)
2.  Выполнил запрос 

```
explain analyze verbose
SELECT body 
FROM posts 
WHERE body ILIKE '%postgres%awesome%'
OR body ILIKE '%postgres%amazing%';
```

- `EXPLAIN`: Эта команда показывает план выполнения запроса. Она объясняет, как база данных планирует выполнить запрос без фактического выполнения его.
    
- `ANALYZE`: Эта команда показывает, какие фактические результаты были получены при выполнении запроса. Это включает в себя информацию о времени выполнения и использовании ресурсов.
    
- `VERBOSE`: Этот модификатор выводит более подробную информацию в плане выполнения и анализе запроса.
-  `WHERE body ILIKE'%postgres%awesome%' OR body ILIKE '%postgres%amazing%'**`: Это условие фильтрации, которое определяет, какие строки должны быть включены в результат, исходя из содержимого столбца `body`. Здесь используется оператор `ILIKE`, который выполняет сравнение строк, игнорируя регистр, а также оператор `%`, который означает любую последовательность символов (включая ноль символов) перед или после слов "postgres". Таким образом, это условие выбирает строки, где столбец `body` содержит фразу "postgres" вместе с "awesome" или "amazing". 

Результат запроса:
![Untitled](https://github.com/Rizikkz/PG/blob/main/image/2.png).

3. Когда база находиться под нагрузкой, лучшие время для создание индекса в периоды низкой активности, чтобы минимизировать воздействие на работу пользователей. Это может быть ночное время или время, когда нагрузка на сервер минимальна.
4. Является ли хорошей практикой создание индекса во время высокой транзакционной нагрузки? - Нет, но если стоит необходимость , то тогда можно использовать создание индексов во внеочередном режиме (CONCURRENTLY): PostgreSQL поддерживает создание индексов в режиме `CONCURRENTLY`, который позволяет пользователям продолжать выполнять операции чтения и записи, в то время как индекс создается. Это может быть полезно в ситуациях, когда невозможно приостановить работу приложений для создания индексов.
5.  Чтобы ускорить создание индекса при повышенной нагрузке в PostgreSQL нужно изменить параметры: 
- **max_worker_processes** -  этот параметр определяет максимальное количество рабочих процессов, которые могут использоваться для параллельного выполнения операций, таких как создание индексов;
- **maintenance_work_mem** - этот параметр определяет объем памяти, выделенный для операций обслуживания базы данных, таких как создание и обновление индексов;
- **work_mem** - этот параметр определяет объем памяти, выделенный для операций сортировки и хэширования данных в рамках отдельного сеанса подключения. Увеличение этого параметра может ускорить операции сортировки, которые могут быть частью создания индексов;
- **temp_buffers** - этот параметр определяет объем памяти, выделенный для временных буферов, которые используются для временного хранения данных во время выполнения операций, таких как создание индексов. Увеличение этого параметра может помочь ускорить операции с временными данными.
Для того чтобы создать индекс  в нагруженной системе необходимо использовать **CONCURRENTLY**.

```bash
CREATE INDEX CONCURRENTLY idx_posts_body ON posts USING gin (body gin_trgm_ops);
```
- **idx_posts_body** - это имя индекса, которое вы определяете. В данном случае, имя индекса - `idx_posts_body`;
    
- **ON posts** - это указание на таблицу, для которой создается индекс. В данном случае, индекс создается для таблицы `posts`;
    
- **USING gin** - этот параметр указывает, что используется тип индекса GIN (Generalized Inverted Index), который хорошо подходит для поиска в текстовых данных и других сложных типах данных ;
    
- **(body gin_trgm_ops)** - это определение столбца и метода индексации. В этой части, `body` - это имя столбца, для которого создается индекс, а `gin_trgm_ops` - это метод индексации, который используется для индексации trigram'ов (троек символов). Метод `gin_trgm_ops` подходит для поиска по части или целым словам в текстовых данных.

Перед тем как выполнить создание индекса, нужно убедится что установлено расширение `pg_trgm`  в PostgreSQL

```bash
SELECT * FROM pg_extension WHERE extname = 'pg_trgm';
```
Если результат пустой, это означает, что расширение не установлено. Чтобы установить его, выполните следующую команду:

```bash
CREATE EXTENSION pg_trgm;
```

делаем запрос после создания индекса:

```bash
explain analyze verbose
SELECT body
FROM posts
WHERE body LIKE '%postgres%awesome%'
OR body LIKE '%postgres%amazing%';
```

в запросе меняем ILIKE на LIKE (**LIKE** - Оператор `LIKE` используется для сравнения строк с учетом регистра. При использовании `LIKE`, символ `%` соответствует любой последовательности символов (включая пустую строку), а символ `_` соответствует одному символу.    
**ILIKE** - Оператор `ILIKE` также используется для сравнения строк, но не учитывает регистр символов. Поэтому `ILIKE` сопоставляет строки без учета регистра символов. В остальном, его работа аналогична оператору `LIKE`.)

Результат запроса:
![Untitled](https://github.com/Rizikkz/PG/blob/main/image/3.png)
