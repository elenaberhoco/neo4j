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

# **C**RUD : Création

::::{grid}
:gutter: 1
  
:::{grid-item-card} Objectif
Dans ce chapitre vous apprendrez à insérer des noeuds et des relations dans une base de données Neo4j.    
:::

::::

Nous allons construire ce graphe : 

```{figure} ../image/exemple_neo4j_creation.png
---
---
Exemple from: [source](https://neo4j.com/docs/cypher-manual/current/subqueries/collect/)
```


## Créer un noeud

### `CREATE`

La clause `CREATE` permet de **créer un ou plusieurs noeuds**. Chaque noeud peut se voir attribuer des **labels** et des **propriétés**. 
La syntaxe est la suivante : 
```
CREATE (<node>:<Labels>{<key>:<value>,...}), ...
```
où :
- `<node>` : nom de la variable qui pointera vers les noeuds sélectionnés, si besoin (voir Chapitre 2)
- `<Labels>` : un ou plusieurs labels
- `{<key>:<value>,...}` : propriétés

Il est possible de spécifier **plusieurs labels** en utilisant :  
- soit `:`, syntaxe : `(<node>:<Label1>:<Label2>:...)`   
- soit `&`, syntaxe : `(<node>:<Label1>&<Label2>:...)`    
 
Vous pouvez créer **plusieurs noeuds** à la fois en les séparant par des virgules. 

````{admonition} Example
:class: tip

```
CREATE (:Person:Director {name:'Tim Burton'}), 
       (:Person {name:'Benedict Cumberbatch'})
```
````
:::{caution}
**Si le noeud existe déjà, il sera dupliqué** (sauf si contrainte d'unicité) ! Pour éviter cela, préférez `MERGE` et posez des contraintes d'unicité. 
`CREATE` est plus adapté à la construction initiale de la base de données, 
tandis que `MERGE` convient mieux lorsque vous souhaitez ajouter des noeuds à une base de données déjà existante. 
:::

::::{grid}
:gutter: 1

:::{grid-item-card} Exercice
1. Créez un noeud `Person` avec pour nom (`name`) `Andy` et pour `age` `36`
2. Créez un noeud de labels `Dog` et `Pet` avec pour nom (`name`) `Ozzy` et pour jouet (`toy`) `Banana` 
:::

::::



### `MATCH` et `CREATE`

`MATCH`, combiné ou non à d'autres clauses, permet d'utiliser les propriétés de noeuds existants lors de la création d'un nouveau noeud.  

````{admonition} Example
:class: tip

Imaginons que vous vouliez avoir facilement accès à l'ensemble des films sortis en 1992, sans avoir besoin de parcourir toutes les propriétés de tous les noeuds 
`Movie` à chaque fois :  
```
MATCH (m:Movie{released:1992})
WITH collect(m.title) as movies
CREATE (:Year{released:1992, movies:movies})
``` 
````

::::{grid}
:gutter: 1

:::{grid-item-card} Exercice        
1. Créez des noeuds `Toy` avec le nom (`name`) des jouets (`toy`) des chiens et des chats. Pensez bien à exclure les noeuds qui n'ont pas de propriété `toy` !
:::

::::


### `MERGE`

La clause `MERGE` agit comme une combinaison de `MATCH` et `CREATE` : elle vérifie d'abord l'existence d'un noeud, avant de le créer s'il n'existe pas. 
Vous ne pouvez spécifier qu'un seul noeud à la fois avec `MERGE`.

````{admonition} Example
:class: tip

Notez la différence du message renvoyé par Neo4j après exécution de ces deux instructions :
```
MERGE (:Person:Director {name:'Tim Burton'})
```
```
MERGE (:Movie {title:'The Hobbit', released:2012, tagline:"From the smallest beginnings come the greatest legends"})
```
````
:::{caution}
Si la **correspondance n'est pas exacte** entre un **noeud existant** et le **noeud à créer**, et si le noeud à créer a des **propriétés** ou des **labels** 
que le noeud existant n'a pas, alors un **nouveau noeud** sera **créé** (sauf si contrainte d'unicité).   
  
Par exemple, écrire :
```
MERGE (:Person:Actor {name:'Benedict Cumberbatch'})
``` 
créerait un nouveau noeud, car le seul noeud existant contenant cette propriété est le noeud `(:Person {name:'Benedict Cumberbatch'})` qui ne possède que 
le label `Person`.   
  
