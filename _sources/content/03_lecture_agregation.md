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

(content:lecture:agregation)=
# Requête d'agrégation

Un **pipeline d'agrégation** consiste en une succession d'opérations à appliquer aux données. On parlera d'étapes d'agrégation. La syntaxe est la suivante : 
```
db.<collection>.aggregate([<stage1>,<stage2>...])
```

````{admonition} Example
:class: tip

Dans la suite, nous utiliserons la base de données suivante conçue pour stocker des commandes :
```javascript
use Sales
```
```javascript
db.orders.insertMany([
  {
    "customer_id": "elise_smith@myemail.com",
    "orderdate": new Date('2020-05-30T08:35:52Z'),
    "value": 198,
    "address": {
                 "street": "12 rue des Acacias",
                 "zip_code": "69007",
                 "city": "Lyon",
                 "country": "France"
    },
    "products": [
       {"name": "reading_lamp", "tags": ["home","lighting","reading"], "price": Decimal128("59")},
       {"name": "leather_notebook", "tags": ["stationery","premium"], "price": Decimal128("35")},
       {"name": "fountain_pen", "tags": ["writing","premium","office"], "price": Decimal128("85")},
       {"name": "porcelain_mug", "tags": ["kitchen","gift"], "price": Decimal128("19")}
    ]
  },
  {
    "customer_id": "elise_smith@myemail.com",
    "orderdate": new Date('2020-01-13T09:32:07Z'),
    "value": 99,
    "address": {
                 "street": "12 rue des Acacias",
                 "zip_code": "69007",
                 "city": "Lyon",
                 "country": "France"
    },
    "products": [
       {"name": "wireless_headset","tags": ["electronics","audio"], "price": Decimal128("99")}
    ]
  },
  {
    "customer_id": "oranieri@warmmail.com",
    "orderdate": new Date('2020-01-01T08:25:37Z'),
    "value": 64,
    "address": {
                 "street": "3 place des Tilleuls",
                 "zip_code": "35000",
                 "city": "Rennes",
                 "country": "France"
               },
    "products": [
       {"name": "reading_lamp", "tags": ["home","lighting","reading"], "price": Decimal128("59")},
       {"name": "bookmark", "tags": ["reading","gift","stationery"], "price": Decimal128("5")}
    ]
  },
  {
    "customer_id": "tj@wheresmyemail.com",
    "orderdate": new Date('2019-05-28T19:13:32Z'),
    "value": 1,
    "address": {
                 "street": "24 Hauptstrasse",
                 "zip_code": "10115",
                 "city": "Berlin",
                 "country": "Germany"
               },
    "products": [
       {"name": "sticker", "tags": ["gift","stationery","fun"], "price": Decimal128("1")}
    ]
  },
  {
    "customer_id": "tj@wheresmyemail.com",
    "orderdate": new Date('2020-11-23T22:56:53Z'),
    "value": 154,
    "address": {
                 "street": "24 Hauptstrasse",
                 "zip_code": "10115",
                 "city": "Berlin",
                 "country": "Germany"
               },
    "products": [
      { "name": "ereader", "tags": ["electronics","reading"], "price": Decimal128("129")},
      { "name": "porcelain_mug", "tags": ["kitchen","gift"], "price": Decimal128("19")},
      { "name": "bookmark", "tags": ["reading","gift","stationery"], "price": Decimal128("6")} 
    ]
  },
  {
    "customer_id": "tj@wheresmyemail.com",
    "orderdate": new Date('2020-08-18T23:04:48Z'),
    "value": 4,
    "address": {
                 "street": "24 Hauptstrasse",
                 "zip_code": "10115",
                 "city": "Berlin",
                 "country": "Germany"
               },
    "products": [
       {"name": "sticker", "tags": ["gift","stationery","fun"], "price": Decimal128("4")}
    ]
  },
  {
    "customer_id": "elise_smith@myemail.com",
    "orderdate": new Date('2020-12-26T08:55:46Z'),
    "value": 4,
    "address": {
                 "street": "12 rue des Acacias",
                 "zip_code": "69007",
                 "city": "Lyon",
                 "country": "France"
    },
    "products": [
      {"name": "bookmark", "tags": ["reading","gift","stationery"], "price": Decimal128("4")}
    ]
  },
  {
    "customer_id": "tj@wheresmyemail.com",
    "orderdate": new Date('2021-02-28T07:49:32Z'),
    "value": 1024,
    "address": {
                 "street": "24 Hauptstrasse",
                 "zip_code": "10115",
                 "city": "Berlin",
                 "country": "Germany"
               },
    "products": [
      {"name": "laptop", "tags": ["electronics","office"], "price": Decimal128("950")},
      {"name": "screen", "tags": ["electronics","office"], "price": Decimal128("74")}
    ]
  },
  {
    "customer_id": "elise_smith@myemail.com",
    "orderdate": new Date('2020-10-03T13:49:44Z'),
    "value": 102,
    "address": {
                 "street": "12 rue des Acacias",
                 "zip_code": "69007",
                 "city": "Lyon",
                 "country": "France"
    },
    "products": [
      {"name": "screen", "tags": ["electronics","office"], "price": Decimal128("102")},
    ]
  }
])
```
```` 
Pour accéder au champ pré-existant ou nouvellement créé d'un document dans un pipeline d'agrégation, if faut précéder le nom du champ par `$`.   

