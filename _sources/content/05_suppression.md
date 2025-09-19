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
Dans ce chapitre vous apprendrez à supprimer des documents, des collections et des bases de données MongoDB.
:::
::::

Pour supprimer une base de données, tapez :
> db.dropDatabase()


`drop()` permet de supprimer une collection. La syntaxe est la suivante :  
```
db.<collection>.drop()
```

`deleteOne()` permet de supprimer un document en fonction d'un ensemble de critères. La syntaxe est la suivante :  
```
db.<collection>.deleteOne(<filter>)
```
Pour supprimer plusieurs documents, utilisez `deleteMany()` :  
```
db.<collection>.deleteMany(<filter>)
```

## Exercice

1. Dans le jeu de données `movies.json`, supprimez tous les films dirigés par Luc Besson de la base de données
2. Dans le jeu de données `movies.json`, supprimez la collection `movies`
3. Supprimez toutes les bases de données utilisées dans ce cours
