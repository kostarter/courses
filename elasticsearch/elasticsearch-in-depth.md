# Elasticsearch 7 and Elastic Stack - In Depth and Hands On!

![alt text](https://i.ibb.co/XXJ2Y4p/c652d73656375726974792d322e6a7066.png)

Elasticsearch
==

##### Présentation

Elasticsearch fait partie de la stack Elastic. Il est basé sur Apache Lucene qui est un framework de recherche de texte basé sur les index inversés.
Il est parfaitement scalable de manière horizontale, il décompose les données en shards et les réplique ce qui le rend fault tolerant. 
Chaque shard est en soi un index Lucene inversé, ce qui rend d'autant plus rapide les recherches sur du texte.
Mais Elasticsearch n'est pas juste un outil de recherche sur du contenu textuel, il peut aussi gérer d'autres types de données telles que les dates, les données numériques et permet donc de faire des aggregations et des calculs.

Elasticsearch offre une API REST qui permet d'exécuter toutes les opérations possibles pour l'indexation des documents, la recherche et la gestion des index.


### Préparation de l'environnement

Ouverture des ports :

* Elasticsearch : 9200
* Kibana : 5601
* SSH : 22


#### Installation Java 8
```sbtshell
$ sudo yum install java-1.8.0-openjdk-devel
```

#### Installation Elasticsearch 7

Importer la clé publique pour elasticsearch :
```sbtshell
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

$ cd /etc/yum.repos.d/
$ nano elasticsearch.repo
[elasticsearch-7.x]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md

$ sudo yum install elasticsearch
```

Les fichiers de configuration et le dossier par défaut pour les fichiers de logs :
```sbtshell
/etc/elasticsearch/elasticsearch.yml
/etc/elasticsearch/jvm.options

/var/log/elasticsearch
```

Pour lancer elasticsearch en tant que service :
```sbtshell
sudo service elasticsearch start
```
### Shakespeare example

Dire à elasticsearch comment stocker le jeu de données Shakespeare :
```sbtshell
$ wget http://media.sundog-soft.com/es7/shakes-mapping.json
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json
```
TODO : Est-ce-que cela crée l'index ? OUI !

```json
{
	"mappings" : {
		"properties" : {
			"speaker" : {"type": "keyword" },
			"play_name" : {"type": "keyword" },
			"line_id" : { "type" : "integer" },
			"speech_number" : { "type" : "integer" }
		}
	}
}
```

Rajouter  à l'index :
```sbtshell
$ wget http://media.sundog-soft.com/es7/shakespeare_7.0.json
$ curl -H 'Content-Type: application/json' -XPOST 'localhost:9200/shakespeare/_doc/_bulk?pretty' --data-binary @shakespeare_7.0.json
```

To make a research :
```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '
{
 "query" : {
  "match_phrase" : {
   "text_entry" : "to be or not to be"
  }
 }
}'

$ curl -XDELETE localhost:9200/shop
```

### Les concepts logiques d'Elasticsearch

- **Index :** 

    Entité haut niveau sur laquelle il est possible de faire des requêtes dans Elasticsearch. Peuvent contenir des collections de documents ou de types (équivaut à une BDD).<br/>
    C'est la structure dans laquelle sont référencés des documents. Du même type donc, depuis la version 7.<br/>
    Chaque index est géré par un index inversé qui nous permet de chercher très rapidement à travers les documents qui y sont indexés.<br/>
    => Index inversé c'est ce qui se trouve à la fin de chaque bouquin, où on a les termes et pour chaque terme les pages où il se trouve.

- **Documents :** 

    On peut les considérer comme les lignes dans une base de données relationnels.<br/>
    Quand on envoie des requêtes des recherches dans elasticsearch c'est ce qu'on recherche. Ces documents ne contiennent pas uniquement du texte, toute donnée structurée JSON peut être prise en compte. 
    Chaque document posséde un identifiant unique, que l'on spécifie lorsqu'on indexe le document, sinon elasticsearch se charge de lui en générer un.<br/>
    Il possède également un type, qui décrit ce que contient le document, les champs, les types... Un document ça peut être par exemple un article Wikipédia ou une ligne de logs crachée par un serveur Apache.

- **Types :** 

    Un schéma ou un mapping commun à plusieurs documents. Il décrit la représentation de ces documents (équivaut à une table en BDD).<br/>
    Le concept Type est devenu obsolète depuis la version 6.<br/>
    C'est ce qui définit la structure du document.
    Comme c'est du semi structuré, ce schéma n'est pas gravé dans le marbre, il est souple. C'est pas parce qu'on définit un type film sans rating que l'on ne pourra pas indéxer des films avec un rating.
    
    Mais voila à partir de la version 7 d'Elasticsearch c'est un concept qui a changé puisqu'il n'est plus possible de stocker des documents de types différents dans le même index.
    Donc ce n'est plus le type qui est porteur du mapping mais l'index.

    **Pour schématiser : On peut parler de base de données en ce qui concerne l'index, de table pour ce qui est des types et de lignes pour les documents.**

- **Shards :**

    Les documents sont dans des shards. Les shards sont répartis sur les noeuds du cluster.<br/>
    Chaque shard contient un index Lucene de lui-même.
    
    Comment fonctionne la réplication des shards en primaires et replicats ?
    Si un noeud contenant un shard primaire, Elasticsearch le détecte et transforme un shard réplicat.
    
    Les opérations d'écriture sont dirigées vers le shard primaire, puis recopiées sur les réplicats.
    Les opérations de lecture sont dirigées vers le shard primaire ou n'importe quel réplicat.
    
    Le nombre de shards primaires ne peux pas être modifiés après création de l'index.
    Multiplier le nombre de réplicats améliore les performances de lecture mais ralentit l'écriture.
    
    **Donc pour choisir le nombre de shards primaires il faut prendre en compte la nature du cluster : est ce qu'il sera sujet à beaucoup d'ecritures ou de lectures ?
    Par exemple pour Wikipedia ou ce sera principalement de la lecture avec beaucoup de recherche avoir peu de shards principaux n'est pas un problème parce que la charge sera absorbée par les replicats.
    Par contre si on a de l'ingestion de logs en continue il vaut mieux multiplier les shards principaux pour répartir la charges d'ectiture qui elle dans un premier temps ne concerne pas les replicats.**
    
    La décision doit se faire avant la création de l'index !!
    Ceci dit ré-indexer un index n'est pas un drame. Cela se fait assez rapidement et peut se faire sans toucher l'autre index.
    
    Donc prévoir large mais pas trop ! On peut anticiper sur l'augmentation des besoins sur une année. Et puis au besoin, ré indexer en ajustant le numéro de shards.
    Il n'y a pas de *best practice* universelle. Chaque contexte a une configuration qui le fera fonctionner de manière optimale. C'est du fine tuning.
    
    Une autre stréatégie est de partir sur une configuration minimale et de stresser l'index créé jusqu'à ce qu'il étouffe et ensuite rajouter de manière incrémentale des shards jusqu'à trouver la configuration optimale.
    Utiliser des alias pour regrouper des index.

- **Mapping :**

    C'est la définition du schéma.
    Cela permet de dire à Elasticsearch comment stocker les données, quel format, comment l'indexer et comment l'analyser.
    
    - Définir le type d'un champs, voir exemple ci-dessous.
    - Définir si un champ est indexé ou pas.
    - Définir ules analyzers : character filters, tokenizer, token filter...

    ```sbtshell
    $ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
    {
     "mappings" : {
      "movie" : {
       "properties" : {
        "year" : { "type": "date" }
       }
      }
     }
    }'
    ```
    
    Version 2 :
    ```sbtshell
    $ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
    {
     "mappings" : {
      "movie" : {
       "properties" : {
        "id" : { "type": "integer" },
        "year" : { "type": "date" },
        "genre" : { "type": "keyword" },
        "title" : { "type": "text",  "analyzer": "english" }
       }
      }
     }
    }'
    ```

### Les opérations de base

- **Création d'un index :**
  
Création d'un index en spécifiant les settings pour ce qui est des nombres de shards ou de réplicats.<br/>
On peut utiliser les templates de création d'index.

```sbtshell
GET /shakespeare/_settings

PUT /testindex
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }
}
  
GET testindex/_settings
```

La liste des index :
```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/_cat/indices?v'
```

- **Lire le mapping d'un index :**

TODO : A verifier.
```sbtshell
$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_mappings/movie?pretty
```

- **Insérer une entrée dans l'index et la récupérer :**
```sbtshell
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/movie/109487 -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar",
  "year": 2014
}'

$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/movie/_search?pretty
```

- **Import :**

TODO : Dans quel index ?
```sbtshell
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @movies.json
```

- **Mise à jour :**

Les documents sont immutables dans Elasticsearch.
A chaque mise à jour, Elasticsearch crée un nouvelle version du document et marque l'ancienne version comme 'à supprimer'. Elle sera supprimée par Elasticsearch plus tard.

Marche avec _update ou avec un simple create sur le même id.
```sbtshell
$ curl -H "Content-Type: application/json" -XPOST 127.0.0.1:9200/movies/movie/109487/_update -d '
{
  "doc": {
    "title": "Interstellat Woo"
  }
}
```

- **Suppression :**
```sbtshell
$ curl -H "Content-Type: application/json" -XDELETE 127.0.0.1:9200/movies/movie/109487
```

- **Accés concurrents :**

Utiliser les numéros de version pour mettre à jour.
```sbtshell
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/movie/109487?version=4 -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar Ho Ha !",
  "year": 2014
}
'
```

WARN : Mais avec la version 7 on n'utilise plus la version mais "_seq_no" et "_primary_term" !!!
```sbtshell
$ curl -H "Content-Type: application/json" -XPUT "127.0.0.1:9200/movies/_doc/109487?if_seq_no=6&if_primary_term=1" -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar Ho Ha !",
  "year": 2014
}'
```
=> Si on le refait une deuxième fois on aura une erreur de concurrence.

TODO : Comment gére-t-il les numéros de séquences ??!!!

Pour se faciliter la vie utiliser le retry_on_conflict !

```sbtshell
$ curl -H "Content-Type: application/json" -XPOST 127.0.0.1:9200/movies/movie/109487/_update?retry_on_conflict=5 -d '
{
  "doc": {
    "title": "Interstellat Woo"
  }
}
```

### Les analyseurs

![alt text](https://i.ibb.co/k0vtf75/analysis-chain.png)

Le type keyword mapping pour supprimer l'analyseur sur le champ : seul une correspondance exacte est admise.<br/>
Le type text pour permettre l'analyse.<br/>
En fonction de l'analyseur les résultats seront sensibles à la casse ou pas, filtrés, associés à des synonymes...

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
 "query" : {
  "match" : {
   "title" : "Star Trek"
  }
 }
}'
```

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
 "query" : {
  "match_phrase" : {
   "genre" : "sci"
  }
 }
}'
```

### Modéle de données

- Normaliser ou dénormaliser ? That is the question !!!!

La normalisation optimise l'espace en évitant la redondance de données mais implique de multiplier les appels pour aggreger les données.<br/>
Actuelement l'espace de stockage ne coute pas très cher, donc peut être vaut-il mieux dénormaliser pour améliorer les perfs.<br/>
Reflechir aussi à la fréquence à laquelle les données peuvent être modifiées. Avoir à propager les modifications sur les données dénormalisées est douloureux aussi !

- Relation Parent / Enfant :
```sbtshell
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/series -d '
{
 "mappings" : {
  "movie" : {
   "properties" : {
    "film_to_franchise" : { "type": "join", "relations": { "franchise": "film"} }
   }
  }
 }
}'
```

Parent et enfant doivent être rajouté dans le même shard.
```sbtshell
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @series.json
```

```json
    {
       "create":{
          "_index":"series",
          "_type":"movie",
          "_id":"1",
          "routing":1
       }
    }
    {
       "id":"1",
       "film_to_franchise":{
          "name":"franchise"
       },
       "title":"Star Wars"
    }
    {
       "create":{
          "_index":"series",
          "_type":"movie",
          "_id":"260",
          "routing":1
       }
    }
    {
       "id":"260",
       "film_to_franchise":{
          "name":"film",
          "parent":"1"
       },
       "title":"Star Wars: Episode IV - A New Hope",
       "year":"1977",
       "genre":[
          "Action",
          "Adventure",
          "Sci-Fi"
       ]
    }
```

Récupérer tous les enfants de la franchise "Star Wars" :
```sbtshell
$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/series/movie/_search?pretty -d '
{
  "query" : {
    "has_parent" : {
      "parent_type" : "franchise", 
      "query" : { "match" : { "title" : "Star Wars" } }
    }
  }
}'
```

Récupérer le parent ayant pour fils :
```sbtshell
$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/series/movie/_search?pretty -d '
{
  "query" : {
    "has_child" : {
      "type" : "film", 
      "query" : { "match" : { "title" : "Star Wars: Episode IV - A New Hope" } }
    }
  }
}'
```

### Recherche
   
#### Query lite
Plutot illisible.
```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?q=+title:star&pretty'
```
```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?q=+year:>2010+title:trek&pretty'
```
  
Attention : Les caractères spéciaux doivent être encodés !<br/>
https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-uri-request.html

#### Json :

##### Filtres
Demande une réponse oui/non aux données.
```console
GET /_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : {"<field_name>" : "<your_value>"}
            }
        }
    }
}
```
##### Requêtes
Retourne les données en se basant sur la pertinence.

Utiliser les filtres autant que possible car : plus rapides et gérés par le cache !

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
    "query" : {
         "match" : {
          "title" : "star"
         }
    }
}'
```

##### Boolean query
Pour combiner des critéres et pour faire un AND elle doit être de type "must" :

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
    "query" : {
         "bool" : {
              "must" : { "term" : { "title" : "trek" }},
              "filter" : { "range" : { "year" : { "gte" : 2020 }}}
         }
    }
}'
```

