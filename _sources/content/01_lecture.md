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

(content:lecture)=
# C**R**UD : Lecture

::::{grid}
:gutter: 1

:::{grid-item-card} Objectif
Dans ce chapitre vous apprendrez à interroger et filtrer une base de données Neo4j.  
:::
::::

**Cypher** est le langage déclaratif de type SQL utilisé dans Neo4j. 
Il est relativement intuitif car le format des requêtes s'inspire de la façon dont on représente habituellement les graphes : des cercles pour les noeuds et des flèches pour les relations. 
Les parenthèses sont une représentation des cercles.         

::::{grid}
:gutter: 2

:::{grid-item-card} Noeud seul 
( n )
:::

:::{grid-item-card} Noeud avec label
( n : Label ) 
:::

::::

::::{grid}
:gutter: 2

:::{grid-item-card} Noeud avec propriétés
( n { clé : valeur, ... } )
:::

:::{grid-item-card} Noeud avec label et propriétés   
( n : Label { clé : valeur ... } )
:::

::::

::::{grid}
:gutter: 2

:::{grid-item-card} Relation orientée sans type
( n1 ) - - > ( n2 )   
( n1 ) < - - ( n2 )   
:::

:::{grid-item-card} Relation non orientée sans type
( n1 ) - - ( n2 )    
:::

::::

::::{grid}
:gutter: 1

:::{grid-item-card} Relation avec type
( n1 ) - [ : RELATION_TYPE] -> ( n2 )   
( n1 ) <- [ : RELATION_TYPE] - ( n2 )  
( n1 ) - [ : RELATION_TYPE] - ( n2 )    
:::

::::

::::{grid}
:gutter: 1

:::{grid-item-card} Relation avec type et propriétés
( n1 ) - [ : RELATION_TYPE { clé : valeur, ... }] -> ( n2 )   
( n1 ) <- [ : RELATION_TYPE { clé : valeur, ... }] - ( n2 )  
( n1 ) - [ : RELATION_TYPE { clé : valeur, ... }] - ( n2 )    
:::

::::

::::{grid}
:gutter: 1

:::{grid-item-card} Chemins
( n1 ) - - > ( n2 ) < - - ( n3 )     
( n1 ) - [ r1 : RELATION_TYPE] -> ( n2 ) <- [ r2 : RELATION_TYPE] - ( n3 )
:::

::::

