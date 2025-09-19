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

# Attributs de type date

* Auteurs/trices : **ABOULKACEM Zakaria, GARY Gaston, HAMELIN Marine, RALIN Kévin**

Ce chapitre traite des attributs de type dates (et sous-cas des listes de dates) et des différents types de requêtes que l'on peut vouloir faire sur de tels attributs.


## Qu'est-ce qu'une date dans MongoDB ?

Les dates sur MongoDB sont définies a la milliseconde près, nous allons nous intéresser aux requêtes utilisant des dates, car les attributs de type date doivent être considérés de façon particulière.
Les dates sont renseignées sous la forme ISO date, nous retrouvons la date de la forme suivante : 

```javascript
("<YYYY-mm-ddTHH:MM:ssZ>")
```
Ce format nous permet de d’avoir une version de référence (UTC) de la date spécifiée.
Lorsque l'on utilise le shell MongoDB, on va créer des dates en Javascript et les utiliser pour nos requêtes.
Dans ce cadre, nous pouvons mettre uniquement l’année ou l’année et le mois sans devoir spécifier tout le langage s’occupe de remplir les autres arguments pour retrouver la forme indiquée plus haut.

Exemple :

```javascript
ma_date = new Date ('2021')
```
nous donne un objet de type date de la forme suivante :
```javascript
ISODate("2021-01-01T00 :00:00Z")
```

Nous 	avons aussi d’autre formats de dates qui donne la date suivant 
fuseau horaire locale de l’utilisateur.
```javascript
("<YYYY-mm-ddTHH:MM:ss>")
```

Ainsi, il faut toujours utiliser une comparaison sous forme d’intervalle pour ces attributs.
Lorsque l’on souhaite effectuer un test sur une date, on utilisera des opérateurs de comparaison.

Ainsi, si nous souhaitions effectuer un regroupement par date, il faudrait préciser à quel degré de précision sur la date l’agrégation doit se faire.



## Manipulations standard

### Utilisation d'un objet date au format Date.

Exemple d'une requête simple dans la db `food`. On veut récupérer la liste des restaurants dont la date de la note est supérieure à une date créée.

```{code-cell}
use food
```

```{code-cell}
:tags: [output_scroll]

madate = new Date("1995-02-13")
db.NYfood.find({"grades.date": {$gt : madate}})
```

### Utilisation d'un objet date dans une requête d'égalité.

Exemple d'une requête simple dans la base de donnée `etudiants`. On veut récupérer les étudiants nés le 13 février 1995. Il est important d'utiliser l'encadrement. En effet, lors de la création d'une date, la précision est à la milliseconde près. 
La requête suivante nous renvoie donc les étudiants nés le 13 février 1995 à 00 heures, 00 minutes et 00 secondes (et 0 millisecondes).

```{code-cell}
use etudiants
```

```{code-cell}
:tags: [output_scroll]

madate = new Date("1995-02-13")
db.notes.find(
    {"ddn": madate}
)
```

Pour avoir les étudiants nés le 13 février 1995, nous utiliserions la requête suivante :
```{code-cell}
:tags: [output_scroll]

madateinf = new Date("1995-02-13");
madatesup = new Date("1995-02-14");

db.notes.find(
    {"ddn": {$gte: madateinf,
             $lt: madatesup}}
)
```

### Utilisation d'un objet date dans une liste.

Exemple d'une requête où l'attribut `date` de type date est inclus dans la liste `grades` dans la base de donnée `food`. La requête suivante nous retourne les restaurants possédant au moins une note attribué après le 5 octobre 2014.

```{code-cell}
use food
```

```{code-cell}
:tags: [output_scroll]

madate = new Date("2014-10-05")
db.NYfood.find(
    {"grades.date": {$gt: madate}}
)
```
Pour avoir les restaurants qui ont obtenu au moins une notes au mois de novembre 2014, nous utiliserions l'opérateur `$elemMatch`. Attention à donner l'attribut de type liste, ici `grades`, avant l'opérateur et à spécifier l'attribut de type date, `date`, dans `$elemMatch`.

