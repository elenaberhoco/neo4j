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

source : [documentation Neo4j](https://neo4j.com/docs/cypher-manual/current/clauses/match/)

Dans Cypher, le comportement d'une requête est défini par ses **clauses**, i.e les différentes étapes qui la composent. 
Chaque clause prend l'**état actuel du graphe** et un tableau de **résultats intermédiaires**, les traite, 
puis transmet l'**état mis à jour** du graphe et les **résultats** à la **clause suivante**. 

La première clause d'une instruction Cypher est généralement `MATCH`. 
Elle permet de **spécifier les motifs à rechercher dans le graphe** de données. La syntaxe générale est la suivante :  
```sql
MATCH ...   
... 
RETURN ...  
``` 
  
`RETURN` permet de définir les noeuds, les relations et les propriétés à inclure dans le **résultat de la requête**.


````{admonition} Example
:class: tip

Nous allons utiliser l'un des jeux de données mis à disposition par Neo4j. 
Rendez-vous sur https://sandbox.neo4j.com/. Créez un compte. 
Créez un nouveau projet et cliquez sur la tuile "Movies" dans la section "Pre-built data".   
````

## Requête sur les noeuds 

Dans Neo4j, vous pouvez effectuer des requêtes à partir des **labels** et des **propriétés** des **noeuds**. 
La syntaxe est la suivante :  
```
MATCH (<node>:<Label>{<key>:<value>,...})                           
RETURN ...  
```
où :
- `<node>` : nom de la variable qui pointera vers les noeuds sélectionnés, si besoin 
- `<Label>` : label 
- `{<key>:<value>,...}` : propriétés 

Il est nécessaire de **lier** les **noeuds sélectionnés** à une **variable** si vous souhaitez **y accéder dans les clauses suivantes**, et notamment dans `RETURN`. 
Les noms des variables peuvent être des lettres ou des mots isolés, et doivent être écrites en **minuscules**.        

Il est possible d'**accéder aux propriétés d'un noeud** via la syntaxe : `<node>.<key>`. 
Cette syntaxe est utile lorsque vous ne souhaitez retourner que certaines informations. 
Utilisez une **virgule** pour lister toutes les informations que vous souhaitez renvoyer avec `RETURN`. 
Vous pouvez utiliser `AS` pour formatter le résultat dans `RETURN`.

Si **aucune condition** sur les labels ou les propriétés n'est spécifiée, tous les noeuds du graphe sont retournés :  
```sql
MATCH (n)
RETURN n
````
Vous pouvez filtrer les noeuds à partir de leurs **labels**. La syntaxe est la suivante : `(<node>:<Label>)`.   
Vous pouvez spécifier **plusieurs labels** à l'aide des opérateurs suivants :
- `|` qui correspond au **OU** logique, syntaxe : `(<node>:<Label1>|<Label2>...)` 
- `&` qui correspond au **ET** logique, syntaxe : `(<node>:<Label1>&<Label2>...` 
  
Il est possible de **combiner les deux**, en écrivant par exemple : `(<node>:(<Label1>&<Label2>)|<Label3>)`.     
Vous pouvez spécifier les **labels à exclure** à l'aide de `!`. La syntaxe est la suivante : `(<node>:!<Label>)`.    


````{admonition} Example
:class: tip

**Exemple n°1**: Pour retourner le titre (`title`) de tous les films (`Movie`) du jeu de données, tapez : 
```sql
MATCH (movie:Movie)
RETURN movie.title
```
**Exemple n°2**: Pour retourner à la fois le titre de tous les films et le nom (`name`) de toutes les personnes (`Person`) du jeu de données, puis renommer les colonnes du tableau renvoyé, tapez :
```sql
MATCH (n:Movie|Person)
RETURN n.title AS title, n.name AS name
```
**Exemple n°3**: Pour exclure les films du résultat, tapez : 
```sql
MATCH (n:!Movie)
RETURN n
```
````

Vous pouvez filtrer les noeuds à partir de leurs labels et de leurs **propriétés**. La syntaxe est la suivante : `(<node>:<Label>{<key>:<value>,...})`
````{admonition} Example
:class: tip

La commande pour retourner toutes les personnes qui se nomment `Tom Hanks` est la suivante :  
```
MATCH (p:Person{name:'Tom Hanks'})
RETURN p
```
````

## Requête sur les relations

Dans Neo4j, vous pouvez rechercher des **motifs** dans le graphe. Les motifs sont une **combinaison de noeuds et de relations**. 
Toute requête impliquant une **relation** doit **obligatoirement** rattacher la relation à deux noeuds : un **noeud de départ** et un **noeud d'arrivée**.    
La syntaxe générale est la suivante (l'orientation de la flèche est purement illustrative) :
```
MATCH (<start_node>)-[<relationship>:<TYPE>{<key>:<value>,...}]->(<end_node>)
RETURN ...
```
où :
- `<start_node>`, `<end_node>` : noeuds tels que définis à la section précédente 
- `<relationship>` : nom de la variable qui pointera vers les relations sélectionnées, si besoin
- `<TYPE>` : type de la relation
- `{<key>:<value>,...}` : propriétés

Comme pour les noeuds, si vous souhaitez **accéder aux relations sélectionnées** par votre requête **dans les clauses suivantes**, 
il est nécessaire de les lier à une **variable**.    

Il est possible d'accéder aux **propriétés** d'une relation via la syntaxe : `<relationship>.<key>`. 
Vous pouvez ne spécifier **aucune direction** pour la ou les relations en écrivant : `--`. 
La **direction** d'une relation peut être spécifiée à l'aide d'une flèche : `-->` ou `<--`. 
Vous pouvez utiliser `AS` pour formatter le résultat. Vous pouvez **spécifier plusieurs types** simultanément à l'aide de l'opérateur `|` (OU logique). 
Vous pouvez spécifier les **types à exclure** à l'aide de `!`.   

Si **aucune condition** sur le type ou les propriétés de la relation n'est spécifiée, **toutes les relations connectées à un ou plusieurs noeuds** sont retournées :
````{admonition} Example
:class: tip

**Exemple n°1**: Pour renvoyer tous les noeuds connectés au noeud de la réalisatrice `Lana Wachowski` quelque soit la direction de la relation, tapez : 
```
MATCH (:Person {name:'Lana Wachowski'})--(n)
RETURN n AS connectedNodes
```
Notez que nous n'avons pas défini de variable pour la relation ni pour le noeud de la réalisatrice, car seul les noeuds reliés au noeud de `Lana Wachowski` nous intéressaient.  

**Exemple n°2**: Pour renvoyer simultanément le noeud de la réalisatrive `Lana Wachowski`, toutes les relations qui partent ou pointent vers ce noeud, ainsi que tous les noeuds connectés, tapez :  
```
MATCH (p:Person {name:'Lana Wachowski'})-[r]-(n)
RETURN p, r, n
```
**Exemple n°3**: Pour trouver tous les noeuds labellisés `Movie` connectés à `Lana Wachowski` par une relation sortante, vous pouvez tapez :
```
MATCH (:Person {name:'Lana Wachowski'})-->(movie:Movie)
RETURN movie.title AS movieTitle
```
````
Vous pouvez également affiner la rechercher en spécifiant un **type** et des **propriétés** pour les relations. La syntaxe est la suivante : `[<relationship>:<TYPE>{<key>:<value>,...}]`.  
````{admonition} Example
:class: tip

**Exemple n°1** : La commande suivante permet d'extraire le nom des acteurs qui ont joué (`ACTED_IN`) dans le film `Charlie Wilson's War` :
```
MATCH (:Movie {title: "Charlie Wilson's War"})<-[:ACTED_IN]-(actor:Person)
RETURN actor.name AS actor
```
**Exemple n°2** : Pour retourner le nom de l'acteur qui a joué le personnage `Kevin Lomax` (`role`) dans un film, ainsi que le titre du film, tapez : 
```
MATCH (a:Person)-[:ACTED_IN {roles:'Kevin Lomax'}]->(b:Movie)
RETURN a.name AS name, b.title AS title
```
Cette commande ne fonctionnera pas avec la base de données utilisée dans ce chapitre, car `role` est une liste. 
La bonne façon de procéder est celle présentée dans le bloc suivant, qui utilise `WHERE` et `IN`, clauses qui seront introduites dans le chapitre 3.2.
````
:::{caution}
Exécutez la commande suivante :  
```
MATCH (a)-[r:ACTED_IN]-(b)
WHERE 'Kevin Lomax' IN r.roles
RETURN a, b
```
Que remarquez-vous ? Ne pas spécifier de direction dégrade les performances : Neo4j doit traverser le graphe dans les deux sens. 
De façon générale, soyez le plus précis possible dans vos requêtes pour améliorer les performances.
:::

## Requête à partir de motifs complexes

Il est possible de construire des **motifs plus complexes** dans le graphe de données : vous pouvez inclure **plusieurs relations** et **plusieurs noeuds** dans le motif de requête.  

````{admonition} Example
:class: tip

**Exemple n°1** : Dans quel(s) film(s) réalisé(s) (`DIRECTED`) par `Vincent Ward` l'acteur `Max von Sydow` a-t-il joué ?
```
MATCH (:Person {name:'Max von Sydow'})-[:ACTED_IN]->(movie:Movie)<-[:DIRECTED]-(:Person {name:'Vincent Ward'})
RETURN movie.title AS title
```
**Exemple n°2** : Avec quel(s) réalisateur(s) l'acteur `Max von Sydow` a-t-il travaillé ?  
```
MATCH (:Person {name:'Max von Sydow'})-[:ACTED_IN]->(movie:Movie)<-[:DIRECTED]-(director:Person)
RETURN movie.title AS title, director.name AS director                                
```
````

Vous pouvez aussi réaliser des requêtes en posant des conditions sur la longueur du chemin reliant deux noeuds. 
La longueur d'un chemin correspond au nombre d'arêtes qu'il faut emprunter pour aller d'une extrémité du chemin à l'autre :
- `[*]` : chemin de longueur quelconque  
- `[*<number>]` : chemin de longueur `<number>`  
- `[*<number1>..<number2>]` : chemin dont la longueur est comprise entre `<number1>` et `<number2>` ; si `<number1>` n'est pas spécifié, la valeur par défaut est 0, si `<number2>` n'est pas spécifié la valeurs par défaut est l'infini   

````{admonition} Example
:class: tip

Quel est l'un des chemins les plus courts qui relie `Tom Hanks` et `Robin Williams` ? Pour répondre à cette question vous pouvez utiliser `SHORTEST`, en spécifiant `1` pour ne retourner que l'un des chemins :   
```
MATCH path = SHORTEST 1 ((:Person{name:"Tom Hanks"})-[*]-(:Person{name:"Robin Williams"}))
RETURN path
```
````
## Requête à plusieurs motifs

Il est possible de sélectionner des noeuds, des relations ou une partie du graphe en fonction d'un **ensemble de motifs**, plutôt que d'un motif unique. 
Les premiers exemples de la section précédente peuvent d'ailleurs être réécrits comme une requête à plusieurs motifs (voir ci-dessous). 
Ces motifs peuvent être définis les uns après les autres au sein d'une même clause `MATCH`--les virgules jouent alors le rôle de ET logique--ou dans plusieurs clauses `MATCH` successives. 
Chacun des motifs doit faire appel à au moins l'une des variables définies au sein des motifs précédents. 


````{admonition} Example
:class: tip

D'après vous, que fait cette requête ?
```
MATCH (:Movie{title:"Ninja Assassin"})<-[:ACTED_IN]-(p:Person)-[:ACTED_IN]->(:Movie{title:"Speed Racer"})
RETURN DISTINCT p.name as actor
```
On peut également écrire cette requête ainsi :
```
MATCH (p:Person)-[:ACTED_IN]->(:Movie{title:"Speed Racer"}), (p)-[:ACTED_IN]->(:Movie{title:"Ninja Assassin"})
RETURN DISTINCT p.name as actor
```
Ou encore ainsi :
```
MATCH (p:Person)-[:ACTED_IN]->(:Movie{title:"Speed Racer"})
MATCH (p)-[:ACTED_IN]->(:Movie{title:"Ninja Assassin"})
RETURN DISTINCT p.name as actor
```
````

## Quelques fonctions utiles   

Neo4j met à disposition un certain nombre de fonctions. Voir la [documentation](https://neo4j.com/docs/cypher-manual/current/functions/).   

Les fonctions suivantes fournissent des informations utiles concernant le graphe :  

| Graphe | Description |
|:---:|:---|
| `labels()` |  Récupérer sous forme de liste tous les labels des noeuds sélectionnés  |  
| `keys()` | Récupérer toutes les clés de propriété des noeuds ou relations sélectionnés |
| `type()` | Récupérer le type d'une ou plusieurs relations | 

Les fonctions suivantes sont des fonctions d'agrégation :  

| Agrégation | Description |
|:---:|:---|
| `avg()` | Calculer la moyenne d'un ensemble de valeurs |
| `max()` | Calculer la valeur maximale d'un ensemble de valeurs |
| `min()` | Calculer la valeur minimale d'un ensemble de valeurs |
| `sum()` | Calculer la somme d'un ensemble de valeurs |  
| `collect()` | Récupérer sous forme de liste les valeurs retournées par une expression |
| `count()` | Compter le nombre d'éléments. `count(*)` pour le nombre total, `count(<expression>)` pour le nombre d'éléments hors valeurs manquantes, `count(DISTINCT <expression>)` pour le nombre d'éléments distincts | 

Faire précéder les fonctions d'agrégation par d'autres expressions revient à **grouper les éléments** en fonction de ces expressions, 
puis à **appliquer les fonctions d'agrégation à chacun des groupes**.  

Autre | Description    
:---:|:---
`DISTINCT` | Récupérer les valeurs distinctes  
`size()` | taille d'une liste ou d'une chaîne de caractères 

````{admonition} Example
:class: tip

**Exemple n°1** : Pour compter le nombre de noeuds dont l'un des labels est `Person`, tapez : 
```sql
MATCH (:Person)
RETURN count(*)
```
**Exemple n°2** : Pour afficher les différents types de relation et le nombre de relations par type : 
```sql
MATCH ()-[r]-()
RETURN type(r) AS relationType, count(r) AS count
```
Dans cet exemple, l'instruction `RETURN` est équivalente à un `GROUP BY` en SQL : 
on commence à grouper les relations par type puis on compte le nombre de relations dans chaque groupe.     
**Exemple n°3** : Pour lister les noms de tous les acteurs, réalisateurs et producteurs du film `That Thing You Do` : 
```
MATCH (p:Person)-[:ACTED_IN|DIRECTED|PRODUCED]->(:Movie {title:'That Thing You Do'})
RETURN collect(p.name)
``` 
Que remarquez-vous ? Pour remédier à ce problème, vous pouvez utiliser `DISTINCT`:
```
MATCH (p:Person)-[:ACTED_IN|DIRECTED|PRODUCED]->(:Movie {title:'That Thing You Do'})
RETURN collect(DISTINCT p.name)
```
````

Vous pouvez également utiliser les opérateurs classiques : `+`, `-`, `*`, `/`, `%` et `^`. 
Vous pouvez concaténer des listes ou des chaînes de caractères avec `||` ou `+`.

## Exercice

1. Affichez tout le graphe  
2. En quelle année est né (`born`) l'acteur `Bill Paxton` ?
3. Quelle est la phrase d'accroche (`tagline`) et la date de parution (`released`) du film `Sleepless in Seattle` ?
4. Dans quel(s) film(s) `Jack Nicholson` a-t-il joué ? Affichez le résultat sous forme de liste.
5. Combien de films `Ron Howard` a-t-il dirigé ? 
6. Affichez le sous-graphe constitué des films qui ont reçu une note, des utilisateurs qui les ont notés et des relations qui lient ces noeuds entre eux.  
7. Quels sont les propriétés des relations `REVIEWED` ? N'affichez que les valeurs uniques.
```{admonition} Aide          
:class: dropdown

Utilisez `keys()` et `DISTINCT`. `DISTINCT` peut s'utiliser en dehors d'une fonction, voir la documentation.
```
8. Quelle moyenne le film `The Replacements` a-t-il obtenu ?
```{admonition} Aide
:class: dropdown

Utilisez la propriété `rating` des relations `REVIEWED` et la propriété `title` des noeuds `Movie`. Utilisez la fonction `avg()`.
```   
9. Avec quel(s) acteur(s) l'actrice `Carrie-Anne Moss` a-t-elle déjà joué ?  
```{admonition} Aide
:class: dropdown

Voir 3.1.3.
```
10. Qui a déjà à la fois réalisé un film et joué dans un film au cours de sa carrière ?
```{admonition} Aide
:class: dropdown

Voir 3.1.4.
```
11. Afficher le nombre d'acteurs par date de naissance.
```{admonition} Aide
:class: dropdown

Voir 3.1.5. Équivalent d'un `GROUP BY`.
```


```{admonition} Bonus
:class: dropdown

12. Quels acteurs ont joué avec des co-acteurs de `Natalie Portman` ?   
  
**Note** : Aucun des co-co-acteurs de Natalie Portman n'a joué avec elle dans un film. 
Autrement dit : personne n'est à la fois co-acteur de Natalie Portman sur un film et co-co-acteur de Natalie Portman sur un autre film.

```
  
    
