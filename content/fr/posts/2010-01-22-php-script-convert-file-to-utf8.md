---
translationKey: "2010-01-22-php-script-convert-file-to-utf8"
title: "Convertir un répertoire en UTF-8 avec PHP"
description: "Un script PHP pour convertir récursivement l'encodage des fichiers d'un répertoire vers UTF-8, sans dépendances externes."
date: '2010-01-22T11:00:00+02:00'
draft: false
---

Il est parfois nécessaire de convertir l'encodage d'un grand nombre de fichiers, par exemple lors de la migration d'un ancien projet. L'encodage `UTF-8` est aujourd'hui la norme, mais de nombreux fichiers plus anciens peuvent utiliser des encodages comme `ISO-8859-1` ou `Windows-1252`.

Voici un script PHP moderne et autonome qui parcourt récursivement un répertoire source, détecte l'encodage de chaque fichier et le convertit en `UTF-8` dans un répertoire de destination. Contrairement à l'ancienne version qui reposait sur des commandes shell (`file`, `iconv`), ce script utilise exclusivement les extensions natives de PHP (`mbstring`, `SPL`), le rendant plus portable et plus fiable.

### Le script de conversion

Ce script utilise les classes `RecursiveDirectoryIterator` et `RecursiveIteratorIterator` pour traverser l'arborescence des fichiers, ce qui est bien plus efficace que l'approche manuelle. La détection et la conversion de l'encodage sont gérées par l'extension `mbstring`.

```php
#!/usr/bin/env php
<?php

/**
 * Convertit récursivement les fichiers d'un répertoire en UTF-8.
 *
 * @param string $sourceDir Le répertoire source.
 * @param string $destDir   Le répertoire de destination.
 */
function convertDirectoryToUtf8(string $sourceDir, string $destDir): void
{
    if (!is_dir($sourceDir)) {
        throw new InvalidArgumentException("Le répertoire source n'existe pas : $sourceDir");
    }

    if (!is_dir($destDir) && !mkdir($destDir, 0755, true)) {
        throw new RuntimeException("Impossible de créer le répertoire de destination : $destDir");
    }

    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($sourceDir, RecursiveDirectoryIterator::SKIP_DOTS),
        RecursiveIteratorIterator::SELF_FIRST
    );

    echo "Début de la conversion...\n";

    foreach ($iterator as $file) {
        $sourcePath = $file->getPathname();
        $destPath = $destDir . DIRECTORY_SEPARATOR . $iterator->getSubPathName();

        if ($file->isDir()) {
            if (!is_dir($destPath)) {
                mkdir($destPath, 0755, true);
            }
        } else {
            // Détecter l'encodage du fichier
            $content = file_get_contents($sourcePath);
            $encoding = mb_detect_encoding($content, ['UTF-8', 'ISO-8859-1', 'Windows-1252', 'ASCII'], true);

            if ($encoding && $encoding !== 'UTF-8') {
                // Convertir le contenu en UTF-8
                $contentUtf8 = mb_convert_encoding($content, 'UTF-8', $encoding);
                file_put_contents($destPath, $contentUtf8);
                echo "Converti : $sourcePath ($encoding -> UTF-8)\n";
            } else {
                // Si déjà en UTF-8 ou encodage non détecté, copier simplement le fichier
                copy($sourcePath, $destPath);
                echo "Copié (inchangé) : $sourcePath\n";
            }
        }
    }

    echo "Conversion terminée.\n";
}

// --- Début du programme ---

if ($argc < 3) {
    echo "Usage: php " . basename(__FILE__) . " <répertoire_source> <répertoire_destination>\n";
    exit(1);
}

$sourceDirectory = $argv[1];
$destinationDirectory = $argv[2];

try {
    convertDirectoryToUtf8($sourceDirectory, $destinationDirectory);
} catch (Exception $e) {
    echo 'Erreur : ' . $e->getMessage() . "\n";
    exit(1);
}
```

### Comment l'utiliser ?

1.  **Enregistrez le script** : Copiez le code ci-dessus et enregistrez-le dans un fichier nommé `convert-utf8.php`.
2.  **Assurez-vous que l'extension `mbstring` est activée** dans votre installation de PHP. C'est généralement le cas par défaut. Vous pouvez vérifier avec la commande `php -m | grep mbstring`.
3.  **Exécutez le script** depuis votre terminal en spécifiant les répertoires source et destination :

```bash
php convert-utf8.php /chemin/vers/votre/projet /chemin/vers/projet-utf8
```

Le script créera le répertoire de destination et y copiera l'intégralité de l'arborescence de votre projet, en s'assurant que tous les fichiers texte sont bien encodés en `UTF-8`.

