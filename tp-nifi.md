# Apache Nifi Workshop

Construire une pipeline Big Data avec Apache Nifi, l'agent de messages Kafka et MongoDB à partir des données Alpha Vantage.

## <img src="https://almond-static.stanford.edu/icons/co.alphavantage.png" width=45px>  C'est quoi Alpha Vantage ?

Alpha Vantage offre des APIs gratuites aux formats JSON et CSV pour des données financières en temps réels ou en archive sur le cours des actions, les taux de change, crypto monnaie, bitcpoin, etc...

Nous allons utiliser Alpha Vantage pour obtenir les données en temps réel sur cours de certaines actions.

Aller https://www.alphavantage.co/ et créer un compte pour obtenir le token qui permet d'appeler l'API Rest. Pour un compte gratuit seuls 5 appels sont acceptés par minute.
<br/>
<br/>

* Exemple d'appel au service Rest que nous allons utiliser :

> https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol=IBM&interval=5min&apikey=TOKEN&datatype=csv

*  Pour chaque action nous recevons les plus hautes et plus basses valeurs de marché, mais aussi les valeurs à l'ouverture et la fermeture des marchés (OHLC) :</br>

> https://www.investopedia.com/terms/o/ohlcchart.asp

##### Exemples d'actions :

- IBM
- TSCO (Tractor Supply Company)
- AAPL (Apple)
- GGOGL (Google)
- GM (General Motor)

## Vue générale de la Pipeline 