**Autre filtres :** term, terms, range, exists, missing, bool ...<br/>
**Types de requêtes :** match, match_all (par défaut), multi_match, bool, prefix, wildcard...

##### Recherche de phrase
Dans les index inversés l'ordre des mots est également stocké.

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
"query" : {
 "match_phrase" : {
  "title" : "star wars"
 }
}
}'
```

##### Le slop
Dans cet ordre mais pas nécessairement l'un après l'autre. Il contient le nombre de mots maximum d'écart entre les deux mots de la phrase.
```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
"query" : {
 "match_phrase" : {
  "title" : { "query" : "star beyond", "slop" : 1 }
 }
}
}'
```

##### Pagination
Définir un interval avec les paramètres *from* et *size*. Penser à mettre des limites max pour ne pas ruiner les performances.

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
    "from" : 2,
    "size" : 2,
    "query" : {
        "match_phrase" : {
            "genre" : "Sci-Fi"
        }
    }
}'
```

##### Tri
Utiliser le paramètre sort. Exemple : sort=year

Un champs text analysé en mode full-text ne peut être utilisé pour trier des documents.<br/>
Ceci parce qu'il existe dans l'index inversé sous forme de mots individuels et non plus comme une phrase complète.<br/>
Pour contourner cela on peut dupliquer le champs sous forme de keyword, qui ne sera pas analysé et sera stocké en tant que phrase, pouvant ainsi être utilisé pour le tri.

