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

# Requête complexe

La clause `MATCH` est souvent associée à une **sous-clause `WHERE`** ainsi qu'à d'**autres clauses et opérateurs** afin d'affiner les requêtes. Neo4j liste toutes les [clauses](https://neo4j.com/docs/cypher-manual/current/clauses/) ainsi que tous les [opérateurs disponibles](https://neo4j.com/docs/cypher-manual/current/expressions/) dans sa documentation.
La syntaxe générale est la suivante :  
```sql
MATCH ...
WHERE ...
...
RETURN
...
```

:::{caution}
Une variable peut être **implicitement passée à la clause suivante** si elle est référencée dans une opération (ex : `max()`). 
Une variable peut également être **explicitement transmise à la clause suivante** à l'aide de la clause `WITH` (voir 3.2.2). 
Si une variable n'est **ni implicitement ni explicitement transmise** à la clause suivante, elle sera **supprimée** et ne sera **plus disponible** pour référence ultérieure dans la requête.  
:::

## Sous-clauses

`WHERE` doit être utilisée avec une clause `MATCH`, `OPTIONAL MATCH` ou `WITH`. Elle ne peut pas être utilisée seule, c'est pourquoi on parle de sous-clause. 
`WHERE` permet de spécifier des **conditions de filtrage supplémentaires**.   

````{admonition} Example
:class: tip

**Exemple n°1.** Les deux instructions suivantes sont équivalentes. Que font-elles ?  
```
MATCH (p:Person{born:1965})
RETURN collect(p.name) AS personBorn1965
```
```
MATCH (p:Person)
WHERE p.born = 1965
RETURN collect(p.name) AS personBorn1965
```
**Exemple n°2.** `WHERE` permet de réaliser des filtrages impossibles à réaliser avec une clause `MATCH` uniquement, comme des filtrages basés sur des conditions d'inégalité. Que fait cette commande ?
```
MATCH (m:Movie)<-[:DIRECTED]-(:Person{name:"Lana Wachowski"})
WHERE m.released < 2005
RETURN m
```
````
`ORDER BY` modifie l'**ordre des résultats** obtenus à l'issue d'un `RETURN` ou d'un `WITH`. 
Depuis la version 5.24 de Neo4j, `ORDER BY` peut aussi être utilisé seul. La syntaxe est la suivante :   
```sql
ORDER BY <property_or_variable> [ASC|DESC]
```
Par défaut, `ORDER BY` trie les résultats en fonction d'une propriété ou d'une variable par ordre croissant (`ASC` pour *ascending*). 
Pour trier par ordre décroissant, il suffit de spécifier `DESC` en fin d'instruction. Il est possible de spécifier plusieurs propriétés et/ou variables à la fois.   
````{admonition} Example
:class: tip

**Exemple n°1.** Pour trier par ordre croissant de parution les films produits ou réalisés par `Nora Ephron`, tapez :
```
MATCH (:Person{name:"Nora Ephron"})-[:PRODUCED|DIRECTED]->(m:Movie)
RETURN m.title AS movies, m.released AS year
ORDER BY m.released 
```
Vous auriez aussi pu placer `ORDER BY` avec `RETURN`.   
**Exemple n°2 [Activité des acteurs].** Pour trier les acteurs par ordre décroissant en fonction du nombre de films dans lesquels ils ont joué, tapez : 
```
MATCH (actor:Person)-[:ACTED_IN]-(:Movie)
RETURN actor.name AS actor, count(*) AS movieCount
ORDER BY movieCount DESC
``` 
**Exemple n°3.** Que fait cette commande ?  
```sql
MATCH (a:Person)-[:ACTED_IN]-(m:Movie)
ORDER BY m.released, a.born DESC
RETURN m.title AS movie, m.released AS date, collect(a.name) AS actors, collect(a.born) AS birthdate
```
````

`LIMIT` **limite le nombre de résultat** renvoyés. `LIMIT` est présentée dans cette section, car bien que cette clause puisse être utilisée seule, 
elle est généralement combinée avec `ORDER BY`. La syntaxe est la suivante : `LIMIT <n>` où `<n>` le nombre de résultats à renvoyer.      
````{admonition} Example
:class: tip

