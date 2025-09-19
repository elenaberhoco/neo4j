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

# **C**RUD : Création

::::{grid}
:gutter: 1
  
:::{grid-item-card} Objectif
Dans ce chapitre vous apprendrez à créer une base de données MongoDB, à y ajouter des collections, puis à y insérer des documents. Vous découvrirez aussi que MongoDB prend en charge divers types de données.  
:::

::::

MongoDB est une **base de données NoSQL orientée documents**. 

Les données sont stockées selon un système de paires clé-valeur dans des **documents** semi-structurés. Contrairement aux **bases de données clé-valeur**, la valeur est consultable : il est possible d'interroger le contenu d'un document et de le modifier---totalement ou partiellement---sans avoir besoin de récupérer l'intégralité du document à l'aide de sa clé. Dans MongoDB, les documents sont stockés au format BSON (Binary JSON). 

Une **collection** est constituée de documents qui présentent des similarités sur le plan structurel et / ou conceptuel. Une base de données peut contenir plusieurs collections. Les collections sont comparables aux tables dans les **bases de données relationnelles**, la différence étant que tous les documents d'une collection n'ont pas nécessairement les mêmes champs (le **schéma** est **flexible**). Les bases de données orientées documents peuvent également stocker une plus grande variété de données que les bases de données relationnelles. 

`````{admonition} Example  
:class: tip

La base de données de l'université Prestige contient deux collections : une collection `Étudiants` pour les étudiants de l'université, et une collection `Employés` pour le personnel de l'université. Chaque document de la collection `Étudiants` contient des informations sur un étudiant de l'université : nom, prénom, âge, numéro étudiant, formation etc. Les documents de la collection `Étudiants` peuvent contenir des champs différents, par exemple : 

::::{grid}
:gutter: 2

:::{grid-item-card} Étudiant boursier

```javascript
{
    "_id" : ObjectId("56011920de43611b917d773d"),
    "nom" : "Zola",
    "prenom" : "Émile",
    "sexe" : "M",
    "age" : 19,
    "bourse" : "échelon 2"
}
```
:::

:::{grid-item-card} Étudiante en alternance

```javascript
{
    "_id" : ObjectId("65511420al87311b100q246t"),
    "nom" : "Christie",
    "prenom" : "Agatha",
    "sexe" : "F",
    "age" : 21,
    "entreprise" : "Éditions du Masque",
    "contrat" : "alternance"
}
```
:::
::::

Notez qu'il n'y a pas d'attributs (ou *clés*) vides dans MongoDB : les *valeurs* nulles sont remplacées par l'absence des champs correspondant. Dans l'exemple ci-dessus : l'attribut `bourse` (resp. `entreprise`, `contrat`) n'apparait pas dans le document de l'étudiante en alternance (resp. étudiant boursier).

`````

## Démarrage

:::{caution}
Avant de pouvoir travailler dans mongosh, il faut démarrer une instance MongoDB.
````{tab} Windows
(à venir)
````
````{tab} Linux
```console
sudo systemctl start mongod
sudo systemctl status mongod
```
````
:::

**Dans un terminal** : Ouvrez un terminal.    
**Dans Visual Studio Code** : Cliquez sur l'icône MongoDB, puis sur `localhost:27017` pour établir la connection. Faites un clic droit sur `localhost:27017` et sélectionnez "Launch MongoDB Shell" (ou équivalent). Un terminal s'ouvre. Agrandissez le.

Pour **démarrez** une session mongosh, tapez :
- Déploiement local sur le port par défaut :
> mongosh
- Déploiement Université de Lorraine :
> mongosh *DNS_name* -u root -p *password*  

Pour **quitter** la session mongosh, tapez :
> exit

Pour **lister les bases de données** présentes sur le serveur, tapez :
> show dbs


## Créer une base de données
 
Pour **créer une base de données**, ou la **sélectionner** si elle existe déjà, tapez `use` suivi du nom de la base de données. En reprenant l'exemple précédent :

````{admonition} Example
:class: tip

```
use University
```

````

Une fois la base de données créée ou sélectionnée, la commande `db.createCollection(<collection>)` permet de **créer explicitement une collection**.

````{admonition} Example
:class: tip

```
db.createCollection("students")
```
````

La base de données `University` contient désormais la collection `students`. Pour s'en assurer, vous pouvez **afficher l'ensemble des collections** de la base de données grâce à la commande :
> show collections

```{admonition} Important
Il n'est pas nécessaire de créer une collection avant d'ajouter des données dans la base de données. La collection peut être créée à la volée au moment de l'insertion parce que MongoDB a un schéma flexible. 
```