```sbtshell
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
"mappings" : {
 "movie" : {
  "properties" : {
   "id" : { "type": "integer" },
   "year" : { "type": "date" },
   "genre" : { "type": "keyword" },
   "title" : { 
    "type" : "text",  
    "analyzer" : "english",
    "fields" : {
     "raw" : {
      "type" : "keyword"
     }
    }
   }
  }
 }
}
}'
```

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?sort=title.raw&pretty'
```

##### Requêtes Fuzzy
Utilise la distance Levenshtein pour rendre la recherche plus tolérante aux erreurs.
Cette distance prend en compte les substitutions, insertions et suppressions de caractère.

```sbtshell
$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
"query": {
 "fuzzy": {
  "title": {"value": "intrsteller", "fuzziness": 2}
 }
}
}'
```

##### Les correspondances partielles
Ne peut se faire que sur des string !

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '{"query": {"prefix": {"year": "201"}}}'
```
=> "reason" : "Can only use prefix queries on keyword and text fields - not on [year] which is of type [date]",

- Query time - search as you type :
```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match_phrase_prefix": {"title": {"query": "sta be","slop": 10}}
  }
}'
```

##### N-grams
Unigram, bigram, trigram, 4-gram ...
On peut aussi utiliser les edge n-grams pour ne calculer les n-grams que pour les débuts et fins des mots.

