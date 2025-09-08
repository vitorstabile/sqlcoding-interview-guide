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
    - [Chapter 1 - Part 11: Pattern Matching with LIKE and REGEX](#chapter1part11)
    - [Chapter 1 - Part 12: Applying Business Rules with CTEs and Joins.](#chapter1part12)
    - [Chapter 1 - Part 13: Fill the Gaps of a Table (gaps and islands)](#chapter1part13)
2. [Chapter 2: Challenges](#chapter2)
    - [Chapter 1 - Part 1: Conditional Aggregation with Window Functions and CTEs. (Pricing Rule Engine Logic)](#chapter1part1)
    - [Chapter 2 - Part 2: Find the second highest distinct salary](#chapter1part2)
    - [Chapter 3 - Part 3: Return the employees who have a salary higher than a manager's.](#chapter1part3)
    - [Chapter 3 - Part 4: Find the ids of products that are both low fat and recyclable](#chapter1part4)
    - [Chapter 3 - Part 5: Find managers with at least five direct reports](#chapter1part5)
  
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

#### <a name="chapter1part11"></a>Chapter 1 - Part 11: Pattern Matching with LIKE and REGEX

OBS: Using DuckDB

Find Cities that starts with a,e,i,o or u

**Using Like**

```sql
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
(1, 'Abruzzo', 'IT', 'ABC', 100000),
(2, 'Roma', 'IT', 'DEF', 99999),
(3, 'Paris', 'FR', 'GHI', 100001),
(4, 'Lima', 'PE', 'JKL', 101001),
(5, 'Abruzzo', 'IT', 'ABC', 100000),
(6, 'Edinburgh', 'GB', 'MNO', 100000);

SELECT * FROM EXAMPLE;

┌───────┬───────────┬─────────────┬──────────┬────────────┐
│  ID   │   NAME    │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │  varchar  │   varchar   │ varchar  │   int32    │
├───────┼───────────┼─────────────┼──────────┼────────────┤
│     1 │ Abruzzo   │ IT          │ ABC      │     100000 │
│     2 │ Roma      │ IT          │ DEF      │      99999 │
│     3 │ Paris     │ FR          │ GHI      │     100001 │
│     4 │ Lima      │ PE          │ JKL      │     101001 │
│     5 │ Abruzzo   │ IT          │ ABC      │     100000 │
│     6 │ Edinburgh │ GB          │ MNO      │     100000 │
└───────┴───────────┴─────────────┴──────────┴────────────┘

SELECT DISTINCT NAME FROM EXAMPLE WHERE 
LOWER(NAME) LIKE 'a%' OR 
LOWER(NAME) LIKE 'e%' OR 
LOWER(NAME) LIKE 'i%' OR
LOWER(NAME) LIKE 'o%' OR
LOWER(NAME) LIKE 'u%';

┌───────────┐
│   NAME    │
│  varchar  │
├───────────┤
│ Abruzzo   │
│ Edinburgh │
└───────────┘
```

**Using Regex**

```sql
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
(1, 'Abruzzo', 'IT', 'ABC', 100000),
(2, 'Roma', 'IT', 'DEF', 99999),
(3, 'Paris', 'FR', 'GHI', 100001),
(4, 'Lima', 'PE', 'JKL', 101001),
(5, 'Abruzzo', 'IT', 'ABC', 100000),
(6, 'Edinburgh', 'GB', 'MNO', 100000);

SELECT * FROM EXAMPLE;

┌───────┬───────────┬─────────────┬──────────┬────────────┐
│  ID   │   NAME    │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │  varchar  │   varchar   │ varchar  │   int32    │
├───────┼───────────┼─────────────┼──────────┼────────────┤
│     1 │ Abruzzo   │ IT          │ ABC      │     100000 │
│     2 │ Roma      │ IT          │ DEF      │      99999 │
│     3 │ Paris     │ FR          │ GHI      │     100001 │
│     4 │ Lima      │ PE          │ JKL      │     101001 │
│     5 │ Abruzzo   │ IT          │ ABC      │     100000 │
│     6 │ Edinburgh │ GB          │ MNO      │     100000 │
└───────┴───────────┴─────────────┴──────────┴────────────┘

SELECT DISTINCT NAME FROM EXAMPLE WHERE regexp_matches(NAME, '^(?:[aeiouAEIOU]|[aeiouAEIOU].*[aeiouAEIOU])$');

┌─────────┐
│  NAME   │
│ varchar │
├─────────┤
│ Abruzzo │
└─────────┘
```

#### <a name="chapter1part12"></a>Chapter 1 - Part 12: Applying Business Rules with CTEs and Joins.

Calculate the effective price for each product, considering active discounts.

OBS: Using DuckDB

```sql
CREATE TABLE products (
	sku VARCHAR(255) PRIMARY KEY,
	name VARCHAR(255),
	category VARCHAR(255),
	color VARCHAR(255),
	size VARCHAR(10),
	price DECIMAL(10,2)
);

CREATE TABLE pricing (
	sku VARCHAR(255),
	barcode VARCHAR(255),
	discount_percent DECIMAL(10,2),
	discount_start_date DATE,
	discount_end_date DATE,
	FOREIGN KEY (sku) REFERENCES products(sku)
);

-- Insert data into the 'products' table
INSERT INTO products (sku, name, category, color, size, price) VALUES
('TS-001', 'Classic Cotton Tee', 'Tops', 'Blue', 'M', 25.00),
('TS-002', 'Striped Linen Shirt', 'Tops', 'White', 'L', 45.50),
('PT-005', 'Slim Fit Jeans', 'Bottoms', 'Blue', '30', 60.00),
('SK-012', 'A-Line Denim Skirt', 'Bottoms', 'Blue', 'S', 35.75),
('DR-021', 'Summer Floral Dress', 'Dresses', 'Multi', 'M', 75.00),
('SW-008', 'Wool Knit Sweater', 'Tops', 'Grey', 'L', 55.20),
('PT-006', 'Chino Trousers', 'Bottoms', 'Beige', '32', 52.99);

-- Insert data into the 'pricing' table
INSERT INTO pricing (sku, barcode, discount_percent, discount_start_date, discount_end_date) VALUES
('TS-001', '1234567890123', 10.00, '2025-05-01', '2025-05-10'),
('PT-005', '9876543210987', 15.50, '2025-04-20', '2025-05-07'),
('SK-012', '1122334455667', 20.00, '2025-05-05', '2025-05-15'),
('TS-001', '1234567890123', 5.00, '2025-03-15', '2025-03-31'),
('DR-021', '5566778899001', 12.75, '2025-05-03', '2025-05-06'),
('SW-008', '2233445566778', NULL, NULL, NULL),
('PT-006', '3344556677889', 8.00, '2025-05-12', '2025-05-20');
```

**Calculating the Effective Price  of all products using CASE and RIGHT JOIN**

```sql
WITH active_discounts AS (
	SELECT
		sku,
        MAX(CASE
            WHEN CURRENT_DATE BETWEEN discount_start_date AND discount_end_date
            THEN discount_percent
            ELSE 0
        END) AS discount_percent
    FROM pricing
    GROUP BY sku
)

SELECT
    prod.sku AS sku,
    price AS price,
    COALESCE(discount_percent, 0.00) AS discount,
    (price - price*(COALESCE(discount_percent/100,0))) AS effective_price
FROM active_discounts AS ac
RIGHT JOIN products AS prod
ON prod.sku = ac.sku;

┌─────────┬───────────────┬───────────────┬─────────────────┐
│   sku   │     price     │   discount    │ effective_price │
│ varchar │ decimal(10,2) │ decimal(12,2) │     double      │
├─────────┼───────────────┼───────────────┼─────────────────┤
│ TS-001  │         25.00 │         10.00 │            22.5 │
│ PT-005  │         60.00 │         15.50 │            50.7 │
│ SK-012  │         35.75 │         20.00 │            28.6 │
│ DR-021  │         75.00 │         12.75 │         65.4375 │
│ SW-008  │         55.20 │          0.00 │            55.2 │
│ PT-006  │         52.99 │          0.00 │           52.99 │
│ TS-002  │         45.50 │          0.00 │            45.5 │
└─────────┴───────────────┴───────────────┴─────────────────┘
```

**Calculating the Effective Price of all products using WHERE, RIGHT JOIN**

```sql
WITH active_discounts AS (
	SELECT 
		sku,
		MAX(discount_percent) AS discount_percent
	FROM pricing
	WHERE CURRENT_DATE BETWEEN discount_start_date AND discount_end_date
	GROUP BY sku
)

SELECT
    prod.sku AS sku,
    price AS price,
    COALESCE(discount_percent, 0.00) AS discount,
    (price - price*(COALESCE(discount_percent/100,0))) AS effective_price
FROM active_discounts AS ac
RIGHT JOIN products AS prod
ON prod.sku = ac.sku;

┌─────────┬───────────────┬───────────────┬─────────────────┐
│   sku   │     price     │   discount    │ effective_price │
│ varchar │ decimal(10,2) │ decimal(10,2) │     double      │
├─────────┼───────────────┼───────────────┼─────────────────┤
│ TS-001  │         25.00 │         10.00 │            22.5 │
│ PT-005  │         60.00 │         15.50 │            50.7 │
│ SK-012  │         35.75 │         20.00 │            28.6 │
│ DR-021  │         75.00 │         12.75 │         65.4375 │
│ TS-002  │         45.50 │          0.00 │            45.5 │
│ SW-008  │         55.20 │          0.00 │            55.2 │
│ PT-006  │         52.99 │          0.00 │           52.99 │
└─────────┴───────────────┴───────────────┴─────────────────┘
```

**Calculating the Effective Price of all products using WHERE, RIGHT JOIN**

```sql
WITH ActiveDiscounts AS (
    SELECT
        sku,
        MAX(discount_percent) AS max_discount_percent
    FROM pricing
    WHERE CURRENT_DATE BETWEEN discount_start_date AND discount_end_date
    GROUP BY sku
)
SELECT
    p.sku,
    p.name,
    p.price,
    ad.max_discount_percent,
    p.price * (1 - ad.max_discount_percent / 100) AS effective_price
FROM products p
LEFT JOIN ActiveDiscounts ad ON p.sku = ad.sku;

┌─────────┬─────────────────────┬───────────────┬──────────────────────┬────────────────────┐
│   sku   │        name         │     price     │ max_discount_percent │  effective_price   │
│ varchar │       varchar       │ decimal(10,2) │    decimal(10,2)     │       double       │
├─────────┼─────────────────────┼───────────────┼──────────────────────┼────────────────────┤
│ TS-001  │ Classic Cotton Tee  │         25.00 │                10.00 │               22.5 │
│ PT-005  │ Slim Fit Jeans      │         60.00 │                15.50 │ 50.699999999999996 │
│ SK-012  │ A-Line Denim Skirt  │         35.75 │                20.00 │               28.6 │
│ DR-021  │ Summer Floral Dress │         75.00 │                12.75 │            65.4375 │
│ TS-002  │ Striped Linen Shirt │         45.50 │                      │                    │
│ SW-008  │ Wool Knit Sweater   │         55.20 │                      │                    │
│ PT-006  │ Chino Trousers      │         52.99 │                      │                    │
└─────────┴─────────────────────┴───────────────┴──────────────────────┴────────────────────┘
```

#### <a name="chapter1part13"></a>Chapter 1 - Part 13: Fill the Gaps of a Table (gaps and islands)

Fill the Gaps of the Column1 until reach the next value of the Column2

```
Input:

Column1|Column2|Column3
A      |123    |Value1
       |234    |Value2
       |345    |Value3
       |456    |Value4
B      |1010   |AnotherValue1
       |1111   |AnotherValue2
       |1212   |AnotherValue3
       |1313   |AnotherValu4

Output:

Column1|Column2|Column3
A      |123    |Value1
A      |234    |Value2
A      |345    |Value3
A      |456    |Value4
B      |1010   |AnotherValue1
B      |1111   |AnotherValue2
B      |1212   |AnotherValue3
B      |1313   |AnotherValu4
```

This is a classic "gaps and islands" problem in SQL. The best way to solve this is by using a combination of window functions, which are very powerful for these types of tasks.

Here's a breakdown of the logic and the SQL query to get your desired result.

**The Strategy**

- Identify the "islands": The first step is to create a way to group rows together. A new group starts whenever Column1 is not NULL.

- Assign a group ID: We can use a window function to assign a unique ID to each group. A common trick is to count the non-NULL values in Column1 up to the current row. This count will only increase when Column1 has a ---- value, effectively creating a group ID.

- Fill down the value: Once you have these groups, you can use another window function to get the first non-NULL value within each group.

**The SQL Query**

Assuming your table is named my_table with columns Column1, Column2, and Column3, and that the order is determined by Column2:

```sql
CREATE TABLE 'raw_table' AS SELECT row_number() OVER () AS line_number, * FROM read_csv('C:\Users\vitor.garcia\Downloads\example.csv', all_varchar=True);

┌─────────────┬─────────┬─────────┬───────────────┐
│ line_number │ Column1 │ Column2 │    Column3    │
│    int64    │ varchar │ varchar │    varchar    │
├─────────────┼─────────┼─────────┼───────────────┤
│           1 │ A       │ 123     │ Value1        │
│           2 │         │ 234     │ Value2        │
│           3 │         │ 345     │ Value3        │
│           4 │         │ 456     │ Value4        │
│           5 │ B       │ 1010    │ AnotherValue1 │
│           6 │         │ 1111    │ AnotherValue2 │
│           7 │         │ 1212    │ AnotherValue3 │
│           8 │         │ 1313    │ AnotherValu4  │
└─────────────┴─────────┴─────────┴───────────────┘

SELECT
    MAX(t1.Column1) OVER (PARTITION BY t2.group_id ORDER BY t1.line_number) AS Column1,
    t1.Column2,
    t1.Column3
FROM raw_table AS t1
JOIN (
      SELECT
      line_number,
      COUNT(Column1) OVER (ORDER BY line_number) AS group_id
FROM raw_table ) AS t2
ON t1.line_number = t2.line_number
ORDER BY t1.line_number;

┌─────────┬─────────┬───────────────┐
│ Column1 │ Column2 │    Column3    │
│ varchar │ varchar │    varchar    │
├─────────┼─────────┼───────────────┤
│ A       │ 123     │ Value1        │
│ A       │ 234     │ Value2        │
│ A       │ 345     │ Value3        │
│ A       │ 456     │ Value4        │
│ B       │ 1010    │ AnotherValue1 │
│ B       │ 1111    │ AnotherValue2 │
│ B       │ 1212    │ AnotherValue3 │
│ B       │ 1313    │ AnotherValu4  │
└─────────┴─────────┴───────────────┘
```

## <a name="chapter2"></a>Chapter 2: Challenges

#### <a name="chapter2part1"></a>Chapter 2 - Part 1: Conditional Aggregation with Window Functions and CTEs. (Pricing Rule Engine Logic)

In a group of code, get the Max brandPrice based in the netPrice. If the NetPrice is equal, get the
-- max brandPrice. If the NetPrice is different, get the Max Brand Price of the Max NetPrice. In the end, concat the code with the id

OBS: Using DuckDB

```
Input:

code |id |netprice|brandPrice
code1|id1|     200|320
code1|id2|     200|320
code1|id3|     200|300
code2|id1|     100|110
code2|id2|      90|110
code2|id3|      90|105
code3|id1|      10|15

OutPut:

┌───────────┬─────────────────┐
│ skuValue  │ brandPriceValue │
│  varchar  │  decimal(18,3)  │
├───────────┼─────────────────┤
│ code1#id1 │         320.000 │
│ code1#id2 │         320.000 │
│ code1#id3 │         320.000 │
│ code2#id1 │         110.000 │
│ code2#id2 │         110.000 │
│ code2#id3 │         110.000 │
│ code3#id1 │          15.000 │
└───────────┴─────────────────┘

```

```sql
CREATE TABLE 'raw_table' AS SELECT row_number() OVER () AS line_number, * FROM read_csv('C:\Users\my.user\Downloads\input_example.csv', all_varchar=True);

WITH RemoveNegativeDiscounts AS (
    SELECT
        code AS code,
        id AS id,
        TRY_CAST(netprice AS DECIMAL) AS netprice,
        TRY_CAST(brandPrice AS DECIMAL) AS brandPrice
    FROM raw_table
),
RankNetPrice AS (
    SELECT
        code AS code,
        id AS id,
        netprice AS netprice,
        brandPrice AS brandPrice,
        DENSE_RANK() OVER (PARTITION BY code ORDER BY netprice DESC) AS netprice_rank
    FROM RemoveNegativeDiscounts
),
rank_info AS (
    SELECT 
        code,
        MIN(netprice_rank) AS min_rank,
        MAX(netprice_rank) AS max_rank
    FROM RankNetPrice
    GROUP BY code
),
GetDiscountBasedInRankAndNetPrice AS (
	SELECT 
		t.code AS code,
		CASE 
			WHEN r.min_rank = r.max_rank 
				THEN MAX(t.brandPrice)
			ELSE (SELECT MAX(t.brandPrice) FROM RankNetPrice t WHERE t.code = r.code AND t.netprice_rank = 1)
		END AS result_price
	FROM RankNetPrice t
	JOIN rank_info r 
    ON t.code = r.code
	GROUP BY t.code, r.code, r.min_rank, r.max_rank
),
RetrieveValues AS (
	SELECT
        CONCAT(gdb.code,'#',rnd.id) AS skuValue,
        gdb.result_price AS brandPriceValue
    FROM RemoveNegativeDiscounts rnd
    JOIN GetDiscountBasedInRankAndNetPrice gdb
    ON gdb.code = rnd.code
)

SELECT * FROM RetrieveValues;

┌───────────┬─────────────────┐
│ skuValue  │ brandPriceValue │
│  varchar  │  decimal(18,3)  │
├───────────┼─────────────────┤
│ code1#id1 │         320.000 │
│ code1#id2 │         320.000 │
│ code1#id3 │         320.000 │
│ code2#id1 │         110.000 │
│ code2#id2 │         110.000 │
│ code2#id3 │         110.000 │
│ code3#id1 │          15.000 │
└───────────┴─────────────────┘
```

#### <a name="chapter2part2"></a>Chapter 2 - Part 2: Find the second highest distinct salary

Write a solution to find the second highest distinct salary from the Employee table. If there is no second highest salary, return null (return None in Pandas).

The result format is in the following example.

OBS: Using DuckDB

```
Input: 
Employee table:
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
Output: 
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+

Example 2:

Input: 
Employee table:
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
+----+--------+
Output: 
+---------------------+
| SecondHighestSalary |
+---------------------+
| null                |
+---------------------+
```

```sql
-- This MAX is necessary to return a NULL value case there is no rows return
SELECT MAX(salary) AS SecondHighestSalary
FROM (
	WITH cast_cte AS (
		SELECT
			CAST(id AS INTEGER) AS id,
			CAST(salary AS INTEGER) AS salary
	FROM raw_table
	),
	rank_salary AS (
		SELECT
			id,
			salary,
			DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
		FROM cast_cte
)
SELECT salary FROM rank_salary WHERE salary_rank = 2
) AS subquery_result;
```

#### <a name="chapter2part3"></a>Chapter 2 - Part 3: Return the employees who have a salary higher than a manager's.

```
employe_id|name  |salary|department|manager_id
111       |John  |10000 |IT        |
222       |Maria |20000 |Sales     |
333       |Mike  |30000 |Maketing  |4040
444       |Ana   |5000  |IT        |
555       |Jason |7000  |HR        |1010
666       |July  |8000  |Marketing |
```

Employees that not have manager_id, is not a manager

OBS: Using DuckDB

```sql
WITH manager_salary AS (
	SELECT
		MIN(TRY_CAST(salary AS INTEGER)) AS min_manager_salary
	FROM raw_table
	WHERE manager_id NOT NULL AND manager_id != ''
),
employer_table AS (
	SELECT
		rt.employe_id,
		rt.name,
		rt.salary,
		rt.department
	FROM raw_table rt
	WHERE (rt.manager_id IS NULL OR rt.manager_id = '') AND TRY_CAST(rt.salary AS INTEGER) > (SELECT ms.min_manager_salary FROM manager_salary ms)
)

SELECT * FROM employer_table;
```

#### <a name="chapter2part4"></a>Chapter 2 - Part 4: Find the ids of products that are both low fat and recyclable

Write a solution to find the ids of products that are both low fat and recyclable.

Return the result table in any order.

The result format is in the following example.

```
Input: 
Products table:
+-------------+----------+------------+
| product_id  | low_fats | recyclable |
+-------------+----------+------------+
| 0           | Y        | N          |
| 1           | Y        | Y          |
| 2           | N        | Y          |
| 3           | Y        | Y          |
| 4           | N        | N          |
+-------------+----------+------------+
Output: 
+-------------+
| product_id  |
+-------------+
| 1           |
| 3           |
+-------------+
Explanation: Only products 1 and 3 are both low fat and recyclable.
```

OBS: Using DuckDB

```sql
SELECT product_id FROM Products WHERE low_fats = 'Y' AND recyclable = 'Y';
```

#### <a name="chapter2part5"></a>Chapter 2 - Part 5: Find managers with at least five direct reports

Write a solution to find managers with at least five direct reports.

Return the result table in any order.

The result format is in the following example.

```
Input: 
Employee table:
+-----+-------+------------+-----------+
| id  | name  | department | managerId |
+-----+-------+------------+-----------+
| 101 | John  | A          | null      |
| 102 | Dan   | A          | 101       |
| 103 | James | A          | 101       |
| 104 | Amy   | A          | 101       |
| 105 | Anne  | A          | 101       |
| 106 | Ron   | B          | 101       |
+-----+-------+------------+-----------+
Output: 
+------+
| name |
+------+
| John |
+------+
```

OBS: Using DuckDB

```sql
WITH cast_table AS (
	SELECT
		CAST(id AS INTEGER) AS id,
		name AS name,
		department AS department,
		CAST(managerId AS INTEGER) AS managerId
	FROM Employee
),
join_table AS (
	SELECT
		ct1.name AS name
	FROM cast_table ct1
	INNER JOIN cast_table ct2
	ON ct1.id = ct2.managerId
	GROUP BY ct1.name, ct1.id
	HAVING COUNT(*) >= 5
)

SELECT * FROM join_table;
```
