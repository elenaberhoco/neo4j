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

# Requête avec opérateurs

On peut effectuer des requêtes plus complexes à l'aide d'opérateurs.   

````{admonition} Example
:class: tip

Dans la suite, nous utiliserons la base de données suivante :
```javascript
use Sport
```
```javascript
db.athletes.insertOne({
                       "nom": "Alice Martin",
                       "age": 22,
                       "genre": "F",
                       "taille": 175,
                       "sport": ["basketball", "volleyball"],
                       "blessures": ["entorse cheville"],
                       "categorie": "amateur",
                       "seances": []
                     })

db.athletes.insertOne({
                       "nom": "Bruno Silva",
                       "age": 28,
                       "genre" : "M",
                       "taille": 182,
                       "sport": ["football"],
                       "blessures": [],
                       "categorie": "professionnel",
                       "seances": [8,8,9,8]
                     })

db.athletes.insertOne({
                       "nom": "Carla Lopez",
                       "age": 19,
                       "genre": "F",
                       "taille": 160,
                       "sport": ["tennis"],
                       "blessures": ["tendinite épaule", "fracture poignet"],
                       "categorie": "professionnel",
                       "seances": [7,7,7,10]
                     })

db.athletes.insertOne({
                       "nom": "David Chen",
                       "age": 31,
                       "genre": "M",
                       "taille": 190,
                       "sport": ["basketball"],
                       "categorie": "semi-professionnel",
                       "seances": [6,5,6,7]
                     })

db.athletes.insertOne({
                       "nom": "Emma Dubois",
                       "age": 25,
                       "genre": "F",
                       "taille": 168,
                       "sport": ["athlétisme", "natation"],
                       "blessures": ["élongation ischio-jambiers"],
                       "categorie": "semi-professionnel",
                       "seances": [6,7,6,7]
                     })

db.athletes.insertOne({
                       "nom": "Alex Legrand",
                       "age": 22,
                       "genre": "M",
                       "taille": 175,
                       "sport": ["MMA"],
                       "blessures": ["entorse cheville","fracture du nez","élongation ischio-jambiers"],
                       "seances" : [8,5,9,10]
                     })
```
````

## Opérateurs de comparaison

Les **opérateurs de comparaison** permettent de comparer deux éléments entre eux. La syntaxe est la suivante :  
```
db.<collection>.find({<key>:{<operator>:<value>}}, <projection>)
```

|  Opérateur  |     Notation mathématique    |    Description    |
|:---:|:---:|:---:|
|  `$eq`  | = |  **eq**ual to  |
|  `$ne`  | ≠ |  **n**ot **e**qual  |
|  `$lt`  | < |  **l**ess **t**han  |
|  `$gt`  | > |  **g**reater **t**han  |
|  `$lte`  | ≤ |  **l**ess **t**han or **e**qual   |
|  `$gte`  | ≥ |  **g**reater **t**han or **e**qual|
|  `$in`  | ∈ |  **in**  |
|  `$nin`  | ∉ |  **n**ot **in**  |
|  `$exists`  | $\exists$ |  **exists**  |

Les opérateurs `$eq`, `$ne`, `$lt`, `$gt`, `$lte` et `$gte` s’utilisent avec des nombres, des booléens, des chaînes de caractères, des dates, des expressions régulières etc.

````{admonition} Example
:class: tip

**Exemple n°1.** Pour récupérer le nom de tous les athlètes de plus de 25 ans (non compris), tapez :
```
db.athletes.find({"age":{"$gt":25}},{"nom":1,"_id":0})
```
**Exemple n°2.** Pour exclure les athlètes hommes, tapez :
```
db.athletes.find({"genre":{"$ne":"M"}})
```
Notez que les commandes suivantes sont équivalentes :
```
db.athletes.find({"genre":{"$eq":"M"}})
db.athletes.find({"genre":"M"}})
```
````

Il est possible d'**enchaîner les opérateurs** de comparaison pour extraire des documents en fonction d'un **intervalle de valeurs**.
````{admonition} Example
:class: tip

Afin de savoir quels sports les athlètes mesurant entre 1m80 et 2m pratiquent, tapez :
```
db.athletes.find({"taille":{"$gte":180,"$lte":200}}, {"sport":1,"_id":0})
```
````

`$nin` étant la négation de `$in`, les deux opérateurs s'utilisent de la même façon. 
Ils doivent être associés à une **liste de valeurs**. 
`$in` (resp. `$nin`) permet de sélectionner (resp. exclure) les documents dont la valeur ou l'une des valeurs d'un champ est **égale à au moins l'une des valeurs de la liste** spécifiée.  
````{admonition} Example
:class: tip

