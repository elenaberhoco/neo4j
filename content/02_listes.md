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

# Titre 2

Mais les cellules peuvent échouer à cause du kernel python :
```{code-cell}
use etudiants
```
Dommage !

IMPORTANT : 

Problème : la requête nous renvoie également les étudiants qui n'ont pas eu de note. C'est logique : si Sophie n'a pas de note, on ne peut pas dire qu'elle ait déjà eu moins que 12. Nous allons donc retirer les étudiants sans notes.

Pour se faire, nous utilisons l'opérateur logique `$nor` en listant les éléments à ne pas prendre en compte. Nous ne voulons pas que la liste `notes` soit vide ou qu'elle comporte ne serait-ce qu'une note inférieure à 12.

```{code-cell}
db.notes.find(
    {$nor: 
        [{"notes": {$lt: 12}},               /*1ère condition : on retire ceux qui ont des notes en dessous de 12*/
        {"notes": {$size: 0}}]               /*2nde condition : on retire ceux qui n'ont pas de notes*/
    }
)
```
Nouveau problème : le document correspondant à l'étudiant Michel est renvoyé parce qu'il n'a pas d'attribut `notes`. Dans ces conditions, on remarque que les listes vides ou inexistantes sont retournées par les requêtes. Il est important de les enlever en rajoutant une condition dans le `$nor`.

Il faut donc retirer les étudiants qui ont une liste `notes` vide mais aussi ceux qui n'ont pas de liste du tout.

```{code-cell}
db.notes.find(
    {$nor: 
        [{"notes": {$exists: false}},         /*1ère condition : on retire les documents ne possédant pas de liste "notes"*/
        {"notes": {$lt: 12}},                 /*2ème condition : on retire ceux qui ont des notes en dessous de 12*/
        {"notes": {$size: 0}}]                /*3ème condition : on retire ceux qui n'ont pas de notes*/
    }
)
```
Cette fois, on ne retourne plus que 2 étudiants qui n'ont que des notes au-dessus de 12.

## Recap

Sympa les bloc gris aussi :

- _Je souhaite que **toutes les conditions** soient vérifiées **au moins une fois** par les éléments de ma liste, comment faire ?_
> Code "classique".

Sans `$elemMatch`, si les conditions sont vérifiées une à une, que ce soit par un élément de la liste ou grâce à plusieurs éléments distincts, alors le document est retourné.

- _Je souhaite que **toutes les conditions** soient **simultanement** vérifiées par **au moins un élément** de ma liste, comment faire ?_
> Utilisation de l'opérateur `$elemMtach`.

Avec `$elemMatch`, on regarde tous les éléments de la liste un par un et on retourne le document si et seulement si au moins un élément est capable de vérifier toutes les conditions à lui tout seul.

- _Je souhaite que **toutes les conditions** soient vérifiées par **tous les éléments** de ma liste, comment faire ?_
> Utilisation de l'opérateur `$nor`.

Avec `$nor`, on liste les conditions que nous ne souhaitons pas retourner. Ainsi, on ne récupère pas les éléments qui valident des conditions. Il faut notamment penser à retirer les éléments vides avec `{$size: 0}` et les éléments inexistants avec `{$exists: false}`.