![Nifi workshop](https://i.ibb.co/qgqBXN0/Pipeline.png)

## Installation des outils

#### Apache NiFi

Télécharger Apache NiFi :
```sbtshell
> wget https://archive.apache.org/dist/nifi/1.11.4/nifi-1.11.4-bin.tar.gz
> tar -xvf nifi-1.11.4-bin.tar.gz
```

Lancer Nifi avec la commande :
```sbtshell
> cd nifi-1.11.4/
> ./bin/nifi.sh start
```

Aller sur : 
http://localhost:8080/nifi/

![Nifi workshop](https://i.ibb.co/xSXNG5v/Screenshot-2021-02-18-Ni-Fi-Flow.png)

#### Apache Kafka et Zookeeper

Télécharger Apache Kafka et Zookeeper :
```sbtshell
> wget https://www.apache.org/dyn/closer.cgi?path=/kafka/2.7.0/kafka_2.12-2.7.0.tgz
> tar -xvf kafka_2.12-2.7.0.tgz

> wget https://apache.mediamirrors.org/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz
> tar -xvf apache-zookeeper-3.6.2-bin.tar.gz
```

1. Lancer ZooKeeper :
```sbtshell
$ ./bin/zkServer.sh start
```

2. Lancer Kafka avec la commande :
```sbtshell
> cd kafka_2.12-2.7.0
> ./bin/kafka-server-start.sh ../config/server.properties
```

## Réalisation

Nous allons découper le traitement en trois lots :
1. Récupération des données Alpha Vantage.
2. Découpage des données et envoi vers un broker Kafka.
3. Récupération des données et stockage dans une base de données MongoDB.

![Nifi workshop](https://i.ibb.co/6H11VgZ/Screenshot-2021-02-18-Ni-Fi-Flow-00.png)

### Etape 1 : Récupération des données Alpha Vantage.
![Nifi workshop](https://i.ibb.co/k9J8kdz/Screenshot-2021-02-18-Ni-Fi-Flow-01.png)

1. Appeler le service Alpha Vantage.</br>

Prévoir un écart de 30 secondes entre deux appels. Faut pas oublier que le compte est gratuit et ne permet pas plus de 5 appels par minute.

![Nifi workshop](https://i.ibb.co/4gkR7Qx/Screenshot-2021-02-18-Ni-Fi-Flow-02.png)

Importer le certificat nécessaire pour appeler le service Rest en HTTPS :

![Nifi workshop](https://i.ibb.co/M7LbZF2/Screenshot-from-2021-02-18-14-52-59.png)

Charger le certificat dans un Keystore. Le chemin vers keystore doit être spécifié dans les properties du Processor GetHttp : 
```sbtshell
keytool -import -v -trustcacerts \
    -file alphavantage.co.cer -alias alphaca \
    -keystore cacerts.jks
```
  
2. Mettre à jour le nom du fichier en sortie avec un nom unique basé sur le current time :

![Nifi workshop](https://i.ibb.co/vwJkJq3/Screenshot-2021-02-18-Ni-Fi-Flow-03.png)

3. Stocker les fichiers csv dans le dossier de sortie :

![Nifi workshop](https://i.ibb.co/tcjxJRz/Screenshot-2021-02-18-Ni-Fi-Flow-04.png)

On voit que les fichiers sont déposés dans le dossier défini dans le processor :

![Nifi workshop](https://i.ibb.co/ZhRgrdx/Screenshot-from-2021-02-18-14-28-00.png)

### Etape 2 Découpage des données et envoi vers un broker Kafka.

![Nifi workshop](https://i.ibb.co/DrDgqRx/Screenshot-2021-02-18-Ni-Fi-Flow-05.png)

1. Récupérer les fichiers stockés dans le dossier de sortie :

![Nifi workshop](https://i.ibb.co/M1BjLsm/Screenshot-2021-02-18-Ni-Fi-Flow-06.png)

2. Déduire le nom du topic Kafka à partir du nom du fichier :

![Nifi workshop](https://i.ibb.co/m6cJmMb/Screenshot-2021-02-18-Ni-Fi-Flow-07.png)

3. Supprimer la ligne d'en-tête du fichier csv :

```sbtshell
timestamp,open,high,low,close,volume
2021-02-16 20:00:00,2109.9900,2110.6000,2109.9900,2110.6000,445
2021-02-16 17:05:00,2110.7000,2110.7000,2110.7000,2110.7000,1725
...
```
![Nifi workshop](https://i.ibb.co/HdY27RD/Screenshot-2021-02-18-Ni-Fi-Flow-08.png)

4. Envoyer les lignes OHLC vers Kafka :

![Nifi workshop](https://i.ibb.co/ZN6m37B/Screenshot-2021-02-18-Ni-Fi-Flow-09.png)

Attention : Pour que le producteur Kafka fonctionne il faut décrire le schéma du fichier CSV.

### Etape 3 : Récupération des données et stockage dans une base de données MongoDB.

> MongoDB est un système de gestion de base de données orienté documents, répartissable sur un nombre quelconque d'ordinateurs et ne nécessitant pas de schéma prédéfini des données.

![Nifi workshop](https://i.ibb.co/VHn9vqW/Screenshot-2021-02-18-Ni-Fi-Flow-10.png)

#### Créer un base de données MongoDB :

MLab est un site qui permet de créer une base de données MongoDB sur le cloud. Les instructions d'inscription et de création d'une BD sont en annexe de ce document.

1. Consommateur Kafka :

![Nifi workshop](https://i.ibb.co/W2ynpbF/Screenshot-2021-02-18-Ni-Fi-Flow-11.png)

MogoDB est une base de données basée sur le format Json. Il faut donc veiller à transformer chaque ligne csv reçue en Json.

2. On commence par extraire les données grâce à une expression régulière :

![Nifi workshop](https://i.ibb.co/0m368Tw/Screenshot-2021-02-18-Ni-Fi-Flow-12.png)

3. On construit le Json en modifiant le contenu du FlowFile à l'aide des valeurs extraites durant l'étape précédente :

![Nifi workshop](https://i.ibb.co/WkP23MJ/Screenshot-2021-02-18-Ni-Fi-Flow-13.png)

4. Envoi du Json créé vers la base de données MongoDB :

![Nifi workshop](https://i.ibb.co/D1pBzNw/Screenshot-2021-02-18-Ni-Fi-Flow-14.png)

### Les données sont dans MongoDB !!!

![Nifi workshop](https://i.ibb.co/d5mX2tc/Screenshot-2021-02-18-Data-Atlas-Mongo-DB-Atlas.png)


# Annexe

## Création d'une base de données MongoDB sur MLab

Aller sur https://mlab.com/ et créer un compte gratuit en selon les étapes suivantes :

![MLab installation](https://i.ibb.co/0hQDL6k/Screenshot-2021-02-18-Mongo-DB-Hosting-Database-as-a-Service-by-m-Lab.png)

![MLab installation](https://i.ibb.co/f8qw7Tr/Screenshot-2021-02-18-Sign-Up-for-Mongo-DB-Atlas-Cloud-Mongo-DB-Hosting.png)

![MLab installation](https://i.ibb.co/fNqnnmw/Screenshot-2021-02-18-Atlas-Onboarding-Mongo-DB.png)

Attention à choisir la troisième option, la gratuite !

![MLab installation](https://i.ibb.co/5MHP6CT/Screenshot-2021-02-18-Choose-a-Path-Atlas-Mongo-DB-Atlas.png)

Choisir AWS comme Cloud Provider et une région européenne pour l'hébergement.

![MLab installation](https://i.ibb.co/pKP3xhd/Screenshot-2021-02-18-Create-Cluster-Atlas-Mongo-DB-Atlas.png)

Après quelques minutes le cluster est créé, il est vide.

![MLab installation](https://i.ibb.co/j44733D/Screenshot-2021-02-18-Clusters-Atlas-Mongo-DB-Atlas.png)

Cliquer sur "Connect" pour définir les infos de connexions :
- Autoriser son l'IP publique de votre machine.
- Créer un utilisateur admin pour se connecter de l'extérieur. 

![MLab installation](https://i.ibb.co/VQ1j36t/Screenshot-2021-02-18-Clusters-Atlas-Mongo-DB-Atlas-1.png)

Choisir "Connect your application" :

![MLab installation](https://i.ibb.co/M2b67p4/Screenshot-2021-02-18-Clusters-Atlas-Mongo-DB-Atlas-2.png)

Choisir Java, version 3.4 et copier la chaine de connexions pour l'utiliser dans le Processor "PutMongo" sur NiFi :

![MLab installation](https://i.ibb.co/92V7ZCz/Screenshot-2021-02-18-Clusters-Atlas-Mongo-DB-Atlas-3.png)

Aller dans "Collection" pour créer une Base de données en cliquant sur "Add My Own Data" :

![MLab installation](https://i.ibb.co/HtkB8WV/Screenshot-2021-02-18-Data-Atlas-Mongo-DB-Atlas.png)

![MLab installation](https://i.ibb.co/ynGjBn8/Screenshot-2021-02-18-Data-Atlas-Mongo-DB-Atlas-1.png)

La base de données et la Collection sont créées. Reste plus qu'à les remplir grâce à NiFi :

![MLab installation](https://i.ibb.co/1Q78Gwp/Screenshot-2021-02-18-Data-Atlas-Mongo-DB-Atlas-2.png)
