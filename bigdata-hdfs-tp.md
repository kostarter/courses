# HADOOP - TP 1
# LE SYSTÈME DE GESTION DE FICHIER HDFS


**BUT DU TP**
Prendre en main et apprendre à manipuler des fichiers dans l’environnement HDFS. :heavy_check_mark: 

### ⚠⚠⚠ NE COPIEZ PAS LES COMMANDES SANS REFLECHIR ⚠⚠⚠
C'est la meilleure façon de faire des erreurs de copier-coller :)
<br/>

---
- #### Enoncé 1 : MANIPULATION Part I

:information_source: CONSULTER PREALABLEMENT LA REFERENCE DES COMMANDES USUELLES EN PAGE 5

1 - Télécharger le fichier suivant dans le Système linux via la commande WGET 
```console
wget https://raw.githubusercontent.com/jpatokal/openflights/master/data/airports.dat
```

2 - Créer un répertoire sous HDFS.

:warning: Spoiler : 
```console
$ hdfs dfs -mkdir /user/mon_prenom
```

3 - Insérer le fichier dans ce répertoire HDFS via la commande PUT dans votre répertoire user.

:warning: Spoiler : 
```console
$ hdfs dfs -put airports.dat /user/mon_prenom
```

4 - Vérifier la présence de ce fichier dans votre répertoire hdfs user via la commande LS (hdfs).

:warning: Spoiler : 
```console
$ hdfs dfs -ls /user/mon_prenom
```

---
- #### Enoncé 2 : MANIPULATION Part II

1 - Créer un répertoire hdfs “data” dans votre répertoire “user/mon_prenom”.

:warning: Spoiler : 
```console
$ hdfs dfs -mkdir /user/mon_prenom/data
```

2 - Déplacer le fichier airports.dat depuis hdfs dans ce répertoire data.

:warning: Spoiler : 
```console
$ hdfs dfs -mv /user/mon_prenom/airports.dat /user/mon_prenom/data
```

3 - Modifier les droits hdfs de ce fichier pour le rendre accessible uniquement à vous.

:warning: Spoiler :
```console
$ hdfs dfs -chmod 700 /user/mon_prenom/data/airports.dat
```

:information_source: Consulter la doc pour les commandes :  mkdir / mv / chmod


---
- #### Enconcé 3 : EXPLORATION DES METADATA

Explorer le mapping réel des fichiers HDFS dans EXT4.
Localiser les données relatives au fichier chargé dans HDFS lors de la première partie.

* Blocs du fichier
* Mapping du fichier et des blocs
* Mapping des blocs vers les serveurs

Infos des blocs de fichiers :
  
```console  
$ hdfs fsck /user/mon_prenom/data/airports.dat -files -blocks -locations  
```  
--- 

:information_source: Ressource Utile :   
<!--https://hortonworks.com/blog/hdfs-metadata-directories-explained/--!>
https://www.waitingforcode.com/hdfs/hdfs-on-disk-explained/read
