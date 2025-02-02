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