Enfin, `mongoimport` permet de **créer une base de données** et / ou une collection **à partir d'un fichier JSON**, CSV ou TSV. Pour importer un fichier JSON comprenant plusieurs documents, la syntaxe est la suivante :
```
sudo mongoimport --db <database> --collection <collection> --file <file.json> --jsonArray
```

Si le fichier ne comporte qu'un seul document, retirez l'option `--jsonArray`. Si la base de données (resp. la collection) existe déjà, les documents seront ajoutés à la base de données (resp. collection). Pour les autres formats, voir [ici](https://www.mongodb.com/developer/products/mongodb/mongoimport-guide/).

:::{warning}
`mongoimport` ne s'utilise pas depuis `mongosh` mais depuis le terminal.
:::

## Ajouter des données

Pour **insérer un document** JSON dans une collection et créer la collection si elle n'existe pas, utilisez `db.<collection>.insertOne(<document>)`. Remplacez les termes entre chevrons (`<` et `>`) par ce qui convient.

````{admonition} Example
:class: tip
```
db.students.insertOne({
                       "surname":"Zola",
                       "firstname":"Émile",
                       "gender":"M",
                       "age":19,
                       "bourse":2
                      })
``` 
````
Vous pouvez vérifier que le document a correctement été ajouté à la collection en exécutant la commande `db.<collection>.countDocuments()`, qui renvoie le nombre de documents dans la collection.

:::{note}
Si vous exécutez la commande `db.students.find()`, vous verrez qu'un champ a été ajouté au document. Dans MongoDB, chaque document stocké dans une collection doit avoir un champ `_id` unique, qui sert de clé primaire. Si un document ne contient pas de champ `_id` à l'insertion, MongoDB génère automatiquement un `ObjectId`.  
:::

La commande pour **insérer plusieurs documents** JSON dans une collection et créer la collection si elle n'existe pas est : `db.<collection>.insertMany(<document_array>)`. Les documents doivent être contenus dans une liste. 

````{admonition} Example
:class: tip
```
db.students.insertMany([{
                         "surname":"Christie",
                         "firstname":"Agatha",
                         "gender":"F",
                         "age":21,
                         "enrolled":true,
                         "status":"alternance",
                         "entreprise":"Éditions du Masque",
                         "grades":{
                                   "literature":[16.3,14.1,17],
                                   "forensic science":[12.3,18]
                                  }
                        },
                        {
                         "surname":"Fogg",
                         "firstname":"Phileas",
                         "gender":"M",
                         "age":24,
                         "enrolled":true,
                         "status":"Erasmus",
                         "country":"Royaume-Uni",
                         "city":"Londres",
                         "university":"Imperial College",
                         "dates":{
                                  "from":ISODate("1872-10-02"),
                                  "to":ISODate("1872-12-21")
                                 }
                        }
                       ])
```
````
À travers l'exemple précédent, vous pouvez constater que **MongoDB prend en charge divers types de données** : du texte, des nombres entiers ou décimaux, des booléens, des dates au format ISODate, mais aussi des **documents imbriqués** et des **listes**, qui permettent de regrouper des informations secondaires.

## Exercice

Une entreprise souhaite se doter d'une base de données dans laquelle elle stockera les échanges entre les clients et le service client, que ce soit par téléphone ou par mail. Elle souhaite également stocker les informations sur les produits qu'elle vend afin de répondre au mieux aux demandes des clients.   
Créez une base de données adaptée, contenant deux collections : `Client` et `Produit`. Stockez les informations suivantes sous la forme de documents JSON dans les collections appropriées. Attention ! Chaque bloc ne correspond pas obligatoirement à un document unique, vous devez construire les documents en fonction des besoins de l'entreprise. Vous pouvez retravailler l'information.

````{admonition} Tip
:class: note
Utilisez un bloc note pour construire vos documents, puis copiez-collez dans MongoDB.
````

::::{grid}
:gutter: 2

:::{grid-item-card}
**De** : Martin Louviers louviers.martin@email.com   
**À** : Service client    
**Date** : 16 août 2025     
**Objet** : Article défectueux — #B88402

Bonjour,  
Le blender reçu fuit au niveau du socle. Puis-je l'échanger ?

Cordialement,  
*Martin Louviers*
:::
:::{grid-item-card}
**De** : Hugo - Service client sav@shoponline.com  
**À** : Martin Louviers  
**Date**: 19 août 2025    
**Objet** : RE: Article défectueux — #B88402

Bonjour,  
Nous vous envoyons dès aujourd'hui un nouveau blender MPG-300 à l'adresse indiquée lors de l'achat.  
Vous trouverez ci-joint un bon de retour prépayé pour le produit défectueux.  
Aucune démarche supplémentaire n’est requise.  

Merci de votre signalement,   
*Hugo — Service client*
:::
::::

````{card}
**Réf** : MPG-300 — **Prix** : 89,90 €   

**Les Points clés**
- 600 W
- 2 vitesses --- variateur de vitesse
- Capacité totale : 2,00 L --- Verre
- Élements compatibles lave-vaisselle

**Pièces** : base moteur, lame inox, bol avec poignée, couvercle avec bouchon doseur.

**SAV** : Garantie 2 ans.
````

````{card}

**Date** : 29 août 2025   

**Heure** : 09:42–09:45 (CEST, UTC+2) — **Durée** : 3 min  

**Canal** : Téléphone (entrant)   

**Agent** : Lina (SAV) — **Client** : Camille Dalida  

**Commande** : #D44120 — Ticket : TCK-2025-08-29-0942  

**Transcription**

**[09:42:03] Client** : Bonjour, j’appelle pour ma commande #D44120. Elle devait arriver hier mais je n’ai rien reçu.   
**[09:42:18] SAV** : Bonjour Camille, je vérifie.   
**[09:42:20] SAV** : Oui, le colis est en transit. La nouvelle livraison est prévue pour demain avant 18:00.  
**[09:43:02] Client** : Très bien. Puis-je recevoir une notif quand le livreur approche ?  
**[09:43:15] SAV** : Bien sûr. J'active le SMS de suivi. Vous devriez recevoir un mail de confirmation dans quelques minutes.  
**[09:44:28] Client** : Merci pour votre aide.  
**[09:44:40] SAV** : Avec plaisir ! Bonne journée.  

**Suivi**    
  
**[09:44:55] Action agent** :
- Alerte SMS activée  
- Mail récapitulatif envoyé  

````

```{admonition} En option
:class: dropdown

::::{grid}
:gutter: 2

:::{grid-item-card}       
**De** : Sophie Pereira sophie.pereira@email.com  
**À** : Service client  
**Objet** : Code promo non appliqué — #C11277  

Bonjour,  
Le code WELCOME10 n’a pas été pris en compte. Pouvez-vous ajuster la facture ?

Merci,  
*Sophie Pereira*
:::
:::{grid-item-card}
**De** : Mehdi - Service client sav@shoponline.com  
**À** : Sophie Pereira  
**Objet** : RE: Code promo non appliqué — #C11277  

Bonjour,  
Après analyse de votre demande, nous vous informons que vous serez remboursé.e de 5.90e par virement bancaire, soit une remise de 10%. 
Cette somme sera créditée sous 3 jours sur le compte que vous avez indiqué.

Bien à vous,  
*Mehdi — Service client*
:::
::::
```

## En résumé

Dans ce chapitre vous avez appris que :
- MongoDB est une **base de données NoSQL orientée documents**.
- Les **documents** constituent la **brique de base** de la base de données.
- Les documents stockent les données sous forme de **paires clé-valeur** (dans un format type JSON).
- MongoDB offre un **schéma flexible** : 
  - Vous pouvez commencer à **stocker des données dès la création de la base de données**, sans définir la structure de la base de données au préalable.
  - Chaque document peut avoir une **structure propre**.
  - Il est possible de stocker des **données structurées** (ex: nom, age, numéro de téléphone), **non structurées** (ex: transcription d'une conversation téléphonique), et d'une **complexité variable** (ex: document imbriqué, liste).
- Les documents présentant des similarités sur le plan structurel ou conceptuel sont regroupés en **collections**.

Dans ce chapitre vous avez manipulé les fonctions suivantes :

Objectif | Syntaxe
--- | ---
Lister les bases de données présentes sur le serveur | `show dbs`  
Créer ou sélectionner une base de données | `use <database>`  
Créer explicitement une collection | `db.createCollection("<collection>")`  
Lister l'ensemble des collections d'une base de données | `show collections`  
Ajouter un document dans une collection | `db.<collection>.insertOne(<document>)`  
Ajouter plusieurs documents dans une collection | `db.<collection>.insertMany(<document_array>)`  
Afficher le nombre de documents dans une collection | `db.<collection>.countDocuments()`  

Vous savez également importer des données dans MongoDB avec `mongoimport`.



:::{warning}
N'oubliez pas d'arrêtez MongoDB quand vous avez terminé !
````{tab} Windows
(à venir)
````
````{tab} Linux
```console 
sudo systemctl stop mongod
```
````
:::