Les principales étapes d'un pipeline d'agrégation sont les suivantes : 

## Sélection de documents

`$match` permet de **sélectionner des documents** en fonction de certains critères. La syntaxe est identique à celle utilisée pour le filtre de sélection de la commande `find()`. Il s'agit généralement de la première étape du pipeline d'agrégation.   

````{admonition} Example
:class: tip

Commençons par ajouter une étape `$match` au pipeline d'agrégation pour sélectionner les commandes passées en 2020 : 
```
db.orders.aggregate([
                      {
                        "$match": { 
                            "orderdate": 
                                {"$gte": new Date("2020-01-01"),    
                                 "$lt": new Date("2021-01-01")}
                            }
                      }
                    ])
```
Cette commande est équivalente à :
```
db.orders.find({
                 "orderdate":
                     {"$gte": new Date("2020-01-01"),
                      "$lt": new Date("2021-01-01")}
               })
```
````

## Formatage de documents

`$project` permet de **sélectionner**, d'**exclure**, de **renommer** ou d'**ajouter des champs**. La syntaxe est similaire à celle utilisée dans la commande `find()` pour la projection. Il s'agit généralement de la dernière étape du pipeline d'agrégation.
`$project` est équivalent à une combinaison de `$unset`, qui peut être utilisé pour **supprimer** certains **champs** du résultat, et `$set`, qui permet d'en **renommer** certains ou d'en **ajouter** de nouveaux.

````{admonition} Example
:class: tip

**Exemple n°1.** Vous pouvez utilisez `$project` pour afficher les commandes passées en 2020 sans les adresses de livraison :
```
db.orders.aggregate([
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      {
                        "$project": {"address" : 0}
                      }
                    ])
```
Vous pouvez également utilisez `$unset` :
```
db.orders.aggregate([
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      {"$unset": "address"}
                    ])
```
Notez qu'il est encore possible d'utiliser la commande `find()` :
```
db.orders.find({
                 "orderdate":
                     {"$gte": new Date("2020-01-01"),
                      "$lt": new Date("2021-01-01")}
               },
               {"address" : 0}
)
```
**Exemple n°2.** Pour afficher **uniquement** l'adresse mail et le pays des commandes passées en 2020 dans un nouveau champ `customer_country`, utilisez `$project` :
```
db.orders.aggregate([
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      {
                        "$project": { 
                            "customer_country" : ["$customer_id", "$address.country"]
                            }
                      }
                    ])
```
Nous aurions aussi pu utiliser `find()` :
```
db.orders.find({
                 "orderdate": {
                     "$gte": new Date("2020-01-01"),
                     "$lt": new Date("2021-01-01")
                 }
               },
               {"customer_country": ["$customer_id", "$address.country"]}
)
```
**Exemple n°3.** Pour afficher toutes les informations associées aux commandes passées en 2020 tout en **ajoutant** le champ `customer_country`, utilisez `$set` :
```
db.orders.aggregate([
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      {
                        "$set": 
                            {"customer_country": ["$customer_id", "$address.country"]}
                      }
                    ])
```
Cette commande est équivalente à la commande suivante utilisant `$project`, qui est bien plus lourde car il faut lister explicitement l'ensemble des champs à inclure :
```
db.orders.aggregate([
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      {
                        "$project": {
                            "customer_id": 1, 
                            "orderdate": 1, 
                            "value": 1, 
                            "address": 1, 
                            "customer_country": ["$customer_id", "$address.country"]
                            }
                      }
                    ])
```
Il est possible de construire une commande similaire avec `find()`.
````