1. Il faut créer un analyseur de type "autocomplete" avec un filtre "edge_ngram".
2. L'utiliser comme analyseur sur le champs.

### Aggregations
   
Créer une aggrégation par rating donne le nombre de documents pour chaque valeur de rating :

```sbtshell
$ curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d ‘
{
 "aggs": {
   "ratings": {
     "terms": {
       "field": "rating"
     }
   }
 }
}'
```

Ajouter un filtre à l'aggrégation en amont :

```sbtshell
$ curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d ‘
{
 "query": {
   "match": {
     "rating": 5.0
   }
 },
 "aggs" : {
   "ratings": {
     "terms": {
       "field" : "rating"
     }
   }
 }
}'
```

Calcul de moyenne pour un film :

```sbtshell
$ curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d ‘
{
 "query": {
   "match_phrase": {
     "title": "Star Wars Episode IV"
   }
 },
 "aggs" : {
   "avg_rating": {
     "avg": {
       "field" : "rating"
     }
   }
 }
}'
```

**Nested aggregations :**<br/>
Calcul de moyenne par film de la série Star Wars :

```sbtshell
$ curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d ‘
{
 "query": {
   "match_phrase": {
     "title": "Star Wars"
   }
 },
 "aggs" : {
   "titles": {
     "terms": {
       "field": "title.raw"
     },
     "aggs": {
       "avg_rating": {
         "avg": {
           "field" : "rating"
         }
       }
     }
   }
 }
}'
```

**Important :**<br/>
Sans title.raw ça ne marcherait pas parce que dans l'index le champs title n'existe pas en soit, car non déclaré en tant que field data.<br/>
Mais même en le déclarant en tant que field data l'aggrégation ne se fera pas sur le titre en entier mais mot à mot.

