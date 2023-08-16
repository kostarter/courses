# Elasticsearch

Moteur de recherche full-text et analytique.  
Il est très puissant pour faire de la recherche. Cela comprend l'auto-complétion, la correction de fautes de frappes, le surlignement des correspondances, la gestion des synonymes, l'ajustement de la pertinence (en boostant les scores en fonction des données), etc.  
Elasticsearch ne fait pas que de la recherche par texte, il permet aussi d'interroger des données structurées telles que des chiffres ou des données agrégées, et être utilisé comme une plateforme analytique.  
Par exemple, il peut être utilisé pour l'analyse de logs et de metriques systèmes (APM : Application Performance Management). Il peut également être utilisé pour faire de l'analyse prédictive avec du Machine Learning (combien d'appel de support recevra tel service ? Quel sera le traffic sur le site web ? etc.). Un autre cas d'utilisation est la détection d'anomalies et configuration d'alertes pour les événements inhabituels.

Dans Elasticsearch les données sont stockées sous forme de document, qui est juste une unité d'information. Cela correspond à une ligne dans une base de données relationnelle. Un document contient des champs qui correspondent aux colonnes d'une BDD relationnelle. Un document est un objet JSON structuré.

Elasticsearch est écrit en Java et est basé sur Apache Lucene. Il est facile d'utilisation et est scalable. Il est cependant complexe lorque l'on veut l'utiliser en exploitant tout son potentiel.  
Il est utilisé par Facebook, Adobe, Netflix, Github, Soundcloud, etc.

###  Kibana :
Une plateforme d'analyse et de visualisation, qui permet de visualiser facilement les données d'Elasticsearch et de les analyser pour en extraire du sens. C'est une sorte de tableau de bord pour créer des visualisations à l'aide de graphiques. Kibana est aussi une console de management où il est possible de configurer les règles de sécurité et de droits d'accés à Elasticsearch.  
Il est possible de créer des tableaux de bord pour administrateurs où visualiser des metriques pour suivre les performances d'un serveur en terme d'utilisation de la CPU, l'état du traffic sur un site web par exemple ou des tableaux de bords pour développeur afin de suivre le nombre d'erreurs dans une application et visualiser les messages d'erreurs.

Démo Kibana : https://demo.elastic.co/app/kibana#/dashboard/welcome_dashboard  
KPI : Key points of interest.

### Logstash :
![Logstash Pipeline](./img/logstash-pipeline.png)

Initialement prévu pour traiter les logs des applications et les envoyer dans Elasticsearch, d'où le nom. C'est toujours un cas d'utilisation populaire, mais Logstash est devenu un outil à usage plus général pour devenir un outil de traitement et de transmission de données.  
Les données reçues par Logstash sont traitées en tant qu'événements : ça peut être des logs, des messages de chat, des commandes de commerce électronique, etc. Ces événements sont traités par Logstash et expédiés vers une ou plusieurs destinations (Elasticsearch, Kafka, Mail, etc.).  
Une pipeline Logstash se compose de trois parties : Inputs, Filters et Outputs. Chaque étape peut utiliser un plugin.
  
Les filters permettent aussi l'enrichissement des données, comme l'ajout une géolocalisation à partir d'une adresse IP.  

**Exemple :** Vidéo 1.3 pour le traitement de logs et extraction des données avec Grok.

###  X-pack :
Rajoute des fonctionnalités à Elasticsearch et Kibana.  
D'abord, la sécurité. X-pack met en place à la fois l'authentification et l'autorisation dans Kibana et Elasticsearch. Pour la partie authentification Kibana peut s'appuyer sur LDAP, Active Directory, etc. Il permet également de rajouter des utilisateurs et des roles pour configurer finement ce à quoi a accés tel ou tel utilisateur en fonction de son rôle.  
Ensuite, X-pack permet de survéiller les performances de la Stack Elastic (CPU, usage de la mémoire, l'espace disque, etc.) afin de détecter facilement tout problème. Il est également possible de configurer des alertes si quelque chose d'inhabituel se produit (par exmple si le CPU dépasse les 90% ou si un utilisateur se connecte 3 fois de pays différents en une heure). Avec le Reporting il est possible d'exporter de Kibana les tableaux de bord sous formats PDF ou CSV. Il est possible de programmer la génération de ces rapports et leur réception par mail (par exemple des rapports quotidiens des indicateurs de performance).  
Elasticsearch SQL offre la possibilité d'ecrire des requêtes en SQL pour consulter les index. Elasticsearch traduit ces requêtes SQL en Query DSL. Il y a aussi une translate API où il est possible d'envoyer une requête SQL et obtenir la traduction en Query DSL.  
X-pack est ce qui permet de faire du Machine Learning dans Kibana. Les cas d'utilisations les plus communs sont la détection d'anomalies, ou l'analyse prédictive.  
Une autre fonctionnalité est celle des graphes pour la création de liens entre comportements. Par exemple, cela peut aider à faire faire des prédictions pour de la recommandation de produits.  

###  Beats :
Une collection d'expéditeurs de données. Ce sont des agents légers pouvant être installés sur les serveurs, et envoient ensuite des données à Logstash ou Elasticsearch. Un des beats les plus communs est Filebeat qui est utilisé pour collecter des fichiers de logs.   
Filebeat est livré avec des modules pour des fichiers de logs communs comme ceux de nginx, serveur web apache ou mySql. Un autre beat est Metricbeat qui collecte des mesures niveau système ou service. Lui aussi est livré avec des modules dédiés à nginx, mySql, etc.  


## Installation :

Elasticsearch embarque un JDK, donc il n'est pas nécessaire d'avoir Java d'installée.

C'est une bonne pratique que de nommer le cluster et les noeuds (dans le fichier de config elasticsearch.yml).
Il est également recommandé de configurer les dossiers de logs et de données en dehors du dossier elasticsearch. Cela peut être pratique en cas d'upgrade de la version d'Elasticsearch.

Le fichier jvm.options permet de configurer la JVM. Ne rien toucher mis à part le heap memory de la JVM.

Elasticsearch utilise le framework de logging log4j2 pour les logs.

Il reste les fichiers de configuration des users et des roles. Il est fortement déconséillé d'y toucher, mais plutot d'utiliser Kibana pour la gestion des utilisateurs.

Il y a également le dossier pour les plugins (vide) et celui pour les modules (comme X-pack). La différence est que les modules sont embarqués dans Elasticsearch, et les plugins sont utilisés pour rajouter des fonctionnalitées customisées à Elasticsearch, ils peuvent être développés par un tiers ou par soi-même. Une autre différence est que les plugins peuvent être supprimés, ce qui n'est pas le cas des modules.
