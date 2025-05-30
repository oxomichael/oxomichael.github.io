---
categories: git svn migrate
date: "2016-01-23T19:00:00Z"
lang: fr
ref: 2016-01-23-migrate-svn-to-git
title: Migration SVN vers Git
---

Afin d'améliorer le processus de mise production, pour mon usage, il est intéressant de passer tous les projets sur Git.

Bon on va commencer par consulter la doc officielle, https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git
ou pour la v1 (https://git-scm.com/book/en/v1/Git-and-Other-Systems-Migrating-to-Git)

Voici donc les différentes étapes pour migrer un projet de Subversion vers Git

1. Créer un fichier
```
$ svn log [PATH] --xml | grep author | sort -u | perl -pe 's/.*>(.*?)<.*/$1 = /' > users.txt
```

2. Modifier le fichier pour obtenir sur chaque pseudo quelque chose de cohérent pout Git
```
pseudo = prénom nom
```

3. Cloner le projet SVN à partir de git-svn
```
$ git svn clone --prefix=origin/ --stdlayout http://[PATH SVN DU PROJET] --authors-file=users.txt --no-metadata -s [PROJET]
```
Cette opération va créer un repository local Git

4. Nettoyage des références créer par la commande git-svn
```
$ git for-each-ref refs/remotes/tags | cut -d / -f 4- | grep -v @ | while read tagname; do git tag "$tagname" "tags/$tagname"; git branch -r -d "tags/$tagname"; done
$ git for-each-ref refs/remotes | cut -d / -f 3- | grep -v @ | while read branchname; do git branch "$branchname" "refs/remotes/$branchname"; git branch -r -d "$branchname"; done
$ cp -Rfv .git/refs/remotes/origin/tags/* .git/refs/tags/
$ rm -Rfv .git/refs/remotes/origin/tags
$ cp -Rfv .git/refs/remotes/* .git/refs/heads/
$ rm -Rfv .git/refs/remotes
```

- Affichage des branches  
```git branch -a```

- Suppression des éléments qui ne vous plaisent plus  
```git branch -d origin/trunk```
ou
```git branch -D origin/[name]```

5. Ajouter la référence au serveur Git
```
$ git remote add origin git@[PATH GIT DU PROJET]
```

6. Pousser le tout et vérifier que tout s'est bien passé.
```
$ git push origin --all
```
Si les tags ne sont pas poussés alors il existe cette commande, peut être compliqué suivant la structure de vos tags
```
$ git push origin --tags
```