Il faut recréer l'index !
En créant un champs raw en tant que keyword dans le champs title.

**Histograms :**

Pour générer des histogrammes sur les données par intervales :

```sbtshell
$ curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d ‘
{
 "aggs" : {
   "whole_ratings": {
     "histogram": {
       "field": "rating",
       "interval": 1.0
     }
   }
 }
}'
```

**Time series :**<br/>
Regrouper les entrées par heures :
Elasticsearch a une très bonne gestion des dates avec la possibilité de lire différents formats, de gérer les fuseaux horaires. Il permet aussi de faire des opérations sur les dates avec des mots clés comme now par exemple.<br/>
Il peut faire des recoupements ou des agrégations par dates de manière assez intuitive.

```sbtshell
$ curl -XGET '127.0.0.1:9200/logstash2015.12.04/_search?size=0&pretty' -d ‘
{
 "aggs" : {
   "timestamp": {
     "date_histogram": {
       "field": "@timestamp",
       "interval": "hour"
     }
   }
 }
}'
```

##### Structures de requêtes

```sbtshell
        query
        
            match_phrase
                aField : "Some text ..."
                
            match
                aField : ""
        
            prefix
                aField : ""
        
            wildcard
                aField : ""
        
            bool
                must
                    term
                filter
                    range 
                        gte
            aggs
                aggName
                    histogram
                        field
                        interval
        
            aggs
                aggName
                    date_histogram
                        field
                        interval
        
        mappings
        
            nameOfMapping (deprecated since V7)
                properties
                    nameOfProperty : { type : aType }
                    nameOfProperty : { index : boolean }
                    nameOfProperty : { analyzer : anAnalyzer }
```

### Administration

##### Plantage de noeuds

```sbtshell
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/_cluster/health?pretty'
```

##### Snapshots

Ajouter dans elasticsearch.yml:
```sbtshell
path.repo: ["/home/<user>/backups"]
```
```sbtshell
PUT _snapshot/backup-repo
{
  "type": "fs",
  "settings": {
    "location": "/home/<user>/backups/backup-repo"
  }
}
```
Créer un spapshot de tous les indexes :
```sbtshell
PUT _snapshot/backup-repo/snapshot-1
```
Restaurer tous les indexes :
```sbtshell
POST /_all/_close
POST _snapshot/backup-repo/snapshot-1/_restore
```

##### Réinitialiser les settings de création d'index

```sbtshell
$ curl -H "Content-Type: application/json" -XPUT 172.21.173.199:9200/_cluster/settings -d '
{
	"transient" : {
		"indices.recovery.*" : null
	}
}'

$ curl -H "Content-Type: application/json" -XPUT 172.21.173.199:9200/_cluster/settings -d '
{
	"persistent" : {
		"indices.recovery.*" : null
	}
}'
```

##### Activer le monitoring
```sbtshell
$ curl -k -u elastic:orange -H "Content-Type: application/json" -XPUT https://172.21.173.199:9200/_cluster/settings -d '
    {
    "persistent" : {
        "xpack" : {
            "monitoring" : {
                "collection" : {
                    "enabled" : "true"
                }
            }
        }
    }
}'
```

##### La santé du cluster
```sbtshell
$ curl -k -u elastic:orange -XGET https://172.21.173.198:9200/_cluster/health?pretty

$ curl -k -u elastic:orange -XGET https://172.21.173.198:9200/_cluster/settings?pretty

$ curl -k -u elastic:orange -XGET https://172.21.173.198:9200/_nodes?pretty | grep quicksearch-
```

##### Activer le mode maintenance
```sbtshell
$ curl -H "Content-Type: application/json" -XPUT https://172.21.173.198:9200/_cluster/settings -d '
{
    "persistent": {
        "cluster.routing.allocation.enable": "none"
    }
}'
```

##### Alias d'index

Une autre stratégie est, au lieu de ré indexer, de garder les anciens indexs et de créer de nouveau avec de nouveaux settings et d'utiliser ce qu'on appelle des alias pour pointer sur tous ces indexs.<br/>
On peut imaginer des indexs par plages horaires. Par exemple des logs par jours.

Du coup on peut utiliser l'alias pour attaquer tous les logs ou bien juste le nom de l'index du jour pour une recherche sur ce qui s'est passé aujourd'hui.<br/>
La alias nous permettent aussi de créer des alias sur des périodes spécifiques. Par semaine, par mois ...
```sbtshell
POST /_aliases
{
	“actions”: [
		{ “add”: { “alias”: “logs_current”, “index”: “logs_2017_06” }},
		{ “remove”: { “alias”: “logs_current”, “index”: “logs_2017_05” }},
		{ “add”: { “alias”: “logs_last_3_months”, “index”: “logs_2017_06” }},
		{ “remove”: { “alias”: “logs_last_3_months”, “index”: “logs_2017_03” }}
	]
}
```