La commande suivante permet d'obtenir l'ensemble des blessures des athlètes amateurs et semi-professionnels :
```
db.athletes.find({"categorie":
                     {"$in":["amateur","semi-professionnel"]}
                  },
                  {"blessures":1}
)
```
````

:::{caution}
Vous pouvez avoir quelques surprises lorsque vous utilisez des **opérateurs de négation**, tels que `$nin`, `$nor` et `$not`. 
Par exemple, en indiquant avec `$nin` que vous souhaitez accéder aux documents pour lesquels tel champ ne contient pas telles valeurs, 
les documents qui ne contiennent pas ledit champ ou pour lequel le champ est vide seront aussi retournés. 
Ces champs ne vérifient effectivement aucune égalité. Ils ne vérifient donc pas non plus les égalités que vous souhaitez exclure. 
Pour vous en convaincre, voir l'exemple en suivant. 
Par conséquence, si vous souhaitez également exclure les documents qui **ne possèdent par le champ** utilisé dans le filtre ou pour lesquels **le champ est vide**, vous devez le faire explicitement.  
:::
````{admonition} Example
:class: tip

Que renvoie la requête suivante ? Exécutez-la.
```
db.athletes.find({"categorie":{"$nin":["amateur"]}})
```
Que remarquez-vous ? Corrigez la commande.
````

Enfin, l'opérateur `$exists` permet de **vérifier l'existence d'une clé** dans un document. `$exists` peut prendre la valeur `true` ou `false`.
````{admonition} Example
:class: tip

Existe-t-il des documents qui n'ont pas de champ `blessures` ?
```
db.athletes.find({"blessures":{"$exists":false}})
``` 
````

## Opérateurs logique

Les **opérateurs logiques** permettent de tester plusieurs conditions simultanément. 

|  Opérateur  |    Description    |
|:---:|:---|
|  `$and` ou virgule  |  Extraire les documents qui vérifient **toutes** les conditions de la requête |
|  `$or`  |  Extraire les documents qui vérifient l'**une** des conditions de la requête |
|  `$nor`  |  Extraire les documents qui ne vérifient **aucune** des conditions de la requête (ni ... ni) |
|  `$not`  |  Extraire les documents qui ne vérifient pas la condition spécifiée  |

La syntaxe pour `$and`, `$or` et `$nor` est la suivante :  
```
db.<collection>.find({<operator>:[<filter1>,<filter2>,...]}, <projection>)
```

Comme vu dans le chapitre précédent, pour extraire les documents qui vérifient **un ensemble de conditions**, il suffit de séparer lesdites conditions par une virgule. Toutefois, vous pouvez également utiliser l'opérateur `$and`.
````{admonition} Example
:class: tip

Pour retourner le nom des athlètes femmes de plus d'1m65 vous pouvez tapez au choix :
```
db.athletes.find({"genre":"F","taille":{"$gt":165}}, {"nom":1})
``` 
ou
```javascript
db.athletes.find({"$and": 
                     [
                       {"genre":"F"},
                       {"taille":{"$gt":165}}
                     ]
                 },
                 {"nom":1}
)
```
````

L'opérateur `$or` permet d'extraire les documents qui vérifient **au moins l'une des conditions** spécifiées.
````{admonition} Example
:class: tip

Pour obtenir des informations sur les athlètes amateurs ou semi-professionnels, tapez :
```
db.athletes.find({"$or":
                     [
                       {"categorie":"amateur"},
                       {"categorie":"semi-professionnel"}
                     ]
                 })
``` 
N'a-t-on pas déjà effectué une requête similaire à l'aide d'un autre opérateur ? MongoDB recommande de privilégier la première option lorsque les conditions d'égalité sont appliquées à un même champ, comme c'est le cas ici avec `categorie`. Cependant, cet exemple permet d'illustrer le fait qu'on peut implémenter une même requête de différentes façons. 
````

L'opérateur `$nor` permet d'extraire les documents qui **ne vérifient aucune** des **conditions** spécifiées.
````{admonition} Example
:class: tip

Pour trouver l'ensemble des athlètes qui ne jouent pas au football et qui ne sont pas des femmes, tapez :
```
db.athletes.find({"$nor":
                     [
                       {"sport":"football"},
                       {"genre":"F"}
                     ]
                 })
```
````

Enfin, `$not` permet d'extraire les documents qui ne vérifient pas la condition spécifiée. La syntaxe est la suivante :
```
db.<collection>.find({<key>:{"$not":{<operator_expression>}}}, <projection>)
```
où `<operator_expression> = {<operator>:<value>, ...}`

````{admonition} Example
:class: tip

