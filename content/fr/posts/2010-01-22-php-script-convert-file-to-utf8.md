---
translationKey: "2010-01-22-php-script-convert-file-to-utf8"
title: 'Convert file to utf8'
description: "How to convert files to utf8 with a php script"
date: '2010-01-22T11:00:00+02:00'
draft: false
---

Voici un script PHP permettant la conversion de l'encodage des fichiers vers UTF-8.
Il exporte un projet du référentiel SVN dans un dossier, et place les fichiers convertis dans un dossier avec un suffixe ".utf8".

```php
#!/usr/bin/php
function recursive_readdir ($dir)
{
  global $dir_in, $dir_out;
  $dir = rtrim ($dir, '/');
  if (is_dir ($dir)){
    $dh = opendir ($dir);
  } else {
    echo $dir, ' n\'est pas un repertoire valide';
    exit;
  }
  while (($file = readdir ($dh)) !== false) {
    if ($file !== '.' && $file !== '..') {
      $path =$dir.'/'.$file;
      $path_out = str_replace($dir_in, $dir_out, $path);
      if (is_dir ($path)) {
        if (!file_exists($path_out)) mkdir($path_out);
        recursive_readdir ($path);
      }
      else {
        $encoding = exec("file -b --mime-encoding $path");
        if($encoding=='iso-8859-1') {
          $cmd = "iconv -o $path_out -f ISO8859-1 -t UTF-8 $path";
        }
        elseif($encoding=='us-ascii') {
          $cmd = "iconv -o $path_out -f US-ASCII -t UTF-8 $path";
        } else {
          $cmd = "cp $path $path_out";
        }
        echo $cmd, "\n";
        passthru($cmd);
      }
    }
  }
  closedir ($dh);
}

// => Début du programme
$dir_in = '/home/chemin/vers/dossier';
$dir_out = '/home/chemin/vers/dossier.utf8';
exec("svn export file:///svn/projet/trunk $dir_in/");
if (!file_exists($dir_in)) {
  exit;
}
if (!file_exists($dir_out)) {
  mkdir($dir_out);
}
recursive_readdir($dir_in);
```
