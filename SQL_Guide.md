# SQL Guide

This repository contains my notes after following as refreshers the [SQL Fundamentals skill track](https://app.datacamp.com/learn/skill-tracks/sql-fundamentals) from Datacamp and [The Complete SQL Bootcamp](https://www.udemy.com/course/the-complete-sql-bootcamp/) by José Marcial Portilla from Udemy.

The [SQL Fundamentals skill track](https://app.datacamp.com/learn/skill-tracks/sql-fundamentals) is composed by the following courses:

1. [Introduction to SQL](https://app.datacamp.com/learn/courses/introduction-to-sql)
2. [Joining Data in SQL](https://app.datacamp.com/learn/courses/joining-data-in-sql)
3. [Intermediate SQL](https://app.datacamp.com/learn/courses/intermediate-sql)
4. [PostgreSQL Summary Stats and Window Functions](https://app.datacamp.com/learn/courses/postgresql-summary-stats-and-window-functions)
5. [Functions for Manipulating Data in PostgreSQL](https://app.datacamp.com/learn/courses/functions-for-manipulating-data-in-postgresql)

The [The Complete SQL Bootcamp](https://www.udemy.com/course/the-complete-sql-bootcamp/) is composed by the following sections:

1. Introduction & Setup
2. SQL Statement Fundamentals
3. GROUP BY Statements
4. JOINS
5. Advanced SQL Commands
6. Creating Databases and Tables
7. Conditional Expressions and Procedures
8. PostGreSQL with Python

The current file is a **summary/guide of my notes and SQL in general**.

**Overview**

1. Introduction & Setup

## 1. Introduction & Setup

Relational databases are collections of inter-related tables able to collect very large amounts of data.
SQL is the interfacing language with which we can interact with those relational databases.
Each table is like a spreadsheet tab (but can contain much more data): columns are fields, rows are entries.
There are many implementations of SQL, being [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) one of them: it is open source, free; PostgreSQL has additional methods, but they are clearly warned here.

Alternatives to PostgreSQL: MySQL, Oracle, etc.

### 1.1 Installation: `PostgreSQL` & `pgAdmin`

We install two things (on a Mac; Windows is equivalent):
- [PostgreSQL](https://www.postgresql.org): SQL engine to interact with our databases
- [pgAdmin](https://www.pgadmin.org): web-based GUI to interact with our databases using PostgreSQL; one of the most popular.

Documenation / Help:
- [PostgreSQL Documentation](https://www.postgresql.org/docs/current/)
- [pgAdmin Documentation](https://www.pgadmin.org/docs/pgadmin4/6.5/index.html)

In the following, installation instructions are provided following an indented style I use typically for howtos.
Also, an example database `dvdrentals` from the Udemy course is restored.

```

Install PostgreSQL
    Download PostgreSQL installer from web (V. 14.2): https://www.postgresql.org/download/macosx/
    Click on intaller
    Select/leave default
    Password
        o-y-a
        do not forget it, otherwise we need to re-install it again!
        PostgreDQL superuser pw
    Port: 5432 (leave suggested one)
        Make sure port is not blocked by any firewall
        PostgreSQL will be listening on that port
    Next, next, next
    Finish

Install pgAdmin
    Download pgAdmin from web: https://www.pgadmin.org/download/
        pgAdmin 4 - v6.5
        Download the DMG file: pgadmin4-6.5.dmg
    Click on DMG installed: drag & drop app to Applications folder

Restart computer
    A new postgres user might appear

Open pgAdmin
    A new pw is required: master pw for pgAdmin
        although it is a different one,
        we can use the same as for PostgreSQL
        o-y-a
    The browser or a web-based GUI is opened
    We can connect to engine servers on the left panel
        introduce postgres pw

Restore Example Database
    Udemy course material: example database file that should not be uncompressed:
    `./data/dvdrental.tar`

    On pgAdmin:
    Select PostgreSQL on left panel > Databases
        Right click: create: dvdrental
            New database appears on left panel
        Right click on dvdrental DB > Restore...
            Path to dvdrental.tar
            Options/data-object
                sections: activate
                    Data, Pre-Data, Post-Data
            Restore
        Right click on dvdrental DB > Refresh
            Changes take place!

Test we can interact with our restored DB using SQL
    On pgAdmin left panel: 
    Right click on dvdrental DB > Query Tool
    Insert
        SELECT * FROM film;
    Run (Play button)
        We get all entries from film table

```

### 1.2 `pgAdmin` Overview

We often interact with a database using SQL in a tool such as `pgAdmin`.
In the following, a schematic overview is given with an indented style.

```
Left panel: Servers: PostgreSQL engine is a server
    Databases hanging from it
    We can have several servers if we have several PostgreSQL versions installed
    Every time we connect to a server, we need to introduce the postgres user pw
Right side: Dashboard
    If selected a database, its activity is shown on dashboard
File > Preferences
    Themes (Dark)
    Query tool: font size, etc.
    Dashboards: choose what to display
We can select an object on the left panel
    and right click for options
    or on menu, then select Object / Tool

DB, right click -> Query Tool (dvdrental)
    Tabs
        Query Editor: we write our SQL queries here
        Query History: list of all SQL queries we have performed, very useful!
            Copy / Copy to Editor
        Output: result after running the query
    To execute a query: Run / Play / F5
```

## 2. SQL Statement Fundamentals

The way we are going to use to interact with the database (DB) `dvdrental` is `pgAdmin`. We select the database > right click > Query Tool. There, we input our queries, capitalizing by convention SQL tokens for readability, although that is not technically necessary.

A database (DB) is composed by tables that contain columns or fields. In `pgAdmin`, those tables and fields can be seen in the `Schemas`:
- select DB, expand its items
- item `Schemas`
- subitems `Tables`, `Columns`

Some general notes:
- Comments are created with `--`; that can be anywhere
- Statements can be in one or multiple lines
- A statement finishes with `;`

We need to think that SQL statements are usually the translation to code of business questions.

###  `SELECT`

 `SELECT` is used to extract the entries of desired columns from desired tables.

```sql
-- Grab two columns from a table
-- Note that query order is irrelevant
-- SQL might return entries/rows in an unordered order, actually the most efficient one
SELECT column_1, column_2 FROM table_name;
-- Grab all columns from a table
-- Do it only if necessary, since we might be pulling a lot of data!
-- It is often a quick & dirty way of visualizing all the columns/fields of a table
SELECT * FROM table_name;
-- 
-- dvdrental examples
SELECT * FROM actor;
-- 
SELECT last_name, first_name FROM actor;
--
SELECT first_name, last_name, email FROM customer;
```

###  `SELECT DISTINCT`

`SELECT DISTINCT` selects entries (rows) in a column that are unique.

```sql
SELECT DISTINCT column_name FROM table_name
-- We should use parenthesis if several columns are queried
SELECT DISTINCT(column_name) FROM table_name
-- dvdrental examples
SELECT DISTINCT release_year FROM film;
SELECT DISTINCT(rental_rate) FROM film;
SELECT DISTINCT(rating) FROM film;
```

###  `COUNT`

The function `COUNT` returns the number of rows/entries that match a condition.

```sql
-- COUNT() is a function, it needs ()
-- It counts the rows/entries in the passed column/table
SELECT COUNT(column_name) FROM table_name;
-- If no condition is passed, it does not matter which column we use, we can pass *
SELECT COUNT(*) FROM table_name;
-- It makes sense to combine it with conditions or other functions
-- Count number of distinct last names from all customers
SELECT COUNT(DISTINCT(last_name)) FROM customer;
--
-- dvdrental examples
SELECT COUNT(*) FROM payment;
--
SELECT COUNT(DISTINCT(amount)) FROM payment;
```

###  `SELECT WHERE`

`SELECT WHERE` allows to define conditions to the rows of the selected columns.
Conditions can be defined with:

- comparison operators: `=, >, <, >=, <=, <>, !=`
- logical operators: `AND, OR, NOT`

Notes:

- There is no `==` and that not-equal can be done in two ways: `<>, !=`.
- The condition columns don't need to appear in the `SELECT` clause.
- Strings are passed with single quotes `'string'` and comparisons are case-sensitive.

```sql
-- General syntax (it can be in one line)
SELECT col_1, col_2
FROM table_a
WHERE condition_alpha;
-- Example: Select entries from fields name & choice, in which name is "David"
SELECT name, choice
FROM table
WHERE name = 'David';
-- Example: multiple conditions
SELECT name, choice
FROM table
WHERE name = 'David' AND choice = 'Red';
--
-- dvdrental examples
SELECT * FROM customer
WHERE first_name = 'Jared';
--
SELECT title FROM film
WHERE rental_rate > 4 AND replacement_cost > 19.99 AND rating = 'R';
--
SELECT COUNT(*) FROM film
WHERE rental_rate > 4 AND replacement_cost > 19.99 AND rating = 'R';
--
SELECT COUNT(*) FROM film
WHERE rating = 'R' OR rating = 'PG-13';
--
SELECT email FROM customer
WHERE first_name = 'Nancy' AND last_name = 'Thomas';
--
SELECT phone FROM address
WHERE address = '23 Workhaven Lane'; 
```

###  `ORDER BY`

SQL queries return entries in different orders, depending on the most efficient way of retrieving them each time. However, we can control the order in which the entries appear. This is done at the end of the query, by indicating the column(s) with respect to which we'd like to order and whether we want ascending/descending order.

Notes:
- Ordering wrt. several columns is usual when a column has duplicate entries, e.g., order by company name and sold amount.
- We can define the order to specific columns.
- Usually, `ORDER BY` appears at the end, but `LIMIT` can go after it.

```sql
-- General syntax
SELECT col_1, col_2
FROM table_a
WHERE condition_alpha
ORDER BY col_3, col_4 DESC; -- default is ASC
--
-- dvdrental examples
-- We can define DESC/ASC to specific columns
SELECT store_id, first_name, last_name FROM customer
ORDER BY store_id DESC, first_name ASC;
```

###  `LIMIT`

`LIMIT` specifies the number of rows/queries we want. It is often used with `ORDER BY` to get the first/last X entries of a business question.

```sql
-- General syntax
SELECT col_1,
FROM table_a
WHERE condition_alpha
ORDER BY col_2 DESC
LIMIT N; -- N is an integer
--
-- dvdrental examples
-- Which are the 5 most recent and valid (!= 0 ) payments?
SELECT * FROM payment
WHERE amount != 0
ORDER BY payment_date DESC
limit 5;
-- Which are the customer IDs of the first 10 customers who created a payment?
SELECT customer_id
FROM payment
WHERE amount != 0
ORDER BY payment_date ASC
LIMIT 10;
-- Which are the titles of the 5 shortest movies?
-- Note that 'length' is a SQL token; we should avoid using such column names!
SELECT title, length
FROM film
ORDER BY length ASC
LIMIT 5;
-- How many movies last 50 minutes or below?
SELECT COUNT(title)
FROM film
WHERE length <= 50;
```

###  `BETWEEN`

`BETWEEN` can be used to define ranges of values in conditions.
It is equivalent to using `>= X AND <= Y`.

Notes:
- We can combine it with `NOT`.
- We can use it with dates if they are formatted following the ISO 8601 `YYYY-MM-DD`; however, note that the ISO format contains hours and minutes, too, and inclusivity/exclusivity issues might arise depending on when it is considered the day starts.

```sql
-- dvdrental examples
-- Payments in value between [8,9]
SELECT *
FROM payment
WHERE amount BETWEEN 8 AND 9;
-- Count the payments out from that region
SELECT COUNT(*)
FROM payment
WHERE amount NOT BETWEEN 8 AND 9;
-- Number of payments the first 2 weeks of February 2007
SELECT COUNT(*)
FROM payment
WHERE payment_date BETWEEN '2007-02-01' AND '2007-02-15';
```

###  `IN`

The `IN` operator allows to write conditions in which a field-entry value must be in a set of possible values; it is equivalent to `BETWEEN` but for categorical data.

```sql
-- General syntax
SELECT *
FROM clothes
WHERE color IN ('red', 'blue')

```
