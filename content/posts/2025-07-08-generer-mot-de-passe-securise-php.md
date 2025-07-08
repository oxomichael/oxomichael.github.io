---
title: 'Générer un mot de passe sécurisé en PHP (Version 2025)'
date: '2025-07-08T10:00:00+02:00'
draft: false
---

La génération de mots de passe robustes est un élément fondamental de la sécurité des applications. Un mot de passe fort doit être long et contenir une combinaison de différents types de caractères.

Une approche dépassée, souvent vue dans d'anciens tutoriels, utilisait des fonctions comme `rand()` et `srand()`. Celles-ci ne sont **pas** considérées comme suffisamment sécurisées pour des tâches cryptographiques. Voici une solution moderne et sûre pour générer des mots de passe en PHP.

### La fonction moderne : `generateStrongPassword`

Cette nouvelle fonction utilise `random_int()`, qui est conçue en PHP pour générer des entiers aléatoires cryptographiquement sûrs. Elle est également beaucoup plus flexible et lisible.

```php
/**
 * Génère un mot de passe sécurisé.
 *
 * @param int  $length Longueur du mot de passe souhaité.
 * @param bool $includeDigits Inclure des chiffres.
 * @param bool $includeUppercase Inclure des majuscules.
 * @param bool $includeLowercase Inclure des minuscules.
 * @param bool $includeSymbols Inclure des symboles.
 * @return string Le mot de passe généré.
 * @throws \Exception Si la génération de nombres aléatoires sécurisés échoue.
 */
function generateStrongPassword(
    int $length = 12,
    bool $includeDigits = true,
    bool $includeUppercase = true,
    bool $includeLowercase = true,
    bool $includeSymbols = true
): string {
    $characterSets = [];
    if ($includeDigits) {
        $characterSets[] = '0123456789';
    }
    if ($includeUppercase) {
        $characterSets[] = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    }
    if ($includeLowercase) {
        $characterSets[] = 'abcdefghijklmnopqrstuvwxyz';
    }
    if ($includeSymbols) {
        // Vous pouvez personnaliser cette liste de symboles
        $characterSets[] = '!@#$%^&*()_+-=[]{}|';
    }

    if (empty($characterSets)) {
        throw new \InvalidArgumentException('Au moins un jeu de caractères doit être sélectionné.');
    }

    $password = '';
    $allCharacters = implode('', $characterSets);

    // Garantit au moins un caractère de chaque jeu demandé pour la robustesse
    foreach ($characterSets as $set) {
        $password .= $set[random_int(0, strlen($set) - 1)];
    }

    // Complète le reste du mot de passe
    $remainingLength = $length - count($characterSets);
    if ($remainingLength > 0) {
        for ($i = 0; $i < $remainingLength; $i++) {
            $password .= $allCharacters[random_int(0, strlen($allCharacters) - 1)];
        }
    }

    // Mélange le mot de passe pour que les premiers caractères ne soient pas prévisibles
    return str_shuffle($password);
}
```

### Pourquoi cette approche est-elle meilleure ?

1.  **Sécurité accrue** : `random_int()` est conçu pour les besoins cryptographiques, contrairement à `rand()` qui produit des nombres prévisibles et ne doit jamais être utilisé pour la sécurité.
2.  **Flexibilité** : Au lieu d'un paramètre `$strength` peu intuitif, la nouvelle fonction utilise des booléens explicites (`$includeDigits`, `$includeUppercase`, etc.). Cela rend le code plus lisible et plus facile à utiliser.
3.  **Robustesse garantie** : La fonction s'assure que si vous demandez des chiffres, des majuscules et des symboles, le mot de passe contiendra *au moins un* caractère de chaque type demandé, avant de compléter le reste.
4.  **Code moderne** : L'utilisation du typage strict (`int`, `bool`, `string`) et des exceptions rend la fonction plus robuste et intégrable dans des projets modernes.

### Explication du code

1.  **`$characterSets`**: Un tableau qui contiendra les chaînes de caractères possibles (chiffres, majuscules, etc.) en fonction des paramètres.
2.  **`random_int(min, max)`**: La pierre angulaire de notre fonction. Elle génère un nombre aléatoire de haute qualité entre `min` et `max`.
3.  **Garantie des jeux de caractères**: La première boucle `foreach` parcourt les jeux de caractères demandés et ajoute un caractère aléatoire de chaque jeu au mot de passe. Cela évite d'avoir un mot de passe qui, par malchance, ne contiendrait que des minuscules alors que vous en aviez demandé d'autres.
4.  **Remplissage**: La boucle `for` suivante complète le mot de passe jusqu'à la longueur désirée en piochant aléatoirement dans l'ensemble de tous les caractères autorisés.
5.  **`str_shuffle()`**: Cette dernière étape est cruciale. Elle mélange la chaîne de caractères finale pour s'assurer que les premiers caractères (ceux garantis par la première boucle) ne suivent pas un ordre prévisible (par exemple, toujours un chiffre, puis une majuscule, etc.).

### Exemples d'utilisation

```php
// Mot de passe de 16 caractères avec tous les types de caractères (par défaut)
echo generateStrongPassword(16);
// Résultat possible : j!6P@sWqK_zR{gE8

// Mot de passe simple de 8 caractères, avec seulement des minuscules et des chiffres
echo generateStrongPassword(8, true, false, true, false);
// Résultat possible : 4p7k1n9z

// Mot de passe de 10 caractères sans symboles
echo generateStrongPassword(10, true, true, true, false);
// Résultat possible : 9aB4rT2zK1
```

En adoptant cette approche moderne, vous améliorez significativement la sécurité de la génération de mots de passe dans vos applications PHP.
