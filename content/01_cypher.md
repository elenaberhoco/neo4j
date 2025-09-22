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

# Langage Cypher

**Cypher** est le langage déclaratif de type SQL utilisé dans Neo4j.
Il est relativement intuitif car le format des requêtes **s'inspire de la façon dont on représente habituellement les graphes** : des **cercles** pour les noeuds et des **flèches** pour les relations.
Cypher utilise des **parenthèses** pour les **noeuds**, des **flèches** (ex : `-->`) et des **crochets** (`[`) pour les **relations**, et des **accolades** (`{`) pour les **propriétés**. Pour faire référence aux **labels** des noeuds et au **type** des relations, il faudra les précéder de **`:`**.  
Cypher fait les recommandations suivantes:  
- Labels : 
    - **C**amel**C**ase (ex : `SocialMedia`, `VehiculeOwner`)
    - Nom au singulier
    - Être aussi précis que possible (ex : préférez `Actor`, `Director` ou `User` à `Person`)
- Relations :
    - Majuscules et underscores (ex : `ACTED_IN`, `OWNS_VEHICLE`)
    - Verbe ou dérivé
    - Être aussi précis que possible (ex : évitez `CONNECTED_TO`)
- Clés des propriétés :
    - **c**amel**C**ase (ex : `firstName`, `address`) 

Exemples non exhaustifs de syntaxe :   

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
:gutter: 1

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


  
