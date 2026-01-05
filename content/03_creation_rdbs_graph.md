---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.12
    jupytext_version: 1.9.1
kernelspec:
  display_name: base
  language: ''
  name: python3
---

# Du relationnel au graphe

Avant toute implémentation en Neo4j, il faut réfléchir à la façon dont vous souhaitez organiser les données en graphe 
et tenir compte de l'utilisation prévue. 
L'outil [arrows.app](https://arrows.app/#/local/) peut être utilisé pour représenter graphiquement les différents éléments de votre graphe.

## Équivalence conceptuelle entre relationnel et graphe

Pour la migration de données d'une base de données relationnelle vers une base de données orientées graphe, voici quelques équivalences utiles :
* **Un nom de table devient un label** : Une table correspond généralement à un label dans une base de données orientée graphe.
* **Une ligne devient un noeud** : Une ligne dans une table correspond généralement à un noeud dans une base de données orientée graphe.
* **Un nom de colonne devient une propriété** : Les colonnes des tables correspondent généralement aux propriétés 
des noeuds et des relations dans une base de données orientée graphe. Parfois, les valeurs stockées dans plusieurs colonnes peuvent correspondre 
à une seule propriété associée à une liste de valeurs dans une base de données orientées graphe. 
Par exemple, les informations stockées dans trois colonnes `email1`, `email2`, `email3` dans une base de données relationnelle 
pourraient être stockées, dans une base de données orientée graphe, dans une seule propriété `email` associée à une liste contenant tous les mails renseignés.
* **Une paire clé primaire / clé étrangère devient une relation** : Une paire clé primaire / clé étrangère correspond généralement à une relation 
dans une base de données orientée graphe. Si elles n'avaient pas d'autre utilité que la jointure, les colonnes associées peuvent ne pas être conservées 
sous forme de propriétés une fois les relations créées.
* **Une table d'association devient une relation** : Une table d'association correspond généralement à une ou plusieurs relations, 
avec éventuellement des propriétés.
* **Une clé primaire devient une propriété avec contrainte d'unicité** : Une clé primaire métier, i.e qui a une signification métier 
(ex : référence produit, identifiant unique de connexion, numéro de sécurité social, adresse e-mail etc.), correspond généralement à une propriété 
dans une base de données orientée graphe, à laquelle est appliquée une contrainte d'unicité, voire d'existence.   

**Attention !** Ces équivalences doivent uniquement servir de point de départ. 
Si cette première mise en correspondance des concepts permet de construire un modèle de graphe fonctionnel, elle est rarement suffisante. 
Les véritables bénéfices d'une approche par graphe apparaissent lorsque le modèle est façonné **en fonction des usages attendus**.

## Exercice n°1

Proposez un modèle de graphe pour la base de données relationnelle ci-dessous.

```{figure} ../image/northwind.jpg
```

## Concevoir le graphe en fonction des cas d'usage

Vous devez identifier les cas d'utilisation à venir pour garantir de **bonnes performances à l'usage** : 
Quelles sont les données à inclure dans le graphe ? Quelles seront les **requêtes les plus fréquentes** ? Lesquelles sont à prioriser ?
Il n'y a pas de solution universelle, la modélisation implique nécessairement des compromis. 
L'objectif est de **limiter la portion du graphe parcourue lors de l'exécution d'une requête**.

* **Formuler les requêtes sous forme de question** peut aider à construire le graphe : 
  * Les noms dans les questions correspondent généralement à des labels (ex : `ACTOR` et `MOVIE` pour "Quels acteurs ont joué dans tel film ?")
  * Les verbes correspondent généralement à des relations (ex : `ACTED_IN` pour "Quels acteurs ont joué dans tel film ?")

* **L'utilisation de labels permet de réduire le nombre de noeuds traversés**. Spécifier un label permet en effet de limiter le nombre de noeuds à partir desquels lancer une **requête**. 
Toutefois, ne créez de labels que s'ils facilitent la plupart des cas d'utilisation : une bonne pratique consiste à **ne pas dépasser quatre labels par noeud**.  
````{admonition} Example
:class: tip

Prenons une base de données contenant des films -- associés au label `Movie` -- les acteurs et réalisateurs de ces films, ainsi que les utilisateurs les ayant notés -- tous associés à un label `Person` mais identifiables 
via le type des relations sortantes, respectivement `ACTED_IN`, `DIRECTED` et `RATED`. 
Accéder aux acteurs nécessiterait de récupérer, dans un premier temps, tous les noeuds de label `Person`, puis de filtrer en fonction de l'existence ou non de relations sortantes de type `ACTED_IN`. 
Récupérer l'ensemble des noeuds `Person` serait sous-optimal pour un graphe de très grande taille. 
Une façon d'améliorer la récupération des noeuds correspondant à des acteurs pourrait être de modifier le graphe afin d'inclure un label `Actor`.
````

