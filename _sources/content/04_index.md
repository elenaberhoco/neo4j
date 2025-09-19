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

# Titre 3

INTÉRESSANT

## Requêtes et Index textuels

Lorsque l'on veut interroger notre base de données sur un champ de type "chaîne de caractères", deux méthodes s'offrent à nous : on peut utiliser soit des requêtes régulières, soit un index textuel qui a été créé sur-le-champ. L'avantage de la première méthode est une très grande précision, et on l'utilisera donc lorsque l'on recherchera du texte très précis, tandis que la seconde méthode utilise la puissance de l'index pour effectuer une recherche de type "moteur de recherche", renvoyant des résultats proches de ce qui a été demandé. Ici, comme on s'intéresse uniquement au index, nous ne développerons que la seconde méthode.


* Création d'un index textuel

Il existe deux manières de créer un index textuel, sur un attribut précis ou alors sur l'ensemble des attributs :

_Pour un attribut précis, ici "cle" :_

```javascript
db.coll.createIndex({"cle" : "text"})
```

_Pour tous les attributs :_

```javascript
db.nomDeLaCollection.createIndex({"$**" : "text"})
```

* Requêtes avancées utilisant un index textuel

Pour effectuer un requête de type "moteur de recherche", on utilise la forme suivante :

```javascript
db.nomDeLaCollection.find({$text : {$search : "ma requête"}})
```

On remarque plusieurs choses : tout d'abord, il n'est pas nécessaire de préciser le ou les champ sur lequel on veut effectuer la recherche. Ce type de requête n'était possible que sur les champs avec un index textuel, c'est sur ces derniers que le langage va requêter (c'est le sens du `"$text"`). Ensuite, on remarque la présence de `"$search"`, nécessaire pour ce type de requête.

Par défaut, lorsque l’on effectue une requête contenant plusieurs termes, un OU logique est effectué : les résultats retournés sont ceux qui contiennent au moins l’un des termes. On peut également effectuer une requête impliquant une expression exacte, qui sera encadrée de guillemets échappés par un caractère "\" :

```javascript
db.nomDeLaCollection.find({$text : {$search : "\"ma requête\""}})
```

De plus, il est possible d'exclure des termes des résultats en utilisant "-", dans ce cas, un ET logique est effectué. Par exemple, dans la requête suivante, on souhaite les documents contenant "ma requête" et ne contenant pas "exemple" en même temps :

```javascript
db.nomDeLaCollection.find({$text : {$search : "ma requête -exemple"}})
```

Enfin, on peut classer les documents par pertinence par rapport à notre requête, en utilisant le score td-idf (plus d'informations disponibles [ici](https://fr.wikipedia.org/wiki/TF-IDF) :

```javascript
db.nomDeLaCollection.find({$text: {$search: "ma requête"}},{"score": {$meta: "textScore"}}).sort({score: {$meta: "textScore"}})
```

Cette requête renvoie une liste des documents ordonnée par pertinence, si vous souhaitez juste afficher le score de chaque document, il suffit d'enlever `.sort({score: {$meta: "textScore"}})`.

_Exemple 1 : Liste des documents comportant le terme "famille" mais pas le terme "politique"._

```use discours```

```{code-cell}
:tags: [output_scroll]

db.discours.find({$text : {$search : "famille -politique"}})
```

_Exemple 2 : Liste ordonnée par pertinence des documents par rapport au terme "écologie"_ :

```{code-cell}
:tags: [output_scroll]

db.discours.find({$text: {$search: "écologie"}},{"name": true, "score": {$meta: "textScore"}}).sort({score: {$meta: "textScore"}})
```
