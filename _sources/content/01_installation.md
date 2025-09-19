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

# Installation de MongoDB

  
## MongoDB Community Edition

Suivez les étapes suivantes pour télécharger et installer MongoDB Community Edition.
   
::::::{tab-set}
:sync-group: category

:::::{tab-item} Windows
:sync: key1

(à venir)
:::::

:::::{tab-item} Linux - Ubuntu
:sync: key2

**Traduction de** : [mongodb.com](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/)

Dans un terminal, exécutez les commandes suivantes.

```console
sudo apt-get install gnupg curl

```
1. Importer la clé GPG publique MongoDB

```console
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
```
2. Créer le fichier `/etc/apt/sources.list.d/mongodb-org-8.0.list`

Vérifiez votre version Ubuntu dans les Paramètres ou tapez `lsb_release -a` dans un terminal. Sélectionnez la commande appropriée.

````{tab} Ubuntu 24.04 (Noble)
```console
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list￼
```
````
````{tab} Ubuntu 22.04 (Jammy)
```console
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```
````
````{tab} Ubuntu 20.04 (Focal)
```console
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```
````

3. Mettre à jour la liste des packages

```console
sudo apt-get update
```
4. Installer MongoDB Community Server

```console
sudo apt-get install -y mongodb-org
```

:::::

::::::

## MongoDB Shell (mongosh)

MongoDB Shell est un outil en ligne de commande qui permet d'intéragir avec les bases de données MongoDB.

::::::{tab-set}
:sync-group: category

:::::{tab-item} Windows
:sync: key1

MongoDB Shell (`mongosh`) n'est pas installé par défaut avec MongoDB Community Server. Suivez les instructions suivantes pour télécharger et installer mongosh séparément.

(à venir)
:::::

:::::{tab-item} Linux - Ubuntu
:sync: key2
MongoDB Shell (mongosh) est installé par défaut avec MongoDB Community Server. Vous n'avez rien à faire !
:::::

::::::

## Visual Studio Code

Nous allons travailler dans Visual Studio Code. Dans l'onglet Extensions, téléchargez "MongoDB for VS Code". Une fois l'installation terminée, vous verrez apparaître une icône feuille à gauche de la fenêtre. Cliquez dessus pour commencer à travailler avec MongoDB.

## Vérification

Afin de vérifier que tout fonctionne bien, suivez les étapes suivantes. 

::::::{tab-set}
:sync-group: category

:::::{tab-item} Windows
:sync: key1

(à venir)
:::::

:::::{tab-item} Linux - Ubuntu
:sync: key2

**Traduction de** : [mongodb.com](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/)

Vérifiez quel programme *init* votre système utilise (systemd ou System V init) :
```console
ps --no-headers -o comm 1
``` 

Sélectionnez l'onglet approprié. Exécutez les commandes suivantes dans un terminal.

````{tab} systemd (systemctl)

1. Démarrez MongoDB
```console
sudo systemctl start mongod
```
Si l'erreur ```Failed to start mongod.service: Unit mongod.service not found.``` apparaît, exécutez la commande suivante puis exécutez de nouveau la première commande :
```console
sudo systemctl daemon-reload
```

2. Vérifiez que MongoDB a démarré correctement
```console
sudo systemctl status mongod
```

3. Arrêtez MongoDB
```console
sudo systemctl stop mongod
```

4. Démarrez une session `mongosh`  

Pour démarrez une session `mongosh`, vous pouvez exécuter `mongosh` sans aucune option (par défaut : localhost, port 27017):
```console
sudo systemctl start mongod
mongosh
```
````
````{tab} System V init (service)

1. Démarrez MongoDB
```console
sudo service mongod start
```

2. Vérifiez que MongoDB a démarré correctement
```console
sudo service mongod status
```

3. Arrêtez MongoDB
```console
sudo service mongod stop
```

4. Démarrez une session `mongosh`  

Pour démarrez une session `mongosh`, vous pouvez exécuter `mongosh` sans aucune option (par défaut : localhost, port 27017):
```console
sudo service mongod start
mongosh
```
````
:::::

::::::
