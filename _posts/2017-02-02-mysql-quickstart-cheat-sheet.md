---
layout: post
title: MySQL Quick Start and Cheat Sheet
permalink: blog/mysql-quickstart-cheat-sheet/
comments: True
excerpt_separator: <!--more-->
---

Although my favorite SQL database is PostgreSQL due to its feature-rich capabilities and amazing performance, MySQL still has its benefits, especially in the replication side. In fact, Google Cloud SQL is only compatible with MySQL right now. If they offered PostgreSQL, I would not be writing this. But I'll stop now on opinions. Let's do a quick start, and compile a cheat sheet afterwards. Also, check out some tips I learned including some best practices.
<!--more-->
## Quick Start

```sh
service mysql start     # start mysql if not started
mysql -u root -p        # login with password prompt
```

### Setup User and Database

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password'; -- change root pass
```

```sql
SHOW DATABASES;
CREATE DATABASE cats;

USE cats;
SHOW TABLES;
```

```sql
CREATE USER 'jump'@'localhost' IDENTIFIED BY 'secret'; -- create a new user
GRANT ALL ON cats.* TO 'jump'@'localhost';             -- grant privileges
FLUSH PRIVILEGES;                                      -- refresh privileges
SHOW GRANTS;                                           -- show users and grants
```

### Create Tables

```sql
CREATE TABLE cats
(
  id              INT unsigned NOT NULL AUTO_INCREMENT, -- Unique ID
  name            VARCHAR(150) NOT NULL,                -- Name of the cat
  owner           VARCHAR(150) NOT NULL,                -- Owner of the cat
  birth           DATE NOT NULL,                        -- Birthday of the cat
  PRIMARY KEY     (id)                                  -- make id primary key
);

ALTER TABLE cats ADD gender CHAR(1) AFTER name;
ALTER TABLE cats DROP gender;

DESCRIBE cats;
```

### CRUD

```sql
INSERT INTO cats ( name, owner, birth) VALUES
  ( 'Sandy', 'Lennon', '2015-01-03' ),
  ( 'Cookie', 'Casey', '2013-11-13' ),
  ( 'Charlie', 'River', '2016-05-21' );
```

```sql 
SELECT * FROM cats;
SELECT name FROM cats WHERE owner = 'Casey';

DELETE FROM cats WHERE name='Cookie';
```

## Cheat Sheet

## Common Shell Commands

```sh
# run the mysql client
mysql -h [host] -u [user] -p 
mysql -h [host] -u [user] -p [database]

# export data
mysqldump -u [user] -p [database] > data_backup.sql
```

## Common MySQL Commands

```sql
\s;                 -- show status
SHOW VARIABLES;     -- show configuration params
SHOW PROCESSLIST;   -- show running queries

SELECT @@version;   -- mysql version
SELECT @@datadir;   -- location of db files
SELECT @@hostname;  -- current hostname
SELECT USER();      -- current user
SELECT DATABASE();  -- current database

SHOW DATABASES;
USE db_name;
SHOW TABLES;
DESCRIBE table_name;
```

## Common Admin Commands 

```sql 
-- list all mysql users in database
SELECT host, user FROM mysql.user;  

-- show grant for all users
SHOW GRANTS;
SHOW GRANTS FOR 'user';

CREATE USER 'username'@'hostname' IDENTIFIED BY 'password';

GRANT ALL ON db_name.* TO 'username'@'hostname' IDENTIFIED BY 'password';

-- refresh privileges
FLUSH PRIVILEGES;

DROP USER 'username'@'hostname';
```

## Common Data Types

|PURPOSE|EXAMPLE|NOTES|
|-|-|-|
|integers|`int(5)`|
|floats|`float(12,3)`|Will round your values|
|money|`decimal(10,2)`|Up to 10 digits. 2 points|
|date|`date`|
|datetime|`timestamp(8)`|(8)YYYYMMDD, (12)YYYYMMDDHHMMSS|
|string|`varchar(20)`|
|large text|`blob`|
|enum|`enum('blue','red','gray')`|Not recommended to use|

## Common Functions

|FUNCTION|NOTES|
|-|-|
|`NOW()`|datetime input|
|`strcomp(str1,str2)`|Compare string|
|`lower(str)`|
|`upper(str)`|
|`ltrim(str)`|
|`substring(str,idx1,idx2)`|
|`password(str)`|Encry password|
|`curdate()`|Get date|
|`curtime()`|Get time|

## Common CRUD

```sql
-- INSERT
INSERT INTO people VALUES ('MyName', '2002­08­31');             
INSERT INTO people (name, company_id, created_at) -- copy rows from same tbl
    SELECT name, 50, NOW() FROM people WHERE company_id = 49; 

-- SELECT
SELECT * FROM tbl;                              -- All values 
SELECT * FROM tbl WHERE rec_name = "value";     -- Some values
SELECT * FROM tbl WHERE rec1 = "value1"         -- Multiple critera
    AND rec2 = "val2"; 

