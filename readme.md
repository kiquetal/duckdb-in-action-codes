#### Learning some commands for duckdb

### Run duckdb for docker

docker run -it --rm -v $(pwd):/data qldrsc/duckdb duckdb 

### Copy from container docker

docker cp CONTAINER:file ./myfile

### Run the example

duckdb -csv \ 
-s "SELECT country, population,birthrate,deathrate from read_csv_auto('https://bit.ly/3KoiZR0') WHERE trim(region)='WESTERN EUROPE'" \
western_europe.csv

### Using inside the container

.mode csv

.output western_europe.csv

### COPY DIRECT FROM mouting directory

duckdb > COPY (SELECT country, population,birthrate,deathrate from read_csv_auto('https://bit.ly/3KoiZR0') WHERE trim(region)='WESTERN EUROPE') TO '/data/western_europe.parquet' (FORMAT PARQUET)