##### Cycle de vie d'un index

* Hot : Activement mis à jour et lu.
* Warm : Lu mais plus de mise à jour.
* Cold : Pas de mise à jour et quelques lectures.
* Delete : Cet index peut être supprimé.

Exemple de gestion de cycle de vie qu'on peut associer à un index via le template index.
Quand un index atteint 50Gb et date d'un mois il passe à l'état delete ou il sera supprimé quand il aura 3 mois. 

Un autre exemple :
```sbtshell
$ curl -k -u elastic:orange -H "Content-Type: application/json" -XPUT "https://172.21.173.198:9200/_ilm/policy/ocm-transaction-ilm" -d '
{
"policy": {
  "phases": {
    "hot": {
      "actions": {
        "set_priority": {
          "priority": 50
        }
      }
    },
    "warm": {
      "min_age": "7d",
      "actions": {
        "allocate": {
          "require": {
            "data": "warm"
          }
        },
        "set_priority": {
          "priority": 25
        }
      }
    },
    "cold": {
      "min_age": "30d",
      "actions": {
        "set_priority": {
          "priority": 0
        },
        "freeze": {},
        "allocate": {
          "require": {
            "data": "cold"
          }
        }
      }
    }
  }
}

```

##### Mémoire

**Investir sur la RAM. Le CPU on s'en fout !**

Heap Memory : comme on l'a vu il vaut mieux laisser la moitié à l'OS pour les opérations d'entrée et sortie de Lucene.
Attention par défaut c'est 1 Go, mais ne jamais dépasser 32Go.
Donc plus on met de la RAM plus elasticsearch pourra cacher des objets en mémoire et sera rapide, par contre cela donne du boulot au garbage collector de la JVM qui, on le sait, est très gourmand et pas assez rapide !

### Sécurité

"Cela signifie que les adminstrateurs peuvent désormais chiffrer le trafic réseau, créer et gérer des utilisateurs, définir des rôles qui protègent les accès au niveau des index et des clusters et sécuriser totalement les espaces Kibana".

- SSL/TLS pour les communications chiffrées : Protégez vos données lorsqu'elles transitent dans le cluster et vers les clients.
- Fichier et domaine natif pour la création et la gestion des utilisateurs.
- Le filtrage IP empêche également les hôtes non approuvés de rejoindre votre cluster ou de communiquer avec lui.
- Contrôle d'accès basé sur les rôles pour contrôler l'accès des utilisateurs aux API et index du cluster. Permet également la location multiple pour Kibana avec la sécurité pour les espaces Kibana.

**Par contre :** 
L'accès à d'autres fonctionnalités de sécurité, telles que l'authentification unique, l'authentification Active Directory / LDAP, la sécurité au niveau du champ et du document, nécessite toujours un abonnement Gold ou Platinum.

##### X-pack

La sécurité dans Elasticsearch est gérée par X-Pack pour garantir l'authentification des différents serveurs ainsi que la confidentialité et l'integrité des données.
Il permet de crypter les données qui circulent entre les noeuds et vers les clients grâce au protocole TLS, de gérer les credentials des utilisateurs, de filtrer les IPs etc ...

X-Pack offre aussi des outils d'audit pour vérifier qui a accés au cluster et quels types de requêtes sont envoyées par utilisateur.

Pour la gestion des users dans Elasticsearch nous avons des users, des groups et des roles.
Des users peuvent être assignés à des groupes et chaque groupe posséde des priviléges qui régulent les accés aux données.
On peut affiner l'accés aux index, aux documents ou aux champs en fonction des utilisateurs.

X-Pack peut aussi s'adosser à des annuaires tels que LDAP ou Active Directory, ce que nous allons faire ici chez vous.

Voir la documentation Elasticsearch.

---

Générer un certificat pour le noeud master :
```sbtshell
$ sudo bin/elasticsearch-certutil cert
```
dans le fichier : /etc/elasticsearch/elastic-certificates.p12

```sbtshell
$ sudo chmod 660 /etc/elasticsearch/elastic-certificates.p12
```

Si le certificat du noeud est sécurisé par un mot de passe, ajouter le mot de passe dans le keystore Elasticsearch : 
```sbtshell
$ sudo bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
$ sudo bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

Modifier config/elasticsearch.yml :
```sbtshell
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: /etc/elasticsearch/elastic-certitificates.p12
xpack.security.transport.ssl.truststore.path: /etc/elasticsearch/elastic-certitificates.p12
```

Générer un mot de passe pour les composants internes :
```sbtshell
$ sudo bin/elasticsearch-setup-passwords auto

Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y