```{code-cell}
use food
```

```{code-cell}
:tags: [output_scroll]

madate1 = new Date("2014-11-01")
madate2 = new Date("2014-12-01")
db.NYfood.find(
    {"grades": {$elemMatch: {"date": {$gte: madate1,$lt: madate2}}
               }
    }
)
```

### Utilisation d'un objet date dans un aggregate

Il est possible d'accéder aux attributs années, mois, jours,... d'une date. Ceci est très utile pour les [requêtes d'agrégations](05_agreg.md).

Exemple d'une requête de regroupement par le mois et l'années. 
Nous voulons afficher mois par mois le nombre d'étudiants ayant leur date de naissance à ce mois dans la base `etudiants`.

```{code-cell}
use etudiants
```

```{code-cell}
db.notes.aggregate([
    {$group:
        {_id: {month: {$month: "$ddn"},
                year: {$year: "$ddn"}},
                nb: {$sum: 1}
        }
    }
])
``` 


## Transformation de formats

Les deux procédés opératoires présentés ci-dessous sont applicable uniquement au sein de la méthode aggregate.

### Présentation des formats

Ci-dessous un récapitulatif des formats disponibles

| abbreviation       |     Description     |        Valeurs possibles |
| :------------ | :-------------: | -------------: |
| %d       |     jour du mois     |        01-31 |
| %G     |   Année dans le format ISO 8601    |      0000-9999 |
| %H        |     Heure      |         00-23 |
| %L        |     Miliseconde      |         000-999 |
| %m        |     Mois      |         01-12 |
| %M        |     Minute      |         00-59 |
| %S        |     Seconde      |         00-60 |
| %u        |     jour par rapport à la semaine      |         1-7 (Lundi-Dimanche) |
| %v        |     Semaine      |         1-53 |
| %Y        |     année      |         0000-9999 |
| %z        |     décalage temporel par rapport à UTC      |         +/-[hh][mm] |
| %Z        |     décalage temporel converti en minutes      |         +/-mmm |
| %%        |     afficher le caractère pourcentage      |         % |

### `$dateFromString`


```javascript
 { $dateFromString: {
     dateString: <dateStringExpression>,
     format: <formatStringExpression>,
     timezone: <tzExpression>,
     onError: <onErrorExpression>,
     onNull: <onNullExpression>
} }
```
La méthode `$datefromstring` permet de convertir une chaine de caractère représentant une date en objet date pour pouvoir effectuer des calculs et comparaison dessus.

La méthode prend en argument : 

* la date en format texte (Obligatoire)
* Le format
* Le fuseau horaire
* Une gestion potentielle des erreurs
+ une gestion potentielle des valeurs nulles


#### Exemples

Partons du postulat qu'il existe une variable de type string qui décrit une temporalité dans la collection notes. 

```javascript
db.notes.aggregate(
   [
     {
       $project: {
          objetdate : { 
              $dateFromString: {
              dateString: "23-04-2021 à 15 heures 25 minutes et 23 secondes", format: "%d-%m-%Y à %H heures %M minutes et %S secondes"
                } 
              }
       }
     }
   ]
)
```

### `$dateToString`

On retrouve la méthode inverse avec des arguments similaires.


```javascript
 { $dateToString: {
    date: <dateExpression>,
    format: <formatString>,
    timezone: <tzExpression>,
    onNull: <expression>
} }
```
La méthode prend en argument : 

* la date en format texte (Obligatoire)
* Le format
* Le fuseau horaire
* Une gestion potentielle des valeurs nulles

#### Exemples

```{code-cell}
db.notes.aggregate(
   [
     {
       $project: {
          year_str: { $dateToString: { format: "%Y", date: "$ddn"} },
          month_str : {$dateToString: { format: "%m", date: "$ddn"} },
          date_str : {$dateToString: { format: "%d-%m-%Y", date: "$ddn"} }
       }
     }
   ]
)
```
