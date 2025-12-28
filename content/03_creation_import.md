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

# À partir d'un fichier

## Importer des données stockées dans un fichier CSV

`LOAD CSV` permet d'**importer des données stockées dans un fichier CSV ou dérivé** (TXT etc.). La syntaxe est la suivante (les retours à la ligne ne sont pas obligatoires) :
```
LOAD CSV [WITH HEADERS] 
FROM <path_or_url> [AS <alias>] 
[FIELDTERMINATOR <delimiter>]
```
où :
* `WITH HEADERS` (optionnel) : 
    * si spécifié, la **première ligne** du fichier est traitée comme un **en-tête**, 
et **chaque ligne** est transformée en **paires clé-valeur** où les clés sont déduites de l'en-tête
    * si non spécifié, **chaque ligne** est importée sous forme de **liste**
* `<path_or_url>` : chemin d'accès ou URL permettant d'accéder au fichier
* `AS <alias>` (optionnel) : **nom de la variable** qui pointera vers les lignes du fichier l'une après l'autre
* `FIELDTERMINATOR <delimiter>` (optionnel) : **délimiteur**, i.e par défaut la virgule, le caractère spécifié sinon

Si Neo4j est déployé localement (_on-premise_, ou "sur site", par opposition à une utilisation _cloud_)
et que vous souhaitez accéder à un fichier stocké localement, vous devez :
1. Placer le fichier dans le **répertoire _import_ de Neo4j**, dont l'emplacement varie en fonction du système d'exploitation (voir la [documentation](https://neo4j.com/docs/operations-manual/2025.11/configuration/file-locations/#neo4j-import))
2. Ajouter le **préfixe `file:///`** avant le nom du fichier.

````{admonition} Example
:class: tip

Supprimons les données déjà existantes :
```
MATCH (n)
DETACH DELETE n
```
Importons les données stockées dans le fichier `people.csv` disponible en ligne :
```
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing-cypher/people.csv' AS row
RETURN row
```
````
Si le fichier a un **en-tête**, une fois importés les différentes champs sont accessibles via la syntaxe : **`<alias>.<key>`**.
Sinon, il faut connaître l'indice de l'élément auquel vous souhaitez accéder dans la **liste** importée et utiliser la syntaxe : **`<alias>[<index>]`**. 

Vous pouvez également ne charger qu'un sous-ensemble des données contenues dans le fichier en utilisant `WITH` et les clauses ou sous-clauses vues aux chapitres précédents.

````{admonition} Example
:class: tip

Importons uniquement les informations relatives aux personnes nées en 1942 :
```
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing-cypher/people.csv' AS row
WITH row WHERE row.birthYear = '1942'
RETURN row
```
Notez que dans la requête, `1942` est entre guillemets. L'explication est donnée en suivant.
````

`LOAD_CSV` lit toutes les données sous forme de **chaînes de caractères**. Vous devez utiliser les fonctions `toInteger()`, `toFloat()`, `toBoolean()`, `date()`, `datetime()`
ou des fonctions smilaires pour convertir les données au type approprié (voir la [documentation](https://neo4j.com/docs/cypher-manual/current/values-and-types/)).

````{admonition} Example
:class: tip

Convertissons les identifiants en entiers et les années de naissance en dates :
```
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing-cypher/people.csv' AS row
WITH toInteger(row.personId) AS personId, date(row.birthYear) AS birthYearDate, row.birthYear AS birthYear
RETURN personId, birthYearDate, birthYear
```
**Attention :** Notez qu'à l'année de naissance a été ajoutée le premier jour du mois de janvier afin de respecter le format `AAAA-MM-JJ`.
Le choix du format `date` n'est donc pas toujours le plus adapté.
````
Pour vérifier que les données ont été correctement importées, vous pouvez **afficher le nombre de lignes** et les **clés des paires** clé-valeur ainsi créées (si le fichier comportait un en-tête), 
puis les comparer aux informations dont vous disposez à propos de la base de données. 
::::{grid}
:gutter: 1

:::{grid-item-card} Exercice
1. Importez le fichier `person.csv` et retournez le nombre de lignes
2. Importez le fichier `person.csv` et retournez les clés associées à chaque ligne    

**Bonus :** Importez le fichier `person.csv` et retournez les clés distinctes
:::

::::

À ce stade, les données sont importées mais elles ne sont pas encore transformées en graphe.

## Convertir un fichier CSV en graphe

Pour créer un graphe à partir des données importées, il faut faire suivre l'import de clauses `MATCH`, `CREATE` (chapitre 3.1), `MERGE` (chapitre 3.1) et / ou `SET` (chapitre 4).   

Pour des questions de performance, il est recommandé d'exécuter plusieurs fois `LOAD CSV` de façon à **créer les noeuds et les relations séparément**.   

Il est également recommandé de **spécifier les contraintes d'unicité avant tout import** (chapitre 3.1).

````{admonition} Example
:class: tip

Le fichier `books.csv` contient des informations sur les auteurs et le contenu des livres. 
Ces deux catégories d'information peuvent être stockées sous forme de noeuds, les uns labellisés `Author`, les autres labellisés `Book`, 
reliés par une relation de type `WROTE`.

Commençons par créer une contrainte d'existence et d'unicité pour l'identifiant des livres, ainsi qu'une contrainte d'existence pour le nom des auteurs et le titre des livres. 
Si vous utilisez la version gratuite de Neo4j, n'exécutez que la contrainte d'unicité.
```
CREATE CONSTRAINT FOR (b:Book) REQUIRE b.id IS NOT NULL;
CREATE CONSTRAINT FOR (b:Book) REQUIRE b.id IS UNIQUE;
CREATE CONSTRAINT FOR (b:Book) REQUIRE b.title IS NOT NULL;
CREATE CONSTRAINT FOR (a:Author) REQUIRE a.name IS NOT NULL
```
Créons les noeuds de label `Author` et `Book` :
```
LOAD CSV WITH HEADERS 
FROM 'file:///books.csv' AS row
MERGE (:Author{name:row.author})
MERGE (:Book{id:row.id, title:row.title, year:row.publication_year})
```
Notez que nous n'avons importé qu'une partie des données contenues dans le fichier.   

Créons les relations :
```
LOAD CSV WITH HEADERS 
FROM 'file:///books.csv' AS row
MATCH (a:Author{name:row.author})
MATCH (b:Book{id:row.id})
MERGE (a)-[:WROTE]->(b)
```
Le fichier `book.csv` comporte aussi une colonne `genre` qui contient le ou les genres de chaque livre. 
Lorsque plusieurs genres sont spécifiés, ils sont séparés par le symbole `|`. 
Nous pouvons créer un noeud pour chacun des genres et lier les livres aux bons genres via une relation de type `CLASSIFIED_AS`. 
Comprenez-vous ce que fait chacune des étapes ci-dessous ?   
Première étape :
```
LOAD CSV WITH HEADERS
FROM 'file:///books.csv' AS row
MATCH (b:Book{id:row.id})
SET b.genre = split(row.genre, '|')
```
Deuxième étape :  
```
MATCH (b:Book)
UNWIND b.genre AS genres
WITH DISTINCT genres AS genre
MATCH (b:Book)
WHERE genre IN b.genre
MERGE (g:$(genre))
MERGE (b)-[:CLASSIFIED_AS]->(g)
```
Troisième étape :  
```
MATCH (b:Book)
REMOVE b.genre
```
Rajoutons la propriété `inPrint` aux noeuds `Book`. Notez que ce champ est parfois vide dans le fichier, avec ou sans espace. 
Les bases de données NoSQL permettent de ne pas stocker inutilement de valeurs `null`. 
Vous pouvez soit les ignorer à l'import, soit les remplacer par des valeurs par défaut. 
Pour ignorer les champs vides, avec ou sans espace :
```
LOAD CSV WITH HEADERS
FROM 'file:///books.csv' AS row 
WITH row WHERE row.still_in_print IS NOT NULL
MATCH (b:Book{id:row.id})
SET b.inPrint = nullIf(trim(row.still_in_print), "")
```
Pour convertir en booléens les valeurs : 
```
MATCH (b:Book)
SET b.inPrint = toBoolean(b.inPrint)
```
````
Vous savez désormais comment transformer concrètement des données importées en graphe. 
Cependant, avant tout import, il vous faut impérativement **réfléchir à la façon dont vous allez transformer la ou les tables en graphes**. 

Dans l'exemple précédent, nous avons par exemple décidé que les genres littéraires seraient stockés dans des noeuds à part, 
un choix de conception qui permet un **accès rapide** à l'information. 
Ce choix se justifie si, à l'usage, les requêtes pour identifier tous les livres associés à un genre particulier sont fréquentes. 
Dans ce cas, laisser l'information du genre littéraire dans les propriétés des noeuds `Book` aurait impliqué un parcours systématique de l'ensemble des
noeuds pour identifier ceux du genre recherché, ce qui aurait été très coûteux en temps. 
En déplaçant l'information au niveau des noeuds, une requête sur un genre littéraire particulier ne consiste plus qu'à parcourir les relations entrantes de type 
`CLASSIFIED_AS` qui pointent vers le noeud `Genre` approprié.

L'objectif du chapitre 3.3 est de vous aider à concevoir la structure de votre graphe en amont de toute implémentation. 

