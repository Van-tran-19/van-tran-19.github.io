---
title: "Introduction to SQLite3: Querying a Sales and Weather Database"
date: 2026-11-03 18:00:00 +0200
categories: [Databases, SQLite]
tags: [SQLite3, SQL, Queries, Joins, Aggregation, Database]
toc: true
---
# SQLite3

## What is SQLite3?

SQLite3 is version 3 of SQLite, a lightweight, self-contained, and serverless relational database management system (RDBMS). Unlike traditional database engines like PostgreSQL, MySQL, or Oracle, SQLite does not run as a separate background server or daemon process.

**What is a daemon process?** 

A daemon process (or background server) is a computer program that runs continuously in the background rather than being directly controlled by an interactive user at a keyboard or terminal. 

- It does not attach to a terminal window but it appears in system monitor tools.
- It starts automatically when the operating system boots and keeps running even if the user logs out.
- It typically sits in an idle loop waiting for external events or network requests on a designated port

**SQLite vs. Client-Server Databases**

| **Feature** | **SQLite3** | **Client-Server (e.g., PostgreSQL, MySQL)** |
| --- | --- | --- |
| **Architecture** | Embedded library running in-process | Independent daemon/server process |
| **Communication** | Direct disk/in-memory I/O | Network sockets (TCP/IP, Unix domain) |
| **Data Storage** | Single self-contained disk file | Directory trees managed by the engine |
| **Concurrency** | Multiple readers, but serialized single-writer | Multi-reader, multi-writer (MVCC) |
| **User Management** | File-system-level permissions only | Granular roles, grants, and passwords |

## Practise

During this lab, I work with sales in weather data. The diagram of tables is: 

<img width="193" height="384" alt="image" src="https://github.com/user-attachments/assets/7c87c027-22db-43ba-85c5-0cd0a4fe32c3" />

### First question

The goal is to find the minimum dewpoint value from the weather data. I will find where is located the dewpoint column: in the weather table and apply to it the MIN aggregation function. 

```sql
SELECT MIN(dewpoint) FROM weather;
#output 
-24
```

The minimum dewpoint value is -24.

### Second question

I have to find the number of days where the sun rises before 5AM. I will query the weather table to inspect the format of the sunrise column. 

```sql
SELECT sunrise FROM weather LIMIT 5
#output
NULL
NULL
NULL
NULL
NULL
```

Based on the response, I must set the sunrise to be NO NULL. 

```sql
SELECT sunrise FROM weather
WHERE sunrise IS NOT NULL 
LIMIT 5
#output
07:16:00
07:16:00
07:16:00
07:16:00
07:16:00
```

I know that time is stored as this xx:xx:xx format. I can set now the condition to be “`sunrise < 05:00:00`”. 

```sql
SELECT COUNT(date) FROM weather WHERE sunrise < '0500'
#output
1,411
```

I notice that they have multiple records of the same date, I will use the DISTINCT function to avoid this. 

```sql
SELECT COUNT(DISTINCT date) FROM weather WHERE sunrise < '0500'
#output
378
```

The total number of records where the sun rose before 5 AM is 378. 

### Third question

The goal is to find the ID of the weather station that has the highest number of stores. We will use GROUP BY, ORDER BY and LIMIT 1. 

```sql
SELECT station_nbr FROM relation GROUP BY station_nbr ORDER BY COUNT(store_nbr) DESC LIMIT 1
#output
17
```

The weather station ID is 17. 

### 4th question

The goal is to find the ID of the item satisfying: date = '2012-01-01', store_nbr = 1, Max(units). I notice that sales_in_weather table contains all the necessary information to process the request: item_nbr column, store_nbr column and date column.

```sql
SELECT item_nbr FROM sales_in_weather 
WHERE store_nbr = 1 AND date = '2012-01-01' 
ORDER BY units DESC LIMIT 1
#output
9
```

The item ID is 9.

### 5th question

The goal is to show the sea level status recorded by the weather station satisfying: store_nbr = 19; date = '2013-02-24'. Sea level status refers to sealevel in the weather table. Information are splitted into two different tables. I will join this two tables using JOIN function. 

```sql
SELECT w.sealevel FROM weather w JOIN relation r ON r.station_nbr = w.station_nbr  
WHERE r.store_nbr = 19 AND w.date = '2013-02-24'
```

The sea level is 29.84.

### 6th question

The goal is to find the number of stores in the weather station with the highest recorded heat of all time. I will try to find the weather station with the highest recorded heat of all time and I will use the IN function to link all of the queries.

```sql
SELECT COUNT(r.store_nbr) FROM relation r 
WHERE r.station_nbr IN 
	(SELECT station_nbr FROM weather WHERE heat = (
	SELECT MAX(heat) FROM weather)) LIMIT 3;
#output
2
```

The number of stores are 2.

### 7th question

The goal is to find the percentage of the units of item_nbr = 5 sold among all units of items sold in store_nbr = 3 on the day with the highest max temperature in 2012. 

#### What is the `CASE WHEN` function?

The `CASE WHEN` expression is SQL’s way of handling conditional logic (similar to **if-then-else** statements in programming languages). It allows you to evaluate conditions and return specific values depending on whether those conditions are true.

There are two main ways to write a `CASE` statement: the **Searched CASE** syntax and the **Simple CASE** syntax.

**The Searched `CASE` Syntax (Most Common)**

This form allows you to evaluate multiple distinct boolean conditions. It checks each `WHEN` condition sequentially from top to bottom and stops at the first one that evaluates to true.

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE default_result
END
```

**The Simple `CASE` Syntax**

This form compares a single expression or column against a list of specific literal values.

```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    ELSE default_result
END
```

I will make a query to find the number of units satisfying store_nbr = 3 AND item_nbr = 5 AND date = t.target_date, a query satisfying store_nbr = 3 AND date = t.target_date where target_date is the date in the 2012 year. Then, in the FROM query, I will set up the last condition which is the highest temperature in the weather station using ORDER BY LIMIT 1. 

```sql
SELECT 
    (
        SELECT SUM(units) 
        FROM sales_in_weather 
        WHERE store_nbr = 3 AND item_nbr = 5 AND date = t.target_date 
    ) * 100.0 / 
    (
        SELECT SUM(units) 
        FROM sales_in_weather 
        WHERE store_nbr = 3 AND date = t.target_date
    ) AS percentage
FROM (
    SELECT w.date AS target_date
    FROM weather w 
    JOIN relation r ON w.station_nbr = r.station_nbr 
    WHERE r.store_nbr = 3 
      AND w.date >= '2012-01-01' 
      AND w.date <= '2012-12-31'
    ORDER BY w.tmax DESC 
    LIMIT 1
) t;
#Output
61.45454545454545
```

The percentage is 61.45.