En revanche, si vous ne spécifiez qu'un **sous-ensemble** des propriétés et/ou des labels, le noeud sera reconnu.   
  
Par exemple, écrire :
```
MERGE (:Person {name:'Tim Burton'})
``` 
sélectionnerait le noeud `(:Person:Director {name:'Tim Burton'})`.

:::

::::{grid}
:gutter: 1

:::{grid-item-card} Exercice
1. Créez un noeud de labels `Dog` et `Pet` avec pour nom (`name`) `Popo`
:::

::::

## Créer une relation   

### `CREATE`     

La clause `CREATE` permet de **créer une ou plusieurs relations ainsi que les noeuds associés**. 
Chaque relation doit avoir **exactement un type** et une **direction**. Elle peut aussi se voir attribuer des **propriétés**. 
La syntaxe est la suivante (l'orientation de la flèche est purement illustrative) :  
```
CREATE (<start_node>)-[<relationship>:<TYPE>{<key>:<value>,...}]->(<end_node>) ...
```
où :
- `<start_node>`, `<end_node>` : noeuds tels que définis à la section précédente
- `<relationship>` : nom de la variable qui pointera vers les relations sélectionnées, si besoin (voir Chapitre 2)
- `<TYPE>` : type de la relation
- `{<key>:<value>,...}` : propriétés

La **direction** d'une relation doit être spécifiée à l'aide d'une flèche : `-->` ou `<--`. 
Vous pouvez créer **plusieurs relations** à la fois en les séparant par des virgules. 
Vous pouvez créer des **motifs complexes**, c'est-à-dire incluant plusieurs relations (donc aussi plusieurs noeuds).  

````{admonition} Example
:class: tip

```
CREATE (:Person {name:'Anthony Hopkins', born:1937})-[:ACTED_IN]->(:Movie {title:"The Silence of the Lambs", released:1991})<-[:DIRECTED]-(:Person {name:'Jonathan Demme', born:1944})
```
````
:::{caution}
**Si la relation ou les noeuds existent déjà, ils seront dupliqués** (sauf si contrainte d'unicité) ! Pour éviter la duplication des noeuds, 
utilisez `CREATE` en combinaison avec `MATCH`. 
Pour éviter toute duplication, préférez `MERGE` et posez des contraintes d'unicité.
:::

::::{grid}
:gutter: 1

:::{grid-item-card} Exercice 
1. Créez une personne (`Person`) avec pour nom (`name`) `Timothy` et pour `age` `25`, 
un chat (`Cat`, `Pet`) avec pour nom `Mittens`, 
et une relation `HAS_CAT` avec une propriété `since` qui vaut `2019` entre les deux 
:::

::::

### `MATCH` et `CREATE`

Pour créer une nouvelle relation entre deux noeuds existants, faites précéder `CREATE` d'une clause `MATCH` (voir Chapitre 2). 

````{admonition} Example
:class: tip

```
MATCH (benedict:Person{name:'Benedict Cumberbatch'})  
MATCH (hobbit:Movie{title:'The Hobbit'})
CREATE (benedict)-[:ACTED_IN{roles:['Smaug','Sauron']}]->(hobbit)
```
````
:::{caution}
**Si la relation existe déjà, elle sera dupliquée** (sauf si contrainte d'unicité) ! Pour éviter toute duplication, 
préférez `MERGE` et posez des contraintes d'unicité.
:::

::::{grid}
:gutter: 1

:::{grid-item-card} Exercice 
1. Créez une relation `HAS_DOG` avec une propriété `since` qui vaut `2016` entre `Andy` et `Popo`
:::

::::

### `MATCH` et `MERGE`

La clause `MERGE` permet de vérifier l'existence d'une relation, et de la créer si elle n'existe pas. 

````{admonition} Example
:class: tip

L'instruction suivante ne crée aucune relation, car la relation existe déjà : 
```
MATCH (benedict:Person{name:'Benedict Cumberbatch'})
MATCH (hobbit:Movie{title:'The Hobbit'})
MERGE (benedict)-[:ACTED_IN{roles:['Smaug','Sauron']}]->(hobbit)
```
````
::::{grid}
:gutter: 1

:::{grid-item-card} Exercice
1. Créez une relation `IS_FRIEND` avec une propriété `since` qui vaut `2002` de `Andy` vers `Timothy`. Et inversement ! 
:::

::::

## Créer des noeuds et des relations

La clause `MERGE` permet de vérifier à la fois l'existence des noeuds et l'existence des relations, voire l'existence d'un motif complet, avant création. 
Si le noeud, la relation, ou le motif existe déjà, il est sélectionné. Sinon, il est créé.  

````{admonition} Example
:class: tip

L'instruction suivante sélectionne le noeud `Tim Burton` déjà existant, crée le noeud `Sleepy Hollow`, qui n'existe pas encore, 
et crée la relation entre ces deux noeuds, qui n'existe pas encore :
```
MERGE (tim:Person{name:'Tim Burton'}) 
MERGE (movie:Movie{title:'Sleepy Hollow', released:1998, tagline:'Sleep If You Can. You can lock the doors. You can bolt the windows. But can you survive the night?'})
MERGE (tim)-[:DIRECTED]->(movie)
```
````

:::{caution}

Lorsque vous utilisez `MERGE` sur des **motifs complets**, soit le motif **complet** existe déjà et il est sélectionné, soit il est créé **dans son intégralité**, 
**même si certaines parties du motif existent** déjà.   
 
Par exemple, écrire : 
```
MERGE (:Person{name:'Benedict Cumberbatch'})-[:ACTED_IN]->(:Movie{title:'The Imitation Game',released:2014})
```
créerait un nouveau noeud `Benedict Cumberbatch` car le motif dans son intégralité n'existe pas : il n'existe pas de noeud `The Imitation Game` ni, donc, 
de relation entre le noeud `Benedict Cumberbatch` et le noeud `The Imitation Game` de type `ACTED_IN`.  
::: 
 
Il faut donc décomposer le motif en fonction des besoins.

````{admonition} Example
:class: tip

Imaginons que vous vouliez créer autant de noeuds que d'années de sortie, et lier les films à l'année qui correspond par une nouvelle relation de type 
`RELEASED_IN` :
```
MATCH (m:Movie)
MERGE (y:Year{released:m.released})
MERGE (m)-[r:RELEASED_IN]->(y)
RETURN m,r,y
```
Lorsqu'un noeud `Movie` correspond à un noeud `Year` qui n'existe pas encore, le noeud `Year` est créé. Cependant, dès lors que le noeud `Year` est créé, 
les prochains noeuds `Movie` correspondant à ce noeud `Year` ne conduiront pas à la création de duplicatas : ils seront simplement connectés au noeud déjà 
existant.  
````
:::{caution}
Décomposer en autant de clause `MERGE` qu'il y a de noeuds et de relations dans le motif que vous souhaitez créer n'est pas toujours la bonne approche. 
Voir l'exercice ci-dessous.
:::

::::{grid}
:gutter: 1

:::{grid-item-card} Exercice
1. Créez une relation `HAS_TOY` entre `Ozzy` et `Banana`
2. Créez `Peter`, le propriétaire de `Ozzy`, âgé de `51` ans, et connectez les deux noeuds par une relation `HAS_DOG` avec pour propriété `since` qui vaut `2018` 
3. Créez pour chaque animal un noeud `HealthRecord` qui contiendra les informations du carnet de santé de l'animal. 
Les deux noeuds sont à connecter avec une relation `HAS_HEALTH_RECORD`. Réfléchissez bien à la façon dont vous allez décomposer la requête.
:::

::::

## Créer un index

L’**indexation** améliore la **vitesse d’exécution des requêtes en lecture**. 
Si certains **champs** sont **fréquemment sollicités**, **créer un index sur ces champs** permet d’améliorer les performances des requêtes qui font appel à ces champs.   
    
Pour **afficher les index**, tapez :  
```
SHOW INDEXES
```
  
Pour **créer un index**, utilisez :
```
CREATE <index_type> INDEX <index_name> FOR <node_or_relation> ON <key>
```
où :
- `<index_type>` : catégorie d'index
- `<index_name>` : nom donné à l'index, optionnel
- `<node_or_relation>` : `(<node>:<Label>)` ou `()-[<relationship>:<type>]-()`
- `<key>` : `<node>.<key>` ou `<relationship>.<key>`, propriété selon laquelle indexer

Il existe quatre catégories d'index : *range* (plage), *text* (texte), *point* (localisation) et *lookup*. 
Chaque index est optimisé pour certaines requêtes en particulier. 
Dans ce cours, nous ne présenterons que la catégorie par défaut : les index de plage. Dans ce cas, vous pouvez omettre `<index_type>`. 
Pour en savoir plus, voir la [documentation](https://neo4j.com/docs/cypher-manual/current/indexes/search-performance-indexes/).   
 
Vous pouvez créer un index simple à partir d'une seule clé, ou un index composite à partir de plusieurs clés.  

````{admonition} Example
:class: tip

Si cet index n'existait pas encore, vous auriez pu créer un index sur le titre des films ainsi : 
```sql
CREATE INDEX FOR (m:Movie) ON m.title
```
Affichez les index. Si les requêtes simultanées sur le titre du film et l'année de sortie sont fréquentes, vous pouvez créer un index composite ainsi :
```sql
CREATE INDEX FOR (m:Movie) ON (m.title, m.released)
```
Affichez les index. Exécutez l'instruction suivante et affichez de nouveau les index : que remarquez-vous ?  
```
MATCH (m:Movie{title:'The Hobbit', released:2012})
RETURN m
```
````
::::{grid}
:gutter: 1

:::{grid-item-card} Exercice
1. Créez un index sur le nom des personnes 
2. Créez un index sur la propriété `since` de la relation `HAS_DOG`
:::

::::

Pour en savoir plus, voir la [documentation](https://neo4j.com/docs/cypher-manual/current/indexes/search-performance-indexes/managing-indexes/).

## Créer une contrainte

Poser des **contraintes** permet de **garantir la qualité et l'intégrité des données**. 
Quatre catégories de contraintes sont disponibles en Neo4j : 
- **Contrainte d'unicité** : Garantir que les valeurs combinées d'un ensemble de propriétés sont uniques  
- **Contrainte d'existence** : Garantir qu'une propriété existe. Cette option n'est disponible que dans la version payante.  
- **Contrainte de type** : Garantir qu'une propriété a le type requis (entiers, chaînes de caractères ...). 
Voir la [documentation](https://neo4j.com/docs/cypher-manual/current/constraints/managing-constraints/#type-constraints-allowed-properties) pour connaître tous les types autorisés. 
Cette option n'est disponible que dans la version payante.
- **Contrainte clé** : Garantir que toutes les propriétés existent et que les valeurs combinées d'un ensemble de propriétés sont uniques. 
Cette option n'est disponible que dans la version payante.   
  
Pour **afficher les contraintes**, tapez :
```
SHOW CONSTRAINTS  
```

Pour **créer une contrainte**, utilisez :  
```
CREATE CONSTRAINT <constraint_name> FOR <node_or_relationship> REQUIRE <key> <constraint_type>
```
où :
- `<constraint_name>` : nom donné à l'index, optionnel
- `<node_or_relationship>` : `(<node>:<Label>)` ou `()-[<relationship>:<type>]-()`
- `<key>` : propriété selon laquelle indexer
- `<constraint_type>` : `IS UNIQUE` (contrainte d'unicité), `IS NOT NULL` (contrainte d'existence), `IS :: <TYPE>` (contrainte de type), `IS NODE KEY` ou `IS RELATIONSHIP KEY` (contrainte clé)

````{admonition} Example
:class: tip

Pour forcer l'existence d'une propriété, tapez : 
```sql
CREATE CONSTRAINT FOR (n:Person) REQUIRE n.name IS NOT NULL
```
Exécutez cette commande : 
```
CREATE (:Person{born:1998})
```
Si la contrainte n'existait pas déjà, vous auriez pu poser une contrainte d'unicité sur le nom des personne ainsi : 
```
CREATE CONSTRAINT FOR (p:Person) REQUIRE p.name IS UNIQUE
```
````
::::{grid}
:gutter: 1

:::{grid-item-card} Exercice
1. Créez une contrainte sur le type de la propriété `since` de `HAS_DOG` et `HAS_CAT`. La valeur renseignée doit être `INTEGER`. 
2. Essayez d'associer un autre chien nommé `Fido` à `Peter` en spécifiant `two thousand and ten` pour la propriété `since`
:::

::::

