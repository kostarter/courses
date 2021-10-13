# Apache NiFi

Sur leur site : **"Apache NiFi supports powerful and scalable directed graphs of data routing, transformation, and system mediation logic."**

## ETL, c'est quoi ?

Un logiciel ETL (Extract Transform Load) permet d’extraire des données brutes depuis une base de données, pour ensuite les restructurer, et enfin les charger dans une Data Warehouse. Ces logiciels existent depuis bien longtemps, mais ont beaucoup évolué pour répondre aux nouveaux besoins liés à l’essor du Cloud et du Big Data.
Ils doivent permettre l’ingestion en temps réel, l’enrichissement de données, la prise en charge de millions voire milliards de transactions. Ils prennent aussi en charge les données structurées ou non structurées en provenance de sources sur site ou sur le Cloud.

Pour résumer : Le défi dans les environnements d'entrepôt de données consiste à intégrer, réorganiser et consolider de grands volumes de données sur de nombreux systèmes, fournissant ainsi une nouvelle base d'information unifiée pour faire de la BI (Business Intelligence).

#### Exemples d'ETLs :

Xplenty : Une plate-forme qui offre une interface visuelle simple et intuitive pour créer des pipelines de données entre un grand nombre de sources et de destinations.

Talend ou Talend Open Studio : Développé par Talend, qui est une entreprise française. Permet d'exécuter des tâches ETL et d’intégration de données simples, d'obtenir des profils graphiques des données et et de la gestion de fichiers. Pour avoir travailler dessus chez LVMH je ne le recommenderais à personne tellement cette outil est une usine à gaz.

ODI (ex Sunopsys) : c'est un logiciel développé en java pour réaliser des tâches de type ETL/EAI (Enterprise Application Integration).
Il offre un environnement de développement graphique , c'est du client lourd. Multi-projets et multi-équipes. Il centralise les développements dans un référentiel indépendant des développements.

Pentaho : c'est un logiciel de business intelligence qui fournit l'intégration de données, les services OLAP, les rapports, les tableaux de bord d'informations, l'exploration et l'extraction de données, la transformation et le chargement.


## Apache NiFi, c'est quoi ?

Apache Nifi est un puissant outil open source de gestion de flux de données.</br>

- Il permet de gérer, automatiser, transformer, transférer des données entre plusieurs systèmes informatiques : Kafka, Amazon S3, HDFS, Base de données...
- Il est scalable et peut être déployé sur des clusters de manière distribuée.
- Il assure le No Data Loss, c'est à dire qu'il garantit la livraison de la donnée.
- Il gére la priorisation des flux, la bufferisatoin des données, la prise en compte des variations de débit (back pressure) et la gestion des flux (stream Vs. batch).
- Il permet la traçabilité des données.
- Il est extensible. Possibilité de créer ses propres processors/scripts.
- Il est sécurisé : SSL, HTTPS, chiffrement, etc...
- Il permet de gérer le versionnage des workflows dans Git grace au GitFlowPersistenceProvider.

Il posséde une interface web. Tout ce qu'il est possible de faire sur la webapp peut être fait via une API Rest.

**Pour résumer :** Le concept général est de créer des flux de données où les données sont acheminées à travers des processeurs pour réaliser des traitements dessus.

**Qui utilise Nifi ?**

Apache NiFi est de plus en plus populaire, surtout aux Etats-Unis. Le nombre de grandes entreprises qui l'utilisent augementent de 4% chaque année et parmi ces compagnies on peut trouver Verizon, American Express ou T Mobile.

**Avantages :**

- Interface web, Prototypage rapide (tester des traitements assez rapidement), facile d'utilisation, gratuit, open-source...
- Bon pour le transfert  sécurisé de données entre différents systèmes.
- Bon également pour la livraison de données sur les plateformes analytiques.
- Enrichissement et préparation des données : Conversion entre formats (Json vers Avro ou XML...), extraction, parsing, routage des données en fonction du type de données...

**Inconvénients :**

- N'est pas fait pour du développement complexe : du calcul distribué, des jointures ou aggregations entre des données se trouvant dans des fichiers différents ou des opérations complexes sur les données ou clacul de métriques.
- Autant utiliser une solution plus classique : Apache Spark par exemple !!


### Mode cluster :

