---
title: 'Generate a password in PHP'
date: '2007-02-23T11:00:00+02:00'
draft: false
---

> **Note de mise à jour (Juillet 2025) :** Cet article est conservé pour l'archive. Les méthodes présentées ici (`rand()`, `srand()`) ne sont plus considérées comme sécurisées. Pour une approche moderne et sûre, veuillez consulter la version mise à jour de cet article :
> **[Générer un mot de passe sécurisé en PHP (Version 2025)](/posts/2025-07-08-generer-mot-de-passe-securise-php)**

---

La sécurisation d'un mot de passe dépend du nombre de caractères différents présent dans celui-ci.
Voici une solution pour rendre les mots de passe sécurisé lors de leur génération:
code :

```php
function random_password($length, $strength) {
  $char_sets = array('48-57','65-90','97-122','35-38','61-64');
  $new_password = '';
  srand(microtime()*10000000);
  for ($i=0; $i < $length; $i++) {
    $random = rand(0, $strength-1);
    list($start, $end) = explode('-', $char_sets[$random]);
    $new_password.= chr(rand($start, $end));
 }
 return($new_password);
}
```

Cette fonction prend comme paramètres `$strength` et `$length`.
La variable `$strength` change le nombre de caractères affectés à la taille du mot de pass :

    $strength = 1:- 0-9
    $strength = 2:- A-Z0-9
    $strength = 3:- A-Za-z0-9
    $strength = 4:- A-Za-z0-9 and # $ % &
    $strength = 5:- A-Za-z0-9 and # $ % & = > ? @

Pour des mots de passe sécurisé, utiliser `$strength = 5`, et pour un mot de passe facilement décriptable utiliser `$strength = 1`.

`$length` défini la longueur du mot de passe.

Explication du code

`$char_sets = array('48-57','65-90','97-122','35-38','61-64');`

`$char_sets` est une table de nombre ascii qui détermine quel seront les caractères inclut dans le mot de passe.

```php
$new_password='';
srand(microtime()*10000000);
```
2 lignes d'initialisation.
`$new_password` sera le mot de pass final.
`srand()` est nécessaire pour initialiser le générateur de nombre aléatoire. L'initialisation se fait par la fonction `microtime()`.
```php
for ($i=0; $i < $length; $i++) {
  $random = rand(0, $strength-1);
  list($start,$end) = explode('-',$char_sets[$random]);
  $new_password.= chr(rand($start,$end));
}
```
Ici on genere aléatoirement les caractères en utilisant la valeur $strength pour déterminer quel caractères seront inclus.
`$random = rand(0,$strength-1);`

Genere un nombre aléatoirement compris entre 0 et strength - 1 ( en relation avec la table $char_sets ).
`list($start,$end) = explode('-',$char_sets[$random]);`

Maintenant sélectionner les caractères choisis et faire un explode() des valeurs.
`$start` et `$end` sont les valeurs haute et basse  de la table de caractères.

`$new_password.= chr(rand($start,$end));`

Et sont finalement affecter à `$new_password`

On utilise `rand()` pour générer un nombre aléatoire entre la valeur haute et basse et ensuite le convertir en caractère en utilisant `chr()`.
Ensuite on répète ce processus suivant la longueur défini pas `$length`.
Et l'on retourne le mot de passe.

Utilisation

```php
echo random_password(10,1);
// would give a result like 4173967312
echo random_password(10,2);
// would give a result like HY9J3X5IF0
echo random_password(10,3);
// would give a result like EKc664aqs4
echo random_password(10,4);
// would give a result like ZY$NP26T6#
echo random_password(10,5);
// would give a result like T$@p=841=b
```