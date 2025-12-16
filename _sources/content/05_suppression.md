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

# CRU**D** : Suppression

::::{grid}
:gutter: 1

:::{grid-item-card} Objectif
Dans ce chapitre vous apprendrez à supprimer des noeuds, des relations, des graphes complets, des propriétés et des labels.  
  
:::
::::

Les exemples réutilisent la base de données du chapitre précédent.

## Noeuds, relations et graphes

La clause `DELETE` permet de supprimer des noeuds, des relations et des graphes entiers.

Pour **supprimer un noeud isolé**, c'est-à-dire sans aucune relation entrante ou sortante, la syntaxe est la suivante : 
```
MATCH (<node>)
DELETE <node>
```

````{admonition} Example
:class: tip

Essayez d'exécuter la commande suivante : 
```
MATCH (p:Person{name:"Fanny"})
DELETE p
```
Que se passe-t-il ?
````

Pour **supprimer une relation** sans supprimer les noeuds liés par cette relation, la syntaxe est la suivante (l'orientation de la flèche est purement illustrative) :
```
MATCH (<start_node>)-[<relationship>]->(<end_node>)
DELETE <relationship>
```

````{admonition} Example
:class: tip

Fanny quitte Sorbonne Université :
```
MATCH (:Person{name:"Fanny"})-[r:AFFILIATED_WITH]->(:University {name:"Sorbonne Université"})
DELETE r
```
Nous pouvons vérifier que tout s'est bien passé :
```
MATCH (p), ()-[r]-()
RETURN p, r
``` 
Notez que nous récupérons séparément les noeuds et les relation dans la clause `MATCH`, de façon à aussi afficher les noeuds isolés.
````

Il n'est pas possible de supprimer des noeuds liés par des relations sans supprimer également ces relations. 
Pour **supprimer à la fois les noeuds et les relations entrantes et sortantes** associées, 
vous pouvez supprimer explicitement les relations ou utiliser la clause `DETACH DELETE` :
```
MATCH (<node>)-[<relationship>]-()
DELETE <node>, <relationship>
```
ou
```
MATCH (<node>)
DETACH DELETE <node>
```
````{admonition} Example
:class: tip

Supprimons le noeud `Claire` et toutes les relations associées :
```
MATCH (c:Person{name:"Claire"})-[r]-()
DELETE c,r
```

Supprimons le noeud `Bob` et toutes les relations associées :
```
MATCH (b:Person {name:"Bob"})
DETACH DELETE b
```
````

Pour **supprimer l'ensemble des noeuds et des relations**, la syntaxe est la suivante : 
```
MATCH (n)
DETACH DELETE n
```
Cette commande ne supprime **ni les index ni les contraintes** d'après la documentation. 
Elle n'est pas non plus adaptée pour supprimer de grandes quantités de données.

Pour **supprimer un grand graphe sans supprimer les index et les contraintes**, préférez :
```
MATCH (n)
CALL (n) {
 DETACH DELETE n
} IN TRANSACTIONS
```

Pour **supprimer l'entièreté du graphe, y compris les index et les contraintes**, il faut recréer la base de données en utilisant la commande : 
```
CREATE OR REPLACE DATABASE <database_name>
```

## Propriétés et labels

La clause `REMOVE` permet de **supprimer des propriétés et des labels**.

La syntaxe pour les **propriétés** est la suivante : 
* Pour un noeud :
```
MATCH (<node>)
REMOVE <node>.<property>
```
* Pour une relation
```
MATCH (<start_node>)-[<relationship>]->(<end_node>)
REMOVE <relationship>.<property>
```

Il est également possible de supprimer des propriétés à l'aide `SET` et de `null`. 
La syntaxe pour les noeuds est la suivante (à transposer aux relations) :
```
MATCH (<node>)
SET <node>.<property> = null
```

Pour supprimer l'**ensemble des propriétés**, une façon de procéder est la suivante (à transposer aux relations) :
```
MATCH (<node>)
SET <node> = {}
```

````{admonition} Example
:class: tip

Supprimons l'`age` de `Fanny` :
```
MATCH (f:Person{name:"Fanny"})
REMOVE f.age
RETURN f.name AS name, f.age AS age
```
Supprimons le statut du projet `PoTATO` :
```
MATCH (p:Project{title:"PoTATO"})
SET p.status = null
RETURN p.title as title, keys(p) 
```
````

La syntaxe pour les **labels** est la suivante : 
```
MATCH (<node>)
REMOVE <node>:Label:...
```

Pour supprimer l'**ensemble des labels**, une façon de procéder est la suivante :
```
MATCH (<node>)
REMOVE <node>:$(labels(<node>))
```

````{admonition} Example
:class: tip

Supprimons le label `Researcher` :
```
MATCH (p:Person)
REMOVE p:Researcher
RETURN p.name AS name, labels(f) AS labels
```
````

````{admonition} Example
:class: tip

Supprimons tout le jeu de données :
```
MATCH (n)
DETACH DELETE n
```
Ou
```
CREATE OR REPLACE DATABASE neo4j
```
````

## Exercice

1. Construisez le graphe ci-dessous
```{figure} ../image/exemple_neo4j_suppression.png
```
2. Supprimez le noeud isolé.
3. Il y a une erreur dans la base de données : `Impression, Soleil levant` se trouve en réalité au `Musée Marmottan-Monet`. 
Supprimez la relation `DISPLAYED_AT` et créez le noeud et la relation adéquats.
4. Supprimez le noeud `Guernica`.
5. Ajoutez au noeud `Claude Monet` une propriété indiquant l'année de naissance et de mort de l'artiste.
6. Plutôt que d'indiquer la date de création des oeuvres dans les propriétés des noeuds `Artwork`, 
vous décidez de déplacer l'information dans les relations `CREATED`. 
Créez la nouvelle propriété à partir de l'ancienne et supprimez l'ancienne.
7. Supprimez tous les labels des noeuds `Museum`.
8. Supprimez tout le graphe.


