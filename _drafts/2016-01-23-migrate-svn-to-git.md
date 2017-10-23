

Migration SVN vers Git
Soumis par michael le sam, 23/01/2016 - 19:12

Afin d'améliorer le processus de mise production, pour mon usage, il est intéressant de passer tous les projets sur Git.

Bon on va commencer par consulter la doc officielle, https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git
ou pour la v1 (https://git-scm.com/book/en/v1/Git-and-Other-Systems-Migrating-to-Git)

Voici donc les différentes étapes pour migrer un projet de Subversion vers Git

1. Créer un fichier
1

$ svn log [PATH] --xml | grep author | sort -u | perl -pe 's/.*>(.*?)<.*/$1 = /' > users.txt

2. Modifier le fichier pour obtenir sur chaque pseudo quelque chose de cohérent pout Git
1

pseudo = prénom nom

3. Cloner le projet SVN à partir de git-svn
1

$ git svn clone --prefix=origin/ --stdlayout http://[PATH SVN DU PROJET] --authors-file=users.txt --no-metadata -s [PROJET]

Cette opération va créer un repository local Git

4. Nettoyage des références créer par la commande git-svn
1

git for-each-ref refs/remotes/tags | cut -d / -f 4- | grep -v @ | while read tagname; do git tag "$tagname" "tags/$tagname"; git branch -r -d "tags/$tagname"; done
1

git for-each-ref refs/remotes | cut -d / -f 3- | grep -v @ | while read branchname; do git branch "$branchname" "refs/remotes/$branchname"; git branch -r -d "$branchname"; done
1
2
3
4

$ cp -Rfv .git/refs/remotes/origin/tags/* .git/refs/tags/
$ rm -Rfv .git/refs/remotes/origin/tags
$ cp -Rfv .git/refs/remotes/* .git/refs/heads/
$ rm -Rfv .git/refs/remotes

- Affichage des branches
1

git branch -a

- Suppression des éléments qui ne vous plaisent plus
1
2
3

git branch -d origin/trunk
#ou
git branch -D origin/[name]

5. Ajouter la référence au serveur Git
1

$ git remote add origin git@[PATH GIT DU PROJET]

6. Pousser le tout et vérifier que tout s'est bien passé.
1

$ git push origin --all

Si les tags ne sont pas poussés alors il existe cette commande, peut être compliqué suivant la structure de vos tags
1

$ git push origin --tags
