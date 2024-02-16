# DuckDB Demo

This is a quick intro to DuckDB and some of it's basic features. Most of this can be found in the [DuckDB Documentation](https://duckdb.org/docs/sql/introduction#creating-a-new-table).

## DuckDB Installation

Follow the steps in the [DuckDB Installation](https://duckdb.org/docs/installation/index?version=latest&environment=cli&installer=binary&platform=win) section to get started. There are multiple ways to install DuckDB depending on the clients/APIs you want to use. This demo uses the Windows Command Line client and the Python API later on.

## The In-Memory Database

By default, DuckDB uses an in-memory database, so none of the data is persisted after closing the connection. To connect to an in-memory database use the `duckdb` command.

Once you've connected to the in-memory database, you'll see the `D` prompt on your command line. Non-SQL commands start with a `.` and provide configs and information about the database. Type `.help` to see a list of commands you can use.

```
> duckdb
D .help
.bail on|off             Stop after hitting an error.  Default OFF 
.binary on|off           Turn binary output on or off.  Default OFF
.cd DIRECTORY            Change the working directory to DIRECTORY 
.changes on|off          Show number of rows changed by SQL
.check GLOB              Fail if output since .testcase does not match
.columns                 Column-wise rendering of query results
.constant ?COLOR?        Sets the syntax highlighting color used for constant...
.constantcode ?CODE?     Sets the syntax highlighting terminal code used for...
.databases               List names and files of attached databases
.dump ?TABLE?            Render database content as SQL
.echo on|off             Turn command echo on or off
.excel                   Display the output of next command in spreadsheet
.exit ?CODE?             Exit this program with return-code CODE
...
```

The `.exit` command will close the connection.

## Using a Persisted Database

To connect to a persisted database use `duckdb [DATABASE_NAME]`. The database is saved as a local file and has the extension `.duckdb` or `.db`. This repo contains a persisted database named `demo.duckdb`. Connect to it by typing `duckdb .\demo.duckdb`. Use the `.tables` command to see the database tables.

```
> duckdb .\demo.duckdb
D .tables
cities         raw_customers  weather
```

There are some existing tables there for you to check out.

## SQL Introduction

DuckDB supports all the standard SQL commands. The SQL syntax is based on PostgreSQL. See below for examples (these can be found in the [DuckDB Documentation](https://duckdb.org/docs/sql/introduction)).

### Creating or Dropping a Table

```sql
-- Create a table
CREATE TABLE weather (
    city    VARCHAR,
    temp_lo INTEGER, -- minimum temperature on a day
    temp_hi INTEGER, -- maximum temperature on a day
    prcp    REAL,
    date    DATE
);
```

```sql
CREATE TABLE cities (
    name VARCHAR,
    lat  DECIMAL,
    lon  DECIMAL
);
```

```sql
-- Drop a table
DROP TABLE [tablename];
```

### Populating a Table

```sql
-- Insert values
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```

```sql
INSERT INTO cities
VALUES ('San Francisco', -194.0, 53.0);
```

```sql
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
```

```sql
INSERT INTO weather (date, city, temp_hi, temp_lo)
VALUES ('1994-11-29', 'Hayward', 54, 37);
```

You can also read data from files. **Note**: You have to create the table first.

```sql
COPY weather
FROM 'weather.csv';
```

### Querying a Table

```sql
SELECT *
FROM weather;
```

```sql
SELECT city, temp_lo, temp_hi, prcp, date
FROM weather;
```

```sql
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date
FROM weather;
```

```sql
SELECT *
FROM weather
WHERE city = 'San Francisco' AND prcp > 0.0;
```

```sql
SELECT *
FROM weather
ORDER BY city;
```

```sql
SELECT *
FROM weather
ORDER BY city, temp_lo;
```

```sql
SELECT DISTINCT city
FROM weather;
```

```sql
SELECT DISTINCT city
FROM weather
ORDER BY city;
```

### Joins Between Tables

```sql
-- It's better to explicitly specify the join here
SELECT *
FROM weather, cities
WHERE city = name;
```

```sql
SELECT city, temp_lo, temp_hi, prcp, date, lon, lat
FROM weather, cities
WHERE city = name;
```

```sql
SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.lon, cities.lat
FROM weather, cities
WHERE cities.name = weather.city;
```

```sql
-- This is easier to see how the tables are joined
SELECT *
FROM weather
INNER JOIN cities ON weather.city = cities.name;
```

```sql
SELECT *
FROM weather
LEFT OUTER JOIN cities ON weather.city = cities.name;
```

### Aggregate Functions

```sql
SELECT max(temp_lo)
FROM weather;
```

**Note: The query below is incorrect.** Don't use aggregate functions in the `WHERE` clause.

```sql
SELECT city
FROM weather
WHERE temp_lo = max(temp_lo);     -- WRONG
```

Use a nested query to filter on aggregates.

```sql
SELECT city
FROM weather
WHERE temp_lo = (SELECT max(temp_lo) FROM weather); -- CORRECT
```

```sql
SELECT city, max(temp_lo)
FROM weather
GROUP BY city;
```

```sql
SELECT city, max(temp_lo)
FROM weather
GROUP BY city
HAVING max(temp_lo) < 40;
```

```sql
SELECT city, max(temp_lo)
FROM weather
WHERE city LIKE 'S%'
GROUP BY city
HAVING max(temp_lo) < 40;
```

### Updates

```sql
UPDATE weather
SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
WHERE date > '1994-11-28';
```

### Deletions

```sql
DELETE FROM weather
WHERE city = 'Hayward';
```

**Note: Delete statements without a `WHERE` clause will delete all rows.**

```sql
DELETE FROM tablename; -- Deletes all rows in a table
```

## Next Steps

From here you can jump over to the [demo.ipynb](./demo.ipynb) notebook to see how the [DuckDB Python API](https://duckdb.org/docs/api/python/overview) works. If you want to learn more about DuckDB, check out the [DuckDB Documentation](https://duckdb.org/docs/). It has tons of integrations and available APIs.