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

# Titre 4

ON PEUT RENOMMER :

Avec cette requête, on peut voir le quartier du restaurant. Par ailleurs, la variable `borough` 
a été renommée `quartier`. On peut également conserver cette 
variable sans la renommer avec cette syntaxe.
```{code-cell}
db.NYfood.aggregate( 
  [
    {$project: {"n_notes" : {$size : '$grades'}, borough : 1}}
  ]
)
```
