#### SQL for tables

```sql
CREATE TABLE IF NOT EXISTS systems(id INTEGER PRIMARY KEY, "name" VARCHAR NOT NULL);
```


```sql
CREATE TABLE IF NOT EXISTS readings(
system_id INTEGER NOT NULL,
read_on TIMESTAMP NOT NULL,
power DECIMAL(10,3) NOT NULL DEFAULT 0 CHECK(power>=0),
PRIMARY KEY(system_id, read_on),
FOREIGN KEY(system_id) REFERENCES systems(id)
);
```

```sql
CREATE SEQUENCE IF NOT EXISTS price_id INCREMENT BY 1 MINVALUE 10;
```

```sql
CREATE TABLE IF NOT EXISTS prices(
id INTEGER primary key DEFAULT(nextval('price_id')),
value DECIMAL(5,2) NOT NULL,
valid_from DATE NOT NULL,
CONSTRAINT price_uk UNIQUE(valid_from));
```


### Fix output

.output stdout

### See in table

.mode table

#### Create from select

CREATE TABLE prices_duplicate AS
SELECT * FROM prices;


### Create view

```sql
CREATE OR REPLACE VIEW v_power_per_day AS
SELECT system_id,
       date_trunc('day', read_on)        AS day,
       round(sum(power)  / 4 / 1000, 2)  AS kWh,
FROM readings
GROUP BY system_id, day;
;

```

### CASTING for sqlitedb

```sql

CREATE OR REPLACE view v_power_per_day AS 
SELECT system_id,
    date_trunc('day', CAST(read_on AS TIMESTAMP)) as day,
    round(sum(power::double) /4 / 1000,2) as kWh
FROM sqlite_db.readings 
GROUP BY system_id,date_trunc('day', CAST(read_on AS TIMESTAMP))
ORDER BY system_id,day;

```



### Data manipulation language queries

- INSERT

INSERT INTO prices
VALUES (1,11.59.'2018-12-01','2019-01-01')
ON CONFLICT DO NOTHING;

### INSERT DATA FROM FILE
INSERT INTO prices(value,valid_from,valid_until)
SELECT * FROM 'prices.cvs' src;


### REMEMBER TO use the volume directory
INSERT INTO prices(value,valid_from,valid_until) SELECT * FROM '/data/prices.csv' src;



### Describe from external location
INSTALL 'httpfs'
LOAD 'httpfs'

DESCRIBE SELECT * FROM 'https://oedi-data-lake.s3.amazonaws.com/pvdaq/csv/systems.csv';


### Insert data from external sources


INSERT INTO systems(id,name) SELECT DISTINCT system_id, system_public_name FROM 'https://oedi-data-lake.s3.amazonaws.com/pvdaq/csv/systems.csv' ORDER by system_id ASC;


### INSERT DATA FROM CSV

```sql
INSERT INTO readings(system_id, read_on, power) 
SELECT SiteId, "Date-Time",
 CASE WHEN ac_power <0 OR ac_power IS NULL THEN 0 
 ELSE ac_power END
 FROM read_csv_auto('https://developer.nrel.gov/api/pvdaq/v3/data_file?api_key=DEMO_KEY&system_id=34&year=2019');
```

#### Reading using date_trunc

SELECT * FROM readings WHERE date_trunc('day',read_on) = '2019-08-26' and power <>0;

#### Using the conclict do

```sql

INSERT INTO readings(system_id, read_on, power)
VALUES (10,'2023-06-05 13:00:00',3000)
ON CONFLICT(system_id, read_on) DO UPDATE
SET power = CASE
 WHEN power = 0 then excluded.power
 ELSE (power + excluded.power) / 2 END;
```


#### USING delete

```sql
DELETE FROM readings
WHERE date_part('minute',read_on) NOT IN (0,15,30,45);

```


#### SELECT statements

```sql
SELECT date_part('year', valid_from) as year,
min(value) as mininum_price,
max(value) as maximum_price
FROM prices
WHERE year BETWEEN 2019 and 2020
group by year
order by year;

```

 
#### COPY values and create TABLE
```sql
CREATE TABLE my_table AS Select * from read_csv_auto('/path/file.csv')
```

#### JOIN

```sql
SELECT * FROM
  ( VALUES(1,'a1'),
          (2,'a2'),
 	  (3,'a3')) l (id, nameA)
JOIN
  (VALUES (1,'b1'),
	  (2,'b2'),
	  (4,'b4'))r (id,nameB)
USING (id);

```

All the rows from left are included.

```sql
SELECT * 
FROM (VALUES (1,'a1'),
	     (2,'a2'),
	     (3,'a3')) l(id,nameA)
LEFT OUTER JOIN
    (VALUES (1,'b1'),
	    (2,'b2'),
	    (4,'b4'))r (id,nameB)
USING (id)
ORDER BY id;
```
#### Using join

```sql
select name, count(*) as number_of_readings from readings JOIN systems ON id = system_id 
group by name;
```

#### CTE

```sql
 select max_power.v, read_on from (select max(power) as v from readings) max_power JOIN readings ON power=max_power.v;
```

Rewriting in CTE

```sql
WITH max_power AS (
    select max(power) as v from readings )
SELECT max_power.v, read_on 
FROM max_power
JOIN readings ON power= max_power.v;
``` 

#### Using more complex example

```sql
WITH per_hour AS(
   SELECT system_id, date_trunc('hour',read_on) as read_on,
	avg(power)/1000 as kWh
   FROM readings
   GROUP BY ALL
)
Select name, max(kWh), arg_max(read_on,kWh) as 'Read on' 
FROM per_hour
    JOIN systems s on s.id= per_hour.system_id
WHERE system_id=34
group by s.name;

```

#### Using recursive

```sql
CREATE TABLE IF NOT EXISTS src(
id INT PRIMARY KEY,
parent_id INT,
name varchar(8)
);
```

```sql
INSERT INTO src
   (VALUES
        (1,null,'root1'),
        (2,1,'ch1a'),
	(3,1,'ch2a'),
        (4,3,'ch3a'),
        (5,null,'root2'),
        (6,5,'ch1b'));
```

```sql
WITH RECURSIVE tree AS (
   select id, id as root_id, [name] as path
 FROM src where parent_id IS NULL
UNION ALL
   select src.id,root_id,list_append(tree.path,src.name) as path
from src
JOIN tree ON (src.parent_id=tree.id)
)
select path from tree;
```

#### EXCLUDE
SELECT * EXCLUDE(ID)
from prices

SELECT COLUMNS('valid.*') from prices LIMIT 3;

FROM prices
WHERE COLUMNS(col -> col LIKE 'valid%')
BETWEEN '2020-01-01' and '2021-01-01'


#### Using alias

```sql
SELECT system_id < 10 as is_not_system10,
date_trunc('month',read_on) as month,
sum(power)/1000 / 1000 as power_per_month
FROM readings
where is_not_system10 = true
group by is_not_system10,month
having power_per_month > 100;
```

### Sample

```sql
select power from readings where power <>0 using sample 10%(bernoulli); 

```
