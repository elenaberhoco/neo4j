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
**Exemple n°2.** `WHERE` permet de réaliser des filtrages impossibles à réaliser avec une clause `MATCH` uniquement, comme des filtrages comprenant des conditions d'inégalité. Que fait cette commande ?
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
```sql
MATCH (actor:Person)-[:ACTED_IN]-(:Movie)
RETURN actor.name AS actor, count(*) AS movieCount
ORDER BY movieCount DESC
LIMIT 1
```
```` 

## Exercice 1

1. Quels acteurs ont joué dans un film entre 2000 et 2005 ? Affichez leur nom, le nombre de films et la liste desdits films. Vous pouvez utiliser `AND`, `<=` et `>=`.
2. Affichez le nombre de films par année de parution dans l'ordre croissant des années.
3. Quels sont les 3 films comptabilisant le plus d'acteurs ?

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
de passer le noeud associé à l'acteur le plus actif aux clauses suivantes (et pas seulement le nom de l'acteur), 
puis de réaliser un nouveau `MATCH` à partir à partir de celui-ci pour récupérer le nom des films :
```
MATCH (actors:Person)-[:ACTED_IN]-(:Movie)
WITH actors, count(*) AS movieCount // agrégation, création d'une nouvelle variable et contrôle des variables transmises
ORDER BY movieCount DESC            // seuls `actors` et `movieCount` sont désormais disponibles
LIMIT 1
MATCH (actors)-[:ACTED_IN]->(movies:Movie)
RETURN actors.name AS actor, movieCount, collect(movies.title) AS movies
```
**Exemple n°2.** Pour trouver les films dans lesquels certains acteurs sont nés la même année, nous pouvons commencer par grouper par film, puis par date de naissance des acteurs, et compter le nombre d'acteurs dans chaque groupe. 
Ensuite, nous pouvons sélectionner les groupes (film, date de naissance) pour lesquels le comptage est strictement plus grand que 1, 
et renvoyer le titres des films :
```
MATCH (m:Movie)<-[:ACTED_IN]-(p:Person)
WITH m.title AS movie, p.born AS birthdate, count(*) AS countBirthdate
WHERE countBirthdate > 1
RETURN DISTINCT movie
```
````

`UNWIND` permet de transformer une liste en plusieurs lignes. 
````{admonition} Example
:class: tip

La propriété `roles` des relations de type `ACTED_IN` est associée à des listes d'une ou plusieurs valeurs. 
Pour trouver tous les rôles uniques de la base de données, nous pouvons utiliser `UNWIND` : 
```sql
MATCH ()-[r:ACTED_IN]-()
UNWIND r.roles AS roles
RETURN DISTINCT roles
```
````

## Exercice 2

1. Quels sont le ou les acteurs les plus vieux a avoir joué dans dans `Joe Versus the Volcano` ? 
2. Y a-t-il des acteurs nés à une autre date dans ce film ? Ne rentrez pas à la main ladite date.

````{admonition} Bonus
:class: dropdown
 

3. Construisez un tableau avec une ligne par acteur et une colonne pour chaque période : `]-inf, 1990[`, `[1990, 1995[`, `[1995, 2000[`, `[2000, 2005[`, `[2005, 2010[`, `[2010, +inf[`  
```{admonition} Aide
:class: dropdown

`count()` peut contenir une expression.
```
````

## Clauses de sous-requêtes