## Agrégation de documents : `$group`

`$group` permet de **combiner plusieurs documents** en fonction de la valeur d'un ou plusieurs champs ou du résultat d'une expression, et selon une méthode d'agrégation donnée. La syntaxe est la suivante :   
```
{
  "$group" :
    {
      "_id" : <group_key>,
      <field> : { <accumulator> : <expression> },
      ...
    }
} 
```
où :
- `_id` est le champ, l'ensemble de champs ou l'expression qui permet de regrouper les documents. Passer `null` à `_id` agrège tous les documents.
- `<field>` est le nom du nouveau champ dans lequel le résultat de l'agrégation doit être stocké. Cet argument est optionnel. Lorsque `_id` est le seul argument spécifié, `$group` renvoie une liste de valeurs uniques. 
- `<accumulator>` est un opérateur d'accumulation, par exemple : `$sum`, `$max`, `$min`, `$count`, `$avg`, `$first`, `$push`.   

`$push` permet de renvoyer sous forme de liste toutes les valeurs associées à un ou plusieurs champs dans l'ensemble des documents. La syntaxe est la suivante :   
```
{"$push": <value>}
``` 
où `<value>` peut être une clé unique ou un document contenant plusieurs clés : `{<new_key1>: <old_key1>, ...}`.      

:::{caution} 
Un pipeline d'agrégation est une succession d'étapes. À chaque étape, seuls les éléments conservés à l'étape précédente sont accessibles.
En particulier, seuls les champs spécifiés dans `$group` seront accessibles à l'étape suivante du pipeline.  
:::

````{admonition} Example
:class: tip

**Exemple n°1.** Quelle est le plus gros achat réalisé pour chaque client en 2020 ?
```
db.orders.aggregate([
                      // 1. Commandes réalisées en 2020
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      // 2.1 Commandes groupées par adresse email
                      // 2.2 Récupération du montant le plus élevé pour chaque client
                      {
                        "$group": {
                            "_id": "$customer_id",
                            "max_purchase": {"$max": "$value"}
                            }
                      },
                      // 3.1 Sélection du champ créé à l'étape précédente "max_purchase"
                      // 3.2 Renommage du champ créé à l'étape précédent "_id" en "customer_id" :
                      // 3.2.1. Création d'un nouveau champ "customer_id"
                      // 3.2.2. Suppression du champ "_id"
                      {
                        "$project": {"max_purchase":1, "customer_id": "$_id", "_id":0}
                      },
                      // 4. Trier les résultats par ordre décroissant 
                      { 
                        "$sort": {"max_purchase":-1}
                      }
                    ])
```
**Exemple n°2.** On peut spécifier plusieurs opérations d'agrégation dans `$group`. On peut par exemple créer une synthèse des achats réalisés par chaque client en 2020 :
```
db.orders.aggregate([
                      // 1. Commandes réalisées en 2020
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      // 2. Trier les commandes par date
                      {"$sort": {"orderdate": 1}},
                      // 3.1 Commandes groupées par adresse email
                      // 3.2 Récupération du montant le plus élevé pour chaque client
                      // 3.3 Calcul du montant total dépensé par client
                      // 3.4 Calcul du nombre de commandes par client
                      // 3.5 Calcul de la moyenne des montants dépensés par client
                      // 3.6 Récupération de la première date d'achat
                      {
                        "$group": {
                            "_id": "$customer_id",
                            "max_purchase": {"$max": "$value"},
                            "total_value": {"$sum": "$value"},
                            "num_orders": {"$sum": 1},
                            "average_purchase": {"$avg": "$value"},
                            "first_purchase_date": {"$first": "$orderdate"} // fonctionne car tri en amont 
                            }
                      },
                      // 4. Création d'un nouveau champ "customer_id"  
                      {
                        "$set": {"customer_id": "$_id"}
                      },
                      // 5. Suppression du champ "_id"  
                      {
                        "$unset": "_id"
                      },
                    ])
``` 
**Exemple n°3.** On peut utiliser `$push` pour conserver l'historique des achats des clients dans la synthèse :  
```
db.orders.aggregate([
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      {"$sort": {"orderdate": 1}},
                      {
                        "$group": {
                            "_id": "$customer_id",
                            "max_purchase": {"$max": "$value"},
                            "total_value": {"$sum": "$value"},
                            "num_orders": {"$sum": 1},
                            "average_purchase": {"$avg": "$value"},
                            "first_purchase_date": {"$first": "$orderdate"},
                            // Historique des achats
                            "orders": {"$push": {"orderdate": "$orderdate", "value": "$value"}}
                            }
                      },
                      {
                        "$set": {"customer_id": "$_id"}
                      },
                      {
                        "$unset": "_id"
                      },
                    ])
```
**Exemple n°4.** Quel est le montant total des ventes en 2020 ?  
```
db.orders.aggregate([
                      // 1. Commandes réalisées en 2020
                      {
                        "$match": {
                            "orderdate":
                                {"$gte": new Date("2020-01-01"),
                                 "$lt": new Date("2021-01-01")}
                            }
                      },
                      // 2. Montant total des ventes
                      {
                        "$group": {
                            "_id": null,
                            "total_value": {"$sum": "$value"}
                            }
                      },
                      // 3. Suppression du champ "_id"
                      {
                        "$unset": "_id"
                      }
                    ])
```
````  

