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
Dans ce chapitre, vous apprendrez à modifier des noeuds et des relations.  
   
:::

::::

La clause `SET` permet de modifier les labels des noeuds et les propriétés des noeuds et des relations.

````{admonition} Example
:class: tip

Dans la suite, nous utiliserons le graphe suivant, proposé dans la [documentation](https://neo4j.com/docs/cypher-manual/current/clauses/set/) : 
```
CREATE
  (a:Person:Swedish {name:'Andy', age:36, hungry:true}),
  (b:Person {name:'Stefan'}),
  (c:Person {name:'Peter', age:34}),
  (d:Person {name:'George'}),
  (a)-[r1:KNOWS]->(c),
  (b)-[r2:KNOWS]->(a),
  (d)-[r3:KNOWS]->(c)
RETURN a,b,c,d,r1,r2,r3
```
````

La syntaxe pour **ajouter ou modifier une propriété** est la suivante :
* Pour un noeud :
```
MATCH (<node>)
SET <node>.<key> = <value> 
```
* Pour une relation (l'orientation de la flèche est purement illustrative) :
```
MATCH (<start_node>)-[<relationship>]->(<end_node>)    
SET <relationship>.<key> = <value>
```
Si la propriété n'existe pas, elle est créée, sinon, la valeur est remplacée. 
Il est possible d'ajouter ou de modifier plusieurs propriétés à la fois en les séparant par des virgules.

````{admonition} Example
:class: tip

Précisons le nom de famille et l'âge de Stefan :
```
MATCH (n:Person{name:'Stefan'})
SET n.surname = 'Smith', n.age = 51
RETURN n.name AS name, n.surname AS surname, n.age AS age 
```
Andy vient de manger et n'a plus faim :
```
MATCH (n:Person{name:'Andy'})
SET n.hungry = false
RETURN n
```
Précisons que George connaît Peter depuis 2001 :
```
MATCH (n:Person{name:'George'})-[r:KNOWS]->(m:Person{name:'Peter'})
SET r.since = 2001 
RETURN n,r,m
```
````

Vous pouvez également **remplacer entièrement les propriétés** d'un noeud ou d'une relation à l'aide de la syntaxe suivante :
* Pour un noeud :
```
MATCH (<node>)
SET <node> = {<key>:<value>,...}
``` 
* Pour une relation (l'orientation de la flèche est purement illustrative) :
```
MATCH (<start_node>)-[<relationship>]->(<end_node>)
SET <relationship> = {<key>:<value>,...}
```

Grâce à l'opérateur `+=`, vous pouvez **simultanément ajouter, modifier ou laisser inchangé** les propriétés d'un noeud ou d'une relation à l'aide de la syntaxe suivante : 
* Pour un noeud :
```
MATCH (<node>)
SET <node> += {<key>:<value>,...}
```
* Pour une relation (l'orientation de la flèche est purement illustrative) :
```
MATCH (<start_node>)-[<relationship>]->(<end_node>)
SET <relationship> += {<key>:<value>,...}
```
Les propriétés spécifiées entre accolades remplaceront les valeurs du noeud ou de la relation pour les clés déjà existantes, seront créées sinon, 
tandis que les propriétés des noeuds et des relations qui n'apparaissent pas dans les accolades seront laissées inchangées.
````{admonition} Example
:class: tip

Modification du nom, ajout d'un poste, et suppression de l'âge, qui n'est pas spécifié dans les accolades :
```
MATCH (p {name:'Peter'})
SET p = {name:'Peter McCartney', position:'Guitarist'}
RETURN p.name AS name, p.age AS age, p.position AS position
```
Ajout de l'âge et d'une propriété `hungry`, modification du nom, pas de modification des propriétés `name` et `position`, qui ne sont pas spécifiées dans les accolades :
```
MATCH (p {name:'Peter McCartney'})
SET p += {name:'Peter', age: 38, hungry: true}
RETURN p.name AS name,
       p.age AS age,
       p.hungry AS isHungry,
       p.position AS position
```
````

La syntaxe pour **ajouter un ou plusieurs labels** à un noeud est la suivante :
```
MATCH (<node>)
SET <node>:<Label>:...
```
````{admonition} Example
:class: tip

George est français :
```
MATCH (n:Person{name:'George'})
SET n:French
RETURN labels(n) as labels
```
````

Les propriétés comme les labels peuvent être définis dynamiquement.

````{admonition} Example
:class: tip

Pour créer un label à partir des postes occupés par individus, vous pouvez faire : 
```
MATCH (n:Person)
WHERE n.position IS NOT NULL
SET n:$(n.position)
RETURN n.name AS name, labels(n) AS labels
```
````

::::{grid}
:gutter: 1

:::{grid-item-card} Exercice

À partir du jeu de données ci-dessous, répondez aux questions suivantes.
```
CREATE
  (a:Person {name:"Alice", age:30, role:"researcher"}),
  (b:Person {name:"Bob", age:26}),
  (c:Person {name:"Claire", age:28, role:"engineer"}),
  (d:Person {name:"Fanny", age:43, role:"researcher"}),

  (p1:Project {title:"PoTATO", status:"ongoing"}),
  (p2:Project {title:"GreenAI", status:"finished"}),

  (u1:University {name: "Université de Lorraine"}),
  (u2:University {name: "Université Paris-Saclay"}),
  (u3:University {name:"Sorbonne Université"})
CREATE
  (a)-[:WORKS_ON]->(p1),
  (b)-[:WORKS_ON]->(p1),
  (c)-[:WORKS_ON]->(p2),
  (d)-[:WORKS_ON]->(p2),

  (a)-[:AFFILIATED_WITH]->(u1),
  (b)-[:AFFILIATED_WITH]->(u1),
  (c)-[:AFFILIATED_WITH]->(u2),
  (d)-[:AFFILIATED_WITH]->(u3),
  (d)-[:AFFILIATED_WITH]->(u2)
```
1. `Bob` est un doctorant. Ajoutez le `role` `PhD`.
2. Le projet `PoTATO` est maintenant terminé. Changez le statut de `ongoing` à `finished`.
3. Ajoutez un label `Researcher`, `Engineer` ou `PhD` à chacun des noeuds `Person` en fonction de la valeur associée à la clé `role`.
4. Pour chaque noeud `Person`, créez une clé `seniority` avec pour valeur `senior` pour les plus de 30 ans et `junior` pour les moins de 30 ans.
5. Créez une clé `since` pour la relation `AFFILIATED_WITH` de `Claire` : `Claire` fait partie de l'Université Paris-Saclay depuis 2022. 

:::
::::
