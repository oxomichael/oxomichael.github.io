---
layout: post
categories: git migrate
date: 2016-02-04 20:59:00 +0200
lang: fr
ref: 2016-02-04-migrate-git-to-another-git
title: "Migrate Git to another Git"
---

... or how to make a backup.

All is simple

Tout est très simple

Clone your project in a local directory
```
$ git clone --bare oldserver/repo.git
```

Enter in the directory created by the clone operation
```
$ cd *repo.git*
```

Push your entire project as a mirror in your new server
```
$ git push --mirror newserver/user/repo.git
```

Voilà.
