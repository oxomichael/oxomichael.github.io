Migrate Git to Another Git
Soumis par michael le jeu, 04/02/2016 - 20:59

Migration d'un Git vers un autre Git ou aussi comment faire un backup

Tout est très simple

On clone tout le projet du le serveur de versioning dans un répertoire
1

$ git clone --bare oldserver/repo.git

Hop un saut dans le répertoire
1

$ cd *repo.git*

Et on pousse le tout sur le nouveau serveur
1

$ git push --mirror newserver/user/repo.git

Voilà.
