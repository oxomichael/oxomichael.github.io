---
title: "Guide Complet : Migrer un Dépôt SVN vers Git"
date: "2016-01-23T19:00:00Z"
summary: "Un guide pas à pas pour migrer un projet de Subversion (SVN) vers Git, en conservant l'historique des commits et les auteurs."
tags: ["git", "svn", "migration", "devops"]
translationKey: "2016-01-23-migrate-svn-to-git"
---

Migrer un projet de Subversion (SVN) vers Git est une étape clé pour moderniser ses processus de développement. Git offre des avantages majeurs comme un modèle de branches plus flexible, un travail en mode déconnecté et de meilleures performances.

Ce guide détaille les étapes pour une migration propre et complète.

### Prérequis

Assurez-vous que `git` et son extension `git-svn` sont installés sur votre système.

```bash
# Sur Debian/Ubuntu
sudo apt-get update && sudo apt-get install git-svn
```

### Étape 1 : Préparer le mapping des auteurs

SVN n'enregistre que le nom d'utilisateur pour chaque commit. Git, lui, attend un nom complet et une adresse e-mail. Nous allons créer un fichier de correspondance.

Exécutez cette commande à la racine de votre projet SVN pour extraire la liste des auteurs :

```bash
svn log --xml | grep author | sort -u | perl -pe 's/.*>(.*?)<.*/$1 = /' > users.txt
```

Ouvrez le fichier `users.txt` et complétez-le pour qu'il respecte le format `Nom d'Utilisateur SVN = Prénom Nom <email@example.com>` :

```ini
# Fichier users.txt
oxomichael = Michael Oxo <michael@example.com>
anotheruser = Another User <another@example.com>
```

### Étape 2 : Cloner le dépôt SVN avec `git-svn`

Maintenant, nous allons cloner le dépôt SVN dans un nouveau dépôt Git local. Cette commande va chercher chaque révision SVN et la transformer en un commit Git, en utilisant le fichier `users.txt` pour attribuer les auteurs correctement.

*   `--stdlayout` : Indique que votre dépôt SVN a une structure standard (`trunk`, `branches`, `tags`).
*   `--no-metadata` : Empêche d'ajouter les métadonnées SVN dans les messages de commit Git.
*   `--authors-file` : Spécifie le fichier de mapping des auteurs.

```bash
git svn clone --stdlayout --no-metadata --authors-file=users.txt http://[PATH_SVN_DU_PROJET] [NOM_DU_PROJET_LOCAL]
```
Cette opération peut être très longue pour les dépôts avec un historique important.

### Étape 3 : Nettoyer et convertir les références

L'outil `git-svn` crée des branches et des tags distants qui ne sont pas des branches et tags Git natifs. Nous devons les convertir.

Placez-vous dans le dossier du projet nouvellement créé (`cd [NOM_DU_PROJET_LOCAL]`) et exécutez les commandes suivantes.

**1. Convertir les tags SVN en tags Git :**

```bash
git for-each-ref refs/remotes/origin/tags | cut -d / -f 5- | grep -v @ | while read tagname; do git tag "$tagname" "refs/remotes/origin/tags/$tagname"; done
```

**2. Convertir les branches distantes SVN en branches locales Git :**

```bash
git for-each-ref refs/remotes/origin | cut -d / -f 4- | grep -v 'trunk' | while read branchname; do git branch "$branchname" "refs/remotes/origin/$branchname"; done
```

**3. Nettoyer les anciennes références :**

Maintenant que les branches et les tags sont convertis, nous pouvons supprimer les références créées par `git-svn`.

```bash
git branch -r -d origin/trunk
git branch -r -d $(git branch -r | grep "origin/tags")
git branch -r -d $(git branch -r | grep "origin/[^t]") # Supprime les branches restantes sauf trunk
```

### Étape 4 : Lier le dépôt Git local à un dépôt distant

Il est temps de connecter votre dépôt local à un nouveau dépôt Git distant (sur GitHub, GitLab, etc.).

```bash
git remote add origin git@[PATH_GIT_DU_PROJET]
```

### Étape 5 : Pousser le projet vers le nouveau dépôt Git

Envoyez tout votre historique, vos branches et vos tags vers le nouveau serveur Git.

```bash
# Pousser toutes les branches
git push origin --all

# Pousser tous les tags
git push origin --tags
```

Votre projet est maintenant entièrement migré sur Git ! Vous pouvez archiver le dépôt SVN.
