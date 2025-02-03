#### SQL for tables

```sql
CREATE TABLE IF NOT EXISTS systems(id INTEGER PRIMARY KEY, "name" VARCHAR NOT NULL);
```


```sql
CREATE TABLE IF NOT EXISTS readings(
system_id INTEGER NOT NULL,
read_on TIMESTAMP NOT NULL,
power DECIMAL(10,3) NOT NULL DEFAULT 0 CHECK(power>=0)
PRIMARY KEY(system_id, read_on),
FOREIGN KEY(system_id) REFERENCES system(id);
```

```sql
CREATE SEQUENCE IF NOT EXISTS price_id INCREMENT BY 1 MINVALUE 10;
```

```sql
CREATE TABLE IF NOT EXISTS prices(
id INTEGER primary key DEFAULT(nextval('price_id')),
value DECIMAL(5,2) NOT NULL,
valid_from DATE NOT NULL,
CONSTRAINT price_uk UNIQUE(valid_from)
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

### Data manipulation language queries

- INSERT

INSERT INTO prices
VALUES (1,11.59.'2018-12-01','2019-01-01')
ON CONFLICT DO NOTHING;

### INSERT DATA FROM FILE
INSERT INTO prices(value,valid_from,valid_until)
SELECT * FROM 'prices.cvs' src;



### Describe from external location
INSTALL 'httpfs'
LOAD 'httpfs'

DESCRIBE SELECT * FROM 'https://oedi-data-lake.s3.amazonaws.com/pvdaq/csv/systems.csv';


