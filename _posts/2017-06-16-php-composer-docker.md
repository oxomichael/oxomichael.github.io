---
layout: post
categories: php composer docker
date: 2017-06-16 08:30:00 +0200
lang: fr
ref: 2017-06-16-php-composer-docker
title: "PHP : Composer in docker"
---

Si vous développer en PHP avec tout votre environnement local de développement
en docker, alors vous voudriez bien aussi utilisez les bonnes pratiques de
docker et placer Composer dans un container.

Pour ce faire, il existe donc l'image officiel de Composer sur le hub de docker (https://hub.docker.com/_/composer/).

Voilà comment, je l'utilise :

`docker pull composer`

Ce container sera utilisé en lancement au premier plan et supprimer ensuite, il est inutile de laisser tourner ce container.

Editer le fichier `.bash_aliases` de votre utilisateur et ajouter (N'oubliez pas source `.bashrc` pour informer le système de vos changements)

```
composer () {
    tty=
    tty -s && tty=--tty
    docker run \
        $tty \
        --interactive \
        --rm \
        --user $(id -u):$(id -g) \
        --volume /etc/passwd:/etc/passwd:ro \
        --volume /etc/group:/etc/group:ro \
        --volume $(pwd):/app \
        --volume $HOME/.composer:/composer \
        composer "$@"
}
```

Les différents paramètres `--user` et `--volume` permettent d'être sur d'avoir les droits utilisateurs sur les fichiers. Le dernier `--volume` permet d'utiliser le cache de composer.

Pour installer certaines dépendances ou des scripts Composer, il y a des vérifications qui bloque l'installation. Vous avez une option:

Utiliser `--ignore-platform-reqs` et `--no-scripts` flags pour faire les opérations `install` ou `update`

Et voilà.
