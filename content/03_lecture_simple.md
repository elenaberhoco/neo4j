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

# Requête simple

La méthode `find()` permet de **sélectionner des documents en fonction de critères de filtrage**. Elle renvoie un **curseur**, c'est-à-dire un pointeur vers les résultats d'une requête. Le curseur permet d'itérer sur les résultats **un lot à la fois**. La syntaxe de la méthode `find()` est la suivante :    

```
db.<collection>.find(<filter>, <projection>)   
```

````{admonition} Example
:class: tip

Dans la suite, nous utiliserons la base de données suivante :
```
use Cuisine
```
```
db.recettes.insertOne({
                       "nom":"Ratatouille",
                       "type":"plat principal",
                       "ingredients": ["aubergine", "courgette", "poivron", "tomate", "oignon","ail"],
                       "origine":{"pays":"France","region":"Provence"}
                     })  

db.recettes.insertOne({
                       "nom":"Tiramisu",
                       "type":"dessert",
                       "ingredients":["oeufs","sucre","mascarpone","boudoirs","café","cacao"],
                       "origine":{"pays":"Italie","region":"Vénétie"},
                       "etapes": [
                           { "numero": 1, "description": "Séparer les blancs des jaunes. Fouetter les jaunes avec le sucre jusqu’à blanchiment." },
                           { "numero": 2, "description": "Incorporer le mascarpone aux jaunes sucrés jusqu’à obtenir une crème lisse." },
                           { "numero": 3, "description": "Monter les blancs en neige et les incorporer délicatement à la crème." },
                           { "numero": 4, "description": "Tremper rapidement les biscuits dans le café froid et tapisser le fond du plat." },
                           { "numero": 5, "description": "Alterner une couche de crème, une couche de biscuits. Terminer par de la crème." },
                           { "numero": 6, "description": "Réfrigérer au moins 4 h. Saupoudrer de cacao avant de servir." } 
                       ]
                     })  

db.recettes.insertOne({
                       "nom":"Soupe à l'oignon",
                       "type":"entrée",
                       "ingredients": ["oignon","vin","gruyère","beurre"]
                     })  

db.recettes.insertOne({
                       "nom":"Quiche lorraine",
                       "type":"plat principal",
                       "ingredients":["pâte brisée", "oeufs", "crème fraîche", "lardons", "gruyère"],
                       "origine":{"pays":"France","region":"Lorraine"}
                     })  

db.recettes.insertOne({
                       "nom":"Mousse au chocolat",
                       "type":"dessert",
                       "ingredients": ["chocolat noir", "oeufs", "sucre"]
                     })  
```
````

## Requête sans condition

Pour **lister l'ensemble des documents de la collection**, il suffit d'utiliser `find()` sans spécifier de conditions. S'il y a plus de documents que ce qui peut être renvoyé en une fois (taille du lot), il faudra taper `it` (*iterate*) une ou plusieurs fois pour accéder aux documents suivants.   
````{admonition} Example
:class: tip

```
db.recettes.find()  
db.recettes.find({})
```
````
La fonction `findOne()` permet de **ne retourner qu'un seul document**.
````{admonition} Example
:class: tip

```
db.recettes.findOne()   
db.recettes.findOne({})
```
````

## Requête avec condition

### Filtre           

Un **filtre de sélection** peut être passé à `find()` pour ne renvoyer que les documents correspondant à certains critères.  

La syntaxe pour extraire les documents dont l'un des champs vérifie une **condition d'égalité** est la suivante :
```
db.<collection>.find({<key>:<value>})
``` 

````{admonition} Example
:class: tip

La commande suivante permet de retourner toutes les recettes de dessert :

```
db.recettes.find({"type":"dessert"})
```
````
:::{caution}
Cette opération se comporte différemment avec les listes. Elle renvoie les documents pour lesquels la liste **contient** la valeur.
:::
````{admonition} Example
:class: tip