## Autres étapes d'agrégations et méthodes

`$unwind` décompose un champ de type liste en autant de documents qu'il y a d'éléments dans la liste. Tous les champs du document initial sont conservés, seule la valeur du champ de type liste est remplacée par l'un des éléments de la liste.
La syntaxe est la suivante :  
```
{"$unwind": {"path": <key>}}
```
Cette commande est utile lorsque l'agrégat ne correspond pas aux besoins.

````{admonition} Example
:class: tip

Dans la base de données, un agrégat correspond à une commande. Si nous souhaitons avoir des informations sur les produits, quelques manipulations supplémentaires sont nécessaires.   
  
**Exemple n°1.** Pour lister les catégories contenant des produits à plus de 50e et le nombre de produits à plus de 50e vendus pour chacune, on peut utiliser `$unwind` :
  
```
db.orders.aggregate([
                      // 1. Décomposer le champ `products`
                      {
                        "$unwind": {"path": "$products"}
                      },
                      // 2. Sélectionner les catégories contenant des produits à 50e ou plus
                      {
                        "$match": {"products.price": {"$gte": 50}}
                      },
                      // 3.1 Grouper les documents par catégorie de produit
                      // 3.2 Compter le nombre de produit à 50e ou plus vendus dans chaque catégorie
                      {
                        "$group": {
                            "_id": "$products.name",
                            "count": {"$sum": 1}
                            }
                      },
                      // 4. Renommer le champ créé à l'étape précédent "_id" en "product_name"
                      {
                        "$set": {"product_name": "$_id"}
                      },
                      {
                        "$unset": "_id"
                      } 
                    ])
```
**Exemple n°2.** Combien l'entreprise a-t-elle vendu de produits par tag ? Pour quelle somme totale ? Quels sont ces produits ?
```
db.orders.aggregate([
                      // 1. Décomposer le champ `products`
                      {
                        "$unwind": {"path": "$products"}
                      },
                      // 2. Décomposer le champ `products.tags`
                      {
                        "$unwind": {"path": "$products.tags"}
                      },
                      // 3.1 Grouper les documents par catégorie de produit et par tag
                      //     pour ne pas avoir de doublons dans la liste finale 
                      //     des catégories associées à chaque tag
                      // 3.2 Compter le nombre de produits vendus par groupe
                      // 3.3 Calculer le montant total par groupe
                      {
                        "$group": {
                           "_id": {"tags": "$products.tags", "products": "$products.name"},
                           "temporary_count": {"$sum": 1},
                           "temporary_sum": {"$sum": "$value"}
                           }
                      },
                      // 4.1 Grouper les documents par tag
                      // 4.2 Lister les catégories de produits par tag
                      // 4.3 Compter le nombre de produits vendus par tag
                      // 4.4 Calculer le montant total par tag
                      {
                        "$group": {
                           "_id": "$_id.tags",
                           "products": {"$push": "$_id.products"},
                           "count": {"$sum": "$temporary_count"},
                           "sum": {"$sum": "$temporary_sum"}
                           }
                      },
                      // 5. Renommer le champ créé à l'étape précédent "_id" en "tag"
                      {
                        "$set": {"tag": "$_id"}
                      },
                      {
                        "$unset": "_id"
                      }
                    ])
                        
```
````