SELECT column_name FROM table;                    -- Selecting specific columns
SELECT DISTINCT column_name FROM table;           -- Retrieving unique outputs
SELECT col1, col2 FROM table ORDER BY col2;       -- Sorting
SELECT col1, col2 FROM table ORDER BY col2 DESC;  -- Sorting Backward
SELECT COUNT(*) FROM table;                       -- Counting Rows
SELECT owner, COUNT(*) FROM table GROUP BY owner; -- Grouping with Counting
SELECT MAX(col_name) AS label FROM table;         -- Maximum value

SELECT pet.name, comment FROM pet, event -- Selecting from multiple tables 
WHERE pet.name = event.name;             

-- SEARCH

SELECT * FROM table WHERE rec LIKE "blah%"; -- % is wildcard ­
SELECT * FROM table WHERE rec LIKE "_____"; -- Find 5­char values: _ is 1 char
SELECT * FROM table WHERE rec RLIKE "^b$";  -- regex

-- JOINS
SELECT * FROM table_1 INNER JOIN table_2 ON conditions;
SELECT * FROM table1 LEFT JOIN table2 ON conditions;

-- UPDATE
UPDATE table SET column_name = "new_value" 
WHERE record_name = "value";

-- DELETE 
DELETE FROM table WHERE condition;

DELETE table_1, table2 FROM table_1
INNER JOIN table_2 ON table_1.column_1 = table_2.column_2
WHERE condition;
```

### Joins 

**INNER JOIN** returns rows when there is at least one match in both tables
based on the condition given

```sql
SELECT t1.*, t2.* FROM table1 t1
INNER JOIN table2 t2 ON t1.ID = t2.ID
```

```sql
-- adding alias to header on columns
SELECT  t1.ID AS t1_id, t1.Value AS t1_v, 
        t2.ID t2_id, t2.Value AS t2_v 
FROM table1 t1
INNER JOIN table2 t2 ON t1.ID = t2.ID
```

**LEFT OUTER JOIN** returns all the rows from the left table with the matching 
rows from the right table. If no columns in right matches, it returns NULL.

**RIGHT OUTER JOIN** returns all the rows from the right table with the matching 
rows from the left table. If no columns in left matches, it returns NULL.

## Common Table Manipulations

```sql
CREATE TABLE table_name (field1_name TYPE(SIZE), field2_name TYPE(SIZE));  

DROP TABLE table_name

ALTER TABLE authors 
    ADD     name VARCHAR(255), 
    CHANGE  author_work_id wokr_id INT,
    DROP    nickname,
    CHANGE `count(*)` cnt bigint(21),  ### renaming
    ALTER   is_rich SET DEFAULT FALSE;

-- Adding a column to an already­created table
ALTER TABLE tbl ADD COLUMN [column_create syntax] AFTER col_name;

-- Removing a column
ALTER TABLE tbl DROP COLUMN col;

-- Adding index to an existing table;
ALTER TABLE table_name ADD INDEX index_name (col_name);
```

## Tips

- Always use proper datatype. For example, don't use `VARCHAR(20)` instead of 
`DATETIME` since it will lead to errors and may store invalid data.
- Use `CHAR(1)` over `VARCHAR(1)` when storing a single character to save space.
- If it's a fixed length, use `CHAR` data type. If not, use `VARCHAR`.
- When using `DATETIME` or `DATE` datatype, always use the `YYYY-MM-DD` date 
formate or ISO date format that suits your SQL engine. Avoid regional formats.
- Make sure you index the columns that are used in the join clauses so the query 
returns the result fast. 
- Do not use functions over indexed columns since it defeats the purpose of 
index. For example, instead of `left(code,2)='CA'` rewrite with `code LIKE 'CA%`
- Use `SELECT *` only when needed. Be explicit and don't blindly use that.
- Use `ORDER BY` only if needed since it is a slow process.
- Chose proper Database Engine. If you app reads more often than write, choose 
MyISAM storage engine. If you develop an application that writes data more than 
reading (for example, bank transactions), choose INNODB storage engine. Choosing
the wrong engine affects performance.
- If you want to check the existence of data, use `IF EXISTS(SELECT*...)`
- Code with cache in mind. Instead of `WHERE date >= CURDATE()`, store date 
value in a variable in your application code. Running dynamic functions 
invalidate the cache.
- Use `LIMIT 1` in your query if you're just looking for 1 unique item. This
allows the database to stop scanning once it finds that specific item. 
`SELECT 1 FROM user WHERE state = 'CA' LIMIT 1`
- Indexes are not just for primary or unique keys. If there are any columns you 
will search by, you should almost always index them.
- When using `JOINS` make sure that the columns you join are index in BOTH sides
- Also, make sure the column you join have the same data type.
- Use `ENUM` over `VARCHAR` if using only few values, ie: active, inactive, 
pending, expired
- Unless you have a very specific reason to use a `NULL` value, you should
always set your colums as `NOT NULL`. If there's no reason to have 0 vs NULL,
you don't need it - they require additional space and add complexity.