Apache NiFi peut également être implémenté en mode cluster : les flux de données peuvent être distribués et traités à travers différents nœuds d’un cluster NiFi. 
Le schéma ci-dessous donne un aperçu d’une architecture NiFi distribuée : Sur cet exemple on voit encore une fois une architecture en mode Master / Slaves où NiFi possède un ou plusieurs nœuds esclaves contrôlés par un simple NCM (NiFi Cluster Manager) faisant office de maître.
Le NCM permet de récolter les informations issues de chaque nœud – dont leur statut – et de répliquer les requêtes associées sur les flux de données qui transitent au sein du système NiFi.

Mais on peut égalament configurer NiFi en mode no master pour éviter d'avoir le Single Point of Failure.

A noter qu’en cas d’indisponibilité du NCM, les nœuds esclaves continueront de fonctionner afin de maintenir la disponibilité des flux. La contrepartie sera, en revanche, qu’aucun nouveau nœud ne pourra rejoindre le cluster et aucune réplication inter-nœuds des changements associés aux flux ne sera effectuée, tant que le NCM ne sera pas restauré.


### FlowFile

Représente les données dans Nifi. Donc chaque pièce de données dans Nifi est un FlowFile.</br>
Chaque FlowFile possède des attributs sous le format Map clé/valeur (dont l'uuid) et un contenu (LA donnée). Ils sont persistés sur le disque dés leur création.

### Processor

Ce sont les processeurs qui travaillent : Ils eçoivent en entrée des FlowFile et en générent des nouveaux en sortie.</br>
Ils sont responsables de la récupération de données, et appliquent dessus des transformations, et des régles de routage... Ils peuvent être planifiés par timer ou par cron.</br>
Ils se refilent les références des FlowFile les uns aux autres pour avancer le traitement des données.</br>
Ils ont accès aux attributs des FlowFile. Ils peuvent les modifier, en rajouter ou en supprimer.</br>
Ils ont également accès au contenu des FlowFile.</br>
Ils peuvent tourner en paralléle (ce sont des threads).
Dérrière chaque processor c'est du Java.

**Astuces :** Show usage pour consulter la documentation d'un processeur.

Les plus connus : GetFile, PutFile, LogMessage, GenerateFlowFile...

Les processeurs sont groupés par catégorie :

1. De data transformation, c'est ce qui va appliquer des transformations sur le contenu d'un FlowFile (par exemple : ReplaceText, EncryptContent, CompressContent...).
2. De routing : Par exemple RouteOnAttribute qui en fonction d'un attribut du FlowFile va le diriger cers tel ou tel connection. Il y a aussi RouteOnContent.
3. Liés aux bases de données comme PutHiveQL qui permet d'exécuter une requête HQL sur une base de données Hive. Il y a aussi ExecuteSQL.
4. De manipulation des attributs, un des plus courants est UpdateAttribute ou ExtractText.
5. D'interraction avec le système, comme ExecuteProcess ou ExecuteStreamCommand.
6. D'ingestion de données : GeTFile, GetHDFS, GetHTTP... créent un FlowFile à partir d'un fichier sur une source particulère (le FileSystem, HDFS, Kafka, une URL...)
7. D'envoi de données : PutFile pour écrire du le FileSystem ou PutHDFS, PublishKafka...
8. Pour la fusion et l'aggrégation de données. Par exemple MergeContent permet de merger plusieurs FlowFile.
9. D'interraction avec le cloud : AWS, Azure...

Sur un cluster Nifi, avec X machines, chaque machine aura son processeur, par exemple GetFile, qui va écouter sur le dossier de sa machine.

**Gestion des attributs :**
UpdateAttribute permet d'accéder et de modifier des attributs du FlowFile.</br>
Il est également possible d'utiliser des attributs du processeur par d'autres processeurs de manière dynamique. Cela est possible pour les attributs pour lesquels est mentionné : "Expression language scope : Variable registry and FlowFile attributes."</br>
La syntaxe pour utiliser un attribut du FlowFile est ${nom_attribut} ou ${filename:toUpper()} pour appeler une fonction.

**Routing :**
RouteOnAttribute permet de gérer le routage entre deux processeurs. Pour cela il faut définir une stratégie de routage dans l'onglet properties.

**Variables et variables registry :**
Dans le fichier conf/nifi.properties vérifier la variable *nifi.variable.registry.properties* !
Cette propriété permet de définir un fichier pour définir des propriétés customisées qui pourront ensuite être utilisées comme attributs de processeur de la manière qui suit : ${udemy.example.in}.</br>
Important : Tout changement des fichiers de properties nécessite le redémarrage de Nifi.

On peut aussi créer des variables directement dans Nifi via l'interface avec un clic droit sur l'arrière plan du board.</br>
Important : Lorsqu'on crée une variable via Nifi, il n'est pas nécessaire de redémarrer Nifi, il s'en occupe lui-même.

### Connector

Ils font les liens entre les processeurs. Ils définissent des régles de filtrage et de priorité des FlowFile à traiter.

**FlowFile expiration :**
Pour définir au bout de combien de temps on veut qu'un FlowFile qui rencontre cette queue soit supprimé.</br>

**Back Pressure :**
Pour définir le nombre d'objets maximum ou la taille maximum que peut contenir une queue. Cela permet d'éviter la surchage et le blocage du traitement.</br>

**Priorités :**
Définit la priorité pour traiter les FlowFile en entrée (FirstInFirstOut, PriorityAttribute, etc...).</br>

**Load Balancing Strategy :**
 Distribuer à travers le cluster ou pas ? Le plus utilisé est le Round Robin pour distribuer équitablement.

### Process Group

Un ensemble de processeurs et de leurs connections. On peut le voir comme dossier sur un système de fichiers.</br>
Ils permettent de découper le flow en fonctions. Pour cela on utilise les outpout et les input ports qui permettent de matérialiser le lien entre le process group et le reste du monde.</br>
Il est conseillé de toujours travailler à l'intérieur des process group, cela permet de démarrer ou d'arrêter les processeurs plus rapidement, cela permet aussi de découper les fonctions. Cela permet aussi de monitorer par process group.</br>

**Remote process group :** Permettent de faire du load balancing.

### Templates

Permet de créer des templates réutilisables contenant des processeurs, des connections, des groupes, etc...
Par exemple un LogMessage avec des valeurs prédéfinis à appliquer sur chaque ligne de code.</br>
Il est possible de télécharger un template au format XML pour le charger sur un autre flow.

https://cwiki.apache.org/confluence/display/NIFI/Example+Dataflow+Templates

### Monitoring

Les informations sont affichées sur la barre de monitoring en dessous de la barre d'outils, elles sont relatives au cluster.</br>
Il y a également les statistiques affichées sur chaque processeurs : **In** (nb de FlowFile en entrée), **Out** (nb de FlowFile en sortie), **Read/write** (le de bytes lu et écrit par le processeur), **Tasks/Time**. Ces informations sont relatives au 5 dernière minutes.</br>
Les informations sont raffraichies toutes les 30 secondes.</br>
Bulletin Indicator : C'est un log d'erreur qui va s'afficher directement sur le processeur. Accessible via le menu contextuel en haut à droite. Egalement sur une période 5 minutes.

**Status History :** Menu contextuel sur les composants.

**NiFi Summary :** Menu contextuel en haut à droite pour une vue globale. Sur l'ecran apparait un lien *System diagnostics* pour des informations sur le système (JVM, RAM, etc...).

**Les logs :** Dans le fichier logs/nifi-app.log !

Pour visualiser les metriques dans un outil externe comme Graphana ou Prometeus :
Aller dans le menu contextuel à droite et choisir *Controller Settings*. Dans l'onglet *Reporting Tasks* vers ou remonter les metriques (par exemple celles d'Ambari).

