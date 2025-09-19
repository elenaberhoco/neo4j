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

# Titre 1

```{code-cell}
:tags: [output_scroll]
import panel as pn
```


## Titre 1.1


```javascript
{
    "_id" : ObjectId("56011920de43611b917d773d"),
    "nom" : "Paul",
    "notes" : [ 
        10.0, 
        12.0
    ],
    "sexe" : "M"
}
```

(find)=
## Titre 1.2
 
 ```{admonition} Remarque
 
Remarque !
```

### Titre 1.2.1

Text text text

::::{grid}
:gutter: 2

:::{grid-item-card} Retourner toutes les occurences d'une collection avec find()
BLI ?
:::

:::{grid-item-card} Retourner uniquement la première occurence d'une collection avec findOne()
BLO ?
:::
::::

Text

```
code
```
Text


```javascript
db.NYfood.find(
    {"cuisine": "Bakery", "borough": "Bronx"}
)
```

````{tab-set} 
```{tab-item} Onglet 1
Blu
```
```{tab-item} Onglet 2
Blo
```
````
Text Text

`````{tab-set}
````{tab-item} Tab 1 title
My first tab
````

````{tab-item} Tab 2 title
```python
def main():
    return
```
bli
````
`````

MORE TABS !! 

````{tab} Python
```python
def main():
    return
```
````
````{tab} C++
```c++
int main(const int argc, const char **argv) {
  return 0;
}
```
````


### Titre 1.2.2

```{code-cell}
:tags: [output_scroll]
db.NYfood.find({"address.zipcode": "10462"})
```
un `mot` de code

### Titre 1.2.3

Text

```{admonition} On peut mettre ce qu'on veut !
:class: tip

Et tout plein de texte là
```

(operateurs)=
## Titre 1.3


 ```{admonition} Remarque
 
La Remarque
```

(comparaison)=
### Un nouveau titre

Super tableau !

| Opérateur logique 	| Mot clé en MongoDB 	| 
|-	|-	|
| = 	| `$eq` 	|
| ≠ 	| `$ne` 	|
| < 	| `$lt` 	|
| > 	| `$gt` 	|
| ≤ 	| `$lte` 	|
| ≥ 	| `$gte` 	|
| ∈ 	| `$in` 	|
| ∉ 	| `$nin` 	|
| clé existante 	| `$exists` 	|
| \|.\| 	| `$size` 	|

Text text

````{panels}

MongoDB
^^^
```javascript
db.notes.find({"notes": {$gte: 13}})
```

---

SQL
^^^
```sql
SELECT *
FROM notes
WHERE notes >= 13 
```

````

La jolie cellule : 

```{code-cell}
db.getCollectionInfos()
```

Encore plus d'onglets : 

::::{tab-set}
:sync-group: category

:::{tab-item} Label1
:sync: key1

Content 1
:::

:::{tab-item} Label2
:sync: key2

Content 2
:::

::::

::::{tab-set}
:sync-group: category

:::{tab-item} Label1
:sync: key1

Content 1
:::

:::{tab-item} Label2
:sync: key2

Content 2
:::

::::

(resume)=
## Conclusion

### Fiche résumé

Encore un joli tableau !

Objectif | Syntaxe 
--- | --- 
Récupérer toutes les occurrences de la collection | `db.collectionName.find({})`
Récupérer la première occurrence de la collection | `db.collectionName.findOne({})`
Filtrer les données | `db.collectionName.find({"x": valeur})`
Limiter l'affichage des clés | `db.collectionName.find({}, {"key": true}) `
Formater les documents de sortie | `db.collectionName.find({}).pretty()`  
Requêtes avec des opérateurs de comparaison | `db.collectionName.find({"x": {operateur: valeur}})`  
Trier les documents de sortie | `db.collectionName.find().sort({"key" : 1})`
Compter les documents de sortie | `db.collectionName.find({}).count()`
Limiter les documents de sortie | `db.collectionName.find({}).limit(2)` 
Valeurs distinctes d'un champ | `db.collectionName.distinct(champ, {})`

(quiz)=
### Quiz 

Trop bien les réponses cachées !

Pour vous tester et être certain que vous avez bien compris, répondez aux questions du quiz ci-dessous.

**Qu'est-ce qui caractérise MongoDB ?**

1. C'est un modèle orienté graphique
2. C'est un modèle orienté document
3. C'est un modèle structuré

```{admonition} Cliquez pour montrer la réponse
:class: dropdown
*Réponse 2 : C'est un modèle orienté document.*
```

**Que nous renvoie cette requête sur la collection `notes` de la base `etudiants` ?**

```javascript
db.notes.find({}, {"nom": true, "_id": false}) 
```

1. Affiche **tout** le contenu de la collection `notes`
2. Les noms des étudiants de la base et la clé `_id` qui identifie chaque étudiant
3. Tous les noms des étudiants de la base, mais pas les autres clés

```{admonition} Cliquez pour montrer la réponse
:class: dropdown

*Réponse 3 : L'argument `"_id": false` permet de retirer l'affichage de la clé `id`. On ne récupère que le noms des étudiants de la base.*
```

**Comment récupérer la liste des étudiants ayant obtenu exactement deux notes ?**

1. `db.notes.find({"notes": {$size: 2}})`
2. `db.notes.find({"notes": {$exists: true}})`
3. `db.notes.find({$or: [{"notes": {$size: 1}},{"notes": {$size: 2}}]})`

```{admonition} Cliquez pour montrer la réponse
:class: dropdown

*Réponse 1 : L'opérateur `$size` permet de prendre en compte la taille de la liste de valeurs, telle qu'une liste de notes. Ici on souhaite que la liste de notes soit précisement de taille 2.*
```

**Quel opérateur permet de ne renvoyer que les documents qui ne vérifient aucune condition de la liste ?**

1. L'opérateur `$eq`
2. L'opérateur `$nor`
3. L'opérateur `$gt`

```{admonition} Cliquez pour montrer la réponse
:class: dropdown

*Réponse 2 : L'opérateur `$nor` permet de ne renvoyer que les documents qui ne vérifient aucune condition de la liste.*
```

**Quelle méthode ci-dessous ne fait pas partie du langage MongoDB ?**

1. `db.collectionName.find({}).orderby({"key" : 1})`
2. `db.collectionName.find({}).sort({"key" : 1})`
3. `db.collectionName.find({}).limit(3)`

```{admonition} Cliquez pour montrer la réponse
:class: dropdown

*Réponse 1 : Bien que `ORDER BY` soit une instruction en SQL, ce n'est pas disponible pour les bases de données en MongoDB.*
```
