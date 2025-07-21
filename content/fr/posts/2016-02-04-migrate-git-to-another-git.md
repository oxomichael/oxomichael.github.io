---
title: "Guide : Migrer un Dépôt Git vers un Autre Serveur"
date: "2016-02-04T20:59:00Z"
summary: "Un guide simple et efficace pour migrer un dépôt Git complet, incluant toutes les branches et les tags, d'un serveur à un autre."
tags: ["git", "migration", "devops"]
translationKey: "2016-02-04-migrate-git-to-another-git"
---

Migrer un dépôt Git d'un serveur à un autre est une opération courante, que ce soit pour changer de plateforme (de GitHub à GitLab, par exemple) ou pour consolider des projets. L'objectif est de transférer l'intégralité de l'historique, y compris toutes les branches et les tags.

La méthode `mirror` est la plus fiable et la plus complète pour cette tâche.

### Étape 1 : Cloner le dépôt en mode miroir

La première étape consiste à créer un clone "miroir" de votre dépôt source. Un clone miroir est une copie exacte de la base de données Git, incluant toutes les références internes, les branches distantes et les tags.

```bash
# Remplacez [URL_DU_DEPOT_SOURCE] par l'URL de votre ancien dépôt
git clone --bare [URL_DU_DEPOT_SOURCE]
```

Cette commande crée un dossier se terminant par `.git` (par exemple, `mon-projet.git`).

### Étape 2 : Pousser le miroir vers le nouveau serveur

Ensuite, placez-vous dans le dossier nouvellement créé et poussez le miroir vers le nouveau dépôt. Assurez-vous d'avoir déjà créé un projet vide sur votre nouveau serveur Git pour y recevoir les données.

```bash
# Entrez dans le dossier du clone miroir
cd mon-projet.git

# Remplacez [URL_DU_NOUVEAU_DEPOT] par l'URL de votre nouveau dépôt
git push --mirror [URL_DU_NOUVEAU_DEPOT]
```

La commande `git push --mirror` garantit que toutes les références (branches, tags, etc.) sont poussées vers le nouveau dépôt. C'est la méthode la plus sûre pour une migration complète.

### Étape 3 : Nettoyage

Une fois la migration terminée, vous pouvez supprimer le dossier `.git` local.

```bash
cd ..
rm -rf mon-projet.git
```

Votre dépôt est maintenant entièrement migré sur le nouveau serveur. Vous pouvez cloner le projet depuis sa nouvelle URL et continuer à travailler normalement.
