#### Advanced aggreation and analysis of data

### Attach and create sqlitedb

ATTACH 'new_sqlite_database.db' AS sqlite_db (TYPE SQLITE);

### Prerequisites

INSTALL sqlite;
LOAD sqlite;

#### Window functions

```sql

WITH ranked_readings AS (
SELECT *, dense_rank() OVER (PARTITION by system_id order by power desc) as rnk
FROM readings )

SELECT * FROM ranked_readings where rnk <=2 ORDER by system_id, rnk ASC;
```

Another sql

```sql

SELECT *, avg(kWh) OVER (PARTITION by system-id) as average_per_system
FROM v_power_per_day;
 ```


#### Framing

Specifies a set of rows relative to each row where the function is evaluated. The distancre from the current row
is given as an expression either preceding or following the current row. This distance can either be specified 
as an integral number of rows or as a range delta expression from the value of the ordering expression

```sql

SELECT system_id, day, kWh, avg(kWh) OVER (
PARTITION by system_id
ORDER by day ASC
RANGE BETWEEN interval 3 days preceding
AND INTERVAL 3 days following
) as "kwh 7-day-moving-average"
FROM v_power_per_day
ORDER BY system_id, day;
```

#### Named windows

