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

# Titre 5

INTÉRESSANT : 

Pour celui-ci : remplacé par updateMany ?

### Modification de plusieurs documents
Pour modifier plusieurs documents à la fois, il est nécessaire d'ajouter `{multi: true}` en fin de requête.
```{code-cell}
db.ventes.update(
	{"nom" : {$in: ["C1", "C2"]}},
	{$set: {"pays": "France"}},
	{multi: true}
)
```
Cette requête, par exemple, ajoute un attribut "pays" ayant la valeur "France" aux modèles C1 **et** C2.

PAS VU :

### Upsert
L'option `upsert` (mélange de "update" et "insert") permet de mettre une condition sur la requête : si aucun document ne correspond aux conditions indiquées en première ligne, alors un nouveau document est créé par les champs renseignés sur la seconde ligne.
```{code-cell}
db.ventes.update(
	{"nom": "C1"},
	{$set : {"nom": "C1", "Nombre de roues": 4}},
	{upsert: true}
)
```
Ici, on ajoute une nouvelle variable "Nombre de roues" à laquelle on attribue la valeur 4 au modèle "C1".

```{code-cell}
db.ventes.update(
	{"nom": "Twingo"},
	{$set : {"Nombre de roues": 4}},
	{upsert: true}
)
```
Cette fois-ci, un nouveau document est ajouté à la base.

PAS VU : 

### Suppression de documents dans une collection
Nous voilà arrivés au gros morceau...
Lorsque l'on veut supprimer certains documents en particulier **sans toucher aux index**, il faut utiliser la commande suivante :
```js
db.nomDeLaCollection.remove({})
```
Lorsque l'on passe en argument un document vide, comme dans l'exemple ci-dessus, on supprime toutes les données contenues dans la collection, mais on en conserve la structure, donc les index.

La fonction _remove_ peut également recevoir des documents précis en argument :
* Condition sous la forme d'un document masque :  
   Tous les documents correspondants à la sélection seront supprimés, par exemple tous ceux dont l'attribut "marque" correspond à "Citroën" :
```{code-cell}
db.ventes.remove({"marque" : "Citroën"})
```
* Suppression d'un seul document :  
   Pour ce faire, il convient d'utiliser l'attribut "_id" puisqu'il est unique :
```js
db.nomDeLaCollection.remove({"_id" : ObjectId("5612c6c0a5c56580cfacc342")})
``` 
