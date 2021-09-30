## Shakespeare

Dire à elasticsearch comment stocker les données dans l'index Shakespeare :
```
$ wget http://media.sundog-soft.com/es7/shakes-mapping.json
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json
```

Importer des documents dans un index a partir d'un fichier JSON :
```
$ wget http://media.sundog-soft.com/es7/shakespeare_7.0.json
$ curl -H 'Content-Type: application/json' -XPOST 'localhost:9200/shakespeare/_doc/_bulk?pretty' --data-binary @shakespeare_7.0.json

$ curl -H 'Content-Type: application/json' -XGET localhost:9200/_cat/indices
```

Pour faire une recherche :
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '
{
 "query" : {
  "match_phrase" : {
   "text_entry" : "to be or not to be"
  }
 }
}'
```
```
$ curl -H "Content-Type: application/json" -XDELETE localhost:9200/movies
```

## Films 

```
$ wget https://files.grouplens.org/datasets/movielens/ml-25m.zip
$ wc -l ratings.csv
```
```
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies?pretty -d '
{
 "mappings" : {
   "properties" : {
    "year" : { "type": "date" }
   }
 }
}'
```

Changer le mapping :
```
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
 "mappings" : {
  "properties" : {
  "id" : { "type": "integer" },
  	"year" : { "type": "date" },
  	"genre" : { "type": "keyword" },
  	"title" : { "type": "text",  "analyzer": "english" }
  }
 }
}'

curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_mappings?pretty
```

##### Inserer une entrée dans l'index et la retrouver :
```
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/_doc/109487 -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar",
  "year": 2014
}'
```

Si on le fait deux il incrémente le numéro de version.

On veut tout le contenu de l'index :
```
$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty
```

Importer d'autres films :
```
$ wget http://media.sundog-soft.com/es7/movies.json
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @movies.json
```

##### Update :

Les documents sont immutables dans Elasticsearch.
A chaque mise à jour, Elasticsearch crée une nouvelle version du document et marque l'ancienne version comme 'à supprimer'. Elle sera supprimée par Elasticsearch plus tard.

Marche avec _update ou avec un simple create sur le même id.
```
$ curl -H "Content-Type: application/json" -XPOST 127.0.0.1:9200/movies/_update/109487?pretty -d '
{
  "doc" : {
    "title": "Interstellat Woo"
    }
}'
```
```
$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_doc/109487?pretty
```

##### Suppression :
```
$ curl -H "Content-Type: application/json" -XDELETE 127.0.0.1:9200/movies/_doc/58559?pretty
```

### Les analyseurs :

![alt text](https://i.ibb.co/zmQtMSk/01-Screenshot-from-2021-03-18-11-00-54.png "Learning Elasticsearch")

La documentation des analyzers embarqués dans Elasticsearch :<br/>
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html

- Keyword mapping pour supprimer l'analyseur sur le champ : seul une correspondance exacte est admise.
- Le type text pour permettre l'analyse.
En fonction de l'analyseur les résultats seront sensibles à la casse ou pas, filtrés, associés à des synonymes...
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
 "query" : {
  "match" : {
   "title" : "Star Trek"
  }
 }
}'
```
Avec des scores différents !

```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
 "query" : {
  "match_phrase" : {
   "genre" : "sci"
  }
 }
}'
```
Marche pas car ce sont des keywords !

##### Recherche

- Query lite : Plutot illisible.
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?q=+title:star&pretty'
```
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?q=+year:>2010+title:trek&pretty'
```
Attention : Les caractères spéciaux doivent être encodés !

https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-uri-request.html

* Filtres : Demande une réponse oui/non aux données.
* Requêtes : Retourne les données en se basant sur la pertinence.

Utiliser les filtres autant que possible car : plus rapides et gérés par le cache !
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
 "query" : {
  "match" : {
   "title" : "star"
  }
 }
}'
```

* Boolean query : 
Pour combiner des critéres et pour faire un AND elle doit être de type "must" :
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
 "query" : {
  "bool" : {
   "must" : { "term" : { "title" : "trek" }},
   "filter" : { "range" : { "year" : { "gte" : 2010 }}}
  }
 }
}'
```

* Recherche de phrase :
Dans les index inversés l'ordre des mots est également stocké. Match_phrase permet de faire une recherche sur un bout de phrase telle qu'elle est dans la requête.
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
 "query" : {
  "match_phrase" : {
   "title" : "star wars"
  }
 }
}'
```

* Le slop : 
Dans cet ordre mais pas nécessairement l'un après l'autre. Il contient le nombre de mots maximum d'écart entre les deux mots de la phrase.
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
 "query" : {
  "match_phrase" : {
   "title" : { "query" : "star beyond", "slop" : 1 }
  }
 }
}'
```

##### Tri

Utiliser le paramètre sort. Exemple : sort=year
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?sort=year&pretty'
```

Un champs text analysé en mode full-text ne peut être utilisé pour trier des documents.
Ceci parce qu'il existe dans l'index inversé sous forme de mots individuels et non plus comme une phrase complète.
Pour contourner cela on peut dupliquer le champs sous format keyword, qui ne sera pas analysé et sera stocker en tant que phrase, pouvant ainsi être utilisé pour le tri.
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/movie/_search?sort=title&pretty'
```
```
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
 "mappings" : {
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
}'
```
```
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?sort=title.raw&pretty'
```
```
$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
  "query":{
    "bool": {
      "must": {
        "match": {"genre": "Sci-Fi"}
      },
      "must_not": {
        "match": {"title": "trek"}
      },
      "filter": {
        "range": {"year": {"gte": 2010, "lt": 2015}}
      }
    }
  }
}'
```

Requêtes Fuzzy :<br/>
Utilise la distance Levenshtein pour rendre la recherche plus tolérante aux erreurs.
Cette distance prend en compte les substitutions, insertions et suppressions de caractères.
```
$ curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
 "query": {
  "fuzzy": {
   "title": {"value": "intrsteller", "fuzziness": 2}
  }
 }
}'
```