La commande suivante permet d'exclure les athlètes qui ont entre 25 et 30 ans :
```
db.athletes.find({"age":
                     {
                       "$not":
                          {"$gte":25,"$lt":30}
                     }
                 })
```
Exclure les athlètes qui ont entre 25 et 30 ans revient entre à extraire les athlètes de moins de 25 ans ou de plus de 30 ans. Sauriez-vous écrire la requête correspondante ?
````

## Requête sur des listes

Les requêtes sur les listes ont quelques spécificités.

|  Opérateur  |    Description    |
|:---:|:---|
|  `$size`  |  Taille de la liste  |
|  `$all`  |  Extraire les documents pour lesquels le champ de type liste contient toutes les valeurs spécifiées |
|  `$elemMatch`  |  Extraire les documents pour lesquels le champ de type liste a au moins un élément qui correspond à tous les critères spécifiés  |

Lorsque vous sélectionnez des documents en fonction du contenu d'une liste, chaque condition que vous posez sera considérée comme remplie si **l'un des éléments** de la liste la vérifie. Par exemple :
- La condition `{<key>:<value>}` sera vérifiée si **l'un des éléments** de la liste vaut `<value>`, c'est-à-dire si la liste contient la valeur, quelque soit les autres éléments de la liste.
- La condition `{<key>:{"$lt":<value>}}` sera vérifiée si **l'un des éléments** de la liste est strictement inférieur à la valeur. Autrement dit, les autres éléments peuvent être supérieurs ou égaux à la valeur. 
- La condition `{<key>:{"$in":<value>}}` sera vérifiée si **l'un des éléments** de la liste appartient à `<value>`, c'est-à-dire si l'un des éléments de la liste est égal à l'une des valeurs de `<value>`.      

Attention, lorsque vous utilisez des **opérateurs de négation**, il suffit que **l'un des éléments** de la liste vérifie la **condition à exclure** pour que le document soit écarté. Par exemple :
- La condition `{<key>:{"$ne":<value>}}` ne sera pas vérifiée si **l'un des éléments** de la liste est égal `<value>`, quelque soit les autres éléments de la liste.   
- La condition `{<key>:{"$nin":<value>}}` ne sera pas vérifiée si **l'un des éléments** de la liste appartient à `<value>`, quelque soit les autres éléments de la liste. 

````{admonition} Example
:class: tip

**Exemple n°1.** La commande suivante sélectionne les joueurs de basketball, de football, ainsi que ceux qui pratiquent les deux sports :
```
db.athletes.find({"sport":{"$in":["basketball","football"]}})
```
**Exemple n°2.** `seances` indique le nombre d'entraînements par semaine. Certains athlètes ont-ils fait des semaines à moins de 6 séances ? 
Notez que cela ne signifie pas que ces athlètes font *systématiquement* des semaines à moins de 6 séances.    
```
db.athletes.find({"seances":{"$lt":6}})
```
**Exemple n°3.** Quels sont les athlètes qui ne font pas de basketball ?
```
db.athletes.find({"sport":{"$nin":["basketball"]}})
```
Remarquez que l'athlète Alice Martin, qui pratique le basketball et le volleyball, a été exclue.
````

`$size` compte et renvoie le nombre d'éléments contenus dans une liste. Cet opérateur permet de filtrer les documents en fonction de la **taille d'une liste**.

````{admonition} Example
:class: tip

La commande suivante permet d'extraire la catégorie des athlètes s'étant blessés deux fois :
```
db.athletes.find({"blessures":{"$size":2}},{"categorie":1})
```
````

:::{caution}
`$size` ne peut pas être combiné à des opérateurs de comparaison. 
Si vous souhaitez retourner la catégorie des athlètes s'étant blessés moins de 2 fois, vous ne pouvez pas tapez : `db.athletes.find({"blessures":{"$size":{"$lte":2}}})`. 
Il faudra utiliser `$or` et lister tous les cas possibles : 
```
db.athletes.find({
                   "$or":
                      [
                        {"blessures":{"$size":2}},
                        {"blessures":{"$size":1}},
                        {"blessures":{"$size":0}}
                      ]
                 },
                 {"categorie":1}
)
``` 
:::

`$all` sélectionne les documents pour lesquels le champ de type liste contient **toutes les valeurs** spécifiées, sans tenir compte de l'ordre ou des autres éléments de la liste. La xyntaxe est la suivante :
```
db.<collection>.find({<key>:{"$all":[<value1>,<value2>,...]}})
```
````{admonition} Example
:class: tip

La commande suivante permet de trouver les athlètes qui ont fait une semaine de 7 séances et une semaine de 6 séances :
```
db.athletes.find({"seances":{"$all":[7,6]}})
```
````