La commande suivante retourne tous les plats contenant des oeufs, c'est-à-dire le tiramisu, la quiche Lorraine et la mousse au chocolat :
```
db.recettes.find({"ingredients":"oeufs"})
```
Notez que cette commande n'est pas équivalente à :
```
db.recettes.find({"ingredients":["oeufs"]})
```
Dans ce cas, aucun document n'est retourné car aucun document n'a un champ `ingredients` *exactement* égal à `["oeufs"]`. Ils contiennent tous des ingrédients supplémentaires.   
````
Il est possible de spécifier plusieurs filtres en les séparant par une virgule. Dans ce cas, seuls les documents vérifiant l'ensemble des conditions d'égalité seront retournés (ET logique).

````{admonition} Example
:class: tip

Pour récupérer tous les plats principaux contenant des oeufs il faut donc taper : 

```
db.recettes.find({"type":"plat principal","ingredients":"oeufs"})
```
````

La syntaxe `<embedded_object>.<embedded_key>` permet d'accéder aux attributs d'un **document imbriqué** ou aux valeurs d'une **liste**. Il devient alors possible d'extraire des documents **en fonction du contenu** de ces objets.

````{admonition} Example
:class: tip

**Exemple n°1.** Que retourne la commande suivante ? 
```
db.recettes.find({"origine.pays":"France"})
```
Notez que cette commande n'est pas équivalente à :
```
db.recettes.find({"origine":{"pays":"France"}})
```
Dans ce cas, aucun document n'est retourné car aucun document n'a un champ `origine` *exactement* égal à `{"pays":"France"}`. Ils contiennent tous un champ imbriqué additionnel : `region`.   

**Exemple n°2.** Que renvoie la commande suivante ?
```
db.recettes.find({"ingredients.1":"oeufs"})
```
````
Cette façon de procéder fonctionne également pour les listes de documents imbriqués. Par exemple :

````{admonition} Example
:class: tip

Pour récupérer tous les documents dont la recette a une troisième étape il suffit de taper :
```
db.recettes.find({"etapes.numero":3})
```
````

### Projection

La plupart du temps, seule une partie de l'information contenue dans les documents nous intéresse. Le second argument de la fonction `find()`, appelé **projection**, permet de **spécifier les champs à renvoyer** dans les documents qui correspondent au filtre de requête. Pour ce faire, vous pouvez :
- Indiquer les champs que vous souhaitez **inclure** : `{<key>:<1_or_true>}`  
- Indiquer les champs que vous souhaitez **exclure** : `{<key>:<0_or_false>}`   

Vous ne pouvez **pas** faire **les deux à la fois**. La seule exception est `_id` que vous pouvez explicitement exclure tout en listant des champs à inclure. L'attribut `_id` est inclus par défaut dans les documents renvoyés.

````{admonition} Example
:class: tip

**Exemple n°1.** Afin de tenir compte des allergies, nous voudrions extraire le nom et le type de toutes les recettes contenant des oeufs. Voici la commande :
```
db.recettes.find({"ingredients":"oeufs"},{"nom":true,"type":true})
```
**Exemple n°2.** Pour trouver la région d'origine de la Ratatouille et exclure le champ `_id` des résultats, il suffit de taper :
```
db.recettes.find({"nom":"Ratatouille"},{"origine.region":1,"_id":0})
```
**Exemple n°3.** Pour connaître le nom de tous les plats disponibles dans notre collection de recettes, il suffit de ne fixer aucune condition sur les documents :  
```
db.recettes.find({},{"nom":true,"_id":false})
``` 
````

## Modification du curseur

Certaines méthodes modifient la manière dont la requête sous-jacente est exécutée. La syntaxe est la suivante:
```
db.<collection>.find(<filter>, <projection>).<cursor_modifier>
```

`count()` modifie le curseur afin qu'il renvoie le nombre de documents qui correspondent au filtre de requête plutôt que les documents eux-mêmes.
````{admonition} Example
:class: tip

