---
translationKey: "2010-01-22-php-script-convert-file-to-utf8"
title: "Convert a Directory to UTF-8 with PHP"
description: "A PHP script to recursively convert the encoding of files in a directory to UTF-8, with no external dependencies."
date: '2010-01-22T11:00:00+02:00'
draft: false
---

It is sometimes necessary to convert the encoding of a large number of files, for example, when migrating an old project. `UTF-8` encoding is the standard today, but many older files may use encodings like `ISO-8859-1` or `Windows-1252`.

Here is a modern, standalone PHP script that recursively scans a source directory, detects the encoding of each file, and converts it to `UTF-8` in a destination directory. Unlike the old version that relied on shell commands (`file`, `iconv`), this script exclusively uses native PHP extensions (`mbstring`, `SPL`), making it more portable and reliable.

### The Conversion Script

This script uses the `RecursiveDirectoryIterator` and `RecursiveIteratorIterator` classes to traverse the file tree, which is much more efficient than the manual approach. Encoding detection and conversion are handled by the `mbstring` extension.

```php
#!/usr/bin/env php
<?php

/**
 * Recursively converts files in a directory to UTF-8.
 *
 * @param string $sourceDir The source directory.
 * @param string $destDir   The destination directory.
 */
function convertDirectoryToUtf8(string $sourceDir, string $destDir): void
{
    if (!is_dir($sourceDir)) {
        throw new InvalidArgumentException("Source directory does not exist: $sourceDir");
    }

    if (!is_dir($destDir) && !mkdir($destDir, 0755, true)) {
        throw new RuntimeException("Could not create destination directory: $destDir");
    }

    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($sourceDir, RecursiveDirectoryIterator::SKIP_DOTS),
        RecursiveIteratorIterator::SELF_FIRST
    );

    echo "Starting conversion...\n";

    foreach ($iterator as $file) {
        $sourcePath = $file->getPathname();
        $destPath = $destDir . DIRECTORY_SEPARATOR . $iterator->getSubPathName();

        if ($file->isDir()) {
            if (!is_dir($destPath)) {
                mkdir($destPath, 0755, true);
            }
        } else {
            // Detect file encoding
            $content = file_get_contents($sourcePath);
            $encoding = mb_detect_encoding($content, ['UTF-8', 'ISO-8859-1', 'Windows-1252', 'ASCII'], true);

            if ($encoding && $encoding !== 'UTF-8') {
                // Convert content to UTF-8
                $contentUtf8 = mb_convert_encoding($content, 'UTF-8', $encoding);
                file_put_contents($destPath, $contentUtf8);
                echo "Converted: $sourcePath ($encoding -> UTF-8)\n";
            } else {
                // If already in UTF-8 or encoding not detected, just copy the file
                copy($sourcePath, $destPath);
                echo "Copied (unchanged): $sourcePath\n";
            }
        }
    }

    echo "Conversion finished.\n";
}

// --- Program Start ---

if ($argc < 3) {
    echo "Usage: php " . basename(__FILE__) . " <source_directory> <destination_directory>\n";
    exit(1);
}

$sourceDirectory = $argv[1];
$destinationDirectory = $argv[2];

try {
    convertDirectoryToUtf8($sourceDirectory, $destinationDirectory);
} catch (Exception $e) {
    echo 'Error: ' . $e->getMessage() . "\n";
    exit(1);
}
```

### How to Use It

1.  **Save the script**: Copy the code above and save it to a file named `convert-utf8.php`.
2.  **Ensure the `mbstring` extension is enabled** in your PHP installation. This is usually the case by default. You can check with the command `php -m | grep mbstring`.
3.  **Run the script** from your terminal, specifying the source and destination directories:

```bash
php convert-utf8.php /path/to/your/project /path/to/project-utf8
```

The script will create the destination directory and copy the entire project structure into it, ensuring that all text files are correctly encoded in `UTF-8`.

```