**Exemple n°1.** Reprenons l'exemple portant sur l'activité des acteurs. Pour ne retourner que l'un des acteurs (dans le cas où plusieurs acteurs seraient à égalité) les plus actifs du jeu de données, vous pouvez utiliser `LIMIT` :   
```
MATCH (actor:Person)-[:ACTED_IN]-(:Movie)
RETURN actor.name AS actor, count(*) AS movieCount
ORDER BY movieCount DESC
LIMIT 1
```
```` 

## Clauses de projection

Les clauses de projection définissent les variables ou expressions à renvoyer en tant que résultat de la requête (`RETURN`), 
ou à transmettre à la clause suivante en tant que résultats intermédiaires.    

La clause `WITH` permet de créer de nouvelles variables à l'aide de `AS`, contrôler quelles variables sont transmises à la clause suivante, 
réaliser des agrégations, supprimer des duplicats à l'aide de `DISTINCT`, etc. En fait, `WITH` peut être vu comme un `RETURN` intermédiaire.   
:::{caution}
Lorsque `WITH` est utilisé pour contrôler quelles variables sont transmises à la clause suivante, 
**toute variable non explicitement référencée dans `WITH` est supprimée** et ne peut pas être référencée par les clauses suivantes. 
De même, **toute variable renommée dans `WITH`** ne peut être **référencée** que par son **nouveau nom** dans les clauses suivantes. 
:::

````{admonition} Example
:class: tip

**Exemple n°1.** Reprenons l'exemple portant sur l'activité des acteurs. Cette fois-ci, non seulement nous souhaitons renvoyer l'un des acteurs le plus actif de la base de données,
mais nous aimerions aussi lister tous les films dans lesquels il a joué. Pour y parvenir, il suffit de remplacer le `RETURN` par un `WITH`, 
de passer tout le noeud associé à l'acteur le plus actif aux clauses suivantes (et pas seulement le nom de l'acteur), 
puis de réaliser un nouveau `MATCH` à partir à partir de celui-ci pour récupérer le nom des films :
```
MATCH (actors:Person)-[:ACTED_IN]-(:Movie)
WITH actors, count(*) AS movieCount // agrégation, création d'une nouvelle variable et contrôle des variables transmises
ORDER BY movieCount DESC // seuls `actors` et `movieCount` sont désormais disponibles
LIMIT 1
MATCH (actors)-[:ACTED_IN]->(movies:Movie)
RETURN actors.name AS actor, movieCount, collect(movies.title) AS movies
```
**Exemple n°2.** Pour trouver les films dans lesquels certains acteurs sont nés la même année, nous pouvons commencer par grouper par film, puis par date de naissance des acteurs, et compter le nombre d'acteurs dans chaque groupe. 
Ensuite, nous pouvons sélectionner les groupes (film, date de naissance) pour lesquels le comptage est strictement plus grand que 1, 
et renvoyer le titres des films :
```
MATCH (m:Movie)<-[:ACTED_IN]-(p:Person)
WITH m.title as movie, p.born as birthdate, count(*) as countBirthdate
WHERE countBirthdate > 1
RETURN DISTINCT movie
```
````

`UNWIND` permet de transformer une liste en plusieurs lignes. 

## Exercice 1

1. Quels acteurs n'ayant jamais réalisé de film ont-ils joué dans un film entre 2000 et 2005 ? Affichez leur nom, le nombre de films et la liste desdits films. Vous pouvez utiliser `AND`.
2. Affichez le nombre de films par année de parution dans l'ordre croissant des années.
3. Quels sont les 3 films comptabilisant le plus d'acteurs ?
4. Quels sont le ou les acteurs les plus vieux a avoir joué dans dans `Joe Versus the Volcano` ? 
5. Y a-t-il des acteurs nés à une autre date dans ce film ? Ne rentrez pas à la main ladite date.

````{admonition} Bonus
:class: dropdown

Nous allons déterminer quelle est la période pendant laquelle chaque acteur a été le plus actif :     
6. Construisez un tableau avec une ligne par acteur et une colonne pour chaque période : `]-inf, 1990[`, `[1990, 1995[`, `[1995, 2000[`, `[2000, 2005[`, `[2005, 2010[`, `[2010, +inf[`  
```{admonition} Aide
:class: dropdown

`count()` peut contenir une expression.
```

````

## Clauses de sous-requêtes

(à venir : `COUNT`, `COLLECT`, `EXISTS`)

## Opérateurs

(à venir : `AND`, `OR`, `NOT`, etc ; >=, <, etc ; `IS NOT NULL`, `IS NULL` ; `STARTS WITH`, `ENDS WITH`)
