# HIVE - TP 2 : TABLES EXTERNES ET PARTITIONNEMENT
<br/>

## Hive: External Tables

* Quick Pre-read : 
  
  https://www.quora.com/Difference-between-Hive-internal-tables-and-external-tables

* Creating external table Example - Create table on weather data :
  
```sql
  CREATE EXTERNAL TABLE weather (temperature INT, date STRING)
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  LOCATION '/hive/data/weather';
```
<br/>

## Chargement depuis HDFS

**Load the data in table**

Load the data from HDFS to Hive using the following command:

```console
LOAD DATA INPATH ‘hdfs:/data/2012.txt’ INTO TABLE weather;
```
<br/>

## Partitionnement

* Hive stores tables in partitions. Partitions are used to divide the table into related parts. Partitions make data querying more efficient. 

* For example in the above weather table the data can be partitioned on the basis of year and month and when query is fired on weather table this partition can be used as one of the column.

```sql
  CREATE EXTERNAL TABLE IF NOT EXISTS weather (temperature INT, date STRING)
    PARTITIONED BY (year INT, month STRING)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ','
    LOCATION '/hive/data/weather';
```

* Loading data in partitioned tables is different than non-partitioned one. 

* There is little manual work of mentioning the partition data. 

* Data can be loaded in partition, year 2012 and month 01 and 02 as follows:
```console
LOAD DATA INPATH ‘hdfs:/data/2012.txt’ INTO TABLE weather PARTITION (year=2012, month=’01’);
LOAD DATA INPATH ‘hdfs:/data/2012.txt’ INTO TABLE weather PARTITION (year=2012, month=’02’);
```

* This creates the partitioned table and makes different folder for each partition which helps in querying data.

### Querying of partitioned table :

Partitioned tables can use partition parameters as one of the column for querying. To retrieve all the data for month of ‘02’ following query can be used on weather table.

```sql
SELECT * FROM weather WHERE month = '02';
```

# Assignment

#### Utiliser le dataset suivant :
https://www1.ncdc.noaa.gov/pub/data/ghcn/daily/ (lire le readme attentivement)  
https://www1.ncdc.noaa.gov/pub/data/ghcn/daily/by_year/

1. Etudier la structure d’un fichier.

2. Charger les données des années 2019-2020 dans HIVE (tables externes).

3. Effectuer un partitionnement adapté.

4. Créer des vues et requêter les données.

5. Visualiser des histogrammes pertinents dans HUE/Zeppelin (Démo formateur)

# HINTS 

- Ajouter une colonne : 
```
hive> ALTER TABLE employee ADD COLUMNS (dept STRING);
```

- Faire un Substring (Pour isoler le code pays dans l'ID de station)

```
syntaxe : 
substr(<input string/column>, int start, int length)

exemple
select substr(id, 1, 2) as country
```

- Splitter une colonne : 

```
create table test (
key string, 
value string )
STORED AS ORC ;

INSERT INTO test (key, value )
VALUES (22, '1001 abc, 1002 pqr, 1003 tuv'),
(33, '1004 def, 1005 xyz');



select key, split(items, ',') as valArray
	from test

result 

+------+---------------------------------------+--+
| key  |                  _c1                  |
+------+---------------------------------------+--+
| 22   | ["1001 abc"," 1002 pqr"," 1003 tuv"]  |
| 33   | ["1004 def"," 1005 xyz"]              |
+------+---------------------------------------+--+

select key, trim(uniqueVal)
from(
 select key, split(items, ',') as valArray
 from test ) a lateral view explode(a.valArray) exploded as uniqueVal ;

+------+-----------+--+
| key  |    _c1    |
+------+-----------+--+
| 22   | 1001 abc  |
| 22   | 1002 pqr  |
| 22   | 1003 tuv  |
| 33   | 1004 def  |
| 33   | 1005 xyz  |
+------+-----------+--+

select key, split(trim(uniqueVal), ' ')[0], split(trim(uniqueVal), ' ')[1]
from(
	select key, split(items, ',') as valArray
	from test 
	) a lateral view explode(a.valArray) exploded as uniqueVal ;

+------+-------+------+--+
| key  |  _c1  | _c2  |
+------+-------+------+--+
| 22   | 1001  | abc  |
| 22   | 1002  | pqr  |
| 22   | 1003  | tuv  |
| 33   | 1004  | def  |
| 33   | 1005  | xyz  |
+------+-------+------+--+

```
ENSUITE, 
INSERER LE RESULTAT DE CETTE DERNIERE OPERATION DANS LA OU LES COLONNES CREES EN VUE DU SPLIT
