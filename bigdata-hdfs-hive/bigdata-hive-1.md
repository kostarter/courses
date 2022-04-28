# HIVE - TP 1 : INGESTION ANALYSE ET TRAITEMENT DE DONNEES

<br/>

## ETAPE 1 : Ligne de commande

* Chercher un fichier CSV Open Data de taille moyenne : [here](https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/how-to-process-data-with-apache-hive/assets/driver_data.zip) :link:

Utiliser la commande wget DANS VOTRE REPERTOIRE LOCAL (/home/hadoop/).

    * Unzip le fichier : unzip votrefichier.zip
    * rm -rf __MACOSX
    * cd driver_data
    * Ouvrir le fichier : head votrefichier.csv
    * Noter le schéma

    * ATTENTION : Renommer la colonne wage-plan en wave_plan (utiliser vi ou nedit)
    
* Lancer le shell Hive : hive

#### :information_source: Hints :

* Lancer le Shell Hive :
```console
$ hive
```

OPTIONNEL SI PERMISSION DENIED :
```console
$ sudo su hdfs
$ cd /usr/bin
$ hive
```

* Afficher les bases :
```console
show databases;
```

* Créer une database mon_prenom_db

```
hive> CREATE DATABASE mon_prenom_db;
```

```console
show databases;
```

* Rentrer dans cette database (Utiliser la commande use)
```console
use mon_prenom_db;
```

Optionnel : (non nécessaire)
* Changer les droits de votre database directement sur hdfs : utiliser la commande ‘chown’ pour réattribuer les droits à votre user

<br/>

## ETAPE 2 : Ligne de commande

* Définir la table associée au fichier csv dans Hive (Create Table) en respectant le schéma noté précédemment :

EXEMPLE :
```sql
CREATE TABLE mon_user_table_xxx (
   customerID INT,
   firstName STRING,
   xxxxxx STRING,
   xxxxxxx TIMESTAMP
  ) ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ','
;

# SUR LA MEME LIGNE (cf ci-dessous)
CREATE TABLE mon_user_table_xxx  (customerID INT,firstName STRING,xxxxxxx STRING, xxxxx TIMESTAMP) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
```

:warning: SKIP HEADER : tblproperties("skip.header.line.count"="1");

<br/>

## ETAPE 2/1 : Ligne de commande

* DANS LA CLI HIVE :

Charger le fichier csv directement dans Hive depuis la ligne de commande (commande load) dans votre table créée précédemment.
```console
LOAD DATA LOCAL INPATH '/chemin/monfichier-a-inserer.csv' OVERWRITE INTO TABLE mon_user_ma_table_xxx;
```

:warning: **ATTENTION !!! FAITES D'ABORD UNE SAUVEGARDE DE VOTRE FICHIER csv AVANT DE LANCER CETTE COMMANDE  
CAR CETTE COMMANDE VA SUPPRIMER LE FICHIER CSV APRES IMPORT !!!!!**

DANS LA CLI LINUX : 
```console
cp /chemin/monfichier-original.csv /chemin/monfichier-a-inserer.csv
```

* Une erreur de localité est levée. Explication.

<br/>

## ETAPE 2/2 : Ligne de commande

* Solution : Charger le fichier dans HDFS (CLi Linux) dans un emplacement de votre choix (sous votre emplacement user).

:warning: Ne pas oublier de faire une copie !!

* Rejouer la commande précédente, cette fois sans le “local”.
```console
LOAD DATA INPATH ’/chemin/sur/hdfs/monfichier-a-inserer.csv' OVERWRITE INTO TABLE mon_user_ma_table_xxx;
```

* Une erreur de format est levée. Explication.
Error: The file that you are trying to load does not match the file format of the destination table.

##### Formatage de la requête 

```sql
CREATE TABLE mon_user_table_xxx (
   customerID INT,
   firstName STRING,
   xxxxxx STRING,
   xxxxxxx TIMESTAMP
) 
   ROW FORMAT DELIMITED
   FIELDS TERMINATED BY ','
STORED AS TEXTFILE; //OBLIGATOIRE SI FICHIER ENTREE = CSV
```

Eventuellement rajouter cette ligne également à la fin de votre requête:
tblproperties("skip.header.line.count"="1");

* Afficher les données insérées :        
```sql
SELECT * FROM mon_user_ma_table_xxx;
```

<br/>

<!--
## HINT

CREATE TABLE driver (driverId STRING,name STRING,ssn STRING,location STRING,certified STRING,wageplan STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE tblproperties("skip.header.line.count"="1");

LOAD DATA INPATH '/user/hadoop/drivers.csv' OVERWRITE INTO TABLE driver;


## Etapes détaillées

:no_entry_sign: **DEPRECATED (NE PAS FAIRE)**

* Dans Ambari Hive View.

* Importer votre fichier dans une nouvelle table dans la base précédemment créée.

* Hive View 2.0 > Tables > Icone + > Import.
* Sélectionner le délimiteur adéquat
* Sélectionner le fichier depuis HDFS.
* Le charger dans Hive.



!-->
