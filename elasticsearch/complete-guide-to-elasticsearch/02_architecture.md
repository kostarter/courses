# Architecture

Dans Elasticsearch chaque noeud est une instance d'elasticsearch, pas forcément une machine. On peut avoir plusieurs noeuds sur la même machine (en production cela est déconseillé).  
Chaque noeud appartient à un cluster, où un cluster est un ensemble de noeuds qui, ensemble, contiennent les données. Chaque cluster doit répondre à un besoin différent, par exemple un cluster pour le e-commerce, un autre pour l'APM, etc.  
Qaund un noeud est lancé, il va rejoindre un cluster existant si configuré, ou il crée son propre cluster contenant juste ce noeud.  
Les unités stockées dans le cluster sont des documents et sont au format JSON. Quand on indexe un document, l'objet JSON original envoyé à Elasticsearch est stocké avec des méta-données. Ces documents sont stockés dans un index. Chaque index regroupe des documents avec des caractéristiques similaires, et fournit des options de configuration pour la disponibilité des données et la scalabilité.

Découvrir un cluster :
```
GET _cluster/health
```
La première partie _cluster est l'API à laquelle on veut accéder, la partie health représente la commande.

```
GET _cat/nodes?v
GET _cat/indices?v
```
Le paramètre `v` dit à Elasticsearch de décrire les données avec un en-tête.

### Sharding et scalabilité :

Le sharding est une manière de diviser un index en plusieurs parties, où chaque partie est appelée un shard. Un shard est stocké sur un seul noeud. 
Le sharding est fait au niveau de l'index et non pas celui du cluster ou du noeud. La raison est qu'un index peut contenir des milliards de documents, alors qu'un autre seulement quelques milliers.

La raison principale pour laquelle on divise un index en plusieurs shard est la capacité de scaler horizontalement.
Un shard peut être considéré comme un index indépendant. Chaque shard est un index Apache Lucene. Il n'a pas de taille prédéfinie ou de taille limite, il grandit au fur et à mesure que des documents y sont stockés.
Une autre raison pour le sharding est l'amélioration des performances car il permet la parallélisation des requêtes, ce qui améliore considérablement le débit.

```
GET _cat/indices?v
```
La colonne `pri` décrit les shards primaires.

Chaque index contient un shard par défaut. Avant la version 7.0.0 les index étaient créés avec 5 shards, ce qui n'était pas nécessaire pour les petits index (over-sharding). A signaler que le nombre de shards n'est pas modifiable une fois que l'index a été créé. Donc pour augmenter le nombre de shards il faut splitter les données grâce à la Split API. Das l'autre sens, réduire le nombre de shards peut se faire avec la Shrink API.

##### Question : Combien de shards ?

Il n'y a pas de réponse définitive à cette question. Cela dépend de plusieurs facteurs : le nombre de noeuds et leurs capacités de stockage, le nombre d'index et leurs tailles, le nombre de requêtes, etc.

Pour des millions de documents, toujours rajouter deux shards. Une bonne stratégie serait de choisir cinq shards.

Si quelques milliers garder la configuration par défaut.

La multiplication de shards ralentit l'ecriture mais rend la lecture très performante.

### Réplication : 

Une panne de disque dur arrive, spécialement lorqu'on en utilise beaucoup. Alors il faut s'assurer qu'une panne de disque dur ne soit pas un désastre.  
Elasticsearch fournit la réplication pour la tolérance à l'erreur, c'est activé par défaut (la valeur par défaut est 1).  
Configurer la réplication peut être pénible pour un grand nombre de base de données, mais cela est extrêmement simple et bien intégré dans Elasticsearch.

##### Comment ça marche ?

Un index est configuré pour stocker ses données dans un nombre de shards qui peuvent être stockés sur plusieurs noeuds.  
La réplication est configurée au niveau de l'index. Elle permet de créer des copies de shards, que l'on appele des réplicas. Un shard qui a été répliqué est appelé un shard primaire.  
Un shard primaire et ses réplicas sont appelés un groupe de réplication.  
Un shard de réplication est une copie compléte d'un shard et peut répondre à des requêtes de recherche comme un shard primaire.  
Un shard de réplication n'est jamais stocké sur le même noeud que son shard primaire. Cela veut dire que si un noeud disparait, il restera au moins une copie des données du shard sur un autre noeud. Combien de copies resteront disponibles dépend du nombre de réplicas configuré pour l'index et du nombre de noeuds que contient le cluster.  
La réplication n'a de sens que sur les clusters avec plusieurs noeuds.

##### Combien de réplicas ?
Typiquement un ou deux réplicas suffisent, mais cela dépend de la criticité de l'environnement. Par exemple si deux noeuds tombent en panne en même temps, est-il possible de restaurer les données à partir d'une autre source de données, comme un repository de fichiers CSV ou une base de données relationnelle ? Est-ce grave si les données ne sont pas disponibles pendant l'opération de restauration ?

Donc, répliquer une fois si la perte des données n'est pas un désastre, et deux fois ou plus pour les systèmes critiques.

Elasticsearch fournit la possibilité de créer des snapshots pour faire des sauvegardes de backup. Elles peuvent être utilisées pour restaurer les données telles qu'elles étaient à un instant donné.  
Il est possible de créer des snapshots pour un index spécifique ou pour l'ensemble du cluster.  
Les snapshots ne donnent pas l'état du cluster en temps réel, mais donnent la possibilité de faire des copies avant de réaliser des grosses opérations de mise à jour sur les données du cluster. Cela permet de revenir sur les modifications si quelque chose se passe mal. Ils peuvent également être utilisés pour des sauvegardes quotidiennes.  