Changed password for user apm_system
PASSWORD apm_system = 6ytAwO7RxXzdBsawNzlZ

Changed password for user kibana
PASSWORD kibana = EuSrh4RNk2wZPiI1LBDq

Changed password for user logstash_system
PASSWORD logstash_system = mCALEPEQtZmokUQmJEuT

Changed password for user beats_system
PASSWORD beats_system = kaBsVQbyII8F583XTY3E

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = fT7UmfeCyXPMteydN3T8

Changed password for user elastic
PASSWORD elastic = Cn3GaleaA2ai9yPoZl1t
```

Mettre à jour le mot de passe Kibana : 
```sbtshell
$ sudo nano /etc/kibana/kibana.yml
```

Ouverture des ports :
```sbtshell
$ firewall-cmd --permanent --add-port=9200/tcp
$ firewall-cmd --permanent --add-port=9300/tcp
$ firewall-cmd --reload
```

```
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: /etc/elasticsearch/certs/ocm-master.p12              
xpack.security.http.ssl.truststore.path: /etc/elasticsearch/certs/ocm-master.p12
```

Si le certificat du noeud est sécurisé avec un mot de passe, ajouter le mot de passe dans le keystore elasticsearch:
```sbtshell
$ sudo bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
$ sudo bin/elasticsearch-keystore add xpack.security.http.ssl.truststore.secure_password
```

Logstash
==

![alt text](https://i.ibb.co/B3JK2tJ/logstash-pipeline.png)

##### Présentation

Logstash sert à nourrir Elasticsearch de données. S'interface entre les données et où l'on veut les stocker.<br/>
Pour cela il peut s'appuyer sur FileBeat qui est un framework que l'on installe en bout de chaines là ou la donnée doit être collectée. FileBeat est très léger et peut donc cohabiter avec les serveurs opérationnels et monitorer les fichiers de logs par exemple ou sur de l'iot pour envoyer des mesures à Logstash qui lui va peut être les transformer, et les transmettre à Elasticsearch afin qu'ils soient indexées.<br/>
Contrairement à ce que beaucoup pensent, Logstash n'est pas dédié à Elasticsearch, il peut être utilisé indépendamment de la stack Elastic et se plugger à énormément d'autres types de sources en sortie et en entrée, allant de systèmes de fichiers aux bases de données comme mySql en passant pas des messages broker tels que Kafka, les serveurs de mails, etc...

Logstash parse, transforme et filtre les données. Il peut changer les structures. Enrichir les données par des infos de géolocalisation. Logstash peut être scalable sur plusieurs noeuds.


> => Scalable + Fault tolerance.<br/>
> => Gére les montées de flux. Il fait le policier de circulation.

#### Installation Logstash

```sbtshell
$ sudo yum install logstash
```

##### Parser et transférer les lignes d'un fichier de logs
```sbtshell
$ /usr/share/logstash

$ sudo ./bin/logstash -f /etc/logstash/conf.d/logstash.conf
$ sudo ./bin/logstash -f /etc/logstash/conf.d/logstash.conf --path.settings /etc/logstash
```

```yaml
    input {
        file{
            path=>"/tmp/access_log"
            start_position => "beginning"
            ignore_older=>0
            sincedb_path => "/dev/null"
        }
    }
    filter{
        grok {
            match => {  "message" => "%{COMBINEDAPACHELOG}" }
        }
        date {
            match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
        }
    }
    output {
        elasticsearch {
            hosts => ["localhost:9200"]
        }
        stdout {
            codec => rubydebug
        }
    }
```

Télécharger un exemple de fichier de logs :
```sbtshell
$ wget media.sundog-soft.com/es/access_log
```

**IMPORTANT :** Il a fallu modifier le fichier pour qu'il soit traité ! Même touch n'a pas suffi.

##### Lire et transférer d'une BDD mySql :

```sbtshell
$ sudo yum install mysql-server
```
```sbtshell
$ sudo mysql -u root -p

mysql> create database movielens;
mysql> create table movielens.movies ( movieID INT PRIMARY KEY NOT NULL, title TEXT, releaseDate DATE );

> sudo ./bin/logstash -f /etc/logstash/conf.d/mysql.conf