Combien y a-t-il de recettes contenant du gruyère ?
```
db.recettes.find({"ingredients":"gruyère"}).count()
```
````

`sort()` renvoie les documents triés dans l'ordre croissant (1) ou décroissant (-1) en fonction du ou des champs spécifiés. La syntaxe est la suivante :  
```
db.<collection>.find(<filter>, <projection>).sort({<key1>:<1_or_-1>,<key2>:<1_or_-1> ...})
```
````{admonition} Example
:class: tip

Pour trier les recettes par `type` dans l'ordre inverse de l'ordre alphabétique, *puis* par `nom` dans l'ordre alphabétique :
```
db.recettes.find().sort({"type":-1,"nom":1})
```
````

`limit()` limite le nombre de documents renvoyés.
````{admonition} Example
:class: tip

Trouvons une idée de dessert à cuisiner :
```
db.recettes.find({"type":"dessert"},{"_id":0}).limit(1)
```
````

## Exercice

1. Téléchargez le jeu de données `movies.json`.
2. Affichez le premier document pour vous familiariser avec le nom des clés. Dans ce jeu de données, tous les documents ont les mêmes clés.
3. Récupérez les informations du film `Blade Runner`.   
```{admonition} Aide (anglais)     
:class: dropdown

En anglais, *titre* se dit *title*.
```
4. Récupérez le titre des films du `genre` `drama`. Que signifie le message qui apparait en bas de votre écran: `Type "it" for more` ? 
5. Récupérez le nom et le prénom du réalisateur de `Kill Bill`.  
```{admonition} Aide (anglais)
:class: dropdown

En anglais, *nom de famille* se dit *last name*, *prénom* se dit *first name*, et *réalisateur* se dit *director*.
```
6. Combien y a-t-il de films français (`FR`) dans la base de données ?  
```{admonition} Aide (anglais)
:class: dropdown

En anglais, *pays* se dit *country*.
```
7. Lister les films sortis en `1995` par ordre alphabétique. Quel est le premier ?   
```{admonition} Aide (anglais)
:class: dropdown

En anglais *année* se dit *year*.
```
8. Dans quels films l'acteur `Bruce Willis` a-t-il joué ? Retournez le titre et le genre des films.    

Avec des requêtes simples, on peut déjà extraire beaucoup d'informations intéressantes !

## En résumé

Dans ce chapitre vous avez appris que :
- Les **valeurs** des paires clé-valeur sont **consultables** : elles sont accessibles et peuvent être utilisées pour filtrer les documents.   
- Sans documentation, il peut être difficile de comprendre la structure des données et donc de les utiliser.

Dans ce chapitre vous avez manipulé les fonctions suivantes :  

Syntaxe | Description
--- | ---
`db.<collection>.find()` | Récupérer tous les documents d'une collection  
`db.<collection>.findOne()` | Récupérer le premier document d'une collection  
`db.<collection>.find({<key>:<value>,...})` | Filtrer les documents en fonction d'une ou plusieurs conditions d'égalité  
`db.<collection>.find(<filter>,<projection>)` où `<projection> = {<key>:<0_or_1>,...}` | Récupérer tous les champs spécifiés dans la projection, pour tous les documents qui répondent aux critères de recherche  
`db.<collection>.find(<filter>,<projection>).count()` | Compter le nombre de documents qui répondent aux critères de recherche   
`db.<collection>.find(<filter>,<projection>).sort(<condition>)` où `<condition> = {<key>:<1_or_-1>,...}` | Trier les documents qui répondent aux critères de recherche
`db.<collection>.find(<filter>,<projection>).limit(<n>)` | Limiter le nombre de documents qui répondent aux critères de recherche à renvoyer

````{admonition} Tip
:class: note

`db.<collection>.find(<filter>,<projection>)` peut se comprendre ainsi : "je veux extraire les documents de la `<collection>` qui vérifient les conditions suivantes : `<filter>`, et pour ces documents je veux récupérer les informations suivantes : `<projection>`."

````
