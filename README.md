# SQL Coding

1. [Chapter 1: Basic Querys](#chapter1)
    - [Chapter 1 - Part 1: Detecting duplicates with different values](#chapter1part1)
    - [Chapter 1 - Part 2: Detecting duplicate lines](#chapter1part2)
    - [Chapter 1 - Part 3: Cleanse and Transform String Data](#chapter1part3)
    - [Chapter 1 - Part 4: Filtering Data Based on a Specific Character](#chapter1part4)
    - [Chapter 1 - Part 5: Unpivoting Data Key-Value String (Create Column from Values)](#chapter1part5)
    - [Chapter 1 - Part 6: Pivoting Data Value String (Row-to-String Aggregation)](#chapter1part6)
    - [Chapter 1 - Part 7: Select just Even Numbers](#chapter1part7)
    - [Chapter 1 - Part 8: Quantify Data Redundancy](#chapter1part8)
    - [Chapter 1 - Part 9: Count the Number of Ocorrences in a String](#chapter1part9)
    - [Chapter 1 - Part 10: Find the Min and Max Length of a String and Ordered Alphabetically](#chapter1part10)
  

  

  

  

## <a name="chapter1"></a>Chapter 1: Basic Querys

#### <a name="chapter1part1"></a>Chapter 1 - Part 1: Detecting duplicates with different values

OBS: Using DuckDB

```sql
CREATE TABLE products (sku VARCHAR(10),price DECIMAL(10,2));

INSERT INTO products (sku, price) VALUES
('AA', 20),
('AA', 30),
('BB', 40),
('CC', 50),
('CC', 50),
('DD', 60),
('DD', 70);

SELECT * FROM products;
┌─────────┬───────────────┐
│   sku   │     price     │
│ varchar │ decimal(10,2) │
├─────────┼───────────────┤
│ AA      │         20.00 │
│ AA      │         30.00 │
│ BB      │         40.00 │
│ CC      │         50.00 │
│ CC      │         50.00 │
│ DD      │         60.00 │
│ DD      │         70.00 │
└─────────┴───────────────┘

SELECT sku FROM products GROUP BY sku HAVING COUNT(DISTINCT price) > 1;

┌─────────┐
│   sku   │
│ varchar │
├─────────┤
│ AA      │
│ DD      │
└─────────┘
```

#### <a name="chapter1part2"></a>Chapter 1 - Part 2: Detecting duplicate lines

OBS: Using DuckDB

```sql
CREATE TABLE products (sku VARCHAR(10),price DECIMAL(10,2));

INSERT INTO products (sku, price) VALUES
('AA', 20),
('AA', 30),
('BB', 40),
('CC', 50),
('CC', 50),
('DD', 60),
('DD', 70);

SELECT * FROM products;
┌─────────┬───────────────┐
│   sku   │     price     │
│ varchar │ decimal(10,2) │
├─────────┼───────────────┤
│ AA      │         20.00 │
│ AA      │         30.00 │
│ BB      │         40.00 │
│ CC      │         50.00 │
│ CC      │         50.00 │
│ DD      │         60.00 │
│ DD      │         70.00 │
└─────────┴───────────────┘

SELECT sku, price, COUNT(*) FROM products GROUP BY sku, price HAVING COUNT(*) > 1 ORDER BY sku ASC;

┌─────────┬───────────────┬──────────────┐
│   sku   │     price     │ count_star() │
│ varchar │ decimal(10,2) │    int64     │
├─────────┼───────────────┼──────────────┤
│ CC      │         50.00 │            2 │
└─────────┴───────────────┴──────────────┘
```

#### <a name="chapter1part3"></a>Chapter 1 - Part 3: Cleanse and Transform String Data

OBS: Using DuckDB

```sql
CREATE TABLE test AS SELECT CONCAT(chr(13),chr(10),'Hello World') AS example;

SELECT * FROM test;

┌─────────────────┐
│     example     │
│     varchar     │
├─────────────────┤
│ \r\nHello World │
└─────────────────┘

SELECT REGEXP_REPLACE(example, '[\n\r]+'::text, ' '::text, 'g'::text) AS transformed_example FROM test;

┌─────────────────────┐
│ transformed_example │
│       varchar       │
├─────────────────────┤
│  Hello World        │
└─────────────────────┘
```

#### <a name="chapter1part4"></a>Chapter 1 - Part 4: Filtering Data Based on a Specific Character

OBS: Using DuckDB

```sql
CREATE TABLE products (sku VARCHAR(10),price DECIMAL(10,2));

INSERT INTO products (sku, price) VALUES
('AA', 20),
('AA', 30),
('B|B', 40),
('CC', 50),
('CC', 50),
('DD', 60),
('DD', 70);

SELECT * FROM products;

┌─────────┬───────────────┐
│   sku   │     price     │
│ varchar │ decimal(10,2) │
├─────────┼───────────────┤
│ AA      │         20.00 │
│ AA      │         30.00 │
│ B|B     │         40.00 │
│ CC      │         50.00 │
│ CC      │         50.00 │
│ DD      │         60.00 │
│ DD      │         70.00 │
└─────────┴───────────────┘

SELECT * FROM products WHERE sku LIKE '%|%';

┌─────────┬───────────────┐
│   sku   │     price     │
│ varchar │ decimal(10,2) │
├─────────┼───────────────┤
│ B|B     │         40.00 │
└─────────┴───────────────┘
```

#### <a name="chapter1part5"></a>Chapter 1 - Part 5: Unpivoting Data Key-Value String (Create Column from Values)

OBS: Using DuckDB

```sql
CREATE TABLE products (line_number INT, sku VARCHAR(10), price DECIMAL(10,2), metafields VARCHAR);

INSERT INTO products (line_number, sku, price, metafields) VALUES
  (1, 'AA', 20.0, 'name|maria#lastname|joao#gender|femea'),
  (2, 'BB', 30.0, 'name|carlos#gender|male'),
  (3, 'CC', 40.0, 'name|joana');
  
SELECT * FROM products;

┌─────────────┬─────────┬───────────────┬───────────────────────────────────────┐
│ line_number │   sku   │     price     │              metafields               │
│    int32    │ varchar │ decimal(10,2) │                varchar                │
├─────────────┼─────────┼───────────────┼───────────────────────────────────────┤
│           1 │ AA      │         20.00 │ name|maria#lastname|joao#gender|femea │
│           2 │ BB      │         30.00 │ name|carlos#gender|male               │
│           3 │ CC      │         40.00 │ name|joana                            │
└─────────────┴─────────┴───────────────┴───────────────────────────────────────┘

WITH pairs AS (
  SELECT
 line_number AS LineNumber,
 sku AS SKU,
 price AS PRICE,
 regexp_split_to_table(metafields, '#') AS metafields
 FROM products
 ),
 key_value AS (
 SELECT
  LineNumber AS LineNumber,
  SKU AS SKU,
  PRICE AS PRICE,
  split_part(metafields, '|', 1) AS key,
 split_part(metafields, '|', 2) AS pair
 FROM pairs
 ),
 firstNameQuery AS (
 SELECT
   LineNumber AS LineNumber,
   SKU AS SKU,
   PRICE AS PRICE,
   CASE 
      WHEN key = 'name' 
      THEN pair 
      ELSE NULL 
  END AS FirstName
 FROM key_value
 )
 
 SELECT
   LineNumber AS LineNumber,
   SKU AS SKU,
   PRICE AS PRICE,
   FirstName AS FIRSTNAME
 FROM firstNameQuery
 WHERE(CASE WHEN FirstName IS NOT NULL AND FirstName != '' THEN 1 ELSE 0 END) >= 1;
 
┌────────────┬─────────┬───────────────┬───────────┐
│ LineNumber │   SKU   │     PRICE     │ FIRSTNAME │
│   int32    │ varchar │ decimal(10,2) │  varchar  │
├────────────┼─────────┼───────────────┼───────────┤
│          1 │ AA      │         20.00 │ maria     │
│          2 │ BB      │         30.00 │ carlos    │
│          3 │ CC      │         40.00 │ joana     │
└────────────┴─────────┴───────────────┴───────────┘
```

#### <a name="chapter1part6"></a>Chapter 1 - Part 6: Pivoting Data Value String (Row-to-String Aggregation)

OBS: Using DuckDB

```sql
Appendix A - Part 6: Split a field and aggregate values
CREATE TABLE products (sku VARCHAR(10),price DECIMAL(10,2), size VARCHAR(10));

INSERT INTO products (sku, price, size) VALUES
 ('AA', 20, 'S'),
 ('AA', 20, 'M'),
 ('AA', 20, 'L'),
 ('BB', 30, '38'),
 ('BB', 30, '39');

SELECT * FROM products;

┌─────────┬───────────────┬─────────┐
│   sku   │     price     │  size   │
│ varchar │ decimal(10,2) │ varchar │
├─────────┼───────────────┼─────────┤
│ AA      │         20.00 │ S       │
│ AA      │         20.00 │ M       │
│ AA      │         20.00 │ L       │
│ BB      │         30.00 │ 38      │
│ BB      │         30.00 │ 39      │
└─────────┴───────────────┴─────────┘

SELECT
   price AS PRICE,
   CONCAT(sku,'|',string_agg(TRIM(size), '_' ORDER BY size)) AS ConcatSize
FROM
   products
GROUP BY
   sku, price;

┌───────────────┬────────────┐
│     PRICE     │ ConcatSize │
│ decimal(10,2) │  varchar   │
├───────────────┼────────────┤
│         30.00 │ BB|38_39   │
│         20.00 │ AA|L_M_S   │
└───────────────┴────────────┘
```

#### <a name="chapter1part7"></a>Chapter 1 - Part 7: Select just Even Numbers 

OBS: Using DuckDB

```sql
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
  (1, 'AA', 'IT', 'ABC', 100000),
  (2, 'BB', 'DE', 'DEF', 99999),
  (3, 'CC', 'FR', 'GHI', 100001);

SELECT * FROM EXAMPLE;

┌───────┬─────────┬─────────────┬──────────┬────────────┐
│  ID   │  NAME   │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │ varchar │   varchar   │ varchar  │   int32    │
├───────┼─────────┼─────────────┼──────────┼────────────┤
│     1 │ AA      │ IT          │ ABC      │     100000 │
│     2 │ BB      │ DE          │ DEF      │      99999 │
│     3 │ CC      │ FR          │ GHI      │     100001 │
└───────┴─────────┴─────────────┴──────────┴────────────┘

SELECT DISTINCT
      *
      FROM EXAMPLE
      WHERE
      CASE
          WHEN MOD(ID,2) = 0
          THEN 1
          ELSE 0
      END;

┌───────┬─────────┬─────────────┬──────────┬────────────┐
│  ID   │  NAME   │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │ varchar │   varchar   │ varchar  │   int32    │
├───────┼─────────┼─────────────┼──────────┼────────────┤
│   2   │ BB      │ DE          │ DEF      │   99999    │
└───────┴─────────┴─────────────┴──────────┴────────────┘
```
#### <a name="chapter1part8"></a>Chapter 1 - Part 8: Quantify Data Redundancy 

OBS: Using DuckDB

Find the difference between the total number of COUNTRYCODE entries in the table and the number of distinct COUNTRYCODE entries in the table

```sql
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
(1, 'AA', 'IT', 'ABC', 100000),
(2, 'BB', 'IT', 'DEF', 99999),
(3, 'CC', 'FR', 'GHI', 100001);

SELECT COUNT(COUNTRYCODE) - COUNT(DISTINCT COUNTRYCODE) AS Difference FROM EXAMPLE;

┌────────────┐
│ Difference │
│   int64    │
├────────────┤
│     1      │
└────────────┘
```

#### <a name="chapter1part9"></a>Chapter 1 - Part 9: Count the Number of Ocorrences in a String

OBS: Using DuckDB

```sql
SELECT (LENGTH('apple banana apple orange apple') - LENGTH(REPLACE('apple banana apple orange apple', 'apple', ''))) / LENGTH('apple') as count;

┌────────┐
│ count  │
│ double │
├────────┤
│    3.0 │
└────────┘
```

#### <a name="chapter1part10"></a>Chapter 1 - Part 10: Find the Min and Max Length of a String and Ordered Alphabetically

OBS: Using DuckDB

**Using Union**

```sql
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
(1, 'Abruzzo', 'IT', 'ABC', 100000),
(2, 'Roma', 'IT', 'DEF', 99999),
(3, 'Paris', 'FR', 'GHI', 100001),
(4, 'Lima', 'PE', 'JKL', 101001);

SELECT * FROM EXAMPLE;

┌───────┬─────────┬─────────────┬──────────┬────────────┐
│  ID   │  NAME   │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │ varchar │   varchar   │ varchar  │   int32    │
├───────┼─────────┼─────────────┼──────────┼────────────┤
│     1 │ Abruzzo │ IT          │ ABC      │     100000 │
│     2 │ Roma    │ IT          │ DEF      │      99999 │
│     3 │ Paris   │ FR          │ GHI      │     100001 │
│     4 │ Lima    │ PE          │ JKL      │     101001 │
└───────┴─────────┴─────────────┴──────────┴────────────┘

WITH name_lengths AS (
  SELECT NAME, LENGTH(NAME) nameLength FROM EXAMPLE ORDER BY NAME),
min_length AS (
  SELECT NAME, nameLength FROM name_lengths WHERE nameLength = (SELECT MIN(nameLength) FROM name_lengths) LIMIT 1),
max_length AS (
  SELECT NAME, nameLength FROM name_lengths WHERE nameLength = (SELECT MAX(nameLength) FROM name_lengths) LIMIT 1),
final_query AS (
  SELECT * FROM min_length
  UNION
  SELECT * FROM max_length
)

SELECT * FROM final_query ORDER BY NAME;

┌─────────┬────────────┐
│  NAME   │ nameLength │
│ varchar │   int64    │
├─────────┼────────────┤
│ Abruzzo │          7 │
│ Lima    │          4 │
└─────────┴────────────┘
```

**Using Inner Joins**

```sql
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
(1, 'Abruzzo', 'IT', 'ABC', 100000),
(2, 'Roma', 'IT', 'DEF', 99999),
(3, 'Paris', 'FR', 'GHI', 100001),
(4, 'Lima', 'PE', 'JKL', 101001);

SELECT * FROM EXAMPLE;

┌───────┬─────────┬─────────────┬──────────┬────────────┐
│  ID   │  NAME   │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │ varchar │   varchar   │ varchar  │   int32    │
├───────┼─────────┼─────────────┼──────────┼────────────┤
│     1 │ Abruzzo │ IT          │ ABC      │     100000 │
│     2 │ Roma    │ IT          │ DEF      │      99999 │
│     3 │ Paris   │ FR          │ GHI      │     100001 │
│     4 │ Lima    │ PE          │ JKL      │     101001 │
└───────┴─────────┴─────────────┴──────────┴────────────┘

WITH name_lengths AS (
  SELECT NAME, LENGTH(NAME) nameLength FROM EXAMPLE ORDER BY NAME),
min_length AS (
  SELECT NAME, nameLength FROM name_lengths WHERE nameLength = (SELECT MIN(nameLength) FROM name_lengths) LIMIT 1),
max_length AS (
  SELECT NAME, nameLength FROM name_lengths WHERE nameLength = (SELECT MAX(nameLength) FROM name_lengths) LIMIT 1)

SELECT nl.NAME, nl.nameLength
FROM name_lengths nl
INNER JOIN min_length minl
ON minl.NAME = nl.NAME
UNION
SELECT nl.NAME, nl.nameLength
FROM name_lengths nl
INNER JOIN max_length maxl
ON maxl.NAME = nl.NAME;

┌─────────┬────────────┐
│  NAME   │ nameLength │
│ varchar │   int64    │
├─────────┼────────────┤
│ Abruzzo │          7 │
│ Lima    │          4 │
└─────────┴────────────┘
```