* Le fan-out (déploiement), qui consiste à **convertir les valeurs répétées de propriétés en noeuds**, peut améliorer les performances des **requêtes** en **réduisant le nombre de noeuds à traverser**. 
Il permet aussi de **limiter la quantité de stockage nécessaire** : la dénormalisation, étroitement liée à la notion d'agrégat, est utile pour améliorer la rapidité de certaines requêtes, mais dupliquer les données 
signifie aussi stocker davantage, et il est parfois intéressant de transformer les valeurs concernées en noeuds uniques. Le principal risque du fan-out est qu’il peut conduire à des nœuds très denses, appelés super-nœuds. 
Il s’agit de nœuds possédant des centaines de milliers de relations entrantes ou sortantes. Les super-nœuds doivent être manipulés avec précaution.   
````{admonition} Example
:class: tip

Dans une base de données contenant des noeuds `Restaurant`, représenter les villes sous forme de noeuds distincts plutôt que de propriétés des noeuds `Restaurant` permet 
de répondre plus efficacement à la question : "Quels restaurants se situent dans tel quartier ?". En effet, plutôt que de parcourir tous les noeuds de label `Restaurant` et d'évaluer pour chacun d'eux 
la propriété `city`, créer des noeuds `City` limitera le nombre de noeuds à examiner puisqu'il suffira à Neo4j d'identifier le noeud `City` avec la propriété `name` adéquate, puis de renvoyer 
les restaurants reliés à ce noeud là.
````
 
* Créer des **relations hautement spécialisées** peut améliorer l'efficacité des requêtes et rendre l'utilisation du graphe plus intuitive. 
Neo4j, en tant que base de données orientée graphe, est en effet **conçu pour parcourir les relations très rapidement**. 
Dans certains cas, il sera ainsi plus efficace d'interroger le graphe en s'appuyant sur les types des relations plutôt que sur les propriétés des noeuds. 
À la création de relations spécialisées, il est recommandé de **ne pas supprimer les relations génériques d'origine**, afin de ne pas impacter les requêtes existantes qui en dépendent.   
````{admonition} Example
:class: tip

Prenons une base de données contenant des films (noeuds `MOVIE`) et des utilisateurs (noeuds `USER`) ayant noté ces films (relations `RATED` avec propriété `rating`). Imaginons que la requête "Combien d'utilisateurs ont 
donné une note inférieure à 5 à tel film ?" soit très fréquente. En l'absence de relation spécialisée, la requête doit parcourir l'ensemble des relations `RATED` et évaluer la propriété `rating` associée. 
Si le graphe est grand, le temps de traitement sera long. Dans ce cas, créer une relation spécialisée `RATED_BELOW_5`, tout en conservant la relation `RATED` originale pour les autres cas d'usage, 
permettra d'accélérer l'exécution puisqu'il suffira à Neo4j de récupérer les noeuds `Movie` vers lesquels pointent une relation de type `RATED_BELOW_5`.
````

* Utilisez des **index** pour les propriétés les plus sollicitées ou celles prenant de nombreuses valeurs distinctes (ex : identifiants uniques, noms d'utilisateurs). 
Les index doivent toutefois être utilisés avec discernement pour les raisons suivantes : chaque index double quasiment l'**espace de stockage** occupé par les données indexées, 
et l'ajout d'index **ralentit les opérations d'écriture** car ils doivent être mis à jour à chaque requête d'écriture. Autrement dit, lorsque les performances en écriture 
sont critiques pour un cas d’usage donné, il peut être préférable de limiter les index pour les propriétés concernées.

**Attention !** **À chaque modification** apportée, n'oubliez pas de **modifier les requêtes concernées** par ces modifications.

Il est bon de noter que Neo4j réalise aussi un travail d'optimisation des requêtes en arrière plan. Lorsque vous soumettez une requête à Neo4j, 
un optimiseur de requête (ou planificateur) détermine quel est le moyen le plus efficace d'exécuter la requête compte tenu de l'état à l'instant *t* 
de la base de données, c'est-à-dire des index et des contraintes disponibles ainsi que de diverses statistiques gérées par la base de données. 
Le planificateur produit alors un plan logique, transformé ensuite en **plan d'exécution**, qui exécute effectivement la requête sur la 
base de données. 

Vous pouvez **tester l'efficacité des modifications réalisées** en faisant précéder la requête à évaluer du mot clé `PROFILE`. 
En plus du résultat de la requête, le **plan d'exécution**, comprenant notamment le **nombre de lignes parcourues** par Neo4j **à chaque étape**, 
sera ainsi renvoyé. Vous pourrez donc comparer la portion du graphe explorée **avant et après modification**. 
N'hésitez pas à retravailler autant de fois que nécessaire votre graphe.  

Vous trouverez de nombreux exemples et conseils dans la documentation Neo4j (par exemple cet [article](https://neo4j.com/docs/getting-started/data-modeling/modeling-designs/), 
qui présente le concept de noeuds intermédiaires).

## Exercice n°2

Voici quelques cas d'utilisation fréquents pour la base de données précédente :
* Publicité ciblée : **Quels clients ont commandé le même produit que tel client ?**
* Publicité ciblée : **Quels sont les produits les plus vendus dans telle ville ?**
* Alerte qualité de service : **Quelles commandes ont été livrées en retard ?**
* Service client : **Quels est l'historique de commande de tel client ?**
* Service RH : **Quels sont les managers de l'entreprise ?**

Comment modifieriez-vous le modèle de graphe conçu à l'exercice n°1 ?