La réplication peut également être utilisée pour augmenter le débit d'un index.
Les shards de réplication d'un groupe de réplication peuvent traiter différentes requêtes de recherche simultanément. Cela augmente le nombre de requêtes qui peuvent être traitées en même temps. Elasticsearch dirige intelligemment les requêtes vers le shard le mieux adapté. Il coordonne où les requêtes seront exécutées et la parallélisation. Comment peut-on exécuter 3 requêtes différentes sur seulement deux noeuds ? Les machines d'aujourd'hui possédant plusieurs coeurs (au moins 4), cela permet au CPU de lancer simultanément plusieurs tâches sur chacun des coeurs en utilisant des Threads.

**Réplication = Disponibilité + Débit !**

```
GET /_cat/shards?v
```
**Important :** Pour les index de Kibana le paramètre auto_expand_replicas est utilisé pour étendre automatiquement les réplicas lorsqu'un noeud est rajouté.

### Nodes :

Les données sont stockées sur des shards, et les hards sur des noeuds.
Pour scaler Elasticsearch, il faut créer des noeuds additionnels dans le cluster.

Sur la même machine - Approche 1 :

1. Télécharger et extraire Elasticsearch.

2. Configurer les noms du cluster et des noeuds. Le noeud par défaut d"un cluster est elasticsearch.

3. Lancer Elasticsearch. 

Sur la même machine - Approche 2 :

1. Réutiliser le même dossier Elasticsearch.

2. Lancer Elasticsearch en specifiant les paramètres cluster.name, node.name, path.data et path.logs en ligne de commande.

##### Les roles des noeuds :

* **Master :** mettre le paramètre `node.master` à true.  
Le noeud master est responsable de la création et suppression des index, le tracking des autres noeuds, l'allocation des shards aux noeuds...  
Un noeud avec ce role ne sera pas automatiquement élu noeud master, à moins qu'il n'y ait aucun autre noeud éligible. La raison est que le noeud master d'un cluster est élu par le biais d'une votation, si plusieurs noeud ont ce role, un d'eux sera élu en tant que master.  
Pour les clusters de grande taille, il peut être utile d'avoir des noeuds master dédiés. Avoir un noeud master stable est crucial pour assurer la stabilité du cluster.  
Si le master élu est trop occupé à faire autre chose, comme des requêtes de recherche, la stabilité du cluster peut être affectée. Les opérations de recherche et d'ecriture sont gourmandes, il serait judicieux d'opter pour un noeud master dédié.  

* **Data :** mettre le paramètre `node.data` à true.  
Habilite un noeud à stocker des données. Ceci implique l'exécution des requêtes en rapport avec la donnée, comme les requêtes de recherche ou de modification.  
Pour les clusters de petite ou moyenne taille, ce role est toujours activé. Mais pour les clusters plus grands, dédier un noeud master, empêche celui-ci de stocker les données.  

* **Ingestion** : mettre le paramètre `node.ingest` à true.  
Habilite un noeud à exécuter des pipelines d'ingestion, c'est à dire une série d'étapes qui sont exécutés lors de l'ingestion de documents dans Elasticsearch. L'ingestion étant l'action d'indexer un document dans Elasticsearch.  
Les étapes d'ingestion sont définis par des processeurs qui manipulent les documents avant leur indexation. Ces manipulations peuvent être l'ajout ou la suppression de champs, le changement de valeurs (transformer une adresse IP en géolocalisation), etc...  
Cela équivaut à une version simplifiée de Logstash dans Elasticsearch. Elle offre beaucoup de fonctionnalités d'ingestion mais pour une solution d'ingestion plus large utiliser Logstash.

* **Machine Learning** : mettre le paramètre `node.ml` et `xpack.ml.enabled` à true.
Le node.ml habilite un noeud à lancer des jobs de machine learning.
Le xpack.ml.enabled habilite le noeud à répondre à l'API de machine learning.  

* **Coordinaiton :**
Par coordination on entend la manière dont Elasticsearch distribue les requêtes en interne. Ce type de noeud, ne cherche pas les données par lui-même, mais délégue cette tâche aux noeuds de données. Cela peut être utile pour les clusters de grande taille où il sera utilisé comme un load balancer.  

* **Voting-only :** mettre le paramètre `node.voting_only` à true.  
Très rarement utilisé pour ne pas dire jamais. Il participe au process de votation pour élire un nouveau noeud master.

Par défaut, chaque noeud posséde un role `dim` (ou cdhilmrstw) : données, ingestion et master. Ce qui veut dire qu'ils sont tous éligibles pour être master. 
```
node.role, r, role, nodeRole
    (Default) Roles of the node. Returned values include c (cold node), d (data node), 
    f (frozen node), h (hot node), i (ingest node), l (machine learning node), 
    m (master-eligible node), r (remote cluster client node), s (content node), 
    t (transform node), v (voting-only node), w (warm node), and - (coordinating node only). 
```

Quand changer les roles des noeuds ?  
Cela dépend. Pour une meilleure traçabilité des requêtes et de l'usage des ressources. Pour optimiser l'exécution des requêtes...  
A ne faire que lorsqu'on sait ce qu'on fait et que l'on a une bonne raison de le faire.