|      50 | Star Wars (1977)                               | 1977-01-01  |
|      62 | Stargate (1994)                                | 1994-01-01  |
|     222 | Star Trek: First Contact (1996)                | 1996-11-22  |
|     227 | Star Trek VI: The Undiscovered Country (1991)  | 1991-01-01  |
|     228 | Star Trek: The Wrath of Khan (1982)            | 1982-01-01  |
|     229 | Star Trek III: The Search for Spock (1984)     | 1984-01-01  |
|     230 | Star Trek IV: The Voyage Home (1986)           | 1986-01-01  |
|     271 | Starship Troopers (1997)                       | 1997-01-01  |
|     380 | Star Trek: Generations (1994)                  | 1994-01-01  |
|     449 | Star Trek: The Motion Picture (1979)           | 1979-01-01  |
|     450 | Star Trek V: The Final Frontier (1989)         | 1989-01-01  |
|    1068 | Star Maker, The (Uomo delle stelle, L') (1995) | 1996-03-01  |
|    1265 | Star Maps (1997)                               | 1997-01-01  |
|    1293 | Star Kid (1997)                                | 1998-01-16  |
|    1464 | Stars Fell on Henrietta, The (1995)            | 1995-01-01  |
```

Kibana
==

##### Présentation
Une application Web qui se branche sur un custer elasticsearch.
Elle permet de faire tout ce qui est possible de faire via l'API REST c'est à dire des recherches, des aggrégations mais aussi de créer des diagrammes de visualisation et des dashboards. Elle permet aussi de monitorer le cluster.
On l'uitilse souvent pour de l'analyse de logs.
On peut également l'utiliser pour l'analyse de click stream, d'ou vient le traffic sur un site web, qui génére tel type d'erreurs, etc...

Ce qui fait que Elasticsearch n'est plus juste un outil de recherche, il est aussi un puissant couteau suisse pour faire de l'analytique.

#### Installation Kibana

```sbtshell
$ sudo yum install kibana
$ sudo vi /etc/kibana/kibana.yml
```
Changer server.host à 0.0.0.0 : A ne pas faire en environnement de production.

```sbtshell
$ sudo /bin/systemctl daemon-reload
$ sudo /bin/systemctl enable kibana.service
$ sudo /bin/systemctl start kibana.service
```

Kibana est accessible sur le port 5601.

![alt text](https://i.ibb.co/bQX7j0Q/c652d73656375726974792d322e6a7067.jpg "Page d'accueil de Kibana")

La console :

```sbtshell
GET _cat/indices?v&pretty

GET _search?pretty
{
  "query": {
    "match_phrase": {
      "title": "Star Wars"
    }
  }
}
```

Beats
==

FileBeat peut servir d'interface entre les données à récupérer et Logstash.
Plus léger que Logstash. Il peut communiquer directement avec Elasticsearch mais en passant par Logstash il peut parler à d'autres systèmes.

Un système très résilient car permet de gérer la charge du flux dans le pipeline en direction de Logstash.
Il offre la possibilité de scaler le cluster horizontalement.

Gestion du flux :<br/>
Filebeat est en bout de chaine et tourne la ou sont générés les fichiers de logs.
Il peut arriver que Logstash n'arrive pas à ingérer la quantité de données qu'il reçoit en entrée.
Au cas où FileBeat lit les fichiers de logs plus vite que Logstash ne peut en ingérer, ils s'autorégulent entre eux pour adapter le rythme d'envoi des données vers Logstash.

Quand c'est un cluster Logstash, Beats fait du load balancing entre les différents noeuds pour répartir la charge d'ingestion.
Pour une meilleure disponibilité il est conseillé d'avoir au minimum deux noeuds Logstash.

L'utilisation des *persistent queues* est plus que recommandée pour la fault tolerance. En cas d'arrêt de logstash pour qu'il puisse retrouver les données.

```sbtshell
$ sudo yum install filebeat

$ cd /usr/share/elasticsearch/
$ sudo bin/elasticsearch-plugin install ingest-geoip (A partir de la version 6.7 ce n'est plus un blugin installable)
$ sudo bin/elasticsearch-plugin install ingest-user-agent (A partir de la version 6.7 ce n'est plus un blugin installable)
$ sudo /bin/systemctl stop elasticsearch.service
$ sudo /bin/systemctl start elasticsearch.service
```

Utiliser Filebeat pour intégrer des dashboards dans Kibana :
```sbtshell
$ cd /usr/share/filebeat/bin
$ sudo filebeat setup --dashboards

$ cd /etc/filebeat/modules.d/
$ sudo mv apache2.yml.disabled apache2.yml
```

```sbtshell
$ sudo vi /etc/filebeat/modules.d/apache2.yml
```
Modifier access et error log paths :
```sbtshell
["/home/<username>/logs/access*"]
["/home/<username>/logs/error*"]
```

Make /home/<username>/logs
```sbtshell
$ cd into it
$ wget http://media.sundog-soft.com/es/access_log
$ sudo /bin/systemctl start filebeat.service
```
