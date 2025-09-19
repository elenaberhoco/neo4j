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

# MongoDB : Base de données NoSQL orientée documents

Ce cours est une introduction à MongoDB, une base de données NoSQL orientée documents.

Il est organisé selon les quatre grandes catégories d'interaction que l'on peut avoir avec une base de données, résumées par l'acronyme **CRUD** :
- **C**reate (Créer) : Toute opération qui consiste à ajouter de nouveaux enregistrements à la base de données.
- **R**ead (Lire) : Toute opération qui consiste à récupérer certaines informations en fonction des critères de recherche spécifiés.
- **U**pdate (Mettre à jour) : Toute opération qui consiste à modifier des enregistrements de la base de données.
- **D**elete (Supprimer) : Toute opération qui consiste à supprimer des enregistrements de la base de données, voire la base de données elle-même.