Voir aussi les étapes d'agrégation : `$count`, `$sort`, `$limit`, `$lookup`.   

Le méthode `distinct()` permet d'obtenir les valeurs uniques pour un champ donné. La syntaxe est la suivante :
```
db.<collection>.distinct(<key>)
```

## Exercice

À partir du jeu de données `movies.json` :

1. Quels sont les différents genres représentés dans la base de données ? 
2. Combien y a-t-il de films par pays ?
3. Combien y a-t-il de films par pays et par année ?
4. Combien y a-t-il de films par pays et par année, triés par pays et par année (dans cet ordre) ?
5. Retourner l'ensemble des films réalisés par réalisateur entre 1975 et 1985.
```{admonition} Aide
:class: dropdown

Utilisez `$match`, `$group` et `$push`. Vous pouvez utiliser l'identifiant des réalisateurs pour le regroupement, mais dans ce cas, utilisez `$first` pour conserver l'information du prénom et du nom des réalisateurs.
```
6. Quel réalisateur a le plus de films à son actif ?
```{admonition} Aide
:class: dropdown

Utilisez `$group`, `$sort` et `$limit`.
```
7. Listez les noms complets de tous les acteurs dont le prénom est `Robert`. Lorsque vous renvoyez les résultats, vous pouvez utilisez `$concat` pour concaténer le prénom et le nom de famille si vous le souhaitez.
```{admonition} Aide     
:class: dropdown

Utilisez `$unwind`, `$match` et `$group`.
```
## En résumé

Dans ce chapitre vous avez appris que :
- Faire des **requêtes** qui **ne correspondent pas** aux cas d'usage en fonction desquels les **agrégats** ont été construits est **plus laborieux**. Cela nécessite des étapes supplémentaires (notamment l'utilisation de `$unwind` pour exploser les agrégats). 
  
Dans ce chapitre vous avez manipulé les fonctions suivantes :
  
|  Étape d'agrégation  |    Description    |
|:---:|:---|
|  `$match`  |  Sélectionner des documents en fonction de certains critère. Idem `<filter>` de `find()`  |
|  `$project`  |  Sélectionner, exclure, renommer ou ajouter des champs. Idem `<projection>` de `find()`  |
|  `$unset`  |  Supprimer des champs  |
|  `$set`  |  Renommer ou ajouter des champs  |
|  `$group`  |  Combiner plusieurs documents en fonction d'un ou plusieurs champs (ou d'une expression) et d'une méthode d'agrégation  | 
|  `$count`  |  Retourner le nombre de documents  |
|  `$sort`  |  Trier les documents en fonction d'un ou plusieurs champs, par ordre croissant ou décroissant  |
|  `$limit`  |  Limiter le nombre de documents retournés  |
|  `$lookup`  |  Extraire les données d'une autre collection à partir d'une clé locale et d'une clé étrangère  |

  
  
|  Opérateur d'accumulation  |    Description    |
|:---:|:---|
|  `$sum`  |  Somme des valeurs  |
|  `$max`  |  Valeur maximale  |
|  `$min`  |  Valeur minimale  |
|  `$avg`  |  Moyenne des valeurs (**av**era**g**e)  |
|  `$count`  |  Nombre de documents  |
|  `$first`  |  Première valeur  |
|  `$push`  |  Création d'une liste à partir de valeurs  | 
