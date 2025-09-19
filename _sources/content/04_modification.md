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

# CR**U**D : Modification

::::{grid}
:gutter: 1

:::{grid-item-card} Objectif
Dans ce chapitre, vous apprendrez à modifier des documents, à remplacer des documents et à définir des index.
:::

::::

````{admonition} Example
:class: tip

Dans la suite, nous utiliserons une base de données conçue pour stocker des paniers d'achat :
```javascript
use Orders
db.carts.insertMany([
  {
    _id: 1,
    email: "alice@example.com",
    orderDate: ISODate("2025-03-01"),
    status: "pending",
    items: [
      {"productID":"P16-110", product: "Mouse", price: 5.0}
    ],
    total: 5.0,
    loyaltyPoints: 100
  }
])

```
````

## Modification

Pour **modifier partiellement** *tous* les documents qui vérifient un ensemble de conditions, utilisez `db.<collection>.updateMany()`. La syntaxe est la suivante :
```
db.<collection>.updateMany(<filter>, <update>, <option>)
```
où :   
- `<update>` peut être :  
  - un **document** contenant **un ou plusieurs opérateurs de modification** 
  - un **pipeline d'agrégation** contenant les étapes d'agrégation `$set`, `$unset` ou `$replaceWith`   
- `<option>` est un document acceptant six paramètres dont `upsert` et `arrayFilters`  

Pour ne modifier qu'*un seul* document vérifiant un ensemble de conditions, utilisez `db.<collection>.updateOne()`.

### Modification via un document