**Data lineage :**
Permet de suivre un FlowFile, toutes les modifications qu'il y a eu dessus, que ce soit au niveau des attributs ou du contenu,  par quel processeur il est passé, etc...
Pour cela aller dans le menu contextuel à droite et choisir *Data Provenance*. Les champs de recherche peuvent être configurés dans le nifi.properties dans la section nifi.provenance.repository.indexed.fields !

#### Créer un processor avec Java

Sous intellij :
https://how-to-nifi.blogspot.com/2017/10/how-to-make-new-nifi-processor.html</br>
Une fois le ficher .nar produit le copier dans le répertoire lib de nifi.

## Nifi et Kafka

Lancer ZooKeeper :

```sbtshell
$ ./zkServer.sh start
```

Lancer Kafka :

```sbtshell
$ ./kafka-server-start.sh ../config/server.properties
```

Creation de topic :

```sbtshell
$ ./kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test-creation
$ ./kafka-topics.sh --zookeeper localhost:2181 --list
```

</br></br>Consumer :

```sbtshell
$ ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-creation
```

Producer :

```sbtshell
$ ./kafka-console-produr.sh --broker-list localhost:9092 --topic test-creation
```

Dans Nifi :</br>
On a ConsumeKafka et ConsumeKafkaRecord. Dans le cas du deuxième on donne un schéma (Schema registry) au consumer.
La même chose pour PublishKafa et PublishKafkaRecord.
