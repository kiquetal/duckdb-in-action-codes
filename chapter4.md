#### Advanced aggreation and analysis of data

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


