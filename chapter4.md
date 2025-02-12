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

SELECT *, avg(kWh) OVER (PARTITION by system_id) as average_per_system
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

When you need three aggregates(min,max and quantiles) for caching outliers and computing the quantiles

```sql
SELECT system_id, day, min(kWh) OVER seven_days as '7-day-min',
quantile(kWh, [0.25,0.5,0.75])
 OVER seven_days AS "kwh 7-day-quartile",
 max(kWh) OVER seven_days as "7-day-max"
FROM v_power_per_day
WINDOW 
  seven_days AS (
	partition by system_id, month(day)
	ORDER by day ASC
	RANGE BETWEEN interval 3 days preceding
	and interval 3 days following
  )
ORDER BY system_id, day;


```

```sql

SELECT valid_from, value, lag(value) OVER validity as  "Previous value",
      value::double - lag(value::double,1,value::double) OVER validity as CHANGE
FROM sqlite_db.prices
WHERE date_part('year',CAST(valid_from as TIMESTAMP)) = 2019
WINDOW validity AS (ORDER BY valid_from)
ORDER BY valid_from;

```


#### More casting

```sql

WITH changes AS (
SELECT value::double - lag(value::double,1,value::double) OVER (ORDER BY valid_from) AS v
	FROM sqlite_db.prices
	WHERE date_part('year',CAST(valid_from AS TIMESTAMP)) = 2019
	ORDER BY valid_from
)
SELECT sum(changes.v) AS total_change
FROM changes;
```
  