`$elemMatch` renvoie les documents pour lesquels au moins **un élément** du champ de type liste **vérifie toutes les conditions** spécifiées. La syntaxe est la suivante :   
```
db.<collection>.find({<key>:{"$elemMatch":<filter>}})
```
où `<filter>` peut être une expression avec ou sans opérateurs.   

Pour faire des recherches sur une liste de documents, la syntaxe est la suivante : 
```
db.<collection>.find({<key>:{"$elemMatch":{<sub_key>:<filter>}}})
```


Pour bien comprendre l'intérêt de cet opérateur, considérez les exemples suivants :

````{admonition} Example
:class: tip

D'après vous, que renvoie la commande suivante ? Exécutez-la.
```
db.athletes.find({"seances":{"$gte":7,"$lt":6}})
```
Est-ce ce à quoi vous vous attendiez ? Que fait cette commande ?

La commande suivante devrait être conforme à vos attentes :
```
db.athletes.find({"seances":{"$elemMatch":{"$gte":7,"$lt":6}}})
```
````

En résumé : 
- Si vous souhaitez qu'un ensemble de **conditions** soient vérifiées au moins une fois par un ou plusieurs éléments de la liste, **simultanément ou non**, n'utilisez pas `$elemMatch`.   
- Si vous souhaitez qu'un ensemble de **conditions** soient vérifiées **simultanément** par **au moins un élément** de la liste, utilisez `$elemMatch`.  
- Si vous souhaitez qu'un ensemble de **conditions** soient vérifiées **simultanément** par **tous les éléments** de la liste, utilisez `$nor` et listez les conditions d'exclusion.  

:::{caution}
**Rappel :** Attention aux surprises lorsque vous appliquez des **opérateurs de négation** à une liste. 
Lorsque vous construisez vos requêtes n'oubliez pas les champs absents et les champs vides (dont les listes vides).
:::

````{admonition} Example
:class: tip

Que renvoie la commande suivante ? Exécutez-la.
```
db.athletes.find({"seances":{"$not":{"$lte":7}}})
```
Que remarquez-vous ? Corrigez la commande.
```` 

## Exercice

À partir du jeu de données utilisé dans ce chapitre : 

1. Retournez les informations des athlètes `David Chen` et `Emma Dubois`.

2. Quels athlètes n'ont pas eu d'élongation des ischio-jambiers (`élongation ischio-jambiers`)?

3. Certains athlètes ont-ils déjà fait à la fois des semaines à plus de 9 séances et des semaines à 5 séances ou moins ?       

4. Quels athlètes ont déjà fait entre 3 et 6 séances lors d'une semaine d'entraînement ?    

5. Quels athlètes font au minimum 1 séance par jour, mais pas plus de 9 séances par semaine ?

6. Quelle est la catégorie des athlètes qui se sont blessés 2 fois ou plus ? 

7. Quels athlètes pratiquent au moins un sport autre que l'athlétisme ?  

À partir du jeu de données `movies.json` :  

1. Listez le titre et la date des films sortis après 2000 par ordre croissant de sortie     

2. Dans combien de films l'acteur John Travolta a-t-il joué ?

3. Combien y a-t-il de films dans lesquels aucun acteur n'est né en 1955 (`birth_date`) ?

4. Listez le titre des films dans lesquels l'un des acteurs est né entre 1955 et 1958. Proposez deux façons d'obtenir cette information.  

**Bonus**:  
5. Dans quel film Anthony Hopkins et Jodie Foster ont-ils joué tous les deux ?  

## En résumé

|  Opérateur  |    Description    |
|:---:|:---|
|  `$eq`  |  **eq**ual to  |
|  `$ne`  |  **n**ot **e**qual  |
|  `$lt`  |  **l**ess **t**han  |
|  `$gt`  |  **g**reater **t**han  |
|  `$lte`  |  **l**ess **t**han or **e**qual   |
|  `$gte`  |  **g**reater **t**han or **e**qual|
|  `$in`  |  **in**  |
|  `$nin`  |  **n**ot **in**  |
|  `$exists`  |  **exists**  |
|  `$and` ou virgule  |  Extraire les documents qui vérifient **toutes** les conditions de la requête |
|  `$or`  |  Extraire les documents qui vérifient l'**une** des conditions de la requête |
|  `$nor`  |  Extraire les documents qui ne vérifient **aucune** des conditions de la requête |
|  `$not`  |  Extraire les documents qui ne vérifient pas la condition spécifiée  |
|  `$size`  |  Taille d'une liste  |
|  `$all`  |  Extraire les documents pour lesquels le champ de type liste contient toutes les valeurs spécifiées |
|  `$elemMatch`  |  Extraire les documents pour lesquels le champ de type liste a au moins un élément qui correspond à tous les critères spécifiés  |

