# Full Text Queries

### Matching flexible avec requête match :
Rechercher les documents de l'index recipe avec la phrase clée "Recipes with pasta or spaghetti" dans le titre.
```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "Recipes with pasta or spaghetti"
    }
  }
}
```

Le premier élément possède deux fois le mot spaghetti et une fois le mot pasta dans le titre.  
Ce qui lui confère le plus haut score de pertinence.  
Pour ce type de requête l'opérateur par défaut est le OR, raison pour laquelle il y a des recettes avec juste un des mots dans le titre.

### Spécifier un opérateur booléen
Pour spécifier de manière explicite l'opérateur logique du match, utiliser le mot clè operator.

L'effet de l'opérateur `and` est que tous les mots de la phrase clée doivent être dans le titre.  
Comme tous les mots de la requête doivent figurer dans le titre, aucun document ne correspond à la recherche :
```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Recipes with pasta or spaghetti",
        "operator": "and"
      }
    }
  }
}
```

Supprimer tous les mots qui nuisent à la pertinence de la recherche (or, with, recipes) pour avoir des résultats :
```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta spaghetti",
        "operator": "and"
      }
    }
  }
}
```

### Match phrase :
Pour cette requête l'ordre des termes compte :  
Faire une recherche avec la phrase séquence de termes "spaghetti puttanesca" dans le titre n'est pas la même chose que dans l'ordre inverse.  
**=>** Donc les deux termes doivent apparaitre dans le titre et dans l'ordre spécifié dans la requête.
```
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "spaghetti puttanesca"
    }
  }
}
```

### Recherche dans différents champs :
Pour cela il faut utiliser une requête multi_match :
```
GET /recipe/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": [ "title", "description" ]
    }
  }
}
```

Le score de pertinence sera celui du champ qui contient le plus d'éléments recherchés.
