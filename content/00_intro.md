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

# Neo4j : Base de données NoSQL orientée graphe

Ce cours est une introduction à Neo4J, une base de données NoSQL orientée graphe.

Il est organisé selon les quatre grandes catégories d'interaction que l'on peut avoir avec une base de données, résumées par l'acronyme **CRUD** :
- **C**reate (Créer) : Toute opération qui consiste à ajouter de nouveaux enregistrements à la base de données.
- **R**ead (Lire) : Toute opération qui consiste à récupérer certaines informations en fonction des critères de recherche spécifiés.
- **U**pdate (Mettre à jour) : Toute opération qui consiste à modifier des enregistrements de la base de données.
- **D**elete (Supprimer) : Toute opération qui consiste à supprimer des enregistrements de la base de données, voire la base de données elle-même.
  
Les données sont organisées en **noeuds** et en **relations**.   
- Un **noeud** représente un objet concret (ex: une personne, un produit). C'est l'équivalent d'une ligne dans une base de données relationnelle.      
- Une **relation** représente un lien entre deux objets (ex : "est ami avec", "a acheté", "travaille à"). 
Les relations sont comparables aux jointures dans les bases de données relationnelles. 
Cependant, dans les bases de données relaitonnelles, les jointures sont calculées *au moment du requêtage*.
A contrario, dans les bases de données NoSQL orientées graphe, les relations sont *stockées explicitement*.   
C'est la raison pour laquelle ces bases de données offrent de meilleures performances en lecture lorsque les données sont fortement interconnectées.  
- Un noeud peut avoir zéro, un ou plusieurs **labels**. Les labels sont comparables aux tables dans les bases de données relationnelles : 
ils permettent de regrouper sous une même étiquette les noeuds qui présentent des similarités (ex: étudiants, clients).   
- Une relation a une direction, un noeud de départ, un noeud d'arrivée, et un **type**. Une relation ne peut avoir qu'un seul type.  
- Les noeuds comme les relations peuvent avoir des **propriétés**. Les propriétés prennent la forme de **paires clé-valeur**.  

Neo4j a un **schéma flexible** :   
- Deux noeuds ou deux relations qui possèdent un même label n'ont pas nécessairement les mêmes propriétés
- Des **index** et des **contraintes** peuvent être introduits au fur et à mesure pour améliorer les performances ou la modélisation


```{figure} ../image/graph_example.svg  
---
---
Exemple from: [source](https://neo4j.com/docs/cypher-manual/current/clauses/match/)
```
   
