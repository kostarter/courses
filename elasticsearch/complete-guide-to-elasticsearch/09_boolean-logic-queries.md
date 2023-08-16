# Requêtes avec logique booléenne

* **must :**
Decrit toutes les clauses qui doivent matcher, comme le ET logique. Elle contribue au score.

* **should :**
Cela signifie qu'au moins une des clauses doit matcher, comme le OU logique. Si la clause matche elle augmente le score, sinon le document est retourné quand même car le critère est non obligatoire. Il devient obligatoire si l'on enlève les autres clauses obligatoires (must, filter, etc.).

* **filter :**
Les clauses filter sont exécutés dans le filter context, le scoring est donc ignoré et les clauses peuvent être gérées par le cache.

* **must_not :**
Les clauses sont exécutées dans le filter context, le scoring est donc ignoré et les clauses peuvent être gérées par le cache. Comme le scoring est ignoré, un score de 0 est assigné à tous les documents retournés.

##### Ajouter la clause must à une requête :
```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

##### Déplacer la clause range (n'apportant rien au calcul du score) dans le filter :
```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

##### Ajouter une clause must_not :
```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

##### Debugger les requêtes bool avec des critères de requêtes labélisées :
Ajouter le champ _name aux différentes clauses de recherche.
```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": {
              "query": "parmesan",
              "_name": "parmesan_must"
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": {
              "query": "tuna",
              "_name": "tuna_must_not"
            }
          }
        }
      ],
      "should": [
        {
          "match": {
            "ingredients.name": {
              "query": "parsley",
              "_name": "parsley_should"
            }
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15,
              "_name": "prep_time_filter"
            }
          }
        }
      ]
    }
  }
}
```

Dans le bas de chaque document retourné il y aura une section matched_queries où sont listées les critères auxquels le document répond positivement.

##### Comment les requêtes match fonctionnent :

Nous savons que les requêtes match utilisent un opérateur par défaut OR et peuvent être configurées avec un AND. C'est une manière simplifier d'être des requêtes bool.  
Les deux requêtes suivantes sont équivalentes :
```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "pasta carbonara"
    }
  }
}

GET /recipe/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "title": "pasta"
          }
        },
        {
          "term": {
            "title": "carbonara"
          }
        }
      ]
    }
  }
}
```

Avec l'opérateur AND dans la requête match, la clause équivalente serait must.