`COLLECT`, comme `collect()`, permet de récupérer un ensemble de valeurs sous forme de liste, 
la différence étant que la clause `COLLECT` doit être appliquée à une sous-requête (i.e motif de graphe). 
Pour en savoir plus, voir la [documentation](https://neo4j.com/docs/cypher-manual/current/subqueries/collect/).

````{admonition} Example
:class: tip

Nous pouvons utiliser `COLLECT` pour afficher dans trois colonnes distinctes la liste des films dans lesquels chaque personne a joué, que chaque personne a réalisé, et que chaque personne a produit :   
```
MATCH (p:Person)
RETURN p.name AS name, 
       COLLECT {MATCH (p)-[:ACTED_IN]->(m:Movie) RETURN m.title} AS movieActedIn,
       COLLECT {MATCH (p)-[:DIRECTED]->(m:Movie) RETURN m.title} AS movieDirected,
       COLLECT {MATCH (p)-[:PRODUCED]->(m:Movie) RETURN m.title} AS movieProduced
```
````

`COUNT`, comme `count()`, permet de compter un ensemble d'éléments, 
la différence étant que la clause `COUNT` doit être appliquée à une sous-requête (i.e motif de graphe). 
Pour en savoir plus, voir la [documentation](https://neo4j.com/docs/cypher-manual/current/subqueries/count/).

````{admonition} Example
:class: tip

Nous pouvons reprendre l'exemple précédent et remplacer `COLLECT` par `COUNT` pour retourner uniquement le nombre de films dans chaque catégorie :  
```
MATCH (p:Person)
RETURN p.name AS name,
       COUNT {MATCH (p)-[:ACTED_IN]->(m:Movie)} AS countMovieActedIn,
       COUNT {MATCH (p)-[:DIRECTED]->(m:Movie)} AS countMovieDirected,
       COUNT {MATCH (p)-[:PRODUCED]->(m:Movie)} AS countMovieProduced
```
````

`EXISTS` permet de déterminer si le motif spécifié existe au moins une fois dans le graphe. Pour en savoir plus, voir la [documentation](https://neo4j.com/docs/cypher-manual/current/subqueries/existential/).
````{admonition} Example
:class: tip

Pour chaque acteur, affichons si l'acteur a déjà réalisé ou produit un film : 
```
MATCH (p:Person)-[:ACTED_IN]->(:Movie)
RETURN p.name AS name, 
       EXISTS {(p:Person)-[:PRODUCED|DIRECTED]->(:Movie)} AS hasDirectedOrProduced
```
````  

## Opérateurs

Les **opérateurs de comparaison** permettent de comparer deux éléments entre eux.
|  Opérateur  |    Description    |
|:---:|:---|
| `=` | Égalité |
| `<>` | Différence |
| `<` | Strictement inférieur |
| `>` | Strictement supérieur |
| `<=` | Inférieur ou égal |
| `>=` | Supérieur ou égal |
| `IS NULL` | Clé absente ou valeur égale à `null` (valeur manquante) |
| `IS NOT NULL` | Clé présente et valeur différente de `null` |

````{admonition} Example
:class: tip

Existe-t-il dans la base de données des acteurs pour lesquels aucune information n'est renseignée pour leur date de naissance ?  
```sql
MATCH (a:Person)
WHERE a.born IS NULL
RETURN a
```
````

Les **opérateurs logiques** permettent de tester plusieurs conditions logiques simultanément.

|  Opérateur  |    Description    |
|:---:|:---|
| `AND` | **Toutes** les conditions doivent être vérifiées (l'une et l'autre) |
| `OR` | **Au moins l'une** des conditions doit être vérifiée (soit l'une, soit l'autre, soit les deux) |
| `XOR` | **Une seule** des conditions doit être vérifiée (soit l'une, soit l'autre, mais pas les deux à la fois) |
| `NOT` | La condition **ne doit pas** être vérifiée  |

````{admonition} Example
:class: tip

Que fait la commande suivante ?
```sql
MATCH (m:Movie)
WHERE (m.released <= 1990) OR (m.released >= 2010)
RETURN m.title AS title, m.released AS released
ORDER BY released
```
Que fait la commande suivante ?   
```sql
MATCH (p:Person)
WHERE NOT (a)-[:DIRECTED]->(:Movie)
RETURN p.name AS name
```
````
:::{caution}
Faites attention à bien utiliser des parenthèses quand vous utilisez `NOT` avec plusieurs conditions.
````{admonition} Example
:class: tip

Exécutez l'une après l'autre les deux instructions suivantes. Que remarquez-vous ?

```sql
MATCH (m:Movie)
WHERE NOT ((m.released < 1990) OR (m.released > 2000))
RETURN m.title AS title, m.released AS released
ORDER BY released
```

```sql
MATCH (m:Movie)
WHERE NOT (m.released < 1990) OR (m.released > 2000)
RETURN m.title AS title, m.released AS released
ORDER BY released
```
````
:::

Voici quelques **opérateurs de listes** et de **chaînes de caractères** : 

|  Opérateur  |    Description    |
|:---:|:---|
| `IN` | Vérifier l'appartenance d'une valeur à une liste de valeurs | 
| `STARTS WITH` | Vérifier qu'une chaîne de caractères commence par un ou plusieurs caractères successifs |
| `ENDS WITH` | Vérifier qu'une chaîne de caractères se termine par un ou plusieurs caractères successifs |
| `CONTAINS` | Vérifier qu'une chaîne de caractères contient un ou plusieurs caractères successifs |

Attention : ces opérateurs sont sensibles à la casse (majuscule / minuscule).

````{admonition} Example
:class: tip

**Exemple n°1.** Que retourne la requête suivante ?    
```sql
MATCH (a:Person)-[r:ACTED_IN]->(m:Movie)
WHERE "Neo" IN r.roles
RETURN m.title AS title, a.name AS actor
```
**Exemple n°2.** Quelle est la date de sortie des films `Top Gun` et `Snow Falling on Cedars` ?  
```sql
MATCH (m:Movie)
WHERE m.title IN ["Top Gun","Snow Falling on Cedars"]
RETURN m.title AS title, m.released AS released
``` 
**Exemple n°3.** Que fait la requête suivante ?
```sql
MATCH (p:Person)
WHERE p.name STARTS WITH "Tom "
RETURN p.name AS name
``` 
````
## Exercice 3

1. Quels sont les films commençant par la lettre C sortis en 2005 ou après ?  
2. Quels sont les films sortis en 1990, en 2000 et en 2012 ? Répondez à la question en une seule requête. Proposez deux façons de procéder. 
3. Quels acteurs ont déjà joué plus d'un rôle sur un film ?  
4. Quelles sont les personnes qui ont déjà eu plus de deux casquettes à la fois sur un film ? C'est-à-dire qui ont par exemple été à la fois acteur et réalisateur, ou encore réalisateur et producteur ? 
Retournez d'abord un tableau avec le nom de la personne, le nom du film, et une liste des différentes casquettes qu'elle a eu sur ce film. 
Puis retournez uniquement les noms des personnes sous forme de liste.
```{admonition} Aide
:class: dropdown

Utilisez `type()`, `collect()`, `size()` et `>`.
```
5. Quelles sont les personnes qui n'ont jamais eu plus de deux casquettes à la fois sur un film ?
```{admonition} Aide
:class: dropdown

Utilisez la question précédente, `WITH`, une deuxième clause `MATCH`, puis `NOT` et `IN`.
```
6. Recommandez de nouveaux partenaires de jeu à Tom Hanks. Toute personne ayant joué dans les mêmes films que des partenaires de Tom Hanks (co-acteurs) 
mais n'ayant jamais joué aux côtés de Tom Hanks lui-même peut convenir. Autrement dit, nous cherchons des co-co-acteurs de Tom Hanks qui n'ont jamais été 
co-acteurs de Tom Hanks.    
```{admonition} Aide
:class: dropdown

Utilisez `NOT` et un motif de graphe. Voir 3.2.6.
```