Tous les opérateurs de modification disponibles sont listés [ici](https://www.mongodb.com/docs/manual/reference/mql/update/).
On distingue notamment les **opérateurs conçus pour des listes** (Section [Arrays](https://www.mongodb.com/docs/manual/reference/operator/update-array/)) des **opérateurs conçus pour des valeurs simples** (Section [Fields](https://www.mongodb.com/docs/manual/reference/operator/update-array/)).   
Ci-dessous, un tableau avec quelques uns de ces opérateurs, où `<expression> = <number_or_operator_expression>` :  

Opérateur | Type | Syntaxe | Description
--- | --- | --- | ---
`$set` | simple | `{"$set": {<key>:<value>,...}}`  | Modifier la valeur d'un champ ou ajouter un champ  
`$unset` | simple | `{"$unset": {<key>:"",...}}`  | Supprimer un champ  
`$rename` | simple | `{"$rename": {<old_key>:<new_key>,...}}`  | Renommer un champ  
`$inc` | simple | `{"$inc": {<key>:<value>,...}}`  | **Inc**rémenter un champ d'une valeur définie (positive ou négative)    
`$mul` | simple | `{"$mul": {<key>:<value>,...}}` | **Mul**tiplier la valeur d'un champ par une valeur définie  
`$pop` | liste | `{"$pop": {<key>:-1_or_1,...}}` | Supprimer le premier (-1) ou le dernier (1) élément d'une liste   
`$pull` | liste | `{"$pull": {<key>:<expression>,...}}` | Supprimer les éléments d'une liste qui correspondent à une valeur ou vérifient une condition
`$push` | liste | `{"$push": {<key>:<value>,...}}` | Ajouter une valeur à la fin de la liste  
`$addToSet` | liste | `{"$addToSet": {<key>:<value>,...}}` | Ajouter une valeur à la fin de la liste **sauf si** cette valeur est déjà dans la liste
`$each` | liste | `<value> = {"$each": [<value1>,...]}`| Dans **`$push`** et **`$addToSet`**, ajouter plusieurs valeurs à la liste
`$position` | liste | `<value> = {"$each": [<value1>,...], "$position": <num>}` | Dans **`$push`**, ajouter une valeur à une position donnée de la liste   
`$[]` | liste | `<key>.$[].<sub_key>` | Modifier la valeur d'un champ pour tous les sous-documents de la liste  

:::{caution}
Si certains de ces opérateurs de modification peuvent vous paraître familiers, notez qu'il y a parfois des **différences de syntaxe** avec leurs homologues des pipelines d'agrégation.
C'est le cas de `$unset` :   
La syntaxe de cet opérateur lorsqu'utilisé dans un **pipeline d'agrégation** est :
```
{ "$unset" : [<key>, ...] }
```  
Tandis que pour l'utiliser en tant qu'**opérateur de modification** il faudra taper :
```
{ "$unset" : {<key> : "", ...} }
```
:::

````{admonition} Example
:class: tip

**Exemple n°1.** L'adresse mail d'Alice est incorrecte, corrigons-là :
```
db.carts.updateOne({"_id": 1}, {"$set": {"email": "alice.bernard@example.com"}})
```
**Exemple n°2.** Alice ajoute un ordinateur à son panier : 
```
db.carts.updateOne({"_id": 1},
                   {"$push": 
                       {"items": 
                         {"productID":"P16-271", "product": "Laptop", price: 1200.0}
                       }
                   })
```
**Exemple n°3.** Ce produit permet à Alice d'obtenir 100 points de fidélité supplémentaires :
```
db.carts.updateOne({"_id": 1}, {"$inc": {loyaltyPoints: 100}})
```
**Exemple n°4.** L'ordinateur est vendu avec une souris, Alice retire donc toutes les souris de son panier d'achat :
```
db.carts.updateOne({"_id": 1}, {"$pull": {items: {product: "Mouse"}}})
```
````

### Modification via un pipeline
 
L'utilisation d'un **pipeline d'agrégation** permet des modifications plus complexes, qui utilisent par exemple les **valeurs de plusieurs champs**. 
Vous trouverez quelques exemples dans la [documentation](https://www.mongodb.com/docs/manual/tutorial/update-documents-with-aggregation-pipeline/#std-label-updateMany-let-example) de MongoDB (ignorez la section dédiée à Atlas).    

Seuls `$set`, `$unset` ou `$replaceWith` sont acceptés comme étapes du pipeline. Il est toutefois possible d'utiliser d'autre opérateurs au sein de chacune de ces étapes.
MongoDB fournit une panoplie d'opérateurs pour transformer les données. 
Outre ceux vus au chapitre précédent, la Section [Expressions](https://www.mongodb.com/docs/manual/reference/mql/expressions/) en liste un certain nombre.
Notez que les opérateurs de la Section Expressions peuvent **également** être utilisés lors des **requêtes de lecture**, au sein des étapes d'agrégation `$project`, `$set`, `$group` ou `$replaceWith` de la fonction `aggregate()` ({ref}`content:lecture:agregation`).   

Ci-dessous, quelques opérateurs utiles, où `<expression> = <number_or_key_or_operator_expression>` : 

Opérateur | Syntaxe | Description
--- | --- | --- |
`$abs` | `{"$abs" : <expression>}` | Valeur absolue  
`$add` | `{"$add" : [<expression>,<expression>, ...]}` | Addition de deux valeurs ou plus  
`$subtract` | `{"$subtract" : [<expression>,<expression>]}` | Soustraction de deux valeurs (uniquement)  
`$multiply` | `{"$multiply" : [<expression>,<expression>]}` |  Multiplication de deux valeurs (uniquement)  
`$divide` | `{"$divide" : [<expression>,<expression>]}` | Division de deux valeurs (uniquement)   
`$ceil` | `{"$ceil" : <expression>}` | Valeur arrondie à l'entier supérieur  
`$floor` | `{"$floor" : <expression>}` | Valeur arrondie à l'entier inférieur  
`$concatArrays` | `{"$concatArrays" : [<array_or_key>, ...]}` | Concaténation de plusieurs listes  
`$mergeOjects` | `{"$mergeObjects" : [<document_or_key>, ...]}` | Concaténation de plusieurs documents  
`$map` | `{ $map: {input: <key_or_expression>, as: <name>, in: <operator_expression>}}` | Application d'une fonction à chaque élément d'une liste   

````{admonition} Example
:class: tip

**Exemple n°1.** Suite à l'ajout de l'ordinateur et au retrait de la souris, le montant total doit être ajusté (`$sum` vu au chapitre précédent):
```
db.carts.updateOne({"_id": 1},
                   [{"$set": {"total": {"$sum": "$items.price"}}}]
                  ) 
```
Cette opération n'aurait pas été possible sans utiliser un pipeline d'agrégation.   
  
**Exemple n°2.** Une opération marketing est en cours : tous les clients cumulant 100 points de fidélité ou plus obtiennent 10% de réduction immédiate si le montant total dépasse 500e :
```
db.carts.updateMany({"loyaltyPoints":{"$gte":100}, "total":{"$gte":500}},
                   [{"$set": {"total": {"$multiply": ["$total", 0.9]}}}]
                  )
```
**Exemple n°3.** Alice ajoute à son panier un clavier et deux stickers pour customiser son futur ordinateur. 
Nous souhaitons créer un nouveau champ pour stocker l'écart de prix entre le produit le plus cher et le produit le moins cher 
(`$concatArrays` permet d'ajouter des éléments à une liste):
```
db.carts.updateOne({"_id": 1},
                   [
                    {"$set":
                        {"items":
                           {"$concatArrays": 
                              ["$items", [{"productID":"P16-448", "product": "Keyboard", price: 45.0},
                                          {"productID":"P05-612", "product": "Stickers", qty:2, price: 1.0}]]
                    }}},
                    {"$set": 
                       {"priceRange":
                          {"$abs":
                             {"$subtract":
                                [{"$min":"$items.price"}, {"$max":"$items.price"}]
                    }}}}
                   ]
                  )
```
Le pipeline d'agrégation permet d'enchaîner plusieurs opérations. Les opérations peuvent être imbriquées.    
  
**Exemple n°4.** Remarquez que le document imbriqué des stickers contient un champ supplémentaire : `qty`. 
Pour **standardiser la structure** des sous-documents qui contiennent les informations produit, nous pouvons utiliser `$mergeObjects`. 
Cet opérateur va nous permettre de créer le champ `qty` et de lui associer une valeur par défaut (1). 
L'opérateur sera utilisé au sein de l'étape `$set`. Comme `$items` est une liste, il faut itérer sur chacun des éléments de la liste à l'aide de `$map`.
```
db.carts.updateOne({"_id": 1},
                   [
                    {"$set": {"items" : 
                       {"$map": {
                          "input": "$items",
                          "as": "product", // variable représentant chaque élément de la liste
                          "in":   
                            {"$mergeObjects" : 
                               [{"productID":"", "product":"", "qty":1, "price":null}, "$$product"]
                            }
                    }}}}
                   ]
                  ) 
```
Si les documents à fusionner ont des champs en commun, seule la valeur du dernier document est retenue. 
Ici, le premier document sert donc à définir des valeurs par défaut pour les champs absents. Les valeurs sont écrasées lorsque le champ existe.   
   
**Exemple n°5.** Nous pouvons désormais recalculer le total et appliquer la promotion de 10%. 
Il va falloir de nouveau itérer sur les éléments de `$items` pour calculer le montant total par produit en multipliant prix et quantité. 
Il suffira alors de sommer ces montants, comme dans l'exemple n°1, et d'appliquer la réduction, comme dans l'exemple n°2.
```
db.carts.updateOne({"_id": 1},
                   [
                    {"$set": {"total": 
                       {"$sum":
                          {"$map": {
                          "input": "$items",
                          "as": "product", 
                          "in": {"$multiply": ["$$product.price","$$product.qty"]}
                    }}}}},
                    {"$set": {"total": {"$multiply": ["$total", 0.9]}}}
                   ]
                  )
```
````

### Modification avec options

Parmi les options possibles listées dans la [documentation](https://www.mongodb.com/docs/manual/reference/method/db.collection.updateMany/) :   
- `upsert: true` permet de **créer un nouveau document** en fonction des paramètres spécifiés dans `<update>` **si aucun document** de la base de données **ne valide les conditions** définie par `<filter>`.        
- `arrayFilters` permet de ne modifier que les éléments d'une liste qui vérifient certaines conditions.

`arrayFilters` s'utilise avec l'opérateur `$[<identifier>]` qui permet de créer une référence vers les éléments de la liste dans `<update>`  
Vous pouvez ensuite utiliser `<identifier>` pour spécifier des conditions dans `<option>`. `arrayFilters` est incompatible avec l'utilisation d'un pipeline. 

````{admonition} Example
:class: tip

Alice décide finalement de prendre 5 stickers :
```
db.carts.updateOne({"_id": 1},
                   {"$set": {"items.$[elem].qty": 5}},
                   {"arrayFilters": [{"elem.productID": "P05-612"}]}
                  )
```   
````

## Remplacement

Il est possible de **remplacer** un document à l'aide de la fonction `db.<collection>.replaceOne()`. La syntaxe est la suivante :  
```
db.<collection>.replaceOne(<filter>, <replacement>, <option>)
``` 
où :
- `<replacement>` est le document par lequel remplacer le document sélectionné
- `<option>` est un document acceptant cinq paramètres dont `upsert`  

Si plus d'un document correspond aux conditions spécifiées, seul le premier document sélectionné sera remplacé.

## Indexing

L'**indexation** améliore la **vitesse d'exécution** des requêtes en lecture. 
Si certains **champs** sont **fréquemment sollicités**, créer un index sur ces champs permet d'améliorer les performances des requêtes qui font appel à ces champs.
Pour créer un index, vous pouvez utiliser la fonction `db.<collection>.createIndex()`. 
Nous ne nous attarderons pas sur cet aspect, mais vous trouverez davantage d'informations dans la documentation MongoDB (par exemple [ici](https://www.mongodb.com/docs/manual/reference/method/db.collection.createIndex/) ou [ici](https://www.mongodb.com/docs/manual/core/indexes/index-types/#std-label-index-types)).

````{admonition} Example
:class: tip

Voici un petit exemple pour que vous puissiez constater par vous-même l'impact d'une indexation sur les performances des requêtes en lecture.  
Le code suivant crée une base de données `bank`, une collection `account`, et stocke 100000 documents contenant chacun deux champs : `clientID` et `balance` (solde bancaire). 
Attention, cette requête peut prendre plusieurs minutes.
```
use Bank
db.createCollection("account")
for (i=1; i<=100000; i++){
    // print(i);
    db.account.insert({"clientID":i,"balance":Math.round(Math.random()*1000000)})
}
db.account.countDocuments()
```
La commande suivante permet de mesurer le temps d'exécution d'une requête récupérant les information du client 69522 :
```
db.account.find({"clientID":69522}).explain("executionStats").executionStats.executionTimeMillis
```
Combien de millisecondes l'exécution de cette commande a-t-elle duré ?   
La banque a très fréquemment besoin d'accéder aux informations d'un client. Elle crée donc un index sur le champ `clientID` :  
```
db.account.createIndex({"clientID":1})
db.account.getIndexes()
```
Combien de temps prend désormais la requête précédente pour s'exécuter ?
````
## Exercice

À partir de la base de données `movies.json` :  

1. Quelles sont les différentes années présentent dans la base de données ? Que remarquez-vous ?
2. Corrigez les erreurs

Créez la base de données suivante, qui contient des informations sur quelques uns des livres disponibles dans trois bibliothèques municipales (en centre-ville, sur le campus universitaire, et en banlieue), leur stock, et les emprunts :
```
db.books.insertMany([
  {
    _id: 1,
    title: "1984",
    author: "George Orwell",
    year: 1949,
    genres: ["dystopia", "politics"],
    copies: [
      { branch: "Central", stock: 4, borrowed: 1 },
      { branch: "Campus", stock: 2, borrowed: 0 }
    ],
    ratings: [5, 4, 5]
  },
  {
    _id: 2,
    title: "The Hobbit",
    author: "J.R.R. Tolkien",
    year: 1937,
    genres: ["fantasy", "adventure"],
    copies: [
      { branch: "Central", stock: 3, borrowed: 2 },
      { branch: "Campus", stock: 1, borrowed: 1 }
    ],
    ratings: [5, 5, 4, 5]
  },
  {
    _id: 3,
    title: "Pride and Prejudice",
    author: "Jane Austen",
    year: 1813,
    genres: ["romance", "classic", "horror"],
    copies: [
      { branch: "Central", stock: 2, borrowed: 0 }
    ],
    ratings: [4, 4, 5]
  },
  {
    _id: 4,
    title: "To Kill a Mockingbird",
    author: "Harper Lee",
    year: 1960,
    genres: ["classic", "justice"],
    copies: [
      { branch: "Central", stock: 5, borrowed: 1 },
      { branch: "Suburban", stock: 2, borrowed: 0 }
    ],
    ratings: [5, 5, 5, 4]
  }
])
```
1. Modifiez le titre du livre `1984` en `Nineteen Eighty-Four`  
2. Ajoutez le genre `thriller` à `To Kill a Mockingbird`
3. Une erreur s'est glissée parmi les genres associés au livre `Pride and Prejudice`. Supprimez le genre `horror`.   
4. Face à la forte demande pour le livre `The Hobbit`, la bibliothèque du centre-ville (`Central`) achète un nouveau exemplaire. 
Incrémentez de 1 le `stock` pour cette bibliothèque.  
```{admonition} Aide
:class: dropdown

Utilisez l'option `arrayFilters`.
```
5. Une personne emprunte le livre `Nineteen Eighty-Four` à la bibliothèque du `Campus`. Ajustez le `stock` et le nombre de livres empruntés (`borrowed`) en une seule requête.  
```{admonition} Aide
:class: dropdown

Utilisez l'option `arrayFilters`. Vous pouvez enchaîner plusieurs modifications en les séparant par des virgules. `$inc` peut être associé à des valeurs positives ou négatives. 
```
6. Créez deux nouveaux champs pour chaque livre, `totalBorrowed` et `totalStock`, qui contiennent respectivement le total de toutes les copies empruntées et le total des livres en stock.
```{admonition} Aide
:class: dropdown

Voir l'exemple n°1 de la partie 4.1.2. Vous pouvez enchaîner plusieurs modifications en les séparant par des virgules.
```
7. Calculez et stockez dans un nouveau champ `avgRating` la moyenne des notes pour chacun des livres. Si vous le souhaitez, vous pouvez utiliser `$trunc` pour ne garder que la première ou les deux premières décimales.
8. Pour chaque livre et chaque bibliothèque, ajoutez un champ `availability` qui stocke la différence entre le nombre de livres en stock et le nombre de livres empruntés.
```{admonition} Aide
:class: dropdown

Utilisez `$map`, `$mergeObjects` et `$subtract`.
```

## En résumé

Dans ce chapitre vous avez appris que :
- Il est possible de créer des index et de forcer des structures dans les données (par exemple en utilisant `$mergeObjects` pour définir des champs "obligatoires" et spécifier des valeurs par défaut). Un schéma flexible ne signifie par l'absence de schéma. Un schéma flexible signifie que la structure des données peut être modifiée à la volée, et que des contraintes peuvent être ajoutées au fur et à mesure, selon les besoins. 
- Modifier des documents imbriqués et des listes de documents imbriqués peut être laborieux. Il est important de bien penser ses agrégats en amont pour limiter les coûts en écriture, ou de revoir les agrégats en fonction de l'évolution des besoins.  
  

